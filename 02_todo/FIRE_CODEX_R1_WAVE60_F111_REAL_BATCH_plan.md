---
id: FIRE-CODEX-R1-WAVE60-F111-real-batch-plan
phase: 本番 v0 中核 / Wave 60-F111-real-batch / 実 ticker 候補生成
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60-F111-real-batch Plan — F111 Real Batch Candidate Generation v1.0

## §1 目的

D1-D5 pilot で実 entry 0/5 となった主要要因 (= sample ticker / 値嵩株 /
F111 朝 batch 不在) を解消するため、AFTER-R1 MVP に **F111_REAL_BATCH_MARKERS**
推定 + **sample ticker exclusion** + **risk filter** + **9th invariant 強制**
を実装し、実在 JP 株候補を pilot 上限内で top に出せる状態にする。

実際の **staging real SELECT runner** は本 wave スコープ外 (= 別 wave)。
本 wave は **AFTER-R1 側の認識・フィルタ logic** + **safe fixture chain smoke**
までを完了させる。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + 設計 |
| 2 | F111_REAL_BATCH_MARKERS + infer 拡張 (= 4 keys、≥3 hit で f111_real_batch) |
| 3 | sample ticker exclusion + tradable filter + risk filter (= 100 株 ≤ 15,000 円) |
| 4 | morning_line_material で 9th invariant 強制 (= 3 種 excluded 候補を top 抑制) |
| 5 | safe fixture chain smoke (= 実在 ticker 4 件) |
| 6 | tests 21 件追加 |
| 7 | Codex 4 lane stdin audit (= 短文 + 単一観点 ≤80 words) |
| 8 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- 実発注 / 楽天 / iSPEED / Computer Use 0
- LINE 送信 / DB write / token / API / launchctl / plist / cron 0
- staging DB 実 SELECT は本 wave スコープ外
- ファイル write は scripts/jobs/ + tests/ + fire-vault/ + /tmp/ のみ
- F111 / F062 / F282 / DATA-R3 / readiness / Ops / wrapper 本体不触
- F282 plist + 3 DB 完全不変

## §4 完了条件

- F111_REAL_BATCH_MARKERS 定義 + 推定拡張
- sample ticker / tradable / risk filter 実装
- 9th invariant 強制 (= 3 種 excluded 候補を morning_line_material top から抑制)
- exclusions_summary 集計 field 追加
- safe fixture chain smoke 成功 (= f111_input_source=f111_real_batch 自動推定)
- tests 全 PASS (= +20+ 件)
- pytest collected の変化を報告
- Codex 4 lane audit 実施 (= 短文 prompt で改善試行)
- DB write / LINE / token / API / launchctl 0
- F282 plist + 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

実発注 / 楽天 / iSPEED / Computer Use / DB write (production/develop/staging
を問わず) / LINE / token / API / launchctl / plist / cron / workflow /
--no-verify / git push / sudo / rm -rf / TODO Excel が必要 → 即停止 + HQ
