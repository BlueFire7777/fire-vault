---
title: F287 決算カレンダー × 決算短信 AI 分析 × スライド/PDF 生成 × LINE 通知ダッシュボード 仕様書 + 要件定義 v1.0
date: 2026-05-08
phase: F287-R0 (仕様設計 + feasibility 確認)
status: 仕様設計 + feasibility 確認 (Phase F287、F288 以降の前提)
related: F285_Research_Lane_requirements_and_spec_2026-05-08, F101_TDnet, F100_J-Quants, F119_Evaluation, F236_LINE5部屋, F210_phase_1b
trigger: HQ 並行作業指示 (2026-05-08、F284/F105 c6 backfill PID 92822 走行中の vault 作業)
---

# F287 決算カレンダー × 決算短信 AI 分析 × スライド/PDF 生成 × LINE 通知ダッシュボード 仕様書 + 要件定義 v1.0

★ 本仕様は **F285 Research Lane の Output Layer** として位置付け、
   保有銘柄・監視銘柄・主要銘柄の決算予定をダッシュボード化、決算短信 /
   IR 資料を AI 分析、スライド / PDF 化して LINE で確認できる
   Research Output 機能を設計する。**自動発注機能ではない**。投資判断は
   Fujiwara が最終確認、LINE 通知は判断材料の提示まで。

## 1. F287 の役割定義

### 1.1 位置付け

F287 は FIRE システムにおける **Research Output Layer** である:
- 上流: F285 Research Lane (Sector Flow / Cyclical Value / Earnings
  Preview / Dividend Growth + Watchlist Ranker) からの A1/A2 候補 +
  Fujiwara の保有 / 監視銘柄 + 主要大型株を入力とする
- 下流: Portfolio Engine (現 holding 評価) + Morning Report (毎朝 8:30) +
  LINE 通知 (Research / 朝レポート / 決算速報枠) に決算分析結果を配信

### 1.2 提供機能

1. **決算カレンダーダッシュボード**: 保有 / 監視 / Research / 主要銘柄の
   決算予定を一覧、重要度・事前注目理由・取得/分析/通知ステータスを管理
2. **決算短信 / IR 資料の自動取得**: TDnet / J-Quants / 会社 IR ページ
   から決算短信 PDF / XBRL / 決算説明資料 PDF を取得
3. **AI 分析パイプライン**: 取得した決算短信 / IR 資料を AI で分析、
   16 分析観点 (§6) を自動生成
4. **スライド / PDF 生成**: 1 銘柄 1 枚の速報スライド + 保有 / 監視
   まとめのデイリーデックを生成
5. **LINE 通知**: 決算速報通知 + 朝レポート枠 + Research 専用部屋に
   要約 + スライド/PDF リンクを配信

### 1.3 自動発注機能ではない (重要前提)

- 投資判断は **Fujiwara が最終確認**
- LINE 通知は分析結果・要約・資料リンク・スライド/PDF リンクの提示まで
- 楽天証券操作の自動化禁止 / Computer Use 禁止 (R-03-01 / R-13-08 準拠)

## 2. 戦略仮説 + 全体アーキ図

仮説: 決算短信は公開情報だが、(i) 取得・(ii) 数値抽出・(iii) コンセン
サス比較・(iv) 翌営業日反応仮説の 4 段階を高速・体系的に行うことで、
Fujiwara の決算後判断 (保有 / 増減 / exit) の質と速度が向上する。
FIRE のエッジ「再現性優位」+ Research Lane の「ファンダ先回り」の
延長線として位置付ける。

```
[情報源]
  TDnet 開示 (F101 Phase 2/3 既、HTML/XBRL 取得経路あり)
  TDnet PDF (決算短信 PDF / 決算説明資料 PDF、新規取得経路要)
  J-Quants /fins/announcement (F101 既)、/fins/statements (precheck 要)
  会社 IR ページ (各社個別、URL 取得後 fetch)
  Research Lane Watchlist (F285、A1/A2 候補)
  Portfolio (現 holding)
        ↓
[F287 Output Layer]
  ① 決算カレンダー DB (event 管理 + status track)
  ② 決算短信 / IR 資料取得パイプライン (PDF/XBRL/HTML)
  ③ AI 分析パイプライン (Claude API 経由、PDF 入力 + 16 観点)
  ④ スライド/PDF 生成 (python-pptx / Markdown→PDF / Claude 連携)
  ⑤ LINE 通知 (Research 専用 / 朝レポート / 決算速報枠)
        ↓
[出力先]
  Morning Report (8:30 JST) 内に決算予告 + 直近決算分析セクション
  Research 専用 LINE 部屋 (決算速報、要約 + リンク)
  Portfolio Engine (現 holding 銘柄の決算サマリ + 増減提案)
  F285 Watchlist Ranker (Earnings Preview Agent §4-3 への feedback)
        ↓
[Fujiwara 最終確認]
  手動発注 / 手動クローズ判断 (R-03-01)
  LINE 通知の要約 + スライド / PDF を確認、判断材料として活用
```

