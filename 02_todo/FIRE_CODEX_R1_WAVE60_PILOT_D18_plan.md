---
id: FIRE-CODEX-R1-WAVE60-PILOT-D18-plan
phase: 本番 v0 中核 / Wave 60-pilot-D18 / HOLD 完全解除判定 (D14 review 反映後)
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-08
pilot_day: D18
---

# Wave 60-pilot-D18 Plan — Review Gap Resolved / HOLD Release Recheck v1.0

## §1 目的

D15-D17 で継続した HOLD を完全解除するため、D14 review 必須 5 項目の記入
状況を確認、paper PnL へ反映、D18 candidate 再生成し、新 top 候補継続確認、
GO_CONDITIONAL 復帰判定。

## §2 安全境界

- 全 0 制約遵守: DB write 0 / production-develop 接続 0 / staging read-only /
  LINE 0 / token 0 / API 0 / launchctl 0 / plist 0 / cron 0 / 実発注 0 /
  楽天 0 / iSPEED 0 / Computer Use 0 / workflow 0 / --no-verify 0 /
  git push 0 / sudo 0 / rm -rf 0 / TODO Excel 0

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D14 review 厳密 check |
| 2 | D14 paper PnL --review-md 再 run |
| 3 | recently_seen 維持 (= 8747,5729,3489,340A0) |
| 4 | D18 chain (= F111 + F062 + AFTER-R1 + paper PnL) |
| 5 | 340A0 demote 維持 + 新 top 継続確認 |
| 6 | D18 hard check + 4 段階判定 |
| 7 | D18 trade plan + review template + paper PnL handoff |
| 8 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- D14 review 確認完了
- D14 paper PnL --review-md 再 run 完了
- D18 candidate generation 完了
- 340A0 demote 維持 + 新 top 継続確認
- D18 hard check + 判定完了
- D18 trade plan + review template 作成完了
- paper PnL handoff 手順明記
- 全 0 制約遵守
- F282 plist + 3 DB mtime/size 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env /
API / LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED /
Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel
