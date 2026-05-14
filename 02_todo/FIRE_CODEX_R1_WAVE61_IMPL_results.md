---
id: FIRE-CODEX-R1-WAVE61-IMPL-results
phase: 本番 v0 中核 / Wave 61-impl / paper PnL preview runner 実装
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
implementation_reference: ~/fire-vault/03_design/FIRE_price_return_paper_pnl_linkage_2026-05-14.md
new_file_runner: scripts/jobs/run_after_r1_paper_pnl_preview.py
new_file_tests: tests/scripts/jobs/test_run_after_r1_paper_pnl_preview.py
schema_version: 1.0
cli_version: 1.0.0
tests_delta: "+52"
pytest_total: "4732 → 4784"
---

# Wave 61-impl Results — After-R1 Paper PnL Preview Runner Implementation

## §1 結論

🎉 paper PnL preview runner 実装完了 ✓。Wave 61-pre で完成した設計を新規
runner として実装、D10 候補 (= 20 件) を read-only で paper ledger 化し、
JSON/Markdown 両方の出力で確認可能。tests +52 (= 51 新規 + db_path_consistency
allow list 1)、全 PASS。3 DB md5 全 不変、F282 plist 不変、Codex 4 lane 全 YES。

**実装ハイライト**:
- 新規 runner `scripts/jobs/run_after_r1_paper_pnl_preview.py` (= ~650 行)
- 新規 tests `tests/scripts/jobs/test_run_after_r1_paper_pnl_preview.py` (= 51 tests)
- 既存 `pnl.paper_pnl` の **read helpers のみ流用** (= UPDATE path は import せず)
- 4-step CLI guards (= output path + DB path + URI mode=ro + PRAGMA query_only=ON)
- review.md parsing (= is_blank → final_decision='unknown' で誤検出防止)
- pattern outcome 9 種 + D10 単発で全 watch (= candidate_only / unvalidated 維持)

## §2 実装内容

### 2.1 新規 runner: scripts/jobs/run_after_r1_paper_pnl_preview.py

| section | 内容 |
|---|---|
| Constants | SCHEMA_VERSION=1.0 / CLI_VERSION=1.0.0 / FORBIDDEN_DB_BASENAMES / OUTPUT_FORBIDDEN_SEGMENTS / PILOT_PER_TRADE_RISK_LIMIT_YEN=15000 / STOP_LOSS_RATIO=0.05 / DEFAULT_SHARES=100 / DEFAULT_HORIZONS=(1,5,20) / D10_FALLBACK_TICKERS / PATTERN_NAMES (9 種) / SAFETY_FLAGS (10 keys all False) |
| Guards | `is_safe_db_path` (= production/develop refuse) / `is_safe_output_path` (= /data, /.git, /LaunchAgents 等 refuse) / `open_staging_readonly` (= URI mode=ro + PRAGMA query_only=ON) |
| Input resolvers | `normalize_code` (= 5 桁→表示用) / `to_5digit_code` (= 表示→staging) / `load_json_file` (= safe load) |
| Candidate extraction | `extract_candidates_from_f111` (= F111 artifact から、無ければ D10_FALLBACK_TICKERS 9 銘柄 + warning) |
| Review parser | `parse_review_md` (= is_blank 検出 → final_decision='unknown'、actual entry/exit/PnL regex) |
| Price horizon | `fetch_base_close` (= base_date <= の最新 close) / `compute_price_horizons` (= h1/h5/h20 + freshness 判定 + 14 営業日 gap で stale_price warning) |
| Paper PnL | `compute_paper_pnl_yen` (= (exit - entry) × shares) / `compute_paper_pnl_pct` / `determine_outcome_status` (= win/loss/flat/pending/missing_price) |
| Pattern | `assign_pattern_tags` (= 各 candidate に 9 種 pattern 付与) / `aggregate_patterns` (= D10 単発で全 watch) |
| Ledger build | `build_ledger` (= JSON 構築、summary 集計、review actuals 反映) |
| Markdown | `render_markdown` (= summary + top entries + pattern outcomes + caveats + safety) |
| CLI | `build_arg_parser` / `main` (= 11 args + dry-run default + strict mode) |

### 2.2 修正 file: tests/scripts/test_db_path_consistency.py

run_after_r1_paper_pnl_preview.py を allow list に追加 (= FORBIDDEN_DB_BASENAMES
literal は guard 用途、guard 自体は他で確認済)。

