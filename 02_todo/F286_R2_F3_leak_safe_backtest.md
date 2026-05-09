---
id: F286-R2-F3
phase: P5: Research Lane R2 (leak-safe historical backtest、true backtest 基盤)
priority: 最高 (R2-F2 の leak 排除 + R2-D2 tuning の前提)
status: 完了 (10 base_dates leak-safe 動作、R2-F2 比較で signal 優位性が大幅減退、score 設計見直し示唆)
owner: Fujiwara
depends_on: [F286-R2-F2 multi-date evaluation, F286-R2-F return evaluation]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R2_F2_multi_date_evaluation, F286_R2_F_return_evaluation, F286_R2_D_watchlist_ranker
---

★ **F286 R2-F3**: Leak-safe historical signal generation の **true
   backtest 基盤** が完成。各 base_date 時点の財務開示
   (disclosure_date <= base_date) のみを使って indicators を
   in-memory 再計算、leak violations 0 を確認。

   ★ **重要発見**: leak-safe にすると **R2-F2 (leak あり) の signal
   優位性が大幅減退**。R2-F2 で見えた top10 の優位は future
   information leak 由来の可能性が高い。R2-D2 tuning で score 設計
   見直しが必要。

# F286-R2-F3 Leak-safe Historical Signal Generation

## サマリ

R2-F2 の leak (indicators が 2026-05-01 base のみ) を排除するため、
各 base_date で **disclosure_date <= base_date** の財務開示のみを
使って indicators を **in-memory 再計算** する基盤を実装。

10 base_dates (2025-06 〜 2026-03) を smoke 実行、leak violations 0、
production / develop DB 完全無触、staging のみ書込み制御を確認。

実装 2 module + tests:
1. **`simulation/research_lane/historical_indicators.py`** (純粋関数)
2. **`scripts/jobs/run_research_leak_safe_backtest.py`** (orchestration runner)

## leak-safe 設計

### 必須条件 (HQ 仕様)

✅ disclosure_date <= base_date を必須 filter (`select_eligible_fy_record`
   で SQL レベル + `build_historical_record` で防御的 re-check)
✅ research_derived_indicators table (= 2026-05-01 base のみ) を **参照
   しない** (独自 SELECT で market_financials_v2 から fetch)
✅ derived_indicators table への **write しない** (in-memory 生成のみ)
✅ price 評価以外の signal 生成段階で future price を使わない
✅ leak violations 1 件で **即 LeakViolationError raise** (= --continue
   -on-error 対象外)

### Codex CRITICAL 4 件 全対応

| # | 内容 | 対応 |
|---|---|---|
| 1 | dry-run + run_return_evaluation = stale signals 評価 | dry-run では eval skip、warning print |
| 2 | --continue-on-error が LeakViolationError を握る | 必ず raise (継続対象外) |
| 3 | persistence failed でも eval を呼ぶ | stats.failed > 0 で LeakSafeRunRefused raise |
| 4 | 再実行で stale 行残存 | write 前に当該 (base_date, source_version) DELETE、deleted_stale_rows 記録 |

### record selection 優先順位

```
WHERE code=? AND disclosure_date <= ? AND type_of_document LIKE 'FY%'
ORDER BY disclosure_date DESC,
  CASE WHEN type_of_document LIKE '%NonConsolidated%' THEN 1 ELSE 0 END ASC,
  type_of_document ASC
LIMIT 1
```

→ disclosure_date 最新 → Consolidated_JP > NonConsolidated_JP

## 対象

- base_dates: **10 件** (HQ 推奨)
  - 2025-06-02, 2025-07-01, 2025-08-01, 2025-09-01, 2025-10-01
  - 2025-11-04, 2025-12-01, 2026-01-15, 2026-02-15, 2026-03-16
- source_version: **r2f3_leaksafe_v1**
- top_n: 100
- horizons: 1, 5, 20

### data availability

- market_prices_daily 範囲: **2025-11-04 〜 2026-05-01**
- market_financials_v2 範囲: 2016-05-09 〜 2026-05-08

→ **2025-06 〜 2025-10 (5 件)** の base_date は base_close 取得不能で
   return 評価不能 (= price 不足 5 件 / 評価可能 5 件)

## 実行結果

### Persistence stats (10 件、全成功)

