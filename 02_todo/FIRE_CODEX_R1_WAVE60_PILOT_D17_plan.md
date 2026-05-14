---
id: FIRE-CODEX-R1-WAVE60-PILOT-D17-plan
phase: 本番 v0 中核 / Wave 60-pilot-D17 / HOLD 解除判定 (D14 review 反映後)
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-05
pilot_day: D17
---

# Wave 60-pilot-D17 Plan — Review Gap Resolved / New Top Candidate Recheck v1.0

## §1 目的

D16 で 340A0 demote により HOLD #2 解消。残る HOLD #1 (= review missing 6
連続) を D14 review 記入で解消可能か判定。D17 で:
1. D14 review 必須 5 項目厳密 check
2. D14 paper PnL --review-md 再 run
3. D17 chain 再生成 (= 340A0 demote 維持)
4. HOLD 解除 → GO_CONDITIONAL 復帰 判定

## §2 安全境界

- 全 0 制約遵守: DB write 0 / production-develop 接続 0 / staging read-only /
  LINE 0 / token 0 / API 0 / launchctl 0 / plist 0 / cron 0 / 実発注 0 /
  楽天 0 / iSPEED 0 / Computer Use 0 / workflow 0 / --no-verify 0 /
  git push 0 / sudo 0 / rm -rf 0 / TODO Excel 0

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D14 review 厳密 check (= resolved/partial/unresolved) |
| 2 | D14 paper PnL --review-md 再 run |
| 3 | recently_seen_codes 維持確認 (= 8747,5729,3489,340A0) |
| 4 | D17 chain (= F111 + F062 + AFTER-R1 + paper PnL) |
| 5 | 340A0 demote 維持確認 + 新 top 候補確認 |
| 6 | D17 hard check + 4 段階判定 |
| 7 | D17 trade plan + review template + paper PnL handoff |
| 8 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- D14 review 確認完了 (= 5 項目厳密 check)
- D14 paper PnL --review-md 再 run 完了
- recently_seen_codes 維持確認
- D17 candidate generation 完了
- 340A0 demote 維持確認
- D17 hard check 完了
- D17 Pilot 判定完了
- D17 trade plan + review template 作成完了
- D17 paper PnL handoff 手順明記
- 全 0 制約遵守
- F282 plist + 3 DB mtime/size 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env /
API / LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED /
Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel
