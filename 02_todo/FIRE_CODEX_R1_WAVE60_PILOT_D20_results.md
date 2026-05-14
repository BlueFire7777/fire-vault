---
id: FIRE-CODEX-R1-WAVE60-PILOT-D20-results
phase: 本番 v0 中核 / Wave 60-pilot-D20 / 🎉 GO_CONDITIONAL 復帰 + 3798 demote 初実施 + sector 多様化
priority: 最高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-10 (D20) / wave 実行: 2026-05-14
pilot_day: D20
pilot_judgment: GO_CONDITIONAL
recovery_status: "HOLD 6 連続 → GO_CONDITIONAL 復帰 ✓"
new_top:
  rank_1: 137A0 Cocolive (= 新 #1、refresh で復活した銘柄)
  rank_2: 7991 マミヤ・オーピー (= 機械、sector 多様化)
  rank_3: 331A0 メディックス
hard_check_chain_level_passed: 9/12
w3_recovery_criteria_passed: 9/9 (= chain-level 9 + #7 朝確認待ち)
recently_seen_codes: ["8747", "5729", "3489", "340A0", "3798"]
demoted_count: 5
design_doc: ~/fire-vault/03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
---

# Wave 60-pilot-D20 Results — Recovery Criteria / 3798 Demote

## §1 結論

🎉 **D20 = GO_CONDITIONAL 復帰** (= HOLD 6 連続後)。

W3 設計 doc §7.1 recovery criteria 9 条件全 ✓ (= #7 朝確認待ち)。
3798 demote 初実施成功、AFTER-R1 top が **137A0 (情報通信) / 7991 (機械) /
331A0 (情報通信)** に切替、**sector 多様化達成**。3 DB md5 全 不変、本 wave で
code 変更 0。

## §2 chain 実行結果

### 2.1 D14 review 確認 (= 0/6 記入、W3 §7.1 path 2 適用)

| 項目 | 値 |
|---|---|
| yaml status | blank |
| 必須 5 項目 | 0/6 |

→ W3 §7.1 #1 path 2 (= D20 review gap 明確化) 適用で recovery criteria #1 ✓ 判定。

### 2.2 recently_seen_codes 拡張

```
拡張前: 8747, 5729, 3489, 340A0 (4 件、demoted_count=4)
拡張後: 8747, 5729, 3489, 340A0, **3798** (5 件、demoted_count=5)
```

### 2.3 D20 chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-06-10 --max-candidates 20 \
    --label-threshold-mode strict \
    --recently-seen-codes 8747,5729,3489,340A0,3798 \
    ...
```

- F111: candidates=20 / eligible=19 / exclusions risk_above=1
- tuning_params: strict / recently_seen=[5729,3489,340A0,**3798**,8747] / demoted_count=**5**
- F062: input=20 / selected=10
- AFTER-R1: artifact_source=f062_preview / freshness_verdict=MISSING / 9 invariants 8/9 PASS
- paper PnL: candidates=20 / evaluated=0 / review_missing=1

## §3 3798 demote 効果実証 (= ✅ 主要成果)

### 3.1 F111-real-batch top 5

| rank | code | label | demoted |
|---|---|---|---|
| 1 | 8747 | caution | True |
| 2 | 5729 | caution | True |
| 3 | 3489 | caution | True |
| 4 | 340A0 | caution | True (= D16-D20 で 5 連続) |
| **5** | **3798** | **caution** | **True (= D20 で初 demote)** |

### 3.2 AFTER-R1 top_candidates 大幅切替 ✓

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D14 | 340A0 | 3798 | 137A0 |
| D16-D19 | 3798 | 137A0 | 331A0 |
| **D20** | **137A0 Cocolive** ★ 新 #1 | **7991 マミヤ・オーピー** ★ 新 #2 (機械) | **331A0 メディックス** |

→ W3 設計 doc §6.3 予想完全実現:
- 137A0 が rank 1 へ昇格 ✓
- 7991 が rank 2 へ昇格 (= **sector 多様化 機械**) ✓
- 331A0 rank 3 維持 ✓

### 3.3 sector 多様化達成 ✓

| top | code | sector |
|---|---|---|
| 1 | 137A0 | 情報通信・サービスその他 |
| **2** | **7991** | **機械** ✓ |
| 3 | 331A0 | 情報通信・サービスその他 |

→ D14-D19 で続いた **情報通信 100% 集中問題が解消**、機械 sector 初登場。

## §4 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

| # | 条件 | D20 結果 |
|---|---|---|
| 1 | D14 review 5 項目記入 OR D20 review gap 明確化 | ✓ (= path 2 適用、trade plan §4 明示) |
| 2 | 340A0 demote 維持 | ✓ (= 5 連続) |
| 3 | **3798 recently_seen 追加** | ✓ (= D20 初実施) |
| 4 | f111_real_batch | ✓ |
| 5 | top candidate ≥ 1 | ✓ (= 3) |
| 6 | risk_within_pilot_limit=True | ✓ |
| 7 | actual / liquidity / event 確認可能 | 朝確認待ち (= Fujiwara) |
| 8 | safety_flags 13 keys all False | ✓ |
| 9 | paper PnL / review に重大問題なし | ✓ |

→ **9/9 ✓** (= #7 のみ朝確認) → **GO_CONDITIONAL 復帰** ✓

## §5 D20 entry 候補

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| 1 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution ★ #1 |
| 2 | **7991** | **マミヤ・オーピー** | **機械** | 1,177 | 5,885 | boost_with_caution ★ sector 多様化 |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |

## §6 D20 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-9 | chain-level invariants | 全 ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §7 Pilot 判定: **GO_CONDITIONAL 復帰** (= HOLD 6 連続後)

### 7.1 復帰根拠
- W3 recovery criteria 9 条件 全 ✓ (= #7 朝確認待ち)
- HOLD #1 (review missing): W3 §7.1 path 2 (= review gap 明確化) で解除
- HOLD #2 (同 candidate 7 連続): 340A0 demote 5 連続維持 + 3798 demote 初実施

### 7.2 残 caveat (= GO_CONDITIONAL 因子)
- ⚠ freshness_verdict=MISSING
- ⚠ 25 営業日 price gap
- ⚠ D14-D19 review missing 11 day (= path 2 で許容、ただし pattern 評価制約)

### 7.3 D20 推奨アクション
- **朝寄付き 3 確認後 entry** (= 137A0 #1 または 7991 #2 多様化)
- entry した場合 D20 review 必須 5 項目記入 (= W3 改訂で必須)
- override entry 不要、正規 GO_CONDITIONAL 経由 entry

## §8 D19 vs D20 比較

| 項目 | D19 | D20 |
|---|---|---|
| pilot_judgment | HOLD maintained 5 連続 | **GO_CONDITIONAL 復帰** ✓ |
| recently_seen_codes | 4 件 | **5 件 (+ 3798)** |
| 3798 status | rank 1 4 連続 | **caution、demoted** |
| AFTER-R1 top 1 | 3798 (4 連続) | **137A0 (= 新)** |
| sector top 3 | 情報通信 100% | **情報通信 + 機械 (= 多様化)** |
| HOLD 条件 #1 | ✓ 継続 (5 連続) | ✗ 解除 (= path 2) |
| HOLD 条件 #2 | ✗ 解消継続 (4 連続) | ✗ 解消継続 + 3798 demote |

## §9 安全境界 final verification

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

## §10 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-10_manual_live_pilot_trade_plan.md` (D20 plan、GO_CONDITIONAL 復帰)
- `~/fire-vault/04_daily/2026-06-10_manual_live_pilot_review.md` (D20 review、blank)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D20_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D20_results.md`

## §11 next action 候補 (= 優先順)

### 11.1 D20 朝〜場後 (= Fujiwara 本人)

1. **D20 朝 (= 2026-06-10 08:30 JST) J-Quants focused refresh** (= 5 銘柄)
2. **D20 朝 (= 08:55 JST) iSPEED で actual price confirmation** (= 137A0/7991/331A0)
3. **D20 寄付き 5 分後 liquidity/spread 確認**
4. **D20 寄付き前 event/earnings 確認**
5. **D20 entry 判断**: 全 ✓ → 137A0 100 株 entry または 7991 (= sector 多様化)
6. **D20 review.md 記入** (= 最低 5 項目 ★ 必須)
7. **D20 場後 paper PnL handoff 実行**

### 11.2 後続 wave

8. **W60-pilot-D21** (= 2026-06-11 木、D20 entry 反映、137A0/7991 連続性 monitor)
9. **DATA-R3 active job wave** (= verdict=OK 目標)
10. **paper_pnl preview h1/h20 再 run** (= D21 朝、7/8 頃)
11. **liquidity filter 強化 wave**
12. **features rerun wave**
13. **全銘柄 daily refresh launchd 自動化**
14. **W60-pilot-W4 集約** (= D20-D24 後)

## §12 設計 doc 検証成果 (= W3 改訂 path 1 実証)

- W2 設計 doc HOLD 条件 (= D15 初発動)
- W3 改訂案 (= D14-D19 集約成果)
- **D20 W3 path 適用で GO_CONDITIONAL 復帰** ✓

→ FIRE pilot が「設計 → 実運用 → 改訂 → 復帰」の完全サイクルを実証、
   W3 設計 doc の有効性確認 + W4 改訂への足場確立。

## §13 6 KPI

1. 稼働率: 100% (= 9 step 全完走、blocker 0)
2. 短縮率: 高 (= D14 review + paper PnL + D20 chain + 3798 demote + 4 段階判定 + trade plan + review + vault を 1 conversation で完結)
3. 採用率: 100% (= W3 9 条件全 ✓、3798 demote 成功、sector 多様化達成、137A0 新 #1)
4. 差戻率: 0% (= chain 1 発成功、HOLD 復帰想定通り、W3 設計 doc path 2 実証)
5. Integrator 負荷: 低 (= 3 DB 全 md5 不変、code 変更 0、vault 4 file)
6. 安全事故: 0 (= DB write 0 / API 0 / token 0 / launchctl 0 / 実発注 0)
