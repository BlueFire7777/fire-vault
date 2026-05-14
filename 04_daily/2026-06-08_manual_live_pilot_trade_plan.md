---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-08
owner: BlueFire7777 (Fujiwara)
pilot_day: D18
status: plan
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
preset_tune_applied: true
label_threshold_mode: strict
recently_seen_codes: ["8747", "5729", "3489", "340A0"]
max_candidates: 20
jquants_refresh_applied: true
market_prices_daily_max_date: 2026-05-14
data_r3_freshness_verdict: MISSING
pilot_judgment: HOLD_maintained
hold_status:
  "#1_review_missing_6_consecutive": "未解消 (= D14 review 0/6、HOLD 4 連続)"
  "#2_same_candidate_7_consecutive": "解消継続 (= 340A0 demote 3 日連続)"
hard_check_chain_level_passed: 9/12
d14_review_gap_status: unresolved (= 0/6 記入)
new_top_persistence: "3798/137A0/331A0 が AFTER-R1 で D16-D18 3 日連続安定"
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D18_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D18_results.md
  - 04_daily/2026-06-08_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
artifacts:
  f111: /tmp/fire_d18_prep/d18_f111_real_batch.json
  f062: /tmp/fire_d18_prep/d18_f062_preview.json
  morning_line_material: /tmp/fire_d18_prep/after_r1/morning_line_material_2026-06-08.json
  paper_pnl_ledger: /tmp/fire_d18_prep/d18_paper_pnl_ledger.json
  d14_paper_pnl_with_review: /tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d18_check.json
---

# Manual Live Pilot — Trade Plan (2026-06-08 / D18)

## D18 特記: **HOLD maintained 4 連続 day** (= 月曜、D14 review 未記入継続)

- **D14 review 0/6 記入** = HOLD #1 残存 (= D15/D16/D17/D18 で 4 連続 HOLD)
- **340A0 demote 3 日連続** = HOLD #2 解消継続
- **3798/137A0/331A0 が D16-D18 で 3 日連続 AFTER-R1 top 3 安定** (= 新 top 確立)
- chain-level 9/12 ✓、ただし HOLD 維持で entry 推奨しない

---

## §1 基本情報

