---
id: F286-R1-B4
phase: P5: Research Lane R1 (market_financials full 5-year backfill)
priority: 高 (R1 完了に必須、R2 派生指標の前提)
status: 完了 (2026-05-09、HQ 必須条件 全クリア、v2 row 1,836 → 164,678)
owner: Fujiwara
depends_on: [F286-R1-B2.5 v2 schema migration + 再 smoke]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R1_B2_5_market_financials_v2_smoke, F286_R1_Sector_Flow_and_financials_backfill
---

★ **F286 R1-B4**: market_financials_v2 full 5-year backfill。
   Tier2 universe (4,449 銘柄) × 2021-05-09 〜 2026-05-09 を
   1 run で完了 (65.28 min)、HQ 必須条件 4 項目 (ignored=0 /
   failed_codes=0 / duplicate_key_count=0 / no_record_loss=true) 全
   クリア。

# F286-R1-B4 market_financials full 5-year backfill

## サマリ

R1-B2.5 で v2 schema (3-key PK) + UPSERT 修正を完遂、HQ 承認を得て
R1-B4 (full 5-year backfill) を実行。Tier2 universe 4,449 銘柄に対し
J-Quants /v2/fins/summary を順次取得し、market_financials_v2 へ
INSERT OR REPLACE で投入。**165,110 record fetched / 162,842 新規 insert /
2,268 既存 replace、loss / failure / duplicate なし**。

## 実行サマリ

| 項目 | 値 |
|---|---|
| started | 2026-05-09T12:41:55 |
| ended | 2026-05-09T13:47:11 |
| elapsed | 3,916.6 sec (65.28 min) |
| target_db | data/fire.staging.db |
| target_table | market_financials_v2 |
| date_from / date_to | 20210509 / 20260509 (5 年) |
| code_count | 4,449 (Tier2 universe) |
| rate_limit_sleep | 0.7 sec/req |

## 結果メトリクス

| 項目 | 結果 |
|---|---|
| **fetched_records** | **165,110** |
| mapped_records | 165,110 |
| skipped_invalid | 0 |
| **inserted_count (新規)** | **162,842** |
| **replaced_count (既存上書き)** | **2,268** |
| skipped_unknown_collision | 0 |
| ignored_count | 0 |
| **failed_code_count** | **0** |
| retry_count | 0 |
| **duplicate_key_count** | **0** |
| multi_doctype_groups (response 内) | 7,999 |
| multi_doctype_records (response 内) | 8,516 |
| **row_count_before** | 1,836 |
| **row_count_after** | **164,678 (+162,842)** |
| db_size_before | 3,889.71 MB |
| db_size_after | 4,341.02 MB (+451.30 MB) |

## HQ 必須条件 全クリア

| 条件 | 結果 |
|---|---|
| ignored = 0 | ✅ 0 |
| failed_codes = 0 | ✅ 0 |
| duplicate_key_count = 0 | ✅ 0 |
| no_record_loss = true | ✅ true |
| skipped_unknown_collision = 0 | ✅ 0 |
| production DB 無触 | ✅ last_modified May 7 |
| develop DB 無触 | ✅ last_modified May 7 |
| 既存 v1 (market_financials) 無破壊 | ✅ 1,723 row 維持 |
| Premium endpoint 不使用 | ✅ Standard plan のみ |
| 外部スクレイピング不使用 | ✅ J-Quants V2 API のみ |

## constraints_check (raw)

```json
{
  "duplicate_keys_clean": true,
  "no_full_backfill": false,
  "production_db_untouched": true,
  "develop_db_untouched": true,
  "staging_only_write": true,
  "premium_endpoint_used": false,
  "external_scraping_used": false,
  "v2_schema_used": true,
  "no_record_loss": true
}
```

(`no_full_backfill: false` は意図的: R1-B4 で **承認済 full backfill を
実行した** ことを表す。 silent full の防止フラグであり、HQ 承認 +
`--hq-approved` フラグ経由で True → False に切り替わる)

## v2 doc_type 分布 (top 10)

| count | type_of_document |
|---:|---|
| 28,586 | FYFinancialStatements_Consolidated_JP |
| 26,986 | 1QFinancialStatements_Consolidated_JP |
| 26,919 | 3QFinancialStatements_Consolidated_JP |
| 26,830 | 2QFinancialStatements_Consolidated_JP |
| 23,484 | EarnForecastRevision |
| 4,574 | FYFinancialStatements_NonConsolidated_JP |
| 4,345 | 1QFinancialStatements_NonConsolidated_JP |
| 4,305 | 3QFinancialStatements_NonConsolidated_JP |
| 4,268 | 2QFinancialStatements_NonConsolidated_JP |
| 3,993 | DividendForecastRevision |

