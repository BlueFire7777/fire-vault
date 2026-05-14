---
id: FIRE-CODEX-R1-WAVE61-PRE-results
phase: 本番 v0 中核 / Wave 61-pre / price-return-paper_pnl 設計
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
implementation_wave: Wave 61-impl
design_doc: ~/fire-vault/03_design/FIRE_price_return_paper_pnl_linkage_2026-05-14.md
---

# Wave 61-pre Results — Price / Return / Paper PnL Linkage Design

## §1 結論

price / return / paper PnL linkage 設計完成 ✓。Wave 61-impl で 新規 runner
`scripts/jobs/run_after_r1_paper_pnl_preview.py` を実装する path 確立。

**設計核心**:
- D10 追跡対象 = **9 銘柄** (= AFTER-R1 top 3 + F111 boost_with_caution 6)
- return horizon = h0 / h1 / h5 / h20 (= daily close 基準)
- price source = `market_prices_daily` (= staging read-only、URI mode=ro)
- paper PnL ledger = read-only JSON (= reports/after_r1/paper_pnl_ledger_*.json)
- manual review 接続 = review.md → actual entry/exit/pnl 読込、未記入なら review_missing warning
- pattern outcome = 9 種 pattern + matched_tickers + decision (promote/suppress/watch)
- **D10 単発で promote しない**、candidate_only/unvalidated 維持

## §2 D10 追跡対象 (= 9 銘柄)

### 2.1 重点 3 (= AFTER-R1 top_candidates)

| rank | ticker | name | score (F062) |
|---|---|---|---|
| 1 | 340A0 | ジグザグ | 175.22 |
| 2 | 3798 | ＵＬＳグループ | 172.65 |
| 3 | 137A0 | Cocolive | 170.85 (= refresh 復活) |

### 2.2 拡張 6 (= F111 boost_with_caution、entry candidate 4-12)

| rank | ticker | name | sector | close (5/14) | risk_yen |
|---|---|---|---|---|---|
| 4 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 |
| 5 | 9130 | 共栄タンカー | 運輸 | 1,410 | 7,050 |
| 6 | 331A0 | メディックス | 情報通信 | 482 | 2,410 |
| 7 | 4389 | プロパティデータバンク | 情報通信 | 863 | 4,315 |
| 8 | 9247 | TRE ホールディングス | 情報通信 | 1,612 | 8,060 |
| 9 | 4317 | レイ | 情報通信 | 508 | 2,540 |

## §3 return horizon

| horizon | 計算 | 用途 |
|---|---|---|
| h0 | base_date close (= 5/14 refreshed) | entry baseline |
| h1 | 翌営業日 close | 即日 trend |
| h5 | 5 営業日後 close | 短期 trend |
| h20 | 20 営業日後 close | 中期 trend (= F286-PNL-R3 既定) |

intraday tick が必要になれば次 wave (Wave 62?) で別途検討。

## §4 paper PnL ledger JSON schema (= 主要 fields)

```
schema_version / generated_at / base_date / evaluation_date /
source_trade_plan / source_review_md / source_after_r1_artifacts /
entries[]:
  ticker / name / rank / action_label / score / reason_tags /
  pattern_tags / estimated_100_share_risk / risk_within_pilot_limit /
  artifact_source / f111_input_source /
  planned_entry / planned_stop_loss / planned_take_profit /
  actual_entry_optional / actual_exit_optional /
  close_price_base_date / next_close / h1_return / h5_return / h20_return /
  paper_entry_price / paper_exit_price / paper_pnl_yen / paper_pnl_pct /
  real_pnl_optional / entry_decision / outcome_status /
  freshness_status / warnings
summary:
  candidate_count / evaluated_count / missing_price_count /
  review_missing_count / paper_win_count / paper_loss_count /
  paper_flat_count / average_h20_return / average_paper_pnl_yen
safety_flags (= 10 keys all false)
```

詳細は `03_design/FIRE_price_return_paper_pnl_linkage_2026-05-14.md`。

## §5 manual review 接続

| review.md field | ledger field |
|---|---|
| §2 entry 価格 actual | actual_entry_optional |
| §2 exit 価格 actual | actual_exit_optional |
| §3 総 PnL | real_pnl_optional |
| §8 final decision (enter/watch/skip) | entry_decision |

review.md `status: blank` または該当 field 全て空 → warnings に "review_missing"
を追加。runner は自動推測しない (= Fujiwara 本人の記入を待つ)。

## §6 pattern outcome 設計 (= 9 種)

| pattern | D10 matched | 初期 decision | evidence_level |
|---|---|---|---|
| refreshed_price_ok | 9 | **watch** | low |
| manual_review_active | 9 | **watch** | low |
| research_boost | 9 | **watch** | low |
| risk_within_pilot_limit | 9 | **watch** | low |
| multi_reason | 9 (= 全 candidate に複数 reason_tags) | **watch** | low |
| low_risk | 5 (340A0/3798/137A0/331A0/4317) | **watch** | low |
| sector_concentration | 3 (340A0/3798/137A0 = 情報通信) | **watch + caveat** | low |
| liquidity_ok | (review 待ち) | **pending** | n/a |
| freshness_ok_high_conf | 0 (= DATA-R3 MISSING) | **n/a** | n/a |

→ 全 **watch + candidate_only 維持** (= D10 単発、sample 不足の正直記述)

## §7 promote/suppress/watch 初期基準

### promote (= 全 ✓ 必要)
- 同 ticker / 同 pattern が 3 D-day 以上で positive outcome
- max drawdown_proxy 軽微 (= h5 内 -3% 以内)
- manual review で「納得感あり」「ルール遵守」
- 寄り天 / liquidity 問題なし
- forbidden_check.passed 維持

