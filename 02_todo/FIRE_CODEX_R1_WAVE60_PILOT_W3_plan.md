---
id: FIRE-CODEX-R1-WAVE60-PILOT-W3-plan
phase: 本番 v0 中核 / Wave 60-pilot-W3 / HOLD 期間集約 + D20 recovery criteria
priority: 最高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-09 (W3 集約日)
aggregation_range: D14 (2026-06-02) - D19 (2026-06-09)
design_doc: ~/fire-vault/03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
---

# Wave 60-pilot-W3 Plan — HOLD Period Aggregation / D20 Recovery Criteria v1.0

## §1 目的

D14-D19 6 day HOLD 期間を集約し、review missing 構造、340A0 demote 効果、
3798 連続候補化リスク、recently_seen 拡張案、HOLD criteria 改訂案を整理。
D20 で HOLD 解除 → GO_CONDITIONAL 復帰する条件を確定する。

## §2 安全境界

- 全 0 制約遵守: DB write 0 / production-develop 接続 0 / staging read-only /
  LINE 0 / token 0 / API 0 / launchctl 0 / plist 0 / cron 0 / 実発注 0 /
  楽天 0 / iSPEED 0 / Computer Use 0 / workflow 0 / --no-verify 0 /
  git push 0 / sudo 0 / rm -rf 0 / TODO Excel 0

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D14-D19 全 artifact 集約 |
| 2 | D14-D19 summary 表 (pilot/review/paper PnL/HOLD status) |
| 3 | review missing 整理 (= 6 day 全 blank) |
| 4 | paper PnL status 整理 (= 全 pending) |
| 5 | 340A0 demote 効果評価 (= 4 連続維持) |
| 6 | 新 top 3 安定性評価 (= 3798/137A0/331A0 4 連続) |
| 7 | 3798 連続候補化リスク評価 (= D22 7 連続懸念) |
| 8 | HOLD criteria 改訂案 (= W2 → W3) |
| 9 | D20 recovery criteria 確定 (= GO_CONDITIONAL 復帰) |
| 10 | D20 handoff (= recently_seen 5 件、新 top 候補、必須確認項目) |
| 11 | Codex 4 lane factual-confirm |
| 12 | vault plan + results + design doc + HQ + 6 KPI |

## §4 完了条件

- D14-D19 集約完了
- review missing 整理完了
- paper PnL status 整理完了
- 340A0 demote 効果評価完了
- 3798 連続候補化リスク評価完了
- recently_seen 拡張案完成 (= 8747,5729,3489,340A0,**3798**)
- HOLD criteria 改訂案完成 (= 5 連続 warning、demote 例外、override review 必須)
- D20 recovery criteria 完成
- D20 handoff 完成
- Codex 4 lane factual-confirm 実施
- 全 0 制約遵守
- F282 plist + 3 DB mtime/size 不変
- vault + design doc + HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env /
API / LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED /
Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel

## §6 Codex 4 lane (factual-confirm、80 words/lane)

- A: D14-D19 review missing / paper PnL status 確認
- B: 340A0 demote 効果 / 新 top 推移 確認
- C: 3798 連続候補化 / recently_seen 拡張案 確認
- D: D20 GO/HOLD/NO-GO criteria / HOLD 改訂案 確認