- date: **2026-06-08 (月 / D18)** (= D17 = 6/5 金から 3 日後の月曜)
- 記入時刻: __:__ JST
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489,340A0 --max-candidates 20`
- demoted_count: 4
- pilot_judgment: **HOLD maintained**

## §2 D15/D16/D17 HOLD recap

| day | pilot_judgment | HOLD #1 | HOLD #2 |
|---|---|---|---|
| D15 (5/30 想定) | HOLD | ✓ 発動 (review missing 6) | ✓ 発動 (340A0 7 連続) |
| D16 (6/3 想定) | HOLD maintained | ✓ 残存 | ✗ 解消 (340A0 demote) |
| D17 (6/5 想定) | HOLD maintained | ✓ 残存 | ✗ 解消継続 (2 日連続) |
| **D18 (6/8 月)** | **HOLD maintained** | **✓ 残存** (= 4 連続) | **✗ 解消継続** (= 3 日連続) |

## §3 D14 review 確認結果 (= 0/6 記入、D15-D18 4 連続)

| 項目 | 値 |
|---|---|
| yaml status | blank |
| §1 記入時刻 / ticker | BLANK |
| §2 entry 価格 actual | BLANK |
| §3 総 PnL | BLANK |
| §5 Reason 1 文 | BLANK |
| §6 final decision ☑ | BLANK |

→ **0/6 記入** = review_gap_UNRESOLVED → HOLD #1 残存継続。

## §4 D14 paper PnL --review-md 再 run (= D18 wave 確認)

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown
- warnings: ["review_missing"]
- output: `/tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d18_check.json`

→ D14 review blank 継続、HOLD #1 解消不可。

## §5 recently_seen_codes 維持 (= D16-D18 で 3 日連続)

- 8747, 5729, 3489, 340A0 (= demoted_count=4、設定変更なし)

## §6 340A0 demote 持続 (= D16-D18 で 3 日連続)

### 6.1 F111-real-batch top 4 (= 340A0 caution 連続)

| day | rank 4 | label | demoted |
|---|---|---|---|
| D14/D15 | 340A0 | boost_with_caution | False |
| D16 | 340A0 | caution | True (= 初実施) |
| D17 | 340A0 | caution | True (= 2 日連続) |
| **D18** | **340A0** | **caution** | **True (= 3 日連続)** |

### 6.2 AFTER-R1 top_candidates (= D16-D18 で 3 日連続安定 ✓)

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D14/D15 | 340A0 ジグザグ | 3798 ＵＬＳ | 137A0 Cocolive |
| D16 | 3798 ＵＬＳ | 137A0 Cocolive | 331A0 メディックス |
| D17 | 3798 ＵＬＳ | 137A0 Cocolive | 331A0 メディックス |
| **D18** | **3798 ＵＬＳ** | **137A0 Cocolive** | **331A0 メディックス** |

→ **新 top 3 = 3798/137A0/331A0 が D16-D18 で 3 日連続安定** = signal 完全定着。

## §7 D18 entry 候補 (= D17 と完全同一、3 日連続安定)

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| **5** | **3798** | **ＵＬＳグループ** | 情報通信 | **505** | **2,525** | **boost_with_caution ★ #1 (= 3 日連続)** |
| 6 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution |
| 7 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 | boost_with_caution (= 多様化) |
| 8 | 9130 | 共栄タンカー | 運輸 | 1,410 | 7,050 | boost_with_caution |
| 9 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |

## §8 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D18 当日 = **2026-06-08 (月)** → **23 営業日 gap** (= D17 22 → 1 営業日拡大、週末経由)
- 朝再 refresh + iSPEED actual confirmation 必須 (= D14-D17 同)

## §9 D18 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1 | f111_input_source=f111_real_batch | ✓ |
| 2 | top_candidate ≥ 1 (= 3) | ✓ |
| 3 | risk_within_pilot_limit=True | ✓ |
| 4 | sample でない | ✓ |
| 5 | tradable_universe=True | ✓ |
| 6 | forbidden_check.passed=True | ✓ |
| 7 | auto_order_allowed=False | ✓ |
| 8 | manual_review_required=True | ✓ |
| 9 | safety_flags 13 keys all False | ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §10 Pilot 判定: **HOLD maintained 4 連続** (= 設計 doc §3.3 #1 のみ残存)

### 10.1 HOLD 条件 status

| # | 条件 | D15 | D16 | D17 | D18 |
|---|---|---|---|---|---|
| 1 | review missing 5 連続超過 | ✓ 発動 | ✓ 継続 | ✓ 継続 | **✓ 継続** (= 4 連続) |
| 2 | 同 candidate 7 連続 | ✓ 発動 | ✗ 解消 | ✗ 解消継続 | **✗ 解消継続** (= 3 日連続) |

→ 5 条件中 1 残存、HOLD 維持 = **4 連続**。

### 10.2 D19 HOLD 完全解除 path (= 主推奨、4 連続呼びかけ)

| step | 内容 | 担当 |
|---|---|---|
| 1 | D14 review.md 必須 5 項目記入 ★最優先 | **Fujiwara 本人** |
| 2 | D14 paper PnL --review-md 再 run で reflection 確認 | 本線 (D19 wave) |
| 3 | D19 chain 再起 → **HOLD #1 解消 → GO_CONDITIONAL 復帰** | 本線 (D19 wave) |

### 10.3 D18 推奨アクション

- **主推奨**: skip (= HOLD 維持) + D14 review 記入推奨 + D19 で完全解除
- **alternative**: 3798 ＵＬＳ に少額 entry (= HOLD override、Fujiwara 責任、3 日連続 #1 で signal 完全定着、最も entry 価値の高いタイミング)

## §11 entry candidate #1 (= 参考、HOLD 中)

| 項目 | 値 |
|---|---|
| code / name | 3798 / ＵＬＳグループ |
| sector | 情報通信・サービスその他 |
| close (5/14 refreshed) | 505 円 |
| score | 0.877 (A1 rank) |
| label | boost_with_caution |
| AFTER-R1 rank | 1 (= D16-D18 3 日連続) |
| 100 株 entry 想定資金 | 50,500 円 |
| stop_loss (5%) | 480 円 = max loss **2,525 円** |
| take_profit (2%) | 515 円 = target gain 1,000 円 |

## §12 actual price confirmation (= HOLD 中、参考)

```bash
# 朝 J-Quants focused refresh
set -a; source /Users/bluefire/fire/.env; set +a
FIRE_ENV=staging \
HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 \
HQ_APPROVE_JQUANTS_TOKEN_READ=1 \
.venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
  --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
  --datasets prices --max-days 14 \
  --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
  --sleep-seconds 0.3 --write \
  --output-json /tmp/d18_jq_refresh.json
```

| 候補 | refreshed (5/14) | actual (6/8) | 乖離率 |
|---|---|---|---|
| 3798 ＵＬＳ (= #1) | 505 | ____ | __.__% |
| 137A0 Cocolive | 739 | ____ | __.__% |
| 331A0 メディックス | 482 | ____ | __.__% |
| 7991 マミヤ (= 多様化) | 1,177 | ____ | __.__% |

## §13 liquidity manual check

| 候補 | 板厚 | 出来高 | spread |
|---|---|---|---|
| 3798 | ____ | ____ | __.__% |
| 137A0 | ____ | ____ | __.__% |
| 331A0 | ____ | ____ | __.__% |

## §14 event / earnings check (= 月曜寄付き前)

週末 (= 5/30/5/31, 6/6/6/7) ニュース + 月曜寄付き event 確認:

| 候補 | 週末 TDnet | 決算予定 | ニュース |
|---|---|---|---|
| 3798 | __________ | __________ | __________ |
| 137A0 | __________ | __________ | __________ |
| 331A0 | __________ | __________ | __________ |

## §15 final decision (= D18 推奨)

- ☐ **skip (= HOLD maintained 4 連続、D14 review 記入推奨)** ★ 主推奨
- ☐ enter 3798 ＵＬＳ 100 株 (= HOLD override、3 日連続 #1、Fujiwara 責任)
- ☐ enter 7991 マミヤ (= sector 多様化、HOLD override)
- ☐ watch

決定理由: __________________________

## §16 D18 review 必須 5 項目

1. ★ §1 記入時刻 + 対応 (= "HOLD maintained 4 連続対応")
2. ★ §2 計画 vs 実際 (= "HOLD、entry skip" 想定)
3. ★ §3 PnL (= "0 円 (HOLD)")
4. ★ §5 Reason (= "HOLD #1 残存 4 連続、D14 review 記入最優先")
5. ★ §6 final decision = skip

## §17 D18 paper PnL handoff

### §17.1 D18 当日 (= 場後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-08 --evaluation-date 2026-06-08 \
  --f111-real-batch-json /tmp/fire_d18_prep/d18_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-08_manual_live_pilot_review.md \
  --output-json /tmp/fire_d18_prep/d18_paper_pnl_ledger_eod.json
```

### §17.2 D19 翌火曜 (= 2026-06-09)

D14 review 記入確認後、D19 で HOLD 完全解除 → GO_CONDITIONAL 復帰可能性大。

## §18 stage 3 突合

| 項目 | 当日 | D1-D18 累積 |
|---|---|---|
| trade 回数 | 0 (= HOLD) | __ / 50 |
| 累積 PnL | 0 円 | __ 円 |
| f111_real_batch 比率 | 1/1 | 12/18 |
| top_candidates ≥1 達成率 | 1/1 | 13/18 |
| **HOLD maintained 連続** | 4 日 (D15-D18) | 4 days |
| 340A0 demote 維持 | 3 日連続 | 3 days |
| 新 top 3 (= 3798/137A0/331A0) | 3 日連続 | 3 days |
| review missing 連続 | 6 連続 (D9-D14) | 6 days |

## §19 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D18 = HOLD maintained 4 連続 (= 月曜 day) /
  340A0 demote 3 日連続 / 3798 新 #1 3 日連続 /
  D14 review 記入後 D19 解除可能性大
