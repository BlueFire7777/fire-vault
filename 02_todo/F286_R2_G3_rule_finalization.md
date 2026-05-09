# F286-R2-G3: Interpretation Rule v2 Finalization

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: R2-F4 baseline signals (`r2f4_baseline_v1`, 22 base_dates)
> **Mode**: 完全 read-only (DB write 一切なし)
> **Selected**: `cautious_mixed_to_suppress` = `r2g3_recommended_v2`
> **Result**: ★★ cautious_mixed (-1.54%/win 35%) 27 件を suppress に
> 移すことで cautious h20 +0.40%→+0.59% 改善、suppress h20 +1.14%→
> +0.85% で明確に弱い集合化 ★★

---

## 目的

R2-G2 で確立した recommended_v1 を土台に、3 点の調整余地を 6 candidates
で検証し、Stage 3 Live Advisory に渡せる recommended_v2 を確定する。

調整候補:
1. cautious_mixed (= 複数 negative factor) は h20 -1.54% / win 34.6%
   と明確 negative。suppress に移すと改善するか
2. cautious_high_volatility_top10 (高ボラ+rank<=10) は h20 -0.11%
   程度で弱いが negative とまでは言えない。normal に戻せるか
3. cautious_downtrend のうち sector strong/moderate は normal に
   戻して取りこぼしを減らせるか (variant F のみ)

---

## R2-G2 結果再掲 (h=20d)

    family              count   mean      win
    use_signal_strong     131  +6.20%   64.1%
    use_signal_normal    1552  +2.88%   59.1%
    use_signal_cautious   293  +0.40%   45.9%
    suppress_signal       224  +1.14%   49.3%
    overall              2200  +2.58%   56.6%

R2-G2 selected variant: `current` (selection_score 0.0620)

---

## 実装サマリ (5 commit + log)

| commit | 内容 |
|---|---|
| beba4a2 | feat: rule finalization module (6 candidates) |
| 20d6fac | chore: rule finalization runner |
| 09e7184 | test: 64 PASS (module 48 + runner 16) |
| (本 commit) | docs: vault |
| (次 commit) | docs: log |

### モジュール構成

    simulation/research_lane/
    └── regime_rule_finalization.py   (1,306 行)
        ├── RuleOutput / RuleCandidateConfig (base_rule で派生元追跡)
        ├── _coerce_rank (= post_cap_rank の int|None 正規化、CRITICAL fix)
        ├── _count_cautious_factors (= 3 因子数え上げ)
        ├── 6 candidate interpreter
        ├── CandidateScore / score_candidate / select_recommended_v2
        ├── RecommendedRuleV2 + build_from_summary
        └── CONDITIONS_BY_CANDIDATE 定数

    scripts/jobs/
    └── run_research_rule_finalization.py   (998 行)
        ├── 完全 read-only、--write 自体なし
        ├── 6 candidates × 22 base_dates × N horizons
        ├── candidate ごとに family / detail / by_market_regime /
        │   by_sector_flow / by_top_bucket / by_strongest_strategy /
        │   by_sector_17 cross-cut 全出力 (CRITICAL fix)
        ├── _summarize_cautious_reclassification: current vs v2
        │   で cautious 集合がどこに移ったか
        ├── _summarize_suppress_reclassification: 集合差分
        ├── _summarize_current_vs_v2: family×horizon 比較表
        └── 8 artifacts + per_candidate/

    tests/  (64 PASS)
    ├── tests/simulation/test_research_lane_regime_rule_finalization.py (48)
    └── tests/scripts/jobs/test_run_research_rule_finalization.py (16)

---

## Codex CRITICAL 3 件 全て即時対応

### feat 1 - #1: base_rule_version field に candidate name

`RecommendedRuleV2.to_dict()` の `base_rule_version` field に
`cfg.base_rule` (= candidate name "current_v1") を入れていたが、
監査ログ用には version 文字列 ("r2g3_current_v1") が必要。

**修正**: `get_candidate(cfg.base_rule).config.version` で解決、
未知 base_rule は name フォールバック、派生なしは自身の version。

### feat 1 - #2: post_cap_rank 型検証なし (= 安全性)

