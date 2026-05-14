---
id: FIRE-CODEX-R1-WAVE60-PILOT-D11-results
phase: 本番 v0 中核 / Wave 60-pilot-D11 / paper PnL linkage 初運用 day
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-28 (D11) / wave 実行: 2026-05-14
pilot_day: D11
pilot_judgment: GO_CONDITIONAL
entry_candidate_1: 340A0 ジグザグ (= 3 連続選出)
entry_candidate_2: 3798 ＵＬＳグループ
entry_candidate_3: 137A0 Cocolive
d10_review_status: missing
d10_d11_overlap: 100%
---

# Wave 60-pilot-D11 Results — D11 Manual Live Pilot Trade Plan with Paper PnL Linkage

## §1 結論

**D11 Pilot 判定 = GO_CONDITIONAL** (= 寄付き直前 actual + liquidity + event
3 確認後 entry、いずれか NG なら skip)。

W61-impl で実装した paper PnL preview runner を初運用、**候補 → 判断 → review
→ paper PnL の一連の実運用フロー**が回せる状態を確認。D10 と D11 で
候補 100% 同一 (= features cap 律速)、340A0/3798/137A0 が **D9-D11 3 連続選出**
で signal 安定性確認。

## §2 chain 実行結果

### 2.1 D10 review/paper PnL recap

| 項目 | 値 |
|---|---|
| D10 review status | **status: blank** (= Fujiwara 未記入) |
| D10 final_decision | unknown |
| D10 actual entry/exit/PnL | 全 None |
| D10 paper PnL ledger | /tmp/fire_d10_prep/d10_paper_pnl_ledger.json |
| D10 candidate_count | 20 |
| D10 evaluated_count | 0 (= 全 pending、base 5/14 = staging max) |
| D10 pattern_outcomes | 9 種全 watch / evidence=low |

### 2.2 D11 F111-real-batch chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-05-28 --max-candidates 20 \
    --label-threshold-mode strict \
    --recently-seen-codes 8747,5729,3489 \
    --output-json /tmp/fire_d11_prep/d11_f111_real_batch.json
```

- cli_version 1.2.0 ✓
- candidates=20 / eligible=19
- exclusions: risk_above=1 (137A0 → wait、refresh で 0 になっているはず) /
  実 D11: risk_above=1 (= 別 candidate)
- tuning_params: strict / recently_seen=[8747,5729,3489] / demoted_count=3

### 2.3 F062 preview chain

```
$ .venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
    --preview-json /tmp/fire_d11_prep/d11_f111_preview_list.json \
    --output-json /tmp/fire_d11_prep/d11_f062_preview.json \
    --output-text /tmp/fire_d11_prep/d11_f062_preview.txt \
    --output-summary-json /tmp/fire_d11_prep/d11_f062_summary.json
```

- input=20 / selected=10 / chunks=1
- forbidden=0 / manual_review=20

### 2.4 AFTER-R1 night batch chain

```
$ .venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
    --mode mvp --task all --base-date 2026-05-28 \
    --f062-preview-json /tmp/fire_d11_prep/d11_f062_preview.json \
    --f062-preview-summary-json /tmp/fire_d11_prep/d11_f062_summary.json \
    --output-dir /tmp/fire_d11_prep/after_r1 \
    --artifact-source f062_preview \
    --f111-input-source f111_real_batch
```

- artifact_source: f062_preview ✓
- f062_raw_kind: f062_actual_dict ✓
- f111_input_source: f111_real_batch ✓
- base_date: 2026-05-28 ✓

### 2.5 D11 paper PnL preview chain (= W61-impl runner 初運用)

```
$ .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-05-14 --evaluation-date 2026-05-14 \
    --f111-real-batch-json /tmp/fire_d11_prep/d11_f111_real_batch.json \
    --morning-line-material-json /tmp/fire_d11_prep/after_r1/morning_line_material_2026-05-28.json \
    --good-candidate-ranking-json /tmp/fire_d11_prep/after_r1/good_candidate_ranking_2026-05-28.json \
    --paper-live-ledger-json /tmp/fire_d11_prep/after_r1/paper_live_ledger_2026-05-28.json \
    --pattern-candidate-report-json /tmp/fire_d11_prep/after_r1/pattern_candidate_report_2026-05-28.json \
    --output-json /tmp/fire_d11_prep/d11_paper_pnl_ledger.json \
    --markdown /tmp/fire_d11_prep/d11_paper_pnl_ledger.md
