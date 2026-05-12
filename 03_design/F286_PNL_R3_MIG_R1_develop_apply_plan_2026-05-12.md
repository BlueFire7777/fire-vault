---
id: F286-PNL-R3-MIG-R1-develop-apply-plan
phase: Wave 11 W11-1b / MIG-R1 develop DB apply plan
priority: 最優先
status: 起票 ☆ 実適用は HQ 別 明示承認 (= W12+)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R3-MIG-R1 (= W10-1 + W10-1a-fix)
  - W11-1a dry-run plan (= 先行確認)
  - HQ Wave 11 W11-1 起票承認 (= 2026-05-12)
chapter: F286-PNL-R3 / R-01-08
---

# F286-PNL-R3-MIG-R1 Develop DB Apply Plan (= Wave 11 W11-1b)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実適用は HQ 別 明示承認、W11-1a dry-run 確認後)

develop DB (= ~/fire/data/fire.develop.db) に paper_reason column を追加する
plan。staging はすでに W9-1c で適用済、develop / production は本起票で
適用判定。

★ **develop DB write が発生する初の実行**。HQ 明示承認必須。

## 前提条件

1. W11-1a dry-run plan で develop DB が「dry_run_would_alter」と判定済
2. HQ approve marker (= env `F286_MIG_R1_HQ_APPROVE=develop`) 設定
3. production DB に touch しない (= mtime unchanged 確認)
4. staging DB に touch しない (= mtime unchanged 確認)

## 実行手順 (= 7 step)

### Step 1: 全 DB mtime 記録

```bash
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db | tee /tmp/w11_1b/pre_mtimes.txt
```

### Step 2: 全 DB schema 記録

```bash
for db in fire.db fire.develop.db fire.staging.db; do
    echo "=== $db ==="
    sqlite3 ~/fire/data/$db \
        "PRAGMA table_info(advisory_decisions);"
done | tee /tmp/w11_1b/pre_schema.txt
```

期待:
- production: paper_reason 不在
- develop: paper_reason 不在
- staging: paper_reason 存在

### Step 3: develop dry-run 再確認 (= 直前 sanity)

```bash
F286_MIG_R1_HQ_APPROVE=develop \
    .venv/bin/python -m scripts.setup.migrate_paper_reason \
    --db-label develop \
    --db-path ~/fire/data/fire.develop.db
```

期待: `action="dry_run_would_alter"`、DB write 0。

### Step 4: develop apply 実行 (= 本番 ALTER)

```bash
F286_MIG_R1_HQ_APPROVE=develop \
    .venv/bin/python -m scripts.setup.migrate_paper_reason \
    --db-label develop \
    --db-path ~/fire/data/fire.develop.db \
    --write
```

期待:
- exit 0
- 出力: `action="altered"`, `pre_has_paper_reason=False`,
  `post_has_paper_reason=True`
- develop DB mtime のみ変更

### Step 5: develop post-apply 確認

```bash
sqlite3 ~/fire/data/fire.develop.db \
    "PRAGMA table_info(advisory_decisions);" | grep paper_reason
# 期待: 21|paper_reason|TEXT|0||0
```

### Step 6: production / staging unchanged 確認

```bash
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db | tee /tmp/w11_1b/post_mtimes.txt
```

期待:
- production mtime: 5/7 16:12 (= unchanged)
- staging mtime: 5/12 00:38 (= W9-1c 以降 unchanged)
- develop mtime のみ変更

### Step 7: idempotent 再実行確認

```bash
F286_MIG_R1_HQ_APPROVE=develop \
    .venv/bin/python -m scripts.setup.migrate_paper_reason \
    --db-label develop \
    --db-path ~/fire/data/fire.develop.db \
    --write
```

期待: `action="skip_already_exists"`, 二重追加なし。

## 受容判定

| 条件 | 期待 |
|---|---|
| Step 3 dry-run exit | 0 |
| Step 4 apply exit | 0 |
| Step 4 action | "altered" |
| Step 5 paper_reason 存在確認 | ✓ |
| Step 6 production mtime unchanged | ✓ |
| Step 6 staging mtime unchanged | ✓ |
| Step 6 develop mtime のみ変更 | ✓ |
| Step 7 idempotent | "skip_already_exists" |
| LINE 送信 | 0 |
| token / secret 値露出 | 0 |

## 安全制約

- ALTER TABLE ADD COLUMN paper_reason TEXT のみ
- UPDATE / DROP / DELETE 不発行
- production / staging DB に touch しない (= mtime unchanged 強制)
- LINE 不送信
- subprocess なし
- token / channel_token / secret 値読出 0
- backup 推奨 (= apply 前に develop DB の copy をどこかに保存可)

## rollback (= 万一)

ALTER TABLE は SQLite で ADD COLUMN を **元に戻す手段が公式に無い**。
ロールバックが必要なら以下のいずれか:

1. DB ファイル backup 復元 (= apply 前に copy しておく)
2. CREATE TABLE NEW ... INSERT SELECT ... DROP TABLE old ... RENAME 等の手動
   migration (= 危険、推奨せず)
3. paper_reason column の値が NULL のままなら無害放置 (= 実害なし)

実害評価: paper_reason column は W6/W8/W9 で使う列、develop で UPDATE 経路は
F286-PNL-R3 compute runner で `--write` するときのみ。compute は staging
でしか実行していない (= W9-1c)。develop で paper_reason が NULL のまま
放置されても直接の害なし。

## 実行前 HQ approve template

```
=== HQ 承認依頼 / W12-X MIG-R1 develop apply 実行 ===

date:               (実行予定日)
plan:               W11-1b develop apply plan 実行
target:             develop DB (= ~/fire/data/fire.develop.db)
mode:               --write (= 実 ALTER 1 回)
予定 schema 変更:    advisory_decisions に paper_reason TEXT 列追加
予定 row 変更:       0 (= ADD COLUMN は row data 変えず)
予定 DB write 範囲: develop DB のみ
予定 mtime 変化:    develop DB のみ
production touch:    なし (= mtime unchanged 期待)
staging touch:       なし
LINE 送信:           0
token 露出:          0
HQ marker:          F286_MIG_R1_HQ_APPROVE=develop env 設定
backup:             develop DB の事前 copy を /tmp/ に保存推奨
所要時間 (est):    ~3 分

承認希望: develop DB ALTER TABLE 実行
```

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE11_plan|Wave 11 plan]]
- [[F286_PNL_R3_MIG_R1_dry_run_plan_2026-05-12|W11-1a dry-run plan]]
- [[F286_PNL_R3_schema_gap_paper_reason_2026-05-12|schema gap source]]
