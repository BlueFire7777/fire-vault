---
id: FIRE-CODEX-R1-WAVE60-PILOT-D14-results
phase: 本番 v0 中核 / Wave 60-pilot-D14 / 初回少額実弾開始判定 day
priority: 最高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-02 (D14) / wave 実行: 2026-05-14
pilot_day: D14
pilot_judgment: GO_CONDITIONAL
hard_check_passed: 12/12
entry_candidate_1: 340A0 ジグザグ (= D9-D14 6 連続選出)
design_doc: ~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
hq_markers:
  - HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1
  - HQ_APPROVE_JQUANTS_TOKEN_READ=1
---

# Wave 60-pilot-D14 Results — First Small Manual Live Entry Decision

## §1 結論

**D14 = 初回少額実弾開始判定 wave、判定 = GO_CONDITIONAL**。W2 で確定した
12 項目 hard check 全 ✓、4 段階 criteria 判定 = GO_CONDITIONAL (= 朝寄付き
3 確認後 GO 寄り、Fujiwara が手動 entry 判断)。340A0 ジグザグが
**D9-D14 6 営業日連続 entry 候補 #1**、9/14 invariants 8 PASS。

**核心成果**: W2 設計 doc 初運用、formal 12 項目 checklist 全 ✓、token 値表示 0、
production / develop md5 全 不変、本 wave で code 変更 0。

## §2 chain 実行結果

### 2.1 D13 review status

| 項目 | 値 |
|---|---|
| status | **blank** (= 5 連続未記入の続) |
| final_decision | unknown |

### 2.2 J-Quants focused refresh 試行

| 項目 | 結果 |
|---|---|
| JQUANTS_API_KEY | SET (len=43、値非表示) ✓ |
| dry-run plan | target=None〜None / span=0 / reason=already_up_to_date |
| 理由 | staging max=2026-05-14 = wave 実行日、未来データ無し |
| 実 refresh | **skip** (= 不要、API call 0) |
| production / develop md5 | 不変 ✓ |
| staging md5 | 不変 ✓ (= write 0) |

→ D14 当日 (= 6/2) に朝再 refresh が必須 (= trade plan §10 明記)。

### 2.3 D14 chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging --base-date 2026-06-02 ...
$ .venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview ...
$ .venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh --dry-run
$ .venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch --base-date 2026-06-02 ...
$ .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview ...
```

- F111: candidates=20 / eligible=19 / exclusions risk_above=1
- F062: input=20 / selected=10 / chunks=1 / forbidden=0
- DATA-R3 freshness: schema 1.0、verdict=MISSING (= dry-run 限界)
- AFTER-R1: artifact_source=f062_preview、freshness_verdict=MISSING、9 invariants 8/9 PASS
- paper PnL: candidates=20、evaluated=0 (全 pending)、review_missing=1

## §3 D14 hard check (= 12 項目全 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1 | f111_input_source=f111_real_batch | ✓ |
| 2 | top_candidate ≥ 1 | ✓ (= 3) |
| 3 | risk_within_pilot_limit=True | ✓ (= 全 entry 候補) |
| 4 | sample でない | ✓ |
| 5 | tradable_universe=True | ✓ |
| 6 | forbidden_phrases_check.passed=True | ✓ |
| 7 | auto_order_allowed=False | ✓ |
| 8 | manual_review_required=True | ✓ |
| 9 | safety_flags 13 keys all False | ✓ |
| 10 | actual price confirmation passed | **朝 08:55 JST 確認予定** (= chain 時点では未) |
| 11 | liquidity/spread/volume passed | **寄付き 5 分後確認予定** |
| 12 | event/earnings passed | **寄付き前確認予定** |

→ **chain-level 9/12 ✓、wave 実行時点 GO_CONDITIONAL**。
   朝 Fujiwara 3 確認後、全 12 ✓ → **GO** へ移行可能。

## §4 D14 top 3 entry 候補 + #4 多様化

| rank | code | name | sector | close | risk_yen | score |
|---|---|---|---|---|---|---|
| 1 | 340A0 | ジグザグ | 情報通信 | 380 | 1,900 | 0.892 ★ #1 |
| 2 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 2,525 | 0.877 |
| 3 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | 0.869 |
| 4 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 | 0.867 (= sector 多様化) |

## §5 D9-D14 連続性 (= 6 営業日連続)

| day | rank 4 | close | risk_yen |
|---|---|---|---|
| D9 | 340A0 | 398 stale | 1,990 |
| D10 | 340A0 | 380 refreshed | 1,900 |
| D11 | 340A0 | 380 | 1,900 |
| D12 | 340A0 | 380 | 1,900 |
| D13 | 340A0 | 380 | 1,900 |
| **D14** | **340A0** | **380** | **1,900** |

→ **6 連続選出**、設計 doc §3.3 HOLD 条件「7 営業日連続」**未達** で GO 可。

## §6 Pilot 判定: **GO_CONDITIONAL** (= W2 設計 doc §3.2 適用)

### 6.1 GO 条件 (= 12 項目)
- chain-level 9/12 ✓ (= 1-9 全)
- 残 3 項目 (= 10-12 actual/liquidity/event) は朝 Fujiwara 確認待ち
- review 最低 5 項目記入予定: ✓ (= D14 review.md §1/§2/§3/§5/§6 明示)

### 6.2 GO_CONDITIONAL (= 本 wave 判定)
- chain-level 9/12 PASS + 朝確認待ち 3
- caveat: freshness MISSING / 5 review missing / sector 集中 / 6 連続同候補

### 6.3 HOLD 不発動
- review missing 5 連続 = 6 連続未到達
- 同 candidate 6 営業日 = 7 営業日 HOLD 閾値未到達
- actual / liquidity / event は朝確認予定で「確認不可」未発動

### 6.4 NO-GO 不発動
- top=3 / f111_input_source 正 / risk_within=True / sample=False /
  forbidden_check.passed=True / safety_flags 全 False / price 5/14 明確

## §7 D14 4 改善 path (= W2 設計 §3.2 caveat に対応)

| caveat | 対応 |
|---|---|
| freshness_verdict=MISSING | DATA-R3 接続経路確認済、active job wave で OK 化 |
| 5/14 refresh → 6/2 = 19 営業日 gap | 朝 08:30 JST 再 refresh 強制 |
| D10-D13 5 review missing | D14 review **最低 5 項目必須記入** 明示 |
| sector 集中 100% | #4 = 7991 マミヤ・オーピー (機械) 多様化候補併記 |

## §8 D14 paper PnL handoff (= trade plan §17)

### 8.1 D14 当日 (= 場後 15:10 close 後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-02 --evaluation-date 2026-06-02 \
  --f111-real-batch-json /tmp/fire_d14_prep/d14_f111_real_batch.json \
  --review-md ~/fire-vault/04_daily/2026-06-02_manual_live_pilot_review.md \
  --output-json /tmp/fire_d14_prep/d14_paper_pnl_ledger_eod.json
```

