---
id: F062-morning-advisory-launchd
phase: 本番 v0 Launch / Phase C Foundation
priority: 高
status: 設計 v1.0 (= Wave 40-post、設計のみ、配置/load/write/API/LINE/token 0)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 38 (= v0 Launch Plan v1.0、D-Day 2026-06-09)
  - Wave 40-pre (= F286-DATA-R3 daily refresh launchd 設計 v1.0)
  - F062-R5.8 (= 1 chunk send + send_guard + name_enrichment 既存資産)
  - F286-PNL-R2 (= record-decisions 既存実装、advisory_decisions full schema)
  - F286-DATA-R2 (= freshness gate 既存、agents/data_freshness_gate.py)
related:
  - 03_design/FIRE_production_v0_launch_plan_2026-05-13.md
  - 03_design/F286_DATA_R3_daily_refresh_launchd_2026-05-13.md
  - 03_design/F282_weekly_snapshot_launchd_2026-05-12.md
chapter: production-v0 / Phase C / F062 morning advisory / launchd
---

# F062 Morning Advisory Launchd + No-Send Trial 設計 — Wave 40-post

最終更新: 2026-05-13

## 0. 要約

v0 Phase C foundation として、F062 morning advisory を月-金 08:45 JST launchd
起動する **設計のみ**。F286-DATA-R3 (= Phase B) 完了 + freshness gate OK を
入力前提とし、preview / no-send mode で 1 週間試走する Wave 46+ への基盤。
**実 plist 配置 / launchctl load / DB write / API / LINE / token 参照 0**。

### Wave 40-post 構成 (= 第 1 陣 8 lane + 第 2 陣 4 lane 二段投入)

| lane | task_id | 担当 | elapsed | verdict |
|---|---|---|---|---|
| L5  | 本線統合 | plan / 設計 doc / vault / HQ 報告 | 全 wave | — |
| L1a | F062 launchd architecture | Codex | 23s | GO with 2 concerns |
| L1b | DATA-R3/freshness/F282 dependency | Codex | 30s | OK / no conflict |
| L2a | no-send trial plan | Codex | 27s | READY |
| L2b | duplicate prevention + record-decisions | Codex | 32s | READY |
| L3  | plist XML draft | Codex | 30s | READY for review |
| L4  | adversarial audit 8 観点 | Codex | 29s | **GO with 0 concerns** |
| L6  | regression + F282/DATA-R3 review | Codex | 21s | unchanged confirmed |
| L7a | token/secret/LINE 経路 static audit | Codex 2 陣 | 128s | clean / 2 findings (LOW) |
| L7b | no-send trial audit | Codex 2 陣 | 38s | SAFE |
| L7c | duplicate prevention audit | Codex 2 陣 | 44s | SOUND |
| L7d | smoke plan 強化 | Codex 2 陣 | 35s | strengthened |

**L4 verdict: GO for integration with 0 concerns** (= CRITICAL 0 / HIGH 0)。
L1a 2 concerns は L4 audit で完全吸収。

**第 2 陣 4 lane 全 CRITICAL 0 / HIGH 0**。L7a 2 findings (LOW) を §1 + §4 に反映:
- send_guard は独立 helper module ではなく、F062 既存パターン
  `--hq-approved-send` 引数 absent → 送信拒否
- chunk_length=738 は履歴 doc 値、ソース実装は `max_chunks=1` で 1 chunk 制限

---

## 1. plist 骨子 (= L1a + L3 + L7a findings 反映)

### file 配置 (= 設計のみ、本 wave 配置 0)

- docs path: `~/fire/docs/launchd/jp.fire.morning-advisory.plist`
- 実配置 path: `~/Library/LaunchAgents/jp.fire.morning-advisory.plist`
  (= 別 wave + `HQ_APPROVE_LAUNCHD_MORNING_ADVISORY` 必須)

