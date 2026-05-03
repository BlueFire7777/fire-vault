---
id: F101
phase: P6: データソース
priority: 高
status: 完了 (Phase 1 + Phase 2 + Phase 3)
owner: Fujiwara
depends_on: [F100, F021]
chapter: "14"
created: 2026-05-01
updated: 2026-05-03
---

# F101: TDnet / IR API 連携

## 概要

第 14 章 R-14-04 のデータソース優先度 4 位を実装。Phase 1 で J-Quants V2
`/fins/announcement` (決算発表予定日) を取得し、`announcements` テーブルに
ALLOWED_MATERIALS マッピング付きで投入。F021 MaterialEvent / F035 Pattern
Research の入力データ供給準備が整う。

## スコープ

- `materials/` パッケージ新規 (client / fetcher / mapper / models)
- `announcements` テーブル新設 (raw_payload も保存、再パース可能)
- J-Quants V2 認証は F100 と共通 (`x-api-key` ヘッダ + `JQUANTS_API_KEY`)
- ALLOWED_MATERIALS マッピング (`mapper.JQUANTS_CODE_MAPPING` + title 補助)
- 定期取得ジョブ `scripts/jobs/fetch_announcements.py`
- AnnouncementSource enum で将来 TDnet HTML / News API を追加可能

## 重要な決定事項 (仕様書差分)

- **エンドポイントは `/fins/announcement`** (仕様書 `/markets/announcement` は誤り)
- **認証は `x-api-key` ヘッダ** (仕様書 `Authorization: Bearer` は誤り、F100 と同じ)
- **環境変数は `JQUANTS_API_KEY`** (仕様書 `JQUANTS_REFRESH_TOKEN` は誤り)
- **base_url は `/v2`** (仕様書 `/v1` は誤り、F100 と同じ)
- **エラークラス**: `JQuantsAuthError` / `JQuantsRateLimitError` は F100 から再利用、
  `JQuantsAnnouncementError` のみ新規定義
- **J-Quants `/fins/announcement` は決算予定日のみ**: 「好決算」内容判定は
  別タスク (F021 / TDnet HTML 連携)。Phase 1 では earnings_upside に低
  confidence (0.5) でマッピング

## announcements テーブル schema (新規)

```sql
CREATE TABLE announcements (
    announcement_id   TEXT PRIMARY KEY,    -- "JQ-ANN-{Code}-{Date}-{Quarter}"
    symbol            TEXT NOT NULL,
    company_name      TEXT,
    announced_at      TEXT NOT NULL,       -- ISO datetime UTC
    announced_date    TEXT NOT NULL,       -- ISO date
    raw_type          TEXT,                -- "決算短信" 等
    raw_subtype       TEXT,                -- "第4四半期" 等
    title             TEXT,                -- 合成タイトル
    material_type     TEXT,                -- "earnings_upside" / etc.
    material_confidence REAL DEFAULT 0.0,
    pdf_url           TEXT,
    source            TEXT NOT NULL DEFAULT 'jquants_announcement',
    raw_payload       TEXT,                -- JSON 全文
    fetched_at        TEXT NOT NULL,
    parsed_at         TEXT,
    parse_error       TEXT
);
-- インデックス: symbol+date / material_type / announced_at / source
```

## 成果物

### 新規ファイル

- `materials/__init__.py`: パッケージ export
- `materials/models.py`: `Announcement` + `AnnouncementSource` enum
- `materials/mapper.py`: `JQUANTS_CODE_MAPPING` + `classify_material_type` +
  title キーワード補助
- `materials/client.py`: `JQuantsAnnouncementClient` (F100 流用 + リトライ)
- `materials/fetcher.py`: `ensure_announcements_schema` +
  `normalize_jquants_announcement` + `AnnouncementFetcher.fetch_and_store`
- `scripts/jobs/fetch_announcements.py`: 定期取得 CLI
- `tests/materials/conftest.py` + `test_models.py` + `test_mapper.py` +
  `test_client.py` + `test_fetcher.py`

