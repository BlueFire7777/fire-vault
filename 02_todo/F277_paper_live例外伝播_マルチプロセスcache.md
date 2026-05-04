---
id: F277
phase: P9: 運用・保守
priority: 最優先
status: 未着手
owner: Fujiwara
depends_on: [F275]
chapter: "13, 17, 20, 35"
created: 2026-05-04
updated: 2026-05-04
---

# F277: paper_live 例外伝播設計 + マルチプロセス stale cache 対策

## 概要

F275 で Codex pre-commit review が指摘した CRITICAL のうち、F275 着手前から
存在する **既存問題 (例外握り潰し)** と F275 では実装スコープを構造的に
超える **マルチプロセス stale cache 対策** を体系的に解消する後続タスク。

Stage 3 移行 (実トレード開始) 前に必須。

## タスク詳細

### F275 で移管された 4 つの Codex CRITICAL

1. **TTL stale window** (F275 で signature SQL 撤廃により writer-side
   invalidation 単独になった) → マルチプロセスで stale 化リスク
2. **features DELETE/UPDATE 検知不能** → patterns 同様 writer-side で
   `reset_similarity_cache()` 呼出が必要
3. **tick.py 例外握り潰し** (`_fetch_ohlc / _fetch_all_symbols / _fetch_symbols_with_features /
   engine.evaluate` で `except Exception: return [] / None` の沈黙)
4. **_TRADE_STATS_CACHE positions invalidation 未実装** (TP/SL/force_close 後の
   stale 化リスク)

### サブスコープ A: tick.py 例外伝播設計

**問題**: SQLite WAL ロック / スキーマ不整合 / features 取得失敗 / engine
評価失敗が「正常な候補なし」と区別できず、Paper Live が無音停止する
リスク。緊急クローズ通知漏れにつながる。

**実装**:
- `_fetch_all_symbols / _fetch_symbols_with_features / _fetch_ohlc /
  engine.evaluate` の `except Exception` を sentinel return から **raise** に変更
- 上位 (`extract_candidates / monitor_virtual_positions / evaluate_force_close`)
  で catch、tick 全体を `status='failed'` で paper_live_runs に記録
- F236 LINE 5 部屋連携: 緊急アラート部屋に「tick 失敗」通知を送る経路追加
- 既存 logger.error() ベース (F275 で追加) は維持、伝播後も記録される

**工数**: 4 時間

### サブスコープ B: positions 書き込み経路からの reset_trade_stats_cache() 統合

**問題**: F275 で `_TRADE_STATS_CACHE` モジュール辞書を追加したが、positions
テーブルへの INSERT/UPDATE 経路で `reset_trade_stats_cache()` が呼ばれない。
TP/SL/force_close 後も win_rate / expected_value が stale のまま次 tick の
判断に使われるリスク。

**実装**:
- PaperLivePositionTracker._insert_position / _update_position 末尾で
  `reset_trade_stats_cache()` を呼出
- 仮想エントリー (evaluate_virtual_entries) / 利確 / 損切 / 強制クローズの
  4 経路すべてで invalidation を保証
- 既存 tests を回帰しないよう care (positions テーブル fixture との整合)

**工数**: 2 時間

### サブスコープ C: マルチプロセス stale cache 対策

**問題**: FIRE Runner / OpenClaw が Stage 3+ で稼働すると、batch_replay /
seeder / tick が別プロセスで動く可能性。`_GLOBAL_CACHE_REGISTRY` は
プロセス内メモリなので、別プロセスの書き込みが反映されない。

**設計選択肢**:
- 案 1: signature SQL 復活 (低 TTL = 5 秒) で別プロセス書き込みを検知
  - per-call +1〜2 ms (許容範囲)
  - 単純で導入コスト低
- 案 2: shared memory (mmap) で cache 共有
  - 実装コスト高、Python の multiprocessing.shared_memory + 排他制御
- 案 3: Redis / SQLite 共有テーブルで cache server 化
  - 過剰、Mac mini 単機運用には不釣合
- 案 4: 単一プロセス強制 (multiprocessing 禁止)
  - FIRE Runner 設計と整合せず

