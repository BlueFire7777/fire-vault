---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-06-24
owner: BlueFire7777 (Fujiwara)
pilot_day: D30
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: GO_CONDITIONAL
post_recovery: 11 連続 maintained
post_demote: "137A0 demote 維持 6 連続 + 7991 demote 本実行初日"
demote_event: "★★★ 7991 demote 本実行成功 (= W4 demote policy §3 段階 3 厳守) ★★★"
new_top_after_demote: "9130 (運輸) / 331A0 (情報通信) / 4389 (情報通信)"
sector_diversification: "★ 一時 2 種縮退 (= 運輸 + 情報通信、機械喪失)"
hold_2_avoidance: 完全継続 (= D15 以降 15 連続)
demoted_count: 7
related:
  - 04_daily/2026-06-24_manual_live_pilot_review.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D30_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D30_results.md
  - 03_design/FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md
---

# Manual Live Pilot — Trade Plan (2026-06-24 / D30)

## §1 D25-D29 GO_CONDITIONAL recap

- D25-D29 で 7991/9130/331A0 が 5 連続安定、sector 3 種多様化 5 連続維持
- D29 で 7991 rank 1 5 連続 warning 発動、demote sim 完了
- GO_CONDITIONAL maintained 10 連続、HOLD #2 完全回避 14 連続

## §2 D29 review status

- yaml status: **blank** (= 0/5 必須項目記入)
- 累積 22 day blank (= D9-D30 想定)
- W3 §7.1 path 2 継続適用

## §3 D29 paper PnL recap

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown

## §4 D30 recently_seen_codes (= 7 件、★ 7991 新追加)

```
8747, 5729, 3489, 340A0, 3798, 137A0, **7991** (= D30 新追加)
demoted_count: 7
```

| 銘柄 | demote 開始 | D30 時点連続維持 |
|---|---|---|
| 8747/5729/3489 | 〜D1 | 25 連続 |
| 340A0 | D16 | 15 連続 |
| 3798 | D20 | 11 連続 |
| 137A0 | D25 | 6 連続 |
| **7991** | **D30** | **1 連続 (= 本日 本実行 初日)** ★ |

## §5 ⚠ 7991 demote 本実行結果 (= 主要成果 ★★★)

### 5.1 F111 chain 確認

- recently_seen_codes: 7 件 (= 8747,5729,3489,340A0,3798,137A0,**7991**)
- demoted_count: 7
- **7991 research_advisory_label = caution** ✓ (= demote 化確認)
- AFTER-R1 top から完全除外

### 5.2 三段階 policy 完遂

| 段階 | 条件 | day | status |
|---|---|---|---|
| 1 warning | rank 1 = 5 | D29 (2026-06-23) | ✓ 発動 + sim 実施 |
| 2 sim | rank 1 = 5-6 | D29 | ✓ 完了 |
| 3 **本実行** | rank 1 = 6 (= 7 連続前) | **D30 (= 本日)** | **★★★ 完遂 ★★★** |

### 5.3 D29 sim 予測 vs D30 本実行 一致確認 (= Codex Lane A YES)

| rank | D29 sim 予測 | D30 本実行結果 | 一致 |
|---|---|---|---|
| 1 | 9130 | 9130 | ✓ |
| 2 | 331A0 | 331A0 | ✓ |
| 3 | 4389 | 4389 | ✓ |
| 4 | 9247 | 9247 | ✓ |
| 5 | 4317 | 4317 | ✓ |

→ **完全一致** (= sim 信頼性実証、demote policy 設計妥当)