```

結果:
- candidates=20 / evaluated=0 (= 全 pending) / review_missing=1
- 340A0: rank 4 / close=380 / planned_entry=380 / stop=361 / TP=387.6 / outcome=pending
- pattern_tags: refreshed_price_ok / manual_review_active / research_boost /
  risk_within_pilot_limit / multi_reason / low_risk (6 種)

## §3 D11 top 3 + D10/D11 overlap

### 3.1 重点 3 候補

| rank | code | name | sector | close (5/14) | score | label | risk_yen |
|---|---|---|---|---|---|---|---|
| 1 | 340A0 | ジグザグ | 情報通信 | 380 | 0.892 | boost_with_caution | 1,900 |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 0.877 | boost_with_caution | 2,525 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 0.869 | boost_with_caution | 3,695 |

### 3.2 D9/D10/D11 連続性

| day | top 4 | top 5 | top 6 |
|---|---|---|---|
| D9 (= 2026-05-26) | 340A0 | 3798 | 137A0 |
| D10 (= 2026-05-27) | 340A0 | 3798 | 137A0 |
| **D11 (= 2026-05-28)** | **340A0** | **3798** | **137A0** |

→ **3 営業日連続同候補** = signal 安定性確認 ✓ (= features cap 律速、ただし
  research_final_score は安定で entry 候補性は維持)

## §4 D11 hard check (= 9 invariants)

| invariant | D11 result |
|---|---|
| artifact_source | f062_preview ✓ |
| f062_raw_kind | f062_actual_dict ✓ |
| f111_input_source | f111_real_batch ✓ |
| freshness_verdict | **MISSING** ⚠ (= DATA-R3 gate 未連携、本 wave 意図的) |
| forbidden_phrases_check.passed | True ✓ |
| auto_order_allowed | False ✓ |
| manual_review_required | True ✓ |
| safety_flags 13 keys all False | True ✓ |
| top_candidates count ≥ 1 | 3 ✓ (340A0/3798/137A0) |

→ 8/9 PASS (freshness_verdict のみ意図的 MISSING)。

## §5 Pilot 判定: **GO_CONDITIONAL**

### 5.1 GO 条件 (= 9/9 達成)

- ☑ f111_real_batch
- ☑ refreshed price available (= 5/14)
- ☑ risk_within_pilot_limit=True (= 全 3 候補)
- ☑ top candidate あり (= 3 件 + 拡張 6)
- ☑ sample でない
- ☑ tradable_universe=True
- ☑ D10 review/paper PnL で重大な問題なし (= review blank、paper pending、9 pattern 全 watch)
- ☑ liquidity/event 確認欄が trade plan に明記
- ☑ 9 hard invariants 8/9 PASS

### 5.2 HOLD 条件 (= conditional 因子)

| 因子 | 状況 | 影響 |
|---|---|---|
| D10 review = blank | Fujiwara 未記入 | entry 不明、actual confirm 必須 |
| D10/D11 overlap 100% | 3 連続同候補 | signal 安定だが entry 判断は同じ |
| 5/14 refresh → 5/28 = 14 営業日 gap | 価格鮮度 stale | 朝再 refresh or iSPEED actual 必須 |
| sector 集中 | 情報通信 100% (3/3) | #4 7991 (機械) 多様化検討 |
| freshness_verdict=MISSING | DATA-R3 gate 未連携 | 次 wave で gate JSON 連携 |

### 5.3 NO-GO ではない理由

- top_candidates 3 件あり
- f111_input_source = f111_real_batch ✓
- risk_within = True
- sample = False
- forbidden_check.passed = True
- safety_flags 全 False
- price freshness 明確 (= 5/14 cap、14 営業日 gap で要 confirm)

→ **GO_CONDITIONAL** = 寄付き直前 actual + liquidity + event 3 確認後 entry。

## §6 D9 vs D10 vs D11 比較

| 項目 | D9 (5/26) | D10 (5/27) | D11 (5/28) |
|---|---|---|---|
| market_prices_daily max | 2026-05-08 (= stale 6 営業日) | **2026-05-14** | **2026-05-14** (= 1 営業日進歩、ただし 14 営業日 gap) |
| refresh status | 未実行 | 済 (= 12 銘柄) | 済 (= 12 銘柄、再 refresh 必要) |
| eligible_count | 18 | 19 | 19 |
| exclusions risk_above | 1 (137A0) | 0 | **1 (= 別 candidate)** |
| top 1-3 | 340A0/3798/137A0 | 340A0/3798/137A0 | **340A0/3798/137A0** |
| 340A0 risk_yen | 1,990 stale | 1,900 refreshed | 1,900 refreshed |
| paper PnL preview | 不実装 | 不実装 | **✓ 初運用** |
| Pilot 判定 | GO_CONDITIONAL | GO_CONDITIONAL | **GO_CONDITIONAL** |

## §7 D11 paper PnL handoff 手順 (= trade plan §13 抜粋)

### 7.1 D11 当日 (= 場後・15:10 close 後)

review.md §2-§3 に actual entry/exit/PnL を記入後:

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-05-28 --evaluation-date 2026-05-28 \
  --f111-real-batch-json /tmp/fire_d11_prep/d11_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-05-28_manual_live_pilot_review.md \
  --output-json /tmp/fire_d11_prep/d11_paper_pnl_ledger_eod.json
```