### plist XML 骨子

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>jp.fire.morning-advisory</string>

    <key>ProgramArguments</key>
    <array>
        <string>/Users/bluefire/fire/.venv/bin/python</string>
        <string>/Users/bluefire/fire/scripts/jobs/run_f062_research_advisory_line_preview.py</string>
        <string>--dry-run</string>
        <string>--no-send</string>
        <string>--preview-only</string>
    </array>

    <key>EnvironmentVariables</key>
    <dict>
        <key>FIRE_ENV</key>
        <string>staging</string>
        <key>PYTHONPATH</key>
        <string>/Users/bluefire/fire</string>
    </dict>

    <key>WorkingDirectory</key>
    <string>/Users/bluefire/fire</string>

    <key>StartCalendarInterval</key>
    <array>
        <dict><key>Weekday</key><integer>1</integer><key>Hour</key><integer>8</integer><key>Minute</key><integer>45</integer></dict>
        <dict><key>Weekday</key><integer>2</integer><key>Hour</key><integer>8</integer><key>Minute</key><integer>45</integer></dict>
        <dict><key>Weekday</key><integer>3</integer><key>Hour</key><integer>8</integer><key>Minute</key><integer>45</integer></dict>
        <dict><key>Weekday</key><integer>4</integer><key>Hour</key><integer>8</integer><key>Minute</key><integer>45</integer></dict>
        <dict><key>Weekday</key><integer>5</integer><key>Hour</key><integer>8</integer><key>Minute</key><integer>45</integer></dict>
    </array>

    <key>StandardOutPath</key>
    <string>/Users/bluefire/fire/logs/cron/morning-advisory.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/bluefire/fire/logs/cron/morning-advisory.err</string>

    <key>LimitLoadToSessionType</key>
    <string>Aqua</string>

    <key>RunAtLoad</key>
    <false/>
</dict>
</plist>
```

### EnvironmentVariables 制約 (= 機密漏洩防止)

**plist `EnvironmentVariables` に絶対含めない**:
- `LINE_CHANNEL_TOKEN` / `LINE_USER_ID` / `LINE_ROOM_*` / `LINE_EMERGENCY_GROUP_ID`
- `JQUANTS_API_KEY` / `JQUANTS_*`
- `CHANNEL_*`
- 一切の secret / token

明示する env:
- `FIRE_ENV=staging` (= 三段ガード対応)
- `PYTHONPATH=/Users/bluefire/fire`

### 既存 plist との命名整合

| plist | 用途 | schedule |
|---|---|---|
| jp.fire.emergency-{HHMM}.plist (5 個) | F236 緊急アラート | 14:45/14:55/15:05/15:10/15:15 |
| jp.fire.weekly-snapshot.plist | F282 weekly snapshot | 土曜 02:00 |
| jp.fire.daily-refresh.plist | DATA-R3 daily refresh | 月-金 06:30 |
| **jp.fire.morning-advisory.plist (本 doc)** | **F062 morning advisory** | **月-金 08:45** |

→ Label 衝突 0、namespace `jp.fire.*` 完全準拠。

### 試走時の引数設計 (= L7a Finding 反映)

| 引数 | 役割 |
|---|---|
| `--dry-run` | DB write 0 / API call 0 |
| `--no-send` | LINE 送信経路を強制 abort |
| `--preview-only` | preview text 生成のみ |
| `--hq-approved-send` (= **付与しない**) | 既存 send_guard pattern、absent で送信拒否 |

source レベル制約:
- `max_chunks=1` (= L7a Finding 2: chunk_length=738 ではなく chunk 数で 1 chunk 制限)
- `partial_delivery=False` (= F062-R5.8)

---

## 2. log path 設計 (= L2b + L7d 反映)

### log 配置

```
/Users/bluefire/fire/logs/
├── cron/
│   ├── weekly-snapshot.{log,err}        ← F282 (= 別 file)
│   ├── daily-refresh.{log,err}          ← DATA-R3 (= 別 file)
│   └── morning-advisory.{log,err}       ← 本 doc (新規)
└── archive/YYYY-MM/                     ← logrotate olddir
    └── morning-advisory.{log,err}-YYYY-MM.gz
