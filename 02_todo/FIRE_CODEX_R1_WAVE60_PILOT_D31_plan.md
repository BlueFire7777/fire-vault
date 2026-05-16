---
id: FIRE-CODEX-R1-WAVE60-PILOT-D31-plan
phase: 本番 v0 中核 / Wave 60-pilot-D31 / post-7991-demote 2 連続 + 9130 monitor
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-25
pilot_day: D31
---

# Wave 60-pilot-D31 Plan — Post-7991-Demote Continuity / 9130 Recurrence Monitor v1.0

## §1 目的

D30 で 7991 demote 本実行成功 → D31 で post-demote 2 連続到達確認、
9130 rank 1 = 2 連続到達、7991 caution 維持、sector 2 種継続 vs 3 種回復、
HOLD #2 完全回避 16 連続継続を判定。GO_CONDITIONAL maintained 12 連続を達成。

## §2 安全境界

全 0 制約遵守 (= read-only chain、code 変更 0)

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D30 review check + paper PnL 再 run |
| 2 | D31 chain (= recently_seen 7 件維持) |
| 3 | 9130 rank 1 2 連続到達確認 |
| 4 | 7991 caution / top 除外維持確認 |
| 5 | sector 2 種維持 vs 3 種回復確認 |
| 6 | HOLD #2 完全回避 16 連続継続確認 |
| 7 | Codex 4 lane factual-confirm |
| 8 | D31 hard check + 4 段階判定 |
| 9 | D31 trade plan + review template + paper PnL handoff |
| 10 | W5 集約引き継ぎ材料 |
| 11 | vault + HQ + 6 KPI |

## §4 完了条件 (= /goal 13 項目)

13 項目全 PASS 必須:
1. baseline 完全一致 (3 DB + F282)
2. D30 review status 明示確認
3. D30 paper PnL --review-md 再 run 完了
4. D31 chain 実行
5. 9130 rank 1 = 2 連続
6. 7991 caution 維持
7. D31 top 3 + D30 差分明示
8. sector 2 種 vs 3 種明示
9. HOLD #2 完全回避 16 連続判定
10. 4 段階判定
11. vault 4 file 作成
12. 安全境界 0 違反
13. Codex 4 lane PASS + D32/D33/W5 引き継ぎ + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**
