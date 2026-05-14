---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-26
owner: BlueFire7777 (Fujiwara)
pilot_day: D9
status: blank
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
preset_tune_applied: true
label_threshold_mode: strict
recently_seen_codes: ["8747", "5729", "3489"]
pilot_judgment: GO_CONDITIONAL
related:
  - 04_daily/2026-05-26_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D9_results.md
---

# Manual Live Pilot — Trade Review (2026-05-26 / D9)

## D9 特記: **W60-F111-preset-tune 初実弾検証 day**

- top 1-3 (8747/5729/3489) は recently_seen demote で caution → 見送り
- top 4 boost_with_caution 候補 = 340A0 ジグザグから entry 検討
- ⚠ market_prices_daily 2026-05-08 stale caveat
- ⚠ freshness_verdict=MISSING

---

## §1 基本情報

- date: 2026-05-26 (火 / D9)
- 記入時刻: __:__ JST
- trade_plan: `04_daily/2026-05-26_manual_live_pilot_trade_plan.md`
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- ticker / name: ____ / __________
- final_decision: ☐ enter / ☐ watch / ☐ skip

## §2 計画 vs 実際 (= 340A0 ジグザグ想定、別銘柄 entry なら更新)

| 項目 | planned | actual |
|---|---|---|
| entry 価格 | 398 (stale) | ____ |
| entry 時刻 | 09:00 寄付き | __:__ |
| entry 株数 | 100 株 | ___ |
| stop loss | entry × 0.95 | ____ |
| take profit | entry × 1.02 / h20 | ____ |
| exit 価格 | ____ | ____ |
| exit 時刻 | __:__ | __:__ |
| 保有期間 | __ 分 | __ 分 |

## §3 PnL

- 総 PnL: ______ 円
- 累積 (D1-D9): ______ 円
- 1 日上限到達?: ☐ No / ☐ Yes
- 連敗?: ☐ No / ☐ Yes

## §4 Reason for entry / skip

- FIRE why_selected: 340A0 = F111-real-batch top 4 / boost_with_caution / score=0.892 / risk=1,990 円 / 情報通信
- D6/D7 残ポジ: __________
- 自分の追加判断: __________
- preset_tune の効き具合 (= caution 3 + boost_with_caution 4 以降): __________

skip 標準理由 (例): "actual price stale 乖離 +5% 超 / 流動性不足 / event リスク"

## §5 Reason for exit

- exit trigger: ☐ stop_loss (5%) / ☐ take_profit (2%) / ☐ 15:10 cutoff / ☐ 手動判断 / ☐ skip
- 判断理由: __________

## §6 Rule followed?

| ルール | 遵守? | 備考 |
|---|---|---|
| 事前 stop_loss 設定 | ☐ Yes / No / skip | 5% |
| 損切ずらさず | ☐ Yes / No / skip | |
| ナンピンなし | ☐ Yes / No / skip | |
| 追加買いなし | ☐ Yes / No / skip | |
| 15:10 close | ☐ Yes / No / skip | |
| 持ち越しなし | ☐ Yes / No / skip | |
| 自動発注なし | ☐ Yes (= 構造的) | |
| 楽天 / iSPEED 手動 | ☐ Yes / No / skip | |
| 1 トレード ≤ 15,000 円 | ☐ Yes / No / skip | |
| 1 日 ≤ 30,000 円 | ☐ Yes / No / skip | |
| **top 1-3 demote skip 認識** | ☐ Yes (= 8747/5729/3489 見送り) | |
| **price stale caveat 認識** | ☐ Yes (= 2026-05-08 cap) | |
| **freshness MISSING 認識** | ☐ Yes (= DATA-R3 gate 未通過) | |

## §7 Liquidity / Spread actual (= 寄付き 5 分後)

- 寄付き 板 5 本目処合計: ___ 株
- 寄付き 5 分 出来高: ___ 株
- 寄付き spread: __.__ %
- D6-D8 と比較した liquidity: __________

## §8 ⚠ market_prices_daily stale caveat impact

- stale close = 398 円 (= 2026-05-08)
- D9 actual price = ___ 円
- 乖離率 = __.__ %
- entry 判断に影響したか: __________
- staging features J-Quants refresh の必要性 (= W60-jquants-daily-refresh-staging): __________

## §9 Rule violation / 反省点

- 寄付き actual price 確認は実施した?: ☐ Yes / No
- 出来高 / spread 確認は実施した?: ☐ Yes / No
- ネガティブニュース確認は実施した?: ☐ Yes / No
- preset_tune 改修 (= label demote / max=20) は判断補助になった?: __________

## §10 Pattern matched

- 該当パターン (= F111-real-batch top 4 boost_with_caution の reproducibility): __________
- 実 entry した場合の pattern score: __________

## §11 What worked / failed

- What worked: __________
- What failed: __________
- D9 運用フロー評価 (= preset_tune 初適用 + GO_CONDITIONAL 判定): __________

## §12 Improvement

- D10 で変えること: __________
- D10 で続けること: __________
- W60-jquants-daily-refresh-staging への期待 (= price 更新): __________
- recently_seen_codes 拡張案 (= D6-D9 を D10 で demote): __________

## §13 Pattern promote/suppress/watch 提案

- 340A0 ジグザグ → promote / suppress / watch: __________
- top 1-3 (8747/5729/3489) → 引き続き suppress: __________
- D9 boost_with_caution group 全体 → watch / promote: __________

## §14 Emergency log

該当しなければ skip。

```
- trigger: __________
- 時刻: __:__ JST
- 対応: __________
```

## §15 Screenshot / 関連 path memo

- iSPEED SS (= entry 確認): __________
- FIRE artifact:
  - F111: `/tmp/fire_d9_prep/f111_real_batch_2026-05-26.json`
  - F062: `/tmp/fire_d9_prep/d9_f062_preview.json`
  - morning_line_material: `/tmp/fire_d9_prep/after_r1/morning_line_material_2026-05-26.json`

## §16 Next action

- ☐ D10 (= 2026-05-27 水) も pilot 継続、recently_seen_codes に D9 entry 銘柄追加
- ☐ liquidity filter 強化 wave 着手
- ☐ W60-jquants-daily-refresh-staging 着手 (= price refresh、HQ approve 必要)
- ☐ W60-pilot-W2 集約 (= D6-D10 後)
- ☐ AFTER-R1 DATA-R3 freshness gate JSON 連携 wave

## §17 Stage 3 突合

| 項目 | 当日 | D1-D9 累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| ルール遵守率 | __% | __% |
| f111_real_batch 比率 | 1/1 | 3/9 |
| top_candidates ≥1 達成率 | 1/1 | 4/9 |
| W60-F111-preset-tune 初実弾 | ✓ | 1 day |

## §18 安全 footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D9 = W60-F111-preset-tune 初検証 + GO_CONDITIONAL
