---
id: FIRE-CODEX-R1-WAVE60-PILOT-D28-plan
phase: 本番 v0 中核 / Wave 60-pilot-D28 / post-137A-demote 4 連続到達 + 7991 D29 warning 準備
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-22
pilot_day: D28
---

# Wave 60-pilot-D28 Plan — Post-Diversification Continuity / 7991 Recurrence Monitor v1.0

## §1 目的

D25-D27 で 3 連続安定した 7991/9130/331A0 が D28 で 4 連続到達するか確認、
sector 3 種多様化維持、HOLD #2 回避継続、GO_CONDITIONAL maintained 9 連続を判定。
7991 rank 1 連続 = 4 到達確認 + D29 demote sim 準備を整える。

## §2 安全境界

全 0 制約遵守 (= demote sim 自体は D29 wave で実施、本 wave では予告のみ)

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D27 review check + D27 paper PnL --review-md 再 run |
| 2 | recently_seen 6 件維持確認 |
| 3 | D28 chain (= F111 + paper PnL preview) |
| 4 | 7991/9130/331A0 4 連続到達確認 |
| 5 | sector 3 種多様化維持確認 (= 4 連続) |
| 6 | HOLD #2 完全回避継続確認 |
| 7 | 7991 rank 1 連続 = 4 確認 + D29 warning 準備 |
| 8 | D28 hard check + W3 9 条件判定 |
| 9 | D28 trade plan + review template + paper PnL handoff |
| 10 | W5 集約引き継ぎ材料整理 |
| 11 | vault + HQ + 6 KPI |

## §4 完了条件

- D27 review 確認完了
- D27 paper PnL 再 run 完了
- D28 candidate generation 完了
- 7991/9130/331A0 4 連続確認
- sector 3 種多様化 4 連続維持確認
- HOLD #2 回避継続確認
- 7991 recurrence monitor 完了 (= rank 1 連続 = 4)
- D29 demote sim 準備記録
- D28 hard check + 判定完了
- D28 trade plan + review template 作成完了
- 全 0 制約遵守
- F282 plist + 3 DB 不変
- HQ + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**
