---
template: manual-live-pilot-review
version: 1.0
date: 2026-06-03
owner: BlueFire7777 (Fujiwara)
pilot_day: D15
status: blank
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: HOLD
hold_trigger: review_missing_6_consecutive + same_candidate_7_consecutive
d9_d14_review_missing_count: 6
d9_d15_overlap: 100%
related:
  - 04_daily/2026-06-03_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D15_results.md
---

# Manual Live Pilot — Trade Review (2026-06-03 / D15)

## D15 特記: **HOLD day** (= review 記入 path 開放)

- D9-D14 6 連続 review missing → 設計 doc §3.3 HOLD #1 発動
- 340A0 D9-D15 7 営業日連続 → 設計 doc §3.3 HOLD #2 同時発動
- D15 では entry skip 推奨、review 記入 path で HOLD 解除

---

## §1 [必須] 基本情報

- date: 2026-06-03 (水 / D15)
- **記入時刻**: __:__ JST ★ 必須
- trade_plan: `04_daily/2026-06-03_manual_live_pilot_trade_plan.md`
- **対応**: ____ ★ 必須記入 (例: "HOLD 対応、entry skip")

## §2 [必須] 計画 vs 実際

| 項目 | planned | actual |
|---|---|---|
| pilot_judgment | HOLD | __________ |
| **entry 銘柄** ★ | (none = HOLD) | __________ |
| **entry 価格** ★ | (none) | ____ |
| **entry 時刻** ★ | (none) | __:__ |
| **entry 株数** ★ | 0 (= HOLD) | ___ |
| exit 価格 | (none) | ____ |
| exit 時刻 | __:__ | __:__ |
| 保有期間 | 0 分 (= HOLD) | __ 分 |

## §3 [必須] PnL

- **総 PnL** ★: ______ 円 (= "0 円 (HOLD)" 想定、entry した場合は実 PnL)
- 累積 (D1-D15): ______ 円

## §4 [必須] D14 review 記入記録 (= HOLD 解除 path)

D14 review.md (= `04_daily/2026-06-02_manual_live_pilot_review.md`) を記入完了した?:

- ☐ Yes (= D14 review 必須 5 項目記入済 → HOLD 解除可)
- ☐ No (= D15 も skip + D16 で再判定)

D14 review 必須 5 項目記入内容 (= 簡易 summary):

1. §1 記入時刻 + entry 銘柄: __________
2. §2 entry 価格 / 株数 / 時刻: __________
3. §3 総 PnL: __________
4. §5 Reason for entry / skip: __________
5. §6 final decision: __________

## §5 [必須] Reason for entry / skip

★ 必須記入:

- HOLD 発動の自覚: __________ (例: "review missing 6 連続 + 340A0 7 連続を確認")
- D15 推奨アクション: __________ (例: "D14 review 記入後 D16 再判定")
- 自分の判断: __________

skip 標準: "D15=HOLD、entry skip、D14 review 記入待ち"

## §6 [必須] final decision

★ 1 つ選択:

- ☐ **skip (= HOLD 発動、推奨)** ★
- ☐ enter 340A0 (= HOLD override、Fujiwara 責任)
- ☐ watch (= 寄付き後再判断)

## §7 actual price check result (= D15 では参考)

| code | latest_available (5/14) | actual (6/3) | 乖離率 |
|---|---|---|---|
| 340A0 | 380 | ____ | __.__% |
| 3798 | 505 | ____ | __.__% |
| 137A0 | 739 | ____ | __.__% |

## §8 D15 = HOLD day の運用記録

- HOLD 発動条件確認: ☐ Yes (= review missing 6 + 同候補 7 連続)
- D14 review 記入着手: ☐ Yes / No (= 必須 5 項目)
- D16 plan 着手予定: ☐ Yes / No
- recently_seen_codes 拡張検討: ☐ Yes / No (例: 340A0 追加)

## §9 What worked / failed

- What worked (D15 = HOLD day): __________
- What failed: __________
- 6 連続 review missing の自己反省: __________

## §10 Improvement

- D16 で変えること: __________ (例: "D14 review 必ず記入後 chain 再起")
- recently_seen_codes 拡張案: __________ (例: 340A0/3798/137A0 追加で多様化)
- 全銘柄 daily refresh 自動化への期待: __________

## §11 Pattern promote/suppress/watch 提案

- 340A0 (= 7 連続選出) → promote / suppress / watch: __________
- 3798 (= 7 連続) → promote / suppress / watch: __________
- 137A0 (= 7 連続) → promote / suppress / watch: __________

## §12 Screenshot / 関連 path memo

- D15 F111: `/tmp/fire_d15_prep/d15_f111_real_batch.json`
- D15 morning_line_material: `/tmp/fire_d15_prep/after_r1/morning_line_material_2026-06-03.json`
- D15 paper PnL ledger: `/tmp/fire_d15_prep/d15_paper_pnl_ledger.json`
- D14 paper PnL (review 付き): `/tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review.json`
- W2 設計 doc: `~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md`

## §13 Next action

- ☐ **D14 review 必須 5 項目記入** (= HOLD 解除 path 主推奨)
- ☐ D16 (= 2026-06-04 木) で recently_seen 拡張 + 判定再起
- ☐ DATA-R3 active job wave (= verdict=OK 目標)
- ☐ liquidity filter 強化 wave
- ☐ W60-pilot-W3 集約 (= D14-D18 後)

## §14 Stage 3 突合

| 項目 | 当日 | D1-D15 累積 |
|---|---|---|
| trade 回数 | 0 (= HOLD) | __ / 50 |
| 累積 PnL | 0 円 | __ 円 |
| ルール遵守率 | (= HOLD 遵守) | __% |
| f111_real_batch 比率 | 1/1 | 9/15 |
| top_candidates ≥1 達成率 | 1/1 | 10/15 |
| **HOLD 発動 day** | ✓ | 1 day (= 初) |
| 340A0 連続選出 | 7 連続 | (D9-D15) |
| review missing 連続 | 6 連続 (D9-D14) | (D15 は HOLD 対応で本 review 記入推奨) |

## §15 安全 footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D15 = HOLD / review 記入後 HOLD 解除 path 開放
