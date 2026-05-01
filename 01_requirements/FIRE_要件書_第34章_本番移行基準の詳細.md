---
type: requirement_chapter
chapter: "34"
title: "本番移行基準の詳細"
version: v3.1
updated: 2026-04-21
---

# 第34章: 本番移行基準の詳細

## 本番移行の基本思想

**「勝っている」だけでは昇格させない**。FIREの本番移行は以下の4つを同時に満たすことが必要。

| 要件 | 内容 |
|---|---|
| 戦略成績 | 期待値がプラスであること |
| 執行品質 | 約定品質・スリッページが許容範囲内 |
| 停止機能の正常性 | 全停止機能がテスト・実運用で正常動作 |
| シミュレーションの現実性 | Optimistic Biasが許容範囲内 |

**原則**: FIREは、収益が出ていることだけを理由に本番移行してはならない。期待値、執行品質、停止品質、シミュレーション現実性、監督可能性が揃った場合にのみ、段階的に昇格する。

## 昇格判断に使うKPI(全系統)

| 系統 | KPI |
|---|---|
| 収益系 | expectancy / avg_rr / success_rate / failure_rate / timeout_rate |
| リスク系 | max_drawdown / avg_mae / stop_hit_rate / daily_stop_trigger_count |
| 執行系 | fill_rate / partial_fill_rate / avg_slippage / forced_close_success_rate |
| システム系 | broker_uptime / market_data_uptime / line_delivery_success_rate / monitoring_delay |
| 現実性系 | optimistic_bias_score / realistic_fill_score / simulation_confidence_score |

## 昇格を止める条件(昇格禁止条件)

以下のいずれかに該当する場合は、**たとえ収益が出ていても昇格させない**。

- 期待値がマイナス
- ドローダウン過大
- Optimistic Biasが高すぎる
- 強制クローズ失敗あり
- 緊急停止が正常に働かない
- 主戦略件数が少なすぎる
- 執行品質が悪すぎる
- 監督負荷が高すぎる

## 推奨する初期しきい値(暫定値)

### Paper Live → Semi Auto

| 条件 | 基準 |
|---|---|
| サンプル数 | 20営業日 or 50トレード(※過去データの高速リプレイ検証とリアルタイムPaper Liveの並列実行でカレンダー日数を待たずクリア可) |
| 期待値 | expectancy > 0 |
| 最大DD | -5% を大きく超えない |
| Optimistic Bias | 許容範囲内 |
| 強制クローズ成功率 | 100% |
| 停止機能 | 全テストパス |

### Semi Auto → Full Auto

| 条件 | 基準 |
|---|---|
| サンプル数 | 20営業日 or 50実注文 |
| 期待値 | 実約定ベースで expectancy > 0 |
| スリッページ | avg_slippage が許容範囲内 |
| 強制クローズ成功率 | 100% |
| システム稼働率 | broker_uptime / system_uptime が十分 |
| Evaluation Agent | 昇格提案あり |
| 最終承認 | ユーザーが承認 |

## Evaluation Agentの昇格提案書

本番移行時にEvaluation Agentは昇格提案書を作成する。内容:

| 項目 | 内容 |
|---|---|
| 成績サマリー | 期待値・勝率・RR・サンプル数 |
| エッジ別成績 | 主戦略・準主戦略ごとの成績 |
| 執行品質 | スリッページ・約定率・部分約定率 |
| システム品質 | 稼働率・遅延・障害履歴 |
| 停止機能の結果 | 全停止機能のテスト結果 |
| Optimistic Bias | シミュレーション現実性スコア |
| 懸念点 | 残存リスク・改善推奨事項 |
| 判定 | 昇格推奨 / 保留 / 否決 |

**重要**: 最終決定は必ず**ユーザーが行う**。Evaluation Agentは提案のみ。

## 本章の要件ID

- **R-34-01** 昇格4同時要件
- **R-34-02** 昇格KPI 5系統
- **R-34-03** 昇格禁止条件8点
- **R-34-04** 推奨初期しきい値 Paper→Semi
- **R-34-05** 推奨初期しきい値 Semi→Full
- **R-34-06** Evaluation昇格提案書


---

## ナビゲーション

[[FIRE_要件書_第33章_人間の運用負荷の上限設計(穴11対策)|← 第33章: 人間の運用負荷の上限設計(穴11対策)]]　|　[[FIRE_要件書_第35章_Mac_mini_常駐基盤の時間帯別分担|第35章: Mac mini 常駐基盤の時間帯別分担 →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
- タスク: [[../02_todo/_index|TODO索引]]
- 運用ルール: [[../タスク運用ルール|タスク運用ルール]]
