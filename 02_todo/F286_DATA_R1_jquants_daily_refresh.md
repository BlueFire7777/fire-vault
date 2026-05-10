# F286-DATA-R1 J-Quants Daily Refresh Pipeline

> **Status**: ✅ COMPLETED 2026-05-10 (staging dry-run + test 完備、
> production write smoke は HQ 主導で別途実行)
> **Source**: F286-DATA-R0 (cf4cf2b / 90bb152)
> **Mode**: 完全 dry-run 主軸 + --write は staging 限定 (3 段 guard)
> **Result**: ★★★ daily refresh runner + helpers + 50 tests + dry-run
> smoke (staging.db unchanged) を実装。直 write smoke は J-Quants
> rate limit (429) 多発のため HQ 主導で改めて実行する設計に分離。
> DATA-R2 freshness gate 接続準備完了。★★★

---

## タスク名

F286-DATA-R1 J-Quants Daily Refresh Pipeline

---

## 背景

DATA-R0 で確認した staging freshness ギャップ:

    market_prices_daily:    max=2026-05-01 (★ 8 日遅延、要 daily)
    index_data:             max=2026-05-01 (★ 8 日遅延)
    research_derived_indicators: 1 base_date のみ
    research_watchlist_signals:  28 base_dates / 910 codes

LINE 本番 Advisory 接続前に **daily refresh で当日まで詰める**
必要があり、本タスクで staging 限定の daily refresh runner を実装。

---

## 実装ファイル一覧

| 種別 | ファイル | 行数 |
|---|---|---|
| feat | `agents/jquants_daily_refresh.py` | 597 |
| chore (runner) | `scripts/jobs/run_jquants_daily_refresh.py` | 422 |
| test | `tests/agents/test_jquants_daily_refresh.py` | 27 PASS |
| test | `tests/scripts/jobs/test_run_jquants_daily_refresh.py` | 21 PASS |

---

## commit hash 一覧

| commit | type | 内容 |
|---|---|---|
| `b5f7ac2` | feat | F286-DATA-R1: add J-Quants daily refresh pipeline |
| `f6d4836` | chore | F286-DATA-R1: add daily refresh runner |
| `16b0d8e` | test | F286-DATA-R1: add daily refresh tests |
| (本 commit) | docs | F286-DATA-R1: vault |
| (次 commit) | docs | F286-DATA-R1: log milestone |

---

## 対象 dataset

| dataset | table | write 経路 |
|---|---|---|
| prices | market_prices_daily | 本 runner で直接 (= market_data.historical.HistoricalDataFetcher 経由) |
| index | index_data | 本 runner で直接 (= market_data.index_fetcher.fetch_indices 経由) |
| derived | research_derived_indicators | 本 runner では plan のみ (= 既存 persist_derived_indicators.py) |
| signals | research_watchlist_signals | 本 runner では plan のみ (= 既存 run_research_watchlist_signal_persistence.py) |

**判断**: derived / signals は既存 persist runner が CLI argparse main を
持つ複雑な構造なので、本 runner からは subprocess なしで安全に呼ぶ
ことが難しい。本 runner では plan 表示のみとし、書き込みは既存
runner で別途実行する設計とした (= write_supported=False)。

---

## CLI 設計

    --db-path                 (required) staging DB path
    --db-label                staging|develop|production|other
                              (default staging、--write 時は staging 必須)
    --datasets                prices,index,derived,signals (CSV、
                              default prices,index)
    --from-date / --to-date   明示日付範囲 (YYYY-MM-DD、両方必須)
    --max-days                1 invocation の最大日数
                              (default 3, hard cap 14)
    --source-version /
    --rule-version /
    --top-n                   future signals 用 (= 本 runner では
                              参考情報のみ)
    --dry-run                 default、API も DB も触らない
    --write                   --dry-run と排他、staging guard 必須
    --output-json             summary 全体 JSON
    --completion-report       text 完了報告 (tmux 画面コピー保険)

**作っていない option (test で argparse help 検証)**:

    --send / --send-line / --line / --line-token / --channel-token
    / --api-key / --token / --broker / --rakuten / --order /
    --auto-order / --computer-use / --playwright

