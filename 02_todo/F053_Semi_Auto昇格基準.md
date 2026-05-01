---
id: F053
phase: P3: Paper Live
priority: 中
status: 完了
owner: Fujiwara
depends_on: [F050, F046, F047, F052]
chapter: "19"
created: 2026-05-01
updated: 2026-05-01
tags: [P3, paper_live, promotion, governance]
---

# F053: Paper Live → Semi Auto 昇格基準実装 (P3 フェーズ完成)

## 概要

R-19-08 (本番移行前必須条件) の **5 項目** を判定する関数群と統合判定オーケストレーター
`evaluate_promotion()` を実装。これで **P3 (Paper Live) フェーズ完成** 🎉。

原則: 「収益が出ていることだけを理由に本番移行してはならない」(R-19-08)

## 5 判定項目 (R-19-08)

| # | 項目 | 判定内容 | 既定閾値 |
|---|---|---|---|
| 1 | サンプル数 | n_ticks ≥ 20 営業日 OR closed positions ≥ 50 トレード | 20 / 50 |
| 2 | 期待値 | 勝率 / 平均損益 / 平均 RR / 最大 DD が基準内 | 0.50 / 0.0 / 1.5 / 0.20 |
| 3 | 停止機能 | halt_reason が一度でも記録されている (暫定判定) | True |
| 4 | クローズ機能 | force_close 件数 ÷ 内 closed 件数 ≥ 100% | 1.0 |
| 5 | Simulation 精度 | F047 optimistic_bias_score ≤ 許容値 | 0.7 |

## 実装内容

### 主要モジュール

- `simulation/paper_live/promotion.py` (新規)
  - `DEFAULT_CRITERIA` (R-19-08 既定値)
  - `check_sample_size` / `check_expected_value` / `check_halt_function` /
    `check_close_function` / `check_simulation_accuracy`
  - `evaluate_promotion(run_id)` — 5 項目をまとめて評価し `PromotionResult` を返す
- `simulation/paper_live/models.py` (拡張)
  - `PromotionCriteria` / `CriterionCheck` / `PromotionResult` dataclass
- `simulation/paper_live/cli.py` (拡張)
  - `--check-promotion RUN_ID` フラグ追加 (5 項目を [PASS]/[FAIL] で表示)
- `simulation/paper_live/__init__.py` (拡張)
  - 公開 API export

### キーポイント

- 閾値はファイル冒頭の `PromotionCriteria` で定数化、`DEFAULT_CRITERIA` でアクセス
- 各 check 関数は run_id 不存在 / データ 0 件でも動作可能 (該当なしで FAIL を返す)
- `check_expected_value` の RR 計算: 損失なし & 利益あり → `+inf` (常に閾値を満たす)
- 最大 DD は累積 PnL のピークからの最大落ち幅率

### 暫定実装 (将来精緻化予定)

- **check_halt_function**: 現状は `halt_reason` の存在で判定。将来 `halt_history`
  テーブルで履歴管理する設計が望ましい (F053 スコープ外)
- **check_simulation_accuracy**: paper_live run と accuracy run の関連付けが弱い。
  最新の `optimistic_bias_score` を参照する形。将来 `PaperLiveRun` から特定
  `accuracy_run_id` を関連付ける機構を追加予定
- **check_close_function**: `force_close` 件数 ÷ 内 closed 件数 で判定。
  実データ流入後にエッジケース検証

## テスト

- `tests/simulation/test_paper_live_promotion.py`: **19 ケース全 PASS**
  - DEFAULT_CRITERIA (1) / sample_size (3) / expected_value (5) /
    halt_function (3) / close_function (3) / simulation_accuracy (3) /
    evaluate_promotion 統合 (1)
- 累計 **228 PASS** (209 → 228、F040-F052 既存 209 は非破壊維持)

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| promotion.py 5 関数 + evaluate_promotion | ✅ |
| models.py に 3 dataclass 追加 | ✅ |
| CLI --check-promotion 追加 | ✅ |
| __init__.py export 更新 | ✅ |
| 19 ケース全 PASS | ✅ |
| F040-F052 既存 209 PASS 非破壊 | ✅ |
| 累計 228 PASS | ✅ |
| CLI 動作確認 (5 項目判定表示) | ✅ |
| **P3 フェーズ完成** | 🎉 |

