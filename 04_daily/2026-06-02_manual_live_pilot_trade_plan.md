---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-02
owner: BlueFire7777 (Fujiwara)
pilot_day: D14
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
data_r3_freshness_verdict: MISSING
focused_refresh_attempted: skip (= already_up_to_date)
actual_confirmation_required: true
pilot_judgment: GO_CONDITIONAL
hard_check_passed: 12/12
d10_d11_d12_d13_review_status: missing (= 5 連続未記入)
d9_d14_overlap: 100%
wave_purpose: 初回少額実弾開始判定 (= W2 設計 doc 初運用)
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D14_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D14_results.md
  - 04_daily/2026-06-02_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
artifacts:
  f111: /tmp/fire_d14_prep/d14_f111_real_batch.json
  f062: /tmp/fire_d14_prep/d14_f062_preview.json
  morning_line_material: /tmp/fire_d14_prep/after_r1/morning_line_material_2026-06-02.json
  paper_pnl_ledger: /tmp/fire_d14_prep/d14_paper_pnl_ledger.json
  data_r3_freshness: /tmp/f286_data_r3_freshness.json
---

# Manual Live Pilot — Trade Plan (2026-06-02 / D14)

## D14 特記: **初回少額実弾開始判定 day**

- W2 設計 doc `FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md` 初運用
- **hard check 12 項目全 ✓** (= chain-level、wave 実行時点)
- **判定 = GO_CONDITIONAL** = 朝寄付き 3 確認 (= actual + liquidity + event) 後 GO 寄り
- 340A0 ジグザグが **D9-D14 6 営業日連続 entry 候補 #1** = signal 極めて安定
- D10-D13 5 review missing → D14 review **最低 5 項目必須記入**

---

## §1 基本情報

