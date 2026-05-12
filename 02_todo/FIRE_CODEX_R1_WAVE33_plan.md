---
id: FIRE-CODEX-R1-WAVE33-plan
phase: ガバナンス / Wave 33 / F282 plist 本番配置 + launchctl load + 1 週間試走開始
priority: 高
status: 進行中 ☆ 6 lane / HQ_APPROVE_F282_PLACE=1 + _LOAD=1 / 12 必須確認
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 32 (= 完了、logrotate install + dry-run 成功)
  - HQ Wave 33 起票承認 + HQ_APPROVE_F282_PLACE=1 + _LOAD=1 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / plist 本番配置 + launchctl load
---

# Wave 33: F282 plist 本番配置 + launchctl load + 1 週間試走開始

F282 完成パイプライン **9/9 最終段階**。実 plist を ~/Library/LaunchAgents/
に配置し、launchctl load で自動実行を開始する。土曜 02:00 JST に
launchd が自動実行 → 1 週間試走 → 5/19 月曜検証。

## 配置前 baseline (= W33 必須確認 #1、取得済)

```
[plist source]
~/fire/docs/launchd/jp.fire.weekly-snapshot.plist (= 1772 bytes、W31 作成)

[~/Library/LaunchAgents/ 現状]
- jp.fire.emergency-1445.plist ... 1515.plist (5 件、F236)
- ai.openclaw.gateway.plist
- com.google.* (3 件)
- jp.fire.weekly-snapshot.plist: **未配置** ✓

[既存 launchctl list]
- jp.fire.emergency-1445/1455/1505/1510/1515 (= 5 件 load 済、F236)
- jp.fire.weekly-snapshot: 未登録 ✓

[既存 DB mtime (= baseline)]
- /data/fire.db: 371,081,216 / 5/12 16:17:24
- /data/fire.develop.db: 371,081,216 / 5/12 16:11:43
- /data/fire.staging.db: 4,804,063,232 / 5/12 18:45:22

[log dir]
- /logs/cron/ exists + writable
- /logs/archive/ exists + writable

[現在時刻]
2026-05-12 火曜 22:44 JST

[次 run 想定]
2026-05-16 土曜 02:00 JST (= 3.2 日後)
```

## Wave 33 sub-task (= 6 lane、本線 2 + Codex 4)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W33-1| L5   | 本線  | plan + baseline + 4 Codex prompt              |
| W33-2| L1a  | Codex | plist 配置 + launchctl load 手順詳細          |
| W33-3| L1b  | Codex | 1 週間試走観測項目 + abort 条件               |
| W33-4| L3   | 本線  | **plist cp + launchctl load + print/list**    |
| W33-5| L4   | Codex | adversarial audit (= 8 観点)                  |
| W33-6| L6   | Codex | regression plan + 本線 pytest                 |
| W33-7| 本線  | 本線  | 12 必須確認 + 6 KPI + commit + 試走開始報告  |

## 実行コマンド (= W33-4 L3)

```bash
# 1. plist cp (= HQ_APPROVE_F282_PLACE=1 下)
cp /Users/bluefire/fire/docs/launchd/jp.fire.weekly-snapshot.plist \
   ~/Library/LaunchAgents/

# 2. 配置確認
ls -la ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist

# 3. plutil -lint (配置後 file)
plutil -lint ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist

# 4. launchctl load (= HQ_APPROVE_F282_LOAD=1 下)
launchctl load ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist

# 5. launchctl list 確認
launchctl list | grep jp.fire.weekly-snapshot

# 6. launchctl print 詳細
launchctl print user/$(id -u)/jp.fire.weekly-snapshot
```

## W33 必須確認 12 項目 (= HQ 指示)

