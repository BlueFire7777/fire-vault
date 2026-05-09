---
title: F286-R1 Sector Flow / market_financials smoke 結果
date: 2026-05-09
phase: F286-R1 / R1-B3 (R1-A2 + R1-B2 smoke 結果 Vault 化)
status: smoke 完了、schema 差異 + Codex CRITICAL 修正経緯記録、R1-B4 full backfill は **未進行** (HQ schema 対応 X2 判断 + 再 smoke 後)
related: F286_R1_Research_Lane_implementation_plan_2026-05-09, F286_Research_Lane_R0_feasibility_2026-05-09, F285_Research_Lane_requirements_and_spec_2026-05-08
trigger: HQ R1-B3 着手指示 (2026-05-09、R1-B2 完了承認後 + 実 schema 差異重大による full backfill 保留)
---

# F286-R1 Sector Flow / market_financials smoke 結果

★ R1-A2 (Sector Flow Agent runner) + R1-B2 (market_financials backfill
   smoke runner) の smoke 結果を Vault 化。HQ 判断要請: 実 schema が
   想定と異なる **重大な構造差異** を検出、R1-B4 full backfill は
   **schema 対応 X2 判断 + 再 smoke 後** まで保留。

## 1. R1-A2 Sector Flow Agent smoke 結果

### 1.1 実施情報

| 項目 | 値 |
|---|---|
| 実施日時 | 2026-05-09 (R1-A2 commit 20dd659 走行直後) |
| 対象 commit (~/fire) | 20dd659 (chore(F286-R1): add Sector Flow Agent runner) + a59bd88 (R1-A1) |
| 走行コマンド | `FIRE_ENV=staging .venv/bin/python -m scripts.jobs.run_sector_flow_snapshot` |
| target_date | **2026-05-01** (最新営業日 自動選択) |
| db_path | /Users/bluefire/fire/data/fire.staging.db (read-only URI) |
| sma_window | 20 / momentum_window | 5 |

### 1.2 結果サマリ

| 項目 | 値 |
|---|--:|
| total_symbols_used | **4,194** |
| skipped_symbols_count | 258 (= 価格欠損 / 上場初日 / NULL adj_close 等) |
| total_sectors_17 | **18 sectors** (= JPX 17 業種 + その他 1) |
| total_sectors_33 | **34 sectors** (= 33 業種 + その他 1) |
| DB read-only 確認 | ✅ market_listings 4,449 / daily 526,764 不変 |
| warnings | `target_date not specified, using latest = 2026-05-01` のみ |

### 1.3 sector_17 結果

up_flow Top 5:
| sector_id | sector_name | n_codes | mean_return | sector_score |
|---|---|--:|--:|--:|
| 11 | 電気・ガス | 28 | +1.34% | 0.877 |
| 8 | 機械 | 209 | +0.31% | 0.853 |
| 3 | 建設・資材 | 270 | -0.02% | 0.771 |
| 9 | 電機・精密 | 279 | -0.03% | 0.747 |
| 13 | 商社・卸売 | 277 | +0.15% | 0.677 |

down_flow Bottom 5:
| sector_id | sector_name | n_codes | mean_return | sector_score |
|---|---|--:|--:|--:|
| 2 | エネルギー資源 | 14 | -0.27% | 0.165 |
| 17 | 不動産 | 132 | -0.16% | 0.177 |
| 7 | 鉄鋼・非鉄 | 70 | -0.08% | 0.212 |
| 16 | 金融 (除く銀行) | 90 | -0.65% | 0.247 |
| 1 | 食品 | 136 | -0.12% | 0.365 |

### 1.4 sector_33 結果

up_flow Top 5:
| sector_id | sector_name |
|---|---|
| 4050 | 電気・ガス業 |
| 3600 | 機械 |
| 5150 | 空運業 |
| 3550 | 金属製品 |
| 5050 | 陸運業 |

down_flow Bottom 5: 1050 / 3500 / 7150 / 3350 / 3100

### 1.5 sector_17 = 18 / sector_33 = 34 の確認事項