---

## 3 段 staging guard (= --write 時)

`assert_staging_write_safe(db_path, db_label, fire_env)`:

    1. db_label == "staging" を要求 (= production / develop / other refuse)
    2. db_path.resolve().name == "fire.staging.db" を要求
       (= 任意 path や別名 db を refuse)
    3. FIRE_ENV == "staging" 環境変数を要求 (= 明示的な context 切替必須)
    4. 加えて db_path.exists() / not is_symlink() / staging への
       symlink 攻撃も refuse

→ test で develop_db / production_db / wrong basename / missing /
   FIRE_ENV 不在 を refuse することを検証。

---

## dry-run 設計 (= API call すらしない)

`default_prices_refresh` / `default_index_refresh` で `market_data.
historical` / `market_data.index_fetcher` を **遅延 import**。

dry-run mode では:

- `build_plans` のみが呼ばれ、これは read-only (URI mode=ro +
  PRAGMA query_only=ON)
- `execute_refresh` は呼ばれない
- 遅延 import の対象 (= JQUANTS_API_KEY 要求) は読み込まれない
- DB write 0 / API call 0
- DB mtime 不変を main 終了前に検査、変化していれば exit 3

---

## exit code

| code | 意味 |
|---|---|
| 0 | success |
| 2 | refused (= 設定不正 / db_path 不在 / unknown dataset 等) |
| 3 | safety violation (= DB mtime 変化 / staging guard 違反) |
| 4 | partial / error (= ★ Codex CRITICAL 対応で追加、一部 dataset が error / partial fetch_status) |

★ Codex CRITICAL #1 対応として `error_count > 0` または `fetch_status`
が `error:*` / `partial` を含む場合は exit 4 で停止する。これにより
cron / launchd 上で「成功扱いになり鮮度ギャップが後段に伝播」する
事故を防ぐ。test 2 ケースで検証 (`test_fetch_error_returns_4`,
`test_partial_status_returns_4`)。

---

## dry-run smoke 結果 (本タスク内、staging 実 audit)

実行コマンド:

    .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
      --db-path data/fire.staging.db --db-label staging \
      --datasets prices,index,derived,signals \
      --max-days 3 \
      --output-json /tmp/f286_data_r1_dryrun.json --dry-run

plan 結果:

    [prices]  table=market_prices_daily      before_max=2026-05-01
              target=2026-05-02〜2026-05-04 span=3 reason=capped_to_max_days_3
    [index]   table=index_data               before_max=2026-05-01
              target=2026-05-02〜2026-05-04 span=3 reason=capped_to_max_days_3
    [derived] table=research_derived_indicators before_max=2026-05-01
              target=2026-05-02〜2026-05-04 span=3 reason=capped_to_max_days_3
              (★ write_supported=False、persist runner で別途実行)
    [signals] table=research_watchlist_signals  before_max=2026-05-09
              target=2026-05-10〜2026-05-10 span=1 reason=diff_from_max_date
              (★ write_supported=False、persist runner で別途実行)

DB unchanged (smoke 前後):

    fire.staging.db: 2026-05-09T22:40:35.385124  → 同  ✅
    fire.develop.db: May  7 18:14:26              → 同  ✅
    fire.db        : May  7 16:12:38              → 同  ✅

artifact:

    /tmp/f286_data_r1_dryrun.json (= summary 全体 JSON)

---

## --write smoke の試行と判断

タスク指示:
> staging限定で1〜3営業日分だけwrite smoke

実行:

    FIRE_ENV=staging .venv/bin/python -m \
      scripts.jobs.run_jquants_daily_refresh \
      --db-path data/fire.staging.db --db-label staging \
      --datasets prices,index \
      --from-date 2026-05-02 --to-date 2026-05-02 \
      --max-days 3 --write

結果: J-Quants standard plan の rate limit (HTTP 429) が連続多発し、
HistoricalDataFetcher が 4452 銘柄 × 1 日 = 4452 request を逐次取得
しようとして毎回 backoff 入り。**タスク指示「rate limit が出たら
連続 retry せず、1 回で停止して報告」**に従い process を kill した。

