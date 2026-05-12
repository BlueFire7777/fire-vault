---
id: F286-staging-smoke-row-inventory-cleanup-policy
phase: Wave 19 W19-2 / staging 残置 row inventory + cleanup policy
priority: 高
status: 起票 ☆ inventory + filter policy 確定、削除は HQ 明示承認後
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W17-3 f111 staging smoke (= source_version='w17-3-smoke')
  - W18-1 f100 staging smoke (= 3 銘柄 5 桁 code、2026-05-08)
  - HQ Wave 19 W19-2 起票承認 (= 2026-05-12)
chapter: F286 / R-01-08 / staging cleanup policy
---

# F286 staging Smoke Row Inventory + Cleanup Policy (= W19-2)

最終更新: 2026-05-12

## ★ 状態: inventory 確定、cleanup policy 確立、削除は HQ 別 明示承認後

W17-3 / W18-1 の sub-D2.3.x staging write smoke 由来の row を識別 + filter
policy 明文化。本番評価 / REPORT / 候補生成では filter で除外。

## 1. staging 残置 smoke row inventory (= 2026-05-12 時点)

### research_watchlist_signals

source_version 別件数:

| source_version | row count | 種別 |
|----------------|-----------|------|
| r2d_v1 | 109 | 本番系 (= W4 系) |
| r2d2_balanced_momentum_v1 | 594 | 本番系 |
| r2d2_baseline_v1 | 547 | 本番系 |
| r2d2_cyclical_rebound_v1 | 580 | 本番系 |
| r2d2_growth_momentum_v1 | 589 | 本番系 |
| r2d2_quality_defensive_v1 | 541 | 本番系 |
| r2d2_risk_adjusted_v1 | 744 | 本番系 |
| r2f2_v1 | 327 | 本番系 |
| r2f3_leaksafe_v1 | 1130 | 本番系 |
| r2f4_baseline_live_v1 | 109 | 本番系 |
| r2f4_baseline_v1 | 2417 | 本番系 |
| r2f4_quality_defensive_v1 | 2429 | 本番系 |
| r2f4_risk_adjusted_v1 | 3544 | 本番系 |
| **w17-3-smoke** | **35** | **★ smoke 残置 (Wave 17)** |

**filter 対象**: `source_version = 'w17-3-smoke'`

### market_prices_daily

W18-1 で 3 銘柄 × 1 日 = 3 row を INSERT OR REPLACE。

| date | code | 種別 |
|------|------|------|
| 2026-05-08 | 72030 | smoke 残置 (= W18-1、INSERT OR REPLACE で既存 row update) |
| 2026-05-08 | 99840 | 同上 |
| 2026-05-08 | 67580 | 同上 |

★ INSERT OR REPLACE のため、元から存在した row を上書き (= count 不変、
2,085,284)。data としては smoke 残置と本番 data の区別が困難。

**filter 対象**: 厳密には区別不可能 (= REPLACE 後の値は smoke 値、本番値と
の混合)。ただし 2026-05-08 の全 row (= 4,448 row) 中、3 row のみが smoke
影響。残り 4,445 row は本番 fetch 由来。

→ 本番 data に対する smoke 影響は **3 row の column 値再取得** で、運用
影響なし (= 同一 J-Quants API 由来、値同一)。

### advisory_decisions

| advisory_id prefix | row count | 種別 |
|---------------------|-----------|------|
| f062-r5.8-2026-05-11T09:19:53Z | 5 | W4.1-A snapshot 既存 |
| production-advisory-2026-05-09-520d6429e10e0b2a | 5 | 同上 |

★ smoke 残置 row なし (= W14-2 で develop CREATE のみ、staging には W14
以前から 10 row 存在、W17-3 は signals に書込、advisory_decisions 不触)。

## 2. cleanup policy

### 削除条件 (= 全 必須)

