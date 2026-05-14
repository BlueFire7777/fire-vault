---
id: FIRE-CODEX-R1-WAVE44.5-post-plan
phase: 本番 v0 Launch / Cutover Runbook v1.0 final Adversarial Audit
priority: 高
status: plan (= Wave 44.5-post 着手前計画)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 44.5-post Plan — Production v0 Cutover Runbook Adversarial Audit v1.0

## §1 目的

W44.5-pre で確定した Production v0 cutover / token / rollback runbook を、
**敵対的観点で監査** し、D-Day 2026-06-09 火曜の本番 v0 開始時に事故になり得る
矛盾 / 漏れ / 曖昧表現 / 危険手順を潰す。

## §2 方針

- 監査 + docs 微修正 + HQ 報告のみ
- FIRE 本体コード変更なし (= CLI / runner 実装変更が必要な場合は次 Wave 候補)
- LINE token / .env / env 全体 / DB / launchctl / plist / cron 触らない

## §3 構成

| lane | 内容 |
|---|---|
| L5 本線 | baseline + 8 lane orchestration + triage + docs fix + 報告 |
| Codex Lane A | D-Day / date / phase timeline 監査 |
| Codex Lane B | HQ marker 順序 / 承認条件 監査 |
| Codex Lane C | LINE token 非参照 / send_guard / no-send → send 切替 監査 |
| Codex Lane D | rollback / token rotate / DB restore / recovery 監査 |
| Codex Lane E | failure matrix 18 種網羅性 監査 |
| Codex Lane F | F282 5/16 drill 連携 監査 (W40.8 / W43-pre / W44-pre) |
| Codex Lane G | readiness CLI v1.1 と runbook 整合 監査 |
| Codex Lane H | docs 表現 / 危険コマンド例 / 曖昧表現 監査 |

期待: 8 lane 全 READY または HIGH 検出 → 本線で docs 修正 → CRITICAL 0 / HIGH 0 で
完了。修正不能な code-level 課題は次 Wave 候補に記録。

## §4 実施項目 (= 8 観点 + triage + fix + 報告)

1. baseline capture (= F282 plist mtime/size、3 DB mtime/size、pytest collect 4202、
   FIRE code git status)
2. 8 lane parallel adversarial audit を codex:codex-rescue 並列起動
3. lane 結果集約 + CRITICAL / HIGH / MEDIUM / LOW triage
4. HIGH / CRITICAL findings の docs minimal fix (= 本 wave で対応可な範囲)
5. CRITICAL の CLI コード問題は次 Wave 候補に記録 (= 修正せず)
6. F282 / DB / pytest 不干渉再確認
7. vault に plan / results 起票
8. HQ 1 ブロック報告

## §5 期待成果物

- `~/fire-vault/03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md`
  (= v1.0 final → audited v1.0 final、minimal fix 9 件)
- `~/fire-vault/04_daily/template_v0_d_day_check.md`
  (= dual-run 期間文言 minimal fix 1 件)
- `~/fire-vault/03_design/FIRE_production_v0_launch_plan_2026-05-13.md`
  (= marker 順序 7 段固定 + 想定→確定、2 件 minimal fix)
- `~/fire-vault/03_design/F062_no_send_trial_checklist_and_v0_readiness_2026-05-14.md`
  (= 想定→確定 + GO 条件 marker 6/7 明示、2 件 minimal fix)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE44_5_POST_plan.md` (= 本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE44_5_POST_results.md`

## §6 停止条件 (= W44.5-pre と同じ)

LINE token / secret / channel_token の値参照 / .env or env 全体参照 /
LINE 送信 / DB write / production・develop・staging DB 接続 / launchctl 実行 /
plist 配置 or 変更 / cron / crontab 変更 / VACUUM / VACUUM INTO /
F282 手動実行 / workflow 変更 / --no-verify / git push / sudo / rm -rf /
TODO Excel 更新 / 楽天証券操作 / 自動発注 / Computer Use が必要 → 即停止 + HQ 確認

## §7 完了条件

- 8 観点 adversarial audit 完了
- CRITICAL / HIGH が 0、または CRITICAL / HIGH があれば修正済 or HQ 判断待ち明記
- D-Day 2026-06-09 火曜 / final strict 2026-06-08 月曜 整合確認
- HQ marker 7 段整合確認
- token 非参照 / no-send / send_guard / freshness gate 整合確認
- rollback / failure matrix 整合確認
- F282 5/16 drill 整合確認
- F282 本番 plist mtime/size/next run 不変確認
- production/develop/staging DB mtime/size 不要変更なし確認
- LINE / token / API / launchd / plist / cron / VACUUM / workflow / --no-verify /
  TODO Excel 全 0 確認
- pytest collected 4202 維持
- vault に Wave 44.5-post 報告作成
- HQ 1 ブロック報告作成
- 6 KPI 報告
