---
id: F286-PNL-SCHEMA-R1
phase: Wave 12 W12-4 起票 / advisory_decisions full schema migration / R-01-08
priority: 最優先
status: 起票 ☆ 実装 + 実適用は HQ 別 approve (= Wave 13+)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R3-MIG-R1 (= W10-1 + W10-1a-fix、staging のみ paper_reason 追加済)
  - W12-2 中断 + W12-3-fix (= safe-skip 修正中)
  - HQ Wave 12 案 A 採用 (= 2026-05-12、advisory_decisions full migration 起票承認)
chapter: F286 PNL / R-01-08 / schema migration
---

# F286-PNL-SCHEMA-R1: advisory_decisions Full Schema Migration (= Wave 12 W12-4 起票)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実装 + 実適用は HQ 別 approve、Wave 13+)

W12-2 develop apply で発見した「**production / develop DB に advisory_decisions
table 自体が存在しない**」問題を構造的に解消する schema migration 設計。

★ staging schema を template、22 列 + PK + 2 indexes + CHECK constraints
を **完全に再現** する idempotent CREATE TABLE migration を起票。

★ 実適用 (= develop / production への CREATE TABLE) は **本起票 scope 外**、
別 wave で HQ 明示承認。

## 1. 背景

F286-PNL-R1 (= advisory_decisions、W4 系) は staging DB のみで管理されており、
production / develop には migration が一度も適用されていない。

W7-4 設計 REPORT-R1 では「production / develop の advisory_decisions を集計」
前提があるが、その前提自体が成立していない。

W12-2 ALTER 試行で初めて「table 不在」が発見され、HQ 案 A 採用で本起票が
依頼された。

## 2. staging schema (= template、template として正式採用)

```sql
CREATE TABLE advisory_decisions (
    advisory_id        TEXT NOT NULL,
    code               TEXT NOT NULL,
    base_date          TEXT,
    source_version     TEXT,
    rule_version       TEXT,
    name               TEXT,
    advisory_label     TEXT,
    decision_label     TEXT,
    fujiwara_decision  TEXT NOT NULL DEFAULT 'unknown'
        CHECK(fujiwara_decision IN
            ('adopted','watched','skipped','rejected','unknown')),
    decision_reason    TEXT,
    actual_trade       TEXT NOT NULL DEFAULT 'none'
        CHECK(actual_trade IN ('none','buy','sell','partial')),
    entry_price        REAL,
    entry_qty          INTEGER,
    exit_price         REAL,
    exit_qty           INTEGER,
    realized_pnl       REAL,
    unrealized_pnl     REAL,
    paper_pnl          REAL,
    notes              TEXT,
    created_at         TEXT NOT NULL,
    updated_at         TEXT NOT NULL,
    paper_reason       TEXT,
    PRIMARY KEY (advisory_id, code)
);

CREATE INDEX idx_advisory_decisions_base_date
    ON advisory_decisions(base_date);
CREATE INDEX idx_advisory_decisions_fujiwara_decision
    ON advisory_decisions(fujiwara_decision);
```

注: staging では W9-1c で `paper_reason TEXT` を末尾 ALTER 追加したため、
schema の column 順は元 22 列 + paper_reason の形。本 migration では
**最初から 22 列定義** で paper_reason を含む完全 schema を作成する。

## 3. 新規 migration script 設計

### 3.1 ファイル: `scripts/setup/migrate_advisory_decisions_full.py`

```python
"""F286-PNL-SCHEMA-R1: advisory_decisions full schema migration.

idempotent CREATE TABLE IF NOT EXISTS で advisory_decisions table を
作成 + 2 indexes 作成. 既存 row があれば不触.

★ table 既存ならスキップ (= safe-skip)
★ schema が staging と異なる場合は warning + skip (= 上書きしない)
★ HQ approve marker (= F286_SCHEMA_R1_HQ_APPROVE) 必須 (= production /
  develop の場合)
"""

# 既存 migrate_paper_reason.py と同 pattern:
# - --db-label / --db-path / --dry-run / --write
# - HQ marker enforce
# - 三段+六段ガード (= staging-only enforce より緩いが、--write 時は
#   production / develop でも label 一致 + marker 必須)
```

### 3.2 schema CREATE 戦略

```python
SCHEMA_DDL = """
CREATE TABLE IF NOT EXISTS advisory_decisions (
    advisory_id        TEXT NOT NULL,
    code               TEXT NOT NULL,
    base_date          TEXT,
    source_version     TEXT,
    rule_version       TEXT,
    name               TEXT,
    advisory_label     TEXT,
    decision_label     TEXT,
    fujiwara_decision  TEXT NOT NULL DEFAULT 'unknown'
        CHECK(fujiwara_decision IN
            ('adopted','watched','skipped','rejected','unknown')),
    decision_reason    TEXT,
    actual_trade       TEXT NOT NULL DEFAULT 'none'
        CHECK(actual_trade IN ('none','buy','sell','partial')),
    entry_price        REAL,
    entry_qty          INTEGER,
    exit_price         REAL,
    exit_qty           INTEGER,
    realized_pnl       REAL,
    unrealized_pnl     REAL,
    paper_pnl          REAL,
    notes              TEXT,
    created_at         TEXT NOT NULL,
    updated_at         TEXT NOT NULL,
    paper_reason       TEXT,
    PRIMARY KEY (advisory_id, code)
);
"""

INDEX_DDL_BASE_DATE = """
CREATE INDEX IF NOT EXISTS idx_advisory_decisions_base_date
    ON advisory_decisions(base_date);
"""

INDEX_DDL_FUJIWARA = """
CREATE INDEX IF NOT EXISTS idx_advisory_decisions_fujiwara_decision
    ON advisory_decisions(fujiwara_decision);
"""
```

