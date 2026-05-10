# F286-DATA-R0 J-Quants Pipeline Audit / Freshness Design

> **Status**: ✅ COMPLETED 2026-05-10
> **Mode**: 完全 read-only audit + 設計ドキュメント (新規本実装なし)
> **Result**: ★★★ 既存 J-Quants pipeline (8 module / 8 dataset) を
> 棚卸し、staging 実 freshness を audit JSON で取得、DATA-R1 daily
> refresh / DATA-R2 freshness gate の実装方針を確定。Stage 3 LINE
> 本番送信前の必須前提が整理された。★★★

---

## タスク名

F286-DATA-R0 J-Quants Pipeline Audit / Freshness Design

---

## 背景

F111-R1 〜 R4 + F062-R1 で ResearchAdvisory を LINE preview text まで
出せるようになったが、**本番 LINE 送信前に J-Quants データが古くなる
問題**を潰す必要がある。

特に staging 実 audit (本タスク 2026-05-10 実行):

    研究 lane signal:        max base_date = 2026-05-09 (= 当日 OK)
    market_prices_daily:     max date     = 2026-05-01 (= 8 日遅延)
    market_financials_v2:    max date     = 2026-05-08 (= 2 日遅延)
    market_prices_intraday:  max date     = 2026-05-01 (= 8 日遅延)
    research_derived_indicators: 2026-05-01 (= 8 日遅延)
    announcements:           max date     = 2026-05-01 (= 9 日遅延)

★ daily_quotes が **当日反映されていない** = LINE 通知の expected
metrics が 1 週間ズレる可能性。今のままで F062-R2 (本番送信) に
進むと「金曜の F119 を翌週月曜に通知」のような stale state が起きる。

---

## 調査したファイル一覧

### J-Quants 取得側 (= write 経路、本タスクでは触らない)

| File | 役割 | DB write 対象 |
|---|---|---|
| `market_data/client.py` | F100 J-Quants V2 client (x-api-key 認証 / retry / rate limit) | (none、HTTP のみ) |
| `market_data/fetcher.py` | listed_info / daily_quotes / fins/summary fetcher + parser | market_listings / market_prices_daily / market_financials |
| `market_data/historical.py` | F100-historical 過去データ取得 batch | market_prices_daily |
| `market_data/intraday.py` | F284 minute bar fetcher + 5min/15min derive | market_prices_intraday |
| `market_data/index_fetcher.py` | F104 /v2/indices fetcher | index_data |
| `market_data/repository.py` | upsert helper (save_listings / save_daily_prices / save_financials) | (above tables) |
| `materials/client.py` | F101 J-Quants /v2/fins/announcement + TDnet | (HTTP のみ) |
| `materials/fetcher.py` | F101 announcement fetcher + DB write | announcements |
| `simulation/research_lane/financials_mapping.py` | R1-B1 /v2/fins/summary → market_financials_v2 mapper | (mapping のみ、書き込みは backfill_market_financials.py) |

### J-Quants runner (CLI、いずれも write を伴う、本タスクでは触らない)

| File | 役割 |
|---|---|
| `scripts/jobs/fetch_historical_market_data.py` | F100 過去日足 batch (--from / --to / --symbols) |
| `scripts/jobs/fetch_intraday_data.py` | F284-c4 分足 backfill (--start / --end / --codes) |
| `scripts/jobs/fetch_announcements.py` | F101 announcement 定期取得 |
| `scripts/jobs/backfill_market_financials.py` | F286-R1-B2 sample / mini backfill (full は HQ 承認制) |

### J-Quants 関連 migration

| File | 用途 |
|---|---|
| `scripts/setup/migrate_market_data.py` | F100 listings / daily_prices schema 初期化 |
| `scripts/setup/migrate_market_financials_pk.py` | R1 案 A: financials UNIQUE 制約付与 |
| `scripts/setup/migrate_index_data.py` | F104 indices schema |
| `scripts/setup/migrate_intraday_columns.py` | F284 intraday schema 拡張 |
| `scripts/setup/migrate_research_watchlist_signals.py` | R2-D signals schema |
| `scripts/setup/migrate_research_derived_indicators.py` | R2-A 派生指標 schema |

### J-Quants 消費側 (= read 経路、本タスクでは構造のみ確認)