| base_date | indicators_total | with_fy_record | leak_violations | signals_generated | inserted |
|---|---:|---:|---:|---:|---:|
| 2025-06-02 | 4,449 | 3,725 | **0** | 117 | 117 |
| 2025-07-01 | 4,449 | 3,726 | 0 | 117 | 117 |
| 2025-08-01 | 4,449 | 3,729 | 0 | 116 | 116 |
| 2025-09-01 | 4,449 | 3,734 | 0 | 116 | 116 |
| 2025-10-01 | 4,449 | 3,735 | 0 | 117 | 117 |
| 2025-11-04 | 4,449 | 3,741 | 0 | 107 | 107 |
| 2025-12-01 | 4,449 | 3,744 | 0 | 107 | 107 |
| 2026-01-15 | 4,449 | 3,748 | 0 | 110 | 110 |
| 2026-02-15 | 4,449 | 3,761 | 0 | 111 | 111 |
| 2026-03-16 | 4,449 | 3,763 | 0 | 112 | 112 |

→ research_watchlist_signals row count: 436 → **1,566**
   (= 109 R2-D + 327 R2-F2 + 1,130 R2-F3)

### Return evaluation (price 取得可能な 5 件のみ)

#### 5d return (★ 主要 horizon)

| base_date | overall mean | overall win | top10 mean | top30 mean | top100 mean |
|---|---:|---:|---:|---:|---:|
| 2025-11-04 | -0.47% | 38% | **-4.13%** | -1.84% | -0.47% |
| 2025-12-01 | +0.08% | 49% | -2.25% | -2.44% | +0.08% |
| 2026-01-15 | -0.60% | 36% | -1.80% | -0.92% | -0.60% |
| 2026-02-15 | +1.05% | 59% | **+4.37%** | +2.79% | +1.05% |
| 2026-03-16 | -1.91% | 22% | -2.98% | -2.53% | -1.91% |
| **avg** | **-0.37%** | 41% | **-1.36%** | **-0.99%** | -0.37% |

#### 20d return

| base_date | overall mean | top10 mean | top30 mean |
|---|---:|---:|---:|
| 2025-11-04 | +0.79% | -5.36% | -1.55% |
| 2025-12-01 | +4.09% | +5.43% | +2.82% |
| 2026-01-15 | +0.76% | -5.33% | -3.24% |
| 2026-02-15 | -0.42% | **+16.58%** | +4.74% |
| 2026-03-16 | +0.30% | -5.18% | -3.14% |
| **avg** | **+1.10%** | **+1.23%** | **-0.07%** |

## ★ 重要発見: R2-F2 (leak あり) vs R2-F3 (leak-safe) 比較

### top10 vs overall 平均差

| horizon | R2-F2 (leak あり、3 期間) | R2-F3 (leak-safe、5 期間) | 差 |
|---|---:|---:|---|
| 1d | +0.32% (-0.28% → +0.04%) | -0.07% (-0.30% → -0.37%)? | -0.40% |
| **5d** | **+0.71%** (-0.40% → +0.31%) | **-0.99%** (-0.37% → -1.36%) | **-1.70%** |
| 20d | +5.74% (-1.91% → +3.83%) | +0.13% (+1.10% → +1.23%) | -5.61% |

→ **leak-safe にすると top10 の優位性が大幅減退、5d で逆効果に**。
   R2-F2 で見えた優位は **2026-05-01 indicators の future information
   leak 由来** の可能性が高い。

### outlier 影響 (2026-02-15 を除いた 4 期間 平均)

| horizon | overall | top10 | top30 |
|---|---:|---:|---:|
| 5d | -0.73% | -2.79% | -1.93% |
| 20d | +1.49% | -2.61% | -1.28% |

→ 2026-02-15 (= top10 +16.58% の outlier) を除くと **top10/top30
   は overall を 全 horizon で下回る**。

### 結論

R2-F3 leak-safe で見ると、現状の **Watchlist Ranker score 設計は
**overall を上回る一貫した signal を出せていない**。
R2-D2 で:
- 重み (QV/EG/CV) の見直し
- sector_relative の精度向上
- factor 追加 (momentum / size / liquidity 等)
- regime-aware ensemble

を検討する必要がある。R2-F2 で「ある程度効いてそう」と見えた結果は
**leak バイアスを反映していた** という解釈が妥当。

## DB 安全性

| DB | last_modified | 状態 |
|---|---|---|
| production (`fire.db`) | 2026-05-07 16:12 | 完全無触 |
| develop (`fire.develop.db`) | 2026-05-07 18:14 | 完全無触 |
| staging (`fire.staging.db`) | 2026-05-09 21:13 | research_watchlist_signals に 1,130 row 追加 (R2-F3) |

`research_watchlist_signals` 内訳:
| source_version | rows | base_dates |
|---|---:|---|
| r2d_v1 | 109 | 2026-05-09 |
| r2f2_v1 | 327 | 2025-12 / 2026-01 / 2026-02 (3 件) |
| **r2f3_leaksafe_v1** | **1,130** | **2025-06 〜 2026-03 (10 件)** |
| **合計** | **1,566** | |