### テスト

- 新規 24 件
  - test_models.py: 4 件 (defaults / enum / JSON / 全フィールド)
  - test_mapper.py: 9 件 (主要 8 マッピング + 辞書整合性)
  - test_client.py: 5 件 (key 不在 / x-api-key / 401 / pagination / 429)
  - test_fetcher.py: 6 件 (schema / normalize / insert / dup / json / 空)
- 累計: 828 → **852 PASS** (+24)
- 既存 828 への影響: **0 件** (新規パッケージのみ)

### Smoke 結果 (mock 3 件 → 実 DB)

```
fetch result: {'fetched': 3, 'inserted': 3, 'skipped': 0, 'errors': 0}

=== material_type 分布 (announcements) ===
  earnings_upside: 3

=== サンプル ===
  JQ-ANN-7203-2026-04-30-第4四半期 | 7203 | 2026-04-30 | earnings_upside (0.50) | トヨタ自動車 第4四半期決算発表予定
  JQ-ANN-9984-2026-04-30-第4四半期 | 9984 | 2026-04-30 | earnings_upside (0.50) | ソフトバンクG 第4四半期決算発表予定
  JQ-ANN-6758-2026-05-01-第4四半期 | 6758 | 2026-05-01 | earnings_upside (0.50) | ソニーG 第4四半期決算発表予定
```

## mapper 判定マトリクス

| raw_type / raw_subtype / title | material_type | confidence |
|---|---|---|
| 「自己株式取得」 | buyback | 0.95 |
| 「資本業務提携」 | alliance | 0.95 |
| 「業務提携」 | alliance | 0.9 |
| 「受注」 | order | 0.95 |
| 「決算短信」 | earnings_upside | 0.5 (内容不明) |
| 「業績予想修正」 | guidance_upside | 0.5 (上下不明) |
| 「配当予想修正」 | dividend_increase | 0.5 (増減不明) |
| title 「自己株式取得に関する...」(raw_type 不明時) | buyback | 0.7 |
| 完全不明 | None | 0.0 |

## スコープ外 (将来別タスク)

- TDnet HTML スクレイピング (Phase 2 / Source 抽象化済)
- 「好/悪決算」「上方/下方修正」内容判定 → F021 / 別タスク
- 前期比較・コンセンサス比較 → F021 別タスク
- ニュース API → F102
- TradingView 連携 → F103
- material_strength_score 精緻化 → F021 既実装の暫定継続

## Phase 2 完了 (2026-05-03)

### 概要

J-Quants `/fins/announcement` の限界 (決算予告のみ、内容不明) を補うため、
TDnet 公式サイト (release.tdnet.info) から日次の開示一覧 HTML を取得 →
タイトル正規表現で「上方/下方修正・増配/減配・自社株買い・TOB」の方向判定。
material_type confidence が **0.5 → 0.85-0.95** に上昇。

### 実装

- `materials/direction.py`: `Direction` enum (UPSIDE/DOWNSIDE/INCREASE/
  DECREASE/POSITIVE/NEUTRAL/UNKNOWN) + `detect_direction(title)`
- `materials/tdnet_client.py`: `TdnetHtmlClient` (1 req/sec レート制限 +
  encoding 自動判定 + 404 で自動打ち切り)
- `materials/tdnet_parser.py`: BeautifulSoup でテーブルパース
  (時刻/コード/会社名/タイトル/PDF/XBRL URL 抽出)
- `materials/mapper.py` 拡張: `classify_material_type_v2(title)` で
  Phase 1 ロジック + direction 判定を合成
- `materials/fetcher.py` 拡張: `AnnouncementFetcher.fetch_from_tdnet()` +
  `ensure_announcements_schema()` に `direction` カラム ALTER TABLE
- `scripts/jobs/fetch_tdnet_html.py`: 定期取得 CLI
- bs4 (beautifulsoup4 4.14.3) 新規インストール

### スキーマ変更