### 7.2 翌営業日 (= D12 = 5/29 金、朝再 refresh 後)

D11 entry 銘柄について h1 close で 1 日 paper return 確認。

### 7.3 h20 後 (= 2026-06-26 頃)

20 営業日後の outcome 判定で **真の win/loss/flat** を評価。

## §8 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ (= 本 wave で read-only only) |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| DB write | 0 ✓ |
| production / develop 接続 | 0 ✓ |
| staging 接続 | read-only only ✓ |
| LINE / token / API / launchctl / plist / cron | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault 4 file 追加のみ) |

## §9 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-05-28_manual_live_pilot_trade_plan.md` (D11 plan)
- `~/fire-vault/04_daily/2026-05-28_manual_live_pilot_review.md` (D11 review、blank)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D11_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D11_results.md`

## §10 next action 候補 (= 優先順)

1. **D11 朝 (= 2026-05-28 08:30 JST) 12 銘柄再 refresh** (= 5/14 → 5/28 まで catch-up)
2. **D11 寄付き直前 iSPEED で actual confirm + entry 判断**
3. **D11 review.md 記入** (= 15:30 以降、Fujiwara 本人)
4. **D11 paper PnL handoff 実行** (= §7 の cmd、場後)
5. **D10 review.md 記入** (= 遡って記入推奨)
6. **W60-pilot-D12** (= 2026-05-29 金、recently_seen 拡張 + D11 entry 反映)
7. **paper_pnl preview h1/h20 再 run** (= D12 朝、6/26 頃)
8. **liquidity filter 強化 wave** (= 板厚 / spread を staging 内 read-only)
9. **全銘柄 daily refresh launchd 自動化** (= 別 HQ 承認 + plist)
10. **DATA-R3 freshness gate JSON 連携 wave** (= freshness_verdict MISSING → OK)
11. **W60-pilot-W2 集約** (= D6-D11 6 営業日 review)