`rank <= 30` 比較で str / float NaN / bool 等が来ると TypeError。
Live Advisory 向け判定なので堅牢化が必要。

**修正**: `_coerce_rank()` 新設、`int | None` に正規化、
不正値は None (= unknown_no_rank) に倒す。bool は int subclass
だが意味的に rank ではないため除外。

### chore - #3: candidate ごとの cross-cut summaries 漏れ

要件 #3 (group: regime / sector_flow / top_bucket / strongest /
sector_17) が漏れて family / detail のみ出していた。candidate が
集約値で PASS しているのに特定 regime / sector で破綻している
ケースを検出できなくなる。

**修正**: candidate_results に `by_market_regime` /
`by_sector_flow` / `by_top_bucket` / `by_strongest_strategy` /
`by_sector_17` を追加、`_summary_to_csv_row` 共通化で 1 CSV に
集約。

---

## Rule candidates

| name | version | 概要 |
|---|---|---|
| current_v1 | r2g3_current_v1 | R2-G2 recommended_v1 と同等 |
| cautious_mixed_to_suppress | r2g3_cautious_mixed_to_suppress_v1 | factors>=2 を suppress に移す |
| high_vol_top10_to_normal | r2g3_high_vol_top10_to_normal_v1 | high_vol+rank<=10 を normal 降格 |
| v2_combined | r2g3_v2_combined_v1 | B + C |
| suppress_mixed_only | r2g3_suppress_mixed_only_v1 | B と同集合だが detail 別 |
| normal_safe_expansion | r2g3_normal_safe_expansion_v1 | C + downtrend で sector 強なら normal |

cautious factors (= variant E と同じ 3 因子):
- `volatility=high AND rank<=10` (high_volatility)
- `regime=downtrend` (downtrend)
- `sector_flow=strong_outflow` (strong_outflow)
- ≥2 件 → mixed と認定

---

## smoke 結果 (staging / 22 base_dates / r2f4_baseline_v1 / top100)

### Selection scores

    candidate                       all_pass  composite
    current_v1                          True   +0.0000
    cautious_mixed_to_suppress          True   +0.0048   ★ selected
    high_vol_top10_to_normal            True   +0.0008
    v2_combined                         True   +0.0037
    suppress_mixed_only                 True   +0.0048
    normal_safe_expansion               False  n/a

★ Selected: **cautious_mixed_to_suppress** (composite +0.0048)
- B と E は同集合 (suppress 集合は同じ、detail label のみ違い)、
  composite 同点 → 順番で B が先に hit
- normal_safe_expansion (F) は cautious 集合が count=0 になり
  score 計算不能 → 過剰緩和の証拠 (= 不採用が正しい)

### Family overview (h=20d)

    candidate                    family             count   mean      win
    ─────────────────────────── ────────────────── ─────  ─────── ──────
    current_v1                   strong              131  +6.20%  64.1%
    current_v1                   normal             1552  +2.88%  59.1%
    current_v1                   cautious            293  +0.40%  45.9%
    current_v1                   suppress            224  +1.14%  49.3%

    cautious_mixed_to_suppress   strong              131  +6.20%  64.1%   ★ 維持
    cautious_mixed_to_suppress   normal             1552  +2.88%  59.1%   ★ 維持
    cautious_mixed_to_suppress   cautious            266  +0.59%  47.0%   ★ +0.19pp 改善
    cautious_mixed_to_suppress   suppress            251  +0.85%  47.7%   ★ -0.29pp 弱化

    high_vol_top10_to_normal     strong              131  +6.20%  64.1%
    high_vol_top10_to_normal     normal             1594  +2.80%  58.7%   ← -0.08pp
    high_vol_top10_to_normal     cautious            251  +0.49%  46.2%   ← +0.09pp
    high_vol_top10_to_normal     suppress            224  +1.14%  49.3%

    v2_combined                  strong              131  +6.20%  64.1%
    v2_combined                  normal             1567  +2.87%  59.1%
    v2_combined                  cautious            251  +0.49%  46.2%   ← +0.09pp
    v2_combined                  suppress            251  +0.85%  47.7%   ← -0.29pp

    suppress_mixed_only          (= cautious_mixed_to_suppress と同集合)

    normal_safe_expansion        strong              131  +6.20%  64.1%
    normal_safe_expansion        normal             1845  +2.49%  57.0%   ← -0.39pp 悪化
    normal_safe_expansion        cautious              0  n/a     n/a    ← 過剰緩和
    normal_safe_expansion        suppress            224  +1.14%  49.3%

