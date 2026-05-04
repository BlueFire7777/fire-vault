---
id: F268
phase: P9: 運用・保守
priority: 中
status: 未着手
owner: Fujiwara
depends_on: [F101, F267]
chapter: "20,32"
created: 2026-05-04
updated: 2026-05-04
---

# [F268] announcements 過去 6 ヶ月遡及取得

## 基本情報

- フェーズ: P9: 運用・保守
- カテゴリ: Data ingestion
- 優先度: 中
- 依存: F101, F267
- 担当: Fujiwara (Claude 補助)
- 想定工数: 0.5 日
- ステータス: 未着手
- 着手日: -
- 完了日: -

## タスク詳細

events=0 切り分けで判明した **副因 B**: announcements テーブルが
**7 件 / 2026-04-30〜05-01 の 2 日分のみ** しかなく、material_initial 系
パターンの実マッチが極端に薄い。

F267 で features は埋まったが、material が紐づく events を増やすには
announcements の遡及取得が必要。直近 6 ヶ月の TDnet HTML / J-Quants
`/fins/announcement` を遡及取得して `announcements` テーブルに格納する。

主要な検討事項:
- TDnet HTML スクレイピング: 既存 F101 の取得経路を遡及拡張可能か確認
- J-Quants `/fins/announcement` API: 過去取得制限 (利用プランの制約) 確認
- material_type の網羅性: earnings_upside / guidance_upside / buyback /
  dividend_increase / order / alliance / policy_benefit / theme_flow の
  8 種すべてが過去 6 ヶ月で揃うか
- decay / confidence の付与ルール (TDnet 0.85〜0.95、JQuants 0.5)

## 成果物

- `scripts/jobs/fetch_announcements_historical.py` 新規 (想定、F101 と
  ファイル名衝突回避)
- `data/historical/announcements_2025-11_2026-04.parquet` (中間ファイル、
  または直接 SQLite 投入)
- `tests/scripts/jobs/test_fetch_announcements_historical.py` (想定)
- 取得結果サマリ (announcements 行数 / 期間 / material_type 分布)

## 関連ドキュメント

- 設計書: [[03_design/F032_F054_diagnosis_2026-05-04|F032/F054 診断レポート]]
  (副因 B セクション)
- 設計書: [[03_design/F267_implementation_2026-05-04|F267 実装レポート]]
  (セクション 6 で言及)
- 関連タスク: [[F267_features_pipeline|F267]] (前提、features 整備)
- 関連タスク: [[F101_TDnet_announcement_ingest]] (本タスクの取得経路源、
  リンク切れ可)
- 関連タスク: [[F266_Stage3_最終ゲート|F266]] (本タスクは F266 通過後の
  運用品質改善)

## 進捗チェックリスト

- [ ] F267 完了確認 (Run a で events>0 確認後に着手判断)
- [ ] J-Quants `/fins/announcement` 過去取得制限の確認
- [ ] TDnet HTML 過去取得経路の調査 (R-?-? 既存仕様を確認)
- [ ] 取得バッチスクリプト実装
- [ ] 過去 6 ヶ月分一括取得 (announcements 推定 数千件)
- [ ] material_type 分布の偏り確認 (earnings_upside 偏重など)
- [ ] decay / confidence 値の妥当性確認

## 作業ログ

- **2026-05-04 朝**: F032/F054 診断レポートで副因 B として確定、起票
- F267 完了 (Run a events>0 確認) 待ち

## 完了条件

過去 6 ヶ月分の announcements が取得され、material_initial 系パターンの実
マッチが Run b (60 営業日) / Run c (120 営業日) で増加することを確認。

events 質 (material 起因の比率) が改善し、F053 / F241 評価で expected_value
が closed trades 由来の値で算出可能になる状態。
