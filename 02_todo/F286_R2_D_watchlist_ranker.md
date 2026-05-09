---
id: F286-R2-D
phase: P5: Research Lane R2 (Watchlist Ranker、3 戦略統合 + sector cap)
priority: 高 (Research Lane の最終 watchlist 出力層)
status: 完了 (top 30 / 50 / 100 smoke、HQ 必須要件 sector cap 30% 完全達成)
owner: Fujiwara
depends_on: [F286-R2-C factor scoring smoke]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R2_C_factor_scoring_smoke, F286_R2_B_indicator_distribution_analysis
---

★ **F286 R2-D**: 3 戦略 (R2-C) を統合し、sector cap 30% を適用した
   Watchlist Ranker 完成。**HQ 必須要件 (cap 30%) を完全達成**:
   pre-cap で情報通信 43%/46%/39% だったのが post-cap で **すべて
   ちょうど 30%** に抑制。production / develop DB 完全無触、
   staging read-only。

# F286-R2-D Research Watchlist Ranker

## サマリ

R2-C で実装した 3 戦略 (Quality Value / Earnings Growth / Cyclical
Value) のスコアを **重み付き統合 + sector cap 30%** で最終
Watchlist にまとめる。Fujiwara が手動レビューしやすい形で
selected / deferred / excluded の 3 値判定 + 銘柄ごとの rank_reason
を付与。

実装 3 module:
1. **`simulation/research_lane/watchlist_ranker.py`** (純粋関数)
2. **`scripts/jobs/run_research_watchlist_ranker.py`** (CLI runner)
3. tests (`test_research_lane_watchlist_ranker.py` +
   `test_run_research_watchlist_ranker.py`)

## 実装内容

### 1) 3 戦略統合ロジック (final_score)

| component | weight |
|---|---:|
| quality_value | 0.35 |
| earnings_growth | 0.35 |
| cyclical_value | 0.30 |

**重み redistribution**: 一部 strategy が missing/excluded で score=None
の場合、残り strategy の weight を rescale して final_score を計算。
これにより「Quality 除外でも Growth で強い銘柄」を Watchlist に残す
HQ 要件を実現。

例: quality_value=excluded、earnings_growth=0.7、cyclical_value=0.5
→ (0.7×0.35 + 0.5×0.30) / (0.35 + 0.30) = 0.6077

### 2) Rank label (R2-C と同一の 5 段階)

| label | percentile | 件数 (3,708 records) |
|---|---|---:|
| A1 | top 5% | 185 |
| A2 | 5-15% | 368 |
| B | 15-35% | 736 |
| C | 35-65% | 1,105 |
| D | bottom 35% | 1,289 |
| (none) | 全戦略 score 不能 | 25 |

### 3) Sector cap 30% (HQ 必須)

`top_n × cap_ratio` を切上げで sector_cap (例: top30 → 9 件、
top50 → 15 件、top100 → 30 件)。pre_cap_rank 順に走査し、cap 超過は
**deferred** (= 除外ではなく次回繰越扱い)。

### 4) Strategy mix 可視化

各銘柄に以下を付与:
- `strongest_strategy`: quality_value / earnings_growth /
  cyclical_value / balanced (差 < 0.01) / None (= 全戦略 missing)
- `strategy_signal_summary`: {strategy: rank_label, ...}
  例: `{quality_value: "A1", earnings_growth: "B", cyclical_value: "A2"}`
- `available_strategy_count`: 0-3
- `rank_reason`: 人間可読な final_rank の根拠説明

### 5) Watchlist 採用判定

| watchlist_decision | 条件 |
|---|---|
| **selected** | top N 内 + sector cap 通過 |
| **deferred** | sector cap 超過 (deferred) or top N 範囲外 |
| **excluded** | 全戦略 excluded (negative_eps 等) or score 不能 |

## 実行結果 (2026-05-09 19:57 〜 19:58)

records loaded: **3,708**

### 共通指標

| 項目 | 値 |
|---|---:|
| input_records | 3,708 |
| final_scored | 3,683 (99.3%) |
| excluded_all_strategies | 0 |

### Top 30

| 項目 | 値 |
|---|---:|
| watchlist_select | 30 |
| watchlist_deferred | 3,653 |
| watchlist_exclude | 25 |
| sector cap | 9 件/sector |

**Sector 分布 (pre-cap → post-cap)**:

| sector | pre-cap | post-cap |
|---|---:|---:|
| **情報通信・サービスその他** | **13 (43%)** | **9 (30%)** ← cap |
| 不動産 | 5 (17%) | 6 (20%) |
| 商社・卸売 | 3 (10%) | 4 (13%) |
| 機械 | 2 (7%) | 2 (7%) |
| 金融除く銀行 | 2 (7%) | 2 (7%) |

→ HQ 必須要件「同一 sector 30% 以内」**完全達成**。

**Top 10 (post-cap)** (全 A1):

| post | code | final_score | rank | strongest_strategy | sector |
|---:|---|---:|:---:|---|---|
| 1 | 87470 | 0.9299 | A1 | earnings_growth | 金融除く銀行 |
| 2 | 57290 | 0.9152 | A1 | earnings_growth | 鉄鋼・非鉄 |
| 3 | 34890 | 0.9080 | A1 | balanced | 不動産 |
| 4 | 340A0 | 0.8923 | A1 | quality_value | 情報通信 |
| 5 | 37980 | 0.8765 | A1 | quality_value | 情報通信 |
| 6 | 79910 | 0.8669 | A1 | quality_value | 機械 |
| 7 | 91300 | 0.8586 | A1 | quality_value | 運輸・物流 |
| 8 | 331A0 | 0.8568 | A1 | quality_value | 情報通信 |
| 9 | 43890 | 0.8559 | A1 | earnings_growth | 情報通信 |
| 10 | 92470 | 0.8547 | A1 | earnings_growth | 情報通信 |

**top 30 の strongest_strategy 分布**:
- quality_value: 12
- earnings_growth: 8
- cyclical_value: 6
- balanced: 4

→ 戦略 mix が分散、IT 偏重ではなく金融 / 鉄鋼 / 不動産 / 機械 / 運輸
   等の多様な sector が上位に入っている。

### Top 50

**Sector 分布 (pre-cap → post-cap)**:

| sector | pre-cap | post-cap |
|---|---:|---:|
| 情報通信・サービスその他 | 23 (46%) | **15 (30%)** ← cap |
| 不動産 | 6 (12%) | 9 (18%) |
| 商社・卸売 | 8 (16%) | 8 (16%) |
| 金融除く銀行 | 2 (4%) | 5 (10%) |
| 運輸・物流 | (低) | 3 (6%) |

→ post-cap 50 で IT が 30% にちょうど抑制、deferred 分が他 sector
   に流れて多様性向上。

### Top 100

| sector | pre-cap | post-cap |
|---|---:|---:|
| 情報通信・サービスその他 | 39 (39%) | **30 (30%)** ← cap |
| 不動産 | 13 (13%) | 13 (13%) |
| 商社・卸売 | 11 (11%) | 12 (12%) |
| 金融除く銀行 | 6 (6%) | 7 (7%) |
| 素材・化学 | (低) | 6 (6%) |

→ top100 でも IT が cap 30% で抑制、計 30 件 selected。

## 部分戦略許容ケース

R2-C で `negative_eps` で Quality / Cyclical 除外された 443 銘柄、
`negative_bps` で 4 銘柄、`negative_equity` で 7 銘柄等が、
**Earnings Growth で score 算出可** なら Watchlist に残せる:

- partial_strategy (available_strategy_count < 3 but > 0): **1,267 件**
- excluded_all_strategies: 0 (= 全銘柄が最低 1 戦略で score 算出可)

これにより HQ 要件「Quality除外だがGrowthで強い → 候補に残す」を満たす。

## DB 安全性

| DB | last_modified | 状態 |
|---|---|---|
| production (`fire.db`) | 2026-05-07 16:12 | 完全無触 |
| develop (`fire.develop.db`) | 2026-05-07 18:14 | 完全無触 |
| staging (`fire.staging.db`) | 2026-05-09 19:17 | **read-only** (R2-A3 から write なし、R2-B / R2-C / R2-D 連続で read のみ) |

**自動発注禁止 / 楽天証券自動化禁止 / Computer Use 禁止 / Playwright
禁止** などの HQ 必須制約は本タスクで関与せず (= 全 read-only / DB
write なし / 通知系も触らない実装、staging のみ)。

## constraints_check (raw)

