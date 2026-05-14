---
id: FIRE-CODEX-R1-WAVE60-F111-pre-plan
phase: 本番 v0 中核 / Wave 60-F111-pre / F111 Real Artifact Path
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60-F111-pre Plan — F111 Morning Batch Real Artifact Design / Implementation Prep v1.0

## §1 目的

D3/D4 で残った最大課題 = **F111 synthetic_sample 依存**を解消するため、
F111 runner output → F062 LINE preview runner → AFTER-R1 MVP の chain を整備し、
**`f111_input_source` 識別 field** を AFTER-R1 MVP に追加。

D5 以降 `f111_input_source = f111_sample` (= F111 runner `--sample` 由来) で
trade plan を作成可能にする。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + F111 existing runner 調査 |
| 2 | F111 ↔ F062 input/output contract 整合監査 |
| 3 | F111 sample → F062 → AFTER-R1 chain smoke (= W60-F111-pre 主要実証) |
| 4 | AFTER-R1 MVP に f111_input_source 識別追加 (= 4 output 反映) |
| 5 | F111 real artifact contract docs (~/fire-vault/03_design/) |
| 6 | D5 用 manual command sequence docs (~/fire-vault/04_daily/) |
| 7 | tests 追加 (= 12+ regression) |
| 8 | Codex 4 lane stdin audit |
| 9 | vault plan + results + HQ + 6 KPI

## §3 安全境界

- 実発注 / 楽天 / iSPEED / Computer Use 0
- LINE 送信 / DB write / token / API / launchctl / plist / cron / VACUUM 0
- ファイル write は scripts/jobs/* + tests/ + fire-vault/* + /tmp/ のみ
- F111 / F062 本体不触 (= 既存仕様確認のみ)
- F282 plist + 3 DB 完全不変

## §4 完了条件

- F111 ↔ F062 ↔ AFTER-R1 chain 実証 (= artifact_source=f062_preview /
  f111_input_source=f111_sample)
- AFTER-R1 MVP に f111_input_source taxonomy + 推定 logic + 4 output 反映
- CLI に `--f111-input-source` 追加 (= manual_seed/f111_sample/f111_real_batch/
  unknown/auto)
- F111 real artifact contract docs 完成
- D5 用 manual command sequence docs 完成
- tests 12+ 件追加、全 PASS
- pytest collected 増分報告
- Codex 4 lane audit 実施 (= 結果は完全 reply / self-audit いずれか)
- DB write / LINE / token / API / launchctl / plist / cron 0
- F282 plist + 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token / API /
launchctl / plist / cron / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
