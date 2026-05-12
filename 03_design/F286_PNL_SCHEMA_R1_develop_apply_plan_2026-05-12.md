---
id: F286-PNL-SCHEMA-R1-develop-apply-plan
phase: Wave 13 W13-2 / SCHEMA-R1 develop DB apply plan
priority: 最優先
status: 起票 ☆ 実適用は HQ 別 明示承認 (= Wave 14+)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-SCHEMA-R1 起票 (= W12-4)
  - W13-1 SCHEMA-R1 implementation (= 実装中)
  - HQ Wave 13 W13-2 起票承認 (= 2026-05-12)
chapter: F286 PNL / R-01-08 / schema migration
---

# F286-PNL-SCHEMA-R1 Develop DB Apply Plan (= Wave 13 W13-2)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実適用は HQ 別 明示承認、Wave 14+)

develop DB (= ~/fire/data/fire.develop.db) に **advisory_decisions table を
CREATE** する plan。staging からの完全 schema 移植 (= 22 列 + PK + 2
indexes + CHECK + paper_reason 含む)。

★ **develop DB に schema 関連 write が発生する初の実行**。HQ 明示承認必須。

## 前提条件

1. W13-1 SCHEMA-R1 impl + tests + audit 完了 (= Wave 13 内)
2. W14-X SCHEMA-R1 dry-run plan 完了 (= 期待値 develop "dry_run_would_create")
3. HQ approve marker (= env `F286_SCHEMA_R1_HQ_APPROVE=develop`) 設定
4. production DB に touch しない (= mtime unchanged 確認)
5. staging DB に touch しない (= mtime unchanged 確認)
6. develop DB を使う他プロセス (= F012 Runner / cron / OpenClaw) と排他確認

## 実行手順 (= 9 step)

### Step 1: pre-mtime 記録

```bash
mkdir -p /tmp/w14_X_develop_apply
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db | tee /tmp/w14_X_develop_apply/pre_mtimes.txt
```

### Step 2: 全 DB schema 記録

```bash
for db in fire.db fire.develop.db fire.staging.db; do
    echo "=== $db ==="
    sqlite3 ~/fire/data/$db ".schema advisory_decisions" 2>&1 \
        || echo "  (advisory_decisions table 不在)"
done | tee /tmp/w14_X_develop_apply/pre_schema.txt
```

期待:
- production: table 不在 (= CREATE 未適用)
- develop: table 不在 (= 本 plan で CREATE 予定)
- staging: table 存在 (= 22 列 + paper_reason)

### Step 3: develop dry-run 再確認

```bash
F286_SCHEMA_R1_HQ_APPROVE=develop \
    .venv/bin/python -m scripts.setup.migrate_advisory_decisions_full \
    --db-label develop \
    --db-path ~/fire/data/fire.develop.db
```

期待: `action="dry_run_would_create"`、develop DB write 0。

### Step 4: develop apply 実行 (= 本番 CREATE)

```bash
F286_SCHEMA_R1_HQ_APPROVE=develop \
    .venv/bin/python -m scripts.setup.migrate_advisory_decisions_full \
    --db-label develop \
    --db-path ~/fire/data/fire.develop.db \
    --write
```

期待:
- exit 0
- 出力: `action="created"`, `pre_has_advisory_decisions_table=False`,
  `post_columns=[22 列 + paper_reason]`
- develop DB mtime のみ変更

### Step 5: develop post-schema 確認

```bash
echo "=== develop schema 確認 ===" && \
    sqlite3 ~/fire/data/fire.develop.db \
        ".schema advisory_decisions" \
    > /tmp/w14_X_develop_apply/post_schema_develop.txt && \
    diff /tmp/w14_X_develop_apply/post_schema_develop.txt \
         <(sqlite3 ~/fire/data/fire.staging.db ".schema advisory_decisions") \
    | head -50
```

期待:
- 22 列 + paper_reason + PK (advisory_id, code) + 2 indexes + CHECK 全完備
- staging schema と論理的に同等 (= 列順は paper_reason の位置で異なる可能性)

