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

## 15. 契約後 minimum 疎通再テスト結果 (2026-05-08、v1.1 追記)

  ★ Fujiwara が J-Quants Standard + Add-ons 契約完了
  (月額 ¥3,300 + ¥5,500 = ¥8,800)、最小疎通再テスト実施。

### 15-1. /v2/equities/bars/minute 疎通結果

  実行: requests.get で 1 銘柄 (7203) × 1 日 (2026-05-01)
  HTTP: **200 ✅** (契約有効化確認)

  response field 確定 (実 response):
  | field | 型 | サンプル値 | 備考 |
  |---|---|---|---|
  | Date | str | "2026-05-01" | yyyy-mm-dd |
  | **Time** | str | "09:00" | **HH:MM 形式 (1 分単位)** |
  | Code | str | "72030" | 5 桁 (daily と同型) |
  | O | float | 3000.0 | bar 開始価格 |
  | H | float | 3015.0 | bar 内高値 |
  | L | float | 2989.5 | bar 内安値 |
  | C | float | 2995.0 | bar 終値 |
  | Vo | float | 2208300.0 | bar 内出来高 |
  | Va | float | 6623430350.0 | bar 内売買代金 |

  top-level keys: `['data']` のみ (daily と同型)

### 15-2. 1 日 bar 数 + 時間帯分布

  total bars: **327**

  | 時間帯 | bar 数 | 備考 |
  |---|--:|---|
  | 09:00 - 11:29 (前場) | 150 | 連続、skip なし ✅ |
  | 11:30 (前場板寄せ) | 1 | 大引け前後 1 bar |
  | 12:30 - 15:24 (後場連続) | 175 | 連続 |
  | 15:25 - 15:29 | **0** | クロージング・オークション (CA) 期間、bar 未生成 |
  | 15:30 (大引け板寄せ) | 1 | 大引け 1 bar |
  | 合計 | **327** | |

  ★ 重要観察: 15:25-15:29 (CA 期間) は bar 生成なし、Phase C2 Lane C
    実装で entry / monitor 経路に注意。entry は 14:00 cutoff (no_new_
    entry_after_hour=14) で安全、close は 15:10 force_close が 15:24 の
    最後の連続 bar より前なので問題なし。

### 15-3. 取引なし minute / 連続性

  - Vo == 0 bars: **0** (取引なし bar は返らない仕様)
  - O is None bars: **0**
  - 前場 09:00-11:29 で 1 分間隔 skip なし、連続 150 bar
  - → 流動性ある銘柄では 1 分連続生成、低流動銘柄は要追加検証 (本検証
    は 7203 Toyota = 高流動銘柄のみ、Lane C universe Tier2 は 30 億
    以上で流動性確保)

### 15-4. pagination

  - top-level に `pagination_key` 不在 = 1 銘柄 1 日では全件 1 response
    で返却
  - 大量取得 (複数日 / 複数銘柄) で pagination 必要となる可能性、
    Phase C1 実装時に多日 / 多銘柄でテスト要

### 15-5. rate limit headers

  response headers に rate limit 情報なし (X-Ratelimit-* 系不在)
  → プラン別固定値 (Standard = **120 req/min**) で実装側で attempt
    cap を実装

### 15-6. /v2/equities/trades 疎通結果

  HTTP: **403** "The requested endpoint does not exist"

  ★ Add-ons 契約済でも path 不存在。仕様変更 / 名称変更の可能性、
    公式 spec 確認推奨。Lane C MVP は **minute で十分**、trades は
    Phase C 範囲外、別タスクで再検証。

### 15-7. Phase C1 進めるかの判定 (更新)

  ★ **判定: 進める ✅** ★

  根拠:
  - Add-ons 契約完了 + minute API HTTP 200 確認
  - response field 確定 (Date / Time / Code / O / H / L / C / Vo / Va)
  - 1 日 327 bar、前場/後場境界 + CA 期間の挙動把握
  - rate limit Standard 120 req/min 想定で backfill 工数 1.7-3.3 h

  Phase C1 着手可、本部 GO 判断要。

