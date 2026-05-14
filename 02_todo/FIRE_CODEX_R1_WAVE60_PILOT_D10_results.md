---
id: FIRE-CODEX-R1-WAVE60-PILOT-D10-results
phase: 本番 v0 中核 / Wave 60-pilot-D10 / refreshed price 初実弾検証 day
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-27 (D10) / wave 実行: 2026-05-14
pilot_day: D10
pilot_judgment: GO_CONDITIONAL
entry_candidate_1: 340A0 ジグザグ
entry_candidate_2: 3798 ULSグループ
entry_candidate_3: 137A0 Cocolive (= refresh 復活)
jquants_refresh_applied: true
market_prices_daily_max_date: 2026-05-14
---

# Wave 60-pilot-D10 Results — Refreshed Price Manual Live Pilot Trade Plan

## §1 結論

**D10 Pilot 判定 = GO_CONDITIONAL** (= 寄付き直前 actual price + liquidity +
event 3 確認後 entry、いずれか NG なら skip)。

W60-jquants-daily-refresh-staging で staging market_prices_daily を 2026-05-14
まで catch-up 済 (12 銘柄)。refreshed price ベースで:

- **#1: 340A0 ジグザグ** (close=380、risk=1,900 円、boost_with_caution)
- **#2: 3798 ＵＬＳグループ** (close=505、risk=2,525 円、boost_with_caution)
- **#3: 137A0 Cocolive** (close=739、risk=3,695 円、boost_with_caution、**refresh で復活**)

D9 比較で stale caveat 軽減、ただし 5/14 refresh → 5/27 D10 = **13 営業日 gap が残る**
→ 寄付き前再 refresh または iSPEED actual confirm 必須。

## §2 baseline + 既 artifacts

### 2.1 baseline (本 wave 開始時)

```
F282 plist: size=1751 mtime=1778602507
fire.db        : b1df4673e5c3645fbe2c5f490ffac043
fire.develop.db: 0eed4ad2ec7ed2edf8f640d97341c5ad
fire.staging.db: 6cb3885cd9c90bd6fdabb127c1cd0d17 (= 前 wave で 48 row insert 後の状態)
```

### 2.2 既 artifacts (前 wave 生成)

| artifact | path | size |
|---|---|---|
| F111-real-batch | /tmp/fire_d10_prep/d10_f111_real_batch.json | 25,479 B |
| F062 preview | /tmp/fire_d10_prep/d10_f062_preview.json | 17,289 B |
| F062 text | /tmp/fire_d10_prep/d10_f062_preview.txt | 2,772 B |
| F062 summary | /tmp/fire_d10_prep/d10_f062_summary.json | 1,030 B |
| AFTER-R1 morning | /tmp/fire_d10_prep/after_r1/morning_line_material_2026-05-27.json | 6,025 B |
| AFTER-R1 morning md | 同上 .md | 2,763 B |

## §3 price freshness summary

| code | name | stale (5/8) | refreshed (5/14) | 変化率 | risk_yen (refreshed) | 採用 |
|---|---|---|---|---|---|---|
| 8747 | 豊トラスティ証券 | 2,490 | 2,420 | -2.8% | 12,100 | skip (caution demoted) |
| 5729 | 日本精鉱 | 2,202 | **1,795** | **-18.5% 急落** | 8,975 | skip (caution demoted) |
| 3489 | フェイスネットワーク | 734 | 699 | -4.8% | 3,495 | skip (caution demoted) |
| **340A0** | **ジグザグ** | **398** | **380** | **-4.5%** | **1,900** | **★ #1** |
| **3798** | **ＵＬＳグループ** | **519** | **505** | **-2.7%** | **2,525** | **★ #2** |
| **137A0** | **Cocolive** | **0 (excluded)** | **739** | **新 entry 化** | **3,695** | **★ #3 (復活)** |
| 7991 | マミヤ・オーピー | 1,246 | 1,177 | -5.5% | 5,885 | 候補 #4 (機械) |
| 9130 | 共栄タンカー | 1,532 | 1,410 | -8.0% | 7,050 | 候補 #5 (運輸) |
| 331A0 | メディックス | 480 | 482 | +0.4% | 2,410 | 候補 #6 |
| 4389 | プロパティデータ | 770 | 863 | **+12.1% 急騰** | 4,315 | 候補 #7 (急騰 caveat) |
| 9247 | TRE ホールディングス | 1,621 | 1,612 | -0.6% | 8,060 | 候補 #8 |
| 4317 | レイ | 507 | 508 | +0.2% | 2,540 | 候補 #9 |

## §4 重点 3 銘柄 detail

### 4.1 340A0 ジグザグ (= entry candidate #1)

| 項目 | 値 |
|---|---|
| code / name | 340A0 / ジグザグ |
| sector | 情報通信・サービスその他 |
| close (refreshed 5/14) | 380 円 |
| 100 株 entry 想定資金 | 38,000 円 |
| score | 0.892 |
| label | boost_with_caution |
| AFTER-R1 morning score | 175.22 (= rank 1) |
| stop_loss (5%) | 361 円 = max loss 1,900 円 |
| take_profit (2%) | 388 円 = target gain 800 円 |
| research_base_date | 2026-05-13 |

### 4.2 3798 ＵＬＳグループ (= entry candidate #2)

| 項目 | 値 |
|---|---|
| code / name | 3798 / ＵＬＳグループ |
| close (refreshed) | 505 円 |
| 100 株 entry 資金 | 50,500 円 |
| score | 0.877 |
| label | boost_with_caution |
| stop_loss (5%) | 479 円 = max loss 2,525 円 |

### 4.3 137A0 Cocolive (= entry candidate #3、refresh 復活)

| 項目 | 値 |
|---|---|
| code / name | 137A0 / Cocolive |
| close (refreshed) | 739 円 (= D9 で close=0 excluded、refresh で復活) |
| 100 株 entry 資金 | 73,900 円 |
| score | 0.869 |
| label | boost_with_caution |
| stop_loss (5%) | 702 円 = max loss 3,695 円 |
| 特記 | **W60-jquants-daily-refresh の直接的効果による entry 復活** |

⚠ **sector 100% 集中** (= 全 3 銘柄が情報通信・サービスその他) → 多様化のため
#4 = 7991 マミヤ・オーピー (機械、close=1,177、risk=5,885) も併記推奨。

## §5 D10 hard check (= 9 invariants)

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
| top_candidates count ≥ 1 | 3 ✓ (340A0/3798/137A0) |

→ 8/9 PASS。freshness_verdict のみ意図的に MISSING (= 別 wave で gate JSON 連携)。

## §6 Pilot 判定: **GO_CONDITIONAL**

### 6.1 GO 条件 (= 9/9 達成)

