---
id: FIRE-CODEX-R1-Wave-40.7
phase: 本番 v0 Launch / Phase D-E readiness CLI 実装
priority: 高
status: 実装 wave 完了
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40.5 (= 56 項目 checklist)
  - Wave 40.6 (= cutover/rollback/token runbook、D-Day 6/9 火曜確定)
related:
  - 03_design/Production_v0_readiness_check_cli_2026-05-14.md
chapter: production-v0 / readiness CLI
---

# Wave 40.7 plan — Production v0 Readiness Check CLI 実装

## 目的

Wave 41/45/52/53 の GO/NO-GO 判定を支援する **read-only CLI** を実装。
56 項目手動 checklist (= W40.5) を機械的検証可能化。F282 試走干渉 0、
実行系変更 0、token 値 0、DB write 0。

## /goal 完了条件 34 項目

(= HQ 起票文書通り、実装 + test + safety + 報告)

## 7 lane 構成 (= L5 本線 + Codex 6 lane、8 lane 不採用)

| lane | task_id | 担当 |
|---|---|---|
| L5 本線 | 統合 + CLI 実装 + pytest 実装 | 本線 |
| Lane A | CLI schema / phase design | Codex |
| Lane B | F282 / launchd / plist checks 設計 | Codex |
| Lane C | DB / token safety checks 設計 | Codex |
| Lane D | D-Day / HQ marker / schedule checks 設計 | Codex |
| Lane E | pytest 15 test 設計 | Codex |
| Lane F | vault doc 10 章 outline | Codex |

### 8 lane 不採用理由

1. F282 本番試走 5/16 直前のため干渉リスクと注意分散回避
2. 実装対象が単一 CLI + 単一 test 中心、6 lane で設計/実装/test/docs/audit
   十分分割可能
3. 過剰分割すると Integrator 負荷増
4. Wave 40-pre/post で 8/11 lane 構成実証済、本 wave で再実証不要

### 第 2 陣 Codex 不採用理由

本 wave は実装 + test 中心、第 2 陣 4 安全 audit (= token / no-send /
duplicate / smoke) は Wave 40-post で網羅済、再 audit 不要。L7a-d 観点を
本 CLI が機械化代替する位置付け。

## 想定成果物

- `~/fire/scripts/jobs/run_production_v0_readiness_check.py` (= CLI、19 check)
- `~/fire/tests/scripts/jobs/test_run_production_v0_readiness_check.py` (= 15 test)
- `~/fire-vault/03_design/Production_v0_readiness_check_cli_2026-05-14.md` (= 10 章)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_7_plan.md` (= 本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_7_results.md`
- `~/fire-vault/log.md` milestone
- `/tmp/codex_wave40_7/results/lane_{a,b,c,d,e,f}.txt`

## 禁止事項

CLI 実装に組み込まない:
- subprocess での launchctl call 0
- token / secret / channel_token 値 read 0
- env 全体 dump 0
- sqlite write 0 (= read-only URI も使わず stat のみで設計)
- LINE API 0
- plist 作成 / 変更 0

本 wave 全体:
- F282 plist mtime 変化 0
- 3 環境 DB mtime 変化 0
- workflow / --no-verify / TODO Excel 0
- 楽天 / 自動発注 / Computer Use 0
