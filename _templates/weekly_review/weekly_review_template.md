---
date: {{date:YYYY-MM-DD}}
week: {{date:YYYY-[W]WW}}
type: weekly_review
stage: ""
m_phase: ""
mood: ""
tags: [weekly_review]
---

# 週次レビュー {{date:YYYY-MM-DD}} ({{date:[W]WW}})

> 記入時刻: {{date:YYYY-MM-DD}} {{time:HH:mm}}  推奨所要 30 分
> 空欄・「該当なし」許容。実数値が後追いで取れる場合はバックフィル可。

<!--
記入対象セクションの目安 (Stage 別):

| Stage           | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 |
|-----------------|---|---|---|---|---|---|---|---|
| Stage 0 (BT)    | ● | ● | △ | ● | ● | ● | ● | ○ |
| Stage 1 (Sim)   | ● | ● | △ | ● | ● | ● | ● | ○ |
| Stage 2 (PL)    | ● | ● | ● | ● | ● | ● | ● | ○ |
| Stage 3 (LA)    | ● | ● | ◎ | ● | ● | ● | ● | ○ |

● = 必須 / ◎ = 主役 / △ = 該当のみ / ○ = 任意

このコメントは記入後に削除して構いません。
-->

## 1. ステージ位置と週次サマリー

| 項目 | 値 |
|---|---|
| 現在 Stage | <Stage 0 / Stage 1 / Stage 2 / Stage 3> |
| 当該月 (M-X) | <M-3 / M-2 / M-1 / M=0 / M+1 / M+2 / M+3> |
| 次 Stage 移行予定 | <YYYY-MM-DD or 未定 / N 日後> |
| Stage Gate 進捗 | <例: F266 ブロック中、events>0 確認待ち> |
| 今週の Mood | <順調 / 想定内 / 想定外 / 黄信号 / 赤信号> |

### 今週の最大トピック (3 行以内)

<!-- 一週間を一文で振り返るなら、を 3 行。最大の進捗 + 最大のブロッカーを必ず含める -->

-
-
-

---

## 2. 開発進捗 (TODO Excel との連携)

<!-- TODO_Master Google Sheets と Obsidian 02_todo/ ノートの両方を参照。
     ID は [[F267_features_pipeline|F267]] のような Wikilink 推奨。
     02_todo/ にノートがなくてもリンク切れで OK (将来作成で自動接続)。 -->

### 今週クローズしたタスク

<!-- TODO_Master ステータスが「未着手/実装中 → 完了」になったもの -->

- [x] **FXXX**: <タスク名> — <短い成果サマリ>

### 今週着手したタスク (進行中)

- [ ] **FXXX**: <タスク名> — 進捗 X% / 完了見込み YYYY-MM-DD / ブロッカー: <あれば>

### 想定外で発生した新規タスク

<!-- 派生起票 (例: F267, F268, F269)。「何が想定外だったか」を一行で。
     原因が log.md / 03_design/ のどこにあるかリンクすると後追いしやすい -->

- **FXXX**: <タスク名> — 起因: <短い説明 + リンク>

### 来週着手予定 (優先度順 3〜5 件)

1. **FXXX**: <タスク名> (優先度: 最優先 / 高 / 中)
2.
3.

---

## 3. 運用成績 (Stage 2 以降)

<!-- Stage 0/1 は「該当なし」と記入してスキップ可。
     値の取得 SQL は _templates/weekly_review/README.md を参照。 -->

### トレード基本指標

| 指標 | 今週 | 累積 |
|---|---|---|
| 営業日数 | <N 日> | <N 日> |
| トレード数 | <N 件> | <N 件> |
| 期待値 (RR × 勝率 - 1) | <±0.XX> | <±0.XX> |
| 最大 DD | <-X.X%> | <-X.X%> |
| 強制クローズ発動 | <N 回> | <N 回> |

### F053 promotion_check (Paper Live → Semi Auto 5 項目)

<!-- 直近 run の `python -m simulation.paper_live --check-promotion <run_id>` 結果 -->

- run_id: <PL-...>
- 全体判定: <PASS / FAIL>
- [ ] sample_size (>= 20 営業日 or >= 50 トレード)
- [ ] expected_value (勝率 / RR / 最大 DD)
- [ ] halt_function (停止機能動作確認)
- [ ] close_function (強制クローズ 100%)
- [ ] simulation_accuracy (optimistic_bias_score <= 0.7)

### F241 live_advisory_check (Stage 3 移行 9 項目)

<!-- 直近 `--check-live-advisory <run_id>` 結果。Stage 3 移行直前のみ重要 -->

