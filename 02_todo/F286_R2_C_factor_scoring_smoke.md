---
id: F286-R2-C
phase: P5: Research Lane R2 (factor scoring 3 戦略)
priority: 高 (Research Lane "何を買うか" 判定の MVP)
status: 完了 (3 戦略 score 計算成功、HQ 解釈待ち: sector_concentration 発火)
owner: Fujiwara
depends_on: [F286-R2-B distribution analysis]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R2_B_indicator_distribution_analysis, F286_R2_A3_full_eligible_indicators
---

★ **F286 R2-C**: Quality Value / Earnings Growth / Cyclical Value の
   3 戦略スコアリング MVP。R2-B の前処理推奨案 (winsorize p95 / sector
   relative / profit_yoy cap 3.0 / 銀行 op_margin 除外 / negative_eps
   除外) を全て取込。R2-A3 永続化済 3,708 銘柄に対し read-only で
   score を計算 → top 30 / sector 偏り / Go/No-Go 判定。

# F286-R2-C Quality Value / Earnings Growth / Cyclical Value scoring MVP

## サマリ

Research Lane で「何を買うか」を判定する初期スコアリング層。
research_derived_indicators 3,708 row を read-only で取得し、R2-B で
確定した前処理 + 各戦略の重み付け平均で 3 系統の score (= [0, 1])
を算出。**A1/A2/B/C/D の 5 段階 rank** + sector 偏り検出 + 異常
warning + R2-D 進行可否を 1 run で出す。

## HQ 必須前処理の取込確認

| 項目 | 取込状況 |
|---|---|
| PER / PBR / operating_margin = sector_relative + p95 winsorize | ✅ `sector_relative()` + `winsorize(0.05, 0.95)` |
| ROE / sales_yoy = absolute 評価可 | ✅ `absolute_percentile()` |
| profit_yoy cap 3.0 | ✅ `PROFIT_GROWTH_YOY_MAX_CAP = 3.0`、`cap_above` |
| 銀行 sector (15) は operating_margin 評価対象外 | ✅ `BANK_SECTOR_CODE = "15"`、net_margin で代替 |
| negative_eps / negative_equity を Quality 系から除外 | ✅ `_has_quality_blocking_reason` で skip_reasons チェック |
| skip_reasons_json を尊重 | ✅ runner で skip_reasons パース → record に注入 |

## 3 戦略の設計

### Quality Value (低 PER × 低 PBR × 高 ROE × 営業利益率)

| component | weight | source |
|---|---:|---|
| per_inverted (sector relative、低=良) | 0.30 | sector_pct → invert |
| pbr_inverted (sector relative、低=良) | 0.20 | sector_pct → invert |
| roe_abs (winsorize / cap 済) | 0.30 | absolute_percentile |
| operating_margin_sec (sector relative) | 0.20 | sector_pct |

**銀行 (sector=15) のみ**: operating_margin → net_margin_sec で代替
(weights は同 0.20)。

**除外条件**: `negative_eps` / `negative_equity` / `negative_bps`
(skip_reasons_json で判定)。

### Earnings Growth (売上 YoY × 利益 YoY × ROE × net_margin sector相対)

| component | weight | source |
|---|---:|---|
| sales_yoy_abs | 0.30 | cap 1.0 + winsorize p5/p95 + abs pct |
| profit_yoy_abs | 0.30 | cap 3.0 + winsorize p5/p95 + abs pct |
| roe_abs | 0.20 | 同上 |
| net_margin_sec | 0.20 | sector relative |

**除外条件**: なし (赤字 EPS でも成長率は意味があるため、Quality
と異なり filter しない)。missing は skipped (excluded=False)。

### Cyclical Value (低 PBR × 低 PER × ROE 下限 × 循環セクター boost)

| component | weight | source |
|---|---:|---|
| pbr_inverted (sector relative) | 0.35 | 中核 (低 PBR が cyclical の本質) |
| per_inverted (sector relative) | 0.25 | |
| roe_abs | 0.20 | absolute_percentile |
| cyclical_sector_boost | 0.20 | 循環 sector で 1.0、それ以外 0.5 |

循環 sector (R2-B 観察より): **不動産 (17) / 自動車・輸送機 (6) /
素材・化学 (4) / 鉄鋼・非鉄 (7) / 金融除く銀行 (16)**

**除外条件**: 
- Quality 系の blocking reasons (`negative_eps` / `negative_bps` / `negative_equity`)
- ROE < 5% で `roe_below_0.05` (= 循環株は最低限の収益性が必要)

### Rank labels

| label | percentile range | 説明 |
|---|---|---|
| A1 | top 5% | watchlist 最優先 |
| A2 | 5-15% | 注目候補 |
| B | 15-35% | 監視対象 |
| C | 35-65% | 中位 |
| D | bottom 35% | 弱含み |