```

### log 内容 (= morning-advisory runner 出力規約)

- 開始: `F062 morning advisory preview start (base_date=YYYY-MM-DD)`
- DATA-R3 結果確認: `DATA-R3 completion-report=...` / `DATA-R3 exit=0`
- freshness gate: `freshness gate=OK` / `freshness gate=NG`
- preview 生成: preview file path
- send: `LINE send=0 (--no-send active)`
- record-decisions: `record-decisions skipped (preview-only)`
- 完了: `F062 morning advisory preview OK`
- 異常時: `ERROR` / `REFUSED` を含む line
- F282 干渉防止: `F282 unchanged confirmed` line

### logrotate (= Wave 45+ 別配置)

`/etc/newsyslog.d/jp.fire.conf` に morning-advisory.{log,err} 追加 plan、30 日 retention。

---

## 3. DATA-R3 / freshness gate / F282 依存設計 (= L1b 反映)

### 月-金 時系列

```
06:30  jp.fire.daily-refresh (DATA-R3)
  ↓ aggregate runner (F100/F101/F111/F119) --dry-run
  ↓ aggregate_dry_run_exit_code=0
  ↓ logs/cron/daily-refresh.log 確認
~08:30 (freshness gate 起動 or F062 内部で確認)
  ↓ freshness OK 判定
08:45  jp.fire.morning-advisory (本 doc)
  ↓ DATA-R3 結果確認 → freshness gate 確認
  ↓ preview 生成 (= LINE template、実送信 0)
  ↓ record-decisions skipped (= preview-only)
  ↓ exit 0
```

### F282 不干渉

| 項目 | F282 | DATA-R3 | morning-advisory |
|---|---|---|---|
| Label | jp.fire.weekly-snapshot | jp.fire.daily-refresh | jp.fire.morning-advisory |
| schedule | 土曜 02:00 | 月-金 06:30 | 月-金 08:45 |
| DB scope | production read → staging/develop copy | staging-only write (= 別 wave) | staging read のみ (= 本 wave preview) |
| log path | weekly-snapshot.{log,err} | daily-refresh.{log,err} | morning-advisory.{log,err} |
| 時刻重複 | 0 | 0 | 0 |
| label 衝突 | 0 | 0 | 0 |

### 月曜運用順序

```
土曜 02:00 F282 weekly snapshot (= 金曜分まで反映)
   ↓ (土日休止)
月曜 06:30 DATA-R3 daily refresh (= 土日含む 2 営業日分追加 fetch)
   ↓
