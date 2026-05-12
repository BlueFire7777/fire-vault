# F286-PNL-R3-MIG-R1-fix-table-not-exists Audit Report (= Wave 12 W12-3-fix-audit)

CRITICAL: 0 / HIGH: 0

## CRITICAL findings

None.

## HIGH findings

None.

## MEDIUM findings

None.

## LOW findings / note

- Static review only. pytest was not executed per instruction.
- Reviewed files:
  - `/Users/bluefire/fire/scripts/setup/migrate_paper_reason.py`
  - `/Users/bluefire/fire/tests/scripts/setup/test_migrate_paper_reason.py`
- The no-table path returns before `_advisory_decision_columns()`, `_has_paper_reason_column()`, or `ALTER TABLE` can run. The `finally: conn.close()` still executes on that early return.
- Script SQL in production code is limited to:
  - `SELECT name FROM sqlite_master WHERE type='table' AND name=?`
  - `PRAGMA table_info(advisory_decisions)`
  - `ALTER TABLE advisory_decisions ADD COLUMN paper_reason TEXT`

## 観点別 verdict (= A-G)

| 観点 | verdict |
|---|---|
| A. table 存在チェック | OK |
| B. no_table_skipped 安全性 | OK |
| C. 既存 action 維持 | OK |
| D. SQL 限定性 | OK |
| E. forbidden import / side effect | OK |
| F. test カバレッジ | OK |
| G. production/develop refuse 不変 | OK |

## 総合判断

- merge_recommendation: APPROVE
- rationale: `_has_advisory_decisions_table()` uses a hardcoded table name with sqlite parameter binding, so maliciously named tables do not satisfy the check. `migrate()` performs the table existence check immediately after connecting and returns `no_table_skipped` for both dry-run and write modes before any PRAGMA/ALTER path is reachable. Existing actions for table-present cases are preserved, rollback on `ALTER` failure remains in place, and production/develop HQ marker enforcement is unchanged.
