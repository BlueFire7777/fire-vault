---
id: FIRE-CODEX-R1-WAVE30-plan
phase: ガバナンス / Wave 30 / R2 v1.2 KPI 駆動 / F282 staging 実 VACUUM INTO 試行
priority: 高
status: 進行中 ☆ 6 lane / 実 VACUUM INTO 1 回 (専用 path / 既存 staging.db 保護)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 29 (= 完了、F282 plist 配置 + 試走計画詳細)
  - HQ Wave 30 起票承認 + 実 VACUUM INTO 試行承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / staging 実 VACUUM INTO
---

# Wave 30: F282 staging 実 VACUUM INTO 試行 (= 専用 path、既存 staging.db 完全保護)

W28 で実装した F282 write path を **本番 production fire.db** を source、
**専用 snapshot path** (= ~/fire/data/snapshot/) を target として 1 回実行
する試行 wave。既存 staging.db / develop.db は **完全不変** を保証。

## Wave 30 sub-task (= 6 lane、本線 2 + Codex 4)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W30-1| L5   | 本線  | plan + mtime baseline + 4 Codex prompt        |
| W30-2| L1a  | Codex | 実行手順詳細 (= 専用 path / safety check)    |
| W30-3| L1b  | Codex | snapshot file retention / cleanup 方針        |
| W30-4| L3   | 本線  | **実 VACUUM INTO 1 回 試行**                 |
| W30-5| L4   | Codex | adversarial audit (= 8 観点)                  |
| W30-6| L6   | Codex | regression plan + 本線 pytest                 |
| W30-7| 本線  | 本線  | 結果検証 + cleanup + 6 KPI + commit + 報告   |

## 実行前 mtime baseline (= 必須)

| file | size (bytes) | mtime |
|---|---|---|
| /Users/bluefire/fire/data/fire.db | 371,081,216 | 2026-05-12 16:17:24 |
| /Users/bluefire/fire/data/fire.develop.db | 371,081,216 | 2026-05-12 16:11:43 |
| /Users/bluefire/fire/data/fire.staging.db | 4,804,063,232 | 2026-05-12 18:45:22 |

実行後にこれら 3 件全て **完全 unchanged** であることを必須確認。

## disk free baseline

- 余裕: 831 GiB
- 必要量: source × 2 = 371 MB × 2 ≒ 742 MB
- snapshot 2 件 (= staging + develop) → 約 742 MB 想定
- 余裕は十分 (= 1000 倍以上)

## 専用 snapshot path (= HQ 指示「snapshot output は専用 staging/snapshot path」)

### target_dir: `/Users/bluefire/fire/data/snapshot/`

- 既存 staging.db / develop.db (= ~/fire/data/) は **完全保護**
- 新規作成: data/snapshot/fire.staging.db / fire.develop.db
- mkdir も本 wave 内で実行 (= 専用 path 一括作成)

### 実行コマンド (= W30-4 L3 で本線実行)

```bash
mkdir -p /Users/bluefire/fire/data/snapshot/

FIRE_ENV=snapshot /Users/bluefire/fire/.venv/bin/python \
  /Users/bluefire/fire/scripts/jobs/run_f282_weekly_snapshot.py \
  --db-source production \
  --db-targets staging,develop \
  --target-dir /Users/bluefire/fire/data/snapshot/
```

期待結果:
- exit 0
- stdout: `F282 snapshot OK: snapshot_count=2, backup_count=0, source_size=N`
- data/snapshot/fire.staging.db (= 371 MB 程度)
- data/snapshot/fire.develop.db (= 371 MB 程度)
- production fire.db / 既存 staging.db / 既存 develop.db **全 mtime + size unchanged**

## file lock 表

### 既存 modified / forbidden (= 未接触)

| file | 状態 | 区分 |
|---|---|---|
| scripts/seed_pattern_layer1.py | M | forbidden |
| simulation/research_lane/historical_indicators.py | M | forbidden |
| .claude/ | ?? | 範囲外 |
| data/fire.staging.db.pre_restore_* | ?? | 範囲外 |

### 本 wave で **本線 L3 が書込む file** (= 実 VACUUM INTO 出力)

