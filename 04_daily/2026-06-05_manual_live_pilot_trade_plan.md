---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-05
owner: BlueFire7777 (Fujiwara)
pilot_day: D17
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
  "#1_review_missing_6_consecutive": "未解消 (= D14 review 0/6 記入、D15/D16 と同)"
  "#2_same_candidate_7_consecutive": "解消継続 (= 340A0 demote 2 日連続)"
hard_check_chain_level_passed: 9/12
d14_review_gap_status: unresolved (= 0/6 記入)
demotion_persistence: "340A0 demote 2 日連続維持、3798 AFTER-R1 rank 1 連続"
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D17_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D17_results.md
  - 04_daily/2026-06-05_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
artifacts:
  f111: /tmp/fire_d17_prep/d17_f111_real_batch.json
  f062: /tmp/fire_d17_prep/d17_f062_preview.json
  morning_line_material: /tmp/fire_d17_prep/after_r1/morning_line_material_2026-06-05.json
  paper_pnl_ledger: /tmp/fire_d17_prep/d17_paper_pnl_ledger.json
  d14_paper_pnl_with_review: /tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d17_check.json
---

# Manual Live Pilot — Trade Plan (2026-06-05 / D17)

## D17 特記: **HOLD maintained 3 日連続 day** (= D14 review 未記入継続)

- **D14 review 0/6 記入** = HOLD #1 残存 (= D15/D16/D17 で 3 日連続 HOLD)
- **340A0 demote 維持** = HOLD #2 解消継続 (= D16 で解消、D17 で 2 日連続)
- **3798 ＵＬＳグループ が AFTER-R1 rank 1 連続** (= D16-D17 で 2 日連続)
- chain-level 9/12 ✓、ただし HOLD 維持で entry 推奨しない

---

## §1 基本情報

- date: **2026-06-05 (金 / D17)**
- 記入時刻: __:__ JST
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489,340A0 --max-candidates 20` (= D16 と同)
- demoted_count: 4 (= 維持)
- pilot_judgment: **HOLD maintained**

## §2 D15/D16 HOLD recap

| day | pilot_judgment | HOLD #1 | HOLD #2 |
|---|---|---|---|
| D15 | HOLD | ✓ 発動 (review missing 6 連続) | ✓ 発動 (340A0 7 連続) |
| D16 | HOLD maintained | ✓ 残存 | ✗ 解消 (340A0 demote) |
| **D17** | **HOLD maintained** | **✓ 残存** (= 0/6 記入) | **✗ 解消継続** (= 340A0 demote 2 日連続) |

## §3 D14 review 確認結果 (= 0/6 記入)

| 項目 | 値 |
|---|---|
| yaml status | blank |
| §1 記入時刻 | BLANK |
| §1 ticker | BLANK |
| §2 entry 価格 actual | BLANK |
| §3 総 PnL | BLANK |
| §5 Reason 1 文 | BLANK |
| §6 final decision ☑ | BLANK |

→ **review_gap_UNRESOLVED** (= 0/6 記入、D15/D16 と同状態)。
   D17 で HOLD #1 解消不可、D18 以降で D14 review 記入後解消可能。

## §4 D14 paper PnL --review-md 再 run 結果

```bash
$ .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-05-14 --evaluation-date 2026-05-14 \
    --f111-real-batch-json /tmp/fire_d14_prep/d14_f111_real_batch.json \
    --review-md ~/fire-vault/04_daily/2026-06-02_manual_live_pilot_review.md \
    --output-json /tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d17_check.json
