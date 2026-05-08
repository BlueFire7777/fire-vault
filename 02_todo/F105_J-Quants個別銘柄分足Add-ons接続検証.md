---
id: F105
phase: P6: データソース
priority: 高 (Lane C 真実装の前提、c6 完了で Phase C2 着手可能)
status: Phase C1 完了 (c6 full backfill exit 0、Phase C2 PASS 5 条件達成、Phase C2 着手可能)
owner: Fujiwara
depends_on: [F100 J-Quants V2 daily]
chapter: "14"
created: 2026-05-08
updated: 2026-05-09
related: F281_Lane_C_design_2026-05-08, F281_Lane_C_universe_precheck_2026-05-08, F105_J-Quants_intraday_validation_2026-05-08, F105_Phase_C1_smoke_2026-05-08, F284_F105_c6_final_result_2026-05-09
---

★ **契約後疎通完了** (2026-05-08): Fujiwara が J-Quants Standard
   ¥3,300 + Add-ons ¥5,500 = **月額 ¥8,800** 契約済。Mac mini 最小
   疎通再テスト完了。詳細結果は [[F105_J-Quants_intraday_validation_2026-05-08|F105
   検証結果 Vault]] §15-16 参照。

  契約後疎通結果 (v1.1):
  - `/v2/equities/bars/minute` 1 銘柄 1 日: **HTTP 200 ✅**
  - response field 確定: Date / Time / Code / O / H / L / C / Vo / Va
  - 1 日 327 bar (前場 150 + 11:30 板寄せ 1 + 後場連続 175 +
    15:30 大引け板寄せ 1)、CA 期間 15:25-15:29 は bar 生成なし
  - 流動性銘柄 (7203) は 1 分連続生成、Vo==0 bar なし
  - pagination_key 不在 (1 銘柄 1 日)
  - rate limit Standard プラン = 120 req/min
  - `/v2/equities/trades` は HTTP 403 path 不存在 (Lane C MVP 範囲外)

  次: **Phase C1 着手可** (HQ GO 判断待ち)。Standard 120 req/min で
       backfill 1.7-3.3 h 想定。実装計画は検証 Vault §16 で確定。

★ **Phase C1 c6 完了** (2026-05-09 02:04 JST): Tier2 universe 478 銘柄 ×
   64 営業日 (2026-02-03 〜 2026-05-01) backfill が **正常完了**。

  c6 最終結果 (詳細は [[F284_F105_c6_final_result_2026-05-09|c6 final
  result Vault]] 参照):
  - exit code 0 (PID 92822、走行 11h54m)
  - fetched 25,448 / skipped_resume 3,231 / no_data 1,913
  - recoverable_errors 0 / rate_limit_429 0 / unresolved_errors 0
  - FATAL / Traceback / Auth / Validation / SQLite / disk error 0
  - 4 条件完了 pair 28,681 (universe 内 28,679 = 478 × 60 - 1 完全一致)
  - DB 3,879.8 MB (+2,503 MB 増分)
  - disk 838 GB available

  差分内訳:
  - +1 件: 149A0 / 2026-03-11 銘柄固有 no_data (合理的説明可能)
  - +2 件: 72030 (Toyota smoke 残) × 04-30/05-01 (Tier2 外、Phase C2
    で universe filter 除外)

  **Phase C2 PASS 5 条件すべて達成、Phase C2 着手可能 ✅** (HQ 承認済)

  次: Phase C2 (Lane C 真実装) 着手判断 (HQ 承認後)、評価対象 universe
       = Tier2 478 codes 固定、smoke 残 72030 除外、staging 専用



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
