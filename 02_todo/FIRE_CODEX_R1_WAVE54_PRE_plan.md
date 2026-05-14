---
id: FIRE-CODEX-R1-WAVE54-pre-plan
phase: 本番 v0 後拡張 / Wave 54-pre
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 54-pre Plan — F286-AFTER-R1 Read-Only Runner Design / Scaffold v1.0

## §1 目的
v0 後拡張 F286-AFTER-R1 の read-only runner scaffold を前倒し整備。
実 task 実行 / DB write / Paper Live 実行 / API call / cron 登録は 0。

## §2 方針
design + scaffold + tests + docs まで。既存 v0 path には触らない。
Codex lane 0/8 (= prescriptive 実装、precedent 継承)。

## §3 実施項目
1. baseline + 既存 AFTER-R1 design 確認
2. CLI scaffold 新規 (= `run_f286_after_r1_night_batch.py`、約 600 行)
3. test 新規 (= 約 600 行 / 56 test、12 class)
4. 03_design に AFTER-R1 read-only design v1.0 起票
5. F282 / DB / pytest 不干渉確認
6. 02_todo に W54-pre plan / results
7. HQ 1 ブロック + 6 KPI

## §4 期待成果物
- `~/fire/scripts/jobs/run_f286_after_r1_night_batch.py` (新規)
- `~/fire/tests/scripts/jobs/test_run_f286_after_r1_night_batch.py` (新規)
- `~/fire-vault/03_design/F286_AFTER_R1_read_only_runner_design_2026-05-14.md` (新規)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE54_PRE_plan.md` (本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE54_PRE_results.md`

## §5 完了条件
- scaffold 完成、default dry-run + read-only
- DB write / DB sqlite 接続 / LINE / token / API / launchctl / cron / VACUUM 全 0
- Paper Live 実実行 / 楽天証券操作 0
- JSON / Markdown / text 出力可
- tests PASS + collected 4423 → 4479 (+56)
- F282 / 3 DB 不変
- vault に W54-pre 報告 + HQ 1 ブロック + 6 KPI

## §6 停止条件 (= W52-post と同じ + 拡張)
DB write / DB sqlite 接続 / production・develop・staging DB / LINE 送信 /
token / secret / channel_token / .env / env 全体参照 / launchctl 実行 /
plist 配置 or 変更 / cron / crontab 変更 / VACUUM / Paper Live 実実行 /
workflow / --no-verify / git push / sudo / rm -rf / TODO Excel / 楽天 /
自動発注 / Computer Use が必要 → 即停止 + HQ 確認
