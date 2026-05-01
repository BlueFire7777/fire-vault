---
id: F040
phase: P2: Simulation/Backtest
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F029, F030, F031]
chapter: "19"
created: 2026-04-26
updated: 2026-04-26
---

# F040: Backtest Mode (Stage 0)

## 概要

`simulation/backtest/` パッケージを新規作成。期間指定で過去データのバックテスト集計を実行する Stage 0 基盤。データ 0 件でも空レポートで完走する「フレームワーク先行」方針。

## 実装内容

### 主要モジュール

- `simulation/backtest/runner.py`: `BacktestRunner` — 期間指定で集計、`run_id` 管理、サマリ生成
- `simulation/backtest/aggregator.py`: 5 カテゴリ集計 (overall / by_pattern / by_lane / by_strategy_type / by_sector)
- `simulation/backtest/report.py`: JSON / Markdown 出力、R-19-09 仮想成績/本番区別の注意書き付き
- `scripts/setup/migrate_backtest.py`: `backtest_runs` / `backtest_results` 新規テーブル

### キーポイント

- フレームワーク先行: F100 (市場データ API) 完了後にデータ流入で即本番動作
- データ 0 件で完走: target_patterns=[] でも `status=completed` で正常終了
- レポートに「これは Backtest (Stage 0) の仮想成績であり、本番取引結果ではない (R-19-09)」を必ず挿入

## テスト

- `tests/simulation/test_backtest.py`: 全 20 PASS

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| backtest パッケージ新規作成 | ✅ |
| 5 カテゴリ集計 | ✅ |
| R-19-09 注意書き | ✅ |
| 全 20 PASS | ✅ |

## 関連リンク

- 要件書: [[FIRE_要件書_第19章_Simulation___Paper_Live_方針]]
- 要件 ID: R-19-01 (Simulation 必須), R-19-02 (5 段階検証), R-19-06 (Accuracy 指標), R-19-09 (Dashboard 区別表示)
- 次タスク: [[F041_Replay_Simulation_Mode_Stage_1]]
- コード: `~/fire/simulation/backtest/`
- テスト: `~/fire/tests/simulation/test_backtest.py`
- 運用ルール: [[../タスク運用ルール|タスク運用ルール]]
