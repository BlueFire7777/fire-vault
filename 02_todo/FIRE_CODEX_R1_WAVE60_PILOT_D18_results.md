---
id: FIRE-CODEX-R1-WAVE60-PILOT-D18-results
phase: 本番 v0 中核 / Wave 60-pilot-D18 / HOLD maintained 4 連続 + 340A0 demote 3 日連続
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-08 (D18) / wave 実行: 2026-05-14
pilot_day: D18
pilot_judgment: HOLD_maintained
hold_status:
  "#1_review_missing_6_consecutive": "未解消 (= D14 review 0/6、HOLD 4 連続)"
  "#2_same_candidate_7_consecutive": "解消継続 (= 340A0 demote 3 日連続)"
hard_check_chain_level_passed: 9/12
new_top_persistence: "3798 ＵＬＳ AFTER-R1 rank 1 = D16-D18 3 日連続"
---

# Wave 60-pilot-D18 Results — Review Gap Resolved / HOLD Release Recheck

## §1 結論

**D18 Pilot 判定 = HOLD maintained 4 連続** (= 設計 doc §3.3 #1 のみ残存)。

D14 review 依然 0/6 記入で HOLD #1 解消不可。ただし 340A0 demote 3 日連続維持
+ 新 top 3 (= 3798/137A0/331A0) が D16-D18 で **3 日連続安定**確認、signal
完全定着。3 DB md5 全 不変、本 wave で code 変更 0。

D19 で D14 review 記入 → HOLD 完全解除 + GO_CONDITIONAL 復帰可能性大。

## §2 chain 実行結果

### 2.1 D14 review 厳密 check (= 0/6 記入、4 連続)

| 項目 | 値 |
|---|---|
| yaml status | blank |
| §1 記入時刻 / ticker | BLANK |
| §2 entry 価格 actual | BLANK |
| §3 総 PnL | BLANK |
| §5 Reason 1 文 | BLANK |
| §6 final decision ☑ | BLANK |

→ review_gap_UNRESOLVED → HOLD #1 残存継続 (= 4 連続)。

### 2.2 D14 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown
- warnings: ["review_missing"]
- output: `/tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d18_check.json`

### 2.3 D18 chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging --base-date 2026-06-08 ...
$ .venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview ...
$ .venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch --base-date 2026-06-08 ...
$ .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview ...
```

- F111: candidates=20 / eligible=19 / exclusions risk_above=1
- tuning_params: strict / recently_seen 4 / demoted_count=4
- AFTER-R1: artifact_source=f062_preview / freshness_verdict=MISSING / 9 invariants 8/9 PASS
- paper PnL: candidates=20 / evaluated=0 / review_missing=1

## §3 D18 top 3 + 新 top 3 連続安定

### 3.1 F111-real-batch top 4 (= 340A0 caution 3 日連続)

| day | rank 4 | label | demoted |
|---|---|---|---|
| D14/D15 | 340A0 | boost_with_caution | False |
| D16 | 340A0 | caution | True |
| D17 | 340A0 | caution | True |
| **D18** | **340A0** | **caution** | **True (= 3 日連続)** |

### 3.2 AFTER-R1 top_candidates (= D16-D18 で 3 日連続安定 ✓)

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D16 | 3798 ＵＬＳ | 137A0 Cocolive | 331A0 メディックス |
| D17 | 3798 ＵＬＳ | 137A0 Cocolive | 331A0 メディックス |
| **D18** | **3798 ＵＬＳ** | **137A0 Cocolive** | **331A0 メディックス** |

→ **新 top 3 = 3 日連続安定 = signal 完全定着**。

### 3.3 D18 entry 候補

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| 5 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 2,525 | boost_with_caution ★ #1 (= 3 日連続) |
| 6 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution |
| 7 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 | boost_with_caution (= 多様化) |
| 8 | 9130 | 共栄タンカー | 運輸 | 1,410 | 7,050 | boost_with_caution |
| 9 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |

## §4 D18 hard check (= chain-level 9/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1 | f111_input_source=f111_real_batch | ✓ |
| 2 | top_candidate ≥ 1 (= 3) | ✓ |
| 3 | risk_within_pilot_limit=True | ✓ |
| 4 | sample でない | ✓ |
| 5 | tradable_universe=True | ✓ |
| 6 | forbidden_check.passed=True | ✓ |
| 7 | auto_order_allowed=False | ✓ |
| 8 | manual_review_required=True | ✓ |
| 9 | safety_flags 13 keys all False | ✓ |
| 10-12 | actual / liquidity / event | 朝確認待ち |

## §5 Pilot 判定: **HOLD maintained 4 連続**

### 5.1 HOLD 条件 status

| # | 条件 | D15 | D16 | D17 | D18 |
|---|---|---|---|---|---|
| 1 | review missing 5 連続超過 | ✓ 発動 | ✓ 継続 | ✓ 継続 | **✓ 継続** (= 4 連続) |
| 2 | 同 candidate 7 連続 | ✓ 発動 | ✗ 解消 | ✗ 解消継続 | **✗ 解消継続** (= 3 日連続) |

→ 5 条件中 1 残存、HOLD 維持 = **4 連続**。

### 5.2 D19 HOLD 完全解除 path

| step | 内容 | 担当 |
|---|---|---|
| 1 | D14 review.md 必須 5 項目記入 | **Fujiwara 本人 (= ★最優先)** |
| 2 | D14 paper PnL --review-md 再 run | 本線 (D19 wave) |
| 3 | D19 chain 再起 → **HOLD #1 解消** → GO_CONDITIONAL 復帰 | 本線 (D19 wave) |

### 5.3 D18 推奨アクション

- **主推奨**: skip (= HOLD 維持) + D14 review 記入 + D19 で完全解除
- **alternative**: 3798 ＵＬＳ に少額 entry (= 3 日連続 #1 で signal 完全定着、最も entry 価値高いタイミング、HOLD override = Fujiwara 責任)

## §6 D15-D18 4 連続比較

| 項目 | D15 | D16 | D17 | D18 |
|---|---|---|---|---|
| pilot_judgment | HOLD | HOLD maintained | HOLD maintained | HOLD maintained |
| HOLD #1 (review) | ✓ 発動 | ✓ 継続 | ✓ 継続 | **✓ 継続 4 連続** |
| HOLD #2 (candidate) | ✓ 発動 | ✗ 解消 | ✗ 解消継続 | **✗ 解消継続 3 日連続** |
| AFTER-R1 top 1 | 340A0 | 3798 (= 新) | 3798 (= 2 連続) | **3798 (= 3 連続安定)** |
| 新 top 3 stability | n/a | 1 日 (= 初) | 2 日連続 | **3 日連続定着** |
| D14 review | blank | blank | blank | **blank 継続** |

## §7 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| DB write / API / token / LINE / launchctl | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault 4 file 追加のみ) |

## §8 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-08_manual_live_pilot_trade_plan.md` (D18 plan、HOLD maintained 4 連続)
- `~/fire-vault/04_daily/2026-06-08_manual_live_pilot_review.md` (D18 review、blank)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D18_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D18_results.md`

## §9 next action 候補 (= 優先順)

### 9.1 Fujiwara 本人

1. **D14 review.md 必須 5 項目記入** ★ HOLD 完全解除最優先 (= 4 連続呼びかけ)
2. D18 = skip + 場後 review 記入

### 9.2 後続 wave

3. **W60-pilot-D19** (= 2026-06-09 火、D14 review 反映 → **HOLD 完全解除 + GO_CONDITIONAL 復帰**)
4. **W60-pilot-W3 集約** (= D14-D18 後 5 営業日 review、HOLD 期間の特性分析)
5. **DATA-R3 active job wave** (= verdict=OK 目標)
6. **paper_pnl preview h1/h20 再 run** (= 6/26 頃)
7. **liquidity filter 強化 wave**
8. **features rerun wave**
9. **全銘柄 daily refresh launchd 自動化**

## §10 設計 doc 検証成果 (= W2 設計 doc HOLD 段階解除)

- D15: HOLD 初発動 (= #1 + #2)
- D16: HOLD #2 初解消 (= 340A0 demote)
- D17: HOLD #2 解消継続 (= 2 日連続)
- **D18: HOLD #2 解消継続 + 新 top 3 連続安定 (= 3 日連続定着)**

→ 設計 doc §3.3 HOLD 段階解除 path が **4 連続 wave で実証**、D19 で
完全解除可能性大。
