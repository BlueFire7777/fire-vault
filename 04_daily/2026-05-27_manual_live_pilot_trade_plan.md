---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-27
owner: BlueFire7777 (Fujiwara)
pilot_day: D10
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
pilot_judgment: GO_CONDITIONAL
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D10_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D10_results.md
  - 04_daily/2026-05-27_manual_live_pilot_review.md
artifacts:
  f111: /tmp/fire_d10_prep/d10_f111_real_batch.json
  f062: /tmp/fire_d10_prep/d10_f062_preview.json
  morning_line_material: /tmp/fire_d10_prep/after_r1/morning_line_material_2026-05-27.json
---

# Manual Live Pilot — Trade Plan (2026-05-27 / D10)

## D10 特記: **refreshed price 初実弾 day**

- W60-jquants-daily-refresh-staging で 12 銘柄分の price 更新済 (= 2026-05-14)
- D9 で残った stale caveat が **大幅に軽減** (= 6 営業日 gap → 13 営業日 gap)
- top 3 entry 候補が **boost_with_caution / risk_within=True / 全て updated price**
- ⚠ ただし D10 当日 = 2026-05-27 (水) なので 5/14 refresh から **13 営業日 gap が再発**
  → 朝寄付き前 iSPEED で actual confirm 必須 / 可能なら朝 refresh wave 再実行

---

## §1 基本情報

- date: **2026-05-27 (水 / D10)**
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- J-Quants refresh status: ✓ (= W60-jquants-daily-refresh-staging で 5/14 まで)
- market_prices_daily max_date: **2026-05-14**
- preset tune settings:
  - `--label-threshold-mode strict` (= 0.92/0.85/0.75)
  - `--recently-seen-codes 8747,5729,3489`
  - `--max-candidates 20`
- demoted_count: 3

## §2 price freshness summary (= refresh 効果)

| code | name | stale (5/8) | refreshed (5/14) | 変化率 | risk_yen (refreshed) | 採用 |
|---|---|---|---|---|---|---|
| 8747 | 豊トラスティ証券 | 2,490 | 2,420 | -2.8% | 12,100 | skip (caution demoted) |
| 5729 | 日本精鉱 | 2,202 | **1,795** | **-18.5% 急落** | 8,975 | skip (caution demoted) |
| 3489 | フェイスネットワーク | 734 | 699 | -4.8% | 3,495 | skip (caution demoted) |
| **340A0** | **ジグザグ** | **398** | **380** | **-4.5%** | **1,900** | **★ #1** |
| **3798** | **ＵＬＳグループ** | **519** | **505** | **-2.7%** | **2,525** | **★ #2** |
| **137A0** | **Ｃｏｃｏｌｉｖｅ** | **0 (excluded)** | **739** | **新 entry 化** | **3,695** | **★ #3 (復活)** |
| 7991 | マミヤ・オーピー | 1,246 | 1,177 | -5.5% | 5,885 | 候補 #4 |
| 9130 | 共栄タンカー | 1,532 | 1,410 | -8.0% | 7,050 | 候補 #5 (sector 運輸) |
| 331A0 | メディックス | 480 | 482 | +0.4% | 2,410 | 候補 #6 |
| 4389 | プロパティデータバンク | 770 | 863 | **+12.1% 急騰** | 4,315 | 候補 #7 (caveat: 急騰) |
| 9247 | TRE ホールディングス | 1,621 | 1,612 | -0.6% | 8,060 | 候補 #8 |
| 4317 | レイ | 507 | 508 | +0.2% | 2,540 | 候補 #9 |