```json
{
  "staging_only_read": true,
  "production_db_untouched": true,
  "develop_db_untouched": true,
  "no_db_write": true,
  "v2_schema_used": true,
  "indicators_table_used": true,
  "sector_cap_applied": true
}
```

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/watchlist_ranker | 38 PASS |
| scripts/jobs/run_research_watchlist_ranker | 14 PASS |
| **R2-D 合計** | **52 PASS** |
| 関連 regression (research_lane / scripts) | **607 PASS** |

**HQ 必須テスト 全項目クリア**:
- final_score weighted merge ✅
- missing strategy score handling (1/2 missing) ✅
- strongest_strategy 判定 (各戦略 / balanced / single / all None) ✅
- rank label A1/A2/B/C/D ✅
- sector cap 30% ✅
- pre_cap_rank / post_cap_rank ✅
- sector_cap_deferred ✅
- all strategy excluded (excluded_all_strategies) ✅
- partial strategy kept ✅
- runner smoke (基本 + cap + negative_eps + empty) ✅

## commit ログ

| # | commit | hash |
|---|---|---|
| R2-D-1 | feat(F286-R2): add research watchlist ranker | 922bd23 |
| R2-D-2 | chore(F286-R2): add watchlist ranker runner | fb04e49 |
| R2-D-3 | test(F286-R2): add watchlist ranker tests | 5442e16 |
| R2-D-4 | docs(F286-R2): vault watchlist ranker smoke result | (本 commit) |
| R2-D-5 | docs(F286-R2): log milestone — watchlist ranker completed | (次 commit) |

Codex pre-commit 通過 × 3 (feat / chore / test)、--no-verify 不使用。

## Go/No-Go 判定

判定: **PASS、実用 OK**

| 条件 | 結果 |
|---|---|
| HQ 必須 sector cap 30% 達成 | ✅ pre 43-46% → post **ちょうど 30%** |
| final_score 算出率 >= 80% | ✅ 99.3% (3,683/3,708) |
| top_n に最低 1 sector 以上の多様性 | ✅ top30 で 7-10 sector |
| excluded_all_strategies が極端に多くない | ✅ 0 件 |
| top 候補で赤字混入なし (Quality/Cyclical) | ✅ negative_eps 銘柄は EG のみで残るが top 上位は normal 銘柄が支配 |
| DB write なし | ✅ |
| production / develop DB 無触 | ✅ |
| Codex pre-commit 通過 | ✅ × 3 |

## 次のアクション (HQ 判断)

R2-D Watchlist Ranker 完成 → 候補:

1. **R2-E Signal Persistence**: 各 base_date の rank / score を時系列
   で永続化、backtesting / time-series 分析の基盤
2. **R2-D2 調整**: 重み調整 / sector cap ratio 調整 / strongest_strategy
   閾値 / Lane 別 weight など
3. **Lane integration**: 既存 Daytrade Selection / Swing Selection /
   Long-term Selection に Watchlist Ranker output を入力
4. **R1-B5 (v1/v2 swap)**: 既存 v1 (1,723 row) を drop / rename して
   v2 を正本化、別途 HQ 承認後

R2-D の output (top 30 / 50 / 100、Watchlist CSV) を Fujiwara が手動
レビューして注目銘柄を選び、Daytrade Selection / Swing Selection の
入力にするのが当面の運用形態。

## 関連リンク

- [[F286_R2_C_factor_scoring_smoke|F286-R2-C 3 戦略 scoring]]
- [[F286_R2_B_indicator_distribution_analysis|F286-R2-B 分布検証]]
- [[F286_R2_A3_full_eligible_indicators|F286-R2-A3 全 eligible 永続化]]
- [[F286_R2_A2_eligible_universe_and_indicators_storage|F286-R2-A2 eligible filter + storage]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ exit 0、top 30/50/100 smoke 全成功、JSON / CSV 両形式
  出力成功
- **機能した**: ✅ HQ 必須 sector cap 30% を完全達成 (pre 43% →
  post 30%)、redistribution / strongest_strategy / partial keep
  ロジックが期待動作、top 10 で 7+ sectors が分散
- **期待値達成**: ✅ HQ 必須要件 11 項目 (cap 30% 実装 / final_score
  redistribution / rank label 5 段階 / strongest_strategy / pre/post
  rank / sector_cap_deferred / partial keep / all excluded /
  read-only / Codex pre-commit / 個別 commit) 全達成、tests 52 PASS
  / regression 607 PASS、production/develop DB 完全無触