## §6 D30 entry 候補 (= 7991 demote 後の新 top 5)

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| **1** ★ | **9130** | **共栄タンカー** | **運輸・物流** | 1,410 | 7,050 | boost_with_caution (= 新 #1) |
| 2 | **331A0** | メディックス | **情報通信** | 482 | 2,410 | boost_with_caution |
| 3 | **4389** | プロパティデータバンク | 情報通信 | 863 | 4,315 | boost_with_caution (= 新登場) |
| 4 (参考) | 9247 | ＴＲＥホールディングス | 情報通信 | 1,612 | 8,060 | boost_with_caution |
| 5 (参考) | 4317 | レイ | 情報通信 | 508 | 2,540 | boost_with_caution |

★ rank 1 推奨: **9130 共栄タンカー** (= 新 #1、運輸 sector、risk 中位)

## §7 ⚠ sector diversification 一時 2 種縮退 (= 主要変化点)

| 観点 | D25-D29 (= 3 種維持期) | D30 (= 本日、7991 demote 後) |
|---|---|---|
| rank 1 | 機械 (7991) | **運輸 (9130)** |
| rank 2 | 運輸 (9130) | **情報通信 (331A0)** |
| rank 3 | 情報通信 (331A0) | **情報通信 (4389)** |
| sector 種数 | **3 種** ✓ | **2 種** (= 運輸 + 情報通信) ⚠ |
| 喪失 sector | n/a | **機械 (= 7991 demote による)** |

### 7.1 評価
- HOLD #2 阻止 = 最優先 → sector 縮退は許容
- 機械 sector 候補が今後登場すれば再 3 種化
- 7991 が rank 7+ 以下に下がるか経過監視 (= D31 で確認)

## §8 HOLD #2 完全回避 (= D15 以降 15 連続継続) ✓

- D15 HOLD #2 発動 (= 340A0 7 連続) 以降、HOLD 再発動 0
- D30 時点で 15 連続 HOLD 完全回避
- 7991 demote で 6 連続到達直前で阻止、HOLD #2 再発 0 維持

## §9 next recurrence warning candidate (= 9130 monitor 開始)

| 銘柄 | D30 status | rank 1 連続 | 段階 1 warning 予想 | 段階 3 本実行予想 |
|---|---|---|---|---|
| **9130** (運輸、新 #1) | rank 1 (1 連続) | 1 | **D34 想定** (= +4 wave) | **D35** |
| 331A0 (情報通信) | rank 2 (1 連続) | 0 (= rank 2) | n/a | n/a |
| 4389 (情報通信) | rank 3 (= 新登場) | 0 | n/a | n/a |

## §10 price freshness summary

- artifact_source: f062_preview
- f062_raw_kind: f062_actual_dict
- f111_input_source: f111_real_batch
- freshness_verdict: **MISSING** (= staging max=5/14 制約継続)
- AFTER-R1 9 invariants 8/9 PASS

## §11 source artifact paths

- D29 paper PnL reload: `/tmp/fire_d30_prep/d29_paper_pnl_reload.{json,md}`
- F111: `/tmp/fire_d30_prep/d30_f111.json`
- paper PnL: `/tmp/fire_d30_prep/d30_paper_pnl.json`
- paper PnL Markdown: `/tmp/fire_d30_prep/d30_paper_pnl.md`

## §12 D30 hard check (= chain-level 11/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1 | artifact_source=f062_preview | ✓ |
| 2 | f062_raw_kind=f062_actual_dict | ✓ |
| 3 | f111_input_source=f111_real_batch | ✓ |
| 4 | forbidden_check passed | ✓ |
| 5 | auto_order_allowed=false | ✓ |
| 6 | manual_review_required=true | ✓ |
| 7 | safety_flags all false | ✓ |
| 8 | top_candidates ≥ 1 | ✓ (= 5) |
| 9 | non-sample ticker | ✓ |
| 10 | tradable_universe=true | ✓ |
| 11 | risk_within_pilot_limit=true | ✓ |
| 12 | 朝 actual / liquidity / event | Fujiwara 朝待ち |

## §13 actual price + liquidity + event (= 朝記入)

| code | actual 6/24 | 乖離率 | liquidity | event | OK/NG |
|---|---|---|---|---|---|
| 9130 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG |
| 331A0 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG |
| 4389 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG |

## §14 final decision (= 朝寄付き前)

- ☐ **enter 9130 (= 運輸、新 #1 推奨)** ★
- ☐ enter 331A0 (= 情報通信、新 #2)
- ☐ enter 4389 (= 情報通信、新 #3、新登場)
- ☐ watch (= 朝確認で見送り)
- ☐ skip (= event/liquidity NG)

## §15 safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
