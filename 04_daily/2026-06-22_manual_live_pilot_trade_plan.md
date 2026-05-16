---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-22
owner: BlueFire7777 (Fujiwara)
pilot_day: D28
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: GO_CONDITIONAL
post_recovery: 9 連続 maintained
post_demote: "137A0 demote 維持 4 連続"
top_continuity: "7991/9130/331A0 D25-D28 4 連続安定 ✓"
sector_diversification: "機械 + 運輸 + 情報通信 = 3 種維持 4 連続 ✓"
hold_2_avoidance: 完全継続
recurrence_warning_pre: "7991 rank 1 連続 = 4、D29 で 5 連続 warning 到達予想"
related:
  - 04_daily/2026-06-22_manual_live_pilot_review.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D28_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D28_results.md
  - 03_design/FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md
---

# Manual Live Pilot — Trade Plan (2026-06-22 / D28)

## §1 D25-D27 GO_CONDITIONAL recap

- D25: 137A0 demote 本実行 → 新 top = 7991/9130/331A0、sector 3 種多様化達成 (初)
- D26: post-demote 2 連続安定
- D27: 3 連続到達、sector 3 種維持、HOLD #2 完全回避継続

→ **GO_CONDITIONAL 8 連続維持** (= D20-D27)、本 wave で **9 連続到達狙い**

## §2 D27 review status

- yaml status: **blank** (= 0/5 必須項目記入)
- 累積 20 day blank (= D9-D28 想定)
- W3 §7.1 path 2 (= review gap 明確化) 継続適用

## §3 D27 paper PnL recap

- candidates=20 / evaluated=0 / **review_missing=1**
- review_actuals: is_blank=True / final_decision=unknown
- staging max=5/14 制約継続、h20 outcome 評価は 7/17 後

## §4 recently_seen_codes (= 6 件維持)

```
8747, 5729, 3489, 340A0, 3798, 137A0
demoted_count: 6
```

| 銘柄 | demote 開始 | D28 時点連続維持 |
|---|---|---|
| 8747/5729/3489 | 〜D1 | 23 連続 |
| 340A0 | D16 | 13 連続 |
| 3798 | D20 | 9 連続 |
| **137A0** | **D25** | **4 連続** |

## §5 137A0 demote 維持結果

- D28 F111: 137A0 caution / demoted 維持 ✓
- AFTER-R1 top 候補から除外維持
- HOLD #2 (= 同 candidate 7 連続) 再発リスクなし

## §6 sector diversification result (= 3 種維持 4 連続)

| rank | code | name | sector |
|---|---|---|---|
| 1 | **7991** | マミヤ・オーピー | **機械** |
| 2 | **9130** | 共栄タンカー | **運輸・物流** |
| 3 | **331A0** | メディックス | **情報通信** |

→ D25-D28 で **3 種多様化 4 連続維持** ✓

## §7 7991 / 9130 / 331A0 continuity (= 4 連続到達確認 ★)

| 銘柄 | D25 rank | D26 rank | D27 rank | D28 rank | rank 1 連続 |
|---|---|---|---|---|---|
| **7991** (機械) | 1 (= 初) | 1 (2) | 1 (3) | **1 (4)** | **4** |
| **9130** (運輸) | 2 (= 初) | 2 (2) | 2 (3) | **2 (4)** | n/a |
| **331A0** (情報通信) | 3 (= D20-) | 3 | 3 | **3 (9)** | n/a |

→ **rank 1 連続性 (W3 §3.3 #2a)**: 7991 = **4 連続** (= **warning まで残 1 wave**、D29 で 5 連続到達予想)

## §8 7991 recurrence monitor / D29 warning preparation ★

### 8.1 D28 時点 7991 状態

- rank 1 連続: **4** (= D25-D28)
- HOLD #2 (= 7 連続) まで: **残 3 wave**

### 8.2 三段階 policy 予想 (= W4 demote policy §3)

| 段階 | 連続 | 予想 day | アクション |
|---|---|---|---|
| 1 warning | 5 | **D29** (= 2026-06-23 火) | trade plan §警告、demote sim 検討開始 |
| 2 sim | 5-6 | D29-D30 | recently_seen 追加 sim chain (= 7991 候補) |
| 3 本実行 | 6 | **D30** (= 2026-06-24 水) | recently_seen 正式追加 |

### 8.3 D28 時点ではまだ demote しない

- D28 = 4 連続 = warning 段階 1 未到達
- W4 demote policy §3 では 5 連続から warning 開始
- D28 trade plan §警告で「D29 で sim 候補化」を予告のみ

### 8.4 sim 後の新 top 予想 (= 7991 demote 仮想 chain)

- recently_seen に 7991 追加 → demoted_count=7
- 新 top 1 候補: **9130 (運輸)** または **331A0 (情報通信)** が rank 1 昇格
- sector 多様化: 機械 sector を一時喪失 → 運輸 + 情報通信 2 種、または別 sector 候補登場

→ D29 sim で実証して確認、D28 では予告のみ

## §9 price freshness summary

- artifact_source: f062_preview
- f062_raw_kind: f062_actual_dict
- f111_input_source: f111_real_batch
- freshness_verdict: **MISSING** (= staging max=5/14 制約継続)
- AFTER-R1 9 invariants 8/9 PASS (= freshness のみ MISSING)
- W3 設計 doc で MISSING = GO_CONDITIONAL の caveat 扱い

## §10 source artifact paths

- D27 paper PnL reload: `/tmp/fire_d28_prep/d27_paper_pnl_reload.json` + .md
- F111: `/tmp/fire_d28_prep/d28_f111.json`
- paper PnL: `/tmp/fire_d28_prep/d28_paper_pnl.json`
- paper PnL Markdown: `/tmp/fire_d28_prep/d28_paper_pnl.md`

## §11 D28 entry 候補

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| **1** ★ | **7991** | **マミヤ・オーピー** | **機械** | **1,177** | **5,885** | boost_with_caution |
| 2 | **9130** | 共栄タンカー | 運輸・物流 | 1,410 | 7,050 | boost_with_caution |
| 3 | **331A0** | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |
| 4 (参考) | 4389 | プロパティデータバンク | 情報通信 | 863 | 4,315 | boost_with_caution |
| 5 (参考) | 9247 | ＴＲＥホールディングス | 情報通信 | 1,612 | 8,060 | boost_with_caution |

★ rank 1 推奨: 7991 (= 4 連続安定、機械 sector、ただし **D29 で demote sim 候補化予想**)

## §12 D28 hard check (= chain-level 11/12 ✓)

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

| code | actual 6/22 | 乖離率 | OK/NG |
|---|---|---|---|
| 7991 | ____ | __.__% | ☐ OK ☐ NG |
| 9130 | ____ | __.__% | ☐ OK ☐ NG |
| 331A0 | ____ | __.__% | ☐ OK ☐ NG |

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

- ☐ **enter 7991 (= 機械、4 連続 #1、推奨、ただし D29 demote sim 候補化予想)** ★
- ☐ enter 9130 (= 運輸、4 連続 #2)
- ☐ enter 331A0 (= 情報通信、4 連続 #3)
- ☐ watch (= 朝確認で見送り)
- ☐ skip (= event/liquidity NG で完全 skip)

## §17 safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
