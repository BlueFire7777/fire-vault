---
title: F286 Research Lane R1 implementation plan
date: 2026-05-09
phase: F286 R1 (Sector Flow Agent MVP + market_financials backfill 設計)
status: 設計案、HQ 承認待ち。実装は承認後に commit 分割で着手
related: F286_Research_Lane_R0_feasibility_2026-05-09, F285_Research_Lane_requirements_and_spec_2026-05-08, F100_J-Quants, F101_TDnet, F276_indices_spec_2026-05-05
trigger: HQ R1 着手指示 (2026-05-09、F286 R0 feasibility 完了承認後)
---

# F286 Research Lane R1 implementation plan

★ R1 = (A) Sector Flow Agent MVP + (B) market_financials に
   /v2/fins/summary を取り込む設計確定。**実装前の設計 Vault 化が
   R1 のスコープ**、本格 backfill / migration / Agent 実装は HQ 承認後
   commit 分割で着手。

## 1. 概観

R1 で確立する 2 系統:

A. **Sector Flow Agent MVP** (= F285 §4-1 の最小実装)
   - 入力: 既存 staging DB (market_listings + market_prices_daily)
   - 追加 endpoint: なし (R0 で確認済、Premium 不要)
   - 出力: sector 別資金流入 ranking + flow_score (-1.0 〜 +1.0)

B. **market_financials backfill 設計**
   - 取得元: /v2/fins/summary (R0 で `available` 確認済、107 fields)
   - 既存 schema: 13 columns (R0 で確認済、row_count=0)
   - 実装: R1 で fetch + insert ロジック確立、R2 で派生指標
     (PBR/PER/ROE/配当性向) 計算

R1 で R2 以降に渡す財務データ基盤を作るため、payload_json で全 107
fields を保存しつつ、主要数値 (Sales / OP / OdP / NP / TA / Eq / CFO)
は schema field に展開する。

## 2. R0 結果の整理 (R1 前提)

R0 feasibility 結果より:
- /v2/fins/summary: ✅ Standard available、234 件 / 5 銘柄、107 fields
- /v2/fins/details / dividend: ❌ Premium、ただし MVP では不要 (派生 + summary で代替)
- /v2/equities/earnings-calendar: ✅ available、144 件 / 14d-30d
- 既存 staging DB: market_listings (4,449) / market_prices_daily (526,764)
  / market_financials (0、schema あり) / features (1,131,331)

R1 で /fins/details / /fins/dividend は使わない (= MVP 範囲外、Premium
不要)。/v2/equities/earnings-calendar は R3 / F287 で扱う。

## 3. A. Sector Flow Agent MVP 仕様

### 3.1 役割

F285 §4-1 準拠: 業種別資金循環パターンを抽出、強い業種を up-flow /
弱い業種を down-flow としてラベリング、Cyclical Value Screener 等の
入力にする。

### 3.2 入力データ (既存 DB のみ、追加 endpoint なし)

| field | 取得元 | 用途 |
|---|---|---|
| code | market_listings | 銘柄識別 |
| sector_17_code / sector_17_name | market_listings | 17 業種分類 |
| sector_33_code / sector_33_name | market_listings | 33 業種分類 |
| scale_category | market_listings | 大型/中型/小型 区分 |
| date / open / high / low / close | market_prices_daily | 日次 OHLC |
| volume | market_prices_daily | 出来高 |
| turnover_value | market_prices_daily | 売買代金 |
| adj_close | market_prices_daily | 配当落ち調整済 (return 計算用) |

### 3.3 集計ロジック

#### 3.3.1 銘柄レベル日次指標 (per code, per date)

```
daily_return = (adj_close_t - adj_close_{t-1}) / adj_close_{t-1}
turnover_jpy = turnover_value
volume = volume
volume_ratio_sma20 = volume / mean(volume_{t-20:t-1})  # 直近 20 営業日 SMA 比
```

#### 3.3.2 業種レベル集計 (per sector, per date)

17 業種 (= sector_17_code) と 33 業種 (= sector_33_code) の両方で集計
(= R3 で Watchlist Ranker が選択できるよう両方提供):

```
sector_mean_return = mean(daily_return for code in sector)
sector_total_turnover = sum(turnover_jpy for code in sector)
sector_total_volume = sum(volume for code in sector)
sector_mean_volume_ratio = mean(volume_ratio_sma20 for code in sector)
```

