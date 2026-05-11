---
id: F286-PNL-R3-schema-gap-paper-reason
phase: Wave 9 / W9-1c schema gap 緊急対応 / R-01-08 整合
priority: 最優先
status: ☆ HQ 案 A 承認待ち、承認後即実行
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 9 W9-1 (= 中断中)
  - HQ Wave 9 approve (= W9-1c 条件付き承認、2026-05-12)
chapter: ガバナンス / R-01-08 / F286 PNL-R3
---

# F286-PNL-R3 Schema Gap: paper_reason column 不在 (= Wave 9 W9-1c 中断要因)

最終更新: 2026-05-12

## ★ 状態: HQ 案 A 承認待ち、承認後即実行可

W9-1c staging UPDATE smoke の Step 2 (= seed INSERT) 実行直前に発見した
schema gap。HQ への報告 + 推奨案 A (= staging のみ ALTER TABLE 適用) の
判断待ち。

## 1. 発見事象

staging DB の `advisory_decisions` table に **`paper_reason` column が
存在しない**。production / develop DB も同様。

W6-1+2 (= compute runner) / W8-1 (= W8-0-fix audit_count_unmatched 強化) /
W9-1a (= seed runner) で実装した SQL が `paper_reason` を参照するため、
実 DB で INSERT/UPDATE 実行時に
`sqlite3.OperationalError: no such column: paper_reason`
が発生する。

### W4.1-A 時点で発見されなかった理由

W4.1-A は R-2 (= advisory_snapshots / advisory_snapshot_rows table) の
snapshot ingest smoke。advisory_decisions の paper_pnl / paper_reason
経路は実行されず、schema gap が顕在化しなかった。

W9-1c が **paper_pnl + paper_reason 経路を実 DB で動かす初回** だったため、
本 gap が判明。

## 2. 影響範囲

| 領域 | 状況 |
|------|------|
| staging DB schema | `paper_reason` 不在 |
| develop DB schema | `paper_reason` 不在 |
| production DB schema | `paper_reason` 不在 |
| migration script | 不在 (= scripts/setup/migrate_*paper_reason* 不在) |
| W6-1+2 compute runner | `paper_reason` を SELECT / UPDATE 経路で使用 |
| W8-1 W8-0-fix | `audit_count_unmatched` 集計時に `paper_reason` を NULL 返却 |
| W9-1a seed runner | INSERT に `paper_reason` 列含む (= NULL default) |
| tests (= tmp DB) | PASS (= tmp DB 内部で paper_reason 含む schema を作る) |

→ tests は PASS、実 DB では fail。schema migration 整備の欠落。

## 3. W4.1-A 時点との不整合

- W4.1-A snapshot 5 row が staging に投入 → `advisory_snapshots` table
- 実は `advisory_decisions` には 10 row 存在 (= 複合 PK (advisory_id, code)
  で 2 advisory_id × 5 code = 10)
  - `f062-r5.8-2026-05-11T09:19:53Z` × 5 code
  - `production-advisory-2026-05-09-520d6429e10e0b2a` × 5 code
- これは W4.1-A 内で `advisory_decisions` にも自動登録されていた可能性大
- 既存 row 数 10 として W9-1c の rollback 検証 baseline を更新

## 4. 本セッションでの DB 変更状況 (= 厳守)

| 項目 | 結果 |
|---|---|
| production DB write | 0 (= mtime 5/7 16:12 unchanged) |
| develop DB write | 0 (= mtime 5/7 18:14 unchanged) |
| staging DB write | 0 (= mtime 5/11 21:25 unchanged、Step 2 INSERT 未実行) |
| LINE 送信 | 0 |
| token / secret 参照 | 0 |
| cron 登録 | 0 |
| external API call | 0 |
| seed runner 実 DB 実行 | 0 (= 発見後即中断) |

## 5. pre-smoke snapshot

```
pre_snapshot hash (= paper_reason 除外版):
  db43e86366a94716892eb45e4a7802de84d0492c5a3daf873038050f7e7c2432

row count: 10
advisory_ids:
  - f062-r5.8-2026-05-11T09:19:53Z (× 5 code)
  - production-advisory-2026-05-09-520d6429e10e0b2a (× 5 code)
```

## 6. HQ 判断選択肢 (= 3 案)

