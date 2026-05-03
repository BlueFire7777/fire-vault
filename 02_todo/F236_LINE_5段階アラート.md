---
id: F236
phase: P5: 通知
priority: 中
status: 完了
owner: Fujiwara
depends_on: [F050, F054]
chapter: "06,38"
created: 2026-05-01
updated: 2026-05-02
tags: [P5, notifications, line, safety_net]
---

# F236: LINE 5 段階アラート (5 部屋構成 + 緊急ポジション整理)

## 概要

第 6 章 v3.3 の **5 部屋体制** (エントリー / 執行・損益 / レポート / システム警告 /
緊急ポジション整理) を LINE Messaging API で実装。緊急ポジション整理アラートは
macOS launchd で独立起動 (OpenClaw 障害時も稼働継続)。

v3.3 の重要メッセージ: **「LINE 通知は FIRE と Fujiwara の唯一の発注指示経路」** —
通知品質が運用品質を直結して決める安全装置。

要件根拠: 第 6 章 v3.3 (LINE 通知設計、5 部屋体制) / 第 38 章 (運用モード) /
第 20 章 R-20-03 (執行優位 14:45-15:15)

## 実装内容

### 主要モジュール (新規 8 ファイル)

- `notifications/__init__.py` (新規パッケージ): 公開 API export
- `notifications/line_bot.py`: `LineBotClient` (line-bot-sdk v3 ラッパー)
  - `dry_run` モード (env LINE_DRY_RUN=true で実送信スキップ、テスト用)
  - 失敗時リトライ (指数バックオフ、デフォルト 2 回)
  - 全送信履歴を `logs/notifications/notifications_line.log` に記録
- `notifications/router.py`: 5 部屋ルーティング
  - `Room` Enum 5 種 + `ROOM_ENV_MAP`
  - `get_room_id(room)`: 解決順 (専用 env → 部屋固有フォールバック → LINE_USER_ID)
  - `send_to_room(room, message)`: 統合送信 API
- `simulation/paper_live/notification.py` (修正):
  `send_to_line(notification: PaperLiveResult) -> dict` — `NotImplementedError` 解消
- `scripts/emergency_alert.py`: 5 段階緊急アラート
- `docs/launchd/`: README.md + 5 つの plist (平日のみ実行)

### 5 部屋ルーティング設計

| event_type | 行先部屋 |
|---|---|
| `candidate` / `virtual_entry` | **ENTRY** |
| `virtual_tp` / `virtual_sl` / `force_close` | **EXECUTION** |
| `notification` (forwarded) | source_event 参照 |
| その他 / 不明 | **SYSTEM** (フォールバック) |

### 5 段階緊急アラート

| 時刻 | 段階 |
|---|---|
| 14:45 | 第一次通知 |
| 14:55 | 第二次通知 |
| 15:05 | 第三次通知 (最重要) |
| 15:10 | 期限到達通知 |
| 15:15 | 最終確認 |

### キーポイント

- **dry_run モード必須**: `LINE_DRY_RUN=true` で実 LINE 送信せずテスト可能
- **エラー時フォールバック**: 部屋 ID 未設定 → `LINE_USER_ID` (Fujiwara 個人) に送信
- **緊急部屋の 2nd フォールバック**: `LINE_ROOM_EMERGENCY` → `LINE_EMERGENCY_GROUP_ID`
- **建玉 0 件は通知スキップ**: 第 6 章 v3.3 明記
- **launchd 独立起動**: OpenClaw 障害時も Mac mini 上で確実に発火

## .env 設定状況 (Step 0 確認結果)

| キー | 状態 |
|---|---|
| `LINE_CHANNEL_TOKEN` | ✅ 設定済 |
| `LINE_USER_ID` | ⚠️ 空文字 (Fujiwara 設定待ち) |
| `LINE_EMERGENCY_GROUP_ID` | ⚠️ 空文字 (Fujiwara 設定待ち) |
| `LINE_ROOM_*` 5 つ | ❌ 新規キー、`.env.example` で追加済 |

→ TOKEN は揃っているので実装は進行可。**実 LINE 送信は Fujiwara 側で部屋 ID 設定後**。

## テスト

- `tests/notifications/`: **24 / 24 PASS**
  - `test_line_bot.py` (7) / `test_router.py` (6) /
    `test_send_to_line.py` (5) / `test_emergency_alert.py` (6)
- 累計 **580 PASS** (556 → 580、F040-F058 + F100 既存全 PASS 維持)

## CLI smoke 結果 (dry_run、実 LINE 送信なし)

```
--- 1. send_to_room (5 部屋) ---
  entry / execution / report / system / emergency → 全て dry_run

--- 2. send_to_line (event_type 別) ---
  virtual_entry → room=entry / virtual_tp → room=execution /
  force_close → room=execution

--- 3. send_emergency_alert (各 stage、現 fire.db: open=1) ---
  14:45 (第一次通知) / 15:05 (第三次通知 最重要) / 15:10 (期限到達通知)
  全て dry_run、open_positions=1
```

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| line-bot-sdk v3 インストール | ✅ |
| notifications/line_bot.py + router.py 新規 | ✅ |
| send_to_line() NotImplementedError 解消 | ✅ |
| scripts/emergency_alert.py + launchd plists 5 個 | ✅ |
| docs/launchd/README.md セットアップ手順 | ✅ |
| .env.example に LINE_ROOM_* 5 つ追加 | ✅ |
| 24 ケース全 PASS | ✅ |
| F040-F058 既存 556 PASS 非破壊 | ✅ |
| 累計 580 PASS | ✅ |

## Fujiwara 側で必要な手動作業

1. `.env` に 5 部屋 ID + `LINE_USER_ID` を実値で設定
2. dry_run 解除して実送信テスト:
   `python -m scripts.emergency_alert --stage 14:45`
3. Mac mini で launchd 設定:
   ```bash
   cp ~/fire/docs/launchd/jp.fire.emergency-*.plist ~/Library/LaunchAgents/
   for s in 1445 1455 1505 1510 1515; do
     launchctl load ~/Library/LaunchAgents/jp.fire.emergency-${s}.plist
   done
   ```
4. システム設定 → エネルギーセーバー → 「コンピュータをスリープさせない」

## 関連リンク

- 要件書: 第 6 章 v3.3 / 第 38 章 / 第 20 章 R-20-03
- 関連: [[F050_Paper_Live_Mode_Stage_2_本体]] (notification.py フック)
- コード: `~/fire/notifications/` / `~/fire/scripts/emergency_alert.py` /
  `~/fire/docs/launchd/`

## スコープ外メモ

- LINE Webhook 受信 (Fujiwara から FIRE に返信) → 将来別タスク
- 後場新規エントリー禁止ルール → F252 関連で別タスク
- 楽天証券実建玉との集約 → F235 (Gmail API 連携)
- F022 / F242 常駐基盤統合 → 別タスク
