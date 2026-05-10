# F119 Evaluation by interpretation × sector × month

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: R2-F4 baseline signals + R2-G3 recommended_v2
> **Mode**: 完全 read-only (DB write 一切なし)
> **Result**: ★★★ 30 strong / 59 avoid / 621 caution candidates 抽出、
> R2-H と完全整合 + 3-key cut で新規 insight 発見 ★★★

---

## 目的

R2-H で取得した直交示唆を「実評価軸」として F119 evaluation 側で
受け取れるようにする。Research Lane signal × R2-G3 recommended_v2
interpretation × sector_17 × month × top_bucket の 12 cut で
PnL / 勝率 / drawdown を集計し、strong / avoid / caution を自動抽出。

Lane integration (= F111 Daytrade Selection に v2 rule 組込み) の
前提となる評価受け皿として機能。

---

## R2-G3 recommended_v2 / R2-H 示唆 再掲

    rule_version:        r2g3_recommended_v2
    selected_candidate:  cautious_mixed_to_suppress

R2-H 結果再掲 (= F119 で再評価して整合確認):

    interpretation       count    h20      win
    use_signal_strong     131  +6.20%  64.1%
    use_signal_normal    1552  +2.88%  59.1%
    use_signal_cautious   266  +0.59%  47.0%
    suppress_signal       251  +0.85%  47.7%
    overall              2200  +2.58%  56.6%

R2-H 主要 strong: top30 × strong / 5月 / 6月 / 8月 / earnings core /
情報通信 × range / 運輸 × uptrend
R2-H 主要 avoid: 3月 / 建設 × downtrend / 自動車 × downtrend /
top10 × suppress / top100 × cautious

---

## 実装サマリ (5 commit + log)

| commit | 内容 |
|---|---|
| 1e710fe | feat: F119 interpretation evaluation module (477 行) |
| f001750 | chore: F119 runner (577 行) |
| 1f96d25 | test: 28 PASS (module 16 + runner 12) |
| (本 commit) | docs: vault |
| (次 commit) | docs: log |

### モジュール構成

    evaluation/
    └── interpretation_evaluation.py   (477 行)
        ├── aggregate_by_three_keys (R2-H に未実装の 3 key 集計)
        ├── aggregate_all_cuts: 12 cut 一気集計
        │   - overall / interpretation / sector_17 / month_of_year /
        │     top_bucket
        │   - 4 つの 2-key cut
        │   - 3 つの 3-key cut
        ├── F119Thresholds (frozen dataclass)
        ├── extract_f119_insights (strong / avoid / caution)
        └── evaluate_signals_with_interpretation (orchestrator)

    scripts/jobs/
    └── run_f119_interpretation_evaluation.py   (577 行)
        ├── 完全 read-only、--write 自体作らない
        ├── --rule-version で recommended_v2 適用
        ├── 個別 output ファイル指定:
        │   --output-json / --output-csv / --insights-json /
        │   --strong-csv / --avoid-csv / --caution-csv
        └── leak-safe (R2-G/G2/G3/H と同じ機構)

    tests/  (28 PASS)
    ├── tests/evaluation/test_interpretation_evaluation.py (16)
    └── tests/scripts/jobs/test_run_f119_interpretation_evaluation.py (12)

---

## Codex CRITICAL 2 件 全て即時対応

### feat 1 - #1: count gate に valid_return_count 不在

count >= min_count だけでは future price 欠損で valid が小さい
集合が strong/avoid に分類される可能性。

**修正**: `meets_count_gate = (cnt >= min_count and valid_h1 >= min_count)`
の double gate に変更。

### chore - #2: `valid_return_count_h20` hard-coded

`--horizons` から 20 を外すと CSV writer の field
`valid_return_count_h{h_primary}` と JSON の固定 key が不整合。

**修正**: `_summary_to_record` で `valid_return_count_h{h1}` に
動的化。tests で h_primary=5 でも整合確認。

---

## 12 Cuts

### 単独 (5)
1. `overall`
2. `interpretation`
3. `sector_17`
4. `month_of_year`
5. `top_bucket`

### 2-key (4)
6. `interpretation × sector_17`
7. `interpretation × month_of_year`
8. `sector_17 × month_of_year`
9. `top_bucket × interpretation`

### 3-key (3)
10. `interpretation × sector_17 × month_of_year`
11. `top_bucket × interpretation × sector_17`
12. `top_bucket × interpretation × month_of_year`

---

## smoke 結果 (staging / 22 base_dates / r2g3_recommended_v2 / top100 / min_count=20)

### Insight summary

    strong_candidates:    30
    avoid_candidates:     59
    caution_candidates:  621