#### 3.3.3 sector_score (per sector, per date)

3 要素で合成 (各 0.0-1.0 にクリップ後加重平均、F285 §6 の重み案):

```
sector_score = (
    0.4 * sector_return_score      # 当日 return の絶対水準と相対 ranking
  + 0.3 * sector_momentum_score    # 過去 N 日累積 return の relative
  + 0.3 * sector_breadth_score     # 出来高/売買代金 SMA 比の業種内平均
)
```

詳細:
- `sector_return_score`: 全 33 業種 (or 17) の当日 mean_return を
  ranking、上位ほど 1.0 寄り (例: percentile rank で正規化)
- `sector_momentum_score`: 過去 5 日 cumulative return ranking
- `sector_breadth_score`: 業種内銘柄の volume_ratio_sma20 平均、
  ranking で正規化

### 3.4 銘柄レベル flow_score (per code, per date)

```
flow_score = sector_score (= 銘柄が属する sector の score を継承)
            (関数 = banner、銘柄個別の調整は R3 で実施)
```

R1 段階では「銘柄 = 所属 sector の score」を base、銘柄個別の出来高
強度などは R3 Watchlist Ranker 統合時に上乗せ。

### 3.5 Up-flow / Down-flow 分類

毎営業日に 33 業種 (or 17) を sector_score 降順で sort:
- Top 5 → up-flow (= 上位 5 業種)
- Bottom 5 → down-flow
- 残り → neutral

### 3.6 Morning Report 接続案

朝レポート (Morning Report、F236 既) に追加するセクション:
```
[Sector Flow]
  up-flow Top 5: 機械 (0.82) / 鉄鋼 (0.78) / 化学 (0.74) / ...
  down-flow Bottom 5: 不動産 (0.21) / 食品 (0.24) / ...
```

R4 で実装、R1 では spec 文書化のみ。

### 3.7 出力スキーマ案 (Phase R1 では DB 書込なし、計算のみ)

R1 段階では計算結果を JSON / Markdown で出力 (DB 書込は R2 以降)。

```python
@dataclass(frozen=True)
class SectorFlowSnapshot:
    date: str
    universe_size: int  # 当日 daily 取得銘柄数
    sector_scope: str   # "17" or "33"
    sectors: dict[str, SectorFlowEntry]  # code → entry
    rank_by_score: list[str]  # sector_id 降順
    up_flow: list[str]   # top 5 sector_id
    down_flow: list[str] # bottom 5 sector_id


@dataclass(frozen=True)
class SectorFlowEntry:
    sector_id: str
    sector_name: str
    n_codes: int
    mean_return: float
    total_turnover_jpy: float
    total_volume: float
    mean_volume_ratio_sma20: float
    return_score: float
    momentum_score: float
    breadth_score: float
    sector_score: float
    classification: str  # "up_flow" / "neutral" / "down_flow"
```

### 3.8 R1 で実装する範囲 (= MVP)

| 項目 | R1 | R2+ |
|---|:-:|:-:|
| 銘柄レベル daily_return / turnover / volume_ratio | ✅ | |
| sector 集計 (17 / 33 両対応) | ✅ | |
| sector_score 合成 (3 要素加重平均) | ✅ | |
| up-flow / down-flow 分類 | ✅ | |
| flow_score per code (= sector_score 継承) | ✅ | |
| 出力 dataclass + JSON serialize | ✅ | |
| 銘柄個別調整 | | ✅ R3 |
| DB 書込 | | ✅ R3 で feature table に保存 or 専用 table |
| Morning Report 接続 | | ✅ R4 |

## 4. B. market_financials /fins/summary backfill 設計

### 4.1 既存 schema (R0 確認済、変更なし)

```sql
-- staging DB の market_financials (確認済 13 columns、row_count=0)
CREATE TABLE market_financials (
  code TEXT,
  disclosure_date TEXT,
  fiscal_year_end TEXT,
  type_of_document TEXT,
  net_sales REAL,
  operating_profit REAL,
  ordinary_profit REAL,
  profit REAL,
  total_assets REAL,
  equity REAL,
  cash_flow_operating REAL,
  payload_json TEXT,
  fetched_at TEXT
);
```

★ R1 では schema 変更なし (= migration 不要)、本 schema を埋める形。
   将来 R2 以降で派生指標専用の column (BPS / EPS / DivAnn 等) を
   追加する場合は別 migration として提案。

