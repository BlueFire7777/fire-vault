---
id: F277
phase: P9: 運用・保守
priority: 最優先
status: 着手中 (Phase 0)
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
- PositionTracker._insert_position / _update_position 末尾で
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

### 実装 (5 ファイル)

- `~/fire/simulation/paper_live/tick.py` — sentinel return 4 箇所を raise に変更、
  上位 (`extract_candidates / monitor_virtual_positions / evaluate_force_close`) で
  catch、tick 全体を `status='failed'` で `paper_live_runs` に記録 (サブスコープ A)
- `~/fire/simulation/paper_live/position.py` — `PositionTracker._insert_position /
  _update_position` 末尾で `reset_trade_stats_cache()` 呼出統合 (サブスコープ B)
- `~/fire/notifications/templates/error.py` — 新規、F236 LINE 5 部屋構成の緊急
  アラート部屋に「tick 失敗」通知 (サブスコープ A)
- `~/fire/patterns/similarity.py` — `DEFAULT_SIGNATURE_TTL_SEC = 0.0` → `5.0`、
  signature SQL 経路 default 有効化 (サブスコープ C)
- `~/fire/patterns/reproducibility.py` — F275 で撤廃した signature 関数復活
  (Phase 1 で範囲確定)、line 62-96 の F277 言及 TODO コメント削除 (サブスコープ B/C)

### Vault 記録 (3 ファイル)

- `~/fire-vault/03_design/F277_phase1_design_<date>.md` — Phase 1 設計記録 (新規)
- `~/fire-vault/03_design/F277_implementation_<date>.md` — Phase 2 実装記録 (新規)
- `~/fire-vault/02_todo/F277_*.md` — 本ファイル進捗チェックリスト更新、完了時
  `02_todo/` から `04_archive/` へ移動

## 進捗チェックリスト

### Phase 0: 着手準備

- [x] 参照すべき Vault / コード 8 件再読 (F277 スタブ / F275 完了レポート /
      task_completion_criteria v1.1 / root_cause_hierarchy / tick.py / position.py /
      reproducibility.py:62-96 / similarity.py 該当 4 箇所)
- [x] スタブ詳細化 (本セクション + 成果物 + 完了条件) + commit
- [ ] 本部に step 2 完了報告 → Phase 1 設計開始の合図受領

### Phase 1: 設計確定 (中間報告 → 本部レビュー → 承認待ち)

本部発行「Phase 1 で確定すべき項目」4 項目への回答を中間報告として提出する
(二段確認制)。Fujiwara レビュー応答を待ってから Phase 2 着手。

- [ ] A: tick.py の raise 対象 5 箇所 (`_has_features` の扱い含む) と上位 catch 配置
- [ ] B: PositionTracker の 4 経路 (仮想エントリー / TP / SL / force_close) から
      `_insert_position / _update_position` への合流確認
- [ ] C: 撤廃済み signature SQL 関数の復活範囲 (`similarity.py` 単独か、
      `reproducibility.py` 含むか) を git log で特定
- [ ] 全体: F236 LINE 5 部屋構成のうちどの部屋に通知するか、メッセージテンプレート設計
- [ ] Phase 1 中間報告 commit + 本部提出

### Phase 2: 実装 (サブスコープ A/B/C 並列実施可)

#### サブスコープ A (4h): tick.py 例外伝播設計

- [ ] `_fetch_all_symbols / _fetch_symbols_with_features / _fetch_ohlc /
      engine.evaluate` の sentinel return → raise
- [ ] 上位 catch (`extract_candidates / monitor_virtual_positions /
      evaluate_force_close`) で `paper_live_runs.status='failed'` 記録
- [ ] F236 LINE 緊急アラート部屋への「tick 失敗」通知経路追加
- [ ] tests: 例外発生時 status='failed' / LINE 通知 end-to-end 実証

#### サブスコープ B (2h): PositionTracker invalidation 統合

- [ ] `_insert_position / _update_position` 末尾で `reset_trade_stats_cache()` 呼出
- [ ] tests: TP/SL/force_close 後の `_TRADE_STATS_CACHE` 更新確認 (4 経路)
- [ ] reproducibility.py:62-96 の F277 言及 TODO コメント削除

#### サブスコープ C (6h): マルチプロセス stale cache 対策 (案 1)

- [ ] `DEFAULT_SIGNATURE_TTL_SEC = 5.0` 変更 + signature SQL 関数復活
- [ ] 既存テストへの影響確認 (signature_ttl_sec=0 前提のテストがあれば対応)
- [ ] マルチプロセスシナリオ単体テスト (別プロセス書込が 5 秒以内に反映)

#### 段階性ガード: 3 ポイント計測 (F271 §6-7 線形外挿楽観視回避)

各サブスコープ完了時点で `ReproducibilityEngine.evaluate()` 経由 (運用経路、
F271 §6-8) の per-call 計測を必須とし、Vault に記録:

- [ ] A 単独完了時点の per-call 計測
- [ ] A+B 完了時点の per-call 計測
- [ ] A+B+C 完了時点の per-call 計測 + Run a 1340 ticks 実走

### 検証

- [ ] Run a 再走で性能維持確認 (per-call < 5 ms 維持、Run a < 60 分維持)
- [ ] Codex pre-commit 全件解消 (F275 で残した CRITICAL 4 件 + 新規 0 件、
      `--no-verify` 不要)

## 完了条件

F271 v1.1 § 4 完了水準 3 段階すべての到達を必須とする。

### 動いた

- [ ] 各サブスコープ A/B/C のユニットテスト PASS、既存テスト 0 件回帰
- [ ] Codex pre-commit 厳格通過 (`--no-verify` 不要)

### 機能した (意味的成功条件、運用経路で実証)

- [ ] **A**: SQLite ロックを意図的に発生させ、tick 全体が `paper_live_runs.status=
      'failed'` で記録され、F236 LINE 緊急アラート部屋に「tick 失敗」通知が届く
      end-to-end を実証
- [ ] **A**: 「正常な候補なし」(空 list) と「例外による失敗」が明確に区別される
- [ ] **B**: TP/SL/force_close 後の次 tick で `_TRADE_STATS_CACHE` が更新後の
      win_rate / expected_value を返すことを実証
- [ ] **C**: マルチプロセス構成で別プロセスの patterns / features 書き込みが
      5 秒以内に反映されることを実証

### 期待値達成 (受入基準)

- [ ] per-call < 5 ms 維持 (F275 ベース 2.53 ms から -2 ms 程度の悪化は許容)
- [ ] Run a 1340 ticks < 60 分維持 (F275 実測 21 分から悪化幅を Vault 記録)
- [ ] F275 で残した Codex CRITICAL 4 件すべて構造的解消:
      (1) TTL stale window / (2) features DELETE/UPDATE 検知不能 /
      (3) tick.py 例外握り潰し / (4) `_TRADE_STATS_CACHE` invalidation 未実装
- [ ] reproducibility.py:62-96 の F277 言及 TODO コメント削除確認

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
