---
id: FIRE-CODEX-R1-WAVE60-pilot-D6-plan
phase: 本番 v0 中核 / Wave 60-pilot-D6 / D6 real_batch + research enriched
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-21
pilot_day: D6
---

# Wave 60-pilot-D6 Plan — Real Batch Research-Enriched Manual Live Pilot v1.0

## §1 目的

W60-research-advisory-staging で確立した **F111-real-batch + research_advisory
enriched** chain で 2026-05-21 (木) D6 の trade plan を作成。**D1-D5 全 day
実 entry 0/5 から、初の実 trade 可能 day** を目指す。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + W60-research-advisory-staging 成果物確認 |
| 2 | enriched chain 再 run (= reports/after_r1/ に 2026-05-21 生成) |
| 3 | D6 hard check (= 9 invariants: 既存 8 + top_candidates ≥1) |
| 4 | Pilot GO/HOLD/NO-GO 判定 |
| 5 | D6 trade plan + review template 作成 (= 中小型 liquidity 手動 check 欄) |
| 6 | D6 → W2 引き継ぎ材料整理 |
| 7 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- staging DB は read-only (= URI mode=ro)
- production / develop DB 接続禁止
- DB write 0 / LINE / token / API / launchctl / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist + 3 DB 完全不変

## §4 完了条件

- D6 chain 再 run 成功 (= reports/after_r1/2026-05-21_*)
- 9 hard invariants 全 PASS
- Pilot GO/HOLD/NO-GO 明確
- top_candidates ≥1 件、または 0 件理由明記
- D6 trade plan 完成 (= 中小型 caveat + liquidity check 必須)
- D6 review template 完成
- F282 / 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

実発注 / 楽天 / iSPEED / Computer Use / DB write / production-develop 接続 /
LINE / token / API / launchctl / plist / cron / workflow / --no-verify /
git push / sudo / rm -rf / TODO Excel が必要 → 即停止 + HQ
