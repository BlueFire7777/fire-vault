---
id: FIRE-CODEX-R1-Wave-40.8
phase: 本番 v0 Launch / F282 試走後検証 CLI 実装
priority: 高
status: 実装 wave 完了
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40.7 (= v0 readiness CLI 実装パターン)
  - W39-temp 教訓 (= 3 点同時確認 auto memory)
related:
  - 03_design/F282_post_run_inspection_report_cli_2026-05-14.md
chapter: production-v0 / F282 post-run inspection CLI
---

# Wave 40.8 plan — F282 Post-Run Inspection Report CLI 実装

## 目的

5/16 02:00 F282 weekly snapshot 試走後、03:00 の実行後チェックを
**read-only / 機械的** に行う CLI を実装。W39-temp 教訓 (= 3 点同時確認)
を組み込み、PID=-/LastExit=0 単独判定誤りを再発防止。

## /goal 完了条件 39 項目

(= HQ 起票文書通り)

## 7 lane 構成 (= L5 本線 + Codex 6 lane、8 lane 不採用)

| lane | task_id | 担当 |
|---|---|---|
| L5 本線 | 統合 + CLI 実装 + pytest 実装 | 本線 |
| Lane A | CLI schema / report schema | Codex |
| Lane B | launchctl print parser / plist parser | Codex |
| Lane C | log / snapshot check | Codex |
| Lane D | baseline compare / DB stat safety | Codex |
| Lane E | pytest 20 test 設計 | Codex |
| Lane F | vault doc 12 章 / usage guide | Codex |

### 8 lane 不採用理由

1. F282 本番試走 5/16 直前のため干渉リスクと注意分散回避
2. 安全寄り運用、最大 6 lane (= HQ 補足明示)
3. 実装対象が単一 CLI + 単一 test 中心、6 lane で設計/parser/test/docs を
   十分分割可能
4. Wave 40-pre/post で 8/11 lane 実証済

### 第 2 陣 Codex 不採用理由

本 wave は実装 + test 中心。第 2 陣 4 安全 audit (= token / no-send /
duplicate / smoke) は Wave 40-post で網羅済、本 CLI が 3 点同時確認を
機械的代替する位置付け。

## 想定成果物

- `~/fire/scripts/jobs/run_f282_post_run_inspection_report.py` (= CLI、20 check)
- `~/fire/tests/scripts/jobs/test_run_f282_post_run_inspection_report.py` (= 21 test)
- `~/fire-vault/03_design/F282_post_run_inspection_report_cli_2026-05-14.md` (= 12 章)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_8_plan.md` (= 本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_8_results.md`
- `~/fire-vault/log.md` milestone
- `/tmp/codex_wave40_8/results/lane_{a-f}.txt`

## 禁止事項

CLI に組み込まない:
- subprocess launchctl call 0
- F282 runner import 0
- VACUUM SQL 発行 0
- production / develop / staging DB sqlite 接続 0
- token / secret / channel_token / env 全体読出 0
- LINE API 0
- plist 配置/変更 0

本 wave 全体:
- F282 plist mtime 変化 0
- 3 環境 DB mtime 変化 0
- workflow / --no-verify / TODO Excel 0
