---
id: F286-PNL-R3-MIG-R1-dry-run-plan
phase: Wave 11 W11-1a / MIG-R1 dry-run 確認 plan
priority: 最優先
status: 起票 ☆ 実行は HQ 別 approve (= W12+)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R3-MIG-R1 (= W10-1 + W10-1a-fix 完了、scripts/setup/migrate_paper_reason.py)
  - HQ Wave 11 W11-1 起票承認 (= 2026-05-12)
chapter: F286-PNL-R3 / R-01-08
---

# F286-PNL-R3-MIG-R1 Dry-Run 確認 Plan (= Wave 11 W11-1a)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実行は HQ 明示承認後)

W10-1 + W10-1a-fix で整備した `scripts/setup/migrate_paper_reason.py` を
**production / develop / staging 3 環境で --dry-run 実行**して、paper_reason
column の存在状況を確認する plan。

★ dry-run でも production / develop は **HQ approve marker (= env
F286_MIG_R1_HQ_APPROVE) 必須** (= W10-1a-fix HIGH #1 対応)。

★ DB write 0 / ALTER 不発生。

## 目的

1. production DB: paper_reason 不在を確認 → 「dry_run_would_alter」
2. develop DB: paper_reason 不在を確認 → 「dry_run_would_alter」
3. staging DB: paper_reason 既存を確認 (= W9-1c で適用済) →
   「skip_already_exists」

## 実行条件

### 必須環境変数

```bash
# production dry-run 用
export F286_MIG_R1_HQ_APPROVE=production

# develop dry-run 用 (= 別実行)
export F286_MIG_R1_HQ_APPROVE=develop

# staging dry-run は marker 不要 (= staging label は default OK)
```

## 実行手順 (= 4 step)

### Step 1: 全 DB mtime 記録

```bash
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db | tee /tmp/w11_1a/pre_mtimes.txt
```

### Step 2: production dry-run

```bash
F286_MIG_R1_HQ_APPROVE=production \
    .venv/bin/python -m scripts.setup.migrate_paper_reason \
    --db-label production \
    --db-path ~/fire/data/fire.db
```

期待:
- exit 0
- 出力: `{'db_path': '.../fire.db', 'db_label': 'production', 'mode': 'dry-run',
  'pre_has_paper_reason': False, 'action': 'dry_run_would_alter',
  'post_has_paper_reason': False}`
- production DB mtime unchanged
- develop / staging DB mtime unchanged

### Step 3: develop dry-run

```bash
F286_MIG_R1_HQ_APPROVE=develop \
    .venv/bin/python -m scripts.setup.migrate_paper_reason \
    --db-label develop \
    --db-path ~/fire/data/fire.develop.db
```

期待:
- exit 0
- 出力: `pre_has_paper_reason=False`, `action='dry_run_would_alter'`
- develop DB mtime unchanged
- production / staging DB mtime unchanged

### Step 4: staging dry-run

```bash
# marker 不要 (= staging は default OK)
.venv/bin/python -m scripts.setup.migrate_paper_reason \
    --db-label staging \
    --db-path ~/fire/data/fire.staging.db
```

期待:
- exit 0
- 出力: `pre_has_paper_reason=True`, `action='skip_already_exists'`
- 全 DB mtime unchanged

### Step 5: 最終 mtime 確認

```bash
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db | tee /tmp/w11_1a/post_mtimes.txt
diff /tmp/w11_1a/pre_mtimes.txt /tmp/w11_1a/post_mtimes.txt
# 期待: diff 0 行 (= dry-run なので全 mtime 不変)
```

## 受容判定

| 条件 | 期待 |
|---|---|
| production dry-run exit | 0 |
| production action | "dry_run_would_alter" |
| develop dry-run exit | 0 |
| develop action | "dry_run_would_alter" |
| staging dry-run exit | 0 |
| staging action | "skip_already_exists" |
| 全 DB mtime 不変 | ✓ |
| LINE 送信 | 0 |
| token / secret 値 露出 | 0 |
| subprocess 起動 | 0 |

## 安全制約

- ALTER TABLE 不発生 (= 全 --dry-run)
- DB write 0
- token / channel_token / secret 値読出 0 (= HQ marker は label 一致のみ確認)
- LINE 不送信
- subprocess なし
- cron 登録 0
- production / develop / staging DB mtime 全 unchanged 期待

## 実行前 HQ approve template

```
=== HQ 承認依頼 / W12-X MIG-R1 dry-run 実行 ===

date:           (実行予定日)
plan:           F286-PNL-R3-MIG-R1 dry-run plan (= W11-1a vault doc 実行)
target:         3 DB (production / develop / staging)
mode:           --dry-run のみ、ALTER 不発生
DB write:       0
LINE 送信:      0
secret 露出:    0 (= HQ marker は label 一致のみ、値は label 文字列)
所要時間:       ~5 分

承認希望: 上記 3 DB dry-run 確認
```

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE11_plan|Wave 11 plan]]
- [[F286_PNL_R3_schema_gap_paper_reason_2026-05-12|schema gap source]]
- W10-1 / W10-1a-fix commit (fire develop)
