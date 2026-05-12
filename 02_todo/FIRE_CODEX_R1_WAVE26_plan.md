---
id: FIRE-CODEX-R1-WAVE26-plan
phase: ガバナンス / Wave 26 / Phase P2 = 8 lane / F282 snapshot script impl
priority: 高
status: 進行中 ☆ 8 lane (= 本線 4 + Codex 4)、impl wave、実 VACUUM INTO 0
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 25 (= 完了、F282 launchd 設計確定)
  - HQ Wave 26 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / cron thaw Step 1 impl
---

# Wave 26: F282 snapshot script implementation (= Phase P2 impl wave)

W25 で確定した F282 weekly snapshot launchd 設計 (= § 5 script 設計) を
**実装**。R2 v1.1 「実装あり 1 回」条件 を P2 で維持する impl wave。

**実 VACUUM INTO / 実 plist 配置 / launchctl load / cron 登録 / DB write
/ LINE / token は全て禁止**。`--dry-run` path のみ実装し、tests は mock
で safety guard と probe 動作を検証。

## Wave 26 sub-task (= 8 lane、本線 4 + Codex 4)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W26-1| L5   | 本線  | Wave 26 plan + file lock + 4 Codex prompt     |
| W26-2| L1a  | Codex | script 構造詳細設計 (= Markdown)              |
| W26-3| L1b  | Codex | safety guard test 設計 (= Markdown)           |
| W26-4| L2a  | 本線  | tests TestStagingOnlyGuard 実装               |
| W26-5| L2b  | 本線  | tests TestDryRun / TestSnapshot 実装          |
| W26-6| L3   | 本線  | scripts/jobs/run_f282_weekly_snapshot.py 実装 |
| W26-7| L4   | Codex | adversarial audit                             |
| W26-8| L6   | Codex | regression PASS plan + 本線 pytest            |
| W26-9| 本線  | 本線  | commit + log.md + HQ 報告                     |

8 lane 構成: 本線 4 (L5 + L2a + L2b + L3) + Codex 4 (L1a + L1b + L4 + L6)。

## file lock 表 (= R2 v1.1 形式)

### 既存 modified (= 本 wave 範囲外)

| file | 状態 | owner | 区分 |
|---|---|---|---|
| /Users/bluefire/fire/scripts/seed_pattern_layer1.py | M | none | **forbidden** |
| /Users/bluefire/fire/simulation/research_lane/historical_indicators.py | M | none | **forbidden** |
| /Users/bluefire/fire/.claude/ | ?? | none | 範囲外 |
| /Users/bluefire/fire/data/fire.staging.db.pre_restore_* | ?? | none | 範囲外 |

### 本 wave で書込む file (= 本線担当)

| file | 状態 | owner_lane | merge_owner | 区分 |
|---|---|---|---|---|
| /Users/bluefire/fire/scripts/jobs/run_f282_weekly_snapshot.py | NEW | 本線 (L3) | 本線 | allowed |
| /Users/bluefire/fire/tests/scripts/jobs/test_f282_weekly_snapshot.py | NEW | 本線 (L2a+L2b) | 本線 | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE26_plan.md | NEW | 本線 (L5) | 本線 | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE26_results.md | NEW | 本線 | 本線 | allowed |
| log.md | MOD | 本線 | 本線 | allowed |

### Codex 4 lane output (= /tmp 配下)

- /tmp/codex_wave26/output/l1a_*.{stdout,last,stderr}
- /tmp/codex_wave26/output/l1b_*.{stdout,last,stderr}
- /tmp/codex_wave26/output/l4_*.{stdout,last,stderr}
- /tmp/codex_wave26/output/l6_*.{stdout,last,stderr}

衝突 0 (= path disjoint、sandbox=read-only)。本線が L2a/L2b/L3 を direct
implement で書込み、Codex は read-only で設計 / audit のみ。

## 成功条件 (= P2 運用条件 11/11)

| 条件 | 期待 |
|---|---|
| 8 lane 全完了 | ✓ |
| file ownership 衝突 0 | ✓ |
| CRITICAL 0 | ✓ |
| L4 HIGH 0 | ✓ |
| safety violation 0 | ✓ |
| 全 pytest PASS | ✓ (= 既存 4,041 + 新規 tests N 件) |
| wave 実時間 < 150 分 | ✓ |
| commit 6 件以内 | ✓ (= fire develop 1-2 + fire-vault 1-2) |
| Codex 直接 commit 0 | ✓ |
| 本線過負荷 0 | ✓ |
| HQ 報告 1 ブロック | ✓ |

## スコープ (= HQ Wave 26 承認範囲)

✓ scripts/jobs/run_f282_weekly_snapshot.py 実装
✓ tests/scripts/jobs/test_f282_weekly_snapshot.py 実装
✓ dry-run path 実装 (= probe only / write_count=0)
✓ production source read-only guard
✓ staging/develop target guard
✓ FIRE_ENV=snapshot 必須
✓ source basename fire.db 限定
✓ target basename fire.staging.db / fire.develop.db 限定
✓ symlink refuse / resolved basename / source-target inode collision refuse
✓ SQLite source URI mode=ro + PRAGMA query_only=ON
✓ adversarial audit
✓ regression 確認
✓ docs / log 更新

## 禁止 (= HQ Wave 26 指示)

✗ 実 DB snapshot write
✗ VACUUM INTO 実行 (= dry-run path のみ実装、実 VACUUM 0)
✗ plist 配置
✗ launchctl load
✗ cron / launchd / crontab 登録
✗ logrotate 設定適用
✗ LINE 送信
✗ token / secret / channel_token 参照
✗ production / develop / staging DB write
✗ 実 API call
✗ workflow 変更
✗ --no-verify
✗ TODO Excel 更新

## Hard abort 条件 (= R2 v1.1 継承)

- audit CRITICAL 1 件以上
- safety violation
- 未承認 LINE / cron 登録 / token 参照 / 実 DB write
- file ownership 衝突 ≥ 2
- HQ 明示 abort

## 関連リンク

- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計 (= W25)]]
- [[../03_design/cron_thaw_design_2026-05-12|W22 cron thaw design]]
- [[../03_design/FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 v1.1 設計]]
- [[FIRE_CODEX_R1_WAVE25_results|Wave 25 results]]