### Step 6: develop row count 0 確認

```bash
sqlite3 ~/fire/data/fire.develop.db \
    "SELECT count(*) FROM advisory_decisions;"
```

期待: 0 (= 新規 table、空のはず)

### Step 7: production / staging unchanged 確認

```bash
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db | tee /tmp/w14_X_develop_apply/post_mtimes.txt
diff /tmp/w14_X_develop_apply/pre_mtimes.txt \
     /tmp/w14_X_develop_apply/post_mtimes.txt
```

期待:
- production mtime unchanged (= 5/7 16:12)
- staging mtime unchanged (= 5/12 00:38)
- develop mtime のみ変更

### Step 8: develop idempotent 再実行確認

```bash
F286_SCHEMA_R1_HQ_APPROVE=develop \
    .venv/bin/python -m scripts.setup.migrate_advisory_decisions_full \
    --db-label develop \
    --db-path ~/fire/data/fire.develop.db \
    --write
```

期待: `action="skip_already_exists"`、二重 CREATE なし。

### Step 9: HQ 1 ブロック報告

各 step exit code、schema 確認、production/staging unchanged、結論。

## 受容判定

| 条件 | 期待 |
|---|---|
| Step 3 dry-run exit | 0 |
| Step 4 apply exit | 0 |
| Step 4 action | "created" |
| Step 5 22 列 + paper_reason 確認 | ✓ |
| Step 5 2 indexes 確認 | ✓ |
| Step 5 PK + CHECK 確認 | ✓ |
| Step 6 row count 0 | ✓ |
| Step 7 production mtime unchanged | ✓ |
| Step 7 staging mtime unchanged | ✓ |
| Step 8 idempotent | "skip_already_exists" |
| LINE 送信 | 0 |
| token / secret 値露出 | 0 |

## 安全制約

- CREATE TABLE IF NOT EXISTS + CREATE INDEX IF NOT EXISTS のみ
- UPDATE / DROP / DELETE / INSERT / ALTER / TRUNCATE 不発行
- production / staging DB に touch しない (= mtime unchanged 強制)
- LINE 不送信 / subprocess なし
- token / channel_token / secret 値読出 0

## rollback (= 万一)

CREATE TABLE 後に異常検出時:

```bash
# DROP は SCHEMA-R1 script で扱わない
sqlite3 ~/fire/data/fire.develop.db \
    "DROP TABLE IF EXISTS advisory_decisions;"
sqlite3 ~/fire/data/fire.develop.db \
    "DROP INDEX IF EXISTS idx_advisory_decisions_base_date;"
sqlite3 ~/fire/data/fire.develop.db \
    "DROP INDEX IF EXISTS idx_advisory_decisions_fujiwara_decision;"
```

注: 本 plan は新規 CREATE で row データなし、rollback 容易。

## 実行前 HQ approve template

```
=== HQ 承認依頼 / W14-X SCHEMA-R1 develop apply 実行 ===

date:                (実行予定日)
plan:                W13-2 develop apply plan 実行
target:              develop DB (= ~/fire/data/fire.develop.db)
mode:                --write (= 実 CREATE 1 回)
予定 schema 変更:    advisory_decisions table CREATE + 2 indexes
予定 row 変更:       0 (= 新規 table)
予定 DB write 範囲:  develop DB のみ
予定 mtime 変化:     develop DB のみ
production touch:    なし (= mtime unchanged 期待)
staging touch:       なし
LINE 送信:           0
token 露出:          0
HQ marker:           F286_SCHEMA_R1_HQ_APPROVE=develop env 設定
backup:              不要 (= 新規 CREATE、既存 data なし)
排他確認:           F012 Runner / cron が develop DB に touch していないこと
所要時間 (est):     ~3 分

承認希望: develop DB CREATE TABLE 実行
```

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE13_plan|Wave 13 plan]]
- [[F286_PNL_SCHEMA_R1_advisory_decisions_full_migration_2026-05-12|SCHEMA-R1 起票]]
- W13-1 commit (fire develop、Wave 13 で確定)