★ **JPX 標準は 17 業種 / 33 業種**、それぞれ +1 sector 多い理由:

- market_listings の sector_*_code 列に「未分類」「その他」相当の
  null/empty 業種コード or 不明業種が混在
- データクレンジング (= sector code/name の正規化) は R1 範囲外、
  Phase R3 Watchlist Ranker 統合時に実施判断
- 影響: up_flow Top 5 / down_flow Bottom 5 計算には影響軽微
  (= 全業種 ranking で多くは正しい主要業種が表示される)

JSON output: `/tmp/f286_r1_sector_flow_snapshot_2026-05-01.json`

## 2. R1-B2 5 銘柄 smoke 結果 (HQ 指定 sample)

### 2.1 実施情報

| 項目 | 値 |
|---|---|
| 対象 commit (~/fire) | f0de504 (= Codex CRITICAL 修正済 INSERT OR IGNORE 版) |
| smoke_type | five_codes |
| sample_codes | 72030 (Toyota) / 67580 (Sony) / 80350 (TEL) / 83060 (MUFG) / 16050 (INPEX) |
| sleep_sec | 0.7 (rate limit 配慮) |

### 2.2 結果

| 項目 | 値 |
|---|--:|
| fetched_records | 234 |
| mapped_records | 234 |
| skipped_invalid | 0 |
| **inserted_count** | **219** |
| **ignored_count** | **15** (= INSERT OR IGNORE で既存 row 保持、loss なし) |
| **duplicate_record_groups** | **14** (= 同 code, disc_date で複数 record の group 数) |
| **total_duplicate_records** | **15** (= 14 group で計 15 件が ignore) |
| row_count_before | 0 |
| row_count_after | **219** |
| db_size_before_mb | 3,879.82 |
| db_size_after_mb | 3,884.59 (+4.77 MB) |
| failed_codes | [] (0 件) |
| retry_count | 0 |
| duplicate_key_count | 0 (DB 内 PK で担保) |
| **schema_migration_recommended** | **True** ★ |

JSON: `/tmp/f286_r1_market_financials_smoke_result.json`

## 3. R1-B2 100 銘柄 mini 結果

### 3.1 抽出基準

market_listings から `code ASC LIMIT 100` で安定抽出 (= 13060 〜 約
20000 番台前半の universe 内銘柄、ただし F286 R0 の Tier2 universe
ではなく **listings 全 4,449 銘柄から code 順 100 件**)。

### 3.2 結果

| 項目 | 値 |
|---|--:|
| fetched_records | **1,609** |
| mapped_records | 1,609 |
| skipped_invalid | 0 |
| **inserted_count** | **1,504** |
| **ignored_count** | **105** |
| **duplicate_record_groups** | **99** |
| **total_duplicate_records** | **105** |
| row_count_before | 219 |
| row_count_after | **1,723** (+1,504) |
| db_size_before_mb | 3,884.59 |
| db_size_after_mb | 3,884.63 (+0.04 MB) |
| failed_codes | [] (0 件) |
| retry_count | 0 |
| duplicate_key_count | 0 (DB 内 PK で担保) |
| **schema_migration_recommended** | **True** ★ |

### 3.3 field 欠損率 (1,609 records)

| field | 欠損率 | 解釈 |
|---|--:|---|
| net_sales | 17.8% | 連結値ない record (= 中間期 Q1 等で連結未開示) |
| operating_profit | 17.6% | 同上 |
| ordinary_profit | 18.7% | 同上 |
| profit | 17.6% | 同上 |
| total_assets | 17.6% | 同上 |
| equity | 17.6% | 同上 |
| **cash_flow_operating** | **61.6%** | **連結のみ、四半期 record で多発** |
| fiscal_year_end | 0.0% | DocType 関連、必ず存在 |
| type_of_document | 0.0% | 必ず存在 |

