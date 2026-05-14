---
id: F286-DATA-R3-daily-refresh-launchd
phase: 本番 v0 Launch / Phase B Foundation
priority: 高
status: 設計 v1.0 (= Wave 40-pre、設計のみ、配置/load/write/API 0)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 38 (= v0 Launch Plan v1.0、D-Day 2026-06-09)
  - Wave 39-temp (= F282 temporary smoke 成功、launchd 経路実証)
  - F282 weekly snapshot 設計 (= 03_design/F282_weekly_snapshot_launchd_2026-05-12.md)
  - 既存 runner: scripts/jobs/run_f286_data_r3_daily_refresh.py (aggregate)
chapter: production-v0 / Phase B / DATA-R3 / launchd
related_chapters: [F282, F236, F286-DATA-R3, F100, F101, F111, F119, F062]
---

# F286-DATA-R3 Daily Refresh Launchd 設計 — Wave 40-pre

最終更新: 2026-05-13

## 0. 要約

v0 Phase B (= 2026-05-19 F282 GO 判定後 〜 2026-05-26) の前倒し準備として、
F286-DATA-R3 daily refresh を launchd で月-金 06:30 JST 起動する **設計のみ**。
**実 plist 配置 / launchctl load / DB write / LINE / token 参照 / API call
0**。本 doc は次 wave 以降の impl の基盤となる設計初版。

### Wave 40-pre 構成 (= 8 lane)

| lane | task_id | 担当 | verdict |
|---|---|---|---|
| L5 | 本線統合 | plan / results / 設計 doc / vault / log / HQ 報告 | — |
| L1a | DATA-R3 launchd architecture | Codex | GO / 0 concerns |
| L1b | F282/freshness/F062 dependency | Codex | OK / no conflict |
| L2a | no-write trial plan | Codex | READY for next wave |
| L2b | log/monitoring/abort plan | Codex | READY |
| L3 | plist XML draft | Codex | READY for review |
| L4 | adversarial audit (8 観点) | Codex | **GO with 2 concerns** |
| L6 | regression / F282 干渉 review | Codex | F282 unchanged 確認 |

**L4 audit verdict: GO for integration with 2 concerns** (= CRITICAL 0 / HIGH 0)。
本 doc は L4 concerns 2 件を反映した最終版:

- CONCERN C: FIRE_ENV を `staging` に統一 (§1 反映済)
- CONCERN E: plist XML に `LimitLoadToSessionType=Aqua` を明示 (§1 反映済)

---

## 1. plist 骨子 (= L3 + L4 CONCERN C/E 反映)

### file 配置 (= 設計のみ、本 wave 配置 0)

- 配置予定 path (= docs): `~/fire/docs/launchd/jp.fire.daily-refresh.plist`
- 実配置 path: `~/Library/LaunchAgents/jp.fire.daily-refresh.plist`
  (= 別 wave + HQ_APPROVE_LAUNCHD_DAILY 必須)

### plist XML 骨子 (= L4 CONCERN C/E 反映後)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>jp.fire.daily-refresh</string>

    <key>ProgramArguments</key>
    <array>
        <string>/Users/bluefire/fire/.venv/bin/python</string>
        <string>/Users/bluefire/fire/scripts/jobs/run_f286_data_r3_daily_refresh.py</string>
        <string>--dry-run</string>
        <string>--execute-dry-run-subprocesses</string>
        <string>--db-path</string>
        <string>data/fire.staging.db</string>
        <string>--db-label</string>
        <string>staging</string>
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
        <dict><key>Weekday</key><integer>1</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>
        <dict><key>Weekday</key><integer>2</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>
        <dict><key>Weekday</key><integer>3</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>
        <dict><key>Weekday</key><integer>4</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>
        <dict><key>Weekday</key><integer>5</integer><key>Hour</key><integer>6</integer><key>Minute</key><integer>30</integer></dict>
    </array>

    <key>StandardOutPath</key>
    <string>/Users/bluefire/fire/logs/cron/daily-refresh.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/bluefire/fire/logs/cron/daily-refresh.err</string>

    <key>LimitLoadToSessionType</key>
    <string>Aqua</string>

    <key>RunAtLoad</key>
    <false/>
