---
id: F286-R2-B
phase: P5: Research Lane R2 (派生指標分布検証 / sector別 / outlier整理)
priority: 高 (R2-C ファクター戦略実装の前提)
status: 完了 (3,708 row 分析、R2-C 進行 PASS、HQ 必須条件 全クリア)
owner: Fujiwara
depends_on: [F286-R2-A3 full eligible indicators]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R2_A3_full_eligible_indicators, F286_R2_A2_eligible_universe_and_indicators_storage
---

★ **F286 R2-B**: research_derived_indicators (3,708 row) について
   分布・sector別・outlier・前処理推奨案・R2-C 進行可否を **DB
   read-only で 1 run** で出す分析。R2-C ファクター戦略実装に進める
   と判定 (9 条件 全 OK)。

# F286-R2-B Research Lane 7 指標 分布検証 / sector別 / outlier 整理

## サマリ

R2-A3 で永続化した 3,708 row について、純粋関数 + read-only runner
構成で 4 観点を 1 回で集計:

1. 全体分布 (n / coverage / mean / median / p5/25/75/95 / min / max)
2. sector_17 別中央値 (件数 + 7 指標 median)
3. outlier 件数 + 代表銘柄 (HQ 指定 9 閾値)
4. R2-C 用前処理推奨案 + Go/No-Go 判定

判定: **R2-C 進行 PASS** (9 条件 全 OK)。

## 実行結果 — 全体分布

| indicator | n | coverage | median | p25 | p75 | p5 | p95 | min | max |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| **PER** | 3,225 | 87.0% | 14.54 | 10.17 | 22.49 | 5.53 | 67.58 | 0.81 | 1,514.86 |
| **PBR** | 3,644 | 98.3% | 1.27 | 0.78 | 2.27 | 0.39 | 6.20 | 0.08 | 302.78 |
| **ROE** | 3,699 | 99.8% | 0.076 | 0.038 | 0.124 | -0.21 | 0.24 | **-138.33** | 1.06 |
| operating_margin | 3,623 | 97.7% | 0.061 | 0.028 | 0.107 | -0.061 | 0.24 | **-1160** | 0.74 |
| net_margin | 3,694 | 99.6% | 0.046 | 0.019 | 0.086 | -0.10 | 0.19 | **-1151** | 1.59 |
| sales_growth_yoy | 3,636 | 98.1% | 0.047 | 0.000 | 0.115 | -0.11 | 0.31 | -1.00 | 7.39 |
| profit_growth_yoy | 3,645 | 98.3% | 0.091 | -0.159 | 0.427 | -1.38 | 2.21 | -283.67 | 1,391.75 |

**観察**:
- 中央値はいずれも **東証スタンダード以上 事業会社** として妥当
  (PER 14.5 / PBR 1.27 / ROE 7.6%)
- min / max に極端値が混入。**op_margin / net_margin で min=-1,160
  / -1,151** はゼロに近い負の小売上での大赤字 (= 売上 ~10 で赤字 1
  万円超のような小数演算結果)。R2-C で winsorize / cap が必須。
- profit_growth_yoy は abs(prior) 正規化で +1,391 のような上方
  outlier が出る (赤字回復 / IPO 翌期等の構造的特性)

## 実行結果 — sector_17 別 中央値

17 セクター全件 + null sector の合計を一覧:

| sector | n | PER | PBR | ROE | OPM | NM | sYoY | pYoY |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| 食品 | 134 | 18.04 | 1.23 | 6.8% | 4.2% | 3.3% | 3.8% | 3.0% |
| 情報通信・サービス他 | 1,228 | 15.24 | 1.83 | 10.1% | 7.7% | 5.4% | 7.5% | 10.3% |
| 電気・ガス | 28 | 10.23 | 0.68 | 7.4% | 6.9% | 4.9% | -0.5% | 13.6% |
| 運輸・物流 | 104 | 12.46 | 0.95 | 7.5% | 6.6% | 5.1% | 4.6% | 10.3% |
| 商社・卸売 | 284 | 11.86 | 0.96 | 7.6% | 3.2% | 2.5% | 4.5% | 8.0% |
| 小売 | 326 | 16.32 | 1.53 | 7.9% | 3.3% | 2.3% | 4.0% | 2.0% |
| **銀行** | 79 | 15.25 | **0.78** | 4.9% | 4.9% | **13.6%** | 8.3% | 28.1% |
| 金融（除く銀行） | 89 | 12.13 | 1.24 | 8.8% | **13.5%** | **12.2%** | 9.2% | 19.2% |
| 不動産 | 131 | 10.47 | 1.25 | **11.4%** | 9.4% | 6.0% | 9.4% | 12.5% |
| エネルギー資源 | 14 | 10.47 | 0.99 | 7.8% | 6.4% | 5.5% | 2.0% | 0.1% |
| 建設・資材 | 275 | 13.57 | 0.94 | 7.1% | 6.2% | 4.6% | 2.9% | 9.8% |
| 素材・化学 | 279 | 14.17 | 0.86 | 5.8% | 6.3% | 4.5% | 2.8% | 4.1% |
| 医薬品 | 79 | 18.11 | 2.16 | **1.5%** | 6.3% | 3.8% | 3.0% | 0.6% |
| 自動車・輸送機 | 99 | 11.55 | 0.73 | 5.4% | 5.0% | 4.0% | 1.3% | 0.0% |
| 鉄鋼・非鉄 | 71 | 13.64 | 0.75 | 5.7% | 5.3% | 3.6% | 0.0% | 13.9% |
| 機械 | 209 | 14.16 | 1.05 | 7.2% | 8.8% | 6.5% | 3.3% | 6.1% |
| 電機・精密 | 279 | 18.60 | 1.37 | 6.8% | 7.1% | 5.7% | 4.0% | 11.6% |

