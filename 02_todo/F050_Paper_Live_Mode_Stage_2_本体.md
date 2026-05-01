---
id: F050
phase: P3: Paper Live
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F043, F047]
chapter: "19"
created: 2026-05-01
updated: 2026-05-01
tags: [P3, paper_live, stage2]
---

# F050: Paper Live Mode (Stage 2) 本体

## 概要

Paper Live (Stage 2) の **tick ベースランナー** を実装。`PaperLiveRunner.tick(as_of_dt)` で
1tick 処理を実行し、候補抽出 / 仮想エントリー判定 / TP・SL 監視 / 強制クローズ判定 /
LINE 通知フックを順次実行する。データ 0 件でも空完走するフレームワーク先行設計。
F100 完了後に実データで本番動作。

## 実装内容

### 主要モジュール

- `simulation/paper_live/models.py`: `PaperLiveRun` / `PaperLiveResult` / `TickResult` /
  `PaperLiveEventType` Enum (7 種) / `make_paper_live_run_id` (`PL-` prefix)
- `simulation/paper_live/tick.py`: `TickContext` + 5 個別関数 + `process_tick`
  - `extract_candidates` / `evaluate_virtual_entries` / `monitor_virtual_positions` /
    `evaluate_force_close` / `collect_notifications`
  - look-ahead bias 排除 (F041 ticker 思想、`WHERE date <= as_of_dt`)
- `simulation/paper_live/runner.py`: `PaperLiveRunner` (ステートフル)
  - `start_session` / `tick` / `end_session` の順序を強制
  - `_current_run_id` で現在のセッションを保持
  - tick 1 回ごとに `n_ticks` をインクリメント
  - end_session で `event_counts` 集計 → `summary_json` に保存
- `simulation/paper_live/notification.py`: LINE 通知フックポイント
  - `format_notification_message`: `[仮想エントリー]` 等のラベル付きメッセージ生成
  - `would_send`: NOTIFICATION イベントかの判定
  - `send_to_line`: `NotImplementedError` (実送信は F050 スコープ外)
- `simulation/paper_live/report.py`: JSON / Markdown レポート (R-19-09 注意書き付)
- `simulation/paper_live/cli.py`: `--start-session` / `--tick` / `--end-session` / `--report`
- `scripts/setup/migrate_paper_live.py`: `paper_live_runs` / `paper_live_results` 新規

### イベント種別 (PaperLiveEventType)

| 値 | 意味 |
|---|---|
| `candidate` | 候補抽出 |
| `virtual_entry` | 仮想エントリー |
| `virtual_tp` | 仮想利確 |
| `virtual_sl` | 仮想損切 |
| `force_close` | 強制クローズ |
| `notification` | LINE 通知 (フックのみ、実送信なし) |
| `no_op` | tick で何も起きなかった |

### キーポイント

- フレームワーク先行: 各 tick 関数は現状空動作 (データ 0 件) で、F100 後に
  `WHERE date <= as_of_dt` の SQL に実データが流れて自然に本番動作する設計
- `PaperLiveRunner` はステートフル — start_session → tick → tick → ... → end_session の順序
- データ 0 件 tick では `paper_live_results` に NO_OP 等の不要レコードを作らない
  (events=[] なら何も保存しない)
- `collect_notifications` は VIRTUAL_ENTRY/TP/SL/FORCE_CLOSE のみを通知対象に判定

## テスト

- `tests/simulation/test_paper_live.py`: **24 ケース全 PASS**
  - make_paper_live_run_id (1) / start_session (4) / tick 基本 (5) /
    tick 個別関数 (5) / collect_notifications (3) / notification (2) /
    end_session (2) / report (2)
- 累計 **172 PASS** (148 + 24、F040-F047 既存 148 は非破壊維持)

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| simulation/paper_live/ 全 8 ファイル | ✅ |
| migrate_paper_live: 2 テーブル作成 | ✅ |
| 24 ケース全 PASS | ✅ |
| 累計 172 PASS (148 非破壊) | ✅ |
| CLI 4 コマンド動作 (start/tick/end/report) | ✅ |
| R-19-09 注意書き Markdown に含む | ✅ |

## 関連リンク

- 要件書: [[FIRE_要件書_第19章_Simulation___Paper_Live_方針]]
- 要件 ID: R-19-01 (Simulation 必須), R-19-02 (5 段階検証), R-19-03 (Paper Live 役割), R-19-09 (区別表示)
- 前タスク: [[F047_Optimistic_Bias_Realistic_Fill_Score]]
- 次タスク: F051 (仮想建玉計算 / 平均建値 / 評価損益 / 実現損益 / 余力管理) / F052 / F053
- コード: `~/fire/simulation/paper_live/`
- マイグレーション: `~/fire/scripts/setup/migrate_paper_live.py`
- テスト: `~/fire/tests/simulation/test_paper_live.py`

## スコープ外メモ

- **スケジューラ**: `tick()` を定時呼び出しする仕組みは F050 のスコープ外。
  P3 + F100 完了後に復帰タスクとして追加予定
- **LINE 実送信**: フックポイントのみ。実 SDK 統合は別タスク
- **仮想建玉計算 / 損益計算**: F051 のスコープ。F050 はイベント記録まで
- **F012 runtime/orchestrator/runner.py**: 既存 FIRERunner クラスは触っていない (将来統合)
