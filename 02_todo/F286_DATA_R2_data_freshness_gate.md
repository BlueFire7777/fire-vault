# F286-DATA-R2 Data Freshness Gate / Stale Data Warning

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: F286-DATA-R0 (cf4cf2b / 90bb152) + DATA-R1 (b5f7ac2 /
> f6d4836 / 16b0d8e) + DATA-R1.1 (420ad90 / b8d8a22)
> **Mode**: 完全 read-only (URI mode=ro + PRAGMA query_only=ON)
> **Result**: ★★★ LINE 本番送信前 gate-1..5 を実装。
> safe-by-default で warning でも `line_send_allowed=False` を返し、
> 鮮度 warning 状態の誤送信事故を構造的に防止。staging smoke で
> 期待通り `refuse` 判定が出ることを確認。★★★

---

## タスク名

F286-DATA-R2 Data Freshness Gate / Stale Data Warning

---

## 背景

DATA-R0 で freshness 設計、DATA-R1 で daily refresh runner、
DATA-R1.1 で 5-10 銘柄 staging write smoke を成功させた。
ただし全銘柄 coverage はまだ不足。LINE 本番 Advisory 送信を許可する
前段として、現状を**正しく refuse / warning と判定できる gate**を作る。

---

## 実装ファイル一覧

| 種別 | ファイル | 行数 |
|---|---|---|
| feat | `agents/data_freshness_gate.py` | 731 |
| chore (runner) | `scripts/jobs/run_data_freshness_gate.py` | 351 |
| test | `tests/agents/test_data_freshness_gate.py` | 27 PASS |
| test | `tests/scripts/jobs/test_run_data_freshness_gate.py` | 21 PASS |

---

## commit hash 一覧

| commit | type | 内容 |
|---|---|---|
| `e8c80f8` | feat | F286-DATA-R2: add data freshness gate |
| `11c31ba` | chore | F286-DATA-R2: add freshness gate runner |
| `56baf6e` | test | F286-DATA-R2: add freshness gate tests |
| (本 commit) | docs | F286-DATA-R2: vault |
| (次 commit) | docs | F286-DATA-R2: log milestone |

---

## gate 仕様

### 5 段の gate

| gate | level | 判定 |
|---|---|---|
| gate-1 prices | required | market_prices_daily.max_date が直近営業日以内 + max_date での distinct_codes >= 4000 |
| gate-2 signals | required | research_watchlist_signals.max_base_date が直近営業日以内 + max_base_date での distinct_codes >= top_n |
| gate-3 index | recommended | index_data.max_date が直近営業日以内 |
| gate-4 derived | recommended | research_derived_indicators.max_base_date が直近営業日以内 |
| gate-5 other | soft | market_financials_v2 / announcements / market_listings の鮮度 + 件数 |

### overall_status と line_send_allowed

| overall_status | 条件 | line_send_allowed (default) | --allow-warning 指定時 |
|---|---|---|---|
| pass | 全 gate pass | True | True |
| warning | 推奨 gate ≥ 1 件 warning | False (= safe-by-default) | True |
| refuse | 必須 gate ≥ 1 件 refuse | False | False (絶対) |

★ Codex CRITICAL (F286-DATA-R2) 対応として safe-by-default に再設計。

### exit code

| code | 意味 |
|---|---|
| 0 | pass |
| 2 | refused (= 設定不正 / db_path 不在 / 不正 as_of_date) |
| 3 | warning (= 推奨 gate 未達) |
| 4 | refuse (= 必須 gate 未達) |

### threshold 一覧 (default)

    prices_min_distinct_codes_at_max_date    : 4000
    prices_max_date_lag_business_days        : 1
    signals_min_distinct_codes_at_max_date   : 100 (= top_n、CLI で変更可)
    signals_max_date_lag_business_days       : 1
    index_max_date_lag_business_days         : 1
    derived_max_date_lag_business_days       : 1
    financials_max_date_lag_business_days    : 5
    announcements_max_date_lag_business_days : 5
    listings_min_codes                       : 3500

---

## 営業日判定 (= JST 暦)

`latest_business_day_on_or_before(d)`:
- 月-金 (Mon=0..Fri=4) を平日とみなし、土日は前金曜にスライド
- 祝日カレンダー非対応 (= 推奨範囲内、より厳密には別 module で対応)

