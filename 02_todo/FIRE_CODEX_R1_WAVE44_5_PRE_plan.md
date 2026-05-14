---
id: FIRE-CODEX-R1-WAVE44.5-pre-plan
phase: 本番 v0 Launch / Cutover / Token / Rollback Runbook v1.0 final
priority: 高
status: plan (= Wave 44.5-pre 着手前計画)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 44.5-pre Plan — Production v0 Cutover / LINE Token / Rollback Runbook Finalization v1.0

## §1 目的

2026-06-09 火 Production v0 D-Day に向け、D-Day 切替 / LINE token 投入 /
no-send → production send 切替 / rollback / 失敗時復旧 / GO/NO-GO 判断を
**安全に実行できる runbook / checklist として最終化** する。

W40.6 初版を、W40.7/40.8/41-pre/42-pre/43-pre/44-pre の foundation を
反映した **v1.0 final** に確定。

## §2 方針

設計 / runbook / checklist 整備のみ。実 LINE token 参照 / 実 LINE 送信 /
DB write / plist 配置 / launchctl load / cron 変更 / VACUUM / git push /
F282 手動実行 / 楽天証券操作 / 自動発注 / Computer Use 0。

Production v0 launch 実行は **後日**、`HQ_APPROVE_LINE_TOKEN_PRODUCTION`
+ `HQ_APPROVE_PRODUCTION_V0_LAUNCH` が揃った場合のみ。

## §3 構成

| lane | 内容 |
|---|---|
| L5 本線 | runbook v1.0 final 化 + 04_daily checklist + drift fix + 不干渉確認 + 報告 |
| Codex lane | 本 wave は本線完結、設計 / docs 中心のため Codex 8 lane 不投入 |

### Codex lane 不投入理由 (= HQ 報告に明記)

- 成果物が runbook + checklist のみ、4 file 編集で本線完結
- W40.6 (= 4 lane) / W40.7 (= 8 lane) / W40.8 (= 8 lane) /
  W41-pre (= 8 lane) / W42-pre (= 8 lane) / W43-pre (= 8 lane) /
  W44-pre (= 8 lane) で十分 audit 済 (= 累積 50+ lane)
- 本 wave は前 wave の集約整理であり、新規論点 0
- 8 lane 投入で Integrator 負荷が見合わない (= 6 KPI 観点で減点)
- 本線で SEND/NO-SEND/HOLD/ROLLBACK matrix を一気書きする方が整合性が高い

## §4 実施項目 (= 9 step)

1. D-Day 日付を 2026-06-09 (火) に確定、6/8 (月) を final strict check 日と整理
2. HQ marker 順序 7 段固定 (= 既存 W40.5 / W40.6 §4 と一致、本 wave で lock)
3. LINE token production setup runbook を token 非参照式に再整理
4. no-send → production send 切替手順を W42-pre freshness consumer 反映
5. rollback 手順 (= 緊急 unload + token rotate + DB restore + 復帰)
6. anomaly / failure handling matrix (= SEND/NO-SEND/HOLD/ROLLBACK 18 種)
7. Production v0 GO/NO-GO checklist (= 6/8 final strict + 6/9 D-Day morning)
8. tests/CLI 変更 0 (= 必要時は HQ 確認のみ)
9. vault に W44.5-pre plan / results / HQ 1 ブロック報告

## §5 期待成果物 (= 4 file)

1. `~/fire-vault/03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md`
   (= v1.0 → v1.0 final、約 40 KB)
2. `~/fire-vault/04_daily/template_v0_d_day_check.md` (= 新規、約 9 KB)
3. `~/fire-vault/03_design/FIRE_production_v0_launch_plan_2026-05-13.md`
   (= 1 行 minimal edit: D-Day 月曜 → 火曜)
4. `~/fire-vault/03_design/F062_no_send_trial_checklist_and_v0_readiness_2026-05-14.md`
   (= 1 行 minimal edit: D-Day 月曜 → 火曜)

加えて vault commit:
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE44_5_PRE_plan.md` (= 本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE44_5_PRE_results.md`

## §6 停止条件

以下を検知したら即停止して HQ 確認:
- LINE token / secret / channel_token の値参照が必要
- .env または env 全体参照が必要
- LINE 送信 / DB write / production/develop/staging DB 接続 / launchctl /
  plist / cron / VACUUM / F282 手動実行 / workflow / --no-verify /
  git push / sudo / rm -rf / TODO Excel / 楽天 / 自動発注 / Computer Use が必要

## §7 完了条件

- Production v0 cutover / token / rollback runbook v1.0 final 完成
- D-Day = 2026-06-09 (火)、final strict = 2026-06-08 (月) として確定
- HQ marker 順序 7 段固定
- LINE token production setup 手順 (= token 非参照) 確定
- no-send → production send 切替手順確定
- rollback 手順確定
- failure 18 種 × 4 種別 判断 matrix 確定
- GO/NO-GO checklist (= 6/8 + 6/9 二日分) 完成
- F282 本番 plist mtime/size/next run 不変確認
- production/develop/staging DB mtime/size に不要変更なし確認
- LINE/token/API/launchd/plist/cron/workflow/--no-verify/TODO Excel 全 0 確認
- fire-vault に W44.5-pre 報告作成
- HQ 1 ブロック報告作成
- 6 KPI 報告
