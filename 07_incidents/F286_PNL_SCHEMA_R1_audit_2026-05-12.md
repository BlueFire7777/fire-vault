# F286-PNL-SCHEMA-R1-impl Audit Report (= Wave 13 W13-1b)

CRITICAL: 0 / HIGH: 1

## CRITICAL findings

なし。

## HIGH findings

1. 既存 `advisory_decisions` の schema mismatch 判定が列名のみで、PK / indexes / CHECK / DEFAULT / NOT NULL の欠落を検出できない。

対象: `scripts/setup/migrate_advisory_decisions_full.py`

`_verify_schema_matches_template()` は `_EXPECTED_COLUMNS - col_names` のみで一致判定しているため、22 列が揃っているが以下が欠落・不一致の既存 table を `skip_already_exists` と判定する。

- composite PK `(advisory_id, code)`
- `idx_advisory_decisions_base_date`
- `idx_advisory_decisions_fujiwara_decision`
- `fujiwara_decision` / `actual_trade` の `CHECK`
- `fujiwara_decision DEFAULT 'unknown'`
- `actual_trade DEFAULT 'none'`
- `advisory_id` / `code` / `fujiwara_decision` / `actual_trade` / `created_at` / `updated_at` の `NOT NULL`

production / develop apply 前の gate としては、既存 table が「列だけ揃った不完全 schema」の場合に warning skip にならず、正常 skip と誤判定される。特に index 欠落は既存 table で起こりやすく、現在の実装は table 既存時に `CREATE INDEX IF NOT EXISTS` も発行しないため補完されない。

推奨: 既存 table 判定で `PRAGMA table_info`, `PRAGMA index_list`, `PRAGMA index_info`, `sqlite_master.sql` を使い、少なくとも PK / expected indexes / CHECK token / DEFAULT / NOT NULL を検査し、不一致時は `schema_mismatch_warning_skipped` にする。既存 table への index 補完を許容するかは SQL 限定性方針と別途判断。

## MEDIUM findings

1. SQL 限定性 test は DDL 変数経由の `execute()` だけを検査しており、literal SQL や helper 内 SQL を検査対象に含めていない。

対象: `tests/scripts/setup/test_migrate_advisory_decisions_full.py`

現状 script に forbidden mutating SQL は見当たらないため実害はないが、将来 `conn.execute("ALTER ...")` のような literal SQL が追加されても、この test は検出できない。

2. schema mismatch test が「列不足」ケースのみで、列は揃っているが constraint / default / PK / index が不一致の境界を網羅していない。

HIGH finding の再発防止 test として、全 expected columns を持つ不完全 table を作り、`schema_mismatch_warning_skipped` になることを確認する test が必要。

## LOW findings / note

- `migrate_paper_reason.py` と同じ basename check / symlink refuse / resolved basename mismatch / HQ marker pattern は概ね継承されている。
- forbidden import は `argparse`, `os`, `sqlite3`, `sys`, `pathlib`, `typing` の範囲で、linebot / requests / aiohttp / 楽天 / Playwright / subprocess import は見当たらない。
- 環境変数参照は `_HQ_APPROVE_ENV = "F286_SCHEMA_R1_HQ_APPROVE"` のみ。
- dry-run は table 不在時に DDL を実行せず `dry_run_would_create` を返す。
- staging は HQ marker 不要、production / develop は marker 値が db_label と一致しない限り拒否する。
- test 件数は静的確認上 20 件。主要 boundary はあるが、既存 full-column bad schema の境界が欠けている。

## 観点別 verdict (= A-H)

- A. SQL 限定性: PASS。実装上、write path の mutating SQL は `CREATE TABLE IF NOT EXISTS` と `CREATE INDEX IF NOT EXISTS` のみ。SELECT / PRAGMA は read-only。
- B. idempotent: PARTIAL。新規作成後の 2 回目 skip と列不足 mismatch warning skip は満たすが、列が揃った schema 不一致を skip_already_exists と誤判定する。
- C. schema integrity: NG。create DDL 自体は 22 columns including `paper_reason`, PK, 2 indexes, CHECK, DEFAULT, NOT NULL を持つが、既存 schema integrity 検証が不十分。
- D. HQ marker 運用: PASS。production / develop は marker enforce、staging は不要。
- E. staging-only / symlink refuse: PASS。basename check、symlink refuse、resolved basename mismatch refuse は参考 script と同系統。
- F. forbidden import / side effect: PASS。禁止 import なし、env 参照は HQ marker key のみ。
- G. test カバレッジ: PARTIAL。20 件はあるが、schema integrity mismatch の重要 boundary と literal SQL 検査が不足。
- H. W10-1 整合性: PASS。参考 migration と同じ安全 gate pattern を継承し、独立 script として分離されている。

## 総合判断

- merge_recommendation: NG until HIGH finding fixed
- Wave 14 着手判定: NG
