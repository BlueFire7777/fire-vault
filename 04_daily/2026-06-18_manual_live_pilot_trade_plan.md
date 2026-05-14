---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-18
owner: BlueFire7777 (Fujiwara)
pilot_day: D26
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
post_recovery_status: "D20-D26 で 7 連続 GO_CONDITIONAL maintained"
hard_check_chain_level_passed: 9/12
demoted_count: 6
new_top_persistence: "7991/9130/331A0 D25-D26 2 連続安定 (= 137A0 demote 後 post-demote 安定)"
hold_2_avoidance: "HOLD #2 完全回避 (= 137A0 demote 維持で 7 連続到達阻止)"
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D26_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D26_results.md
  - 04_daily/2026-06-18_manual_live_pilot_review.md
  - 03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
artifacts:
  f111: /tmp/fire_d26_prep/d26_f111_real_batch.json
  f062: /tmp/fire_d26_prep/d26_f062_preview.json
  morning_line_material: /tmp/fire_d26_prep/after_r1/morning_line_material_2026-06-18.json
  paper_pnl_ledger: /tmp/fire_d26_prep/d26_paper_pnl_ledger.json
---

# Manual Live Pilot — Trade Plan (2026-06-18 / D26)

## D26 特記: **post-137A-demote 2 連続安定 + sector 3 種多様化維持 + HOLD 完全回避**

- **D20-D26 で 7 連続 GO_CONDITIONAL maintained**
- **D25 137A0 demote 本実行 → D26 で 7991/9130/331A0 = 2 連続安定**
- sector 3 種多様化 (= 機械 + 運輸 + 情報通信) 維持
- HOLD #2 完全回避 (= 137A0 demote 維持で 7 連続到達阻止)
- recently_seen_codes 6 件維持

---

## §1 基本情報

- date: **2026-06-18 (木 / D26)**
- 記入時刻: __:__ JST
- preset tune: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489,340A0,3798,137A0 --max-candidates 20`
- demoted_count: 6 (= D25 から維持)
- pilot_judgment: **GO_CONDITIONAL maintained 7 連続**

## §2 D25 GO_CONDITIONAL + demote recap

| day | pilot | 137A0 rank 1 連続 | demoted_count |
|---|---|---|---|
| D20-D24 | 1-5 連続 (warning) | 1-5 | 5 |
| D25 | 6 連続 (= demote 本実行) | demoted | 6 |
| **D26** | **7 連続 maintained** | **demoted 維持** | **6** |

## §3 D25 review 確認 (= 0/5 記入)

→ W3 §7.1 path 2 継続適用。

## §4 D25 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown

## §5 recently_seen_codes 維持 (= 6 件)

- 8747, 5729, 3489, 340A0, 3798, 137A0 (= D25 から維持)

## §6 7991/9130/331A0 D25-D26 2 連続安定 ✓

### 6.1 AFTER-R1 top_candidates

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D25 | 7991 (機械) | 9130 (運輸) | 331A0 (情報通信) |
| **D26** | **7991 (機械)** | **9130 (運輸)** | **331A0 (情報通信)** |

→ **D25-D26 2 連続安定** = 137A0 demote 後の新 top signal 定着。

### 6.2 sector 3 種多様化維持 ✓

| rank | code | sector |
|---|---|---|
| 1 | 7991 | 機械 |
| 2 | 9130 | 運輸・物流 |
| 3 | 331A0 | 情報通信 |

→ **3 種多様化が D26 で維持** ✓

### 6.3 HOLD #2 完全回避確認

- D20-D24: 137A0 5 連続 = warning 段階
- D25: 137A0 demote 本実行で連続中断
- D26: 137A0 demote 維持で 7 連続到達阻止 ✓
- → **HOLD #2 完全回避**

## §7 D26 entry 候補

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| 1 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 | boost_with_caution ★ #1 (= 2 連続) |
| 2 | 9130 | 共栄タンカー | 運輸・物流 | 1,410 | 7,050 | boost_with_caution (= 2 連続) |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution (= 2 連続) |

## §8 ⚠ price freshness caveat

- staging max_date = **2026-05-14**
- D26 当日 = **2026-06-18 (木)** → **31 営業日 gap**
- 朝再 refresh + iSPEED actual confirmation 必須

## §9 D26 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §10 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

→ **GO_CONDITIONAL maintained 7 連続**

## §11 entry candidate #1: 7991 マミヤ・オーピー (= 2 連続 #1)

| 項目 | 値 |
|---|---|
| code / name | 7991 / マミヤ・オーピー |
| sector | 機械 |
| close | 1,177 円 |
| 100 株 entry 想定資金 | 117,700 円 |
| stop_loss (5%) | 1,118 円 = max loss 5,885 円 |

## §12 actual price confirmation

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
  --output-json /tmp/d26_jq_refresh.json
```

| 候補 | refreshed (5/14) | actual (6/18) | 乖離率 |
|---|---|---|---|
| 7991 | 1,177 | ____ | __.__% |
| 9130 | 1,410 | ____ | __.__% |
| 331A0 | 482 | ____ | __.__% |

