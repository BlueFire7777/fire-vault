---
id: F100
phase: P2: Simulation/Backtest
priority: 高
status: 未着手
owner: Fujiwara
depends_on: []
chapter: "19"
created: 2026-04-24
updated: 2026-05-01
---

# F100: 市場データ API 統合

## 概要

J-Quants 等の市場データ API を統合し、F040-F047 のフレームワーク (現状はデータ 0 件で空完走) に実データを流入させる。これで Simulation / Backtest が本番動作する。

## 影響範囲

- F040 BacktestRunner: ohlcv_daily / features への実データ流入
- F041 ReplayRunner: as_of_date 時点の features を実値で取得
- F046 SimulationAccuracyRunner: replay_results から実 PnL を集計 (現状は 0 固定)

## 関連リンク

- 関連: [[F040_Backtest_Mode_Stage_0]], [[F041_Replay_Simulation_Mode_Stage_1]], [[F046_Simulation_Accuracy指標算出]]