```

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: final_decision=unknown / is_blank=True
- warnings: ["review_missing"]

→ D14 review blank 継続、HOLD #1 解消不可。

## §5 recently_seen_codes 維持

- D16 拡張: 8747,5729,3489,340A0 (= demoted_count=4)
- **D17 維持**: 同 (= 設定変更なし)

## §6 340A0 demote 持続 (= D16-D17 で 2 日連続)

### 6.1 F111-real-batch top 4

| day | rank 4 | label | demoted |
|---|---|---|---|
| D14/D15 | 340A0 | boost_with_caution | False |
| D16 | 340A0 | **caution** | **True (= demote 初実施)** |
| **D17** | **340A0** | **caution** | **True (= demote 維持)** |

### 6.2 AFTER-R1 top_candidates

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D14/D15 | 340A0 ジグザグ | 3798 ＵＬＳ | 137A0 Cocolive |
| D16 | 3798 ＵＬＳ | 137A0 Cocolive | 331A0 メディックス |
| **D17** | **3798 ＵＬＳ** | **137A0 Cocolive** | **331A0 メディックス** |

→ **D16-D17 で 2 日連続 新 top 3 (= 3798/137A0/331A0)** = signal 安定。

### 6.3 D17 entry 候補

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| **5** | **3798** | **ＵＬＳグループ** | 情報通信 | **505** | **2,525** | **boost_with_caution ★ #1 (= 2 日連続)** |
| 6 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution |
| 7 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 | boost_with_caution (= 多様化) |
| 8 | 9130 | 共栄タンカー | 運輸 | 1,410 | 7,050 | boost_with_caution |
| 9 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |

## §7 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D17 当日 = **2026-06-05 (金)** → **22 営業日 gap** (= D16 21 → 1 日拡大)
- 朝再 refresh + iSPEED actual confirmation 必須 (= D14-D16 同)

## §8 D17 hard check (= chain-level 9/12)

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

→ chain-level 9/12 ✓、HOLD maintained で entry 推奨しない。

## §9 Pilot 判定: **HOLD maintained** (= 3 日連続)

### 9.1 HOLD 条件 status

| # | 条件 | D17 状況 | 発動 |
|---|---|---|---|
| 1 | review missing 5 連続超過 | D9-D14 6 連続 blank、**未解消** | **✓ 継続** |
| 2 | 同 candidate 7 連続 | 340A0 demote 維持、**新 top 2 日連続** | **✗ 解消継続** |
| 3 | actual 確認不可 | 朝確認次第 | 未 |
| 4 | liquidity 不可 | 朝確認次第 | 未 |
| 5 | event リスク | 朝確認次第 | 未 |

→ **5 条件中 1 残存** (= #1 のみ)、HOLD 維持。

### 9.2 HOLD 完全解除 path (= 主推奨、3 連続呼びかけ)

1. **D14 review.md 必須 5 項目記入** (= Fujiwara 本人、必須)
2. D14 paper PnL --review-md 再 run で reflection 確認
3. D18 chain 再起 → **HOLD #1 解消 → GO_CONDITIONAL 復帰**

### 9.3 D17 推奨アクション

- **主推奨**: skip (= HOLD 維持) + D14 review 記入推奨 + D18 で完全解除目標
- **alternative**: 3798 ＵＬＳ に少額 entry (= HOLD override、Fujiwara 責任、D16-D17 で 2 日連続 rank 1 で signal 安定)

## §10 entry candidate #1 (= 参考、HOLD 中)

| 項目 | 値 |
|---|---|
| code / name | 3798 / ＵＬＳグループ |
| sector | 情報通信・サービスその他 |
| close (5/14 refreshed) | 505 円 |
| score | 0.877 (A1 rank) |
| label | boost_with_caution |
| AFTER-R1 rank | 1 (= D16-D17 2 日連続) |
| 100 株 entry 想定資金 | 50,500 円 |
| stop_loss (5%) | 480 円 = max loss **2,525 円** |
| take_profit (2%) | 515 円 = target gain 1,000 円 |

## §11 actual price confirmation (= HOLD 中、参考)

| 候補 | refreshed (5/14) | actual (6/5) | 乖離率 |
|---|---|---|---|
| 3798 ＵＬＳ (= #1) | 505 | ____ | __.__% |
| 137A0 Cocolive | 739 | ____ | __.__% |
| 331A0 メディックス | 482 | ____ | __.__% |
| 7991 マミヤ (= 多様化) | 1,177 | ____ | __.__% |

## §12 liquidity manual check

| 候補 | 板厚 | 出来高 | spread |
|---|---|---|---|
| 3798 | ____ | ____ | __.__% |
| 137A0 | ____ | ____ | __.__% |
| 331A0 | ____ | ____ | __.__% |

## §13 event / earnings check

金曜引け前なので翌週決算予定確認:

| 候補 | TDnet | 決算予定 (来週) | ニュース |
|---|---|---|---|
| 3798 | __________ | __________ | __________ |
| 137A0 | __________ | __________ | __________ |
| 331A0 | __________ | __________ | __________ |

## §14 final decision (= D17 推奨)

- ☐ **skip (= HOLD maintained、D14 review 記入推奨)** ★ 推奨
- ☐ enter 3798 ＵＬＳ 100 株 (= HOLD override、Fujiwara 責任)
- ☐ enter 7991 マミヤ (= sector 多様化、HOLD override)
- ☐ watch

決定理由: __________________________

## §15 D17 review 必須 5 項目

1. ★ §1 記入時刻 + 対応 (= "HOLD maintained 対応" 等)
2. ★ §2 計画 vs 実際 (= "HOLD、entry skip" 想定)
3. ★ §3 PnL (= "0 円 (HOLD)")
4. ★ §5 Reason (= "HOLD #1 残存 3 連続、D14 review 記入推奨")
5. ★ §6 final decision = skip

## §16 D17 paper PnL handoff

### §16.1 D17 当日 (= 場後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-05 --evaluation-date 2026-06-05 \
  --f111-real-batch-json /tmp/fire_d17_prep/d17_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-05_manual_live_pilot_review.md \
  --output-json /tmp/fire_d17_prep/d17_paper_pnl_ledger_eod.json
```

### §16.2 D18 翌週月曜 (= 2026-06-08)

D14 review 記入確認後、D18 で HOLD 完全解除 → GO_CONDITIONAL 復帰可能性大。

## §17 stage 3 突合

| 項目 | 当日 | D1-D17 累積 |
|---|---|---|
| trade 回数 | 0 (= HOLD) | __ / 50 |
| 累積 PnL | 0 円 | __ 円 |
| f111_real_batch 比率 | 1/1 | 11/17 |
| top_candidates ≥1 達成率 | 1/1 | 12/17 |
| **HOLD maintained 連続** | 3 日 (D15-D17) | 3 days |
| 340A0 demote 維持 | 2 日連続 | 2 days |
| 新 top 3 (= 3798/137A0/331A0) | 2 日連続 | 2 days |
| review missing 連続 | 6 連続 (D9-D14) | 6 days |

## §18 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D17 = HOLD maintained 3 日連続 /
  340A0 demote 2 日連続 / 3798 新 #1 2 日連続 /
  D14 review 記入後 D18 解除可能性大
