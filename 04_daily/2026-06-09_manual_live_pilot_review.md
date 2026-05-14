---
template: manual-live-pilot-review
version: 1.0
date: 2026-06-09
owner: BlueFire7777 (Fujiwara)
pilot_day: D19
status: blank
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: HOLD_maintained
hold_status:
  "#1_review_missing": "未解消 (= 5 連続)"
  "#2_same_candidate": "解消継続 (= 340A0 demote 4 連続)、ただし 3798 4 連続懸念"
related:
  - 04_daily/2026-06-09_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D19_results.md
---

# Manual Live Pilot — Trade Review (2026-06-09 / D19)

## D19 特記: **HOLD maintained 5 連続 + 3798 4 連続懸念 day**

- D14 review 0/6 → HOLD #1 残存 5 連続
- 340A0 demote 4 連続維持 + 3798 AFTER-R1 #1 4 連続
- D22 で 3798 7 連続到達 → HOLD #2 再発動懸念
- W3 集約 wave で 3798 追加 demote 検討推奨

---

## §1 [必須] 基本情報

- date: 2026-06-09 (火 / D19)
- **記入時刻**: __:__ JST ★ 必須
- **対応**: ____ ★ 必須

## §2 [必須] 計画 vs 実際

| 項目 | planned | actual |
|---|---|---|
| pilot_judgment | HOLD maintained | __________ |
| **entry 銘柄** ★ | (none = HOLD) or 3798 | __________ |
| **entry 価格** ★ | (none) or 505 | ____ |
| **entry 時刻** ★ | (none) or 09:00 | __:__ |
| **entry 株数** ★ | 0 or 100 | ___ |

## §3 [必須] PnL

- **総 PnL** ★: ______ 円 (= "0 円 (HOLD)" or 実 PnL)
- 累積 (D1-D19): ______ 円

## §4 [必須] D14 review 記入記録 (= 5 連続呼びかけ)

D14 review.md 必須 5 項目記入完了?:

- ☐ Yes (= 記入完了 → D20 で HOLD #1 解消可)
- ☐ No (= D20 でも HOLD #1 継続)

## §5 [必須] Reason

★ 必須記入:

- HOLD maintained 5 連続認識: ____
- 340A0 demote 4 連続評価: ____
- 3798 AFTER-R1 #1 4 連続 + 7 連続懸念認識: ____
- W3 集約での 3798 demote 検討: ____
- D14 review 記入予定 / 不可理由: ____

## §6 [必須] final decision

★ 1 つ選択:

- ☐ **skip (= HOLD maintained、5 連続推奨)** ★
- ☐ enter 3798 ＵＬＳ (= 4 連続 #1、HOLD override、Fujiwara 責任)
- ☐ enter 7991 マミヤ (= sector 多様化、HOLD override)
- ☐ watch

## §7 actual price check result (= 参考)

| code | latest_available (5/14) | actual (6/9) | 乖離率 |
|---|---|---|---|
| 3798 (= #1) | 505 | ____ | __.__% |
| 137A0 | 739 | ____ | __.__% |
| 331A0 | 482 | ____ | __.__% |
| 7991 (= 多様化) | 1,177 | ____ | __.__% |

## §8 340A0 demote 4 連続 + 3798 連続懸念 評価

- 340A0 demote 4 連続維持の実感: __________
- 3798 4 連続 + 7 連続懸念認識: __________
- W3 集約での recently_seen 拡張案 (= 3798/137A0/331A0): __________

## §9 Rule followed?

| ルール | 遵守? |
|---|---|
| HOLD 維持 (= entry skip 推奨) | ☐ Yes / No |
| 340A0 demote 認識 | ☐ Yes |
| D14 review 記入推奨認識 (= 5 連続呼びかけ) | ☐ Yes |
| 3798 7 連続懸念認識 | ☐ Yes |
| 自動発注なし | ☐ Yes (構造的) |
| 楽天 / iSPEED 手動 | ☐ Yes / No / skip |

## §10 Pattern matched

- 該当パターン (= 3798): __________
- D19 = HOLD maintained 5 連続 + 3798 連続懸念 day としての評価: __________

## §11 What worked / failed

- What worked: __________
- What failed: __________

## §12 Improvement

- D20 (= 6/10 水) で変えること: __________
- W3 集約 wave 着手予定 (= D14-D19 6 営業日 review): __________

## §13 Pattern promote/suppress/watch 提案

- 3798 (= 新 #1 4 連続、7 連続懸念) → promote / suppress / watch / **recently_seen 追加候補**: __________
- 137A0 (= 新 #2 4 連続) → promote / suppress / watch: __________
- 331A0 (= 新 entry 4 連続) → promote / suppress / watch: __________

## §14 Screenshot / 関連 path memo

- D19 F111: `/tmp/fire_d19_prep/d19_f111_real_batch.json`
- D19 F062: `/tmp/fire_d19_prep/d19_f062_preview.json`
- D19 morning_line_material: `/tmp/fire_d19_prep/after_r1/morning_line_material_2026-06-09.json`
- D19 paper PnL ledger: `/tmp/fire_d19_prep/d19_paper_pnl_ledger.json`

## §15 Next action

- ☐ **D14 review 必須 5 項目記入** ★ HOLD 完全解除最優先 (5 連続呼びかけ)
- ☐ D20 (= 2026-06-10 水) chain 再起 → HOLD 完全解除可能性大
- ☐ **W60-pilot-W3 集約 wave** ★ 推奨 (= D14-D19 6 営業日 review、3798 demote 検討)
- ☐ DATA-R3 active job wave
- ☐ liquidity filter 強化 wave

## §16 Stage 3 突合

| 項目 | 当日 | D1-D19 累積 |
|---|---|---|
| trade 回数 | 0 (= HOLD) | __ / 50 |
| 累積 PnL | 0 円 | __ 円 |
| HOLD maintained 連続 | 5 日 (D15-D19) | 5 days |
| 340A0 demote 維持 | 4 日連続 | 4 days |
| 新 top 3 連続 | 4 日連続 | 4 days |
| 3798 AFTER-R1 #1 連続 | 4 日連続 (= 7 連続まで残 3 営業日) | 4 days |

## §17 安全 footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D19 = HOLD maintained 5 連続 /
  340A0 demote 4 連続 / 3798 #1 4 連続 (7 連続懸念) / W3 集約推奨
