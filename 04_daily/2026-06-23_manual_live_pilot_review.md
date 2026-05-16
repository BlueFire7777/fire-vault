---
template: manual-live-pilot-review
version: 1.0
date: 2026-06-23
owner: BlueFire7777 (Fujiwara)
pilot_day: D29
status: blank
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: GO_CONDITIONAL
post_recovery: 10 連続 maintained
recurrence_warning: "7991 rank 1 5 連続 = warning 段階 1 発動"
demote_sim_result: "新 top 1 = 9130 (運輸)、sector 2 種一時縮退"
d30_demote_recommendation: "★★★ D30 で 7991 demote 本実行強推奨"
related:
  - 04_daily/2026-06-23_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D29_results.md
---

# Manual Live Pilot — Trade Review (2026-06-23 / D29)

## §1 [必須] 基本情報

- date: 2026-06-23 (火 / D29)
- 記入時刻: __:__ JST ★
- ticker / name: ____ / __________ ★

## §2 [必須] 計画 vs 実際

| 項目 | planned | actual |
|---|---|---|
| entry 銘柄 | 7991 (機械、warning) or 9130 (運輸) or 331A0 | __________ |
| entry 価格 | 1,177 / 1,410 / 482 | ____ |
| entry 時刻 | 09:00 | __:__ |
| entry 株数 | 100 | ___ |

## §3 [必須] PnL

- 総 PnL ★: ______ 円
- 累積 (D1-D29): ______ 円

## §4 [必須] actual price + liquidity + event

| code | actual (6/23) | 乖離率 | liquidity | event |
|---|---|---|---|---|
| 7991 (機械) | ____ | __.__% | ☐ OK/NG | ☐ OK/NG |
| 9130 (運輸) | ____ | __.__% | ☐ OK/NG | ☐ OK/NG |
| 331A0 (情報通信) | ____ | __.__% | ☐ OK/NG | ☐ OK/NG |

## §5 [必須] Reason

★: __________

## §6 [必須] final decision

- ☐ **enter 7991 (= 機械、5 連続 warning #1、D30 demote 直前)** ★
- ☐ enter 9130 (= 運輸、5 連続 #2、D30 sim 新 #1 候補)
- ☐ enter 331A0 (= 情報通信、5 連続 #3)
- ☐ watch / skip

## §7 7991 recurrence warning + demote sim 評価

- 7991 rank 1 5 連続認識: ☐ Yes
- D30 で 6 連続 → HOLD #2 直前リスク認識: ☐ Yes
- demote sim (= 新 top 1 = 9130 運輸) 確認: ☐ Yes
- **D30 で 7991 demote 本実行意思**: ☐ Yes (= 案 B 推奨) / ☐ No
- sector 2 種一時縮退の容認: ☐ Yes / ☐ No

## §8 demote 後の新 top 評価 (= 参考)

- 9130 (運輸、新 #1 候補): __________
- 331A0 (情報通信、新 #2 候補): __________
- 4389 (情報通信、新 #3 候補): __________
- sector 一時 2 種化への所感: __________

## §9 W5 集約 (= D20-D33 14 day) 引き継ぎ memo

- 7991 warning 到達の所感: __________
- 9130 rank 1 昇格時の運用方針: __________
- sector 多様化縮退期間の見通し: __________

## §10 Next action

- ☐ paper PnL handoff (= 場後、`--review-md` 付きで再 run)
- ☐ D30 (= 2026-06-24 水) で **7991 demote 本実行 wave**
- ☐ D31 (= 2026-06-25 木) post-7991-demote 1 連続確認
- ☐ W5 集約 wave (= D20-D33 後)

## §11 safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
