---
id: F286-R2-F4
phase: P5: Research Lane R2 (broader historical sampling、22 base_dates 検証)
priority: 最高 (R2-D2 5 sample からの統計的 power 確保)
status: 完了 (22 base_dates × 3 presets、★ baseline が最強と判明、R2-D2 risk_adjusted は broader sample で消える)
owner: Fujiwara
depends_on: [F286-R2-D2 watchlist tuning, F286-R2-F3 leak-safe backtest]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R2_D2_watchlist_tuning, F286_R2_F3_leak_safe_backtest
---

★ **F286 R2-F4**: J-Quants から過去 18 ヶ月分の price を staging に
   backfill (155 万行) し、22 base_dates × 3 preset で leak-safe
   broader sampling を実施。

   ★ **重大発見**: R2-D2 (5 base) で見えた risk_adjusted の優位性は
   **broader sampling で消失**。逆に **baseline が 22 base 平均で
   20d top10 +3.07% / win 50.8%** と最強。R2-D2 で見えた baseline の
   弱は 5 sample の **サンプリングバイアス** だった。

# F286-R2-F4 Broader Historical Sampling

## サマリ

R2-D2 で risk_adjusted / quality_defensive を「採用候補」と判定したが、
sample 数 5 で統計的 power 不足。R2-F4 では:
1. J-Quants から **過去 18 ヶ月分 price を backfill** (= staging のみ)
2. 月次 22 base_dates で baseline / risk_adjusted / quality_defensive
   を leak-safe で並列保存
3. 22 base 平均で preset 比較

★ 結論: **baseline が最強**。R2-D2 の risk_adjusted 採用候補は
   サンプリングバイアスだった。

実装 3 module + tests:
1. **`simulation/research_lane/price_coverage.py`** (純粋関数)
2. **`scripts/jobs/run_research_price_backfill_for_backtest.py`** (price wrapper)
3. **`scripts/jobs/run_research_broader_historical_sampling.py`** (orchestration)

## price coverage (実施結果)

### Backfill 実施

| 項目 | 値 |
|---|---:|
| target date range | 2024-05-01 〜 2025-11-03 (= 18 ヶ月 missing) |
| target codes | 4,449 |
| inserted_rows | **1,554,067** |
| failed_codes | **0** |
| elapsed | 2,464 sec (= 41 min) |

### Coverage before/after

| | min_date | max_date | total_rows |
|---|---|---|---:|
| **before** | 2025-11-04 | 2026-05-01 | 526,764 |
| **after** | 2024-05-01 | 2026-05-01 | **2,080,831** |

→ J-Quants V2 / Standard plan で過去 18 ヶ月 (= 4,449 codes × ~360
   trading days) を 41 分で取得、429 rate limit 数回ヒットも retry
   で全件成功。

## base_date 選定

- 月次 first day of month: 2024-06-01 〜 2026-03-01 → 22 件
- 全 22 件 valid (= base_close + 20d future price 取れる)
- selection rejected: 0

## preset (R2-F4 source_version)

| preset | source_version | weights |
|---|---|---|
| baseline | r2f4_baseline_v1 | QV/EG/CV 0.35/0.35/0.30 |
| risk_adjusted | r2f4_risk_adjusted_v1 | QV/EG/CV/risk 0.35/0.30/0.25/0.10 |
| quality_defensive | r2f4_quality_defensive_v1 | QV/EG/CV 0.50/0.25/0.25 |

momentum 系 preset は R2-D2 で全悪化と判明済のため、R2-F4 では
HQ 推奨通り **3 preset に絞る** (= 余計な並列 run を避ける)。

## leak-safe 確認

| 項目 | 結果 |
|---|---|
| max_disclosure_date_used <= base_date | ✅ 全 66 combinations |
| max_price_date_used_for_signal <= base_date | ✅ 全 66 combinations |
| leak violations | **0** |
| future return usage in signal | **なし** |
| research_derived_indicators write | **0 件** |

## 実行結果 (22 base_dates × 3 preset = 66 combinations)

failures: **0**
全 base_date / 全 horizon で valid_return_count = 100/100 (= 100%
価格取得成功)。

### ★ 主要結果 — preset 比較 22 base_dates 平均

#### 1d return

| preset | overall | top10 | top30 | top10 win | drawdown |
|---|---:|---:|---:|---:|---:|
| baseline | -0.31% | -0.34% | -0.33% | 38.0% | -1.01% |
| **risk_adjusted** | **-0.15%** | **+0.05%** | **+0.01%** | **48.8%** | **-0.62%** |
| quality_defensive | -0.30% | -0.38% | -0.29% | 37.7% | -1.00% |

→ **risk_adjusted が 1d で明確に強い**: top10 win 48.8% (baseline
  38%、+10.8 ポイント)、drawdown 最浅。

#### 5d return

| preset | overall | top10 | top30 | top10 win | drawdown |
|---|---:|---:|---:|---:|---:|
| **baseline** | +0.06% | **+0.05%** | +0.15% | **46.5%** | -3.35% |
| risk_adjusted | -0.09% | -0.08% | -0.18% | 45.1% | -2.16% |
| quality_defensive | +0.02% | -0.44% | +0.12% | 45.5% | -3.33% |