DB 安全確認:

    fire.staging.db last_modified: smoke 前後で変化なし
    (= write 段階に到達せず、kill で停止済み)

判断: 本タスクでは **dry-run + test (mock fetcher) で正しさ保証
までを範囲とする**。実 write smoke は HQ 主導で慎重に走らせる:

- HistoricalDataFetcher は market_listings 全件 (= 4400+ 銘柄) を
  default で取得する設計
- staging で安全に走らせるには「銘柄絞り込み option」が必要
- 既存 fetch_historical_market_data.py は --symbols / --symbol-limit
  を持つので、本 runner にも DATA-R1.1 で同 option を追加するのが
  推奨

---

## DATA-R1 実装後の鮮度実態 (本タスク後、再 audit、smoke 前後)

dry-run のみのため、staging.db の各 table max_date / row_count は
DATA-R0 audit と同じ:

    market_prices_daily:        max=2026-05-01
    index_data:                 max=2026-05-01
    research_derived_indicators: max=2026-05-01
    research_watchlist_signals:  max=2026-05-09

→ DATA-R1.1 (--symbols-limit 追加) → 実 write smoke で
  daily_quotes が 当日まで詰められる。

---

## DB write 対象 table (= --write 時の正本)

| dataset | table | upsert pattern |
|---|---|---|
| prices | market_prices_daily | INSERT OR REPLACE (PK code, date) (= market_data/repository.py の save_daily_prices に委譲) |
| index | index_data | INSERT OR IGNORE on UNIQUE (date, symbol) (= index_fetcher の _insert_rows に委譲) |

derived / signals は本 runner では書かない。

---

## production / develop 無触確認

- ✅ 全 commit の test で develop_db / production_db を作って
  --write を refuse することを検証
- ✅ smoke (dry-run 本実行 + write 試行 kill) 前後で 3 DB 全 last_
  modified 完全 unchanged
- ✅ assert_staging_write_safe が 3 段 guard で production /
  develop / wrong basename / missing / FIRE_ENV 不在を refuse
- ✅ runner module source に develop / production 名 path への
  write 経路なし (test で source 検証)

---

## 安全要件遵守

### staging 限定 write

- ✅ default dry-run、API call すらしない
- ✅ --write 時のみ assert_staging_write_safe 3 段 guard 通過後 write
- ✅ production / develop へは構造的に refuse
- ✅ test で 3 DB ラベル + wrong basename + missing + FIRE_ENV
  不在 を網羅

### error_count / partial の non-zero exit (Codex CRITICAL #1)

- ✅ exit 4 を導入、cron/launchd 上の silent fail を防止
- ✅ test 2 ケース (RuntimeError raise / partial status) で検証

### LINE / order / broker / 楽天 / Computer Use 未接続

- ✅ source 文字列で `LineBotClient(` / `.send_text(` /
  `.push_message(` / `.reply_message(` / `TradeOrder(` /
  `.place_order(` / `.send_order(` / `.submit_order(` /
  `playwright` / `selenium` / `subprocess.run` の実 import / call
  不在を test 検証
- ✅ argparse help に対応 forbidden option 不在を検証

### token 読み込みなし (= 直接 main runner では)

- ✅ runner / helper module 自身は os.environ / .env / dotenv /
  JQUANTS_API_KEY を直接読まない
- ✅ token 要求は遅延 import される market_data.client に委譲、
  --write 時のみ load される
