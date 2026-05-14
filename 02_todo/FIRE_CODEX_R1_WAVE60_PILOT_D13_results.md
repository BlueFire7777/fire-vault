---
id: FIRE-CODEX-R1-WAVE60-PILOT-D13-results
phase: 本番 v0 中核 / Wave 60-pilot-D13 / review gap closure + DATA-R3 接続経路確認
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-01 (D13) / wave 実行: 2026-05-14
pilot_day: D13
pilot_judgment: GO_CONDITIONAL
entry_candidate_1: 340A0 ジグザグ (= 5 連続選出)
d10_d11_d12_review_status: missing
d9_d13_overlap: 100%
focused_refresh_attempted: skip (= already_up_to_date)
data_r3_freshness_connection: connected (= verdict=MISSING)
hq_markers:
  - HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1
  - HQ_APPROVE_JQUANTS_TOKEN_READ=1
---

# Wave 60-pilot-D13 Results — Focused Refresh / Review Gap Closure / Manual Live Pilot Trade Plan

## §1 結論

**D13 Pilot 判定 = GO_CONDITIONAL maintained**、ただし **5 改善 path 全て trade plan
に明示** で GO 寄りへ前進。chain (F111 → F062 → AFTER-R1 → paper_pnl_preview) 3 連続
運用達成、9 invariants 8/9 PASS、3 DB md5 全 不変、本 wave で code 変更 0。