### ★ Top 15 strong candidates (h=20d)

    cut                                 group_key                                     count   h20      win
    ─────────────────────────────────  ────────────────────────────────────────── ────── ──────── ──────
    interp_sector_17_month              normal × 情報通信 × 05                          25  +11.42%  88.0%   ★ 最強
    top_bucket_interp_month             top50 × normal × 08                            20  +10.76%  85.0%
    interp_sector_17_month              normal × 情報通信 × 08                          30  +10.01%  80.0%
    interp_month                        normal × 08                                    87  +9.47%  78.2%
    top_bucket_interp_month             top50 × normal × 05                            20  +9.44%  85.0%
    interp_month                        strong × 02                                    27  +9.31%  63.0%
    sector_17_month                     情報通信 × 05                                    30  +9.04%  80.0%
    top_bucket_interp_sector_17         top100 × normal × 自動車・輸送機                  27  +7.42%  61.5%
    interp_month                        normal × 05                                    90  +7.14%  71.9%
    top_bucket_interp_month             top100 × normal × 08                           50  +7.09%  74.0%
    top_bucket_interp_month             top30 × normal × 06                            26  +7.08%  69.2%
    month_of_year                       05                                            100  +7.04%  71.7%
    top_bucket_interp_month             top100 × normal × 05                           50  +6.95%  70.0%
    sector_17_month                     不動産 × 06                                     22  +6.88%  81.8%
    interp_sector_17_month              normal × 不動産 × 06                            22  +6.88%  81.8%

★ **3-key 切り口で新規 insight**:
- `情報通信 × 5月 × use_signal_normal` (count 25, +11.42%, win 88%)
  → 単独で見ても過去最高水準
- `top50 × normal × 8月` (count 20, +10.76%, win 85%)
  → top50 まで広げても 8月 normal は強い
- `top50 × normal × 5月` (count 20, +9.44%, win 85%)

★ R2-H 既知 strong との整合:
- 5月 +7.04% / win 71.7% (= R2-H 一致)
- 6月 強い (top30 × normal × 6月 +7.08%)
- 8月 強い h20 (h5 警告は別途検証)

### ★ Top 10 avoid candidates (h=20d)

    cut                                 group_key                                     count   h20      win
    ─────────────────────────────────  ────────────────────────────────────────── ────── ──────── ──────
    sector_17_month                     不動産 × 03                                    21  -5.36% 33.3%   ★ 最弱
    top_bucket_interp_month             top50 × normal × 03                           40  -5.17% 32.5%
    interp_month                        cautious × 10                                  73  -3.36% 23.3%
    sector_17_month                     情報通信 × 03                                   60  -2.95% 40.0%
    interp_sector_17_month              normal × 情報通信 × 03                          60  -2.95% 40.0%
    interp_sector_17_month              cautious × 情報通信 × 10                        27  -2.79% 22.2%
    top_bucket_interp_month             top100 × cautious × 10                        37  -2.71% 29.7%
    top_bucket_interp_month             top30 × normal × 03                           33  -2.33% 39.4%
    interp_sector_17_month              normal × 情報通信 × 09                          30  -2.25% 43.3%
    sector_17_month                     情報通信 × 10                                   60  -1.90% 33.3%

★ **3月の弱さは sector 関係なく顕著**:
- 不動産 × 3月: -5.36% / win 33%
- top50 × normal × 3月: -5.17% / win 33%
- 情報通信 × 3月: -2.95% / win 40%
- top30 × normal × 3月: -2.33% / win 39%
→ 3月新規 entry は完全見送り推奨

★ **9-10月の cautious は弱い**:
- cautious × 10月: -3.36% / win 23%
- cautious × 情報通信 × 10月: -2.79% / win 22%
- top100 × cautious × 10月: -2.71% / win 30%

### Interpretation cut (h=20d / h=5d、= R2-G3 完全一致)

    family               count   h20             h5
    use_signal_strong     131  +6.20% / 64.1%  +1.32% / 58.8%
    use_signal_normal    1552  +2.88% / 59.1%  +1.21% / 56.8%
    use_signal_cautious   266  +0.59% / 47.0%  -3.06% / 24.6%
    suppress_signal       251  +0.85% / 47.7%  -4.48% / 22.0%

★ F119 と R2-G3 の interpretation 集計が完全一致 (= rule v2 適用ロジックの
妥当性確認)。

---

## leak safety

    {
      "regime": {"violations": 0, "passed": true,
                 "max_price_date_used_for_regime": "2026-02-27"},
      "sector_flow": {"violations": 0, "passed": true,
                       "max_price_date_used_for_sector_flow": "2026-02-27"}
    }

---

## DB 安全確認

    target              before              after               status
    fire.production.db  存在しない           存在しない            (= 触りようがない)
    fire.develop.db     2026-05-07 18:14:26 2026-05-07 18:14:26 unchanged
    fire.staging.db     2026-05-09 22:40:35 2026-05-09 22:40:35 unchanged

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
  CRITICAL 2 件即修正
- ✅ --no-verify 不使用、個別 commit 厳守
- ✅ `scripts/seed_pattern_layer1.py` の既存 modified は触らず
- ✅ `simulation/research_lane/historical_indicators.py` も触らず

---

## tests

F119 全 28 PASS:
- module 16 PASS (3-key aggregator / aggregate_all_cuts / dynamic field /
  3 タグ抽出 / valid count gate verify / Thresholds / orchestrator)
