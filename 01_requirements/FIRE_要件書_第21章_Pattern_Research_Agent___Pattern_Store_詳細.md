---
type: requirement_chapter
chapter: "21"
title: "Pattern Research Agent / Pattern Store 詳細"
version: v3.4
updated: 2026-05-03
---

# 第21章: Pattern Research Agent / Pattern Store 詳細

第11章のPattern Store概要を補完する詳細設計方針。FIREは再現性の高い勝ちパターンを継続的に発見・蓄積・参照・監査することで、運用精度と信頼性を高める。

## 基本思想

FIREは単にその場でシグナルを生成するのではなく、過去に機能した勝ちパターンを蓄積し、現在の局面をそれに照合し、再現性の高い局面だけを優先的に実行する構造を採用する。また、過去に有効だったパターンが市場環境の変化によって劣化することを前提とし、リアルタイム相場における観測結果を通じて既存パターンの現役度を継続監査する。

**三層構造**: 過去から学ぶ → 現在に当てはめる → 運用中に生死を監視する

## Pattern Research Agent の役割

Pattern Research Agentは、FIREのパターン研究責任者とする。単なる分析補助ではなく、勝ちパターン候補の発見・定義・更新提案を担う研究エージェントと位置づける。

**主な役割**: 過去相場データの分析 / 類似局面のクラスタ化 / 勝ちパターン候補の発見 / 負けパターン候補の発見 / Candidate Pattern Poolの更新 / 既存パターンの劣化検知 / 新規パターン候補の提案 / DEATH NOTE候補の提案 / REHAB候補の提案

**重要方針**: 過去データだけを研究対象にしない。リアルタイム相場の観測ログも研究対象に含める。市場レジーム別に再現性を評価する。後知恵だけでパターンを作らない。パターンの採用・凍結・廃止は提案までとし、**自動反映しない**。

## DEATH NOTE の詳細ルール

DEATH NOTEは「現時点で無効化・凍結されたパターン」の保管庫であり、**永久削除ではない**。

| ステップ | 条件 | 判定 |
|---|---|---|
| Step 1 | 直近5回で4敗 | 要警戒 |
| Step 2 | 直近10回で勝率30%未満、平均RRも低下 | 凍結候補 |
| Step 3 | 直近20回で期待値マイナス、地合い補正後も弱い | DEATH NOTE入り |

**重要ルール**: 単純なN連敗だけで即廃止しない。期待値と市場レジームを考慮。DEATH NOTE入りは**提案まで**とし、最終承認はユーザーが行う。

## REHAB Queue

一度凍結またはDEATH NOTE入りしたパターンのうち、再検証する価値があるものの保管庫。
- 決算シーズンのみ復活の可能性
- 強地合いでのみ機能
- セクター限定で復活の可能性
- 執行品質改善後に再挑戦できる

**方針**: REHABは**自動復帰しない**。Pattern Research AgentとEvaluation Agentが再検証候補として提案する。採用再開もユーザー承認制。

## 勝ちパターンの採用と承認フロー

Pattern Research Agentは勝ちパターン候補を提案できるが、Pattern Libraryへの正式採用は**自動で行ってはならない**。

**流れ**: Pattern Research Agentが候補を発見 → Candidate Pattern Poolに登録 → Simulation / Paper Liveで検証 → Evaluation Agentが評価 → ユーザーが承認 → 承認後にPattern Libraryへ正式採用

## セクター/銘柄個別性への対応(穴12対策)  
  
パターンには「半導体株の米SOX連動寄付ギャップ」「トヨタの為替連動」「海運株のBDI連動」のような、セクターまたは特定銘柄群でしか機能しない特性を持つものが存在する。命名規則(R-31-01: `material-regime-price_action-time-liquidity` 5セグメント)だけではこれらを表現できないため、パターンに以下のメタデータを持たせて対応する。詳細は[[FIRE_要件書_第41章_セクター_銘柄特有性対応_穴12対策_|第41章: セクター/銘柄特有性対応]]を参照。  
  
| 項目 | 内容 |  
|---|---|  
| sector_filter | パターンが機能するセクター名のリスト(例: `["半導体","電気機器"]`)。空ならセクター制限なし |  
| symbol_whitelist | パターンが機能する銘柄コードのリスト(例: `["8035","6920"]`)。空なら銘柄制限なし |  
| symbol_blacklist | パターンを適用しない銘柄コードのリスト |  
  
