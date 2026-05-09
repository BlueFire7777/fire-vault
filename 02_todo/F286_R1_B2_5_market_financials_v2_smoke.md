---
id: F286-R1-B2.5
phase: P5: Research Lane R1 (market_financials v2 schema migration + 再 smoke)
priority: 高 (R1-B4 full backfill 着手前の最終ゲート)
status: 完了 (5 銘柄 + 100 銘柄 mini で no_record_loss=True、HQ 承認待ち)
owner: Fujiwara
depends_on: [F286-R1-B2 backfill smoke runner, F286-R1-B3 vault smoke result]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R1_Sector_Flow_and_financials_backfill, F286_R1_Research_Lane_implementation_plan_2026-05-09
---

★ **F286 R1-B2.5**: market_financials_v2 (3-key PK) schema migration +
   UPSERT 修正 + 5 銘柄 / 100 銘柄 mini 再 smoke。R1-B2 で確認した
   schema 制約問題 (= INSERT OR IGNORE で multi-doc-type record loss)
   を v2 schema 採用で解消、HQ 案 X2 (replacement table) 完遂。

# F286-R1-B2.5 market_financials v2 schema migration + 再 smoke

## サマリ

R1-B2 smoke で発見した schema 制約問題:

> staging DB の `market_financials` は PK (code, disclosure_date) のみ
> で、同 (code, date) で別 doc_type の record (例: FY 決算 +
> DividendForecastRevision の同日開示) が複数件来ると、INSERT OR IGNORE
> で **後続 record が落ちる** (1 銘柄あたり 5-7%、推定 100 銘柄で 105 件
> 規模の loss)。

HQ 判断 (X2 採用):
- 既存 `market_financials` (v1) は drop / rename せず残置
- 新規 `market_financials_v2` を作成 (PK: code, disclosure_date,
  type_of_document) → 全 row を migration
- backfill は v2 への INSERT OR REPLACE 3-key で全 record 保持
- DocType 欠損は "UNKNOWN" 正規化 (NOT NULL 制約対応、raw 値は
  payload_json に保持)

## 実行結果

### Phase 1: Migration (commit b6bfccc)

```
=== F286-R1-B2.5 market_financials migration ===
db_path: data/fire.staging.db
source_count_before: 1723 (v1)
v2_count_before: 0
rows_inserted: 1723
rows_unknown_normalized: 0
rows_skipped_due_to_dup_key: 0
v2_count_after: 1723
v2_duplicate_key_count: 0
source_unchanged: True (= v1 無触確認)
```

### Phase 2: 5 銘柄 smoke (commit e084162)

| 観点 | v1 (R1-B2) | v2 (R1-B2.5) |
|---|---|---|
| fetched | 234 | 234 |
| inserted | 219 | **14** (= 新規多 doc_type record) |
| ignored / replaced | 15 (loss) | 220 (replaced、loss なし) |
| skipped_unknown_collision | — | 0 |
| duplicate_key_count | 0 | 0 |
| failed_codes | 0 | 0 |
| multi_doctype_groups | 14 (検出のみ) | 14 (全 record 保持) |
| row_count delta | (+219) | 1723 → 1737 (+14) |

5 銘柄の +14 row は v1 の INSERT OR IGNORE で落ちていた multi-doc-type
record (主に決算 + 配当予想修正の同日開示)。

### Phase 3: 100 銘柄 mini smoke

| 項目 | 結果 |
|---|---|
| fetched | 1609 |
| inserted (新規) | 99 |
| replaced | 1510 |
| skipped_unknown_collision | 0 |
| duplicate_key_count | 0 |
| failed_codes | 0 |
| multi_doctype_groups | 99 |
| multi_doctype_records | 105 |
| row_count delta | 1737 → 1836 (+99) |