### 4.2 /v2/fins/summary → market_financials field mapping

R0 で確認した 107 fields のうち、market_financials の 13 column に
対する 1 次 mapping:

| market_financials field | /fins/summary primary | fallback (NC* 非連結) | 備考 |
|---|---|---|---|
| `code` | `Code` | - | 5 桁化、内部標準形 |
| `disclosure_date` | `DiscDate` | - | YYYY-MM-DD |
| `fiscal_year_end` | `CurFYEn` | - | 当期 FY 終了日 |
| `type_of_document` | `DocType` | - | 開示種別 (FYFinancialStatements 等) |
| `net_sales` | `Sales` | `NCSales` | 連結優先、不在時非連結 fallback |
| `operating_profit` | `OP` | `NCOP` | |
| `ordinary_profit` | `OdP` | `NCOdP` | |
| `profit` | `NP` | `NCNP` | 純利益 |
| `total_assets` | `TA` | `NCTA` | |
| `equity` | `Eq` | `NCEq` | 自己資本 |
| `cash_flow_operating` | `CFO` | (なし) | 連結のみ、非連結 fallback なし |
| `payload_json` | (全 107 fields の生 JSON) | - | R2 派生計算用に raw 保存 |
| `fetched_at` | (= 取込時刻 ISO) | - | |

連結優先 + 非連結 fallback の理由:
- 上場企業の大部分は連結決算、Sales 等は連結値 (= IFRS / JGAAP 連結)
- 非連結のみの企業 (e.g., 銀行・保険等の個別決算) では NC* fallback

### 4.3 重複・修正開示の扱い (主キー設計)

#### 4.3.1 R1 主キー案

既存 schema は PRIMARY KEY 不在。R1 で UNIQUE 制約 migration を提案:

```sql
-- R1 着手後の migration 案 (HQ 承認後別 commit)
CREATE UNIQUE INDEX idx_market_financials_uniq
  ON market_financials (code, disclosure_date, type_of_document, disc_no);
```

ただし:
- 既存 schema に `disc_no` column がない → R1 migration で追加が必要
- migration 影響を最小化したい場合は `(code, disclosure_date, type_of_document)`
  で先行 (= 同一銘柄 同一日 同一 doc_type の重複を 1 行に統合)

★ HQ 判断必須 (Q1):
- 案 A: `(code, disclosure_date, type_of_document)` で UNIQUE、disc_no
  は payload_json 内に格納 → migration 最小、運用シンプル
- 案 B: `disc_no` column 追加 + `(code, disc_date, doc_type, disc_no)`
  UNIQUE → 1 銘柄 1 日複数開示を独立行で記録、修正開示も保存

R1 推奨: **案 A** (シンプル、修正開示は payload_json 内 RetroRst /
ChgByASRev フラグで識別可能)

#### 4.3.2 修正開示 (RetroRst / ChgByASRev / MatChgSub) の扱い

修正開示は同一 (code, DiscDate, DocType) で複数発生し得る。R1 では:
- 同 key 重複 → INSERT OR REPLACE で **最新 fetched_at の値を残す**
- 修正フラグ (ChgByASRev / RetroRst / MatChgSub / ChgAcEst /
  ChgNoASRev / SigChgInC) は payload_json 内に保存、R2 で個別判定可

### 4.4 backfill 対象期間

R0 sample で 5 銘柄合計 234 件 (= 1 銘柄 30-50 件 × 4-5 期 / 年 × 5-10
年) を確認。

R1 backfill 対象 (HQ 判断 Q2):
- 案 A: 過去 5 年 (= 概ね 4 期 × 5 年 = 20 件 / 銘柄 × 4,449 銘柄 ≈
  88,000 件)
- 案 B: 過去 3 年 (= 12 件 / 銘柄 × 4,449 ≈ 53,000 件)
- 案 C: 過去 10 年 (= 40 件 / 銘柄 × 4,449 ≈ 178,000 件)

推奨: **案 A** (5 年)
- Cyclical Value で過去のバリュー底打ち判定に十分
- Growth で 4 期 × 5 年 = 20 期分の trend 判定可
- 現実的な fetch 時間 (sleep 0.7 sec × 4,449 銘柄 = 約 52 分 / 1 期、
  全期間で 6-8 時間想定)

### 4.5 incremental update 方針

