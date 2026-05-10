# F286-DATA-R1.1 J-Quants Limited Write Smoke / Rate Limit Safe Refresh

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: F286-DATA-R1 (b5f7ac2 / f6d4836 / 16b0d8e)
> **Mode**: staging-only write (3 段 guard) + 銘柄絞り込み
> + rate limit safe (例外 / 戻り値 両経路で即停止)
> **Result**: ★★★ --symbols-limit 5 / 10 で staging write smoke
> 完全成功 (rows 増分 +5 → +11 / max_date 2026-05-01 → 2026-05-08)。
> rate limit 0 / production-develop unchanged。Codex CRITICAL を
> 例外経路 + 戻り値経路の両方で対処。★★★

---

## タスク名

F286-DATA-R1.1 J-Quants Limited Write Smoke / Rate Limit Safe Refresh

---

## 背景

DATA-R1 で daily refresh runner は実装したが、staging write smoke
時に J-Quants standard plan の HTTP 429 rate limit が連続多発し、
process kill で安全停止 → DB write 段階に到達せず終了。

DATA-R1.1 では:

- 銘柄を絞れる `--symbols-limit` / `--symbols-csv` を追加
- 5 銘柄 / 10 銘柄から段階的に staging write smoke を成功させる
- rate limit 検出時の連続 retry を構造的に防ぐ
- staging.db に実 row を入れて max_date を進める

---

## 追加 option 一覧