| file | 初期 | 実行後 | 区分 |
|---|---|---|---|
| /Users/bluefire/fire/data/snapshot/ | 不在 | dir 作成 | allowed (= 新規 dir、HQ 承認下) |
| /Users/bluefire/fire/data/snapshot/fire.staging.db | 不在 | 新規 ~371 MB | allowed (= VACUUM INTO 出力) |
| /Users/bluefire/fire/data/snapshot/fire.develop.db | 不在 | 新規 ~371 MB | allowed (= VACUUM INTO 出力) |

### 本 wave で **完全不変** な file

- /Users/bluefire/fire/data/fire.db (= production source、read-only 接続)
- /Users/bluefire/fire/data/fire.staging.db (= 既存 staging、別 dir なので未接触)
- /Users/bluefire/fire/data/fire.develop.db (= 同上)

### 本線 vault commit

- 02_todo/FIRE_CODEX_R1_WAVE30_plan.md (NEW)
- 02_todo/FIRE_CODEX_R1_WAVE30_results.md (NEW)
- log.md (MOD)

## W30 必須確認 (= HQ 指示)

1. ✓ 実行前 mtime/size 記録 (= 上記 baseline)
2. ✓ disk free 確認 (= 831 GiB)
3. mkdir -p data/snapshot/ (= 専用 path 作成)
4. 実 VACUUM INTO 実行
5. exit 0 確認
6. snapshot output file 存在 + size 検証 (= ~371 MB)
7. integrity_ok=True 確認 (= log)
8. size_ok=True 確認
9. **実行後 production / 既存 staging / 既存 develop mtime + size unchanged 確認** (= 最重要)
10. snapshot file retention / cleanup 方針確定
11. 結果を 1 ブロックで HQ 報告 + 6 KPI table

## スコープ (= HQ Wave 30 承認範囲)

✓ staging 向け実 VACUUM INTO 試行
✓ production は read-only source
✓ snapshot output は専用 staging/snapshot path
✓ 事前 mtime/size 記録
✓ disk free 確認
✓ output path safety 確認
✓ no overwrite 確認 (= data/snapshot/ は新規、上書きなし)
✓ integrity 確認
✓ post mtime/size 確認
✓ snapshot file retention / cleanup 方針
✓ audit
✓ HQ 報告

## 禁止 (= HQ Wave 30 指示)

✗ plist 配置
✗ launchctl load
✗ 自動実行開始
✗ production DB write
✗ develop DB write (= ~/fire/data/fire.develop.db、既存)
✗ LINE 送信
✗ token / secret / channel_token 参照
✗ cron / launchd / crontab 登録
✗ workflow 変更
✗ --no-verify
✗ TODO Excel 更新

## 成功条件 (= P2 + W30 固有 + 6 KPI)

| 条件 | 期待 |
|---|---|
| 6 lane 全完了 | ✓ |
| file ownership 衝突 0 | ✓ |
| CRITICAL 0 | ✓ |
| L4 HIGH 0 | ✓ |
| safety violation 0 | ✓ (= 絶対条件) |
| 全 pytest PASS | ✓ (= 4,090 維持) |
| wave 実時間 < 150 分 | ✓ (= 40-60 分想定) |
| commit 6 件以内 | ✓ (= fire-vault 2 件) |
| Codex 直接 commit 0 | ✓ |
| 本線過負荷 0 | ✓ |
| **実 VACUUM INTO exit 0** | ✓ |
| **production / 既存 staging / 既存 develop mtime+size unchanged** | ✓ (= 最重要) |
| HQ 報告 1 ブロック + 6 KPI table | ✓ |

## Hard abort 条件

- production fire.db mtime / size 変化 → 即時 abort + incident
- 既存 staging/develop mtime / size 変化 → 即時 abort + incident
- VACUUM INTO 失敗 → backup auto-restore (= W28 修正) で復旧
- audit CRITICAL 1 件以上 → wave 中断

## 関連リンク

- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計 (= W29 で § 7.1/7.2/7.3 追加)]]
- [[FIRE_CODEX_R1_WAVE29_results|Wave 29 results]]
- [[FIRE_CODEX_R1_WAVE28_results|Wave 28 results (= write path impl)]]