**運用ルール**:  
- パターン命名規則(R-31-01)はそのまま維持。`material-regime-price_action-time-liquidity` の5セグメント  
- セクター/銘柄絞り込みはパターンの metadata として管理(命名規則は変更しない)  
- 類似局面検索(R-37-08)はマッチング前にこのフィルタを適用  
- Pattern Research Agentはセクター別成績集計・銘柄群絞込み提案を行う(R-21-15)  
- フィルタ変更もユーザー承認制(R-21-13と同じ扱い)

## 責務の一言まとめ

| 役割 | 担当 |
|---|---|
| 作る人(パターン発見・定義) | Pattern Research Agent |
| 保管庫 | Pattern Store(Feature Store / Live Research Log含む) |
| 使う人(実行時に参照) | Daytrade Selection Agent / Trade Decision Agent |
| 監査する人 | Evaluation Agent |
| 最終承認者 | ユーザー(Fujiwara) |

## 空売りパターン保存項目 (v3.4 追加)

空売りパターン (direction="short") は買いパターンの保存項目に加え、以下の項目を保存する:

| カラム | 型 | 説明 |
|---|---|---|
| direction | enum("long","short") | 必須。"short" 固定 |
| setup_type | enum("S1_悪材料初動","S2_決算失望","S3_主役崩れ","S4_VWAP割れ","S5_セクター流出") | 空売りレーン |
| borrow_status | enum("貸借可","貸借不可","不明") | 貸借区分 |
| short_sell_regulation_status | enum("規制なし","トリガー抵触","売禁") | 規制有無 |
| inventory_status | enum("在庫あり","在庫なし","不明") | 一般信用売り在庫 |
| reverse_interest_risk | enum("制度未使用","低","中","高") | 逆日歩リスク (制度信用使用時のみ) |
| entry_price | numeric | 信用売建価格 |
| stop_price | numeric | 返済買い損切価格 (entry_price より上) |
| take_profit_price | numeric | 返済買い利確価格 (entry_price より下) |
| max_adverse_excursion | numeric | 最大不利方向到達価格 (空売り中最も上昇した価格) |
| max_favorable_excursion | numeric | 最大有利方向到達価格 (空売り中最も下落した価格) |
| execution_quality | numeric | 執行品質スコア (R-17-09 と同基準) |
| pattern_win_rate | numeric | 同 setup_type の過去勝率 |
| average_profit | numeric | 平均利益 |
| average_loss | numeric | 平均損失 |
| expected_value | numeric | 期待値 |
| regime | enum("上昇","下落","レンジ","急反発","急落") | 当日レジーム |
| sector_flow | enum("流入","中立","流出") | セクター資金フロー |
| index_trend | enum("強上昇","弱上昇","レンジ","弱下落","強下落") | 当日指数トレンド |

### 本章追加要件ID (v3.4 — 既存 R-21-08/09 と衝突回避のため R-21-21 から採番)
- **R-21-21** 空売りパターン保存項目 (direction/setup_type/borrow_status/regulation/inventory/reverse_interest/MAE/MFE/regime/sector_flow/index_trend 等)
- **R-21-22** STOP > ENTRY > TP の不変条件 (空売り invariant)

## 本章の要件ID

- **R-21-01** 三層構造: 過去→現在→運用中生死監視
- **R-21-02** Pattern Research Agent役割
- **R-21-03** 過去パターン研究対象
- **R-21-04** リアルタイムパターン研究
- **R-21-05** Pattern Library保存項目
- **R-21-06** DEATH NOTE段階判定
- **R-21-07** DEATH NOTEユーザー承認
- **R-21-08** REHAB Queue
- **R-21-09** REHAB自動復帰禁止・承認制
- **R-21-10** 実行エージェントのPattern Store参照
- **R-21-11** Feature Store
- **R-21-12** Live Research Log
- **R-21-13** 採用・承認フロー
- **R-21-14** 段階的実装4 Phase
- **R-21-15** セクター/銘柄絞り込みパターン対応(穴12対策)


---

## ナビゲーション

[[FIRE_要件書_第20章_FIREの勝率優位性の根拠|← 第20章: FIREの勝率優位性の根拠]]　|　[[FIRE_要件書_第22章_データソース依存と縮退運転設計(穴3対策)|第22章: データソース依存と縮退運転設計(穴3対策) →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
- タスク: [[../02_todo/_index|TODO索引]]
- 運用ルール: [[../タスク運用ルール|タスク運用ルール]]
