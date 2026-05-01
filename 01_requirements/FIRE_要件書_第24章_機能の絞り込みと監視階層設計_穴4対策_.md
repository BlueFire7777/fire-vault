---
type: requirement_chapter
chapter: "24"
title: "機能の絞り込みと監視階層設計(穴4対策)"
version: v3.1
updated: 2026-04-21
---

# 第24章: 機能の絞り込みと監視階層設計(穴4対策)

## 穴4の本質

FIREは最初から多機能でよい。ただし、**最初の本番運用で主役を絞らないこと**が問題。最初から全部を同じ熱量で回そうとすると、監視対象が増える・判断軸が増える・ルールが複雑になる・デバッグが難しくなる・どこが勝ち筋かわからなくなる・ダッシュボードも散らかる。

**結論**: システムとしては将来の拡張を持つ。**ただし本番で最初に回す戦略は絞る**。

## FIRE v1 の本番主軸

| 区分 | 内容 |
|---|---|
| 本番主軸 | デイトレ自動化のみ。主戦略2本(高品質材料の初動+主役テーマ連動の値嵩株前場初動) |
| 後回し | スイングの自動執行(候補提案まではよい)/ 長期の自動売買(通知と候補提案のみ)/ 準スキャル本番化 / パターンの細かい枝分かれ |

## 監視階層(3段階)

| 階層 | 対象 | 件数 | 用途 |
|---|---|---|---|
| Tier 1: 実行監視 | 実行候補 | 1〜3銘柄 | 最優先。自動執行対象 |
| Tier 2: 研究監視 | 朝の高確率機会 | 5件程度 | Paper検証対象。実行しないが追跡 |
| Tier 3: 参考監視 | スイング・長期・分散候補 | 制限なし | 通知・提案のみ |

## エージェントの優先順位

| 優先度 | エージェント |
|---|---|
| 本番コア | Market Intelligence / Daytrade Selection / Trade Decision / Monitoring & Alert / Execution Engine / Evaluation |
| 準コア | Pattern Research / Dashboard |
| 補助 | Swing Selection / Long-term Selection / Portfolio Balance |

## パターン数の上限

最初から多くのパターンを持つと管理が崩れる。初期の上限目安:

| 区分 | 上限 |
|---|---|
| Primary Active Patterns(本番実行対象) | 2〜4 |
| Candidate Patterns(検証中) | 5〜10 |
| DEATH NOTE / REHAB | 制限なし |

## 毎日やること vs 週次・バッチ処理でよいこと

| 頻度 | 内容 |
|---|---|
| 毎日やる | 朝レポート / デイトレ候補抽出(実行候補1〜3銘柄)/ 自動監視 / 仮想または実執行 / 引け後評価 / Dashboard更新 |
| 週次・夜間・バッチ処理でよい | スイングの濃い分析 / 長期の深掘り / 新規パターン大量発見 |

## Dashboardで表示すべき主役の可視化

Dashboard上に常に「**いまFIREは何を戦っているのか**」を表示する。

- Current Primary Edge(現在の主戦略)
- Current Secondary Edge(現在の準主戦略)
- Disabled Edge(無効化中の戦略)
- Active Strategy / Active Patterns / Today's Execution Candidates
- Research Candidates / Disabled Patterns / Candidate Overload Warning

## 本章の要件ID

- **R-24-01** FIRE v1本番主軸: デイトレ自動化のみ
- **R-24-02** スイング自動執行は後回し
- **R-24-03** 長期自動売買は後回し
- **R-24-04** 監視階層3段階
- **R-24-05** エージェント優先順位
- **R-24-06** パターン数上限
- **R-24-07** 毎日vs週次バッチ区分
- **R-24-08** Dashboard主役可視化


---

## ナビゲーション

[[FIRE_要件書_第23章_FIRE_Runtime___常駐基盤と稼働スケジュール|← 第23章: FIRE Runtime / 常駐基盤と稼働スケジュール]]　|　[[FIRE_要件書_第25章_戦略階層と頻度設計|第25章: 戦略階層と頻度設計 →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
- タスク: [[../02_todo/_index|TODO索引]]
- 運用ルール: [[../タスク運用ルール|タスク運用ルール]]
