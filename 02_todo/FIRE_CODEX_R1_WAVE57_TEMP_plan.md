---
id: FIRE-CODEX-R1-WAVE57-temp-plan
phase: 本番 v0 Launch / Wave 57-temp / F062 temporary launchd no-send smoke
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 57-temp Plan — F062 Temporary Launchd No-Send Smoke + Engineering GO Gate

## §1 目的
5/16 本番 F282 を待たずに F062 Morning Advisory を別 label・別 plist・別 log・
別 output で no-send launchd smoke。LINE 送信 / token 参照 / DB write 不実施で
preview 生成まで進むか検証。

## §2 HQ 承認範囲 (= 本 wave のみ)
- HQ_APPROVE_F062_TEMP_LAUNCHD_NO_SEND_SMOKE=1
- 一時 label / plist / launchctl bootstrap・print・bootout / 削除 / 一時 output / log 作成

絶対禁止: 本番 F062 label 配置/load、LINE 送信、token 値参照、API call、DB write、
本番 DATA-R3 / F282 操作、cron / workflow 変更。

## §3 一時 smoke 仕様
- label: `jp.fire.morning-advisory-temp-smoke`
- plist: `~/Library/LaunchAgents/jp.fire.morning-advisory-temp-smoke.plist`
- log: `~/fire/logs/cron/morning-advisory-temp-smoke.{log,err}`
- preview output: `/tmp/fire_morning_advisory_temp_smoke/`
- freshness: W56.5-temp 確認済 `/tmp/fire_w565_recheck_freshness.json` (= verdict=OK)
- input: synthetic F111-R4 preview JSON (= 3 rows、全 manual_review_required=True)
- RunAtLoad: true、StartCalendarInterval: なし、実行 1 回
- args: `--dry-run --require-freshness-ok --freshness-report-json <W56.5 OK>
   --preview-json <synthetic> --output-text /tmp/.../preview.txt
   --output-json --output-summary-json --completion-report --max-candidates 3`
- `--hq-approved-send` 絶対不使用 (= preview runner には存在しない)

## §4 完了条件
- 一時 label 1 回起動 / exit 0
- temp log / preview artifact 生成 (= 4 file: preview.txt / payload.json / summary.json / completion_report.txt)
- freshness gate "freshness OK" 通過
- safety: line_send_count=0 / token_read_count=0 / safety_footer=true / auto_order_allowed_true_count=0
- cleanup (= bootout + plist 削除) 完了
- 本番 F282 / DATA-R3 / F062 plist 不変、3 DB / W30 snapshot 不変
- LINE / token / API / cron / workflow 全 0
- Engineering GO/HOLD/NO-GO 判定

## §5 停止条件
本番 plist 変更 / DB write / DB sqlite 接続 / LINE / token / API / launchctl 本番 /
cron / VACUUM / workflow / --no-verify / 楽天 が必要 → 即停止 + HQ