★ **重要発見**:
1. **cautious_mixed_to_suppress (B)** が最良 balance:
   - cautious +0.19pp 改善、suppress -0.29pp 弱化、strong/normal 不変
   - 27 件 (= mixed factor 該当) が cautious から suppress に移動
2. **high_vol_top10_to_normal (C)** は無改善:
   - normal h20 -0.08pp 低下 (= cautious_high_volatility_top10 が
     -0.11% で normal 平均を引き下げる)
   - cautious h20 +0.09pp 改善するも限定的
   - composite +0.0008 (≈ 0)
3. **v2_combined (D)** は B より劣る:
   - C の効果が薄いため、B 単独より composite 低下
4. **normal_safe_expansion (F)** は all_pass=False:
   - cautious 集合が空になる過剰緩和、normal h20 -0.39pp 悪化
   - 不採用が正しい

### cautious reclassification (B / D で発生)

    detail change                                              count
    cautious_high_volatility_top10 → normal_high_vol_top10        42
    cautious_downtrend → suppress_cautious_mixed                  17
    cautious_high_volatility_top10 → suppress_cautious_mixed      10

→ B では (cautious_downtrend → suppress) + (cautious_high_volatility_top10 →
   suppress) で計 27 件が suppress に移動 (= 期待通り)

### suppress reclassification (current vs v2_combined)

    group                            count
    suppress_only_in_current_v1          0
    suppress_only_in_v2_combined        27
    suppress_in_both                   224

→ v2 で 27 件追加、減少なし (= 安全な拡張)

---

## Recommended rule v2 (= r2g3_recommended_v2)

    {
      "rule_version": "r2g3_recommended_v2",
      "base_rule_version": "r2g3_current_v1",
      "selected_candidate": "cautious_mixed_to_suppress",
      "use_signal_strong": {
        "conditions": [
          "regime=uptrend",
          "alignment in (aligned_strong_sector, aligned_moderate)",
          "post_cap_rank <= 30"
        ],
        "sample_count": 131,
        "mean_return_20d": 0.0620,
        "win_rate_20d": 0.6412,
        "mean_return_5d":  0.0132,
        "win_rate_5d":  0.5878
      },
      "use_signal_normal": {
        "conditions": ["fallback (other branches not matched)"],
        "sample_count": 1552,
        "mean_return_20d": 0.0288,
        "win_rate_20d": 0.5905,
        "mean_return_5d":  0.0121,
        "win_rate_5d":  0.5677
      },
      "use_signal_cautious": {
        "conditions": [
          "(volatility=high AND rank <= 10) — 単独 factor",
          "OR (regime=downtrend AND alignment NOT in contrarian_*) — 単独 factor"
        ],
        "sample_count": 266,
        "mean_return_20d":  0.0059,
        "win_rate_20d": 0.4697,
        "mean_return_5d": -0.0306,
        "win_rate_5d":  0.2462
      },
      "suppress_signal": {
        "conditions": [
          "regime=downtrend AND alignment in (contrarian_*)",
          "OR cautious_factors >= 2 (high_volatility / downtrend / strong_outflow のうち 2 つ以上)"
        ],
        "sample_count": 251,
        "mean_return_20d":  0.0085,
        "win_rate_20d": 0.4774,
        "mean_return_5d": -0.0448,
        "win_rate_5d":  0.2195
      },
      "expected_overall": {
        "count": 2200,
        "mean_return_20d":  0.0258,
        "win_rate_20d": 0.5663
      }
    }

