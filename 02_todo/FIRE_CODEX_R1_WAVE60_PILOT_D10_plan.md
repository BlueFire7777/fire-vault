---
id: FIRE-CODEX-R1-WAVE60-PILOT-D10-plan
phase: 本番 v0 中核 / Wave 60-pilot-D10 / refreshed price 初実弾検証 day
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-27
pilot_day: D10
---

# Wave 60-pilot-D10 Plan — Refreshed Price Manual Live Pilot Trade Plan v1.0

## §1 目的

W60-jquants-daily-refresh-staging で staging market_prices_daily を 2026-05-08
→ 2026-05-14 へ catch-up (12 銘柄、48 row insert)。D10 では更新後価格を使って
**refreshed price 初実弾 day** の trade plan + review template を作る。

D9 で残った stale caveat (= 6 営業日 gap) が refresh で軽減。340A0/3798/137A0
の 3 候補を中心に Pilot GO/HOLD/NO-GO 判定。

## §2 安全境界

- DB write: **禁止** (本 wave は read-only 確認のみ)
- production / develop DB 接続: **禁止**
- staging DB: **read-only のみ** (前 wave で write 完了済)
- LINE / token / API / launchctl / plist / cron / VACUUM / workflow /
  --no-verify / git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED /
  Computer Use / TODO Excel: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist + 3 DB md5 + 前 wave artifacts 確認) |
| 2 | price freshness summary (= 12 銘柄 stale 5/8 vs refreshed 5/14) |
| 3 | 重点 3 銘柄 (340A0/3798/137A0) risk + label + score 詳細 |
| 4 | D10 hard check (= 9 invariants) |
| 5 | Pilot GO/HOLD/NO-GO 判定 |
| 6 | D10 trade plan 作成 (= 04_daily/2026-05-27_..._plan.md) |
| 7 | D10 review template 作成 (= 同 _review.md) |
| 8 | W2/W61 引き継ぎ項目 整理 |
| 9 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- D10 refreshed price chain 確認完了
- price freshness summary 作成
- top_candidates 1 件以上確認 (= 3 件)
- D10 hard check 完了 (= 9 invariants)
- Pilot GO/HOLD/NO-GO 判定完了
- D10 trade plan 作成完了
- D10 review template 作成完了 (= actual price confirmation + liquidity + event check 欄あり)
- DB write 0 / LINE 0 / token 0 / API 0
- launchctl 0 / plist 0 / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist mtime/size/next run 不変
- production/develop/staging DB mtime/size 不変 (= 本 wave で write 0)
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token / API /
launchctl / plist / cron / workflow / --no-verify / git push / sudo /
rm -rf / TODO Excel

## §6 Codex 運用

日次 trade plan = 本線主導。本 wave は code 変更なし、artifacts 既生成済の
read-only 確認のみ → **Codex skip 妥当**。W2 集約 or ロジック変更時に 4 lane 以上を使う。
