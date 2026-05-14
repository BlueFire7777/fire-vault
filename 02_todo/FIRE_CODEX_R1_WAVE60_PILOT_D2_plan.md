---
id: FIRE-CODEX-R1-WAVE60-pilot-D2-plan
phase: 本番 v0 中核 / Wave 60-pilot-D2 / D2 Manual Live Pilot
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
pilot_day: D2
---

# Wave 60-pilot-D2 Plan — Artifact Source Aware Day 2 Manual Live Pilot Trade Plan v1.0

## §1 目的

2026-05-15 (金) D2 の少額手動実弾パイロット trade plan を作成。
W60-integration の **artifact_source 判定** を必ず使い、
**実 F062 batch 由来 / synthetic / unknown** を明示したうえで、藤原さんが
入る / 見送る を手動判断できる状態にする。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + 既存 fixture 確認 (= 実 F062 batch 稼働状況確認) |
| 2 | D2 MVP artifact 生成 (= reports/after_r1/、base_date=2026-05-15) |
| 3 | artifact_source 判定 + Pilot GO/HOLD/NO-GO |
| 4 | D2 trade plan 作成 (= 2026-05-15_manual_live_pilot_trade_plan.md) |
| 5 | D2 review template 作成 (= 2026-05-15_manual_live_pilot_review.md) |
| 6 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- 実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token / API call /
  launchctl / plist / cron 全 0
- ファイル write は reports/after_r1/ + fire-vault/04_daily,02_todo/ + /tmp/ のみ
- FIRE 本体コード変更 0
- F282 plist + 3 DB 完全不変

## §4 完了条件

- D2 MVP artifact 4 種類生成
- artifact_source 判定完了 + 4 outputs 反映
- Pilot GO/HOLD/NO-GO 明確
- D2 trade plan 完成
- D2 review template 完成
- top 1-3 整理 + D2 1 銘柄推奨 or skip 推奨
- リスク上限明記
- iSPEED 手動発注のみ明記
- F282 / 3 DB 不変
- vault に plan + results + HQ 1 ブロック + 6 KPI

## §5 停止条件

実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token / API /
launchctl / plist / cron / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
