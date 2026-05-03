---
id: F116
phase: P3: Paper Live
priority: 最優先
status: 完了
owner: Fujiwara
depends_on: [F054, F236, F062]
chapter: "04,06"
created: 2026-05-02
updated: 2026-05-02
---

# F116: Monitoring & Alert Agent (Paper Live 価格到達 → LINE 利確/損切指示)

## 概要

F054 `monitor_virtual_positions` が生成する VIRTUAL_TP / VIRTUAL_SL イベントを
第 6 章 v3.3 EXECUTION 部屋向けメッセージに整形し、F236 経由で送信。
Paper Live 限定 (Live Advisory は将来 F260 / 別タスク)。

## スコープ

- `notifications/templates/exit.py`: 純粋関数 (format_tp_message /
  format_sl_message / format_exit_message)
- `agents/monitoring_alert.py`: MonitoringAlertAgent + MonitoringAlertResult
- `paper_live_notifications` テーブル新規 (CREATE TABLE IF NOT EXISTS、idempotent)
- duplicate 防止: (position_id, event_type) 複合キーで二重通知抑止
- 補助情報: paper_live_positions から avg_entry_price / entry_dt を取得して
  保有期間 + 損益を計算
- F236 send_to_room(Room.EXECUTION) 経由で送信
- 送信エラーは継続 (1 件失敗で他は処理続行)、`skip_send=True` でテスト用回避

## 重要な決定事項

- **paper_live_positions の実カラムは `entry_dt`** (仕様書の `opened_at`
  ではない)。`position_id` は INTEGER。F132 と同じ確認結果を再利用。
- **TickContext は simulation/paper_live/tick.py に定義**
  (仕様書の models.py ではない)。Smoke 用に `target_patterns=[]` /
  `fill_model='strict'` の必須引数あり。
- **paper_live=True がデフォルト**: Stage 2 中は常に Paper Live マーク、
  Stage 3 移行時に呼び出し側で False に切り替え。
- **テンプレートとエージェントを分離**: 純粋関数 (テンプレート) と DB 副作用
  (エージェント) を疎結合に。テンプレート単体テストが軽量。

## 成果物

### 新規ファイル

- `notifications/templates/exit.py`: TP/SL/exit メッセージ整形 + PAPER_LIVE_MARK
- `agents/monitoring_alert.py`: MonitoringAlertAgent (process_events) +
  duplicate 判定 + 補助情報計算 + paper_live_notifications テーブル管理
- `tests/notifications/templates/test_exit.py`: 13 件
- `tests/agents/test_monitoring_alert.py`: 10 件

### 変更ファイル

- `notifications/templates/__init__.py`: F116 シンボル export 追加

### テスト

- 新規 23 件 (F116 関連)
  - exit テンプレート 13 件 (TP 3 + SL 2 + paper_live 2 + 補助 3 + dispatch 3)
  - monitoring_alert 10 件 (process 4 + duplicate 3 + 補助情報 2 + エラー 1)
- 累計: 732 → **755 PASS** (+23)
- 既存 732 への影響: **0 件** (F054/F236 既存ロジックは F116 を呼ばない)

### Smoke 結果

```
=== TP メッセージ (Paper Live) ===
【利確指示】
銘柄: 7203
利確: 1,575 円で 100株 売り (信用買返済)
保有期間: 約2時間
損益: +7,500円

※ Paper Live (仮想売買、本番取引ではありません)

=== SL メッセージ (Paper Live) ===
【損切指示】
銘柄: 7203
損切: 1,470 円で 100株 売り (信用買返済)
保有期間: 約4時間
損益: -3,000円

※ Paper Live (仮想売買、本番取引ではありません)
```

F054 → F116 連結 (空 DB):
```
F054 events: 0
F116: input=0, sent=0, skipped=0, failed=0
```

## Paper Live 実用ループ完成

```
[エントリー側]
F054 候補抽出 → F111 厳選 → F115 4段ガード (F133/F132/F140) →
F062 LINE 整形 → F236 ENTRY 部屋

[退出側 (新規)]
F054 TP/SL 判定 → F116 LINE 整形 (duplicate 防止 + 補助情報) →
F236 EXECUTION 部屋
```

## スコープ外 (将来別タスク)

- TP/SL タッチ判定 → F054 既実装
- 緊急ポジション整理アラート (14:45-15:15) → F236 既実装
- 強制クローズ通知 (15:10) → F054 evaluate_force_close + F236
- 特買い/特売り検知 → F142
- 約定報告解釈 (LINE コマンド受信) → F063
- 当日損益更新通知 → F072 (引け後レビュー)
- Live Advisory モード対応 → F260 / 別タスク
- OpenClaw monitoring_alert agent への登録 → F260

## 関連リンク

- 要件書: 第 4 章 R-04-11 / 第 6 章 v3.3 EXECUTION 部屋
- 関連: [[F054_Paper_Live_tick]] (イベント生成元) /
  [[F236_LINE_5段階アラート]] (送信基盤) /
  [[F062_LINE注文完成形テンプレート]] (姉妹タスク)
- コード: `notifications/templates/exit.py` /
  `agents/monitoring_alert.py`
