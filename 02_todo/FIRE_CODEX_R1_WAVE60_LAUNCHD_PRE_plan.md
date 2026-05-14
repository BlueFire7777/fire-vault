---
id: FIRE-CODEX-R1-WAVE60-launchd-pre-plan
phase: 本番 v0 中核 / Wave 60-launchd-pre / Real Artifact Generation Path Preparation
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60-launchd-pre Plan — AFTER-R1 / F062 Real Artifact Generation Path Preparation v1.0

## §1 目的

D3 (= 2026-05-18 月) 以降の少額手動実弾パイロットで
`artifact_source = f062_preview` を狙うため、
**実 F062 preview / DATA-R3 freshness / AFTER-R1 MVP** の生成経路を
**read-only / no-send / no-write** で整える。

本 wave は **設計 + wrapper + manual command + temp/smoke 手順 + docs + tests**
のみ。本番 launchd 配置 / load は **しない**。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + F062 / DATA-R3 / AFTER-R1 runner 既存 args 確認 |
| 2 | F111 synth preview → F062 runner → AFTER-R1 MVP の 3 step smoke |
| 3 | D3 用 manual command sequence docs (~/fire-vault/04_daily/template_d3_real_artifact_prep.md) |
| 4 | F111 synth preview JSON 雛形 (= template_f111_synthetic_preview.json) |
| 5 | operation_plan §10.1 を D3 用 4 step に update |
| 6 | tests 追加: F062 runner 実 output → AFTER-R1 f062_preview 判定 |
| 7 | Codex 4 lane stdin smoke (= W60.5 安定 pattern) |
| 8 | Codex finding 修正 + 再 self-check |
| 9 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- LINE 送信 / DB write / token / API / launchctl / plist / cron / VACUUM 全 0
- 実発注 / 楽天 / iSPEED / Computer Use / 自動発注 全 0
- 楽天 / iSPEED 言及 は禁止項目として記述のみ
- F062 / DATA-R3 / AFTER-R1 本体は不触 (= args 確認のみ)
- F282 plist + 3 DB 完全不変
- ファイル write は ~/fire/scripts/jobs/tests/* + ~/fire-vault/* + /tmp/ のみ
- 本番 launchd 配置 / load は **本 wave 範囲外** (= Wave 41/45 へ)

## §4 完了条件

- F111 synth → F062 runner → AFTER-R1 MVP chain smoke 成功
- `artifact_source = f062_preview` 取得実証
- D3 用 manual command sequence docs 完成
- F111 synth preview 雛形完成
- operation_plan §10.1 D3 用 4 step 化
- tests 全 PASS、pytest collected の変化を報告
- Codex audit 実施 + finding 修正
- F282 plist + 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token / API call /
launchctl / plist / cron / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
