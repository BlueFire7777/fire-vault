# F286-R2-G2: Interpretation Rule Refinement

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: R2-F4 baseline signals (`r2f4_baseline_v1`, 22 base_dates)
> **Mode**: 完全 read-only (DB write 一切なし)
> **Selected rule**: `current` (= R2-G initial、selection_score=0.0620)
> **Result**: ★★★ strong_top30 が最良 balance、cautious_mixed が真の弱集合
> (-1.54%) と判明 ★★★

---

## 目的

R2-G で確認された interpretation rule (use_signal_strong / normal /
cautious / suppress) の **rule variant 比較** と **rule refinement**。

R2-G の課題:
- `use_signal_strong` (count 131 / +6.20%) が条件厳しすぎる可能性
- `use_signal_cautious` (+0.40% / win 45.9%) が弱い、原因不明
- `suppress_signal` (+1.14% / win 49.3%) が baseline より弱い程度で
  完全 negative ではない、本当に「捨てる集合」か疑問

R2-G2 で 6 variants で比較し、Stage 3 Live Advisory に渡せる
recommended rule を確定する。

---

## R2-G 結果再掲 (h=20d, baseline 比較)

| family | count | mean | win | vs baseline |
|---|---|---|---|---|
| baseline overall | 2,200 | +2.58% | 56.6% | — |
| use_signal_strong | 131 | +6.20% | 64.1% | +3.62 pp |
| use_signal_normal | 1,552 | +2.88% | 59.1% | +0.30 pp |
| use_signal_cautious | 293 | +0.40% | 45.9% | -2.18 pp |
| suppress_signal | 224 | +1.14% | 49.3% | -1.44 pp |

---

## 実装サマリ (5 commit + log)

| commit | 内容 |
|---|---|
| 48512cc | feat: rule refinement module (6 variants + RuleOutput / RecommendedRule) |
| 581b933 | chore: rule refinement runner (read-only, 7 artifacts) |
| b05c454 | test: 51 PASS (rule_refinement 33 + runner 18) |
| (本 commit) | docs: vault |
| (次 commit) | docs: log |

### モジュール構成

    simulation/research_lane/
    └── regime_rule_refinement.py   (771 行)
        ├── RuleOutput dataclass
        ├── RuleVariantConfig dataclass (= JSON serializable)
        ├── 6 variant interpreter 関数
        ├── VARIANTS registry + get_variant()
        ├── apply_variant() / export_variant_configs()
        └── RecommendedRule + build_recommended_rule_from_summary()

    scripts/jobs/
    └── run_research_rule_refinement.py   (1,041 行)
        ├── 6 variants × 22 base_dates × 4 horizons
        ├── _build_joined_rows / _summarize_group_by_key
        ├── _summarize_top_buckets / _summarize_strong_thresholds
        ├── _summarize_cautious_split / _summarize_suppress_validation
        ├── _select_best_variant (= count>=30 + max mean)
        └── 7 artifact 出力 + per_variant/

    tests/  (51 PASS)
    ├── tests/simulation/test_research_lane_regime_rule_refinement.py (33)
    └── tests/scripts/jobs/test_run_research_rule_refinement.py (18)

---

## Codex CRITICAL 4 件 全て即時対応

### feat 1 - #1: variant C (strict_sector) の suppress が rank 無制限

config では `sector_flow=strong_outflow AND rank<=30 -> suppress_signal`
だが、実装は rank に関係なく全 `strong_outflow` を suppress していた。
top_n=50/100 で本来 cautious/normal にフォールスルーすべき rank>30 の
strong_outflow まで suppress に分類されると summary が偏る。

**修正**: `if sec_flow == "strong_outflow" and rank <= 30:` に変更、
detail を `suppress_strong_outflow_top30` に rename。

### chore - #2: stdout summary count=None で TypeError

family が空 (= 該当 row 0 件) のケースで `count=None` のまま
`f"{cnt:>5}"` に流すと TypeError。runner が successful run で
非 zero exit する。

**修正**: `cnt = s.get("count") or 0` でフォールバック。

### chore - #3: recommended_rule が常に horizon=20

`--horizons` から 20 を外すと `mean_return_20d` が None になり、
`r2g2_recommended_rules.json` が空っぽに。

**修正**: `display_horizon = 20 if 20 in horizons else max(horizons)`、
build_recommended_rule_from_summary に display_horizon を渡す。