R1 implementation 後の運用:
- 日次: 当日 DiscDate の差分のみ fetch (= 全銘柄 1 日分 ≈ 数十件)
- 週次: 過去 7 日分 re-fetch (修正開示の遅延取込)
- backfill: 初回のみ HQ 承認後の本格 fetch (約 6-8 時間想定)

### 4.6 R2 派生指標の前提 (R1 設計で整合性確保)

R2 で計算する派生指標と、R1 で保存する元 field (= payload_json 内
含む全 107 fields からの抽出):

| 派生指標 | 計算式 | 必要 field (payload_json 内) |
|---|---|---|
| **PBR** | `close / BPS` | `BPS` (連結) or `NCBPS` (非連結) |
| **PER** | `close / EPS` | `EPS` / `DEPS` (希薄化後) / `NCEPS` |
| **ROE** | `profit / equity` | `NP / Eq` (= market_financials.profit / equity 直接利用) |
| **営業利益率** | `operating_profit / net_sales` | `OP / Sales` (= 直接利用) |
| **時価総額** | `close × AvgSh` | `AvgSh` / `ShOutFY` (発行済株式数) |
| **配当性向 (実績)** | `DivAnn / EPS` or `PayoutRatioAnn` | `PayoutRatioAnn` (直接) / `DivAnn` |
| **配当性向 (予想)** | `FDivAnn / FEPS` or `FPayoutRatioAnn` | `FPayoutRatioAnn` (直接) |
| **売上成長率** | `(Sales_t - Sales_{t-1}) / Sales_{t-1}` | `Sales` 連結 4 期 trend |
| **EPS 成長率** | `(EPS_t - EPS_{t-1}) / EPS_{t-1}` | `EPS` |
| **会社予想成長率** | `(NxFSales - FSales) / FSales` | `FSales` / `NxFSales` |
| **上方修正フラグ** | `ChgByASRev / ChgNoASRev = "1"` | `ChgByASRev` / `ChgNoASRev` |
| **連続増配年数** | `DivAnn` の 5 期増加判定 | `DivAnn` 5 期 trend |

★ **必要な全 fields は /fins/summary 単独で取得可能** (R0 検証で確認済)。
   R2 派生計算で `market_prices_daily.close` と JOIN するだけ。

## 5. R1 実装 commit 分割案 (HQ 承認後着手)

| # | commit | 内容 | scope |
|---|---|---|---|
| **R1-A1** | `feat(F286-R1): add Sector Flow Agent MVP` | sector 集計 + score 計算 + dataclass | ~/fire 側、test 必須 |
| **R1-A2** | `chore(F286-R1): add Sector Flow Agent runner` | CLI script + smoke (5 営業日 sample) | ~/fire 側 |
| **R1-B1** | `feat(F286-R1): /fins/summary -> market_financials mapping` | parser + INSERT OR REPLACE ロジック + test | ~/fire 側 |
| **R1-B2** | `chore(F286-R1): backfill_market_financials runner` | CLI script (sample 5 銘柄 smoke、本格 backfill は HQ 承認後) | ~/fire 側 |
| **R1-B3** | `chore(F286-R1): vault Sector Flow / market_financials smoke result` | smoke 結果を Vault 化 | vault のみ |
| **R1-B4** | `chore(F286-R1): full backfill 5 years for market_financials` | 全銘柄 backfill 走行 (HQ 承認 + 走行確認後) | ~/fire 側 (走行 log + JSON) |
| **R1-C1** | `docs(F286-R1): vault R1 result + Phase R2 着手判断` | R1 完了後の Vault 化 | vault のみ |

各 commit Codex pre-commit 必須、--no-verify 禁止、個別 commit 厳守。
test と migration は **必要時のみ最小実装**、本実装に直結しない work
は別 commit にしない。

## 6. 受入基準 (R1)

A. Sector Flow Agent MVP:
- ✅ 既存 staging DB (read-only) のみで動作
- ✅ 17 業種 / 33 業種 両方の sector_score を計算
- ✅ up_flow / down_flow ranking が当日分出力される
- ✅ smoke (5 営業日分) で既存 DB から sector_score を計算、出力検証
- ✅ regression なし

B. market_financials backfill 設計:
- ✅ /fins/summary 107 fields → market_financials 13 columns mapping
  確定
