---
id: F286-R2-F
phase: P5: Research Lane R2 (return / backtest evaluation foundation)
priority: 高 (signal 品質検証の本格基盤、A1/A2/B/C/D が将来 return と
  どう紐付くか検証する Lane)
status: 完了 (framework PASS、price data 不足で statistical conclusion 保留 = HQ 想定シナリオ)
owner: Fujiwara
depends_on: [F286-R2-E signal persistence]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R2_E_signal_persistence, F286_R2_D_watchlist_ranker
---

★ **F286 R2-F**: research_watchlist_signals × market_prices_daily を
   join し、N 営業日後 return / max drawdown / rank-strategy-sector
   別集計を read-only で計算する基盤完成。

   **framework として実用 PASS**、ただし **staging の price 最大日
   2026-05-01 < base_date 2026-05-09** で future return data が無く、
   statistical conclusion は **保留** (= HQ 想定通り、R2-F2 で
   複数 base_date + 過去 price による本格検証へ)。

# F286-R2-F Research Return / Backtest Evaluation Foundation

## サマリ

R2-E で永続化した signal について、Research Lane の rank / strategy /
sector が **N 営業日後 return** とどう紐付くかを検証する基盤を実装。
完全 read-only、4 module + tests + smoke + vault。

実装 module:
1. **`simulation/research_lane/return_evaluation.py`** (純粋関数 + 集計)
2. **`scripts/jobs/run_research_return_evaluation.py`** (CLI runner)
3. tests 2 ファイル

## price join 仕様

| 概念 | 実装 |
|---|---|
| 営業日 | `market_prices_daily` の trading day record (= カレンダー日でなく実 record) |
| `base_trading_date` | `max(date) FROM prices WHERE code=? AND date <= base_date` (= base_date 当日含む直近以前) |
| `base_close` | `adj_close` 優先 / なければ `close` |
| `future_trading_date_n` | `base_trading_date` より後 (含まない) で n 番目の trading day |
| `return_n` | `future_close_n / base_close - 1` |
| `max_drawdown_close_n` | `min(close)` (n 営業日内) を base_close と比較した下落率、上昇のみで 0 |

## metrics

### 銘柄単位 (EvaluationRow)

`code / base_date / source_version / final_score / final_rank_label /
post_cap_rank / pre_cap_rank / strongest_strategy / sector_17_*  /
base_trading_date / base_close / horizon_future_dates /
horizon_future_closes / horizon_returns / horizon_max_drawdown_close /
price_data_status`

`price_data_status` 値:
- `"ok"`: 全 horizon で計算成功
- `"missing_base"`: base_close 取得不能 (= 該当 code に price 皆無)
- `"missing_future_1,5,20"` 等: 該当 horizon で future 不足

### 集計単位 (GroupSummary)

各 horizon ごとに:
- `count` / `valid_return_count` / `missing_price_count`
- `mean_return / median_return / win_rate`
- `min_return / max_return / std_return`
- `sharpe_like = mean / std` (std=0 / valid<2 で `None`)
- `avg_max_drawdown` / `worst_max_drawdown`

### 集計の粒度 (HQ 必須)

| 粒度 | 実装 |
|---|---|
| 全体 | `summarize_group(rows, "overall")` |
| `final_rank_label` 別 | `summarize_by_groups(...key=label)` |
| `strongest_strategy` 別 | `summarize_by_groups(...key=strategy)` |
| `sector_17` 別 | `summarize_by_groups(...key=sector)` |
| top bucket (10/30/50/100) | `summarize_by_top_buckets` |

## missing future price の扱い (HQ 想定シナリオ)

- 計算可能 horizon のみ集計、不足は `None` で明示
- `summary["missing_summary"]` に `base_close_missing` /
  `future_missing_by_horizon` / `total_signals` を出力
- runner output (JSON/CSV) で全 row に `price_data_status` 付与
- 結論: **framework は動く、statistical conclusion 保留**

## Runner 仕様

```
FIRE_ENV=staging python -m scripts.jobs.run_research_return_evaluation \
    --db staging \
    --base-date 2026-05-09 --source-version r2d_v1 \
    --horizons 1,5,20 --top-n 100 \
    --output-json /tmp/r2f_returns.json \
    --output-csv /tmp/r2f_returns.csv \
    --summary-json /tmp/r2f_summary.json
```

