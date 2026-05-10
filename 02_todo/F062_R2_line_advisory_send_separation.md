# F062-R2 LINE Advisory Send / Production Separation

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: F062-R1 (4ebf285 / 7b6ad6c / 0f1254f / 4023577) +
> DATA-R2 (e8c80f8 / 11c31ba / 56baf6e) + DATA-R1.3 (運用 smoke)
> **Mode**: 完全 dry-run / 実 LINE API path 持たず (= F062-R3 予定)
> **Result**: ★★★ 多段 guard (G-1 chunk + safety footer / G-2 row
> safety / G-3 gate / G-4 line_send_allowed / G-5 mode) で本番送信
> 可否を判定。Codex CRITICAL 4 件即修正で safe-by-default 完成。
> production runner 経路では構造的に実 LINE API は呼ばれない。★★★

---

## タスク名

F062-R2 LINE Advisory Send / Production Separation (= LINE Send
dry-run / production 分離)

---

## 実装ファイル一覧

| 種別 | ファイル | 行数 |
|---|---|---|
| feat | `agents/line_advisory_send.py` | 478 |
| chore (runner) | `scripts/jobs/run_f062_line_advisory_send.py` | 371 |
| test | `tests/agents/test_line_advisory_send.py` | 31 PASS |
| test | `tests/scripts/jobs/test_run_f062_line_advisory_send.py` | 21 PASS |

---

## commit hash 一覧

| commit | type | 内容 |
|---|---|---|
| `a508ebb` | feat | F062-R2: add LINE advisory send router |
| `90e2bb8` | chore | F062-R2: add LINE advisory send runner |
| `64882f1` | test | F062-R2: add LINE send separation tests |
| (本 commit) | docs | F062-R2: vault |
| (次 commit) | docs | F062-R2: log milestone |

---

## send / dry-run 仕様

### 多段 guard (= 全 pass まで送信不可)

| guard | 内容 | level |
|---|---|---|
| G-1 chunks + footer + forbidden | chunks 非空、各 chunk に F062-R1 SAFETY_FOOTER_LINES 全 8 行が runtime で含まれる、forbidden_phrase_count=0 + 13 種 phrase の二重検査 | required |
| G-2 row safety | selected_rows 全件で manual_review_required=True かつ auto_order_allowed=False (= bool 厳密判定、str truthy 値も refuse) | required |
| G-3 gate overall | gate_result.overall_status == "pass" (default、warning は allow_warning=True 指定時のみ許可、refuse は絶対不可) | required |
| G-4 gate line_send_allowed | gate_result.line_send_allowed == True (= bool 型) | required |
| G-5 mode | mode == "send" + _test_send_stub or production_send_callable が用意されている (F062-R2 では production path なし、stub のみ) | required |

### overall mode

| mode | 動作 | sent_count | line_api_call_count |
|---|---|---|---|
| dry_run (default) | 何もしない | 0 | 0 |
| send (production runner、stub=None) | guard pass しても stub なしで実送信 path 不在のため自動 refuse (= notification-miss 防止) | 0 | 0 |
| send (test、stub 指定) | TEST 専用 stub に chunks を流す | N | 0 (= stub call は API call ではない) |

### exit code

| code | 意味 |
|---|---|
| 0 | dry-run 完了 / send_allowed=True (= test 経由のみ) |
| 2 | refused (= 設定不正 / file 不在) |
| 3 | F062R2SendRefused exception |
| 4 | send_allowed=False (= 多段 guard 違反、production runner --send 経路で必ず exit 4) |

---

## gate 連携仕様

DATA-R2 gate result を JSON で受け取り `_validate_gate` で検査:

- `overall_status`:
  - `"pass"` → guard pass
  - `"warning"` → default refuse、`allow_warning=True` 指定時のみ pass
  - `"refuse"` → 絶対 refuse (allow_warning=True でも pass しない)
  - その他 → "invalid" として refuse
- `line_send_allowed`:
  - `True` (bool) → guard pass
  - `False` (bool) → refuse
  - 非 bool 型 (= "true" 等の str) → refuse

★ DATA-R2 が safe-by-default で `line_send_allowed=False` を返した
場合は本 router でも必ず refuse される (= 二重防御)。

---

## smoke 結果

### Phase 1: dry-run (F062-R1 + DATA-R2 pass gate)

入力:
- `--payload-json /tmp/f062_r1_line_preview_payload.json`
  (F062-R1 で生成、20 candidates / 5 chunks / forbidden 0 / safety_footer
   present / 全 row manual=True / auto=False)
- `--gate-json /tmp/f286_data_r1_3_gate_after.json`
  (DATA-R1.3 完了時点、overall=pass / line_send_allowed=True)

実行:

    .venv/bin/python -m scripts.jobs.run_f062_line_advisory_send \
      --payload-json /tmp/f062_r1_line_preview_payload.json \
      --gate-json    /tmp/f286_data_r1_3_gate_after.json \
      --output-json  /tmp/f062_r2_send_result_dryrun.json \
      --completion-report /tmp/f062_r2_completion_report.txt