★ **cash_flow_operating 61.6% 欠損の解釈**:
   /v2/fins/summary は四半期 (Q1/Q2/Q3) の record で CFO を返さない
   ことが多い (= CFO は通常通期 BS で初開示)。年次 record (= FY) では
   CFO が含まれる。R2 で派生指標 (例: ROE = NP / Eq) を計算する際は
   **連結 NP / Eq があれば十分**、CFO 欠損は許容可。

JSON: `/tmp/f286_r1_market_financials_mini100_result.json`

## 4. Codex CRITICAL 修正経緯

### 4.1 初版 INSERT OR REPLACE の問題 (Codex CRITICAL 指摘)

R1-B2 初版 commit (失敗 commit、確定せず) で `_upsert_row` を
`INSERT OR REPLACE` で実装。Codex pre-commit が以下を CRITICAL 指摘:

> CRITICAL: `_upsert_row()` uses `INSERT OR REPLACE` on actual PK
> `(code, disclosure_date)`, so multiple J-Quants records for the same
> code/date with different `type_of_document` overwrite each other.
> This loses financial disclosure rows and contradicts the R1案A key
> `(code, disclosure_date, type_of_document)`.

具体例:
- 同銘柄 同 DiscDate で `FY` (本決算) + `ForecastRevision` (上方修正)
  の 2 record が /fins/summary から返ることがある
- INSERT OR REPLACE では PK `(code, disclosure_date)` の制約により
  最後の row だけ残り、もう 1 件は **永久に loss**
- HQ 案 A の (code, disc_date, doc_type) UNIQUE と矛盾

### 4.2 INSERT OR IGNORE への修正

`_safe_insert_row` で `INSERT OR IGNORE` 採用:
- 既存 (code, disc_date) row があれば **insert を ignore** (= 既存
  row 保持、loss なし)
- 同 (code, disc_date) で 2 件目以降の record (= 異 doc_type) は
  ignored_count に集計
- response 内の同 (code, disc_date) 多 record は
  `_detect_duplicate_records_in_response` で検出、
  `duplicate_record_groups` / `total_duplicate_records` /
  `examples` を JSON 出力
- `schema_migration_recommended=True` で HQ 判断要請 flag

### 4.3 INSERT OR IGNORE の意味的問題 (full backfill では NG)

- INSERT OR IGNORE は smoke では loss を防ぐ安全策
- しかし、運用時 (= 修正開示が後で来る場合)、既存 row が残るため
  **修正開示 (RetroRst / ChgByASRev / ForecastRevision 等) が反映
  されない** semantic 問題
- → full backfill 継続策としては NG (HQ 確認済)

## 5. schema 差異

### 5.1 想定 vs 実態

| 項目 | HQ 案 A 想定 | 実 staging schema |
|---|---|---|
| PRIMARY KEY | (code, disclosure_date, **type_of_document**) | (code, disclosure_date) |
| 同日複数 doc_type 保持 | ✅ 可能 | ❌ 不可 |
| 修正開示の独立保持 | ✅ 可能 | ❌ 上書き or ignore |

### 5.2 影響定量

| smoke | 同 (code, disc_date) 複数 record | loss/ignore 件数 |
|---|--:|--:|
| 5 銘柄 (HQ sample) | 14 group | 15 件 (= ignore で保護) |
| 100 銘柄 mini | 99 group | 105 件 (= ignore で保護) |

★ **full backfill (= 4,449 銘柄 × 過去 5 年) では数千〜数万件の
   record が ignore (= 修正開示など) され、情報欠落リスク** ★

100 銘柄 mini で 105 件 / 1,609 records = **6.5% が同 (code, disc_date)
複数 record**。全銘柄 5 年で同率なら、推定 88,000 records × 6.5% ≈
5,720 件の ignore / loss 候補が発生。

### 5.3 実 schema 確認 SQL

