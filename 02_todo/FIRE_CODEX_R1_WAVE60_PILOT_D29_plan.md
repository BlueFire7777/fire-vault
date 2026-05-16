---
id: FIRE-CODEX-R1-WAVE60-PILOT-D29-plan
phase: 本番 v0 中核 / Wave 60-pilot-D29 / 7991 rank 1 5 連続 warning + demote sim 完了
priority: 最高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-23
pilot_day: D29
---

# Wave 60-pilot-D29 Plan — 7991 Recurrence Warning / Demote Simulation v1.0

## §1 目的

D25-D28 で 4 連続 rank 1 となった 7991 について、D29 で 5 連続 warning 段階に
到達するか確認、到達した場合は demote sim を実施 + D30 本実行判断を確定する。

## §2 安全境界

全 0 制約遵守 (= demote sim は read-only chain、本実行は D30 wave で別途 HQ approve)

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D28 review check + paper PnL 再 run |
| 2 | D29 chain (= 現行 recently_seen 6 件) |
| 3 | 7991 rank 1 5 連続到達確認 |
| 4 | 7991 demote sim 実施 (= 参考、6 件 + 7991) |
| 5 | sim 結果 (= 新 top + sector 構成) 確認 |
| 6 | D30 demote 判断 (= 即実施 / D30 推奨 / D31 delay) |
| 7 | D29 hard check + 4 段階判定 |
| 8 | D29 trade plan + review template + paper PnL handoff |
| 9 | W5 集約引き継ぎ材料 |
| 10 | vault + HQ + 6 KPI |

## §4 完了条件

- D28 review 確認完了
- D29 candidate generation 完了
- 7991 5 連続到達確認
- 7991 demote sim 完了
- D30 demote 判断完了 (= 案 B = D30 本実行強推奨)
- D29 hard check + 判定完了
- D29 trade plan + review template 作成完了
- 全 0 制約遵守
- F282 plist + 3 DB 不変
- HQ + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**