write 対象: `research_watchlist_signals` のみ。
**`research_derived_indicators` への write なし** (= leak-safe 設計に従い
in-memory 生成のみ、既存 R2-A3 の 2026-05-01 base 保存を尊重)。

write guard: production / develop --write-signals → LeakSafeRunRefused
発火 OK (test で確認済)。

## constraints_check

```json
{
  "leak_safe_signal_generation": true,
  "indicators_table_write_avoided": true,
  "staging_only_write": true,
  "production_db_untouched": true,
  "develop_db_untouched": true,
  "default_dry_run": true,
  "future_information_leak": false
}
```

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/historical_indicators | 23 PASS |
| scripts/jobs/run_research_leak_safe_backtest | 23 PASS |
| **R2-F3 合計** | **46 PASS** |
| 関連 regression (research_lane / scripts) | **809 PASS** |

**HQ 必須テスト 全 16 項目クリア** (詳細省略、test commit message 参照)。

## commit ログ

| # | commit | hash |
|---|---|---|
| R2-F3-1 | feat(F286-R2): add leak-safe historical indicator generation | 75da7dd |
| R2-F3-2 | feat(F286-R2): add leak-safe backtest runner | 68e9f75 |
| R2-F3-3 | test(F286-R2): add leak-safe historical backtest tests | 0c979eb |
| R2-F3-4 | docs(F286-R2): vault leak-safe backtest result | (本 commit) |
| R2-F3-5 | docs(F286-R2): log milestone — leak-safe backtest completed | (次 commit) |

Codex pre-commit 通過 × 3、CRITICAL 4 件発生 → 即修正。
--no-verify 不使用、個別 commit 厳守。

## Go/No-Go 判定

| 条件 | 結果 |
|---|---|
| **framework PASS** | ✅ 10/10 base_dates で signal 生成 + 永続化 + 評価成功 |
| leak violations 0 | ✅ 全 base_date で leak_check passed |
| disclosure_date filter 動作 | ✅ max_disclosure_date_used <= base_date 全 base_date で確認 |
| research_derived_indicators 不変 | ✅ in-memory 生成のみ、table write なし |
| price 不足の date は continue | ✅ 5/10 件 (2025-06 〜 10) で base_close 取れず、評価のみ skip |
| Codex CRITICAL 4 件 全対応 | ✅ |
| **leak-safe true backtest として使えるか** | ✅ |
| **statistical 初回判断 (signal 有望か)** | ⚠ **R2-F2 の優位性は leak 由来、R2-D2 score tuning 必須** |

## 次のアクション (HQ 判断)

R2-F3 完成 → 候補:

1. **R2-D2 tuning with source_version comparison** ← 最優先
   - Quality / Earnings / Cyclical 重み調整 (QV/EG/CV)
   - sector cap / strongest 閾値
   - momentum / size factor 追加検討
   - 複数 source_version (r2d2_*) で並列保存、leak-safe で比較
2. **R2-F4 broader historical sampling**
   - market_prices_daily に過去 data 追加 (J-Quants から)
   - より長い period (= 1 年以上) でサンプリング
   - 統計 power 上げ
3. **Lane integration**: signal output を Daytrade/Swing/Long-term
   Selection に
4. **R1-B5 (v1/v2 swap)**: 別承認

R2-D2 が最優先 — R2-F3 で「現状の score 設計では signal 効果なし」が
確認されたので、tuning 段階へ進む段階。

## 関連リンク

- [[F286_R2_F2_multi_date_evaluation|F286-R2-F2 multi-date (leak あり)]]
- [[F286_R2_F_return_evaluation|F286-R2-F Return Evaluation foundation]]
- [[F286_R2_E_signal_persistence|F286-R2-E Signal Persistence]]
- [[F286_R2_D_watchlist_ranker|F286-R2-D Watchlist Ranker]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ 10/10 base_dates で exit 0、JSON / CSV / leak_check
  artifact 全 出力、persistence 成功 (合計 +1,130 row)、return
  evaluation 5/10 件で 100% valid
- **機能した**: ✅ leak violations 0 で leak-safe 性確認、price 不足
  base_dates も continue で正常完了、Codex CRITICAL 4 件 全対応で
  stale eval / leak / write-fail / stale row 全防止、production/develop
  完全無触
- **期待値達成**: ✅ HQ 必須要件 16 項目 全 PASS、framework として
  true backtest 基盤完成、★ R2-F2 比較で signal 優位性減退の重要発見
  → R2-D2 tuning への path 明確、tests 46 PASS / regression 809 PASS、
  Codex pre-commit 通過 × 3