## CLI 出力サンプル

```
$ python -m simulation.paper_live --check-promotion PL-20260501095521-744F
Promotion check for run_id: PL-20260501095521-744F
Overall: FAIL
Evaluated at: 2026-05-01T09:55:21.722551+00:00

  [FAIL] sample_size: n_ticks=0 (>=20? False), n_trades=0 (>=50? False)
  [FAIL] expected_value: no closed trades to evaluate
  [FAIL] halt_function: halt_reason recorded? False (暫定判定...)
  [FAIL] close_function: no force_close events (untested)
  [PASS] simulation_accuracy: optimistic_bias_score=0.000 (<= 0.7? True)
```

## 関連リンク

- 要件書: [[FIRE_要件書_第19章_Simulation___Paper_Live_方針]]
- 要件 ID: R-19-08 (本番移行前必須条件), R-19-07 (Bias Score 連携)
- 前タスク: [[F052_Paper_Live厳しめ約定モデル標準化]]
- 連携: [[F046_Simulation_Accuracy指標算出]] / [[F047_Optimistic_Bias_Realistic_Fill_Score]]
- コード: `~/fire/simulation/paper_live/promotion.py`
- テスト: `~/fire/tests/simulation/test_paper_live_promotion.py`

## P3 (Paper Live) フェーズ完成 🎉

| Task | 内容 |
|---|---|
| F050 | Paper Live tick ベースランナー |
| F051 | 仮想建玉 / 平均建値 / 評価損益 / 実現損益 / 余力管理 |
| F052 | 厳しめ約定モデル標準化 (R-19-05) |
| **F053** | **Semi Auto 昇格基準 (R-19-08) ← 完成** |

**24 時間稼働への最短ルート**: F100 (市場データ API) で実データ流入を開始すれば、
F040-F053 のフレームワークが自動的に本番動作する設計。

## スコープ外メモ

- 自動昇格 (判定 PASS で自動的に Stage 3 移行) — Live Advisory フェーズで別タスク
- halt_history テーブル (将来精緻化、別タスク)
- paper_live run と accuracy run の正規連携 (将来)
- ダッシュボード表示連携 (将来)

## 2026-05-01 修正: check_sample_size を DISTINCT date(tick_dt) ベースに変更

### 問題

修正前は `paper_live_runs.n_ticks` (tick 呼び出し回数) で営業日数を近似していた。
これだと「1 営業日に複数 tick しただけで 20 を超える」バグがあった
(本来の R-19-08「20 営業日のサンプル」を保証していない)。

### 修正後

`paper_live_results.tick_dt` から `COUNT(DISTINCT date(tick_dt))` で「ユニークな
営業日数」を計算。過去データ高速回し / リアル稼働の両モードで正しくカウント。
1 日に何回 tick しても 1 日扱い。

### 戦略的位置付け (Fujiwara さんの洞察)

24 時間稼働 + シミュレーションループが FIRE の本質的エッジ。
R-19-08 の 20 営業日条件は「過去データ高速回しで満たす」ことを許容する設計。
CLAUDE.md にも「FIRE の戦略的強み: シミュレーション主導戦略」セクションで明記。

### テスト変更

- `TestSampleSize` 既存 3 ケース修正:
  - `test_passes_with_20_distinct_days` (旧 `test_n_ticks_meets_threshold_passes`)
  - `test_passes_with_50_trades_even_without_tick_dt` (旧 `test_n_trades_meets_threshold_passes`)
  - `test_fails_with_both_below` (旧 `test_both_below_threshold_fails`)
- 新規 1 ケース追加:
  - `test_counts_unique_days_only` (同日複数 tick → 1 日カウント検証)
- F053 全 19 → 20 ケース、累計 228 → **229 PASS**

### Git コミット

- `fix(F053): check_sample_size を DISTINCT date(tick_dt) ベースに修正` (~/fire)
- `docs(CLAUDE): シミュレーション主導戦略を明記` (~/fire)
- `docs(02_todo): F053 補修メモに DISTINCT date(tick_dt) 修正を記録` (~/fire-vault)
