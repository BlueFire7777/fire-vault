---
id: FIRE-CODEX-R1-WAVE60-PILOT-D11-plan
phase: 本番 v0 中核 / Wave 60-pilot-D11 / paper PnL linkage 初運用 day
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-28
pilot_day: D11
---

# Wave 60-pilot-D11 Plan — D11 Manual Live Pilot Trade Plan with Paper PnL Linkage v1.0

## §1 目的

W61-impl で paper PnL preview runner 実装後の初 D-day。D11 trade plan +
review template + D10 review 確認 + D10/D11 paper PnL preview 連携で、
**候補 → 判断 → review → paper PnL を一連の実運用フロー**として回す。

## §2 安全境界

- DB write: **禁止** (本 wave は read-only 確認のみ)
- production / develop DB 接続: **禁止**
- staging DB: **read-only のみ** (= URI mode=ro + PRAGMA query_only=ON)
- LINE / token / API / launchctl / plist / cron / VACUUM / workflow /
  --no-verify / git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED /
  Computer Use / TODO Excel: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist + 3 DB md5) |
| 2 | D10 review 確認 (= status: blank?) |
| 3 | D10 paper PnL preview 確認 (= /tmp/fire_d10_prep/...) |
| 4 | D11 F111-real-batch chain (= strict + recent + max=20) |
| 5 | F062 preview chain |
| 6 | AFTER-R1 night batch chain |
| 7 | D11 paper PnL preview chain |
| 8 | top_candidates 整理 + D10/D11 overlap |
| 9 | D11 hard check + GO/HOLD/NO-GO 判定 |
| 10 | D11 trade plan + review template + paper PnL handoff |
| 11 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- D10 review 確認完了
- D10 paper PnL preview 確認完了
- D11 candidate generation 完了
- D11 hard check 完了 (= 9 invariants)
- Pilot GO/HOLD/NO-GO 判定完了
- D11 trade plan + review template 作成完了
- D11 paper PnL handoff 手順明記 (= 場後 / 翌日 / h20)
- DB write 0 / LINE 0 / token 0 / API 0
- launchctl 0 / plist 0 / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist + 3 DB mtime/size 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token / API /
launchctl / plist / cron / workflow / --no-verify / git push / sudo /
rm -rf / TODO Excel

## §6 Codex 運用

日次 trade plan = 本線主導。本 wave は code 変更なし → Codex skip 妥当。
ロジック変更 / W2 集約時に 4 lane 以上を使う。