**Sector 別観察**:
- **不動産** が ROE 中央値 11.4% で最高 (= REIT 含まないので普通の
  不動産事業会社ベース)
- **医薬品** ROE 1.5% は驚くほど低 → 大型新薬開発失敗 / 業界全体の
  成熟期。**R2-C Quality 戦略で sector 別中央値で評価する必要性**
- **銀行** は op_margin 4.9% (普通株比較不可)、net_margin 13.6% (高)
  → R2-C で銀行は ROE / net_margin 中心で評価
- **PBR 最低**は 自動車・輸送機 0.73、銀行 0.78 (循環株 / 構造的
  低 PBR)
- 情報通信は最大セクター (n=1,228、全体の 33%)、PBR 1.83 / ROE 10.1%
  と成長性 (sales_yoy 7.5%、profit_yoy 10.3%) が高い

## 実行結果 — outlier (HQ 指定 9 閾値)

| indicator | threshold | direction | count | 代表 (上位 3) |
|---|---|---|---:|---|
| **PER** | high_100 | above | **100** | 69970=1,515 / 98540=1,465 / 70830=1,447 |
| PER | high_300 | above | 20 | (上記同) |
| PBR | high_20 | above | 22 | 175A0=302 / 485A0=61 / 69930=48 |
| ROE | high_0_5 (>50%) | above | 16 | 63660=1.06 / 330A0=0.79 / 88760=0.61 |
| **ROE** | low_neg_0_5 (<-50%) | below | **91** | 175A0=-138 / 44360=-17 / 36280=-13 |
| operating_margin | high_0_6 (>60%) | above | 6 | 24770=0.74 / 33500=0.71 / 71640=0.70 |
| net_margin | high_0_5 (>50%) | above | 6 | 67360=1.59 / 48890=0.86 / 87060=0.58 |
| sales_growth_yoy | high_1_0 (>100%) | above | 29 | 33500=7.39 / 48940=6.61 / 88940=5.21 |
| **profit_growth_yoy** | high_3_0 (>300%) | above | **124** | 47840=1,392 / 24040=82.7 / 62640=70.3 |

**outlier 観察**:
- **PER > 100 が 100 件 (3.1%)**: 利益が極小の銘柄 (グロース or
  業績悪化中)。R2-C で winsorize 必須
- **ROE < -50% が 91 件 (2.5%)**: 債務超過に近い赤字銘柄 (例:
  175A0=-138 は IPO 直後 IT)。R2-C Quality 戦略では除外 / "loser"
  ラベルの根拠
- **profit_yoy > 300% が 124 件 (3.34%)**: abs(prior) 正規化由来の
  赤字回復 / IPO 翌期。**R2-C で max cap 3.0 を推奨**
- 175A0 が PBR / ROE で双方 outlier ← 同銘柄の data 整合性確認候補
  (= IPO 直後 + 大型減損等の組合せ)

## R2-C 用前処理推奨案 (要旨)

| indicator | cap / winsorize | 評価モード |
|---|---|---|
| **PER** | winsorize p95 (= 67.6) / 絶対 cap 100 | sector_relative 推奨 |
| **PBR** | winsorize p95 (= 6.2) / 絶対 cap 20 | sector_relative 推奨 |
| **ROE** | winsorize p5/p95 (= [-0.21, 0.24]) | absolute 評価可 |
| operating_margin | winsorize p95 / cap 0.6 | sector_relative |
| net_margin | winsorize p5/p95 | hybrid (sector + 絶対) |
| sales_growth_yoy | winsorize p95 (= 0.31) / cap 1.0 | absolute 評価可 |
| profit_growth_yoy | winsorize p5/p95 / cap 3.0 | hybrid |

