---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-23
owner: BlueFire7777 (Fujiwara)
pilot_day: D29
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: GO_CONDITIONAL
post_recovery: 10 連続 maintained
post_demote: "137A0 demote 維持 5 連続"
top_continuity: "7991/9130/331A0 D25-D29 5 連続安定"
sector_diversification: "機械 + 運輸 + 情報通信 = 3 種維持 5 連続"
hold_2_avoidance: 完全継続 (= D15 以降 14 連続)
recurrence_warning: "7991 rank 1 5 連続 = warning 段階 1 発動 ★"
demote_sim_result: "新 top 1 = 9130 (運輸)、sector 2 種 (= 機械喪失、運輸+情報通信)"
d30_demote_recommendation: "★★★ D30 で 7991 demote 本実行強推奨 (= W4 demote policy §3 段階 3)"
related:
  - 04_daily/2026-06-23_manual_live_pilot_review.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D29_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D29_results.md
  - 03_design/FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md
---

# Manual Live Pilot — Trade Plan (2026-06-23 / D29)

## §1 D25-D28 GO_CONDITIONAL recap

- D25-D28 で 7991/9130/331A0 が 4 連続安定、sector 3 種多様化 4 連続維持
- HOLD #2 完全回避 9 連続継続、GO_CONDITIONAL maintained 9 連続

## §2 D28 review status

- yaml status: **blank** (= 0/5 必須項目記入)
- 累積 21 day blank (= D9-D29 想定)
- W3 §7.1 path 2 継続適用

## §3 D28 paper PnL recap

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown

## §4 recently_seen_codes (= 現行 6 件維持)

```
8747, 5729, 3489, 340A0, 3798, 137A0
demoted_count: 6
```

| 銘柄 | demote 開始 | D29 時点連続維持 |
|---|---|---|
| 8747/5729/3489 | 〜D1 | 24 連続 |
| 340A0 | D16 | 14 連続 |
| 3798 | D20 | 10 連続 |
| **137A0** | **D25** | **5 連続** |

## §5 137A0 demote 維持結果

- D29 F111: 137A0 caution / demoted 維持 ✓
- AFTER-R1 top 候補から除外維持
- HOLD #2 再発リスクなし

## §6 sector diversification result (= 3 種維持 5 連続)

| rank | code | name | sector |
|---|---|---|---|
| 1 | **7991** | マミヤ・オーピー | **機械** |
| 2 | **9130** | 共栄タンカー | **運輸・物流** |
| 3 | **331A0** | メディックス | **情報通信** |

→ D25-D29 で 3 種多様化 5 連続維持 ✓

## §7 7991 / 9130 / 331A0 continuity (= 5 連続到達 ★)

| 銘柄 | D25 | D26 | D27 | D28 | D29 | rank 1 連続 |
|---|---|---|---|---|---|---|
| **7991** (機械) | 1 (初) | 1 | 1 | 1 | **1** | **5** = warning ★ |
| 9130 (運輸) | 2 (初) | 2 | 2 | 2 | **2** | n/a |
| 331A0 (情報通信) | 3 | 3 | 3 | 3 | **3** | n/a |

## §8 ⚠ 7991 recurrence warning (= 段階 1 発動) ★★★

### 8.1 状態
- 7991 rank 1 連続 = **5** (= D25-D29)
- W4 demote policy §3.1 段階 1 (= warning 5 連続) **発動** ★
- HOLD #2 (= 7 連続) まで: 残 2 wave (= D30/D31)

### 8.2 三段階 policy 適用

| 段階 | 条件 | 日 | 状態 |
|---|---|---|---|
| 1 warning | rank 1 = 5 | **D29** (本日) | **発動 ✓** |
| 2 sim | rank 1 = 5-6 | **D29** (本日) | **実施済 ↓** |
| 3 本実行 | rank 1 = 6 (= 7 連続前) | **D30 (= 2026-06-24 水)** | **強推奨 ★★★** |

## §9 7991 demote simulation 結果 (= 参考 read-only chain)

### 9.1 sim 条件

```
recently_seen_codes: 8747, 5729, 3489, 340A0, 3798, 137A0, **7991** (= 追加)
demoted_count: 7
```

### 9.2 sim 後の新 top 5

| rank | code | name | sector | close | risk_yen |
|---|---|---|---|---|---|
| 1 | **9130** | 共栄タンカー | **運輸・物流** | 1,410 | 7,050 |
| 2 | **331A0** | メディックス | **情報通信** | 482 | 2,410 |
| 3 | **4389** | プロパティデータバンク | 情報通信 | 863 | 4,315 |
| 4 | 9247 | ＴＲＥホールディングス | 情報通信 | 1,612 | 8,060 |
| 5 | 4317 | レイ | 情報通信 | 508 | 2,540 |

