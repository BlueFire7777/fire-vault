---
title: F105 J-Quants 個別銘柄分足 Add-ons 接続検証 + staging 取込設計 結果
date: 2026-05-08
phase: F105
status: 検証完了 (現契約では API 利用不可、Fujiwara 契約判断待ち)
related: F281_Lane_C_design_2026-05-08, F281_Lane_C_universe_precheck_2026-05-08, F100_市場データAPI
trigger: HQ Lane C Phase C1 前提タスク (HQ Q-Lane-C-C0-2 確定)
---

# F105 J-Quants 個別銘柄分足 Add-ons 接続検証 + staging 取込設計

## 1. 検証実施日 / 範囲

  - 実施日: 2026-05-08
  - Mac mini 実施範囲 (HQ 制約):
    - 公式料金プラン / API 仕様 docs 確認 (WebFetch)
    - 現契約 + 既存 API key で **最小件数の API 疎通テスト** (1 銘柄 1 日)
    - staging table 設計確定
    - Tier2 universe 60 営業日 backfill 工数見積もり
  - Mac mini 不実施 (Fujiwara 判断必須):
    - 契約変更 / 有料 Add-ons 申込
    - 大量データの実取得

## 2. J-Quants 公式仕様確認結果

### 2-1. 料金プラン (jpx-jquants.com 公式、2026-05-08 確認)

  | プラン | 月額 (税込) | API コール制限 | 取得可能期間 |
  |---|--:|---|---|
  | Free | ¥0 | 5 件/分 | 2 年間 (直近 12 週除く) |
  | Light | ¥1,650 | 60 件/分 | 5 年間 |
  | Standard | ¥3,300 | 120 件/分 | 10 年間 |
  | Premium | ¥16,500 | 500 件/分 | 20 年間 |

### 2-2. Add-ons プラン

  | Add-on | 月額 (税込) | 適用可能プラン |
  |---|--:|---|
  | **株価 分足・ティック (2 年間)** | **¥5,500** | Light / Standard / Premium |

  ★ Free プランには Add-on 追加不可。Lane C 真実装には **Light + Add-ons 月額 ¥7,150** が最小コスト。

### 2-3. API 仕様 docs 状況

  - jpx.gitbook.io/j-quants-ja: 分足 / ティック API endpoint 個別仕様の
    docs **未公開** (確認結果)
  - 公式サイト記載のみ: 「1 分単位 OHLC、出来高、売買代金、過去 2 年間」
  - response field 詳細は契約後の実 response で確認必要
  - rate limit / pagination / retry / 429 / 503 仕様: 既存 daily と同等
    と推定 (契約後確認)

## 3. 現契約での API 疎通テスト結果

  実行: `requests.get` で最小 1 銘柄 (7203 / Toyota) × 1 日 (2026-05-01)

  | endpoint | HTTP status | 備考 |
  |---|--:|---|
  | `/v2/equities/bars/daily` (既存) | **200** | 現契約有効 ✅ response 正常取得 (O/H/L/C/Vo/Va 等) |
  | `/v2/equities/bars/minute` (Add-ons) | **403** | "This API is not available on your subscription" |
  | `/v2/equities/trades` (Add-ons) | **403** | "The requested endpoint does not exist" |

  ★ 結論:
    - 現契約 (J-Quants V2 daily) は有効、API key 正常稼働
    - **Add-ons (分足・ティック) 契約必要、現契約では使えない**
    - `/v2/equities/trades` の path は契約後 / docs 確認後に確定

## 4. 現在権限での API 利用可否

  ★ **利用不可** ★

  - daily / 既存 endpoint は利用可
  - 分足 / ティック Add-ons 契約済の場合のみ利用可
  - **Fujiwara 判断: Add-ons 契約 (月額 ¥5,500) の決裁必要**

  Mac mini は契約操作しない (HQ 制約厳守)。

## 5. response field 一覧 (推定、契約後に実 response で確定要)

  既存 `/v2/equities/bars/daily` response field 命名規則を踏襲した推定:

  | field | 型 | 備考 |
  |---|---|---|
  | DateTime / Date+Time | string (ISO) | 1 分 bar 開始時刻 |
  | Code | string | 銘柄コード (5 桁、daily と同型) |
  | O / Open | float | bar 開始価格 |
  | H / High | float | bar 内高値 |
  | L / Low | float | bar 内安値 |
  | C / Close | float | bar 終値 |
  | Vo / Volume | float | bar 内出来高 |
  | Va / TurnOverValue | float | bar 内売買代金 |
  | (UL / LL / AdjFactor 系) | optional | daily と同型なら存在の可能性 |

  ★ 上記は daily の命名規則からの推定、契約後の実 response で field
    名 + 型を確定する。