```sql
ALTER TABLE announcements ADD COLUMN direction TEXT DEFAULT NULL;
CREATE INDEX idx_announcements_direction ON announcements (direction);
```

direction 値: 'upside' / 'downside' / 'increase' / 'decrease' /
'positive' / 'neutral' / NULL。`ensure_announcements_schema` が冪等で
既存テーブルにも自動 ALTER。実 DB に動作確認済 (2026-05-03)。

### 重要な決定事項

- **ALLOWED_MATERIALS にネガ系材料がない**: DOWNSIDE / DECREASE / NEUTRAL は
  direction だけ記録、material_type=None / confidence=0.0。F021 が後段で
  「下方修正の翌日売り戦略」等を実装する余地を残す。
- **TDnet PK 衝突回避**: DisclosureNumber がないため
  `TDNET-{date}-{symbol}-{md5(title)[:10]}` で一意化
- **レート制限厳格遵守**: 1 リクエスト/秒、`_rate_limit()` で `time.sleep()`
- **encoding 自動判定**: `apparent_encoding` で Shift_JIS/UTF-8 両対応
- **PDF/XBRL は URL のみ保存**: ダウンロード + パースは Phase 3 へ

### Direction 判定マトリクス

| タイトル例 | direction | conf | material_type | mat_conf |
|---|---|---|---|---|
| 通期業績予想の上方修正に関するお知らせ | upside | 0.90 | guidance_upside | 0.90 |
| 通期業績予想の下方修正に関するお知らせ | downside | 0.90 | None | 0.00 |
| 配当予想の修正(増配)に関するお知らせ | increase | 0.85 | dividend_increase | 0.90 |
| 配当予想の修正(減配)に関するお知らせ | decrease | 0.85 | None | 0.00 |
| 自己株式取得に係る事項の決定 | positive | 0.90 | buyback | 0.95 |
| 公開買付応募契約の締結に関するお知らせ | neutral | 0.85 | None | 0.00 |
| 第○○回定時株主総会招集ご通知 | neutral | 0.85 | None | 0.00 |
| 代表取締役の異動に関するお知らせ | unknown | 0.00 | None | 0.00 |
| 業績予想の修正に関するお知らせ (上下不明) | unknown | 0.00 | guidance_upside | 0.50 (Phase 1 fallback) |

### テスト

- 新規 29 件
  - test_direction.py: 10 件 (7 パターンマッチ + 3 None/空/無関係)
  - test_tdnet_client.py: 5 件 (URL 形式 / UA / レート制限 / 404 打ち切り / max_pages)
  - test_tdnet_parser.py: 9 件 (3 行抽出 + フィールド + URL 絶対化 + XBRL +
    不正行 skip + 空 + helper 2)
  - test_fetcher_tdnet.py: 5 件 (insert / dup / material+direction / ALTER /
    raw_payload JSON)
- 累計: 852 → **881 PASS** (+29)
- 既存 852 への影響: **0 件** (新規ファイル + ALTER TABLE は idempotent)

### Smoke 結果 (mock TDnet 4 件 → 実 DB)

```
fetch result: {'fetched': 4, 'inserted': 4, 'skipped': 0, 'errors': 0}

source 分布:
  jquants_announcement: 3
  tdnet_html: 4

TDnet direction 分布:
  upside: 1 / positive: 1 / increase: 1 / downside: 1

サンプル:
  7203 トヨタ自動車    | upside    | guidance_upside    (0.90) | 通期業績予想の上方修正...
  9984 ソフトバンクG  | positive  | buyback            (0.95) | 自己株式取得に係る...
  6758 ソニーG       | increase  | dividend_increase  (0.90) | 配当予想の修正(増配)...
  4063 信越化学      | downside  | None               (0.00) | 通期業績予想の下方修正...
```

### 法的・倫理的注意

- 個人投資家としての一般公開ページ閲覧の範囲を超えない
- 1 req/sec レート制限を厳守 (`MIN_INTERVAL_SEC = 1.0`)
- User-Agent に「FIRE-Personal-Research」明示
- 商用利用しない、利用規約変更時は再評価

