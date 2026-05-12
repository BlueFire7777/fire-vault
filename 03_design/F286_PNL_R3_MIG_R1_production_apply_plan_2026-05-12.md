---
id: F286-PNL-R3-MIG-R1-production-apply-plan
phase: Wave 11 W11-1c / MIG-R1 production DB apply plan
priority: 最優先
status: 起票 ☆ 実適用は HQ 別 明示承認 (= W12+、develop apply 完了後)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R3-MIG-R1 (= W10-1 + W10-1a-fix)
  - W11-1a dry-run plan (= 先行確認)
  - W11-1b develop apply plan (= 先行適用、verify pattern)
  - HQ Wave 11 W11-1 起票承認 (= 2026-05-12)
chapter: F286-PNL-R3 / R-01-08
---

# F286-PNL-R3-MIG-R1 Production DB Apply Plan (= Wave 11 W11-1c)

最終更新: 2026-05-12

## ★ 状態: 中断 → F286-PNL-SCHEMA-R1 待ち (2026-05-12 W12-2 中断後)

W12-2 develop apply で発見: **production DB にも `advisory_decisions` table
自体が存在しない**。本 plan の paper_reason column ALTER は **table 不在の
現状では実行不可**。

→ F286-PNL-SCHEMA-R1 (= advisory_decisions full schema migration) で
table を production に CREATE した後、本 plan は不要化 (= schema に
paper_reason 含むため) もしくは W11-1c として再活用判定。

詳細: [[F286_PNL_SCHEMA_R1_advisory_decisions_full_migration_2026-05-12]]

(以下、修正前の plan 内容は履歴として保持)

---

## (旧) 状態: 起票 (= 実適用は HQ 別 明示承認、develop apply 完了後)

production DB (= ~/fire/data/fire.db) に paper_reason column を追加する
plan。**production DB write が発生する初の実行**、最大級の慎重さで進める。

★ **必ず W11-1a dry-run + W11-1b develop apply 成功後に実施**。

## 前提条件

1. W11-1a dry-run plan で production DB が「dry_run_would_alter」と判定済
2. W11-1b develop apply plan が完了 (= develop で動作確認済)
3. HQ approve marker (= env `F286_MIG_R1_HQ_APPROVE=production`) 設定
4. **production DB の backup を事前作成** (= 必須)
5. develop / staging DB に touch しない

## 実行手順 (= 9 step、最も慎重に)

### Step 1: production DB backup (= 必須)

```bash
mkdir -p /tmp/w11_1c
cp ~/fire/data/fire.db /tmp/w11_1c/fire.db.pre_mig_r1_$(date +%Y%m%d_%H%M%S)
ls -la /tmp/w11_1c/fire.db.pre_mig_r1_*
# backup size と origin size が一致確認
```

### Step 2: 全 DB mtime 記録

```bash
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db | tee /tmp/w11_1c/pre_mtimes.txt
```

### Step 3: 全 DB schema 記録

```bash
for db in fire.db fire.develop.db fire.staging.db; do
    echo "=== $db ==="
    sqlite3 ~/fire/data/$db \
        "PRAGMA table_info(advisory_decisions);"
done | tee /tmp/w11_1c/pre_schema.txt
```

期待:
- production: paper_reason 不在
- develop: paper_reason 存在 (= W11-1b で適用済)
- staging: paper_reason 存在 (= W9-1c で適用済)

### Step 4: production dry-run 再確認

```bash
F286_MIG_R1_HQ_APPROVE=production \
    .venv/bin/python -m scripts.setup.migrate_paper_reason \
    --db-label production \
    --db-path ~/fire/data/fire.db
```

期待: `action="dry_run_would_alter"`、production DB write 0。

### Step 5: production apply 実行 (= 本番 ALTER)

```bash
F286_MIG_R1_HQ_APPROVE=production \
    .venv/bin/python -m scripts.setup.migrate_paper_reason \
    --db-label production \
    --db-path ~/fire/data/fire.db \
    --write
```

期待:
- exit 0
- 出力: `action="altered"`, `pre_has_paper_reason=False`,
  `post_has_paper_reason=True`
- production DB mtime のみ変更

### Step 6: production post-apply 確認

```bash
sqlite3 ~/fire/data/fire.db \
    "PRAGMA table_info(advisory_decisions);" | grep paper_reason
# 期待: NN|paper_reason|TEXT|0||0
```

### Step 7: production row 数 / 既存 row hash 不変確認

