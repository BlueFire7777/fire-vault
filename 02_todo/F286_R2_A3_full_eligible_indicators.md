---
id: F286-R2-A3
phase: P5: Research Lane R2 (eligible universe 全件 indicators)
priority: 高 (R2-B 分布検証 / R2-C ファクター戦略の前提 data)
status: 完了 (3,708 row 永続化、HQ 必須条件 全クリア)
owner: Fujiwara
depends_on: [F286-R2-A2 eligible universe + storage]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R2_A2_eligible_universe_and_indicators_storage, F286_R2_A1_derived_indicators_smoke
---

★ **F286 R2-A3**: Research eligible universe 全件 (3,752 銘柄) で
   派生指標 7 種を計算し、`research_derived_indicators` に **3,708 row
   を 1 run で永続化** (no_fy 44 は eligible 残存)。HQ 必須条件 4 項目
   (failed=0 / duplicate=0 / production・develop DB 無触 / DB size
   想定通り) 全クリア。

# F286-R2-A3 Research eligible universe 全件 派生指標 永続化

## サマリ

R2-A2 で確認した eligible filter (3,752 銘柄、Tier2 4,449 から
ETF/REIT/PRO 697 件除外) を全件適用、market_financials_v2 +
market_prices_daily を read-only で結合し、7 指標を 1 run で
research_derived_indicators に upsert。R2-B (分布検証) / R2-C
(ファクター戦略) で前提となる Research Lane の **唯一の正本データ**。

★ 「Research eligible universe 全件」≠ Lane C Tier2 478 銘柄。前者は
   Prime/Standard/Growth の事業会社全体 (3,752)、後者は Lane C の
   別概念 (478)。本タスクで作るのは前者。

## preflight 結果 (2026-05-09 19:08)

| 項目 | 値 |
|---|---|
| eligible_count | 3,752 |
| excluded_count | 697 |
| ・non_stock_market_other (0109) | 517 |
| ・tokyo_pro_market (0105) | 179 |
| ・other_sector (sector_17="99" in 0112) | 1 |
| row_count_current | 47 (R2-A2 smoke 残) |
| expected_row_count_after (lower / upper) | 47 / 3,799 |
| db_size_mb | 4,341.05 |
| disk_free_mb | 857,880.21 (≈ 838 GB) |
| estimated_elapsed | 1.87 min |

数値は R2-A2 推定 (3,752 eligible / 697 excluded) と完全一致 → HQ
事前承認に従い実行。

## 実行結果 (2026-05-09 19:17)

| 項目 | 値 |
|---|---|
| started | 2026-05-09T19:15:14 |
| ended | 2026-05-09T19:17:12 |
| elapsed | ~120 sec (preflight 推定通り) |
| target_db | data/fire.staging.db |
| target_table | research_derived_indicators |
| eligible_count | **3,752** |
| excluded_count | 697 |
| **processed_count** | **3,752** |
| **calculated_count** | **3,708** |
| no_fy_record_count | 44 |
| **failed_code_count** | **0** |
| inserted (新規) | 3,661 |
| replaced (R2-A2 47 row 上書き) | 47 |
| **duplicate_key_count** | **0** |
| row_count_before | 47 |
| row_count_after | **3,708** |
| db_size_before | 4,341.05 MB |
| db_size_after | 4,343.32 MB |
| db_size 増分 | **+2.26 MB** |

## HQ 必須条件 全クリア

| 条件 | 結果 |
|---|---|
| failed_codes = 0 | ✅ 0 |
| duplicate_key_count = 0 | ✅ 0 |
| no_fy_record 想定範囲内 (= 1.2%) | ✅ 44/3,752 = 1.17% |
| DB size 想定範囲内 (~50-100 MB 推定) | ✅ +2.26 MB (= 想定より小さい) |
| eligible_count 想定通り | ✅ 3,752 完全一致 |
| production DB 無触 | ✅ last_modified May 7 |
| develop DB 無触 | ✅ last_modified May 7 |
| 既存 v1 / v2 schema 無破壊 | ✅ 両 table 維持 |

## 7 指標 coverage (eligible 3,708 ベース)

| indicator | computed | rate | 主な skip 理由 |
|---|---:|---:|---|
| **ROE** | 3,699 | **99.8%** | missing 2 / negative_equity 7 |
| net_margin | 3,694 | 99.6% | missing 13 / zero_denominator 1 |
| profit_growth_yoy | 3,645 | 98.3% | missing 60 / zero_denominator 3 |
| PBR | 3,644 | 98.3% | missing 44 / negative_bps 20 |
| sales_growth_yoy | 3,636 | 98.1% | missing 71 / zero_denominator 1 |
| operating_margin | 3,623 | 97.7% | missing 84 / zero_denominator 1 |
| **PER** | 3,225 | **87.0%** | missing 40 / **negative_eps 443** |

**観察**:
- ROE 99.8% は驚異的高 coverage (= profit / equity の有る銘柄が
  ほぼ全体)
- PER の skip は **赤字 EPS 443 件 (12%)** が支配的。これは赤字決算
  銘柄が多く、PER 計算不能 → R2-C ファクター戦略では「赤字フラグ」
  として活用可能