## 3. 決算カレンダー DB schema 案 (HQ 確認項目 1)

`earnings_calendar` テーブル (staging schema、F287 Phase で migration 追加):

| field | type | NOT NULL | 内容 |
|---|---|---|---|
| `code` | TEXT | ✅ | 証券コード (5 桁、F100 既 schema と整合) |
| `name` | TEXT | ✅ | 銘柄名 (J-Quants /equities/master 既) |
| `expected_date` | TEXT (YYYY-MM-DD) | ✅ | 決算発表予定日 |
| `expected_time` | TEXT (HH:MM) | | 発表予定時刻 (例 "15:30"、寄付き前 / 引け後 等) |
| `period_kind` | TEXT | ✅ | 決算種別 (例: "Q1 / Q2 / Q3 / Q4 / FY", "本決算 / 中間 / 四半期") |
| `holding_status` | TEXT | ✅ | 保有 / 監視 / Research_A1 / Research_A2 / 主要 / セクター代表 / 注目 |
| `importance` | INTEGER | ✅ | 重要度 1-5 (5 が最高、保有 + Research A1 等で高く) |
| `prior_attention_reason` | TEXT | | 事前注目理由 (例: "保有中 + Research A1 + 上方修正期待") |
| `tdnet_status` | TEXT | ✅ | 決算短信取得ステータス (`pending / fetched / failed / not_found`) |
| `ai_analysis_status` | TEXT | ✅ | AI 分析ステータス (`pending / analyzing / completed / failed`) |
| `slide_status` | TEXT | ✅ | スライド/PDF 生成ステータス (`pending / generating / completed / failed`) |
| `line_notified_at` | TEXT | | LINE 通知済みタイムスタンプ (ISO、未通知なら NULL) |
| `created_at` | TEXT | ✅ | レコード作成 (ISO) |
| `updated_at` | TEXT | ✅ | 最終更新 (ISO) |

主キー: `(code, expected_date, period_kind)` (再修正可能、status 上書き)

index 案: `expected_date` / `holding_status` / `importance DESC, expected_date`

外部参照テーブル (将来):
- `earnings_documents`: 取得した PDF / XBRL のローカル保存パス
- `earnings_analysis`: AI 分析結果 (16 観点の構造化 JSON)
- `earnings_slides`: 生成スライド/PDF パス
- `earnings_notifications`: LINE 通知履歴

## 4. 対象 Universe (HQ 確認項目 2)

| 区分 | 内容 | データソース | 想定件数 |
|---|---|---|---|
| 保有銘柄 | 現在の Fujiwara 保有 | FIRE positions テーブル (F051 既) | 5-20 |
| 監視銘柄 | Fujiwara が手動 watchlist 登録 | 新規 watchlist テーブル (F287 Phase) | 20-100 |
| Research Lane A1/A2 | F285 Watchlist Ranker の A1/A2 出力 | F285 R3 完了後 | 10-30 |
| 主要大型株 | TOPIX Core30 / Large70 | J-Quants /equities/master の市場区分 | 100 |
| セクター代表銘柄 | 各 33 業種の時価総額上位 | J-Quants daily 集計 (F285 Sector Flow Agent と連動) | 33 |
| 決算注目銘柄 | 直近で上方修正 / アナリスト評価変更があった銘柄 | TDnet (F101) + Earnings Preview (F285 §4-3) | 10-50 |

合計想定 universe: 200-300 銘柄 (重複除外、決算シーズンに集中)

## 5. 決算短信 / IR 資料の取得元 (HQ 確認項目 3)

