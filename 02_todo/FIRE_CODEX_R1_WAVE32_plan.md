---
id: FIRE-CODEX-R1-WAVE32-plan
phase: ガバナンス / Wave 32 / logrotate install + 配置 + dry-run
priority: 高
status: 進行中 ☆ 6 lane / /goal モード / 完了条件 14 項目
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 31 (= 完了、log dir + plist + logrotate config 作成)
  - HQ Wave 32 起票承認 + HQ_APPROVE_LOGROTATE_INSTALL=1 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / logrotate
---

# Wave 32: F282 logrotate install + 設定配置 + dry-run / syntax check

## 環境前提 (= W32 開始時の確認)

- brew available: /opt/homebrew/bin/brew (Homebrew 5.1.10)
- /opt/homebrew/etc/ 権限: bluefire/admin、**sudo 不要** ✓
- logrotate package: 未 installed ✓ (= 本 wave で install 対象)
- W31 完了: docs/logrotate.d/fire (= リポジトリ正本) 配置済

## Wave 32 sub-task (= 6 lane、本線 2 + Codex 4)

| sub  | lane | owner | task                                    |
|------|------|-------|-----------------------------------------|
| W32-1| L5   | 本線  | plan + 環境確認 + 4 Codex prompt        |
| W32-2| L1a  | Codex | brew install 手順詳細 + 確認 step       |
| W32-3| L1b  | Codex | logrotate -d dry-run 出力解釈           |
| W32-4| L3   | 本線  | **brew install + cp + logrotate -d 実行**|
| W32-5| L4   | Codex | adversarial audit (= 7 観点)            |
| W32-6| L6   | Codex | regression plan + 本線 pytest           |
| W32-7| 本線  | 本線  | 完了条件 14 項目検証 + commit + 報告    |

## 実行コマンド (= W32-4 L3)

```bash
# 1. logrotate install (= HQ_APPROVE_LOGROTATE_INSTALL=1 下)
brew install logrotate

# 2. install 確認
which logrotate
logrotate --version

# 3. /opt/homebrew/etc/logrotate.d/ 確認 (= brew install で作成想定)
ls -ld /opt/homebrew/etc/logrotate.d/

# 4. 設定 file 配置 (= sudo 不要、user 所有)
cp /Users/bluefire/fire/docs/logrotate.d/fire \
   /opt/homebrew/etc/logrotate.d/fire

# 5. 配置確認
ls -la /opt/homebrew/etc/logrotate.d/fire

# 6. dry-run / syntax check (= 実 rotation 0 必須)
logrotate -d /opt/homebrew/etc/logrotate.d/fire
# -d オプション = debug mode (= 実 rotation せず syntax + 動作シミュレートのみ)
```

## 完了条件 14 項目 (= HQ 指示)

1. logrotate install 状態確認 (= which + version)
2. docs/logrotate.d/fire を本番設定場所 (= /opt/homebrew/etc/logrotate.d/fire) へ配置
3. logrotate -d で dry-run / syntax check 実行
4. **実 rotation が発生していないことを確認** (= log file mtime 不変)
5. logs/cron/ と logs/archive/ の path 整合確認
6. 実 LINE 送信 0
7. DB write 0
8. token / secret / channel_token 参照 0
9. plist 配置 0
10. launchctl load 0
11. tests または必要な確認コマンド PASS (= pytest 4,090 維持)
12. audit CRITICAL 0 / HIGH 0
13. fire-vault docs/log 更新
14. 6 KPI table を含む HQ 1 ブロック報告

## 停止条件 (= /goal モード、HQ 指示)

- brew install で想定外の権限要求・失敗・interactive 入力
- /opt/homebrew/etc/logrotate.d/ への配置で sudo 権限問題
- logrotate -d が実 rotation ではなく dry-run であることを確認できない
- audit CRITICAL 1 件以上
- safety violation
- 未承認 DB write
- 未承認 LINE 送信
- token / secret / channel_token 参照
- plist 配置または launchctl load が必要になった場合
- 150 分超過
- file ownership 衝突 ≥ 2

## file lock 表

### 既存 modified / 範囲外

| file | 状態 | 区分 |
|---|---|---|
| scripts/seed_pattern_layer1.py | M | forbidden |
| simulation/research_lane/historical_indicators.py | M | forbidden |
| .claude/ | ?? | 範囲外 |
| data/fire.staging.db.pre_restore_* | ?? | 範囲外 |
| data/snapshot/fire.staging.db | ?? | W30 retention (= 5/19 まで) |
| data/snapshot/fire.develop.db | ?? | W30 retention |

### 本 wave 書込み

| file | 状態 | owner | 区分 |
|---|---|---|---|
| /opt/homebrew/etc/logrotate.d/fire | NEW (= cp 配置) | 本線 (L3) | allowed (= HQ approve 下) |
| 02_todo/FIRE_CODEX_R1_WAVE32_plan.md | NEW | 本線 | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE32_results.md | NEW | 本線 | allowed |
| log.md | MOD | 本線 | allowed |

### 本 wave で **install** される (= brew install)

- logrotate package (= /opt/homebrew/Cellar/logrotate/...)
- /opt/homebrew/bin/logrotate (symlink)
- /opt/homebrew/etc/logrotate.d/ (= 自動作成想定)

fire develop: commit 0 (= 設計 file は W31 で commit 済、本 wave は実行のみ)。

## 関連リンク

- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[FIRE_CODEX_R1_WAVE31_results|Wave 31 results (= 設定 file 作成)]]
