---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-04
owner: BlueFire7777 (Fujiwara)
pilot_day: D16
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
  "#1_review_missing_6_consecutive": "未解消 (= D14 review 0/6 記入)"
  "#2_same_candidate_7_consecutive": "解消済 (= 340A0 demote、新 top = 3798/137A0/331A0)"
hard_check_chain_level_passed: 9/12
d14_review_gap_status: unresolved (= 0/6 記入)
demotion_effect: 340A0 → caution (rank 1 → 4)、new top = 3798/137A0/331A0
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D16_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D16_results.md
  - 04_daily/2026-06-04_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
artifacts:
  f111: /tmp/fire_d16_prep/d16_f111_real_batch.json
  f062: /tmp/fire_d16_prep/d16_f062_preview.json
  morning_line_material: /tmp/fire_d16_prep/after_r1/morning_line_material_2026-06-04.json
  paper_pnl_ledger: /tmp/fire_d16_prep/d16_paper_pnl_ledger.json
  d14_paper_pnl_with_review: /tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d16_check.json
---

# Manual Live Pilot — Trade Plan (2026-06-04 / D16)

## D16 特記: **340A0 demote 成功 + review_gap 未解消 day**

- **340A0 demote 成功**: recently_seen_codes 拡張で 340A0 → caution、新 top = **3798/137A0/331A0**
- **D14 review 未記入**: 0/6 必須項目記入、review_gap_unresolved (= HOLD #1 残存)
- **HOLD maintained**: 設計 doc §3.3 #1 のみ未解消、#2 は demote で解消済
- **D17 解除可能性高**: Fujiwara が D14 review 記入すれば HOLD 完全解除 path 開放

---

## §1 基本情報

- date: **2026-06-04 (木 / D16)**
- 記入時刻: __:__ JST
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489,**340A0** --max-candidates 20`
- demoted_count: **4** (= 8747/5729/3489 + 340A0 新追加)
- pilot_judgment: **HOLD maintained**

## §2 D15 HOLD 発動 recap

- D15 で設計 doc §3.3 HOLD 条件 #1 + #2 同時発動
- HOLD 解除 path:
  1. D14 review 必須 5 項目記入 (= Fujiwara 本人) → HOLD #1 解消
  2. recently_seen に 340A0 追加 (= 本 wave 実施) → HOLD #2 解消

## §3 D14 review 確認結果 (= review_gap_unresolved)

| 項目 | 値 |
|---|---|
| yaml status | blank |
| §1 記入時刻 | BLANK |
| §1 ticker | BLANK |
| §2 entry 価格 actual | BLANK |
| §3 総 PnL | BLANK |
| §5 Reason 1 文 | BLANK |
| §6 final decision ☑ | BLANK |

→ **0/6 記入** = review_gap_unresolved → HOLD #1 残存。

## §4 D14 paper PnL --review-md 再 run 結果

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: final_decision=unknown / is_blank=True
- warnings: ["review_missing"]
- output: `/tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d16_check.json`

→ D15 wave と同状態、D14 review が記入されない限り same。

## §5 recently_seen_codes 拡張結果

| 拡張前 (D15 まで) | 拡張後 (D16) |
|---|---|
| 8747, 5729, 3489 | **8747, 5729, 3489, 340A0** |

demoted_count: 3 → **4** ✓

## §6 340A0 demote 効果 (= 主要成果)

### 6.1 F111-real-batch top 4 比較

| day | rank 1-3 (caution) | rank 4 |
|---|---|---|
| D14/D15 | 8747/5729/3489 | 340A0 (boost_with_caution) |
| **D16** | **8747/5729/3489** | **340A0 (caution、demoted)** ✓ |

### 6.2 AFTER-R1 top_candidates 切替

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D14/D15 | 340A0 ジグザグ | 3798 ＵＬＳ | 137A0 Cocolive |
| **D16** | **3798 ＵＬＳ** | **137A0 Cocolive** | **331A0 メディックス (= 新 entry)** |

→ 340A0 demote 効果で **新 top 3 候補確立**。

### 6.3 D16 entry 候補一覧 (= boost_with_caution 維持)

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| **5** | **3798** | **ＵＬＳグループ** | 情報通信 | **505** | **2,525** | **boost_with_caution ★ #1** |
| 6 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution |
| 7 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 | boost_with_caution (= 多様化) |
| 8 | 9130 | 共栄タンカー | 運輸 | 1,410 | 7,050 | boost_with_caution |
| 9 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |

→ 新 entry candidate #1 = **3798 ＵＬＳグループ** (= 旧 #2 が #1 へ昇格)

## §7 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D16 当日 = **2026-06-04 (木)** → **21 営業日 gap** (= D15 20 → 1 日拡大)
- 朝再 refresh + iSPEED actual confirmation 必須 (= D14/D15 と同)

## §8 D16 hard check (= chain-level 9/12)

| # | 項目 | 結果 |
|---|---|---|
| 1 | f111_input_source=f111_real_batch | ✓ |
| 2 | top_candidate ≥ 1 (= 3) | ✓ |
| 3 | risk_within_pilot_limit=True | ✓ (= 全 entry 候補) |
| 4 | sample でない | ✓ |
| 5 | tradable_universe=True | ✓ |
| 6 | forbidden_check.passed=True | ✓ |
| 7 | auto_order_allowed=False | ✓ |
| 8 | manual_review_required=True | ✓ |
| 9 | safety_flags 13 keys all False | ✓ |
| 10 | actual price confirmation | (朝確認待ち) |
| 11 | liquidity/spread/volume | (朝確認待ち) |
| 12 | event/earnings | (朝確認待ち) |

→ chain-level 9/12 ✓、wave 実行時点 HOLD 維持。

## §9 Pilot 判定: **HOLD maintained** (= 設計 doc §3.3 #1 のみ未解消)

### 9.1 HOLD 条件 status

| # | 条件 | D16 状況 | 発動 |
|---|---|---|---|
| 1 | review missing 5 連続超過 | D9-D14 で 6 連続 blank、**未解消** | **✓ 継続** |
| 2 | 同 candidate 7 営業日以上連続 | **340A0 demote で 7 連続中断、新 top に切替** | **✗ 解消済** |
| 3 | actual price 確認できない | 朝確認次第 | 未確認 |
| 4 | liquidity 確認できない | 朝確認次第 | 未確認 |
| 5 | event リスク強い | 朝確認次第 | 未確認 |

→ **5 条件中 1 発動** (= #1 のみ)、HOLD 維持。

### 9.2 HOLD 完全解除 path (= 主推奨)

| step | 内容 |
|---|---|
| 1 | D14 review.md (= 2026-06-02) 必須 5 項目記入 (= Fujiwara 本人) |
| 2 | D14 paper PnL --review-md 付き再 run で review_actuals 反映確認 |
| 3 | D17 (= 2026-06-05 金) で chain 再起 + 判定見直し → **HOLD 解除 → GO_CONDITIONAL** へ |

### 9.3 D16 推奨アクション

**主推奨**: D14 review 記入後、D16 = skip (= entry しない)、D17 で HOLD 解除。

ただし、**alternative**: D16 で 3798 ＵＬＳグループ に少額実弾 entry も可能 (= chain-level 9/12 ✓、340A0 demote で新候補確立)。
ただし HOLD 維持中の override となり、Fujiwara 責任での判断。

## §10 entry candidate #1 (= 参考情報、HOLD 中は entry skip 推奨)

| 項目 | 値 |
|---|---|
| code / name | 3798 / ＵＬＳグループ |
| sector | 情報通信・サービスその他 |
| close (5/14 refreshed) | 505 円 |
| score | 0.877 (A1 rank) |
| label | boost_with_caution (= 条件付き買い推奨 🟡) |
| AFTER-R1 rank | 1 (= 340A0 demote 後の新 #1) |
| 100 株 entry 想定資金 | 50,500 円 |
| stop_loss (5%) | 480 円 = max loss **2,525 円** (= pilot 上限 15,000 円の 16.8%) |
| take_profit (2%) | 515 円 = target gain 1,000 円 |

## §11 actual price confirmation 欄 (= HOLD 中は参考)

```bash
# 朝 J-Quants focused refresh (= 必要なら)
set -a; source /Users/bluefire/fire/.env; set +a
FIRE_ENV=staging \
HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 \
HQ_APPROVE_JQUANTS_TOKEN_READ=1 \
.venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
  --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
  --datasets prices --max-days 14 \
  --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
  --sleep-seconds 0.3 --write \
  --output-json /tmp/d16_jq_refresh.json
```

| 候補 | refreshed (5/14) | actual (6/4) | 乖離率 |
|---|---|---|---|
| 3798 ＵＬＳ (= 新 #1) | 505 | ____ | __.__% |
| 137A0 Cocolive | 739 | ____ | __.__% |
| 331A0 メディックス | 482 | ____ | __.__% |
| 7991 マミヤ・オーピー (= 多様化) | 1,177 | ____ | __.__% |

## §12 liquidity manual check (= HOLD 中は参考)

| 候補 | 板厚 | 出来高 | spread |
|---|---|---|---|
| 3798 | ____ | ____ | __.__% |
| 137A0 | ____ | ____ | __.__% |
| 331A0 | ____ | ____ | __.__% |

## §13 event / earnings check (= HOLD 中は参考)

| 候補 | TDnet | 決算予定 | ニュース |
|---|---|---|---|
| 3798 | __________ | __________ | __________ |
| 137A0 | __________ | __________ | __________ |
| 331A0 | __________ | __________ | __________ |

## §14 final decision (= D16 推奨)

- ☐ **skip (= HOLD 維持、D14 review 記入推奨)** ★ 推奨
- ☐ enter 3798 ＵＬＳグループ 100 株 (= 新 #1、HOLD override、Fujiwara 責任)
- ☐ enter 7991 マミヤ・オーピー 100 株 (= sector 多様化、HOLD override)
- ☐ watch (= 寄付き後再判断)

決定理由: __________________________

## §15 D16 review 必須 5 項目 (= HOLD day も継続要求)

1. ★ §1 記入時刻 + 対応
2. ★ §2 計画 vs 実際 (= "HOLD、entry skip" 想定)
3. ★ §3 PnL (= "0 円 (HOLD)")
4. ★ §5 Reason (= "HOLD #1 残存、D14 review 記入後 D17 解除予定")
5. ★ §6 final decision = skip

## §16 D16 paper PnL handoff 手順

### §16.1 D16 当日 (= 場後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-04 --evaluation-date 2026-06-04 \
  --f111-real-batch-json /tmp/fire_d16_prep/d16_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-04_manual_live_pilot_review.md \
  --output-json /tmp/fire_d16_prep/d16_paper_pnl_ledger_eod.json
```

### §16.2 D17 翌朝 (= 2026-06-05 金)

D14 review 記入確認後、D17 chain で HOLD 解除可能性大。

## §17 stage 3 突合

| 項目 | 当日 | D1-D16 累積 |
|---|---|---|
| trade 回数 | 0 (= HOLD) | __ / 50 |
| 累積 PnL | 0 円 | __ 円 |
| f111_real_batch 比率 | 1/1 | 10/16 |
| top_candidates ≥1 達成率 | 1/1 | 11/16 |
| **340A0 demote 初実施** | ✓ | 1 day (= 初) |
| **HOLD #2 解消** | ✓ | 1 day |
| 340A0 連続選出 | 7 連続 (D9-D15) → demote で中断 | 7 days |
| review missing 連続 | 6 連続 (D9-D14) | 6 days |

## §18 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D16 = HOLD maintained / 340A0 demote 成功 /
  新 top = 3798/137A0/331A0 / D17 で HOLD 完全解除可能性高
