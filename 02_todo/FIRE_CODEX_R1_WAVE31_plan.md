---
id: FIRE-CODEX-R1-WAVE31-plan
phase: ガバナンス / Wave 31 / log dir + logrotate 設定 + path 検証
priority: 高
status: 進行中 ☆ 5-6 lane / logrotate 未 install 発覚 / 設定 file 作成 + path 検証に scope 縮小
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 30 (= 完了、F282 staging 実 VACUUM INTO 試行成功)
  - HQ Wave 31 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / log dir + logrotate
---

# Wave 31: log dir 作成 + logrotate 設定配置 + path 検証

W29 § 7.3 で設計した log dir 作成 + logrotate 設定配置を実施。

## ★ 前提発覚 (= 本 wave 開始時)

```bash
$ which logrotate
logrotate not found

$ ls -ld /opt/homebrew/etc/logrotate.d/ /usr/local/etc/logrotate.d/
ls: No such file or directory
```

**logrotate が未インストール**。W22 / W25 / W29 設計では logrotate 採用を
前提としていたが、実環境では `brew install logrotate` 未実施。

### scope 縮小判断 (= HQ 報告で明示)

本 wave の HQ 承認範囲「logrotate dry-run / syntax check」は logrotate
install が必要。HQ 明示承認なしでの `brew install` は実施せず、本 wave は
以下に **scope 縮小**:

| 元 scope | 本 wave の実行 |
|---|---|
| log dir 作成 | ✓ 実施 |
| logrotate 設定ファイル作成 | ✓ 実施 (= リポジトリ内、~/fire/docs/logrotate.d/fire) |
| logrotate dry-run / syntax check | △ logrotate install 待ち (= 別 HQ approve) |
| plist placement 前の log path 検証 | ✓ 実施 (= dir 存在 + 権限確認) |
| audit | ✓ 実施 |
| docs | ✓ 実施 |

logrotate 未インストール状態を HQ 報告で明示、install + dry-run の HQ
別 approve を仰ぐ。

## Wave 31 sub-task (= 6 lane、本線 2 + Codex 4)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W31-1| L5   | 本線  | plan + 4 Codex prompt + 環境発覚記録          |
| W31-2| L1a  | Codex | logrotate 設定 file 内容詳細設計              |
| W31-3| L1b  | Codex | log path 検証手順設計                         |
| W31-4| L3   | 本線  | log dir 作成 + logrotate 設定 file 作成      |
| W31-5| L4   | Codex | adversarial audit (= 7 観点、logrotate 未 install 含む) |
| W31-6| L6   | Codex | regression plan + 本線 pytest                 |
| W31-7| 本線  | 本線  | 検証 + commit + 6 KPI + HQ 報告              |

## file lock 表 (= R2 v1.1)

### 既存 modified (= forbidden)

| file | 状態 | 区分 |
|---|---|---|
| scripts/seed_pattern_layer1.py | M | forbidden |
| simulation/research_lane/historical_indicators.py | M | forbidden |
| .claude/ | ?? | 範囲外 |
| data/fire.staging.db.pre_restore_* | ?? | 範囲外 |
| data/snapshot/fire.staging.db | ?? | W30 retention (= 5/19 まで保持) |
| data/snapshot/fire.develop.db | ?? | W30 retention |

### 本 wave 書込み

| file | 状態 | owner | 区分 |
|---|---|---|---|
| /Users/bluefire/fire/logs/cron/ | NEW dir | 本線 (L3) | allowed |
| /Users/bluefire/fire/logs/archive/ | NEW dir | 本線 (L3) | allowed |
| /Users/bluefire/fire/docs/logrotate.d/fire | NEW file | 本線 (L3) | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE31_plan.md | NEW | 本線 (L5) | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE31_results.md | NEW | 本線 | allowed |
| log.md | MOD | 本線 | allowed |

fire develop: 1 commit (= docs/logrotate.d/fire 追加)。

## スコープ (= HQ Wave 31 承認範囲、scope 縮小後)

✓ log dir 作成 (= ~/fire/logs/cron/ + archive/)
✓ logrotate 設定ファイル作成 (= ~/fire/docs/logrotate.d/fire)
△ logrotate dry-run / syntax check → **logrotate 未 install のため別 HQ approve 待ち**
✓ plist placement 前の log path 検証
✓ audit
✓ docs
✓ 6 KPI table 付き HQ 報告

## 禁止 (= HQ Wave 31 指示)

✗ plist 本番配置
✗ launchctl load
✗ 自動実行開始
✗ 実 LINE 送信
✗ token / secret / channel_token 参照
✗ production / develop / staging DB write
✗ F101 staging probe
✗ workflow 変更
✗ --no-verify
✗ TODO Excel 更新

## 追加禁止 (= 本 wave で発覚)

✗ `brew install logrotate` (= HQ 明示承認なし、新規 software install)
✗ /opt/homebrew/etc/ への書込み (= sudo 必要、HQ 承認なし)

## 成功条件

| 条件 | 期待 |
|---|---|
| 6 lane 全完了 | ✓ |
| CRITICAL 0 | ✓ |
| L4 HIGH 0 | ✓ |
| safety violation 0 | ✓ |
| 全 pytest PASS | ✓ (= 4,090 維持) |
| wave 実時間 < 150 分 | ✓ (= 40-50 分想定) |
| commit 6 件以内 | ✓ (= fire develop 1 + fire-vault 2) |
| log dir 2 件作成 | ✓ |
| logrotate 設定 file 作成 | ✓ |
| logrotate 未 install 明示 | ✓ (= HQ 報告) |
| 6 KPI table | ✓ |

## 関連リンク

- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計 (§ 7.3 logrotate)]]
- [[FIRE_CODEX_R1_WAVE30_results|Wave 30 results]]
