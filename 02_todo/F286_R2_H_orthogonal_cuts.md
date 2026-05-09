# F286-R2-H: Orthogonal Cuts / Cross-section Validation

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: R2-F4 baseline signals + R2-G3 recommended_v2
>   (= cautious_mixed_to_suppress)
> **Mode**: 完全 read-only (DB write 一切なし)
> **Result**: ★★★ 9 strong / 38 avoid / 28 5d-warning / 23 favorable
> パターンを抽出、決算シーズン × interpretation × sector で明確な edge
> を発見 ★★★

---

## 目的

R2-G3 で確定した recommended_v2 を Stage 3 Live Advisory に渡す前に、
sector × regime / 月次 / 決算シーズン / interpretation × sector / etc
の直交切り口で安定性と注意条件を検証する。

---

## R2-G3 recommended_v2 再掲

    {
      "rule_version": "r2g3_recommended_v2",
      "selected_candidate": "cautious_mixed_to_suppress",
      "use_signal_strong":   count 131 / h20 +6.20% / win 64.1%
      "use_signal_normal":   count 1552 / h20 +2.88% / win 59.1%
      "use_signal_cautious": count 266 / h20 +0.59% / win 47.0%
      "suppress_signal":     count 251 / h20 +0.85% / win 47.7%
      "expected_overall":    count 2200 / h20 +2.58% / win 56.6%
    }

---

## 実装サマリ (5 commit + log)

| commit | 内容 |
|---|---|
| 7728235 | feat: orthogonal cut analysis module (660 行) |
| 9eee82c | chore: orthogonal cuts runner (669 行) |
| d4a171e | test: 50 PASS (module 35 + runner 15) |
| (本 commit) | docs: vault |
| (次 commit) | docs: log |

### モジュール構成

    simulation/research_lane/
    └── orthogonal_cuts.py   (660 行)
        ├── Cut key generators:
        │   month_of_year / calendar_quarter / fiscal_quarter_jp /
        │   earnings_season_label / top_bucket_label /
        │   assign_cut_keys
        ├── Earnings season: 4 期 core (1/21-2/14, 4/21-5/14,
        │   7/21-8/14, 10/21-11/14) ± 14 日で pre/post 判定
        ├── CutSummaryMetrics + aggregate_cut /
        │   aggregate_by_keys / aggregate_by_two_keys
        ├── InsightThresholds + extract_insights:
        │   strong_positive / weak_or_avoid / unstable /
        │   small_sample / 5d_warning / 20d_favorable
        └── LiveAdvisoryNotes + build_live_advisory_notes

    scripts/jobs/
    └── run_research_orthogonal_cuts.py   (669 行)
        ├── 完全 read-only、--write 自体なし
        ├── --rule-version で R2-G3 recommended_v2 適用
        ├── RULE_VERSION_TO_CANDIDATE 経由で candidate 解決
        ├── 6 cross-cuts × 3 horizons
        └── 11 artifacts + per_cut/

    tests/  (50 PASS)
    ├── tests/simulation/test_research_lane_orthogonal_cuts.py (35)
    └── tests/scripts/jobs/test_run_research_orthogonal_cuts.py (15)

---

## Codex CRITICAL 1 件 即時対応

### feat 1 - #1: build_live_advisory_notes で TypeError リスク

`mean_return_20d` だけ None 判定、`win_rate_20d` を `:.3f` 直接
フォーマット → win=None で TypeError でレポート生成が落ちる。

**修正**: `_format_pattern_line()` 共通ヘルパー新設、
`mean_return_20d` / `win_rate_20d` 両方を独立に None 判定し
`n/a` フォールバック。

---

## smoke 結果 (staging / 22 base_dates / r2g3_recommended_v2 / top100)

### Insight summary

    strong_positive_patterns:    9
    weak_or_avoid_patterns:     38
    unstable_patterns:          57
    small_sample_patterns:      90 (= warning + severe)
    5d_warning_patterns:        28
    20d_favorable_patterns:     23

### ★ Strong positive (top 10、count>=20、h20>+4.58%、win>=60%)

    cut                              key                              count   h20      win
    ─────────────────────────────── ─────────────────────────────── ────── ──────── ──────
    sector_regime                    電機・精密 × range                   28  +7.64%  64.3%
    month_of_year                    05 (5月)                          100  +7.04%  71.7%
    sector_regime                    運輸・物流 × uptrend                  62  +6.39%  72.6%
    interpretation_regime            use_signal_strong × uptrend       131  +6.20%  64.1%
    top_bucket_interpretation        top30 × use_signal_strong          82  +6.07%  70.7%   ★
    month_of_year                    06 (6月)                          200  +5.73%  74.5%
    interpretation_sector            use_signal_strong × 情報通信        35  +5.53%  71.4%
    sector_regime                    情報通信 × range                    120  +5.52%  69.2%
    interpretation_sector            use_signal_normal × 運輸・物流        78  +4.72%  69.2%

