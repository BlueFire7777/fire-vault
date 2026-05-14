---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-26
owner: BlueFire7777 (Fujiwara)
pilot_day: D9
status: plan
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
preset_tune_applied: true
label_threshold_mode: strict
recently_seen_codes: ["8747", "5729", "3489"]
max_candidates: 20
pilot_judgment: GO_CONDITIONAL
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D9_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D9_results.md
  - 04_daily/2026-05-26_manual_live_pilot_review.md
artifacts:
  f111: /tmp/fire_d9_prep/f111_real_batch_2026-05-26.json
  f062: /tmp/fire_d9_prep/d9_f062_preview.json
  morning_line_material: /tmp/fire_d9_prep/after_r1/morning_line_material_2026-05-26.json
---

# Manual Live Pilot — Trade Plan (2026-05-26 / D9)

## D9 特記: GO_CONDITIONAL day

- W60-F111-preset-tune 改修 (strict + recently_seen + max=20) を **初適用**
- D6-D8 で 100% 重複した 8747/5729/3489 を caution へデモート → 見送り
- top 4 以降の boost_with_caution 7 銘柄から **entry 候補 #1 = 340A0 ジグザグ**
- ⚠ **market_prices_daily cap=2026-05-08 で 6 営業日 stale** → 朝寄付き actual で必ず再確認

---

## §1 基本情報

- date: **2026-05-26 (火 / D9)**
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- F111 preset tune settings:
  - `--label-threshold-mode strict` (= 0.92/0.85/0.75)
  - `--recently-seen-codes 8747,5729,3489` (= D6-D8 反復候補)
  - `--max-candidates 20`
- demoted_count: 3 (= 8747/5729/3489 → caution)

## §2 ⚠ market_prices_daily stale caveat

- staging market_prices_daily 最新 = **2026-05-08** (= 6 営業日前)
- 真の解決 = J-Quants daily refresh (W60-jquants-daily-refresh-staging 待ち、HQ approve 必要)
- 結果として:
  - close 価格 = 2026-05-08 の終値 (= D9 朝寄付きと乖離可能性)
  - risk 計算 (= 100 株 × 5% stop loss) は stale close base
  - 寄付き直前に **iSPEED で actual price 必ず確認**
  - 寄付き actual が stale close +5% 以上乖離 → entry 見送り

## §3 top 1-3 caution / skip section (= 反復候補、デモート対象)

| rank | code | name | label | recently_seen | 推奨 |
|---|---|---|---|---|---|
| 1 | 8747 | 豊トラスティ証券 | caution | True (D6-D8 demoted) | **skip** |
| 2 | 5729 | 日本精鉱 | caution | True | **skip** |
| 3 | 3489 | フェイスネットワーク | caution | True | **skip** |

skip 理由: D6-D8 で 3 営業日連続選出、recently_seen demote 適用。staging features 律速で内容更新なし。

## §4 top 4-10 candidate evaluation (= boost_with_caution、entry 候補)

| rank | code | name | sector | close (stale) | score | label | risk_within | risk_yen | entry 適性 |
|---|---|---|---|---|---|---|---|---|---|
| **4** | **340A0** | **ジグザグ** | 情報通信・サービス | 398 | 0.892 | boost_with_caution | True | 1,990 | **★ #1 候補** |
| 5 | 3798 | ＵＬＳグループ | 情報通信・サービス | 519 | 0.877 | boost_with_caution | True | 2,595 | #2 候補 |
| 6 | 137A0 | Ｃｏｃｏｌｉｖｅ | 情報通信・サービス | 0 | 0.869 | boost_with_caution | **False** | 0 | **除外 (close=0)** |
| 7 | 7991 | マミヤ・オーピー | 機械 | 1,246 | 0.867 | boost_with_caution | True | 6,230 | #3 候補 |
| 8 | 9130 | 共栄タンカー | 運輸・物流 | 1,532 | 0.859 | boost_with_caution | True | 7,660 | #4 候補 |
| 9 | 331A0 | メディックス | 情報通信・サービス | 480 | 0.857 | boost_with_caution | True | 2,400 | #5 候補 |
| 10 | 4389 | プロパティデータバンク | 情報通信・サービス | 770 | 0.856 | boost_with_caution | True | 3,850 | #6 候補 |

entry 適性 = risk_within=True + boost_with_caution + sector 多様性。

### §4.1 sector concentration 確認

- 情報通信・サービス: 5 銘柄 (340A0/3798/137A0/331A0/4389) — **集中**
- 機械: 1 (7991)
- 運輸・物流: 1 (9130)

→ sector 多様化のため #3 候補 = 7991 (機械) / #4 候補 = 9130 (運輸) も検討対象。

## §5 entry candidate #1: 340A0 ジグザグ

| 項目 | 値 |
|---|---|
| 銘柄コード / 名 | 340A0 / ジグザグ |
| sector | 情報通信・サービスその他 |
| close (= 2026-05-08 stale) | 398 円 |
| score (research_final_score) | 0.892 |
| research_advisory_label | boost_with_caution |
| sector_cap_status | selected (= top_N + sector cap 通過) |
| 100 株 entry 想定資金 | 39,800 円 (= 398 × 100、stale ベース) |
| stop_loss ratio | 5% (= R-05-12 必須) |
| 想定 max loss | 1,990 円 (= pilot 上限 15,000 円以内、余裕 13,010 円) |
| target take_profit | 2% or h20 終値 |