UNKNOWN: **0 件** (= 実 data に DocType 欠損 record は 1 件もなし、
R1-B2.5 で導入した UNKNOWN 正規化 + 衝突 skip ロジックは保険として
機能する状態)

## 期間カバレッジ

date range: **2016-05-09 〜 2026-05-08**
(後方は R1-B2.5 migration で v1 から引き継いだ古い row、前方は
date_to=20260509 に対して直近開示 2026-05-08 を取得済)

## field 欠損率

| field | missing_rate |
|---|---:|
| net_sales | 17.18% |
| operating_profit | 19.23% |
| ordinary_profit | 21.38% |
| profit | 17.09% |
| total_assets | 17.09% |
| equity | 17.09% |
| **cash_flow_operating** | **58.21%** |
| fiscal_year_end | 0.00% |
| **type_of_document** | **0.00%** |

cash_flow_operating の高欠損は /fins/summary が CFO を常に返さない
仕様 (= 期予想 / 修正系 record では未掲載)。R2 派生指標で必要に
応じて XBRL parse の対象 (F021) と組み合わせる方針。

## Codex 致命指摘 #3 (実装中検出 + 修正済)

> `_fetch_summary_for_codes()` silently truncates data when pagination
> exceeds 50 pages: it prints a warning, breaks the pagination loop,
> does not mark the code as failed, and still proceeds to mapping/upsert.

対応 (commit 585f478):
- 1 code 単位で per_code_records buffer を導入、pagination 完了後のみ
  all_records に flush
- 50 page 超えで `JQuantsAPIError` raise → 例外 catch で failed_codes
  に記録、部分 record は all_records に取込まない
- TestFetchPaginationHardLimit (2 tests) 追加

実 run では発火せず (= 4,449 銘柄の最大 page 数も 50 未満、
1 code 165,110 / 4,449 ≈ 37 record/code 平均、heaviest でも 1 page
で完結)。理論健全性のみ確保。

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/financials_mapping | 58 PASS |
| scripts/setup/migrate_market_financials_pk | 26 PASS |
| scripts/jobs/backfill_market_financials | 46 PASS |
| **合計 (R1 trio)** | **130 PASS** |

(R1-B2.5 119 PASS から +11 = 5 new helpers/preflight/full guard +
2 pagination hard limit + 4 preflight tests)

## commit ログ

| # | commit | hash |
|---|---|---|
| R1-B4-1 | chore(F286-R1): run market financials full backfill | 585f478 |
| R1-B4-2 | docs(F286-R1): vault market financials full backfill result | (本 commit) |
| R1-B4-3 | docs(F286-R1): log milestone — market financials full backfill completed | (次 commit) |

## 性能・安定性メモ

- 推定 57.1 min → 実測 65.28 min (~14% over)
  原因: 一部の長い履歴銘柄で 2-page pagination 発生 (50 page hard
  limit には届かず)
- retry_count = 0 (= rate limit 一切ヒットせず、0.7 sec/req は十分余裕)
- failed_code_count = 0 (= API 全成功、auth / rate / 5xx すべて発生せず)
- DB size 増加 451.30 MB は preflight 推定 (150-300 MB) を超過。
  原因: payload_json (107 fields の raw JSON) が想定より大きい (1 row
  ~ 2.7 KB)。disk free 858 GB に対し全く問題なし。

## 次のアクション

- **R1-B5 (vault swap 判断)**: 既存 v1 (market_financials, 1,723 row) を
  drop / rename して v2 を正本化するか、HQ 別途承認後に判断。当面は
  v2 単独で R2 派生指標の前提として利用可能。
- **R2 (派生指標 7 種実装)**: BPS / EPS / DivAnn / FSales 等、payload_json
  の raw 107 fields を参照して計算。R2-A1 から着手予定。
- **F021 連携**: 個別企業の F-1 (Earnings Preview) 用に
  MaterialEvent (F101 phase 3 既実装) と組み合わせ、cash_flow 等の高
  欠損 field を XBRL から補強。

## 関連リンク

- [[F286_R1_Sector_Flow_and_financials_backfill|F286-R1 親タスク]]
- [[F286_R1_B2_5_market_financials_v2_smoke|F286-R1-B2.5 v2 schema 完遂]]
- [[F286_R1_Research_Lane_implementation_plan_2026-05-09|F286-R1 plan v1.0]]
- [[F286_Research_Lane_R0_feasibility|F286 R0 feasibility]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ exit 0、JSON / DB 出力成功
- **機能した**: ✅ +162,842 row v2 投入、UNKNOWN 0、loss 0、
  doc_type 分布が想定通り (FY/Q1/Q2/Q3 が均等)
- **期待値達成**: ✅ HQ 必須条件 4 項目 全クリア、production /
  develop DB 完全無触、disk / DB size 余裕、5-year window 完備
