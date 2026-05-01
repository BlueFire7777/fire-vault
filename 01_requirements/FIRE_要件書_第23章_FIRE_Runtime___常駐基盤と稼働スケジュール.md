---
type: requirement_chapter
chapter: "23"
title: "FIRE Runtime / 常駐基盤と稼働スケジュール(v3.3: OpenClaw採用)"
version: v3.3
updated: 2026-04-24
---

# 第23章: FIRE Runtime / 常駐基盤と稼働スケジュール(v3.3: OpenClaw採用)

## v3.3 での重大変更

v3.1 では「OpenClaw そのものは必須ではない。FIRE 専用ランナーの方が管理しやすい」と記載していたが、v3.2 で OpenClaw 採用に方針転換、v3.3 でも継続採用する。専用ランナー自作では24時間稼働・並列実行・自動復旧を全部自作する必要があり、開発期間が数ヶ月伸びる。

## 基本方針(v3.3)

FIRE は「常駐基盤をUIで制御する構造」を採用する。オーケストレーション層には **OpenClaw** を採用し、Mac mini M4 Pro 上で 24 時間常駐させる。

| 役割 | 担当 |
|---|---|
| オーケストレーション層 | **OpenClaw on Mac mini**(常駐、スケジュール、並列実行、自動復旧、セッション永続化) |
| 推論層 | Claude Code + 10 エージェント(OpenClaw 上で動作) |
| 実行層 | **Fujiwara さん(人間)**(LINE 通知を見て iSPEED/楽天Web で手動発注) |
| 制御 UI | Dashboard(ブラウザ) + LINE(通知・緊急アラート・承認コマンド) |

## OpenClaw 採用の理由

### 自作すると必要な機能(数週間〜数ヶ月)
1. launchd + Python の常駐プロセス
2. 各エージェントのプロセス管理(起動・停止・再起動)
3. エージェント間の状態共有(メッセージキュー・DB設計)
4. スケジューラ(8:00朝分析、15:10強制クローズアラート等)
5. 障害時の自動復旧(プロセス死んだら再起動、ログ、通知)
6. Paper Live と Pattern Research の並列実行制御
7. セッション状態の永続化・復旧
8. デバッグ用の挙動追跡ログ

### OpenClaw が標準機能で提供

1. ハートビートループ、スケジュール、バックグラウンドジョブ
2. タスク実行、監視復旧、エージェント起動
3. 構造化オーケストレーション(分岐、反復、並列ブランチ、明示的結合、再起動安全な実行状態)
4. セッション状態(メッセージ、ツール呼び出し/結果)のディスク永続化
5. ツールサンドボックス(Docker ベース)
6. transcripts によるデバッグ可視化

**結論**: 自作で数ヶ月失うより、OpenClaw の学習コスト(1〜2日)を払って、エッジ開発に集中する方が圧倒的に有利。

## FIRE Runtime のコンポーネント構成(OpenClaw 上で動作)

| コンポーネント | 役割 | OpenClaw での実装 |
|---|---|---|
| FIRE Scheduler | 時間帯別ジョブの起動・停止 | OpenClaw schedule API |
| FIRE Monitor | データソース健全性監視・縮退モード判定 | OpenClaw background job |
| FIRE Decision Engine | 候補抽出・パターン照合・発注指示生成 | 複数エージェントの並列実行(OpenClaw parallel branches) |
| FIRE Pattern Research Loop | 過去パターン研究・Candidate Pool 更新 | OpenClaw 夜間バッチジョブ |
| FIRE Notification Worker | LINE 通知送信・ログ保存 | OpenClaw tool call(HTTP API) |
| FIRE Reconciliation Worker(v3.3 新規) | Gmail API から約定メール取込、楽天証券建玉と FIRE 想定建玉の整合チェック | OpenClaw scheduled job + Gmail connector |

## なぜ Mac mini が必要か

- 仕事用 PC(StudioZ 業務用)を占有しない
- 24時間起動しやすい(省電力、静音)
- Dashboard や FIRE 監視の母艦
- Stage 3 (Live Advisory) 時の安定性
- OpenClaw を常駐させる専用マシン

## 稼働スケジュール(v3.3)

