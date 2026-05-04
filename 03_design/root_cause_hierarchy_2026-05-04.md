# 真因階層振り返り — events=0 から F275 完了まで (2026-05-04)

**作成日**: 2026-05-04
**位置付け**: 2026-05-04 1 日に発生した F273〜F275 の真因階層 (D / D' / D'' / D''') を 9 層で整理した振り返りレポート
**起点**: [[F032_F054_diagnosis_2026-05-04]]
**完了**: [[F275_similarity_optimization_complete_2026-05-04]]

---

## 1. 全体タイムライン (2026-05-04 1 日の出来事)

| 時刻 (JST) | イベント | レイヤー判明 |
|---|---|---|
| 朝 (前日深夜投入の Chain 完走確認) | events=0 で 3 run 完走 | Layer 1 (表面) |
| 午前 | 仮説 A〜D 切り分け、F267 で features 不足解消 | Layer 2 |
| 午前 | F273 Phase 2-A smoke test (4 件 seed 成功) | Layer 3 → Layer 4 訂正 |
| 午後 | F273 Phase 2-B (Core500 fullscale seed)、Phase 2-C 走行前中止 | Layer 5 (D''' 発覚) |
| 夕方 | F274 numpy + cache 化、Phase 2-D 中止 | Layer 6 → Layer 7 |
| 夜 | F275 Phase 1 (運用経路 cProfile)、Phase 2-A〜D | Layer 7 → Layer 8 |
| 夜 (後半) | F275 Run a 21 分完走、案 D 確定 commit (3a222cb) | Layer 9 (現在) |

---

## 2. レイヤー別整理 (Layer 1 → Layer 9)

### Layer 1: 表面 (Chain が events=0 で完走)

**観察事実**:
- 前日深夜投入の `full_chain.sh` が exit 0 で完走
- 3 run (Run a/b/c) すべて `paper_live_runs.summary_json` で `n_events_total=0`
- F053 promotion check 5/5 中 4/5 FAIL、唯一 PASS した `simulation_accuracy` も
  events=0 由来の偽 PASS (= F271 § 6-6 アンチパターン)

**判明**: 
完走 = 成功ではない。意味的成功条件 (events / trades / positions の件数)
を確認する必要 (F271 § 6-2 制定の起点)。

### Layer 2: F267 で features 不足解消 (主因 A)

**仮説 A〜D 切り分け** (~/fire-vault/03_design/F032_F054_diagnosis_2026-05-04.md):
- A: features 未生成 → **確定** (87 行 / 5 銘柄 / 1 週間のみ、market_prices_daily 526K の 0.0009%)
- B: announcements 不足 (副因)
- C: パターン起動条件
- D: F032 score 閾値

**F267 実装**:
- `extract_features.py` batch モード追加 + Core500 universe + material wiring
- backfill 実行: 60,000 task / 1,074,280 features 行 / 2 分 44 秒 / err=0

**結果**: features 87 → **1,074,280 行**、`_has_features=True` カバレッジ 100%。

### Layer 3 → Layer 4 訂正: D' (Layer 4 instances 空) → D'' (Layer 1 がテンプレデータ)

**F273 Phase 1 (F040 調査)**:
- F267 後も Run a で events=0 が再現
- 当初 D' = 「Pattern Store の Layer 4 (instances) が空」と仮定
- → 要件書第 37 章 v3.4 を読み返して Layer 4 = **Death Note / Rehab** と判明
- D' は **誤り**、Layer 1 (Pattern Archive) こそ実データ紐付き不在の主因

**真の主因 D''**:
- patterns 9 件すべて (symbol="TEST_E2E" or "0000"、detected_at=ダミー時刻) で
  テンプレートのみ
- `fetch_pattern_feature_vector()` が 0 行返却 → `SimilarityEngine.search()` で
  `if not vec_past: continue` 全 skip → 空 list 返却 → decision="pass"

### Layer 4: D'' を F273 Phase 2-A で立証

**Phase 2-A (1 銘柄 × 10 日 smoke test)**:
- 99840 ソフトバンクG × 2026-04-17〜2026-05-01 で `breakout_flag=1` が 4 件発火
- F273_BREAKOUT_001〜004 として patterns に candidate INSERT
- ReproducibilityEngine 再実行: similar_count **0 → 4**、decision **pass → reference_only**、
  score **0 → 0.5787**

→ **D'' (Layer 1 テンプレデータ) は確定的に解消経路あり**。

### Layer 5: D''' SimilarityEngine の SQL N+1 問題

**Phase 2-B (Core500 fullscale seed)**:
- 4,700 件 patterns に candidate INSERT (~2 分 44 秒)
- patterns 総件数 9 → 4,709

**Phase 2-C (Run a 走行前中止)**:
- SimilarityEngine.search() per-call を計測 → **2,733 ms / call**
- per-pattern 0.58 ms × 4,709 = O(N) 線形スキャン
- Run a 1340 tick × 500 銘柄 = 670,000 search × 2.733 秒 = **約 21 日!**
- → 走行前中止判断 (F271 § 6-7 制定の起点 = 線形外挿楽観視回避)

**真因 D'''**:
- `SELECT * FROM patterns WHERE 1=1` で全件取得 + 各 pattern で
  `fetch_pattern_feature_vector()` (内部 2 SQL) → **9,418 SQL / search()**
- SimilarityEngine の SQL N+1 問題

### Layer 6: F274 で numpy + cache 化 → 280 倍高速化

**F274 Phase 2 (numpy 行列キャッシュ)**:
- 起動時 1 SQL JOIN で全 (pattern, features) を numpy 行列に load
- per-call: 2,733 ms → SimilarityEngine 単独で 1.5 ms (1,800 倍高速化、
  ReproducibilityEngine 経由で 9.81 ms = 280 倍高速化)
- ただし Run a 投入時に Codex pre-commit が 6 件 CRITICAL 指摘
- 5 件解消 + 6 件目 (writer-side invalidation のマルチプロセス制限) は F275 移管
- F274 commit `9d501aa` で `--no-verify` 例外承認 (1 回目)

**Phase 2-D 中止 (per-tick 4.57 秒、Run a 102 分予測)**:
- 中止判断 (F271 § 6-2 アンチパターン再発): SimilarityEngine 単独計測 (1.5 ms) を
  per-call と勘違い、本来は ReproducibilityEngine 経由 (9.81 ms) を測るべき

### Layer 7: 計測対象取り違え (F275 § 6-8 制定の起点)

**F275 Phase 1 (Phase 2 着手前の cProfile 内訳測定)**:
- `ReproducibilityEngine.evaluate()` per-call: **9.81 ms** (運用経路)
- 内訳:
  - SQL execute 40 回/call: 5.48 ms (45%)
  - sector_filter Python loop 4,700 回: 2.42 ms (20%)
  - fetch_pattern_feature_vector 5 回: 2.94 ms (24%)
  - signature SQL: 1.94 ms
  - fetch_pattern_trade_stats 5 回: 1.86 ms

→ F274 で測ったのは SimilarityEngine.search() (= 6.86 ms / call の一部) のみ、
ReproducibilityEngine の追加コスト (5.80 ms) を見逃していた。

**教訓 (F271 § 6-8 として制度化)**:
> 計測は必ず運用で支配的な経路で行う

### Layer 8: score 構造的停滞 (F275 Phase 1 で確定)

**最重要発見**:
72030/2026-04-30 の score 0.5787 = **頭打ち** の構造分析:

```
score = 0.20 * 1.0  (similarity max)
      + 0.25 * 0.5  (win_rate, positions 空 default)
      + 0.20 * 0.5  (expected_value, default)
      + 0.10 * 0.5  (regime_fit, F104 未稼働 default)
      + 0.10 * 0.5  (fill_quality, default)
      + 0.10 * 0.1867 (priority_boost, active=1 のみ)
      + 0.05 * 0.7  (risk_fit)
      = 0.5787 < SCORE_THRESHOLD_EXECUTE (0.65)
```

→ **F275 で性能改善しても decision="execute" 到達不可能**:
- positions テーブル空 → win_rate=0.5 / expected_value=0.5 default
- F104 (regime collector) 未稼働 → regime_fit=0.5 default
- Layer 3 active=1 件のみ → priority_boost=0.1867 (弱)

→ **events_total >= 50 達成は F275 のスコープ外**、F276 (positions seeding +
F104 + Layer 3 拡大) で対応する判断を Fujiwara 承認。

### Layer 9: F275 で per-call 3.13 ms 達成 (Stage 3 性能ブロッカー解消)

**F275 案 D 確定 (commit 3a222cb)**:
- Phase 2-A: writer-side invalidation
- Phase 2-B: sector_filter numpy 化
- Phase 2-C: trade_stats cache + top_match features 再取得撤廃
- Phase 2-D: tick.py _has_features 一括 SQL + logger 例外記録

**性能改善 (運用経路 ReproducibilityEngine.evaluate() 実測)**:
- per-call: 9.81 ms → **3.13 ms** (-68%)
- 1 tick × 500 銘柄: 4.9 秒 → **約 1.6 秒** (-67%)
- Run a: 110 分予測 → **34.9 分予測** (aab0aac vault に **21 分実測** も併記)
- F273 当初 21 日見積 → 35 分 = **約 870 倍高速化**

**Stage 3 ブロッカー判定**:
- ✅ 性能ブロッカー解消 (F275)
- ❌ events ブロッカー (F276 移管)
- ❌ 例外伝播 + マルチプロセス cache (F277 移管、既存問題 + 過剰防御)

---

## 3. 残課題 (F276 / F277 へ移管)

### F276: events_total >= 50 達成

スコープ:
- positions テーブル seeding (F040 Backtest 経由) → win_rate / expected_value 稼働
- F104 (regime collector) 解禁 → regime_fit 稼働
- Layer 3 active 拡大 (R-13-08 規律下) → priority_boost 強化

工数: 2〜3 日 (F040 既実装活用)

### F277: paper_live 例外伝播設計 + マルチプロセス stale cache 対策

スコープ:
- F277-A: tick.py 例外伝播設計 (raise + caller catch + run status='failed')
- F277-B: positions 書き込み経路から `reset_trade_stats_cache()` 統合呼出
- F277-C: マルチプロセス stale cache 対策 (signature SQL 復活 / shared memory / etc)

工数: 1〜2 日

---

## 4. 教訓 (F271 § 6-7 / 6-8 として制度化済み)

### § 6-7「線形外挿で楽観視」(F273 Phase 2-C / F274 Phase 2-D 事例)

- 13 件 patterns で 0.58 ms/call → 4,700 件で 2,733 ms/call (360 倍鈍化を
  実測検証せず楽観視 = 21 日見積で発覚)
- F274 Phase 2-A/B/C で SimilarityEngine 単独 1.5 ms → Run a 17 分外挿 →
  実 ReproducibilityEngine 経由 9.81 ms / 102 分で中止

### § 6-8「計測対象の取り違え」(F274 Phase 2-A/B/C 事例)

- SimilarityEngine.search() 単独 = 運用で支配的な経路ではない (= サブモジュール)
- 本来計測すべきは ReproducibilityEngine.evaluate() (= 運用経路)
- F275 では cProfile で内訳を取り、運用経路で per-call を測定して再発防止

→ 仕様書 [[task_completion_criteria]] v1.1 に追記済 (F275 で対応、commit
`aab0aac` の Vault push、~/fire side では既存 v1.0 のまま、F278 で同期予定)。

---

## 5. 現状の進捗 (Stage 3 移行マイルストーン)

| ブロッカー | 状態 | 移管先 |
|---|---|---|
| 性能 (Run a 60 分以内) | ✅ **解消** (F275、実測 21 分 / 予測 35 分) | — |
| events_total >= 50 | ❌ 未解消 | **F276** |
| 例外処理 + マルチプロセス cache | ❌ 既存問題 (Stage 0-2 で実害なし、Stage 3 前必須) | **F277** |
| git ガバナンス整備 | ⏸ Stage 3 開始前必須 | **F278** |
| 休日専業モード | 🔮 M+1 以降の設計議論済み | **F279** |

→ Stage 3 移行 (F266 = 移行ゲート) には F276 + F277 + F278 完了が必須、
F279 は M+1 以降。

---

## 6. F271 完了基準 (本振り返り自体の自己評価)

| 段階 | 結果 | 根拠 |
|---|---|---|
| 動いた | ✅ | 9 層で時系列整理、関連リンク網羅、F273〜F275 の経緯を 1 文書に集約 |
| 機能した | ✅ | レイヤー間の因果関係 (D' 誤認 → D'' 訂正、計測対象取り違え) を明示、F271 § 6-7/6-8 起点との整合 |
| 期待値達成 | ✅ | F276/F277/F278/F279 着手時の振り返り材料が揃った、F273-F275 の事故が完了基準仕様書 v1.1 として制度化済み |

---

## 7. 関連リンク

- 起点診断: [[F032_F054_diagnosis_2026-05-04]]
- F267 (features 解消): [[F267_implementation_2026-05-04]]
- F273 Phase 1: [[F040_backtest_status_2026-05-04]]
- F273 Phase 2-A: [[F273_phase2a_smoke_test_2026-05-04]]
- F273 Phase 2-B/C: [[F273_phase2bc_results_2026-05-04]]
- F274 Phase 1: [[F274_phase1_design_2026-05-04]]
- F275 Phase 1: [[F275_phase1_design_2026-05-04]]
- F275 完了: [[F275_similarity_optimization_complete_2026-05-04]]
- 完了基準 v1.1: [[task_completion_criteria]]
- 後続タスク: [[02_todo/F276_events達成_positions_seeding]] /
  [[02_todo/F277_paper_live例外伝播_マルチプロセスcache]] /
  [[02_todo/F278_git_ガバナンス整備]] /
  [[02_todo/F279_休日専業モード設計仕様書]]
- 関連設計議論: [[git_governance_2026-05-04]] /
  [[operation_mode_holiday_intensive_draft_2026-05-04]]

---

(以上、F276 / F277 / F278 / F279 着手時の振り返り起点として活用)