`business_day_lag(expected, actual)`:
- expected 日以前を 1 日ずつ巻き戻し、平日のみカウント
- actual >= expected なら 0
- actual が None なら None

---

## smoke 結果 (staging read-only / 2026-05-08 / top_n=100)

実行コマンド:

    .venv/bin/python -m scripts.jobs.run_data_freshness_gate \
      --db-path data/fire.staging.db --db-label staging \
      --as-of-date 2026-05-08 --top-n 100 \
      --output-json /tmp/f286_data_r2_gate.json \
      --output-text /tmp/f286_data_r2_gate.txt \
      --completion-report /tmp/f286_data_r2_completion_report.txt

### 結果

| gate | level | status | 詳細 |
|---|---|---|---|
| gate-1 prices | required | **refuse** | max_date=2026-05-08 の distinct_codes=10 < 4000 (= DATA-R1.1 で 10 銘柄のみ update) |
| gate-2 signals | required | **pass** | max_base_date=2026-05-09 / lag=0 / distinct_codes=109 |
| gate-3 index | recommended | **pass** | max_date=2026-05-08 / lag=0 |
| gate-4 derived | recommended | **warning** | max_base_date=2026-05-01 / lag=5 営業日 > 1 |
| gate-5 other | soft | **pass** | financials / announcements / listings いずれも OK |

- overall_status: **refuse**
- line_send_allowed: **False**
- exit code: **4**

### refuse 理由 (reasons に出力)

    [gate-1-prices/required/refuse] market_prices_daily の coverage
    不足: max_date=2026-05-08 の distinct_codes=10 < 4000

    [gate-4-derived/recommended/warning] derived_indicators 鮮度
    不足: max_base_date=2026-05-01 / lag=5 営業日 > 1

### LINE 送信可否

- `line_send_allowed: False` ★
- gate-1 prices が必須 refuse のため、`--allow-warning` を指定しても
  False を維持 (= 必須 refuse は絶対に送信不可、test で検証済み)

---

## DB unchanged 確認

| target | before | after |
|---|---|---|
| fire.staging.db | 2026-05-10 16:23:40 | unchanged ✅ |
| fire.develop.db | 2026-05-07 18:14:26 | unchanged ✅ |
| fire.db | 2026-05-07 16:12:38 | unchanged ✅ |

★ runner / helper module ともに sqlite3.connect は URI mode=ro のみ、
   PRAGMA query_only=ON で write SQL は構造的に refuse される。

---

## 安全要件遵守

### DB write なし / production / develop / staging 無触

- ✅ open_readonly_connection が URI mode=ro + PRAGMA query_only=ON
- ✅ INSERT / UPDATE / DELETE / DROP / CREATE TABLE が
  sqlite3.OperationalError で refuse されることを test で検証
- ✅ smoke 前後で 3 DB の last_modified 完全 unchanged

### LINE 本番送信なし

- ✅ source 文字列で `LineBotClient(` / `linebot` /
  `from notifications.line_bot` / `.send_text(` / `.push_message(`
  / `.reply_message(` の実 import / call なし (test で検証)
- ✅ `line_send_allowed: bool` を結果として返すのみ
- ✅ argparse help に `--send` / `--line` / `--token` / `--api-key`
  option 不在
- ✅ Codex CRITICAL 対応で safe-by-default (warning でも False)

### J-Quants API call なし

- ✅ source 文字列で `JQuantsClient(` / `from market_data.client`
  の実 import / call なし
- ✅ HTTP request 不発生

### token / channel_secret / .env / dotenv 読み込みなし

- ✅ runner / helper module 自身は os.environ / load_dotenv /
  JQUANTS_API_KEY / LINE_CHANNEL_TOKEN を直接読まない
