# F286-REPORT-R1 LINE Delivery Preview Audit Report (= Wave 11 W11-2b)

CRITICAL: 1 / HIGH: 2

## CRITICAL findings

1. `evaluate_send_guard()` reads the whole process environment, which violates the token/channel secret non-reference requirement.
   - Evidence: `/Users/bluefire/fire/fire/report/line_delivery.py:170-171` uses `env = dict(os.environ)` when `env` is omitted.
   - Impact: even though the code does not look up `LINE_CHANNEL_ACCESS_TOKEN` by name or print it, it still copies every environment value into the function scope. If LINE tokens or secrets exist in the process environment, this path references them. This contradicts the audit requirement "env から token / channel_token / secret 値 不 読出".
   - Related runner path: `/Users/bluefire/fire/scripts/jobs/run_f286_report_r1_line_preview.py:160-162` calls `evaluate_send_guard(preview, args.db_label)` without passing a restricted env mapping when `--evaluate-send-guard` is set.

## HIGH findings

1. `send_guard` masking validation is too permissive and can allow `can_send=True` for labels containing raw recipient material.
   - Evidence: `/Users/bluefire/fire/fire/report/line_delivery.py:69-74` only checks `target_room.startswith("REPORT (")`, `target_room.endswith(")")`, and presence of `"***"`.
   - Example by inspection: `LineDeliveryPreview(target_room="REPORT (U1234567890***)", estimated_send_count=1, ...)` with `db_label="production"` and `F286_LINE_HQ_APPROVE=production` would pass the masking predicate, despite retaining most of the raw ID.
   - Impact: the generated preview currently uses `_mask_recipient_id()`, but `evaluate_send_guard()` is a public helper and should enforce masking independently. This weakens D. send_guard refuse behavior and C. recipient masking enforce.

2. Preview chunks do not enforce the iPhone copy format of "each chunk is one code fence with no internal fence".
   - Evidence: `/Users/bluefire/fire/fire/report/line_delivery.py:77-100` splits raw markdown into chunks, and `/Users/bluefire/fire/fire/report/line_delivery.py:108-112` stores those chunks directly.
   - Evidence: `/Users/bluefire/fire/scripts/jobs/run_f286_report_r1_line_preview.py:133-136` prints each raw chunk after a `[chunk n/m]` label, without wrapping each chunk in its own single fence or escaping internal fences.
   - Test gap: `/Users/bluefire/fire/tests/report/test_line_delivery.py:73-78` checks only the joined body for two fences, not that every individual chunk has exactly one opening and one closing fence, and not that internal fences are absent after splitting.

## MEDIUM findings

1. Test coverage does not cover all required A-H invariants.
   - `line_delivery.py` has an AST forbidden-import test at `/Users/bluefire/fire/tests/report/test_line_delivery.py:190-199`, but no AST assertion forbids `os.environ`, `os.getenv`, token/secret literals, `Authorization`, or `Bearer` in that module.
   - Runner token coverage at `/Users/bluefire/fire/tests/scripts/jobs/test_run_f286_report_r1_line_preview.py:127-132` checks only the runner source and omits `line_delivery.py`, where the environment read exists.
   - No test asserts `evaluate_send_guard()` refuses a masked-looking label containing raw recipient text such as `REPORT (U1234567890***)`.
   - No test asserts per-chunk code fence format after chunk splitting.

## LOW findings / note

1. Actual LINE send paths were not found in the audited implementation.
   - No `linebot`, `requests`, `aiohttp`, or `subprocess` imports in the target implementation files.
   - No HTTP `POST`, push, broadcast, reply, or send helper path was found beyond dataclass field names and guard diagnostic text.

2. Read-only DB handling is delegated to the existing daily runner helper and appears consistent.
   - Evidence: `/Users/bluefire/fire/scripts/jobs/run_f286_report_r1_daily.py:176-180` opens `file:...?mode=ro` with `uri=True` and executes `PRAGMA query_only=ON`.
   - Evidence: the LINE preview runner uses `_connect_readonly()` at `/Users/bluefire/fire/scripts/jobs/run_f286_report_r1_line_preview.py:148`.

3. W8-3 / W10-2 / W10-3 report generators are used as read-only report producers.
   - Evidence: `/Users/bluefire/fire/scripts/jobs/run_f286_report_r1_line_preview.py:77-96` calls `generate_daily_report`, `generate_weekly_report`, and `generate_monthly_report`.
   - The inspected report orchestrators state no DB writes and call aggregate/render helpers only.

4. `fire.report.__init__` re-export exists.
   - Evidence: `/Users/bluefire/fire/fire/report/__init__.py:30-37` imports the LINE preview helpers, and `/Users/bluefire/fire/fire/report/__init__.py:73-86` includes them in `__all__`.

## 観点別 verdict

| 観点 | verdict |
|---|---|
| A. 実 LINE 送信不発生 | OK |
| B. token 不参照 | NG |
| C. recipient masking | NG |
| D. send_guard refuse | NG |
| E. 責任分離 | OK |
| F. forbidden import | NG |
| G. iPhone format | NG |
| H. read-only DB | OK |
| I. test カバレッジ | NG |
| J. 既存 contract 整合 | OK |

## 総合判断

- merge_recommendation: Do not merge as-is.
- Rationale: actual LINE delivery routes are absent, and DB access is read-only, but the hard audit requirement for token non-reference is violated by copying `os.environ`. The send_guard masking predicate is also insufficiently strict, and the preview chunk format does not enforce the required iPhone copy contract.
- Required before merge: remove whole-environment reads; pass/read only `F286_LINE_HQ_APPROVE`; make `send_guard` accept only the exact masked recipient label shape generated by the preview helper; enforce or explicitly validate per-chunk code fence formatting; add AST/tests for token/env access and per-chunk format.
