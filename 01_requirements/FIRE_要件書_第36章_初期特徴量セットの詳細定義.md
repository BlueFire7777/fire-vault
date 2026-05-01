---
type: requirement_chapter
chapter: "36"
title: "初期特徴量セットの詳細定義"
version: v3.1
updated: 2026-04-21
---

# 第36章: 初期特徴量セットの詳細定義

## 基本思想

特徴量は「多ければ多いほど強い」ではない。最初は以下の**5条件**を満たすものだけに絞る。

- 再現性に効く
- 取得しやすい
- 欠損しにくい
- 説明しやすい
- 実行判断に直結する

## A. 材料特徴量(最重要)

FIREの主戦略「高品質材料の初動」に直結する。

| 特徴量 | 内容 |
|---|---|
| material_type | 好決算 / 上方修正 / 自社株買い / 増配 / 受注 / 提携 / 政策恩恵 / テーマ資金流入 |
| material_freshness | 当日 / 前日夜 / 数営業日前 |
| material_strength_score | 材料の強さを段階評価 |
| multi_material_flag | 複数材料が同時に出ているか |
| earnings_related_flag | 決算由来か |
| shareholder_return_flag | 還元強化(増配・自社株買い)か |

## B. 地合い特徴量

同じパターンでも地合いで成功率が変わるため必須。

| 特徴量 | 内容 |
|---|---|
| nikkei_direction | 日経225の方向 |
| topix_direction | TOPIXの方向 |
| sector_strength_rank | セクター強弱ランク |
| theme_strength_rank | テーマ強弱ランク |
| market_regime | 強地合い / 弱地合い / レンジ / 決算相場 / テーマ集中相場 |
| usd_jpy_trend | ドル円トレンド |
| overnight_us_market_bias | 前夜の米国市場バイアス |

## C. 出来高 / 流動性特徴量

勝てるシグナルでも流動性が悪いと再現できないため必須。

| 特徴量 | 内容 |
|---|---|
| volume_ratio | 当日出来高 / 平均出来高 |
| turnover | 売買代金 |
| liquidity_score | 流動性スコア |
| spread_estimate | スプレッド推定 |
| high_price_stock_flag | 値嵩株フラグ |
| board_thickness_score | 板の厚さスコア(初期は粗めでよい) |

## D. 値動き / テクニカル特徴量

パターン照合に必要な「形」を最小限表すために必須。

| 特徴量 | 内容 |
|---|---|
| gap_pct | ギャップ率 |
| intraday_range_pct | 当日値幅率 |
| breakout_flag | ブレイクアウトフラグ |
| pullback_flag | 押し目フラグ |
| vwap_position | VWAPに対する位置 |
| moving_average_alignment | 移動平均の並び |
| relative_strength_score | 相対強度スコア |
| price_action_pattern | 前場高値ブレイク / 初押し / 再加速 / GU失速 等 |
| distance_to_resistance | 直近抵抗線までの距離 |
| distance_to_support | 直近支持線までの距離 |

## E. 執行品質特徴量

FIREはシグナルだけでなく**執行品質込みで再現性を見る**ため必須。

| 特徴量 | 内容 |
|---|---|
| entry_slippage_estimate | エントリースリッページ推定 |
| stop_slippage_estimate | 損切スリッページ推定 |
| partial_fill_risk_score | 部分約定リスクスコア |
| fill_quality_score | 約定品質スコア |
| forced_close_risk_score | 強制クローズリスクスコア |

## F. 時間帯特徴量

同じ材料でも時間帯で優位性が全く違うため必須。

| 特徴量 | 内容 |
|---|---|
| session_phase | 寄り直後 / 前場中盤 / 前場引け前 / 後場寄り / 後場中盤 / 引け前 |
| minutes_from_open | 開場からの経過分数 |
| minutes_to_force_close | 強制クローズまでの残り分数 |
| material_to_entry_lag_minutes | 材料発表からエントリーまでのラグ(分) |

## 初期に入れなくてよい特徴量(後回し)

| 種別 | 理由 |
|---|---|
| 細かすぎる板特徴量 | 取得が重い・欠損しやすい |
| ティック単位の歩み値特徴量 | 初期主戦略には不要 |
| SNS感情分析 | 説明しにくい・欠損しやすい |
| 有名トレーダー動向 | 再現性に直結しない |
| 複雑な自然言語埋め込み | 取得が重い |
| 企業IR資料の全文特徴量 | 初期主戦略には不要 |
| マクロ指標の細かい多変量解析 | 初期主戦略には不要 |
| 高度な画像パターン認識 | 初期主戦略には不要 |

## パターン照合の最小コア特徴量(10個)

類似局面検索の中核として使う最小セット。

material_type / material_strength_score / market_regime / sector_strength_rank / volume_ratio / turnover / gap_pct / price_action_pattern / vwap_position / fill_quality_score

## 類似度計算の重み付け(初期)

| レイヤ | 重み |
|---|---|
| 材料特徴量 | 30% |
| 地合い特徴量 | 20% |
| 出来高 / 流動性 | 20% |
| 値動き / テクニカル | 20% |
| 執行品質 | 10% |
| 時間帯 | 補正要素(重みに加算) |

## 本章の要件ID

- **R-36-01** 特徴量5条件
- **R-36-02** A材料特徴量6項目
- **R-36-03** B地合い特徴量7項目
- **R-36-04** C流動性特徴量6項目
- **R-36-05** D値動きテクニカル10項目
- **R-36-06** E執行品質5項目
- **R-36-07** F時間帯4項目
- **R-36-08** 初期に入れない特徴量
- **R-36-09** パターン照合最小コア10個
- **R-36-10** 類似度重み初期


---

## ナビゲーション

[[FIRE_要件書_第35章_Mac_mini_常駐基盤の時間帯別分担|← 第35章: Mac mini 常駐基盤の時間帯別分担]]　|　[[FIRE_要件書_第37章_Pattern_Storeの4階層構造とActive_Priority_Set|第37章: Pattern Storeの4階層構造とActive Priority Set →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
- タスク: [[../02_todo/_index|TODO索引]]
- 運用ルール: [[../タスク運用ルール|タスク運用ルール]]
