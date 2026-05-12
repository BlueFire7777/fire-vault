# DATA-R3 Write Guard Audit (= Wave 16 W16-5)

CRITICAL: 0 / HIGH: 3

Scope: static review only. No runner execution, no pytest, no DB open, no LINE/API call, no token value read.

## CRITICAL findings

None.

## HIGH findings

1. HIGH: f100 direct write has no staging-only guard.
   - File: `/Users/bluefire/fire/scripts/jobs/fetch_historical_market_data.py`
   - Evidence: CLI accepts only `--db-path` and `--dry-run`; no `--db-label`, no basename/symlink/resolved basename/FIRE_ENV guard before real fetch/write. Real mode instantiates `HistoricalDataFetcher(db_path=Path(args.db_path))` and calls `fetch_daily()` (lines 140-175).
   - Write SQL: delegated to `/Users/bluefire/fire/market_data/repository.py`, `INSERT OR REPLACE INTO market_prices_daily (...)` in `save_daily_prices()` (lines 50-72).
   - Impact: a direct W17+ write invocation can write to any supplied sqlite path, including production/develop, if operator passes that path.
   - Gate verdict: HOLD for real write until f100 has the same staging-only guard pattern or is invoked only through a separately guarded wrapper that is the exclusive approved write path.

2. HIGH: f101 direct write has no staging-only guard, and schema migration writes can happen before fetch.
   - File: `/Users/bluefire/fire/scripts/jobs/fetch_announcements.py`
   - Evidence: CLI accepts only `--db-path` and `--dry-run`; no `--db-label`, no basename/symlink/resolved basename/FIRE_ENV guard. Real mode constructs `AnnouncementFetcher(db_path=Path(args.db_path))` (line 135).
   - Write SQL: `/Users/bluefire/fire/materials/fetcher.py` calls `ensure_announcements_schema()` in `AnnouncementFetcher.__init__()` (lines 201-212). That function can `CREATE TABLE`, `ALTER TABLE`, `CREATE INDEX`, and commit (lines 54-125). Main fetch path then `INSERT INTO announcements (...)` (lines 267-291).
   - Impact: direct invocation against production/develop can mutate schema and rows before any staging-only check.
   - Gate verdict: HOLD for real write until f101 has staging-only guard before `AnnouncementFetcher` construction.

3. HIGH: F119 LINE disable is available in lower-level components, but no F286-specific hard disable flag is present.
   - Files: `/Users/bluefire/fire/evaluation/orchestrator.py`, `/Users/bluefire/fire/notifications/line_bot.py`, `/Users/bluefire/fire/notifications/router.py`, `/Users/bluefire/fire/scripts/jobs/run_f119_interpretation_evaluation.py`
   - Evidence: `run_evaluation(..., send_line=True)` defaults to LINE send and imports `send_to_room(Room.REPORT, ...)` when `send_line` is true (orchestrator lines 56-63, 132-145). `LineBotClient` supports `LINE_DRY_RUN=true` (line_bot lines 51-60, 95-99). The audited f119 runner does not call `evaluation.orchestrator.run_evaluation`; it is read-only and writes only output artifacts under caller paths (runner lines 402-437).
   - Impact: this specific f119 runner is LINE-safe, but R-13-08/F119 REPORT notification paths outside this runner need an explicit smoke-time disable contract. There is no `F286_LINE_DISABLE` flag found in the reviewed code.
   - Gate verdict: HOLD for any smoke path that uses `evaluation.orchestrator.run_evaluation` unless it explicitly passes `send_line=False` or sets `LINE_DRY_RUN=true` with verified no token propagation.

## MEDIUM findings

1. MEDIUM: daily_refresh write guard is weaker than the six-stage pattern if called through helper functions.
   - File: `/Users/bluefire/fire/scripts/jobs/run_f286_data_r3_daily_refresh.py`
   - Evidence: `_assert_staging_write_allowed()` checks only db label membership and raw path basename (lines 216-226). It does not reject symlinks, does not inspect resolved basename, and does not require `FIRE_ENV=staging`.
   - Mitigation in current CLI path: `--write` enters `plan_daily_refresh(... read_only=False)`, which raises `NotImplementedError` before schema write (lines 532-536, 640-655). Therefore current CLI real write is blocked.
   - Residual risk: `ensure_daily_refresh_schema()` is a callable helper that writes after only the weak guard (lines 577-590).

2. MEDIUM: f119 is read-only for DB but writes local output artifacts by default.
   - File: `/Users/bluefire/fire/scripts/jobs/run_f119_interpretation_evaluation.py`
   - Evidence: no DB write option exists, DB is opened with `mode=ro` for evaluation (line 289), but default artifact paths are `/tmp/f119_eval_*.json/csv` and writers create parent dirs/files (lines 402-437, 596-616).
   - Impact: OK for DB-write gate, but not "write 0" globally unless artifact paths are accepted.

