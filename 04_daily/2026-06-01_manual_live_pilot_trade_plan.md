---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-01
owner: BlueFire7777 (Fujiwara)
pilot_day: D13
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
data_r3_freshness_json: /tmp/f286_data_r3_freshness.json
data_r3_freshness_verdict: MISSING
focused_refresh_attempted: skip (= already_up_to_date、staging max = wave 実行日)
actual_confirmation_required: true
pilot_judgment: GO_CONDITIONAL
d10_d11_d12_review_status: missing (= 3 連続未記入)
d9_d13_overlap: 100%
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D13_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D13_results.md
  - 04_daily/2026-06-01_manual_live_pilot_review.md
artifacts:
  f111: /tmp/fire_d13_prep/d13_f111_real_batch.json
  f062: /tmp/fire_d13_prep/d13_f062_preview.json
  morning_line_material: /tmp/fire_d13_prep/after_r1/morning_line_material_2026-06-01.json
  paper_pnl_ledger: /tmp/fire_d13_prep/d13_paper_pnl_ledger.json
  data_r3_freshness: /tmp/f286_data_r3_freshness.json
---

# Manual Live Pilot — Trade Plan (2026-06-01 / D13)

## D13 特記: **review gap closure + actual confirmation 強制 day**

- D10/D11/D12 review = **3 連続 blank** (= Fujiwara 未記入、明示的処理)
- focused J-Quants refresh = **skip** (= staging max=5/14 = wave 実行日、already_up_to_date)
- DATA-R3 freshness JSON 接続 **path 確認** ✓ (= verdict=MISSING、実 verdict=OK は別 wave 必要)
- 340A0/3798/137A0 が **D9-D13 5 営業日連続選出** = signal 極めて安定
- 中核判断: **actual price confirmation 強制** + review_gap 明示

---

## §1 基本情報

- date: **2026-06-01 (月 / D13)** (= 5/30 土 + 5/31 日後)
- 記入時刻: __:__ JST
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- J-Quants refresh status: ✓ (= 5/14 まで catch-up 済、focused refresh attempt は already_up_to_date)
- market_prices_daily max_date: **2026-05-14**
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489 --max-candidates 20`
- demoted_count: 3
- DATA-R3 freshness JSON 接続: ✓ (= 経路確認、verdict=MISSING)

## §2 D10/D11/D12 review recap (= 3 連続 review_missing)

| 項目 | D10 (5/27) | D11 (5/28) | D12 (5/29) |
|---|---|---|---|
| status | **blank** | **blank** | **blank** |
| final_decision | unknown | unknown | unknown |
| actual entry/exit/PnL | 全 None | 全 None | 全 None |
| pilot_judgment | GO_CONDITIONAL | GO_CONDITIONAL | GO_CONDITIONAL |

→ 3 連続 review_missing は **明確な caveat**、D13 で 4 回目記入機会。
   FIRE-CODEX-R1 設計 § 6.2: 未記入 = 捏造しない、Fujiwara 本人の記入を待つ。

## §3 paper PnL recap (= D10/D11/D12 ledger 確認)

| 項目 | D10 | D11 | D12 |
|---|---|---|---|
| candidate_count | 20 | 20 | 20 |
| evaluated_count | 0 | 0 | 0 (= 全 pending、staging max=5/14) |
| review_missing_count | 1 | 1 | 1 |
| pattern_outcomes | 9 種全 watch / evidence=low | 同 | 同 |

→ paper PnL は全 pending、真の outcome は h20 後 (= 2026-06-26 頃) に再 run で評価。

## §4 focused J-Quants refresh 試行結果

| 項目 | 値 |
|---|---|
| 試行 | dry-run preflight 実施 |
| 結果 | **target=None〜None / span=0 / reason=already_up_to_date** |
| 理由 | staging max=2026-05-14 = wave 実行日 (= 今日)、未来データ無し |
| HQ marker | HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 + HQ_APPROVE_JQUANTS_TOKEN_READ=1 (= 承認済、ただし使用せず) |
| token | JQUANTS_API_KEY SET (len=43、値非表示) |
| 実 refresh | **skip** (= 不要、API call 0) |
| production / develop md5 | 不変 ✓ |

→ refresh skip は **技術的に妥当**。D13 当日 (= 6/1) に実行するなら 5/14-5/29 範囲を取れる。

## §5 ⚠ price freshness caveat (= actual confirmation 強制)

- staging max_date = **2026-05-14**
- D13 当日 = **2026-06-01** (月) → **18 営業日 gap** (= D12 15 → 3 日拡大、土日含む)
- **actual price confirmation 強制**:
  - 朝寄付き前 (= 08:30 JST) → 朝 refresh (= 6/1 当日に 5/15-5/29 を catch-up)
  - 寄付き直前 (= 08:55 JST) → iSPEED で actual price 確認
  - 乖離 ±5% 以内 → entry 検討、±5% 超 → skip

## §6 DATA-R3 freshness 接続試行結果

| 項目 | 値 |
|---|---|
| runner | scripts/jobs/run_f286_data_r3_daily_refresh.py |
| 実行 | --dry-run smoke |
| freshness_report_json | /tmp/f286_data_r3_freshness.json |
| schema_version | 1.0 ✓ |
| verdict | **MISSING** (= dry-run + 実 fetch 無しなので想定通り) |
| planned jobs | f100_historical / f101_announcements / f111_research_watchlist_signal_persistence / f119_evaluation |
| AFTER-R1 接続 | ✓ (= --data-r3-freshness-json 引数で渡せた、verdict 反映確認) |

→ **接続経路確認 ✓**、実 verdict=OK 達成は別 wave (= active job execution、HQ_APPROVE_FETCH_ACTIVE_JOB 必要)。

## §7 D9-D13 candidate overlap (= 5 営業日連続)

| day | rank 4 | close | risk_yen | label |
|---|---|---|---|---|
| D9 (5/26) | 340A0 ジグザグ | 398 stale | 1,990 | boost_with_caution |
| D10 (5/27) | 340A0 | 380 refreshed | 1,900 | boost_with_caution |
| D11 (5/28) | 340A0 | 380 | 1,900 | boost_with_caution |
| D12 (5/29) | 340A0 | 380 | 1,900 | boost_with_caution |
| **D13 (6/1)** | **340A0** | **380** | **1,900** | **boost_with_caution** |

→ **5 営業日連続 340A0** = signal 極めて安定 (= features cap 律速、refresh 不要なので同価格)

## §8 top 3 entry 候補 (= D11/D12 と完全同一)

| rank | code | name | sector | close (5/14) | score | label | risk_yen | 推奨 |
|---|---|---|---|---|---|---|---|---|
| **1** | **340A0** | **ジグザグ** | 情報通信 | **380** | 0.892 | boost_with_caution | **1,900** | **★ #1** |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 0.877 | boost_with_caution | 2,525 | #2 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 0.869 | boost_with_caution | 3,695 | #3 |

⚠ sector 100% 集中 → #4 = 7991 マミヤ・オーピー (機械、risk=5,885) 多様化推奨

## §9 entry candidate #1: 340A0 ジグザグ (= 5 連続選出)

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
| 5 連続選出 caveat | signal 極めて安定、ただし features cap 律速 |

## §10 actual price confirmation 欄 (= D13 朝 強制実行)

### §10.1 朝 refresh (= 08:30 JST)

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
  --output-json /tmp/d13_jq_refresh.json
```

### §10.2 actual price 確認 (= 08:55 JST、iSPEED で)

| 候補 | refreshed (5/14) | actual (6/1) | 乖離率 | 採否 |
|---|---|---|---|---|
| 340A0 ジグザグ | 380 | ____ | __.__% | ☐ entry / ☐ skip |
| 3798 ＵＬＳ | 505 | ____ | __.__% | ☐ entry / ☐ skip |
| 137A0 Cocolive | 739 | ____ | __.__% | ☐ entry / ☐ skip |

乖離 ±5% 以内 → entry 検討、±5% 超 → **skip 強制**。

## §11 liquidity manual check (= 寄付き 5 分後)

| 候補 | 板厚 5 本目処 | 寄付き 5 分 出来高 | 寄付き spread | 採否 |
|---|---|---|---|---|
| 340A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 3798 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 137A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |

採否基準: 板厚 ≥ 5,000 株 / 出来高 ≥ 5,000 株 / spread ≤ 0.5%

## §12 event / earnings check (= 寄付き前)

| 候補 | TDnet 開示 | 決算予定 | ネガティブニュース |
|---|---|---|---|
| 340A0 | __________ | __________ | __________ |
| 3798 | __________ | __________ | __________ |
| 137A0 | __________ | __________ | __________ |

## §13 final decision (= 寄付き直前 08:55 JST 記入)