月曜 08:45 F062 morning advisory (= DATA-R3 完了 + freshness OK 前提)
```

順序競合 0、DB write target 競合 0。

### 緊急アラート干渉 0

| 時間 | label |
|---|---|
| 06:30 | jp.fire.daily-refresh |
| 08:45 | jp.fire.morning-advisory (本 doc) |
| 14:45/14:55/15:05/15:10/15:15 | jp.fire.emergency-{HHMM} |
| 土 02:00 | jp.fire.weekly-snapshot |

→ 全時刻重複 0。

---

## 4. freshness gate 接続設計

### 確認内容

- staging DB の market_prices_daily / announcements の base_date 当日
- mtime 当日朝以降
- DATA-R3 完了 log (= "F286-DATA-R3 daily_refresh dry-run OK") の有無

### NG 時の挙動

- F062 advisory 生成 0
- LINE preview 生成 0
- exit 0 (= silent abort、launchd は次回 schedule まで待機)
- log line `freshness gate=NG, advisory aborted`
- report 出力: `/tmp/f062_morning_advisory_completion_YYYY-MM-DD.txt` に NG 理由

### HQ 報告項目 (= 本番 v0 開始後)

- freshness gate NG 件数 (= daily)
- 連続 NG 検出 (= 2 日連続で異常通知)
- DATA-R3 結果との整合

---

## 5. no-send trial plan (= L2a + L7b + L7d 反映)

### 目的

- launchd 経路で F062 morning advisory が正常起動
- DATA-R3 完了 + freshness OK で advisory 生成
- LINE preview のみ、実送信 0
- token 不参照で動作確認
- 1 chunk (= max_chunks=1) 制限遵守
- 1 週間 (= 月-金 5 営業日)

### 期間

Wave 46 候補 (= v0 Launch Plan §5 Phase D)、想定: 2026-06-02 〜 2026-06-09 (D-Day 前)

### daily 確認項目

- `logs/cron/morning-advisory.log` mtime = 試走日
- "F062 morning advisory preview OK" log line
- exit 0
- LINE 送信痕跡 0 (= `LINE send=0 (--no-send active)`)
- preview file 生成 (= /tmp/f062_morning_preview_YYYY-MM-DD.txt 等)
- record-decisions skipped log

### token 不参照確認 (= L7a Finding 1 反映)

- plist EnvironmentVariables に LINE_* / CHANNEL_* / JQUANTS_* 0
- log に token-like 値 0
- `--hq-approved-send` 引数 absent → send_guard refuse

### DB / API / LINE 0 確認

- production / develop DB mtime 完全 unchanged
- staging DB mtime read のみ (= 本 wave 試走中)
- J-Quants / TDnet API call 0 (= dry-run)
- LINE Messaging API call 0

### record-decisions 連携方針 (= L2b)

- 本 wave 試走中: DB write 0 (= advisory_decisions に INSERT 0)
- 別 wave (= 本番投入前) で staging DB write を HQ 承認後追加
- production DB write は v0 D-Day 後 HQ_APPROVE_LINE_TOKEN_PRODUCTION 経由

### GO/NO-GO 判定

- GO: 5 営業日全 PASS + LINE 0 + token 0 + DB write 0
- NO-GO: 任意失敗 → 修正 + 再試走

---

## 6. duplicate prevention + record-decisions 設計 (= L2b + L7c 反映)

### duplicate 範囲

| scope | 方法 |
|---|---|
| 同日二重 send 防止 | launchd 08:45 月-金 1 回 + LimitLoadToSessionType=Aqua |
| 同日二重 record-decisions 防止 | advisory_decisions PRIMARY KEY (advisory_id, code) UNIQUE + INSERT OR IGNORE |
| preview 再実行時 | preview file overwrite 可、DB write 0 |
| chunk 数超過 | `max_chunks=1` 超過時 send 0 + ERROR log + abort |

### advisory_id 命名

```
morning_advisory_YYYY-MM-DD_<source_version>
```

### send_id

```
LINE 送信 1 回 = 1 send_id (chunk_idx 含む)
本 wave 試走では生成 0 (= LINE send 0)
```

### idempotency

同日 2 回起動 → 2 回目 exit 0 + send 0 + record-decisions 0 件 (= silent skip)。

### partial_delivery=False enforcement (= F062-R5.8 既存)

- 1 chunk 制限 (= max_chunks=1)
- chunk 数超過時 send 0 + ERROR log + abort
- test 想定: `test_chunk_count_exceeds_one_aborts`

---

## 7. abort 条件 (= 9 項目)

| 条件 | action |
|---|---|
| DATA-R3 未完了 (= daily-refresh.log 不在 or exit non-zero) | F062 abort + exit 0 + report |
| freshness NG | F062 abort + exit 0 + report |
| LINE token が必要になった (= 設計違反検出) | 即 stop + token rotate |
| send_guard が送信許可しそうになった | 即 stop + 調査 |
| DB write が必要になった | 即 stop + 別 wave HQ 承認 |
| duplicate 検出 (= advisory_decisions UNIQUE conflict) | 2 回目以降 silent skip、exit 0 |
| exit non-zero | launchd 次回 schedule まで再起動なし |
| F282 干渉検出 (= weekly-snapshot mtime 変化) | 即 stop + 調査 |
| log 出力失敗 (= disk full) | launchd 自動異常終了 |

---

## 8. F282 + DATA-R3 不干渉確認 (= L6 反映)

### 本 wave 進行中の不変項目

- F282 本番 plist mtime=1778593597 size=1772 完全不変
- F282 launchctl LastExit 0 / state=not running / 5/16 02:00 維持
- DATA-R3 (= 前 wave 設計) plist は本 wave で touch 0
- ~/Library/LaunchAgents/ に morning-advisory.plist 不在
- ~/fire/docs/launchd/ に morning-advisory.plist 不在
- 3 環境 DB + W30 snapshot 全 unchanged
- pytest 4090 collected 維持 (= python code 変更 0)

---

## 9. Wave 41 以降 最短順 (= v0 Launch Plan 反映)

```
Wave 40-pre (= DATA-R3 設計) 完了
Wave 40-post (= F062 設計、本 doc) 完了
   ↓
