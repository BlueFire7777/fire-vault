---
id: FIRE-CODEX-R1-WAVE51-plan
phase: 本番 v0 Launch / readiness CLI v1.2 CRITICAL fix
priority: 高
status: plan (= Wave 51 着手前計画)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 51 Plan — Production v0 Readiness CLI v1.2 CRITICAL Fix

## §1 目的

W44.5-post adversarial audit で発見された readiness CLI v1.1 CRITICAL 3 件を
修正し、Production v0 D-Day の GO/NO-GO 判定を **安全側に倒す**。

## §2 W44.5-post で指摘された CRITICAL 3 件

1. HQ marker phase gating bug (= silent GO リスク)
2. D-Day weekday Tuesday 固定 WARN (= strict false NO-GO リスク)
3. F282 post-run report 設計 doc vs 実装 gap

## §3 方針

- readiness CLI v1.2 修正 + tests + docs のみ
- DB / LINE / token / launchctl / plist / cron / VACUUM 触らない
- FIRE 本体コード変更は CLI ファイル + tests ファイルのみ
- 既存 v1.1 32 test と後方互換 (= check_id schema / 25 check 構成維持)
- CRITICAL fix の挙動変化 (= WARN→FAIL 等) は意図的、test 側更新で対応

## §4 実施項目

1. baseline capture (= F282 / 3 DB / pytest)
2. CLI v1.1 + tests 読込で現状理解
3. (8 Codex lane 投入候補だが、CRITICAL 所在明確なので 0 で本線完結を予定。
   理由は results で明記)
4. CLI v1.2 実装:
   - CLI_VERSION = "1.2"
   - PHASE_ORDER / HQ_MARKER_FIRST_REQUIRED_PHASE / D_DAY_HQ_LOCKED 定数追加
   - `_check_hq_marker` を cumulative phase gating に変更
   - `check_d_day_weekday_hq_decision_pending` lock 判定追加
   - `check_f282_post_run_report_exists` phase 別 WARN/FAIL 切替
   - `collect_missing_markers` + JSON output に missing_markers 追加
5. tests 追加 (= 5 class / 約 27 件 + AST safety 6 件)
6. pytest 全 PASS 確認 + 全体 collect 数報告
7. docs 起票 (= 03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14.md)
8. F282 / DB 不干渉再確認
9. vault に W51 plan / results 起票
10. HQ 1 ブロック報告 + 6 KPI

## §5 期待成果物

- `~/fire/scripts/jobs/run_production_v0_readiness_check.py` (= v1.1 → v1.2)
- `~/fire/tests/scripts/jobs/test_run_production_v0_readiness_check.py` (= +27 test)
- `~/fire-vault/03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14.md` (= 新規)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE51_plan.md` (= 本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE51_results.md`

## §6 停止条件 (= W44.5-post と同じ)

DB write / DB sqlite 接続 / production・develop・staging DB / LINE 送信 /
token / secret / channel_token / .env / env 全体参照 / launchctl 実行 /
plist 配置 or 変更 / cron / crontab 変更 / VACUUM / VACUUM INTO /
F282 手動実行 / workflow 変更 / --no-verify / git push / sudo / rm -rf /
TODO Excel 更新 / 楽天証券操作 / 自動発注 / Computer Use が必要 → 即停止 + HQ 確認

## §7 完了条件

- readiness CLI v1.2 修正完了
- W44.5-post CRITICAL 3 件全解消
- marker 不足で silent GO にならない
- D-Day 2026-06-09 火曜 lock が strict false NO-GO にならない
- F282 post-run report path / schema が W40.8 / W44-pre と整合
- pytest 全 PASS + collected 数 (= 4202 → 4229) 報告
- F282 本番 plist mtime/size/next run 不変確認
- production/develop/staging DB mtime/size 不要変更なし確認
- LINE/token/API/launchd/plist/cron/VACUUM/workflow/--no-verify/TODO Excel 全 0
- fire-vault に W51 報告作成
- HQ 1 ブロック報告 + 6 KPI
