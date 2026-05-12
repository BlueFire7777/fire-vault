---
id: F286-PNL-SCHEMA-R1-production-apply-plan
phase: Wave 13 W13-3 / SCHEMA-R1 production DB apply + backup plan
priority: 最優先
status: 起票 ☆ 実適用は HQ 別 明示承認 (= Wave 14+、develop apply 完了後)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-SCHEMA-R1 起票 (= W12-4)
  - W13-1 SCHEMA-R1 implementation
  - W13-2 develop apply plan (= 先行適用、verify pattern)
  - HQ Wave 13 W13-3 起票承認 (= 2026-05-12)
chapter: F286 PNL / R-01-08 / schema migration
---

# F286-PNL-SCHEMA-R1 Production DB Apply + Backup Plan (= Wave 13 W13-3)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実適用は HQ 別 明示承認、Wave 14+、develop apply 完了後)

production DB (= ~/fire/data/fire.db) に **advisory_decisions table を
CREATE** する plan。**production DB に schema 関連 write が発生する初の
実行**、最大級の慎重さ + backup 必須。

★ **必ず W14-X develop apply 成功後に実施**。

## 前提条件

1. W13-1 SCHEMA-R1 impl + tests + audit 完了
2. W14-X SCHEMA-R1 dry-run plan 完了
3. **W14-X develop apply 完了** (= W13-2 plan 実行成功)
4. HQ approve marker (= env `F286_SCHEMA_R1_HQ_APPROVE=production`) 設定
5. **production DB backup を事前作成** (= 必須、最大級の安全策)
6. production DB を使う他プロセス (= F012 Runner / cron) と排他確認
7. develop / staging DB に touch しない

## 実行手順 (= 10 step、最も慎重)

### Step 1: production DB backup (= 必須、最重要)

```bash
mkdir -p /tmp/w14_Y_production_apply
BACKUP_PATH=/tmp/w14_Y_production_apply/fire.db.pre_schema_r1_$(date +%Y%m%d_%H%M%S)
cp ~/fire/data/fire.db "$BACKUP_PATH"
ls -la "$BACKUP_PATH"
sha256sum ~/fire/data/fire.db "$BACKUP_PATH"
# 期待: 両 hash 一致 (= backup 完全成功)
```

backup ファイルサイズ + sha256 一致を必ず HQ 報告に含める。

### Step 2: pre-mtime + schema 記録

```bash
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db | tee /tmp/w14_Y_production_apply/pre_mtimes.txt

for db in fire.db fire.develop.db fire.staging.db; do
    echo "=== $db ==="
    sqlite3 ~/fire/data/$db ".schema advisory_decisions" 2>&1 \
        || echo "  (advisory_decisions table 不在)"
done | tee /tmp/w14_Y_production_apply/pre_schema.txt
```

期待:
- production: table 不在 (= 本 plan で CREATE 予定)
- develop: table 存在 (= W14-X develop apply で CREATE 済)
- staging: table 存在 (= 既存)

### Step 3: production schema 確認

```bash
sqlite3 ~/fire/data/fire.db \
    "SELECT name FROM sqlite_master WHERE type='table' AND name='advisory_decisions';"
# 期待: 出力なし (= table 不在)
```

### Step 4: production dry-run 再確認

```bash
F286_SCHEMA_R1_HQ_APPROVE=production \
    .venv/bin/python -m scripts.setup.migrate_advisory_decisions_full \
    --db-label production \
    --db-path ~/fire/data/fire.db
```

期待: `action="dry_run_would_create"`、production DB write 0。

### Step 5: production apply 実行 (= 本番 CREATE、最も慎重)

```bash
F286_SCHEMA_R1_HQ_APPROVE=production \
    .venv/bin/python -m scripts.setup.migrate_advisory_decisions_full \
    --db-label production \
    --db-path ~/fire/data/fire.db \
    --write
```

期待:
- exit 0
- 出力: `action="created"`, `pre_has_advisory_decisions_table=False`,
  `post_columns=[22 列 + paper_reason]`
- production DB mtime のみ変更

### Step 6: production post-schema 確認

```bash
sqlite3 ~/fire/data/fire.db ".schema advisory_decisions" \
    > /tmp/w14_Y_production_apply/post_schema_production.txt
diff /tmp/w14_Y_production_apply/post_schema_production.txt \
     <(sqlite3 ~/fire/data/fire.staging.db ".schema advisory_decisions") \
    | head -50
```

期待: staging schema と論理的に同等 (= 22 列 + paper_reason + PK + 2 indexes
+ CHECK)。

### Step 7: production row count 0 確認