| 系統 | 取得対象 | 既存実装 | 新規実装範囲 |
|---|---|---|---|
| TDnet | 適時開示 HTML + XBRL ZIP (決算短信本体) | F101 Phase 2 (HTML 取得 + direction 判定) / Phase 3 (XBRL parse) | PDF 直接取得経路追加 (= 短信 PDF / 決算説明資料 PDF) |
| J-Quants | /fins/announcement (決算予告 metadata) | F101 Phase 1 既 | カレンダー連動の自動更新 |
| J-Quants /fins/statements | 業績ヒストリー (4 期トレンド等) | F285 Phase R0 precheck で取得可否確認 | precheck 結果次第 |
| 会社 IR ページ | 決算説明資料 PDF (TDnet と並行公開) | なし | URL 取得後 fetch (爆発的多様性、各社個別 URL パターン) |
| 適時開示 PDF | 短信本体 PDF (XBRL 不在時の fallback) | なし | TDnet PDF URL から fetch |

★ TDnet HTML / XBRL は F101 で取得済、本タスクで **新規追加するのは
   PDF 直接取得 + IR ページ PDF**。

PDF 取得時の留意事項:
- 各社 IR ページの URL パターンが個別 (BeautifulSoup で IR ページから
  決算資料 link を抽出する必要)
- 大量取得は IR サーバー負荷になるため rate limit / robots.txt 確認
- PDF 文字化け / OCR 必要なケースもある (古い銘柄の scan PDF 等)

## 6. AI 分析観点 16 項目 (HQ 確認項目 6)

### 6.1 分析パイプライン

```
入力: 決算短信 PDF / XBRL / 説明資料 PDF
     ↓
[Claude API (claude-sonnet-4-6 / claude-opus-4-7)]
  prompt: 構造化分析テンプレート (16 観点 + JSON 出力指示)
  PDF input サポート: PDF 直接添付、構造化抽出可能
     ↓
[構造化 JSON 出力]
  16 観点を含む structured analysis result
     ↓
[earnings_analysis テーブルに保存]
```

### 6.2 16 分析観点

| # | 観点 | 抽出内容 |
|---|---|---|
| 1 | 売上 | 当期実績、前年同期比、会社予想比、市場予想比 |
| 2 | 営業利益 | 当期実績、前年同期比、会社予想比、市場予想比 |
| 3 | 経常利益 | 同上 |
| 4 | 純利益 | 同上 |
| 5 | 前年同期比 | 売上/各利益の YoY 増減率 |
| 6 | 会社予想比 | 通期会社ガイダンスに対する進捗、上振れ/下振れ |
| 7 | 市場予想比 | アナリスト予想中央値との比較 (取得可能な範囲) |
| 8 | 進捗率 | 通期予想に対する累計進捗 (Q1: 25% / Q2: 50% / Q3: 75% 基準) |
| 9 | 上方修正可能性 | 進捗率 + 過去パターンから上方修正確度を判定 |
| 10 | 増配可能性 | 配当性向 / フリーキャッシュフロー / 増配履歴から判定 |
| 11 | 配当性向 | 純利益に対する配当総額の比率 |
| 12 | 自社株買い | 自社株買い announcement の有無 / 規模 |
| 13 | セグメント別変化 | 主要セグメントの売上/利益 YoY、ミックス変化 |
| 14 | 為替/原材料/金利影響 | 決算短信の質的記述から impact factor 抽出 |
| 15 | ポジティブ材料 | 自由記述で 3-5 点抽出 |
| 16 | ネガティブ材料 | 自由記述で 3-5 点抽出 |

加えて派生分析:
- **翌営業日の株価反応仮説**: 16 観点を統合した上昇/横ばい/下落の確度
- **Research Lane Rank への影響**: F285 §4-3 Earnings Preview Agent
  へのフィードバック (rank 変更可能性)
- **保有判断への影響**: 保有銘柄なら継続 / 増 / 減 / exit の参考意見
  (提案のみ、Fujiwara が最終判断)

### 6.3 出力フォーマット (JSON 例)

```json
{
  "code": "72030",
  "expected_date": "2026-05-12",
  "period_kind": "Q4_FY",
  "metrics": {
    "revenue": { "actual": 1234567, "yoy_pct": 5.2, "vs_company_pct": 1.5, "vs_market_pct": 2.1 },
    "operating_profit": { ... },
    "net_profit": { ... }
  },
  "guidance_progress_pct": 78.5,
  "upside_revision_probability": 0.65,
  "dividend_increase_probability": 0.40,
  "buyback_announced": false,
  "segments": [ ... ],
  "fx_commodity_rate_impact": "...",
  "positive_factors": ["...", "..."],
  "negative_factors": ["...", "..."],
  "next_day_reaction_hypothesis": "上昇 (確度 60%)",
  "research_rank_impact": "Earnings Preview score +0.15、A2 → A1 候補",
  "holding_advice_for_fujiwara": "保有継続を推奨、ただし上方修正前の利食い検討余地あり"
}
```