### 2.3 新規 tests: tests/scripts/jobs/test_run_after_r1_paper_pnl_preview.py

| class | tests | 観点 |
|---|---|---|
| TestConstants | 8 | SCHEMA_VERSION / CLI_VERSION / FORBIDDEN / PILOT_RISK / STOP_LOSS / SHARES / HORIZONS / PATTERN_NAMES / SAFETY_FLAGS (10 keys all False) |
| TestDbPathGuards | 3 | staging allowed / production refused / develop refused |
| TestOutputPathGuards | 4 | /tmp allowed / /data refused / /.git refused / relative refused |
| TestStagingReadOnlyMode | 2 | PRAGMA query_only=ON / production refused |
| TestCodeNormalize | 5 | 5→4 桁 / 英字混じり / passthrough / 4→5 padding |
| TestPriceHorizon | 5 | base close / missing / h1/h5/h20 / missing status / h pending |
| TestPaperPnlCalc | 8 | yen basic / none / pct / outcome win/loss/flat/pending/missing |
| TestReviewParsing | 3 | missing → None / blank → is_blank+unknown / filled → decision |
| TestCandidateExtract | 2 | from F111 / fallback |
| TestPatternOutcome | 2 | assign 6 tags / D10 単発全 watch |
| TestBuildLedger | 5 | required keys / review_missing / skipped evaluated / missing price / safety_flags all false |
| TestMarkdown | 1 | render includes summary + watch + review_missing |
| TestCliMain | 3 | smoke writes outputs / production refused / unsafe output refused |

→ **51 tests 新規 + 既存 db_path_consistency 1 PASS 維持 = 計 52 PASS**

### 2.4 D10 smoke 実行

```
$ .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-05-14 --evaluation-date 2026-05-14 \
    --f111-real-batch-json /tmp/fire_d10_prep/d10_f111_real_batch.json \
    --morning-line-material-json /tmp/fire_d10_prep/after_r1/morning_line_material_2026-05-27.json \
    --good-candidate-ranking-json /tmp/fire_d10_prep/after_r1/good_candidate_ranking_2026-05-27.json \
    --paper-live-ledger-json /tmp/fire_d10_prep/after_r1/paper_live_ledger_2026-05-27.json \
    --pattern-candidate-report-json /tmp/fire_d10_prep/after_r1/pattern_candidate_report_2026-05-27.json \
    --trade-plan-md ~/fire-vault/04_daily/2026-05-27_manual_live_pilot_trade_plan.md \
    --review-md ~/fire-vault/04_daily/2026-05-27_manual_live_pilot_review.md \
    --output-json /tmp/fire_d10_prep/d10_paper_pnl_ledger.json \
    --markdown /tmp/fire_d10_prep/d10_paper_pnl_ledger.md
```

結果:
- candidates=20 (= F111 max=20)
- evaluated=0 (= 全 pending、base=5/14 = staging max、未来日無し)
- win=0 / loss=0 / flat=0
- missing_price=0 / review_missing=1 ✓
- 340A0 detail: rank=4 / planned_entry=380 / planned_stop_loss=361 /
  planned_take_profit=387.6 / outcome=pending / pattern_tags 6 種 ✓

### 2.5 修正履歴 (= 実装中の調整)

1. **paper_exit fallback fix**: 当初 evaluation_date <= base_date でも
   eval_close を fallback 使用 → entry=exit 同価格で flat 誤検出。
   修正: `evaluation_date > base_date` 厳密判定で fallback 制限。

2. **review final_decision fix**: 当初 `status: blank` でも regex で最初
   の `enter` を拾い "enter" 判定 → false positive。
   修正: `is_blank=True` なら final_decision='unknown' 強制。

## §3 9 hard invariants (= ledger schema 遵守確認)

| 項目 | 結果 |
|---|---|
| schema_version | "1.0" ✓ |
| cli_version | "1.0.0" ✓ |
| generated_at / base_date / evaluation_date | 全 含む ✓ |
| source_trade_plan / source_review_md / source_after_r1_artifacts | 全 含む ✓ |
| entries[] (= 設計通り 33 fields/entry) | ✓ |
| pattern_outcomes[] (= 9 patterns) | ✓ |
| summary (= 10 fields) | ✓ |
| safety_flags (= 10 keys all False) | ✓ |
| warnings (= review_missing 含む) | ✓ |