### 9.3 sector 多様化への影響 (= ⚠ 一時縮退)

| 観点 | D29 現行 (= 3 種) | D30 7991 demote 後 (= sim) |
|---|---|---|
| rank 1 | 機械 (7991) | 運輸 (9130) |
| rank 2 | 運輸 (9130) | 情報通信 (331A0) |
| rank 3 | 情報通信 (331A0) | 情報通信 (4389) |
| sector 種数 | **3 種** | **2 種** (= 機械喪失) |

→ **7991 demote で sector 多様化は一時 2 種へ縮退**。
   ただし HOLD #2 阻止が優先 (= 同 candidate 7 連続到達は致命的)。

### 9.4 demote sim source artifact path

- `/tmp/fire_d29_prep/demote_sim/d29_7991_demote_sim_f111.json`

## §10 D30 demote recommendation (= ★★★ 強推奨)

### 10.1 D30 で 7991 demote 本実行する判断根拠

- W4 demote policy §3.1 段階 3 = rank 1 6 連続 (= 7 連続前) 本実行
- D30 で 7991 が 6 連続到達予想 → 本実行 timing 一致
- sim で新 top 1 = 9130 (運輸) 昇格を確認済、entry 候補は維持される
- sector 一時 2 種化は HOLD #2 阻止より優先度低い

### 10.2 D30 推奨案 (= 案 B 強推奨)

| 案 | 内容 | 推奨度 |
|---|---|---|
| A. D29 内即 demote | 段階 1 warning と同時に本実行 | ★★ (= W4 policy では段階 3 で実行、跳ばし非推奨) |
| **B. D30 で本実行** | 段階 3 (= 6 連続前) で正式 demote | **★★★ 推奨** (= W4 policy 厳守) |
| C. D31 まで delay | rank 1 7 連続到達リスク | ★ (= HOLD #2 発動懸念) |

### 10.3 案 B 採用時の D30 recently_seen_codes

```
8747, 5729, 3489, 340A0, 3798, 137A0, **7991** (= D30 新追加)
demoted_count: 7
```

## §11 price freshness summary

- artifact_source: f062_preview
- f062_raw_kind: f062_actual_dict
- f111_input_source: f111_real_batch
- freshness_verdict: **MISSING** (= staging max=5/14 制約継続)
- AFTER-R1 9 invariants 8/9 PASS

## §12 source artifact paths

- D28 paper PnL reload: `/tmp/fire_d29_prep/d28_paper_pnl_reload.json` + .md
- F111 (現行): `/tmp/fire_d29_prep/d29_f111.json`
- paper PnL (現行): `/tmp/fire_d29_prep/d29_paper_pnl.json`
- F111 demote sim: `/tmp/fire_d29_prep/demote_sim/d29_7991_demote_sim_f111.json`

## §13 D29 entry 候補

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| **1** ★ | **7991** | マミヤ・オーピー | **機械** | 1,177 | 5,885 | boost_with_caution (= **warning 段階**) |
| 2 | 9130 | 共栄タンカー | 運輸・物流 | 1,410 | 7,050 | boost_with_caution |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |
| 4 (参考) | 4389 | プロパティデータバンク | 情報通信 | 863 | 4,315 | boost_with_caution |
| 5 (参考) | 9247 | ＴＲＥホールディングス | 情報通信 | 1,612 | 8,060 | boost_with_caution |

★ 推奨: 7991 (= 5 連続安定、ただし **D30 demote 本実行直前の最終 day**)

## §14 D29 hard check (= chain-level 11/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-11 | chain-level invariants | 全 ✓ |
| 12 | actual / liquidity / event | Fujiwara 朝確認待ち |

## §15 actual price + liquidity + event (= 朝記入)

| code | actual 6/23 | 乖離率 | liquidity | event | OK/NG |
|---|---|---|---|---|---|
| 7991 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG |
| 9130 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG |
| 331A0 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG |

## §16 final decision (= 朝寄付き前)

- ☐ **enter 7991 (= 機械、5 連続 warning #1、D30 demote 直前)** ★
- ☐ enter 9130 (= 運輸、5 連続 #2、D30 sim 新 #1 候補)
- ☐ enter 331A0 (= 情報通信、5 連続 #3)
- ☐ watch (= 朝確認で見送り)
- ☐ skip (= event/liquidity NG)

## §17 safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