## 7. スライド生成方式 (HQ 確認項目 5)

### 7.1 候補比較

| 形式 | 利点 | 欠点 | FIRE 自動化 |
|---|---|---|---|
| **Markdown** | 軽量、Obsidian 互換、AI 生成と相性良い | 視覚的訴求弱い | 容易 (Claude API 直生成) |
| **HTML** | スライド機能 (reveal.js 等)、ブラウザ閲覧 | リンクのみ、PDF 化に追加処理 | 容易 |
| **PDF (Markdown→PDF)** | 添付・印刷適、共有しやすい | スタイル制限 | pandoc / weasyprint で容易 |
| **PowerPoint (.pptx)** | 編集可能、企業資料風 | 複雑、Mac 環境依存 | python-pptx で可能 |

### 7.2 推奨方式 (Phase 案)

- **F291 Phase 1**: Markdown ベースで生成 (Obsidian 内で閲覧 / 共有)
- **F291 Phase 2**: Markdown → PDF 自動変換 (pandoc / weasyprint)
- **F291 Phase 3**: 必要なら python-pptx で PPT 生成 (企業資料風が必要な場合のみ)
- **Claude for PowerPoint 等のクラウド連携**: feasibility precheck で
  Claude.ai の PowerPoint 連携 plugin (もし存在) を確認、自動パイプ
  ラインから呼び出し可能か検証 (現状不明、§14 参照)

### 7.3 スライド種別

| 種別 | 内容 | 配信先 |
|---|---|---|
| 1 銘柄 1 枚速報スライド | 決算 16 観点要約 + 翌営業日反応仮説 | LINE 決算速報枠 |
| デイリーデック (保有/監視まとめ) | 当日決算発表全銘柄を 1 デック化 | 朝レポート枠 (翌朝 8:30) |
| Research Watchlist デック (週次) | A1/A2 候補の決算スケジュール一覧 | 週次レビュー (Fujiwara) |

### 7.4 自動生成と手動レビューの境界

- **自動**: 数値抽出、スライド構造、色分け、グラフ生成
- **手動レビュー必須** (= Fujiwara が確認してから保存):
  - 投資判断 (保有 / 増減 / exit) を含む箇所
  - Research Lane Rank の変更提案
  - 重要 holding 銘柄 (importance >= 4)
- 自動生成は draft (`draft/` ディレクトリ) に置き、Fujiwara が承認後に
  公式 (`approved/`) へ移動。LINE 通知は approved 限定。

## 8. LINE 通知方式 (HQ 確認項目 7、§F285 §10 と整合)

### 8.1 通知部屋 (F285 §10 修正 4 と整合、ENTRY 部屋と分離)

| 通知種別 | 通知部屋 | タイミング | 内容 |
|---|---|---|---|
| 決算速報 | **Research / 決算専用部屋** (新設候補、F285 §10 案 A) | 発表後 30 分以内 (PDF 取得 + AI 分析完了次第) | 16 観点要約 + スライドリンク + 翌日反応仮説 |
| 決算予告 | 朝レポート枠 (REPORT 部屋に Research セクション追加) | 毎朝 8:30 JST | 当日 / 翌営業日の決算予定 + 事前注目理由 |
| 重要銘柄速報 | Research / 決算専用部屋 | 重要度 5 (保有 + Research A1 等) | 即時通知 |
| 一般銘柄速報 | デイリーデックに集約 | 翌朝 8:30 JST | 一括要約 |
| 朝レポート | REPORT 部屋 | 毎朝 8:30 JST | 決算予告 + 直近決算分析サマリ |

★ **短期 ENTRY 部屋には決算速報を入れない** (F285 §10 修正 4 と整合)。

### 8.2 通知本文構成 (LINE Flex Message 想定)

