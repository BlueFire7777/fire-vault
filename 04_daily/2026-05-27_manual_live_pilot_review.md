---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-27
owner: BlueFire7777 (Fujiwara)
pilot_day: D10
status: blank
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
preset_tune_applied: true
label_threshold_mode: strict
recently_seen_codes: ["8747", "5729", "3489"]
jquants_refresh_applied: true
market_prices_daily_max_date: 2026-05-14
pilot_judgment: GO_CONDITIONAL
related:
  - 04_daily/2026-05-27_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D10_results.md
---

# Manual Live Pilot — Trade Review (2026-05-27 / D10)

## D10 特記: **refreshed price 初実弾 day**

- W60-jquants-daily-refresh で 12 銘柄 5/14 price 更新済
- 137A0 Cocolive が close=0→739 で entry 候補復活
- 5729 -18.5% 急落 / 4389 +12.1% 急騰を refresh で検知
- ⚠ 5/14 refresh → 5/27 D10 当日 = 13 営業日 gap、朝寄付き actual 必須

---

## §1 基本情報

- date: 2026-05-27 (水 / D10)
- 記入時刻: __:__ JST
- trade_plan: `04_daily/2026-05-27_manual_live_pilot_trade_plan.md`
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- ticker / name (entry 銘柄): ____ / __________
- final_decision: ☐ enter / ☐ watch / ☐ skip

## §2 計画 vs 実際

| 項目 | planned (refreshed 5/14) | actual (5/27) |
|---|---|---|
| 候補 #1 close (340A0) | 380 | ____ |
| 候補 #2 close (3798) | 505 | ____ |
| 候補 #3 close (137A0) | 739 | ____ |
| entry 銘柄 | __________ | __________ |
| entry 価格 | ____ | ____ |
| entry 時刻 | 09:00 | __:__ |
| entry 株数 | 100 | ___ |
| stop loss (5%) | ____ | ____ |
| take profit (2%) | ____ | ____ |
| exit 価格 | ____ | ____ |
| exit 時刻 | __:__ | __:__ |
| 保有期間 | __ 分 | __ 分 |

## §3 PnL

- 総 PnL: ______ 円
- 累積 (D1-D10): ______ 円
- 1 日上限到達?: ☐ No / ☐ Yes
- 連敗?: ☐ No / ☐ Yes

## §4 actual price vs refreshed price

| code | refreshed (5/14) | actual (5/27) | 乖離率 | 判断 |
|---|---|---|---|---|
| 340A0 | 380 | ____ | __.__% | __________ |
| 3798 | 505 | ____ | __.__% | __________ |
| 137A0 | 739 | ____ | __.__% | __________ |

- 乖離が±5%以内 → refresh の効果評価 ___
- 乖離が±5%超 → stale caveat 残存度 ___

## §5 Reason for entry / skip

- FIRE why_selected: __________ (例: 340A0 = F111 rank 4、boost_with_caution、refresh 後 risk=1,900)
- D9 残ポジ: __________
- 自分の追加判断: __________
- preset_tune の効き (= caution 3 demoted + boost_with_caution top 3): __________
- refresh の効き (= 137A0 復活、5729 急落判明): __________

skip 標準理由: "actual 5/27 と refresh 5/14 の乖離大 / 流動性不足 / event リスク"

## §6 Reason for exit

- exit trigger: ☐ stop_loss / ☐ take_profit / ☐ 15:10 cutoff / ☐ 手動判断 / ☐ skip
- 判断理由: __________

## §7 Rule followed?

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
| **refreshed price 認識** | ☐ Yes (= 5/14 cap) | |
| **13 営業日 gap caveat 認識** | ☐ Yes | |
| **freshness MISSING 認識** | ☐ Yes | |

## §8 Liquidity / Spread actual (= 寄付き 5 分後)

| 候補 | 板厚 5 本目処 | 寄付き 5 分 出来高 | 寄付き spread |
|---|---|---|---|
| 340A0 | ___ 株 | ___ 株 | __.__% |
| 3798 | ___ 株 | ___ 株 | __.__% |
| 137A0 | ___ 株 | ___ 株 | __.__% |

- D6-D9 と比較した liquidity: __________
- 中小型 (情報通信集中) の板薄印象: __________

## §9 J-Quants refresh 効果 (= 主観)

- 137A0 復活 (= excluded → entry 候補) は判断補助になった?: __________
- 5729 急落察知 (= -18.5%) は caution skip 確信を上げた?: __________
- 4389 急騰 (= +12.1%) は急騰銘柄回避に役立った?: __________
- 価格鮮度 (5/14) で 13 営業日 gap、当日 actual で十分?: __________

## §10 Rule violation / 反省点

- 寄付き actual price 確認は実施した?: ☐ Yes / No
- 出来高 / spread 確認は実施した?: ☐ Yes / No
- ネガティブニュース確認は実施した?: ☐ Yes / No
- sector 集中 caveat 認識 (= 情報通信 100%) は entry 判断に影響した?: __________

## §11 Pattern matched

- 該当パターン (= F111-real-batch top 4 boost_with_caution、refreshed price): __________
- 実 entry した場合の pattern score: __________

## §12 What worked / failed

- What worked: __________
- What failed: __________
- D10 運用フロー評価 (= refresh + preset_tune + GO_CONDITIONAL): __________

## §13 Improvement

- D11 で変えること: __________
- D11 で続けること: __________
- 全銘柄 daily refresh 自動化 (= launchd 別 wave) への期待: __________
- liquidity filter 強化 wave への期待: __________
- DATA-R3 freshness gate JSON 連携 (= freshness_verdict MISSING → OK): __________

## §14 Pattern promote/suppress/watch 提案

- 340A0 ジグザグ → promote / suppress / watch: __________
- 3798 ＵＬＳ → promote / suppress / watch: __________
- 137A0 Cocolive (= refresh 復活) → promote / suppress / watch: __________
- top 1-3 (8747/5729/3489) → 引き続き suppress: __________

## §15 Emergency log

該当しなければ skip。

```
- trigger: __________
- 時刻: __:__ JST
- 対応: __________
```

## §16 Screenshot / 関連 path memo

- iSPEED SS (= entry 確認): __________
- FIRE artifact:
  - F111: `/tmp/fire_d10_prep/d10_f111_real_batch.json`
  - F062: `/tmp/fire_d10_prep/d10_f062_preview.json`
  - morning_line_material: `/tmp/fire_d10_prep/after_r1/morning_line_material_2026-05-27.json`
  - jquants_refresh: `/tmp/fire_d10_prep/jq_refresh.json`

## §17 Next action

- ☐ D11 (= 2026-05-28 木) も pilot 継続、recently_seen に D10 entry 銘柄追加
- ☐ liquidity filter 強化 wave 着手
- ☐ paper_pnl 翌営業日 sim (= F286-PNL-R3 既実装、D9/D10 entry の sim)
- ☐ DATA-R3 freshness gate JSON 連携 wave
- ☐ 全銘柄 daily refresh 自動化 (= launchd 経路、別 HQ 承認)
- ☐ W60-pilot-W2 集約 (= D6-D10 後)

## §18 Stage 3 突合

| 項目 | 当日 | D1-D10 累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| ルール遵守率 | __% | __% |
| f111_real_batch 比率 | 1/1 | 4/10 |
| top_candidates ≥1 達成率 | 1/1 | 5/10 |
| refresh 後初実弾 | ✓ | 1 day (= D10) |
| 137A0 復活 entry 検討 | ✓ (= candidates 化) | 1 day |

## §19 安全 footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D10 = refreshed price 初検証 + 137A0 復活確認