★ **DB write 一切なし**、`--write` option を作らない設計。
★ `--db` は staging/develop/production から選べるが、原則 staging。
   どれを指定しても read-only のみ。

## 実行結果 (2026-05-09 20:37)

```
target_db:      /Users/bluefire/fire/data/fire.staging.db
db_label:       staging
base_date:      2026-05-09
source_version: r2d_v1
horizons:       [1, 5, 20]
top_n:          100
mode:           READ-ONLY (no DB write)

signals loaded: 100
base_close available: 100 / 100   ★ 全銘柄 base 取得 (fallback 動作 OK)
  return_1d available:  0  ★ price 不足
  return_5d available:  0  ★ price 不足
  return_20d available: 0  ★ price 不足
```

### 全体 summary (HQ 想定: framework PASS / conclusion 保留)

| horizon | valid | mean | win_rate | sharpe |
|---|---:|---|---|---|
| 1d | 0 | - | - | - |
| 5d | 0 | - | - | - |
| 20d | 0 | - | - | - |

→ valid=0 で「実用 conclusion は保留」、ただし `framework` は動作。

### rank label 別

A1: count=100 / return 全 horizon `-`
(R2-E で top_n=100 を保存したので 全銘柄 A1)

### top bucket 別

| bucket | count |
|---|---:|
| top10 | 10 |
| top30 | 30 |
| top50 | 50 |
| top100 | 100 |

→ counts は機能、return は全て `-` (price 不足)。

### missing summary

| 項目 | 値 |
|---|---:|
| `base_close_missing` | 0 |
| `future_missing_1d` | 100 |
| `future_missing_5d` | 100 |
| `future_missing_20d` | 100 |

`base_close_missing=0` は重要 — base date 2026-05-09 当日の price が
無くても、fallback で 2026-05-01 系の close が取れている (= signals
の 100 銘柄全て)。

## 出力 artifact

| path | size |
|---|---|
| `/tmp/r2f_returns.json` | 全銘柄 evaluation_rows + summary |
| `/tmp/r2f_returns.csv` | 101 lines (header 1 + 100 row、horizons 動的 column) |
| `/tmp/r2f_summary.json` | overall + 4 種 group + missing_summary のみ (rows 含まず) |

CSV header (horizons 動的):
```
code,base_date,source_version,final_score,final_rank_label,
post_cap_rank,pre_cap_rank,strongest_strategy,
sector_17_code,sector_17_name,base_trading_date,base_close,
price_data_status,
future_date_1d,future_close_1d,return_1d,max_drawdown_1d,
future_date_5d,future_close_5d,return_5d,max_drawdown_5d,
future_date_20d,future_close_20d,return_20d,max_drawdown_20d
```

CSV head (sample):
```
87470,2026-05-09,r2d_v1,0.93,A1,1,1,earnings_growth,16,金融（除く銀行）,
  2026-05-01,2591.0,"missing_future_1,5,20",,,,,,,,,,,,
57290,2026-05-09,r2d_v1,0.92,A1,2,2,earnings_growth,7,鉄鋼・非鉄,
  2026-05-01,2219.0,"missing_future_1,5,20",,,,,,,,,,,,
```

## DB 安全性

| DB | last_modified | 状態 |
|---|---|---|
| production (`fire.db`) | 2026-05-07 16:12 | 完全無触 |
| develop (`fire.develop.db`) | 2026-05-07 18:14 | 完全無触 |
| staging (`fire.staging.db`) | 2026-05-09 20:21 | **R2-E 終了から変化なし** (R2-F は完全 read-only、staging も触れず) |

→ R2-F は **production/develop/staging の全 DB に変更を加えていない**。
   完全 read-only で signal 評価を実施。

## constraints_check

```json
{
  "staging_only_read": true,
  "no_db_write": true,
  "production_db_untouched": true,
  "develop_db_untouched": true
}
```

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/return_evaluation | 41 PASS |
| scripts/jobs/run_research_return_evaluation | 17 PASS |
| **R2-F 合計** | **58 PASS** |
| 関連 regression (research_lane / scripts) | **734 PASS** |

