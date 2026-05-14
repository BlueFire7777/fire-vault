---
id: FIRE-CODEX-R1-WAVE60-PILOT-D9-plan
phase: 本番 v0 中核 / Wave 60-pilot-D9 / W60-F111-preset-tune 初実弾検証 day
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-26
pilot_day: D9
---

# Wave 60-pilot-D9 Plan — D9 Strict Recent Real-Batch Manual Live Pilot Trade Plan v1.0

## §1 目的

W60-F111-preset-tune で実装した 3 軸 (strict + recently_seen + max=20) を D9
に **初適用**。D6-D8 で 100% 重複した top 1-3 (= 8747/5729/3489) を caution
へデモート → 見送り、top 4 以降の boost_with_caution 候補から実弾候補 #1 を
特定し、Pilot GO/HOLD/NO-GO を判定する。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist + 3 DB md5 + chain runner 確認) |
| 2 | F111-real-batch chain (strict + recent=8747,5729,3489 + max=20) |
| 3 | F062 preview chain (= --preview-json list 形式) |
| 4 | AFTER-R1 night batch (= --mode mvp --task all、9 invariants) |
| 5 | top_candidates 評価 (= top 1-3 caution skip、top 4 以降 entry) |
| 6 | Pilot GO/HOLD/NO-GO 判定 |
| 7 | D9 trade plan + review template 作成 (= 04_daily/) |
| 8 | vault plan + results + HQ 1-block + 6 KPI |

## §3 安全境界

- API call / token / .env / env 全体参照: **禁止**
- DB write (production / develop / staging): **禁止**
- production / develop DB 接続: **禁止**
- staging DB: **read-only のみ** (URI mode=ro)
- LINE / workflow / cron / launchctl / plist / VACUUM / --no-verify /
  git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED / Computer Use: **禁止**
- TODO Excel 更新: **禁止**

## §4 完了条件

- D9 candidate generation 完了 (= strict + recent + max=20 適用確認)
- top 1-3 repeated candidates が caution / skip 扱い
- top 4 以降の候補評価完了
- Pilot GO/HOLD/NO-GO 判定完了
- D9 trade plan 作成完了 (= 2026-05-26_manual_live_pilot_trade_plan.md)
- D9 review template 作成完了 (= 同 _review.md)
- DB write 0 / production-develop 接続 0 / staging read-only のみ
- LINE 0 / token 0 / API 0 / launchctl 0 / plist 0 / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist + 3 環境 DB mtime/size 不変
- vault に plan + results
- HQ 1-block + 6 KPI

## §5 停止条件

16 項目のいずれかが必要になったら **即停止 + HQ 確認**:
実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token /
.env / API / launchctl / plist / cron / workflow / --no-verify /
git push / sudo / rm -rf / TODO Excel

## §6 Codex 運用

日次 trade plan = 本線主導。Codex は read-only factual-confirm のみ、
必要に応じて 4 lane 以下、短文単一観点。W2 集約またはロジック変更時に
4 lane 以上を使う。本 wave では code 変更なし → Codex skip 可。