```
[決算速報] トヨタ自動車 (7203) — Q4 FY2026
─────────────
📊 売上: 12,345 億円 (YoY +5.2%、会社予想 +1.5%、市場予想 +2.1%)
📈 営業利益: 3,456 億円 (YoY +12.3%、会社予想 +3.4%)
💰 配当: 増配可能性 40%、配当性向 35%
🔄 自社株買い: なし
📊 進捗率: 78.5% (Q4 終了時点、通期予想に対し)

⬆️ 翌営業日反応仮説: 上昇 (確度 60%)
🎯 Research Rank 影響: A2 → A1 候補
💼 保有判断 (参考): 保有継続を推奨

📎 スライド: [link]
📄 短信 PDF: [link]
─────────────
※ 投資判断は Fujiwara が最終確認してください。
```

## 9. FIRE 既存機能との接続 (HQ 確認項目 8)

| 接続先 | 接続内容 |
|---|---|
| **F285 Research Lane** | Output Layer として接続、Earnings Preview Agent §4-3 にフィードバック (Rank 変更可能性) |
| **Portfolio Engine** | 現 holding の決算サマリ + 増減提案 (Fujiwara 手動承認、自動発注禁止) |
| **Morning Report** | 朝レポート 8:30 JST に決算予告 + 直近決算分析サマリ追加 |
| **Watchlist Ranker (F285 §4-5)** | A1/A2 候補の決算スケジュールをカレンダー化、決算後の Rank 再評価 |
| **保有銘柄管理** | positions テーブル (F051 既) と連携、保有銘柄の決算予定を自動カレンダー化 |
| **LINE 通知 (F236 5 部屋)** | 6 部屋目 = Research 部屋を新設候補、または既存 REPORT に統合 (Phase F292 で HQ 承認) |

## 10. 制約 (HQ 確認項目 9)

1. **自動発注禁止** (R-03-01 / R-13-08): Fujiwara が最終確認、提案のみ
2. **楽天証券操作自動化禁止**: iSPEED / 楽天 Web 操作の自動化なし
3. **Computer Use 禁止**: スクショ・GUI 操作型自動化なし
4. **API / Plugin 利用は権限確認必須**: Claude API / SDK / Plugin の
   利用前に契約・権限を確認
5. **金融分析は Fujiwara レビュー前提**: 投資判断含む出力は手動レビュー必須
6. **LINE 通知は判断材料提示まで**: 「買え / 売れ」の指示はしない
7. **Stage 飛ばし禁止**: F287 → F288 → F289 → F290 → F291 → F292 を
   段階着手、各 Phase 受入合格まで次に進まない
8. **Evaluation Agent 提案制** (R-13-08): 分析 prompt / 出力 schema 等
   の調整は F119 提案 + Fujiwara 承認

## 11. Phase 分割案 (HQ 確認項目 10)

| Phase | 内容 | 目安期間 | 状態 |
|---|---|---|---|
| **F287** | 仕様設計 + feasibility 確認 (Claude API / Plugin / スライド方式) | 1 週間 | **本タスク完了で仕様 ✅、feasibility 一次完了** |
| F288 | 決算カレンダー DB / ダッシュボード実装 (earnings_calendar テーブル + CLI) | 1-2 週間 | 未着手 |
| F289 | 決算短信 / IR 資料取得パイプライン (TDnet PDF + 会社 IR PDF) | 2-3 週間 | 未着手 |
| F290 | AI 分析パイプライン (Claude API + 16 観点 + JSON 出力) | 2 週間 | 未着手 |
| F291 | スライド / PDF 生成 (Markdown → PDF / python-pptx) | 1-2 週間 | 未着手 |
| F292 | LINE 通知 / 朝レポート接続 (Research 専用部屋 or 既存 REPORT) | 1 週間 | 未着手 |

合計目安: 8-12 週間 (本タスク仕様 + feasibility + 実装 5 段階)。

## 12. 受入基準

### F287 受入 (本仕様)
- 仕様書 (本ドキュメント) 完成 ✅
- feasibility 確認 (§14) 完了
- F285 Output Layer としての位置付け明示
- 11 章 (Phase 分割) の段階着手計画明示

### F288 受入
- earnings_calendar テーブル作成、保有 / 監視 / Research / 主要 / セクター代表 /
  注目の 6 区分が登録可能、CLI で一覧 / 編集可能
- 当日 / 翌営業日 / 当週の決算予定が一覧表示できる

### F289 受入
- 当週決算銘柄の TDnet 短信 PDF / 説明資料 PDF が自動取得できる (>= 80%)
- 取得失敗銘柄は failed status として可視化、再試行可能

