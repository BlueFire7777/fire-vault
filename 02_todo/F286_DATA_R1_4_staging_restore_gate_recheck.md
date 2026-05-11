---
id: F286-DATA-R1.4
phase: P5: ResearchLane R1 / 運用復旧
priority: 最優先 (= F062-R5.2 再開のブロッカー解除)
status: 完了 ★ (2026-05-11、staging restore + DATA-R2 gate pass 復活)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-OPS-R0 (= 巻き戻り root cause 確定、bak ファイル存在確認)
  - F282 環境分離 (= weekly snapshot 設計、本タスクで bak から逆復旧)
  - F062-R5.2 (= 本問題で停止、本タスク完了で再開可能に)
chapter: 第 10 章 (F282 環境分離) / 第 26 章
---

# F286-DATA-R1.4: Staging Restore from Backup / Gate Recheck

最終更新: 2026-05-11

## ★ 状態: 完了

FIRE-OPS-R0 で確定した bak ファイル
`data/fire.staging.db.bak.20260511_070004` から staging.db を 1
コマンド restore、DATA-R2 gate を再確認して **overall=pass /
line_send_allowed=True / 5 段全 PASS** に復活。production / develop
は完全に unchanged。F062-R5.2 を再開可能な状態。

## 実施内容

### Step 1: 作業前 3 DB 状態記録 (pre)

| DB | mtime | size | inode |
|---|---|---|---|
| data/fire.db          | May  7 16:12:38 2026 | 371,064,832 | 1194717 |
| data/fire.develop.db  | May  7 18:14:26 2026 | 371,064,832 | 139614166 |
| data/fire.staging.db  | May 11 07:00:05 2026 | 371,064,832 | 168902927 |

backup file:
- `data/fire.staging.db.bak.20260511_070004`
- mtime: May 11 01:35:30 2026
- size: 4,803,829,760 bytes (4.8 GB)

sidecar (restore 前):
- `data/fire.staging.db-shm` (32 KB, May 11 10:56)
- `data/fire.staging.db-wal` (0 bytes, May 11 07:00)

### Step 2: 現 staging.db 退避 ✅

```
cp data/fire.staging.db data/fire.staging.db.pre_restore_20260511_112053
```

退避ファイル:
- `data/fire.staging.db.pre_restore_20260511_112053`
- size: 371,064,832 bytes (= 巻き戻り後の 371 MB スナップショット保持)

### Step 3: sidecar 削除 ✅

```
rm -f data/fire.staging.db-wal data/fire.staging.db-shm
```

理由: F282 snapshot script と同じ方針 (= 前 DB に紐付いた stale sidecar
が新 DB の参照を corruption させるリスクを排除)。新規 sidecar は
SQLite が次回 open 時に自動生成。

### Step 4: backup → staging.db restore ✅

```
cp data/fire.staging.db.bak.20260511_070004 data/fire.staging.db
```

結果:
- size: 371,064,832 → **4,803,829,760 bytes** (= bak と同一)
- inode は同一 (168902927、Mac filesystem の inode 再利用挙動)
- mtime: May 11 11:20:59 (= cp 完了時刻)

### Step 5: integrity_check ✅

```
sqlite3 data/fire.staging.db "PRAGMA integrity_check;"
```

結果: **ok**

### Step 6: DATA-R2 gate 再実行 (read-only) ✅

```
.venv/bin/python -m scripts.jobs.run_data_freshness_gate \
  --db-path data/fire.staging.db --db-label staging \
  --output-json /tmp/f286_data_r1_4_gate_after_restore.json
```

| field | result |
|---|---|
| overall_status                              | **pass** ✅ |
| line_send_allowed                           | **True** ✅ |
| as_of_date                                  | 2026-05-11 |
| db_label                                    | staging |
| reasons                                     | [] |
| gate-1-prices (required)                    | pass (max=2026-05-08 lag=1 / codes=4448) |
| gate-2-signals (required)                   | pass (max_base=2026-05-09 lag=0 / codes=109) |
| gate-3-index (recommended)                  | pass (max=2026-05-08 lag=1) |
| gate-4-derived (recommended)                | pass (max_base=2026-05-08 lag=1 / codes=42) |
| gate-5-other (soft)                         | pass (financials / announcements / listings OK) |

### Step 7: restored staging data 確認 ✅

| table | max_date / max_base_date | row_count |
|---|---|---|
| market_prices_daily               | 2026-05-08 | 2,085,284 |
| market_prices_daily distinct codes | -          | 4,452 |
| research_watchlist_signals        | 2026-05-09 | 13,551 |
| research_derived_indicators       | 2026-05-08 | 3,750 |
| announcements                     | 2026-05-08 | 1,098 |