### suppress (= いずれか ✓)
- 同 ticker / 同 pattern が 2 D-day 以上で negative outcome (= h20 -5% 以下)
- liquidity issue 連続 (= 板薄 / spread 大)
- risk_notes に重大 caveat
- 寄り天傾向 複数回

### watch (= D10 単発、default)
- positive / negative outcome 混在
- まだ判断不能

## §8 optional helper 設計

### 推奨: 案 A (新規 runner)

```
scripts/jobs/run_after_r1_paper_pnl_preview.py
```

- args: --base-date / --morning-line-material / --f111-real-batch / --review-md /
        --db-path / --output-json / --horizon-days / --dry-run
- guards: production/develop DB refuse / output path guard / DB write 0
- 既存 `pnl.paper_pnl` の read helpers 流用 (= fetch_market_close_after_business_days /
  fetch_market_open_next_business_day)
- DB UPDATE path (= F286-PNL-R3) は import しない

### 不推奨: 案 B (既存 runner 拡張)

既存 `run_f286_after_r1_night_batch.py` に `--task paper-pnl-preview` 追加。
ただし既存 task との separation 不明確 → 案 A 推奨。

## §9 tests 設計 (= 10+ tests、Wave 61-impl)

| test class | 観点 |
|---|---|
| TestPaperPnlLedgerSchema | schema valid (= keys + types) |
| TestMissingPriceWarning | row なし → outcome_status="missing_price" + warning |
| TestReviewMissingWarning | review.md status=blank → warnings=["review_missing"] |
| TestSkippedDecisionStillEvaluated | entry_decision=skip でも paper_pnl 計算 |
| TestPatternOutcomeAggregation | pattern_tags 集計 + decision |
| TestNoDbWriteAfterLedgerBuild | runner 終了後 staging md5 不変 |
| TestProductionDevelopRefused | fire.db / fire.develop.db → refuse |
| TestStagingReadOnlyMode | URI mode=ro 確認 |
| TestSafetyFlagsAllFalse | ledger safety_flags 全 false |
| TestOutputPathGuard | /data/ / /.git/ → refuse |

## §10 Codex 4 lane factual-confirm

| lane | 観点 | reply |
|---|---|---|
| **A** | D10 候補 / artifact / review source (= morning_line_material 2026-05-27 内 top_candidates) | **YES** ✓ — "top_candidates contains 340A0/3798/137A0 with score and entry_guideline present" |
| **B** | price-return horizon (= pnl/paper_pnl.py read-only helpers、h1/h5/h20 business day offset) | **YES** ✓ — "read-only helpers for next business-day open and N-business-day close from market_prices_daily, supporting h1/h5/h20-style offsets with no DB writes or API calls" |
| **C** | schema reuse (= read helper のみ流用、UPDATE path 不要) | **YES** ✓ — "paper_pnl.py は read-only 計算、R3 runner の UPDATE は staging 限定、Wave 61-pre は AFTER-R1 の read helper だけ再利用する新規 read-only JSON ledger 設計で進められます" |
| **D** | promote 基準 (= D10 単発で promote しない、candidate_only / unvalidated 維持) | **YES** ✓ — "Wave 61-pre is design-only/advisory; with a single D10 sample outcomes remain candidate_only/unvalidated, and suppress requires repeated negative outcome plus liquidity issue plus heavy risk_notes" |

→ 全 lane 設計通り ✓。実装方針確定、Wave 61-impl 開始可能。

## §11 next implementation wave (= Wave 61-impl)

| 項目 | 内容 |
|---|---|
| 推定工数 | 1 wave (= 設計 → 実装 → tests → vault) |
| 新規 file | `scripts/jobs/run_after_r1_paper_pnl_preview.py` |
| 新規 tests | `tests/scripts/jobs/test_run_after_r1_paper_pnl_preview.py` (= 10+ tests) |
| 流用 module | `pnl.paper_pnl` (= read helpers only) |
| 設計 reference | `03_design/FIRE_price_return_paper_pnl_linkage_2026-05-14.md` |
| 安全境界 | DB write 0 / API 0 / token 0 / staging read-only / production-develop refuse |
| Codex 4 lane (= 実装 wave) | A=schema / B=guards / C=helper integration / D=safety_flags |

## §12 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ (= 本 wave で read-only ですらない) |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| DB write | 0 ✓ |
| production / develop 接続 | 0 ✓ |
| staging 接続 | 1 回 (= read-only baseline 確認のみ、URI mode=ro) |
| LINE / token / API / launchctl / plist / cron | 全 **0** ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 **0** ✓ |
| git tracked file 変更 | **0** (= vault docs 3 file 追加のみ) |

## §13 vault docs (= 本 wave 追加)

- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE61_PRE_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE61_PRE_results.md`
- `~/fire-vault/03_design/FIRE_price_return_paper_pnl_linkage_2026-05-14.md`

## §14 next action 候補 (= 優先順)

1. **Wave 61-impl** (= 設計 doc を Implementation reference として使用、
   新規 runner + 10+ tests + vault)
2. **D10 review.md 記入** (= 寄付き actual + 15:30 以降の outcome)
3. **W60-pilot-D11** (= 2026-05-28 木、recently_seen 拡張)
4. **liquidity filter 強化 wave** (= 板厚 / spread を staging 内 read-only)
5. **DATA-R3 freshness gate JSON 連携 wave** (= freshness_verdict MISSING → OK)
6. **全銘柄 daily refresh launchd 自動化** (= 別 HQ 承認)
7. **W60-pilot-W2 集約** (= D6-D10 5 営業日 review、KPI 算出)