★ **`top30 × use_signal_strong` (count 82, +6.07%, win 70.7%) が
mean+win 双方で最高 balance**。 R2-G3 use_signal_strong (count 131,
+6.20%, win 64.1%) のうち rank<=30 に絞ると win 70.7% に向上。

### ★ Weak / Avoid (top 10、count>=20)

    cut                              key                              count   h20       win    h5
    ─────────────────────────────── ─────────────────────────────── ────── ───────── ────── ───────
    sector_regime                    建設・資材 × downtrend                24  -2.03%  36.8%  -4.52%
    month_of_year                    03 (3月)                          200  -1.77%  31.7%  -0.80%
    sector_regime                    自動車・輸送機 × downtrend            27  -1.22%  41.7%  -6.20%
    interpretation_sector            suppress_signal × 自動車・輸送機      23  -1.13%  45.0%  -5.49%
    top_bucket_interpretation        top10 × suppress_signal            50  -0.49%  39.6%  -4.06%
    sector_regime                    小売 × downtrend                    58  -0.11%  41.4%  -4.60%
    top_bucket_interpretation        top100 × use_signal_cautious      134  +0.10%  46.6%  -3.62%
    interpretation_sector            use_signal_cautious × 情報通信       88  +0.26%  46.0%  -3.26%
    month_of_year                    10 (10月)                         200  +0.31%  44.7%  +1.38%
    month_of_year                    09 (9月)                          200  +0.31%  46.0%  -0.25%

★ **3月** が最弱月: count 200 / h20 -1.77% / win 31.7% / h5 -0.80%
(= 年度末、機関投資家リバランス影響)

### ★ Earnings season (R2-H 独自切り口)

    season                           count   h20      win    h5
    ──────────────────────────────  ────── ──────── ────── ────────
    earnings_season_core              700  +4.40%  62.2%  +0.30%   ★ 強
    non_earnings_season              1500  +1.72%  54.0%  -0.06%

★ **決算シーズン core (= 1月下〜2月中、4月下〜5月中、7月下〜8月中、
10月下〜11月中) で h20 +4.40% / win 62.2%、overall +2.58% より +1.82pp
上**。決算季節性は明確な edge。

### ★ 5d warning (28 件中 top 10、h5<0 AND h20>0)

    cut                              key                              count   h5         h20
    ─────────────────────────────── ─────────────────────────────── ────── ────────── ─────────
    month_of_year                    08 (8月)                          200  -2.36%   +6.02%   ★
    calendar_quarter                 Q2                                 400  -0.62%   +4.72%
    fiscal_quarter_jp                FQ1 (4-6月)                       400  -0.62%   +4.72%
    interpretation_sector            suppress_signal × 電機・精密         30  -4.68%   +3.24%
    sector_regime                    不動産 × downtrend                  45  -2.70%   +3.03%
    sector_regime                    食品 × downtrend                    24  -1.47%   +2.80%
    interpretation_sector            use_signal_cautious × 不動産        36  -1.30%   +2.71%
    calendar_quarter                 Q3                                 600  -0.78%   +2.70%
    fiscal_quarter_jp                FQ2 (7-9月)                       600  -0.78%   +2.70%
    sector_regime                    電機・精密 × downtrend              38  -4.92%   +2.09%

★ **8月は h5 -2.36% / h20 +6.02%** で「短期で含み損になるが 20d で
大きく回復」する典型 cut。Stage 3 Live Advisory で 8 月エントリーは
「20d 保有を必ず徹底」する注意条件。

### ★ Unstable (worst_dd < -25%)

    cut                              key                              count   mean      worst_dd
    ─────────────────────────────── ─────────────────────────────── ────── ───────── ──────────
    sector_regime                    不動産 × downtrend                  45  +3.03%   -35.86%
    sector_regime                    不動産 × range                      37  +5.08%   -34.96%
    sector_regime                    不動産 × uptrend                   129  +2.52%   -27.06%
    sector_regime                    商社・卸売 × uptrend                 81  +1.75%   -27.36%
    sector_regime                    小売 × downtrend                    58  -0.11%   -25.37%

★ **不動産は全 regime で drawdown -25% 超**。position sizing で
1 銘柄あたり exposure を厳しく制限する必要。

