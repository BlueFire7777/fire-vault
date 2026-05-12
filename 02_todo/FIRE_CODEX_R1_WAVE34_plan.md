---
id: FIRE-CODEX-R1-WAVE34-plan
phase: ガバナンス / Wave 34 / F282 1 週間試走監視テンプレ + cleanup 設計
priority: 高
status: 進行中 ☆ /goal モード / 5 lane (本線 1 + Codex 4) / launchd touch 0
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 33 (= 完了、F282 本番投入 + 1 週間試走開始)
  - HQ Wave 34 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / 試走監視 + cleanup
---

# Wave 34: F282 1 週間試走監視テンプレ + W30 snapshot retention cleanup 設計

W33 で F282 本番投入完了、5/16 土曜 02:00 launchd 自動実行待機中。本 wave
では試走中の **監視テンプレート整備** + **W30 snapshot cleanup 設計** を
行う。launchd / DB / plist / token は **一切触らない** (= read-only のみ)。

## 試走前 baseline (= 完了条件 #2 + 必須作業 #2、取得済)

### F282 launchd 状態 (= read-only 確認、完了条件 #1)

```
launchctl list:
- jp.fire.weekly-snapshot  PID=- LastExitStatus=0  ✓ 登録済
- jp.fire.emergency-1445 〜 1515 (5 件、既存 F236)

launchctl print gui/501/jp.fire.weekly-snapshot:
- state = not running
- type = LaunchAgent
- path = ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist
- stdout = /Users/bluefire/fire/logs/cron/weekly-snapshot.log
- stderr = /Users/bluefire/fire/logs/cron/weekly-snapshot.err

plist file:
- /Users/bluefire/Library/LaunchAgents/jp.fire.weekly-snapshot.plist
- 1772 bytes / 5/12 22:46
```

### 既存 DB mtime + size baseline

| file | size | mtime |
|---|---|---|
| /data/fire.db (production) | 371,081,216 | 5/12 16:17:24 |
| /data/fire.develop.db | 371,081,216 | 5/12 16:11:43 |
| /data/fire.staging.db | 4,804,063,232 | 5/12 18:45:22 |

### data/snapshot/ (= W30 retention)

| file | size | mtime | sha256 |
|---|---|---|---|
| fire.staging.db | 353,128,448 | 5/12 21:43:59 | 749916c8...e17ee72 |
| fire.develop.db | 353,128,448 | 5/12 21:44:00 | 749916c8...e17ee72 (同一) |

→ 両者 sha256 一致は VACUUM INTO の決定論的出力 (= 同 source) で期待通り。

### logs/cron/ + logs/archive/

```
logs/cron/    空 dir (= launchd 未実行のため log 不在)
logs/archive/ 空 dir
```

### 現在 / 次 run

- 現在: 2026-05-12 火曜 23:03 JST
- 次 run: 2026-05-16 土曜 02:00 JST (= 2 日 + 約 3 時間後)
- disk free: 830 GiB (= 余裕大)

## Wave 34 sub-task (= 5 lane = 本線 1 + Codex 4)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W34-1| L5   | 本線  | plan + baseline + 4 Codex prompt              |
| W34-2| L1a  | Codex | 5/16 + daily check + GO/NO-GO 判定テンプレ     |
| W34-3| L1b  | Codex | W30 snapshot retention cleanup 設計            |
| W34-4| L4   | Codex | adversarial audit (= 7 観点、HIGH 0)          |
| W34-5| L6   | Codex | regression plan + 本線 pytest                 |
| W34-6| 本線  | 本線  | 2 design doc 確定 + 15 条件 + 6 KPI + 報告   |

KPI 駆動: task 量「中 (sub 4-6)」 → 5 lane。

## 完了条件 15/15 達成計画 (= /goal)

1. F282 launchd 状態 read-only 確認済み ✓ (= 上記 baseline)
2. 試走前 baseline 記録済み ✓ (= 上記)
3. 5/16 03:00 実行後チェックテンプレート → § 03_design/F282_trial_monitoring
4. daily check テンプレート → 同上
5. 5/19 GO/NO-GO 判定テンプレート → 同上
6. abort 手順明文化 → 同上
7. W30 snapshot retention cleanup plan → § 03_design/F282_retention_cleanup
8. F282 手動実行 0 ✓ (= 本 wave で実行しない)
9. launchctl load/unload 0 ✓ (= read-only のみ)
10. DB write 0 ✓ (= read-only)
11. LINE 送信 0 ✓
12. token/secret/channel_token 参照 0 ✓
13. plist 変更 0 ✓
14. docs/vault/log 更新 → 本 wave で実施
15. 6 KPI table 付き HQ 報告 → 本 wave 完了時

## 禁止項目 (= HQ Wave 34 指示)

✗ F282 手動実行
✗ VACUUM INTO 実行
✗ launchctl load / unload
✗ plist 変更 / 再配置
✗ cron / launchd / crontab 登録変更
✗ logrotate 設定変更
✗ DB write
✗ LINE 送信
✗ token / secret / channel_token 参照
✗ production / develop / staging DB 変更
✗ F101 staging probe
✗ workflow 変更
✗ --no-verify
✗ TODO Excel 更新
✗ scripts/seed_pattern_layer1.py 変更
✗ simulation/research_lane/historical_indicators.py 変更

## /goal 停止条件 監視 (= 13 件、全 clear 想定)

- F282 手動実行が必要 → 該当なし (= 本 wave 監視のみ)
- VACUUM INTO 実行が必要 → 該当なし
- launchctl load / unload が必要 → 該当なし
- plist 変更が必要 → 該当なし
- DB write が必要 → 該当なし
- LINE 送信が必要 → 該当なし
- token / secret 参照が必要 → 該当なし
- cron 登録変更が必要 → 該当なし
- audit CRITICAL 1 件以上 → 想定なし
- safety violation → 想定なし
- file ownership 衝突 ≥ 2 → 想定なし
- 150 分超過 → 想定なし (= 約 30-40 分)
- HQ abort 指示 → 想定なし

## file lock 表 (= R2 v1.1)

### 既存 modified / 範囲外 / read-only

| file | 状態 | 区分 |
|---|---|---|
| scripts/seed_pattern_layer1.py | M | forbidden |
| simulation/research_lane/historical_indicators.py | M | forbidden |
| .claude/ | ?? | 範囲外 |
| data/fire.staging.db.pre_restore_* | ?? | 範囲外 |
| data/snapshot/fire.staging.db | ?? | W30 retention (5/19 まで)、read-only |
| data/snapshot/fire.develop.db | ?? | W30 retention、read-only |
| ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist | NEW(W33) | read-only |

### 本 wave 書込み (= vault のみ)

| file | 状態 | owner | 区分 |
|---|---|---|---|
| 02_todo/FIRE_CODEX_R1_WAVE34_plan.md | NEW | 本線 (L5) | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE34_results.md | NEW | 本線 | allowed |
| 03_design/F282_weekly_snapshot_trial_monitoring_2026-05-12.md | NEW | 本線 | allowed |
| 03_design/F282_snapshot_retention_cleanup_plan_2026-05-12.md | NEW | 本線 | allowed |
| log.md | MOD | 本線 | allowed |

fire develop: commit 0 (= 設計のみ、read-only)。

## 関連リンク

- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[FIRE_CODEX_R1_WAVE33_results|Wave 33 results (= F282 本番投入)]]
- [[FIRE_CODEX_R1_WAVE30_results|Wave 30 results (= 実 VACUUM INTO + snapshot 2 件)]]