**特殊扱い (HQ 説明用)**:
- **negative_eps**: PER skip (R2-A1 既実装、`skipped_reason="negative_eps"`)
  → R2-C で「赤字フラグ」として保存
- **negative_equity**: ROE skip (= 債務超過、Quality factor 除外)
- **銀行 / 保険 sector_17="15"** の operating_margin missing は data
  特性、**operating_margin 評価対象から外す** (= ROE / net_margin
  で評価)
- **no_fy_record (= eligible でも FY 開示なし、~1.2%)** は research_derived_indicators 不在
  → R2-C scoring から自動除外
- **同 code に複数 base_date / disclosure_date** の重複は別 row
  扱い、最新のみで scoring を回す orchestrator が必要 (R2-C 実装側)

## R2-C 進行可否判定

判定結果: **PASS (9/9 条件)**

| 条件 | 観測値 | 判定 |
|---|---|---|
| PER coverage >= 80% | 87.0% | OK |
| PBR coverage >= 80% | 98.3% | OK |
| ROE coverage >= 80% | 99.8% | OK |
| operating_margin coverage >= 80% | 97.7% | OK |
| net_margin coverage >= 80% | 99.6% | OK |
| sales_growth_yoy coverage >= 80% | 98.1% | OK |
| profit_growth_yoy coverage >= 80% | 98.3% | OK |
| sector_count >= 10 | 17 | OK |
| profit_growth_yoy > 3.0 が n の 5% 未満 | 3.34% (124/3,708) | OK |

→ **R2-C ファクター戦略実装に進める**。本 report の cap / winsorize
   / sector 推奨案を実装に取り込む。

## constraints_check

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

production / develop DB last_modified May 7 完全無触。staging DB も
read-only 接続のみ (R2-A3 から write なし)。

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/indicator_analysis | 21 PASS |
| scripts/jobs/analyze_indicator_distributions | 18 PASS |
| **R2-B 合計** | **39 PASS** |
| 関連 regression (research_lane / scripts) | **481 PASS** |

## commit ログ

| # | commit | hash |
|---|---|---|
| R2-B-1 | chore(F286-R2): analyze derived indicator distributions | 3114314 |
| R2-B-2 | docs(F286-R2): vault derived indicator distribution result | (本 commit) |
| R2-B-3 | docs(F286-R2): log milestone — indicator distribution analysis completed | (次 commit) |

## 次のアクション (HQ 判断)

R2-B PASS → **R2-C ファクター戦略実装** に進行可能。

R2-C 候補 (F285 R2 仕様):
1. **Quality Value Screener**: 高 ROE × 低 PER × 低 PBR (sector
   relative)、negative_eps / negative_equity 除外
2. **Earnings Growth Screener**: 高 sales_growth_yoy × 高 profit_growth_yoy、
   profit_yoy cap 3.0 適用、sector 別 standard score
3. **Cyclical Value Screener**: 不動産 / 自動車 / 鉄鋼の低 PBR × 高
   ROE 銘柄、sector_relative 必須
4. **Dividend Quality Screener** (R2-D 候補): payload_json 内
   DivAnn / FDivAnn を活用、F285 R2 仕様に従い後続検討

R2-C 実装で本 R2-B report の cap / winsorize / 評価モード推奨案を
取り込み、追加判定 (Quality / Growth / Value gates) を加える設計。

## 関連リンク

- [[F286_R2_A3_full_eligible_indicators|F286-R2-A3 全 eligible 永続化]]
- [[F286_R2_A2_eligible_universe_and_indicators_storage|F286-R2-A2 eligible filter + storage]]
- [[F286_R2_A1_derived_indicators_smoke|F286-R2-A1 派生指標 MVP]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ exit 0、JSON / コンソール report 出力、records 3,708
  全件読込
- **機能した**: ✅ 7 指標分布 / 17 sector 中央値 / 9 outlier 条件 /
  前処理推奨案 / Go/No-Go 判定 が全て生成、R2-C で参照可能な情報を
  網羅
- **期待値達成**: ✅ HQ 必須条件 (read-only / production / develop DB
  無触 / market_financials_v2 + research_derived_indicators 明示参照)
  全クリア、9 条件すべて OK で R2-C 進行可、tests 39 PASS / regression
  481 PASS、Codex pre-commit 通過
