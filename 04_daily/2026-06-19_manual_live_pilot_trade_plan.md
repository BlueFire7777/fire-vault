---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-19
owner: BlueFire7777 (Fujiwara)
pilot_day: D27
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: GO_CONDITIONAL
post_recovery: 8 連続 maintained
post_demote: "137A0 demote 維持 3 連続"
top_continuity: "7991/9130/331A0 D25-D27 3 連続安定 ✓"
sector_diversification: "機械 + 運輸 + 情報通信 = 3 種維持 ✓"
hold_2_avoidance: 完全継続
related:
  - 04_daily/2026-06-19_manual_live_pilot_review.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D27_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D27_results.md
  - 03_design/FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md
---

# Manual Live Pilot — Trade Plan (2026-06-19 / D27)

## §1 D25-D26 GO_CONDITIONAL recap

- D25: 137A0 demote 本実行 → 新 top = 7991/9130/331A0、sector 3 種多様化達成 (初)
- D26: post-demote 2 連続安定、同 top 維持、HOLD #2 完全回避

→ **GO_CONDITIONAL 7 連続維持** (= D20-D26)

## §2 D26 review status

- yaml status: **blank** (= 0/5 必須項目記入)
- W3 §7.1 path 2 (= review gap 明確化) 継続適用

## §3 D26 paper PnL recap

- candidates=9 / evaluated=0 / **review_missing=1**
- review_actuals: is_blank=True / final_decision=unknown
- staging max=5/14 制約継続、h20 outcome 評価は 7/16 後

## §4 recently_seen_codes (= 6 件維持)

```
8747, 5729, 3489, 340A0, 3798, 137A0
demoted_count: 6
```

| 銘柄 | demote 開始 | D27 時点連続維持 |
|---|---|---|
| 8747/5729/3489 | 〜D1 | 22 連続 |
| 340A0 | D16 | 12 連続 |
| 3798 | D20 | 8 連続 |
| **137A0** | **D25** | **3 連続** |

## §5 137A0 demote 維持結果

- D27 F111: 137A0 caution / demoted=true 維持 ✓
- AFTER-R1 top 候補から除外維持
- HOLD #2 (= 同 candidate 7 連続) 再発リスクなし

## §6 sector diversification result

| rank | code | name | sector |
|---|---|---|---|
| 1 | **7991** | マミヤ・オーピー | **機械** |
| 2 | **9130** | 共栄タンカー | **運輸・物流** |
| 3 | **331A0** | メディックス | **情報通信** |

→ **3 種多様化 = D25-D27 で 3 連続維持** ✓

## §7 7991 / 9130 / 331A0 continuity (= 3 連続到達確認 ★)

| 銘柄 | D25 rank | D26 rank | D27 rank | rank 1 連続 |
|---|---|---|---|---|
| **7991** (機械) | 1 (= 初) | 1 (2 連続) | **1 (3 連続)** | **3** |
| **9130** (運輸) | 2 (= 初) | 2 (2 連続) | **2 (3 連続)** | n/a (= rank 2) |
| **331A0** (情報通信) | 3 (= D20-) | 3 | **3 (8 連続 rank 3)** | n/a (= rank 3) |

→ **rank 1 連続性 (W3 §3.3 #2)**: 7991 = 3 連続 (= warning まで残 2 wave、D29 想定で 5 連続到達)

## §8 next recurrence warning candidate

| 銘柄 | rank 1 連続 (D27 時点) | 段階 1 warning 予想 | 段階 3 demote 本実行予想 |
|---|---|---|---|
| **7991** (機械) | 3 | **D29** (= 月) | **D30** (= 火) |
| 9130 (運輸) | 0 (rank 2) | n/a | n/a |
| 331A0 (情報通信) | 0 (rank 3) | n/a | n/a |

→ D29 で 7991 demote sim 検討、D30 で本実行候補 (= W4 demote policy §5)

## §9 price freshness summary

- artifact_source: f062_preview
- f062_raw_kind: f062_actual_dict
- f111_input_source: f111_real_batch
- freshness_verdict: **MISSING** (= staging max=5/14 制約継続)
- AFTER-R1 9 invariants 8/9 PASS (= freshness のみ MISSING)
- W3 設計 doc で MISSING = GO_CONDITIONAL の caveat 扱い、HOLD trigger ではない

## §10 source artifact paths

- F111: `/tmp/fire_d27_prep/d27_f111.json`
- paper PnL: `/tmp/fire_d27_prep/d27_paper_pnl.json`
- paper PnL Markdown: `/tmp/fire_d27_prep/d27_paper_pnl.md`

## §11 D27 entry 候補

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| **1** ★ | **7991** | **マミヤ・オーピー** | **機械** | **1,177** | **5,885** | boost_with_caution |
| 2 | **9130** | 共栄タンカー | 運輸・物流 | 1,410 | 7,050 | boost_with_caution |
| 3 | **331A0** | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |
| 4 (参考) | 4389 | プロパティデータバンク | 情報通信 | 863 | 4,315 | boost_with_caution |
| 5 (参考) | 9247 | ＴＲＥホールディングス | 情報通信 | 1,612 | 8,060 | boost_with_caution |

★ rank 1 推奨: 7991 (= 3 連続安定、機械 sector、risk 控えめ)

## §12 D27 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1 | artifact_source=f062_preview | ✓ |
| 2 | f062_raw_kind=f062_actual_dict | ✓ |
| 3 | f111_input_source=f111_real_batch | ✓ |
| 4 | forbidden_check passed | ✓ |
| 5 | auto_order_allowed=false | ✓ |
| 6 | manual_review_required=true | ✓ |
| 7 | safety_flags all false | ✓ |
| 8 | top_candidates ≥ 1 | ✓ (= 3) |
| 9 | non-sample ticker | ✓ |
| 10 | tradable_universe=true | ✓ |
| 11 | risk_within_pilot_limit=true | ✓ |
| 12 | 朝 actual / liquidity / event 確認 | Fujiwara 朝待ち |

## §13 actual price confirmation (= Fujiwara 朝記入)

| code | actual 6/19 | 乖離率 | OK/NG |
|---|---|---|---|
| 7991 | ____ | __.__% | ☐ OK ☐ NG |
| 9130 | ____ | __.__% | ☐ OK ☐ NG |
| 331A0 | ____ | __.__% | ☐ OK ☐ NG |

許容乖離: ±5% (W3 §5 基準)

## §14 liquidity manual check (= 8:55-9:00 JST iSPEED)

| code | 板厚 | spread | 出来高 | OK/NG |
|---|---|---|---|---|
| 7991 | __ | __ | __ | ☐ OK ☐ NG |
| 9130 | __ | __ | __ | ☐ OK ☐ NG |
| 331A0 | __ | __ | __ | ☐ OK ☐ NG |

## §15 event / earnings check (= 8:30 JST TDnet)

| code | 開示 | 決算予定 | event 強度 | OK/NG |
|---|---|---|---|---|
| 7991 | __ | __ | __ | ☐ OK ☐ NG |
| 9130 | __ | __ | __ | ☐ OK ☐ NG |
| 331A0 | __ | __ | __ | ☐ OK ☐ NG |

## §16 final decision (= 朝寄付き前)

- ☐ **enter 7991 (= 機械、3 連続 #1、推奨)** ★
- ☐ enter 9130 (= 運輸、3 連続 #2)
- ☐ enter 331A0 (= 情報通信、3 連続 #3)
- ☐ watch (= 朝確認で見送り)
- ☐ skip (= event/liquidity NG で完全 skip)

## §17 safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
