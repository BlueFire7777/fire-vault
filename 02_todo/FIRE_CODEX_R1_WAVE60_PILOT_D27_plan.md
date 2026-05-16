---
id: FIRE-CODEX-R1-WAVE60-PILOT-D27-plan
phase: 本番 v0 中核 / Wave 60-pilot-D27 / post-137A-demote 3 連続到達確認
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-19
pilot_day: D27
---

# Wave 60-pilot-D27 Plan — Post-137A-Demote 3 連続到達 / GO_CONDITIONAL 8 連続 v1.0

## §1 目的

D25 で 137A0 demote 本実行 → D26 で 2 連続 → D27 で 3 連続到達を確認、
sector 3 種多様化維持、HOLD #2 完全回避継続、GO_CONDITIONAL maintained 8 連続を
判定する。次の demote 対象 (= 7991 rank 1 連続性) も monitor。

## §2 安全境界

全 0 制約遵守

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D26 review check + D26 paper PnL --review-md 再 run |
| 2 | recently_seen 6 件維持確認 |
| 3 | D27 chain (= F111 + paper PnL preview、F062/AFTER-R1 既存 pattern 再利用) |
| 4 | 7991/9130/331A0 3 連続到達確認 |
| 5 | sector 3 種多様化維持確認 |
| 6 | HOLD #2 完全回避継続確認 |
| 7 | 7991 rank 1 連続性 monitor (= 次の demote 候補) |
| 8 | D27 hard check + W3 9 条件判定 |
| 9 | D27 trade plan + review template + paper PnL handoff |
| 10 | W5 集約引き継ぎ材料整理 |
| 11 | vault + HQ + 6 KPI |

## §4 完了条件

- D26 review 確認完了
- D26 paper PnL 再 run 完了
- D27 candidate generation 完了
- 7991/9130/331A0 3 連続確認
- sector 3 種多様化維持確認
- HOLD #2 回避継続確認
- 7991 rank 1 連続性 monitor 完了
- D27 hard check + 判定完了
- D27 trade plan + review template 作成完了
- 全 0 制約遵守
- F282 plist + 3 DB 不変
- HQ + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**
