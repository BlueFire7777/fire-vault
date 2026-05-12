---
id: FIRE-CODEX-R1-WAVE25-plan
phase: ガバナンス / Wave 25 / Phase P2 = 8 lane / F282 weekly snapshot launchd 化設計
priority: 高
status: 進行中 ☆ 8 lane (本線 + Codex 7)、F282 設計のみ、実 plist 配置 0
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 24 (= 完了、R2 v1.1 正本、P2 = 8 lane 正式運用)
  - HQ Wave 25 起票承認 + F282 launchd 化設計範囲承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / cron thaw Step 1
---

# Wave 25: F282 weekly snapshot launchd 化設計 + dry-run 試走計画

cron thaw Step 1 (= W22 設計) を Phase P2 = 8 lane で着手。**設計のみ**、
実 plist 配置 / launchctl load / launchd 登録 / cron / DB write / LINE /
token は **全て禁止**。本 wave は次の 4 deliverable を生成:

1. F282 weekly snapshot 用 plist 骨子
2. log path 設計 + logrotate 統合
3. dry-run 試走計画 (= 7 step + 1 週間試走)
4. F282 snapshot script の設計案 (= 実 impl 別 wave)

## Wave 25 sub-task (= 8 lane、本線 L5 + Codex 7)

| sub  | lane | owner | task                                       |
|------|------|-------|--------------------------------------------|
| W25-1| L5   | 本線  | plan + git status + 8 prompt 準備 + file lock 表 |
| W25-2| L1a  | Codex | F282 weekly snapshot plist 設計骨子        |
| W25-3| L1b  | Codex | log path 設計 + logrotate 統合             |
| W25-4| L2a  | Codex | dry-run 7 step + 1 週間試走計画 (主)       |
| W25-5| L2b  | Codex | 失敗時 rollback 計画 + safety nets (副)    |
| W25-6| L3   | Codex | F282 snapshot script 設計案 (= 実 impl 別 wave) |
| W25-7| L4   | Codex | adversarial audit (= 8 観点、HIGH 0 を狙う) |
| W25-8| L6   | Codex | regression plan + 本線 pytest 実行         |
| W25-9| 本線  | 本線  | F282 launchd 設計 doc 確定 + commit + 報告 |

## file lock 表 (= R2 v1.1 形式、Wave 24 で実装した運用)

### 既存 modified / untracked (= 本 wave 範囲外、触らない)

| file | 初期状態 | owner_lane | merge_owner | 区分 | 備考 |
|---|---|---|---|---|---|
| /Users/bluefire/fire/scripts/seed_pattern_layer1.py | M | none | 本線 | **forbidden** | 既存 modified、Wave 21 以前から、本 wave 範囲外 |
| /Users/bluefire/fire/simulation/research_lane/historical_indicators.py | M | none | 本線 | **forbidden** | 同上 |
| /Users/bluefire/fire/.claude/ | ?? | none | 本線 | untracked、範囲外 | 触らない |
| /Users/bluefire/fire/data/fire.staging.db.pre_restore_* | ?? | none | 本線 | staging backup、範囲外 | 触らない |

### 本 wave で本線が書き込む file

| file | 初期状態 | owner_lane | merge_owner | 区分 |
|---|---|---|---|---|
| 02_todo/FIRE_CODEX_R1_WAVE25_plan.md | ?? → NEW | 本線 (L5) | 本線 | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE25_results.md | ?? → NEW | 本線 | 本線 | allowed |
| 03_design/F282_weekly_snapshot_launchd_2026-05-12.md | ?? → NEW | 本線 | 本線 | allowed (= F282 設計本体) |
| log.md | MOD | 本線 | 本線 | allowed |

### Codex 7 lane output (= /tmp 配下、vault 不書込み)

| path | owner_lane |
|---|---|
| /tmp/codex_wave25/output/l1a_*.{stdout,last,stderr} | L1a |
| /tmp/codex_wave25/output/l1b_*.{stdout,last,stderr} | L1b |
| /tmp/codex_wave25/output/l2a_*.{stdout,last,stderr} | L2a |
| /tmp/codex_wave25/output/l2b_*.{stdout,last,stderr} | L2b |
| /tmp/codex_wave25/output/l3_*.{stdout,last,stderr} | L3 |
| /tmp/codex_wave25/output/l4_*.{stdout,last,stderr} | L4 |
| /tmp/codex_wave25/output/l6_*.{stdout,last,stderr} | L6 |

衝突 0 (= path disjoint、sandbox=read-only)。

## スコープ (= HQ Wave 25 承認範囲)

✓ F282 weekly snapshot launchd 化設計
✓ dry-run 試走計画
✓ plist 設計
✓ log path 設計
✓ no-write / no-send 方針
✓ F282 weekly snapshot 順序確認 (= W22 cron thaw Step 1 整合)
✓ audit
✓ docs
✓ 8 lane 運用可

## 禁止 (= HQ Wave 25 指示)

✗ 実 plist 配置 (= ~/Library/LaunchAgents/ への copy)
✗ launchctl load
✗ cron / launchd / crontab 登録
✗ DB write 全
✗ LINE 送信
✗ token / secret / channel_token 参照
✗ production / develop / staging DB write
✗ 実 API call
✗ workflow 変更
✗ --no-verify
✗ TODO Excel 更新
✗ F101 staging probe (= 別 HQ approve、未承認)

## 成功条件 (= P2 運用条件 + Wave 25 固有)

| 条件 | 期待 |
|---|---|
| 8 lane 全完了 | ✓ |
| file ownership 衝突 0 | ✓ (= 既存 forbidden 未接触) |
| CRITICAL 0 | ✓ |
| L4 HIGH 0 | ✓ (= Wave 24 達成水準) |
| safety violation 0 | ✓ |
| 全 pytest PASS | ✓ (= 4,041 維持) |
| wave 実時間 < 150 分 | ✓ (= 8 lane で 40-60 分想定) |
| commit 6 件以内 | ✓ (= fire-vault 2-3 件想定) |
| Codex 直接 commit 0 | ✓ |
| 本線過負荷 0 | ✓ |
| HQ 報告 1 ブロック | ✓ |

## Hard abort 条件 (= R2 v1.1 継承)

- audit CRITICAL 1 件以上
- safety violation
- 未承認 LINE / cron 登録 / token 参照
- file ownership 衝突 ≥ 2
- HQ 明示 abort

## 関連リンク

- [[../03_design/cron_thaw_design_2026-05-12|cron thaw 設計 (= Wave 22)]]
- [[../03_design/F282_environment_isolation_2026-05-08|F282 環境分離]]
- [[../03_design/FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 v1.1 設計本体]]
- [[../03_design/FIRE_CODEX_R2_codex_prompt_template_v1_2026-05-12|R2 prompt template v1.0]]
- [[FIRE_CODEX_R1_WAVE24_results|Wave 24 results]]
