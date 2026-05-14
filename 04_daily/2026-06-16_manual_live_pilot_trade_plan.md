---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-16
owner: BlueFire7777 (Fujiwara)
pilot_day: D24
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
post_recovery_status: "D20-D24 で 5 連続 GO_CONDITIONAL maintained"
hard_check_chain_level_passed: 9/12
w3_recovery_criteria_passed: 9/9 (= #7 朝確認待ち)
d23_review_gap_status: unresolved (= 0/5 記入)
new_top_persistence: "137A0/7991/331A0 D20-D24 5 連続安定"
hold_2_warning: "137A0 rank 1 5 連続 = warning 段階到達 (= W3 設計 doc §3.3 #2a)"
demote_simulation_result: "137A0 demote sim 完了 = 新 top 7991 (機械) / 9130 (運輸) / 331A0 (情報通信) で sector 3 種多様化"
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D24_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D24_results.md
  - 04_daily/2026-06-16_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
artifacts:
  f111: /tmp/fire_d24_prep/d24_f111_real_batch.json
  f062: /tmp/fire_d24_prep/d24_f062_preview.json
  morning_line_material: /tmp/fire_d24_prep/after_r1/morning_line_material_2026-06-16.json
  paper_pnl_ledger: /tmp/fire_d24_prep/d24_paper_pnl_ledger.json
  demote_sim_f111: /tmp/fire_d24_prep/demote_sim/d24_137A0_demote_sim_f111.json
  demote_sim_morning: /tmp/fire_d24_prep/demote_sim/after_r1/morning_line_material_2026-06-16.json
---

# Manual Live Pilot — Trade Plan (2026-06-16 / D24)

## D24 特記: **137A0 5 連続 warning 到達 + demote sim 完了 + sector 3 種多様化 path 確立**

- **D20-D24 で 5 連続 GO_CONDITIONAL maintained**
- **137A0 rank 1 が D20-D24 で 5 連続 = warning 段階到達** (= W3 §3.3 #2a)
- **137A0 demote sim 完了**: 新 top 候補 = **7991 (機械) / 9130 (運輸) / 331A0 (情報通信)** = **sector 3 種多様化**
- HOLD #2 まで残 2 日 (= D26 で 7 連続)
- **D25 で 137A0 demote 実施推奨**、または W4 集約で正式化

---

## §1 基本情報

- date: **2026-06-16 (火 / D24)**
- 記入時刻: __:__ JST
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489,340A0,3798 --max-candidates 20`
- demoted_count: 5 (= D20-D24 維持)
- pilot_judgment: **GO_CONDITIONAL maintained 5 連続**
- ⚠ **137A0 5 連続 warning 段階**

## §2 D20-D23 GO_CONDITIONAL recap

| day | pilot | 137A0 rank 1 連続 |
|---|---|---|
| D20 | 復帰 (初) | 1 |
| D21 | 2 連続 | 2 |
| D22 | 3 連続 | 3 |
| D23 | 4 連続 | 4 |
| **D24** | **5 連続 = warning** | **5** |

## §3 D23 review 確認 (= 0/5 記入)

→ W3 §7.1 path 2 継続適用。

## §4 D23 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown
- output: `/tmp/fire_d23_prep/d23_paper_pnl_ledger_with_review_d24_check.json`

## §5 recently_seen_codes 維持 (= 現行)

- 8747, 5729, 3489, 340A0, 3798 (= 5 件、demoted_count=5、D20 から維持)

## §6 ⚠ 137A0 5 連続 warning 段階到達

| day | 137A0 rank 1 連続 | HOLD #2 まで残日数 | 段階 |
|---|---|---|---|
| D20-D23 | 1-4 連続 | 6-3 日 | 監視 |
| **D24** | **5 連続** | **2 日** | **warning ✓** |
| D25 (予想) | 6 連続 | 1 日 | demote 強推奨 |
| D26 (予想) | 7 連続 | 0 日 | **HOLD #2 再発動** |

## §7 137A0 demote sim 結果 (= 参考、本 wave 内 read-only 実施)

### 7.1 cmd (= 参考実行、本 D24 trade plan には未適用)

```bash
.venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
  --base-date 2026-06-16 --max-candidates 20 \
  --label-threshold-mode strict \
  --recently-seen-codes 8747,5729,3489,340A0,3798,**137A0** \
  --output-json /tmp/fire_d24_prep/demote_sim/d24_137A0_demote_sim_f111.json
```

### 7.2 demote sim 後の予想 top (= 137A0 demote sim)

| rank | code | name | sector | label |
|---|---|---|---|---|
| 1 | **7991** | **マミヤ・オーピー** | **機械** | boost_with_caution ✓ 新 #1 |
| 2 | **9130** | **共栄タンカー** | **運輸・物流** | boost_with_caution ✓ 新 #2 (= 運輸 多様化) |
| 3 | 331A0 | メディックス | 情報通信 | boost_with_caution |

→ **sector 3 種多様化** (= 機械 + 運輸 + 情報通信) ← 重要成果

### 7.3 demote 判断

| 案 | 内容 | 推奨度 |
|---|---|---|
| A. D24 内即 demote | trade plan を即更新、D24 actual entry は demote sim top で | ★ 当日変更影響大 |
| **B. D25 で demote 実施** | **D24 = warning 認識、D25 で正式 demote、D25 chain で新 top 採用** | **★★★ 推奨** |
| C. W4 集約で正式化 | D24/D25 は GO_CONDITIONAL 維持、D26 (= 7 連続) HOLD 直前で W4 集約 | ★★ HOLD 再発リスクで非推奨 |

→ **案 B (= D25 で demote 実施) 推奨**

## §8 D24 entry 候補 (= 現行 recently_seen 5 件、137A0/7991/331A0 5 連続)

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| 1 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution ★ #1 (= 5 連続 warning) |
| 2 | **7991** | **マミヤ・オーピー** | **機械** | 1,177 | 5,885 | boost_with_caution ★ 多様化 (= 5 連続) |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution (= 5 連続) |

## §9 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D24 当日 = **2026-06-16 (火)** → **29 営業日 gap**
- 朝再 refresh + iSPEED actual confirmation 必須

## §10 D24 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §11 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

→ **GO_CONDITIONAL maintained 5 連続**、ただし **137A0 warning 段階**

## §12 entry candidate #1: 137A0 Cocolive

| 項目 | 値 |
|---|---|
| code / name | 137A0 / Cocolive |
| sector | 情報通信 |
| close | 739 円 |
| score | 0.869 |
| 100 株 entry 資金 | 73,900 円 |
| stop_loss (5%) | 702 円 = max loss 3,695 円 |
| **連続 caveat** | **5 連続 warning、D26 で 7 連続 HOLD #2 リスク** |

## §13 alt candidate #2: 7991 マミヤ・オーピー (= 推奨)

| 項目 | 値 |
|---|---|
| code / name | 7991 / マミヤ・オーピー |
| sector | **機械** (= sector 多様化) |
| close | 1,177 円 |
| score | 0.867 |
| 100 株 entry 資金 | 117,700 円 |
| stop_loss (5%) | 1,118 円 = max loss 5,885 円 |
| **推奨理由** | **D25 で 137A0 demote 後の新 #1 候補、D24 で先行 entry 価値高い** |

## §14 actual price confirmation

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
  --output-json /tmp/d24_jq_refresh.json
```

| 候補 | refreshed (5/14) | actual (6/16) | 乖離率 |
|---|---|---|---|
| 137A0 | 739 | ____ | __.__% |
| 7991 (= 機械、推奨) | 1,177 | ____ | __.__% |
| 331A0 | 482 | ____ | __.__% |

## §15 liquidity manual check

| 候補 | 板厚 | 出来高 | spread |
|---|---|---|---|
| 137A0 | ____ | ____ | __.__% |
| 7991 | ____ | ____ | __.__% |
| 331A0 | ____ | ____ | __.__% |

## §16 event / earnings check

| 候補 | TDnet | 決算予定 | ニュース |
|---|---|---|---|
| 137A0 | __________ | __________ | __________ |
| 7991 | __________ | __________ | __________ |
| 331A0 | __________ | __________ | __________ |

## §17 final decision

- ☐ **enter 7991 マミヤ (= 機械、多様化、推奨 #1)** ★★★ 推奨 (= D25 demote 前先行 entry)
- ☐ enter 137A0 (= 5 連続 #1、ただし warning 段階)
- ☐ enter 331A0
- ☐ watch / skip

決定理由: __________________________

## §18 D24 review 必須 5 項目

1. ★ §1 記入時刻 + entry 銘柄
2. ★ §2 entry 価格 / 株数 / 時刻
3. ★ §3 総 PnL
4. ★ §5 Reason for entry / skip
5. ★ §6 final decision

## §19 D24 paper PnL handoff

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-16 --evaluation-date 2026-06-16 \
  --f111-real-batch-json /tmp/fire_d24_prep/d24_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-16_manual_live_pilot_review.md \
  --output-json /tmp/fire_d24_prep/d24_paper_pnl_ledger_eod.json
```

## §20 D25 demote 推奨手順 (= 次 wave)

D25 wave で recently_seen に 137A0 追加:
- recently_seen_codes: 8747, 5729, 3489, 340A0, 3798, **137A0** (= 6 件)
- demoted_count: 5 → 6
- 期待新 top: 7991 (機械) / 9130 (運輸) / 331A0 (情報通信) = sector 3 種多様化

## §21 stage 3 突合

| 項目 | 当日 | D1-D24 累積 |
|---|---|---|
| trade 回数 | 0-1 | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| **GO_CONDITIONAL 連続** | 5 日 (D20-D24) | 5 days |
| **137A0 5 連続 warning** | ✓ (= 初) | 5 days |
| **137A0 demote sim 完了** | ✓ (= 初、参考 chain) | 1 day |
| sector 3 種多様化 path | sim で確認 | 1 day |
| 340A0 demote 維持 | 9 連続 (D16-D24) | 9 days |
| 3798 demote 維持 | 5 連続 | 5 days |

## §22 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D24 = 5 連続 maintained + 137A0 warning + demote sim 完了 /
  D25 で 137A0 demote 実施推奨
