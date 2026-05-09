---
id: F286-R2-A1
phase: P5: Research Lane R2 (派生指標 7 種 MVP)
priority: 高 (R2 派生指標群の根幹、後続 R2 ファクター計算の前提)
status: 完了 (5 銘柄 / 100 銘柄 mini smoke、HQ 必須条件 全クリア)
owner: Fujiwara
depends_on: [F286-R1-B4 market_financials_v2 full backfill]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R1_B4_market_financials_full_backfill, F286_R1_B2_5_market_financials_v2_smoke
---

★ **F286 R2-A1**: market_financials_v2 + market_prices_daily 結合
   による派生指標 7 種 (PER / PBR / ROE / op_margin / net_margin /
   sales_yoy / profit_yoy) の純粋関数 + smoke runner。**DB write
   なし**、staging read-only のみ。HQ 5 銘柄 / 100 銘柄 mini
   smoke 完遂。

# F286-R2-A1 Research Lane derived indicators MVP

## サマリ

R1-B4 で投入された v2 financials (164,678 row) と既存
market_prices_daily を結合し、Research Lane で使う 7 つの派生指標を
計算する純粋関数群 + smoke runner を実装。HQ 必須条件 (read-only /
v2 明示参照 / 欠損・赤字・ゼロ割の明示 skip) 全クリア。

## 7 指標と skip 規約

| # | indicator | formula | skip 条件 |
|---|---|---|---|
| 1 | PER | price / EPS | missing / zero_denominator / negative_eps / non_positive_price |
| 2 | PBR | price / BPS | missing / zero_denominator / negative_bps / non_positive_price |
| 3 | ROE | profit / equity | missing / zero_denominator / negative_equity (= 債務超過、赤字 profit は許容) |
| 4 | operating_margin | operating_profit / net_sales | missing / zero_denominator / negative_sales (= 防御的) |
| 5 | net_margin | profit / net_sales | 同上 |
| 6 | sales_growth_yoy | (cur - prior) / abs(prior) | missing / zero_denominator |
| 7 | profit_growth_yoy | 同上 (profit) | 同上 |

赤字・マイナス成長は **計算成功** で負値を返す (= ROE / margin /
YoY が負値も投資判断に意味)。**ゼロ割** / **母数 0** / **意味なし
パターン** (negative_eps / negative_bps / negative_equity) のみ skip。

## 実行結果 — 5 銘柄 smoke

| code | PER | PBR | ROE | op_margin | net_margin | sales_yoy | profit_yoy |
|---|---:|---:|---:|---:|---:|---:|---:|
| 72030 (Toyota) | 10.16 | 0.98 | 9.4% | 7.4% | 7.6% | +5.5% | -19.2% |
| 67580 (Sony) | skip[neg_eps] | 2.28 | -3.8% | 11.6% | -2.6% | -3.7% | -128.6% |
| 80350 (Tokyo Electron) | 37.82 | 10.55 | 27.8% | 25.6% | 23.5% | +0.5% | +5.6% |
| 83060 (MUFG) | 17.49 | 1.57 | 8.6% | skip[missing] | 13.7% | +14.6% | +25.0% |
| 16050 (INPEX) | 12.45 | 1.01 | 7.8% | 56.5% | 19.6% | -11.2% | -7.8% |

**観察**:
- Toyota / INPEX の PBR ≈ 1 (= 株価 ≈ 簿価) は妥当
- 80350 (TEL) の高 PER (37.8) / 高 ROE (27.8%) は半導体製造装置の
  高成長を反映、PBR 10.55 も妥当
- Sony は当期赤字 EPS (-54.7) で PER は意味なし → skip[negative_eps]
- MUFG (銀行) は /fins/summary に operating_profit を返さない (銀行
  業の特殊会計) → op_margin skip[missing]、他指標は計算可能

skip 集計:

| indicator | computed | skip_rate | skip 理由 |
|---|---:|---:|---|
| PER | 4/5 | 20.0% | negative_eps × 1 |
| PBR | 5/5 | 0% | — |
| ROE | 5/5 | 0% | — |
| operating_margin | 4/5 | 20.0% | missing × 1 (銀行) |
| net_margin | 5/5 | 0% | — |
| sales_growth_yoy | 5/5 | 0% | — |
| profit_growth_yoy | 5/5 | 0% | — |

## 実行結果 — 100 銘柄 mini smoke

100 銘柄 (market_listings code 昇順 LIMIT 100、code 13010 〜 1xxx)
で実行。

skip 集計:

| indicator | computed | skip_rate | 主な理由 |
|---|---:|---:|---|
| PER | 38/100 | 62% | no_fy_record × 58、negative_eps × 3、missing × 1 |
| PBR | 41/100 | 59% | no_fy_record × 58、missing × 1 |
| ROE | 42/100 | 58% | no_fy_record × 58 |
| operating_margin | 41/100 | 59% | no_fy_record × 58、missing × 1 |
| net_margin | 41/100 | 59% | 同上 |
| sales_growth_yoy | 41/100 | 59% | 同上 |
| profit_growth_yoy | 42/100 | 58% | no_fy_record × 58 |

