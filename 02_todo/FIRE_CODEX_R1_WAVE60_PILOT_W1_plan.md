---
id: FIRE-CODEX-R1-WAVE60-pilot-W1-plan
phase: 本番 v0 中核 / Wave 60-pilot-W1 / D1-D5 集約 + F111-real-batch 要件
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-21
---

# Wave 60-pilot-W1 Plan — Small Manual Live Pilot W1 Aggregation / F111-Real-Batch Requirements v1.0

## §1 目的

D1-D5 = 5 営業日 pilot を read-only で集約し、**実 entry 0/5 の理由を分類**。
F111-real-batch wave で「**実在 ticker + pilot 損失上限内 + 取引可能銘柄**」の
候補を出すための要件を確定。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + D1-D5 5 artifact 集約 read-only |
| 2 | 実 entry 0/5 理由分類 (= 5 blocker) |
| 3 | pattern promote/suppress/watch 仮分類 |
| 4 | F111-real-batch 要件 docs (~/fire-vault/03_design/F111_real_batch_requirements_2026-05-14.md) |
| 5 | W61-pre 要件 (price/return/paper_pnl 連携) |
| 6 | next wave priority 確定 |
| 7 | Codex 4 lane stdin audit (= 短文 prompt) |
| 8 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- 実発注 / 楽天 / iSPEED / Computer Use 0
- LINE 送信 / DB write / token / API / launchctl / plist / cron 0
- ファイル write は fire-vault/* のみ
- FIRE 本体コード変更 0
- F282 plist + 3 DB 完全不変

## §4 完了条件

- D1-D5 集約 (= 5 artifact / trade plan / review 全 read-only 確認)
- 実 entry 0/5 理由分類完了
- pattern 仮分類完了
- F111-real-batch 要件 docs 完成
- W61-pre 要件整理
- next wave priority 確定
- Codex 4 lane audit 実施 (= 完全 reply / self-audit いずれか)
- F282 / 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token / API /
launchctl / plist / cron / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
