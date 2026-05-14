---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-15
owner: BlueFire7777 (Fujiwara)
pilot_day: D23
status: plan
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
preset_tune_applied: true
label_threshold_mode: strict
recently_seen_codes: ["8747", "5729", "3489", "340A0", "3798"]
max_candidates: 20
jquants_refresh_applied: true
market_prices_daily_max_date: 2026-05-14
data_r3_freshness_verdict: MISSING
pilot_judgment: GO_CONDITIONAL
post_recovery_status: "D20-D23 で 4 連続 GO_CONDITIONAL maintained"
hard_check_chain_level_passed: 9/12
w3_recovery_criteria_passed: 9/9 (= #7 朝確認待ち)
d22_review_gap_status: unresolved (= 0/5 記入)
new_top_persistence: "137A0/7991/331A0 D20-D23 4 連続安定"
hold_2_re_emergence_warning: "137A0 rank 1 4 連続到達、D24 = warning 5 連続、D26 で 7 連続 HOLD 再発動"
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D23_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D23_results.md
  - 04_daily/2026-06-15_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
artifacts:
  f111: /tmp/fire_d23_prep/d23_f111_real_batch.json
  f062: /tmp/fire_d23_prep/d23_f062_preview.json
  morning_line_material: /tmp/fire_d23_prep/after_r1/morning_line_material_2026-06-15.json
  paper_pnl_ledger: /tmp/fire_d23_prep/d23_paper_pnl_ledger.json
---

# Manual Live Pilot — Trade Plan (2026-06-15 / D23)

## D23 特記: **GO_CONDITIONAL 4 連続 + ⚠ 137A0 4 連続到達 (= HOLD #2 まで残 3 日)**

- **D20-D23 で 4 連続 GO_CONDITIONAL maintained**
- **137A0 rank 1 が D20-D23 で 4 連続到達** → HOLD #2 まで残 3 日
- 7991/331A0 も 4 連続維持 (= sector 多様化機械維持)
- **D24 = 5 連続 warning 段階 → 137A0 demote 検討必須化**
- D22 review = 0/5 (= path 2 適用継続)

---

## §1 基本情報

- date: **2026-06-15 (月 / D23)**
- 記入時刻: __:__ JST
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489,340A0,3798 --max-candidates 20`
- demoted_count: 5 (= D20-D23 維持)
- pilot_judgment: **GO_CONDITIONAL maintained 4 連続**

## §2 D20-D22 GO_CONDITIONAL recap

| day | pilot | top 1 | review |
|---|---|---|---|
| D20 | GO_CONDITIONAL 復帰 | 137A0 (新 #1) | partial 1/6 |
| D21 | maintained 2 連続 | 137A0 | 0/5 |
| D22 | maintained 3 連続 | 137A0 | 0/5 |
| **D23** | **maintained 4 連続** | **137A0 (4 連続)** | (本 review) |

## §3 D22 review (= 0/5 記入)

→ W3 §7.1 path 2 適用継続、HOLD #1 解除 path で復帰維持。

## §4 D22 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown

## §5 recently_seen_codes 維持

- 8747, 5729, 3489, 340A0, 3798 (= 5 件、demoted_count=5、D20 から維持)

## §6 D20-D23 連続性 (= 4 連続安定 ✓)

### 6.1 AFTER-R1 top_candidates

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D20 | 137A0 | 7991 (機械) | 331A0 |
| D21 | 137A0 | 7991 | 331A0 |
| D22 | 137A0 | 7991 | 331A0 |
| **D23** | **137A0 (4 連続)** | **7991 (機械、4 連続)** | **331A0 (4 連続)** |

→ **4 連続安定** = signal 完全定着、sector 多様化維持。

## §7 ⚠ 137A0 連続候補化リスク (= warning 前段階)

| day | 137A0 rank 1 連続 | HOLD #2 まで残日数 |
|---|---|---|
| D20-D22 | 1-3 連続 | 6-4 日 |
| **D23** | **4 連続** | **3 日** |
| **D24 (予想)** | **5 連続 (= warning 段階)** | **2 日** |
| D25 | 6 連続 | 1 日 |
| D26 | **7 連続 = HOLD #2 再発動懸念** | 0 日 |

### 7.1 D24 で 137A0 demote 検討必須化推奨

- **W4 集約 wave (= D20-D24 後) 推奨**
- または **D24 で 137A0 を recently_seen に追加** (= 5 連続 warning 段階で先行)
- 新 top 想定: 7991 (機械) が rank 1 へ、331A0 が rank 2、新 candidate が rank 3

## §8 D23 entry 候補 (= D20-D23 で 4 連続安定)

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| 1 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution ★ #1 (= 4 連続) |
| 2 | **7991** | **マミヤ・オーピー** | **機械** | 1,177 | 5,885 | boost_with_caution ★ 多様化 (= 4 連続) |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution (= 4 連続) |

## §9 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D23 当日 = **2026-06-15 (月)** → **28 営業日 gap**
- 朝再 refresh + iSPEED actual confirmation 必須

## §10 D23 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §11 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

| # | 条件 | D23 結果 |
|---|---|---|
| 1 | review 5 項目 OR gap 明確化 | ✓ (= path 2 継続) |
| 2 | 340A0 demote 維持 | ✓ (= 8 連続) |
| 3 | 3798 recently_seen 維持 | ✓ (= 4 連続) |
| 4 | f111_real_batch | ✓ |
| 5 | top candidate ≥ 1 | ✓ (= 3) |
| 6 | risk_within_pilot_limit | ✓ |
| 7 | actual/liquidity/event 確認可能 | 朝確認待ち |
| 8 | safety_flags 全 False | ✓ |
| 9 | paper PnL/review 重大問題なし | ✓ |

→ **GO_CONDITIONAL maintained 4 連続**

## §12 entry candidate #1: 137A0 Cocolive (= 4 連続)

| 項目 | 値 |
|---|---|
| code / name | 137A0 / Cocolive |
| sector | 情報通信 |
| close | 739 円 |
| score | 0.869 |
| 100 株 entry 資金 | 73,900 円 |
| stop_loss (5%) | 702 円 = max loss 3,695 円 |
| **連続 caveat** | **D20-D23 で 4 連続 rank 1、D26 で 7 連続到達 = HOLD #2 リスク高** |

## §13 actual price confirmation

```bash
set -a; source /Users/bluefire/fire/.env; set +a
FIRE_ENV=staging \
HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 \
HQ_APPROVE_JQUANTS_TOKEN_READ=1 \
.venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
  --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
  --datasets prices --max-days 14 \
  --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
  --sleep-seconds 0.3 --write \
  --output-json /tmp/d23_jq_refresh.json
```

| 候補 | refreshed (5/14) | actual (6/15) | 乖離率 |
|---|---|---|---|
| 137A0 | 739 | ____ | __.__% |
| 7991 (= 機械) | 1,177 | ____ | __.__% |
| 331A0 | 482 | ____ | __.__% |

## §14 liquidity manual check

| 候補 | 板厚 | 出来高 | spread |
|---|---|---|---|
| 137A0 | ____ | ____ | __.__% |
| 7991 | ____ | ____ | __.__% |
| 331A0 | ____ | ____ | __.__% |

## §15 event / earnings check (= 月曜寄付き前)

週末 (= 6/13/6/14) のニュース + 月曜寄付き event 確認:

| 候補 | TDnet | 決算予定 | ニュース |
|---|---|---|---|
| 137A0 | __________ | __________ | __________ |
| 7991 | __________ | __________ | __________ |
| 331A0 | __________ | __________ | __________ |

## §16 final decision

- ☐ enter 137A0 (= 4 連続 #1、ただし 7 連続まで残 3 日)
- ☐ **enter 7991 マミヤ (= 機械、多様化推奨)** ★
- ☐ enter 331A0
- ☐ watch / skip

決定理由: __________________________

## §17 D23 review 必須 5 項目

1. ★ §1 記入時刻 + entry 銘柄
2. ★ §2 entry 価格 / 株数 / 時刻
3. ★ §3 総 PnL
4. ★ §5 Reason for entry / skip
5. ★ §6 final decision

## §18 D23 paper PnL handoff

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-15 --evaluation-date 2026-06-15 \
  --f111-real-batch-json /tmp/fire_d23_prep/d23_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-15_manual_live_pilot_review.md \
  --output-json /tmp/fire_d23_prep/d23_paper_pnl_ledger_eod.json
```

## §19 stage 3 突合

| 項目 | 当日 | D1-D23 累積 |
|---|---|---|
| trade 回数 | 0-1 | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| **GO_CONDITIONAL 連続** | 4 日 (D20-D23) | 4 days |
| 137A0/7991/331A0 連続 | 4 日連続 | 4 days |
| **137A0 rank 1 連続** | **4 日 (= 7 連続まで残 3 日)** | 4 days |
| 340A0 demote 維持 | 8 連続 (D16-D23) | 8 days |
| 3798 demote 維持 | 4 連続 | 4 days |

## §20 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D23 = post-recovery 4 連続 maintained /
  137A0 4 連続 = HOLD #2 リスク (= 7 連続まで残 3 日、D24 demote 検討必須化)
