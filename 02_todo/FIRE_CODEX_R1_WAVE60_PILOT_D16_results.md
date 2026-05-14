---
id: FIRE-CODEX-R1-WAVE60-PILOT-D16-results
phase: 本番 v0 中核 / Wave 60-pilot-D16 / 340A0 demote 成功 + HOLD maintained
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-04 (D16) / wave 実行: 2026-05-14
pilot_day: D16
pilot_judgment: HOLD_maintained
hold_status:
  "#1_review_missing_6_consecutive": "未解消 (= D14 review 0/6 記入)"
  "#2_same_candidate_7_consecutive": "解消済 (= 340A0 demote)"
hard_check_chain_level_passed: 9/12
demotion_effect: "340A0 → caution、new top = 3798/137A0/331A0"
recently_seen_codes_expanded: "8747,5729,3489 → 8747,5729,3489,340A0"
---

# Wave 60-pilot-D16 Results — Review Gap Closure + Recently Seen Expansion

## §1 結論

**D16 Pilot 判定 = HOLD maintained** (= 設計 doc §3.3 #1 のみ残存)。

**主要成果**:
- ✅ **340A0 demote 成功** (= recently_seen 拡張で caution へ、AFTER-R1 top = 3798/137A0/331A0 へ切替)
- ✅ **HOLD #2 (= 同 candidate 7 連続) 解消**
- ❌ **HOLD #1 (= review missing 6 連続) 未解消** (= D14 review 0/6 記入)
- ✅ chain-level 9/12、3 DB md5 全 不変、本 wave で code 変更 0

→ D17 で D14 review 記入 → HOLD 完全解除可能性高。

## §2 chain 実行結果

### 2.1 D14 review 厳密 check (= 0/6 記入)

| 項目 | 値 |
|---|---|
| yaml status | blank |
| §1 記入時刻 | BLANK |
| §1 ticker | BLANK |
| §2 entry 価格 actual | BLANK |
| §3 総 PnL | BLANK |
| §5 Reason 1 文 | BLANK |
| §6 final decision ☑ | BLANK |

→ **0/6 記入** = review_gap_unresolved → HOLD #1 残存。

### 2.2 D14 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: final_decision=unknown / is_blank=True
- warnings: ["review_missing"]
- output: `/tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review_d16_check.json`

### 2.3 recently_seen_codes 拡張

```
拡張前: 8747, 5729, 3489 (demoted_count=3)
拡張後: 8747, 5729, 3489, **340A0** (demoted_count=4)
```

### 2.4 D16 chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-06-04 --max-candidates 20 \
    --label-threshold-mode strict \
    --recently-seen-codes 8747,5729,3489,340A0 \
    --output-json /tmp/fire_d16_prep/d16_f111_real_batch.json
```

- F111: candidates=20 / eligible=19 / exclusions risk_above=1
- tuning_params: strict / recently_seen 4 / demoted 4
- F062: input=20 / selected=10
- AFTER-R1: artifact_source=f062_preview / freshness_verdict=MISSING / 9 invariants 8/9 PASS
- paper PnL: candidates=20 / evaluated=0 / review_missing=1

## §3 340A0 demote 効果実証 (= 主要成果)

### 3.1 F111-real-batch top 4 (= 340A0 caution へ)

| rank | code | label | demoted |
|---|---|---|---|
| 1 | 8747 | caution | True |
| 2 | 5729 | caution | True |
| 3 | 3489 | caution | True |
| **4** | **340A0** | **caution** | **True (= 新追加)** |
| 5 | 3798 | boost_with_caution | False |
| 6 | 137A0 | boost_with_caution | False |

### 3.2 AFTER-R1 top_candidates 切替

| day | rank 1 | rank 2 | rank 3 |
|---|---|---|---|
| D14/D15 | 340A0 ジグザグ | 3798 ＵＬＳ | 137A0 Cocolive |
| **D16** | **3798 ＵＬＳ (= 新 #1)** | **137A0 Cocolive** | **331A0 メディックス (= 新 entry)** |

→ 340A0 が top_candidates から除外、新 top 3 確立。

### 3.3 D16 entry 候補一覧 (= boost_with_caution、5 銘柄)

| rank | code | name | sector | close | risk_yen | label |
|---|---|---|---|---|---|---|
| 5 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 2,525 | boost_with_caution ★ 新 #1 |
| 6 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | boost_with_caution |
| 7 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 | boost_with_caution (= 多様化) |
| 8 | 9130 | 共栄タンカー | 運輸 | 1,410 | 7,050 | boost_with_caution |
| 9 | 331A0 | メディックス | 情報通信 | 482 | 2,410 | boost_with_caution |

## §4 D16 hard check (= chain-level 9/12)

| # | 項目 | 結果 |
|---|---|---|
| 1 | f111_input_source=f111_real_batch | ✓ |
| 2 | top_candidate ≥ 1 (= 3、新 top) | ✓ |
| 3 | risk_within_pilot_limit=True | ✓ |
| 4 | sample でない | ✓ |
| 5 | tradable_universe=True | ✓ |
| 6 | forbidden_check.passed=True | ✓ |
| 7 | auto_order_allowed=False | ✓ |
| 8 | manual_review_required=True | ✓ |
| 9 | safety_flags 13 keys all False | ✓ |
| 10 | actual price confirmation | 朝確認待ち |
| 11 | liquidity/spread/volume | 朝確認待ち |
| 12 | event/earnings | 朝確認待ち |

## §5 Pilot 判定: **HOLD maintained** (= 設計 doc §3.3 #1 のみ未解消)

### 5.1 HOLD 条件 status

| # | 条件 | D15 | D16 | 解消 |
|---|---|---|---|---|
| 1 | review missing 5 連続超過 | ✓ 発動 | ✓ 継続 (= 0/6 記入) | × |
| 2 | 同 candidate 7 連続 | ✓ 発動 | **✗ 解消** (= 340A0 demote) | ✓ |
| 3 | actual 確認不可 | 未 | 未 | - |
| 4 | liquidity 不可 | 未 | 未 | - |
| 5 | event リスク | 未 | 未 | - |

→ **5 条件中 1 発動** (= #1 のみ)、HOLD 維持。

### 5.2 HOLD 完全解除 path

| step | 内容 | 担当 |
|---|---|---|
| 1 | D14 review.md 必須 5 項目記入 | **Fujiwara 本人** |
| 2 | D14 paper PnL --review-md 付き再 run で reflection 確認 | 本線 (D17 wave) |
| 3 | D17 chain 再起 + 判定見直し → **HOLD 解除 → GO_CONDITIONAL** | 本線 (D17 wave) |

### 5.3 D16 推奨アクション

- **主推奨**: D14 review 記入 + D16 = skip + D17 で HOLD 解除
- **alternative**: D16 で 3798 ＵＬＳ に少額 entry (= HOLD override、Fujiwara 責任)

## §6 D15 vs D16 比較

| 項目 | D15 | D16 |
|---|---|---|
| pilot_judgment | HOLD | HOLD maintained |
| HOLD 条件 #1 (review) | ✓ 発動 | ✓ 継続 |
| HOLD 条件 #2 (candidate) | ✓ 発動 | **✗ 解消** |
| recently_seen_codes | 3 (= 8747/5729/3489) | **4 (= +340A0)** |
| AFTER-R1 top 1 | 340A0 | **3798 (= 新)** |
| 340A0 status | rank 4 boost_with_caution | **caution、demoted** |

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

- `~/fire-vault/04_daily/2026-06-04_manual_live_pilot_trade_plan.md` (D16 plan、HOLD maintained)
- `~/fire-vault/04_daily/2026-06-04_manual_live_pilot_review.md` (D16 review、blank)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D16_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D16_results.md`

## §9 next action 候補 (= 優先順)

### 9.1 Fujiwara 本人

1. **D14 review.md 必須 5 項目記入** ★ HOLD 解除主推奨
2. **D16 = skip + 場後 D16 review 記入** (= HOLD maintained 対応記録)

### 9.2 後続 wave

3. **W60-pilot-D17** (= 2026-06-05 金、D14 review 反映 → HOLD 解除)
4. **W60-pilot-W3 集約** (= D14-D18 後)
5. **DATA-R3 active job wave** (= verdict=OK 目標)
6. **paper_pnl preview h1/h20 再 run** (= 6/26 頃)
7. **liquidity filter 強化 wave**
8. **features rerun wave**
9. **全銘柄 daily refresh launchd 自動化**

## §10 設計 doc 検証成果 (= W2 設計 doc 解除 path 実証)

- 340A0 demote (= recently_seen 拡張) で HOLD #2 解消 ✓
- D17 で D14 review 記入後 HOLD #1 解消 path 確立
- 設計 doc §3.3 4 段階 criteria の **解除 path 実証** = W2 設計 doc の有効性確認