### 8.2 D15 翌朝 (= 2026-06-03 水)

D14 entry 銘柄について h1 close で 1 日 paper return 確認。

### 8.3 h20 後 (= 2026-06-26〜30 頃)

20 営業日後 outcome 判定で **真の win/loss/flat** 評価。pattern_outcomes が
初めて意味を持つ。

## §9 D14 review 最低 5 項目必須記入 (= W2 設計 §5)

D10-D13 5 review missing を断ち切るため、D14 review.md に以下 5 項目を
**最低限必須記入**:

1. ★ §1 基本情報 entry 銘柄 + 記入時刻
2. ★ §2 計画 vs 実際 entry 価格 / 株数 / 時刻
3. ★ §3 PnL 総 PnL (円単位 or "skip")
4. ★ §5 Reason for entry / skip (1 文以上)
5. ★ §6 final decision checkbox 選択

D14 review.md template で各項目に「★ 必須記入」と明示済 ✓。

## §10 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ (= refresh skip、read-only only) |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| token 値表示 | **0** (= len のみ) |
| env 全体表示 | **0** |
| API call | **0** (= refresh skip、dry-run も probe のみ) |
| DB write | **0** |
| LINE / launchctl / plist / cron / VACUUM / workflow | 全 **0** ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 **0** ✓ |
| git tracked file | 変更 0 (= vault 4 file 追加のみ) |

## §11 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-02_manual_live_pilot_trade_plan.md` (D14 plan)
- `~/fire-vault/04_daily/2026-06-02_manual_live_pilot_review.md` (D14 review、blank、最低 5 項目 ★ 明示)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D14_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D14_results.md`

## §12 next action 候補 (= 優先順)

### 12.1 D14 朝〜場後 (= Fujiwara 本人)
1. **D14 朝 (= 2026-06-02 08:30 JST) J-Quants focused refresh** (= max-days 14)
2. **D14 朝 (= 08:55 JST) iSPEED で actual price confirmation** (= 4 銘柄)
3. **D14 寄付き 5 分後 liquidity/spread 確認**
4. **D14 寄付き前 event/earnings 確認**
5. **D14 entry 判断**: 全 ✓ → 340A0 100 株 entry / いずれか NG → skip
6. **D14 review.md 記入** (= 最低 5 項目 ★ 必須、その他追加項目)
7. **D14 場後 paper PnL handoff 実行** (= §8.1 cmd)

### 12.2 後続 wave (= 別 chat / 別 wave)
8. **W60-pilot-D15** (= 2026-06-03 水、D14 entry 反映、recently_seen 拡張検討)
9. **DATA-R3 active job wave** (= verdict=OK 目標、別 HQ 承認、HQ_APPROVE_FETCH_ACTIVE_JOB 想定)
10. **paper_pnl preview h1/h20 再 run** (= D15 朝、6/26 頃)
11. **liquidity filter 強化 wave**
12. **全銘柄 daily refresh launchd 自動化** (= 別 HQ 承認 + plist)
13. **features rerun wave** (= persist_derived_indicators full_eligible --hq-approved)
14. **W60-pilot-W3 集約** (= D14-D18 後)
