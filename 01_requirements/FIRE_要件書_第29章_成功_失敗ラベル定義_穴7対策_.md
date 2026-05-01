---
type: requirement_chapter
chapter: "29"
title: "成功/失敗ラベル定義(穴7対策)"
version: v3.1
updated: 2026-04-21
---

# 第29章: 成功/失敗ラベル定義(穴7対策)

## 穴7の本質

FIREは再現性で勝つ思想だが、**「何を成功とするか・何を失敗とするか」が曖昧だとPattern Library全体が弱くなる**。ラベルは1個では足りず、**主ラベル+補助ラベル群**にするのが正しい。

## 主ラベルの定義(4分類)

| ラベル | 定義 |
|---|---|
| Success | エントリー後、パターンごとのtarget_holding_window内に、損切より先に利確1へ到達した場合 |
| Failure | 損切に先に到達した場合(利確1未到達) |
| Timeout | 利確1にも損切にも届かず、有効期限切れ・前場終了・強制クローズ時間到来 |
| Invalid | エントリー条件成立前に無効条件に該当、またはエントリー未成立 |

**重要**: 主ラベルは「**先にどちらに届いたか**」で判定する。「前場引けでプラスなら成功」にしない。

## なぜ「前場引けでプラス」を成功にしないのか

- 利確設計の良し悪しが見えない
- 損切設計の良し悪しが見えない
- 途中で利確できたかどうかが曖昧
- Paper と Real の比較がぶれる

## target_holding_window(パターンごとに可変)

保有時間は固定ではなく、パターンごとに持つ。

| パターン種別 | target_holding_window |
|---|---|
| 材料初動型 | 10〜30分 |
| 主役テーマ再加速型 | 15〜45分 |
| 準スキャル型 | 3〜10分 |

## ラベル判定の具体式

前提として以下を持つ: entry_price / stop_price / take_profit_1 / target_holding_window / invalid_condition / expiry_time

| ラベル | 判定条件 |
|---|---|
| Success | first_hit == take_profit_1 AND within_target_holding_window == true |
| Failure | first_hit == stop_price |
| Timeout | first_hit == none AND expiry_or_window_end == true |
| Invalid | entry_not_filled OR invalid_condition_hit_before_entry == true |

## 補助メトリクス(必ず保存する)

| 指標 | 内容 |
|---|---|
| mfe_pct | 最大含み益率 |
| mae_pct | 最大逆行率 |
| time_to_mfe | 最大含み益到達時間 |
| time_to_mae | 最大逆行到達時間 |
| time_to_tp | 利確1到達時間 |
| time_to_sl | 損切到達時間 |
| pnl_at_timeout | Timeout時点の損益 |
| pnl_at_close | 引け時点の損益 |
| fill_quality_score | 約定品質スコア |

補助メトリクスにより「利確早すぎ・損切広すぎ・保有時間長すぎ」が後から分かる。

## Pattern Library採用に使う最重要指標

| 指標 | 内容 |
|---|---|
| success_rate | 成功率 |
| failure_rate | 失敗率 |
| timeout_rate | タイムアウト率 |
| expectancy | 期待値 |
| avg_rr | 平均RR |
| avg_mae | 平均最大逆行 |
| avg_mfe | 平均最大含み益 |
| regime_specific_success_rate | レジーム別成功率 |
| fill_quality_adjusted_expectancy | 約定品質調整済み期待値 |

**重要**: 勝率だけで採用しない。

## 注意点(3つ)

1. **TimeoutをFailureに全部入れない**: 「勝てるけど遅いパターン」と「そもそもダメなパターン」が混ざる
2. **利確2を主ラベルにしない**: 最初は利確1到達を主ラベル。利確2は補助指標で十分
3. **エントリー未成立と失敗を分ける**: Invalid / Unfilledは別。混ぜるとPatternの評価が崩れる

## 本章の要件ID

- **R-29-01** 主ラベル4分類 Success/Failure/Timeout/Invalid
- **R-29-02** Success定義
- **R-29-03** Failure定義
- **R-29-04** Timeout定義
- **R-29-05** Invalid定義
- **R-29-06** 前場引けプラス成功禁止理由
- **R-29-07** target_holding_window パターン別
- **R-29-08** ラベル判定式
- **R-29-09** 補助メトリクス必ず保存
- **R-29-10** Pattern Library採用最重要指標9個
- **R-29-11** 注意点3つ


---

## ナビゲーション

[[FIRE_要件書_第28章_監督者付き自律運用システムの設計(穴6対策)|← 第28章: 監督者付き自律運用システムの設計(穴6対策)]]　|　[[FIRE_要件書_第30章_特徴量の初期絞り込み(穴8対策)|第30章: 特徴量の初期絞り込み(穴8対策) →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
- タスク: [[../02_todo/_index|TODO索引]]
- 運用ルール: [[../タスク運用ルール|タスク運用ルール]]