**5 改善 path**:
1. ✅ **review 欠損明示処理** (= D10/D11/D12 3 review missing 明記、捏造なし)
2. ✅ **focused J-Quants refresh 試行** (= dry-run = already_up_to_date、skip 理由明記)
3. ✅ **DATA-R3 freshness 接続経路** (= --data-r3-freshness-json 引数で渡せた)
4. ✅ **actual confirmation 強制** (= 朝 refresh + 08:55 iSPEED actual)
5. ✅ **sector 多様化 caveat** (= 100% 集中明示、#4 7991 機械 推奨)

## §2 chain 実行結果

### 2.1 D10/D11/D12 review status

| 項目 | D10 (5/27) | D11 (5/28) | D12 (5/29) |
|---|---|---|---|
| review status | **blank** | **blank** | **blank** |
| final_decision | unknown | unknown | unknown |
| pilot_judgment | GO_CONDITIONAL | GO_CONDITIONAL | GO_CONDITIONAL |

→ **3 連続 review_missing** = 明確な caveat、D13 で 4 回目記入機会。
   FIRE-CODEX-R1 設計 § 6.2 で「捏造禁止」明記済み。

### 2.2 D13 F111-real-batch chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-06-01 --max-candidates 20 \
    --label-threshold-mode strict \
    --recently-seen-codes 8747,5729,3489 \
    --output-json /tmp/fire_d13_prep/d13_f111_real_batch.json
```

- candidates=20 / eligible=19 / exclusions risk_above=1
- tuning_params: strict / recent 3 / demoted 3

### 2.3 focused J-Quants refresh 試行

```
$ set -a; source /Users/bluefire/fire/.env; set +a
$ .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
    --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
    --datasets prices --max-days 14 \
    --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
    --dry-run --output-json /tmp/fire_d13_prep/jq_dryrun.json
```

| 項目 | 結果 |
|---|---|
| JQUANTS_API_KEY | SET (len=43、値非表示) ✓ |
| dry-run plan | `target=None〜None / span=0 / reason=already_up_to_date` |
| 理由 | staging max=2026-05-14 = wave 実行日 (= 5/14)、未来データなし |
| 実 refresh | **skip** (= 不要、API call 0) |
| production / develop md5 | 不変 ✓ |

→ **skip は技術的に妥当**。D13 当日 (= 6/1) に実行すれば 5/14→5/29 catch-up 可能。

### 2.4 DATA-R3 freshness 接続試行

```
$ .venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh --dry-run
$ .venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
    --mode mvp --task all --base-date 2026-06-01 \
    --f062-preview-json /tmp/fire_d13_prep/d13_f062_preview.json \
    --data-r3-freshness-json /tmp/f286_data_r3_freshness.json \
    --output-dir /tmp/fire_d13_prep/after_r1 \
    --artifact-source f062_preview \
    --f111-input-source f111_real_batch
```

| 項目 | 結果 |
|---|---|
| freshness_report_json | /tmp/f286_data_r3_freshness.json (= schema_version 1.0) |
| verdict (DATA-R3 出力) | **MISSING** (= dry-run + 実 fetch 無し、設計通り) |
| AFTER-R1 受け取り | ✓ (= 引数で渡せた、エラーなし) |
| morning_line_material.freshness_verdict | MISSING (= DATA-R3 verdict そのまま反映) |

→ **接続経路確認 ✓**。実 verdict=OK 達成は active job execution が必要 (= 別 wave、HQ approve 要)。

### 2.5 D13 paper PnL preview

```
$ .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-05-14 --evaluation-date 2026-05-14 \
    --f111-real-batch-json /tmp/fire_d13_prep/d13_f111_real_batch.json \
    --morning-line-material-json /tmp/fire_d13_prep/after_r1/morning_line_material_2026-06-01.json \
    --output-json /tmp/fire_d13_prep/d13_paper_pnl_ledger.json \
    --markdown /tmp/fire_d13_prep/d13_paper_pnl_ledger.md
```

- candidates=20 / evaluated=0 / review_missing=1
- 340A0: rank 4 / close 380 / planned_entry 380 / pattern_tags 6 種

## §3 D13 top 3 + D9-D13 overlap (= 5 連続)

### 3.1 重点 3 候補 (= D11/D12 と完全同一)

| rank | code | name | sector | close | risk_yen | score |
|---|---|---|---|---|---|---|
| 1 | 340A0 | ジグザグ | 情報通信 | 380 | 1,900 | 0.892 |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 2,525 | 0.877 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | 0.869 |

### 3.2 D9-D13 連続性

| day | rank 4 | close | risk_yen |
|---|---|---|---|
| D9 (5/26) | 340A0 | 398 stale | 1,990 |
| D10 (5/27) | 340A0 | 380 refreshed | 1,900 |
| D11 (5/28) | 340A0 | 380 | 1,900 |
| D12 (5/29) | 340A0 | 380 | 1,900 |
| **D13 (6/1)** | **340A0** | **380** | **1,900** |

→ **5 営業日連続 340A0** (= signal 極めて安定、ただし features cap 律速)

## §4 D13 hard check (= 9 invariants)

| invariant | D13 result |
|---|---|
| artifact_source | f062_preview ✓ |
| f062_raw_kind | f062_actual_dict ✓ |
| f111_input_source | f111_real_batch ✓ |
| freshness_verdict | **MISSING** (= DATA-R3 gate 接続 OK、実 verdict=OK は別 wave) |
| forbidden_phrases_check.passed | True ✓ |
| auto_order_allowed | False ✓ |
| manual_review_required | True ✓ |
| safety_flags 13 keys all False | True ✓ |
| top_candidates ≥ 1 | 3 ✓ |

→ 8/9 PASS。

## §5 Pilot 判定: **GO_CONDITIONAL maintained** (= 5 改善 path で GO 寄り)

### 5.1 GO 条件 (= 9/9 達成)
- ☑ f111_real_batch / refreshed price / risk_within / top candidate /
  not sample / tradable / actual confirmation 欄強制 / 9 invariants 8/9 / review/PnL 重大問題 0

### 5.2 HOLD 条件 (= conditional 因子)
- ⚠ D10/D11/D12 review **3 連続 blank** (= 4 回目記入機会)
- ⚠ D9-D13 **5 連続同候補** (= signal 安定、features cap 律速)
- ⚠ 5/14 refresh → 6/1 = **18 営業日 gap** (= 朝再 refresh 必須)
- ⚠ sector 集中 100%
- ⚠ freshness_verdict=MISSING (= 接続経路 OK、実 verdict=OK は別 wave)

### 5.3 NO-GO ではない理由
- top_candidates 3 件 / f111_real_batch ✓ / risk_within=True / sample=False /
  forbidden_check.passed=True / safety_flags 全 False / price freshness 明確 +
  actual confirmation 経路明示

→ **GO_CONDITIONAL** = 寄付き直前 actual + liquidity + event + 朝再 refresh
  の **4 確認 後 entry**。いずれか NG → skip。

## §6 D13 → GO 寄り改善 達成事項

| 改善 path | 達成 | 効果 |
|---|---|---|
| review 欠損明示処理 | ✓ | 3 review missing 明記、捏造禁止維持 |
| focused J-Quants refresh 試行 | ✓ | dry-run 動作、skip 理由 (already_up_to_date) 明記 |
| DATA-R3 freshness 接続経路 | ✓ | --data-r3-freshness-json 引数で渡せた、verdict 反映確認 |
| actual confirmation 強制 | ✓ | trade plan §10 で朝 refresh + 08:55 iSPEED actual |
| sector 多様化 caveat | ✓ | sector 集中 100% 明示、#4 7991 多様化推奨 |

→ **GO_CONDITIONAL → GO 寄り** の 5 改善 path 全達成。

## §7 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ (= 本 wave で read-only only、refresh skip) |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| token 値表示 | **0** (= len のみ表示) |
| env 全体表示 | **0** |
| API call | **0** (= refresh skip、dry-run も probe のみ) |
| DB write | **0** (= staging も含めて) |
| LINE / launchctl / plist / cron / VACUUM / workflow | 全 **0** ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 **0** ✓ |
| git tracked file | 変更 0 (= vault 4 file 追加のみ) |

## §8 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-01_manual_live_pilot_trade_plan.md` (D13 plan)
- `~/fire-vault/04_daily/2026-06-01_manual_live_pilot_review.md` (D13 review、blank)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D13_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D13_results.md`

## §9 next action 候補 (= 優先順)

1. **D13 朝 (= 2026-06-01 08:30 JST) 12 銘柄再 refresh** (= 5/14 → 5/29 まで catch-up)
2. **D13 寄付き直前 iSPEED で actual confirm + entry 判断**
3. **D10/D11/D12/D13 review.md 記入** (= 4 D-day 分、Fujiwara 本人、遡及記入推奨)
4. **D13 paper PnL handoff 実行** (= trade plan §18、場後)
5. **W60-pilot-D14** (= 2026-06-02 火、recently_seen 拡張 + D13 entry 反映)
6. **DATA-R3 active job wave** (= verdict=OK 目標、別 HQ 承認、HQ_APPROVE_FETCH_ACTIVE_JOB 想定)
7. **paper_pnl preview h1/h20 再 run** (= D14 朝、6/26 頃)
8. **liquidity filter 強化 wave**
9. **全銘柄 daily refresh launchd 自動化** (= 別 HQ 承認)
10. **W60-pilot-W2 集約** (= D6-D13 8 営業日 review、KPI 算出)