```bash
sqlite3 ~/fire/data/fire.db \
    "SELECT count(*) FROM advisory_decisions;"
```

期待: 0 (= 新規 table)。

### Step 8: develop / staging unchanged 確認

```bash
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db | tee /tmp/w14_Y_production_apply/post_mtimes.txt
diff /tmp/w14_Y_production_apply/pre_mtimes.txt \
     /tmp/w14_Y_production_apply/post_mtimes.txt
```

期待:
- production mtime のみ変更
- develop mtime unchanged (= W14-X develop apply 後の時刻)
- staging mtime unchanged (= 5/12 00:38)

### Step 9: production idempotent 再実行確認

```bash
F286_SCHEMA_R1_HQ_APPROVE=production \
    .venv/bin/python -m scripts.setup.migrate_advisory_decisions_full \
    --db-label production \
    --db-path ~/fire/data/fire.db \
    --write
```

期待: `action="skip_already_exists"`。

### Step 10: HQ 1 ブロック報告

backup 情報 + 各 step exit + schema 確認 + develop/staging unchanged + 結論。

## 受容判定

| 条件 | 期待 |
|---|---|
| Step 1 backup 完了 (sha256 一致) | ✓ |
| Step 4 dry-run exit | 0 |
| Step 5 apply exit | 0 |
| Step 5 action | "created" |
| Step 6 schema 完全 | ✓ |
| Step 7 row count 0 | ✓ |
| Step 8 production mtime のみ変更 | ✓ |
| Step 8 develop / staging mtime unchanged | ✓ |
| Step 9 idempotent | "skip_already_exists" |
| LINE 送信 | 0 |
| token / secret 値露出 | 0 |

## 失敗時 rollback

CREATE 後に異常検出した場合:

```bash
# 案 A: DROP で復旧 (= 新規 CREATE で row なしなので安全)
sqlite3 ~/fire/data/fire.db \
    "DROP TABLE IF EXISTS advisory_decisions;"
sqlite3 ~/fire/data/fire.db \
    "DROP INDEX IF EXISTS idx_advisory_decisions_base_date;"
sqlite3 ~/fire/data/fire.db \
    "DROP INDEX IF EXISTS idx_advisory_decisions_fujiwara_decision;"

# 案 B: backup 復元 (= 最も慎重、application 全停止して実施)
# 1. F012 / cron 停止
# 2. cp $BACKUP_PATH ~/fire/data/fire.db
# 3. sha256sum で hash 一致確認
# 4. F012 / cron 再開
```

rollback 後 verify:
```bash
sqlite3 ~/fire/data/fire.db \
    "SELECT name FROM sqlite_master WHERE type='table' AND name='advisory_decisions';"
# 期待: 出力なし
```

## 安全制約

- CREATE TABLE IF NOT EXISTS + CREATE INDEX IF NOT EXISTS のみ
- UPDATE / DROP / DELETE / INSERT / ALTER / TRUNCATE 不発行 (本 script 内)
- develop / staging DB に touch しない (= mtime unchanged 強制)
- LINE 不送信 / subprocess なし
- token / channel_token / secret 値読出 0
- backup 必須 (= apply 前)
- production DB 使用他プロセスとの排他確認

## 実行前 HQ approve template

```
=== HQ 承認依頼 / W14-Y SCHEMA-R1 production apply 実行 ===

date:                (実行予定日)
plan:                W13-3 production apply plan 実行
target:              production DB (= ~/fire/data/fire.db)
mode:                --write (= 実 CREATE 1 回)
予定 schema 変更:    advisory_decisions table CREATE + 2 indexes
予定 row 変更:       0 (= 新規 table)
予定 DB write 範囲:  production DB のみ
予定 mtime 変化:     production DB のみ
develop touch:       なし (= W14-X develop apply 後の状態維持)
staging touch:       なし (= mtime unchanged 期待)
LINE 送信:           0
token 露出:          0
HQ marker:           F286_SCHEMA_R1_HQ_APPROVE=production env 設定
backup:              **必須** (= /tmp/w14_Y_production_apply/fire.db.pre_schema_r1_*)
前提:                W14-X develop apply 完了済
排他確認:           F012 Runner / cron が production DB に touch していないこと
所要時間 (est):     ~5-10 分

承認希望: production DB CREATE TABLE 実行 (= 最も慎重に)
```

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE13_plan|Wave 13 plan]]
- [[F286_PNL_SCHEMA_R1_advisory_decisions_full_migration_2026-05-12|SCHEMA-R1 起票]]
- [[F286_PNL_SCHEMA_R1_develop_apply_plan_2026-05-12|W13-2 develop apply plan]]
