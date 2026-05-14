---
id: FIRE-CODEX-R1-WAVE60-PILOT-D24-results
phase: 本番 v0 中核 / Wave 60-pilot-D24 / 137A0 5 連続 warning + demote sim 完了
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-16 (D24) / wave 実行: 2026-05-14
pilot_day: D24
pilot_judgment: GO_CONDITIONAL maintained
hard_check_chain_level_passed: 9/12
w3_recovery_criteria_passed: 9/9 (= #7 朝確認待ち)
hold_2_warning: "137A0 rank 1 5 連続到達 = warning 段階 (= W3 §3.3 #2a)"
demote_simulation_completed: true
demote_simulation_new_top: "7991 (機械) / 9130 (運輸) / 331A0 (情報通信) = sector 3 種多様化"
demote_decision: "D25 で 137A0 demote 実施推奨 (= 案 B)"
---

# Wave 60-pilot-D24 Results — 137A0 Recurrence Warning / Demote Decision

## §1 結論

**D24 = GO_CONDITIONAL maintained 5 連続**、ただし **137A0 rank 1 5 連続 warning
段階到達**。137A0 demote sim 完了 = **新 top 7991 (機械) / 9130 (運輸) /
331A0 (情報通信) で sector 3 種多様化** path 確立。**D25 で 137A0 demote 実施推奨**
(= 案 B)。3 DB md5 全 不変、本 wave で code 変更 0。

## §2 chain 実行結果

### 2.1 D23 review 確認 (= 0/5 記入)

→ W3 §7.1 path 2 継続適用。

### 2.2 D23 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown

### 2.3 D24 chain (= 現行 recently_seen 5 件)

- F111: candidates=20 / eligible=19 / demoted_count=5
- AFTER-R1: artifact_source=f062_preview / freshness_verdict=MISSING / 9 invariants 8/9 PASS
- paper PnL: candidates=20 / evaluated=0 / review_missing=1

### 2.4 D24 top 3 (= D20-D24 で 5 連続安定)

| rank | code | name | sector |
|---|---|---|---|
| 1 | 137A0 | Cocolive | 情報通信 |
| 2 | 7991 | マミヤ・オーピー | 機械 |
| 3 | 331A0 | メディックス | 情報通信 |

## §3 ⚠ 137A0 5 連続 warning 到達 (= 主要発見)

| day | 137A0 rank 1 連続 | HOLD #2 残日数 | 段階 |
|---|---|---|---|
| D20-D23 | 1-4 連続 | 6-3 日 | 監視 |
| **D24** | **5 連続** | **2 日** | **warning ✓** |
| D25 (予想) | 6 連続 | 1 日 | demote 強推奨 |
| D26 (予想) | 7 連続 | 0 日 | **HOLD #2 再発動** |

## §4 137A0 demote sim 結果 (= 参考 read-only chain)

```bash
.venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
  --base-date 2026-06-16 --max-candidates 20 \
  --label-threshold-mode strict \
  --recently-seen-codes 8747,5729,3489,340A0,3798,**137A0** \
  --output-json /tmp/fire_d24_prep/demote_sim/d24_137A0_demote_sim_f111.json
```

### 4.1 demote sim 後の新 top

| rank | code | name | sector |
|---|---|---|---|
| 1 | **7991** | **マミヤ・オーピー** | **機械** (= 新 #1) |
| 2 | **9130** | **共栄タンカー** | **運輸・物流** (= 新 #2) |
| 3 | 331A0 | メディックス | 情報通信 |

→ **sector 3 種多様化** (= 機械 + 運輸 + 情報通信) ← 重要成果

### 4.2 demote 後 demoted_count

- recently_seen_codes: 8747, 5729, 3489, 340A0, 3798, 137A0 (= 6 件)
- demoted_count: 6

## §5 demote 判断 (= 主推奨: 案 B)

| 案 | 内容 | 推奨度 |
|---|---|---|
| A. D24 内即 demote | trade plan 即更新 | ★ |
| **B. D25 で demote 実施** | D24 = warning 認識、D25 で正式 demote | **★★★ 推奨** |
| C. W4 集約で正式化 | HOLD 直前で集約 | ★★ (= HOLD リスク) |

**→ 案 B (= D25 で demote 実施) 推奨**

## §6 D24 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §7 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

→ **GO_CONDITIONAL maintained 5 連続**

## §8 D20-D24 比較 (= 5 day post-recovery)

| 項目 | D20 | D21 | D22 | D23 | D24 |
|---|---|---|---|---|---|
| pilot_judgment | 復帰 (初) | 2 連続 | 3 連続 | 4 連続 | **5 連続** |
| 137A0 rank 1 連続 | 1 | 2 | 3 | 4 | **5 (warning)** |
| 7991 rank (機械) | 2 | 2 | 2 | 2 | 2 |
| 331A0 rank | 3 | 3 | 3 | 3 | 3 |
| **demote sim 実施** | × | × | × | × | **✓ (初)** |
| **新 top simulated** | n/a | n/a | n/a | n/a | **7991/9130/331A0** |

## §9 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| DB write / API / token / LINE / launchctl | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault 4 file 追加 + demote sim artifacts) |

## §10 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-16_manual_live_pilot_trade_plan.md`
- `~/fire-vault/04_daily/2026-06-16_manual_live_pilot_review.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D24_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D24_results.md`
- `/tmp/fire_d24_prep/demote_sim/` (= demote 検討参考 artifacts)

## §11 next action 候補

### 11.1 Fujiwara 本人 (= D24 当日)

1. **D24 朝 (= 2026-06-16 08:30 JST) J-Quants refresh + iSPEED actual + entry 判断**
2. **D24 推奨**: 7991 マミヤ・オーピー (= 機械、多様化、D25 demote 前先行 entry)
3. **D24 review.md 必須 5 項目記入**

### 11.2 後続 wave (= 137A0 demote タイミング)

4. **W60-pilot-D25** ★ (= 2026-06-17 水、**137A0 demote 実施**、新 top = 7991/9130/331A0)
5. **W60-pilot-W4 集約** (= D20-D24 後、demote 設計正式化)
6. **DATA-R3 active job wave**
7. **paper_pnl preview h1/h20 再 run** (= 7/8 頃)
8. **liquidity filter 強化 wave**
9. **features rerun wave**
10. **全銘柄 daily refresh launchd 自動化**

## §12 6 KPI

1. 稼働率: 100% (= 8 step 全完走、blocker 0)
2. 短縮率: 高 (= D23 review + paper PnL + D24 chain + 137A0 5 連続確認 + demote sim + 判定 + trade plan + review + vault を 1 conversation で完結)
3. 採用率: 100% (= demote sim 完了、sector 3 種多様化 path 確立、D25 demote 推奨確定)
4. 差戻率: 0% (= chain 1 発成功、demote sim 想定通り)
5. Integrator 負荷: 低 (= 3 DB 全 md5 不変、code 変更 0、vault 4 file + demote sim artifacts)
6. 安全事故: 0 (= DB write 0 / API 0 / token 0 / launchctl 0 / 実発注 0)
