---
id: F286-R2-A2
phase: P5: Research Lane R2 (eligible universe + 派生指標 storage)
priority: 高 (R2 派生指標を再現性 / 集計可能な形で保存する基盤)
status: 完了 (5 銘柄 / 100 銘柄 mini smoke、HQ 必須条件 全クリア、no_fy_record 58 → 0 削減)
owner: Fujiwara
depends_on: [F286-R2-A1 derived indicators MVP]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R2_A1_derived_indicators_smoke, F286_R1_B4_market_financials_full_backfill
---

★ **F286 R2-A2**: Research Lane の (1) eligible universe filter
   (普通株 / 事業会社中心) + (2) `research_derived_indicators` 保存先
   table + (3) eligible 通過 → 計算 → 永続化 する smoke runner。
   R2-A1 で 58/100 だった no_fy_record が **0/42 に削減**。

# F286-R2-A2 eligible universe + indicator storage

## サマリ

R2-A1 で発見した「100 銘柄 mini smoke の 58/100 が ETF / index ファンド
由来 no_fy_record」問題に対処。market_listings の `market_code` /
`sector_17_code` を使って **普通株 / 事業会社中心** の universe を
分離し、派生指標を `research_derived_indicators` table に永続化する。

3 つの新モジュール構成:

1. **`simulation/research_lane/eligible_universe.py`** (純粋関数)
2. **`scripts/setup/migrate_research_derived_indicators.py`** (migration)
3. **`scripts/jobs/persist_derived_indicators.py`** (smoke runner)

## eligible filter ロジック

| market_code | 件数 | 扱い |
|---|---:|---|
| 0111 (プライム) | 1,575 | **eligible** |
| 0112 (スタンダード) | 1,580 | **eligible** (sector_17="99" の 1 件のみ除外) |
| 0113 (グロース) | 598 | **eligible** |
| 0109 (その他) | 517 | excluded: `non_stock_market_other` (ETF/REIT 等) |
| 0105 (TOKYO PRO MARKET) | 179 | excluded: `tokyo_pro_market` |

合計 4,449 → eligible **3,752** / excluded **697** (推定)。

判定優先順位:
1. `market_code` が EXCLUDED (0109 / 0105) → 即除外
2. `market_code` が ELIGIBLE 集合 (0111/0112/0113) でない → `unknown_market_code`
3. `sector_17_code == "99"` → `other_sector` (普通株セグメント混入の非事業)
4. `sector_17_code` 空 / None → `missing_sector`
5. それ以外 → eligible

## research_derived_indicators schema

```sql
CREATE TABLE research_derived_indicators (
    code             TEXT NOT NULL,
    base_date        TEXT NOT NULL,  -- = price_date / 計算基準日
    disclosure_date  TEXT NOT NULL,
    type_of_document TEXT NOT NULL,
    fiscal_year_end  TEXT,
    close_price      REAL,
    per              REAL, pbr REAL, roe REAL,
    operating_margin REAL, net_margin REAL,
    sales_growth_yoy REAL, profit_growth_yoy REAL,
    skip_reasons_json TEXT,  -- {"per": "negative_eps", ...}
    payload_json     TEXT,   -- 入力 snapshot (eps/bps/sales 等)
    created_at       TEXT NOT NULL,
    PRIMARY KEY (code, base_date, disclosure_date, type_of_document)
);
CREATE INDEX idx_research_derived_indicators_code_base
  ON research_derived_indicators(code, base_date);
CREATE INDEX idx_research_derived_indicators_base_date
  ON research_derived_indicators(base_date);
```

PK 4-key の意味:
- 同 code でも別 base_date (= 別 price_date) で再計算可能
- 同 (code, base_date) でも別 disclosure_date (= 期次違い) で並列保存
- 同 disclosure_date でも別 doc_type で独立 (= R1-B2.5 v2 schema 整合)

## 実行結果 — migration

```
target_table:        research_derived_indicators
table_existed_before: False
row_count: 0 -> 0
schema_verified: True
rollback_strategy: DROP TABLE research_derived_indicators
```

冪等性検証: 再 run = no-op で確認済 (test 経由)。

## 実行結果 — 5 銘柄 smoke

| 項目 | 値 |
|---|---|
| eligible_count | 5 |
| excluded_count | 0 |
| calculated_count | 5 |
| no_fy_record_count | 0 |
| upsert: inserted | 5 |
| upsert: replaced | 0 |
| row_count | 0 → 5 (+5) |

7 指標 coverage:
| indicator | computed | skip 理由 |
|---|---:|---|
| PER | 4/5 | negative_eps × 1 (Sony) |
| PBR | 5/5 | — |
| ROE | 5/5 | — |
| operating_margin | 4/5 | missing × 1 (MUFG 銀行) |
| net_margin | 5/5 | — |
| sales_growth_yoy | 5/5 | — |
| profit_growth_yoy | 5/5 | — |

R2-A1 と同じ skip パターン、storage に正しく永続化された。

## 実行結果 — 100 銘柄 mini smoke (★ 主要結果)

