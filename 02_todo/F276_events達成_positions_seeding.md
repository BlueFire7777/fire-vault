---
id: F276
phase: P9: 運用・保守
priority: 最優先
status: 未着手
owner: Fujiwara
depends_on: [F275]
chapter: "11, 13, 19"
created: 2026-05-04
updated: 2026-05-04
---

# F276: events>0 達成 (positions seeding + F104 + Layer 3 拡大)

## 概要

F275 完了で Stage 3 性能ブロッカーは解消したが、`decision="execute"` に到達
できず events_total=0 のまま。F275 Phase 1 の score 構造分析で「**positions
空 + regime 未稼働 + active=1 件**」が頭打ちの構造的原因と確定 → F275 スコープ外、
本タスク (F276) で体系対応。

## タスク詳細

### F275 Phase 1 で確定した score 構造的限界

72030/2026-04-30 評価実測:
```
score = 0.20*1.0  (similarity max)
      + 0.25*0.5  (win_rate, positions 空 default)
      + 0.20*0.5  (expected_value, default)
      + 0.10*0.5  (regime_fit, F104 未稼働 default)
      + 0.10*0.5  (fill_quality, default)
      + 0.10*0.1867 (priority_boost, active=1 のみ)
      + 0.05*0.7  (risk_fit)
      = 0.5787 < SCORE_THRESHOLD_EXECUTE (0.65)
```

→ events_total >= 50 達成には以下 3 サブスコープが必要 (Fujiwara 起票時に
  F276-A/B/C のサブタスク分割を判断)。

### サブスコープ A: positions seeding (F040 Backtest 経由)

- 過去 6 ヶ月 × Core500 で F040 Backtest を実行
- 実 fill model (B = 現実寄り or C = 厳しめ) で過去候補を仮想エントリー →
  TP/SL/timeout で仮想クローズ → positions テーブルに INSERT (label 付き)
- これで `fetch_pattern_trade_stats(pattern_id)` が実 win_rate / expected_value
  を返すようになる
- 期待効果: win_rate / expected_value が default 0.5 → 実値 (例 0.6〜0.8) に
  上昇、score +0.10〜+0.15 増 → execute 達成可

### サブスコープ B: F104 (regime collector 解禁)

- F104 (指数四本値取得) を実装、TOPIX / 日経 225 / 米株指数の四本値取得
- regime collector を ready=True 化、`market_regime` / `nikkei_direction` /
  `topix_direction` 等の features を生成
- F267 系の extract_features に regime collector 分岐を追加
- 期待効果: regime_fit が default 0.5 → 実値 (例 0.7〜0.9) に上昇、score
  +0.02〜+0.04 増

### サブスコープ C: Layer 3 active 拡大 (R-13-08 規律下)

- 現状 Layer 3 active=1 件のみ (`material_initial-strong_market-breakout-AM-highliq@v1.0`)
- F029 S-X 格付けを過去 positions 蓄積 (= サブスコープ A 後) で再評価
- ランク S/A のパターンを Layer 3 (approved_active) に昇格
- Fujiwara 承認制で 5〜10 件まで拡大
- 期待効果: priority_boost が 0.1867 → 0.5〜0.8 (LAYER_BOOST=1.5 適用済) に
  上昇、score +0.03〜+0.06 増

### 合計期待効果

3 サブスコープすべて完了で score 0.5787 → 0.7〜0.85 → **decision="execute"
到達 (SCORE_THRESHOLD_EXECUTE=0.65 突破)** → events_total >= 50 達成見込み。

## 成果物

(F276 着手時に詳細化)

予定:
- ~/fire/scripts/jobs/run_backtest_seeding.py (新規、F040 を活用した seeding スクリプト)
- ~/fire/data/fire.db: positions テーブルに過去仮想トレード INSERT
- ~/fire/market_data/regime/ (F104 実装)
- ~/fire/patterns/ Layer 3 拡大 (PatternStore.transition で approved_active 昇格)
- ~/fire-vault/03_design/F276_implementation_<date>.md (新規)
- ~/fire-vault/02_todo/F276_*.md (本ファイル更新、サブタスク分割記録)

## 進捗チェックリスト

(F276 着手時に詳細化)

- [ ] サブスコープ判断 (A/B/C を一括で進める or 分割か)
- [ ] サブタスク F276-A / F276-B / F276-C への分割判断
- [ ] サブスコープ A: F040 を seeding 用途に拡張 + 過去 6 ヶ月実行
- [ ] サブスコープ B: F104 (指数四本値) 実装 + regime collector 解禁
- [ ] サブスコープ C: F029 S-X 再評価 + Layer 3 active 5〜10 件拡大 (承認制)
- [ ] Run a 再走で events_total >= 50 確認
- [ ] R-19-08 「20 営業日 OR 50 トレード」条件達成

## 完了条件

- [ ] Run a (1340 tick × Core500) で **events_total >= 50** を達成
- [ ] event_counts に candidate_extracted / entry / exit / force_close / halt
      の各内訳件数あり
- [ ] paper_live_results / paper_live_positions が Run a 由来件数で
      非ゼロ (R-19-08 サンプルサイズ満たす)
- [ ] F053 promotion check で 5/5 PASS (偽 PASS でなく実 PASS)
- [ ] Stage 3 移行 (F266) ゲート条件 events 部分を満たす

## 工数想定

合計 **2〜3 日** (サブスコープ並列):
- A (positions seeding): 1 日
- B (F104 regime): 1 日
- C (Layer 3 拡大): 0.5 日 (A 完了後の S-X 再評価)
- Run a 再走 + 検証: 0.5 日

## 着手時期

**F275 完了済 → F277 完了後に着手** (F277 で Stage 3 例外処理ブロッカーを
先に解消する判断。Stage 3 で実トレード経路が動き始めると、現状の例外握り潰しと
stale cache invalidation 不在が即座に実害化するため、F276 (events 達成) の前に
F277 (安全性確保) が筋)。

ただし Fujiwara 判断で並走の選択肢もあり (F277 と F276 は独立スコープ、
コード変更箇所もほぼ被らない)。

## 関連リンク

- 設計記録: [[../03_design/root_cause_hierarchy_2026-05-04]] (Layer 8 で
  score 構造的停滞を確定)
- 起点: [[../03_design/F275_phase1_design_2026-05-04]] (Phase 1-C で events
  達成は F276 移管と確定)
- F275 完了: [[../03_design/F275_similarity_optimization_complete_2026-05-04]]
- 完了基準 v1.1: [[../03_design/task_completion_criteria]]
- 前提タスク: [[F275_SimilarityEngine_最適化]]
- 並走 / 後続タスク: [[F277_paper_live例外伝播_マルチプロセスcache]] /
  [[F278_git_ガバナンス整備]]
- 関連既存タスク: [[F040_Backtest_Mode_Stage_0]] (本タスクで活用)
- 運用ルール: [[../タスク運用ルール]]