→ **案 1 推奨** (TTL 短縮版)、実装は F275 で撤廃した signature SQL を
TTL=5 秒で復活させる形。

**工数**: 6 時間

## 成果物

(F277 着手時に詳細化)

予定:
- ~/fire/simulation/paper_live/tick.py (例外伝播 + run status='failed')
- ~/fire/simulation/paper_live/position.py (reset_trade_stats_cache 呼出)
- ~/fire/notifications/templates/error.py (新規、tick 失敗通知)
- ~/fire/patterns/similarity.py (signature SQL TTL=5 秒で復活)
- ~/fire/patterns/reproducibility.py (positions signature 設計、F275 で削除した
  関数を復活)
- ~/fire-vault/03_design/F277_implementation_<date>.md (新規)
- ~/fire-vault/02_todo/F277_*.md (本ファイル更新)

## 進捗チェックリスト

(F277 着手時に詳細化)

- [ ] サブスコープ A: tick.py 例外伝播設計 + 上位 catch 実装
- [ ] サブスコープ A: F236 LINE 緊急アラート連携 (tick 失敗通知)
- [ ] サブスコープ A: tests 追加 (例外発生時の run status='failed' 確認)
- [ ] サブスコープ B: positions 書き込み 4 経路の reset_trade_stats_cache 統合
- [ ] サブスコープ B: tests 追加 (TP/SL/force_close 後の stats 更新確認)
- [ ] サブスコープ C: マルチプロセス対策の設計選択 (案 1 推奨)
- [ ] サブスコープ C: signature SQL 復活 (TTL=5 秒) + 性能再計測
- [ ] サブスコープ C: マルチプロセスシナリオ単体テスト
- [ ] Run a 再走で性能維持確認 (per-call < 5 ms 維持目標)
- [ ] Codex pre-commit 全件解消 (F275 で残した 4 件 + 新規追加 0 件)

## 完了条件

- [ ] tick.py の例外が **沈黙ではなく run status='failed' に伝播**
- [ ] LINE 緊急アラート部屋に tick 失敗通知が届く (テストで確認)
- [ ] positions 更新後の `_TRADE_STATS_CACHE` が **次 tick で反映**される
- [ ] マルチプロセス構成で他プロセスの patterns / features 書き込みが
      **5 秒以内に反映**される (TTL=5 秒の場合)
- [ ] Codex pre-commit 全件解消 (`--no-verify` 不要)
- [ ] 性能目標維持: per-call < 5 ms、Run a < 60 分 (F275 ベースから -2 ms 程度の
      悪化は許容)

## 工数想定

合計 **1〜2 日** (サブスコープ並列):
- A (例外伝播): 4 時間
- B (positions invalidation 統合): 2 時間
- C (マルチプロセス対策): 6 時間
- 検証 + テスト: 4 時間

## 着手時期

**F275 完了済 → F277 を F276 より先に着手 (推奨)**

理由:
- Stage 3 で実トレード経路 (TP/SL/force_close) が動き始めると、現状の例外
  握り潰しと stale cache invalidation 不在が即座に実害化
- F276 (events 達成) の前に F277 (安全性確保) を完了する方が、F276 完了時点で
  Stage 3 移行可能な状態になる
- F277 と F276 はコード変更箇所もほぼ被らないため、並走の選択肢もあり

## 関連リンク

- 設計記録: [[../03_design/root_cause_hierarchy_2026-05-04]] (Layer 7-9 で
  F275 で残した課題として整理)
- F275 完了: [[../03_design/F275_similarity_optimization_complete_2026-05-04]]
- 完了基準 v1.1: [[../03_design/task_completion_criteria]]
- 前提タスク: [[F275_SimilarityEngine_最適化]]
- 並走 / 関連タスク: [[F276_events達成_positions_seeding]] /
  [[F278_git_ガバナンス整備]]
- 関連既存タスク: [[F236_LINE_5部屋構成]] (緊急アラート連携)、
  [[F050_Paper_Live_Mode_Stage_2_本体]]
- 運用ルール: [[../タスク運用ルール]]