### F290 受入
- 取得済 PDF を Claude API に投入し、16 観点 JSON が生成される
- 主要数値 (売上 / 営業利益 / 純利益) の抽出精度 >= 95% (Fujiwara 検証)
- 翌営業日反応仮説が提示される

### F291 受入
- 1 銘柄 1 枚スライド + デイリーデックが生成される
- Markdown / PDF 形式で配信可能

### F292 受入
- 決算速報 / 朝レポート / Research 部屋の通知が稼働
- ENTRY 部屋に Research 通知が混入しない
- Fujiwara が LINE 通知の要約 + リンクで判断材料を確認できる

## 13. リスク

1. **TDnet PDF / IR PDF の取得安定性**: 各社 IR ページの URL パターンが
   個別、HTML 構造変更で fetch 失敗の可能性
2. **AI 分析の数値抽出精度**: PDF から数値抽出する際の OCR / レイアウト
   依存、precision 検証必要 (Fujiwara スポットチェック)
3. **Claude API コスト**: 16 観点 × 大量銘柄 × PDF 入力で月額コスト
   上昇 (Claude Sonnet 4.6 / Opus 4.7 利用想定、Phase F290 で実測)
4. **Plugin 連携の不確定性**: Claude.ai 専用 plugin (financial-services /
   PowerPoint 等) は Claude Code から直接呼び出せない可能性 → Claude API
   経由で代替実装する方針
5. **rate limit (TDnet / 各 IR ページ)**: 大量取得時の rate limit 配慮
   (sleep / robots.txt 確認)
6. **法令 / IR ガイドライン**: 公開 PDF のスクレイピング、個別企業の
   利用規約確認
7. **決算発表時刻の不確定性**: 適時開示は予定時刻通りでない場合あり
   (前倒し / 遅延)、polling 機構必要
8. **重要度判定の自動化**: holding_status × Research Rank × 注目度の
   組み合わせで重要度を自動算出するロジックの calibration 必要

## 14. feasibility 確認結果 (HQ 確認項目 4 + 5)

### 14.1 Claude financial-services / Financial Analysis plugin の利用可否

| 項目 | 現状認識 | 結論 |
|---|---|---|
| Claude Code (CLI) から Claude.ai 専用 plugin を呼ぶ | Claude.ai web UI 内の Skills / Plugins は API 経由で直接呼び出せないと推定 (Anthropic ドキュメントで要確認) | **本タスクは Claude API 経由で代替実装する方針**。Plugin 経由ではない |
| Claude API (Anthropic SDK) で財務分析 | PDF input サポート (Claude 3.5+) で決算短信 PDF を直接渡し分析可能、tool use で構造化出力 | **可能、推奨方式** |
| Claude Code Agent SDK | カスタム Agent 化可能、Mac mini 上で常駐 OK | **F290 で活用候補** |

### 14.2 Claude Code から利用可能な範囲

| 用途 | Claude Code で可能か | 実装手段 |
|---|---|---|
| ローカル PDF / Markdown 操作 | ✅ | Bash / Read / Edit / Write |
| TDnet PDF 自動取得 | ✅ | Python (requests / aiohttp) |
| Claude API 呼び出し | ✅ | Python (anthropic SDK) |
| 構造化 JSON 出力 | ✅ | Claude tool use (claude-sonnet-4-6 + JSON schema) |
| LINE Flex Message 送信 | ✅ | F236 既、line-bot-sdk |
| スライド生成 (Markdown) | ✅ | Claude API + テンプレート |
| スライド生成 (PDF) | ✅ | pandoc / weasyprint |
| スライド生成 (PPT) | ⚠ | python-pptx (要 Phase 検証) |
| Microsoft 365 Copilot 連携 | ⚠ | Mac mini 環境では要 Microsoft 365 個別契約、自動化は困難 |
| Claude for PowerPoint plugin | ⚠ | Claude.ai 専用、Claude Code から直接呼び出し不可 (推定) |

### 14.3 認証 / 権限 / 出力保存

| 要素 | 必要なもの |
|---|---|
| Claude API key | ANTHROPIC_API_KEY (要 Fujiwara 契約、Anthropic Console から発行) |
| J-Quants API key | JQUANTS_API_KEY (F100 既、Fujiwara 契約済 Standard + Add-ons) |
| TDnet | 公開、認証不要 |
| 会社 IR ページ | 公開、robots.txt 確認 |
| LINE Bot | LINE_CHANNEL_ACCESS_TOKEN (F236 既) |
| 出力保存 | ローカル staging DB + ファイルシステム (Mac mini) |

