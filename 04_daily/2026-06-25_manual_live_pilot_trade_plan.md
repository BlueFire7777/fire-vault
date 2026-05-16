---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-25
owner: BlueFire7777 (Fujiwara)
pilot_day: D31
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: GO_CONDITIONAL
post_recovery: 12 連続 maintained
post_demote: "137A0 demote 維持 7 連続 + 7991 demote 維持 2 連続"
top_continuity: "9130/331A0/4389 D30-D31 2 連続安定 ✓"
sector_diversification: "2 種維持 (= 運輸 + 情報通信)、機械喪失 2 連続"
hold_2_avoidance: 完全継続 (= D15 以降 16 連続)
demoted_count: 7
related:
  - 04_daily/2026-06-25_manual_live_pilot_review.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_results.md
  - 03_design/FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md
---

# Manual Live Pilot — Trade Plan (2026-06-25 / D31)

## §1 D25-D30 GO_CONDITIONAL recap

- D25-D29: 7991/9130/331A0 5 連続安定、sector 3 種
- D29: 7991 5 連続 warning + demote sim
- D30: **7991 demote 本実行** → 新 top 1 = 9130 (運輸)、sector 一時 2 種縮退
- D30 = GO_CONDITIONAL maintained 11 連続 + HOLD #2 完全回避 15 連続

## §2 D30 review status

- yaml status: **blank** (= 0/5 必須項目記入)
- 累積 23 day blank (= D9-D31 想定)
- W3 §7.1 path 2 継続適用

## §3 D30 paper PnL recap

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown

## §4 D31 recently_seen_codes (= 7 件維持)

```
8747, 5729, 3489, 340A0, 3798, 137A0, 7991
demoted_count: 7
```

| 銘柄 | demote 開始 | D31 時点連続維持 |
|---|---|---|
| 8747/5729/3489 | 〜D1 | 26 連続 |
| 340A0 | D16 | 16 連続 |
| 3798 | D20 | 12 連続 |
| 137A0 | D25 | 7 連続 |
| **7991** | **D30** | **2 連続** |

## §5 7991 demote 効果維持確認 ✓ (= Codex Lane C YES)

- D31 F111: 7991 research_advisory_label = **caution** ✓
- top 5 から完全除外、rank 7
- W3 §3.3 #2-exclusion 適用継続

## §6 9130 / 331A0 / 4389 continuity (= D30-D31 2 連続安定 ✓)

| 銘柄 | D30 rank | D31 rank | rank 1 連続 (9130) |
|---|---|---|---|
| **9130** (運輸) | 1 (初) | **1** | **2** |
| **331A0** (情報通信) | 2 | **2** | n/a |
| **4389** (情報通信、新登場) | 3 | **3** | n/a |
| 9247 (情報通信) | 4 | 4 | n/a |
| 4317 (情報通信) | 5 | 5 | n/a |

→ Codex Lane A YES (= D30 vs D31 top 5 + sector 構成 100% 一致)

## §7 sector diversification (= 2 種維持、機械喪失 2 連続)

| 観点 | D29 (= 3 種維持期) | D30 (= demote 本実行) | **D31 (= 本日)** |
|---|---|---|---|
| rank 1 | 機械 (7991) | 運輸 (9130) | **運輸 (9130)** |
| rank 2 | 運輸 (9130) | 情報通信 (331A0) | **情報通信 (331A0)** |
| rank 3 | 情報通信 (331A0) | 情報通信 (4389) | **情報通信 (4389)** |
| sector 種数 | 3 種 | 2 種 | **2 種** (= 機械喪失 2 連続) |

→ Codex Lane D YES (= 2 種維持、3 種未回復)

## §8 HOLD #2 完全回避 (= D15 以降 16 連続継続) ✓

- D15 HOLD #2 発動 (= 340A0 7 連続) 以降、HOLD 再発動 0
- D31 時点で **16 連続** HOLD 完全回避
- 9130 rank 1 = 2 連続 (= W4 warning 5 連続まで残 3 wave)
- Codex Lane E YES

## §9 next recurrence warning candidate (= 9130 monitor 継続)

| 銘柄 | D31 状態 | rank 1 連続 | 段階 1 warning 予想 | 段階 3 本実行予想 |
|---|---|---|---|---|
| **9130** (運輸) | rank 1 (2 連続) | 2 | **D34 (= +3 wave、2026-06-30 月)** | **D35 (= 2026-07-01 火)** |
| 331A0 (情報通信) | rank 2 (2 連続) | 0 | n/a | n/a |
| 4389 (情報通信) | rank 3 (2 連続) | 0 | n/a | n/a |

## §10 price freshness summary

- artifact_source: f062_preview
- f062_raw_kind: f062_actual_dict
- f111_input_source: f111_real_batch
- freshness_verdict: **MISSING** (= staging max=5/14 制約継続)

## §11 source artifact paths

- D30 paper PnL reload: `/tmp/fire_d31_prep/d30_paper_pnl_reload.{json,md}`
- F111: `/tmp/fire_d31_prep/d31_f111.json`
- paper PnL: `/tmp/fire_d31_prep/d31_paper_pnl.{json,md}`

## §12 D31 entry 候補

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| **1** ★ | **9130** | 共栄タンカー | **運輸・物流** | 1,410 | 7,050 | boost_with_caution |
| 2 | **331A0** | メディックス | **情報通信** | 482 | 2,410 | boost_with_caution |
| 3 | **4389** | プロパティデータバンク | **情報通信** | 863 | 4,315 | boost_with_caution |
| 4 (参考) | 9247 | ＴＲＥホールディングス | 情報通信 | 1,612 | 8,060 | boost_with_caution |
| 5 (参考) | 4317 | レイ | 情報通信 | 508 | 2,540 | boost_with_caution |

★ rank 1 推奨: **9130 共栄タンカー** (= 運輸、2 連続安定)

## §13 D31 hard check (= chain-level 11/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-11 | chain-level invariants | 全 ✓ |
| 12 | actual / liquidity / event | Fujiwara 朝確認待ち |

## §14 actual price + liquidity + event (= 朝記入)

| code | actual 6/25 | 乖離率 | liquidity | event | OK/NG |
|---|---|---|---|---|---|
| 9130 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG |
| 331A0 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG |
| 4389 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG |

## §15 final decision

- ☐ **enter 9130 (= 運輸、2 連続 #1、推奨)** ★
- ☐ enter 331A0 (= 情報通信、2 連続 #2)
- ☐ enter 4389 (= 情報通信、2 連続 #3)
- ☐ watch / skip

## §16 safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