</dict>
</plist>
```

### EnvironmentVariables 制約 (= 機密漏洩防止)

**plist `EnvironmentVariables` に絶対含めない**:

- `LINE_CHANNEL_TOKEN`
- `LINE_USER_ID`
- `LINE_ROOM_*`
- `LINE_EMERGENCY_GROUP_ID`
- `JQUANTS_API_KEY` / `JQUANTS_*`
- `CHANNEL_*`
- 一切の secret / token

理由: daily refresh は **LINE 送信なしの maintenance job**、staging-only write
+ 4 sub-runner --dry-run 経路のみ。token 不要。

明示する env:
- `FIRE_ENV=staging` (= 三段ガード対応、CONCERN C 反映)
- `PYTHONPATH=/Users/bluefire/fire`

### 既存 plist との命名整合

| plist | 用途 | schedule |
|---|---|---|
| jp.fire.emergency-{HHMM}.plist (5 個) | F236 緊急アラート | 14:45/14:55/15:05/15:10/15:15 |
| jp.fire.weekly-snapshot.plist | F282 weekly snapshot | 土曜 02:00 |
| **jp.fire.daily-refresh.plist (本 doc)** | **DATA-R3 daily refresh** | **月-金 06:30** |
| jp.fire.morning-advisory.plist (= Wave 43+ 別 doc) | F062 morning advisory | 月-金 08:45 |

→ Label 衝突 0、namespace `jp.fire.*` 完全準拠。

---

## 2. log path 設計 + logrotate 統合 (= L2b 反映)

### log 配置

```
/Users/bluefire/fire/logs/
├── cron/
│   ├── weekly-snapshot.log / .err          ← F282 (= 別 file)
│   ├── daily-refresh.log / .err            ← 本 doc (新規)
│   └── morning-advisory.log / .err         ← Wave 43+
├── notifications/
│   └── launchd_emergency.log               ← F236
└── archive/
    └── YYYY-MM/
        ├── daily-refresh.log-YYYY-MM.gz
        └── daily-refresh.err-YYYY-MM.gz
```

### log 内容 (= aggregate runner 出力規約)

- 開始: `F286-DATA-R3 daily_refresh dry-run start (base_date=YYYY-MM-DD)`
- 4 sub-runner: 各 name + exit code
- 最終: `aggregate_dry_run_exit_code=<N>`
- completion-report path
- 異常時: `ERROR` / `REFUSED` を含む line
- F282 干渉防止: `F282 unchanged confirmed` line

### logrotate 設定 (= Wave 41+ 別配置)

`/etc/newsyslog.d/jp.fire.conf` に daily-refresh.{log,err} を追加 plan。
F282 weekly-snapshot と同等 30 日 retention。本 wave 配置 0。

---

## 3. 実行順序 / 依存グラフ (= L1b 反映)

### 月-金 時系列

```
06:30 jp.fire.daily-refresh (= 本 doc)
  ↓ aggregate runner 起動 (= subprocess 4 並列ではなく直列)
  ├─ F100 fetch_historical_market_data --dry-run
  ├─ F101 fetch_announcements --dry-run
  ├─ F111 run_research_watchlist_signal_persistence --dry-run
  └─ F119 run_f119_interpretation_evaluation --dry-run
  ↓ aggregate_dry_run_exit_code
  ↓ (試走中: 0 で完了)
  ↓ 別 wave で freshness gate (= F286-DATA-R2) 接続
08:45 jp.fire.morning-advisory (= Wave 43+ 別 doc)
  ↓ DATA-R3 success + freshness gate OK 必須
  ↓ F062 advisory 生成 → record-decisions → LINE
```

### 土曜時系列 (= F282 並走)

```
02:00 jp.fire.weekly-snapshot (= F282、別系統)
  ↓ production → staging/develop 完全 copy
  (土曜のみ、daily 系と時刻重複 0)
```

### 月曜運用順序

```
土曜 02:00 F282 weekly snapshot (= 金曜分まで反映)
   ↓ (2 営業日休止)
