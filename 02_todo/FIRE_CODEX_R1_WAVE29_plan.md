---
id: FIRE-CODEX-R1-WAVE29-plan
phase: ガバナンス / Wave 29 / R2 v1.2 KPI 駆動 / F282 plist 配置 + 1 週間試走計画
priority: 高
status: 進行中 ☆ 6 lane (本線 1 + Codex 5)、設計のみ、実 plist 配置 0
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 28 (= 完了、F282 write path impl + Codex CRITICAL 5 修正)
  - HQ Wave 29 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / plist 配置 + 試走計画
---

# Wave 29: F282 plist 配置 + 1 週間試走計画 (= R2 v1.2 task 量「中」)

W25 で plist 骨子と dry-run 7 step を設計、W26 で dry-run path impl、W27
で dry-run probe 成功、W28 で write path impl + CRITICAL 5 修正。本 wave
で **plist 配置先と試走中の観察項目を impl-ready に詳細化**。実 plist 配置
/ launchctl load は HQ 未承認のため別 wave。

## Wave 29 sub-task (= 6 lane、本線 1 + Codex 5)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W29-1| L5   | 本線  | plan + git status + 5 Codex prompt            |
| W29-2| L1a  | Codex | plist 配置手順詳細 (= 配置先 / 配置時 check)   |
| W29-3| L1b  | Codex | 1 週間試走 観察項目 + 異常検知 + abort 条件   |
| W29-4| L1c  | Codex | logrotate 設定適用手順 + log dir 事前作成    |
| W29-5| L4   | Codex | adversarial audit (= 7 観点、HIGH 0)         |
| W29-6| L6   | Codex | regression plan (= 本 wave code 0、pytest 維持) |
| W29-7| 本線  | 本線  | F282 設計 doc 更新 + commit + 報告           |

R2 v1.2 lane 選択: task 量「中 (sub 5-7)」→ 6 lane 推奨範囲。Codex 5 lane
で並列起動。

## file lock 表 (= R2 v1.1)

### 既存 modified (= forbidden、未接触)

| file | 状態 | 区分 |
|---|---|---|
| /Users/bluefire/fire/scripts/seed_pattern_layer1.py | M | forbidden |
| /Users/bluefire/fire/simulation/research_lane/historical_indicators.py | M | forbidden |
| /Users/bluefire/fire/.claude/ | ?? | 範囲外 |
| /Users/bluefire/fire/data/fire.staging.db.pre_restore_* | ?? | 範囲外 |

### 本 wave 書込み (= 本線のみ)

| file | 状態 | owner | 区分 |
|---|---|---|---|
| 02_todo/FIRE_CODEX_R1_WAVE29_plan.md | NEW | 本線 | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE29_results.md | NEW | 本線 | allowed |
| 03_design/F282_weekly_snapshot_launchd_2026-05-12.md | MOD | 本線 | allowed (= 配置詳細追加) |
| log.md | MOD | 本線 | allowed |

fire develop: commit 0 (= 本 wave 設計のみ)。

## スコープ (= HQ Wave 29 承認範囲)

✓ plist 設計 (= 配置先 / 命名 / 配置時 check)
✓ 配置先設計 (= ~/Library/LaunchAgents/ への copy 手順)
✓ 1 週間 dry-run 試走計画 (= 観察項目 / 異常検知 / abort)
✓ no-write / no-send / no-token 確認
✓ log path / logrotate 設計適用手順
✓ F282 weekly snapshot 順序確認 (= W22 cron thaw Step 1 → Step 2 判断)
✓ audit
✓ docs

## 禁止 (= HQ Wave 29 指示)

✗ 実 plist 配置 (= ~/Library/LaunchAgents/ への実 copy)
✗ launchctl load
✗ 実 VACUUM INTO
✗ 実 DB snapshot 作成 (= production/develop/staging)
✗ LINE 送信
✗ token / secret / channel_token 参照
✗ cron / launchd / crontab 登録
✗ F101 staging probe
✗ workflow 変更
✗ --no-verify
✗ TODO Excel 更新

## 成功条件 (= P2 + W29 固有)

| 条件 | 期待 |
|---|---|
| 6 lane 全完了 | ✓ |
| file ownership 衝突 0 | ✓ |
| CRITICAL 0 | ✓ |
| L4 HIGH 0 | ✓ |
| safety violation 0 | ✓ (= 絶対条件) |
| 全 pytest PASS | ✓ (= 4,090 維持) |
| wave 実時間 < 150 分 | ✓ (= 30-45 分想定) |
| commit 6 件以内 | ✓ (= fire-vault 2 件) |
| Codex 直接 commit 0 | ✓ |
| 本線過負荷 0 | ✓ |
| HQ 報告 1 ブロック + 6 KPI table | ✓ |

## 関連リンク

- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計 (W25)]]
- [[../03_design/cron_thaw_design_2026-05-12|W22 cron thaw design]]
- [[FIRE_CODEX_R1_WAVE28_results|Wave 28 results (= write path impl)]]
