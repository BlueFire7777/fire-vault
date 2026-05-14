---
id: FIRE-CODEX-R1-WAVE60-JQUANTS-DAILY-REFRESH-STAGING-plan
phase: 本番 v0 中核 / Wave 60-jquants-daily-refresh-staging / 価格鮮度真の解決
priority: 最高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
hq_markers:
  - HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1
  - HQ_APPROVE_JQUANTS_TOKEN_READ=1
---

# Wave 60-jquants-daily-refresh-staging Plan — J-Quants Daily Refresh Staging / Price Freshness Recovery v1.0

## §1 目的

W60-pilot-D9 で entry candidate #1 = **340A0 ジグザグ** が boost_with_caution +
risk=1,990 円 で GO_CONDITIONAL 判定された。ただし market_prices_daily が
**2026-05-08 で 6 営業日 stale** であり、寄付き実価格との乖離リスクを抱えていた。

本 wave は staging 限定で J-Quants V2 daily refresh を実行し、market_prices_daily
の鮮度を回復、D10 以降の F111-real-batch / research advisory / AFTER-R1 MVP
候補が **真の最新価格** で risk 判定できる状態にする。

## §2 安全境界

- **production / develop DB 接続**: 禁止 (= md5 不変確認)
- **staging DB 以外への write**: 禁止
- **token 値表示 / env 全体表示**: 禁止 (= 存在のみ len で確認)
- **API call**: J-Quants V2 daily refresh に必要な範囲のみ
- LINE / launchctl / plist / cron / VACUUM / workflow / --no-verify /
  git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED / Computer Use: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist + 3 DB md5 + market_prices_daily latest) |
| 2 | J-Quants runner 確認 (= scripts/jobs/run_jquants_daily_refresh.py) |
| 3 | 3-4 段 guard 確認 (= assert_staging_write_safe) |
| 4 | token 存在確認 (値非表示) |
| 5 | dry-run smoke (= API 0 / DB write 0、plan 確認のみ) |
| 6 | staging refresh 実行 (= HQ marker 2 個 + FIRE_ENV=staging + 12 銘柄 csv) |
| 7 | downstream smoke (F111 → F062 → AFTER-R1) |
| 8 | D10 候補比較 (refresh 前 vs 後) |
| 9 | Codex 4 lane factual-confirm |
| 10 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- J-Quants daily refresh 経路確認完了
- token 値非表示のまま必要確認完了
- staging 限定 refresh 実行
- market_prices_daily latest date 確認完了 (= 2026-05-08 → 2026-05-14 目標)
- downstream F111/F062/AFTER-R1 smoke 完了
- D10 で使うべき価格鮮度状態が明確
- production / develop DB 接続 0
- staging 以外への DB write 0
- LINE 0 / launchctl 0 / plist 0 / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist mtime/size/next run 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
token 値表示 / env 全体表示 / production-develop DB 接続 / staging 以外への write /
schema migration / J-Quants 以外の API call / LINE / launchctl / plist / cron /
workflow / --no-verify / git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED /
Computer Use / TODO Excel
