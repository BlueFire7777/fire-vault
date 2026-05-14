---
id: FIRE-CODEX-R1-WAVE56-temp-plan
phase: 本番 v0 Launch / Wave 56-temp / DATA-R3 temporary launchd no-write smoke
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 56-temp Plan — DATA-R3 Temporary Launchd No-Write Smoke + Engineering Gate

## §1 目的
W55-temp Engineering GO を受けて、5/16 本番 F282 を待たずに DATA-R3 daily refresh を
別 label・別 plist・別 log・別 output で no-write / dry-run smoke する。
本番 DATA-R3 plist は配置/load しない。

## §2 HQ 承認範囲 (= 本 wave のみ)
- HQ_APPROVE_DATA_R3_TEMP_LAUNCHD_SMOKE=1
- 一時 label `jp.fire.daily-refresh-temp-smoke` の plist 作成 / launchctl
  bootstrap / print / bootout / 削除 / 一時 output / log 作成

絶対禁止: 本番 DATA-R3 label 作成、本番 plist 配置/load、F282/F062 label 変更、
DB write、LINE 送信、token 値参照、API call、cron / workflow 変更。

## §3 一時 smoke 仕様
- label: `jp.fire.daily-refresh-temp-smoke`
- plist: `~/Library/LaunchAgents/jp.fire.daily-refresh-temp-smoke.plist`
- log: `~/fire/logs/cron/daily-refresh-temp-smoke.{log,err}`
- freshness: `/tmp/fire_daily_refresh_temp_smoke_freshness.json` (= 一時 path)
- RunAtLoad: true、StartCalendarInterval: なし、実行 1 回
- args: `--dry-run --execute-dry-run-subprocesses --db-path data/fire.staging.db
   --db-label staging --freshness-report-json <temp> --stale-threshold-hours 24`

## §4 完了条件
- 一時 label 1 回起動 / log / freshness report 生成
- cleanup (= bootout + plist 削除) 完了
- 本番 F282 / DATA-R3 / F062 plist 不変
- 3 DB / W30 snapshot 不変
- LINE / token / API / cron / workflow 全 0
- Engineering GO/HOLD/NO-GO 判定

## §5 停止条件 (= W55-temp と同じ)
本番 plist 変更 / DB write / DB sqlite 接続 / LINE / token / API /
launchctl 本番 / cron / VACUUM / workflow / --no-verify / sudo / 楽天 が必要 → 即停止 + HQ
