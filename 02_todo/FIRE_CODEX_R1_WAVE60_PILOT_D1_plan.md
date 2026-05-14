---
id: FIRE-CODEX-R1-WAVE60-pilot-D1-plan
phase: 本番 v0 中核 / Wave 60-pilot-D1 / Small Manual Live Pilot D1
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
pilot_day: D1
---

# Wave 60-pilot-D1 Plan — Small Manual Live Pilot Day 1 Trade Plan v1.0

## §1 目的

W60-pilot-pre で正式化した運用 plan に基づき、**初日 (= 2026-05-14 木) の
trade plan + review template を作成**。藤原さんが朝に候補を確認し、
**入る / 見送る を手動判断できる状態** にする。FIRE / Claude Code は発注しない。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + reports/after_r1 確認 |
| 2 | 5/14 MVP artifact 生成 (= synthetic fixture 入力、reports/after_r1/ に出力) |
| 3 | Pilot GO/NO-GO 判定 (= 4 成果物 + freshness + forbidden + auto_order + manual_review) |
| 4 | 2026-05-14_manual_live_pilot_trade_plan.md 作成 |
| 5 | 2026-05-14_manual_live_pilot_review.md 作成 (= blank 雛形) |
| 6 | vault plan + results + HQ 1 ブロック + 6 KPI |

## §3 安全境界

- 実発注 / 楽天証券操作 / iSPEED 操作 / Computer Use / 自動発注 全 0
- DB write / LINE / token / API / launchctl / plist / cron 全 0
- ファイル write は reports/after_r1/ + fire-vault/04_daily/ のみ
- FIRE 本体コード変更 0
- F282 plist + 3 DB 完全不変

## §4 完了条件

- 4 成果物 reports/after_r1/ に生成 (= base_date=2026-05-14)
- Pilot GO/NO-GO 判定明確
- 5/14 trade plan 完成
- 5/14 review template 完成
- top 1-3 候補整理 + 初日 1 銘柄推奨
- リスク上限明記
- iSPEED 手動発注のみ明記
- F282 / 3 DB 不変
- vault に plan + results + HQ 1 ブロック + 6 KPI

## §5 停止条件

実発注 / 楽天証券 / iSPEED / Computer Use / DB write / LINE / token / API /
launchctl / plist / cron / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
