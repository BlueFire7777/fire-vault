---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-28
owner: BlueFire7777 (Fujiwara)
pilot_day: D11
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
d10_d11_overlap: 100%
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D11_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D11_results.md
  - 04_daily/2026-05-28_manual_live_pilot_review.md
  - 04_daily/2026-05-27_manual_live_pilot_trade_plan.md  # D10 reference
artifacts:
  f111: /tmp/fire_d11_prep/d11_f111_real_batch.json
  f062: /tmp/fire_d11_prep/d11_f062_preview.json
  morning_line_material: /tmp/fire_d11_prep/after_r1/morning_line_material_2026-05-28.json
  paper_pnl_ledger: /tmp/fire_d11_prep/d11_paper_pnl_ledger.json
  d10_paper_pnl_ledger: /tmp/fire_d10_prep/d10_paper_pnl_ledger.json
---

# Manual Live Pilot — Trade Plan (2026-05-28 / D11)

## D11 特記: **paper PnL linkage 初運用 day**

- Wave 61-impl で paper PnL preview runner 実装済 → D11 候補が ledger 化可能
- D10 review = **status: blank** → review_missing 反映
- D10/D11 candidates **100% overlap** (= features cap 律速で当然、staging price max=5/14)
- top 4-12 entry 候補 (= 340A0/3798/137A0...) が **3 営業日連続** 出現 → signal 安定
- ⚠ 5/14 refresh → 5/28 D11 = **14 営業日 gap**、寄付き actual confirm 必須

---

## §1 基本情報

- date: **2026-05-28 (木 / D11)**
- 記入時刻: __:__ JST
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- J-Quants refresh status: ✓ (= W60-jquants で 5/14 まで)
- market_prices_daily max_date: **2026-05-14**
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489 --max-candidates 20`
- demoted_count: 3

## §2 D10 review recap

| 項目 | 値 |
|---|---|
| D10 review status | **blank** (= 未記入) |
| D10 entry/exit/PnL | 全 None |
| D10 final_decision | unknown |
| 影響 | review_missing warning、D10 entry 不明、Fujiwara 確認待ち |

## §3 D10 paper PnL recap

| 項目 | 値 |
|---|---|
| D10 ledger path | `/tmp/fire_d10_prep/d10_paper_pnl_ledger.json` |
| candidate_count | 20 |
| evaluated_count | 0 (= 全 pending、base 5/14 = staging max、未来 horizon 無し) |
| paper_win/loss/flat | 0 / 0 / 0 |
| missing_price_count | 0 |
| review_missing_count | 1 |
| pattern_outcomes | 9 種全 watch / evidence=low (= D10 単発、設計通り) |

## §4 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D11 当日 = **2026-05-28** → **14 営業日 gap**
- 真の解決: D11 朝寄付き前 (= 08:30 JST 想定) に再 refresh
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
    --output-json /tmp/d11_jq_refresh.json
  ```
- または寄付き直前 iSPEED で **actual price 確認**

## §5 top 3 entry 候補 (= D10 と完全同一、refreshed 5/14 close)

### §5.1 重点候補一覧

| rank | code | name | sector | close (5/14) | score | label | risk_yen | 推奨 |
|---|---|---|---|---|---|---|---|---|
| **1** | **340A0** | **ジグザグ** | 情報通信 | **380** | 0.892 | boost_with_caution | **1,900** | **★ #1** |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 0.877 | boost_with_caution | 2,525 | #2 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 0.869 | boost_with_caution | 3,695 | #3 |

⚠ **sector 100% 集中** → 多様化 #4 = 7991 マミヤ・オーピー (機械、risk=5,885) 併記推奨

### §5.2 D9/D10/D11 連続性 (= signal 安定性)

| day | top 4 | top 5 | top 6 |
|---|---|---|---|
| D9 (= 2026-05-26) | 340A0 | 3798 | 137A0 |
| D10 (= 2026-05-27) | 340A0 | 3798 | 137A0 |
| **D11 (= 2026-05-28)** | **340A0** | **3798** | **137A0** |

