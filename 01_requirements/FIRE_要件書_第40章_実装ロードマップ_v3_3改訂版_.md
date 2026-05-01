---
type: requirement_chapter
chapter: "40"
title: "実装ロードマップ(v3.3改訂版)"
version: v3.3
updated: 2026-04-24
---

# 第40章: 実装ロードマップ(v3.3改訂版)

## v3.3 での位置づけ

v3.2 で新設された章。v3.3 では発注の自動化を廃止したため、実装着手順を再定義する。

## 実装着手順(v3.3)

### Phase 1: 基盤構築(P0 最優先)

1. **Mac mini M4 Pro 受領・初期設定**
2. **OpenClaw セットアップ**(オーケストレーション層)
   - Mac mini上でのインストール
   - 常駐・スケジュール基盤の確立
   - ハートビート・自動復旧テスト
3. **Pattern Store / Feature Store / Runtime 骨組み**
   - これが FIRE の心臓
   - OpenClaw 上で動くエージェント群の基盤

### Phase 2: エッジ開発(P0〜P1)

4. Pattern Research Agent 実装(過去パターン研究・Candidate Pool 更新)
5. Feature Store 実装(6系統特徴量 - 第36章)
6. 類似局面検索エンジン実装(再現性スコア算出)
7. Daytrade Selection Agent 実装(1〜3銘柄厳選)
8. Market Intelligence Agent 実装(朝レポート生成)
9. Trade Decision Agent 実装(**注文指示生成 → LINE 通知**)
10. Monitoring & Alert Agent 実装(**緊急ポジション整理アラート含む**)
11. Review & Journal Agent 実装

### Phase 3: 半自動執行基盤(P1)

12. **LINE通知基盤**(5部屋体制:エントリー/執行損益/レポート/システム警告/緊急ポジション整理)
13. **朝レポート PDF 生成**(NotebookLM風)
14. **引け後レビュー PDF 生成**
15. **Dashboard(運用コンソール)**構築

### Phase 4: 約定取込(M+2以降)

16. Gmail API 連携
17. 楽天証券約定通知メールのパース
18. 自動取込・建玉整合チェック

### Phase 5: 補助機能(中優先)

19. Google Calendar 連携(Fujiwara さんスケジュール認識)
20. Evaluation Agent(評価提案書生成)
21. OpenClaw 障害時の縮退運転機構(macOS Cron による緊急ポジション整理アラート独立稼働)

### Phase 6: 検証・昇格(P2以降)

22. Stage 0 (Backtest) 実装
23. Stage 1 (Replay Simulation) 実装
24. Stage 2 (Paper Live) 実装
25. Paper Live で 20営業日 or 50トレード以上の実績蓄積
26. 昇格条件(第32章)クリア確認
27. **Stage 3 (Live Advisory) 移行 = M=0**

### Phase 7: 運用・改善(継続)

28. 日次・週次・月次レビュー
29. Pattern Library 定期棚卸し
30. Evaluation Agent 提案の検討・採用

### Phase 8: 将来オプション(M+3 以降の必要時)

31. Stage 4 (Full Auto) 検討
32. 証券会社公式 API 経由の自動発注(楽天証券公式 REST API が提供されれば)

## 着手順の絶対原則

1. **OpenClaw 基盤が最初**(全ての土台)
2. **Pattern Store / Feature Store が最優先**(エッジの本質)
3. **LINE 通知基盤は並行開発可**(Phase 2 の終盤から着手)
4. **約定取込は M+2 以降でよい**(初期は LINE 手動報告で十分)
5. **Stage 飛ばし禁止**

## 崩してはならない前提(v3.3 更新版)

以下は絶対原則。v3.3 でも変更なし:

1. FIRE のエッジは**再現性優位**(ブラックボックス AI 予測ではない)
2. Pattern Store は**大量蓄積 + 階層管理**(Archive/Library/Active Priority Set/Death Note・Rehab)
3. パターンは段階検証を通過して昇格(**Stage 飛ばし禁止**: 0→1→2→3)
4. Evaluation Agent は**提案のみ**(自動反映禁止・ユーザー承認制)
5. **Mac mini 常駐ランタイム前提**(OpenClaw で実現)
6. UI は**レトロ炎テーマ**
7. 初期本番は**デイトレ主軸**(スイング・長期は通知・提案のみ)
8. 全自動は目指さない(v3.3で方針転換)。**Live Advisory (半自動) が FIRE の完成形**
9. **完全放置はしない**(監督者付き自律運用)

## 本章の要件ID(v3.3)

- **R-40-01** Phase 1: 基盤構築(Mac mini + OpenClaw + Pattern Store 骨組み)
- **R-40-02** Phase 2: エッジ開発(エージェント群)
- **R-40-03** Phase 3: 半自動執行基盤(LINE通知 + Dashboard)
- **R-40-04** Phase 4: 約定取込(Gmail API)
- **R-40-05** Phase 5: 補助機能(Calendar、Evaluation、縮退運転)
- **R-40-06** Phase 6: 検証・昇格(Stage 0〜3)
- **R-40-07** Phase 7: 運用・改善(継続)
- **R-40-08** Phase 8: 将来オプション(Full Auto)
- **R-40-09** 着手順の絶対原則 5点
- **R-40-10** 崩してはならない前提 9点(v3.3 で「全自動目指さない」を追加)

## ナビゲーション

[[FIRE_要件書_第39章_運用立ち上げロードマップ_M-3_〜_M+3_|← 第39章: 運用立ち上げロードマップ]]　|　[[FIRE_要件書_第41章_セクター_銘柄特有性対応_穴12対策_|第41章: セクター/銘柄特有性対応(穴12対策) →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
