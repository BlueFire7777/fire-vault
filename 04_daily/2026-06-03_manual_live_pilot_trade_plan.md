---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-03
owner: BlueFire7777 (Fujiwara)
pilot_day: D15
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
pilot_judgment: HOLD
hold_trigger: review_missing_6_consecutive
hard_check_chain_level_passed: 9/12
d9_d14_review_missing_count: 6
d9_d15_overlap: 100%
wave_purpose: D14 review 反映後の継続判断 day (= HOLD 発動、Fujiwara review 記入待ち)
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D15_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D15_results.md
  - 04_daily/2026-06-03_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
artifacts:
  f111: /tmp/fire_d15_prep/d15_f111_real_batch.json
  f062: /tmp/fire_d15_prep/d15_f062_preview.json
  morning_line_material: /tmp/fire_d15_prep/after_r1/morning_line_material_2026-06-03.json
  paper_pnl_ledger: /tmp/fire_d15_prep/d15_paper_pnl_ledger.json
  d14_paper_pnl_with_review: /tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review.json
---

# Manual Live Pilot — Trade Plan (2026-06-03 / D15)

## D15 特記: **HOLD 発動 day** (= review missing 6 連続、設計 doc §3.3 #1)

- **D9-D14 で 6 連続 review missing 確定** → 設計 doc §3.3 HOLD 条件 #1 発動
- chain-level 9/12 ✓ (= candidate 自体は問題なし)
- 解除条件: **Fujiwara が D10-D14 review のうち最低 1 つ記入** → HOLD 解除 → GO_CONDITIONAL へ
- D15 では **新規 entry を推奨しない**、ただし候補 chain は記録継続
- 340A0 が D9-D15 で **7 営業日連続** 選出 → 設計 doc §3.3 #2 (同 candidate 7 営業日連続) も同時発動可能

---

## §1 基本情報

- date: **2026-06-03 (水 / D15)**
- 記入時刻: __:__ JST
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- 設計 doc: `~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md`
- preset tune: strict + recently_seen=8747,5729,3489 + max=20

## §2 D14 review recap (= 6 連続 review missing 確定)

| 項目 | 値 |
|---|---|
| status | **blank** |
| 必須 5 項目記入 | 全 BLANK (= §1 記入時刻 / ticker / §2 entry / §3 PnL / §5 reason / §6 final decision) |
| final_decision | unknown |
| actual entry/exit/PnL | 全 None |

→ D10/D11/D12/D13/D14/D9 全 status: blank、**6 連続 review missing**。

## §3 D9-D14 review status 一覧

| day | date | status | final_decision |
|---|---|---|---|
| D9 | 2026-05-26 | blank | unknown |
| D10 | 2026-05-27 | blank | unknown |
| D11 | 2026-05-28 | blank | unknown |
| D12 | 2026-05-29 | blank | unknown |
| D13 | 2026-06-01 | blank | unknown |
| D14 | 2026-06-02 | blank | unknown |

→ **6 連続 review missing** = 設計 doc §3.3 HOLD 条件 #1 (= 5 連続超過) 発動。

## §4 D14 paper PnL --review-md 付き再 run 結果

```bash
$ .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-05-14 --evaluation-date 2026-05-14 \
    --f111-real-batch-json /tmp/fire_d14_prep/d14_f111_real_batch.json \
    --review-md ~/fire-vault/04_daily/2026-06-02_manual_live_pilot_review.md \
    --output-json /tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review.json
```

| 項目 | 値 |
|---|---|
| candidates | 20 |
| evaluated_count | 0 (= 全 pending) |
| paper_win/loss/flat | 0 / 0 / 0 |
| review_missing_count | 1 |
| review_actuals.final_decision | unknown |
| review_actuals.is_blank | True |
| warnings | ["review_missing"] |

→ review 取込済、ただし blank → unknown / is_blank=True 確認。

## §5 ⚠ price freshness caveat (= D14 と同)