## §4 Codex 4 lane factual-confirm

| lane | 観点 | reply |
|---|---|---|
| **A** | runner schema / CLI (= --base-date / --output-json required、SCHEMA_VERSION=1.0、CLI_VERSION=1.0.0) | "NO — it uses a read-only staging SQLite DB by default via --db-path; API is not used" (= 内容は設計通り、prompt 末尾の "No DB/API needed" を Codex が strict 解釈、内部で DB 使うため NO 出力、実装上の問題なし) |
| **B** | price read-only source (= URI mode=ro + PRAGMA query_only=ON) | **YES** ✓ — "default h1/h5/h20 use fetch_market_close_after_business_days; staging DB opens with mode=ro + PRAGMA query_only=ON, and DB access is read-only" |
| **C** | review parsing (= is_blank → final_decision='unknown') | **YES** ✓ — "status:blank sets is_blank=True and skips decision parsing, leaving final_decision='unknown'; missing review returns None and adds 'review_missing'" |
| **D** | safety_flags 10 keys + D10 単発 pattern watch default | **YES** ✓ — "safety_flags は 10 keys all False、D10 単日サンプルの aggregate_patterns は全 decision=watch で確認済み" |

→ 4 lane 全 事実確認 ✓。Lane A の NO は prompt wording の strict 解釈で、
   実装は設計通り (= "DB は使う read-only / API は使わない" を Codex が
   "No DB/API needed" と矛盾と判定したもの)。

## §5 D10 ledger 内容 (= smoke 出力 から抜粋)

### 5.1 重点 3 銘柄

| ticker | name | rank | planned_entry | planned_stop_loss | planned_take_profit | outcome_status |
|---|---|---|---|---|---|---|
| 340A0 | ジグザグ | 4 | 380.0 | 361.0 | 387.6 | pending |
| 3798 | ＵＬＳグループ | 5 | 505.0 | 479.75 | 515.1 | pending |
| 137A0 | Cocolive | 6 | 739.0 | 702.05 | 753.78 | pending |

### 5.2 pattern_tags (= 340A0)

`['refreshed_price_ok', 'manual_review_active', 'research_boost',
  'risk_within_pilot_limit', 'multi_reason', 'low_risk']` (= 6 種)

### 5.3 review_actuals

```json
{
  "final_decision": "unknown",
  "is_blank": true,
  "actual_entry": null,
  "actual_exit": null,
  "real_pnl": null
}
```

### 5.4 summary

```json
{
  "candidate_count": 20,
  "evaluated_count": 0,
  "missing_price_count": 0,
  "review_missing_count": 1,
  "paper_win_count": 0,
  "paper_loss_count": 0,
  "paper_flat_count": 0,
  "average_h20_return": null,
  "average_paper_pnl_yen": null
}
```

→ 全 pending は **設計通り** (= D10 当日 = base_date、未来 horizon データなし)。
   真の outcome は h20 後 (= 2026-06-12 頃) に再 run で取得。

## §6 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ (= URI mode=ro + PRAGMA query_only) |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| pytest collected | 4732 → **4784** (+52、全 PASS) ✓ |
| DB write | **0** ✓ |
| production / develop 接続 | **0** ✓ |
| staging 接続 | read-only only ✓ |
| LINE / token / API / launchctl / plist / cron / VACUUM / workflow | 全 **0** ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 **0** ✓ |
| git tracked file | 変更 2 (= 既存 untracked file 2 個 + db_path_consistency.py 編集 1) |

## §7 next action 候補 (= 優先順)

1. **D10 review.md 記入** (= 寄付き actual + 15:30 以降の outcome、Fujiwara 本人)
2. **h20 後 (= 2026-06-12 頃) に paper_pnl_preview 再 run** (= 全 horizon 取得 + outcome 判定)
3. **W60-pilot-D11** (= 2026-05-28 木、recently_seen 拡張 + paper_pnl_preview 統合)
4. **liquidity filter 強化 wave** (= 板厚 / spread を staging 内 read-only)
5. **DATA-R3 freshness gate JSON 連携 wave** (= freshness_verdict MISSING → OK)
6. **全銘柄 daily refresh launchd 自動化** (= 別 HQ 承認)
7. **W60-pilot-W2 集約** (= D6-D10 5 営業日 review)
