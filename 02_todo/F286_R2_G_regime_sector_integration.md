# F286-R2-G: Regime / Sector Flow Integration

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: R2-F4 baseline signals (`r2f4_baseline_v1`, 22 base_dates)
> **Mode**: 完全 read-only (DB write 一切なし)
> **Result**: ★★★ 解釈ルールに有意な分離力あり ★★★

---

## 目的

R2-F4 で確立した baseline signals (QV/EG/CV 0.35/0.35/0.30) に **市場局面 (regime)** と
**セクターフロー (sector flow)** の文脈を重ね、「どんな状況で signal が機能し、
どんな状況で抑制すべきか」を定量化する。

R2-D2 / R2-F4 では「signal 単体としては baseline が最強」が判明したが、それ以上の
リターン引き出しには「使う/使わない」の判定軸が必要。R2-G はその第一歩。

---

## 実装サマリ (3 commit + tests + vault + log)

| commit | 内容 |
|---|---|
| 0072f82 | feat: market regime features (`simulation/research_lane/market_regime.py`) |
| 7e07f6c | feat: sector flow signal integration (`sector_flow_features.py`) |
| ab84d79 | chore: integration runner (`scripts/jobs/run_research_regime_sector_integration.py`) |
| ef1f6fb | test: regime sector integration tests (3 files / 56 PASS) |
| (本 commit) | docs: vault |
| (次 commit) | docs: log |

### モジュール構成

```
simulation/research_lane/
├── market_regime.py            (528 行)
│   ├── MarketRegimeFeatures dataclass
│   ├── fetch_market_aggregate_closes  (= 銘柄横断 AVG を市場 proxy)
│   ├── assign_regime_label             (uptrend/downtrend/rebound/range/unknown)
│   ├── compute_volatility_thresholds   (★ at_base_date leak filter)
│   ├── reapply_volatility_labels_leak_safe
│   └── assert_regime_leak_safe
└── sector_flow_features.py     (522 行)
    ├── SectorFlowFeatures dataclass
    ├── fetch_sector_aggregate_closes  (sector_17 × date AVG with LEFT JOIN)
    ├── build_sector_flow_features     (rank + label assignment)
    ├── assign_signal_sector_alignment
    └── assert_sector_flow_leak_safe

scripts/jobs/
└── run_research_regime_sector_integration.py  (689 行)
    ├── _interpret_signal               (4-rule 判定)
    ├── _join_signal_with_regime_sector
    ├── _summarize_signals_by_group     (★ (base_date, code) tuple key)
    └── run_regime_sector_integration   (END-to-END)

tests/  (56 PASS)
├── tests/simulation/test_research_lane_market_regime.py            (21)
├── tests/simulation/test_research_lane_sector_flow_features.py     (15)
└── tests/scripts/jobs/test_run_research_regime_sector_integration.py (20)
```

---

## Codex CRITICAL 対応

### #1: `compute_volatility_thresholds` future leak

**問題**: 全 features の `market_volatility_20d` から閾値を計算していたが、後日
features (= base_date 後) の volatility も含むため、各 base_date の volatility_label
判定に **未来情報がリーク**。

**修正**: `at_base_date` パラメータを追加し、その日以前の features のみで分布を
計算する版に変更。`reapply_volatility_labels_leak_safe` が各 base_date で
独立に閾値を計算 → label 再付与する設計に。

```python
def compute_volatility_thresholds(
    features: list[MarketRegimeFeatures],
    *,
    at_base_date: str | None = None,
    high_pct: float = HIGH_VOLATILITY_PCT,
    low_pct: float = LOW_VOLATILITY_PCT,
) -> tuple[float | None, float | None]:
    if at_base_date is not None:
        features = [f for f in features if f.base_date <= at_base_date]
    # ... 分布計算
```

### #2: `_summarize_signals_by_group` code-only key で多日上書き

**問題**: 元実装は `eval_by_code = {r.code: r}` の dict を作り、joined.append
で集計。22 base_dates × 100 銘柄 = 最大 2,200 件あるが、**同一銘柄が複数日に登場
した場合に最後の 1 件で上書き** され、regime / sector / alignment / top bucket
集計が壊れる。

**修正**: `(base_date, code)` tuple key を採用。
`_summarize_signals_by_group` と `top_buckets` の両方で適用。

```python
eval_by_key = {
    (r.base_date, r.code): r for r in eval_rows
}
# ...
eval_row = eval_by_key.get(
    (j.get("base_date"), j.get("code")),
)
```

両件とも pre-commit Codex review で発見 → 即修正 → re-review 通過。

---

## 解釈ルール (R2-G initial)

```
suppress_signal:
  regime=downtrend AND alignment in (contrarian_outflow,
    contrarian_strong_outflow)

use_signal_strong:
  regime=uptrend AND alignment in (aligned_strong_sector,
    aligned_moderate) AND post_cap_rank <= 30

use_signal_cautious:
  volatility_label=high AND post_cap_rank <= 10
  | regime=downtrend (single)

use_signal_normal:
  fallback (上記いずれにも該当せず)
```

(R2-G2 で backtest 結果に基づき refine 想定)

---

## smoke run 結果 (staging / 22 base_dates / r2f4_baseline_v1 / top100)

### overall

| horizon | mean_return | win_rate | count |
|---|---|---|---|
| 1d  | -0.31% | 41.2% | 2,200 |
| 5d  | +0.06% | 49.1% | 2,200 |
| 20d | **+2.58%** | **56.6%** | 2,200 |

### regime label distribution

```
uptrend:    11 base_dates
downtrend:   5
range:       4
rebound:     1
unknown:     1   (= 2024-06-01、データ不足)
```