3. MEDIUM: daily_refresh subprocess env intentionally strips secrets but can make f100/f101 dry-runs fail config probes.
   - File: `/Users/bluefire/fire/scripts/jobs/run_f286_data_r3_daily_refresh.py`
   - Evidence: `_subprocess_env()` returns only `{"PYTHONUNBUFFERED": "1"}` (lines 324-325). f100/f101 dry-run probes require J-Quants env key existence (f100 lines 45-57; f101 lines 39-51).
   - Impact: good for token isolation, but aggregate dry-run may report failure unless probe env expectations are adjusted.

## LOW findings / note

1. f111 has the strongest direct write guard among the 4 runners.
   - It rejects production/develop labels, restricts allowed label to staging, requires existing DB, rejects symlink, checks resolved basename `fire.staging.db`, and requires `FIRE_ENV=staging` before write (lines 94-133).

2. W15-3-fix effect is present for dry-run probes.
   - f100/f101 check only env key existence in dry-run and deliberately skip auth/API fetch (f100 lines 75-87; f101 lines 69-81). daily_refresh subprocess env excludes `LINE_CHANNEL_TOKEN` and other inherited secrets by construction.

3. W14 SCHEMA-R1 structural assumption is reflected in f111/f119 probes.
   - f111 dry-run requires `advisory_decisions` table existence (lines 168-179).
   - f119 accepts staging/develop/production labels and probes `advisory_decisions`, `advisory_snapshots`, `evaluation_reports` read-only (lines 69-71, 144-160).

4. PK collision review.
   - f111 writes `research_watchlist_signals`, not `advisory_decisions`; PK is `(base_date, code, source_version)` via `INSERT OR REPLACE` (signal_persistence lines 267-337). Therefore advisory_id prefixes such as `f062-r5.8-*`, `production-advisory-*`, and `staging-smoke-*` are not touched by this runner.
   - daily_refresh schema defines `advisory_decisions(advisory_id, code)` but current daily_refresh does not insert advisory rows.

## 観点別 verdict (= A-H)

| 観点 | verdict |
|---|---|
| A. write 経路 | HOLD: f100 writes `market_prices_daily`; f101 writes/migrates `announcements`; f111 writes `research_watchlist_signals`; f119 DB write none, artifact writes only. f100/f101 direct write guard missing. |
| B. staging-only refuse | HOLD: f111 mostly satisfies six-stage guard. daily_refresh helper guard is weak but CLI write is blocked. f100/f101 have no staging-only guard. f119 has no DB write. |
| C. forbidden import / external API | OK with exceptions: f100/f101 `requests` are expected data fetch paths; daily_refresh `subprocess` is expected dry-run sub-runner path. No aiohttp/Playwright/Rakuten in audited runners. LINE imports exist only in notification/orchestrator paths. |
| D. secret 露出経路 | OK/HOLD: dry-run probes use key existence only; daily_refresh subprocess env strips tokens. Real f100/f101 fetch necessarily read J-Quants API key values in client code, which is contextually expected but must not happen in smoke. |
| E. W14 SCHEMA-R1 前提 | OK: f111/f119 probe/read paths require `advisory_decisions`; f119 label set includes production/develop/staging and read-only open. |
| F. LINE 通知 disable | HOLD: audited f119 runner does not send LINE; orchestrator has `send_line=False` and `LINE_DRY_RUN=true` paths, but no `F286_LINE_DISABLE` hard flag found. |
| G. PK 衝突回避 | OK: f111 does not insert `advisory_decisions`; signal PK is `(base_date, code, source_version)`. No advisory_id collision path in current 4-runner DATA-R3 write scope. |
| H. 4 runner 個別実行の独立性 | OK/HOLD: direct individual execution is process-independent. daily_refresh dry-run loop continues per runner and aggregates failures. Real write orchestration is not implemented; f100/f101 direct safety remains HOLD. |

## 4 runner 個別 verdict (= 実 write 着手判定)

| runner | verdict | 着手前要対応 |
|---|---|---|
| f100 | HOLD | Add/enforce staging-only guard before `HistoricalDataFetcher` real write path, or prove an exclusive guarded wrapper is the only approved write entrypoint. |
| f101 | HOLD | Add/enforce staging-only guard before `AnnouncementFetcher` construction because constructor can migrate schema. |
| f111 | OK | For write smoke, invoke only with `--db staging`, basename/resolved `fire.staging.db`, non-symlink path, and `FIRE_ENV=staging`. |
| f119 | OK for DB write / HOLD for LINE-report smoke | DB is read-only; for any R-13-08 report/orchestrator smoke use `send_line=False` or `LINE_DRY_RUN=true` and document no `F286_LINE_DISABLE` exists. |

## 総合判断

- merge_recommendation: HOLD for W17+ "4 runner staging write" as a combined gate, because f100/f101 do not have direct staging-only write guards and f101 can perform schema writes during object construction.
- Proceedable subset: f111 staging write can proceed after normal staging DB path/FIRE_ENV checks. f119 can proceed for DB-read evaluation/artifact generation, but not as a LINE REPORT smoke unless LINE disable is explicitly configured.
- No code changes were made. No DB files were opened. No tests were run.