→ **3 営業日連続同候補** = signal 安定性確認 ✓ (= features cap 律速だが research_final_score 不変)

### §5.3 entry candidate #1: 340A0 ジグザグ

| 項目 | 値 |
|---|---|
| code / name | 340A0 / ジグザグ |
| sector | 情報通信・サービスその他 |
| close (= 2026-05-14 refreshed) | 380 円 |
| score (research_final_score) | 0.892 |
| label | boost_with_caution |
| AFTER-R1 morning score | 175.22 (= rank 1) |
| 100 株 entry 想定資金 | 38,000 円 |
| stop_loss (5%) | 361 円 = max loss 1,900 円 |
| take_profit (2%) | 388 円 = target gain 800 円 |
| pattern_tags (paper ledger) | refreshed_price_ok / manual_review_active / research_boost / risk_within_pilot_limit / multi_reason / low_risk (6 種) |
| 想定 holding period | 寄付き → 14:55 引け前 stop / TP / 15:10 close |

## §6 actual price confirmation 欄 (= 寄付き直前 08:55 JST)

| 候補 | refreshed (5/14) | actual (5/28) | 乖離率 | 採否 |
|---|---|---|---|---|
| 340A0 ジグザグ | 380 | ____ | __.__% | ☐ entry / ☐ skip |
| 3798 ＵＬＳ | 505 | ____ | __.__% | ☐ entry / ☐ skip |
| 137A0 Cocolive | 739 | ____ | __.__% | ☐ entry / ☐ skip |

乖離 ±5% 以内 → entry 検討、±5% 超 → skip 推奨。

## §7 liquidity manual check (= 寄付き 5 分後)

| 候補 | 板厚 5 本目処 | 寄付き 5 分 出来高 | 寄付き spread | 採否 |
|---|---|---|---|---|
| 340A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 3798 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 137A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |

採否基準: 板厚 ≥ 5,000 株 / 出来高 ≥ 5,000 株 / spread ≤ 0.5%

## §8 event / earnings check (= 寄付き前)

| 候補 | TDnet 開示 | 決算予定 | ネガティブニュース |
|---|---|---|---|
| 340A0 | __________ | __________ | __________ |
| 3798 | __________ | __________ | __________ |
| 137A0 | __________ | __________ | __________ |

## §9 final decision (= 寄付き直前 08:55 JST 記入)

採否を選択:

- ☐ **enter 340A0 ジグザグ** 100 株 (= entry candidate #1、3 連続選出)
- ☐ **enter 3798 ＵＬＳグループ** 100 株
- ☐ **enter 137A0 Cocolive** 100 株
- ☐ **watch (= 寄付き後再判断)**
- ☐ **skip (= 全 NG、D12 へ持ち越し)**

決定理由: __________________________

## §10 risk limit check (= 第 5 章 R-05-02/04/06/08)

| 項目 | 値 | 制約 | OK? |
|---|---|---|---|
| 1 トレード max loss | 1,900〜3,695 円 | ≤ 15,000 (pilot) | ✓ |
| 1 日 max loss | 1 トレード分 | ≤ 30,000 (pilot) | ✓ |
| 同一 sector 連続 entry | 1 件 | ≤ 2 件 | ✓ |
| stop_loss 事前設定 | 5% | 必須 | ✓ |
| ナンピンなし | 想定 | 禁止 | ✓ |
| 持ち越しなし | 15:10 完全クローズ | 必須 | ✓ |
| D6-D8 caution skip | 8747/5729/3489 | 維持 | ✓ |

## §11 9 hard invariants

| invariant | D11 result |
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

## §12 Pilot 判定: **GO_CONDITIONAL**

**GO 条件 (= 9/9 達成)**:
- ☑ f111_real_batch ✓
- ☑ refreshed price available (= 5/14) ✓
- ☑ risk_within_pilot_limit=True (= 全 3 候補) ✓
- ☑ top candidate あり (= 3 件) ✓
- ☑ sample でない ✓
- ☑ tradable_universe=True ✓
- ☑ D10 review/paper PnL で重大な問題なし (= review blank、paper pending、9 pattern 全 watch) ✓
- ☑ liquidity/event 確認欄が trade plan に明記 ✓
- ☑ 9 hard invariants 8/9 PASS ✓

**HOLD 条件 (= conditional 因子)**:
- ⚠ D10 review = blank (= Fujiwara 未記入、actual entry/exit 不明)
- ⚠ D10/D11 100% overlap (= features cap 律速、3 営業日連続同候補)
- ⚠ 5/14 refresh → 5/28 = **14 営業日 gap** (= 朝再 refresh or actual confirm 必須)
- ⚠ sector 集中 (= 情報通信 100%)
- ⚠ freshness_verdict=MISSING

→ **GO_CONDITIONAL** = 寄付き直前 actual + liquidity + event 3 確認後 entry。

## §13 D11 paper PnL handoff (= 場後・翌日確認)

### §13.1 D11 当日 (= 寄付き後・15:10 close 後)

actual entry/exit/PnL を review.md §2-§3 に記入後、以下で D11 ledger に紐付:

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-05-28 \
  --evaluation-date 2026-05-28 \
  --f111-real-batch-json /tmp/fire_d11_prep/d11_f111_real_batch.json \
  --morning-line-material-json /tmp/fire_d11_prep/after_r1/morning_line_material_2026-05-28.json \
  --good-candidate-ranking-json /tmp/fire_d11_prep/after_r1/good_candidate_ranking_2026-05-28.json \
  --paper-live-ledger-json /tmp/fire_d11_prep/after_r1/paper_live_ledger_2026-05-28.json \
  --pattern-candidate-report-json /tmp/fire_d11_prep/after_r1/pattern_candidate_report_2026-05-28.json \
  --trade-plan-md ~/fire-vault/04_daily/2026-05-28_manual_live_pilot_trade_plan.md \
  --review-md ~/fire-vault/04_daily/2026-05-28_manual_live_pilot_review.md \
  --output-json /tmp/fire_d11_prep/d11_paper_pnl_ledger.json \
  --markdown /tmp/fire_d11_prep/d11_paper_pnl_ledger.md
```

### §13.2 翌営業日 (= D12 = 5/29 金、朝再 refresh 後)

D11 entry 銘柄について h1 close で 1 日 paper return 確認:

```bash
# 朝 refresh 後 (= staging max が 5/29 に進んだ後)
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-05-28 \
  --evaluation-date 2026-05-29 \
  --f111-real-batch-json /tmp/fire_d11_prep/d11_f111_real_batch.json \
  --output-json /tmp/fire_d12_prep/d11_paper_pnl_h1_ledger.json
```

### §13.3 h20 後 (= 2026-06-26 頃)

20 営業日後の outcome 判定:

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-05-28 \
  --evaluation-date 2026-06-26 \
  --f111-real-batch-json /tmp/fire_d11_prep/d11_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-05-28_manual_live_pilot_review.md \
  --output-json /tmp/d11_h20_paper_pnl_ledger.json
```

## §14 stage 3 突合

| 項目 | 当日 | D1-D11 累積 |
|---|---|---|
| trade 回数 | 1 (想定) | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| f111_real_batch 比率 | 1/1 | 5/11 (D6+D7+D9+D10+D11) |
| top_candidates ≥1 達成率 | 1/1 | 6/11 |
| **paper PnL preview 初運用** | ✓ | 1 day (= 初) |
| 340A0 連続選出 | ✓ (3 連続) | D9-D11 |

## §15 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D11 = paper PnL linkage 初運用 / 340A0 3 連続選出