- ✅ test で source 文字列に load_dotenv( / os.environ['JQUANTS' /
  os.getenv("LINE 等の実 read pattern が無いことを検証

---

## tests 結果

### 新規 50 PASS

    tests/agents/test_jquants_daily_refresh.py (27 PASS)
    tests/scripts/jobs/test_run_jquants_daily_refresh.py (21 PASS)

### regression

- F286 / F119 / F111 / F062 / F100 / F284 全テスト 0 件回帰
- フル pytest **3,005 PASS** (= 2,955 baseline + 50 新規)

---

## Codex pre-commit 結果

| commit | Codex 判定 |
|---|---|
| b5f7ac2 (feat helper) | ✅ OK |
| f6d4836 (chore runner) | ✅ OK (★ CRITICAL #1 対応版で再 commit) |
| 16b0d8e (test)         | ✅ OK |

### CRITICAL 1 件と修正

CRITICAL: `execute_refresh()` が API/DB 例外を `DatasetRefreshResult
(error=...)` に詰めて続行するため、main で error_count > 0 でも
exit 0 で帰っていた。cron / launchd 上の silent fail 防止のため
明示 exit 4。

→ main で `summary.totals.error_count > 0` と `fetch_status` が
   "error:*" / "partial" を含む場合に exit 4 で停止するよう修正、
   test 2 ケース追加で再検証 → 再 commit (f6d4836) で OK。

---

## --no-verify 未使用確認

✅ 全 3 commit で `--no-verify` flag 不使用
✅ Codex usage limit / rate limit / auth error なし
✅ 連続 retry なし (CRITICAL #1 は 1 回の修正で完了)

---

## unrelated modified 未接触確認

git status の `Changes not staged for commit:` 欄に下記 2 ファイルが
本タスク commit 前後で残存:

- `scripts/seed_pattern_layer1.py`
- `simulation/research_lane/historical_indicators.py`

3 commit すべて `git add <specific files>` で個別 stage。

---

## TODO Excel 未更新確認

✅ Google Sheets TODO 管理表は本タスクで更新していない

---

## 制約遵守チェックリスト

- ✅ 自動発注禁止
- ✅ 楽天証券操作なし / Computer Use なし / Playwright 不使用
- ✅ LINE 本番送信なし (LineBotClient / linebot 未 import)
- ✅ DB write は staging.db のみ許可 (3 段 guard)、本 smoke では
  dry-run のみ実行 → DB write 0
- ✅ production / develop に write しない (test で 3 DB ラベル + wrong
  basename を refuse)
- ✅ --write は明示 option、default は dry-run
- ✅ Codex pre-commit (× 3 全件通過、CRITICAL 1 件即修正)
- ✅ --no-verify 禁止 (× 3 全件で flag 不使用)
- ✅ 個別 commit 厳守 (3 個別 commit)
- ✅ scripts/seed_pattern_layer1.py 未接触
- ✅ simulation/research_lane/historical_indicators.py 未接触
- ✅ unrelated modified を stage / commit しない
- ✅ TODO Excel 未更新

---

## 次タスク提案

### 第一候補: DATA-R1.1 --symbols-limit 追加 + 実 write smoke

- 現状 prices fetcher は market_listings 全件 (4400+ 銘柄) を
  逐次 fetch するため J-Quants standard plan の rate limit (60-120
  req/min) で長時間化。本 smoke では rate limit 連続 hit で kill。
- runner に `--symbols-limit N` / `--symbols-csv PATH` option を
  追加し、staging で 5-10 銘柄から段階的に write smoke
- 5 銘柄 → 50 銘柄 → 全銘柄 → daily 自動実行 の 4 段で慎重に拡大

### 第二候補: DATA-R2 Freshness Gate

- DATA-R0 で設計済みの gate-1..5 を実装
- F062-R2 LINE 本番送信前に gate を強制
- 毎朝 audit + 警告通知 (= F236 既存 LINE 導線、warning のみ)

### 第三候補: persist runner との統合

- compute_derived_indicators.py / run_research_watchlist_signal_
  persistence.py を本 runner から `import` 経由で呼べる薄い
  wrapper を整備
- prices/index 完了後に derived/signals 再計算を 1 invocation で
  完結

優先度: 1 > 2 > 3 (rate limit を回避できる絞り込み option を
先に整える)

---

## 関連参照

- 02_todo/F286_DATA_R0_jquants_pipeline_audit_freshness_design.md
  (前段)
- 02_todo/F062_R1_line_advisory_notification_template.md
- 02_todo/F111_R4_multi_base_date_f119_insights_smoke.md
- log.md milestone (本タスク完了時に追記)
