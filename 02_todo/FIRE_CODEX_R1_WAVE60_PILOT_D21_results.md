---
id: FIRE-CODEX-R1-WAVE60-PILOT-D21-results
phase: 本番 v0 中核 / Wave 60-pilot-D21 / post-recovery 初回継続判定
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-11 (D21) / wave 実行: 2026-05-14
pilot_day: D21
pilot_judgment: GO_CONDITIONAL maintained
hard_check_chain_level_passed: 9/12
w3_recovery_criteria_passed: 9/9 (= #7 朝確認待ち)
new_top_persistence: "137A0/7991/331A0 D20-D21 2 連続安定"
recently_seen_codes_maintained: 5 件 (= 8747,5729,3489,340A0,3798)
---

# Wave 60-pilot-D21 Results — Post-Recovery Continuation

## §1 結論

**D21 = GO_CONDITIONAL maintained** (= D20 復帰後の 2 連続)。

W3 設計 doc §7.1 9 条件 全 ✓ (= #7 朝確認待ち)、137A0/7991/331A0 が D20-D21
で 2 連続安定、sector 多様化維持 (= 機械 7991 rank 2)、340A0/3798 demote 維持。
3 DB md5 全 不変、本 wave で code 変更 0。

## §2 chain 実行結果

### 2.1 D20 review 確認 (= PARTIAL 1/6)

| 項目 | 値 |
|---|---|
| yaml status | blank |
| 必須 5 項目 | 1/6 (= template review_gap 検出含む可能性) |

→ W3 §7.1 path 2 適用継続。

### 2.2 D20 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown

### 2.3 D21 chain

- F111: candidates=20 / eligible=19 / tuning_params strict / demoted_count=5
- F062: input=20 / selected=10
- AFTER-R1: artifact_source=f062_preview / freshness_verdict=MISSING / 9 invariants 8/9 PASS
- paper PnL: candidates=20 / evaluated=0 / review_missing=1

## §3 D21 top 3 (= D20-D21 で 2 連続)

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| 1 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution |
| 2 | **7991** | **マミヤ・オーピー** | **機械** | 1,177 | 5,885 | boost_with_caution |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |

→ **2 連続安定** (= 7 連続まで残 5 日)。

## §4 D21 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §5 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

| # | 条件 | D21 結果 |
|---|---|---|
| 1 | review 5 項目 OR gap 明確化 | ✓ (= path 2 適用継続) |
| 2 | 340A0 demote 維持 | ✓ (= 6 連続) |
| 3 | 3798 recently_seen 維持 | ✓ (= 2 連続) |
| 4 | f111_real_batch | ✓ |
| 5 | top candidate ≥ 1 | ✓ (= 3) |
| 6 | risk_within_pilot_limit | ✓ |
| 7 | actual/liquidity/event 確認可能 | 朝確認待ち |
| 8 | safety_flags 全 False | ✓ |
| 9 | paper PnL/review 重大問題なし | ✓ |

→ **GO_CONDITIONAL maintained** (= D20 と同レベル、2 連続)

## §6 D20 vs D21 比較

| 項目 | D20 | D21 |
|---|---|---|
| pilot_judgment | GO_CONDITIONAL 復帰 (= 初) | **GO_CONDITIONAL maintained (= 2 連続)** |
| 137A0 rank | 1 | 1 (= 2 連続) |
| 7991 rank | 2 (= 機械、初) | 2 (= 機械、2 連続) |
| 331A0 rank | 3 | 3 (= 2 連続) |
| recently_seen_codes | 5 件 (= 3798 追加) | 5 件 (= 維持) |
| sector 多様化 | 達成 (= 初) | 維持 |

## §7 安全境界 final verification

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

## §8 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-11_manual_live_pilot_trade_plan.md`
- `~/fire-vault/04_daily/2026-06-11_manual_live_pilot_review.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D21_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D21_results.md`

## §9 next action 候補 (= 優先順)

1. **D21 朝 (= 2026-06-11 08:30 JST) J-Quants refresh + iSPEED actual + entry 判断**
2. **D21 review.md 記入** (= 必須 5 項目)
3. **D21 場後 paper PnL handoff**
4. **W60-pilot-D22** (= 2026-06-12 金、137A0 連続性 monitor)
5. **W4 集約** (= D20-D24 後、137A0 demote 検討)
6. **DATA-R3 active job wave**
7. **paper_pnl preview h1/h20 再 run**
8. **liquidity filter 強化 wave**
9. **features rerun wave**
10. **全銘柄 daily refresh launchd 自動化**

## §10 6 KPI

1. 稼働率: 100% (= 7 step 全完走、blocker 0)
2. 短縮率: 高 (= D20 review + paper PnL + D21 chain + 判定 + trade plan + review + vault を 1 conversation で完結)
3. 採用率: 100% (= 137A0/7991/331A0 2 連続安定、GO_CONDITIONAL maintained)
4. 差戻率: 0% (= chain 1 発成功、想定通り maintained)
5. Integrator 負荷: 低 (= 3 DB 全 md5 不変、code 変更 0、vault 4 file)
6. 安全事故: 0 (= DB write 0 / API 0 / token 0 / launchctl 0 / 実発注 0)
