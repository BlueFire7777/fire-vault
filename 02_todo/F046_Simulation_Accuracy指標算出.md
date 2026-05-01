---
id: F046
phase: P2: Simulation/Backtest
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F045]
chapter: "19"
created: 2026-05-01
updated: 2026-05-01
---

# F046: Simulation Accuracy 指標算出

## 概要

`simulation/accuracy/` パッケージ新規作成。F045 ParallelReplayRun (PRP-...) を親に取り、3 モデル比較から 5 指標 (optimism_gap / pessimism_gap / robustness_score / fill_quality_avg / range_width) を算出。

## 実装内容

### 主要モジュール

- `simulation/accuracy/models.py`: `SimulationAccuracyRun` / `SimulationAccuracyResult` / `AccuracyMetricKey` Enum / `make_accuracy_run_id` (`ACC-` prefix)
- `simulation/accuracy/calculator.py`: `ModelOutcome` dataclass + 5 指標算出関数 + `compute_all_metrics` (ゼロ除算ガード付)
- `simulation/accuracy/runner.py`: `SimulationAccuracyRunner` — parent_run_id (PRP-...) → ACC-... 形式で結果保存
- `simulation/accuracy/report.py`: JSON / Markdown、R-19-09 仮想成績/本番区別の注意書き付
- `simulation/accuracy/cli.py`: `python -m simulation.accuracy --parent-run-id ... / --report ACC-...`
- `scripts/setup/migrate_simulation_accuracy.py`: `simulation_accuracy_runs` / `simulation_accuracy_results` 新規

### 5 指標の計算式

| 指標 | 式 | 意味 |
|---|---|---|
| optimism_gap | (IDEAL - REALISTIC) / abs(IDEAL) | 楽観バイアス |
| pessimism_gap | (REALISTIC - STRICT) / abs(REALISTIC) | 悲観シナリオ耐性 |
| robustness_score | STRICT / IDEAL | 最悪ケースで残る成果比率 |
| fill_quality_avg | avg(全モデルの fill_quality) | 約定品質 |
| range_width | IDEAL - STRICT | 結果の散らばり |

ゼロ除算ガード: `ideal=0` で optimism_gap / robustness_score を 0、`realistic=0` で pessimism_gap を 0。

### キーポイント

- フレームワーク先行: 現状は `pnl=0 / fill_quality=0 / n_trades=0` 固定、F100 後に replay_results から実集計
- F045 ParallelReplayRunner に `fill_models` オプション追加 (互換性保証、未指定時 `["ideal"]`)
- 既存 F045 25 PASS は非破壊維持 (デフォルト動作不変)

## テスト

- `tests/simulation/test_accuracy.py`: 18 ケース全 PASS
- `tests/simulation/test_parallel_replay.py`: F046 拡張 3 ケース追加 (F045 既存 25 + F046 拡張 3 = 28)
- 累計 136 PASS

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| accuracy パッケージ全 6 ファイル | ✅ |
| 5 指標計算 + ゼロ除算ガード | ✅ |
| simulation_accuracy_runs/results テーブル | ✅ |
| R-19-09 注意書き | ✅ |
| F045 25 PASS 非破壊 | ✅ |
| 18 + 3 PASS 追加 | ✅ |

## 関連リンク

- 要件書: [[FIRE_要件書_第19章_Simulation___Paper_Live_方針]]
- 要件 ID: R-19-06 (Simulation Accuracy 指標), R-19-09 (Dashboard 区別表示), R-19-05 (3 段階モデル)
- 前タスク: [[F045_過去データ高速リプレイ並列実行]]
- 次タスク: [[F047_Optimistic_Bias_Realistic_Fill_Score]]
- コード: `~/fire/simulation/accuracy/`
- テスト: `~/fire/tests/simulation/test_accuracy.py`