### Phase 2 のスコープ外

- PDF/XBRL ダウンロード + パース (Phase 3 / F021 連携)
- キー数値抽出 (売上/利益/前期比) (Phase 3)
- コンセンサス比較 (商用 API 必要、対象外)
- 過去日付の大量取得 (利用規約配慮で当日分が基本)
- F021 連携 (別タスク)

## Phase 3 完了 (2026-05-03)

### 概要

「announcements → MaterialEvent → features」の一気通貫パイプラインが稼働。
F021 既実装の `MaterialFeatureCollector.collect()` 経由で features テーブルに
material_type / strength_score / freshness / 各種フラグが自動投入される。
Pattern Research が実材料起因候補を抽出可能に。

### 実装

- `materials/xbrl_parser.py`: stdlib `xml.etree.ElementTree` ベースの軽量
  XBRL パーサー (lxml 不採用) + 主要 14 タグ + 派生指標 (前期比/利益率)
- `materials/event_bridge.py`: `announcement_to_material_event` で DB row →
  F021 `MaterialEvent` 変換、`parsed_metrics.forecast_*_yoy_change` を
  F021 用 `metadata.surprise_pct` に変換
- `materials/tdnet_client.py` 拡張: `fetch_xbrl_zip(url)` (レート制限維持)
- `materials/fetcher.py` 拡張: `parse_xbrl_for_announcement` +
  `sync_announcement_to_features` + `sync_all_pending_to_features`
- announcements に 4 カラム ALTER TABLE: `parsed_metrics` / `xbrl_url` /
  `xbrl_parsed_at` / `xbrl_parse_error` (idempotent)
- `scripts/jobs/sync_announcements_to_features.py`: 一括 sync ジョブ

### F021 実 API 確認結果 (仕様書差分)

| 仕様書推測 | 実体 |
|---|---|
| `MaterialEvent(symbol, dt, material_type, strength_indicator, ...)` | `MaterialEvent(symbol, title, published_at, source, body, metadata)` |
| `collector.collect_from_event(event)` | `collector.collect(symbol, dt, events=...)` |
| F101 mapper の英語 material_type ("earnings_upside") を直接渡す | F021 自身が `classify_material_type(event)` で日本語名 ("好決算"/"上方修正") に再分類 |
| 我々が `_estimate_strength` を実装 | F021 既存 `calculate_strength_score(event, type)` が `metadata.surprise_pct` で自動計算 |

→ event_bridge は薄い変換層に集約、F021 ロジックを最大限活用。

### 重要な決定事項

- **lxml 不採用**: stdlib `xml.etree.ElementTree` で十分 (依存最小化)
- **ローカル名マッチ**: XBRL ネームスペース prefix を正規表現で除去、
  `jpcrp_cor:NetSales` / `jppfs_cor:NetSales` の両方に対応
- **degradation 設計**: XBRL 取得失敗 → parsed_metrics=NULL でも
  direction だけで F021 連携可能 (surprise_pct は ±5.0 fallback)
- **surprise_pct への変換**: yoy_change=0.30 → surprise_pct=30.0
  (F021 の `calculate_strength_score` が ≥10% で +0.8、≥20% で +1.5 を
  加算するロジックに対応)
- **冪等 ALTER TABLE**: 既存テーブルに 4 カラム順次追加、
  `PRAGMA table_info` でカラム存在チェック

### スキーマ追加

```sql
ALTER TABLE announcements ADD COLUMN parsed_metrics TEXT DEFAULT NULL;
ALTER TABLE announcements ADD COLUMN xbrl_url TEXT DEFAULT NULL;
ALTER TABLE announcements ADD COLUMN xbrl_parsed_at TEXT DEFAULT NULL;
ALTER TABLE announcements ADD COLUMN xbrl_parse_error TEXT DEFAULT NULL;
```

### XBRL 主要 14 タグ (jpcrp_cor / jppfs_cor)

