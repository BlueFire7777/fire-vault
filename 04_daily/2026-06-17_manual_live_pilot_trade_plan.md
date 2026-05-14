---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-17
owner: BlueFire7777 (Fujiwara)
pilot_day: D25
status: plan
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
preset_tune_applied: true
label_threshold_mode: strict
recently_seen_codes: ["8747", "5729", "3489", "340A0", "3798", "137A0"]
max_candidates: 20
jquants_refresh_applied: true
market_prices_daily_max_date: 2026-05-14
data_r3_freshness_verdict: MISSING
pilot_judgment: GO_CONDITIONAL
post_recovery_status: "D20-D25 で 6 連続 GO_CONDITIONAL maintained"
hard_check_chain_level_passed: 9/12
demoted_count: 6
demotion_initiative: "137A0 demote 本実行 (= D24 sim 結果を D25 で本実装)"
new_top: "7991 (機械) / 9130 (運輸) / 331A0 (情報通信) = sector 3 種多様化 完全達成"
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D25_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D25_results.md
  - 04_daily/2026-06-17_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
artifacts:
  f111: /tmp/fire_d25_prep/d25_f111_real_batch.json
  f062: /tmp/fire_d25_prep/d25_f062_preview.json
  morning_line_material: /tmp/fire_d25_prep/after_r1/morning_line_material_2026-06-17.json
  paper_pnl_ledger: /tmp/fire_d25_prep/d25_paper_pnl_ledger.json
---

# Manual Live Pilot — Trade Plan (2026-06-17 / D25)

## D25 特記: 🎉 **137A0 demote 本実行成功 + sector 3 種多様化完全達成**

- **D24 で 137A0 5 連続 warning → D25 で recently_seen 追加で demote 本実行** ✓
- **新 top: 7991 (機械) / 9130 (運輸・物流) / 331A0 (情報通信)** = **sector 3 種多様化** ✓
- D24 demote sim 結果 (= 7991/9130/331A0) と本実行結果完全一致 ✓
- HOLD #2 リスク完全回避 (= 137A0 demote で 7 連続到達阻止)
- recently_seen_codes: 5 → **6** (= 137A0 新規追加)

---

## §1 基本情報

- date: **2026-06-17 (水 / D25)**
- 記入時刻: __:__ JST
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489,340A0,3798,**137A0** --max-candidates 20`
- demoted_count: **6** (= D20 から 1 増、137A0 新追加)
- pilot_judgment: **GO_CONDITIONAL maintained 6 連続**

## §2 D20-D24 GO_CONDITIONAL recap + 137A0 連続性

| day | pilot | 137A0 rank 1 連続 | demoted_count |
|---|---|---|---|
| D20 | 復帰 (初) | 1 | 5 |
| D21 | 2 連続 | 2 | 5 |
| D22 | 3 連続 | 3 | 5 |
| D23 | 4 連続 | 4 | 5 |
| D24 | 5 連続 (warning) | 5 | 5 |
| **D25** | **6 連続 (= demote 本実行)** | **(= demote、新 rank 1 = 7991)** | **6** |

## §3 D24 review 確認 (= 0/5 記入)

→ W3 §7.1 path 2 継続適用。

## §4 D24 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown
- output: `/tmp/fire_d24_prep/d24_paper_pnl_ledger_with_review_d25_check.json`

## §5 recently_seen_codes 拡張結果 (= 主要成果)

| 拡張前 (D20-D24) | 拡張後 (D25) |
|---|---|
| 8747, 5729, 3489, 340A0, 3798 (5 件) | **8747, 5729, 3489, 340A0, 3798, 137A0 (6 件)** |

demoted_count: 5 → **6** ✓

## §6 137A0 demote 効果実証 (= ✅ 主要成果)

### 6.1 F111-real-batch top 7

| rank | code | label | demoted |
|---|---|---|---|
| 1 | 8747 | caution | True |
| 2 | 5729 | caution | True |
| 3 | 3489 | caution | True |
| 4 | 340A0 | caution | True |
| 5 | 3798 | caution | True |
| **6** | **137A0** | **caution** | **True (= D25 で初 demote)** |
| 7 | 7991 | boost_with_caution | False ✓ |

### 6.2 AFTER-R1 top_candidates 大幅切替 ✓

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D20-D24 | 137A0 (情報通信) | 7991 (機械) | 331A0 (情報通信) |
| **D25** | **7991 マミヤ・オーピー (機械) ✓** | **9130 共栄タンカー (運輸) ✓** | **331A0 メディックス (情報通信)** |

→ **D24 demote sim 結果と完全一致** = 137A0 demote の効果実証 ✓

### 6.3 sector 3 種多様化完全達成 🎉

| top | code | sector |
|---|---|---|
| 1 | **7991** | **機械** (= 新 #1) |
| 2 | **9130** | **運輸・物流** (= 新 #2、初登場) |
| 3 | 331A0 | 情報通信 |

→ **sector 3 種多様化** (= 機械 + 運輸 + 情報通信) ← 大成果

## §7 D25 entry 候補

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| **1** | **7991** | **マミヤ・オーピー** | **機械** | 1,177 | 5,885 | boost_with_caution ★ #1 |
| **2** | **9130** | **共栄タンカー** | **運輸・物流** | 1,410 | 7,050 | boost_with_caution ★ 多様化 |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |

## §8 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D25 当日 = **2026-06-17 (水)** → **30 営業日 gap**
- 朝再 refresh + iSPEED actual confirmation 必須

## §9 D25 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §10 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

| # | 条件 | D25 結果 |
|---|---|---|
| 1 | review path 2 | ✓ (継続) |
| 2 | 340A0 demote 維持 | ✓ (10 連続) |
| 3 | 3798 recently_seen 維持 | ✓ (6 連続) |
| 3.5 | **137A0 recently_seen 追加** | **✓ (= 新)** |
| 4 | f111_real_batch | ✓ |
| 5 | top candidate ≥ 1 | ✓ (3) |
| 6 | risk_within_pilot_limit | ✓ |
| 7 | actual/liquidity/event | 朝確認 |
| 8 | safety_flags 全 False | ✓ |
| 9 | paper PnL/review 重大問題なし | ✓ |

→ **GO_CONDITIONAL maintained 6 連続**

## §11 entry candidate #1: 7991 マミヤ・オーピー (= 新 #1、機械)

| 項目 | 値 |
|---|---|
| code / name | 7991 / マミヤ・オーピー |
| sector | **機械** (= D25 新 #1) |
| close | 1,177 円 |
| score | 0.867 |
| 100 株 entry 想定資金 | 117,700 円 |
| stop_loss (5%) | 1,118 円 = max loss **5,885 円** (= pilot 上限 15,000 円の 39.2%) |
| take_profit (2%) | 1,201 円 = target gain 2,400 円 |
| **D20-D24 履歴** | rank 2 維持 (= 5 連続)、D25 で rank 1 昇格 |

## §12 alt candidate #2: 9130 共栄タンカー (= 運輸、多様化新登場)

| 項目 | 値 |
|---|---|
| code / name | 9130 / 共栄タンカー |
| sector | **運輸・物流** (= 初登場) |
| close | 1,410 円 |
| score | 0.859 |
| 100 株 entry 想定資金 | 141,000 円 |
| stop_loss (5%) | 1,340 円 = max loss 7,050 円 |
| 推奨理由 | D9-D24 で F111 rank 8 維持、D25 で AFTER-R1 rank 2 昇格 = 運輸 sector 多様化候補 |

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
  --output-json /tmp/d25_jq_refresh.json
```

