---
id: FIRE-CODEX-R1-WAVE44.6-pre-plan
phase: 本番 v0 Launch / Daily Command Center CLI
priority: 高
status: plan (= Wave 44.6-pre 着手前計画)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 44.6-pre Plan — Production v0 Ops Summary / Daily Command Center CLI v1.0

## §1 目的

Production v0 運用開始後、毎朝 FIRE の状態を 1 コマンドで確認できる
read-only Ops Summary CLI を新規実装。F282 / DATA-R3 / F062 /
readiness CLI v1.2 / GO-NO-GO checklist を統合し、
GO / NO-GO / HOLD / UNKNOWN を機械的に判定。

## §2 方針

- read-only CLI 新規実装 + 包括 test + docs
- DB / LINE / token / launchctl / plist / cron / VACUUM 触らない
- 既存 CLI / module への変更 0 (= 新規 file のみ)
- W43-pre / W51 で確立した AST safety pattern 継承

## §3 構成

| lane | 内容 |
|---|---|
| L5 本線 | CLI 実装 + 73 test + smoke + docs + 不干渉確認 + 報告 |

Codex lane 投入: **0/8 を計画**。理由:
- user spec が完全 prescriptive (= CLI args 11 個 / input 5 / output 3 全列挙)
- 既存 wave で source 仕様確定済 (= W40.8 / W41-pre / W42-pre / W51 / W44.5-pre)
- 局所新規実装 (= 1 CLI + 1 test file)
- W51 で「local 実装は本線完結」パターン確立
- AST 7 safety test で Codex Lane F 同等カバレッジ

詳細は results §11 で明記。

## §4 実施項目

1. baseline capture (= F282 / 3 DB / pytest 4229 / git status)
2. CLI 設計 (= phase 7 種 / verdict 4 段階 / output 3 format)
3. `scripts/jobs/run_production_v0_ops_summary.py` 新規実装
4. `tests/scripts/jobs/test_run_production_v0_ops_summary.py` 新規実装
5. pytest 全 PASS + 全体 collected 数報告
6. smoke 動作確認 (= pre-f282 / daily phase 実行)
7. 03_design に CLI v1.0 設計 doc 起票
8. F282 / DB 不干渉再確認
9. 02_todo に W44.6-pre plan / results 起票
10. HQ 1 ブロック報告 + 6 KPI

## §5 期待成果物 (= 5 file)

- `~/fire/scripts/jobs/run_production_v0_ops_summary.py` (= 新規、約 580 行)
- `~/fire/tests/scripts/jobs/test_run_production_v0_ops_summary.py` (= 新規、約 530 行 / 73 test)
- `~/fire-vault/03_design/Production_v0_ops_summary_cli_v1_0_2026-05-14.md` (= 新規)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE44_6_PRE_plan.md` (= 本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE44_6_PRE_results.md`

## §6 停止条件 (= W51 / W44.5-post と同じ)

DB write / DB sqlite 接続 / production・develop・staging DB / LINE 送信 /
token / secret / channel_token / .env / env 全体参照 / launchctl 実行 /
plist 配置 or 変更 / cron / crontab 変更 / VACUUM / VACUUM INTO /
F282 手動実行 / workflow 変更 / --no-verify / git push / sudo / rm -rf /
TODO Excel 更新 / 楽天証券操作 / 自動発注 / Computer Use が必要 → 即停止 + HQ 確認

## §7 完了条件

- Production v0 Ops Summary CLI 実装完了
- JSON / Markdown / text 出力可能
- F282 / DATA-R3 / F062 / readiness / checklist 統合
- missing/NG を phase 別に GO / NO-GO / HOLD へ整理
- readiness CLI v1.2 missing_markers 取り込み
- tests PASS + collected 数 (= 4229 → 4302) 報告
- F282 本番 plist mtime/size 不変 + 3 DB mtime/size 不変
- LINE / token / API / launchd / plist / cron / VACUUM / workflow /
  --no-verify / TODO Excel 全 0
- AST safety で sqlite3 / subprocess / linebot / VACUUM SQL / --no-verify /
  os.environ direct iteration 全 0
- output path guard 動作 (= data/ / .git/ / .fire_secrets/ / LaunchAgents/ refuse)
- fire-vault に W44.6-pre 報告作成
- HQ 1 ブロック報告 + 6 KPI
