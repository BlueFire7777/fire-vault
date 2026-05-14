---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-09
owner: BlueFire7777 (Fujiwara)
pilot_day: D19
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
  "#1_review_missing_6_consecutive": "未解消 (= D14 review 0/6、HOLD 5 連続)"
  "#2_same_candidate_7_consecutive": "解消継続 (= 340A0 demote 4 日連続)、ただし 3798 が AFTER-R1 #1 で 4 連続 → D22 で 7 連続懸念"
hard_check_chain_level_passed: 9/12
d14_review_gap_status: unresolved (= 0/6 記入、5 連続)
new_top_persistence: "3798/137A0/331A0 = D16-D19 で 4 日連続安定"
hold_2_re_emergence_warning: "3798 が AFTER-R1 rank 1 で 4 連続、D22 で 7 連続到達 → HOLD #2 再発動懸念"
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D19_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D19_results.md
  - 04_daily/2026-06-09_manual_live_pilot_review.md
artifacts:
  f111: /tmp/fire_d19_prep/d19_f111_real_batch.json
  f062: /tmp/fire_d19_prep/d19_f062_preview.json
  morning_line_material: /tmp/fire_d19_prep/after_r1/morning_line_material_2026-06-09.json
  paper_pnl_ledger: /tmp/fire_d19_prep/d19_paper_pnl_ledger.json
---

# Manual Live Pilot — Trade Plan (2026-06-09 / D19)

## D19 特記: **HOLD maintained 5 連続 + 3798 4 連続懸念 day**

- **D14 review 0/6 記入** = HOLD #1 残存 (= D15/D16/D17/D18/D19 で **5 連続 HOLD**)
- **340A0 demote 4 日連続** = HOLD #2 解消継続
- **3798 AFTER-R1 rank 1 が D16-D19 で 4 日連続** = signal 完全定着、ただし **D22 で 7 連続到達懸念** → HOLD #2 再発動可能性
- chain-level 9/12 ✓、ただし HOLD 維持で entry 推奨しない
- **次 wave (= W3 集約) で 3798 を recently_seen に追加検討**

---

## §1 基本情報