| 時間帯 | 動作内容 |
|---|---|
| 06:30 | 起動。ニュース・開示・候補準備 |
| 08:00 | 朝分析開始 |
| 08:30 | 朝レポート送信(LINE) |
| 08:40 | 候補確定(実行候補 1〜3件) |
| 08:40 〜 09:00 | Fujiwara さんが寄り前指値予約発注(指示に従って) |
| 09:00 〜 15:30 | 監視・LINE通知(場中フル稼働) |
| **14:45 / 14:55 / 15:05 / 15:10 / 15:15** | **🚨 緊急ポジション整理アラート(建玉残あり時)** |
| 15:30 〜 16:30 | 引け後集計・レビューレポート送信 |
| 夜間 | Pattern Research / Evaluation Agent 分析 / ダッシュボード更新 |

## OpenClaw 障害時の縮退運転設計(v3.3 新規)

**Fujiwara さんの要望**: OpenClaw が止まった時、どのエージェントは完全停止し、どの機能は稼働させ続けるかを明確化する。

### 完全停止させる機能(OpenClaw 依存が大きい)

| 機能 | 理由 |
|---|---|
| 候補抽出エージェント群 | エージェント間連携が必須 |
| Paper Live(仮想売買シミュレーション) | 並列実行基盤が必須 |
| Pattern Research Loop | 夜間バッチ基盤が必須 |
| Evaluation Agent の分析 | 複数ソース統合が必須 |
| Dashboard Backend | 複数データ集約 |

### 独立で動かす機能(OpenClaw 障害でも継続可能)

| 機能 | 実装方式 |
|---|---|
| **LINE 緊急ポジション整理アラート** | macOS 標準 Cron で独立スクリプト起動、OpenClaw 非依存 |
| **OpenClaw 停止通知 LINE** | macOS の監視スクリプト(ハートビート検知)が LINE 送信 |
| **Dashboard 最終表示**(読み取り専用) | 静的な DB 内容の表示のみ |

### 縮退モード遷移

OpenClaw が死んだ瞬間に:

1. macOS Cron が OpenClaw のハートビート不在を検知(1分以内)
2. 自動で「FIRE 縮退モード起動通知」を Fujiwara さんに LINE 送信
3. 「**現在 FIRE は縮退運転中です。新規発注は推奨しません。既存建玉の管理のみ続けてください**」と明示
4. Fujiwara さんは全手動で運用継続(FIRE がない状態と同じ)
5. 場中なら既存建玉のみ管理、新規エントリーは見送り
6. OpenClaw 復旧後、FIRE が想定建玉と楽天証券実態を照合し直す

## Stage ごとの常駐要件

| Stage | 常駐の必要性 |
|---|---|
| Stage 0: Backtest | 常駐不要(手動実行) |
| Stage 1: Replay Simulation | 常駐不要 |
| Stage 2: Paper Live | 取引時間中の常駐が必要 |
| **Stage 3: Live Advisory** | **取引時間中の常駐が必須**、夜間は Pattern Research |
| Stage 4: Full Auto(将来) | 24時間フル常駐必須 |

## 本章の要件ID(v3.3)

- **R-23-01** 常駐基盤をUIで制御
- **R-23-02** **(v3.3改訂)オーケストレーション層 = OpenClaw on Mac mini(採用)**
- **R-23-03** **(v3.3改訂)実行層 = Fujiwara さん(人間)による手動発注**
- **R-23-04** FIRE Runtime コンポーネント6種(v3.3 で Reconciliation Worker追加)
- **R-23-05** Mac mini 必要性 4理由
- **R-23-06** 稼働スケジュール推奨(緊急ポジション整理アラート含む)
- **R-23-07** Stage ごと常駐要件
- **R-23-08** **(v3.3新規)OpenClaw 障害時の縮退運転設計**
- **R-23-09** **(v3.3新規)独立で動く機能:LINE緊急アラート(macOS Cron)、OpenClaw停止通知、Dashboard 読み取り専用**
- **R-23-10** **(v3.3新規)OpenClaw 停止時に Fujiwara さんに即 LINE 通知**

## v3.3 で撤回した要件

v3.1 R-23-03「OpenClaw 不採用 専用ランナー」→ **撤回。OpenClaw 採用**

## ナビゲーション

[[FIRE_要件書_第22章_データソース依存と縮退運転設計_穴3対策_|← 第22章]] | [[FIRE_要件書_第24章_機能の絞り込みと監視階層設計_穴4対策_|第24章 →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
