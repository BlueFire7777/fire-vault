---
id: F105
phase: P6: データソース
priority: 高 (Lane C 真実装の前提)
status: 起票案 (HQ 確認待ち)
owner: Fujiwara
depends_on: [F100 J-Quants V2 daily]
chapter: "14"
created: 2026-05-08
updated: 2026-05-08
related: F281_Lane_C_design_2026-05-08, F281_Lane_C_universe_precheck_2026-05-08
---

# F105: J-Quants 個別銘柄分足 Add-ons 接続検証 + staging 取込設計

## 1. 概要

J-Quants (公式 API) の Add-ons プラン経由で個別銘柄分足 / Tick データの
取得経路を確立する。F281 Lane C「前日強銘柄初押し戦略」の真実装に必要な
intraday data 取得を本タスクで検証 + 設計し、後続の Phase C1 (intraday
取込実装) で staging に取り込む。

★ 本タスクは Lane C 真実装の **前提タスク**。Phase C1 / C2 着手前に
   完了必須。

## 2. 背景 (HQ 修正情報、2026-05-08)

Mac mini 当初報告では「J-Quants V2 に個別銘柄 intraday API なし」と
されていたが、HQ 最新情報により以下が利用可能と確認:

- **2026-01-19**: J-Quants 公式に株価分足・株価ティックデータ追加
- Add-ons API:
  - `/v2/equities/bars/minute` (1 分足 OHLC + 出来高 + 売買代金、
    取得可能期間 過去 2 年間)
  - `/v2/equities/trades` (Tick データ)

→ market_data/historical.py の「J-Quants V2 対応 API なし」comment は
   古い、本タスクで最新公式仕様で再検証する。

## 3. 検証項目 (本タスクで確認)

### 3-1. 契約 / 権限

  - Add-ons プランの契約条件 (月額料金、利用規約、再配布制限)
  - 既存 J-Quants V2 plan からの追加契約手順
  - API token / 認証方式 (既存と共通か別か)

### 3-2. API 疎通

  - `/v2/equities/bars/minute` の actual response (任意 1 銘柄 1 日で)
  - rate limit (req/min、req/day)
  - 認証 / pagination / リトライ仕様
  - エラーコード (429 / 503 等) 検証

### 3-3. データ仕様

  - 取得可能期間 (過去 2 年で実 lookback 可能か)
  - フィールド (OHLC、volume、turnover_value、その他)
  - intraday VWAP / 累積 high / 累積 low が API 提供されるか / 自前計算か
  - 09:00 〜 15:00 の bar 数 (5 min: 67 / 15 min: 26 想定確認)
  - 10:00 break (もしあれば) や前場 / 後場 boundary

### 3-4. 保存設計

  - 保存元 = **1 分足** (HQ Q-Lane-C-C0-4 確定、第一候補)
  - 評価 MVP = 5 分足 (1 分足から派生集計)
  - 15 分足 = 比較用 / 軽量版 (1 分足から派生集計)
  - market_prices_intraday table 設計確定 (Lane C 設計記録 §3 参照)
  - migration 設計 (staging 専用、B-strict-2a 同 4 段 guard)

### 3-5. backfill 設計

  - 60 営業日分 1 分足 backfill のサイズ見積もり
    (4,500 銘柄 × 60 day × 360 min = 約 9,720 万 row、数 GB)
  - 全銘柄 / Lane C universe (Tier1) 1,592 銘柄に絞り込みか
  - rate limit 内での backfill 工数 (時間 / 並列度)

## 4. 制約 (HQ 厳守)

- production / develop DB に触れない
- staging 前提で設計
- 自動発注禁止 (本タスク無関係だが規律として)
- LINE_DRY_RUN=true 維持 (intraday fetch ジョブで通知発生する場合)
- Codex pre-commit 必須、--no-verify 禁止
- 個別 commit 厳守
- 楽天証券 API は本タスク対象外 (HQ Q-Lane-C-C0-3 で 4 位、後回し)

## 5. 完了基準

- J-Quants Add-ons (`/v2/equities/bars/minute`) 疎通確認 (1 銘柄 1 日
  で response 取得 + フィールド確認)
- 契約 / コスト / Fujiwara 判断結果 Vault 化
- staging market_prices_intraday table 設計確定 (Lane C 設計記録 §3
  との整合)
- migration script 設計案 + 1 銘柄 1 日 sample 取込 PoC
- backfill 工数見積もり (60 営業日 / Tier1 universe で X 時間)
- 結果 Vault 化: 03_design/F105_J-Quants_intraday_validation_YYYY-MM-DD.md

## 6. 後続タスク

- Phase C1: market_data/intraday.py 実装 + 60 営業日 backfill
- Phase C2: extract_lane_c_candidates (intraday rule-based) 実装
- Phase C3: Historical Paper Live 20 営業日

## 7. 関連リンク

- 要件書: 第 14 章 R-14-02 (data source)
- [[F281_Lane_C_design_2026-05-08|Lane C 設計記録]]
- [[F281_Lane_C_universe_precheck_2026-05-08|Lane C universe precheck 結果]]
- [[F100_市場データAPI|F100 J-Quants V2 daily]]
- [[F104_指数四本値取得|F104 指数 daily (別系列)]]
