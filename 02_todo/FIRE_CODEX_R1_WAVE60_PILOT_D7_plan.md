---
id: FIRE-CODEX-R1-WAVE60-pilot-D7-plan
phase: 本番 v0 中核 / Wave 60-pilot-D7 / 再現性確認 day
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-22
pilot_day: D7
---

# Wave 60-pilot-D7 Plan — Real Batch Research-Enriched Manual Live Pilot Day 7 v1.0

## §1 目的

D6 同 chain で 2026-05-22 (金) D7 の trade plan を作成し、**chain 再現性 +
D6 review 反映 + 流動性確認** を確認。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + D6 review status 確認 |
| 2 | chain 再 run for base_date=2026-05-22 |
| 3 | 9 hard invariants check + D6/D7 candidate overlap 確認 |
| 4 | D7 GO/HOLD/NO-GO 判定 |
| 5 | D7 trade plan + review 作成 (= D6 missing 反映 + 同候補 caveat) |
| 6 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

staging DB read-only / production-develop 接続禁止 / DB write 0 / LINE 0 /
token 0 / API 0 / launchctl 0 / 実発注 0 / 楽天 0 / iSPEED 0 / Computer Use 0

## §4 完了条件

- D7 chain 動作 + 9 invariants PASS
- D6/D7 overlap 確認 + caveat 記載
- D7 trade plan + review 作成
- D6 review missing/反映明記
- F282 plist + 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

DB write / production-develop 接続 / LINE / token / API / launchctl / plist /
cron / 実発注 / 楽天 / iSPEED / Computer Use / workflow / --no-verify /
git push / sudo / rm -rf / TODO Excel が必要 → 即停止 + HQ