1. ✓ 配置前 baseline 確認 (= 上記)
2. plist 配置先確認 (= ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist)
3. plutil -lint (= OK 必須)
4. launchctl load (= exit 0)
5. launchctl print / list 確認 (= 登録成功)
6. next run 想定確認 (= 2026-05-16 土曜 02:00 JST)
7. log path 存在確認 (= logs/cron/ + logs/archive/)
8. 既存 production / staging / develop DB mtime 確認 (= 配置直後は unchanged)
9. LINE 送信 0
10. token / secret 参照 0
11. 自動実行開始後の観測項目整理 (= 別 doc に記録)
12. 1 週間試走の abort 条件明記 (= 別 doc に記録)

## 試走中 abort 条件 (= HQ 指示、5/16-5/19 に観察)

| abort 条件 | 検出方法 | 即時対応 |
|---|---|---|
| launchd が想定外に複数回実行 | launchctl list の PID + exit count、log 重複 | launchctl unload 即時 |
| 既存 DB mtime が変化 (= 非土曜 02:00 で) | stat -f "%m" の継続監視 | unload + incident |
| snapshot output が専用 path 外に出る | ls data/ + data/snapshot/ 比較 | unload + incident |
| log 出力失敗 | logs/cron/weekly-snapshot.{log,err} 不在 / 空 | launchctl print で trace |
| exit non-zero | .err に traceback + exit code | unload + 修正 |
| disk 容量異常 | df -h での急減 | unload + 調査 |
| token / LINE / API に触れた痕跡 | grep logs + git status data/ | unload + incident + token rotate |
| HQ abort 指示 | HQ メッセージ | 即時 unload |

## 試走後 GO / NO-GO 判定 (= 5/19 月曜)

| 判定 | 条件 |
|---|---|
| GO | 1 回成功 + production 不変 + log clean + 異常なし → 本番化 |
| NO-GO | 1 回でも失敗 / production 変化 / abort trigger 発生 → unload + 原因分析 |

## file lock 表 (= R2 v1.1)

### 既存 modified / 範囲外

| file | 状態 | 区分 |
|---|---|---|
| scripts/seed_pattern_layer1.py | M | forbidden |
| simulation/research_lane/historical_indicators.py | M | forbidden |
| .claude/ | ?? | 範囲外 |
| data/fire.staging.db.pre_restore_* | ?? | 範囲外 |
| data/snapshot/fire.staging.db | ?? | W30 retention (5/19) |
| data/snapshot/fire.develop.db | ?? | W30 retention (5/19) |

### 本 wave 書込み (= HQ 承認下)

| file | 状態 | owner | 区分 |
|---|---|---|---|
| ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist | NEW | 本線 (L3) | allowed (= HQ_APPROVE_F282_PLACE=1) |
| 02_todo/FIRE_CODEX_R1_WAVE33_plan.md | NEW | 本線 | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE33_results.md | NEW | 本線 | allowed |
| log.md | MOD | 本線 | allowed |

### 本 wave で **launchd システム** 変化

- launchctl load による jp.fire.weekly-snapshot 登録
- 次 run: 2026-05-16 土曜 02:00 JST (= 自動実行待機)

fire develop: commit 0 (= 既存 plist の本番配置のみ)。

## スコープ (= HQ Wave 33 承認範囲)

✓ F282 plist 本番配置
✓ launchctl load
✓ launchctl print / list 確認
✓ 土曜 02:00 JST の自動実行試走開始
✓ F282 専用 log path 確認
✓ 1 週間試走開始報告
✓ 6 KPI table 付き HQ 報告

## 禁止 (= HQ Wave 33 指示)

✗ LINE 送信
✗ token / secret / channel_token 参照
✗ F101 staging probe
✗ workflow 変更
✗ --no-verify
✗ TODO Excel 更新
✗ 楽天 / 自動発注 / Computer Use

## 関連リンク

- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[FIRE_CODEX_R1_WAVE32_results|Wave 32 results (= logrotate install)]]
- [[FIRE_CODEX_R1_WAVE30_results|Wave 30 results (= 実 VACUUM INTO 成功)]]
