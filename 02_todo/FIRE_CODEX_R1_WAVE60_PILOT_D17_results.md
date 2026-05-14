---
id: FIRE-CODEX-R1-WAVE60-PILOT-D17-results
phase: 本番 v0 中核 / Wave 60-pilot-D17 / HOLD maintained 3 連続 + 340A0 demote 維持
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-05 (D17) / wave 実行: 2026-05-14
pilot_day: D17
pilot_judgment: HOLD_maintained
hold_status:
  "#1_review_missing_6_consecutive": "未解消 (= D14 review 0/6 記入)"
  "#2_same_candidate_7_consecutive": "解消継続 (= 340A0 demote 2 日連続)"
hard_check_chain_level_passed: 9/12
new_top_persistence: "3798 ＵＬＳ AFTER-R1 rank 1 = D16-D17 2 日連続"
---

# Wave 60-pilot-D17 Results — Review Gap Resolved / New Top Candidate Recheck

## §1 結論

**D17 Pilot 判定 = HOLD maintained (= 3 連続、設計 doc §3.3 #1 のみ残存)**。

D14 review が依然 0/6 記入で HOLD #1 解消できず。ただし 340A0 demote は維持
され HOLD #2 解消継続、新 top 3 (= 3798/137A0/331A0) が D16-D17 で 2 日連続
安定確認。3 DB md5 全 不変、本 wave で code 変更 0。

D18 (= 2026-06-08 月) で D14 review 記入後 HOLD 完全解除 → GO_CONDITIONAL
復帰可能性大。

## §2 chain 実行結果

### 2.1 D14 review 厳密 check (= 0/6 記入、D15/D16 と同)

| 項目 | 値 |
|---|---|
| yaml status | blank |
| §1 記入時刻 | BLANK |
| §1 ticker | BLANK |
| §2 entry 価格 actual | BLANK |
| §3 総 PnL | BLANK |
| §5 Reason 1 文 | BLANK |
| §6 final decision ☑ | BLANK |

→ **0/6 記入** = review_gap_UNRESOLVED → HOLD #1 残存継続。

### 2.2 D14 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown
- warnings: ["review_missing"]
- output: `/tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d17_check.json`

→ D14 review blank 継続、HOLD #1 解消不可。

### 2.3 D17 chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-06-05 --max-candidates 20 \
    --label-threshold-mode strict \
    --recently-seen-codes 8747,5729,3489,340A0 \
    ...
```

- F111: candidates=20 / eligible=19
- tuning_params: strict / recently_seen=[8747, 5729, 340A0, 3489] / demoted_count=4
- F062: input=20 / selected=10
- AFTER-R1: artifact_source=f062_preview / freshness_verdict=MISSING / 9 invariants 8/9 PASS
- paper PnL: candidates=20 / evaluated=0 / review_missing=1

## §3 D17 top 3 (= D16 と完全同一、2 日連続)

### 3.1 F111-real-batch top 4 (= 340A0 caution 維持)

| rank | code | label | demoted |
|---|---|---|---|
| 1-3 | 8747/5729/3489 | caution | True (= D6-D17 通算 12 日 demote) |
| **4** | **340A0** | **caution** | **True (= D16-D17 2 日連続 demote)** |
| 5+ | 3798/137A0/7991/9130/331A0/... | boost_with_caution | False |

### 3.2 AFTER-R1 top_candidates (= D16-D17 で 2 日連続安定)

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D16 | 3798 ＵＬＳ | 137A0 Cocolive | 331A0 メディックス |
| **D17** | **3798 ＵＬＳ** | **137A0 Cocolive** | **331A0 メディックス** |

→ **新 top 3 = 3798/137A0/331A0 が 2 日連続安定** = 340A0 demote 効果定着。

### 3.3 D17 entry 候補

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| 5 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 2,525 | boost_with_caution ★ #1 (= 2 連続) |
| 6 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution |
| 7 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 | boost_with_caution (= 多様化) |
| 8 | 9130 | 共栄タンカー | 運輸 | 1,410 | 7,050 | boost_with_caution |
| 9 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |

## §4 D17 hard check (= chain-level 9/12 ✓)

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

## §5 Pilot 判定: **HOLD maintained** (= D15/D16/D17 3 連続)

### 5.1 HOLD 条件 status

| # | 条件 | D15 | D16 | D17 |
|---|---|---|---|---|
| 1 | review missing 5 連続超過 | ✓ 発動 | ✓ 継続 | **✓ 継続** |
| 2 | 同 candidate 7 連続 | ✓ 発動 | ✗ 解消 | **✗ 解消継続 (= 2 日連続)** |
| 3-5 | actual/liquidity/event | 未 | 未 | 未 |

→ 5 条件中 1 残存、HOLD 維持 = 3 連続。

### 5.2 D18 HOLD 完全解除 path (= 主推奨)

| step | 内容 | 担当 |
|---|---|---|
| 1 | D14 review.md 必須 5 項目記入 | Fujiwara 本人 |
| 2 | D14 paper PnL --review-md 再 run | 本線 (D18 wave) |
| 3 | D18 chain 再起 → **HOLD #1 解消** → GO_CONDITIONAL 復帰 | 本線 (D18 wave) |

### 5.3 D17 推奨アクション

- **主推奨**: skip (= HOLD 維持) + D14 review 記入 + D18 で完全解除
- **alternative**: 3798 ＵＬＳ に少額 entry (= HOLD override、Fujiwara 責任、2 日連続 #1 で signal 安定)

## §6 D15/D16/D17 比較

| 項目 | D15 | D16 | D17 |
|---|---|---|---|
| pilot_judgment | HOLD | HOLD maintained | HOLD maintained |
| HOLD #1 (review) | ✓ 発動 | ✓ 継続 | ✓ 継続 (= D17 3 連続) |
| HOLD #2 (candidate) | ✓ 発動 | ✗ 解消 | ✗ 解消継続 (= 2 日連続) |
| recently_seen_codes | 3 | 4 (+ 340A0) | 4 (= 維持) |
| AFTER-R1 top 1 | 340A0 | **3798** (= 新) | **3798** (= 2 日連続) |
| 新 top 3 stability | n/a | 1 日 (= 初) | **2 日連続** |
| D14 review | blank | blank | blank |

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

- `~/fire-vault/04_daily/2026-06-05_manual_live_pilot_trade_plan.md` (D17 plan、HOLD maintained)
- `~/fire-vault/04_daily/2026-06-05_manual_live_pilot_review.md` (D17 review、blank)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D17_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D17_results.md`

## §9 next action 候補 (= 優先順)

### 9.1 Fujiwara 本人 (= HOLD #1 解除)

1. **D14 review.md 必須 5 項目記入** ★ HOLD 完全解除主推奨
2. **D17 = skip + 場後 review 記入** (= HOLD maintained 3 連続対応記録)

### 9.2 後続 wave

3. **W60-pilot-D18** (= 2026-06-08 月、D14 review 反映 → **HOLD 完全解除 + GO_CONDITIONAL 復帰**)
4. **W60-pilot-W3 集約** (= D14-D18 後 5 営業日 review)
5. **DATA-R3 active job wave** (= verdict=OK 目標)
6. **paper_pnl preview h1/h20 再 run** (= 6/26 頃)
7. **liquidity filter 強化 wave**
8. **features rerun wave**
9. **全銘柄 daily refresh launchd 自動化**

## §10 設計 doc 検証成果 (= W2 設計 doc 連続性)

- D15: HOLD 初発動 (= #1 + #2 同時)
- D16: HOLD #2 解消 (= 340A0 demote)、#1 維持 → HOLD maintained
- **D17: HOLD #2 解消継続 (= 2 日連続)、#1 維持 → HOLD maintained 3 連続**

→ 設計 doc §3.3 HOLD 条件の **段階的解除 path** が実運用で機能、D18 で
完全解除可能性大。
