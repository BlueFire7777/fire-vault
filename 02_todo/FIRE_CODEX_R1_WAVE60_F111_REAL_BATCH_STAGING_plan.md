---
id: FIRE-CODEX-R1-WAVE60-F111-real-batch-staging-plan
phase: 本番 v0 中核 / Wave 60-F111-real-batch-staging / staging real candidate 供給
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60-F111-real-batch-staging Plan — F111 Real Batch Staging Read-Only Candidate Source v1.0

## §1 目的

staging DB を **read-only** で参照し、**実在 JP 株 ticker + risk 上限内 + tradable**
候補を生成する `run_f111_real_batch_staging.py` を実装。F062 → AFTER-R1 chain で
`f111_input_source = f111_real_batch` 自動推定。

## §2 構成

| step | 内容 |
|---|---|
| 1 | staging DB schema 確認 (= read-only probe) |
| 2 | run_f111_real_batch_staging.py 新規実装 (= ~360 行) |
| 3 | sample 除外 / tradable / risk filter (= W60-F111-real-batch logic 整合) |
| 4 | F062 actual format 互換 output (= chunks/selected_rows/...) |
| 5 | staging → F111-real-batch → F062 → AFTER-R1 完全 chain smoke |
| 6 | tests 37 件追加 |
| 7 | Codex 4 lane stdin audit (= 短文 factual confirm) |
| 8 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- **staging DB は read-only** (= sqlite URI `mode=ro`)
- **production / develop DB 接続は禁止** (= path 検知 refuse)
- DB write 0
- LINE / token / API / launchctl / cron / plist 変更 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist + 3 DB 完全不変

## §4 完了条件

- run_f111_real_batch_staging.py 完成
- staging DB read-only 接続 + production/develop refuse 動作
- ticker 5 桁 → 4 桁 正規化
- sample / tradable / risk filter 適用
- F062 actual format 互換 output
- 完全 chain smoke で f111_input_source=f111_real_batch 自動推定確認
- tests 全 PASS
- Codex 4 lane audit 実施
- DB write 0 / 接続 staging read-only のみ
- F282 plist + 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

DB write / production-develop 接続 / LINE / token / API / launchctl / plist /
cron / workflow / --no-verify / git push / sudo / rm -rf / TODO Excel が必要
→ 即停止 + HQ
