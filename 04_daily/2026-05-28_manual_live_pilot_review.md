---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-28
owner: BlueFire7777 (Fujiwara)
pilot_day: D11
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
d10_review_status: missing
d10_d11_overlap: 100%
related:
  - 04_daily/2026-05-28_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D11_results.md
---

# Manual Live Pilot — Trade Review (2026-05-28 / D11)

## D11 特記: **paper PnL linkage 初運用 day**

- W61-impl paper PnL preview runner 実装後初の D-day
- 340A0 / 3798 / 137A0 が **D9-D11 3 連続選出** = signal 安定性確認
- D10 review = blank、本 review が初の記入機会
- 5/14 refresh → 5/28 = 14 営業日 gap、寄付き actual 必須

---

## §1 基本情報

- date: 2026-05-28 (木 / D11)
- 記入時刻: __:__ JST
- trade_plan: `04_daily/2026-05-28_manual_live_pilot_trade_plan.md`
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- ticker / name (entry 銘柄): ____ / __________
- final_decision: ☐ enter / ☐ watch / ☐ skip

## §2 計画 vs 実際

| 項目 | planned (refreshed 5/14) | actual (5/28) |
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
- 累積 (D1-D11): ______ 円
- 1 日上限到達?: ☐ No / ☐ Yes
- 連敗?: ☐ No / ☐ Yes

## §4 actual price vs latest available close

| code | latest_available (5/14) | actual (5/28) | 乖離率 | 判断 |
|---|---|---|---|---|
| 340A0 | 380 | ____ | __.__% | __________ |
| 3798 | 505 | ____ | __.__% | __________ |
| 137A0 | 739 | ____ | __.__% | __________ |

- 乖離 ±5% 以内 → refresh 効果評価 ___
- 乖離 ±5% 超 → stale caveat 残存度 ___

## §5 Reason for entry / skip

- FIRE why_selected: __________ (例: 340A0 = F111 rank 4、boost_with_caution、3 連続選出、risk=1,900)
- D9/D10 残ポジ: __________
- 自分の追加判断: __________
- preset_tune 効き (= caution 3 demoted + boost_with_caution top 3): __________
- paper PnL preview 初運用の感触 (= pending 中心、24h 後 outcome 評価): __________

skip 標準理由: "actual 5/28 と refresh 5/14 の乖離大 / 流動性不足 / event リスク / 3 連続見送り判断"

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
| **14 営業日 gap caveat 認識** | ☐ Yes | |
| **freshness MISSING 認識** | ☐ Yes | |
| **paper PnL handoff 実行予定** | ☐ Yes (= 場後 + 翌日) | |

## §8 Liquidity / Spread actual (= 寄付き 5 分後)

| 候補 | 板厚 5 本目処 | 寄付き 5 分 出来高 | 寄付き spread |
|---|---|---|---|
| 340A0 | ___ 株 | ___ 株 | __.__% |
| 3798 | ___ 株 | ___ 株 | __.__% |
| 137A0 | ___ 株 | ___ 株 | __.__% |

- D6-D10 と比較した liquidity: __________
- 中小型 (情報通信集中) の板薄印象: __________

## §9 paper PnL handoff 実行記録

### §9.1 D11 当日 (= 場後)

- 実行 cmd:
  ```
  .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-05-28 --evaluation-date 2026-05-28 \
    --f111-real-batch-json /tmp/fire_d11_prep/d11_f111_real_batch.json \
    --review-md ~/fire-vault/04_daily/2026-05-28_manual_live_pilot_review.md \
    --output-json /tmp/fire_d11_prep/d11_paper_pnl_ledger_eod.json
  ```
- 実行時刻: __:__ JST
- 結果: __________

### §9.2 翌営業日 (= D12 = 5/29 金、朝)

- 実行時刻: __:__ JST
- staging max_date 進捗: 5/14 → __ 営業日
- 結果: __________

### §9.3 h20 後 (= 2026-06-26 頃)