### chore - #4: recommended_rule の base_variant が常に "current" 固定

6 variants 比較しているのに recommended_rule は常に current 固定。
他 variant が outperform したケースで誤った rule を推奨してしまう。

**修正**: `_select_best_variant()` を新設、`use_signal_strong` の
count >= 30 + mean_return が最大の variant を採用。該当なしなら
current にフォールバック。variant ごとに conditions テキストを保持。

---

## Rule variants

| name | version | 概要 |
|---|---|---|
| current | r2g_current_v1 | R2-G と同一 (baseline 比較用) |
| strong_top50 | r2g2_strong_top50_v1 | strong rank <=30 → <=50 緩和 |
| strong_top30_strict_sector | r2g2_strong_strict_sector_v1 | regime in (uptrend/range/rebound) + sector strong only |
| strong_regime_only | r2g2_strong_regime_only_v1 | regime のみで strong (sector 緩和) |
| cautious_split | r2g2_cautious_split_v1 | cautious detail 細分化 |
| suppress_strict | r2g2_suppress_strict_v1 | downtrend AND strong_outflow に絞る |

---

## smoke 結果 (staging / 22 base_dates / r2f4_baseline_v1 / top100)

### Variant overview (h=20d, 全 family)

    variant                       family             count   mean      win
    ────────────────────────────  ─────────────────  ─────  ───────  ──────
    current                       use_signal_strong   131  +6.20%   64.1%   ★ max mean
    current                       use_signal_normal  1552  +2.88%   59.1%
    current                       use_signal_cautious 293  +0.40%   45.9%
    current                       suppress_signal     224  +1.14%   49.3%

    strong_top50                  use_signal_strong   209  +4.54%   65.6%   ★ max win
    strong_top50                  use_signal_normal  1474  +2.94%   58.6%
    strong_top50                  use_signal_cautious 293  +0.40%   45.9%
    strong_top50                  suppress_signal     224  +1.14%   49.3%

    strong_top30_strict_sector    use_signal_strong   195  +5.51%   60.3%
    strong_top30_strict_sector    use_signal_normal  1430  +2.92%   59.8%
    strong_top30_strict_sector    use_signal_cautious 284  +0.33%   45.6%
    strong_top30_strict_sector    suppress_signal     291  +1.09%   49.3%

    strong_regime_only            use_signal_strong   480  +3.96%   58.3%   ★ count 多
    strong_regime_only            use_signal_normal  1220  +2.81%   59.9%
    strong_regime_only            use_signal_cautious 276  +0.27%   45.1%
    strong_regime_only            suppress_signal     224  +1.14%   49.3%

    cautious_split                use_signal_strong   131  +6.20%   64.1%
    cautious_split                use_signal_normal  1313  +3.35%   60.2%
    cautious_split                use_signal_cautious 532  +0.36%   49.1%
    cautious_split                suppress_signal     224  +1.14%   49.3%

    suppress_strict               use_signal_strong   131  +6.20%   64.1%
    suppress_strict               use_signal_normal  1552  +2.88%   59.1%
    suppress_strict               use_signal_cautious 455  +0.62%   47.6%
    suppress_strict               suppress_signal      62  +1.46%   45.2%

### ★ Strong threshold comparison (= 最重要)

    threshold                       count   mean      win    worst_dd
    ──────────────────────────────  ─────  ───────  ──────  ─────────
    strong_top10                       49  +6.42%   53.1%   -21.98%
    strong_top30                      131  +6.20%   64.1%   -21.98%   ★ best balance
    strong_top50                      209  +4.54%   65.6%   -34.78%
    strong_top100_regime_sector       696  +3.62%   63.5%   -34.96%

★ **`strong_top30` が最良 balance**:
- top10: count 不足 (49) + win 53% で安定性低い
- top30: count 131、mean +6.20%、win 64.1%、drawdown -22% で頭打ち
- top50: mean -1.66 pp 落ちるが drawdown は -35% に拡大 (= 1.6 倍悪化)
- top100 + regime/sector: count 5 倍だが mean は半分弱

### ★ Cautious split (variant E、20d / 5d)

    detail                          count   m5         m20        w20
    ──────────────────────────────  ─────  ─────────  ─────────  ──────
    cautious_downtrend                251   -3.36%    +0.49%    46.2%
    cautious_strong_outflow           239   +1.09%    +0.30%    52.9%
    cautious_high_volatility           15   +1.91%    +2.37%    60.0%   ★ positive
    cautious_mixed                     27   -4.25%    -1.54%    34.6%   ★ 最弱

