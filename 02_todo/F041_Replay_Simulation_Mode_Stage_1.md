---
id: F041
phase: P2: Simulation/Backtest
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F040]
chapter: "19"
created: 2026-04-30
updated: 2026-04-30
---

# F041: Replay Simulation Mode (Stage 1)

## 概要

`simulation/replay/` パッケージを新規作成。指定期間を 1 日ずつ昇順に再生し、各営業日 (as_of_date) 時点で利用可能なデータのみで候補抽出を実行する Stage 1。look-ahead bias を構造的に排除。

## 実装内容

### 主要モジュール

- `simulation/replay/ticker.py`: `TimeReplayTicker` — 日付昇順 1 日ずつ再生、`LookAheadError` で違反検出
- `simulation/replay/runner.py`: `ReplayRunner` — 各営業日時点で候補抽出、`replay_results` に時系列記録
- `simulation/replay/aggregator.py`: 4 カテゴリ集計 (overall / by_date / by_pattern / by_lane) (in-memory)
- `simulation/replay/report.py`: JSON / Markdown 出力、R-19-09 注意書き付き
- `simulation/replay/cli.py`: `python -m simulation.replay --start --end --patterns`
- `scripts/setup/migrate_replay.py`: `replay_runs` / `replay_results` 新規テーブル (`as_of_date` カラム追加)

### キーポイント

- look-ahead bias 排除: `as_of_date` 時点で利用可能な features のみを `TickFrame` に入れる
- フレームワーク先行: F100 完了後に market data 流入で即本番動作
- Stage 1 では候補抽出の時系列記録のみ。損益計算は F042-F044 で扱う

## テスト

- `tests/simulation/test_replay.py`: 全 20 PASS (累計 40)

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| replay パッケージ新規作成 | ✅ |
| TimeReplayTicker (look-ahead 排除) | ✅ |
| 4 カテゴリ集計 | ✅ |
| 全 20 PASS | ✅ |

## 関連リンク

- 要件書: [[FIRE_要件書_第19章_Simulation___Paper_Live_方針]]
- 要件 ID: R-19-01, R-19-03 (Stage 1 Replay), R-19-09
- 前タスク: [[F040_Backtest_Mode_Stage_0]]
- 次タスク: [[F042_理想約定モデル_A]]
- コード: `~/fire/simulation/replay/`
- テスト: `~/fire/tests/simulation/test_replay.py`
