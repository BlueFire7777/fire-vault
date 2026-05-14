---
id: FIRE-CODEX-R1-WAVE52.5-pre-plan
phase: 本番 v0 Launch / Wave 52.5-pre / consistency cleanup
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 52.5-pre Plan — Production v0 Final Consistency Cleanup / Timeline & Naming Alignment

## §1 目的

2026-05-16 03:00 F282 drill 前に Production v0 関連 docs / CLI 名 / timeline /
marker 名 / report 名のズレを消す。新規機能を増やさず、5/16 drill /
5/19 GO/NO-GO / 6/8 final strict / 6/9 D-Day で人間が迷わない状態にする。

## §2 方針

- docs / templates / naming alignment 中心
- FIRE 本体コード変更 0
- CLI 修正が必要な不整合は次 Wave 候補に記録
- TODO Excel 更新なし

## §3 構成

| lane | 内容 |
|---|---|
| L5 本線 | baseline + 6 観点 sweep + minimal docs edit + 1-page drill quick reference + 報告 |

Codex lane 投入: **0/8 を計画**。理由:
- prescriptive cleanup wave (= user spec 6 確認・修正観点を列挙)
- grep + 最小編集で完結、Codex 並列の追加価値小
- W44.5-pre / W51 / W44.6-pre / W52-pre precedent (= 0/8 accepted)

## §4 実施項目 (= 6 観点 sweep)

1. baseline (= F282 / 3 DB / pytest 4423 / git status)
2. timeline 整合 sweep: 6/9 月曜 / 想定 / 期間ズレ
3. CLI 名・version 整合 sweep: readiness v1.2 / Ops Summary v1.0.1 / wrapper v1.0.1
4. marker 名整合: HQ_APPROVE_SEND_MARKER 残存 / 7 段固定
5. 5/16 drill 1-page quick reference 新規作成
6. dangerous command 表記確認
7. F282 / DB / pytest 不干渉再確認
8. vault に W52.5-pre plan / results / HQ 1 ブロック + 6 KPI

## §5 期待成果物

修正:
- `~/fire-vault/04_daily/template_v0_d_day_check.md` (= readiness CLI v1.1 → v1.2、5 箇所)
- `~/fire-vault/04_daily/template_f282_post_run.md` (= readiness CLI v1.1 → v1.2、2 箇所)
- `~/fire-vault/03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14.md` (= §12 残課題 HQ_APPROVE_SEND_MARKER 解消注記)
- `~/fire-vault/03_design/F062_wave42_pre_no_send_runner_enhancement_2026-05-14.md` (= §5 / §11 W52-pre/post 統合注記)
- `~/fire-vault/03_design/F062_no_send_wrapper_preview_cli_v1_0_2026-05-14.md` (= JSON example cli_version "1.0" → "1.0.1")
- `~/fire-vault/03_design/Production_v0_ops_summary_cli_v1_0_2026-05-14.md` (= JSON example cli_version "1.0" → "1.0.1")

新規:
- `~/fire-vault/04_daily/template_f282_drill_quick_reference.md` (= 1-page drill quick reference)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE52_5_PRE_plan.md` / `_results.md`

## §6 停止条件 (= W52-post と同じ)

DB write / LINE 送信 / token / .env / launchctl / plist / cron / VACUUM /
F282 手動実行 / workflow / --no-verify / git push / sudo / TODO Excel /
楽天 / Computer Use が必要 → 即停止 + HQ 確認

## §7 完了条件

- v0 関連 docs / CLI 名 / timeline / marker 名の主要ズレが解消
- 5/16 drill 1-page 手順完成
- D-Day = 2026-06-09 火 / final strict = 2026-06-08 月 で統一
- readiness CLI v1.2 / Ops Summary v1.0.1 / wrapper v1.0.1 整合
- HQ_APPROVE_SEND_MARKER の使用箇所が「廃止注記」のみ
- pytest collected 4423 維持 (= 不要変化なし)
- F282 / 3 DB / launchd / cron / VACUUM / workflow / TODO Excel 全 0
- vault に W52.5-pre 報告 + HQ 1 ブロック + 6 KPI
