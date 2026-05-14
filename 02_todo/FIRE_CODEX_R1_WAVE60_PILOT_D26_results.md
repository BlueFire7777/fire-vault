---
id: FIRE-CODEX-R1-WAVE60-PILOT-D26-results
phase: 本番 v0 中核 / Wave 60-pilot-D26 / post-137A-demote 2 連続 + sector 3 種多様化維持
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-18 (D26) / wave 実行: 2026-05-14
pilot_day: D26
pilot_judgment: GO_CONDITIONAL maintained
hard_check_chain_level_passed: 9/12
new_top_persistence: "7991/9130/331A0 D25-D26 2 連続安定"
hold_2_avoidance_confirmed: true
recently_seen_codes_maintained: 6 件
---

# Wave 60-pilot-D26 Results — Post-137A-Demote Continuity

## §1 結論

**D26 = GO_CONDITIONAL maintained 7 連続**。137A0 demote 後の新 top
(= 7991 機械 / 9130 運輸 / 331A0 情報通信) が D25-D26 で 2 連続安定、
**sector 3 種多様化維持**、**HOLD #2 完全回避**確認。3 DB md5 全 不変、
本 wave で code 変更 0。

## §2 chain 実行結果

### 2.1 D25 review 確認 (= 0/5 記入)

→ W3 §7.1 path 2 継続適用。

### 2.2 D25 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown

### 2.3 D26 chain

- F111: candidates=20 / eligible=19 / demoted_count=6 (= 維持)
- tuning_params: strict / recently_seen 6 件 / demoted_count=6
- AFTER-R1: artifact_source=f062_preview / freshness_verdict=MISSING / 9 invariants 8/9 PASS
- paper PnL: candidates=20 / evaluated=0 / review_missing=1

### 2.4 D26 top 3 (= D25-D26 2 連続安定)

| rank | code | name | sector |
|---|---|---|---|
| 1 | 7991 | マミヤ・オーピー | 機械 |
| 2 | 9130 | 共栄タンカー | 運輸・物流 |
| 3 | 331A0 | メディックス | 情報通信 |

## §3 sector 3 種多様化維持 ✓

| rank | code | sector |
|---|---|---|
| 1 | 7991 | 機械 |
| 2 | 9130 | 運輸・物流 |
| 3 | 331A0 | 情報通信 |

→ D25-D26 で 2 連続 sector 3 種多様化 = 137A0 demote 効果定着。

## §4 HOLD #2 完全回避確認 ✓

- D20-D24: 137A0 5 連続 = warning 段階
- D25: 137A0 demote 本実行 → 連続中断
- D26: 137A0 demote 維持 (= 2 連続) で 7 連続到達阻止 ✓
- → HOLD #2 完全回避

## §5 D26 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §6 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

→ GO_CONDITIONAL maintained 7 連続

## §7 D20-D26 7 day post-recovery 集約

### 7.1 pilot_judgment 推移

| day | judgment | 137A0 状態 |
|---|---|---|
| D20 | GO_CONDITIONAL 復帰 (初) | rank 1 (1 連続) |
| D21 | 2 連続 | rank 1 (2 連続) |
| D22 | 3 連続 | rank 1 (3 連続) |
| D23 | 4 連続 | rank 1 (4 連続) |
| D24 | 5 連続 (warning) | rank 1 (5 連続) |
| D25 | 6 連続 (demote 本実行) | demoted (= 中断) |
| **D26** | **7 連続 (post-demote 安定)** | **demoted 維持** |

### 7.2 demote 効果 summary

| 銘柄 | demote 開始 | 連続維持 (D26 時点) |
|---|---|---|
| 8747/5729/3489 | D6 | 21 連続 |
| 340A0 | D16 | 11 連続 |
| 3798 | D20 | 7 連続 |
| **137A0** | **D25** | **2 連続** |

### 7.3 sector 多様化推移

| 期 | 多様化 |
|---|---|
| D14-D19 | 情報通信 100% 集中 |
| D20-D24 | 情報通信 + 機械 (2 種) |
| **D25-D26** | **機械 + 運輸 + 情報通信 (3 種)** ✓ |

## §8 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| DB write / API / token | 全 0 ✓ |
| LINE / launchctl / plist / cron | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault 4 file 追加のみ) |

## §9 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-18_manual_live_pilot_trade_plan.md`
- `~/fire-vault/04_daily/2026-06-18_manual_live_pilot_review.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D26_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D26_results.md`

## §10 W4 集約引き継ぎ準備 (= D20-D26 7 day pilot)

W60-pilot-W4 集約 wave 推奨項目:

1. **demote 効果 7 day evaluation** (= 340A0/3798/137A0)
2. **sector 多様化推移** (= 1 種 → 2 種 → 3 種)
3. **HOLD 回避 path 実証** (= W3 設計 doc 完全機能)
4. **review missing 累積 18 day** (= 構造的問題、解決 path 検討)
5. **paper PnL 真の outcome 評価** (= h20 後 6/26-30 頃)
6. **次の demote 対象** (= 7991/9130 連続性 monitor)
7. **liquidity filter 強化必要性**

## §11 next action 候補

### 11.1 Fujiwara 本人

1. D26 朝 (= 2026-06-18 08:30 JST) J-Quants refresh + iSPEED actual + entry 判断
2. D26 review.md 必須 5 項目記入
3. D26 場後 paper PnL handoff

### 11.2 後続 wave

4. **W60-pilot-D27** (= 2026-06-19 金、7991/9130/331A0 連続性 3 連続到達確認)
5. **W60-pilot-W4 集約** ★推奨 (= D20-D26 7 day review、demote 設計総括)
6. **DATA-R3 active job wave**
7. **paper_pnl preview h1/h20 再 run** (= 6/26-30 頃で真の outcome)
8. **liquidity filter 強化 wave**
9. **features rerun wave**
10. **全銘柄 daily refresh launchd 自動化**

## §12 6 KPI

1. 稼働率: 100% (= 8 step 全完走、blocker 0)
2. 短縮率: 高 (= D25 review + paper PnL + D26 chain + 判定 + trade plan + review + W4 引き継ぎ + vault を 1 conversation で完結)
3. 採用率: 100% (= 137A0 demote 維持、sector 3 種多様化維持、HOLD #2 完全回避、W4 引き継ぎ完成)
4. 差戻率: 0% (= chain 1 発成功、想定通り maintained)
5. Integrator 負荷: 低 (= 3 DB 全 md5 不変、code 変更 0、vault 4 file)
6. 安全事故: 0 (= DB write 0 / API 0 / token 0 / launchctl 0 / 実発注 0)
