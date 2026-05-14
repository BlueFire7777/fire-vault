---
id: FIRE-CODEX-R1-WAVE52-pre-plan
phase: 本番 v0 Launch / Wave 52-pre / no-send preparation
priority: 高
status: plan (= Wave 52-pre 着手前計画)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 52-pre Plan — HQ Marker / Wrapper / DB Labels No-Send Preparation v1.0

## §1 目的

Production v0 D-Day に向けて、F062 morning advisory の no-send wrapper、
send marker semantics、DB record-decisions labels の準備を行う。
今回は本番送信 / token 参照 / DB write を **一切行わない**。

## §2 方針

- no-send / no-token / no-DB-write の準備 wave
- 実 LINE token 投入 / 実 LINE 送信 / DB write / launchd 配置 / 変更 0
- HQ_APPROVE_LINE_TOKEN_PRODUCTION / HQ_APPROVE_PRODUCTION_V0_LAUNCH 発行 0
- send marker は **本物作成しない、tmpdir/fixture 上の mock のみ可**

## §3 構成

| lane | 内容 |
|---|---|
| L5 本線 | baseline + CLI 新規実装 + 72 test + runbook minimal edit + 設計 doc + 報告 |

Codex lane 投入: **0/8 を計画**。理由 (= results §7 / 設計 doc §7 で詳述):
- user spec が完全 prescriptive
- 既存 wave (= W44.5-pre / W51 / W44.6-pre/post) で marker 名 / source 仕様確定済
- 局所新規実装 (= 1 CLI + 1 test)
- AST 10 safety test で Codex Lane F 機能等価
- W44.6-pre / W51 precedent 確立

## §4 実施項目

1. baseline capture (= F282 / 3 DB / pytest 4332 / git status)
2. F062 runner CLI 引数仕様確認 (= preview / send 各 runner の flags)
3. CLI 新規実装: `scripts/jobs/run_f062_no_send_wrapper_preview.py`
   - no-send wrapper command builder (= --dry-run + freshness gate)
   - production send wrapper stub builder (= W52 本番、本 wave 不実行)
   - marker fixture validation (= tmpdir 限定 + forbidden alias refuse)
   - DB labels preview JSON generator
   - text / JSON / Markdown 出力
   - output path guard
4. test 追加: 12 class / 72 test
5. cutover runbook §6.2 / §7.4 / §10.2 / §14 / §16 で `HQ_APPROVE_SEND_MARKER`
   曖昧名廃止 → marker 7 (`HQ_APPROVE_PRODUCTION_V0_LAUNCH`) 統合 (= minimal edit 4 箇所)
6. 03_design に CLI v1.0 設計 doc 起票
7. F282 / DB / pytest 不干渉再確認
8. 02_todo W52-pre plan / results 起票
9. HQ 1 ブロック + 6 KPI

## §5 期待成果物 (= 5 file)

- `~/fire/scripts/jobs/run_f062_no_send_wrapper_preview.py` (= 新規、約 750 行)
- `~/fire/tests/scripts/jobs/test_run_f062_no_send_wrapper_preview.py` (= 新規、約 750 行 / 72 test)
- `~/fire-vault/03_design/F062_no_send_wrapper_preview_cli_v1_0_2026-05-14.md` (= 新規)
- `~/fire-vault/03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md` (= minimal edit 4 箇所 / 7 references を marker 7 統合)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE52_PRE_plan.md` / `_results.md` (= 新規)

## §6 停止条件 / 完了条件 (= W44.6-post と同じ + 追加)

追加:
- production send 実行
- 本物の HQ marker 作成
- LINE token / .env / env 全体参照
- DB write / DB sqlite 接続

完了条件:
- no-send wrapper / marker / DB labels 準備完了
- production send 未実行
- token / DB / launchctl 全 0
- mock marker tmpdir 限定
- DB labels preview JSON 定義済み
- readiness CLI v1.2 / Ops Summary v1.0.1 / cutover runbook 整合
- tests PASS + collected 数 (= 4332 → 4404) 報告
- F282 / 3 DB / launchd / cron / VACUUM / workflow / TODO Excel 全 0
- vault に W52-pre 報告 + HQ 1 ブロック + 6 KPI