### 案 A (= 本線推奨): staging のみ ALTER TABLE で migrate、W9-1c 続行

**migration SQL**:
```sql
ALTER TABLE advisory_decisions ADD COLUMN paper_reason TEXT;
```

**実行手順**:
1. staging DB のみ ALTER TABLE 実行
2. 既存 10 row は `paper_reason=NULL` default で touch なし
3. W9-1c Step 2 (= seed INSERT) 再開
4. 7 step 完了 + HQ 報告

**準拠**:
- HQ "staging DB のみ" → ✓ (= production/develop 触らず)
- HQ "production/develop DB write 禁止" → ✓
- HQ "既存 row 不触" → ✓ (= ADD COLUMN は schema 変更、row data 変わらず)
- HQ "UPDATE 対象は paper_pnl + updated_at のみ" → ✓ (= W9-1c の row UPDATE
  は seed row のみ、`paper_reason` も seed row でのみ NULL→値 になる可能性
  あり、これも HQ 制約「seed row の paper_pnl + updated_at のみ」の範囲
  なのか確認要)

**残課題** (= W9-1 完了後別 wave):
- production / develop に同 migration 適用
- migration script を `scripts/setup/migrate_NN_advisory_decisions_paper_reason.py`
  として整備
- 各 DB の schema version 追跡

### 案 B: seed/compute runner を paper_reason 不在でも動くように修正

**変更範囲**:
- W6-1+2 / W8-1 / W9-1a の paper_reason 参照を全 optional 化
- column 存在チェック + 条件分岐実装
- tests 大幅修正 (= 51+65+34 件の paper_reason 関連 test 改修)

**判定**: 大改修、別 wave 必要、本タスクの scope 大幅超

### 案 C: W9-1c 一時保留、別 wave で migration 整備

**実施範囲**:
- W9-1c 保留
- 別 wave (= W9-X-migration) で:
  - migration script 作成
  - production / develop / staging 全 DB に適用
  - tests / audit
- migration 完了後 W9-1c 再開

**判定**: 最も慎重、最も時間かかる、HQ "1 ブロック報告" を遅延

## 7. W9-1c 続行手順 (= 案 A 承認時)

### Step 1a: schema migration (= 新規追加 step)

```bash
# 念のため backup (= staging DB が大きいので時間かかる可能性)
# 既存 backup は data/fire.staging.db.pre_restore_20260511_112053 がある
# 必要なら再作成

# ALTER TABLE 実行
sqlite3 ~/fire/data/fire.staging.db \
    "ALTER TABLE advisory_decisions ADD COLUMN paper_reason TEXT;"

# verify
sqlite3 ~/fire/data/fire.staging.db \
    "PRAGMA table_info(advisory_decisions);" | grep paper_reason
# → 21|paper_reason|TEXT|0||0 が出ること

# pre-snapshot (= paper_reason 含む版で再キャプチャ)
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT advisory_id, code, base_date, decision_label,
            fujiwara_decision, actual_trade, notes,
            created_at, updated_at, paper_pnl, paper_reason
     FROM advisory_decisions ORDER BY advisory_id;" \
    > /tmp/w9_1_smoke/pre_snapshot_with_paper_reason.txt
sha256sum /tmp/w9_1_smoke/pre_snapshot_with_paper_reason.txt
```

期待:
- ALTER TABLE exit 0
- column 追加成功
- 既存 10 row は `paper_reason=NULL`
- 既存 row data 列 (= advisory_id / code / decision_label / 等) hash 不変

### Step 2-7: W9-1 plan §「W9-1c 実行手順」の手順通り続行

## 8. 残課題 (= Wave 10+ 検討)

| 課題 | 対応案 |
|------|--------|
| production / develop DB に paper_reason 不在 | 別 wave で migration script + 全 DB 適用 |
| migration script の標準化 | `scripts/setup/migrate_NN_*.py` pattern 整備 |
| schema version 追跡 | 既存 fire.db schema versioning 確認 + 整備 |
| W4.1-A 以降の advisory_decisions 自動 INSERT 経路の確認 | 10 row が R-2 snapshot ingest 時に自動で入った経路を整理 |

## 9. 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE9_plan|Wave 9 plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE8_results|Wave 8 results]]
- [[F286_PNL_R3_staging_update_smoke_plan_2026-05-11|W7-1 plan]]
- [[../log]]
