---
id: F282-weekly-snapshot-trial-monitoring
phase: F282 1 週間試走監視 (= W33-W35 期間)
priority: 高
status: 確定 v1.0 (= Wave 34 W34-2 L1a 反映)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-12
depends_on:
  - Wave 33 (= F282 plist 配置 + launchctl load 完了)
  - Wave 34 起票承認 (= 2026-05-12)
---

# F282 weekly snapshot 1 週間試走監視テンプレート

## 0. 試走スケジュール

| 日時 (JST) | イベント | 担当 |
|---|---|---|
| 2026-05-12 火 22:47 | W33 完了 (= plist 配置 + load) | 本線 |
| 5/13 〜 5/15 | 待機 (= launchd 未実行) | - |
| **5/16 土 02:00** | **launchd 自動実行 (= 1 回目)** | (自動) |
| 5/16 土 03:00 | 実行後 1 時間チェック | 本線 / Fujiwara |
| 5/16 〜 5/19 月 | daily check | Fujiwara |
| **5/19 月** | **GO / NO-GO 判定** | Fujiwara + HQ |

## 1. 5/16 土曜 03:00 実行後チェックテンプレ (= 完了条件 #3)

実行後 1 時間以内に本線 / Fujiwara が以下を実施:

### 1-1. launchd 状態

```bash
# 1-1-a: launchctl list
launchctl list | grep jp.fire.weekly-snapshot
# 期待: -  0  jp.fire.weekly-snapshot
#       (PID="-" = 未実行に戻った、LastExitStatus 0 = 正常終了)
# 異常: PID=N (= まだ実行中) or LastExitStatus 非 0

# 1-1-b: launchctl print 詳細
launchctl print gui/$(id -u)/jp.fire.weekly-snapshot
# 期待: "last exit code = 0" / "last fork count = 1" (= 1 回のみ)
# 異常: "last fork count" が 2 以上 (= 想定外複数実行)
```

### 1-2. 実行 log 確認

```bash
# 1-2-a: stdout log
tail -50 ~/fire/logs/cron/weekly-snapshot.log
# 期待: "F282 snapshot OK: snapshot_count=2, backup_count=0, source_size=N"
# 異常: log 不在 / 空 / traceback

# 1-2-b: stderr log
cat ~/fire/logs/cron/weekly-snapshot.err
# 期待: 空 or warning のみ
# 異常: traceback / error
```

### 1-3. snapshot 出力検証

```bash
# 1-3-a: snapshot output 確認 (= data/ 配下に新 staging/develop)
ls -lT ~/fire/data/fire.staging.db ~/fire/data/fire.develop.db
# 期待: mtime が 5/16 02:0X (= 自動実行で更新)

# 1-3-b: integrity_check
.venv/bin/python -c "
import sqlite3
for p in ['~/fire/data/fire.staging.db', '~/fire/data/fire.develop.db']:
    import os; p = os.path.expanduser(p)
    conn = sqlite3.connect(p)
    try:
        r = conn.execute('PRAGMA integrity_check').fetchone()
        print(f'{p}: {r[0]}')
    finally:
        conn.close()
"
# 期待: ok / ok
```

### 1-4. ★ production fire.db 完全保護検証 (= 最重要)

```bash
stat -f "%m %z %N" ~/fire/data/fire.db
# 期待: 5/12 16:17:24 / 371,081,216 bytes (= baseline 一致 ★)
# 異常: mtime / size 変化 → 即時 abort + incident
```

### 1-5. backup 3 点セット確認

```bash
ls -lt ~/fire-backups/ | head -10
# 期待:
#   fire.staging.db.bak.20260516_020X    (= 本体 backup)
#   fire.staging.db-wal.bak.20260516_020X (= 存在時のみ)
#   fire.staging.db-shm.bak.20260516_020X (= 存在時のみ)
#   fire.develop.db.bak.20260516_020X
#   (+ WAL/SHM 同様)
```

### 1-6. 専用 path 外への出力検出

```bash
# 1-6-a: data/ 配下に予期しない新 file 0
ls -la ~/fire/data/
# baseline: fire.db / fire.develop.db / fire.staging.db / .bak / .pre_restore_ /
#           snapshot/
# 5/16 後追加期待: なし (= 既存 file 更新のみ)

# 1-6-b: snapshot dir 外への出力 0
find ~/fire/data/ -newer ~/fire/data/snapshot/fire.staging.db -type f
# 期待: 5/16 02:0X 以降に作成された file は data/ 直下の staging.db / develop.db のみ
```

### 1-7. token / LINE / API 痕跡確認

```bash
grep -E '(LINE_|TOKEN|SECRET|channel_token|JQUANTS_)' \
  ~/fire/logs/cron/weekly-snapshot.log \
  ~/fire/logs/cron/weekly-snapshot.err 2>&1
# 期待: 0 件
# 異常: 任意の値 → 即時 abort + incident + token rotate
```

### 1-8. disk free 確認

```bash
df -h ~/fire/data/
# 期待: baseline (= 830 GiB) からの急減なし (= 数 GB 増加程度なら snapshot 分)
```

## 2. daily check テンプレ (= 5/16 〜 5/19 月曜、完了条件 #4)

