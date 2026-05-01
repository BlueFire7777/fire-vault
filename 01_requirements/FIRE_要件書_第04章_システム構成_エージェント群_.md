---
type: requirement_chapter
chapter: "04"
title: "システム構成(3層ハイブリッド・アーキテクチャ)"
version: v3.3
updated: 2026-04-24
---

# 第04章: システム構成(3層ハイブリッド・アーキテクチャ)

## v3.3 での大きな変更

v3.1 では「Mac mini上の常駐ランタイムを前提とし、独立したエージェントで構成」とのみ定義していたが、v3.2 で OpenClaw 採用と 3層構造を導入、v3.3 で発注の自動化を廃止し半自動主軸モデルに再編する。

## 3層ハイブリッド・アーキテクチャ

### 第1層: オーケストレーション層(OpenClaw)

- **役割**: Mac mini に常駐し、各エージェントのスケジュール起動、状態保持、セッション永続化、並列実行制御、障害自動復旧、ツールサンドボックスを提供する
- **採用理由**:
  - 24時間稼働・ハートビート・自動復旧を自作せずに済む(開発工数を数ヶ月削減)
  - エージェント間の状態共有、並列実行(Paper Live + Pattern Research 同時稼働)を標準機能で実現
  - セッション transcripts によるデバッグ性が高い
- **v3.2 からの変更**: Computer Use との統合は不要(発注を人間が行うため)
- **実装**: OpenClaw on Mac mini M4 Pro

### 第2層: 推論層(Claude Code + 10 エージェント)

既存の独立エージェント群。OpenClaw 上で動作する。

| エージェント名 | 主な役割 |
|---|---|
| Market Intelligence Agent | 朝の市況・ニュース・セクター分析、朝レポート生成 |
| Daytrade Selection Agent | デイトレ候補の抽出・厳選(実行候補1〜3銘柄)、スコア付与 |
| Swing Selection Agent | スイング候補抽出、中期トレンド分析 |
| Long-term Selection Agent | 高配当・成長株抽出、長期スコア付与 |
| Portfolio Balance Agent | セクター偏り点検、バランス改善提案 |
| Trade Decision Agent | エントリー・利確・損切・株数提案、**注文指示の生成(LINE通知へ)** |
| Monitoring & Alert Agent | 価格到達監視、LINE通知(**緊急ポジション整理含む**) |
| Review & Journal Agent | 引け後レビュー、成績分析、学習データ保存 |
| Goal Tracking Agent | 3,000万円目標の進捗・乖離管理 |
| Evaluation Agent | 評価提案書の作成(自動反映禁止、ユーザー承認制) |

### 第3層: 実行層(人間中心・半自動)

v3.2 までの「Computer Use による自動発注 + 独立 Cron による強制クローズ」を **全廃**し、以下に置き換える。

| 機能 | 担当 | 方式 |
|---|---|---|
| 発注 | **Fujiwara さん** | FIRE の LINE 通知を見て、iSPEED / 楽天Webで手動発注 |
| 強制クローズ | **Fujiwara さん** | FIRE からの LINE 緊急アラート(14:45/14:55/15:05/15:10/15:15)を見て手動クローズ |
| 約定確認 | **Fujiwara さん → FIRE** | 初期: LINE で約定報告 / M+2 以降: 楽天証券の約定通知メール → Gmail API で FIRE 自動取込 |
| 建玉管理 | **Fujiwara さん + FIRE** | 正本は楽天証券。FIRE は約定データ取込み時に整合チェック |

### 変更理由(v3.3 の設計思想)

1. FIREのエッジは再現性優位であり、自動化優位ではない(第26章参照)
2. Computer Use による発注は、楽天証券の 2FA / UI変更 / 規約グレーの3重苦でリスクが高い
3. 半自動化により、開発リソースをエッジ開発(Pattern Store / Feature Store / 類似検索)に集中できる
4. 兼業投資家にとって、寄り前の指値予約 + 場中 LINE通知 + 強制クローズアラートの運用で実務的に十分
5. 将来エッジが確立した後に、全自動化を段階的に追加することは可能(後回し = オプション)

## アーキテクチャ図(概念)

```
[情報源] ニュース / TDnet / 市場データ / 楽天証券約定メール
    ↓
[第1層: オーケストレーション層 (OpenClaw on Mac mini)]
    - 常駐・スケジュール・並列実行・自動復旧・セッション永続化
    ↓
[第2層: 推論層 (Claude Code + 10 エージェント)]
    - Pattern Store / Feature Store / エッジ判定 / 候補抽出 / 評価
    ↓
[LINE通知 / Dashboard]
    ↓
[第3層: 実行層(人間)]
    - Fujiwara さんが手動発注・手動クローズ
    ↓
[約定結果]
    - M=0〜M+1: LINE で Fujiwara さん手動報告
    - M+2以降: Gmail API経由で自動取込
    ↓
[FIRE 学習ループ] → 第2層へフィードバック
```

## 本章の要件ID(v3.3)

- **R-04-01** 3層ハイブリッド・アーキテクチャの明記
- **R-04-02** 第1層: OpenClaw on Mac mini
- **R-04-03** 第2層: Claude Code + 10エージェント
- **R-04-04** 第3層: 人間中心・半自動
- **R-04-05** Market Intelligence Agent
- **R-04-06** Daytrade Selection Agent(1-3銘柄厳選)
- **R-04-07** Swing Selection Agent
- **R-04-08** Long-term Selection Agent
- **R-04-09** Portfolio Balance Agent
- **R-04-10** Trade Decision Agent(注文指示生成)
- **R-04-11** Monitoring & Alert Agent(緊急ポジション整理含む)
- **R-04-12** Review & Journal Agent
- **R-04-13** Goal Tracking Agent
- **R-04-14** Evaluation Agent(自動反映禁止・承認制)
- **R-04-15** 発注は人間(Fujiwara さん)が手動実行
- **R-04-16** 強制クローズは LINE 緊急アラート → 人間対応
- **R-04-17** 約定取込は初期LINE手動、M+2以降Gmail API
- **R-04-18** 建玉正本は楽天証券、FIRE は整合チェックのみ
- **R-04-19** Computer Use は採用しない(v3.3での明示削除)
- **R-04-20** 独立Cron + Playwright による強制クローズ自動化は採用しない

## v3.3 削除要件

- v3.2 Computer Use による楽天証券ブラウザ直接操作 → 削除
- v3.2 セルフレビュー(注文画面スクショ検証) → 削除
- v3.2 独立 Cron + Playwright による強制クローズ → 削除(LINE 緊急アラートに置換)

## ナビゲーション

[[FIRE_要件書_第03章_FIREの主戦略_第一エッジ_|← 第03章]] | [[FIRE_要件書_第05章_許容損失とリスク管理_準攻撃型_|第05章 →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