- ✅ test で source 文字列に実 read pattern (= os.environ['JQUANTS'
  / os.getenv("LINE 等) が無いことを検証

### order / broker / 楽天証券 / Computer Use 未接続

- ✅ source 文字列で `TradeOrder(` / `.place_order(` / `.send_order(`
  / `.submit_order(` の実 import / call なし
- ✅ `playwright` / `selenium` / `subprocess.run` 不在
- ✅ argparse help に `--broker` / `--rakuten` / `--order` /
  `--auto-order` / `--computer-use` / `--playwright` 不在

### unrelated modified 未接触

- ✅ `scripts/seed_pattern_layer1.py` 未接触
- ✅ `simulation/research_lane/historical_indicators.py` 未接触
- ✅ 全 commit `git add <specific files>` で個別 stage

### TODO Excel 未更新

✅ Google Sheets TODO 管理表は本タスクで更新していない

---

## tests 結果

### 新規 48 PASS

    tests/agents/test_data_freshness_gate.py (27 PASS)
      - TestBusinessDayHelpers (7)
      - TestGate1Prices / Gate2Signals (8)
      - TestGate3Index / Gate4Derived / Gate5Other (6)
      - TestEvaluateAll (6) ★ safe-by-default 3 ケース含む
      - TestModuleSourceSafety (3)

    tests/scripts/jobs/test_run_data_freshness_gate.py (21 PASS)
      - TestParseArgs (4)
      - TestArgParserSafety (2、--allow-warning option 検証含む)
      - TestMain (6) ★ default warning blocks send / allow-warning
                       permits send 含む
      - TestOutputs (2)
      - TestRunnerSourceSafety (3)

### regression

- F286 / F119 / F111 / F062 / F100 / F284 全テスト 0 件回帰
- フル pytest **3,079 PASS** (= 3,031 baseline + 48 新規)

---

## Codex pre-commit 結果

| commit | Codex 判定 |
|---|---|
| e8c80f8 (feat) | ✅ OK (★ CRITICAL #1 対応版で再 commit) |
| 11c31ba (chore runner) | ✅ OK |
| 56baf6e (test) | ✅ OK |

### CRITICAL 1 件と修正

CRITICAL: `evaluate_all()` が `overall_status == "warning"` かつ
`strict=False` のときに `line_send_allowed=True` を返すが、ファイル
冒頭の仕様は「`overall_status == "pass"` のときだけ True、それ以外
False」と矛盾。鮮度 warning 状態で本番 Advisory LINE 送信が許可される
ため通知配信ゲートとして危険。

→ 対応 (safe-by-default に再設計):
- `strict` 引数を `allow_warning` に rename、default False
- default は `pass` のみ True、warning でも False (= 安全側に倒す)
- caller が明示的に `allow_warning=True` を渡したときのみ warning
  を許可
- 必須 gate refuse 時は allow_warning に関わらず False を維持
- runner CLI も `--strict` を `--allow-warning` に rename
- test 3 ケース追加 (default warning blocks / allow_warning
  permits / refuse never allowed)

✅ 全 3 commit で Codex review 通過、CRITICAL 1 件即修正

---

## --no-verify 未使用確認

✅ 全 3 commit で `--no-verify` flag 不使用
✅ Codex usage limit / rate limit / auth error なし

---

## 次タスク提案

### 第一候補: F062-R2 LINE Send dry-run / production 分離

DATA-R2 gate で line_send_allowed: bool が確定したので、
F062-R2 で LINE 本番送信導線に gate を組み込む。
- `--send` 指定時は gate を必ず通過、refuse / warning なら refuse
- `--allow-warning` で warning 許可

### 第二候補: DATA-R1.2 銘柄絞り込み拡大 (50 → 500 → 全件)

DATA-R1.1 の `--symbols-limit` を 50 / 200 / 500 に拡大して
staging で daily refresh を継続実行、gate-1 prices coverage を
4000+ 銘柄まで埋める。

### 第三候補: persist runner 統合 (derived / signals)

derived / signals を本 runner から薄い wrapper 経由で呼び、
1 invocation で prices → index → derived → signals を完結させる。

優先度: 1 > 2 > 3 (LINE 送信導線が最優先、gate を本番接続)

---

## 関連参照

- 02_todo/F286_DATA_R0_jquants_pipeline_audit_freshness_design.md
  (DATA-R0、freshness 設計)
- 02_todo/F286_DATA_R1_jquants_daily_refresh.md (DATA-R1 daily
  refresh runner)
- 02_todo/F286_DATA_R1_1_jquants_limited_write_smoke.md
  (DATA-R1.1 limited write smoke)
- 02_todo/F062_R1_line_advisory_notification_template.md
- log.md milestone (本タスク完了時に追記)