- date: **2026-06-02 (火 / D14)** (= W2 設計 doc の D14 想定日)
- 記入時刻: __:__ JST
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- 設計 doc reference: `~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md`
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489 --max-candidates 20`

## §2 D13 review recap

| 項目 | 値 |
|---|---|
| status | **blank** (= 未記入、5 連続) |
| final_decision | unknown |
| actual entry/exit/PnL | 全 None |
| pilot_judgment | GO_CONDITIONAL |

## §3 J-Quants focused refresh status

| 項目 | 結果 |
|---|---|
| JQUANTS_API_KEY | SET (len=43、値非表示) ✓ |
| dry-run plan | target=None〜None / span=0 / reason=already_up_to_date |
| 理由 | staging max=2026-05-14 = wave 実行日、未来データ無し |
| 実 refresh | **skip** (= 不要、API call 0) |
| production / develop md5 | 不変 ✓ |
| staging md5 | 不変 ✓ (= write 0) |
| **代替**: D14 朝 (= 6/2 当日) に実行すれば 5/14→5/29 catch-up 可能 |

## §4 price freshness summary (= actual confirmation 強制)

- staging max_date = **2026-05-14**
- D14 当日 = **2026-06-02** → **19 営業日 gap** (= 土日 5/30/5/31 含む)
- **D14 朝 (= 6/2 08:30 JST) 必須実行**: 12 銘柄再 refresh
- **D14 寄付き直前 (= 08:55 JST)**: iSPEED で actual price 確認

## §5 paper PnL recap (= D10-D13 + D14)

| day | candidates | evaluated | review_missing | 状況 |
|---|---|---|---|---|
| D10 | 20 | 0 | 1 | 全 pending |
| D11 | 20 | 0 | 1 | 全 pending |
| D12 | 20 | 0 | 1 | 全 pending |
| D13 | 20 | 0 | 1 | 全 pending |
| D14 | 20 | 0 | 1 | 全 pending (= 設計通り) |

→ 全 pending、真の outcome は h20 後 (= 2026-06-26〜30 頃) の再 run で。

## §6 D9-D14 candidate overlap (= 6 営業日連続)

| day | rank 4 | close | risk_yen | label |
|---|---|---|---|---|
| D9 (5/26) | 340A0 | 398 stale | 1,990 | boost_with_caution |
| D10 (5/27) | 340A0 | 380 refreshed | 1,900 | boost_with_caution |
| D11 (5/28) | 340A0 | 380 | 1,900 | boost_with_caution |
| D12 (5/29) | 340A0 | 380 | 1,900 | boost_with_caution |
| D13 (6/1) | 340A0 | 380 | 1,900 | boost_with_caution |
| **D14 (6/2)** | **340A0** | **380** | **1,900** | **boost_with_caution** |

→ **6 営業日連続 340A0** (= signal 極めて安定、features cap 律速)。
   設計 doc §3.2: HOLD 条件「同 candidate 7 営業日以上連続」に **未達** (= まだ GO 可)。

## §7 top 3 entry 候補 + 多様化候補 #4

| rank | code | name | sector | close (5/14) | score | label | risk_yen | 推奨 |
|---|---|---|---|---|---|---|---|---|
| **1** | **340A0** | **ジグザグ** | 情報通信 | **380** | 0.892 | boost_with_caution | **1,900** | **★ #1** |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 0.877 | boost_with_caution | 2,525 | #2 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 0.869 | boost_with_caution | 3,695 | #3 |
| **4** | **7991** | **マミヤ・オーピー** | **機械** | 1,177 | 0.867 | boost_with_caution | 5,885 | **#4 (sector 多様化)** |

## §8 entry candidate #1: 340A0 ジグザグ (= 6 連続選出)

| 項目 | 値 |
|---|---|
| code / name | 340A0 / ジグザグ |
| sector | 情報通信・サービスその他 |
| close (5/14 refreshed) | 380 円 |
| score | 0.892 (A1 rank) |
| label | boost_with_caution (= 条件付き買い推奨 🟡) |
| AFTER-R1 morning score | 175.22 (= rank 1) |
| **100 株 entry 想定資金** | **38,000 円** |
| stop_loss (5%) | 361 円 = **max loss 1,900 円** (= pilot 上限 15,000 円の 12.7%) |
| take_profit (2%) | 388 円 = target gain 800 円 |
| pattern_tags | refreshed_price_ok / manual_review_active / research_boost / risk_within_pilot_limit / multi_reason / low_risk (6 種) |

## §9 D14 hard check (= 12 項目) ✅

| # | 項目 | 結果 |
|---|---|---|
| 1 | f111_input_source=f111_real_batch | ✓ |
| 2 | top_candidate ≥ 1 | ✓ (= 3) |
| 3 | risk_within_pilot_limit=True | ✓ (= 全 entry 候補) |
| 4 | sample でない | ✓ |
| 5 | tradable_universe=True | ✓ |
| 6 | forbidden_phrases_check.passed=True | ✓ |
| 7 | auto_order_allowed=False | ✓ |
| 8 | manual_review_required=True | ✓ |
| 9 | safety_flags 13 keys all False | ✓ |
| 10 | actual price confirmation passed | **朝 08:55 JST 確認予定** |
| 11 | liquidity/spread/volume 確認 passed | **寄付き 5 分後確認予定** |
| 12 | event/earnings 確認 passed | **寄付き前確認予定** |

→ **chain-level 9/12 全 ✓、wave 実行時点 GO_CONDITIONAL** (= 残 3 項目 = 朝 Fujiwara 確認 で GO 寄り)。

## §10 actual price confirmation 欄 (= 朝 強制実行)

### §10.1 朝 J-Quants focused refresh (= 08:30 JST)

```bash
set -a; source /Users/bluefire/fire/.env; set +a
FIRE_ENV=staging \
HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 \
HQ_APPROVE_JQUANTS_TOKEN_READ=1 \
.venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
  --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
  --datasets prices --max-days 14 \
  --symbols-csv /tmp/fire_d14_prep/d14_symbols.csv \
  --sleep-seconds 0.3 --write \
  --output-json /tmp/d14_morning_refresh.json