- staging max_date = **2026-05-14**
- D15 当日 = **2026-06-03 (水)** → **20 営業日 gap**
- focused refresh: D14 wave で dry-run = already_up_to_date (= staging max = wave 実行日)
- 朝再 refresh が D15 でも必須 (= D14 同様の actual confirmation 強制)

## §6 D9-D15 candidate overlap (= 7 営業日連続 340A0)

| day | rank 4 | close | risk_yen | label |
|---|---|---|---|---|
| D9 | 340A0 | 398 stale | 1,990 | boost_with_caution |
| D10 | 340A0 | 380 refreshed | 1,900 | boost_with_caution |
| D11 | 340A0 | 380 | 1,900 | boost_with_caution |
| D12 | 340A0 | 380 | 1,900 | boost_with_caution |
| D13 | 340A0 | 380 | 1,900 | boost_with_caution |
| D14 | 340A0 | 380 | 1,900 | boost_with_caution |
| **D15** | **340A0** | **380** | **1,900** | **boost_with_caution** |

→ **7 営業日連続 340A0** = 設計 doc §3.3 HOLD 条件 #2 (= 同 candidate 7 営業日連続) も発動可能。

## §7 top 3 entry 候補 (= 候補 chain は記録継続)

| rank | code | name | sector | close | score | label | risk_yen |
|---|---|---|---|---|---|---|---|
| 1 | 340A0 | ジグザグ | 情報通信 | 380 | 0.892 | boost_with_caution | 1,900 |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 0.877 | boost_with_caution | 2,525 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 0.869 | boost_with_caution | 3,695 |

→ candidate 自体は問題なし、ただし HOLD 発動で **D15 entry 非推奨**。

## §8 D15 hard check (= 12 項目)

| # | 項目 | 結果 |
|---|---|---|
| 1 | f111_input_source=f111_real_batch | ✓ |
| 2 | top_candidate ≥ 1 | ✓ (= 3) |
| 3 | risk_within_pilot_limit=True | ✓ |
| 4 | sample でない | ✓ |
| 5 | tradable_universe=True | ✓ |
| 6 | forbidden_phrases_check.passed=True | ✓ |
| 7 | auto_order_allowed=False | ✓ |
| 8 | manual_review_required=True | ✓ |
| 9 | safety_flags 13 keys all False | ✓ |
| 10 | actual price confirmation passed | 朝確認待ち |
| 11 | liquidity/spread/volume passed | 朝確認待ち |
| 12 | event/earnings passed | 朝確認待ち |

→ chain-level 9/12 ✓、ただし **HOLD 発動で D15 は entry しない**。

## §9 Pilot 判定: **HOLD** (= 設計 doc §3.3 #1 + #2 発動)

### 9.1 HOLD 発動条件 (= 設計 doc §3.3)

| # | 条件 | D15 状況 | 発動 |
|---|---|---|---|
| 1 | review missing が 5 連続超過 | **D9-D14 で 6 連続 blank** | **✓ 発動** |
| 2 | 同 candidate が 7 営業日以上連続 | **340A0 D9-D15 で 7 連続** | **✓ 発動** |
| 3 | actual price 確認できない | (朝確認次第) | 未確認 |
| 4 | liquidity 確認できない | (朝確認次第) | 未確認 |
| 5 | event risk 強い | (朝確認次第) | 未確認 |

→ 5 条件中 **2 条件発動** で HOLD 確定。

### 9.2 HOLD 解除条件

| # | 条件 | 達成方法 |
|---|---|---|
| 1 | review missing < 6 連続 | **D10-D14 review のうち少なくとも 1 つを記入** (= Fujiwara 本人、最低 5 項目) |
| 2 | 同 candidate < 7 営業日連続 | 自然に解消 (= D15 で記録、D16 で recently_seen に 340A0 を入れる) |

→ Fujiwara 介入が **review 記入** で HOLD 解除 path 開放。

## §10 D15 推奨アクション (= HOLD day)

### 10.1 主推奨: review 記入後 HOLD 解除