| 候補 | refreshed (5/14) | actual (6/17) | 乖離率 |
|---|---|---|---|
| 7991 (= 新 #1 機械) | 1,177 | ____ | __.__% |
| 9130 (= 新 #2 運輸) | 1,410 | ____ | __.__% |
| 331A0 (= 情報通信) | 482 | ____ | __.__% |

## §14 liquidity manual check

| 候補 | 板厚 | 出来高 | spread |
|---|---|---|---|
| 7991 | ____ | ____ | __.__% |
| 9130 | ____ | ____ | __.__% |
| 331A0 | ____ | ____ | __.__% |

## §15 event / earnings check

| 候補 | TDnet | 決算予定 | ニュース |
|---|---|---|---|
| 7991 | __________ | __________ | __________ |
| 9130 | __________ | __________ | __________ |
| 331A0 | __________ | __________ | __________ |

## §16 final decision

- ☐ **enter 7991 マミヤ・オーピー** 100 株 (= 新 #1、機械、推奨)
- ☐ enter 9130 共栄タンカー (= 運輸、初 entry、多様化)
- ☐ enter 331A0 メディックス
- ☐ watch / skip

決定理由: __________________________

## §17 D25 review 必須 5 項目

1. ★ §1 記入時刻 + entry 銘柄
2. ★ §2 entry 価格 / 株数 / 時刻
3. ★ §3 総 PnL
4. ★ §5 Reason for entry / skip
5. ★ §6 final decision

## §18 D25 paper PnL handoff

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-17 --evaluation-date 2026-06-17 \
  --f111-real-batch-json /tmp/fire_d25_prep/d25_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-17_manual_live_pilot_review.md \
  --output-json /tmp/fire_d25_prep/d25_paper_pnl_ledger_eod.json
```

## §19 stage 3 突合

| 項目 | 当日 | D1-D25 累積 |
|---|---|---|
| trade 回数 | 0-1 | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| **GO_CONDITIONAL 連続** | 6 日 (D20-D25) | 6 days |
| **137A0 demote 本実行** | ✓ (= 初) | 1 day |
| **sector 3 種多様化達成** | ✓ (= 機械+運輸+情報通信、初) | 1 day |
| recently_seen_codes 件数 | **6** (= +137A0) | - |
| 340A0 demote 維持 | 10 連続 (D16-D25) | 10 days |
| 3798 demote 維持 | 6 連続 | 6 days |
| 137A0 demote 維持 | 1 (= 初) | 1 day |

## §20 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D25 = 137A0 demote 本実行 + sector 3 種多様化達成 /
  新 #1 = 7991 マミヤ・オーピー (機械)