結果:

    mode: dry_run
    send_allowed: True
    sent_count: 0
    stub_invocations: 0
    line_api_call_count: 0  ★
    token_read_count: 0  ★
    exit code: 0

### Phase 2: send mode (production runner、stub=None)

実行:

    .venv/bin/python -m scripts.jobs.run_f062_line_advisory_send \
      --payload-json /tmp/f062_r1_line_preview_payload.json \
      --gate-json    /tmp/f286_data_r1_3_gate_after.json \
      --send

結果:

    mode: send
    send_allowed: False  ★ (= F062-R2 #3 CRITICAL 対応で構造的に refuse)
    sent_count: 0
    stub_invocations: 0
    line_api_call_count: 0
    refused_reasons:
      - F062-R2: mode=send requires _test_send_stub (TEST ONLY);
        production runner has no real LINE API path. F062-R3 will
        add production_send_callable.
    exit code: 4

### Phase 3: refuse gate smoke

合成 refuse gate JSON (`overall_status=refuse / line_send_allowed=False`)
で `--send`:

    refused_reasons:
      - gate_result.overall_status is 'refuse' (= 必須 gate 未達)
      - gate_result.line_send_allowed is False
    exit code: 4

---

## counts 一覧 (smoke)

### dry-run smoke
- token_read_count: **0**
- line_api_call_count: **0**
- sent_count: **0**
- stub_invocations: 0
- forbidden_phrase_count: 0
- auto_order_allowed_true_count: 0
- manual_review_required_count: (= input row 数、F062-R1 で 20)

### send smoke (production runner)
- token_read_count: **0**
- line_api_call_count: **0**
- sent_count: **0**  (★ refuse のため)
- send_allowed: **False**
- exit code: 4

---

## forbidden phrase 検査結果

F062-R2 で 13 種 forbidden phrase を runtime 二重検査:
- 買え / 発注せよ / 発注しろ / 発注して
- 自動で注文 / そのまま約定 / そのまま発注
- 楽天で実行
- 確実に勝てる / 必ず勝てる / 確実に儲かる / 必ず儲かる / 必勝

smoke では 0 件 (= F062-R1 payload 自体が clean)。

---

## auto_order_allowed_true_count / manual_review_required_count

bool 厳密判定で:
- F062-R1 payload の selected_rows: 全 20 件で manual=True / auto=False
- auto_order_allowed_true_count: 0 (期待通り)
- manual_review_required_count: 20 (= 全 row 数と一致)

★ str "true" / int 1 / list [] 等の非 bool 値は refuse される
(= F062-R1 で対応した bool 厳密判定を継承)

---

## 安全要件遵守

### LINE 本番 API 未呼び出し確認

- ✅ 本 module / runner ともに `LineBotClient` / `linebot` /
  `from notifications.line_bot` を import / 呼び出さない
- ✅ `.send_text(` / `.push_message(` / `.reply_message(` の実
  call 無し (test で source 文字列検証)
- ✅ `line_api_call_count` は本 module 構造で必ず 0
- ✅ smoke で 3 phase (dry-run / send-no-stub / send-refuse) すべて
  `line_api_call_count: 0`

### token 読み込みなし確認

- ✅ source に `load_dotenv(` / `from dotenv` / `import dotenv` /
  `os.environ['JQUANTS` / `os.getenv("LINE` 等の実 read pattern なし
- ✅ `token_read_count: 0` を全 smoke で確認
- ✅ argparse help に `--line-token` / `--channel-token` /
  `--api-key` / `--token` option 不在

### DB write / DB access 0 確認

- ✅ `import sqlite3` / `sqlite3.connect` の実 import / call 無し
- ✅ runner / helper module ともに DB 不接続

### production / develop 無触

- ✅ DB 不接続のため staging / develop / production 全て無触
- ✅ smoke 前後で 3 DB last_modified 完全 unchanged (= DATA-R1.3
  完了時の値のまま、 staging 5/10 18:22 / develop 5/7 / production 5/7)

### order / broker / 楽天 / Computer Use 未接続

- ✅ source に `TradeOrder(` / `.place_order(` / `.send_order(` /
  `.submit_order(` の実 import / call なし
- ✅ `playwright` / `selenium` / `subprocess.run` 不在
- ✅ argparse help に `--broker` / `--rakuten` / `--order` /
  `--auto-order` / `--computer-use` / `--playwright` / `--write`
  option 不在

### unrelated modified 未接触

- ✅ `scripts/seed_pattern_layer1.py` 未接触
- ✅ `simulation/research_lane/historical_indicators.py` 未接触
- ✅ 全 commit `git add <specific files>` で個別 stage

### TODO Excel 未更新

✅ Google Sheets TODO 管理表は本タスクで更新していない

---

## tests 結果

### 新規 52 PASS

    tests/agents/test_line_advisory_send.py (31)
      - TestAssertSendAllowed (15)
      - TestExecuteSend (8) ★ Codex CRITICAL #3 対応 1 ケース含む
      - TestModuleSourceSafety (3)
      - その他 (per_chunk_footer_missing / safety_marker_only_not_enough
              / all_chunks_full_footer_present_pass の 3 ケース、
              CRITICAL #1 + #4 対応で追加)

    tests/scripts/jobs/test_run_f062_line_advisory_send.py (21)
      - TestParseArgs (3)
      - TestArgParserSafety (2)
      - TestLoaders (5)
      - TestMainPaths (8) ★ CRITICAL #3 対応 + dry_run_gate_pass
                              _does_not_require_stub
      - TestOutputFormatters (1)
      - TestRunnerSourceSafety (3)

### regression

- F062 / F111 / F119 / F286 / DATA / 全 module で 0 件回帰
- フル pytest **3,131 PASS** (= 3,079 baseline + 52 新規)

---

## Codex pre-commit 結果

| commit | Codex 判定 |
|---|---|
| a508ebb (feat) | ✅ OK (★ CRITICAL 4 件即修正後の最終版) |
| 90e2bb8 (chore runner) | ✅ OK |
| 64882f1 (test) | ✅ OK |

### CRITICAL 4 件と修正

#### CRITICAL #1: docstring と各 chunk safety footer 検査の不整合

指摘: docstring「各 chunk に safety footer 含む」と書いたが、
実装は payload-level の `safety_footer_present` フラグだけを見て
各 chunk 個別検査していなかった。

対応: 各 chunk 内で safety footer marker (= F062-R1 SAFETY_FOOTER_LINES[0]
"Safety") を runtime 検査するよう追加。test 2 ケース追加。

#### CRITICAL #2: 実 API callable と test stub の構造的混同

指摘: `line_send_callable` という名前だと実 API callable と test
stub が同じ path を通り、SendResult.line_api_call_count の契約
(= F062-R2 では必ず 0) と矛盾しうる。

対応:
- callable を `_test_send_stub` (= TEST 専用) に rename
- `SendResult.stub_invocations` field を追加し
  `line_api_call_count` から分離
- `line_api_call_count` は本 module 構造で必ず 0
- F062-R3 で `production_send_callable` 別 path を追加予定

#### CRITICAL #3: notification-miss リスク

指摘: `mode=send` + `_test_send_stub=None` で `send_allowed=True
/ dry_run=False / sent_count=0 / line_api_call_count=0` という
曖昧状態を「成功 0 件送信」と誤解釈されうる、notification-miss
リスク。

対応: `mode=send` で stub=None の場合 `send_allowed=False` に
明示的に上書き、`refused_reasons` に F062-R2 注記追加。
production runner は常に `_test_send_stub=None` を渡すため
mode=send 経路は構造的に refuse される。

#### CRITICAL #4: "Safety" marker 単語だけでは弱い

指摘: `_SAFETY_FOOTER_MARKER = "Safety"` の包含チェックだけだと、
本文中に "Safety" (= "FIRE Safety in product name only" 等) が
偶発出現しただけで送信許可されうる。

対応: F062-R1 SAFETY_FOOTER_LINES と整合する全 8 行を必須に
強化、各 chunk 内で 8 行全てが揃って含まれるか runtime 検査。
test で「Safety 単語 1 つだけでは refuse される」を検証
(`test_safety_marker_only_not_enough`)。

✅ 全 4 CRITICAL を 1 round ずつで修正完了、最終 commit で全 OK
✅ Codex usage limit / rate limit / auth error なし

---

## --no-verify 未使用確認

✅ 全 3 commit (feat / chore / test) で `--no-verify` flag 不使用
✅ 連続 retry なし (CRITICAL は順次修正)

---

## 次タスク提案

### 第一候補: F062-R3 LINE Production Send Path 追加

F062-R2 で構造を完成、次は実 LINE API への bind:
- `agents/line_advisory_send.py` に `production_send_callable` field
  を追加 (= LineBotClient.send_text の wrapper)
- HQ 承認 flag (`--enable-real-send`) を runner に追加、default は
  必ず False
- `production_send_callable` 経由でのみ `line_api_call_count` を
  > 0 にする
- token は `LineBotClient(channel_token=...)` 経由で読む
  (= 本 router 自身は無触)
- HQ 承認 + 5 段 gate 全 pass + production_send_callable enable
  時のみ実送信
- 実送信ログ / retry / failure handling

### 第二候補: F286-DATA-R3 daily refresh production 化

DATA-R1.1/1.2/1.3 で staging 完成、daily refresh runner を
production に伝播 (= cron / launchd 自動実行、HQ 承認後)。

### 第三候補: persist runner 統合

prices → derived → signals 連鎖を 1 invocation で完結する薄い
orchestrator wrapper、DATA-R1.3 で確認した 300-batch を default に。

優先度: 1 > 2 > 3 (LINE 本番送信導線の完成が最優先、HQ 承認後)

---

## 関連参照

- 02_todo/F062_R1_line_advisory_notification_template.md (前段)
- 02_todo/F286_DATA_R2_data_freshness_gate.md (gate 連携元)
- 02_todo/F286_DATA_R1_3_remaining_symbols_refresh.md (gate pass 達成)
- 02_todo/F111_R4_multi_base_date_f119_insights_smoke.md
- log.md milestone (本タスク完了時に追記)