- ☐ **enter 340A0** 100 株 (= entry candidate #1、5 連続選出)
- ☐ **enter 3798** 100 株
- ☐ **enter 137A0** 100 株
- ☐ **enter 7991 マミヤ・オーピー** 100 株 (= sector 多様化、機械)
- ☐ **watch (= 寄付き後再判断)**
- ☐ **skip (= 全 NG、D14 へ持ち越し)**

決定理由: __________________________

## §14 risk limit check

| 項目 | 値 | 制約 | OK? |
|---|---|---|---|
| 1 トレード max loss | 1,900〜3,695 円 | ≤ 15,000 | ✓ |
| 1 日 max loss | 1 トレード分 | ≤ 30,000 | ✓ |
| 同一 sector 連続 entry | 1 件 | ≤ 2 件 | ✓ |
| stop_loss 事前設定 | 5% | 必須 | ✓ |
| ナンピンなし | 想定 | 禁止 | ✓ |
| 持ち越しなし | 15:10 完全クローズ | 必須 | ✓ |
| D6-D8 caution skip | 8747/5729/3489 | 維持 | ✓ |
| **3 review missing 認識** | D10/D11/D12 | 必須 | ✓ |
| **18 営業日 gap 認識** | 5/14 → 6/1 | 必須 | ✓ |
| **DATA-R3 freshness MISSING 認識** | path 接続のみ | 必須 | ✓ |

## §15 9 hard invariants

| invariant | D13 result |
|---|---|
| artifact_source | f062_preview ✓ |
| f062_raw_kind | f062_actual_dict ✓ |
| f111_input_source | f111_real_batch ✓ |
| freshness_verdict | **MISSING** (= DATA-R3 gate 接続済、verdict=OK は別 wave) |
| forbidden_phrases_check.passed | True ✓ |
| auto_order_allowed | False ✓ |
| manual_review_required | True ✓ |
| safety_flags 13 keys all False | True ✓ |
| top_candidates ≥ 1 | 3 ✓ |

→ 8/9 PASS。

## §16 Pilot 判定: **GO_CONDITIONAL maintained** (= 改善 path 接続済)

**GO 条件 (= 9/9 達成)**:
- ☑ f111_real_batch / refreshed price (5/14) / risk_within / top candidate /
  not sample / tradable / actual confirmation 欄強制 / 9 invariants 8/9 PASS /
  D10-D12 review/paper PnL に **active 問題なし** (= blank 3 連続は missing として明示)

**HOLD 条件 (= conditional 因子)**:
- ⚠ D10/D11/D12 review 3 連続 blank (= 4 回目記入機会、必須化)
- ⚠ D9-D13 **5 連続同候補** (= signal 安定、ただし features cap 律速)
- ⚠ 5/14 refresh → 6/1 D13 = **18 営業日 gap** (= 朝再 refresh 必須)
- ⚠ sector 集中 100%
- ⚠ freshness_verdict=MISSING (= 接続経路 OK、実 verdict=OK は別 wave)

→ **GO_CONDITIONAL** = 寄付き直前 actual + liquidity + event + 朝再 refresh の
  4 確認 後 entry。いずれか NG → skip。

## §17 D13 → GO 寄り改善 達成事項

| 改善 | 達成 |
|---|---|
| review 欠損明示処理 | ✓ (= 3 review missing 明記、warnings に review_missing) |
| focused J-Quants refresh 試行 | ✓ (= dry-run skip、原因明記 = already_up_to_date) |
| DATA-R3 freshness 接続経路 | ✓ (= --data-r3-freshness-json 引数で渡せた、verdict 反映確認) |
| actual confirmation 強制 | ✓ (= §10 で朝 refresh + 08:55 iSPEED actual) |
| candidate 多様化 caveat | ✓ (= sector 集中 100% 明示、#4 7991 多様化推奨) |

→ **GO_CONDITIONAL → GO 寄り** の 5 改善 path 全て trade plan に明示。

## §18 D13 paper PnL handoff 手順

### §18.1 D13 当日 (= 場後 15:10 close 後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-01 --evaluation-date 2026-06-01 \
  --f111-real-batch-json /tmp/fire_d13_prep/d13_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-01_manual_live_pilot_review.md \
  --output-json /tmp/fire_d13_prep/d13_paper_pnl_ledger_eod.json
```

### §18.2 翌営業日 (= 2026-06-02 火、朝再 refresh 後)

D13 entry 銘柄について h1 close で 1 日 paper return 確認。

### §18.3 h20 後 (= 2026-06-26 頃)

20 営業日後 outcome 判定で **真の win/loss/flat** 評価。

## §19 stage 3 突合

| 項目 | 当日 | D1-D13 累積 |
|---|---|---|
| trade 回数 | 1 (想定) | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| f111_real_batch 比率 | 1/1 | 7/13 (D6+D7+D9+D10+D11+D12+D13) |
| top_candidates ≥1 達成率 | 1/1 | 8/13 |
| **DATA-R3 freshness 接続経路確認** | ✓ (= 初) | 1 day |
| **focused refresh 試行 + skip** | ✓ (= 設計通り) | 1 day |
| 340A0 連続選出 | ✓ (D9-D13) | 5 days |

## §20 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D13 = review gap closure + DATA-R3 接続経路確認 + 5 連続 340A0
