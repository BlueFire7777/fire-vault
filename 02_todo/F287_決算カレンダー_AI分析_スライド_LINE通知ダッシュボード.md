---
id: F287
phase: P5: Research Output Layer (F285 下流)
priority: 高 (Phase F287 仕様完了、F288 着手前)
status: 仕様設計 + feasibility 確認完了 (v1.0)、F288 着手前
owner: Fujiwara
depends_on: [F285 Research Lane, F101 TDnet, F100 J-Quants V2, F119 Evaluation, F236 LINE 5 部屋, F051 positions]
chapter: "27"
created: 2026-05-08
updated: 2026-05-08
related: F287_Earnings_Dashboard_AI_Briefing_requirements_and_spec_2026-05-08, F285_Research_Lane_requirements_and_spec_2026-05-08, F101, F236, F119
---

★ **F287 決算カレンダー × 決算短信 AI 分析 × スライド/PDF 生成 ×
   LINE 通知ダッシュボード**: HQ 並行作業指示 (2026-05-08、F284/F105
   c6 backfill PID 92822 走行中の vault 作業) で仕様 + feasibility
   確認を vault 化。F285 Research Lane の Output Layer として位置付け。

# F287 決算ダッシュボード + AI 分析 + LINE 通知 (Research Output Layer)

## サマリ

決算カレンダー DB + 決算短信 / IR PDF 取得 + Claude API 経由 AI 分析
(16 観点) + スライド/PDF 生成 + LINE 通知の 6 段 Phase で実装。F285
Research Lane の Output Layer として、保有/監視/A1A2/主要/セクター
代表/注目銘柄の決算予定を一元管理し、Fujiwara の決算後判断を支援。

★ 自動発注機能ではない。投資判断は Fujiwara が最終確認、LINE 通知は
判断材料提示まで (R-03-01 / R-13-08 準拠)。

## F287 が対象とすること

- **決算カレンダー DB**: 6 区分 universe (保有 / 監視 / Research A1A2 /
  主要大型 / セクター代表 / 注目) の決算予定を一元管理
- **決算短信 / IR 資料の自動取得**: TDnet (HTML/XBRL/PDF) + J-Quants +
  会社 IR ページから PDF 取得
- **AI 分析パイプライン**: Claude API (claude-sonnet-4-6 / claude-opus-4-7)
  経由で 16 分析観点を構造化 JSON 出力
- **スライド/PDF 生成**: 1 銘柄 1 枚速報スライド + デイリーデック +
  Watchlist デック (Markdown 主 / PDF 副 / PPT 必要時のみ)
- **LINE 通知**: 決算速報 / 朝レポート / Research 専用部屋に要約 +
  リンク配信 (短期 ENTRY 部屋とは分離)
- **FIRE 既存接続**: F285 Research Lane (Output Layer) / Portfolio
  Engine / Morning Report / Watchlist Ranker / 保有銘柄管理 / LINE 通知

## 詳細仕様

[[F287_Earnings_Dashboard_AI_Briefing_requirements_and_spec_2026-05-08|F287 仕様書 v1.0]]
を参照 (16 章 + メタ、要件 ID R-287-01〜11)。

## Phase 着手順

| Phase | 内容 | 目安期間 | 状態 |
|---|---|---|---|
| **F287** | 仕様設計 + feasibility 確認 | 1 週間 | **仕様 ✅ + feasibility 一次完了** |
| F288 | 決算カレンダー DB / ダッシュボード実装 | 1-2 週間 | 未着手 |
| F289 | 決算短信 / IR 資料取得パイプライン | 2-3 週間 | 未着手 |
| F290 | AI 分析パイプライン (Claude API + 16 観点) | 2 週間 | 未着手 |
| F291 | スライド / PDF 生成 (Markdown / PDF / PPT) | 1-2 週間 | 未着手 |
| F292 | LINE 通知 / 朝レポート接続 | 1 週間 | 未着手 |

合計目安: 8-12 週間 (本タスク仕様 + 実装 5 段階)。

## feasibility 確認 結論 (詳細は仕様書 §14)

★ **Claude API 経由で全自動パイプライン実装可能** ★

- Claude.ai 専用 Plugin (financial-services / PowerPoint) は Claude
  Code (CLI) から直接呼び出し不可と推定 → **Claude API 経由で代替**
- PDF input サポート (Claude 3.5+) で決算短信 PDF を直接渡し分析可能
- tool use で構造化 JSON 出力、16 観点を一括抽出
- スライドは Markdown → PDF が現実的、PPT は必要時のみ
- 半自動代替は重要度に応じた hybrid (重要度 5 = 保有 + Research A1
  のみ手動レビュー必須) も検討候補

必要な追加契約 (Fujiwara 判断):
- ANTHROPIC_API_KEY (Claude API)、月額コスト要 Phase F290 で実測
- J-Quants は既契約 (Standard + Add-ons ¥8,800/月)
- TDnet / 会社 IR ページは公開、認証不要

## 受入基準 (詳細は仕様書 §12)

### F287 受入 (本タスク)
- 仕様書 (本タスクで vault 化済) ✅
- feasibility 確認 ✅
- F285 Output Layer としての位置付け明示 ✅

### F288-F292 受入
- F288: earnings_calendar テーブル + CLI 一覧
- F289: 当週決算銘柄 PDF 取得率 >= 80%
- F290: 主要数値抽出精度 >= 95% (Fujiwara 検証) + 翌日反応仮説生成
- F291: 1 銘柄スライド + デイリーデック生成
- F292: 決算速報 / 朝レポート / Research 部屋通知稼働、ENTRY 部屋に
  Research 通知が混入しない

## 制約 (R-03-01 / R-13-08 / FIRE 全体ルール準拠)

- **自動発注禁止**、楽天証券操作自動化禁止、Computer Use 禁止
- **金融分析は Fujiwara レビュー前提**
- **LINE 通知は判断材料提示まで** (買え/売れの指示はしない)
- **API / Plugin 利用は権限確認必須**
- **Stage 飛ばし禁止** (F287 → F288 → ... → F292 を段階着手)
- **Evaluation Agent 提案制** (R-13-08): 分析 prompt / 出力 schema 等
  の調整は F119 提案 + Fujiwara 承認

## 後続タスク候補

| タスク ID | 内容 | 後続条件 |
|---|---|---|
| F288 | earnings_calendar DB + CLI | F287 完了 (本タスク) |
| F289 | TDnet PDF / 会社 IR PDF 取得 | F288 完了 |
| F290 | Claude API による 16 観点分析 | F289 完了 + ANTHROPIC_API_KEY 取得 |
| F291 | スライド / PDF 生成 | F290 完了 |
| F292 | LINE 通知接続 (Research 専用 / 朝レポート) | F291 完了 + LINE 部屋設計 (F236 + 6 部屋目検討) |

## 関連リンク

- [[F287_Earnings_Dashboard_AI_Briefing_requirements_and_spec_2026-05-08|F287 仕様書 v1.0]]
- [[F285_Research_Lane_requirements_and_spec_2026-05-08|F285 Research Lane (上流)]]
- [[F101|F101 TDnet]] (PDF 取得経路、F289 で拡張)
- [[F236|F236 LINE 5 部屋]] (F292 で 6 部屋目検討)
- [[F119_Evaluation|F119 Evaluation]] (分析品質の受入評価)
- [[F210_phase_1b|F210 Phase 1B]] (3000 万円 2 年目標)