★ **重大発見**: cautious_mixed (= 複数 negative factor 重複) が h=20d で
**-1.54%** / win 34.6% と唯一明確 negative 集合。Variant F の strict
suppress (count 62 / +1.46%) よりも厳しく弱い。
**cautious_mixed は suppress に回すべき**。

★ 一方 `cautious_high_volatility` は +2.37% / win 60% で実質 normal に
近い (count 15 と少ないが、これは normal に戻せる候補)。

### ★ Suppress validation

    candidate                          count   mean      win
    ─────────────────────────────────  ─────  ───────  ──────
    current_suppress                     224  +1.14%   49.3%
    downtrend_only                       500  +0.65%   46.9%   ← 弱
    strong_outflow_only                  303  +0.57%   51.3%   ← 弱
    downtrend_and_strong_outflow          62  +1.46%   45.2%   (= variant F)
    high_vol_and_strong_outflow           86  +2.69%   51.2%   ★ NOT 弱

★ **Suppress 候補で完全 negative なものは無し**。最弱でも +0.57%
(strong_outflow_only)。
- variant F の strict suppress (count 62 / +1.46%) は **suppress として弱い**
  (current より高い)、取りこぼし。
- high_vol_and_strong_outflow は +2.69% で完全に suppress 不適。

→ **suppress 集合の拡張余地は小さい**。current の downtrend +
contrarian_* は適当な分類。むしろ variant E の cautious_mixed を
suppress に組み込む方が効果的。

### Current variant の interpretation_detail (h=20d)

    detail                                count   mean      win
    ────────────────────────────────────  ─────  ───────  ──────
    strong_top30_uptrend_aligned            131  +6.20%   64.1%
    cautious_high_volatility_top10           42  -0.11%   43.9%   ← negative
    cautious_downtrend                      251  +0.49%   46.2%
    suppress_downtrend_outflow              224  +1.14%   49.3%
    normal_default                         1552  +2.88%   59.1%

★ `cautious_high_volatility_top10` (rank<=10 + volatility=high) は
count 42 / -0.11% で **唯一明確に negative**。これも suppress 候補。

(Variant E の cautious_high_volatility は rank 制約なしで count 15、+2.37%
で結果が異なる。両者は別集合。)

---

## Recommended rule: r2g2_recommended_v1

`_select_best_variant` が **`current`** を選定 (selection_score=0.0620、
count 131 が min_strong_count=30 を満たす中で max mean)。

    {
      "rule_version": "r2g2_recommended_v1",
      "base_rule_variant": "current",
      "use_signal_strong": {
        "conditions": [
          "regime=uptrend",
          "alignment in (aligned_strong_sector, aligned_moderate)",
          "post_cap_rank <= 30"
        ],
        "sample_count": 131,
        "mean_return_20d": 0.0620,
        "win_rate_20d": 0.6412
      },
      "use_signal_normal": {
        "conditions": ["fallback (other branches not matched)"],
        "sample_count": 1552,
        "mean_return_20d": 0.0288,
        "win_rate_20d": 0.5905
      },
      "use_signal_cautious": {
        "conditions": [
          "(volatility=high AND rank<=10)",
          "OR regime=downtrend (not also contrarian_*)"
        ],
        "sample_count": 293,
        "mean_return_20d": 0.0040,
        "win_rate_20d": 0.4586
      },
      "suppress_signal": {
        "conditions": [
          "regime=downtrend AND alignment in (contrarian_outflow, contrarian_strong_outflow)"
        ],
        "sample_count": 224,
        "mean_return_20d": 0.0114,
        "win_rate_20d": 0.4931
      }
    }

### 推奨 rule への将来 enhancement (= R2-G3 / R2-H 候補)

1. **cautious_mixed (= 複数 negative factor) を suppress に組み込む**
   (count 27 / -1.54%、明確 negative)
2. **cautious_high_volatility_top10 (rank<=10 + high vol) を suppress
   または cautious から外す** (count 42 / -0.11%、negative)
3. strong rule は **rank<=30 維持** (top50 緩和は mean -1.66pp 落ちる)
4. position sizing: strong family の worst_drawdown -22% を考慮し、
   1 銘柄あたり exposure 制限を Stage 3 で導入検討