| option | 型 | default | 役割 |
|---|---|---|---|
| `--symbols-limit` | int | None | market_listings 先頭 N 件を採用 (hard cap 4500) |
| `--symbols-csv` | Path | None | CSV から code list を読み込み (1 列 / header 'code' / # コメント / 重複 dedupe / TSV 対応) |
| `--sleep-seconds` | float | 0.0 | dataset 間 sleep (= rate limit 緩和、ok 時のみ次へ進む前に挿入) |

排他: `--symbols-limit` と `--symbols-csv` は同時指定不可。

未指定時の挙動: prices fetcher の既存 default (= market_listings 全件)
だが、本 runner では rate limit 観点で **常に `--symbols-limit` /
`--symbols-csv` を指定推奨**。

---

## 実装ファイル一覧

| 種別 | ファイル | 主な変更 |
|---|---|---|
| feat | `agents/jquants_daily_refresh.py` | `_load_symbols_from_csv` / `resolve_target_symbols` 追加、`execute_refresh` に `prices_symbols` / `sleep_seconds` 引数追加、★ rate limit safe break (例外 + 戻り値両経路)、`build_summary` に symbols 情報追加 (= F286-DATA-R1.1-v1) |
| chore | `scripts/jobs/run_jquants_daily_refresh.py` | `--symbols-limit` / `--symbols-csv` / `--sleep-seconds` CLI 追加、parse_args で排他 / 0 以下 / 負 sleep refuse、main で resolve_target_symbols + execute_refresh 伝播、bad_status 判定に `aborted_after_error` / `aborted_after_partial` / `unknown` 追加 |
| test | `tests/agents/test_jquants_daily_refresh.py` | +20 新規 (CSV / resolve / Exception 後 abort) |
| test | `tests/scripts/jobs/test_run_jquants_daily_refresh.py` | +6 新規 (parse_args validation / main symbols 経路 / 戻り値 partial 後 abort) |

---

## commit hash 一覧

| commit | type | 内容 |
|---|---|---|
| `420ad90` | feat | F286-DATA-R1.1: add limited J-Quants refresh controls |
| `b8d8a22` | test | F286-DATA-R1.1: add limited refresh tests |
| (本 commit) | docs | F286-DATA-R1.1: vault |
| (次 commit) | docs | F286-DATA-R1.1: log milestone |

---

## rate limit safe 実装 (= Codex CRITICAL 2 件と修正)

### CRITICAL #1: 例外経路の継続 → 修正済

指摘: `execute_refresh` で前 dataset の Exception を catch して
`error` 詰めにしていたが、後続 plan を `continue` で続行していた。
J-Quants 429 retry exhaustion 後も次 API call を発行しうる。

修正: catch 後に `current_idx = plans.index(plan)` で残り plan を
全て `aborted_after_error` status で記録し `break` する。
fake_index に `AssertionError("must not be called")` を入れた
test で「呼ばれていない」ことを保証。

### CRITICAL #2: 戻り値経路の継続 → 修正済

指摘: `prices_refresh` が `error_msg` / `failed_symbols` 入り dict
を返した場合や `index_refresh` が `status="partial"` を返した場合、
Exception ではないため CRITICAL #1 の break 経路に乗らず、後続 plan
が継続されていた。安全文言 "rate limit safe: stop on first 429
retry exhaustion" と実装が不一致。

修正: 各 dataset の result append 直後に `if status != "ok"` で
判定し、後続 plan を `aborted_after_partial` で skip して `break`。
sleep_seconds は `ok` のときのみ次に進む前に挿入。これで例外経路
+ 戻り値経路の両方で「最初の 429 で停止」が成立。

### exit code 一覧

| code | 意味 |
|---|---|
| 0 | success |
| 2 | refused (= 設定不正 / db_path 不在 / unknown dataset / symbols 同時指定) |
| 3 | safety violation (= DB mtime 変化 / staging guard 違反) |
| 4 | partial / error / aborted (= ★ Codex CRITICAL 対応、`error:*` / `partial` / `aborted_after_error` / `aborted_after_partial` / `unknown` を全て exit 4) |

---

## smoke 条件 / 結果

### Phase 1: dry-run smoke (--symbols-limit 5)

    .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
      --db-path data/fire.staging.db --db-label staging \
      --datasets prices,index \
      --from-date 2026-05-02 --to-date 2026-05-02 \
      --symbols-limit 5 \
      --output-json /tmp/f286_data_r1_1_dryrun.json --dry-run

結果:
- symbols source=limit:5 / count=5 (= 13010 / 13050 / 13060 /
  13080 / 13090)
- target=2026-05-02〜2026-05-02 / span=1 / reason=explicit_from_to
- rows_inserted=0 / write_invoked_count=0 / error_count=0
- staging.db / develop.db / fire.db 全 last_modified unchanged

### Phase 2: staging write smoke (--symbols-limit 5、土曜 = データなし)

    --from-date 2026-05-02 --to-date 2026-05-02 (= 土曜)
    --symbols-limit 5 --write

結果:
- prices: executed=True / write_invoked=True / inserted=0 /
  status=ok (= 土曜のため J-Quants にデータなし、行は増えない
  が rate limit hit 0、API 経路は正常動作)
- staging.db unchanged (write 0 行)
- develop.db / fire.db unchanged

### Phase 3: staging write smoke (--symbols-limit 5、木曜 = 営業日)

    --from-date 2026-05-07 --to-date 2026-05-07 (= 木曜)
    --symbols-limit 5 --write

結果 (★ 実際に行が入った最初の smoke):
- prices: executed=True / write_invoked=True / inserted=5 /
  status=ok / error=None
- staging.db rows: 2,080,831 → 2,080,836 (+5)
- staging.db max_date: 2026-05-01 → 2026-05-07
- 5 銘柄それぞれ 489 → 490 行
- develop.db / fire.db 完全 unchanged
- 429 rate limit hit 0

### Phase 4: staging write smoke (--symbols-limit 10 + index、金曜)

    --from-date 2026-05-08 --to-date 2026-05-08 (= 金曜)
    --symbols-limit 10 --datasets prices,index
    --sleep-seconds 0.5 --write

結果:
- prices: executed=True / inserted=10 / status=ok
- index:  executed=True / inserted=1 / status=ok
- staging.db rows: 2,080,836 → 2,080,846 (+10) /
  index_data 58 → 59 (+1)
- staging.db max_date: 2026-05-07 → 2026-05-08 /
  index_data max: 2026-05-01 → 2026-05-08
- develop.db / fire.db 完全 unchanged
- 429 rate limit hit 0
- sleep_seconds 0.5 が prices 後 → index 前に挿入された

---

## staging before/after 集計

| metric | before | after |
|---|---|---|
| market_prices_daily rows | 2,080,831 | 2,080,846 (+15) |
| market_prices_daily max_date | 2026-05-01 | 2026-05-08 |
| index_data rows | 58 | 59 (+1) |
| index_data max_date | 2026-05-01 | 2026-05-08 |
| staging.db last_modified | May 9 22:40:35 | May 10 16:23:40 |
| develop.db last_modified | May 7 18:14:26 | unchanged ✅ |
| fire.db last_modified | May 7 16:12:38 | unchanged ✅ |

★ DB write は staging.db のみ。production / develop は完全 unchanged。

---

## 安全要件遵守

### staging-only write

- ✅ 3 段 guard (DATA-R1 から継承): db_label / db_path basename /
  FIRE_ENV
- ✅ test で 3 DB ラベル + wrong basename + missing + FIRE_ENV
  不在 を refuse

### rate limit 連続 retry 禁止

- ✅ Codex CRITICAL #1 (例外経路): catch 後に後続 plan を
  `aborted_after_error` で skip して break、test で
  `must not be called` AssertionError を fake_index に置いて検証
- ✅ Codex CRITICAL #2 (戻り値経路): status != "ok" で後続 plan を
  `aborted_after_partial` で skip して break、test で同じく
  `must not be called` AssertionError 検証
- ✅ status `unknown` も exit 4 対象

### LINE / order / broker / 楽天 / Computer Use 未接続

- ✅ source 文字列で `LineBotClient(` / `.send_text(` /
  `TradeOrder(` / `.place_order(` / `playwright` / `selenium` /
  `subprocess.run` の実 import / call なし (test 検証)

### token 直接読み込みなし

- ✅ runner / helper module 自身は os.environ / .env / dotenv /
  JQUANTS_API_KEY を直接読まない、--write 時のみ market_data.client
  に委譲

### unrelated modified 未接触

- ✅ `scripts/seed_pattern_layer1.py` 未接触
- ✅ `simulation/research_lane/historical_indicators.py` 未接触
- ✅ 全 commit `git add <specific files>` で個別 stage

### TODO Excel 未更新

✅ Google Sheets TODO 管理表は本タスクで更新していない

---

## tests 結果

### 新規 26 PASS (DATA-R1 から +26)

    tests/agents/test_jquants_daily_refresh.py: 27 → 47 PASS (+20)
      - TestLoadSymbolsFromCsv (7)
      - TestResolveTargetSymbols (7)
      - TestExecuteRefreshSymbolsPropagation (3)
      - 既存 TestExecuteRefresh / TestRateLimit に
        aborted_after_error 追加 assertion (3)

    tests/scripts/jobs/test_run_jquants_daily_refresh.py: 21 → 29 PASS (+8)
      - TestParseArgs に symbols 排他 / 0 / negative sleep (3)
      - TestMainSymbolsPropagation (4)
      - TestMainErrorExit に test_partial_return_value_aborts_remaining (1)

### regression

- F286 / F119 / F111 / F062 / F100 / F284 全テスト 0 件回帰
- フル pytest **3,031 PASS** (= 3,005 baseline + 26 新規)

---

## Codex pre-commit 結果

| commit | Codex 判定 |
|---|---|
| 420ad90 (feat helper + runner) | ✅ OK (★ CRITICAL 2 件即修正版で再 commit) |
| b8d8a22 (test)                  | ✅ OK |

### CRITICAL 2 件と修正

#### CRITICAL #1 (例外経路): catch 後の継続を停止

修正: `execute_refresh` で Exception catch 後に残り plan を全
`aborted_after_error` で skip して break。test 1 ケース追加。

#### CRITICAL #2 (戻り値経路): partial / error_msg 後の継続を停止

修正: status != "ok" で残り plan を全 `aborted_after_partial` で
skip して break、ok 時のみ次 dataset へ進む前に sleep。test 1
ケース追加。

---

## --no-verify 未使用確認

✅ 全 2 commit で `--no-verify` flag 不使用
✅ Codex usage limit / rate limit / auth error なし

---

## 次タスク提案

### 第一候補: DATA-R2 Freshness Gate

DATA-R0 で設計済みの gate-1..5 を実装し、F062-R2 LINE 本番送信前に
gate を強制する。本 DATA-R1.1 で daily refresh が実用段階になった
ので freshness gate を作る土台が整った。

### 第二候補: DATA-R1.2 全営業日カバレッジ拡大

`--symbols-limit 100` / `--symbols-limit 500` で段階的に staging
書き込み範囲を拡大、daily refresh の所要時間と rate limit 影響を
測定。十分安定したら全 4400 銘柄に拡大。

### 第三候補: persist runner 統合

derived / signals を本 runner から薄い wrapper 経由で呼び、
1 invocation で prices → index → derived → signals を完結させる。
本 runner の rate limit safe break 機構が persist runner にも
適用されるよう拡張。

優先度: 1 > 2 > 3 (LINE 通知接続準備が最優先)

---

## 関連参照

- 02_todo/F286_DATA_R0_jquants_pipeline_audit_freshness_design.md
  (DATA-R0、freshness 設計)
- 02_todo/F286_DATA_R1_jquants_daily_refresh.md (DATA-R1、本タスク
  の前段)
- 02_todo/F062_R1_line_advisory_notification_template.md
- log.md milestone (本タスク完了時に追記)
