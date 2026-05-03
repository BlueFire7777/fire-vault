---
id: F100
phase: P6: データソース
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F009]
chapter: "14"
created: 2026-04-24
updated: 2026-05-01
tags: [P6, market_data, jquants, data_source]
---

# F100: 市場データ API 連携 (J-Quants V2)

## 概要

J-Quants V2 API でデータ取得・SQLite 保存の骨格を実装。3 エンドポイント
(銘柄一覧 / 株価四本値 / 財務サマリー)。データ流入が始まり、
Pattern Research Agent + Simulation 基盤 (F040-F053) が実データで稼働可能に。

要件根拠: 第 14 章 R-14-02 (優先度 2 位: 市場データ API)

## 実装内容

### 主要モジュール

- `~/fire/market_data/` パッケージ新規作成 (7 ファイル)
  - `client.py`: `JQuantsClient` (V2 x-api-key 認証、リトライ機構付き)
  - `models.py`: `ListedInfo` / `DailyPrice` / `FinSummary` dataclass
  - `repository.py`: SQLite UPSERT 保存 + 一覧取得
  - `fetcher.py`: 高レベル統合 API (`fetch_and_save_*`) + V2 フィールドパーサ
  - `cli.py`: `--fetch-listings / --fetch-prices / --fetch-financials`
- `scripts/setup/migrate_market_data.py`:
  `market_listings` / `market_prices_daily` / `market_financials` 新規

### V2 API 実装メモ (重要)

実装中に J-Quants V2 公式 spec と spec ドキュメントの記載が異なる箇所が
あったため、実 API レスポンスを元に修正:

| 観点 | 当初想定 | V2 実態 |
|---|---|---|
| 銘柄一覧 path | `/listed/info` | **`/equities/master`** |
| 銘柄一覧 top-key | `info` | **`data`** |
| 株価四本値 top-key | `daily_quotes` | **`data`** |
| 株価四本値 フィールド | `Open / High / Low / Close` | **`O / H / L / C`** (短縮形) |
| 出来高 / 売買代金 | `Volume / TurnoverValue` | **`Vo / Va`** |
| 銘柄名 / セクター | `CompanyName / Sector17Code` | **`CoName / S17`** (短縮形) |
| 財務 path | `/fins/statements` | **`/fins/details`** (V2) |
| 銘柄コード | 4 桁 (`8697`) | **5 桁 (`86970`)** に正規化 |

`fetcher.py` の `_parse_listing` / `_parse_daily_price` で V2 短縮フィールドを
モデルにマッピング。

### Standard プラン制限

`/fins/details` は **Premium プラン専用** (Standard プランでは 403 返却)。
`get_fins_statements()` は呼び出すと `JQuantsAPIError` で 403 メッセージ
("This API is not available on your subscription") を含む。Premium 加入後に
利用可能になる予定。

### CLI 動作確認 (実 API)

```bash
$ python -m market_data --fetch-listings
銘柄一覧 保存件数: 4449

$ python -m market_data --fetch-prices --code 8697 --date 20260428
株価四本値 保存件数: 1

$ python -m market_data --fetch-prices --code 8697 \
    --date-from 20260401 --date-to 20260430
株価四本値 保存件数: 21
```

DB 確認:
```sql
SELECT code, company_name, sector_17_name, market_name
FROM market_listings WHERE code IN ('86970', '72030');
-- 72030 | トヨタ自動車        | 自動車・輸送機     | プライム
-- 86970 | 日本取引所グループ | 金融（除く銀行）| プライム

SELECT date, code, open, high, low, close, volume
FROM market_prices_daily WHERE code='86970' ORDER BY date DESC LIMIT 3;
-- 2026-04-30 | 86970 | 1912.0 | 1925.5 | 1839.5 | 1863.5 | 5193700.0
```

## テスト

- `tests/market_data/test_jquants_client.py`: **20 ケース全 PASS**
  - 認証 (3) / HTTP 通信 mock (5) / get_listed_info (3) /
    get_daily_prices (2) / get_fins_statements (1) /
    fetcher 統合 (3) / repository (3)
- 全テスト累計: 449 → **469 PASS** (+20)

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| market_data パッケージ全 7 ファイル | ✅ |
| migrate_market_data: 3 テーブル | ✅ |
| 20 ケース全 PASS | ✅ |
| F040-F053 既存 449 PASS 非破壊 | ✅ |
| 累計 469 PASS | ✅ |
| CLI smoke (実 API、4449 銘柄 + 21 日分) | ✅ |
| .env.example に JQUANTS_API_KEY 追加 | ✅ |
| .env は gitignore 済 | ✅ |

## 24 時間稼働への寄与

F100 完了で「シミュレーション主導戦略」が実行可能に:

1. ✅ パターン発掘 (Pattern Research Agent + 過去データ流入)
2. ✅ シミュレーション検証 (F040-F050 + 過去データ)
3. ❌ tick.py の中身実装 (extract_candidates / monitor_virtual_positions)
4. ❌ リアル稼働の停止/クローズ機能検証 (1-2 週間)
5. ❌ F053 全項目 PASS → Semi Auto 昇格

## 関連リンク

- 要件書: [[FIRE_要件書_第14章_データソース優先順位と縮退運転]]
- 要件 ID: R-14-02 (優先度 2 位: 市場データ API)
- 前タスク: [[F053_Semi_Auto昇格基準]]
- 次タスク候補: F101 (TDnet/IR API), F236 (LINE 5 段階アラート), tick.py 中身実装
- コード: `~/fire/market_data/`
- 公式 spec: https://jpx-jquants.com/spec/

## スコープ外メモ

- 指数四本値 (/indices)、信用残高、空売り残高 — 必要になったら追加
- 取引カレンダー — 必要になったら追加
- リアルタイムストリーミング — J-Quants は基本バッチ API
- 財務 (`/fins/details`) — Premium プランで利用可能 (Standard では 403)
- 大量取得時の chunked 取得 / cursor pagination — 現状は 1 回の GET で完結
