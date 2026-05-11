---
id: FIRE-OPS-R0
phase: P9: 運用基盤 / F282 環境分離
priority: 最優先 (= F062-R5 系再開のブロッカー)
status: 調査完了 (root cause 確定、復旧案待ち HQ 判断)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F282 環境分離 (= 週次 staging snapshot 実装)
  - F062-R5.2 (= 本問題で停止した直接タスク)
chapter: 第 10 章 (F282 環境分離) / 第 26 章
---

# FIRE-OPS-R0: Staging DB Rollback Root Cause Audit

最終更新: 2026-05-11

## 結論 (root cause)

★ **F282 環境分離の週次 staging snapshot (cron: 月曜 07:00 JST) が
設計通りに動作。** 案 A / R2 系で staging に書いた本番運用データが
production fire.db スナップショットで上書きされ消失。

- crontab: `0 7 * * 1 /Users/bluefire/fire/bin/fire-snapshot-staging.sh`
- 2026-05-11 (月曜) 07:00 JST に script 実行
- staging.db mtime: **2026-05-11 07:00:05** ← cron 時刻と一致 ★
- snapshot_log.txt 末尾:
  `2026-05-10T22:00:00Z production -> staging snapshot OK (.backup)`
  (= UTC、JST 07:00:00)

scripts/fire-snapshot-staging.sh の動作:
1. 既存 staging.db を `data/fire.staging.db.bak.<ts>` に退避 ★
2. WAL/SHM sidecar 削除
3. `sqlite3 fire.db ".backup fire.staging.db"` で **production fire.db
   を staging.db に上書き** (= SQLite Online Backup API)
4. integrity_check
5. production WAL checkpoint
6. 90 日以上前の bak 削除

### 巻き戻り発生推定時刻

| 観察 | 時刻 (JST) | 時刻 (UTC) |
|---|---|---|
| 案 A 完了 (staging に announcements +1,091 行) | 2026-05-11 01:35 | 2026-05-10 16:35 |
| F062-R5 production 送信 (sent=1) | 2026-05-11 01:50 | 2026-05-10 16:50 |
| 直近の F062-R5.1 commits | 2026-05-11 04 時台 | 2026-05-10 19 時台 |
| ★ snapshot cron 実行 | **2026-05-11 07:00:05** | **2026-05-10 22:00:00** |
| F062-R5.2 開始 / gate refuse 検知 | 2026-05-11 07:30 以降 | 2026-05-10 22:30 以降 |

## 調査結果 (詳細)

### 1. launchd
- `~/Library/LaunchAgents/`: fire 系 plist は 5 件 (jp.fire.emergency-1445
  / 1455 / 1505 / 1510 / 1515) のみ。すべて LINE 緊急アラート用 (= F236)、
  DB 操作なし。`ai.openclaw.gateway.plist` あり、DB touch 言及なし。
- `launchctl list | grep -iE "fire|openclaw"`: 上記 5 件 + ai.openclaw.gateway
  のみ、DB write 系なし。
- LaunchAgents 内 grep ("fire.staging.db" / "cp .*fire.*db" / "rsync" /
  "sqlite"): **0 件**

### 2. cron ★

```
$ crontab -l
# F282 環境分離: 週 1 staging snapshot (月曜 7:00 JST)
0 7 * * 1 /Users/bluefire/fire/bin/fire-snapshot-staging.sh >> /Users/bluefire/fire/data/cron.log 2>&1

# F282 環境分離: 月 1 develop 初期化 (毎月 1 日 7:00 JST)
0 7 1 * * /Users/bluefire/fire/bin/fire-init-develop.sh >> /Users/bluefire/fire/data/cron.log 2>&1
```

- `/etc/cron*`: 該当パス無し (= system cron 不使用)
- staging snapshot を月曜 07:00 JST に実行する F282 設計通りの仕掛け

### 3. shell history
- `~/.zsh_history` に `fire-snapshot` / `fire-init` / `cp.*staging` /
  `rsync.*fire` のいずれも **0 件**