### §5.1 entry 手順 (= GO 時)

1. 寄付き 5 分前 (= 08:55 JST) iSPEED で **actual price 確認**
   - actual が stale 398 ±5% 以内 (= 378-418) → 通常 entry
   - actual が +5% 以上乖離 → entry 見送り (= price update 必要)
2. 出来高 5 分目処 = ≥ 5000 株 → 通常 entry
   - 5000 株未満 → 見送り (= 流動性不足)
3. spread = ≤ 0.5% → 通常 entry
   - 0.5% 超 → 見送り
4. ネガティブニュース確認 (= 寄付き直前)
   - 該当 → entry 見送り
5. すべて OK → iSPEED で寄付き成行で 100 株 entry
6. entry 完了後、stop_loss = entry_price × 0.95、take_profit = entry_price × 1.02 を mental note

## §6 liquidity check (= 寄付き 5 分後 実測)

- 板厚 5 本目処合計: ____ 株 (要 ≥ 5,000)
- 寄付き 5 分 出来高: ____ 株 (要 ≥ 5,000)
- 寄付き spread: __.__ % (要 ≤ 0.5%)
- D6-D8 と比較した liquidity 印象: __________

## §7 event / earnings check (= 寄付き前)

- 直近 1 週間の TDnet 開示: ____ (= 決算 / 業績修正 / 不祥事 etc.)
- 決算予定日 = 直近 5 営業日内: ☐ Yes / ☐ No
- ネガティブニュース有無: ☐ Yes / ☐ No
- 該当 → entry 見送り

## §8 final decision (= 寄付き直前 8:55 JST 記入)

- ☐ **enter** (= 全 check ✓ で 100 株 entry)
- ☐ **watch** (= 流動性 / event 確認したい、寄付き後再判断)
- ☐ **skip** (= price stale / liquidity 不足 / event リスク)

決定理由: __________________________

## §9 risk limit check (= 第 5 章 R-05-02/04/06/08)

| 項目 | 値 | 制約 | OK? |
|---|---|---|---|
| 1 トレード max loss | 1,990 円 | ≤ 15,000 (pilot) | ✓ |
| 1 日 max loss | 1,990 円 (= 1 トレードのみ) | ≤ 30,000 (pilot) | ✓ |
| 同一 sector 連続 entry | 1 件 (= 340A0 情報通信) | ≤ 2 件 | ✓ |
| stop_loss 事前設定 | 5% (R-05-12 必須) | 必須 | ✓ |
| ナンピンなし | (D6 持ち越しなし前提) | 禁止 | ✓ |
| 持ち越しなし | 15:10 完全クローズ | 必須 | ✓ |

## §10 stage 3 突合

| 項目 | 当日 | D1-D9 累積 |
|---|---|---|
| trade 回数 | 1 (想定) | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| f111_real_batch 比率 | 1/1 (D9) | 3/9 (D6+D7+D9) |
| top_candidates ≥1 達成率 | 1/1 | 4/9 |
| **W60-F111-preset-tune 初適用** | ✓ (D9) | 1 day |
| **boost_with_caution entry** | ✓ (340A0) | 1 day (= 初) |

## §11 9 hard invariants (= morning_line_material 確認)

| invariant | D9 result |
|---|---|
| artifact_source | f062_preview ✓ |
| f062_raw_kind | f062_actual_dict ✓ |
| f111_input_source | f111_real_batch ✓ |
| freshness_verdict | **MISSING** (= DATA-R3 gate 未渡し、本 wave 意図的、要確認 next wave) |
| forbidden_phrases_check.passed | True ✓ |
| auto_order_allowed | False ✓ |
| manual_review_required | True ✓ |
| safety_flags 13 keys all False | True ✓ |
| top_candidates ≥1 | 3 ✓ (340A0 / 3798 / 331A0) |

## §12 Pilot GO/HOLD/NO-GO 判定

### 判定: **GO_CONDITIONAL**

**GO 条件 (= 8/8 達成)**:
- ☑ f111_real_batch ✓
- ☑ top 4 以降に risk_within_pilot_limit=True 候補 ≥1 (= 7 銘柄)
- ☑ research label boost_with_caution 以上 (= 7 銘柄全て)
- ☑ repeated top 1-3 ではない (= recently_seen demote 適用、top 4 以降)
- ☑ sample でない (= 340A0 = ジグザグ実在ティッカー)
- ☑ tradable_universe=True
- ☑ liquidity/event 確認欄が trade plan に明記
- ☑ 9 hard invariants 8/9 PASS (freshness_verdict のみ MISSING、意図的)

**HOLD 条件 (= conditional 因子)**:
- ⚠ market_prices_daily cap=2026-05-08 → **6 営業日 stale**
- ⚠ freshness_verdict=MISSING (= DATA-R3 gate 未通過)
- ⚠ sector 集中 (情報通信 5/7 boost_with_caution)

→ 推奨 = **「GO_CONDITIONAL」** (= 寄付き直前に actual price + liquidity + event の 3 確認後 entry、いずれか NG なら skip)

### NO-GO ではない理由
- top 4 以降に risk_ok 候補 7 銘柄あり (= NO-GO 第 1 条件「候補なし」不該当)
- 9 invariants の重要項目 (artifact_source / forbidden_check / safety_flags / auto_order_allowed=False) 全 PASS

## §13 safety footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / W60-F111-preset-tune 初実弾検証
