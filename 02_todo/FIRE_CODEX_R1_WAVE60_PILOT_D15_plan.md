---
id: FIRE-CODEX-R1-WAVE60-PILOT-D15-plan
phase: 本番 v0 中核 / Wave 60-pilot-D15 / D14 review 反映 + HOLD 判定 day
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-03
pilot_day: D15
design_doc: ~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
---

# Wave 60-pilot-D15 Plan — D15 Review-Linked Manual Live Pilot Trade Plan v1.0

## §1 目的

D14 review と paper PnL preview を D15 trade plan に接続。D14 で enter/watch/skip
のどれだったか、actual price / liquidity / event 確認がどうだったかを確認し、
D15 の GO/GO_CONDITIONAL/HOLD/NO-GO 判定に反映する。

## §2 安全境界

- DB write: **禁止** (本 wave は read-only 確認のみ)
- production / develop DB 接続: **禁止**
- staging DB: **read-only のみ**
- LINE / token / API / launchctl / plist / cron / VACUUM / workflow /
  --no-verify / git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED /
  Computer Use / TODO Excel: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist + 3 DB md5) + D14 review 必須 5 項目確認 |
| 2 | D14 paper PnL --review-md 付き再 run |
| 3 | D15 F111 + F062 + AFTER-R1 + paper PnL chain |
| 4 | D15 hard check (= 12 項目) + 設計 doc §3 4 段階判定 |
| 5 | D15 trade plan + review template |
| 6 | paper PnL handoff 手順 |
| 7 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- D14 review 確認完了
- D14 paper PnL review 付き再 run 完了 (= review_actuals 反映確認)
- D15 candidate generation 完了
- D15 hard check 完了
- D15 4 段階判定完了 (= GO/GO_CONDITIONAL/HOLD/NO-GO)
- D15 trade plan + review template 作成完了
- D15 paper PnL handoff 手順明記
- DB write 0 / LINE 0 / token 0 / API 0
- launchctl 0 / plist 0 / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist + 3 DB mtime/size 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env /
API / LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED /
Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel
