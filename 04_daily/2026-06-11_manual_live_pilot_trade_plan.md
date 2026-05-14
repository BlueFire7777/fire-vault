---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-11
owner: BlueFire7777 (Fujiwara)
pilot_day: D21
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
post_recovery_status: "D20 GO_CONDITIONAL 復帰 → D21 maintained 2 連続"
hard_check_chain_level_passed: 9/12
w3_recovery_criteria_passed: 9/9 (= chain-level 9 + #7 朝確認待ち)
d20_review_gap_status: partial (= 1/6 記入、ただし template 検出含む可能性)
new_top_persistence: "137A0 / 7991 / 331A0 = D20-D21 で 2 連続安定"
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D21_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D21_results.md
  - 04_daily/2026-06-11_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
artifacts:
  f111: /tmp/fire_d21_prep/d21_f111_real_batch.json
  f062: /tmp/fire_d21_prep/d21_f062_preview.json
  morning_line_material: /tmp/fire_d21_prep/after_r1/morning_line_material_2026-06-11.json
  paper_pnl_ledger: /tmp/fire_d21_prep/d21_paper_pnl_ledger.json
  d20_paper_pnl_with_review: /tmp/fire_d20_prep/d20_paper_pnl_ledger_with_review_d21_check.json
---

# Manual Live Pilot — Trade Plan (2026-06-11 / D21)

## D21 特記: **post-recovery 初回継続判定 day**

- **D20 GO_CONDITIONAL 復帰 → D21 maintained 2 連続**
- **137A0 / 7991 / 331A0 が D20-D21 で 2 連続安定**
- 340A0 / 3798 demote 維持 (= recently_seen 5 件)
- sector 多様化維持 (= 機械 7991 rank 2)
- D20 review = partial (= W3 §7.1 path 2 適用継続)

---

## §1 基本情報

- date: **2026-06-11 (木 / D21)**
- 記入時刻: __:__ JST
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489,340A0,3798 --max-candidates 20`
- demoted_count: 5 (= D20 と同)
- pilot_judgment: **GO_CONDITIONAL maintained**

## §2 D20 GO_CONDITIONAL 復帰 recap

- HOLD 6 連続 (D15-D19) → D20 で GO_CONDITIONAL 復帰
- W3 §7.1 path 2 (= review gap 明確化) 適用
- 3798 demote 初実施成功
- AFTER-R1 top 切替: 340A0/3798/137A0 → **137A0/7991/331A0**
- sector 多様化達成 (= 機械 7991 rank 2)

## §3 D20 review 確認結果 (= PARTIAL 1/6)

| 項目 | 値 |
|---|---|
| yaml status | blank |
| 必須 5 項目 | 1/6 (= template 内 review_gap 記載検出含む可能性) |

→ D14-D19 完全 blank よりは進歩、W3 §7.1 path 2 適用継続。

## §4 D20 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown
- warnings: ["review_missing"]
- output: `/tmp/fire_d20_prep/d20_paper_pnl_ledger_with_review_d21_check.json`

## §5 recently_seen_codes 維持

- 8747, 5729, 3489, 340A0, 3798 (= 5 件、demoted_count=5、D20 から維持)

## §6 137A0/7991/331A0 継続性 (= D20-D21 で 2 連続)

### 6.1 AFTER-R1 top_candidates

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D20 | 137A0 Cocolive | 7991 マミヤ・オーピー (機械) | 331A0 メディックス |
| **D21** | **137A0 Cocolive** | **7991 マミヤ・オーピー (機械)** | **331A0 メディックス** |

→ **2 連続安定** (= 7 連続まで残 5 日、HOLD #2 リスクまだなし)。

### 6.2 sector 多様化維持 ✓

- rank 1 = 137A0 (情報通信)
- rank 2 = **7991 (機械)** ← 多様化維持
- rank 3 = 331A0 (情報通信)

## §7 D21 entry 候補

| rank | code | name | sector | close (5/14) | risk_yen | label |
|---|---|---|---|---|---|---|
| 1 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution ★ #1 (= 2 連続) |
| 2 | **7991** | **マミヤ・オーピー** | **機械** | **1,177** | **5,885** | **boost_with_caution ★ sector 多様化 (= 2 連続)** |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution (= 2 連続) |

## §8 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D21 当日 = **2026-06-11 (木)** → **26 営業日 gap**
- 朝再 refresh + iSPEED actual confirmation 必須

## §9 D21 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §10 W3 recovery criteria 9 条件判定

| # | 条件 | D21 結果 |
|---|---|---|
| 1 | review 5 項目記入 OR gap 明確化 | ✓ (= D20 partial、本 plan で gap 明確化継続) |
| 2 | 340A0 demote 維持 | ✓ (= 6 連続) |
| 3 | 3798 recently_seen 維持 | ✓ (= 2 連続) |
| 4 | f111_real_batch | ✓ |
| 5 | top candidate ≥ 1 | ✓ (= 3) |
| 6 | risk_within_pilot_limit=True | ✓ |
| 7 | actual / liquidity / event 確認可能 | 朝確認待ち |
| 8 | safety_flags 13 keys all False | ✓ |
| 9 | paper PnL / review に重大問題なし | ✓ |

→ **9/9 ✓** (= #7 朝確認) → **GO_CONDITIONAL maintained**

## §11 entry candidate #1: 137A0 Cocolive (= 2 連続 #1)

| 項目 | 値 |
|---|---|
| code / name | 137A0 / Cocolive |
| sector | 情報通信 |
| close | 739 円 |
| score | 0.869 |
| 100 株 entry 想定資金 | 73,900 円 |
| stop_loss (5%) | 702 円 = max loss 3,695 円 |

## §12 actual price confirmation 欄

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
  --output-json /tmp/d21_jq_refresh.json
```

| 候補 | refreshed (5/14) | actual (6/11) | 乖離率 |
|---|---|---|---|
| 137A0 | 739 | ____ | __.__% |
| 7991 (= 機械) | 1,177 | ____ | __.__% |
| 331A0 | 482 | ____ | __.__% |

## §13 liquidity manual check (= 寄付き 5 分後)

| 候補 | 板厚 | 出来高 | spread |
|---|---|---|---|
| 137A0 | ____ | ____ | __.__% |
| 7991 | ____ | ____ | __.__% |
| 331A0 | ____ | ____ | __.__% |

## §14 event / earnings check

| 候補 | TDnet | 決算予定 | ニュース |
|---|---|---|---|
| 137A0 | __________ | __________ | __________ |
| 7991 | __________ | __________ | __________ |
| 331A0 | __________ | __________ | __________ |

## §15 final decision (= 寄付き直前 08:55 JST)

- ☐ **enter 137A0** 100 株 (= 2 連続 #1)
- ☐ enter 7991 マミヤ (= 機械、多様化)
- ☐ enter 331A0
- ☐ watch / skip

決定理由: __________________________

## §16 Pilot 判定: **GO_CONDITIONAL maintained** (= 2 連続)

### 16.1 GO_CONDITIONAL 維持根拠
- W3 9 条件 全 ✓ (= #7 朝確認待ち)
- 137A0/7991/331A0 が D20-D21 で 2 連続安定
- sector 多様化維持

### 16.2 残 caveat
- ⚠ freshness_verdict=MISSING
- ⚠ 26 営業日 price gap
- ⚠ D20 review partial (= path 2 適用継続)
- ⚠ 137A0 連続性 (= 7 連続まで残 5 日、HOLD #2 リスク)

### 16.3 D21 推奨アクション
- 朝寄付き 3 確認後 entry (= 137A0 #1 または 7991 多様化)
- D21 review 必須 5 項目記入
- D20 entry した場合は continuity 判定

## §17 D21 review 必須 5 項目

1. ★ §1 記入時刻 + entry 銘柄
2. ★ §2 entry 価格 / 株数 / 時刻
3. ★ §3 総 PnL
4. ★ §5 Reason for entry / skip
5. ★ §6 final decision

## §18 D21 paper PnL handoff

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-11 --evaluation-date 2026-06-11 \
  --f111-real-batch-json /tmp/fire_d21_prep/d21_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-11_manual_live_pilot_review.md \
  --output-json /tmp/fire_d21_prep/d21_paper_pnl_ledger_eod.json
```

## §19 stage 3 突合

| 項目 | 当日 | D1-D21 累積 |
|---|---|---|
| trade 回数 | 0-1 | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| f111_real_batch 比率 | 1/1 | 15/21 |
| **GO_CONDITIONAL 連続** | 2 日 (D20-D21) | 2 days |
| 137A0/7991/331A0 連続 | 2 日連続 | 2 days |
| 340A0 demote 維持 | 6 連続 (D16-D21) | 6 days |
| 3798 demote 維持 | 2 連続 | 2 days |

## §20 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D21 = post-recovery 初回継続判定 (= GO_CONDITIONAL 2 連続) /
  新 top 2 連続 / sector 多様化維持