**重要観察 (= 想定外ではないが記録)**:

100 銘柄サンプル (code 昇順 LIMIT 100) は **58 銘柄が no_fy_record**
(= market_financials_v2 に開示記録 0 件)。確認の結果、これらは ETF
/ index ファンド (例: 13050 = iFreeETF TOPIX、13060 = NEXT FUNDS
TOPIX、13260 = SPDR ゴールドシェア etc.) で、**FY 開示を提出しない
類型**であり想定通り。Tier2 universe 4,449 銘柄全体では 株式銘柄
が圧倒的多数なので、本問題は **mini_100 サンプル選択方法のバイアス**。

→ 後続 R2-A2 以降では `market_listings` から ETF / fund を除外する
   filter (例: `sector_17_code != '其他' OR market_code IN (...)`)
   を導入候補。R2-A1 の純粋関数 / runner ロジックには問題なし。

## 数値分布 (100 銘柄 mini, computed 銘柄のみ)

| indicator | n | min | median | mean | max |
|---|---:|---:|---:|---:|---:|
| PER | 38 | 2.83 | 15.08 | 27.46 | 148.67 |
| PBR | 41 | 0.29 | 1.68 | 2.26 | 16.40 |
| ROE | 42 | -0.47 | 0.09 | 0.08 | 0.31 |
| operating_margin | 41 | -0.004 | 0.071 | 0.080 | 0.279 |
| net_margin | 41 | -0.077 | 0.039 | 0.056 | 0.199 |
| sales_growth_yoy | 41 | -0.189 | 0.076 | 0.134 | 0.829 |
| profit_growth_yoy | 42 | -2.31 | 0.111 | 0.410 | 9.62 |

PER 中央値 15、PBR 中央値 1.68、ROE 中央値 9.2% は東証スタンダード
中堅クラスの分布として妥当。profit_growth_yoy 範囲が広い (-230% 〜
+962%) のは小型銘柄の前期赤字 / 当期回復による振れ幅 (abs(prior)
正規化が機能している証左)。

## constraints_check (5 銘柄 / 100 銘柄 共通)

```json
{
  "staging_only_read": true,
  "production_db_untouched": true,
  "develop_db_untouched": true,
  "no_db_write": true,
  "v2_schema_used": true,
  "tier2_full_executed": false
}
```

production / develop DB last_modified May 7 で完全無触。
staging DB も **read-only 接続のみ** (本 runner は出力 JSON のみ、
DB に書き込み一切なし)。

## tests

| 対象 module | tests |
|---|---|
| simulation/research_lane/derived_indicators (純粋関数) | 54 PASS |
| scripts/jobs/compute_derived_indicators (runner) | 29 PASS |
| **R2-A1 trio 合計** | **83 PASS** |
| 関連 regression (research_lane / scripts) | 379 PASS (= 296 R1 + 83 R2) |

## commit ログ

| # | commit | hash |
|---|---|---|
| R2-A1-1 | feat(F286-R2): add research lane derived indicators | 8e670d3 |
| R2-A1-2 | chore(F286-R2): add derived indicators smoke runner | b716756 |
| R2-A1-3 | docs(F286-R2): vault derived indicators smoke result | (本 commit) |
| R2-A1-4 | docs(F286-R2): log milestone — derived indicators smoke completed | (次 commit) |

## 次のアクション

R2-A1 完了 → R2-A2 以降の選択肢 (HQ 判断要):

1. **R2-A2 派生指標保存先 table 設計**:
   `derived_indicators_v1` (or 同等) の schema 確定 →
   smoke result を DB に永続化、後続 R2 ファクター計算の前提に
2. **R2-B 7 指標の分布検証 (Tier2 全件)**:
   4,449 銘柄全件で計算、distribution / outlier / sector 別分布の
   sanity check
3. **R2 ETF / fund フィルタ導入**:
   `market_listings` の market_code / sector で実際の事業会社のみ
   抽出
4. **R2-C ファクター戦略実装**:
   PER / PBR / ROE 等を組み合わせた Cyclical Value / Quality /
   Growth strategies (F285 R2 仕様に従う)

## 関連リンク

- [[F286_R1_B4_market_financials_full_backfill|F286-R1-B4 v2 full backfill]]
- [[F286_R1_B2_5_market_financials_v2_smoke|F286-R1-B2.5 v2 schema]]
- [[F286_R1_Sector_Flow_and_financials_backfill|F286-R1 親タスク]]
- [[F286_Research_Lane_R0_feasibility|F286 R0 feasibility]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ exit 0、JSON 出力成功、5/100 銘柄 smoke 完遂
- **機能した**: ✅ 7 指標すべて期待動作 (赤字 EPS で PER skip、
  銀行 op_margin missing、ETF no_fy_record skip、各指標の値分布が
  実市場と整合)
- **期待値達成**: ✅ HQ 必須条件 (read-only / v2 明示参照 / 欠損
  明示 skip) 全クリア、production / develop DB 完全無触、tests
  83 PASS、Codex pre-commit 通過 × 2
