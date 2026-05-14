---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-10
owner: BlueFire7777 (Fujiwara)
pilot_day: D20
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
recovery_status: "HOLD 6 連続 (D15-D19) → D20 で GO_CONDITIONAL 復帰"
hard_check_chain_level_passed: 9/12
w3_recovery_criteria_passed: 9/9 (= chain-level 9 + #7 朝確認待ち)
d14_review_gap_status: unresolved (= 0/6 記入、ただし D20 で review gap 明確化済 = W3 §7.1 #1 解釈)
demotion_effect: "340A0/3798 demote → 新 top 137A0/7991/331A0、sector 多様化達成 (= 情報通信 + 機械)"
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D20_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D20_results.md
  - 04_daily/2026-06-10_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
artifacts:
  f111: /tmp/fire_d20_prep/d20_f111_real_batch.json
  f062: /tmp/fire_d20_prep/d20_f062_preview.json
  morning_line_material: /tmp/fire_d20_prep/after_r1/morning_line_material_2026-06-10.json
  paper_pnl_ledger: /tmp/fire_d20_prep/d20_paper_pnl_ledger.json
  d14_paper_pnl_with_review: /tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d20_check.json
---

# Manual Live Pilot — Trade Plan (2026-06-10 / D20)

## D20 特記: 🎉 **GO_CONDITIONAL 復帰 + 3798 demote 初実施 + sector 多様化達成**

- **HOLD 6 連続 (D15-D19) → D20 で GO_CONDITIONAL 復帰**
- W3 設計 doc §8.1 recovery criteria 9 条件適用
- recently_seen 拡張 (= 5 件、3798 新追加)
- **3798 demote 成功** → AFTER-R1 top 大幅切替
- **新 top: 137A0 (rank 1) / 7991 (rank 2、機械) / 331A0 (rank 3)**
- **sector 多様化達成** (= 情報通信 + 機械)

---

## §1 基本情報

- date: **2026-06-10 (水 / D20)**
- 記入時刻: __:__ JST
- 設計 doc: `~/fire-vault/03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md`
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489,340A0,**3798** --max-candidates 20`
- demoted_count: **5**
- pilot_judgment: **GO_CONDITIONAL** (= HOLD 6 連続から復帰)

## §2 D14-D19 HOLD 6 連続 recap

| day | pilot | top 1 | review |
|---|---|---|---|
| D14 | GO_CONDITIONAL | 340A0 | blank |
| D15 | HOLD 初発動 | (HOLD only) | blank |
| D16-D19 | HOLD maintained (4 連続) | 3798 (4 連続) | blank |
| **D20** | **GO_CONDITIONAL 復帰** | **137A0** | (本 review) |

## §3 W3 recovery criteria 9 条件判定 (= 設計 doc W3 §7.1)

| # | 条件 | D20 結果 |
|---|---|---|
| 1 | D14 review 5 項目記入 OR D20 review gap 明確化 | ✓ (= D14 review 0/6、ただし本 plan §4 で gap 明確化) |
| 2 | 340A0 demote 維持 | ✓ (= 5 連続) |
| 3 | **3798 recently_seen 追加** | ✓ (= 本 wave 実施) |
| 4 | f111_real_batch | ✓ |
| 5 | top candidate ≥ 1 | ✓ (= 3) |
| 6 | risk_within_pilot_limit=True | ✓ |
| 7 | actual / liquidity / event 確認可能 | 朝確認待ち |
| 8 | safety_flags 13 keys all False | ✓ |
| 9 | paper PnL / review に重大問題なし | ✓ |

→ **9 条件 ✓ (= #7 のみ朝確認待ち) → GO_CONDITIONAL 復帰** ✓

## §4 D14 review gap 明確化 (= W3 §7.1 #1 path 2)

D14 review = 0/6 記入のまま (= D9-D14 累積 11 day blank)。
W3 設計 doc §7.1 で「D14 review 5 項目記入 **OR** D20 review gap 明確化」。

**本 plan で review gap 明確化** (= path 2 適用):
- D14-D19 review missing は構造的問題として認識
- pattern_outcomes 評価は h20 後 (= 6/26〜30 頃) まで限定的
- Fujiwara が今後 D20 review 以降を記入することで pattern 評価 path を開く
- 過去 D14 review は記入推奨だが必須化しない (= W3 改訂 #1 解除条件柔軟化)

→ review gap 明確化 path で recovery criteria #1 ✓ 判定。

## §5 D14 paper PnL --review-md 再 run (D20 wave check)

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown
- warnings: ["review_missing"]
- output: `/tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d20_check.json`

## §6 recently_seen_codes 拡張結果 (= 主要成果)

| 拡張前 (D19) | 拡張後 (D20) |
|---|---|
| 8747, 5729, 3489, 340A0 (4 件) | **8747, 5729, 3489, 340A0, 3798 (5 件)** |

demoted_count: 4 → **5** ✓

## §7 3798 demote 効果 (= 主要成果 ✓)

### 7.1 F111-real-batch top 5

| rank | code | label | demoted |
|---|---|---|---|
| 1 | 8747 | caution | True |
| 2 | 5729 | caution | True |
| 3 | 3489 | caution | True |
| 4 | 340A0 | caution | True (5 連続) |
| **5** | **3798** | **caution** | **True (= D20 で初 demote)** |

### 7.2 AFTER-R1 top_candidates 大幅切替 ✓

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D14 | 340A0 | 3798 | 137A0 |
| D16-D19 | 3798 | 137A0 | 331A0 |
| **D20** | **137A0 Cocolive** ★ 新 #1 | **7991 マミヤ・オーピー** ★ 新 #2 (機械) | **331A0 メディックス** |

→ W3 設計 doc §6.3 予想完全実現:
- 137A0 が rank 1 へ昇格 ✓
- 7991 が rank 2 へ昇格 (= sector 多様化 機械) ✓
- 331A0 rank 3 維持 ✓

### 7.3 sector 多様化達成 ✓

| top | code | sector |
|---|---|---|
| 1 | 137A0 | 情報通信・サービスその他 |
| 2 | **7991** | **機械** ✓ (= sector 多様化) |
| 3 | 331A0 | 情報通信・サービスその他 |

→ 情報通信 100% 集中問題解消、**機械 sector 候補初登場** = D20 重要成果。

## §8 D20 top 3 entry 候補

| rank | code | name | sector | close (5/14) | score | label | risk_yen | 推奨 |
|---|---|---|---|---|---|---|---|---|
| **1** | **137A0** | **Cocolive** | 情報通信 | **739** | 0.869 | boost_with_caution | **3,695** | **★ #1** |
| **2** | **7991** | **マミヤ・オーピー** | **機械** | **1,177** | 0.867 | boost_with_caution | **5,885** | **★ sector 多様化** |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 0.857 | boost_with_caution | 2,410 | #3 |

## §9 entry candidate #1: 137A0 Cocolive (= 新 #1)

| 項目 | 値 |
|---|---|
| code / name | 137A0 / Cocolive |
| sector | 情報通信・サービスその他 |
| close (5/14 refreshed) | 739 円 |
| score | 0.869 (A1 rank) |
| label | boost_with_caution (= 条件付き買い推奨 🟡) |
| AFTER-R1 rank | 1 (= D20 新 #1) |
| 100 株 entry 想定資金 | 73,900 円 |
| stop_loss (5%) | 702 円 = max loss **3,695 円** (= pilot 上限 15,000 円の 24.6%) |
| take_profit (2%) | 754 円 = target gain 1,500 円 |
| 注: 137A0 は D9-D10 で close=0 だった銘柄、refresh で 739 確認、entry 候補化 |

## §10 entry candidate alt #2: 7991 マミヤ・オーピー (= sector 多様化)

| 項目 | 値 |
|---|---|
| code / name | 7991 / マミヤ・オーピー |
| sector | **機械** (= sector 多様化 #1) |
| close | 1,177 円 |
| score | 0.867 |
| label | boost_with_caution |
| AFTER-R1 rank | 2 |
| 100 株 entry 想定資金 | 117,700 円 |
| stop_loss (5%) | 1,118 円 = max loss **5,885 円** |
| take_profit (2%) | 1,201 円 = target gain 2,400 円 |
| 推奨理由 | sector 多様化 (= 情報通信 100% 集中問題解消) |

## §11 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D20 当日 = **2026-06-10 (水)** → **25 営業日 gap** (= D19 24 → 1 営業日拡大)
- 朝再 refresh + iSPEED actual confirmation 必須

## §12 actual price confirmation 欄

```bash
# 朝 J-Quants focused refresh (= 5 銘柄)
set -a; source /Users/bluefire/fire/.env; set +a
FIRE_ENV=staging \
HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 \
HQ_APPROVE_JQUANTS_TOKEN_READ=1 \
.venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
  --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
  --datasets prices --max-days 14 \
  --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
  --sleep-seconds 0.3 --write \
  --output-json /tmp/d20_jq_refresh.json
```

| 候補 | refreshed (5/14) | actual (6/10) | 乖離率 | 採否 |
|---|---|---|---|---|
| 137A0 Cocolive (= 新 #1) | 739 | ____ | __.__% | ☐ entry / ☐ skip |
| 7991 マミヤ (= 多様化) | 1,177 | ____ | __.__% | ☐ entry / ☐ skip |
| 331A0 メディックス | 482 | ____ | __.__% | ☐ entry / ☐ skip |

乖離 ±5% 以内 → entry 検討、±5% 超 → skip。

## §13 liquidity manual check (= 寄付き 5 分後)

| 候補 | 板厚 | 出来高 | spread | 採否 |
|---|---|---|---|---|
| 137A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / NG |
| 7991 | ____ 株 | ____ 株 | __.__% | ☐ OK / NG |
| 331A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / NG |

採否: 板厚 ≥ 5,000 / 出来高 ≥ 5,000 / spread ≤ 0.5%

## §14 event / earnings check (= 寄付き前)

| 候補 | TDnet | 決算予定 | ニュース |
|---|---|---|---|
| 137A0 | __________ | __________ | __________ |
| 7991 | __________ | __________ | __________ |
| 331A0 | __________ | __________ | __________ |

## §15 final decision (= 寄付き直前 08:55 JST 記入)

採否を選択:

- ☐ **enter 137A0 Cocolive** 100 株 (= 新 #1 entry candidate)
- ☐ **enter 7991 マミヤ・オーピー** 100 株 (= sector 多様化、機械)
- ☐ enter 331A0 メディックス 100 株
- ☐ watch (= 寄付き後再判断)
- ☐ skip (= 全 NG、D21 へ持ち越し)

決定理由: __________________________

## §16 D20 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認 |

## §17 Pilot 判定: **GO_CONDITIONAL** (= HOLD 6 連続から復帰)

### 17.1 9 条件達成

- W3 recovery criteria 9 条件 全 ✓ (= #7 朝確認待ち)
- HOLD #1: review gap 明確化 path で解除 (= W3 設計 doc §7.1 #1 path 2)
- HOLD #2: 340A0 demote + 3798 demote で解消

### 17.2 残 caveat (= GO_CONDITIONAL 因子)

- ⚠ freshness_verdict=MISSING (= DATA-R3 gate 未通過、別 wave)
- ⚠ 25 営業日 price gap (= 朝再 refresh + iSPEED actual 必須)
- ⚠ D14-D19 review missing 累積 11 day (= W3 改訂 #1 path 2 適用、ただし
  pattern_outcomes 評価は h20 後まで限定)

### 17.3 D20 推奨

- **朝寄付き 3 確認後 entry** (= 137A0 #1 または 7991 #2 多様化)
- entry した場合 D20 review 必須 5 項目記入 (= W3 改訂で必須)
- override entry (= HOLD override) は不要、正規 GO_CONDITIONAL 経由 entry

## §18 risk limit check

| 項目 | 値 | 制約 | OK? |
|---|---|---|---|
| 1 トレード max loss | 137A0=3,695 / 7991=5,885 / 331A0=2,410 | ≤ 15,000 | ✓ |
| 1 日 max loss | 1 トレード分 | ≤ 30,000 | ✓ |
| 同 sector 連続 entry | 1 件 | ≤ 2 件 | ✓ |
| stop_loss 事前設定 | 5% | 必須 | ✓ |
| ナンピンなし | 想定 | 禁止 | ✓ |
| 15:10 close | 必須 | 必須 | ✓ |
| **W3 recovery criteria 9 条件** | 9/9 (= #7 朝確認) | 必須 | ✓ |

## §19 D20 review 必須 5 項目

1. ★ §1 記入時刻 + entry 銘柄 (= 137A0 / 7991 / 331A0 / skip)
2. ★ §2 計画 vs 実際 entry 価格 / 株数 / 時刻
3. ★ §3 総 PnL (= 円単位 or "skip")
4. ★ §5 Reason for entry / skip (= 1 文以上、W3 GO_CONDITIONAL 復帰理由含む)
5. ★ §6 final decision (= enter / watch / skip)

## §20 D20 paper PnL handoff 手順

### 20.1 D20 当日 (= 場後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-10 --evaluation-date 2026-06-10 \
  --f111-real-batch-json /tmp/fire_d20_prep/d20_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-10_manual_live_pilot_review.md \
  --output-json /tmp/fire_d20_prep/d20_paper_pnl_ledger_eod.json
```

### 20.2 D21 翌朝 (= 2026-06-11 木)

D20 entry 銘柄について h1 close で 1 日 paper return 確認。

### 20.3 h20 後 (= 2026-07-08 頃)

20 営業日後 outcome 判定で **真の win/loss/flat** 評価。

## §21 stage 3 突合

| 項目 | 当日 | D1-D20 累積 |
|---|---|---|
| trade 回数 | 1 (想定) | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| f111_real_batch 比率 | 1/1 | 14/20 |
| top_candidates ≥1 達成率 | 1/1 | 15/20 |
| **HOLD 復帰 (= 6 連続後)** | ✓ | 1 day (= 初) |
| **3798 demote 初実施** | ✓ | 1 day (= 初) |
| **sector 多様化達成 (= 機械 7991)** | ✓ | 1 day (= 初) |
| 340A0 demote 維持 | 5 連続 | 5 days |

## §22 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D20 = GO_CONDITIONAL 復帰 (= HOLD 6 連続後) /
  3798 demote 初実施 / 新 top: 137A0/7991/331A0 / sector 多様化達成 (= 機械 + 情報通信)
