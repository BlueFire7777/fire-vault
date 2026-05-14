---
id: FIRE-CODEX-R1-WAVE60-research-advisory-staging-plan
phase: 本番 v0 中核 / Wave 60-research-advisory-staging / 実 ticker label 付与
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60-research-advisory-staging Plan — Research Advisory Label Integration v1.0

## §1 目的

W60-F111-real-batch-staging で staging real ticker を candidate 化したが、
全 `research_advisory_label=neutral` のため morning_line_material top_candidates が
0 件だった。本 wave で staging の **research_watchlist_signals** (= 13,695 rows) を
read-only で参照し、real_batch 候補に **boost / boost_with_caution / caution /
neutral** を付与。**top_candidates に実在 ticker を 1 件以上出す**。

## §2 構成

| step | 内容 |
|---|---|
| 1 | staging research source 調査 (= research_watchlist_signals / advisory_*) |
| 2 | label mapping 設計 (= watchlist_decision + final_rank_label + final_score) |
| 3 | run_f111_real_batch_staging.py に enrichment 実装 |
| 4 | INNER JOIN で signal-required mode 追加 (`--require-research-signal`) |
| 5 | chain smoke (= top_candidates 出現確認) |
| 6 | tests 18 件追加 |
| 7 | Codex 4 lane stdin audit (= 短文 factual confirm) |
| 8 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- staging DB は **read-only** (= URI mode=ro)
- production / develop DB 接続禁止
- DB write 0
- LINE / token / API / launchctl / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist + 3 DB 完全不変

## §4 完了条件

- staging research source 調査完了
- label mapping 実装 + 9 件 test
- enrichment 実装 + 8 件 test (= signal join / 安全 invariant)
- chain smoke で top_candidates ≥1 件確認
- f111_input_source = f111_real_batch 維持
- tests 全 PASS
- Codex 4 lane audit 実施 (= 完全 reply or self-audit)
- DB write 0
- F282 plist + 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

DB write / production-develop 接続 / schema migration / LINE / token / API /
launchctl / plist / cron / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