| 項目 | 値 | R2-A1 比較 |
|---|---:|---|
| 候補 listings | 100 | 100 |
| eligible_count | **42** | (フィルタなし、100) |
| excluded_count | 58 | — |
| excluded reasons | non_stock_market_other 49 / tokyo_pro_market 9 | — |
| calculated_count | 42 | 42 |
| **no_fy_record_count** | **0** | **58** ← 大幅改善 |
| upsert: inserted | 42 | — (storage なし) |
| row_count | 5 → 47 (+42) | — |

7 指標 coverage (eligible 42 銘柄ベース):

| indicator | computed | skip rate | skip 理由 |
|---|---:|---:|---|
| PER | 38/42 | 9.5% | missing × 1 / negative_eps × 3 |
| PBR | 41/42 | 2.4% | missing × 1 |
| **ROE** | **42/42** | **0%** | — |
| operating_margin | 41/42 | 2.4% | missing × 1 (銀行系) |
| net_margin | 41/42 | 2.4% | missing × 1 |
| sales_growth_yoy | 41/42 | 2.4% | missing × 1 (前期 data なし) |
| profit_growth_yoy | 42/42 | 0% | — |

**観察**: eligible filter で no_fy_record (R2-A1 では 58 件) が **0 に
削減**。計算成功率も大幅向上 (PER 38% → 90.5%、ROE 42% → 100%)。
残りの skip は赤字 EPS / 銀行 (operating_profit 未掲載) 等の **正当な
data 特性に起因するもののみ**。

## DB 書き込み確認

```
data/fire.db (production):    May 7 16:12:38  371,064,832 bytes (無触)
data/fire.develop.db:         May 7 18:14:26  371,064,832 bytes (無触)
data/fire.staging.db:         May 9 19:06:42  4,551,925,760 bytes (R2-A2 で +47 row)
```

production / develop DB last_modified May 7、smoke 完全無触。
staging DB は research_derived_indicators table のみ追加 / 47 row 書込。

## skip_reasons_json 永続化サンプル

```
67580 → {"per": "negative_eps"}
83060 → {"operating_margin": "missing"}
130A0 → {"per": "negative_eps", "operating_margin": "missing",
         "net_margin": "missing", "sales_growth_yoy": "missing"}
13320 → {"per": "missing", "pbr": "missing"}
```

各銘柄の skip 理由が JSON で永続化されており、後続のファクター
計算 / 集計時に参照可能。

## constraints_check

```json
{
  "staging_only_write": true,
  "production_db_untouched": true,
  "develop_db_untouched": true,
  "v2_schema_used": true,
  "indicators_table_used": true,
  "tier2_full_executed": false
}
```

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/eligible_universe | 22 PASS |
| scripts/setup/migrate_research_derived_indicators | 15 PASS |
| scripts/jobs/persist_derived_indicators | 19 PASS |
| **R2-A2 trio 合計** | **56 PASS** |
| 関連 regression (research_lane / scripts) | 435 PASS |

## commit ログ

| # | commit | hash |
|---|---|---|
| R2-A2-1 | feat(F286-R2): add research eligible universe filter | ef9adba |
| R2-A2-2 | chore(F286-R2): add derived indicators storage migration | 9885947 |
| R2-A2-3 | chore(F286-R2): add derived indicators storage smoke runner | 1c15fd3 |
| R2-A2-4 | docs(F286-R2): vault eligible universe and indicator storage smoke result | (本 commit) |
| R2-A2-5 | docs(F286-R2): log milestone — derived indicator storage smoke completed | (次 commit) |

## 次のアクション (HQ 判断)

R2-A2 完了 → R2 後続選択肢:

1. **R2-A3 Tier2 全件 indicators 計算 + persist**: 4,449 銘柄 →
   eligible ~3,752 → research_derived_indicators 永続化、HQ 別承認後
2. **R2-B 7 指標の分布検証**: eligible 全件で per / pbr / roe 等の
   distribution / outlier / sector 別分布を確認、Lane / strategy
   別の閾値定義の前提
3. **R2-C ファクター戦略実装**: PER + ROE で Quality Value Screener、
   Sales/Profit YoY で Growth Screener など (F285 R2 仕様)
4. **R1-B5 v1/v2 swap**: 既存 v1 (1,723 row) を drop / rename して
   v2 を正本化、別途 HQ 承認後

## 関連リンク

- [[F286_R2_A1_derived_indicators_smoke|F286-R2-A1 派生指標 MVP]]
- [[F286_R1_B4_market_financials_full_backfill|F286-R1-B4 v2 full backfill]]
- [[F286_R1_Sector_Flow_and_financials_backfill|F286-R1 親タスク]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ migration / 5 銘柄 / 100 銘柄 mini 全て exit 0
- **機能した**: ✅ eligible filter で no_fy_record 58 → 0 (削減 100%)、
  ROE coverage 42% → 100%、47 row が research_derived_indicators
  に永続化、skip_reasons_json で skip 理由が追跡可能
- **期待値達成**: ✅ HQ 必須条件 (production/develop DB 無触 /
  staging only / market_financials_v2 明示参照 / Codex pre-commit
  通過 × 3) 全クリア、tests 56 PASS / regression 435 PASS