- 別 Claude Code session / 手動 copy の痕跡なし
- → cron 自動実行のみが原因と確定

### 4. repo scripts
- `~/fire/bin/fire-snapshot-staging.sh`: F282 環境分離設計の
  公式 snapshot script。SQLite Online Backup API (`sqlite3 .backup`)
  経由で production -> staging 上書き
- `~/fire/bin/fire-init-develop.sh`: 月 1 develop 初期化、同様 API
- F282 関連 vault docs: `03_design/F282_environment_isolation_2026-05-08.md`、
  log.md milestone `[2026-05-08] F282 環境分離 (3 環境化) 完了`
- → F282 設計通りの実装、想定外の DB 操作 script は **存在しない**

### 5. file metadata
```
data/fire.db          May  7 16:12:38 2026  size=371,064,832  inode=1194717
data/fire.develop.db  May  7 18:14:26 2026  size=371,064,832  inode=139614166
data/fire.staging.db  May 11 07:00:05 2026  size=371,064,832  inode=168902927
```

- 3 DB ともに同 size 371,064,832 bytes (= production snapshot 由来)
- inode は別々 (= 別 file、hard link ではない)
- staging.db のみ 5/11 07:00 mtime (= 直近の snapshot 実行時刻)

### 6. snapshot 前の bak ファイル ★ 朗報

```
data/fire.staging.db.bak.20260511_070004   size=4,803,829,760 (4.8 GB)
                                            mtime=5月 11 01:35
```

- 案 A / R2 系で書いた本番運用データを **完全保持**:
  - market_prices_daily: **max=2026-05-08 / 2,085,284 行** ★
  - research_watchlist_signals: max_base_date=2026-05-09 / **13,551 行** ★
  - research_derived_indicators: max_base_date=2026-05-08 / **3,750 行** ★
  - announcements: max=2026-05-08 / **1,098 行** ★
- → **このファイルから restore すれば F062-R5.2 に必要な staging 状態を即座に復元可能**

### 7. recent logs
- `logs/notifications/notifications_line.log` のみ直近 1 日 mtime
- snapshot 言及 0 件 (= LINE 関連で snapshot は触らない)

## 疑わしい process / script / launchd / cron

| | 結果 |
|---|---|
| launchd (jp.fire.* / ai.openclaw.*) | snapshot に **無関係** (LINE 緊急アラートのみ) |
| ★ cron (fire-snapshot-staging.sh、月曜 07:00 JST) | **本問題の原因** |
| shell history (cp / rsync 手動 copy) | 該当 0 件 (= 別 session 操作の痕跡なし) |
| OpenClaw / FIRE Runner 系の未完成処理 | DB 操作の痕跡なし |
| Claude Code 別 session | 痕跡なし |
| stagingを develop/production から再作成する仕様 | **F282 設計通り**、新規発見ではなく既存設計 |

## root cause 候補 (= 確定)

**表面的原因**: F282 環境分離の週次 staging snapshot (cron) が設計通り
動作。production fire.db を staging.db に `.backup` 経由で上書き。

**深層原因**: 過去タスク (F286-DATA-R1.3 / 案 A / F286-R2 系) で「**本番
運用データを staging に書く**」と判断していたが、F282 設計では staging
は週次で production snapshot で上書きされる「**実験用 DB**」。本番運用
データは production fire.db に書くべき。この前提認識のズレが累積し、
週次 snapshot で「本番データが消えた」ように見える事態に。

## 再発防止策 (要 HQ 判断)

| 案 | 内容 | 影響 |
|---|---|---|
| **案 1 (推奨)**: 本番運用データを production fire.db に書く運用に統一 | fetch_historical / fetch_tdnet_html / signals 生成パイプラインの target DB を production に変更 | F282 設計尊重、最も小さい影響 |
| 案 2: F282 snapshot を月曜以外に変更 | cron 時刻調整 (= 月末日曜深夜など) | 設計変更小、根本解決ではない |
| 案 3: F282 snapshot を一時停止 (cron 削除) | staging を本番運用 DB に格上げ | F282 設計の全面見直し、影響大 |
| 案 4: staging 用「不変データレイヤ」追加 | snapshot 対象外の table を別 .db に分離 | 実装大、複雑度増 |

