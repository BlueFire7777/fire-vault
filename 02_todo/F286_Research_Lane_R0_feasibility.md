---
id: F286
phase: P5: Research Lane R0 (data feasibility precheck)
priority: 高 (R0 完了、R1 着手判断待ち)
status: R0 precheck 完了、MVP 即着手範囲特定済 (Premium 不要)、HQ 判断要請中
owner: Fujiwara
depends_on: [F100 J-Quants V2 daily, F101 TDnet, F276 indices, F285 Research Lane 仕様 v1.1]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_Research_Lane_R0_feasibility_2026-05-09, F285_Research_Lane_requirements_and_spec_2026-05-08, F285_Research_Lane中長期銘柄発掘エンジン
---

★ **F286 Research Lane R0 feasibility precheck**: F285 で定義した
   Research Lane の必要データを実機検証。**重要発見**: /v2/fins/summary
   が Premium 不要で BPS/EPS/Eq/Div* を網羅、MVP 範囲が想定より広い。

# F286 Research Lane R0 feasibility

## サマリ

J-Quants Standard で 4 endpoint precheck + 既存 staging DB schema 確認:

- ✅ /v2/fins/summary (107 fields、Cyclical Value/Growth/Dividend 派生 OK)
- ❌ /v2/fins/details (Premium 必要、ただし MVP 不要)
- ❌ /v2/fins/dividend (Premium 必要、ただし MVP 不要)
- ✅ /v2/equities/earnings-calendar (Earnings Preview 利用可)

既存 staging DB:
- market_listings (4,449)、market_prices_daily (526,764)、
  features (1,131,331)、announcements (7) 等
- market_financials は schema あるが row_count=0 (R1 で fetch 必要)

★ MVP 範囲: F285 5 Agent + Watchlist Ranker すべて Premium 不要で実装可能。

## 詳細

[[F286_Research_Lane_R0_feasibility_2026-05-09|F286 feasibility report v1.0]]
を参照 (14 章、precheck 結果詳細 + Agent 別取得可否 + Phase R1+ 推奨)。

## R0 完了内容

- 4 endpoint 実機 precheck (5 銘柄 × 4 = 計 23 calls)
- 既存 staging DB schema 確認 (read-only)
- Research Lane Agent 別マッピング 6 種
- MVP 推奨範囲 + 不足データ + 代替案
- Phase R1〜R6 提案

## 次工程候補 (HQ 判断要請)

優先度高 (R1 即着手):
- **F286 R1-1**: Sector Flow Agent MVP 実装 (既存 DB のみ)
- **F286 R1-2**: /fins/summary backfill + market_financials 充填

優先度中 (R2 範囲):
- F286 R2-1: Cyclical Value Screener (PBR/PER/ROE 派生計算)
- F286 R2-2: Growth Screener (4 期 trend + revision 判定)
- F286 R2-3: Dividend Growth Agent (連続増配 + 配当性向)

優先度後 (R3+):
- F286 R3: Earnings Preview Agent + earnings-calendar fetch
- F286 R3: Watchlist Ranker
- F286 R4: 朝レポート / LINE / Portfolio / Lane C 接続

並行候補:
- F287 (決算ダッシュボード) と F286 R3 が earnings-calendar で共通、
  並行進行可否は HQ 判断

## HQ 判断ポイント (詳細は仕様書 §12)

- Q1: F286 R1 即着手判断 (Sector Flow から)
- Q2: F285 §11 Phase 分割の見直し (R1 で 5 Agent 一括着手も可)
- Q3: R1 着手順序選好
- Q4: F287 並行着手の可否
- Q5: Premium 課金判断 (現時点 不要、将来 R5 評価で再検討)

## 制約 (R0 範囲、R1+ で継承想定)

- 自動発注 / 楽天証券操作 / Computer Use 禁止
- DB migration / 本格 ETL は R0 範囲外
- production / develop DB 無触
- staging DB read-only
- 外部スクレイピング禁止
- Premium 課金は Fujiwara 判断必須

## 関連 commit

~/fire 側:
- **e718f1a** chore(F286): add Research Lane R0 precheck script

vault 側:
- (本 commit) docs(F286): update Research Lane R0 TODO

## 関連リンク

- [[F286_Research_Lane_R0_feasibility_2026-05-09|F286 feasibility report v1.0]]
- [[F285_Research_Lane_requirements_and_spec_2026-05-08|F285 仕様 v1.1]] (上流)
- [[F285_Research_Lane中長期銘柄発掘エンジン|F285 TODO]]
- [[F284_F105_c6_final_result_2026-05-09|F284/F105 c6 完了]] (前提 data infrastructure)
- [[F101|F101 TDnet]] (announcements、Research Lane 補完)
- [[F100|F100 J-Quants V2]] (上位)