- 実行予定: ____
- outcome 期待: __________

## §10 J-Quants refresh 効果 (= 主観、3 連続選出 day としての評価)

- 137A0 連続選出 (= 5/14 close=739 で復活後 D9-D11) の妥当性: __________
- 5729 急落察知 (= -18.5%) は引き続き有効?: __________
- 4389 急騰 (= +12.1%) の追跡 (= entry 候補 #6 だったが): __________
- 全銘柄 daily refresh 自動化への期待: __________

## §11 Rule violation / 反省点

- 寄付き actual price 確認は実施した?: ☐ Yes / No
- 出来高 / spread 確認は実施した?: ☐ Yes / No
- ネガティブニュース確認は実施した?: ☐ Yes / No
- sector 集中 caveat 認識: __________
- 3 連続同候補での疲労感 / 慎重度: __________

## §12 Pattern matched (= paper PnL ledger reference)

- 該当パターン (= 6 種: refreshed_price_ok / manual_review_active / research_boost / risk_within_pilot_limit / multi_reason / low_risk): __________
- 実 entry した場合の pattern score: __________
- D9-D11 3 連続選出 pattern が今後 promote 候補?: __________

## §13 What worked / failed

- What worked: __________
- What failed: __________
- D11 運用フロー評価 (= paper PnL linkage 初運用 + GO_CONDITIONAL): __________

## §14 Improvement

- D12 で変えること: __________
- D12 で続けること: __________
- paper PnL preview の使い勝手評価: __________
- 全銘柄 daily refresh 自動化 wave (= launchd 別 wave) への期待: __________
- DATA-R3 freshness gate JSON 連携 (= MISSING → OK): __________

## §15 Pattern promote/suppress/watch 提案

- 340A0 ジグザグ (= 3 連続) → promote / suppress / watch: __________
- 3798 ＵＬＳ (= 3 連続) → promote / suppress / watch: __________
- 137A0 Cocolive (= 3 連続) → promote / suppress / watch: __________
- 8747/5729/3489 (= D6-D11 6 連続 demoted) → 引き続き suppress: __________

## §16 Emergency log

該当しなければ skip。

```
- trigger: __________
- 時刻: __:__ JST
- 対応: __________
```

## §17 Screenshot / 関連 path memo

- iSPEED SS (= entry 確認): __________
- FIRE artifact:
  - D11 F111: `/tmp/fire_d11_prep/d11_f111_real_batch.json`
  - D11 F062: `/tmp/fire_d11_prep/d11_f062_preview.json`
  - D11 morning_line_material: `/tmp/fire_d11_prep/after_r1/morning_line_material_2026-05-28.json`
  - D11 paper PnL ledger (pre-EOD): `/tmp/fire_d11_prep/d11_paper_pnl_ledger.json`
  - D10 paper PnL ledger (reference): `/tmp/fire_d10_prep/d10_paper_pnl_ledger.json`

## §18 Next action

- ☐ D12 (= 2026-05-29 金) も pilot 継続、recently_seen 再評価
- ☐ liquidity filter 強化 wave 着手
- ☐ paper_pnl preview h1/h20 再 run (= D12 朝、6/26 頃)
- ☐ DATA-R3 freshness gate JSON 連携 wave
- ☐ 全銘柄 daily refresh launchd 自動化 (= 別 HQ 承認)
- ☐ W60-pilot-W2 集約 (= D6-D11 6 営業日 review)

## §19 Stage 3 突合

| 項目 | 当日 | D1-D11 累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| ルール遵守率 | __% | __% |
| f111_real_batch 比率 | 1/1 | 5/11 |
| top_candidates ≥1 達成率 | 1/1 | 6/11 |
| paper PnL preview 初運用 | ✓ | 1 day |
| 340A0 連続選出 | ✓ (D9-D11) | 3 days |

## §20 安全 footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D11 = paper PnL linkage 初運用 / 340A0 3 連続選出
