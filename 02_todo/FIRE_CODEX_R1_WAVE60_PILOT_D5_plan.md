---
id: FIRE-CODEX-R1-WAVE60-pilot-D5-plan
phase: 本番 v0 中核 / Wave 60-pilot-D5 / D5 f062_preview + f111_sample
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-20
pilot_day: D5
---

# Wave 60-pilot-D5 Plan — D5 f062_preview + f111_sample Manual Live Pilot v1.0

## §1 目的

2026-05-20 (水) D5 の少額手動実弾パイロット trade plan を作成。
W60-F111-pre 確立 chain (= F111 --sample → F062 → AFTER-R1) で
**artifact_source=f062_preview + f111_input_source=f111_sample** の成果物を
生成、**8 invariants hard check** で Pilot GO 判定 + W1 集約準備。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + W60-F111-pre 成果物確認 |
| 2 | F111 runner --sample 実行 (= /tmp/fire_d5_prep/) |
| 3 | DATA-R3 freshness 選択肢 A (= dry-run runner) |
| 4 | F062 LINE preview runner --dry-run --require-freshness-ok |
| 5 | AFTER-R1 MVP --mode mvp --task all → reports/after_r1/ |
| 6 | 8 invariants hard check (= W60-launchd-pre 7 + W60-F111-pre 追加 #8) |
| 7 | Pilot GO/HOLD/NO-GO 判定 |
| 8 | D5 trade plan + review template 作成 |
| 9 | W1 集約材料整理 |
| 10 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- 実発注 / 楽天 / iSPEED / Computer Use / 自動発注 0
- LINE 送信 / DB write / token / API / launchctl / plist / cron 0
- ファイル write は reports/after_r1/ + fire-vault/* + /tmp/ のみ
- FIRE 本体コード変更 0
- F282 plist + 3 DB 完全不変

## §4 完了条件

- D5 5 step chain 実行完了 (= F111 → DATA-R3 → F062 → AFTER-R1)
- artifact_source = f062_preview ✓
- f062_raw_kind = f062_actual_dict ✓
- f111_input_source = f111_sample ✓ (= W60-F111-pre 達成)
- 8 invariants hard check 全 PASS
- Pilot GO/HOLD/NO-GO 明確
- D5 trade plan 完成 (= minor caveat + サンプル ticker 構造的 skip 明記)
- D5 review template 完成
- W1 集約材料整理
- F282 / 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token / API /
launchctl / plist / cron / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