### 3.3 schema integrity check

既存 table がある場合に **schema 整合性を検証**:

```python
def _verify_schema_matches_template(conn) -> dict:
    """staging template schema との一致を確認.

    不一致なら warning + 詳細を返す (= 上書きしない).
    """
    expected_cols = [
        ("advisory_id", "TEXT", 1, None, 1),
        ("code", "TEXT", 1, None, 2),
        # ... 22 列 + paper_reason
    ]
    actual_cols = conn.execute(
        "PRAGMA table_info(advisory_decisions)"
    ).fetchall()
    # name / type / notnull / dflt_value / pk を比較
    # 不一致 col 一覧を返す
```

### 3.4 actions

| 状況 | action |
|------|--------|
| table 不在 + dry-run | "dry_run_would_create" |
| table 不在 + write | "created" |
| table 既存 + schema 一致 | "skip_already_exists" |
| table 既存 + schema 不一致 | "schema_mismatch_warning_skipped" |

## 4. 実装フェーズ (= Wave 13+ で起票)

| sub | 担当 | 内容 |
|-----|------|------|
| W13-1 | Codex L3+L2 | migrate_advisory_decisions_full.py + tests |
| W13-1b | Codex L4 | audit |
| W13-2 | 本線 | develop apply plan vault doc |
| W13-3 | 本線 | production apply plan vault doc |
| W13-4 | 本線 | backup plan (= production apply 前) |
| W13-5 | 本線 (HQ approve 後) | dry-run 3 環境 |
| W13-6 | 本線 (HQ approve 後) | develop apply 実行 |
| W13-7 | 本線 (HQ approve 後) | production apply 実行 |

★ 各 step は HQ 個別 approve、まとめ apply 禁止。

## 5. 既存 W11-1a/b/c plan との関係

W11-1a/b/c は migrate_paper_reason の plan。本 schema migration が完了
した後の **paper_reason 列追加 plan として再活用可能**:

- F286-PNL-SCHEMA-R1 完了 → advisory_decisions table が全 DB に存在
- → W11-1a dry-run: production/develop/staging 全 "skip_already_exists"
  (= 既に paper_reason 含む完全 schema)
- → W11-1b/c は **不要** (= schema migration で paper_reason 含む)

或いは:

- F286-PNL-SCHEMA-R1 の schema を **paper_reason 除く 22 列** に絞り、
  paper_reason は W11-1 で別途 ALTER で追加
- → schema_migration と paper_reason migration を分離

**設計判断**: 統合派 (= schema に paper_reason 含む) を推奨。理由:
- 1 回の CREATE で 22 列 + paper_reason 完備
- W11-1 plan は不要化
- staging との schema 完全一致

## 6. backup plan (= production apply 前)

```bash
# 必須
cp ~/fire/data/fire.db \
   /tmp/w13_X/fire.db.pre_schema_r1_$(date +%Y%m%d_%H%M%S)
ls -la /tmp/w13_X/fire.db.pre_schema_r1_*
sha256sum ~/fire/data/fire.db /tmp/w13_X/fire.db.pre_schema_r1_*
```

backup の保存期間 / 削除責任は HQ 別途指示。

## 7. 安全制約

- CREATE TABLE IF NOT EXISTS のみ (= idempotent)
- 既存 row 不触 (= INSERT / UPDATE / DELETE 不発行)
- DROP / ALTER 不発行
- LINE 不送信
- subprocess なし
- token / channel_token / secret 不参照
- HQ marker (= F286_SCHEMA_R1_HQ_APPROVE) 経由のみ (= label 一致確認、
  値は label 文字列、secret ではない)
- production / develop apply は別 HQ 明示承認

## 8. 受入基準 (= 3 段階)

### 動いた
- script exit 0
- table 作成成功

### 機能した
- 22 列 + 2 indexes + CHECK + PK 完全作成
- staging schema との完全一致

### 期待値達成
- 全 DB で schema 一致
- W7-4 REPORT-R1 が production / develop で動作可能
- W11-1 paper_reason migration が不要化

## 9. 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE12_results|Wave 12 results]] (= 起票時点)
- [[F286_PNL_R3_schema_gap_paper_reason_2026-05-12|paper_reason 緊急 migrate 経緯]]
- [[F286_PNL_R3_MIG_R1_dry_run_plan_2026-05-12|W11-1a dry-run plan (= 期待値修正対象)]]
- [[F286_PNL_R3_MIG_R1_develop_apply_plan_2026-05-12|W11-1b develop apply plan]]
- [[F286_PNL_R3_MIG_R1_production_apply_plan_2026-05-12|W11-1c production apply plan]]
- W10-1 / W10-1a-fix commit (fire develop)
