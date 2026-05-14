---
id: FIRE-CODEX-R1-WAVE61-PRE-plan
phase: 本番 v0 中核 / Wave 61-pre / price-return-paper_pnl 設計
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
implementation_wave: Wave 61-impl
---

# Wave 61-pre Plan — Price / Return / Paper PnL Linkage Design v1.0

## §1 目的

D10 候補のその後の値動き、paper PnL、manual review、pattern outcome を
つなぐ設計を作る。FIRE を「候補を出す」から「**候補が利益につながったか
検証して改善する**」段階へ進める。本 wave は **設計 only** (= DB write 0
/ API 0 / token 0)。実装は Wave 61-impl。

## §2 安全境界

- DB write: **禁止**
- production / develop DB 接続: **禁止**
- staging DB: **read-only のみ** (= 本 wave で staging touch なし)
- LINE / token / API / launchctl / plist / cron / VACUUM / workflow /
  --no-verify / git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED /
  Computer Use / TODO Excel: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist + 3 DB md5) + 既存 F286-PNL-R3 構造確認 |
| 2 | D10 候補追跡対象定義 (= 重点 3 + 拡張 6 = 9 銘柄) |
| 3 | return horizon 定義 (= h0/h1/h5/h20) |
| 4 | price source 設計 (= staging read-only) |
| 5 | paper PnL ledger JSON schema 設計 |
| 6 | manual review 接続設計 |
| 7 | pattern outcome 設計 |
| 8 | promote/suppress/watch 初期基準設計 |
| 9 | optional helper 設計 (= 案 A 推奨: 新規 runner) |
| 10 | tests 設計 (= 10+ tests 案) |
| 11 | Codex 4 lane factual-confirm |
| 12 | vault plan + results + 03_design doc + HQ 1-block + 6 KPI |

## §4 完了条件

- price/return/paper_pnl linkage design 完成 (= 03_design doc)
- D10 候補追跡対象定義完了 (= 9 銘柄)
- return horizon 定義完了 (= h0/h1/h5/h20)
- paper PnL ledger schema 完成 (= JSON schema)
- manual review 接続設計完成 (= review.md → ledger 紐付)
- pattern outcome 設計完成 (= 9 種 pattern + outcome aggregation)
- promote/suppress/watch 初期基準完成 (= D10 単発で promote しない)
- next implementation wave 案完成 (= Wave 61-impl、案 A 新規 runner)
- Codex 4 lane factual-confirm audit 実施
- DB write 0 / production-develop 接続 0 / staging write 0
- LINE 0 / token 0 / API 0
- launchctl 0 / plist 0 / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist + 3 DB mtime/size 不変
- vault に plan + results + 03_design doc
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env /
API / LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED /
Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel

## §6 Codex 4 lane (factual-confirm、80 words/lane)

- A: D10 候補 / artifact / review source 確認
- B: price-return horizon / market_prices_daily read-only 確認
- C: paper PnL schema / outcome 設計確認 (= 既存 F286-PNL-R3 helpers 流用)
- D: pattern promote/suppress/watch 基準確認 (= D10 単発で promote しない)
