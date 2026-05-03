---
id: F100-historical
phase: P9: Stage 3 移行
priority: 最優先
status: 完了
owner: Fujiwara
depends_on: [F100, F230]
chapter: "14"
created: 2026-05-03
updated: 2026-05-03
---

# F100 拡張: 過去データ取得バッチ (J-Quants V2 過去 6 ヶ月〜1 年分)

## 概要

F230 (Paper Live バッチ実行) が実データで動く前提を整える。J-Quants V2
`/equities/bars/daily` から過去 6 ヶ月〜1 年分の日足を一括取得し、
market_prices_daily に投入する。約 4,449 銘柄 × 120 営業日 = 約 53 万行を
想定。

## スコープ

- 日足データ (market_prices_daily) の過去 6 ヶ月〜1 年取得
- レート制限は既存 client (429 リトライ + バックオフ) に依存
- 重複排除は既存 `save_daily_prices` の UPSERT 機能を活用
- 個別銘柄失敗で続行 (failed_symbols に蓄積)
- 進捗表示 (50 銘柄ごと、--symbol-limit でテスト用上限指定可)

## 重要な決定事項 (仕様書差分)

- **`JQuantsClient.get_daily_prices(code, date_from, date_to)`** が既存 API
  (仕様書 `fetch_daily_quotes` は誤り)
- **`save_daily_prices(prices, db_path)`** は **UPSERT** で実装済 (PK 衝突
  時は REPLACE)。仕様書の自前 INSERT OR IGNORE は不要、既存ヘルパーを流用。
- **`_parse_daily_price(raw, fetched_at)`** で V2 `O/H/L/C/Vo/Va/Adj*`
  短縮形を `DailyPrice` に変換 (仕様書の自前 INSERT は不要)
- **分足取得 API は F100 client に存在しない**: J-Quants V2 公式に
  分足取得 API が無いため、Phase 1 では daily のみ実装。minute は将来
  Phase 2 で別ソース検討。
- **date 形式は YYYYMMDD** (J-Quants 仕様)、内部の `DateClass.strftime("%Y%m%d")`
  で変換

## 成果物

### 新規ファイル

- `market_data/historical.py`:
  - `HistoricalFetchResult` dataclass (target / from_date / to_date /
    n_symbols / n_rows_inserted / failed_symbols / started_at /
    completed_at)
  - `get_target_symbols(db_path, limit)` - market_listings から取得対象を
    返す
  - `HistoricalDataFetcher.fetch_daily(from_date, to_date, symbols=None)`
    - 全銘柄一括取得、個別失敗続行
- `scripts/jobs/fetch_historical_market_data.py`:
  - CLI: `--from` / `--to` / `--symbols` / `--symbol-limit` / `--target`
- `tests/market_data/test_historical.py`: 14 ケース

### テスト

- 新規 14 件
  - get_target_symbols: 4 件 (all / limit / empty / sorted)
  - HistoricalFetchResult: 1 件 (defaults)
  - fetch_daily: 4 件 (insert / per-symbol failure / empty / listings 自動取得)
  - 重複排除 (UPSERT): 2 件 (idempotent / close 値更新)
  - パース耐性: 2 件 (unparseable skip / empty rows)
  - 進捗ログ: 1 件 (smoke)
- 累計: 923 → **937 PASS** (+14)
- 既存 923 への影響: **0 件** (新規ファイルのみ)

### Smoke 結果 (mock 3 銘柄 × 3 営業日 → 実 DB)

```
=== fetch_daily 結果 ===
  target: daily
  期間: 2026-04-28 〜 2026-05-01
  銘柄数: 3
  挿入: 9 行
  失敗: 0
  client.get_daily_prices 呼出: 3 回

=== 実 DB 投入後 ===
  market_prices_daily: 30 件 (delta=9)
  6758: 3 件 / 7203: 3 件 / 9984: 3 件
```

## CLI 使い方

```bash
# テスト: 3 銘柄 × 3 営業日
python scripts/jobs/fetch_historical_market_data.py \
    --from 2026-04-28 --to 2026-04-30 \
    --symbols 7203,9984,6758

# 本番: 過去 6 ヶ月分の日足を全銘柄
python scripts/jobs/fetch_historical_market_data.py \
    --from 2025-11-01 --to 2026-05-02

# テスト用: 最初の 100 銘柄のみ
python scripts/jobs/fetch_historical_market_data.py \
    --from 2026-04-01 --to 2026-05-02 --symbol-limit 100
```

## Stage 3 移行への最短パス

1. **F100 historical** (本タスク完了): 過去 6 ヶ月分 一括取得
2. **F230 batch_replay**: 直近 20 営業日 Paper Live バッチ実行
3. **F053 evaluate_promotion**: 自動判定 (5 項目)
4. **PASS** → F119 評価提案書 → 承認 → **Stage 3 開始** ★

リアルタイム待ち 4 週間 → 過去データバッチで数時間に短縮。

## スコープ外 (将来 / 別タスク)

- 並列化 (Phase 2、ProcessPoolExecutor で銘柄並列)
- 全銘柄分足取得 (データ量過大、対象銘柄絞り)
- TDnet HTML 過去取得 (F101 別タスク)
- announcements 過去取得 (F101 既存 sync ジョブで対応)
- リアルタイム F100 の改修 (既存で動く)

## 実本番実行時の注意

- **API キー必須**: `JQUANTS_API_KEY` を `.env` に設定 (F100 と共有)
- **実行時間想定**: 4,449 銘柄 × 6 ヶ月 = 30-120 分 (レート制限による)
- **進捗確認**: 50 銘柄ごとに INFO ログ出力
- **中断対応**: Ctrl+C で中断、既に取得済の行は UPSERT で保持される (再実行
  で続きから可能、ただし進捗状態の永続化は無いので全銘柄を最初から再走査)

## 関連リンク

- 要件書: 第 14 章 R-14-04 / R-14-02 (J-Quants V2 取得)
- 関連: [[F100_市場データAPI]] (基盤、認証共有) /
  [[F230_Paper_Live_batch_replay]] (本タスクで動作可能に) /
  [[F053_Paper_Live_昇格基準実装]] (バッチ後の判定)
- コード: `market_data/historical.py`,
  `scripts/jobs/fetch_historical_market_data.py`