## 16. Phase C1 実装計画 (再提示、契約後確定版)

### 16-1. 実装ステップ + commit 分割案

  c1: **feat(intraday): client に minute endpoint 追加 + test**
      - market_data/client.py に `get_intraday_minute_prices(code, date,
        date_from, date_to, pagination_key)` method 追加
      - 既存 retry / 401 / 429 / 4xx ハンドリング流用
      - tests/market_data/test_client_intraday.py 新規

  c2: **feat(intraday): migrate_intraday_columns.py + test**
      - scripts/setup/migrate_intraday_columns.py 新規
      - staging 専用 4 段 guard (B-strict-2a 同パターン:
        FIRE_ENV='staging' / db_path.is_symlink() reject /
        db_path.resolve().name == 'fire.staging.db' / 既存列冪等)
      - market_prices_intraday table CREATE + 3 INDEX
      - tests/scripts/test_migrate_intraday_columns.py 新規

  c3: **feat(intraday): intraday.py (1分→5分/15分 派生 ETL) + test**
      - market_data/intraday.py 新規
      - 1 分足 fetch + 5 分足 / 15 分足派生集計 (SQL group by)
      - vwap_cumulative 計算 (cumsum(typical_price × volume) /
        cumsum(volume))
      - tests/market_data/test_intraday_etl.py 新規

  c4: **feat(intraday): fetch_intraday_data.py (backfill ジョブ) + test**
      - scripts/jobs/fetch_intraday_data.py 新規
      - Tier2 universe 銘柄リストを daily precheck 結果から抽出
      - 60 営業日 × 銘柄 × 1 リクエスト
      - rate limit 自動制御 (sleep 0.5 秒/req with safety margin =
        実効 120 req/min)
      - INSERT OR REPLACE で冪等
      - staging 専用 (DB_PATH guard)
      - 中断時 resume 対応 (既取得分 skip)
      - tests/scripts/jobs/test_fetch_intraday_data.py 新規

  c5: **chore(intraday): 60 営業日 Tier2 backfill 実行 + 結果 Vault 化**
      - 実 backfill 走行 (Standard 120 req/min で 1.7-3.3 h 想定)
      - 1 分 / 5 分 / 15 分派生 ETL 実行 + サンプル検証 (1 銘柄で
        7203 5/1 327 bar との整合)
      - 結果 Vault 化: 03_design/F105_Phase_C1_backfill_result_*.md

### 16-2. 制約再確認

  - **staging 専用** migration / 書き込み (production / develop 完全
    隔離、4 段 guard 厳守)
  - rate limit Standard 120 req/min で実効 sleep 0.5 sec/req、29 req
    余裕で safety margin 設定 (=4 倍 backoff、実効 120 → 100 req/min)
  - 中断時 resume: 既取得 (code, date) を SELECT で除外
  - Codex pre-commit 必須、--no-verify 不使用、個別 commit 厳守

### 16-3. リスク (更新)

  R-1: 多日 / 多銘柄 fetch で pagination 出現の可能性
       → fetch ジョブで pagination_key ループ実装
  R-2: 低流動銘柄で取引なし minute の挙動
       → Tier2 universe (30 億以上) は流動性確保、ただし Phase C1
          backfill で実銘柄サンプル確認
  R-3: CA 期間 (15:25-15:29) bar 生成なし
       → Lane C entry 14:00 cutoff / close 15:10 で問題なし、設計
          書類化済
  R-4: J-Quants 仕様変更で field 命名変更
       → 月次で疎通再確認 (運用ジョブ化検討、Phase C1 範囲外)

## 17. 改訂履歴

  - v1.0 (2026-05-08): 初版、Mac mini 範囲の検証完了、Fujiwara 契約
    判断待ち
  - v1.1 (2026-05-08): 契約後 minimum 疎通再テスト結果追記
    (HTTP 200 / response field 確定 / 327 bar / rate limit 確認、
     Phase C1 進める判定確定)
