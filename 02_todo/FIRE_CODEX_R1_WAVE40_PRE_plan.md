---
id: FIRE-CODEX-R1-Wave-40-pre
phase: 本番 v0 Launch / Phase B Foundation
priority: 高
status: 設計のみ (= 設計 doc / plist draft / no-write trial plan)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 38 (= v0 Launch Plan v1.0 確定、D-Day 2026-06-09)
  - Wave 39-temp (= F282 temporary smoke 完了、launchd 経路実証)
  - F282 試走 5/16 02:00 待機 (= 干渉禁止)
  - F286-DATA-R3 既存 runner skeleton (active 4 sub-runner)
related:
  - 03_design/FIRE_production_v0_launch_plan_2026-05-13.md
  - 03_design/F282_weekly_snapshot_launchd_2026-05-12.md
chapter: production-v0 / Phase B / DATA-R3
---

# FIRE-CODEX-R1 Wave 40-pre plan — F286-DATA-R3 Daily Refresh Launchd 設計

## 目的

F282 本番試走 (5/16 土曜 02:00) を待つ間に、Production v0 Phase B
の前倒し準備として、DATA-R3 daily refresh launchd 設計 / no-write
trial plan を作成する。**本 wave は設計のみ**、実 plist 配置 /
launchctl load / DB write / API call / LINE 送信 0。

## /goal 完了条件 21 項目

1. DATA-R3 daily refresh launchd 設計がある
2. F100/F101/F111/F119 の実行順序が整理されている
3. F282 との順序整合が整理されている
4. no-write / no-send 試走計画がある
5. freshness gate 連携が整理されている
6. log / monitoring / abort 条件がある
7. plist draft 案がある
8. Wave 41 以降の最短順がある
9. F282 本番試走に干渉していない
10. DB write 0
11. LINE 送信 0
12. token / secret / channel_token 参照 0
13. launchctl load/unload 0
14. plist 配置 0
15. 実 API call 0
16. L4 audit CRITICAL 0 / HIGH 0
17. 4,090 PASS 維持または必要範囲の regression 確認
18. docs/vault/log 更新済み
19. 6 KPI table 付き HQ 報告
20. なぜその lane 数を選んだか明記
21. 8 lane を使わなかった場合は理由明記

## 8 lane 構成 (= HQ 補足方針 8 lane 第一候補化に従う)

| lane | task_id | 担当 |
|---|---|---|
| L5 | 本線 | plan / results / 統合 / vault / log / HQ 報告 |
| L1a | DATA-R3 launchd architecture | Codex (= Label/時刻/環境/引数) |
| L1b | F282/freshness/morning advisory dependency | Codex (= 依存グラフ) |
| L2a | no-write trial test plan | Codex (= dry-run + DB mtime + LINE 0) |
| L2b | log/monitoring/abort test plan | Codex (= log path/abort 条件) |
| L3 | plist draft / command draft | Codex (= XML + command 例) |
| L4 | adversarial audit | Codex (= 8 観点 HIGH 0 目標) |
| L6 | regression / F282 interference review | Codex (= 4090 PASS 維持) |

採用理由: HQ 補足「8 lane 第一候補」明示 + 本 wave は分割可能な
独立観点が 7 個 (= architecture / dependency / test plan x2 / draft /
audit / regression) で 8 lane が自然分割。

## 想定成果物

- `~/fire-vault/03_design/F286_DATA_R3_daily_refresh_launchd_2026-05-13.md`
  (= 主成果、設計 doc 統合版)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_PRE_plan.md` (= 本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_PRE_results.md` (= 完了報告)
- `~/fire-vault/log.md` milestone 追記
- `/tmp/codex_wave40pre/results/l{1a,1b,2a,2b,3,4,6}.txt` (= 7 lane stdout)

## 禁止事項

- 実 plist 配置 (= ~/Library/LaunchAgents/jp.fire.daily-refresh.plist は作らない)
- launchctl load / unload
- DB write
- LINE 送信
- token / secret / channel_token 参照
- 実 API call (= J-Quants / TDnet / LINE)
- F282 手動実行
- VACUUM INTO
- 本番 F282 plist 変更
- workflow 変更 / --no-verify / TODO Excel 更新