月曜 06:30 DATA-R3 daily refresh (= 土日含む 2 営業日分追加)
   ↓ 順序競合 0、DB write target 競合 0 (= F282=production read、DATA-R3=staging write)
```

### 緊急アラート干渉 0

| 時間 | label |
|---|---|
| 06:30 | jp.fire.daily-refresh (本 doc) |
| 08:45 | jp.fire.morning-advisory (Wave 43+) |
| 14:45/14:55/15:05/15:10/15:15 | jp.fire.emergency-{HHMM} (F236) |
| 土 02:00 | jp.fire.weekly-snapshot (F282) |

→ 全時刻重複 0。

---

## 4. no-write / no-send 試走計画 (= L2a 反映)

### 目的

- launchd 経路から aggregate runner が正常起動するか
- 4 sub-runner --dry-run 経路が走るか
- DB mtime before/after unchanged
- exit 0
- F282 試走 5/16-5/19 と並走で干渉 0

### 期間

1 週間 (= 月-金 5 営業日)、Wave 41 以降で実施。
F282 試走 5/16 02:00 と並走、daily 06:30 で完全分離。

### 日次確認項目

- `logs/cron/daily-refresh.log` mtime = 試走日
- log に `F286-DATA-R3 daily_refresh dry-run OK`
- `logs/cron/daily-refresh.err` 空
- `aggregate_dry_run_exit_code = 0`
- 想定外 traceback / WARN / sub-runner non-zero 0

### DB mtime before/after (= 全 unchanged 必須)

- `data/fire.db` (production) → unchanged
- `data/fire.develop.db` → unchanged
- `data/fire.staging.db` → unchanged (= dry-run 中 write 0)

### no-send / secret 確認

- log に LINE 送信痕跡 0
- `line-bot-sdk` import 0
- plist `EnvironmentVariables` に token / channel_token 不在
- log に token-like 値 0

### API call 0 確認

- J-Quants API 0 (= --dry-run)
- TDnet API 0 (= --dry-run)
- probe-help のみ (= 既存 dry-run 仕様)
- record-decisions 未接続
- F286-PNL-R2 ingest 別系統、daily-refresh から呼ばない

### GO/NO-GO 判定

- GO: 5 営業日 全 PASS + DB write 0 + LINE 0 + token 0 + API 0
- NO-GO: 任意失敗 → 修正 + 再試走 (= 別 wave HQ approve)

---

## 5. log / monitoring / abort 条件 (= L2b 反映)

### abort 条件

| 条件 | action |
|---|---|
| sub-runner exit non-zero | aggregate exit non-zero → launchd 次回まで再起動なし |
| DB mtime 変化 | 即 abort + 調査 (= dry-run write 0 違反) |
| LINE 送信痕跡 | 即 abort + token rotate |
| token 露出 (plist / log) | 即 abort + secret hygiene review |
| log file 書き込み不能 (disk full) | launchd 自動異常終了 |
| 同一 Label 二重 load | 配置 wave 側 abort |

### multi-run 防止

- `RunAtLoad=false`: load 時の即時起動なし
- `StartCalendarInterval` 5 個 (月-金) のみで 1 日 1 回起動
- retry loop / self-restart / 常駐 loop 設計しない
- `LimitLoadToSessionType=Aqua` (= L4 CONCERN E 反映): GUI session のみ

### daily monitoring

- 翌朝 (= 試走中 1 週間)
- log mtime / exit code / DB mtime 確認
- 5 営業日 PASS → trial GO 候補

### disk check (= 別 wave 配置)

- `df` / `du` daily check
- 空き 1 GB 以下 → 警告 (= alert hook 別 wave)

### alert hook (= 別 wave)

- exit non-zero → log line に `ERROR` / `REFUSED`
- LINE 通知 hook は本 wave 0、Wave 43+ で追加検討

---

## 6. F282 不干渉確認 (= L1b + L6 反映)

### 物理分離

| 項目 | F282 | DATA-R3 (本 doc) |
|---|---|---|
| Label | jp.fire.weekly-snapshot | jp.fire.daily-refresh |
| schedule | 土曜 02:00 | 月-金 06:30 |
| DB scope | production read → staging/develop copy | staging write only |
| log path | weekly-snapshot.{log,err} | daily-refresh.{log,err} |
| 時刻重複 | 0 | 0 |
| label 衝突 | 0 | 0 |
| DB write 競合 | 0 | 0 |

### 5/16 F282 本番試走への影響 (= 本 wave 進行中)

- 本 wave は **設計のみ**、実 plist 配置 / launchctl 0
- F282 plist mtime=1778593597 size=1772 完全不変 (= 2026-05-13 baseline 通り)
- F282 next run descriptor: Weekday=7 Hour=2 Minute=0 維持
- 本 wave 進行中 (= 10:42-11:11 JST) で F282 にいかなる touch も 0

---

## 7. Wave 41 以降 最短順 (= v0 Launch Plan Phase B 反映)

```
Wave 40-pre (= 本 doc) 完了
   ↓
5/16 02:00 F282 本番試走 (= 本流、変更なし)
   ↓
5/19 F282 GO/NO-GO 判定
   ↓ (GO 時)
Wave 41 候補: F286-DATA-R3 plist 実配置 (= HQ_APPROVE_LAUNCHD_DAILY)
   - ~/fire/docs/launchd/jp.fire.daily-refresh.plist 配置
   - ~/Library/LaunchAgents/jp.fire.daily-refresh.plist 配置
   - plutil -lint 検証
   - launchctl load 実行
   - launchctl list 登録確認
   - F282 不干渉再確認
   ↓
Wave 42 候補: DATA-R3 1 週間 no-write 試走 (= 月-金 5 営業日)
   - 日次 log / DB mtime / exit code 確認
   - F282 不干渉確認
   - 5/19-5/26 期間想定
   ↓
Wave 43 候補: F062 morning advisory launchd 設計 (= Phase C 起点)
Wave 44 候補: freshness gate 統合確認
Wave 45 候補: F062 launchd 実配置 + no-send mode (= HQ_APPROVE_LAUNCHD_MORNING_ADVISORY)
Wave 46 候補: 1 週間 no-send 試走 (= HQ_APPROVE_NO_SEND_TRIAL)
   ↓
Wave 52 候補: production LINE token 投入 (= HQ_APPROVE_LINE_TOKEN_PRODUCTION)
Wave 53 候補: D-Day 朝 advisory 実送信 1 回目 (= 2026-06-09 想定)
```

---

## 8. 安全要件 (= 本 doc 自体)

| 項目 | 結果 |
|---|---|
| 実 plist 配置 | 0 (= docs path にも未配置、設計のみ) |
| launchctl load / unload | 0 |
| cron / launchd / crontab 登録変更 | 0 |
| DB write | 0 |
| LINE 送信 | 0 |
| token / channel_token / secret 参照 | 0 |
| 実 API call (J-Quants / TDnet / LINE) | 0 |
| F282 試走干渉 | 0 (= plist mtime 不変、3 環境 DB unchanged) |
| F101 staging probe | 0 |
| 楽天 / 自動発注 / Computer Use | なし |
| workflow / --no-verify / TODO Excel | 0 |
| Codex 直接 commit | 0 |
| pytest 実行 | 0 (= 設計のみ、4090 PASS 静的維持) |

---

## 9. 関連リンク

- [[FIRE_production_v0_launch_plan_2026-05-13|v0 Launch Plan v1.0]]
- [[F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[F282_temporary_launchd_smoke_2026-05-13|F282 temporary smoke (Wave 39-temp)]]
- [[../02_todo/FIRE_CODEX_R1_WAVE40_PRE_plan|Wave 40-pre plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE40_PRE_results|Wave 40-pre results]]
- `/tmp/codex_wave40pre/results/l{1a,1b,2a,2b,3,4,6}.txt` (= 7 lane stdout)
- 既存 runner: `~/fire/scripts/jobs/run_f286_data_r3_daily_refresh.py`
