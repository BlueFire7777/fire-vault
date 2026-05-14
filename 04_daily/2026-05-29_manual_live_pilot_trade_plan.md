---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-29
owner: BlueFire7777 (Fujiwara)
pilot_day: D12
status: plan
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
preset_tune_applied: true
label_threshold_mode: strict
recently_seen_codes: ["8747", "5729", "3489"]
max_candidates: 20
jquants_refresh_applied: true
market_prices_daily_max_date: 2026-05-14
pilot_judgment: GO_CONDITIONAL
d10_review_status: missing
d11_review_status: missing
d9_d12_overlap: 100%
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D12_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D12_results.md
  - 04_daily/2026-05-29_manual_live_pilot_review.md
  - 04_daily/2026-05-27_manual_live_pilot_trade_plan.md  # D10
  - 04_daily/2026-05-28_manual_live_pilot_trade_plan.md  # D11
artifacts:
  f111: /tmp/fire_d12_prep/d12_f111_real_batch.json
  f062: /tmp/fire_d12_prep/d12_f062_preview.json
  morning_line_material: /tmp/fire_d12_prep/after_r1/morning_line_material_2026-05-29.json
  paper_pnl_ledger: /tmp/fire_d12_prep/d12_paper_pnl_ledger.json
---

# Manual Live Pilot — Trade Plan (2026-05-29 / D12)

## D12 特記: **review/paper PnL feedback 反映 day**

- 340A0/3798/137A0 が **D9-D12 4 営業日連続選出** = signal 極めて安定
- D10 review = **status: blank** / D11 review = **status: blank** (= 2 連続未記入)
- D10/D11 paper PnL ledger = 全 pending (= base 5/14、未来 horizon 無し)
- features cap = 2026-05-14 で **15 営業日 gap** (= D12 = 5/29)
- 中核判断: review pending continued + 同候補 4 連続 → **朝寄付き actual + iSPEED 確認 後 GO**

---

## §1 基本情報