| File | 役割 |
|---|---|
| `simulation/research_lane/signal_persistence.py` | research_watchlist_signals load_* |
| `simulation/research_lane/return_evaluation.py` | market_prices_daily を使い h_horizon return を計算 |
| `simulation/research_lane/market_regime.py` | TOPIX 等の market regime classification (index_data + market_prices_daily) |
| `simulation/research_lane/sector_flow_features.py` | sector_flow / sector_17 features |
| `evaluation/interpretation_evaluation.py` | F119 evaluate (signals + regime + sector + return) |
| `agents/research_advisory.py` (F111-R1) | build_advisory (advisory dataclass) |

### F286-DATA-R0 で新規追加 (本タスク、read-only audit script のみ)

| File | 役割 |
|---|---|
| `scripts/jobs/audit_jquants_freshness.py` | read-only audit、JSON report 出力 |
| `tests/scripts/jobs/test_audit_jquants_freshness.py` | 16 PASS、URI mode=ro / write SQL refuse / source 文字列 safety 検証 |

---

## 既存 DB table 一覧と役割

### Tier-1 (= J-Quants 直接由来、daily refresh 対象)

| Table | Source endpoint | Owner | 役割 | PK / UNIQUE |
|---|---|---|---|---|
| `market_prices_daily` | `/v2/prices/daily_quotes` | F100 | 全銘柄日足 OHLCV (return_evaluation の前提) | (code, date) |
| `market_prices_intraday` | `/v2/prices/prices_am, prices_pm` | F284-c1 | tier2 銘柄 1分足 + 5分/15分派生 | (code, dt, interval_min) |
| `market_financials_v2` | `/v2/fins/statements` | F286-R1-B | 決算 / 進捗 (4 期 trend / 上方修正) | (code, disclosure_date, type_of_document) |
| `market_listings` | `/v2/listed/info` | F100 | 銘柄 master (会社名 / 17 業種 / 33 業種 / 規模) | (code) |
| `index_data` | `/v2/indices` | F104 | TOPIX / N225 等 index OHLCV | (date, symbol) |
| `announcements` | `/v2/fins/announcement` + TDnet | F101 | 適時開示 / 決算予告 | (announcement_id) |

### Tier-2 (= J-Quants 由来 derived、daily 再計算対象)

| Table | Owner | 派生元 | 役割 |
|---|---|---|---|
| `research_derived_indicators` | F286-R2-A | market_prices_daily + listings + financials_v2 | volatility / momentum / volume_rank 等の派生指標 |
| `research_watchlist_signals` | F286-R2-D | derived_indicators + factor_scoring + sector_cap | 日次 signal (final_score / post_cap_rank / sector_17_name 等) |

### Tier-3 (= R&D 系、本 audit ではトレンド観測のみ)

| Table | 役割 |
|---|---|
| `features` | F021-F026 / F101 phase3 由来の銘柄×日付特徴量 (1.13M rows) |
| `paper_live_*` | Paper Live mode 関連 |
| `replay_*` / `backtest_*` | Backtest / Replay |
| `evaluation_proposals` | F119 Evaluation Agent 提案 |
| `live_research_log` | F036 Live Research Log |
| `patterns` / `pattern_*` | Pattern Store / DEATH NOTE 等 |

---

## 各 table の freshness 確認方法 (= audit 結果)

audit script:

    .venv/bin/python -m scripts.jobs.audit_jquants_freshness \
      --db-path data/fire.staging.db \
      --db-label staging \
      --output-json /tmp/f286_data_r0_audit.json

staging 実 audit 結果 (2026-05-10 06:39 UTC):

    table                         min            max            dates  codes
    ───────────────────────────── ────────────── ────────────── ────── ──────
    market_prices_daily           2024-05-01     2026-05-01     489    4,452
    market_prices_intraday        2026-02-03     2026-05-01     60     479
    market_financials_v2          2016-05-09     2026-05-08     2,464  3,787
    market_listings               (snapshot)     (snapshot)     -      4,449
    index_data                    2026-02-05     2026-05-01     58     1
    announcements                 2026-04-30     2026-05-01     2      4
    research_derived_indicators   2026-05-01     2026-05-01     1      3,708
    research_watchlist_signals    2024-06-01     2026-05-09     28     910

### 観測 (= DATA-R1 / DATA-R2 で対応すべき gap)