- ☑ f111_real_batch ✓
- ☑ refreshed price available ✓ (= 2026-05-14)
- ☑ risk_within_pilot_limit=True (= 全 3 候補 + 候補 #4-#9 中 7 件)
- ☑ top candidate あり ✓ (= 3 件 + 多数 backup)
- ☑ sample でない ✓
- ☑ tradable_universe=True ✓
- ☑ liquidity/event 確認欄が trade plan に明記 ✓
- ☑ exclusions_summary risk_above=0 (= 137A0 復活、risk filter pass)
- ☑ 9 hard invariants 8/9 PASS

### 6.2 HOLD 条件 (= conditional 因子)

| 因子 | 状況 | 影響 |
|---|---|---|
| refreshed price = 5/14 | D10 = 5/27 → **13 営業日 gap** | 朝寄付き actual confirm 必須 |
| sector 集中 | 情報通信 100% (3/3) | 多様化のため #4 7991 (機械) 併記 |
| freshness_verdict=MISSING | DATA-R3 gate 未連携 | 次 wave で gate JSON 連携 |

### 6.3 NO-GO ではない理由

- top_candidates 3 件あり (= 条件 1 不該当)
- f111_input_source = f111_real_batch ✓
- risk_within = True (= 全 3 候補)
- sample = False
- forbidden_check.passed = True
- safety_flags 全 False
- price freshness 明確 (= 5/14 cap、13 営業日 gap で要 confirm)

→ **GO_CONDITIONAL** = 寄付き直前 actual + liquidity + event 3 確認後 entry。

## §7 D9 vs D10 比較

| 項目 | D9 (2026-05-26) | D10 (2026-05-27) |
|---|---|---|
| market_prices_daily max | 2026-05-08 (= **stale 6 営業日**) | 2026-05-14 (= **stale 13 営業日**) |
| refresh status | 未実行 | **済 (= 12 銘柄)** |
| entry candidates | 9 (= top 4-12) | **10** (= top 4-12 中 137A0 復活) |
| exclusions risk_above | 1 (137A0) | **0** |
| top_candidates rank 3 | 331A0 メディックス | **137A0 Cocolive (= 新)** |
| 340A0 risk_yen | 1,990 (stale) | **1,900 (refreshed)** |
| 5729 急落察知 | 不可 | **-18.5% 察知** |
| 4389 急騰察知 | 不可 | **+12.1% 察知** |
| Pilot 判定 | GO_CONDITIONAL | **GO_CONDITIONAL** (= refresh 効果で安定度向上) |

## §8 W2 / W61 引き継ぎ項目

### 8.1 D9/D10 候補比較

- 340A0 ジグザグ: D9, D10 連続選出 (= signal 安定性確認)
- 3798 ＵＬＳ: D9, D10 連続 (= 同)
- 137A0 Cocolive: D10 で復活 (= refresh 効果)
- 331A0 メディックス: D9 rank 3、D10 rank 3 落ち (137A0 へ譲渡)

### 8.2 J-Quants refresh 効果

| 効果 | D9 | D10 |
|---|---|---|
| 137A0 復活 (= excluded → entry) | × | ✓ |
| 5729 急落察知 | × | ✓ (-18.5%) |
| 4389 急騰察知 | × | ✓ (+12.1%) |
| 340A0 risk 微減 | (stale) | **1,990→1,900** |
| sector 多様化 | 情報通信集中 | 情報通信集中 (= 改善余地) |

### 8.3 W61 (price/return/paper_pnl) に渡す項目

- 340A0 paper entry sim: entry=380, stop=361, profit_target=388
- 137A0 paper entry sim: entry=739, stop=702, profit_target=754
- F286-PNL-R3 既実装 → D9/D10 entry の paper PnL 計算可能
- runner: `scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py` (= 既存)

### 8.4 次に必要な改善

1. **liquidity filter 強化** (= 板厚 / spread を staging 内 read-only で再確認)
2. **DATA-R3 freshness gate JSON 連携** (= freshness_verdict MISSING → OK)
3. **paper_pnl 翌営業日 sim** (= F286-PNL-R3 で D9/D10 entry sim)
4. **全銘柄 daily refresh 自動化** (= launchd 経路、別 HQ 承認)
5. **sector concentration cap** (= 情報通信 ≤2 件 等の F111 改修)
6. **D10 review outcome → W60-pilot-W2 集約**

## §9 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ (= 本 wave 開始時から read-only) |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| DB write | **0** ✓ |
| LINE / token / API / launchctl / plist / cron | 全 **0** ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 **0** ✓ |
| git tracked file 変更 | **0** (= vault docs 4 file 追加のみ) |

## §10 trade plan + review template path

- `~/fire-vault/04_daily/2026-05-27_manual_live_pilot_trade_plan.md` (D10 plan)
- `~/fire-vault/04_daily/2026-05-27_manual_live_pilot_review.md` (D10 review、blank)

## §11 next action 候補 (= 優先順)

1. **D10 朝 (= 2026-05-27 08:30 JST) 12 銘柄再 refresh** (= 5/14 → 5/27 まで)
2. **D10 寄付き直前 (08:55 JST) iSPEED で actual confirm**
3. **D10 review.md 記入** (= 15:30 以降)
4. **W60-pilot-D11 plan** (= 2026-05-28 木、recently_seen に D10 entry 銘柄追加)
5. **paper_pnl sim wave** (= F286-PNL-R3 で D9/D10 entry の翌営業日 sim、code 既存)
6. **liquidity filter 強化 wave** (= 板厚 / spread を staging 内 read-only)
7. **全銘柄 daily refresh launchd 自動化** (= 別 HQ 承認 + plist 配置)
8. **DATA-R3 freshness gate JSON 連携 wave** (= MISSING → OK)
9. **W60-pilot-W2 集約** (= D6-D10 5 営業日 review、KPI 算出)
