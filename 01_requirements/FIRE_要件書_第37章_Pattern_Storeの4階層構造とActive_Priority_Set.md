---
type: requirement_chapter
chapter: "37"
title: "Pattern Storeの4階層構造とActive Priority Set"
version: v3.4
updated: 2026-05-03
---

# 第37章: Pattern Storeの4階層構造とActive Priority Set

## 「保存数」と「実行で使う有効数」は別

ユーザーのイメージ(勝ちパターンをどんどんストックし、DBに参照しにいき、成功の再現性を判定する)は正しい。ただし**「大量に保存すること」と「大量を同列に実行判断へ入れること」は分ける**。

## Pattern Storeの4階層構造

| 階層 | 名称 | 内容 | 件数目安 |
|---|---|---|---|
| Layer 1 | Pattern Archive | 全履歴。勝ち・負け・候補・廃止・Rehab候補を全部含む巨大な蓄積庫 | 上限ほぼなし(100〜200以上も可) |
| Layer 2 | Valid Pattern Library | 現在の市場で意味があると判定されたパターン群 | 20〜100でもよい |
| Layer 3 | Active Priority Set | 本番の執行判断で強く使う「現在の主力パターン」 | 最初は3〜8、成熟後でも10〜15まで |
| Layer 4 | Death Note / Rehab | 凍結・再検証対象 | 制限なし |

## パターンの状態管理(7状態)

| 状態 | 内容 | 実行で使えるか |
|---|---|---|
| candidate | 研究中・検証前 | 不可 |
| paper_validating | Paper Live検証中 | 不可(仮想のみ) |
| approved_active | 本番有効(Active Priority Set) | 可 |
| watchlist | 有効だが優先度低い | 条件付きで可 |
| frozen | 凍結中 | 不可 |
| death_note | 廃止 | 不可(除外) |
| rehab | 再検証中 | 不可 |

## パターンの格付け(S〜X)

数を制限するよりランクを持たせる方が重要。

| ランク | 内容 |
|---|---|
| S | 現在の主力中の主力。再現性が高く、今の市場でも有効 |
| A | 有効パターン。十分参照価値あり |
| B | 有望だが使用優先度は低い |
| C | 研究対象・補助参照 |
| D | 凍結候補 |
| X | DEATH NOTE(廃止) |

## 実行エージェントのPattern Store参照フロー

1. Pattern Library全体を検索(類似局面を広く引く)
2. 類似スコア順に並べる
3. 類似件数 / 勝率 / 期待値 / レジーム適合を確認
4. Active Priority Set(approved_active)に含まれるものは重みを上げる
5. watchlist パターンは参考点として扱う
6. death_note は除外

**要約**: 全部を検索するが、全部を同じ重みでは扱わない構造。

## Pattern IDと命名規則

各パターンは以下を必ず持つ。

| フィールド | 内容 |
|---|---|
| pattern_id | 一意のID |
| pattern_name | 命名規則に従った名称 |
| pattern_version | バージョン番号 |
| parent_pattern_id | 派生元(派生がある場合) |

**命名規則**: `[material]-[market_regime]-[price_action]-[time_window]-[liquidity_band]`

例: `earnings_upside-strong_market-breakout-AM-highliq` / `buyback-neutral_market-pullback-AM-highliq`

## バージョン管理の原則

- 同じ名前のまま中身を変えない。変更したら新バージョンにする
- 必須フィールド: version / created_at / updated_at / created_by / approved_by / change_reason

## Pattern Storeの定期棚卸し(週次)

Evaluation Agentが週次で以下を確認・提案する。

- approved_active が増えすぎていないか
- candidate が滞留していないか
- death_note が古いまま放置されていないか
- rehab 候補を再評価すべきか
- 類似パターン統合が必要か

## 重複パターン統合ルール

以下の条件に該当するパターンは統合候補とする。

- コア特徴量がほぼ同じ
- 成績差が小さい
- 地合い補正後も差が小さい
- 実行条件が実質同じ

**対応**: 統合 / 親子関係にする / 片方をalias扱いにする

**Pattern Storeは辞書であって、ゴミ置き場ではない**。

## 空売り固有の劣化判定 (v3.4 追加)

DEATH NOTE 判定 (R-21-06 Step1/2/3) に加え、空売りパターンには以下の追加劣化判定を適用する。

### 空売り追加劣化条件

| 条件 | 対応 |
|---|---|
| 直近10回の期待値マイナス | watchlist へ降格 |
| 直近20回の期待値が過去平均の50%未満 | Frozen |
| **踏み上げ損切りが3回連続** | **DEATH NOTE 入り** (買い側より厳しい) |
| 指数反発局面 (regime="急反発") での負けが多い | そのレジームでは使用禁止 (regime-conditional freeze) |
| 貸借・在庫・規制で実行不能が一定割合超 | 執行品質低下として降格 |

### 空売り Rehab 条件

凍結後の復活条件は買い側より厳しくする:
- 類似レジーム下で Paper Live 10回以上
- 期待値プラス
- 最大DD想定内
- 執行可能性あり (貸借・在庫・規制チェック通過率が80%以上)
- 規制・在庫チェックの通過率が80%以上

### 本章追加要件ID (v3.4 — 既存 R-37-07/08/09 と衝突回避のため R-37-13 から採番)
- **R-37-13** 空売り追加劣化条件: 踏み上げ3連続で DEATH NOTE、レジーム別禁止、執行不能多発で降格
- **R-37-14** 空売り Rehab 条件: 類似レジーム10回以上 + 期待値+ + 規制通過率80%以上
- **R-37-15** 状態遷移は direction 別に独立 (空売りパターンの遷移は空売り履歴のみで判定)

## 本章の要件ID

- **R-37-01** 保存数と実行で使う有効数は別
- **R-37-02** Layer1 Pattern Archive(100+)
- **R-37-03** Layer2 Valid Pattern Library(20-100)
- **R-37-04** Layer3 Active Priority Set(3-15)
- **R-37-05** Layer4 Death Note/Rehab
- **R-37-06** パターン7状態
- **R-37-07** ランク6段階 S/A/B/C/D/X
- **R-37-08** 実行エージェント参照フロー5ステップ
- **R-37-09** pattern_id命名規則 派生管理
- **R-37-10** バージョン管理原則
- **R-37-11** 週次定期棚卸し
- **R-37-12** 重複パターン統合ルール


---

## ナビゲーション

[[FIRE_要件書_第36章_初期特徴量セットの詳細定義|← 第36章: 初期特徴量セットの詳細定義]]　|　[[FIRE_要件書_第38章_運用モード___休止___旅行モード方針|第38章: 運用モード / 休止 / 旅行モード方針 →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
- タスク: [[../02_todo/_index|TODO索引]]
- 運用ルール: [[../タスク運用ルール|タスク運用ルール]]
