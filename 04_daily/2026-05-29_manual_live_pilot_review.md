---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-29
owner: BlueFire7777 (Fujiwara)
pilot_day: D12
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
d11_review_status: missing
d9_d12_overlap: 100%
related:
  - 04_daily/2026-05-29_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D12_results.md
---

# Manual Live Pilot — Trade Review (2026-05-29 / D12)

## D12 特記: **review/paper PnL feedback 反映 day**

- D9-D12 で 340A0/3798/137A0 が 4 営業日連続選出 = signal 極めて安定
- D10/D11 review = blank 2 連続 → 本 D12 review が **3 回目の記入機会**
- 5/14 refresh → 5/29 = 15 営業日 gap、必ず朝寄付き actual + 朝再 refresh
- paper PnL preview chain 連続運用 (= D11/D12)

---

## §1 基本情報

- date: 2026-05-29 (金 / D12)
- 記入時刻: __:__ JST
- trade_plan: `04_daily/2026-05-29_manual_live_pilot_trade_plan.md`
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- ticker / name (entry 銘柄): ____ / __________
- final_decision: ☐ enter / ☐ watch / ☐ skip

## §2 計画 vs 実際

| 項目 | planned (refreshed 5/14) | actual (5/29) |
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
- 累積 (D1-D12): ______ 円
- 1 日上限到達?: ☐ No / ☐ Yes
- 連敗?: ☐ No / ☐ Yes

## §4 actual price vs latest available close

| code | latest_available (5/14) | actual (5/29) | 乖離率 | 判断 |
|---|---|---|---|---|
| 340A0 | 380 | ____ | __.__% | __________ |
| 3798 | 505 | ____ | __.__% | __________ |
| 137A0 | 739 | ____ | __.__% | __________ |

- 乖離 ±5% 以内 → entry 検討
- 乖離 ±5% 超 → stale caveat 残存度、skip 推奨度: ___

## §5 Reason for entry / skip

- FIRE why_selected: __________ (例: 340A0 = F111 rank 4、boost_with_caution、4 連続選出、risk=1,900)
- D9-D11 残ポジ: __________
- D10/D11 review 未記入の影響: __________
- 自分の追加判断: __________
- preset_tune 効き (= caution 3 demoted + boost_with_caution top 3): __________
- paper PnL preview chain 連続運用の感触: __________
- 4 連続同候補 → 5 連続前に entry すべきか / D13 持ち越しか: __________

skip 標準理由: "actual 5/29 と refresh 5/14 の乖離大 / 流動性不足 / event / 4 連続見送り判断"

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
| **15 営業日 gap caveat 認識** | ☐ Yes | |
| **freshness MISSING 認識** | ☐ Yes | |
| **4 連続同候補 認識** | ☐ Yes | |
| **paper PnL handoff 実行予定** | ☐ Yes (= 場後 + 翌日) | |

## §8 Liquidity / Spread actual (= 寄付き 5 分後)

| 候補 | 板厚 5 本目処 | 寄付き 5 分 出来高 | 寄付き spread |
|---|---|---|---|
| 340A0 | ___ 株 | ___ 株 | __.__% |
| 3798 | ___ 株 | ___ 株 | __.__% |
| 137A0 | ___ 株 | ___ 株 | __.__% |

- D6-D11 と比較した liquidity: __________
- 中小型 (情報通信集中) の板薄印象: __________
- 金曜引け前の板の薄さ: __________

## §9 paper PnL handoff 実行記録

### §9.1 D12 当日 (= 場後)

- 実行 cmd:
  ```
  .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-05-29 --evaluation-date 2026-05-29 \
    --f111-real-batch-json /tmp/fire_d12_prep/d12_f111_real_batch.json \
    --review-md ~/fire-vault/04_daily/2026-05-29_manual_live_pilot_review.md \
    --output-json /tmp/fire_d12_prep/d12_paper_pnl_ledger_eod.json
  ```
- 実行時刻: __:__ JST
- 結果: __________

### §9.2 翌営業日 (= 2026-06-01 月、朝)