## 6. rate limit / pagination / retry 方針 (推定)

  - rate limit: プラン別共通 (Light=60、Standard=120、Premium=500
    req/min)、daily と同等と推定
  - 既存 market_data/client.py の retry 経路を流用:
    - 401 → JQuantsAuthError 即時例外
    - 429 → 指数バックオフ (DEFAULT_RETRY_BACKOFF_SEC × 2^attempt) で
            DEFAULT_RETRY_COUNT 回リトライ
    - 4xx (403 含む) → JQuantsAPIError
    - 503 → 既存 retry でカバー (request 例外として再試行)
  - pagination: pagination_key 仕様は契約後確認 (daily は pagination
    なしで全件返るが、分足は 1 銘柄 1 日 360 bar = pagination 不要の可能性)
  - 1 リクエスト = 1 銘柄 1 日 が rate limit カウント単位と推定

## 7. 取得可能期間

  公式記載: **過去 2 年間** (Add-ons "株価 分足・ティック (2 年間)")
  → 60 営業日 (約 3 ヶ月) backfill は十分カバー

## 8. staging table 最終案

```sql
CREATE TABLE market_prices_intraday (
    code TEXT NOT NULL,
    dt TEXT NOT NULL,                  -- 1 分 bar 開始 ISO timestamp
                                        -- (例: '2026-05-01T09:00:00+09:00')
    interval_min INTEGER NOT NULL,     -- 1 / 5 / 15 (1 = 元データ、
                                        -- 5/15 は派生集計)
    open REAL NOT NULL,
    high REAL NOT NULL,
    low REAL NOT NULL,
    close REAL NOT NULL,
    volume REAL,                        -- bar 内出来高
    turnover_value REAL,                -- bar 内売買代金
    vwap_cumulative REAL,               -- 当日累積 VWAP (cumsum 計算結果、
                                        -- ETL で計算)
    fetched_at TEXT,                    -- ISO timestamp (取得時刻)
    PRIMARY KEY (code, dt, interval_min)
);
CREATE INDEX idx_mpi_code_date
  ON market_prices_intraday(code, substr(dt, 1, 10));
CREATE INDEX idx_mpi_dt
  ON market_prices_intraday(dt);
CREATE INDEX idx_mpi_interval
  ON market_prices_intraday(interval_min);
```

  保存方針 (HQ Q-Lane-C-C0-4 確定):
  - **保存元: 1 分足** (interval_min=1)
  - **評価 MVP: 5 分足** (interval_min=5、ETL で 1 分から派生集計)
  - **15 分足: 比較用 / 軽量版** (interval_min=15、ETL で 1 分から派生)

  派生集計 ETL:
  ```sql
  -- 5 分足派生 (1 分から)
  INSERT OR REPLACE INTO market_prices_intraday
  (code, dt, interval_min, open, high, low, close, volume,
   turnover_value, vwap_cumulative, fetched_at)
  SELECT
    code,
    -- dt: 5 分単位の bar 開始時刻に floor
    substr(dt, 1, 14) || printf('%02d', (CAST(substr(dt, 15, 2) AS INTEGER) / 5) * 5) || ':00+09:00' AS dt5,
    5 AS interval_min,
    -- 5 分内の最初の open
    -- 5 分内の max high / min low
    -- 5 分内の最後の close
    -- 5 分内の volume / turnover の sum
    -- vwap_cumulative は当日累積、5 分単位で再計算
    ...
  FROM market_prices_intraday
  WHERE interval_min = 1
  GROUP BY code, dt5
  ```

  migration: `scripts/setup/migrate_intraday_columns.py` 新規
  - **staging 専用** (B-strict-2a と同 4 段 guard:
    FIRE_ENV='staging' / db_path.is_symlink() reject /
    db_path.resolve().name == 'fire.staging.db' / 既存 column 冪等)
  - production / develop には適用しない

## 9. Tier2 universe 60 営業日 backfill 見積もり

### 9-1. データ規模

  Tier2 universe (HQ 確定 primary):
  - 60 営業日合計 candidate: **897 件**
  - 1 日平均 candidate: **14.95 件**
  - ユニーク銘柄推定: 60 営業日通じて **200-400 銘柄** (universe
    メンバーシップで重複カウント済)

  1 銘柄 1 日 1 分足の row 数:
  - 取引時間 09:00-11:30 + 12:30-15:00 = **300 min**
  - = 300 row/銘柄/日 (1 分足)

  60 営業日 × 200-400 銘柄 × 300 min/銘柄/日:
  - **360 万 〜 720 万 row** (1 分足)

  サイズ (1 row 約 150 bytes inline):
  - **約 540 MB 〜 1.08 GB** (1 分足のみ)

  + 5 分足 / 15 分足 派生 (1 分から集計):
  - 5 分: 60 row/銘柄/日 → 72 万 - 144 万 row (108-216 MB)
  - 15 分: 20 row/銘柄/日 → 24 万 - 48 万 row (36-72 MB)

  staging DB 増分予想: **約 700 MB - 1.4 GB** (現 1.38 GB → 2.1-2.8 GB)