## §13 liquidity manual check

| 候補 | 板厚 | 出来高 | spread |
|---|---|---|---|
| 7991 | ____ | ____ | __.__% |
| 9130 | ____ | ____ | __.__% |
| 331A0 | ____ | ____ | __.__% |

## §14 event / earnings check

| 候補 | TDnet | 決算予定 | ニュース |
|---|---|---|---|
| 7991 | __________ | __________ | __________ |
| 9130 | __________ | __________ | __________ |
| 331A0 | __________ | __________ | __________ |

## §15 final decision

- ☐ enter 7991 マミヤ (機械、2 連続 #1)
- ☐ enter 9130 共栄タンカー (運輸、2 連続)
- ☐ enter 331A0 メディックス (情報通信)
- ☐ watch / skip

決定理由: __________________________

## §16 D26 review 必須 5 項目

1. ★ §1 記入時刻 + entry 銘柄
2. ★ §2 entry 価格 / 株数 / 時刻
3. ★ §3 総 PnL
4. ★ §5 Reason for entry / skip
5. ★ §6 final decision

## §17 D26 paper PnL handoff

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-18 --evaluation-date 2026-06-18 \
  --f111-real-batch-json /tmp/fire_d26_prep/d26_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-18_manual_live_pilot_review.md \
  --output-json /tmp/fire_d26_prep/d26_paper_pnl_ledger_eod.json
```

## §18 W4 集約引き継ぎ (= D20-D26 7 day pilot)

### 18.1 demote 効果 summary

| 銘柄 | demote 開始 | 連続維持 | 効果 |
|---|---|---|---|
| 8747 | D6 (recently_seen 初) | 21 連続 | suppress 維持 |
| 5729 | D6 | 21 連続 | suppress 維持 |
| 3489 | D6 | 21 連続 | suppress 維持 |
| 340A0 | D16 | 11 連続 | demote 効果定着 |
| 3798 | D20 | 7 連続 | demote 効果定着 |
| **137A0** | **D25** | **2 連続** | **demote 効果初期実証** |

### 18.2 sector diversification 推移

| 期 | top 1 sector | top 2 | top 3 | 多様化 |
|---|---|---|---|---|
| D14 | 情報通信 (340A0) | 情報通信 (3798) | 情報通信 (137A0) | 100% 集中 |
| D16-D19 | 情報通信 (3798) | 情報通信 (137A0) | 情報通信 (331A0) | 100% 集中 |
| D20-D24 | 情報通信 (137A0) | 機械 (7991) | 情報通信 (331A0) | 2 種 |
| **D25-D26** | **機械 (7991)** | **運輸 (9130)** | **情報通信 (331A0)** | **3 種** ✓ |

### 18.3 D20-D26 pilot_judgment 推移

| day | judgment |
|---|---|
| D20 | GO_CONDITIONAL 復帰 (初) |
| D21 | maintained 2 連続 |
| D22 | maintained 3 連続 |
| D23 | maintained 4 連続 |
| D24 | maintained 5 連続 (137A0 warning) |
| D25 | maintained 6 連続 (137A0 demote 本実行) |
| **D26** | **maintained 7 連続 (post-demote 安定)** |

### 18.4 review missing / paper PnL pending 状況

- 累積 review_missing: D9-D14 + D15-D26 (=18 day blank)
- paper PnL: 全 wave pending (= staging max=5/14 で未来 horizon 無し)
- 真の outcome 評価は h20 後 (= 6/26-30 頃)

### 18.5 次の demote 対象?

- 7991 (機械) が D25-D26 で 2 連続 rank 1 → 7 連続まで残 5 日 (= D32 想定)
- 9130 (運輸) も同期間連続化中
- W4 集約後の D27 以降で monitoring 継続、5 連続 warning で demote 検討

### 18.6 liquidity filter 強化必要性

- D14-D26 通して actual liquidity check は Fujiwara 朝確認のみ
- staging 内に板厚 / spread データ無し
- → liquidity filter 強化 wave (= 板厚 / 出来高 / spread を staging 内 で確認) 検討推奨

## §19 stage 3 突合

| 項目 | 当日 | D1-D26 累積 |
|---|---|---|
| trade 回数 | 0-1 | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| **GO_CONDITIONAL 連続** | 7 日 (D20-D26) | 7 days |
| 7991/9130/331A0 連続 | 2 連続 | 2 days |
| 137A0 demote 維持 | 2 連続 | 2 days |
| 340A0 demote 維持 | 11 連続 (D16-D26) | 11 days |
| 3798 demote 維持 | 7 連続 | 7 days |
| sector 3 種多様化 | 2 連続 | 2 days |

## §20 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D26 = post-137A-demote 2 連続 + sector 3 種多様化維持 /
  HOLD #2 完全回避 / 7991 機械 #1 / 9130 運輸 #2 / 331A0 情報通信 #3