---

## leak safety

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

---

## DB 安全確認

    target              last_modified before / after
    ──────────────────  ─────────────────────────────────────────
    fire.production.db  存在しない (= 触りようがない、安全)
    fire.develop.db     2026-05-07 18:14:26 / 2026-05-07 18:14:26
                        (R2-G/G2 期間 unchanged)
    fire.staging.db     2026-05-09 22:40:35 / 2026-05-09 22:40:35
                        (R2-G2 で touch されず、read-only 保証)

write 対象 tables:
- market_prices_daily: write 0
- market_listings: write 0
- research_watchlist_signals: write 0
- research_derived_indicators: write 0
- features tables: write 0

★ `--write` option 自体存在しない設計。

---

## 制約遵守

- ✅ DB write 一切なし
- ✅ FIRE_ENV=staging で実行、production / develop 完全無触
- ✅ market_prices_daily / market_listings / research_watchlist_signals
  全て read-only
- ✅ rule variants / interpretation_detail / recommended_rule 全て
  in-memory または artifact 出力のみ
- ✅ pre-commit Codex review 通過 × 3、CRITICAL 4 件即修正
- ✅ --no-verify 不使用、個別 commit 厳守
- ✅ scripts/seed_pattern_layer1.py の既存 modified は触らず

---

## tests

R2-G2 全 51 PASS:
- rule_refinement 33 PASS (variant config / 6 interpreter / apply_variant
  / INTERPRETATION_FAMILIES / RecommendedRule)
- runner 18 PASS (parse / select_best_variant 3 / build_joined / group /
  top buckets / strong / cautious / suppress / smoke / refusal /
  alt_horizon)

regression 2,517 PASS (全テスト)。

---

## Stage 3 Live Advisory への示唆

1. **`use_signal_strong` (rank<=30 + uptrend + aligned)** を Stage 3 の
   第一フィルタに採用 (count 131 / +6.20% / win 64% / dd -22%)
2. **`use_signal_normal`** は通常 PnL モニター (count 1,552、+2.88%)
3. **`cautious_mixed`** は実質 suppress 扱い (-1.54% / win 34.6%)
4. **`cautious_high_volatility_top10`** は除外候補 (-0.11% / win 44%)
5. **`suppress_signal`** は控えめ抑制 (= 完全 negative ではないが
   baseline 以下の集合)
6. position sizing: strong family の worst drawdown -22% を考慮し、
   1 銘柄あたり exposure 制限 + ATR-based stop を併用

---

## Go/No-Go 判断

- ✅ framework PASS (51 + 2,517 全 tests / leak 0 / DB write 0)
- ✅ recommended rule (= current) 採用可能
- ✅ Stage 3 Live Advisory への素材確定
  (use_signal_strong = rank<=30 + uptrend + aligned)
- 🔶 R2-G3 候補: cautious_mixed の suppress 組込み

---

## 次工程候補

1. **R2-G3 rule v2** (cautious_mixed → suppress、
   cautious_high_volatility_top10 → 除外)
2. **R2-H orthogonal cuts** (sector × regime cross /
   月次効果 / 決算シーズン効果)
3. **Lane integration** (F111 Daytrade Selection に
   regime + sector フィルタ追加、F119 Evaluation で
   interpretation 別 PnL)
4. **R1-B5 v1/v2 swap** (factor scoring v1 → v2 移行)

---

## 出力ファイル

    /tmp/r2g2_rule_refinement/
    ├── r2g2_rule_variant_summary.json    (256 KB、6 variants × 全 group × horizon)
    ├── r2g2_rule_variant_summary.csv
    ├── r2g2_interpretation_detail_summary.csv
    ├── r2g2_strong_threshold_comparison.csv
    ├── r2g2_cautious_split_summary.csv
    ├── r2g2_suppress_validation_summary.csv
    ├── r2g2_recommended_rules.json
    ├── r2g2_leak_check_summary.json
    └── per_variant/                       (6 variants × 2 ファイル)

---

## 参照

- commit b05c454: test(F286-R2): add rule refinement tests
- commit 581b933: chore(F286-R2): add rule refinement runner
- commit 48512cc: feat(F286-R2): add interpretation rule refinement
- 関連: F286_R2_G_regime_sector_integration.md (= R2-G initial rule)
- 関連: F286_R2_F4_broader_historical_sampling.md (= signal source)