## 実行結果 (2026-05-09 19:41)

records loaded: **3,708** (1 row per code、最新 base_date)

### Quality Value

| 項目 | 値 |
|---|---:|
| processed | 3,708 |
| scored | 3,206 (86.5%) |
| excluded | 447 |
| skipped | 55 |

**excluded 内訳**:
- `negative_eps`: 443 (赤字決算の 12% を除外)
- `negative_bps`: 4 (債務超過候補)

**skipped 内訳**: missing components (= operating_margin / pbr 等が
data 欠損)。

**top 10 (全 A1)**:
```
37120  score=0.9438  per_inv=0.997  roe_abs=0.841
340A0  score=0.9348  per_inv=0.996  roe_abs=0.933
79910  score=0.9293  per_inv=1.000  roe_abs=0.899
87470  score=0.9288  per_inv=0.976  roe_abs=0.975
48280  score=0.9244  per_inv=0.986  roe_abs=0.975
37980  score=0.9227  per_inv=0.999  roe_abs=0.842
61960  score=0.9217  per_inv=0.979  roe_abs=0.932
91270  score=0.9179  per_inv=0.980  roe_abs=0.938
74630  score=0.9094  per_inv=0.989  roe_abs=0.786
331A0  score=0.9053  per_inv=0.983  roe_abs=0.975
```

**top 30 sector 分布**:
- 情報通信・サービスその他: **16 (53%)** ← 警告閾値超過
- 機械 / 運輸・物流 / 電気・ガス / 建設・資材: 各 2
- 金融除く銀行 / 小売 / 鉄鋼・非鉄: 各 1

### Earnings Growth

| 項目 | 値 |
|---|---:|
| processed | 3,708 |
| scored | 3,623 (97.7%) |
| excluded | 0 |
| skipped | 85 |

**top 10 (全 A1)**:
```
90240  score=0.9761  roe_abs=0.975
241A0  score=0.9652  roe_abs=0.975
81360  score=0.9595  roe_abs=0.975
57290  score=0.9574  roe_abs=0.919
73780  score=0.9559  roe_abs=0.975
68570  score=0.9547  roe_abs=0.975
51360  score=0.9482  roe_abs=0.975
73180  score=0.9453  roe_abs=0.975
87470  score=0.9424  roe_abs=0.975
92710  score=0.9380  roe_abs=0.975
```

**top 30 sector 分布**:
- 情報通信・サービスその他: **16 (53%)** ← 警告閾値超過
- 商社・卸売 / 電機・精密 / 自動車・輸送機 / 不動産: 各 2
- 運輸・物流 / 鉄鋼・非鉄 / 金融除く銀行: 各 1

### Cyclical Value

| 項目 | 値 |
|---|---:|
| processed | 3,708 |
| scored | 2,484 (67.0%) |
| excluded | 1,196 |
| skipped | 28 |

**excluded 内訳**:
- `roe_below_0.05`: 749 (= ROE 5% 未満の循環株は除外)
- `negative_eps`: 443
- `negative_bps`: 4

**top 10 (全 A1)**:
```
86990  score=0.9484  per_inv=0.988  roe_abs=0.818
41190  score=0.9419  per_inv=0.992  roe_abs=0.945
87290  score=0.9365  per_inv=1.000  roe_abs=0.683
29910  score=0.9321  per_inv=0.992  roe_abs=0.929
87470  score=0.9165  per_inv=0.976  roe_abs=0.975
34890  score=0.9163  per_inv=1.000  roe_abs=0.975
49140  score=0.9160  per_inv=1.000  roe_abs=0.593
32800  score=0.9146  per_inv=0.967  roe_abs=0.777
41160  score=0.9024  per_inv=0.996  roe_abs=0.523
96330  score=0.9022  per_inv=0.975  roe_abs=0.908
```

**top 30 sector 分布** (★ 設計通り循環セクターが上位):
- **素材・化学: 8** (循環 boost 適用)
- **不動産: 8** (循環 boost 適用)
- **金融除く銀行: 4** (循環 boost 適用)
- **鉄鋼・非鉄: 4** (循環 boost 適用)
- 情報通信・サービスその他: 3 (boost なし、ROE で押し上げ)
- 小売 / 食品 / 自動車・輸送機: 各 1

→ Cyclical Value は **sector boost が機能** し、循環セクターが top
   30 の **80% (24/30)** を占める。warning なし。

## sector_concentration warning の解釈

Quality Value / Earnings Growth で「1 sector が top 30 の 53%」警告
が発火。これは **東証 universe の sector 分布**を反映した自然な結果:

- 情報通信・サービス他: eligible 3,752 中 **約 33%** (R2-B の n=1,228)
- → Quality / Earnings Growth で IT 由来の高 score 銘柄が多くなる
  のは構造的

**HQ 解釈オプション**:
1. **このまま受容**: top 30 = "watchlist 候補" として、IT 偏重も
   そのまま採用 (= IT が好調なら IT 中心の戦略も妥当)
2. **R2-D で sector cap 導入**: 1 sector あたり top の 30% 上限など、
   分散制約をスコア後に適用 (= portfolio diversification 観点)
3. **Cyclical Value 同様の sector boost**: Quality / Growth 戦略で
   非 IT セクターを bonus する weight 調整 (= 設計判断)

R2-D / Watchlist Ranker 実装で 2 が現実的。本 R2-C は **生スコアの
妥当性確認が目的**で、運用層の sector cap は後続。

## Go/No-Go 判定

判定結果: **fail (warnings あり)**

| 条件 | 値 | 判定 |
|---|---|---|
| Quality Value scored >= 50% | 86.5% | OK |
| Earnings Growth scored >= 50% | 97.7% | OK |
| Cyclical Value scored >= 50% | 67.0% | OK |
| Quality Value warning なし | 1 件 (sector_concentration) | NG |
| Earnings Growth warning なし | 1 件 (sector_concentration) | NG |
| Cyclical Value warning なし | 0 件 | OK |

**HQ 判断要請**: sector_concentration warning は **universe 構造由来**
で実質的に避けられない。R2-D で sector cap 運用を導入する前提なら
**実用上 PASS**。HQ 承認後に R2-D へ進行。

## constraints_check (raw)

```json
{
  "staging_only_read": true,
  "production_db_untouched": true,
  "develop_db_untouched": true,
  "no_db_write": true,
  "v2_schema_used": true,
  "indicators_table_used": true
}
```

production / develop DB last_modified May 7 完全無触。staging も
read-only (R2-A3 から write なし、R2-B / R2-C 連続で read のみ)。

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/preprocessing | 26 PASS |
| simulation/research_lane/factor_scoring | 29 PASS |
| scripts/jobs/score_factor_strategies | 19 PASS |
| **R2-C 合計** | **74 PASS** |
| 関連 regression (research_lane / scripts) | **555 PASS** |

## commit ログ

| # | commit | hash |
|---|---|---|
| R2-C-1 | feat(F286-R2): add research lane factor scoring | 5fa8778 |
| R2-C-2 | chore(F286-R2): add factor scoring runner | ff56618 |
| R2-C-3 | docs(F286-R2): vault factor scoring smoke result | (本 commit) |
| R2-C-4 | docs(F286-R2): log milestone — factor scoring smoke completed | (次 commit) |

## 次のアクション (HQ 判断)

R2-C MVP 完成 → R2-D / Watchlist Ranker 候補:

1. **R2-D Watchlist Ranker**: 3 戦略の score 統合 + sector cap +
   Lane / strategy 紐付け、最終 watchlist (= Daytrade Selection /
   Swing Selection / Long-term Selection への入力) 生成
2. **R2-E Signal 永続化**: 各日次 base_date での score / rank を
   研究テーブルに保存、時系列 backtesting 用基盤
3. **sector cap 運用設計**: Quality / Earnings で 1 sector あたり
   top の 30% 上限、超過分は次位に繰り越し
4. **追加 strategy 検討**: Dividend Quality / Momentum / 等

R2-C の output (top 30 × 3 戦略 = 計 90 銘柄、重複あり) を Fujiwara
さんが手動レビューして watchlist に取り込むのが当面の運用形態。

## 関連リンク

- [[F286_R2_B_indicator_distribution_analysis|F286-R2-B 分布検証]]
- [[F286_R2_A3_full_eligible_indicators|F286-R2-A3 全 eligible 永続化]]
- [[F286_R2_A2_eligible_universe_and_indicators_storage|F286-R2-A2 eligible filter + storage]]
- [[F286_R2_A1_derived_indicators_smoke|F286-R2-A1 派生指標 MVP]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ exit 0、3 戦略 score 計算 + JSON / コンソール report
- **機能した**: ✅ Quality 86.5% / Growth 97.7% / Cyclical 67.0%
  scored、negative_eps 443 件で除外、Cyclical で循環 sector boost
  が機能 (top 30 の 80% が循環セクター)、A1/A2/B/C/D rank が付与
- **期待値達成**: ✅ HQ 必須前処理 6 項目 全取込、production /
  develop DB 完全無触、staging read-only、tests 74 PASS / regression
  555 PASS、Codex pre-commit 通過 × 2、sector_concentration 警告は
  universe 構造由来で R2-D で運用 cap で対処予定 (HQ 解釈待ち)