| 観測 | 内容 |
|---|---|
| ★ daily_quotes 8 日遅延 | max=2026-05-01 (本日=2026-05-10)。J-Quants standard plan は前営業日まで取得可なので daily refresh で当日まで詰められる |
| ★ index_data 8 日遅延 | 同上、TOPIX が古い → market_regime classification も古い |
| ★ announcements は 1 件しか書かれていない | F101 fetch 単発実行のみで cron が無く、定期取得経路が未稼働 |
| ★ market_prices_intraday は 60 日分のみ | F284-c1 backfill 結果、当日分は未取得 |
| ★ research_derived_indicators dates=1 | 1 日分 (2026-05-01) のみ。R2-A の re-calc が未スケジュール化 |
| ★ research_watchlist_signals は 28 base_date のみ | sparse coverage、daily 自動生成が未稼働 |
| ★ market_listings updated_at 列なし | snapshot 全置換型、日次差分追跡が schema 上できない |

---

## daily refresh 対象 dataset

DATA-R1 で daily 自動更新する優先順:

### Tier-A (★ 必須、LINE 本番送信に直結)

1. `market_prices_daily` (= return_evaluation / regime の前提)
2. `index_data` (= regime label / TOPIX baseline)
3. `research_derived_indicators` (= signal 生成の前提)
4. `research_watchlist_signals` (= advisory 候補そのもの)

### Tier-B (★ 推奨、表現精度に効く)

5. `market_financials_v2` (= 上方修正 / 決算進捗)
6. `announcements` (= F101 適時開示、material event 検知)
7. `market_listings` (= 銘柄 master、上場 / 廃止反映、週次でも可)

### Tier-C (★ 当面は手動 backfill 維持)

8. `market_prices_intraday` (= F284 tier2 のみ、daily よりは select 銘柄で)

---

## backfill 済み範囲と daily 差分取得の境界

各 table の境界 (= 「ここから先は daily refresh で詰める」):

    table                         backfilled_through    daily refresh start
    ───────────────────────────── ───────────────────── ──────────────────────
    market_prices_daily           2026-05-01            2026-05-02 〜
    index_data                    2026-05-01            2026-05-02 〜
    market_financials_v2          2026-05-08            2026-05-09 〜
                                                        (= 当日早朝の差分)
    announcements                 2026-05-01            2026-05-02 〜
                                                        (= 当日 + 過去 1 日)
    market_listings               (snapshot 4449)        週次 全置換 (R-X)
    market_prices_intraday        2026-05-01            (Tier-C、当面手動)
    research_derived_indicators   2026-05-01            (= daily_quotes /
                                  (1 base_date のみ)     listings /
                                                         financials の差分
                                                         反映後に再計算)
    research_watchlist_signals    2026-05-09             (= derived_indicators
                                  (sparse 28 dates)      再計算後に factor
                                                         scoring + cap)

---

## idempotent upsert 方針

### 既存 upsert pattern (= 本 audit で確認済)

- `market_data/repository.py / save_listings`: INSERT OR REPLACE
  (PK code)
- `market_data/repository.py / save_daily_prices`: INSERT OR REPLACE
  (PK code, date)
- `market_data/repository.py / save_financials`: INSERT OR REPLACE
  (PK code, disclosure_date, type_of_document)
- `market_data/intraday.py / save_minute_bars`: INSERT OR REPLACE
  (PK code, dt, interval_min)
- `simulation/research_lane/signal_persistence.py / upsert_signals`:
  DELETE → INSERT (key: base_date, code, source_version)

### DATA-R1 で踏襲する原則

1. **冪等**: 同 key 再投入で値が更新されるが、行数は増えない
2. **PK / UNIQUE 制約必須**: schema 移行で必ず PK を持たせる
   (= F286-R1-B2 で finalcials に UNIQUE 制約を migrate_market_financials_pk.py
   で追加した経緯と整合)
3. **transaction**: 1 base_date 分は 1 transaction にまとめ、途中失敗で
   半端 state を作らない
4. **fetched_at / generated_at column**: 各 row に source-fetch 時刻を
   保持、debug 性を確保
5. **resume safe**: 失敗した base_date を SELECT で除外して continue

---

## staging から始める実装方針

### DATA-R1 (next task) の実装順序