5/16 02:00 F282 本番試走
5/19 F282 GO/NO-GO 判定
   ↓ (GO 時)
Wave 41: DATA-R3 plist 実配置 + launchctl load + 1 週間 no-write 試走
         (= HQ_APPROVE_LAUNCHD_DAILY)
   ↓ (5/19-5/26)
Wave 45: F062 morning advisory plist 実配置 + launchctl load + 1 週間 no-send 試走
         (= HQ_APPROVE_LAUNCHD_MORNING_ADVISORY)
   ↓ (6/2-6/9)
Wave 52: production LINE token 投入
         (= HQ_APPROVE_LINE_TOKEN_PRODUCTION)
Wave 53: D-Day 朝 advisory 実送信 1 回目 (= 2026-06-09 想定)
         (= HQ_APPROVE_PRODUCTION_V0_LAUNCH)
```

---

## 10. 安全要件 (= 本 doc 自体)

| 項目 | 結果 |
|---|---|
| 実 plist 配置 | 0 (= docs/launchd/ + LaunchAgents/ 共に不在) |
| launchctl load / unload | 0 |
| cron / launchd / crontab 登録変更 | 0 |
| DB write | 0 |
| LINE 送信 | 0 |
| token / channel_token / secret 参照 | 0 |
| env 全体読み取り | 0 |
| 実 API call | 0 |
| F282 試走干渉 | 0 |
| DATA-R3 設計干渉 | 0 |
| F101 staging probe | 0 |
| 楽天 / 自動発注 / Computer Use | なし |
| workflow / --no-verify / TODO Excel | 0 |
| Codex 直接 commit | 0 |
| pytest 実行 | 0 (= 4090 PASS 静的維持) |

---

## 11. 関連リンク

- [[FIRE_production_v0_launch_plan_2026-05-13|v0 Launch Plan v1.0]]
- [[F286_DATA_R3_daily_refresh_launchd_2026-05-13|DATA-R3 launchd 設計 (前 wave)]]
- [[F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[F282_temporary_launchd_smoke_2026-05-13|F282 temporary smoke (Wave 39-temp)]]
- [[../02_todo/FIRE_CODEX_R1_WAVE40_POST_plan|Wave 40-post plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE40_POST_results|Wave 40-post results]]
- `/tmp/codex_wave40post/results/l{1a,1b,2a,2b,3,4,6,7a,7b,7c,7d}.txt` (= 11 lane stdout)
- 既存 runner:
  - `~/fire/scripts/jobs/run_f062_research_advisory_line_preview.py`
  - `~/fire/scripts/jobs/run_f062_line_production_send_smoke.py` (= F062-R5.8)
  - `~/fire/agents/data_freshness_gate.py`
  - `~/fire/pnl/storage.py` + `pnl/snapshot.py`