- 実行時刻: __:__ JST
- staging max_date 進捗: 5/14 → __ 営業日
- 結果: __________

### §9.3 h20 後 (= 2026-06-26 頃)

- 実行予定: ____
- outcome 期待: __________

## §10 J-Quants refresh 効果 (= 4 連続選出 day としての評価)

- 137A0 連続選出 (= 5/14 close=739 で復活後 D9-D12) の妥当性: __________
- 5729 急落察知 (= -18.5%) は引き続き有効?: __________
- 4389 急騰 (= +12.1%) の追跡: __________
- 全銘柄 daily refresh 自動化への期待: __________

## §11 Rule violation / 反省点

- 寄付き actual price 確認は実施した?: ☐ Yes / No
- 出来高 / spread 確認は実施した?: ☐ Yes / No
- ネガティブニュース確認は実施した?: ☐ Yes / No
- 朝再 refresh 実施した?: ☐ Yes / No
- 4 連続同候補での判断疲労感: __________

## §12 Pattern matched

- 該当パターン (= 6 種: refreshed_price_ok / manual_review_active / research_boost / risk_within_pilot_limit / multi_reason / low_risk): __________
- 実 entry した場合の pattern score: __________
- D9-D12 4 連続選出 pattern が promote 候補?: __________

## §13 What worked / failed

- What worked: __________
- What failed: __________
- D12 運用フロー評価 (= 連続 paper PnL chain + GO_CONDITIONAL): __________

## §14 Improvement

- D13 で変えること: __________
- D13 で続けること: __________
- paper PnL preview 使い勝手評価 (= 2 day 連続運用後): __________
- 全銘柄 daily refresh 自動化 wave (= launchd) への期待: __________
- DATA-R3 freshness gate JSON 連携 (= MISSING → OK): __________

## §15 Pattern promote/suppress/watch 提案

- 340A0 ジグザグ (= 4 連続) → promote / suppress / watch: __________
- 3798 ＵＬＳ (= 4 連続) → promote / suppress / watch: __________
- 137A0 Cocolive (= 4 連続) → promote / suppress / watch: __________
- 8747/5729/3489 (= 7 連続 demoted) → 引き続き suppress: __________

## §16 Emergency log

該当しなければ skip。

```
- trigger: __________
- 時刻: __:__ JST
- 対応: __________
```

## §17 Screenshot / 関連 path memo

- iSPEED SS: __________
- FIRE artifact:
  - D12 F111: `/tmp/fire_d12_prep/d12_f111_real_batch.json`
  - D12 F062: `/tmp/fire_d12_prep/d12_f062_preview.json`
  - D12 morning_line_material: `/tmp/fire_d12_prep/after_r1/morning_line_material_2026-05-29.json`
  - D12 paper PnL ledger (pre-EOD): `/tmp/fire_d12_prep/d12_paper_pnl_ledger.json`
  - D11 paper PnL ledger (reference): `/tmp/fire_d11_prep/d11_paper_pnl_ledger.json`
  - D10 paper PnL ledger (reference): `/tmp/fire_d10_prep/d10_paper_pnl_ledger.json`

## §18 Next action

- ☐ D13 (= 2026-06-01 月) も pilot 継続、recently_seen 再評価
- ☐ liquidity filter 強化 wave 着手
- ☐ paper_pnl preview h1/h20 再 run (= D13 朝、6/26 頃)
- ☐ DATA-R3 freshness gate JSON 連携 wave
- ☐ 全銘柄 daily refresh launchd 自動化
- ☐ W60-pilot-W2 集約 (= D6-D12 7 営業日 review、KPI 算出)

## §19 Stage 3 突合

| 項目 | 当日 | D1-D12 累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| ルール遵守率 | __% | __% |
| f111_real_batch 比率 | 1/1 | 6/12 |
| top_candidates ≥1 達成率 | 1/1 | 7/12 |
| paper PnL chain 連続運用 | ✓ | 2 days (D11+D12) |
| 340A0 連続選出 | ✓ (D9-D12) | 4 days |

## §20 安全 footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D12 = review/paper PnL feedback 反映 / 4 連続 340A0