100 銘柄の +99 record も同様、v1 で落ちていた行が v2 で全保持された。
**no_record_loss=True**、HQ 必須条件 (ignored=0 / failed_codes=0 /
duplicate_key_count=0) を全クリア。

## constraints_check (5 銘柄 + 100 銘柄 共通)

```json
{
  "duplicate_keys_clean": true,
  "no_full_backfill": true,
  "production_db_untouched": true,
  "develop_db_untouched": true,
  "staging_only_write": true,
  "premium_endpoint_used": false,
  "external_scraping_used": false,
  "v2_schema_used": true,
  "no_record_loss": true
}
```

production / develop DB last_modified = May 7 (smoke 開始前) で
**全 phase 通じて完全無触**を確認。

## Codex 致命指摘 2 件と対応

### CRITICAL #1 — schema 検証なしで write

> backfill_market_financials は v2 を前提とするが、migration 未実行 /
> schema 不一致の DB に write すると silent loss が発生する。

対応 (commit e084162):
- `_verify_v2_schema_or_raise` を追加 (PRAGMA table_info で table 存在
  + PK 3-key + doc_type NOT NULL を検証)
- `run_smoke` 冒頭で必ず呼ぶ → 不在 / 不正 schema で `BackfillRunRefused`
- TestV2SchemaVerification 5 ケース追加

### CRITICAL #2 — UNKNOWN doc_type 衝突で record loss

> DocType 欠損行を UNKNOWN 正規化したまま 3-key PK にすると、同
> (code, disc_date, UNKNOWN) で複数 record が来ると INSERT OR REPLACE
> で後続が先行を loss させる。

対応 (commit e084162):
- `_upsert_row_v2` で UNKNOWN 衝突時のみ skip して先行 row を保持
  (`skipped_unknown_collision`)
- 非 UNKNOWN doc_type は通常の REPLACE (= amended disclosure 上書き)
- 結果: smoke 実行で skipped_unknown_collision = 0 (UNKNOWN 衝突は
  実 data に存在しなかった)
- test_upsert_unknown_collision_skipped 追加

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/financials_mapping | 58 PASS (無 regression) |
| scripts/setup/migrate_market_financials_pk | 26 PASS |
| scripts/jobs/backfill_market_financials | 35 PASS |
| **合計 (R1 trio)** | **119 PASS** |

## commit ログ

| # | commit | hash |
|---|---|---|
| R1-B2.5-1 | chore(F286-R1): add market financials schema migration | b6bfccc |
| R1-B2.5-2 | chore(F286-R1): update market financials upsert for document type key | e084162 |
| R1-B2.5-3 | docs(F286-R1): vault market financials migration smoke result | (本 commit) |
| R1-B2.5-4 | docs(F286-R1): log milestone — market financials schema migration smoke completed | (次 commit) |

## 次のアクション (HQ 判断待ち)

R1-B2.5 完了 → HQ 承認後 R1-B4 (full 5-year backfill) 着手判断:

- Tier2 universe (4,449 銘柄) × 5 年 (2021-2026)
- 推定 fetched volume: ~50,000-100,000 record (multi-doc-type 含む)
- DB size 増加見込み: ~150-200 MB (payload_json 含む)
- 所要時間: ~50-90 min (sleep 0.7 sec、4,449 銘柄)
- 既存 market_financials (v1) は最終 swap commit で drop / rename
  (HQ 別承認後)

## 関連リンク

- [[F286_R1_Sector_Flow_and_financials_backfill|F286-R1 親タスク]]
- [[F286_R1_Research_Lane_implementation_plan_2026-05-09|F286-R1 plan v1.0]]
- [[F286_Research_Lane_R0_feasibility|F286 R0 feasibility]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ migration / smoke 両方 exit 0
- **機能した**: ✅ no_record_loss=True、+14 / +99 row が v1 で
  loss していた multi-doc-type record と一致
- **期待値達成**: HQ 承認待ち (X2 schema 完遂、HQ 必須条件 全クリア)