```
1. D14 review.md (2026-06-02) を開く
2. 必須 5 項目を記入:
   - §1 記入時刻 + entry 銘柄 (= skip でも "skip" と記入)
   - §2 entry 価格 / 株数 / 時刻 (= skip なら "skip" と記入)
   - §3 総 PnL (= skip なら "0 円 (skip)" と記入)
   - §5 Reason for entry / skip (= 1 文以上)
   - §6 final decision checkbox (= enter / watch / skip いずれか選択)
3. D14 paper PnL --review-md 付き再 run で review_actuals 反映確認
4. D15 chain 再起 + 判定見直し (= HOLD 解除可能)
```

### 10.2 代替案: D15 skip + D16 で recently_seen 拡張

D14 review 記入が困難なら、D15 は **完全 skip**、D16 で:
- recently_seen_codes に 340A0/3798/137A0 を追加 (= 同候補 demote)
- 多様化候補 (= 7991 機械、4389 不動産系 etc) を top に押し上げ
- review 記入再開

## §11 entry candidate #1 (= 参考情報、entry はしない)

| 項目 | 値 |
|---|---|
| code / name | 340A0 / ジグザグ |
| close (5/14 refreshed) | 380 円 |
| score / label | 0.892 / boost_with_caution |
| 100 株 entry 想定資金 | 38,000 円 |
| stop_loss (5%) | 361 円 = max loss 1,900 円 |
| **D15 推奨** | **HOLD 発動で entry しない** |

## §12 actual price confirmation 欄 (= HOLD day、参考)

D15 では entry 推奨しないため、actual confirmation は **省略可**。
朝 refresh を実行する場合は D14 同様の手順。

## §13 final decision (= D15 推奨)

採否を選択:

- ☐ **skip (= HOLD 発動、D14 review 記入後 D16 で再判定)** ★ 推奨
- ☐ enter 340A0 100 株 (= HOLD override、Fujiwara 自身の責任)
- ☐ watch (= 寄付き観察後再判断)

決定理由: __________________________

## §14 D15 paper PnL handoff 手順

### §14.1 D15 当日 (= 場後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-03 --evaluation-date 2026-06-03 \
  --f111-real-batch-json /tmp/fire_d15_prep/d15_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-03_manual_live_pilot_review.md \
  --output-json /tmp/fire_d15_prep/d15_paper_pnl_ledger_eod.json
```

### §14.2 D16 翌朝 (= 2026-06-04 木)

D15 entry 銘柄 (= 通常 0 = HOLD) について h1 close 確認は不要。

### §14.3 h20 後 (= 2026-06-30 頃)

D15 = HOLD day なら outcome なし。entry した場合のみ評価。

## §15 D15 review 必須記入項目

D14 review 記入 path 開放のため、D15 review でも **最低 5 項目** 記入推奨:

1. ★ §1 記入時刻 + 候補対応 (= skip でも "HOLD 対応" 記入)
2. ★ §2 計画 vs 実際 (= "D15=HOLD、entry skip" 記入)
3. ★ §3 PnL (= "0 円 (HOLD)" 記入)
4. ★ §5 Reason (= "HOLD 発動: review missing 6 連続 + 同候補 7 連続" 1 文以上)
5. ★ §6 final decision = skip 選択

## §16 stage 3 突合

| 項目 | 当日 | D1-D15 累積 |
|---|---|---|
| trade 回数 | 0 (= HOLD) | __ / 50 |
| 累積 PnL | 0 円 (= HOLD) | __ 円 |
| f111_real_batch 比率 | 1/1 | 9/15 |
| top_candidates ≥1 達成率 | 1/1 | 10/15 |
| **HOLD 発動 (= 設計 doc §3.3 初発動)** | ✓ | 1 day (= 初) |
| 340A0 連続選出 | ✓ (D9-D15) | 7 days |
| review missing 連続 | 6 連続 (D9-D14) | 6 days |

## §17 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D15 = HOLD (= review missing 6 連続 + 同候補 7 連続) /
  review 記入後 HOLD 解除 path 開放