- date: **2026-05-29 (金 / D12)**
- 記入時刻: __:__ JST
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- J-Quants refresh status: ✓ (= W60-jquants で 5/14 まで catch-up)
- market_prices_daily max_date: **2026-05-14**
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489 --max-candidates 20`
- demoted_count: 3

## §2 D10/D11 review recap (= 2 連続 review_missing)

| 項目 | D10 (5/27) | D11 (5/28) |
|---|---|---|
| review status | **blank** | **blank** |
| entry/exit/PnL | 全 None | 全 None |
| final_decision | unknown | unknown |
| pilot_judgment | GO_CONDITIONAL | GO_CONDITIONAL |
| 影響 | Fujiwara entry 不明 | 同じ |

→ 2 連続 review_missing は明確な caveat。D12 で Fujiwara 確認 + 記入が
   running pilot の連続性確保に必須。

## §3 paper PnL recap (= D10/D11 ledger)

| 項目 | D10 ledger | D11 ledger |
|---|---|---|
| path | /tmp/fire_d10_prep/d10_paper_pnl_ledger.json | /tmp/fire_d11_prep/d11_paper_pnl_ledger.json |
| candidate_count | 20 | 20 |
| evaluated_count | 0 (= 全 pending) | 0 (= 全 pending) |
| review_missing_count | 1 | 1 |
| pattern_outcomes | 9 種全 watch / evidence=low | 同 |

→ paper PnL は全 pending (= staging max=5/14、未来 horizon 無し)。
   真の outcome は h20 後 (= 2026-06-26 頃) に再 run で評価。

## §4 ⚠ price freshness caveat (= 強化版)

- staging max_date = **2026-05-14**
- D12 当日 = **2026-05-29** → **15 営業日 gap** (= D11 14 営業日より 1 日進歩)
- 必須対応:
  1. **朝 (= 08:30 JST 想定) 12 銘柄再 refresh** (= 5/14 → 5/29)
  2. **寄付き直前 (= 08:55 JST) iSPEED で actual price 確認**
- 朝 refresh cmd:
  ```
  set -a; source /Users/bluefire/fire/.env; set +a
  FIRE_ENV=staging \
  HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 \
  HQ_APPROVE_JQUANTS_TOKEN_READ=1 \
  .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
    --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
    --datasets prices --max-days 14 \
    --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
    --sleep-seconds 0.3 --write \
    --output-json /tmp/d12_jq_refresh.json
  ```

## §5 D9-D12 candidate overlap (= 4 営業日連続)

### §5.1 top 4 = entry 候補 #1 連続性

| day | rank 4 | close | risk_yen | label |
|---|---|---|---|---|
| D9 (5/26) | 340A0 ジグザグ | **398 (stale)** | 1,990 | boost_with_caution |
| D10 (5/27) | 340A0 ジグザグ | **380 (refreshed)** | 1,900 | boost_with_caution |
| D11 (5/28) | 340A0 ジグザグ | 380 | 1,900 | boost_with_caution |
| **D12 (5/29)** | **340A0 ジグザグ** | **380** | **1,900** | **boost_with_caution** |

→ **4 営業日連続 340A0** (= signal 極めて安定、ただし features cap 律速で
  内容変化なし)

### §5.2 D11 vs D12 完全同一

D11 candidates と D12 candidates の top 12 は **100% 完全同一** (= 全 ticker
順位 / close / risk_yen 同じ)。これは:
- staging market_prices_daily max = 5/14 (= D11/D12 共通)
- features rerun なし
- signal rerun なし

→ 真の意味の signal 変化には:
1. **全銘柄 daily refresh** (= 別 wave、launchd 自動化)
2. **persist_derived_indicators rerun** (= 別 wave、HQ_APPROVE_STAGING_FEATURES_RERUN)
3. **research_watchlist_signal rerun**

## §6 top 3 entry 候補 (= D11 と完全同一)

| rank | code | name | sector | close (5/14) | score | label | risk_yen | 推奨 |
|---|---|---|---|---|---|---|---|---|
| **1** | **340A0** | **ジグザグ** | 情報通信 | **380** | 0.892 | boost_with_caution | **1,900** | **★ #1** |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 0.877 | boost_with_caution | 2,525 | #2 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 0.869 | boost_with_caution | 3,695 | #3 |

⚠ **sector 100% 集中** → 多様化 #4 = 7991 マミヤ・オーピー (機械、close=1,177、risk=5,885) 併記推奨

## §7 entry candidate #1: 340A0 ジグザグ (= 4 連続選出)

| 項目 | 値 |
|---|---|
| code / name | 340A0 / ジグザグ |
| sector | 情報通信・サービスその他 |
| close (= 2026-05-14 refreshed) | 380 円 |
| score | 0.892 |
| label | boost_with_caution |
| AFTER-R1 morning score | 175.22 (= rank 1) |
| 100 株 entry 想定資金 | 38,000 円 |
| stop_loss (5%) | 361 円 = max loss **1,900 円** |
| take_profit (2%) | 388 円 = target gain 800 円 |
| pattern_tags (paper ledger) | refreshed_price_ok / manual_review_active / research_boost / risk_within_pilot_limit / multi_reason / low_risk (6 種) |
| 4 連続選出 caveat | signal 安定、ただし features cap 律速、真の更新には refresh + features rerun |

## §8 actual price confirmation 欄 (= 寄付き直前 08:55 JST)

| 候補 | refreshed (5/14) | actual (5/29) | 乖離率 | 採否 |
|---|---|---|---|---|
| 340A0 ジグザグ | 380 | ____ | __.__% | ☐ entry / ☐ skip |
| 3798 ＵＬＳ | 505 | ____ | __.__% | ☐ entry / ☐ skip |
| 137A0 Cocolive | 739 | ____ | __.__% | ☐ entry / ☐ skip |

乖離 ±5% 以内 → entry 検討、±5% 超 → skip 推奨 (= 15 営業日 gap)。

## §9 liquidity manual check (= 寄付き 5 分後)

| 候補 | 板厚 5 本目処 | 寄付き 5 分 出来高 | 寄付き spread | 採否 |
|---|---|---|---|---|
| 340A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 3798 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 137A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |

採否基準: 板厚 ≥ 5,000 株 / 出来高 ≥ 5,000 株 / spread ≤ 0.5%

## §10 event / earnings check (= 寄付き前)

| 候補 | TDnet 開示 | 決算予定 | ネガティブニュース |
|---|---|---|---|
| 340A0 | __________ | __________ | __________ |
| 3798 | __________ | __________ | __________ |
| 137A0 | __________ | __________ | __________ |

## §11 final decision (= 寄付き直前 08:55 JST 記入)

- ☐ **enter 340A0** 100 株 (= entry candidate #1、4 連続選出)
- ☐ **enter 3798** 100 株
- ☐ **enter 137A0** 100 株
- ☐ **enter 7991 マミヤ・オーピー** 100 株 (= sector 多様化、機械)
- ☐ **watch (= 寄付き後再判断)**
- ☐ **skip (= 全 NG、D13 へ持ち越し)**

決定理由: __________________________

## §12 risk limit check

| 項目 | 値 | 制約 | OK? |
|---|---|---|---|
| 1 トレード max loss | 1,900〜3,695 円 | ≤ 15,000 | ✓ |
| 1 日 max loss | 1 トレード分 | ≤ 30,000 | ✓ |
| 同一 sector 連続 entry | 1 件 | ≤ 2 件 | ✓ |
| stop_loss 事前設定 | 5% | 必須 | ✓ |
| ナンピンなし | 想定 | 禁止 | ✓ |
| 持ち越しなし | 15:10 完全クローズ | 必須 | ✓ |
| D6-D8 caution skip | 8747/5729/3489 | 維持 | ✓ |

## §13 9 hard invariants

| invariant | D12 result |
|---|---|
| artifact_source | f062_preview ✓ |
| f062_raw_kind | f062_actual_dict ✓ |
| f111_input_source | f111_real_batch ✓ |
| freshness_verdict | **MISSING** ⚠ (= 本 wave 意図的) |
| forbidden_phrases_check.passed | True ✓ |
| auto_order_allowed | False ✓ |
| manual_review_required | True ✓ |
| safety_flags 13 keys all False | True ✓ |
| top_candidates ≥ 1 | 3 ✓ |

## §14 Pilot 判定: **GO_CONDITIONAL** (= D11 と同維持)

**GO 条件 (= 9/9 達成)**:
- ☑ f111_real_batch / refreshed price (5/14) / risk_within / top candidate /
  not sample / tradable / liquidity-event 欄明記 / 9 invariants 8/9

**HOLD 条件 (= conditional 因子強化)**:
- ⚠ D10/D11 review 2 連続 blank (= Fujiwara 未記入、actual confirm 必須)
- ⚠ D9-D12 4 連続同候補 (= features cap 律速、refresh + features rerun が真の解決)
- ⚠ 5/14 refresh → 5/29 D12 = **15 営業日 gap**
- ⚠ sector 集中 100%
- ⚠ freshness_verdict=MISSING

**判断**:
- 4 連続選出 = signal 安定の証左、ただし真の更新が来てないので Fujiwara が
  「同候補で entry する確信」を持てるかが核心
- 推奨: **D12 朝 (= 5/29 08:30 JST) 必ず再 refresh + iSPEED actual confirm**
- 全 ✓ → entry (= 340A0 #1)
- いずれか NG → skip + D13 へ持ち越し
- 持ち越し連続が続く場合は recently_seen 拡張 + sector 多様化 (= #4 7991 機械)

## §15 D12 paper PnL handoff 手順

### §15.1 D12 当日 (= 場後 15:10 close 後)

review.md §2-§3 に actual entry/exit/PnL を記入後:

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-05-29 --evaluation-date 2026-05-29 \
  --f111-real-batch-json /tmp/fire_d12_prep/d12_f111_real_batch.json \
  --morning-line-material-json /tmp/fire_d12_prep/after_r1/morning_line_material_2026-05-29.json \
  --review-md ~/fire-vault/04_daily/2026-05-29_manual_live_pilot_review.md \
  --output-json /tmp/fire_d12_prep/d12_paper_pnl_ledger_eod.json
```

### §15.2 翌営業日 (= 2026-06-01 月、朝再 refresh 後)

D12 entry 銘柄について h1 close で 1 日 paper return 確認。

### §15.3 h20 後 (= 2026-06-26 頃)

20 営業日後 outcome 判定で **真の win/loss/flat** 評価。

## §16 stage 3 突合

| 項目 | 当日 | D1-D12 累積 |
|---|---|---|
| trade 回数 | 1 (想定) | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| f111_real_batch 比率 | 1/1 | 6/12 (D6+D7+D9+D10+D11+D12) |
| top_candidates ≥1 達成率 | 1/1 | 7/12 |
| **paper PnL preview chain 連続運用** | ✓ | 2 days (D11+D12) |
| **340A0 連続選出** | ✓ (4 連続) | D9-D12 |

## §17 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D12 = review/paper PnL feedback 反映 / 4 連続 340A0