```sql
SELECT sql FROM sqlite_master WHERE name='market_financials';

-- CREATE TABLE market_financials (
--     code TEXT NOT NULL,
--     disclosure_date TEXT NOT NULL,
--     fiscal_year_end TEXT,
--     type_of_document TEXT,
--     net_sales REAL,
--     operating_profit REAL,
--     ordinary_profit REAL,
--     profit REAL,
--     total_assets REAL,
--     equity REAL,
--     cash_flow_operating REAL,
--     payload_json TEXT,
--     fetched_at TEXT NOT NULL,
--     PRIMARY KEY (code, disclosure_date)   ← HQ 案 A と異なる
-- )
```

## 6. schema 対応案 比較

### 6.1 案 X1: 現状維持 INSERT OR IGNORE

実装:
- 現実装 (R1-B2 確定 f0de504)
- 同 (code, disc_date) の 2 件目以降を ignore

利点:
- migration 不要、即時運用可能
- DB schema 変更による既存依存への影響ゼロ

欠点:
- 修正開示 (RetroRst / ChgByASRev) が反映されない
- 業績修正情報の loss
- R2 派生指標 (= 業績修正フラグ) の計算精度低下

★ **判定: 非推奨** (R1 全体の意味的整合性を損なう)

### 6.2 案 X2: schema migration で type_of_document or disc_no を PK に追加 ★ **HQ 採用候補** ★

実装:
- staging DB の market_financials に migration:
  ```sql
  -- 案 X2-A: type_of_document を PK に追加
  ALTER TABLE market_financials DROP CONSTRAINT pk_old;
  -- 既存 SQLite は ALTER で PK 変更不可、新 table → INSERT → swap

  -- 実装案:
  CREATE TABLE market_financials_new (
    code TEXT NOT NULL,
    disclosure_date TEXT NOT NULL,
    fiscal_year_end TEXT,
    type_of_document TEXT NOT NULL,  -- NOT NULL 追加
    net_sales REAL, ..., payload_json TEXT, fetched_at TEXT NOT NULL,
    PRIMARY KEY (code, disclosure_date, type_of_document)
  );
  INSERT INTO market_financials_new SELECT * FROM market_financials;
  DROP TABLE market_financials;
  ALTER TABLE market_financials_new RENAME TO market_financials;
  CREATE INDEX idx_market_financials_date ON market_financials(disclosure_date);
  ```

利点:
- 同 (code, disc_date) 複数 doc_type を全保持 (= loss なし)
- 修正開示 / 上方修正 / 本決算が独立 row として保存
- HQ 案 A と整合

欠点:
- migration 必要 (staging のみ、但し既存 data 移行 + index 再作成)
- type_of_document が NULL の record は migration で扱い注意
  (= "Unknown" 等で埋める or NOT NULL 緩和)
- INSERT OR REPLACE / IGNORE のロジック調整必要

★ **判定: HQ 採用候補** ★ HQ 確認済の方針。

実装範囲 (R1-B2.5 として別 commit 想定):
1. scripts/setup/migrate_market_financials_pk.py 新規 (= staging のみ
   guard、既存 data 保全 migration)
2. scripts/jobs/backfill_market_financials.py 修正 (= INSERT OR
   REPLACE で 3-key UPSERT、修正開示の最新を保持)
3. test 追加 (migration / upsert)
4. 5 銘柄 smoke + 100 mini を再実行 (= duplicate 0 / ignored 0 期待)

### 6.3 案 X3: INSERT OR REPLACE 維持

実装:
- 現実装の INSERT OR REPLACE を維持
- 同 (code, disc_date) で別 record が来たら最新で上書き

利点:
- migration 不要
- 修正開示の「最新」だけが残る (= 現業務的に十分かも)

欠点:
- **本決算 + 修正開示** の 2 record があれば本決算が消える可能性
- record loss 発生
- HQ 案 A 違反

★ **判定: NG** (HQ 確認済、Codex CRITICAL の指摘理由)

### 6.4 推奨

★ **案 X2 採用** ★

R1-B2.5 として別 commit で:
1. staging schema migration (PK 拡張)
2. backfill_market_financials を INSERT OR REPLACE 3-key UPSERT に変更
3. 5 銘柄 + 100 mini 再 smoke
4. 結果 vault 化 → R1-B4 full backfill 進行可否を HQ 判断