- negative_bps 20 件 / negative_equity 7 件は **債務超過候補**、
  R2 Quality Screener で除外対象になる
- skip rate 全体的に低く、indicator 別に skip 理由が data 特性 (赤字
  / 債務超過 / 銀行業 / 空売上等) と一致 → ロジック健全

## 数値分布 (computed 銘柄、参考)

| indicator | n | min | median | max |
|---|---:|---:|---:|---:|
| PER | 3,225 | 0.81 | 14.54 | 1,514.86 |
| ROE | 3,699 | -138.33 | 0.076 | 1.06 |

PER 中央値 14.54、ROE 中央値 7.6% は東証スタンダード以上の事業会社
分布として妥当。max PER 1,515 は超高 PER 銘柄 (= 利益が極小 + 高株価)、
min ROE -138 は債務超過に近い赤字銘柄 (= 例外値、後続 R2-B 分布検証
での outlier 整理候補)。

## constraints_check (raw)

```json
{
  "staging_only_write": true,
  "production_db_untouched": true,
  "develop_db_untouched": true,
  "v2_schema_used": true,
  "indicators_table_used": true,
  "duplicate_keys_clean": true,
  "no_failed_codes": true,
  "full_eligible_executed": true
}
```

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/eligible_universe | 22 PASS |
| simulation/research_lane/derived_indicators | 54 PASS |
| simulation/research_lane/financials_mapping | 58 PASS |
| scripts/setup/migrate_research_derived_indicators | 15 PASS |
| scripts/jobs/persist_derived_indicators | 26 PASS (= R2-A2 19 + R2-A3 7) |
| **R2 関連 trio 合計** | **175 PASS** |
| 関連 regression (research_lane / scripts) | 442 PASS |

## commit ログ

| # | commit | hash |
|---|---|---|
| R2-A3-1 | chore(F286-R2): run full eligible derived indicators | 80386c0 |
| R2-A3-2 | docs(F286-R2): vault full eligible derived indicators result | (本 commit) |
| R2-A3-3 | docs(F286-R2): log milestone — full eligible indicators completed | (次 commit) |

## 性能・安定性メモ

- 推定 1.87 min → 実測 ~120 sec (preflight 通り、誤差 ~5%)
- DB-only 計算 (API 呼出なし) なので安定、rate limit / network 由来
  の retry は無関係
- pagination もなし (DB query は 1 銘柄あたり 2 SELECT + 1 UPSERT)
- failed_code_count = 0 (= sqlite3.Error 例外も発生せず、PK 4-key の
  upsert が全銘柄で成功)
- DB size 増分 +2.26 MB は preflight 推定 (~50-100 MB) より遥かに
  小さい。理由: payload_json が R1-B4 のような raw 107 fields を
  保存するのではなく、計算用 snapshot (eps/bps/sales/op/profit/
  equity 等の数値のみ) なので **1 row ~600 byte** に収まる

## 異常停止条件 (発火なし)

| 条件 | 観測値 | 判定 |
|---|---|---|
| no_fy_record が想定以上 | 44/3,752 = 1.17% | OK (想定 1-3% 範囲内) |
| duplicate_key_count > 0 | 0 | OK |
| failed_codes > 0 | 0 | OK |
| DB size 急増 | +2.26 MB | OK (想定下限以下) |
| eligible_count 想定外ズレ | 3,752 (推定 3,752) | OK |
| production / develop DB 触 | last_modified May 7 | OK |

異常停止条件は **1 件も発火せず**、stop して HQ 確認すべき事象なし。

## 次のアクション (HQ 判断)

R2-A3 完了 → R2 後続:

1. **R2-B 7 指標分布検証 (Tier2 = eligible 3,708)**
   - sector 別 distribution / outlier 整理
   - 業種別中央値 / 範囲 / skip 率
   - 異常値 (PER 1,515 等) の調査 → 計算ロジック保護
2. **R2-C ファクター戦略実装 (F285 R2 仕様)**
   - Quality Value: 高 ROE × 低 PER × 低 PBR
   - Earnings Growth: 高 sales_growth × 高 profit_growth
   - Cyclical Value: sector 別 PBR 反転 戦略
3. **R1-B5 v1 / v2 swap (別承認)**: 既存 v1 (1,723 row) を drop /
   rename して v2 を正本化、market_financials_v2 名前を market_financials
   に rename

## 関連リンク

- [[F286_R2_A2_eligible_universe_and_indicators_storage|F286-R2-A2 eligible filter + storage]]
- [[F286_R2_A1_derived_indicators_smoke|F286-R2-A1 派生指標 MVP]]
- [[F286_R1_B4_market_financials_full_backfill|F286-R1-B4 v2 full backfill]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ exit 0、JSON 出力成功、3,708 row 永続化
- **機能した**: ✅ failed=0 / duplicate=0 / no_fy 1.17% (想定通り) /
  ROE 99.8% / PER 87.0% (赤字銘柄 12% を除けば妥当)、skip 理由が
  data 特性と一致
- **期待値達成**: ✅ HQ 必須条件 全クリア、production / develop DB
  完全無触、Codex pre-commit 通過、tests 175 PASS / regression 442 PASS、
  DB size 増分 +2.26 MB は想定より小さく余裕
