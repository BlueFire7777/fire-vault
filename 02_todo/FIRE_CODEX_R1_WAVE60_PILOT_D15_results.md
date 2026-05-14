---
id: FIRE-CODEX-R1-WAVE60-PILOT-D15-results
phase: 本番 v0 中核 / Wave 60-pilot-D15 / HOLD 発動 + review 記入 path 開放
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-03 (D15) / wave 実行: 2026-05-14
pilot_day: D15
pilot_judgment: HOLD
hold_trigger:
  - "review_missing_6_consecutive (= 設計 doc §3.3 #1)"
  - "same_candidate_7_consecutive (= 340A0、設計 doc §3.3 #2)"
hard_check_chain_level_passed: 9/12
d9_d14_review_missing_count: 6
d9_d15_overlap: 100% (= 340A0 7 連続)
design_doc: ~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
---

# Wave 60-pilot-D15 Results — Review-Linked Manual Live Pilot Trade Plan

## §1 結論

**D15 Pilot 判定 = HOLD** (= 設計 doc §3.3 HOLD 条件 #1 + #2 同時発動)。

W2 設計 doc 通り「review missing 5 連続超過」+ 「同 candidate 7 営業日連続」の
2 条件が同時 ✓ で HOLD 発動。chain-level 9/12 は維持、ただし **Fujiwara が D14
review 記入後 HOLD 解除 path 開放**。340A0 が D9-D15 で **7 営業日連続選出**、
3 DB md5 全 不変、本 wave で code 変更 0。

## §2 chain 実行結果

### 2.1 D14 review 確認結果 (= 必須 5 項目全 BLANK)

| 項目 | 値 |
|---|---|
| status | blank (= 6 連続未記入の続) |
| §1 記入時刻 | BLANK |
| §1 ticker | BLANK |
| §2 entry 価格 | BLANK |
| §3 総 PnL | BLANK |
| §5 Reason 1 文 | BLANK |
| §6 final decision | BLANK |

→ 必須 5 項目 全 BLANK 確認、D9-D14 で 6 連続 blank。

### 2.2 D9-D14 review status

| day | date | status |
|---|---|---|
| D9 | 2026-05-26 | blank |
| D10 | 2026-05-27 | blank |
| D11 | 2026-05-28 | blank |
| D12 | 2026-05-29 | blank |
| D13 | 2026-06-01 | blank |
| D14 | 2026-06-02 | blank |

→ **6 連続 review missing** 確定 → 設計 doc §3.3 HOLD 条件 #1 (= 5 連続超過) 発動。

### 2.3 D14 paper PnL --review-md 付き再 run

```bash
$ .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-05-14 --evaluation-date 2026-05-14 \
    --f111-real-batch-json /tmp/fire_d14_prep/d14_f111_real_batch.json \
    --morning-line-material-json /tmp/fire_d14_prep/after_r1/morning_line_material_2026-06-02.json \
    --review-md ~/fire-vault/04_daily/2026-06-02_manual_live_pilot_review.md \
    --output-json /tmp/fire_d14_prep/d14_paper_pnl_ledger_with_review.json
```

結果:
- candidates=20 / evaluated=0 / review_missing=1
- review_actuals: final_decision=unknown / is_blank=True / actual_entry/exit/PnL=None
- warnings: ["review_missing"]

→ review 取込済、blank → unknown / is_blank=True 正常確認。

### 2.4 D15 chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging --base-date 2026-06-03 ...
$ .venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview ...
$ .venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch --base-date 2026-06-03 ...
$ .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview ...
```

- F111: candidates=20 / eligible=19 / exclusions risk_above=1
- F062: input=20 / selected=10 / chunks=1 / forbidden=0
- AFTER-R1: artifact_source=f062_preview / freshness_verdict=MISSING
- paper PnL: candidates=20 / evaluated=0 / review_missing=1

## §3 D15 top 3 + D9-D15 7 連続

### 3.1 重点 3 候補 (= D11-D14 と完全同一)

| rank | code | name | sector | close | risk_yen |
|---|---|---|---|---|---|
| 1 | 340A0 | ジグザグ | 情報通信 | 380 | 1,900 |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 2,525 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 |

### 3.2 D9-D15 連続性 (= 7 営業日連続)

| day | rank 4 | close | risk_yen |
|---|---|---|---|
| D9 | 340A0 | 398 stale | 1,990 |
| D10 | 340A0 | 380 | 1,900 |
| D11 | 340A0 | 380 | 1,900 |
| D12 | 340A0 | 380 | 1,900 |
| D13 | 340A0 | 380 | 1,900 |
| D14 | 340A0 | 380 | 1,900 |
| **D15** | **340A0** | **380** | **1,900** |

→ **7 営業日連続** = 設計 doc §3.3 HOLD 条件 #2 (= 同 candidate 7 営業日連続) 発動。

## §4 D15 hard check (= chain-level 9/12)

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
| 10 | actual price confirmation | (朝確認待ち) |
| 11 | liquidity/spread/volume | (朝確認待ち) |
| 12 | event/earnings | (朝確認待ち) |

→ chain-level 9/12 ✓、ただし **HOLD 発動で D15 entry 非推奨**。

## §5 Pilot 判定: **HOLD** (= 設計 doc §3.3 #1 + #2 発動)

### 5.1 HOLD 発動条件

| # | 条件 | D15 状況 | 発動 |
|---|---|---|---|
| 1 | review missing 5 連続超過 | **D9-D14 で 6 連続 blank** | **✓ 発動** |
| 2 | 同 candidate 7 営業日以上連続 | **340A0 D9-D15 で 7 連続** | **✓ 発動** |
| 3 | actual price 確認できない | 朝確認次第 | 未確認 |
| 4 | liquidity 確認できない | 朝確認次第 | 未確認 |
| 5 | event risk 強い | 朝確認次第 | 未確認 |

→ **5 条件中 2 発動** で HOLD 確定。

### 5.2 HOLD 解除 path (= 主推奨)

| # | 条件 | 達成方法 |
|---|---|---|
| 1 | review missing < 6 連続 | **D10-D14 review のうち少なくとも 1 つを Fujiwara が記入** (= 必須 5 項目) |
| 2 | 同 candidate < 7 営業日連続 | D16 で recently_seen_codes に 340A0 追加 → 同候補 demote、自然解消 |

→ **D14 review 記入 + D16 で recently_seen 拡張** で HOLD 解除可能。

### 5.3 D15 推奨アクション

1. **D14 review.md (= 2026-06-02) を Fujiwara が記入** (= 必須 5 項目)
2. **D14 paper PnL --review-md 付き再 run** で review_actuals 反映確認
3. **D15 = skip + 場後 review.md 記入** (= D15 HOLD 対応も記録)
4. **D16 (= 2026-06-04 木) で recently_seen_codes 拡張** + 判定再起

## §6 D14 vs D15 比較

| 項目 | D14 | D15 |
|---|---|---|
| pilot_judgment | GO_CONDITIONAL | **HOLD** |
| chain-level hard check | 9/12 ✓ | 9/12 ✓ |
| top 3 candidates | 340A0/3798/137A0 | 同 |
| 340A0 連続 | 6 連続 | **7 連続** |
| review missing 連続 | (= D14 当日 blank、累計 5) | **D9-D14 6 連続** |
| 設計 doc §3.3 発動 | 不 | **#1 + #2** |
| 推奨 entry | 朝 3 確認 → 340A0 100 株 | **skip 推奨 + review 記入** |

## §7 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| DB write | 0 ✓ |
| production / develop 接続 | 0 ✓ |
| staging 接続 | read-only only ✓ |
| LINE / token / API / launchctl / plist / cron | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault 4 file 追加のみ) |

## §8 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-03_manual_live_pilot_trade_plan.md` (D15 plan、HOLD)
- `~/fire-vault/04_daily/2026-06-03_manual_live_pilot_review.md` (D15 review、blank)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D15_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D15_results.md`

## §9 next action 候補 (= 優先順)

### 9.1 D15 朝〜場後 (= Fujiwara 本人)

1. **D14 review.md 必須 5 項目記入** ★ HOLD 解除 path 主推奨
2. **D14 paper PnL --review-md 付き再 run** で reflection 確認
3. **D15 = skip + 場後 D15 review 記入** (= HOLD 対応記録)
4. **D15 paper PnL handoff** (= 場後 cmd 実行)

### 9.2 D16 以降 (= 別 chat / 別 wave)

5. **W60-pilot-D16** (= 2026-06-04 木、recently_seen 拡張 + 判定再起)
6. **W60-pilot-W3 集約** (= D14-D18 後)
7. **DATA-R3 active job wave** (= verdict=OK 目標)
8. **paper_pnl preview h1/h20 再 run** (= 6/26 頃)
9. **liquidity filter 強化 wave**
10. **features rerun wave** (= persist_derived_indicators full_eligible)
11. **全銘柄 daily refresh launchd 自動化**

## §10 設計 doc 検証 (= W2 設計 doc HOLD 条件が機能)

W2 設計 doc §3.3 HOLD 条件が **正規に発動**したのは本 wave (= D15) が初。
これは:
- 設計 doc の HOLD 条件が **実運用で機能**することを実証
- 「review missing 5 連続超過」「同 candidate 7 営業日連続」が **適切な threshold**
- Fujiwara に **強い signal** (= 「これ以上 entry せず review 記入に集中せよ」) を与える

→ 設計 doc 初運用 + HOLD 初発動の 2 重の検証成果。