## 7. R1-B4 進む条件 (= full 5-year backfill 着手判定)

★ R1-B4 進行は **以下すべて成立後** ★

| 条件 | 状態 |
|---|---|
| schema 対応方針 (X2) を HQ が承認 | **未** (本 vault で要請中) |
| staging migration (R1-B2.5) 実施 | **未** (X2 承認後) |
| INSERT OR REPLACE 3-key UPSERT 実装 | **未** (X2 承認後) |
| 5 銘柄 再 smoke 実施 + duplicate_record_groups=0 | 未 |
| 100 銘柄 mini 再 smoke + ignored=0 期待 | 未 |
| schema_migration_recommended=False で再 vault | 未 |
| production / develop DB 無触 | ✅ (smoke で確認済) |

それまで R1-B4 (full 5-year backfill、4,449 銘柄 × 60 営業日 / 期 ×
20 期 = 約 88,000 records) は **禁止**。

## 8. 制約遵守 (R1-B3)

| 制約 | 結果 |
|---|---|
| Vault のみ編集 | ✅ ~/fire 触らず |
| ~/fire 側追加実装なし (本タスク) | ✅ |
| full backfill 禁止 | ✅ |
| staging DB 追加 write 禁止 | ✅ Vault のみ |
| production / develop DB 無触 | ✅ |
| 自動発注 / 楽天 / Computer Use 禁止 | ✅ |
| --no-verify 不使用 | ✅ |
| 個別 commit 厳守 | ✅ (本 commit + log milestone 別 commit) |
| scripts/seed_pattern_layer1.py 触らず | ✅ |

## 9. 関連 commit

~/fire 側:
| commit | 内容 |
|---|---|
| e718f1a | R0 precheck script + 3 endpoint client 拡張 |
| a59bd88 | R1-A1 Sector Flow Agent MVP |
| 20dd659 | R1-A2 Sector Flow runner |
| 334f793 | R1-B1 fins summary mapping |
| **f0de504** | R1-B2 backfill smoke runner (Codex 修正済 INSERT OR IGNORE 版) |

vault 側:
| commit | 内容 |
|---|---|
| 9fa91af | R0 feasibility report |
| d4f53a2 | R0 TODO |
| fc9784c | R0 milestone |
| d334eb4 | R1 plan v1.0 |
| 2e3f7ab | R1 TODO |
| 4992153 | R1 milestone |
| 本 commit | R1-B3 smoke result Vault 化 |

## 10. HQ 判断要請

Q1: **schema 対応案 X2 採用承認**
   staging market_financials の PK を `(code, disclosure_date,
   type_of_document)` に migration する案で OK か?
   (推奨、本 vault §6.4 提示)

Q2: **R1-B2.5 着手判断**
   X2 承認後、別 commit (R1-B2.5) で:
   1. scripts/setup/migrate_market_financials_pk.py
   2. backfill_market_financials.py の UPSERT 戦略変更
   3. 5 銘柄 + 100 mini 再 smoke + 結果 vault 化
   この順序で進めて OK か?

Q3: **type_of_document NULL record の扱い**
   migration 時に既存 row で type_of_document が NULL の場合、
   - 案: "Unknown" 等で埋める / NOT NULL 緩和 / migration で skip
   どの方針か?

Q4: **R1-B4 full backfill 着手条件 (§7) で OK か**
   schema 対応 + 再 smoke 完了後の進行で問題ないか?

Q5: **その他補正あれば指示**

## 11. 改訂履歴

- v1.0 (2026-05-09): 初版、R1-A2 + R1-B2 smoke 結果 Vault 化、
  schema 差異 (HQ 案 A 想定 vs 実 PK code,disc_date) を発見、
  Codex CRITICAL (INSERT OR REPLACE record loss) 修正経緯記録、
  X1/X2/X3 比較で X2 採用推奨、R1-B4 full backfill は schema 対応 +
  再 smoke 後まで保留、HQ 判断 5 項目要請。