- 全体判定: <PASS / FAIL / 未実施>
- F053 5 項目: <X / 5 PASS>
- LINE 5 部屋疎通: <PASS / FAIL / 未実施>
- 緊急アラート (14:45/14:55/15:05/15:10/15:15): <PASS / FAIL / 未実施>
- Fujiwara 受容体制 (`--fujiwara-accept`): <PASS / 未承認>
- F119 昇格提案書: <生成済 / 未生成>

---

## 4. データ・パイプライン健全性

<!-- 値の取得 SQL は README.md の「データ取得 SQL」を参照。
     Chain 直近実行は ~/fire/data/fire.db に依存するため、Run a/b/c の
     実行ターミナルが空いているタイミングで取得 -->

| 項目 | 値 | 前週差分 |
|---|---|---|
| features 行数 | <N 行> | <±N> |
| features カバレッジ (Core500 比) | <X.X%> | <±X.X%> |
| announcements 行数 | <N 件> | <±N> |
| announcements 直近 6 ヶ月カバレッジ | <X.X%> | <±X.X%> |
| market_prices_daily 銘柄数 | <N 銘柄> | <±N> |
| market_prices_daily 行数 | <N 行> | <±N> |
| パターン状態: active | <N 件> | <±N> |
| パターン状態: candidate | <N 件> | <±N> |
| パターン状態: archived | <N 件> | <±N> |

### Chain 直近実行結果

| run_id | events | n_ticks | 結果 |
|---|---|---|---|
| <PL-...> | <N> | <N> | <success / events=0 異常 / 中断> |

---

## 5. リスク・ブロッカー

<!-- log.md と 07_incidents/ のクロスリンクが理想 -->

### 今週発生したインシデント

- **<日付>**: <事象> — log: [[log.md]] / 07_incidents: [[YYYY-MM-DD_<title>]]

### 未解消ブロッカー

| TODO ID | ブロック内容 | 解消見込み |
|---|---|---|
| **FXXX** | <内容> | <YYYY-MM-DD or Run X 結果次第> |

### Stage 移行ブロッカー

<!-- F266 のような最終ゲート系。Stage 移行直前のみ重要 -->

- **FXXX**: <ゲート項目> — 残作業: <内容>

---

## 6. 学び・改善

<!-- log.md (Karpathy 式) からの抽出。「なぜ起きたか」「次回どう防ぐか」を
     セットで記述すると価値が高い -->

### 今週の発見・気づき

-
-

### パターン発見・修正 (Pattern Store)

<!-- R-13-08 (Pattern Store 規律) 承認案件。candidate → active 昇格 / 廃番など -->

- **<pattern_id>**: <承認/廃番/修正> — 理由: <短い説明>

### ルール追加・修正提案 (要件書 v3.x への反映候補)

<!-- 要件書 v3.4 などへの差分案。即反映ではなく「次回バージョンに繰越し」 -->

- **R-XX-XX**: <提案内容> — 動機: <短い説明>

---

## 7. 来週の計画

### 最優先タスク (1〜3 件)

1. **FXXX**: <タスク名> — 完了基準: <チェック可能な条件>
2.
3.

### Stage 移行イベント

<!-- 該当なしなら「該当なし」と記入 -->

- <YYYY-MM-DD>: <Stage X → Stage Y 移行ターゲット日 / マイルストーン>

### 外部依存

- J-Quants API: <メンテ予定 / 仕様変更 / 該当なし>
- TDnet: <該当なし / 仕様変更>
- 楽天証券 iSPEED: <該当なし>
- LINE Messaging API: <該当なし>

### Fujiwara さんの個人スケジュール影響

<!-- F239 連携、対応困難日、出張・休暇など。記述したくない場合は省略可 -->

- <YYYY-MM-DD〜YYYY-MM-DD>: <影響内容>

---

## 8. メタ・ジャーナル (任意)

### 今週やってよかったこと (3 つ)

1.
2.
3.

### 来週やめること (1 つ)

- <例: 「動いた」と「機能した」を区別せず Chain exit 0 を success と扱う>

### 投資家としての心理状態

<!-- オーバーフィット警戒、損失受容度、規律維持の自己観察など。
     一行でも、長文でも、空欄でも OK -->

-

---

## 関連リンク

- 直近 log.md エントリ: [[log.md]]
- 直近 03_design/ ドキュメント: [[03_design/<最新ファイル>]]
- 直近 07_incidents/: [[07_incidents/<最新ファイル>]]
- 前週レビュー: [[weekly_review_<前週日付>_W<前週番号>]]
- 月次レビュー: [[06_monthly/<該当月>]]