**併走改善案**:
- CLAUDE.md / 03_design/F282_environment_isolation_*.md に
  「本番運用データは production、staging は実験用 (= 週次で消える)」
  を明示
- 各 production write job (= fetch_historical / fetch_tdnet_html /
  research signals 生成) で default DB を production fire.db に
- F286-DATA-R3 daily refresh cron 化を案 1 とセットで設計 (= production
  に書く daily refresh)

## staging 再構築に進んでよいか

**判断**: 本タスク仕様「staging 再構築はまだしない」を遵守、本タスク
内では実施しない。HQ (Fujiwara) が以下のいずれかを選択後に着手:

- 案 R1 (即時復旧): `cp data/fire.staging.db.bak.20260511_070004
  data/fire.staging.db` で **bak ファイルから 1 コマンド restore**
  (= 案 A 完了状態に巻き戻し、F062-R5.2 を再開可能)
- 案 R2 (本格): production fire.db に必要データを書き込み、次の月曜
  snapshot で staging も整合させる (= 再発防止策案 1 と整合、ただし
  時間がかかる)
- 案 R3 (延期): 5/12 平日 daily refresh 後に再判断

## 復旧案 R1 の dry-run プレビュー (= 実行はしない)

```
# 復旧コマンド (= 本タスクで実行しない、HQ 承認後に Fujiwara が実施)
cp data/fire.staging.db.bak.20260511_070004 data/fire.staging.db.restore
rm -f data/fire.staging.db-wal data/fire.staging.db-shm
mv data/fire.staging.db.restore data/fire.staging.db

# 整合性 check
sqlite3 data/fire.staging.db "PRAGMA integrity_check;"

# DATA-R2 gate 再確認
.venv/bin/python -m scripts.jobs.run_data_freshness_gate \
  --db-path data/fire.staging.db --db-label staging \
  --output-json /tmp/f062_r5_3_gate.json
```

期待: overall=pass / line_send_allowed=True / 5 段全 PASS。

## 安全要件遵守 (本タスク)

| 項目 | 結果 |
|---|---|
| DB write                                          | 0 ✅ (read-only 調査のみ) |
| staging 再構築                                    | 実施せず ✅ (本タスク仕様遵守) |
| LINE 送信                                         | 0 ✅ |
| production / develop DB に触れる                   | していない (= read-only stat / sqlite mode=ro のみ) ✅ |
| 自動発注 / 楽天操作 / Computer Use                 | 0 ✅ |
| TODO Excel 未更新                                  | ✅ |
| --no-verify                                       | 不使用 ✅ |
| unrelated modified を stage / commit              | しない ✅ |
| scripts/seed_pattern_layer1.py / historical_indicators.py | 未接触 ✅ |
| token / recipient leak                             | N/A (= 本タスクで触らず) |

## 次タスク

1. ★ FIRE-OPS-R0 完了 (= 本書記録、root cause 確定)
2. HQ (Fujiwara) が以下を判断:
   - 復旧: 案 R1 (bak から restore、推奨) / R2 / R3 を選択
   - 再発防止: 案 1 (本番データは production)、案 2-4 のいずれかを選択
3. 案 R1 採用なら:
   - bak から staging restore (= 1 コマンド)
   - DATA-R2 gate 再確認
   - F062-R5.2 production + compact 再起動
4. 並走候補:
   - F286-DATA-R3 daily refresh cron 化 (= 再発防止策 1 と整合)
   - F242 OpenClaw / F022 FIRE Runner / F013 launchd
   - 03_design/F282_environment_isolation_*.md の運用ルール明確化