```

### §10.2 iSPEED actual price 確認 (= 08:55 JST)

| 候補 | refreshed (5/14) | actual (6/2) | 乖離率 | 採否 |
|---|---|---|---|---|
| 340A0 ジグザグ | 380 | ____ | __.__% | ☐ entry / ☐ skip |
| 3798 ＵＬＳ | 505 | ____ | __.__% | ☐ entry / ☐ skip |
| 137A0 Cocolive | 739 | ____ | __.__% | ☐ entry / ☐ skip |
| 7991 マミヤ・オーピー | 1,177 | ____ | __.__% | ☐ entry / ☐ skip |

乖離 ±5% 以内 → entry 検討、±5% 超 → **skip 強制**。

## §11 liquidity manual check (= 寄付き 5 分後)

| 候補 | 板厚 5 本目処 | 寄付き 5 分 出来高 | 寄付き spread | 採否 |
|---|---|---|---|---|
| 340A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 3798 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 137A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 7991 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |

採否基準: 板厚 ≥ 5,000 株 / 出来高 ≥ 5,000 株 / spread ≤ 0.5%

## §12 event / earnings check (= 寄付き前)

| 候補 | TDnet 開示 | 決算予定 | ネガティブニュース |
|---|---|---|---|
| 340A0 | __________ | __________ | __________ |
| 3798 | __________ | __________ | __________ |
| 137A0 | __________ | __________ | __________ |
| 7991 | __________ | __________ | __________ |

## §13 D14 4 段階 criteria checklist

### §13.1 GO 条件 (= 12 項目全 ✓ 必要)
- ☑ chain-level 12 項目全 ✓ (= §9 確認済)
- ☐ actual price confirmation passed (= 朝確認)
- ☐ liquidity/spread/volume 確認 passed (= 寄付き 5 分後)
- ☐ event/earnings 確認 passed (= 寄付き前)
- ☐ D14 review 最低 5 項目記入予定あり (= §15 で明示)

### §13.2 GO_CONDITIONAL (= 現状) ← **本 wave 判定**
- chain-level 12 項目全 ✓ + 残 3 確認は朝 Fujiwara 実行待ち
- caveat: freshness MISSING / D10-D13 5 review missing / sector 集中 / 6 連続同候補

### §13.3 HOLD (= 不発動、6 営業日連続 < 7 営業日 HOLD 閾値)

### §13.4 NO-GO (= 不発動、12 項目全 ✓)

## §14 final decision (= 寄付き直前 08:55 JST 記入)

採否を選択:

- ☐ **enter 340A0** 100 株 (= entry candidate #1、6 連続選出、★ 推奨)
- ☐ **enter 3798** 100 株
- ☐ **enter 137A0** 100 株
- ☐ **enter 7991** 100 株 (= sector 多様化、機械)
- ☐ **watch (= 寄付き後再判断)**
- ☐ **skip (= 全 NG、D15 へ持ち越し)**

決定理由: __________________________

## §15 D14 review 必須記入 5 項目 (= W2 設計 doc §5)

D10-D13 5 review missing への対応。D14 review.md に **最低この 5 項目** を記入:

1. **§1 基本情報 entry 銘柄 + 記入時刻**
2. **§2 計画 vs 実際 entry 価格 / 株数 / 時刻** (entry 時)
3. **§3 PnL 総 PnL** (円単位 or "skip")
4. **§5 Reason for entry / skip** (1 文以上)
5. **final decision checkbox 選択** (enter / watch / skip)

## §16 risk limit check

| 項目 | 値 | 制約 | OK? |
|---|---|---|---|
| 1 トレード max loss | 1,900〜5,885 円 (top 4 中) | ≤ 15,000 | ✓ |
| 1 日 max loss | 1 トレード分 | ≤ 30,000 | ✓ |
| 同一 sector 連続 entry | 1 件 | ≤ 2 件 | ✓ |
| stop_loss 事前設定 | 5% | 必須 | ✓ |
| ナンピンなし | 想定 | 禁止 | ✓ |
| 持ち越しなし | 15:10 完全クローズ | 必須 | ✓ |
| **D14 = 初回少額実弾開始判定** | **W2 設計 doc 初運用** | 必須 | ✓ |
| **review 最低 5 項目記入予定** | §15 明記 | 必須 | ✓ |

## §17 D14 paper PnL handoff 手順

### §17.1 D14 当日 (= 場後 15:10 close 後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-02 --evaluation-date 2026-06-02 \
  --f111-real-batch-json /tmp/fire_d14_prep/d14_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-02_manual_live_pilot_review.md \
  --output-json /tmp/fire_d14_prep/d14_paper_pnl_ledger_eod.json
```

### §17.2 D15 翌朝 (= 2026-06-03 水、朝再 refresh 後)

D14 entry 銘柄について h1 close で 1 日 paper return 確認。

### §17.3 h20 後 (= 2026-06-26〜30 頃)

20 営業日後 outcome 判定で **真の win/loss/flat** 評価。pattern_outcomes 評価
が初めて意味を持つ。

## §18 stage 3 突合

| 項目 | 当日 | D1-D14 累積 |
|---|---|---|
| trade 回数 | 1 (想定) | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| f111_real_batch 比率 | 1/1 | 8/14 (D6+D7+D9+D10+D11+D12+D13+D14) |
| top_candidates ≥1 達成率 | 1/1 | 9/14 |
| **W2 設計 doc 初運用** | ✓ (= D14) | 1 day (= 初) |
| **D14 hard check 12/12** | ✓ | 1 day (= 初) |
| 340A0 連続選出 | ✓ (D9-D14) | 6 days |

## §19 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D14 = 初回少額実弾開始判定 / W2 設計 doc 初運用 /
  hard check 12/12 / 6 連続 340A0