→ FIRE-OPS-R0 で確認した bak content と完全一致。

### Step 8: production / develop unchanged 確認 ✅

| DB | pre / post mtime | 不変 |
|---|---|---|
| data/fire.db          | May  7 16:12:38 → May  7 16:12:38 | ✅ |
| data/fire.develop.db  | May  7 18:14:26 → May  7 18:14:26 | ✅ |
| data/fire.staging.db  | May 11 07:00:05 → **May 11 11:20:59** (= restore) | (本タスク対象) |

production / develop の mtime / inode / size 全て不変。本タスクでは
staging.db のみ書き込み (= cp ベース restore)。

## 安全要件遵守

| 項目 | 結果 |
|---|---|
| production / develop DB に触れる                   | 触れない (= mtime/inode/size 全 unchanged) ✅ |
| staging DB の書き込み                              | restore のみ (= cp による 1 回の上書き、pre_restore 退避済) ✅ |
| LINE 送信                                          | 0 ✅ |
| 自動発注 / 楽天操作 / Computer Use                  | 0 ✅ |
| 注文価格 / 数量 / 執行指示                          | 送信していない ✅ |
| TODO Excel                                         | 未更新 ✅ |
| --no-verify                                       | 不使用 ✅ |
| scripts/seed_pattern_layer1.py / historical_indicators.py | 未接触 ✅ |
| unrelated modified を stage / commit              | しない ✅ |
| token / recipient leak                             | N/A (本タスクで送信なし) |
| F062-R5.2 本体の送信                                | **実施せず** (= 本タスクは restore + gate 確認まで、送信は HQ 判断後) |

## F062-R5.2 再開可否

**★ 再開可能**。

- DATA-R2 gate 全 5 段 PASS / line_send_allowed=True ✅
- 案 A 完了状態 (= 5/8 prices / 5/9 signals / 5/8 announcements / 5/8
  derived) が完全に復元
- env (token length=516 / ASCII / no whitespace、recipient prefix='U'
  length=33) は F062-R5.1 commits 前から変更なし
- F062-R5.1 のコード修正 (production message mode / compact LINE UX
  / payload freshness guard) はすでに main develop branch に含まれて
  おり、本タスクで触らない

本タスクは「restore + gate 復活確認」までを完了。F062-R5.2 production
+ compact 送信は HQ (Fujiwara) 判断後に開始。

## 注意: F282 設計上のリスク (= 次回月曜 07:00 JST に巻き戻り再発)

本 restore は **応急処置**。F282 weekly staging snapshot は次回月曜
(= 2026-05-18 07:00 JST) に再実行され、現状の staging が production
fire.db で再び上書きされる。F062-R5.2 を実施するタイミングは:

- 案 X1 (即時推奨): 本タスク完了後すぐ F062-R5.2 を再起動
  (= 5/11 〜 5/17 の 1 週間以内に送信完了)
- 案 X2: 並行で FIRE-OPS-R0 「再発防止策案 1」(= 本番運用データを
  production に書く運用統一) を実装、次回 snapshot 後は production
  → staging snapshot で staging も自動整合

## 退避 / バックアップファイル

| 用途 | パス | size |
|---|---|---|
| F282 snapshot 退避 (= 案 A データ含む) | data/fire.staging.db.bak.20260511_070004 | 4.8 GB |
| 本タスク pre_restore 退避 (= 巻き戻り後の 371 MB) | data/fire.staging.db.pre_restore_20260511_112053 | 371 MB |

snapshot script は 90 日以上前の bak を自動削除する仕様
(`find data/ -name "fire.staging.db.bak.*" -mtime +90 -delete`)。
pre_restore ファイルはこの自動削除の対象外 (= filename pattern 違い)。

## commits

- fire (develop):     変更なし (= コード変更なし、運用 restore のみ)
- fire-vault (main):
  - (commit シーケンスで生成、本書記録の commits)

## 次タスク

1. ★ F286-DATA-R1.4 完了 (= 本書、staging restore + gate pass 確認)
2. **HQ (Fujiwara) 判断**: F062-R5.2 production + compact 再起動の
   タイミング (= 即時 / 翌営業日朝)
3. F062-R5.2 再起動時:
   - env 確認 → DATA-R2 gate 再確認 → production + compact payload
     生成 → dry-run → --send + --hq-approved-send で 1 chunk 実 send
4. 並走候補:
   - **FIRE-OPS-R0 再発防止策の実装** (= 案 1 推奨: 本番運用データを
     production fire.db に書く運用統一)
   - F286-DATA-R3 daily refresh cron 化 (= 案 1 と整合)
   - 03_design/F282_environment_isolation_*.md の運用ルール明文化
