---
id: FIRE-CODEX-R1-WAVE39-PRE-plan
phase: ガバナンス / Wave 39-pre / F282 temporary launchd smoke 設計
priority: 高 (= v0 前倒し準備)
status: 進行中 ☆ /goal モード / 8 lane (本線 1 + Codex 7) / 設計のみ
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 38 (= 完了、Production v0 Launch Plan v1.0)
  - HQ Wave 39-pre 起票承認 (= 2026-05-13)
chapter: ガバナンス / R-01-08 / F282 temporary smoke 設計
---

# Wave 39-pre: F282 temporary launchd smoke 設計

## ★ 目的

5/19 F282 GO/NO-GO 判定を待たずに **launchd 経由実行リスクを前倒し検証**
し、Production v0 準備 (= DATA-R3 / F062 / no-send 試走設計) を止めない。

本 wave は **設計のみ**。実 plist 配置 / launchctl / VACUUM INTO / DB write /
LINE / token は 0。**本番 plist 不干渉、5/16 本番試走に影響しない**。

## lane 数選定理由 (= HQ 必須追加)

**8 lane 第一候補採用** (= HQ 補足方針)。本 wave は 7 sub に自然分割可能:
- L1a A/B/C 案比較 / L1b temporary smoke 設計 /
- L2a verification checklist / L2b abort / cleanup /
- L3 plist draft / command draft / L4 audit / L6 regression

**8 lane 不採用ケース**: 該当なし。本線短縮率 67% で目標達成想定。

## baseline (= 本 wave 開始時、不変確認)

| 項目 | 状態 |
|---|---|
| 本番 plist jp.fire.weekly-snapshot.plist | ~/Library/LaunchAgents/ 配置済、mtime 5/12 22:46 |
| 本番 launchctl LastExitStatus | 0 維持 |
| 本番 次 run | 2026-05-16 土曜 02:00 JST 待機 |
| /data/fire.db (production) | 371,081,216 / 5/12 16:17:24 |
| /data/fire.staging.db | 4,804,063,232 / 5/12 18:45:22 |
| /data/fire.develop.db | 371,081,216 / 5/12 16:11:43 |
| /data/snapshot/ (= W30 retention) | 2 file (5/19 まで保持) |
| 現在 | 2026-05-13 水曜 01:03 JST |

## Wave 39-pre sub-task (= 8 lane = 本線 1 + Codex 7)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W39p-1| L5  | 本線  | plan + baseline + 7 Codex prompt              |
| W39p-2| L1a | Codex | A/B/C 案比較 + 推奨理由                       |
| W39p-3| L1b | Codex | temporary smoke 詳細設計                       |
| W39p-4| L2a | Codex | verification checklist / test plan            |
| W39p-5| L2b | Codex | abort / cleanup plan                          |
| W39p-6| L3  | Codex | plist XML draft + smoke command draft         |
| W39p-7| L4  | Codex | adversarial audit (= 8 観点)                  |
| W39p-8| L6  | Codex | regression / F282 本番 plist 干渉確認         |
| W39p-9| 本線 | 本線  | design doc 統合 + 18 条件 + 6 KPI + 報告      |

## file lock 表

### 既存 / read-only / 不変保証

| file | 状態 | 区分 |
|---|---|---|
| ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist | NEW(W33) | **read-only 維持** |
| ~/fire/docs/launchd/jp.fire.weekly-snapshot.plist | NEW(W31) | read-only |
| 3 環境 DB | - | mtime + size 完全 unchanged |
| /data/snapshot/ W30 2 file | - | 完全保持 |
| scripts/seed_pattern_layer1.py + historical_indicators.py | M | forbidden |

### 本 wave 書込み (= vault のみ)

| file | 状態 | owner | 区分 |
|---|---|---|---|
| 02_todo/FIRE_CODEX_R1_WAVE39_PRE_plan.md | NEW | 本線 (L5) | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE39_PRE_results.md | NEW | 本線 | allowed |
| 03_design/F282_temporary_launchd_smoke_2026-05-13.md | NEW | 本線 | allowed |
| log.md | MOD | 本線 | allowed |

fire develop: commit 0 (= 設計のみ)。

## 関連リンク

- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 本番 launchd 設計]]
- [[../03_design/F282_weekly_snapshot_trial_monitoring_2026-05-12|F282 試走監視 (W34)]]
- [[../03_design/FIRE_production_v0_launch_plan_2026-05-13|v0 Launch Plan (W38)]]