| カテゴリ | タグ | feature 名 |
|---|---|---|
| 売上 | NetSales / Revenue / RevenuesIFRS | net_sales / revenue / revenue_ifrs |
| 利益 | OperatingIncome / OperatingProfitLossIFRS / OrdinaryIncome / ProfitLoss / ProfitLossAttributableToOwnersOfParent | operating_income / operating_profit_ifrs / ordinary_income / net_income / profit_attributable_to_owners |
| 配当 | DividendPaidPerShareSummaryOfBusinessResults | dividend_per_share |
| 業績予想 | ForecastNetSales / ForecastOperatingIncome / ForecastProfitLossAttributableToOwnersOfParent | forecast_net_sales / forecast_operating_income / forecast_net_income |
| メタ | DocumentName / FilerNameInJapaneseDEI | document_name / filer_name |
| 派生 | (計算) | forecast_net_sales_yoy_change / forecast_operating_income_yoy_change / operating_margin |

### テスト

- 新規 25 件
  - test_xbrl_parser.py: 11 件 (success/bad zip/no xbrl/XML error/namespace/
    text field/zero division/IFRS/required keys/large numbers)
  - test_event_bridge.py: 8 件 (basic/surprise_pct/title 不在/symbol 不在/
    parsed_metrics 不在/downside/normalize_source/derive priority)
  - test_fetcher_phase3.py: 6 件 (not found/no url/success/sync success/
    no title skip/sync_all)
- 累計: 881 → **906 PASS** (+25)
- 既存 881 への影響: **0 件** (新規 + ALTER TABLE は冪等)

### 一気通貫 Smoke 結果 (real DB)

```
=== 実 DB schema check ===
  parsed_metrics: OK / xbrl_url: OK / xbrl_parsed_at: OK / xbrl_parse_error: OK

=== sync_all_pending_to_features ===
result: {'processed': 7, 'succeeded': 7, 'failed': 0, 'skipped': 0}

=== features 投入後の material 系 (抜粋) ===
  7203 (上方修正) | material_strength_score = 7.8  ← surprise +0.8 加算
  9984 (自社株買い) | shareholder_return_flag = 1.0
  6758 (増配) | material_strength_score = 6.0
  4063 (下方修正) | material_strength_score = 3.8 (その他扱い)
```

### Phase 3 のスコープ外

- PDF パース (Phase 3 不採用、ROI 低)
- 全件 XBRL 取得 (レート制限配慮、新規分から)
- コンセンサス比較 (商用 API 必要)
- Pattern Research の改修 (F035 既実装で動く)
- F035 動作確認 (実エントリー発生まで待つ)

## F101 全体 (Phase 1+2+3) 達成サマリ

| Phase | 内容 | 完了 |
|---|---|---|
| Phase 1 | J-Quants Announcement (決算予告、confidence 0.5) | 2026-05-02 |
| Phase 2 | TDnet HTML + direction 判定 (confidence 0.85-0.95) | 2026-05-03 |
| Phase 3 | XBRL パース + F021 連携 (features 投入、一気通貫) | 2026-05-03 |

合計テスト: 78 件 / 累計 PASS: 906 件 / 第 14 章 R-14-04 完全達成。

## 関連リンク

- 要件書: 第 14 章 R-14-04 / 第 36 章 A 材料特徴量
- 関連: [[F100_市場データAPI]] (認証共有) /
  [[F021_材料特徴量収集エンジン]] (連携先) /
  [[F035_Pattern_Research_Agent]] (下流候補抽出) /
  [[F102_ニュースAPI連携]] (未着手) /
  [[F104_指数四本値]] (regime 動的化、未着手)
- コード: `materials/`,
  `announcements` テーブル (Phase 1+2+3 拡張済),
  `scripts/jobs/fetch_announcements.py` (J-Quants),
  `scripts/jobs/fetch_tdnet_html.py` (TDnet HTML),
  `scripts/jobs/sync_announcements_to_features.py` (Phase 3 一気通貫)