5/16 03:00 確認後、毎日 1 回 (= Fujiwara の任意時間):

```bash
# A. launchd 状態 (= 想定外再実行検出)
launchctl list jp.fire.weekly-snapshot | grep -E "LastExit|fork"
# 期待: LastExitStatus 0 維持

launchctl print gui/$(id -u)/jp.fire.weekly-snapshot | grep "fork"
# 期待: "last fork count = 1" 維持 (= 5/16 1 回のみ)

# B. ★ production fire.db mtime 不変監視
stat -f "%m %z" ~/fire/data/fire.db
# 期待: 5/12 16:17:24 / 371,081,216 (= 完全不変)

# C. staging/develop mtime (= 5/16 02:0X 以降変化なし期待)
stat -f "%m %z" ~/fire/data/fire.staging.db ~/fire/data/fire.develop.db

# D. log 増分確認
wc -l ~/fire/logs/cron/weekly-snapshot.log
# 期待: 5/16 1 回分のみ、追加なし

# E. disk free 急減なし
df -h ~/fire/data/
```

## 3. 5/19 月曜 GO / NO-GO 判定テンプレ (= 完了条件 #5)

### GO 条件 (= 全 必須充足)

| # | 条件 | 確認方法 |
|---|---|---|
| 1 | 初回自動実行が 1 回のみ | launchctl print "last fork count = 1" |
| 2 | exit 0 | launchctl list "LastExitStatus 0" |
| 3 | snapshot 出力が専用 path (= data/、不正 path 0) | find ~/fire/data/ -newer ... 検証 |
| 4 | integrity_check ok | PRAGMA integrity_check (両 DB) |
| 5 | **production fire.db unchanged** | mtime + size 完全一致 (★ 必須) |
| 6 | log 正常 | "F282 snapshot OK" 1 行のみ |
| 7 | safety incident 0 | abort trigger 0 件 |

**全 7 件 PASS → GO** → 本番化承認 (= Wave 35 候補)

### NO-GO 条件 (= 1 件でも該当)

| # | trigger | 判定 |
|---|---|---|
| 1 | abort 条件 (= 8 件) のいずれか発生 | NO-GO |
| 2 | exit non-zero | NO-GO |
| 3 | production fire.db mtime 変化 | NO-GO + 緊急 incident |
| 4 | snapshot path 逸脱 | NO-GO + incident |
| 5 | launchd 想定外複数実行 (= fork count ≥ 2) | NO-GO |
| 6 | log 欠落 | NO-GO |
| 7 | token / LINE / API 痕跡 | NO-GO + 緊急 incident |

**NO-GO → launchctl unload + 原因分析 + 修正 + 再試走** (= 別 wave)

## 4. Abort 手順明文化 (= 完了条件 #6、本 wave で実行しない)

### 試走中 abort trigger 8 件 (= W33 で確定)

1. launchd が想定外に複数回実行
2. 既存 production DB mtime が変化
3. snapshot output が専用 path 外に出る
4. log 出力失敗
5. exit non-zero
6. disk 容量異常
7. token / LINE / API に触れた痕跡
8. HQ abort 指示

### Abort 実行コマンド (= 緊急時、Fujiwara が実行、本 wave では実行しない)

```bash
# 即時停止 (= unload のみ、plist は削除しない、復活容易)
launchctl unload ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist

# 確認: list から消える
launchctl list | grep jp.fire.weekly-snapshot
# 期待: 出力なし
```

### Abort 後の本線責務

- 即時 HQ 報告 (= 1 ブロック)
- incident doc 作成 (= 07_incidents/F282_trial_abort_YYYY-MM-DD.md)
- production DB integrity 緊急確認
- 原因分析 → 修正 plan → HQ 承認 → 再試走 (= 別 wave)

### ★ 本 wave (= Wave 34) で abort 実行しない理由

- 試走が安全に進行中 (= 5/16 まだ未実行)
- abort 条件 trigger なし
- HQ 指示「実行しない、HQ 承認を得る」

## 5. 試走中本線責務

| 日時 | 責務 |
|---|---|
| 5/16 03:00 (= 実行後 1h) | 1-1 〜 1-8 全実施、結果を log に記録 |
| 5/16 21:00 (= 当日夜) | daily check 簡易実施 |
| 5/17 / 5/18 (= 週末) | daily check (= mtime + launchctl print) |
| 5/19 月 朝 | GO / NO-GO 判定 + HQ 報告 |

## 6. 試走完了後 next step

### GO 判定 (= 5/19) → Wave 35 候補

- 安定運用継続承認
- W30 snapshot retention cleanup (= 別 wave、別 HQ approve)
- logrotate launchd 登録 (= 月初 03:00 自動 rotation)
- cron thaw Step 2 着手 (= daily refresh F100/F101/F111/F119)

### NO-GO 判定 → 修正 wave

- launchctl unload 即時
- 原因分析 + 修正 + 再試走
- 過去 wave (= W25-W33) に遡って設計問題確認

## 7. 関連リンク

- [[F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[F282_snapshot_retention_cleanup_plan_2026-05-12|W30 retention cleanup plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE33_results|Wave 33 results (= F282 本番投入)]]
- [[../02_todo/FIRE_CODEX_R1_WAVE34_plan|Wave 34 plan]]