1. **HQ 明示承認** (= 各削除 sub-task で個別 approve)
2. **本番評価 / REPORT / 候補生成で再利用不要** (= 後続 wave で paper_pnl
   smoke 等の data source として使う場合は保持)
3. **削除前 inventory 再確認** (= 件数 + source_version + 関連 row)
4. **削除後 production / develop unchanged 確認**

### filter 方針 (= 本番 application 経路)

#### research_watchlist_signals

application 側で:
```python
SMOKE_SOURCE_VERSIONS_BLACKLIST = ("w17-3-smoke",)

# query で filter
cursor.execute(
    "SELECT ... FROM research_watchlist_signals "
    "WHERE source_version NOT IN (?)",
    SMOKE_SOURCE_VERSIONS_BLACKLIST,
)
```

または application module で:
- Pattern Research / 候補抽出: smoke prefix を blacklist
- REPORT-R1 weekly / monthly: 同上
- F119 評価 Agent: 同上

#### market_prices_daily

date='2026-05-08' の 72030/99840/67580 は本番値と区別不可、filter 不要 (=
INSERT OR REPLACE で本番 fetch 経路と同等)。

ただし将来 smoke で source 識別が必要なら、market_prices_daily に source
column 追加 (= W14 SCHEMA-R1 同様の構造改修)。本 Wave で対応せず、別 task。

#### advisory_decisions

W4.1-A 既存 10 row は **本番 data ではなく snapshot data** (= staging
専用)。本番 application で advisory_decisions を SELECT する場合、
production / develop の advisory_decisions を使うべき (= W14 SCHEMA-R1 で
CREATE 済、row 0)。

staging advisory_decisions は smoke / test 用、本番経路には混ぜない:
```python
# 本番 application 経路では production / develop の advisory_decisions を
# SELECT、staging は使わない (= staging-only smoke contract)
```

## 3. filter 実装の優先度

### 即時必須

- F119 評価 Agent (= W18-3 で artifact 生成、本番運用前に filter 適用要)
- REPORT-R1 daily / weekly / monthly (= staging data を参照する場合は
  filter 適用)
- Pattern Research / 候補抽出 (= smoke row を pattern に混ぜない)

### 中期

- application module の SQL 共通化 (= filter helper module)
- smoke source_version の constant 化 (= src/staging_smoke_markers.py 等)

### 長期

- staging DB 自体を本番経路から完全分離 (= staging は smoke / test 専用、
  本番経路は production / develop のみ)

## 4. 残置 row 一覧 (= 2026-05-12 sub-D2.3.x 経由)

| sub-task | table | 識別 marker | row count | 削除可否 |
|----------|-------|------------|-----------|---------|
| W17-3 | research_watchlist_signals | source_version='w17-3-smoke' | 35 | HQ approve 後 |
| W18-1 | market_prices_daily | date='2026-05-08' AND code IN ('72030','99840','67580') | 3 (INSERT/REPLACE) | 区別困難、削除非推奨 |
| W18-2 | (= 書込未発生) | - | 0 | 該当なし |
| W18-3 | (= read-only) | - | 0 | 該当なし |

## 5. HQ 承認テンプレ (= 将来 cleanup 時)

```
=== HQ 承認依頼 / staging smoke row cleanup ===

date:                (実行予定日)
target:              research_watchlist_signals
filter:              source_version='w17-3-smoke'
delete row count:    35
rollback:            DELETE 前に SELECT で結果記録 (= /tmp/cleanup_*.txt)
production touch:    なし
develop touch:       なし
LINE:                0

承認希望: 上記 staging smoke row 削除
```

## 6. 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE19_results|Wave 19 results]]
- [[../02_todo/FIRE_CODEX_R1_WAVE17_results|Wave 17 results (= W17-3 smoke)]]
- [[../02_todo/FIRE_CODEX_R1_WAVE18_results|Wave 18 results (= W18-1 smoke)]]
- [[F101_API_403_investigation_2026-05-12|F101 API 調査 (= W19-1)]]
