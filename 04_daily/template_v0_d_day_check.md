---
id: template-v0-d-day-check
type: template
status: blank (= Fujiwara 記入用、2026-06-08 月 + 2026-06-09 火 で 2 回使用)
chapter: production-v0 / D-Day GO/NO-GO checklist
created_by: Wave 44.5-pre (= cutover runbook v1.0 final と integrated)
related:
  - 03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md
  - 03_design/Production_v0_readiness_check_cli_v1_1_2026-05-14.md
---

# Production v0 D-Day GO/NO-GO Checklist Template

> Wave 44.5-pre で作成。2026-06-08 (月) final strict check 日と
> 2026-06-09 (火) D-Day morning の 2 回、Fujiwara が記入。
>
> 記入後の保存 path:
> - `~/fire-vault/04_daily/2026-06-08_v0_final_strict_check.md`
> - `~/fire-vault/04_daily/2026-06-09_v0_d_day_morning_check.md`

## §1 ヘッダー

- 実施日: ______-______-______
- 実施区分: [ ] 6/8 final strict check  [ ] 6/9 D-Day morning check
- 報告者: BlueFire7777 (Fujiwara)
- 記入時刻: ______:______:______ JST

## §2 共通: HQ marker 取得状態 (= 7 段)

| # | marker | 取得済 | 備考 |
|---|---|---|---|
| 1 | F282 GO 判定 | [ ] ✓ [ ] ✗ | 5/19 W40.8 / W43-pre CLI で記録済 |
| 2 | `HQ_APPROVE_LAUNCHD_DAILY` | [ ] ✓ [ ] ✗ | Wave 41 開始時 |
| 3 | DATA-R3 no-write 試走 GO | [ ] ✓ [ ] ✗ | Wave 41 完了時 |
| 4 | `HQ_APPROVE_LAUNCHD_MORNING_ADVISORY` | [ ] ✓ [ ] ✗ | Wave 45 開始時 |
| 5 | F062 no-send trial GO | [ ] ✓ [ ] ✗ | Wave 45 完了時 |
| 6 | `HQ_APPROVE_LINE_TOKEN_PRODUCTION` | [ ] ✓ [ ] ✗ | Wave 52 開始時 |
| 7 | `HQ_APPROVE_PRODUCTION_V0_LAUNCH` | [ ] ✓ [ ] ✗ | **D-Day morning check で必須** |

## §3 6/8 (月) final strict check (= 該当時のみ記入)

### §3.1 readiness CLI v1.2 strict 結果

```
.venv/bin/python -m scripts.jobs.run_production_v0_readiness_check \
  --phase pre-v0-launch --strict --json > /tmp/v0_readiness_2026-06-08.json
```

- cli_version: ______
- exit code: ______ (= 0 期待)
- summary: PASS=____ WARN=____ FAIL=____ SKIP=____
- 全 25 check の verdict 一覧 (= JSON 添付):
  - [ ] F282 launchd/plist 5 件全 PASS
  - [ ] DB safety 3 件全 PASS
  - [ ] Plist absent/present 2 件全 PASS
  - [ ] Token safety 2 件全 PASS
  - [ ] No-send mode 1 件 PASS
  - [ ] HQ marker 4 件全 PASS (= marker 1-6 取得済)
  - [ ] Date 2 件全 PASS
  - [ ] DATA-R3 freshness 2 件全 PASS
  - [ ] F062 capability 2 件全 PASS
  - [ ] F282 post-run 1 件 PASS
  - [ ] D-Day weekday HQ 1 件 PASS (= 6/9 火 確定後は PASS 想定)

### §3.2 F282 試走 GO 記録 (= 5/16 から引継)

- 5/16 03:00 W40.8 CLI verdict: [ ] GO [ ] NO-GO
- 5/16 03:00 W43-pre CLI verdict (= post-f282): [ ] GO [ ] NO-GO
- `04_daily/2026-05-16_f282_post_run.md` commit 済: [ ] ✓ [ ] ✗

### §3.3 DATA-R3 launchd 稼働状況

- `launchctl list jp.fire.daily-refresh` 存在: [ ] ✓ [ ] ✗
- 月-金 06:30 起動 (= 過去 5 営業日): runs +______
- last exit code 0: [ ] ✓ [ ] ✗
- freshness report (= `/tmp/f286_data_r3_freshness.json`) verdict == OK 連続: ______ 日