### 9-2. fetch 工数 (rate limit 別)

  仮定: 1 リクエスト = 1 銘柄 × 1 日の全 bar (= 300 row)

  Tier2 universe 60 営業日 backfill:
  - リクエスト数 = 200-400 銘柄 × 60 営業日 = **12,000-24,000 req**

  rate limit 別所要時間:
  | プラン | rate limit | 12,000 req | 24,000 req |
  |---|---|--:|--:|
  | Light | 60 req/min | **3.3 h** | 6.7 h |
  | Standard | 120 req/min | 1.7 h | 3.3 h |
  | Premium | 500 req/min | 24 min | 48 min |

  ★ Mac mini 推奨: **Light + Add-ons (月額 ¥7,150)** で 3.3-6.7 h
    backfill が現実的。Standard / Premium は Lane C MVP には過剰。

### 9-3. backfill 設計

  scripts/jobs/fetch_intraday_data.py 新規 (Phase C1):
  - Tier2 universe 銘柄リストを daily precheck 結果から抽出
  - 60 営業日 × ユニーク銘柄で順次 fetch
  - rate limit 自動制御 (sleep 1 秒/req with safety margin)
  - INSERT OR REPLACE で冪等
  - staging のみ書込み (DB_PATH guard)
  - tqdm 等で進捗表示
  - 中断時 resume 対応 (既取得分 skip)

## 10. Phase C1 へ進めるかの判定

  ★ **判定: Fujiwara 判断待ち** ★

  進める条件:
  - Add-ons (株価 分足・ティック) 月額 ¥5,500 の契約
    + Light/Standard/Premium プラン契約 (現契約に応じて追加 / アップグレード)
  - 月額コスト判断: 最小 Light + Add-ons = ¥7,150/月

  ★ Mac mini は契約操作しない (HQ 制約)。Fujiwara が公式サイトで
    Add-ons 契約後に Phase C1 着手可能。

  進めない場合の代替案:
  1. 楽天証券 API (Mac mini 推奨度低、HQ Q-Lane-C-C0-3 で 4 位)
  2. 他社商用 API (TradingView 等、コスト + 利用規約要確認)
  3. Lane C 凍結、Lane B / Lane D 等の別 strategy 評価へ進む

## 11. 完了基準チェック (HQ §5 F105 完了基準)

  | # | 基準 | 結果 |
  |--:|---|---|
  | 1 | J-Quants Add-ons (`/v2/equities/bars/minute`) 疎通確認 | △ (現契約では 403 reject、契約必要を確認) |
  | 2 | 契約 / コスト / Fujiwara 判断結果 Vault 化 | ✅ (本ファイル §2-3) |
  | 3 | staging market_prices_intraday table 設計確定 | ✅ (§8) |
  | 4 | migration script 設計案 + 1 銘柄 1 日 sample 取込 PoC | ⚠️ migration 設計案のみ (PoC は契約後) |
  | 5 | backfill 工数見積もり | ✅ (§9) |
  | 6 | 結果 Vault 化 | ✅ (本ファイル) |

  → 完了基準 6 項目中 4 項目完了、2 項目 (#1, #4) は契約後に再検証必要。

## 12. 次工程

  ### Phase C1 (Fujiwara Add-ons 契約後に着手)

    C1-1: 既存 J-Quants client (market_data/client.py) に分足 endpoint
          追加 (`get_intraday_minute_prices` method)
    C1-2: scripts/setup/migrate_intraday_columns.py 新規 (staging 専用、
          B-strict-2a 同 4 段 guard)
    C1-3: market_data/intraday.py 新規 (1 分 → 5 分 / 15 分派生 ETL)
    C1-4: scripts/jobs/fetch_intraday_data.py 新規 (Tier2 universe
          60 営業日 backfill、rate limit 制御 + 冪等)
    C1-5: 60 営業日分 Tier2 universe backfill 実行 (3.3-6.7 h、Light
          + Add-ons 想定)
    C1-6: 1 分 / 5 分 / 15 分の派生集計 ETL 実行 + 検証
    C1-7: Phase C1 結果 Vault 化

  ### Phase C2 (Lane C 真実装、Phase C1 完了後)

    Lane C 設計記録 §9-Phase C2 参照

## 13. 関連 Vault リンク

  - [[F281_Lane_C_design_2026-05-08|Lane C 設計記録]]
  - [[F281_Lane_C_universe_precheck_2026-05-08|Lane C universe precheck 結果]]
  - [[F105_J-Quants個別銘柄分足Add-ons接続検証|F105 起票 todo]]
  - [[F100_市場データAPI|F100 J-Quants V2 daily]]

## 14. 改訂履歴

  - v1.0 (2026-05-08): 初版、Mac mini 範囲の検証完了、Fujiwara 契約
    判断待ち
