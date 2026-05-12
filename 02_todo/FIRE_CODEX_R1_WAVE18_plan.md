---
id: FIRE-CODEX-R1-WAVE18-plan
phase: ガバナンス / Wave 18 起票 / R-01-08
priority: 最優先
status: 起票 ☆ 4 sub-task 実行中 (2026-05-12)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 17 (= 完了、guard + f111 smoke)
  - HQ Wave 18 一括承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / sub-D2.3.x staging write 完成
---

# Wave 18: f100 / f101 / f119 staging write smoke + audit

W17-1-fix / W17-2-fix / W17-4-fix で guard 実装済 + f111 smoke 成功 (W17-3) を受け、
残り 3 runner を順次 staging write smoke 実行。

| sub | lane | task |
|---|---|---|
| W18-1 | 本線 | f100 staging write smoke (= market_prices_daily) |
| W18-2 | 本線 | f101 staging write smoke (= announcements + parsed_metrics) |
| W18-3 | 本線 | f119 no-line smoke (= send_line=False + F286_LINE_DISABLE=1) |
| W18-4 | Codex L4 | 全 smoke 結果 audit |

HQ 条件: staging only、FIRE_ENV=staging、production/develop write 禁止、
mtime before/after、LINE 0 / token 0 / cron 0、1 ブロック HQ 報告。

f119 特別条件: send_line=False 明示 + F286_LINE_DISABLE=1、DB read-only、
artifact のみ。
