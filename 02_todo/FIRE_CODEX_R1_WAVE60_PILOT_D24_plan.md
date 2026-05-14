---
id: FIRE-CODEX-R1-WAVE60-PILOT-D24-plan
phase: 本番 v0 中核 / Wave 60-pilot-D24 / 137A0 5 連続 warning + demote 検討
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-16
pilot_day: D24
---

# Wave 60-pilot-D24 Plan — 137A Recurrence Warning / Demote Decision v1.0

## §1 目的

D20-D23 で 4 連続安定した 137A0/7991/331A0 について、D24 で 137A0 が 5 連続
warning 段階到達するか確認、demote 即実施 vs W4 集約待ち判断。

## §2 安全境界

全 0 制約遵守

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D23 review check + paper PnL 再 run |
| 2 | D24 chain (= 現行 recently_seen 5 件) |
| 3 | 137A0 5 連続到達確認 |
| 4 | 137A0 demote 検討 run (= 参考、6 件 simulation) |
| 5 | demote 判断 (= 即実施 / D25 / W4) |
| 6 | D24 hard check + 4 段階判定 |
| 7 | D24 trade plan + review template + paper PnL handoff |
| 8 | vault + HQ + 6 KPI |

## §4 完了条件

- D23 review 確認完了
- D24 candidate generation 完了
- 137A0 5 連続到達確認
- 137A0 demote sim 完了 (= 参考)
- demote 判断完了 (= D25 実施 推奨)
- D24 hard check + 判定完了
- D24 trade plan + review template 作成完了
- 全 0 制約遵守
- F282 plist + 3 DB 不変
- HQ + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**
