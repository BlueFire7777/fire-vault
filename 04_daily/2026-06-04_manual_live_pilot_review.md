---
template: manual-live-pilot-review
version: 1.0
date: 2026-06-04
owner: BlueFire7777 (Fujiwara)
pilot_day: D16
status: blank
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: HOLD_maintained
hold_status:
  "#1_review_missing": "未解消"
  "#2_same_candidate": "解消済 (= 340A0 demote)"
d14_review_gap_status: unresolved
demotion_effect: "340A0 → caution、new top = 3798/137A0/331A0"
related:
  - 04_daily/2026-06-04_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D16_results.md
---

# Manual Live Pilot — Trade Review (2026-06-04 / D16)

## D16 特記: **340A0 demote 成功 + HOLD maintained day**

- 340A0 demote 効果実証 = AFTER-R1 top が 3798/137A0/331A0 へ切替
- D14 review 0/6 → HOLD #1 未解消、HOLD #2 解消
- D17 で D14 review 記入後 HOLD 完全解除可能性高

---

## §1 [必須] 基本情報

- date: 2026-06-04 (木 / D16)
- **記入時刻**: __:__ JST ★ 必須
- **対応**: ____ ★ 必須 (例: "HOLD maintained、340A0 demote 確認、D14 review 記入推奨")

## §2 [必須] 計画 vs 実際

| 項目 | planned | actual |
|---|---|---|
| pilot_judgment | HOLD maintained | __________ |
| **entry 銘柄** ★ | (none = HOLD) or 3798 | __________ |
| **entry 価格** ★ | (none) or 505 | ____ |
| **entry 時刻** ★ | (none) or 09:00 | __:__ |
| **entry 株数** ★ | 0 or 100 | ___ |
| exit 価格 | (none) | ____ |
| exit 時刻 | __:__ | __:__ |

## §3 [必須] PnL

- **総 PnL** ★: ______ 円 (= "0 円 (HOLD)" or 実 PnL)
- 累積 (D1-D16): ______ 円

## §4 [必須] D14 review 記入記録

D14 review.md 必須 5 項目記入完了?:

- ☐ Yes (= D14 review 記入済 → D17 で HOLD #1 解消可)
- ☐ No (= D17 でも HOLD #1 残存)

## §5 [必須] Reason

★ 必須記入:

- HOLD #2 解消 (= 340A0 demote): ____
- HOLD #1 (= D14 review) 状況: ____
- 新 top 候補 (= 3798/137A0/331A0) の評価: ____

## §6 [必須] final decision

★ 1 つ選択:

- ☐ **skip (= HOLD maintained、推奨)** ★
- ☐ enter 3798 ＵＬＳグループ (= 新 #1、HOLD override、Fujiwara 責任)
- ☐ enter 7991 マミヤ・オーピー (= sector 多様化、HOLD override)
- ☐ watch (= 寄付き後再判断)

## §7 actual price check result (= 参考)

| code | latest_available (5/14) | actual (6/4) | 乖離率 |
|---|---|---|---|
| 3798 (= 新 #1) | 505 | ____ | __.__% |
| 137A0 | 739 | ____ | __.__% |
| 331A0 | 482 | ____ | __.__% |
| 7991 (= 多様化) | 1,177 | ____ | __.__% |

## §8 340A0 demote 効果評価

- 340A0 が caution に demote されたことの実感: __________
- 新 top 候補 (= 3798/137A0/331A0) の signal 安定性: __________
- 7991 (機械) の多様化候補としての適切性: __________
- HOLD #2 解消の効果評価: __________

## §9 Rule followed?

| ルール | 遵守? |
|---|---|
| HOLD 維持 (= entry skip 推奨) | ☐ Yes / No |
| 340A0 demote 認識 | ☐ Yes |
| D14 review 記入推奨認識 | ☐ Yes |
| 自動発注なし | ☐ Yes (構造的) |
| 楽天 / iSPEED 手動 | ☐ Yes / No / skip |
| 1 トレード ≤ 15,000 円 | ☐ Yes / No / skip |

## §10 Pattern matched

- 該当パターン (= 3798 ＵＬＳ): __________
- D16 = HOLD maintained + 340A0 demote 成功 day としての評価: __________

## §11 What worked / failed

- What worked: __________
- What failed: __________
- D16 運用フロー評価 (= demote 効果 + HOLD maintained): __________

## §12 Improvement

- D17 で変えること: __________ (例: "D14 review 記入後 chain 再起")
- D17 で続けること: __________
- recently_seen_codes さらなる拡張案 (= 3798/137A0 追加?): __________

## §13 Pattern promote/suppress/watch 提案

- 340A0 (= 7 連続 + 本日 demote) → promote / suppress / watch: __________
- 3798 (= 新 #1) → promote / suppress / watch: __________
- 137A0 (= 新 #2) → promote / suppress / watch: __________
- 331A0 (= 新 entry #3) → promote / suppress / watch: __________

## §14 Screenshot / 関連 path memo

- D16 F111: `/tmp/fire_d16_prep/d16_f111_real_batch.json`
- D16 F062: `/tmp/fire_d16_prep/d16_f062_preview.json`
- D16 morning_line_material: `/tmp/fire_d16_prep/after_r1/morning_line_material_2026-06-04.json`
- D16 paper PnL ledger: `/tmp/fire_d16_prep/d16_paper_pnl_ledger.json`
- D14 paper PnL with review: `/tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d16_check.json`

## §15 Next action

- ☐ **D14 review 必須 5 項目記入** (= HOLD 解除主推奨)
- ☐ D17 (= 2026-06-05 金) chain 再起 + 判定見直し → HOLD 完全解除?
- ☐ DATA-R3 active job wave (= verdict=OK 目標)
- ☐ liquidity filter 強化 wave
- ☐ W60-pilot-W3 集約 (= D14-D18 後)

## §16 Stage 3 突合

| 項目 | 当日 | D1-D16 累積 |
|---|---|---|
| trade 回数 | 0 or 1 | __ / 50 |
| 累積 PnL | 0 or 実 PnL | __ 円 |
| 340A0 demote 効果 | ✓ (= 初) | 1 day |
| HOLD #2 解消 | ✓ | 1 day |
| HOLD #1 残存 | ✓ | 7 連続 (D9-D14) |

## §17 安全 footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D16 = HOLD maintained / 340A0 demote 成功 /
  D17 で HOLD 完全解除可能性高
