---
id: FIRE-CODEX-R1-WAVE55-temp-plan
phase: 本番 v0 Launch / Wave 55-temp / F282 temporary launchd smoke
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 55-temp Plan — F282 Temporary Launchd Smoke + Engineering GO Gate v1.0

## §1 目的
5/16 02:00 本番 F282 試走を待たずに、別 label・別 plist・別 log・別 output で
launchd 経由起動を即時検証する。成功なら Engineering GO として
Wave 56-temp DATA-R3 一時 smoke に進行可とする。
Official F282 GO は 5/16 本番 label 実行後の別判定。

## §2 HQ 承認範囲 (= 本 wave のみ)
- HQ_APPROVE_F282_TEMP_LAUNCHD_SMOKE=1
- 一時 label `jp.fire.weekly-snapshot-early-smoke` の plist 作成 / launchctl
  bootstrap / launchctl print / launchctl bootout / plist 削除 / 一時 output / log 作成

絶対禁止: 本番 label `jp.fire.weekly-snapshot` の plist / unload / load / schedule 変更、
本番 F282 next run の変更、production / develop / staging DB write、LINE 送信、
token 値参照、API call、cron 変更、workflow 変更、--no-verify、TODO Excel、
楽天 / 自動発注 / Computer Use。

## §3 一時 smoke 仕様
- label: `jp.fire.weekly-snapshot-early-smoke`
- plist: `~/Library/LaunchAgents/jp.fire.weekly-snapshot-early-smoke.plist`
- output: `~/fire/data/snapshot-smoke-early/`
- log: `~/fire/logs/cron/weekly-snapshot-early-smoke.{log,err}`
- RunAtLoad: true、StartCalendarInterval: なし、実行回数: 1 回のみ
- cleanup: bootout + plist 削除

## §4 実施項目
1. baseline (= 本番 F282 plist / 3 DB / W30 snapshot / pytest 4489)
2. F282 runner CLI 確認 (= `--target-dir` で出力先上書き可)
3. 一時 plist 作成 + plutil lint OK
4. launchctl bootstrap → RunAtLoad で 1 回起動 → 完了待ち
5. log / output / exit code 確認
6. launchctl bootout + plist 削除
7. 本番 F282 plist mtime/size、3 DB、W30 snapshot 不変確認
8. post-run chain smoke (= readiness CLI v1.2 / Ops Summary v1.0.1)
9. Engineering GO / HOLD / NO-GO 判定
10. vault に W55-temp plan / results

## §5 Engineering GO 判定基準
全 7 条件 PASS で GO:
- 一時 label 1 回起動 (= runs=1)
- log / output 生成
- cleanup 完了 (= bootout + plist 削除)
- 本番 F282 plist mtime/size 不変
- 3 DB 不変
- W30 snapshot 不変
- LINE / token / API / cron / workflow / TODO Excel 0

## §6 Official F282 GO との分離
- Engineering GO = 開発継続の仮 GO、5/16 本番 trial を待たない
- Official F282 GO = 5/16 02:00 本番 label `jp.fire.weekly-snapshot` 実行後の
  5/19 W41 着手前判定
- 一時 smoke 成功 ≠ Official F282 GO (= 別判定)