- date: **2026-06-09 (火 / D19)**
- 記入時刻: __:__ JST
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489,340A0 --max-candidates 20`
- demoted_count: 4
- pilot_judgment: **HOLD maintained**

## §2 D15-D18 HOLD recap

| day | pilot_judgment | HOLD #1 | HOLD #2 |
|---|---|---|---|
| D15 | HOLD | ✓ 発動 | ✓ 発動 |
| D16 | HOLD maintained | ✓ 継続 | ✗ 解消 |
| D17 | HOLD maintained | ✓ 継続 | ✗ 解消継続 (2 連続) |
| D18 | HOLD maintained | ✓ 継続 (4 連続) | ✗ 解消継続 (3 連続) |
| **D19** | **HOLD maintained** | **✓ 継続 5 連続** | **✗ 解消継続 4 連続** |

## §3 D14 review 確認結果 (= 0/6 記入、5 連続未記入)

| 項目 | 値 |
|---|---|
| yaml status | blank |
| §1 記入時刻 / ticker | BLANK |
| §2 entry 価格 actual | BLANK |
| §3 総 PnL | BLANK |
| §5 Reason 1 文 | BLANK |
| §6 final decision ☑ | BLANK |

→ **5 連続未記入** = review_gap_UNRESOLVED → HOLD #1 残存。

## §4 D14 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown
- warnings: ["review_missing"]
- output: `/tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d19_check.json`

## §5 recently_seen_codes 維持 (= D16-D19 で 4 連続)

- 8747, 5729, 3489, 340A0 (= demoted_count=4)

## §6 新 top 3 が 4 日連続安定 ✓ (= D16-D19)

### 6.1 AFTER-R1 top_candidates

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D16 | 3798 ＵＬＳ | 137A0 Cocolive | 331A0 メディックス |
| D17 | 3798 ＵＬＳ | 137A0 Cocolive | 331A0 メディックス |
| D18 | 3798 ＵＬＳ | 137A0 Cocolive | 331A0 メディックス |
| **D19** | **3798 ＵＬＳ** | **137A0 Cocolive** | **331A0 メディックス** |

→ **4 日連続安定** = signal 完全定着。

### 6.2 ⚠ 3798 連続性 caveat (= W3 集約 注意点)

| day range | 3798 rank | 状態 |
|---|---|---|
| D9-D15 | F111 rank 5 (= boost_with_caution、AFTER-R1 rank 2) | top 5 連続 |
| D16-D19 | AFTER-R1 rank 1 (= 4 連続) | AFTER-R1 top 4 連続 |

- 3798 が AFTER-R1 rank 1 として **4 連続**
- 設計 doc §3.3 HOLD 条件 #2 = "同 candidate 7 営業日連続"
- **D20 で 5 連続、D21 で 6 連続、D22 で 7 連続到達 → HOLD #2 再発動懸念**
- → W3 集約で **3798 を recently_seen に追加検討** 推奨

## §7 D19 entry 候補 (= D17/D18 と完全同一、4 日連続安定)

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| 5 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 2,525 | boost_with_caution ★ #1 (= 4 連続) |
| 6 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution |
| 7 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 | boost_with_caution (= 多様化) |
| 8 | 9130 | 共栄タンカー | 運輸 | 1,410 | 7,050 | boost_with_caution |
| 9 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |

## §8 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D19 当日 = **2026-06-09 (火)** → **24 営業日 gap** (= D18 23 → 1 営業日拡大)
- 朝再 refresh + iSPEED actual confirmation 必須 (= D14-D18 同)

## §9 D19 hard check (= chain-level 9/12 ✓)

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

## §10 Pilot 判定: **HOLD maintained 5 連続** (= 設計 doc §3.3 #1 のみ残存)

### 10.1 HOLD 条件 status

| # | 条件 | D15 | D16 | D17 | D18 | D19 |
|---|---|---|---|---|---|---|
| 1 | review missing 5 連続超過 | ✓ | ✓ | ✓ | ✓ | **✓ 5 連続** |
| 2 | 同 candidate 7 連続 | ✓ | ✗ | ✗ | ✗ | **✗ 解消 (340A0)、ただし 3798 4 連続懸念** |

### 10.2 D20 HOLD 完全解除 path (= 主推奨、5 連続呼びかけ)

| step | 内容 | 担当 |
|---|---|---|
| 1 | D14 review.md 必須 5 項目記入 ★最優先 | **Fujiwara 本人** |
| 2 | D14 paper PnL --review-md 再 run | 本線 (D20 wave) |
| 3 | D20 chain 再起 → **HOLD #1 解消** → GO_CONDITIONAL 復帰 | 本線 (D20 wave) |

### 10.3 D19 推奨アクション

- **主推奨**: skip (= HOLD 維持) + D14 review 記入 + W3 集約で 3798 demote 検討
- **alternative**: 3798 ＵＬＳ に少額 entry (= 4 連続 #1 で signal 完全定着、HOLD override = Fujiwara 責任)
- **W3 集約 (= 別 wave) で必須検討**:
  - 3798 を recently_seen に追加 (= D22 7 連続到達前に demote)
  - 137A0 / 331A0 も追加検討
  - 7991 多様化候補の評価

## §11 entry candidate #1 (= 参考、HOLD 中)

| 項目 | 値 |
|---|---|
| code / name | 3798 / ＵＬＳグループ |
| sector | 情報通信・サービスその他 |
| close (5/14 refreshed) | 505 円 |
| score | 0.877 (A1 rank) |
| label | boost_with_caution |
| AFTER-R1 rank | 1 (= D16-D19 4 連続) |
| 100 株 entry 想定資金 | 50,500 円 |
| stop_loss (5%) | 480 円 = max loss 2,525 円 |
| take_profit (2%) | 515 円 = target gain 1,000 円 |

## §12 actual price confirmation 欄 (= 参考)

| 候補 | refreshed (5/14) | actual (6/9) | 乖離率 |
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

## §14 event / earnings check

| 候補 | TDnet | 決算予定 | ニュース |
|---|---|---|---|
| 3798 | __________ | __________ | __________ |
| 137A0 | __________ | __________ | __________ |
| 331A0 | __________ | __________ | __________ |

## §15 final decision (= D19 推奨)

- ☐ **skip (= HOLD maintained 5 連続、D14 review 記入推奨)** ★ 主推奨
- ☐ enter 3798 ＵＬＳ 100 株 (= 4 連続 #1、HOLD override、Fujiwara 責任)
- ☐ enter 7991 マミヤ (= sector 多様化、HOLD override)
- ☐ watch

決定理由: __________________________

## §16 D19 review 必須 5 項目

1. ★ §1 記入時刻 + 対応 (= "HOLD maintained 5 連続対応")
2. ★ §2 計画 vs 実際 (= "HOLD、entry skip" 想定)
3. ★ §3 PnL (= "0 円 (HOLD)")
4. ★ §5 Reason (= "HOLD #1 残存 5 連続、3798 4 連続懸念、W3 集約 demote 検討")
5. ★ §6 final decision = skip

## §17 D19 paper PnL handoff

### §17.1 D19 当日 (= 場後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-09 --evaluation-date 2026-06-09 \
  --f111-real-batch-json /tmp/fire_d19_prep/d19_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-09_manual_live_pilot_review.md \
  --output-json /tmp/fire_d19_prep/d19_paper_pnl_ledger_eod.json
```

### §17.2 D20 翌水曜 (= 2026-06-10)

D14 review 記入確認後、D20 で HOLD 完全解除 → GO_CONDITIONAL 復帰可能性大。

## §18 stage 3 突合

| 項目 | 当日 | D1-D19 累積 |
|---|---|---|
| trade 回数 | 0 (= HOLD) | __ / 50 |
| 累積 PnL | 0 円 | __ 円 |
| f111_real_batch 比率 | 1/1 | 13/19 |
| top_candidates ≥1 達成率 | 1/1 | 14/19 |
| **HOLD maintained 連続** | 5 日 (D15-D19) | 5 days |
| 340A0 demote 維持 | 4 日連続 | 4 days |
| 新 top 3 連続安定 | 4 日連続 | 4 days |
| 3798 AFTER-R1 #1 連続 | 4 日連続 | 4 days |
| review missing 連続 | 6 連続 (D9-D14) | 6 days |

## §19 W3 集約準備 (= 次 wave 候補)

D14-D19 6 営業日累積で **W60-pilot-W3 集約 wave** を実施推奨:
- 6 day pilot_judgment 推移 (= D14 GO_CONDITIONAL → D15 HOLD → D16-D19 HOLD maintained)
- 340A0 demote 効果評価
- 新 top 3 (= 3798/137A0/331A0) 4 連続安定の signal 評価
- 3798 連続性懸念 + recently_seen 拡張案
- D14 review missing 5 連続の構造分析
- D20 entry criteria 改訂案

## §20 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D19 = HOLD maintained 5 連続 (= 火曜) /
  340A0 demote 4 連続 / 3798 #1 4 連続 / 7 連続懸念