**HQ 必須テスト 全 14 項目クリア**:
- N営業日後価格取得 ✅
- base_date当日価格あり ✅
- base_date当日価格なしで直近価格fallback ✅
- future price不足 ✅ (HQ 想定シナリオ、framework PASS)
- return計算 ✅
- win_rate計算 ✅
- median/mean/std計算 ✅
- sharpe_like std=0/null handling ✅
- rank_label別集計 ✅
- strongest_strategy別集計 ✅
- sector別集計 ✅
- top_n filtering ✅
- runner smoke ✅
- read-only / no write behavior ✅

## commit ログ

| # | commit | hash |
|---|---|---|
| R2-F-1 | feat(F286-R2): add research return evaluation | 2ded6be |
| R2-F-2 | chore(F286-R2): add return evaluation runner | 98096b7 |
| R2-F-3 | test(F286-R2): add return evaluation tests | 84af6da |
| R2-F-4 | docs(F286-R2): vault return evaluation smoke result | (本 commit) |
| R2-F-5 | docs(F286-R2): log milestone — return evaluation completed | (次 commit) |

Codex pre-commit 通過 × 3 (feat / chore / test)、--no-verify 不使用、
個別 commit 厳守。

## Go/No-Go 判定

判定: **Framework PASS / Statistical conclusion 保留**

| 条件 | 結果 |
|---|---|
| price join 機能 | ✅ base_close 100/100 取得 (fallback OK) |
| return 計算ロジック | ✅ tests 41 PASS、framework 健全 |
| missing handling | ✅ HQ 想定通り、status / summary で明示 |
| 集計粒度 (rank/strategy/sector/top) | ✅ 全 4 種実装 + tests |
| read-only 動作 | ✅ DB write 一切なし、3 DB last_modified 確認済 |
| Codex pre-commit | ✅ 3 commit 全通過 |
| **statistical conclusion 可能か** | ❌ **保留** (price max 2026-05-01 < base 2026-05-09 で future なし) |

→ HQ 想定通り **R2-F2 で複数 base_date + 過去 price による本格検証へ**。

## 次のアクション (HQ 判断)

R2-F (single-date framework) 完成 → 候補:

1. **R2-F2 multi-date signal generation + evaluation** ← 最有力
   - 過去 base_date (例: 2025-12-01 / 2026-01-15 / 2026-02-15 等) で
     signal を再生成 → R2-E に保存 → R2-F で N 営業日後 return を
     検証
   - market_prices_daily は 2025-11-04〜2026-05-01 まであるので、
     例えば 2025-12 base で 5 営業日後 = 2025-12 中、20 営業日後
     = 2026-01 で実 return 計算可能
   - source_version 別に並列保存し、戦略 tuning 比較に
2. **R2-D2 tuning with source_version comparison**
   - 重み調整 (QV/EG/CV、cap_ratio、strongest 閾値) を別 source_version
     で並列保存、R2-F で return 比較
3. **Lane integration**: Daytrade/Swing/Long-term Selection に signal
   output を入力
4. **R1-B5 (v1/v2 swap)**: 別承認

R2-F2 が最優先 — R2-F の framework は完成しているので、過去日で再生成
すれば即 statistical conclusion を出せる。

## 関連リンク

- [[F286_R2_E_signal_persistence|F286-R2-E Signal Persistence]]
- [[F286_R2_D_watchlist_ranker|F286-R2-D Watchlist Ranker]]
- [[F286_R2_C_factor_scoring_smoke|F286-R2-C 3 戦略 scoring]]
- [[F286_R2_B_indicator_distribution_analysis|F286-R2-B 分布検証]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ exit 0、JSON/CSV/summary 3 種出力、framework 完全動作
- **機能した**: ✅ 100/100 で base_close fallback 機能、missing
  handling が HQ 想定通り、rank/strategy/sector/top bucket 4 種集計、
  read-only / DB 完全無触
- **期待値達成**: ✅ HQ 必須テスト 14 項目 全 PASS、
  framework PASS / conclusion 保留 (= HQ 想定シナリオ)、
  next step (R2-F2) への道筋明確、production/develop/staging 完全無触、
  tests 58 PASS / regression 734 PASS、Codex pre-commit 通過 × 3
