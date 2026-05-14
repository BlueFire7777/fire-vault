---
id: FIRE-CODEX-R1-WAVE60-PILOT-D21-plan
phase: 本番 v0 中核 / Wave 60-pilot-D21 / post-recovery 初回継続判定
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-11
pilot_day: D21
---

# Wave 60-pilot-D21 Plan — Post-Recovery Manual Live Pilot v1.0

## §1 目的

D20 GO_CONDITIONAL 復帰後の初回継続判定。D20 review / paper PnL / D21 候補
継続性 / 137A0-7991-331A0 安定確認 → D21 GO/GO_CONDITIONAL/HOLD/NO-GO 判定。

## §2 安全境界

全 0 制約遵守 (= DB write 0 / production-develop 0 / staging read-only /
LINE 0 / token 0 / API 0 / launchctl 0 / plist 0 / cron 0 / 実発注 0 /
楽天 0 / iSPEED 0 / Computer Use 0)

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D20 review check + paper PnL --review-md 再 run |
| 2 | D21 chain (= recently_seen 5 件維持) |
| 3 | 137A0/7991/331A0 継続性 + sector 多様化維持確認 |
| 4 | D21 hard check + W3 9 条件判定 |
| 5 | 4 段階判定 |
| 6 | D21 trade plan + review template + paper PnL handoff |
| 7 | vault + HQ + 6 KPI |

## §4 完了条件

- D20 review check 完了
- D20 paper PnL 再 run 完了
- D21 candidate generation 完了
- 137A0/7991/331A0 継続性確認
- D21 hard check + 4 段階判定完了
- D21 trade plan + review template 作成完了
- paper PnL handoff 明記
- 全 0 制約遵守
- F282 plist + 3 DB 不変
- HQ + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**
