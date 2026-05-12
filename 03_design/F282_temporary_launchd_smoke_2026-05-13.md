---
id: F282-temporary-launchd-smoke
phase: F282 temporary smoke (= v0 前倒し準備)
priority: 高
status: 設計 v1.0 (= Wave 39-pre、impl は別 wave + 3 段 HQ approve)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - W33 (= F282 本番 plist 配置完了)
  - W34 (= F282 試走監視テンプレ)
  - HQ Wave 39-pre 起票承認 (= 2026-05-13)
---

# F282 temporary launchd smoke 設計 v1.0

5/19 F282 GO/NO-GO 判定を待たずに **launchd 経路リスクを前倒し検証** し、
Production v0 準備を加速する。**本番 plist 完全不干渉**、5/16 本番試走に
影響しない。

## 0. 目的と非目的

### 目的

- launchd → run_f282_weekly_snapshot.py 起動経路の前倒し検証
- venv / cwd / env / log path 動作確認
- snapshot 専用 path 出力確認
- 既存 DB 不変確認

### 非目的 (= temporary smoke で代替不可)

- 土曜 02:00 自然起動の検証 (= 5/16 本番試走で確認)
- 5/19 までの安定性確認 (= 5/16-5/19 試走で確認)
- 本番 GO 判定の代替 (= temporary smoke 成功でも 5/19 GO 判定残す)

## 1. A/B/C 案比較 (= W39p-2 L1a)

| 観点 | A 待機 | B 本番変更 | **C temporary** |
|---|---|---|---|
| 安全性 | 高 | 低 | 中 |
| 5/19 試走への影響 | 0 | 大 | **0** |
| 前倒し効果 | 0 | 高 | 中-高 |
| 復旧コスト | 0 | 高 | 中 |
| HQ 承認複雑度 | 低 | 高 | 中 |
| 適用 risk | 0 | 高 | 中 |

### 推奨: **C 第一候補** (= 5/16 試走影響 0 + 前倒し効果 + 復旧容易)

### A / B 不採用理由

- A: v0 準備が空く、DATA-R3 / F062 設計 task が遅延
- B: 本番 plist の StartCalendarInterval 書換 → 戻し忘れ risk → 5/16 試走
  影響大、launchd 内部状態に副作用

## 2. C 案 temporary smoke 詳細設計 (= W39p-3 L1b + W39p-6 L3)

### 完全分離原則

| 項目 | 本番 (= W33 配置) | temporary smoke |
|---|---|---|
| Label | jp.fire.weekly-snapshot | **jp.fire.weekly-snapshot-smoke** |
| plist file | ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist | ~/Library/LaunchAgents/**jp.fire.weekly-snapshot-smoke**.plist |
| StartCalendarInterval | 土曜 02:00 (= 毎週繰返し) | **単一日時 (= 平日近未来 1 回)** |
| log path | logs/cron/weekly-snapshot.{log,err} | logs/cron/**weekly-snapshot-smoke**.{log,err} |
| snapshot output | data/{staging,develop}.db (= 本番) または data/snapshot/ (= W30) | **data/snapshot-smoke/{staging,develop}.db** |
| 触れる本番 file | - | **0 (= 完全分離)** |

### plist XML draft (= 別 wave で作成)

配置元: `~/fire/docs/launchd/jp.fire.weekly-snapshot-smoke.plist`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>jp.fire.weekly-snapshot-smoke</string>

    <key>ProgramArguments</key>
    <array>
        <string>/Users/bluefire/fire/.venv/bin/python</string>
        <string>/Users/bluefire/fire/scripts/jobs/run_f282_weekly_snapshot.py</string>
        <string>--db-source</string>
        <string>production</string>
        <string>--db-targets</string>
        <string>staging,develop</string>
        <string>--target-dir</string>
        <string>/Users/bluefire/fire/data/snapshot-smoke/</string>
    </array>

    <!-- 単一実行: 平日近未来 1 時刻、本番 (= 土 02:00) と別時間帯 -->
    <key>StartCalendarInterval</key>
    <dict>
        <key>Year</key><integer>2026</integer>
        <key>Month</key><integer>5</integer>
        <key>Day</key><integer>13</integer>
        <key>Hour</key><integer>1</integer>
        <key>Minute</key><integer>30</integer>
    </dict>

    <key>WorkingDirectory</key>
    <string>/Users/bluefire/fire</string>

    <!-- 本番と同じ EnvironmentVariables、secret 0 -->
    <key>EnvironmentVariables</key>
    <dict>
        <key>FIRE_ENV</key>
        <string>snapshot</string>
        <key>PYTHONPATH</key>
        <string>/Users/bluefire/fire</string>
    </dict>

    <key>StandardOutPath</key>
    <string>/Users/bluefire/fire/logs/cron/weekly-snapshot-smoke.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/bluefire/fire/logs/cron/weekly-snapshot-smoke.err</string>

    <key>RunAtLoad</key>
    <false/>
</dict>
</plist>
```

### 実行 timing 選択

- **平日近未来 1 時刻** (= 5/13 火曜 01:30 等)
- 本番 (= 土曜 02:00) と完全別時間帯
- 1 回のみ起動、StartCalendarInterval 単一日時
- 起動後 launchctl unload で削除

### snapshot 出力専用 path

- `data/snapshot-smoke/fire.staging.db`
- `data/snapshot-smoke/fire.develop.db`
- 本番 `data/snapshot/` (= W30 retention) と命名空間分離
- W30 file には **絶対に touch しない**

## 3. verification checklist (= W39p-4 L2a)

### 配置前 check (= temporary smoke 起動前、別 wave 実行時)

- 本番 jp.fire.weekly-snapshot launchctl 状態 (= PID=- LastExit 0)
- 本番 plist mtime W33 不変
- 3 環境 DB mtime baseline
- data/snapshot/ W30 file mtime baseline (= 不変期待)
- data/snapshot-smoke/ 不在確認 (= 新規作成)
- logs/cron/weekly-snapshot-smoke.log 不在確認
- plutil -lint で temporary plist syntax OK

### 起動時 check (= launchctl load 直後)

- launchctl list jp.fire.weekly-snapshot-smoke 登録 (= LastExit 0、PID=-)
- 本番 jp.fire.weekly-snapshot 状態不変
- 本番 plist mtime 不変

### 実行直後 check (= 起動 5 分後)

- logs/cron/weekly-snapshot-smoke.log に "F282 snapshot OK" 等
- exit code 0 (= launchctl list LastExitStatus)
- data/snapshot-smoke/ に fire.staging.db / fire.develop.db 新規 (~353 MB)
- PRAGMA integrity_check ok
- **本番 plist + 本番 launchctl + 3 環境 DB + W30 snapshot 完全不変** ★

### 単一実行確認

- launchctl print "last fork count" = 1 (= 1 回のみ)
- 連続実行 / loop なし

## 4. abort / cleanup plan (= W39p-5 L2b)

### abort trigger (= 即時 unload + HQ 報告)

| trigger | 検出方法 | 対応 |
|---|---|---|
| 本番 plist mtime 変化 | stat 監視 | unload temp + 本番状態確認 |
| 本番 label 干渉 | launchctl list 異常 | unload temp |
| 3 環境 DB mtime 変化 | stat 監視 | unload temp + incident |
| data/snapshot/ W30 mtime 変化 | stat 監視 | unload temp + incident |
| snapshot 専用 path 外出力 | find 比較 | unload temp + incident |
| exit non-zero | launchctl list | unload temp + log 確認 |
| launchd 複数実行 (= fork ≥ 2) | launchctl print | unload temp 即時 |
| token / LINE / API 痕跡 | log grep | unload temp + token rotate |
| HQ abort 指示 | HQ メッセージ | unload temp 即時 |

### 通常 cleanup 手順

```bash
# Step 1: unload
launchctl unload ~/Library/LaunchAgents/jp.fire.weekly-snapshot-smoke.plist

# Step 2: plist 削除
rm ~/Library/LaunchAgents/jp.fire.weekly-snapshot-smoke.plist

# Step 3: log 確認 + 保管 (= 1 週間 retention、5/20 頃に rm)

# Step 4: snapshot output 削除 (= 任意、1 週間 retention 後)
# rm data/snapshot-smoke/*.db
# rmdir data/snapshot-smoke

# Step 5: 本番 plist 不変確認
launchctl list | grep jp.fire.weekly-snapshot
# 期待: jp.fire.weekly-snapshot PID=- LastExit 0 維持
```

### 本番 plist 隔離原則

- 本番 plist は **絶対に touch しない**
- 本番 launchctl 状態を read のみ確認
- 本番 plist 配置 path への mv / cp / rm 全禁止

## 5. HQ 必要承認 marker (= 3 段)

| marker | 用途 |
|---|---|
| HQ_APPROVE_F282_TEMP_PLACE=1 | temporary plist 配置 (= ~/Library/LaunchAgents/ への cp) |
| HQ_APPROVE_F282_TEMP_LOAD=1 | launchctl load |
| HQ_APPROVE_F282_TEMP_UNLOAD=1 | launchctl unload + cleanup |

各 marker は **独立 HQ 明示承認** 必要。

## 6. temporary smoke で確認できる / できない

### 確認できる

- launchd → run_f282_weekly_snapshot.py 起動経路 OK
- venv (= .venv/bin/python) path OK
- cwd (= /Users/bluefire/fire) OK
- env (= FIRE_ENV=snapshot) 渡し OK
- StandardOut/Err log 出力 OK
- snapshot 専用 path 出力 OK
- production / 既存 staging / 既存 develop DB mtime 不変 OK
- exit 0 OK
- launchd 単一実行 OK

### 代替できない (= 5/16 本番試走で必須)

- 土曜 02:00 自然起動 (= timezone / cron 固有)
- 5/19 までの安定性 (= 単一実行のみ)
- 1 週間連続不変性

## 7. Production v0 への前倒し効果

### temporary smoke 成功後 5/19 前に進めて良い

- DATA-R3 sub-D2.3.x runner production 切替設計 (= Wave 40 候補)
- F062 morning advisory launchd 設計準備
- no-send 試走計画詳細化
- log rotation 設定本番配置 (= 別 wave、別 HQ approve)

### temporary smoke 失敗時 / まだ 5/19 まで待つもの

- F282 本番 plist 配置 / launchctl load → 既に W33 完了、再変更なし
- 土曜自動実行確認 → 5/16 本番試走
- 1 週間連続安定性 → 5/19 GO 判定

## 8. リスクと対策

| risk | 影響 | 対策 |
|---|---|---|
| temporary plist label 衝突 | launchctl 二重登録 | label 完全別 (= -smoke suffix) |
| 起動時刻 本番被り | 本番 advisory 連動異常 | 平日近時刻で設定、土曜 02:00 回避 |
| unload 忘れ | temporary plist が永続 | cleanup 手順明示、HQ_APPROVE_F282_TEMP_UNLOAD |
| snapshot 専用 path 漏れ | data/ 直下に出力 | --target-dir 厳格指定、L4 audit 確認 |
| 本番 plist 誤 touch | 本番 5/16 試走影響 | 本番 plist は read のみ、cp/mv 全禁止 |

## 9. 想定 next wave (= 別 wave、HQ 個別 approve)

- Wave 39-impl 候補: temporary plist 配置 + launchctl load (= 2 marker 要)
- Wave 39-cleanup 候補: unload + cleanup (= 1 marker 要)
- ただし HQ「迷うなら v0 優先」方針照らし、temporary smoke 実行は v0
  経路を実質遅らせない範囲のみ実施

## 10. 安全 (= 本 doc 自体)

| 項目 | 結果 |
|---|---|
| 実 plist 配置 | 0 |
| launchctl load / unload | 0 |
| 実 DB write | 0 |
| 実 LINE 送信 | 0 |
| 実 VACUUM INTO | 0 |
| token / channel_token / secret 参照 | 0 |
| 本番 plist 変更 | 0 |
| cron / launchd 登録変更 | 0 |
| F282 試走干渉 | 0 |

## 11. L4 audit verdict (= W39p-7 反映)

8 観点 PASS: A 本番不干渉 / B label 衝突防止 / C snapshot path 分離 /
D 実行 timing 安全 / E secret 除外 / F cleanup 完全性 / G HQ marker 3 段 /
H 代替不能項目明示。

CRITICAL 0 / HIGH 0。

## 12. 関連リンク

- [[F282_weekly_snapshot_launchd_2026-05-12|F282 本番 launchd 設計]]
- [[F282_weekly_snapshot_trial_monitoring_2026-05-12|F282 試走監視 (W34)]]
- [[FIRE_production_v0_launch_plan_2026-05-13|Production v0 Launch Plan (W38)]]
- [[../02_todo/FIRE_CODEX_R1_WAVE39_PRE_plan|Wave 39-pre plan]]