```bash
sqlite3 ~/fire/data/fire.db "SELECT count(*) FROM advisory_decisions;"
# 期待: pre と同じ row 数

sqlite3 ~/fire/data/fire.db \
    "SELECT advisory_id, code, base_date, decision_label, \
            fujiwara_decision, actual_trade, notes, \
            created_at, updated_at, paper_pnl \
     FROM advisory_decisions ORDER BY advisory_id;" \
    > /tmp/w11_1c/post_existing_hash.txt
# paper_reason 除外版で hash 取得、pre とどう比較するか:
# - apply 前にも paper_reason 除外で hash を取っておく
# - apply 後の hash が pre と一致すれば既存 row 不触担保
```

### Step 8: develop / staging unchanged 確認

```bash
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db | tee /tmp/w11_1c/post_mtimes.txt
diff /tmp/w11_1c/pre_mtimes.txt /tmp/w11_1c/post_mtimes.txt
```

期待:
- production mtime のみ変更
- develop / staging mtime unchanged

### Step 9: idempotent 再実行確認

```bash
F286_MIG_R1_HQ_APPROVE=production \
    .venv/bin/python -m scripts.setup.migrate_paper_reason \
    --db-label production \
    --db-path ~/fire/data/fire.db \
    --write
```

期待: `action="skip_already_exists"`, 二重追加なし。

## 受容判定

| 条件 | 期待 |
|---|---|
| Step 1 backup 完了 | ✓ |
| Step 4 dry-run exit | 0 |
| Step 5 apply exit | 0 |
| Step 5 action | "altered" |
| Step 6 paper_reason 存在確認 | ✓ |
| Step 7 production row 数 不変 | ✓ |
| Step 7 既存 row hash (paper_reason 除外) 不変 | ✓ |
| Step 8 production mtime のみ変更 | ✓ |
| Step 8 develop / staging mtime unchanged | ✓ |
| Step 9 idempotent | "skip_already_exists" |
| LINE 送信 | 0 |
| token / secret 値露出 | 0 |

## 失敗時 rollback

ALTER TABLE 後に何らかの異常を検出した場合:

```bash
# DB を backup から復元
cp /tmp/w11_1c/fire.db.pre_mig_r1_YYYYMMDD_HHMMSS ~/fire/data/fire.db

# verify
sqlite3 ~/fire/data/fire.db \
    "PRAGMA table_info(advisory_decisions);" | grep paper_reason
# 期待: 出力なし (= paper_reason 不在に戻った)
```

### rollback 注意

- DB lock の取り合い: production DB を使う他のプロセス (= F012 FIRE Runner /
  cron 等) が無いことを apply 前に確認
- 復元後の整合性: 復元時点の DB state に戻る、apply 後の他 transaction は
  巻き戻る (= 本 plan は ALTER のみなので影響範囲狭い)

## 安全制約

- ALTER TABLE ADD COLUMN paper_reason TEXT のみ
- UPDATE / DROP / DELETE 不発行
- develop / staging DB に touch しない (= mtime unchanged 強制)
- LINE 不送信
- subprocess なし
- token / channel_token / secret 値読出 0
- backup 必須 (= apply 前)
- production DB を使う他プロセスとの排他確認

## 実行前 HQ approve template

```
=== HQ 承認依頼 / W12-X MIG-R1 production apply 実行 ===

date:               (実行予定日)
plan:               W11-1c production apply plan 実行
target:             production DB (= ~/fire/data/fire.db)
mode:               --write (= 実 ALTER 1 回)
予定 schema 変更:    advisory_decisions に paper_reason TEXT 列追加
予定 row 変更:       0 (= ADD COLUMN)
予定 DB write 範囲: production DB のみ
予定 mtime 変化:    production DB のみ
develop touch:       なし (= mtime unchanged 期待)
staging touch:       なし
LINE 送信:           0
token 露出:          0
HQ marker:           F286_MIG_R1_HQ_APPROVE=production env 設定
backup:              **必須** (= /tmp/w11_1c/fire.db.pre_mig_r1_*)
前提:                W11-1b develop apply 完了済
所要時間 (est):     ~5-10 分
他プロセス排他:      F012 Runner / cron が production DB に touch していない
                    ことを apply 直前に確認

承認希望: production DB ALTER TABLE 実行
```

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE11_plan|Wave 11 plan]]
- [[F286_PNL_R3_MIG_R1_dry_run_plan_2026-05-12|W11-1a dry-run plan]]
- [[F286_PNL_R3_MIG_R1_develop_apply_plan_2026-05-12|W11-1b develop apply plan]]
- [[F286_PNL_R3_schema_gap_paper_reason_2026-05-12|schema gap source]]