1. **staging dry-run smoke** (= existing fetcher を staging に対して
   "差分のみ" で叩き、DB write を staging 限定にする 4 段 guard を
   必ず通す)
2. **staging 連続稼働** (= 1-3 営業日、daily refresh 結果が想定通り
   蓄積されることを確認)
3. **develop へ伝播** (= staging で検証済みの差分処理を develop に
   pin、production に伝播するのは更にその後)
4. **production には DATA-R3 で慎重に伝播**

### staging で使う 4 段 guard (= F286-R1-B 系で確立済み)

    1. argparse の --db に staging しか default で許可しない
    2. resolve_db_path で staging Path を強制
    3. caller config で `FIRE_ENV=staging` を要求
    4. write 実行前に DB Path を再確認、production / develop に向いて
       いれば SystemExit

---

## production / develop に触らない方針

本タスク (DATA-R0):

- ✅ audit script は **read-only mode=ro + PRAGMA query_only=ON**
- ✅ J-Quants API call なし、HTTP fetch なし
- ✅ token / api key 読まない
- ✅ smoke は staging のみ、develop / production には接続しない
- ✅ smoke 前後の develop.db / fire.db (production) last_modified
  完全 unchanged

DATA-R1 以降での原則:

- production / develop への daily refresh は **DATA-R3 以降**に分離
- DATA-R1 は staging 限定の smoke
- DATA-R2 は freshness gate の合否判定 (= 監視のみ、書き込みなし)
- production への伝播時は HQ 承認 + manual run のみ

---

## LINE 本番送信前に必要な freshness gate 条件

DATA-R2 で実装する gate:

### gate-1 (★ 必須): daily_quotes coverage

    market_prices_daily.max_date >= 直近営業日 (= JST)
    かつ
    distinct_codes >= 一定数 (= 4000 など、銘柄 master の 90% 以上)

未達: LINE 本番送信を refuse (= dry-run / preview のみ許可)。

### gate-2 (★ 必須): research_watchlist_signals coverage

    research_watchlist_signals (max base_date) >= 直近営業日
    かつ
    distinct_codes (当該 base_date) >= top_n (= 100)

未達: signal が古いので advisory 鮮度 NG → LINE 送信 refuse。

### gate-3 (★ 推奨): index_data coverage

    index_data.max_date >= 直近営業日

未達: market_regime classification が古い → advisory 解釈が
ズレる可能性 → warning + 手動レビュー必須。

### gate-4 (★ 推奨): research_derived_indicators 鮮度

    research_derived_indicators.max_date >= 直近営業日

未達: derived feature が古い → advisory 構築前に refresh 推奨。

### gate-5 (★ 緩い): financials_v2 / announcements 鮮度

    market_financials_v2.max_date >= 直近 5 営業日
    announcements.max_date >= 直近 5 営業日

未達: 単独では LINE 送信 refuse しない (= 緊急の決算発表があった
場合に手動更新できる導線を残すため、 warning のみ)。

---

## DATA-R1 / DATA-R2 で実装すべき項目

### DATA-R1: J-Quants Daily Refresh (staging 限定)

| 項目 | 内容 | 想定 commit |
|---|---|---|
| daily_quotes 差分取得 | `market_data/historical.py` を再利用、--from = max_date+1、--to = 直近営業日 | feat / chore / test |
| index_data 差分取得 | `market_data/index_fetcher.py` 再利用、TOPIX / N225 | chore / test |
| financials_v2 daily | `backfill_market_financials.py` を直近 N 営業日に絞り込み + sample limit 解除 | chore / test |
| announcements daily | `scripts/jobs/fetch_announcements.py` 再利用 + cron 化想定 | chore / test |
| research_derived_indicators daily | R2-A の compute_derived_indicators.py を staging で再実行 | chore / test |
| research_watchlist_signals daily | R2-D の run_research_watchlist_ranker.py を staging で再実行 | chore / test |
| daily refresh runner | `scripts/jobs/run_daily_refresh.py` 新規、上記を順番に呼ぶ | chore / test |

### DATA-R2: Freshness Gate (LINE 送信前検査)

