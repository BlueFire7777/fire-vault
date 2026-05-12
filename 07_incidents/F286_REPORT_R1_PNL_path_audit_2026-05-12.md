# REPORT-R1 / PNL Path Audit (= Wave 15 W15-3)

CRITICAL: 0 / HIGH: 3

## CRITICAL findings

なし。

## HIGH findings

1. `scripts/jobs/run_f286_report_r1_line_preview.py` が send guard marker を明示 input ではなく環境変数から読む
   - evidence: `import os` と `os.environ.get(_HQ_SEND_MARKER)` が `scripts/jobs/run_f286_report_r1_line_preview.py:12` / `:162-169` に存在。
   - impact: W11-2a-fix / 観点 H の「hq_approve_marker 明示 input」とずれる。LINE client / `--send` は未実装なので実送信には直結しないが、process env に `F286_LINE_HQ_APPROVE=production` が残っているだけで `can_send=True` 診断になり得る。
   - recommendation: runner に `--hq-approve-marker` のような明示引数を追加し、未指定なら常に `None` として `can_send=False` にする。`line_delivery.evaluate_send_guard()` 自体は明示 input 設計になっている。

2. `scripts/jobs/run_f286_pnl_r2_ingest_snapshot.py` の output 書き込みが atomic create ではなく上書き
   - evidence: `_write_output()` / `_write_completion_report()` が `Path.write_text()` を使う (`scripts/jobs/run_f286_pnl_r2_ingest_snapshot.py:316-321`, `:350-351`)。
   - impact: 観点 C の atomic create (`x` mode) を満たさない。DB/WAL/SHM basename・suffix・既存 symlink・production/develop inode は拒否しているが、通常ファイルへの上書きは可能。
   - recommendation: REPORT runners / PNL-R3 runner と同じく `open(path, "x", encoding="utf-8")` に統一し、既存 file は `FileExistsError` で refuse する。

3. PNL-R3 compute runner の schema validation が `paper_reason` を必須列に含めていない
   - evidence: `_validate_required_schema()` の `advisory_decisions` 必須列は `advisory_id/code/base_date/decision_label/paper_pnl/updated_at` までで、`paper_reason` がない (`scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py:357-367`)。
   - impact: 観点 F の「paper_pnl + paper_reason 列」前提を runner 自身では検査できない。W14-3 の full schema DDL には 22 列目 `paper_reason` が含まれているため、migration 側の schema 定義は成立している。
   - recommendation: PNL-R3 の required schema に `paper_reason` を追加し、SCHEMA-R1 前提のズレを fail-fast にする。

## MEDIUM findings

1. `REPORT-R1` weekly/monthly/line_preview は F119 latest path の自動探索で filesystem を読む
   - evidence: weekly/monthly runner と line_preview runner が `find_latest_f119_report_path()` を呼ぶ (`scripts/jobs/run_f286_report_r1_weekly.py`, `scripts/jobs/run_f286_report_r1_monthly.py`, `scripts/jobs/run_f286_report_r1_line_preview.py:83`)。
   - note: 観点 B の「markdown_renderer が filesystem 読まないこと」は満たす。renderer は入力文字列だけを render する。ただし runner 層まで完全 deterministic にしたい場合は `--f119-report-path` 明示指定を推奨。

## LOW findings / note

- REPORT-R1 daily/weekly/monthly runner は `file:<resolved>?mode=ro` + `PRAGMA query_only=ON` を使用し、DB write SQL は見当たらない。
- `fire/report/aggregators.py` は `SELECT` と `sqlite_master` existence check のみ。`advisory_decisions` / `actual_trades` がない場合は 0 集計を返すため、空 data Markdown 生成の fail-safe は成立。
- `fire/report/markdown_renderer.py` は filesystem / DB / env / HTTP / LINE client を読まない。
- `fire/report/line_delivery.py` は LINE client / HTTP client を import せず、raw recipient は `_mask_recipient_id()` で `REPORT (***)` または `REPORT (X***)` に変換し、strict regex で guard する。
- `--send` flag は audit 対象内に見当たらない。
- `linebot` / `requests` / `aiohttp` import、`LINE_CHANNEL_*` 参照、生 recipient ID の stdout/JSON 露出は audit grep では見当たらない。
- W14 backup:
  - `/Users/bluefire/fire-backups/fire.db.pre_schema_r1_20260512_161611` exists。
  - `.sha256` exists。
  - sha256 match: `7068b959995eb612817abfdf4ae70952fb84544d170696dba7ac2cf44a4418e9`。

## 観点別 verdict (= A-H)

A. production / develop read-only 保証: PASS
- REPORT-R1 daily/weekly/monthly/line_preview は read-only URI + `PRAGMA query_only=ON`。
- REPORT path の SQL は SELECT/PRAGMA/table existence check のみ。

B. REPORT-R1 DB write 不在: PASS
- `fire/report/aggregators.py`, `daily_report.py`, `weekly_report.py`, `monthly_report.py`, `markdown_renderer.py` に DB write SQL は見当たらない。
- `markdown_renderer.py` は filesystem を読まない。

C. output path safety: PARTIAL / HIGH finding
- REPORT-R1 runners と PNL-R3 runner / seed runner は `open(..., "x")`。
- PNL-R2 ingest runner は `Path.write_text()` で既存通常ファイルを上書き可能。

D. secret / token 参照なし: PARTIAL / HIGH finding
- LINE token / `LINE_CHANNEL_*` / HTTP client / LINE client import は見当たらない。
- line_preview runner は secret ではないが process env を読むため、「env 全体読出排除 / marker 明示 input」方針とは不一致。

E. schema mismatch 時の fail-safe: PASS
- SCHEMA-R1 migration は 22 列 template + 2 indexes + CHECK constraint を検証し、production/develop mismatch は non-zero。
- REPORT-R1 aggregators は対象 table 不在/空 data で 0 aggregate を返し、Markdown renderer は空 data でも正常文字列を生成する構造。

F. PNL-R1/R2/R3 schema 前提成立: PARTIAL / HIGH finding
- `migrate_advisory_decisions_full.py` は `paper_pnl` と `paper_reason` を含む 22 列 schema を定義。
- PNL-R3 compute runner の runtime schema validation は `paper_reason` を必須にしていない。

G. W14 backup 保持状態: PASS
- backup file と `.sha256` file の存在を確認。
- sha256 は一致。

H. LINE delivery send guard: PARTIAL / HIGH finding
- `line_delivery.evaluate_send_guard()` は marker 明示 input、marker 不在なら `can_send=False`、recipient mask regex も厳格。
- runner が marker を env から読むため「明示 input」の運用保証が崩れる。
- `--send` flag は未実装。

## 総合判断

- merge_recommendation: HOLD

REPORT-R1 の read-only / no-send 中核は概ね成立。production/develop DB write や LINE 実送信に直結する CRITICAL は見当たらない。

ただし W15 application path activation の audit gate としては、HIGH 3 件を解消してから merge 推奨。特に line_preview の marker 明示 input 化、PNL-R2 output の atomic create 化、PNL-R3 `paper_reason` schema validation 追加は小さく直せる。
