---
id: FIRE-CODEX-R1-WAVE60-PILOT-D25-results
phase: 本番 v0 中核 / Wave 60-pilot-D25 / 🎉 137A0 demote 本実行 + sector 3 種多様化達成
priority: 最高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-17 (D25) / wave 実行: 2026-05-14
pilot_day: D25
pilot_judgment: GO_CONDITIONAL maintained
hard_check_chain_level_passed: 9/12
demotion_initiative: "137A0 demote 本実行 = D24 sim 検証通り"
new_top:
  rank_1: 7991 マミヤ・オーピー (機械)
  rank_2: 9130 共栄タンカー (運輸・物流)
  rank_3: 331A0 メディックス (情報通信)
sector_diversification: "機械 + 運輸 + 情報通信 = 3 種多様化完全達成"
recently_seen_codes: ["8747", "5729", "3489", "340A0", "3798", "137A0"]
demoted_count: 6
---

# Wave 60-pilot-D25 Results — 137A0 Demote / Sector Diversification

## §1 結論

🎉 **137A0 demote 本実行成功 + sector 3 種多様化完全達成**。

D24 demote sim 結果通り、新 top = **7991 (機械) / 9130 (運輸) / 331A0 (情報通信)**
で sector 3 種多様化が完全達成。HOLD #2 リスク (= 137A0 7 連続到達) を完全回避、
GO_CONDITIONAL maintained 6 連続維持。3 DB md5 全 不変、本 wave で code 変更 0。

## §2 chain 実行結果

### 2.1 D24 review 確認 (= 0/5 記入)

→ W3 §7.1 path 2 継続適用。

### 2.2 D24 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown

### 2.3 D25 chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-06-17 --max-candidates 20 \
    --label-threshold-mode strict \
    --recently-seen-codes 8747,5729,3489,340A0,3798,137A0 \
    ...
```

- F111: candidates=20 / eligible=19 / **demoted_count=6** (= D24 から +1)
- tuning_params: strict / recently_seen 6 件 / demoted_count=6
- F062: input=20 / selected=10
- AFTER-R1: artifact_source=f062_preview / freshness_verdict=MISSING / 9 invariants 8/9 PASS
- paper PnL: candidates=20 / evaluated=0 / review_missing=1

## §3 137A0 demote 本実行効果 (= 主要成果 ✓)

### 3.1 F111-real-batch top 7 (= 137A0 caution へ)

| rank | code | label | demoted |
|---|---|---|---|
| 1 | 8747 | caution | True |
| 2 | 5729 | caution | True |
| 3 | 3489 | caution | True |
| 4 | 340A0 | caution | True (10 連続) |
| 5 | 3798 | caution | True (6 連続) |
| **6** | **137A0** | **caution** | **True (= D25 で初 demote、本実行)** |
| **7** | **7991** | **boost_with_caution** | **False ← 新 #1 候補** |

### 3.2 AFTER-R1 top_candidates 完全切替 ✓

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D20-D24 | 137A0 (情報通信) | 7991 (機械) | 331A0 (情報通信) |
| **D25** | **7991 マミヤ・オーピー (機械) ✓** | **9130 共栄タンカー (運輸・物流) ✓** | **331A0 メディックス (情報通信)** |

→ D24 demote sim 結果と完全一致 = 137A0 demote 効果実証

### 3.3 sector 3 種多様化完全達成 🎉

| rank | code | sector |
|---|---|---|
| 1 | **7991** | **機械** |
| 2 | **9130** | **運輸・物流** ← 初登場 |
| 3 | 331A0 | 情報通信 |

→ **D20-D24 で 100% 情報通信集中 → D25 で機械 + 運輸 + 情報通信の 3 種多様化**

## §4 D25 entry 候補

| rank | code | name | sector | close | risk_yen |
|---|---|---|---|---|---|
| **1** | **7991** | **マミヤ・オーピー** | **機械** | 1,177 | 5,885 |
| 2 | **9130** | **共栄タンカー** | **運輸・物流** | 1,410 | 7,050 |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 2,410 |

## §5 D25 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §6 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

→ GO_CONDITIONAL maintained 6 連続

## §7 D20-D24 連続性 + D25 demote 比較

| 項目 | D20-D24 | D25 |
|---|---|---|
| pilot_judgment | GO_CONDITIONAL 1-5 連続 | **6 連続 maintained** |
| 137A0 rank | 1 (5 連続) | **demoted (rank 6)** |
| AFTER-R1 rank 1 | 137A0 | **7991 (新 #1 機械)** |
| AFTER-R1 rank 2 | 7991 (= 機械、多様化) | **9130 (= 運輸、初登場)** |
| AFTER-R1 rank 3 | 331A0 | 331A0 (= 維持) |
| sector | 情報通信中心 + 機械 1 | **3 種完全多様化** |
| recently_seen 件数 | 5 | **6** |

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

- `~/fire-vault/04_daily/2026-06-17_manual_live_pilot_trade_plan.md`
- `~/fire-vault/04_daily/2026-06-17_manual_live_pilot_review.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D25_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D25_results.md`

## §10 next action 候補

### 10.1 Fujiwara 本人

1. **D25 朝 (= 2026-06-17 08:30 JST) J-Quants refresh + iSPEED actual + entry 判断**
2. **D25 推奨**: 7991 マミヤ・オーピー (= 新 #1 機械、推奨)
3. **D25 review.md 必須 5 項目記入**

### 10.2 後続 wave

4. **W60-pilot-D26** (= 2026-06-18 木、7991/9130/331A0 連続性 monitor、HOLD 完全回避確認)
5. **W60-pilot-W4 集約** ★推奨 (= D20-D25 後、137A0 demote 効果実証、設計改訂)
6. **DATA-R3 active job wave**
7. **paper_pnl preview h1/h20 再 run** (= 7/8 頃)
8. **liquidity filter 強化 wave**
9. **features rerun wave**
10. **全銘柄 daily refresh launchd 自動化**

## §11 6 KPI

1. 稼働率: 100% (= 6 step 全完走、blocker 0)
2. 短縮率: 高 (= D24 review + paper PnL + D25 chain + 137A0 demote 本実行 + 判定 + trade plan + review + vault を 1 conversation で完結)
3. 採用率: 100% (= 137A0 demote 成功、sector 3 種多様化、HOLD #2 リスク回避、7991/9130/331A0 新 top)
4. 差戻率: 0% (= D24 demote sim 結果と完全一致、chain 1 発成功)
5. Integrator 負荷: 低 (= 3 DB 全 md5 不変、code 変更 0、vault 4 file)
6. 安全事故: 0 (= DB write 0 / API 0 / token 0 / launchctl 0 / 実発注 0)

## §12 設計成果総括

| 項目 | 達成 |
|---|---|
| W3 設計 doc HOLD criteria 改訂 | ✅ 実証 (= D15 初発動 → D20 復帰 → D24 warning → D25 demote) |
| 340A0 demote (D16) | ✅ 維持 10 連続 |
| 3798 demote (D20) | ✅ 維持 6 連続 |
| **137A0 demote (D25)** | **✅ 本実行成功** |
| **sector 3 種多様化達成** | **✅ 機械 + 運輸 + 情報通信** |
| recently_seen 件数推移 | 3 → 4 → 5 → 6 (= 段階的拡張、改訂 path 機能) |
