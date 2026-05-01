---
type: requirement_chapter
chapter: "15"
title: "Claude Code実装に向けた次のステップ(v3.3)"
version: v3.3
updated: 2026-04-24
---

# 第15章: Claude Code実装に向けた次のステップ(v3.3)

## 実装順序(v3.3 優先順位)

1. この決定事項レジスターを元に **README.md および CLAUDE.md** の初稿を作成する
2. Claude Code に渡す実装開始指示を、このレジスターの内容を削らずに作成する
3. **【最優先】OpenClaw セットアップ**(オーケストレーション層) - v3.2 から継続採用
4. **【最優先】Pattern Store / Feature Store / Runtime / Simulation(過去データリプレイ環境)の骨組み**
5. リアルタイム Paper Live 環境と **LINE通知基盤**(5部屋体制)を構築
6. 第40章の実装ロードマップに従い、Live Advisory まで段階的に構築

## 崩してはならない前提(v3.3: 9項目)

- FIREのエッジは**再現性優位**
- Pattern Storeは**大量蓄積+階層管理**
- パターンは**段階検証を通過して昇格**(Stage飛ばし禁止: 0→1→2→3)
- **Evaluation Agentは提案のみ**(自動反映禁止)
- Mac mini常駐ランタイム前提(**OpenClawで実現**)
- UIはレトロ炎テーマ
- 初期本番はデイトレ主軸
- **(v3.3改訂)全自動は目指さない。Live Advisory(半自動主軸)がFIREの完成形**
- 完全放置はしない(監督者付き自律運用)

## v3.3 で撤回された前提

- v3.1/v3.2 の「全自動を目指す」は撤回
- 「全自動は目指すが完全放置はしない」→「全自動は目指さない。Live Advisoryが完成形」

## 本章の要件ID(v3.3)

- **R-15-01** README.md/CLAUDE.md初稿作成
- **R-15-02** **(v3.3改訂)OpenClaw セットアップを最優先に追加**
- **R-15-03** Pattern Store/Feature Store/Runtime/Simulation 骨組み最優先
- **R-15-04** Paper Live環境 + LINE通知基盤(5部屋)
- **R-15-05** **(v3.3改訂)実弾執行基盤=人間発注(LINE通知経由)**
- **R-15-06** エージェントプロンプトファイル整備
- **R-15-07** **(v3.3改訂)崩してはならない前提9点(「全自動目指さない」追加)**

## ナビゲーション

[[FIRE_要件書_第14章_データソース優先順位と縮退運転|← 第14章]] | [[FIRE_要件書_第16章_Mac_miniセットアップ|第16章 →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