→ 5d では **baseline が top10 mean +0.05% / win 46.5% で最強**。
  risk_adjusted は drawdown 改善 (-2.16% vs -3.35%) だが return / win
  は劣後。

#### 20d return (★ 最重要)

| preset | overall | top10 | top30 | top10 win | drawdown |
|---|---:|---:|---:|---:|---:|
| **baseline** | **+2.57%** | **+3.07%** | **+3.08%** | **50.8%** | -5.33% |
| risk_adjusted | +1.27% | -0.35% | -0.27% | 40.2% | -3.57% |
| quality_defensive | +2.42% | +2.47% | +3.02% | 50.2% | -5.32% |

→ **baseline が 20d で圧倒的に最強**: top10 +3.07% / 勝率 50.8% / top30
  +3.08% / 勝率 53.5% (実用 edge 候補水準)。
  risk_adjusted は **20d で大幅劣化** (top10 -0.35% / win 40.2%)。

### 月別 detail (一部、20d top10 mean)

| base_date | baseline 20d top10 | risk_adjusted | quality_defensive |
|---|---:|---:|---:|
| 2024-06-03 | +6.54% | +0.35% | +5.93% |
| 2024-09-02 | +5.62% | -2.13% | +4.07% |
| 2024-12-02 | -1.32% | -7.93% | -2.53% |
| 2025-04-01 | +4.21% | -0.13% | +4.40% |
| 2025-08-01 | +5.70% | +1.78% | +5.33% |
| 2025-12-01 | +5.43% | +2.42% | +4.71% |
| 2026-02-15 | (= R2-D2 同期間) | (= R2-D2 同期間) | (= R2-D2 同期間) |
| 2026-03-01 | -3.91% | -1.42% | -3.50% |

(全 22 件は CSV に保存)

improvement months (vs baseline) for risk_adjusted 20d top10:
- 改善: 6 件
- 悪化: 16 件
→ **明確に baseline 劣後**

improvement months for quality_defensive 20d top10:
- 改善: 7 件
- 悪化: 15 件
→ baseline ほぼ同等、わずかに弱い

## R2-D2 vs R2-F4 比較 (★ サンプルサイズの影響)

| 指標 | R2-D2 (5 base) | R2-F4 (22 base) | 解釈 |
|---|---|---|---|
| baseline 20d top10 | +1.23% | **+3.07%** | R2-D2 は弱い期間 (2025-11/2026-01) を多く含むサンプリングバイアス |
| risk_adjusted 5d top10 win | 44.2% | 45.1% | broader sample で baseline 同等に近づく |
| risk_adjusted 5d drawdown | -1.94% | -2.16% | broader sample でわずかに深い |
| risk_adjusted 20d top10 | (R2-D2: top10 -1.42%) | -0.35% | broader sample で baseline から -3.42% 劣後 |
| quality_defensive 20d overall | +1.32% | +2.42% | 全期間平均で改善 (が baseline 比はほぼ同等) |

→ **R2-D2 で見えた risk_adjusted の優位性 (5d win rate 44.2% など) は
   5 base sample のバイアス**。22 base では消える。

## 過剰最適化リスク

✅ **R2-F4 で過剰最適化が露呈**:
- R2-D2 (5 sample) で risk_adjusted を採用候補と判断 → broader 検証
  で消失
- 同様に quality_defensive も baseline ほぼ同等
- **5 sample での tuning 判断は信頼できない**、最低 20 sample 以上
  必要が再確認

## constraints_check

```json
{
  "leak_safe_signal_generation": true,
  "indicators_table_write_avoided": true,
  "prices_table_write_avoided": false,
  "staging_only_write": true,
  "production_db_untouched": true,
  "develop_db_untouched": true
}
```

注: prices_table_write_avoided=false は **意図的** (= R2-F4 で
staging market_prices_daily に過去 18 ヶ月分の追加が必要だったため)。
他 DB は完全無触。

## DB 安全性

| DB | last_modified | 状態 |
|---|---|---|
| production (`fire.db`) | 2026-05-07 16:12 | 完全無触 |
| develop (`fire.develop.db`) | 2026-05-07 18:14 | 完全無触 |
| staging (`fire.staging.db`) | 2026-05-09 22:40 | market_prices_daily +1,554,067 / research_watchlist_signals +8,390 |

`research_watchlist_signals` 内訳 (全 source_version):
| source_version | rows |
|---|---:|
| r2d_v1 | 109 |
| r2f2_v1 | 327 |
| r2f3_leaksafe_v1 | 1,130 |
| r2d2_baseline_v1 | 547 |
| r2d2_quality_defensive_v1 | 541 |
| r2d2_growth_momentum_v1 | 589 |
| r2d2_cyclical_rebound_v1 | 580 |
| r2d2_risk_adjusted_v1 | 744 |
| r2d2_balanced_momentum_v1 | 594 |
| **r2f4_baseline_v1** | **2,417** |
| **r2f4_risk_adjusted_v1** | **3,544** |
| **r2f4_quality_defensive_v1** | **2,429** |
| **合計** | **13,551** |

