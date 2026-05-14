---
id: FIRE-CODEX-R1-WAVE60-PILOT-D13-plan
phase: 本番 v0 中核 / Wave 60-pilot-D13 / review gap closure + DATA-R3 接続経路確認
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-01
pilot_day: D13
hq_markers:
  - HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1
  - HQ_APPROVE_JQUANTS_TOKEN_READ=1
---

# Wave 60-pilot-D13 Plan — D13 Focused Refresh / Review Gap Closure / Manual Live Pilot Trade Plan v1.0

## §1 目的

D12 まで GO_CONDITIONAL 3 連続 + review_missing 3 連続。D13 は **GO_CONDITIONAL
を GO 寄りに改善** を目標とし:
- D10/D11/D12 review 欠損を明示処理
- focused J-Quants refresh で価格鮮度を上げる試行 (= 可能なら staging write、不可なら actual confirm 強制)
- DATA-R3 freshness gate 接続経路を可能な範囲で確立
- D9-D13 candidate overlap 整理

## §2 安全境界

- production / develop DB 接続: **禁止**
- staging DB: **read-only 基本**、focused refresh 時のみ staging write
- token 値表示 / env 全体表示: **禁止**
- API call: J-Quants daily_quotes の D13 候補 focused refresh に限定
- LINE / launchctl / plist / cron / VACUUM / workflow / --no-verify / git push /
  sudo / rm -rf / 実発注 / 楽天 / iSPEED / Computer Use / TODO Excel: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist + 3 DB md5) |
| 2 | D10/D11/D12 review status 確認 (= 全 blank 想定) |
| 3 | D10/D11/D12 paper PnL preview 既存確認 |
| 4 | D13 F111-real-batch chain (pre-refresh) |
| 5 | focused J-Quants refresh 試行 (= 12 銘柄 csv、HQ marker 2 個、staging-only) |
| 6 | DATA-R3 freshness JSON 生成試行 + AFTER-R1 接続 |
| 7 | D13 chain 再 run (= F062 + AFTER-R1 + paper PnL preview) |
| 8 | D9-D13 candidate overlap 確認 |
| 9 | D13 hard check + GO/HOLD/NO-GO 判定 |
| 10 | D13 trade plan + review template + paper PnL handoff |
| 11 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- D10-D12 review 確認完了 (= 3 連続 blank 明記)
- D10-D12 paper PnL preview 確認完了
- D13 candidate generation 完了
- focused refresh 試行完了 (= 実行 or skip 理由明記)
- DATA-R3 freshness 接続試行 (= path 確認、verdict 反映)
- D13 hard check 完了 (= 9 invariants)
- Pilot 判定完了
- D13 trade plan + review template 作成完了
- D13 paper PnL handoff 手順明記
- production / develop DB 接続 0
- staging 変更があれば md5 + row 数報告
- LINE 0 / token 値表示 0 / env 全体表示 0
- launchctl 0 / plist 0 / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist mtime/size/next run 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
production-develop 接続 / token 値表示 / env 全体表示 /
J-Quants 以外 API / LINE / launchctl / plist / cron / 実発注 / 楽天 /
iSPEED / Computer Use / workflow / --no-verify / git push / sudo /
rm -rf / TODO Excel
