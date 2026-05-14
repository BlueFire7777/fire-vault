---
id: FIRE-CODEX-R1-WAVE39-TEMP-plan
phase: ガバナンス / Wave 39-temp / F282 temporary smoke 実行
priority: 高
status: 進行中 ☆ /goal モード / HQ 3 段 marker / 本番 plist 不干渉
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 39-pre (= 完了、temporary smoke 設計 v1.0)
  - HQ Wave 39-temp 起票承認 + 3 段 marker (= 2026-05-13)
chapter: ガバナンス / R-01-08 / F282 temporary smoke 実行
---

# Wave 39-temp: F282 temporary launchd smoke 実行

W39-pre 設計に従い、temporary plist (= jp.fire.weekly-snapshot-smoke) を
1 回だけ launchctl load → 起動確認 → unload + cleanup。

**本番 plist (= jp.fire.weekly-snapshot) は完全不干渉**、5/16 02:00 本番
試走と 5/19 GO/NO-GO 判定は残す。

## 必須 #1-#3 実行前 baseline (= 取得済)

### 本番 plist mtime (= 不変必須)

- ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist
- mtime=1778593597 (= 2026-05-12 22:46:37)
- size=1772

### 本番 launchctl 状態 (= 不変必須)

- Label: jp.fire.weekly-snapshot
- LastExitStatus: 0
- OnDemand: true
- state: not running
- 次 run: 2026-05-16 土曜 02:00 JST 待機

### 3 環境 DB mtime + size (= 不変必須)

| file | mtime | size |
|---|---|---|
| /data/fire.db | 1778570244 (5/12 16:17:24) | 371,081,216 |
| /data/fire.staging.db | 1778579122 (5/12 18:45:22) | 4,804,063,232 |
| /data/fire.develop.db | 1778569903 (5/12 16:11:43) | 371,081,216 |

### W30 snapshot (= 不変必須)

| file | mtime | size |
|---|---|---|
| /data/snapshot/fire.staging.db | 1778589839 (5/12 21:43:59) | 353,128,448 |
| /data/snapshot/fire.develop.db | 1778589840 (5/12 21:44:00) | 353,128,448 |

### temporary smoke 関連 path (= 不在確認)

- ~/Library/LaunchAgents/jp.fire.weekly-snapshot-smoke.plist: 不在 ✓
- /data/snapshot-smoke/: 不在 ✓
- /logs/cron/weekly-snapshot-smoke.*: 不在 ✓

### 現在: 2026-05-13 水曜 01:14:30 JST

## 起動時刻設定

- 2026-05-13 水曜 **01:18 JST** (= 約 3.5 分後)
- 平日近未来、本番土曜 02:00 と完全別時間帯

## Wave 39-temp sub-task (= 8 lane = 本線 1 + Codex 6)

| sub | lane | owner | task |
|---|---|---|---|
| W39t-1 | L5 | 本線 | plan + baseline + 6 Codex prompt |
| W39t-2 | L1a | Codex | 実行手順詳細 |
| W39t-3 | L2 | Codex | post-smoke verification 手順 |
| W39t-4 | L3 | 本線 | temporary plist 作成 + 配置 + load + 待機 |
| W39t-5 | L4 | Codex | adversarial audit |
| W39t-6 | L6 | Codex | regression / 本番 plist 不干渉確認 |
| W39t-7 | 本線 | 本線 | post 検証 + unload + cleanup + 報告 |

## lane 選定理由

8 lane 第一候補 (= HQ 補足方針通り、設計レビュー / audit 並走可)。
ただし実行系 (= L3) は本線専属、Codex 6 並走で task 量「中」。
**8 lane 不採用**: L2a/L2b を統合 L2 にしたため 7 lane (= 本線 1 + Codex 6)。
これは sub-task 数 5 件で適切な分割で、Codex を無駄に 7 lane 立てる必要なし。

## /goal 完了条件 19/19 計画

実行前 baseline 取得 #1-#3、temporary plist 配置 #4-#6、launchctl load #7、
log 確認 #8、snapshot output #9、integrity #10、既存 DB unchanged #11、
本番 plist unchanged #12、本番 launchctl state 維持 #13、unload #14、
cleanup #15、LINE 0 #16、token 0 #17、API 0 #18、6 KPI 報告 #19

## 関連リンク

- [[../03_design/F282_temporary_launchd_smoke_2026-05-13|F282 temp smoke 設計 (W39-pre)]]
- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 本番 launchd 設計]]