★ **5d パフォーマンス警告**:
- cautious h5 -3.06% / win 24.6%、suppress h5 -4.48% / win 22%
- → **5d 短期保有では cautious / suppress は完全に外す**べき。
- → 20d 中心保有を想定、Stage 3 Live Advisory でも 1-4 週保有目安。

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
                        (R2-G/G2/G3 期間 unchanged)
    fire.staging.db     2026-05-09 22:40:35 / 2026-05-09 22:40:35
                        (R2-G3 で touch されず、read-only 保証)

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
- ✅ rule candidates / interpretation_detail / recommended_v2 全て
  in-memory または artifact 出力のみ
- ✅ pre-commit Codex review 通過 × 3 (feat / chore / test)、
  CRITICAL 3 件即修正
- ✅ --no-verify 不使用、個別 commit 厳守
- ✅ scripts/seed_pattern_layer1.py の既存 modified は触らず

---

## tests

R2-G3 全 64 PASS:
- module 48 PASS (_coerce_rank / _count_cautious_factors / 6
  candidate interpreter / score_candidate / select_recommended_v2 /
  RecommendedRuleV2 / alt_horizon)
- runner 16 PASS (parse / build_joined / group / cross-cut /
  reclassification / runner smoke / refusal / alt_horizon)

regression 2,581 PASS (全テスト)。

---

## Stage 3 Live Advisory への示唆

1. **rule v2 採用 (= cautious_mixed_to_suppress)**:
   - 既存 v1 と差分 27 件のみ (= 安全な変更)
   - `--rule-version=r2g3_recommended_v2` を Live Advisory 第一フィルタ
2. **保有期間**: 5d パフォーマンスが弱い (cautious -3.06% / suppress
   -4.48%) ので、20d 中心保有を想定。1-4 週の swing 主体。
3. **position sizing**:
   - strong: 1 銘柄 余力の 5-10% 上限 (worst dd -22% を考慮)
   - normal: 通常 (R-05-02 ルール準拠)
   - cautious: normal の半分目安、新規買い優先度低
   - suppress: 原則見送り、強い裁量根拠時のみ Fujiwara 再判断
4. **LINE 通知**:
   - interpretation_detail を同梱、Fujiwara が判断材料に使う
   - suppress 銘柄は通知から除外、または `suppress` ラベルで明示
   - cautious 銘柄は出すが優先度低を明記

---

## Go/No-Go 判断

- ✅ framework PASS (64 + 2,581 全 tests / leak 0 / DB write 0)
- ✅ recommended_v2 採用可能 (selected = cautious_mixed_to_suppress)
- ✅ Stage 3 Live Advisory 第一フィルタ確定
- 🔶 5d 期間は別 rule 設計が必要 (= 別タスク R2-G4 候補)

---

## 次工程候補

1. **R2-H orthogonal cuts** (sector × regime cross / 月次効果 /
   決算シーズン)
2. **Lane integration** (F111 Daytrade Selection に v2 rule 組込み、
   F119 Evaluation で interpretation 別 PnL)
3. **R2-G4 5d rule** (= h5 用に別 rule 設計、cautious/suppress を
   normal に戻す方向)
4. **R1-B5 v1/v2 swap** (factor scoring v1 → v2 移行)

---

## 出力ファイル

    /tmp/r2g3_rule_finalization/
    ├── r2g3_candidate_summary.json    (~600 KB、全 cross-cut 込み)
    ├── r2g3_candidate_summary.csv
    ├── r2g3_interpretation_detail_summary.csv
    ├── r2g3_cautious_reclassification_summary.csv
    ├── r2g3_suppress_reclassification_summary.csv
    ├── r2g3_current_vs_v2_summary.csv
    ├── r2g3_recommended_rules.json
    ├── r2g3_leak_check_summary.json
    └── per_candidate/                 (6 candidates × 2 ファイル)

---

## 参照

- commit beba4a2: feat(F286-R2): add interpretation rule finalization
- commit 20d6fac: chore(F286-R2): add rule finalization runner
- commit 09e7184: test(F286-R2): add rule finalization tests
- 関連: F286_R2_G2_rule_refinement.md (= recommended_v1 ベース)
- 関連: F286_R2_G_regime_sector_integration.md (= initial rule)
- 関連: F286_R2_F4_broader_historical_sampling.md (= signal source)