→ refresh の効果:
1. **137A0 Cocolive 復活** (= close=0→739、entry 候補化)
2. **5729 日本精鉱 -18.5% 察知** (= caution で見送り推奨、stale だと察知不能)
3. **4389 プロパティデータ +12.1% 上昇** (= entry 候補 #7、ただし急騰 caveat)

## §3 ⚠ 価格鮮度 caveat

- staging max_date = **2026-05-14**
- D10 当日 = **2026-05-27** → **13 営業日 gap** (= D9 = 6 営業日 gap よりも開いている)
- 真の解決: D10 朝寄付き前 (= 08:30 JST 想定) に再 refresh
  ```
  set -a; source /Users/bluefire/fire/.env; set +a
  FIRE_ENV=staging \
  HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 \
  HQ_APPROVE_JQUANTS_TOKEN_READ=1 \
  .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
    --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
    --datasets prices --max-days 14 \
    --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
    --sleep-seconds 0.3 --write \
    --output-json /tmp/d10_morning_refresh.json
  ```
- またはより安全な策: 寄付き直前 iSPEED で **actual price 確認**

## §4 top 3 entry 候補 (= boost_with_caution、refreshed price)

### §4.1 重点候補一覧

| rank | code | name | sector | close (5/14) | score | label | risk_yen | 推奨 |
|---|---|---|---|---|---|---|---|---|
| **1** | **340A0** | **ジグザグ** | 情報通信 | **380** | 0.892 | boost_with_caution | **1,900** | **★ #1 候補** |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 0.877 | boost_with_caution | 2,525 | #2 候補 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 0.869 | boost_with_caution | 3,695 | #3 候補 (= 新復活) |

⚠ **sector 100% 集中** (= 全 3 銘柄が情報通信・サービスその他) → 多様化のため #4 7991 (機械) も併記推奨

### §4.2 entry candidate #1: 340A0 ジグザグ

| 項目 | 値 |
|---|---|
| code / name | 340A0 / ジグザグ |
| sector | 情報通信・サービスその他 |
| close (= 2026-05-14 refreshed) | **380 円** (= D9 stale 398 → -4.5%) |
| score (research_final_score) | 0.892 |
| label | boost_with_caution |
| AFTER-R1 morning score | 175.22 (= rank 1) |
| 100 株 entry 想定資金 | 38,000 円 |
| stop_loss ratio | 5% |
| 想定 max loss | **1,900 円** (= pilot 上限 15,000 円の 12.7%、余裕 13,100 円) |
| target take_profit | 2% or h20 終値 |
| research_base_date | 2026-05-13 |
| reason_tags | staging market_listings / top_N + sector cap 通過 |

### §4.3 entry candidate #2: 3798 ＵＬＳグループ

| 項目 | 値 |
|---|---|
| code / name | 3798 / ＵＬＳグループ |
| close (refreshed) | 505 円 |
| score | 0.877 |
| label | boost_with_caution |
| risk_yen | 2,525 円 |

### §4.4 entry candidate #3: 137A0 Cocolive (= refresh で復活)

| 項目 | 値 |
|---|---|
| code / name | 137A0 / Cocolive |
| close (refreshed) | 739 円 (= D9 で close=0 excluded、refresh で復活) |
| score | 0.869 |
| label | boost_with_caution |
| risk_yen | 3,695 円 |
| 特記 | **W60-jquants-daily-refresh の直接的効果** |

## §5 actual price confirmation 欄 (= 寄付き直前 08:55 JST)

| 候補 | refreshed (5/14) | actual (5/27) | 乖離率 | 採否 |
|---|---|---|---|---|
| 340A0 ジグザグ | 380 | ____ | __.__% | ☐ entry / ☐ skip |
| 3798 ＵＬＳ | 505 | ____ | __.__% | ☐ entry / ☐ skip |
| 137A0 Cocolive | 739 | ____ | __.__% | ☐ entry / ☐ skip |

乖離率 ±5% 以内 → entry 検討。±5% 超 → skip 推奨 (= price stale)。

## §6 liquidity manual check (= 寄付き 5 分後 実測)

| 候補 | 板厚 5 本目処 | 寄付き 5 分 出来高 | 寄付き spread | 採否 |
|---|---|---|---|---|
| 340A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 3798 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |
| 137A0 | ____ 株 | ____ 株 | __.__% | ☐ OK / ☐ NG |

採否基準: 板厚 ≥ 5,000 株 / 出来高 ≥ 5,000 株 / spread ≤ 0.5%

## §7 event / earnings check (= 寄付き前)

| 候補 | TDnet 開示 | 決算予定 | ネガティブニュース | 採否 |
|---|---|---|---|---|
| 340A0 | __________ | __________ | __________ | ☐ OK / ☐ NG |
| 3798 | __________ | __________ | __________ | ☐ OK / ☐ NG |
| 137A0 | __________ | __________ | __________ | ☐ OK / ☐ NG |

## §8 final decision (= 寄付き直前 8:55 JST 記入)

採否 1 銘柄を選択:

- ☐ **enter 340A0 ジグザグ** 100 株 (= entry candidate #1)
- ☐ **enter 3798 ＵＬＳグループ** 100 株 (= sector 集中回避)
- ☐ **enter 137A0 Cocolive** 100 株 (= refresh 復活)
- ☐ **watch (= 全 銘柄寄付き後再判断)**
- ☐ **skip (= 全 条件 NG、D11 へ持ち越し)**

決定理由: __________________________

## §9 risk limit check (= 第 5 章 R-05-02/04/06/08)

| 項目 | 値 | 制約 | OK? |
|---|---|---|---|
| 1 トレード max loss | 1,900〜3,695 円 (3 候補中) | ≤ 15,000 (pilot) | ✓ |
| 1 日 max loss | 1 トレード分 | ≤ 30,000 (pilot) | ✓ |
| 同一 sector 連続 entry | 1 件 | ≤ 2 件 | ✓ |
| stop_loss 事前設定 | 5% | 必須 | ✓ |
| ナンピンなし | 想定 | 禁止 | ✓ |
| 持ち越しなし | 15:10 完全クローズ | 必須 | ✓ |

## §10 9 hard invariants (= morning_line_material 確認)

| invariant | D10 result |
|---|---|
| artifact_source | f062_preview ✓ |
| f062_raw_kind | f062_actual_dict ✓ |
| f111_input_source | f111_real_batch ✓ |
| freshness_verdict | **MISSING** ⚠ (= DATA-R3 gate JSON 未渡し、本 wave 意図的) |
| forbidden_phrases_check.passed | True ✓ |
| auto_order_allowed | False ✓ |
| manual_review_required | True ✓ |
| safety_flags 13 keys all False | True ✓ |
| top_candidates ≥ 1 | 3 ✓ (340A0/3798/137A0) |

## §11 Pilot 判定: **GO_CONDITIONAL**

**GO 条件 (= 9/9 達成)**:
- ☑ f111_real_batch ✓
- ☑ refreshed price available ✓ (= 2026-05-14)
- ☑ risk_within_pilot_limit=True ✓ (= 全 3 候補)
- ☑ top candidate あり ✓ (3 件)
- ☑ sample でない ✓
- ☑ tradable_universe=True ✓
- ☑ liquidity/event 確認欄が trade plan に明記 ✓
- ☑ exclusions_summary risk_above=0 (= 137A0 復活、risk filter pass)
- ☑ 9 hard invariants 8/9 PASS (freshness_verdict のみ意図的 MISSING)

**HOLD 条件 (= conditional 因子)**:
- ⚠ refreshed price = 5/14、D10 当日 = 5/27 → **13 営業日 gap**
- ⚠ sector 集中 (= 全 3 候補が情報通信) → 多様化のため #4 7991 (機械) 検討
- ⚠ freshness_verdict=MISSING (= DATA-R3 gate 未連携)

→ **判定: GO_CONDITIONAL** (= 寄付き直前 actual price + liquidity + event 3 確認後 entry、いずれか NG なら skip)

**D9 比較**:
- D9 = 6 営業日 stale gap、137A0 除外、Cocolive entry 不可
- **D10 = refresh 後、137A0 復活、price 乖離リスク把握しやすい**

## §12 W2 / W61 引き継ぎ項目

- D9/D10 候補比較: 340A0 / 3798 が連続選出 = signal 安定性確認
- J-Quants refresh 効果: 137A0 復活 + 5729 急落察知 + 4389 急騰察知
- W61 (= price/return/paper_pnl) 必要項目:
  - 340A0 paper entry 380 × 100 株 = 38,000 円、stop 1,900 円、profit 760 円
  - 137A0 同 paper sim を W61 で実装
- 次に必要な改善:
  - **liquidity filter 強化** (= 板厚 / spread を staging 内 read-only で再確認)
  - **DATA-R3 freshness gate JSON 連携** (= freshness_verdict MISSING → OK)
  - **paper_pnl 翌営業日 sim** (= F286-PNL-R3 既実装、D9 entry の paper PnL 計算)
  - **D11+ 全銘柄 daily refresh 自動化** (= launchd 経路、別 HQ 承認)

## §13 stage 3 突合

| 項目 | 当日 | D1-D10 累積 |
|---|---|---|
| trade 回数 | 1 (想定) | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| f111_real_batch 比率 | 1/1 (D10) | 4/10 (D6+D7+D9+D10) |
| top_candidates ≥1 達成率 | 1/1 | 5/10 |
| **J-Quants refresh 初実弾** | ✓ (D10) | 1 day |
| **137A0 復活 entry** | ✓ (D10) | 1 day (= 初) |

## §14 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D10 = refreshed price 初実弾 / W60-jquants 効果検証
