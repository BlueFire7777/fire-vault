---
id: FIRE-CODEX-R1-WAVE17-plan
phase: ガバナンス / Wave 17 起票 / R-01-08
priority: 最優先
status: 起票 ☆ 7 sub-task 並列実行中 (2026-05-12)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 16 (= 完了)
  - HQ Wave 17 一括承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / sub-D2.3.x activation
---

# Wave 17: sub-D2.3.x guard implementation + f111 staging write smoke

W16-5 audit HOLD 解消 + f111 実 staging write smoke。

| sub | lane | task |
|---|---|---|
| W17-1-fix | Codex L3+L2 | f100 guard + tests |
| W17-2-fix | Codex L3+L2 | f101 guard + schema migration 制御 + tests |
| W17-3 | 本線 | f111 staging write smoke (= 実 staging write) |
| W17-4-fix | Codex L3+L2 | f119 LINE disable contract + tests |
| W17-5 | Codex L4 | 全 fix + smoke 再 audit |

W17-3 HQ 条件:
- FIRE_ENV=staging、target research_watchlist_signals
- production/develop write 禁止、pre/post mtime 記録
- inserted/updated/skipped/row_count、PK 衝突回避
- LINE 0、token 0、cron 0、1 ブロック HQ 報告
