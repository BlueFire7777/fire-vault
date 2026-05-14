---
id: FIRE-CODEX-R1-WAVE60-PILOT-W2-plan
phase: 本番 v0 中核 / Wave 60-pilot-W2 / D9-D13 集約 + D14 entry criteria 確定
priority: 最高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-01
design_doc: ~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
---

# Wave 60-pilot-W2 Plan — Manual Live Pilot W2 Aggregation / D14 Entry Criteria Finalization v1.0

## §1 目的

D9-D13 で **GO_CONDITIONAL が 5 連続維持**。本 W2 集約 wave は:
1. D9-D13 の trade plan / review / paper PnL を read-only で集約
2. 340A0/3798/137A0 の継続性を評価
3. review missing / price gap / DATA-R3 freshness / sector 集中の扱いを決定
4. **D14 で初回少額実弾 GO 条件を確定**
5. D14 trade plan へ引き継ぐ候補・条件・確認項目を確定

## §2 安全境界

- DB write: **禁止**
- production / develop DB 接続: **禁止**
- staging DB: **read-only のみ**
- LINE / token / API / launchctl / plist / cron / VACUUM / workflow /
  --no-verify / git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED /
  Computer Use / TODO Excel: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D9-D13 全 artifact 集約 |
| 2 | D9-D13 summary 表 (= pilot_judgment / top / review / paper PnL) |
| 3 | candidate overlap 分析 (= 340A0 5 連続) |
| 4 | review missing 整理 (= 5 連続 blank) |
| 5 | paper PnL status 整理 (= D10-D13 全 pending) |
| 6 | price freshness / actual confirmation 条件 |
| 7 | DATA-R3 freshness MISSING 扱い |
| 8 | sector concentration 扱い |
| 9 | D14 GO/HOLD/NO-GO criteria 確定 |
| 10 | D14 handoff (= 候補 + 確認項目 + paper PnL handoff) |
| 11 | Codex 4 lane factual-confirm |
| 12 | vault plan + results + 03_design + HQ 1-block + 6 KPI |

## §4 完了条件

- D9-D13 集約完了 (= 5 day pilot_judgment + 4 day paper PnL + 5 day review)
- candidate overlap 分析完了 (= 340A0 5 連続、3798/137A0 5 連続)
- review missing 整理完了 (= 5 連続 blank)
- paper PnL status 整理完了 (= 全 pending、設計通り)
- price freshness / actual confirmation 条件整理完了
- DATA-R3 freshness MISSING 扱い整理完了
- D14 GO/HOLD/NO-GO criteria 確定 (= 4 段階明文化)
- D14 handoff 完成 (= 候補 + 確認項目 + paper PnL 手順)
- Codex 4 lane factual-confirm 実施
- DB write 0 / LINE 0 / token 0 / API 0
- launchctl 0 / plist 0 / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist + 3 DB mtime/size 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env /
API / LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED /
Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel

## §6 Codex 4 lane (factual-confirm、80 words/lane)

- A: D9-D13 candidate overlap (= 340A0/3798/137A0 5 連続)
- B: review missing / paper PnL status (= 5 review blank、4 paper PnL pending)
- C: price freshness / DATA-R3 freshness (= 5/14 cap、DATA-R3 接続 OK / verdict MISSING)
- D: D14 GO/HOLD/NO-GO criteria 確認