### §3.4 F062 no-send trial 結果 (= W45 から引継)

- 5 営業日全 PASS: [ ] ✓ [ ] ✗
- LINE 送信痕跡 0 確認: [ ] ✓ [ ] ✗
- token 参照 0 確認: [ ] ✓ [ ] ✗
- DB write 0 確認 (= advisory_decisions row 不変): [ ] ✓ [ ] ✗
- send_guard refuse 100% 確認: [ ] ✓ [ ] ✗

### §3.5 token 投入確認 (= Wave 52 から引継、§5.4 値非表示方式)

- `ls -l ~/.fire_secrets/line.production.env` 結果:
  - 存在: [ ] ✓ [ ] ✗
  - 権限: ______ (= -rw------- 期待)
- `wc -c ~/.fire_secrets/line.production.env`: ______ bytes
- `awk -F= '{print $1"=***MASKED***"}'` 結果: 8 行 LINE_* keys
  - [ ] 8 行揃っている
  - [ ] LINE_CHANNEL_TOKEN / LINE_USER_ID / LINE_ROOM_* 6 個揃っている
- plist 内 token literal 検出 0: [ ] ✓ [ ] ✗

### §3.6 wrapper script 状態 (= Wave 53 で配置済前提)

- `~/fire/scripts/wrappers/run_f062_production_send.sh` 存在: [ ] ✓ [ ] ✗
- 権限 chmod 700: [ ] ✓ [ ] ✗
- `set -euo pipefail` 含む: [ ] ✓ [ ] ✗
- `--freshness-report-json` + `--require-freshness-ok` 含む: [ ] ✓ [ ] ✗
- `--hq-approved-send` 含む: [ ] ✓ [ ] ✗
- token 値が source 経由のみ (= literal 0): [ ] ✓ [ ] ✗

### §3.7 backup 状況

- `~/fire/data/backup/fire.db.pre_v0_2026-06-08.db` 存在: [ ] ✓ [ ] ✗
- backup size: ______ bytes (= 元 fire.db と一致期待)
- checksum 一致: [ ] ✓ [ ] ✗

### §3.8 安全 incident 0 確認

- 5/13 〜 6/8 期間内の `07_incidents/` 新規 incident: ______ 件 (= 0 期待)
- safety 違反 (= LINE 露出 / token 露出 / DB 意図せぬ書込) 0 件: [ ] ✓ [ ] ✗

### §3.9 rollback owner / 手順確認

- rollback owner: ______ (= Fujiwara、24h 連絡可能)
- §11 緊急 unload 手順 6 step 暗記 + 紙印刷: [ ] ✓ [ ] ✗
- §11.3 token rotate 手順 暗記: [ ] ✓ [ ] ✗
- §9.2 anomaly judgment matrix 18 種 確認: [ ] ✓ [ ] ✗

### §3.10 6/8 final strict 判定

- [ ] **GO** (= 6/9 火 D-Day 朝に進める)
- [ ] **NO-GO** (= 即時 D-Day 延期、最低 1 週間 = 6/16 火)
- [ ] **HOLD** (= 残課題があり、6/8 夜の追加確認後判定)

## §4 6/9 (火) D-Day morning check (= 該当時のみ記入)

### §4.1 marker 7 取得状態

- `HQ_APPROVE_PRODUCTION_V0_LAUNCH` (= marker 7) 取得済: [ ] ✓ [ ] ✗
- 取得日時: ______-______-______ ______:______ (= 6/8 月夕方想定)

### §4.2 D-Day 当日 send marker

- `HQ_APPROVE_SEND_MARKER` 当日生成 + export 完了: [ ] ✓ [ ] ✗
- 生成時刻: 6/9 ______:______ (= 07:30 想定)
- marker 文字数: ______ (= 32 文字以上期待)

### §4.3 06:30 DATA-R3 起動状況

- DATA-R3 daily-refresh 自動起動: [ ] ✓ [ ] ✗ (= 06:30 launchd)
- 完了時刻: 6/9 06:______:______
- exit code: ______ (= 0 期待)
- freshness report 生成: [ ] ✓ [ ] ✗
- verdict == OK: [ ] ✓ [ ] ✗

### §4.4 wrapper / plist 切替確認