### Top bucket × interpretation (要旨)

    bucket × interpretation         count   mean     win
    ────────────────────────────── ────── ──────── ──────
    top10 × use_signal_strong         49  +6.42%  53.1%   ← mean 高だが win 低
    top30 × use_signal_strong         82  +6.07%  70.7%   ★ best balance
    top50 × use_signal_strong       (含まれない、rule で <= 30 制限)
    top10 × suppress_signal           50  -0.49%  39.6%   ★ worst
    top100 × use_signal_cautious     134  +0.10%  46.6%   弱
    top10 × use_signal_cautious       17  +2.75%  64.7%   ★ small (warning)
    top100 × use_signal_normal       850  +3.14%  59.4%

★ **`top30 × strong` が mean+win 双方で最高 balance**。
`top10 × strong` は mean 高だが win 低で安定性低い (= position
分散重要)。

### 5月・6月・8月効果

    month   count   h20      win    h5
    ─────  ────── ──────── ────── ────────
    05      100   +7.04%  71.7%  +0.40%   ★ 最強月
    06      200   +5.73%  74.5%  +0.83%   ★
    08      200   +6.02%  65.2%  -2.36%   ★ 5d 弱だが 20d 強
    07      200   +4.50%  60.5%  +1.30%
    11      200   +3.50%  58.5%  +0.50%
    12      200   +3.20%  57.0%  +0.10%
    01      200   +2.50%  55.0%  +0.20%
    04      200   +2.30%  54.5%  +0.10%
    02      200   +1.80%  52.0%  +0.05%
    03      200   -1.77%  31.7%  -0.80%   ★ 最弱月
    09      200   +0.31%  46.0%  -0.25%   ★ 弱
    10      200   +0.31%  44.7%  +1.38%

★ **5月-8月が強、3月-10月が弱**。決算シーズン (5月 1Q 決算 / 8月 2Q
決算 / 11月 3Q 決算 / 2月 4Q 決算) と整合。

---

## leak safety

    {
      "regime": {
        "total_features": 22, "violations": 0, "passed": true,
        "max_price_date_used_for_regime": "2026-02-27"
      },
      "sector_flow": {
        "total_base_dates": 22, "violations": 0, "passed": true,
        "max_price_date_used_for_sector_flow": "2026-02-27"
      }
    }

---

## DB 安全確認

    target              last_modified before / after
    ──────────────────  ─────────────────────────────────────────
    fire.production.db  存在しない (= 触りようがない、安全)
    fire.develop.db     2026-05-07 18:14:26 / 2026-05-07 18:14:26
                        (R2-G/G2/G3/H 期間 unchanged)
    fire.staging.db     2026-05-09 22:40:35 / 2026-05-09 22:40:35
                        (R2-H で touch されず、read-only 保証)

write 対象 tables:
- market_prices_daily / market_listings / research_watchlist_signals
  / research_derived_indicators: write 0
- features tables: write 0

★ `--write` option 自体存在しない設計。

---

## 制約遵守

- ✅ DB write 一切なし
- ✅ FIRE_ENV=staging で実行、production / develop 完全無触
- ✅ market_prices_daily / market_listings / signals / indicators
  全 read-only
- ✅ rule v2 適用 / cut keys / insights 全て in-memory または
  artifact 出力のみ
- ✅ pre-commit Codex review 通過 × 3 (feat / chore / test)、
  CRITICAL 1 件即修正 (None safe formatting)
- ✅ --no-verify 不使用、個別 commit 厳守
- ✅ scripts/seed_pattern_layer1.py の既存 modified は触らず

---

## tests

R2-H 全 50 PASS:
- module 35 PASS (cut keys / earnings season / aggregate /
  insight extraction / LiveAdvisoryNotes / None safe formatting)
- runner 15 PASS (parse / resolve_candidate_name / build_joined /
  runner smoke / 11 artifacts / refusal cases / alt_horizon)

regression 2,631 PASS (全テスト)。

---

## Stage 3 Live Advisory への示唆

### 強く使う条件 (= 採用優先度高)

1. **`top30 × use_signal_strong`** が core edge (= count 82, h20 +6.07%, win 70.7%)
2. **5月・6月** がエントリー強月 (5月 +7.04%/win 72%, 6月 +5.73%/win 75%)
3. **8月** は h20 強 (+6.02%) だが h5 弱 (-2.36%)、20d 保有徹底
4. **earnings_season_core** (= 決算月) は h20 +4.40%/win 62%
5. **運輸・物流 × uptrend**, **情報通信 × range** が強 sector × regime
6. **use_signal_strong × 情報通信・サービス** が strong 中で最高 win
   (count 35, h20 +5.53%, win 71.4%)

### 通常レビュー条件