### by market regime (h=20d)

| regime | count | mean | win |
|---|---|---|---|
| uptrend | 1,100 | **+2.85%** | 60.3% |
| range | 400 | **+3.53%** | 58.7% |
| rebound | 100 | +3.01% | 62.2% |
| downtrend | 500 | +0.65% | 46.9% |
| unknown | 100 | +4.76% | 50.5% |

★ downtrend で +0.65% (win 47%) と顕著な性能低下を確認。

### by sector flow (h=20d)

| flow | count | mean | win |
|---|---|---|---|
| strong_inflow | 244 | **+3.39%** | 59.8% |
| moderate_inflow | 782 | +2.92% | 57.9% |
| outflow | 871 | +2.74% | 56.5% |
| strong_outflow | 303 | **+0.57%** | 51.3% |

★ strong_outflow で +0.57% / win 51% に大幅低下、strong_inflow とは 2.8% pt 差。

### **★ by interpretation (h=20d) — R2-G の本丸 ★**

| interpretation | count | mean | win | vs baseline (+2.58%) |
|---|---|---|---|---|
| **use_signal_strong** | **131** | **+6.20%** | **64.1%** | **+3.62 pp** |
| use_signal_normal | 1,552 | +2.88% | 59.1% | +0.30 pp |
| **suppress_signal** | **224** | **+1.14%** | **49.3%** | **-1.44 pp** |
| **use_signal_cautious** | **293** | **+0.40%** | **45.9%** | **-2.18 pp** |

★ ルールに **明確な分離力** あり:
- `use_signal_strong` → +6.20% / win 64% (= 2.4 倍)
- `suppress_signal` / `use_signal_cautious` → +1% 未満 / win < 50%

### by top bucket (h=20d)

| bucket | count | mean | win |
|---|---|---|---|
| top10 | 220 | +3.14% | 51.2% |
| top30 | 660 | +3.09% | 55.4% |
| top50 | 1,100 | +2.60% | 56.7% |
| top100 | 2,200 | +2.58% | 56.6% |

★ top10 / top30 で mean は若干高いが win は逆転 (51% < 56%)。R2-G の interpretation
ルールは top bucket よりも強い分離 (+6.20% vs +3.14%)。

---

## leak safety

```json
{
  "regime": {
    "total_features": 22,
    "violations": 0,
    "passed": true,
    "max_price_date_used_for_regime": "2026-02-27"
  },
  "sector_flow": {
    "total_base_dates": 22,
    "violations": 0,
    "passed": true,
    "max_price_date_used_for_sector_flow": "2026-02-27"
  }
}
```

base_date 2026-03-01 の直近営業日が 2026-02-27 (土日除く) で、全 base_date で
`max_price_date_used <= base_date` を確認。

---

## 制約遵守

- ✅ DB write 一切なし (`--write` option 自体存在しない)
- ✅ FIRE_ENV=staging で実行 (production / develop 触らず)
- ✅ market_prices_daily / market_listings / research_watchlist_signals 全て read-only
- ✅ market_regime / sector_flow features は in-memory 生成のみ
- ✅ pre-commit Codex review 通過 (CRITICAL #1 / #2 修正後)
- ✅ `seed_pattern_layer1.py` の既存変更状態は触らず

---

## 次タスク候補

### R2-G2 (R2-G ルール refine 案)

R2-G initial rule は仮置き。実 backtest 結果から:

1. **strong rule の拡張**: `use_signal_strong` (+6.20%) は count 131 と少なめ。
   条件緩和 (例: post_cap_rank <= 50) で count 増えるか確認
2. **suppress rule の検証**: `suppress_signal` (+1.14%) は完全 0 にすべきか、
   それとも -2% のフィルタとしてすでに機能か (= ノイズ除去で +0.06% から +1.14% に
   shift できれば十分)
3. **cautious の詳細分解**: `use_signal_cautious` (+0.40%) は volatility=high と
   downtrend single の合算。分解して優先度差を見る

### R2-H 案 (orthogonal な切り口)

- セクター × regime クロス (例: 情報通信 × uptrend = 強い? 弱い?)
- 月次効果 / 決算シーズン効果
- factor 係数の自動再 tuning (R2-D2 拡張)

### Stage 3 への布石

R2-G の `use_signal_strong` 集合 (= +6.20% / win 64%) が安定していれば、
- F111 Daytrade Selection に「regime + sector alignment フィルタ」を追加
- F119 Evaluation で interpretation 別 PnL を集計
- Live Advisory で「強い signal にだけ大きく賭ける」運用が可能

---

## 出力ファイル

```
/tmp/r2g_regime_sector/
├── r2g_regime_sector_summary.json   (50 KB、全 group × horizon)
├── r2g_regime_sector_summary.csv    (12 KB、表計算用)
├── signal_regime_join.csv           (600 KB、2,200 行 × 21 列)
├── r2g_leak_check_summary.json
├── r2g_interpretation_rules.json
└── per_base_date/
    └── r2g_<bd>_market_regime.json + r2g_<bd>_sector_flow.csv (22 セット)
```

---

## 参照

- `commit ef1f6fb`: test(F286-R2): add regime sector integration tests
- `commit ab84d79`: chore(F286-R2): add regime sector integration runner
- `commit 7e07f6c`: feat(F286-R2): add sector flow signal integration
- `commit 0072f82`: feat(F286-R2): add market regime features
- 関連: `F286_R2_F4_broader_historical_sampling.md` (= signal source)
- 関連: `F286_R2_D2_watchlist_tuning.md` (= preset 比較)