write 対象 tables:
- ✅ market_prices_daily (= R2-F4 専用、過去 18 ヶ月の backfill)
- ✅ research_watchlist_signals (= 各 preset の signal、+8,390 row)
- ✅ research_derived_indicators 完全無触 (= in-memory 生成のみ)

write guard:
- ✅ production / develop --write-prices 即拒否
- ✅ production / develop --write-signals 即拒否

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/price_coverage | 19 PASS |
| scripts/jobs/run_research_price_backfill_for_backtest | 18 PASS |
| scripts/jobs/run_research_broader_historical_sampling | 16 PASS |
| **R2-F4 合計** | **53 PASS** |
| 関連 regression | **929 PASS** |

**HQ 必須テスト 全 17 項目クリア**:
- price coverage summary / base_date sampler / 20 営業日後判定 /
  price write dry-run / production・develop price write 拒否 /
  staging price write / preset source_version mapping / leak check /
  dry-run signal no write / staging signal write / multi-date ×
  preset loop / comparison summary aggregation / improvement vs
  baseline / output artifact / partial missing / continue-on-error /
  runner smoke

## commit ログ

| # | commit | hash |
|---|---|---|
| R2-F4-1 | feat(F286-R2): add research price coverage and base-date sampling | 6a49e17 |
| R2-F4-2 | feat(F286-R2): add staging price backfill for research backtests | 076f569 |
| R2-F4-3 | feat(F286-R2): add broader historical sampling runner | ed13258 |
| R2-F4-4 | test(F286-R2): add broader historical sampling tests | 7f193b9 |
| R2-F4-5 | docs(F286-R2): vault broader historical sampling result | (本 commit) |
| R2-F4-6 | docs(F286-R2): log milestone — broader historical sampling completed | (次 commit) |

Codex pre-commit 通過 × 4、CRITICAL 1 件発生 → 即修正:
- (price backfill wrapper) HistoricalDataFetcher / get_target_symbols
  に db_path=db_path を明示 (= import 時 default capture を防ぐ)
--no-verify 不使用、個別 commit 厳守。

## Go/No-Go 判定

| 条件 | 結果 |
|---|---|
| **framework PASS** | ✅ price backfill 成功 (155 万 row、0 failed) / 22 base × 3 preset = 66 combinations 全成功 |
| broader sampling として十分 | ✅ 22 valid base_dates、R2-D2 (5 base) からの大幅増 |
| leak violations 0 | ✅ |
| Codex CRITICAL 全対応 | ✅ |
| **risk_adjusted 残ったか** | ❌ **broader sample で baseline 大幅劣後** (20d -3.42%) |
| **quality_defensive 残ったか** | ❌ baseline ほぼ同等、明確な改善なし |
| **baseline で十分か** | ✅ **baseline が 20d top10 +3.07% / win 50.8% で実用 edge 候補水準** |
| 採用判断 | **★ baseline 採用、tuning preset 全不採用** |

## 次のアクション (HQ 判断)

R2-F4 で **R2-D2 の tuning preset は全て不採用** という決定的結果。
次の優先順位:

1. **R2-G market regime / sector flow integration** ← 最優先
   - baseline で十分 edge を持つことが確認された (= 20d top10
     +3.07% / win 50.8%)
   - 次はその edge を **regime-aware** に強化する方向 (= 上昇期 /
     下落期 / 反発期で signal の効果が異なる、F286-R1 Sector Flow
     Agent と連携可能)
2. **Lane integration** (Daytrade/Swing/Long-term Selection)
   - baseline output (top30、20d edge +3.07%) を Fujiwara が手動
     レビューして Daytrade Selection に入力する運用
3. **R2-D3 fine tuning は保留**
   - R2-D2 で見えた risk_adjusted は broader で消失したため、
     重み微調整の前に regime / sector などの **次元追加** が優先
4. **R1-B5 v1/v2 swap** (別承認)

## 関連リンク

- [[F286_R2_D2_watchlist_tuning|F286-R2-D2 tuning (5 base)]]
- [[F286_R2_F3_leak_safe_backtest|F286-R2-F3 leak-safe (10 base)]]
- [[F286_R2_F2_multi_date_evaluation|F286-R2-F2 multi-date (3 base、leak あり)]]
- [[F286_R2_D_watchlist_ranker|F286-R2-D Watchlist Ranker]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ price backfill 成功 (1.55M row、0 failed)、broader
  sampling 66/66 combinations 成功、artifact 全出力
- **機能した**: ✅ leak violations 0、22 base_dates 全 valid、
  preset_aggregate / vs_baseline / improvement_months 集計が機能
- **期待値達成**: ✅ HQ 必須要件 17 項目 全 PASS、★ R2-D2 の
  サンプリングバイアスを露呈し **baseline 採用** に decision を
  pivot、production/develop 完全無触、tests 53 PASS / regression
  929 PASS、Codex pre-commit 通過 × 4、staging のみで price + signal
  両方を controlled に追加
