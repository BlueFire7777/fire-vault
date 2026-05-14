---
id: FIRE-CODEX-R1-WAVE60-PILOT-D12-plan
phase: 本番 v0 中核 / Wave 60-pilot-D12 / review-paper PnL feedback 反映 day
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-29
pilot_day: D12
---

# Wave 60-pilot-D12 Plan — D12 Manual Live Pilot Trade Plan with Review and Paper PnL Feedback v1.0

## §1 目的

D11 で paper PnL chain 初運用、D12 では D10/D11 review、paper PnL preview、
D9-D12 候補 overlap、価格鮮度、流動性確認を踏まえ、GO/HOLD/NO-GO を判定する。
FIRE を「候補を出す」だけでなく **review と paper PnL を踏まえて継続判断する運用**
へ進める。

## §2 安全境界

- DB write: **禁止** (本 wave は read-only 確認のみ)
- production / develop DB 接続: **禁止**
- staging DB: **read-only のみ**
- LINE / token / API / launchctl / plist / cron / VACUUM / workflow /
  --no-verify / git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED /
  Computer Use / TODO Excel: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist + 3 DB md5) |
| 2 | D10/D11 review 確認 (= 両方 blank? 想定) |
| 3 | D10/D11 paper PnL preview 既存確認 |
| 4 | D12 F111-real-batch chain |
| 5 | F062 preview chain |
| 6 | AFTER-R1 night batch chain |
| 7 | D12 paper PnL preview chain |
| 8 | D9-D12 candidate overlap 確認 |
| 9 | D12 hard check + GO/HOLD/NO-GO 判定 |
| 10 | D12 trade plan + review template + paper PnL handoff |
| 11 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- D10/D11 review 確認完了
- D10/D11 paper PnL preview 確認完了
- D12 candidate generation 完了
- D12 hard check 完了 (= 9 invariants)
- D9-D12 overlap 確認完了
- Pilot GO/HOLD/NO-GO 判定完了
- D12 trade plan + review template 作成完了
- D12 paper PnL handoff 手順明記
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
W2 集約またはロジック変更時に 4 lane 以上を使う。