- runner 12 PASS (parse / resolve / build_joined / runner smoke /
  alt_horizon CSV 整合 / refusal cases)

regression 2,659 PASS (= F119 commit 前)
regression 2,687 PASS (= F119 commit 後、= 2,659 + 28)

---

## Stage 3 Live Advisory への示唆

R2-H で確立した示唆を F119 軸で再評価し、3-key cut で精度向上:

### 強く採用する条件 (count>=20 / h20 > overall+2pp / win>=55%)

1. **`use_signal_normal × 情報通信 × 5月`** (count 25, +11.42%, win 88%)
2. **`top50 × normal × 8月`** (count 20, +10.76%, win 85%)
3. **`use_signal_normal × 情報通信 × 8月`** (count 30, +10.01%, win 80%)
4. **`use_signal_normal × 8月`** (count 87, +9.47%, win 78%)
5. **`use_signal_strong × 2月`** (count 27, +9.31%, win 63%)
6. **`top30 × normal × 6月`** (count 26, +7.08%, win 69%)
7. **`不動産 × 6月`** (count 22, +6.88%, win 82%) (※ 但し dd 注意)

### 通常レビュー条件 (= 平常 PnL モニター)

- `use_signal_normal × <他の月>` (大半の組合せ)
- 5月 / 6月 / 8月の normal 系全般

### 慎重条件 (= 5d 注意)

- caution_candidates 621 件 中、特に高 dd / 短期負 / count 不足
- caution_candidates の主因: 「h20 ok だが count 不足」が圧倒的多数
  (= 3-key cut で count<20 の組合せが多い)

### 見送り条件

- **3月新規 entry は完全見送り**:
  - 不動産 × 3月: -5.36% / win 33%
  - 情報通信 × 3月: -2.95% / win 40%
  - top50 × normal × 3月: -5.17% / win 33%
  - top30 × normal × 3月: -2.33% / win 39%
- **9-10月の cautious 系**:
  - cautious × 10月: -3.36% / win 23%
  - cautious × 情報通信 × 10月: -2.79% / win 22%

### Position sizing 注意

- 不動産系 sector × month で worst_dd 注意 (= R2-H 不動産系全 regime
  -25〜-36% との整合)
- top10 × strong は count 49 で安定性低い (= F119 では 1-key cut で
  確認、 R2-H 既知)

### 5d / 20d 保有期間

- normal / strong: 5d でも positive (h5 +1.21% / +1.32%)
- cautious / suppress: 5d 大きく negative (-3% / -4.5%)、20d 保有徹底
- 8月の strong + normal: h5 短期含み損許容で 20d 保有

---

## Go/No-Go 判断

- ✅ framework PASS (28 + 2,687 全 tests / leak 0 / DB write 0)
- ✅ R2-H 示唆と完全整合 (= rule v2 適用ロジックの妥当性確認)
- ✅ 3-key cut で新規 insight 発見:
  - 情報通信 × 5月 × normal +11.42%/win 88%
  - top50 × normal × 8月 +10.76%/win 85%
  - 3月の sector 別弱性が明確化 (不動産 × 3月 -5.36%)
- ✅ Stage 3 Live Advisory 接続準備完了
- 🔶 caution_candidates 621 件 = 多くは count<20 の信頼度低、
  Lane integration では count threshold を厳しく設定推奨

---

## 出力ファイル

    /tmp/f119_eval_summary.json           (1.7 MB、全 12 cut + insights)
    /tmp/f119_eval_summary.csv            (575 KB、全 cut × horizon)
    /tmp/f119_eval_insights.json          (420 KB、3 タグ詳細)
    /tmp/f119_eval_strong_candidates.csv  (6.9 KB、30 件)
    /tmp/f119_eval_avoid_candidates.csv   (15 KB、59 件)
    /tmp/f119_eval_caution_candidates.csv (151 KB、621 件)

---

## 次工程候補

1. **Lane integration** (F111 Daytrade Selection に
   r2g3_recommended_v2 + F119 strong 条件 (情報通信 × 5月 × normal
   等) を組込み、F119 Evaluation で interpretation 別 PnL を継続記録)
2. **R2-G4 5d rule** (= 5d 短期保有用、cautious/suppress を normal 化、
   対象月限定)
3. **R1-B5 v1/v2 swap** (factor scoring v1 → v2 移行、F119 評価軸で
   v2 採用判断)
4. **caution_candidates 621 件 の整理** (= count<20 を除外した
   絞り込み版を別 artifact 化検討)

---

## 参照

- commit 1e710fe: feat(F119): add interpretation evaluation module
- commit f001750: chore(F119): add interpretation evaluation runner
- commit 1f96d25: test(F119): add interpretation evaluation tests
- 関連: F286_R2_H_orthogonal_cuts.md (= cross-section validation)
- 関連: F286_R2_G3_rule_finalization.md (= recommended_v2 確定)
- 関連: F286_R2_F4_broader_historical_sampling.md (= signal source)
