---
id: FIRE-CODEX-R1-WAVE60-PILOT-D12-results
phase: 本番 v0 中核 / Wave 60-pilot-D12 / review-paper PnL feedback 反映 day
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-29 (D12) / wave 実行: 2026-05-14
pilot_day: D12
pilot_judgment: GO_CONDITIONAL
entry_candidate_1: 340A0 ジグザグ (= 4 連続選出)
d10_review_status: missing
d11_review_status: missing
d9_d12_overlap: 100%
---

# Wave 60-pilot-D12 Results — D12 Manual Live Pilot with Review and Paper PnL Feedback

## §1 結論

**D12 Pilot 判定 = GO_CONDITIONAL** (= D11 と同維持、ただし conditional 因子強化)。

D11 で paper PnL chain 初運用 → D12 で 2 連続運用達成。D10/D11 review = 両方 blank
(= 2 連続未記入)、340A0/3798/137A0 が **D9-D12 4 営業日連続選出** で signal 極めて安定、
ただし features cap = 5/14 律速で内容更新なし、15 営業日 gap 拡大。

**chain 完走 + 9 invariants 8/9 PASS + 3 DB md5 全 不変**。

## §2 chain 実行結果

### 2.1 D10/D11 review status

| 項目 | D10 (5/27) | D11 (5/28) |
|---|---|---|
| review status | **blank** | **blank** |
| final_decision | unknown | unknown |
| pilot_judgment | GO_CONDITIONAL | GO_CONDITIONAL |
| → caveat | 2 連続未記入 |

### 2.2 D10/D11 paper PnL ledger (= 既存 read-only 確認)

| 項目 | D10 | D11 |
|---|---|---|
| path | /tmp/fire_d10_prep/d10_paper_pnl_ledger.json | /tmp/fire_d11_prep/d11_paper_pnl_ledger.json |
| candidate_count | 20 | 20 |
| evaluated_count | 0 (= 全 pending) | 0 (= 全 pending) |
| review_missing_count | 1 | 1 |

### 2.3 D12 chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-05-29 --max-candidates 20 \
    --label-threshold-mode strict \
    --recently-seen-codes 8747,5729,3489 \
    --output-json /tmp/fire_d12_prep/d12_f111_real_batch.json

$ .venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview ...

$ .venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
    --mode mvp --task all --base-date 2026-05-29 ...

$ .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-05-14 --evaluation-date 2026-05-14 \
    --f111-real-batch-json /tmp/fire_d12_prep/d12_f111_real_batch.json ...
```

- F111: candidates=20 / eligible=19 / exclusions risk_above=1 / tuning_params strict + recent 3 + demoted 3
- F062: input=20 / selected=10 / chunks=1 / forbidden=0 / manual_review=20
- AFTER-R1: artifact_source=f062_preview / f111_input_source=f111_real_batch / base_date=2026-05-29 ✓
- paper PnL: candidates=20 / evaluated=0 / review_missing=1

## §3 D12 top 3 + D9-D12 overlap

### 3.1 重点 3 候補

| rank | code | name | sector | close (5/14) | score | label | risk_yen |
|---|---|---|---|---|---|---|---|
| 1 | 340A0 | ジグザグ | 情報通信 | 380 | 0.892 | boost_with_caution | 1,900 |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 0.877 | boost_with_caution | 2,525 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 0.869 | boost_with_caution | 3,695 |

### 3.2 D9-D12 連続性 (= 4 営業日連続)

| day | rank 4 | close | risk_yen | label |
|---|---|---|---|---|
| D9 (5/26) | 340A0 ジグザグ | 398 stale | 1,990 | boost_with_caution |
| D10 (5/27) | 340A0 ジグザグ | 380 refreshed | 1,900 | boost_with_caution |
| D11 (5/28) | 340A0 ジグザグ | 380 | 1,900 | boost_with_caution |
| **D12 (5/29)** | **340A0 ジグザグ** | **380** | **1,900** | **boost_with_caution** |

→ **4 営業日連続 340A0** = signal 極めて安定。features cap 律速で内容変化なし。

### 3.3 D11 vs D12 完全同一

D11 candidates と D12 candidates の top 12 は **100% 完全同一** (= 全 ticker
順位 / close / risk_yen 同じ)。staging max=5/14 で features rerun 無いため。

## §4 D12 hard check (= 9 invariants)

| invariant | D12 result |
|---|---|
| artifact_source | f062_preview ✓ |
| f062_raw_kind | f062_actual_dict ✓ |
| f111_input_source | f111_real_batch ✓ |
| freshness_verdict | **MISSING** ⚠ |
| forbidden_phrases_check.passed | True ✓ |
| auto_order_allowed | False ✓ |
| manual_review_required | True ✓ |
| safety_flags 13 keys all False | True ✓ |
| top_candidates count ≥ 1 | 3 ✓ |

→ 8/9 PASS。

## §5 Pilot 判定: **GO_CONDITIONAL maintained**

### 5.1 GO 条件 (= 9/9 達成)

- ☑ f111_real_batch / refreshed price / risk_within / top candidate /
  not sample / tradable / liquidity-event 欄明記 / 9 invariants 8/9 PASS /
  D10/D11 review/paper PnL に重大な問題なし (= blank + pending、active "問題" は無い)

### 5.2 HOLD 条件 (= conditional 因子強化)

| 因子 | 状況 | 影響 |
|---|---|---|
| D10/D11 review 2 連続 blank | Fujiwara 未記入 2 連続 | D12 で 3 回目記入機会、必須化 |
| D9-D12 4 連続同候補 | features cap 律速 | 真の更新には refresh + features rerun |
| 5/14 refresh → 5/29 D12 = **15 営業日 gap** | 拡大 | 朝再 refresh + iSPEED actual 必須 |
| sector 集中 100% | 情報通信 3/3 | #4 7991 (機械) 多様化検討 |
| freshness_verdict=MISSING | DATA-R3 gate 未連携 | 次 wave で gate JSON 連携 |

### 5.3 NO-GO ではない理由

- top_candidates 3 件あり
- f111_input_source = f111_real_batch ✓
- risk_within = True
- sample = False
- forbidden_check.passed = True
- safety_flags 全 False
- price freshness 明確 (= 5/14 cap、15 営業日 gap で要 confirm)

→ **GO_CONDITIONAL** = 寄付き直前 actual + liquidity + event 3 確認後 entry。

## §6 D9 vs D10 vs D11 vs D12 比較

| 項目 | D9 | D10 | D11 | D12 |
|---|---|---|---|---|
| market_prices_daily max | 2026-05-08 | 2026-05-14 | 2026-05-14 | 2026-05-14 |
| stale gap | 6 営業日 | 0 → 7 営業日 | 14 営業日 | **15 営業日** |
| eligible_count | 18 | 19 | 19 | 19 |
| exclusions risk_above | 1 (137A0) | 0 | 1 | 1 |
| top 1-3 | 340A0/3798/137A0 | 340A0/3798/137A0 | 同 | **同** |
| 340A0 close/risk | 398 stale / 1990 | 380 refreshed / 1900 | 380 / 1900 | **380 / 1900** |
| paper PnL preview | 不実装 | 不実装 | ✓ 初運用 | **✓ 2 連続運用** |
| review.md status | (= 設計中) | blank | blank | **(blank 想定)** |
| Pilot 判定 | GO_CONDITIONAL | GO_CONDITIONAL | GO_CONDITIONAL | **GO_CONDITIONAL** |

## §7 D12 paper PnL handoff 手順 (= trade plan §15)

### 7.1 D12 当日 (= 場後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-05-29 --evaluation-date 2026-05-29 \
  --f111-real-batch-json /tmp/fire_d12_prep/d12_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-05-29_manual_live_pilot_review.md \
  --output-json /tmp/fire_d12_prep/d12_paper_pnl_ledger_eod.json
```

### 7.2 翌営業日 (= 2026-06-01 月、朝)

D12 entry 銘柄について h1 close で 1 日 paper return 確認。

### 7.3 h20 後 (= 2026-06-26 頃)

20 営業日後 outcome 判定で **真の win/loss/flat** 評価。

## §8 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ (= read-only only) |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| DB write | 0 ✓ |
| production / develop 接続 | 0 ✓ |
| staging 接続 | read-only only ✓ |
| LINE / token / API / launchctl / plist / cron | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault 4 file 追加のみ) |

## §9 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-05-29_manual_live_pilot_trade_plan.md` (D12 plan)
- `~/fire-vault/04_daily/2026-05-29_manual_live_pilot_review.md` (D12 review、blank)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D12_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D12_results.md`

## §10 next action 候補 (= 優先順)

1. **D12 朝 (= 2026-05-29 08:30 JST) 12 銘柄再 refresh** (= 5/14 → 5/29 まで)
2. **D12 寄付き直前 iSPEED で actual confirm + entry 判断**
3. **D10/D11/D12 review.md 記入** (= 3 D-day 分、Fujiwara 本人、遡及記入推奨)
4. **D12 paper PnL handoff 実行** (= §7 cmd、場後)
5. **W60-pilot-D13** (= 2026-06-01 月、recently_seen 拡張 + D12 entry 反映)
6. **paper_pnl preview h1/h20 再 run** (= D13 朝、6/26 頃)
7. **liquidity filter 強化 wave**
8. **全銘柄 daily refresh launchd 自動化** (= 別 HQ 承認)
9. **DATA-R3 freshness gate JSON 連携 wave**
10. **W60-pilot-W2 集約** (= D6-D12 7 営業日 review、KPI 算出)