- 08:00 launchctl unload morning-advisory: [ ] ✓ [ ] ✗
- plist ProgramArguments wrapper に切替: [ ] ✓ [ ] ✗
- plutil -lint OK: [ ] ✓ [ ] ✗
- 08:15 launchctl load: [ ] ✓ [ ] ✗
- 08:20 launchctl list 登録確認 + PID=-: [ ] ✓ [ ] ✗

### §4.5 F282 / DB baseline 比較 (= 08:30 時点)

- F282 plist mtime: ______ (= 1778593597 期待 / baseline) ※ 5/16 試走後は値変動可
- F282 plist size: ______ (= 1772 期待 / baseline) ※ 5/16 試走後は値変動可
- production fire.db mtime: ______
- 直近 1 週間で意図せぬ mtime 変動: [ ] 0 件 [ ] ____ 件
- develop / staging DB mtime baseline 比較: [ ] ✓ [ ] ✗

### §4.6 readiness CLI v1.2 D-Day 朝 (= 08:30 実施)

```
.venv/bin/python -m scripts.jobs.run_production_v0_readiness_check \
  --phase pre-v0-launch --strict --json > /tmp/v0_readiness_2026-06-09.json
```

- exit code: ______ (= 0 期待)
- summary: PASS=____ WARN=____ FAIL=____ SKIP=____

### §4.7 08:45 自動起動 + 初回 production send

- 08:45 起動: [ ] ✓ [ ] ✗
- freshness gate OK: [ ] ✓ [ ] ✗
- send_guard OK: [ ] ✓ [ ] ✗
- LINE 着信 (= Fujiwara 目視): [ ] ✓ 09:______:______ [ ] ✗
- chunk 数: ______ (= 1 期待)
- partial_delivery: [ ] False (= 期待) [ ] True (= 違反)

### §4.8 D-Day 09:00-09:30 確認

- record-decisions row +1: [ ] ✓ [ ] ✗
- advisory_id = `morning_advisory_2026-06-09_*`: [ ] ✓ [ ] ✗
- UNIQUE conflict 0: [ ] ✓ [ ] ✗
- log path `~/fire/logs/cron/morning-advisory.log` 当日 mtime: ______:______:______
- log 内 ERROR / Exception / Traceback: 0 件 / ______ 件
- exit code 0: [ ] ✓ [ ] ✗

### §4.9 D-Day 判定

- [ ] **D-Day 成功** (= log.md に milestone 記録、dual-run day 1 完了 = 6/9 火、翌 6/10〜6/15 で day 2-5 継続)
- [ ] **D-Day 異常** (= §9.2 matrix 該当異常を記録、§11 rollback 発動)
- [ ] **D-Day 部分成功** (= HOLD、HQ 相談、6/10 朝判定)

### §4.10 §9.2 anomaly 該当 (= 異常時のみ記入)

異常 # (= §9.2 matrix): ______

判断: [ ] SEND [ ] NO-SEND [ ] HOLD [ ] ROLLBACK

詳細:
______________________________________________
______________________________________________

action 履歴:
- ______:______ ____________________________________
- ______:______ ____________________________________

## §5 次 action (= 該当時のみ記入)

- 6/8 GO → 6/9 D-Day 朝 08:00 unload 待機
- 6/8 NO-GO → 即時 D-Day 延期、HQ_APPROVE_PRODUCTION_V0_LAUNCH 無効化、別 wave で残課題対応
- 6/9 成功 → 6/10 dual-run monitoring 開始
- 6/9 異常 → §11 rollback 即発動、`07_incidents/d_day_rollback_2026-06-09.md` 起票

## §6 添付

- `/tmp/v0_readiness_<date>.json` (= readiness CLI v1.2 stdout)
- `/tmp/f286_data_r3_freshness.json` (= DATA-R3 freshness report)
- `/tmp/f062_morning_preview_<date>.txt` (= preview ファイル)
- `~/fire/logs/cron/morning-advisory.log` (= 当日 log 抜粋)
- `~/fire/logs/cron/daily-refresh.log` (= 当日 log 抜粋)

---

> 記入完了後、本 file を vault commit。
> 異常検知時は `~/fire-vault/07_incidents/d_day_rollback_<date>.md` も並行起票。
> 関連 doc:
> - [[../03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14|cutover runbook v1.0 final]]
> - [[../03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14|readiness CLI v1.2 (= W51 CRITICAL fix 後)]]
