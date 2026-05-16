---
id: FIRE-CODEX-R1-WAVE60-PILOT-D30-plan
phase: 本番 v0 中核 / Wave 60-pilot-D30 / 7991 demote 本実行 + sector 一時縮退
priority: 最高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-24
pilot_day: D30
---

# Wave 60-pilot-D30 Plan — 7991 Demote 本実行 / Sector 一時縮退 v1.0

## §1 目的

D29 で 7991 rank 1 5 連続 warning 発動 + demote sim 完了。本 wave で
7991 を recently_seen に追加 (= demoted_count=7) して **demote 本実行**、
新 top 1 = 9130 (運輸) 昇格、sector 一時 2 種縮退を記録、HOLD #2 完全回避
継続を判定。GO_CONDITIONAL maintained 11 連続を達成する。

## §2 安全境界

全 0 制約遵守 (= demote 本実行は read-only F111 chain で確認、DB write 0)

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D29 review check + paper PnL 再 run |
| 2 | D30 F111 chain (= recently_seen 7 件) ★ 本実行 |
| 3 | D30 paper PnL preview |
| 4 | 7991 demote 化確認 (= caution status) |
| 5 | 新 top 5 確認 (= 9130 / 331A0 / 4389 / 9247 / 4317) |
| 6 | sector 一時 2 種縮退確認 |
| 7 | HOLD #2 完全回避継続 (= 15 連続) 確認 |
| 8 | D30 hard check + 4 段階判定 |
| 9 | Codex factual-confirm (= D29 sim vs D30 本実行 一致) |
| 10 | D30 trade plan + review template + paper PnL handoff |
| 11 | W5 集約引き継ぎ材料 |
| 12 | vault + HQ + 6 KPI |

## §4 完了条件 (= /goal 13 項目)

1. baseline 取得 (3 DB md5 + F282 plist) と final 完全一致
2. D29 review status 明示確認
3. D29 paper PnL --review-md 再 run 完了
4. D30 chain 実行 (= recently_seen 7 件、demoted_count=7)
5. 7991 が caution/demoted へ落ちる
6. D30 top 3 = 9130 / 331A0 / 4389
7. sector 3 種 → 2 種一時縮退記録
8. HOLD #2 完全回避継続判定
9. GO/GO_CONDITIONAL/HOLD/NO-GO 提示
10. vault 4 file 作成
11. 安全境界 0 違反
12. Codex factual-confirm 1 lane 以上 PASS
13. D31 引き継ぎ + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**