| 項目 | 内容 | 想定 commit |
|---|---|---|
| gate evaluator | `agents/freshness_gate.py` 純関数群、audit JSON を入力に gate-1..5 判定 | feat / test |
| F062-R2 接続 | F062 LINE 本番送信前に gate を呼び、未達なら refuse | feat / chore / test |
| 監視 cron | `scripts/jobs/run_freshness_gate.py` を 毎朝 staging で走らせ、未達なら LINE 警告 (= F236 既存導線) | chore / test |

### 優先度

- DATA-R1 daily_quotes 差分取得が最優先 (= advisory 鮮度の前提)
- DATA-R2 gate-1 / gate-2 が次優先
- DATA-R3 (production 伝播) は DATA-R1 / R2 が staging で 1 週間
  以上稼働してから検討

---

## 実装ファイル一覧 (本 DATA-R0 タスク)

| 種別 | ファイル | 行数 |
|---|---|---|
| chore | `scripts/jobs/audit_jquants_freshness.py` | 244 |
| test | `tests/scripts/jobs/test_audit_jquants_freshness.py` | 16 PASS |

---

## smoke 結果 (staging read-only audit)

実行コマンド:

    .venv/bin/python -m scripts.jobs.audit_jquants_freshness \
      --db-path data/fire.staging.db \
      --db-label staging \
      --output-json /tmp/f286_data_r0_audit.json

artifact:

    /tmp/f286_data_r0_audit.json (4 KB)

table 別 freshness (上の表参照)。

DB unchanged 検証:

    target              before                       after
    fire.staging.db     2026-05-09T22:40:35.385124   unchanged ✅
    fire.develop.db     May  7 18:14:26              unchanged ✅
    fire.db             May  7 16:12:38              unchanged ✅

---

## 安全要件遵守チェックリスト

- ✅ DB write 0 件 (URI mode=ro + PRAGMA query_only=ON、INSERT が
  sqlite3.OperationalError で refuse、CREATE TABLE も refuse、
  tests で検証)
- ✅ J-Quants API call なし (= HTTP request 不発生、token/api_key
  読み込みなし、tests で source 検証)
- ✅ production / develop / staging DB に書き込みなし
- ✅ smoke は staging のみ read-only access、develop / production
  に一切接続せず、DB last_modified 完全 unchanged
- ✅ LINE 本番送信なし (LineBotClient / send_text を import / 呼び
  出さない、tests で source 検証)
- ✅ token / channel secret / .env / dotenv 読み込みなし
- ✅ order / broker / 楽天証券 / Computer Use / Playwright /
  Selenium / subprocess 全て構造的非接続 (tests で source 検証)
- ✅ scripts/seed_pattern_layer1.py 未接触
- ✅ simulation/research_lane/historical_indicators.py 未接触
- ✅ unrelated modified を stage / commit しない (3 commit すべて
  `git add <specific files>` で個別 stage)
- ✅ TODO Excel 未更新

---

## tests 結果

- 新規 16 PASS
- フル pytest **2,955 PASS** (= 2,939 baseline + 16 新規)
- regression 0 件失敗

---

## Codex pre-commit 結果

(commit 順に追記予定)

---

## --no-verify 未使用確認

✅ 全 commit で `--no-verify` flag 不使用

---

## 次タスク提案

### 第一候補: DATA-R1 J-Quants Daily Refresh (staging 限定)

- daily_quotes 差分取得 + index_data 差分取得 + research_derived
  / signals 再計算を staging で連結
- 4 段 guard で write を staging に限定
- 1-3 営業日 dry-run で挙動確認

### 第二候補: DATA-R2 Freshness Gate

- gate-1 (daily_quotes) / gate-2 (signals) / gate-3 (index_data)
  / gate-4 (derived) / gate-5 (financials/announcements) の純関数群
- F062-R2 LINE 送信前に gate を強制
- 毎朝の audit + 警告通知 (LINE F236 既存導線)

### 第三候補: F062-R2 LINE Send dry-run / production 分離

- DATA-R1 / R2 完了後に着手
- gate 通過時のみ本番送信、未通過なら refuse

優先度: 1 > 2 > 3 (LINE 送信を急ぐ前に freshness を整える)

---

## 関連参照

- 02_todo/F062_R1_line_advisory_notification_template.md (前段)
- 02_todo/F111_R4_multi_base_date_f119_insights_smoke.md
- 02_todo/F286_R2_H_orthogonal_cuts.md
- 02_todo/F119_interpretation_evaluation.md
- log.md milestone (本タスク完了時に追記)