- ✅ 連結優先 + 非連結 fallback の logic 設計
- ✅ 主キー設計確定 (HQ Q1 判断後)
- ✅ payload_json で全 107 fields 保存 (R2 派生計算前提)
- ✅ 5 銘柄 smoke で insert/upsert 動作確認 (HQ Q2 判断後)
- ✅ R2 派生指標の必要 field 確認

## 7. 制約 (HQ 厳守、R0 から継承)

- 自動発注 / 楽天証券操作 / Computer Use 禁止
- production / develop DB 無触
- staging DB のみ (write は backfill 時のみ、それまでは read-only)
- API key / 認証情報を log / Vault に書かない
- 外部スクレイピング禁止
- migration / 本格 ETL は HQ 承認後 (R1-B1 以降)
- いきなり全銘柄 backfill しない (= R1-B2 で 5 銘柄 smoke → R1-B4 で
  HQ 承認後 全銘柄)
- Premium 課金禁止 (MVP では不要)
- Codex pre-commit 必須 / --no-verify 禁止 / 個別 commit 厳守

## 8. リスク

1. **/fins/summary field 仕様変更**: V2 docs 上の field 定義が将来
   変わる可能性。R1 実装で `payload_json` に raw 保存することで耐性
   確保。R2 派生計算は schema 適応的に書く。

2. **連結 / 非連結の field 切替**: 銘柄の決算開示が連結 / 非連結 で
   切替わる場合、Sales 等が NULL になる可能性。fallback logic で
   NCSales を採用するが、混在する場合の trend 判定 (= R2 で問題化)
   は R2 implementation で確認。

3. **修正開示の頻度**: 同 (code, DiscDate, DocType) の上書きが頻発
   する場合、payload_json の最新が常に正しいとは限らない。R2 で
   RetroRst / ChgByASRev フラグ参照を必須化する。

4. **backfill 走行時間**: 全銘柄 5 年 ≈ 6-8 時間。F284/F105 c6 で
   経験済 (= 11h54m)、同様の rate limit 配慮 (sleep 0.7-1.0 sec) で
   回避可能。

5. **market_financials schema の前提**: 既存 13 columns で MVP 十分
   (R0 確認済)、ただし R2 で派生指標専用 column (BPS/EPS/DivAnn 等)
   が必要になった場合は別 migration として提案 (= migration を R1
   範囲外に保つ)。

## 9. HQ 判断要請

Q1: **market_financials 主キー設計 (§4.3.1)**
   案 A: `(code, disclosure_date, type_of_document)` で UNIQUE、
        disc_no は payload_json 内
   案 B: `disc_no` column 追加 + `(code, date, doc_type, disc_no)`
        UNIQUE
   推奨: 案 A (migration 最小、シンプル)

Q2: **backfill 対象期間 (§4.4)**
   案 A: 過去 5 年 (推奨、約 88,000 件、6-8 時間)
   案 B: 過去 3 年
   案 C: 過去 10 年
   推奨: 案 A (Cyclical Value / Growth に十分、現実的時間)

Q3: **commit 分割 R1-A〜R1-C (§5)**
   7 commit 案で OK か?
   先行着手 (= R1-A1 から) 順序選好あれば指示。

Q4: **F287 並行着手**
   F287 (決算ダッシュボード) と F286-R1 を並行進行は許容?
   R1-B1 (mapping) 完了後に F287 着手が現実的、HQ 判断。

Q5: **R1 完了基準**
   §6 受入 A+B 全項目 PASS で R1 完了 OK か?

## 10. 関連 Vault ファイル

- [[F286_Research_Lane_R0_feasibility_2026-05-09|F286 R0 feasibility]] (上位、R1 前提)
- [[F285_Research_Lane_requirements_and_spec_2026-05-08|F285 仕様 v1.1]] (Research Lane 全体仕様)
- [[F284_F105_c6_final_result_2026-05-09|c6 final result]] (data infrastructure 前提)
- [[F286_Research_Lane_R0_feasibility|F286 R0 TODO]]
- [[F101|F101 TDnet]] (announcements、補完用)
- [[F100|F100 J-Quants V2]] (上位 client)

## 11. 改訂履歴

- v1.0 (2026-05-09): 初版、HQ R1 着手指示後の設計 Vault 化、Sector
  Flow Agent MVP 仕様 + /fins/summary → market_financials mapping +
  R2 派生指標前提 + commit 分割 7 案 + HQ 判断 5 項目要請。
