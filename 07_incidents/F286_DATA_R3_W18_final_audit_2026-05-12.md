# F286-DATA-R3-W18 Final Audit Report

task_id: F286-DATA-R3-W18-final-audit  
lane: Codex-Audit / W18-4  
scope: W18-1 f100, W18-2 f101, W18-3 f119 smoke artifact audit  
mode: static artifact review only

## Severity Summary

- CRITICAL: 0
- HIGH: 0
- LOW / INFO observations: 3

## Findings

### CRITICAL

なし。

### HIGH

なし。

### LOW-1: W18-1 の 5 桁 code 形式は許可成果物だけでは行単位再検算不可

`/tmp/w18_1/step2_stdout.txt` は対象期間、銘柄数、挿入 3 行、失敗 0 を示しているが、実際に保存された `code` 値の行ダンプは含まれていない。DB open 禁止のため、`72030/99840/67580` の 5 桁保存は本 audit では直接再検算していない。

Verdict: blocker ではない。W18 briefing の主張として扱うが、次回以降は smoke 側で `SELECT code,date ...` の結果を `/tmp/w18_1/inserted_rows.txt` に残すと audit 独立性が上がる。

### LOW-2: W18-2 は production/develop の post mtime が直接成果物に含まれない

`/tmp/w18_2/pre_mtimes.txt` は production/develop/staging を含む一方、`/tmp/w18_2/post_mtime.txt` は staging のみ。API 403 で fetch が停止し、`announcements` count は 1098 のまま、staging mtime も 18:45 で不変のため書込未発生とは整合する。ただし production/develop の post mtime は W18-2 単体成果物だけでは直接比較できない。

Verdict: safety defect ではない。次回 smoke では post 側も pre と同じ 3 DB mtime 形式に揃えること。

### INFO-1: `/tmp/f119_eval_summary.json` は W18-3 の正式成果物ではなく stale artifact

W18-3 stdout の artifact 一覧は `/tmp/w18_3/result.json`、`/tmp/f119_eval_summary.csv`、`/tmp/f119_eval_insights.json`、3 candidate CSV。`/tmp/f119_eval_summary.json` は May 10 12:44 の古いファイルで、W18-3 の run_at とは一致しない。

Verdict: W18-3 成功判定には影響なし。正式 JSON は `/tmp/w18_3/result.json`。

## Audit Verdict By Viewpoint

### A. W18-1 f100 smoke

Verdict: PASS with evidence note.

- `/tmp/w18_1/step2_stdout.txt`: `挿入: 3 行`, `失敗銘柄: 0`
- `market_prices_daily` count: pre `2085284`, post `2085284`
- mtime: staging `18:24` -> `18:45`; production `16:17` unchanged; develop `16:11` unchanged
- Count unchanged is consistent with `INSERT OR REPLACE` updating existing `(code, date)` rows rather than adding net new rows
- Guard implementation requires `--db-label staging`, basename `fire.staging.db`, non-symlink, forbidden inode mismatch, and `FIRE_ENV=staging`
- 5 digit code claim is not row-dumped in W18-1 artifacts; see LOW-1

### B. W18-2 f101 smoke

Verdict: EXPECTED PARTIAL FAIL / SAFETY PASS.

- `/tmp/w18_2/step2_stdout.txt`: `JQuantsAnnouncementError: HTTP 403`
- Error body states requested endpoint does not exist / check API spec
- `announcements` count: pre `1098`, post `1098`
- staging mtime remains `18:45`, unchanged from W18-1 post
- F101 guard implementation requires staging label, staging basename, non-symlink, forbidden inode mismatch, and `FIRE_ENV=staging`
- Failure occurred during API fetch before write; this is consistent with no staging write
- Endpoint/plan issue is outside this audit scope

### C. W18-3 f119 no-line smoke

Verdict: PASS.

- stdout: mode `READ-ONLY (no DB write)`
- stdout/result: `count=0`, `strong_candidates=0`, `avoid_candidates=0`, `caution_candidates=0`
- target DB: `/Users/bluefire/fire/data/fire.staging.db`
- pre/post mtimes unchanged for production, develop, and staging
- `/tmp/w18_3/result.json` has `constraints_check.no_db_write=true`, `production_db_untouched=true`, `develop_db_untouched=true`
- `count=0` is data availability / condition mismatch, not runner failure
- script is read-only by construction via `sqlite3.connect(... mode=ro)` and only writes JSON/CSV artifacts
- No LINE path is present in this runner; `evaluation/orchestrator.py` separately has `F286_LINE_DISABLE=1` fail-safe for `send_line`

### D. Whole W18 Safety

Verdict: PASS.

- Production DB mtime unchanged in W18-1 and W18-3 direct pre/post evidence; W18-2 has no post production/develop artifact but no write path reached
- Develop DB mtime unchanged in W18-1 and W18-3 direct pre/post evidence; W18-2 same evidence note as above
- Staging writes only observed in W18-1; W18-2 and W18-3 show no staging mtime change
- No token/channel token/secret value appears in reviewed stdout/result artifacts
- Static grep of allowed runner files found no `subprocess`, `Popen`, cron registration, Rakuten, or Computer Use execution path
- No pytest, DB open, commit, or production/test code change was required for this audit

### E. W17 Fix Effect

Verdict: PASS.

- F100 staging-only guard is active and W18-1 completed a staging write while production/develop stayed unchanged
- F101 staging-only guard is active; W18-2 passed guard far enough to reach API fetch, then stopped before write on HTTP 403
- F119 read-only/no-line path is active; W18-3 wrote only `/tmp` artifacts and did not change DB mtimes

### F. Residual Issues

Verdict: NON-BLOCKING / FOLLOW-UP.

- F101 HTTP 403 requires separate API endpoint / plan investigation
- Add row-level W18-1 evidence for inserted/upserted `code,date` values if future audit must independently verify 5 digit code format without DB access
- Normalize W18-2 post mtime artifact to include production/develop/staging
- Rollback need: none indicated by current evidence; HQ can decide, but no production/develop write evidence was found

## Smoke Verdicts

- W18-1 f100 staging write smoke: PASS
- W18-2 f101 staging write smoke: PARTIAL FAIL EXPECTED / SAFETY PASS
- W18-3 f119 no-line smoke: PASS

## Overall Judgment

W18 safety posture is acceptable. No CRITICAL/HIGH issue was found. The only failed smoke behavior is W18-2 API HTTP 403, which is an upstream endpoint/plan problem and not evidence of a staging guard or DB safety failure.
