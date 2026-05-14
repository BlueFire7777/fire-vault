---
id: FIRE-CODEX-R1-WAVE60-PILOT-D16-plan
phase: 本番 v0 中核 / Wave 60-pilot-D16 / HOLD 解除判定 day
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-04
pilot_day: D16
---

# Wave 60-pilot-D16 Plan — Review Gap Closure + Recently Seen Expansion Trade Plan v1.0

## §1 目的

D15 で発動した HOLD を解除するため、D14 review の必須 5 項目を確認し、
paper PnL に review を反映し、recently_seen_codes へ 340A0 を追加して、
D16 trade plan で GO_CONDITIONAL 以上に戻せるか判定する。

## §2 安全境界

- DB write 0 / production-develop 接続 0 / staging read-only / LINE 0 /
  token 0 / API 0 / launchctl 0 / plist 0 / cron 0 / 実発注 0 / 楽天 0 /
  iSPEED 0 / Computer Use 0 / workflow 0 / --no-verify 0 / git push 0 /
  sudo 0 / rm -rf 0 / TODO Excel 0

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D14 review 必須 5 項目確認 (= resolved/partial/unresolved 判定) |
| 2 | D14 paper PnL --review-md 再 run (= review_actuals 反映) |
| 3 | recently_seen_codes 拡張 (= 8747,5729,3489,**340A0**) |
| 4 | D16 chain (= F111 + F062 + AFTER-R1 + paper PnL) |
| 5 | 340A0 demote 効果確認 (= AFTER-R1 top_candidates 切替) |
| 6 | D16 hard check + 設計 doc §3 4 段階判定 |
| 7 | D16 trade plan + review template + paper PnL handoff |
| 8 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- D14 review 確認完了 (= 5 項目厳密 check)
- D14 paper PnL --review-md 再 run 完了
- recently_seen_codes 拡張完了
- D16 candidate generation 完了
- 340A0 demote 効果確認
- D16 hard check 完了
- D16 Pilot 判定完了
- D16 trade plan + review template 作成完了
- D16 paper PnL handoff 手順明記
- 全 0 制約遵守、F282 plist + 3 DB md5 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env /
API / LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED /
Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel
