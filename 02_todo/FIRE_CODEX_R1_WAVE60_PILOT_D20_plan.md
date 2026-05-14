---
id: FIRE-CODEX-R1-WAVE60-PILOT-D20-plan
phase: 本番 v0 中核 / Wave 60-pilot-D20 / GO_CONDITIONAL 復帰判定 + 3798 demote 初実施
priority: 最高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-10
pilot_day: D20
design_doc: ~/fire-vault/03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
---

# Wave 60-pilot-D20 Plan — D20 Recovery Criteria / 3798 Demote v1.0

## §1 目的

W3 で確定した D20 recovery criteria 9 条件を初運用。recently_seen を 5 件
(= 8747,5729,3489,340A0,3798) に拡張、3798 demote 初実施で D20 候補再生成。
期待 top: 137A0 / 331A0 / 7991。HOLD 解除 → GO_CONDITIONAL 復帰判定。

## §2 安全境界

- 全 0 制約遵守: DB write 0 / production-develop 接続 0 / staging read-only /
  LINE 0 / token 0 / API 0 / launchctl 0 / plist 0 / cron 0 / 実発注 0 /
  楽天 0 / iSPEED 0 / Computer Use 0 / workflow 0 / --no-verify 0 /
  git push 0 / sudo 0 / rm -rf 0 / TODO Excel 0

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D14 review check + paper PnL --review-md 再 run |
| 2 | recently_seen 拡張 (= 5 件、3798 追加) |
| 3 | D20 chain (= F111 + F062 + AFTER-R1 + paper PnL) |
| 4 | 3798 demote 動作確認 |
| 5 | 新 top 確認 (= 137A0 / 7991 / 331A0 想定) |
| 6 | W3 recovery criteria 9 条件判定 |
| 7 | 4 段階判定 (= GO/GO_CONDITIONAL/HOLD/NO-GO) |
| 8 | D20 trade plan + review template + paper PnL handoff |
| 9 | vault plan + results + HQ + 6 KPI |

## §4 完了条件

- D14 review 確認完了
- D14 paper PnL --review-md 再 run 完了
- recently_seen 5 件拡張完了 (= 3798 追加)
- D20 candidate generation 完了
- 3798 demote 動作確認 (= caution へ)
- 新 top 確認 (= 137A0 rank 1、7991 rank 2 機械)
- W3 9 条件判定完了
- D20 Pilot 判定完了
- D20 trade plan + review template 作成完了
- D20 paper PnL handoff 手順明記
- 全 0 制約遵守
- F282 plist + 3 DB mtime/size 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env /
API / LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED /
Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel
