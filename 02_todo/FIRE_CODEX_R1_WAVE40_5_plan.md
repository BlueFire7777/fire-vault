---
id: FIRE-CODEX-R1-Wave-40.5
phase: 本番 v0 Launch / Phase D-E readiness
priority: 中 (= 軽量 wave、F282 試走前)
status: 設計のみ
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40-pre + 40-post (= Phase B + C foundation 完成)
  - HQ Wave 40.5 起票指示 (= 5/14-5/15 軽量 wave 推奨)
  - F282 試走 5/16 02:00 待機
related:
  - 03_design/FIRE_production_v0_launch_plan_2026-05-13.md
  - 03_design/F062_morning_advisory_launchd_2026-05-13.md
chapter: production-v0 / Phase D-E readiness
---

# FIRE-CODEX-R1 Wave 40.5 plan — F062 No-Send Trial Checklist + v0 Readiness

## 目的

F282 本番試走 5/16 を控え、実行系を増やさずに v0 Phase C/D/E 直結の
設計・チェックリストを補強する。本線最推奨案 1 (= no-send trial
checklist + v0 残課題整理) を HQ 承認に基づき実施。

## /goal 完了条件 25 項目

1-7  F062 no-send checklist (pre/run/post/abort) + DATA-R3 依存 + Phase D/E
     GO/NO-GO + HQ marker 順序 + 引き継ぎ
8-20 安全要件 (= F282 干渉 0 / plist/launchctl/DB write/LINE/token/env/API/
     F282 手動/VACUUM/workflow/--no-verify/TODO Excel 全 0)
21   pytest 4090 維持
22   vault doc 更新
23   log.md milestone
24   HQ 1 ブロック完了報告
25   6 KPI 報告

## 3 lane 構成 (= 軽量 wave、第一候補 8 lane 不採用)

| lane | task_id | 担当 |
|---|---|---|
| L5 (本線) | 統合 + audit + vault | plan / 設計 doc / vault / log / HQ 報告 |
| Lane A | no-send trial checklist draft | Codex |
| Lane B | v0 Phase D/E dependency + marker review | Codex |

### lane 数選定理由 (= HQ 報告先取り)

**8 lane を採用しない**:
- HQ 補足明示「F282 本番試走直前、実行系を増やしすぎない」「軽量 wave、
  本線 + 最大 2 Codex lane 推奨」
- 成果物が単一 doc (= 12 章 1 file)、Integrator 負荷を低く維持
- 干渉リスク最小化 (= F282 5/16 03:00 試走への注意分散回避)
- Wave 40-pre + 40-post で 8/11 lane 構成は実証済、本 wave で再実証不要

### 第 2 陣 Codex を採用しない

- 軽量 wave で第 2 陣の 4 安全 audit 観点は適用範囲外
  (= 本 wave で DB / token / LINE 経路に触らないため audit 観点が抽象的)
- Wave 40-post で同観点を網羅済、再 audit 不要

## 想定成果物

- `~/fire-vault/03_design/F062_no_send_trial_checklist_and_v0_readiness_2026-05-14.md` (= 12 章、主成果)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_5_plan.md` (= 本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_5_results.md`
- `~/fire-vault/log.md` milestone 追記
- `/tmp/codex_wave40_5/results/lane_{a,b}.txt` (= 2 lane stdout)

## 禁止事項

実 file 変更 (= 設計 doc + plan/results + log.md 以外 全 0) / plist 配置 /
launchctl / cron / launchd / crontab / DB write / LINE / token / channel_token
/ secret / env 全体読出 / 実 API / F101 staging probe / F282 手動実行 /
VACUUM INTO / 本番 F282 plist 変更 / workflow / --no-verify / TODO Excel /
楽天 / 自動発注 / Computer Use