### 14.4 FIRE 自動パイプラインへの組込

**組込可能** (Claude API 経由で代替実装、Plugin 不要):

```
cron 8:30 JST 朝レポート
  ↓
fetch_earnings_calendar (decay 当日 / 翌営業日 / 当週)
  ↓
fetch_tdnet_pdf + fetch_company_ir_pdf
  ↓
analyze_with_claude_api (PDF input + 16 観点 prompt)
  ↓
generate_slide_markdown_pdf
  ↓
send_line_notification (Research / 朝レポート 部屋)
```

各ステップは独立した Python script として F288-F292 で実装。

### 14.5 半自動運用案 (Plugin が使えない場合)

**主案: Claude API 経由の自動パイプライン** で全自動化可能。

代替案 (Claude API でカバー困難な場合):
1. **手動ステップを介在**: 決算 PDF 取得は自動、AI 分析は Claude.ai web
   に Fujiwara が手動で投入、結果を所定フォーマットで FIRE に貼付け
2. **Claude.ai Skills 経由**: Fujiwara が Claude.ai web で財務 Skill
   実行、結果を Markdown export → FIRE 自動取込
3. **Hybrid**: 重要度 5 (保有 + Research A1) は手動、その他は自動

主案で十分な品質が出ない場合のみ代替案を検討 (Phase F290 で品質検証)。

### 14.6 feasibility 結論

★ **Claude API 経由で全自動パイプライン実装可能** ★
- Plugin 連携は不確定だが、API + SDK で代替実装する方針で進める
- スライドは Markdown → PDF が現実的、PPT は必要時のみ
- 半自動代替は重要度に応じた hybrid を検討

## 15. 要件 ID 表

| ID | 内容 | 関連章 |
|---|---|---|
| R-287-01 | F287 は F285 の Output Layer、自動発注機能ではない | §1 |
| R-287-02 | 決算カレンダー DB schema = 12 field + 主キー (code, expected_date, period_kind) | §3 |
| R-287-03 | 対象 Universe = 6 区分 (保有 / 監視 / Research A1A2 / 主要 / セクター代表 / 注目) | §4 |
| R-287-04 | 取得元 = TDnet (HTML/XBRL/PDF) + J-Quants + 会社 IR + 適時開示 | §5 |
| R-287-05 | AI 分析 16 観点 + 翌日反応 / Rank 影響 / 保有判断派生 | §6 |
| R-287-06 | スライド方式 = Markdown 主、PDF 副、PPT 必要時のみ (F291 Phase 分割) | §7 |
| R-287-07 | LINE 通知 = Research/決算専用部屋 + 朝レポート枠、ENTRY 部屋と分離 | §8 |
| R-287-08 | F285 / Portfolio / Morning / Watchlist / 保有 / LINE と接続 | §9 |
| R-287-09 | 自動発注禁止 / Computer Use 禁止 / 楽天証券操作禁止、Fujiwara 最終確認 | §10 |
| R-287-10 | Phase F287-F292 段階着手、各 Phase 受入合格まで次へ進まない | §11, §12 |
| R-287-11 | feasibility 結論 = Claude API 経由で全自動可、Plugin 不要 | §14 |

## 16. 関連 Vault ファイル / 改訂履歴

### 関連

- [[F285_Research_Lane_requirements_and_spec_2026-05-08|F285 Research Lane]] — 上流、F287 は Output Layer
- [[F101|F101 TDnet]] — 既存 TDnet HTML/XBRL 取得経路、本タスクで PDF 経路追加
- [[F100|F100 J-Quants V2]] — daily / fins データソース
- [[F119_Evaluation|F119 Evaluation Agent]] — 受入評価実施者
- [[F236|F236 LINE 5 部屋]] — 既存 LINE 部屋構成、F292 で 6 部屋目検討
- [[F210_phase_1b|F210 Phase 1B]] — 3000 万円 2 年目標
- [[F287_決算カレンダー_AI分析_スライド_LINE通知ダッシュボード|F287 TODO]]

### 改訂履歴

- v1.0 (2026-05-08): 初版、HQ 並行作業指示中の vault 化、F285 Research
  Lane Output Layer として位置付け、Phase F287-F292 分割、feasibility
  確認 (Claude API 主案、Plugin 副案)、自動発注禁止 + Fujiwara 最終
  確認の前提を全章で明示