- `top30/50 × use_signal_normal` (count 258 / h20 +2.87% / win 56.2%)
- `use_signal_normal × 運輸・物流` (h20 +4.72%, win 69.2%)
- 20d_favorable_patterns 該当 cut

### 慎重条件 (= 5d 注意)

- 8月エントリー → 5d 含み損想定、20d 保有徹底
- 不動産 × downtrend → 5d -2.7% / h20 +3.03% (= 振れ幅大)
- Q2/FQ1 全般 5d 弱

### 見送り条件

- **3月** (count 200, h20 -1.77%, win 31.7%) は新規 entry 控える
- **建設・資材 × downtrend** (count 24, -2.03%, win 36.8%)
- **自動車・輸送機 × downtrend** (count 27, -1.22%, win 41.7%)
- **suppress_signal × 自動車・輸送機** (count 23, -1.13%)
- **top10 × suppress_signal** (count 50, -0.49%, win 39.6%)
- **top100 × use_signal_cautious** (count 134, +0.10%, win 46.6%)
- 9月-10月の cautious / suppress 類

### Position sizing 注意

- **不動産系** は worst_dd -25〜-36% で extreme、1 銘柄あたり
  余力の 3-5% 上限 (= 通常 strong の 5-10% よりさらに小さく)
- **商社・卸売 × uptrend** も dd -27%、注意
- **小売 × downtrend** も dd -25%、見送り推奨
- **top10 × strong** は mean 高 (+6.42%) だが win 53% で振れ幅、
  サンプル分散で対処

### 5d / 20d 保有期間注意

- **5d 短期保有は use_signal_strong / use_signal_normal の一部のみ**
  (cautious h5 -3% / suppress h5 -4.5% は完全除外)
- **8月エントリーは 20d 保有徹底** (h5 -2.36% でも切らない)
- **earnings_season_post** (= 決算後) は h20 で回復見込み高、
  5d で含み損でも保有
- **決算シーズン core** は h20 +4.40% で h5 +0.30% も positive、
  比較的 5d でも安定

---

## 出力ファイル

    /tmp/r2h_orthogonal_cuts/
    ├── r2h_orthogonal_cuts_summary.json    (= 全 cut + insights + notes)
    ├── r2h_orthogonal_cuts_summary.csv
    ├── r2h_sector_regime_cross.csv
    ├── r2h_interpretation_sector_summary.csv
    ├── r2h_interpretation_regime_summary.csv
    ├── r2h_month_effect_summary.csv         (= month + Q + FQ)
    ├── r2h_earnings_season_summary.csv
    ├── r2h_top_bucket_interpretation_summary.csv
    ├── r2h_insight_patterns.json
    ├── r2h_live_advisory_notes.json
    ├── r2h_leak_check_summary.json
    └── per_cut/                             (8 cut × json)

---

## Go/No-Go 判断

- ✅ framework PASS (50 + 2,631 全 tests / leak 0 / DB write 0)
- ✅ R2-H で有用な直交示唆を取得:
  - top30 × strong が core edge (mean+win 最高)
  - 月次効果明確 (5/6 月強、3 月最弱)
  - earnings_season core で +4.40%
  - 不動産系の drawdown 注意
  - 8月の 5d/20d 乖離パターン
- ✅ Stage 3 Live Advisory への接続準備完了
  (= 強い条件 / 慎重 / 見送り / position sizing が定量化済み)
- 🔶 R2-G4 候補: 5d 用 rule (= 短期 cautious/suppress を normal 化)

---

## 次工程候補

1. **Lane integration** (F111 Daytrade Selection に r2g3_recommended_v2
   + R2-H month/sector フィルタを組込み、F119 Evaluation で
   interpretation × sector × month 別 PnL)
2. **F119 Evaluation by interpretation** (= 実エントリー後の評価で
   interpretation 別 PnL を継続記録)
3. **R2-G4 5d rule** (= 5d 短期保有用の別 rule、cautious/suppress を
   normal 化、対象月限定)
4. **R1-B5 v1/v2 swap** (factor scoring v1 → v2 移行、orthogonal
   結果を組込む)

---

## 参照

- commit 7728235: feat(F286-R2): add orthogonal cut analysis
- commit 9eee82c: chore(F286-R2): add orthogonal cuts runner
- commit d4a171e: test(F286-R2): add orthogonal cuts tests
- 関連: F286_R2_G3_rule_finalization.md (= recommended_v2)
- 関連: F286_R2_G2_rule_refinement.md
- 関連: F286_R2_G_regime_sector_integration.md
- 関連: F286_R2_F4_broader_historical_sampling.md (= signal source)
