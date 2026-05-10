# F062-R3: LINE Production Send Path / Final Safety Smoke

最終更新: 2026-05-10

## 目的

F062-R2 で「dry-run / production の経路分離 + 多段 guard」を実装したが、
実 LINE API 送信 path は `_test_send_stub` (TEST 専用) にしか流せない
構造で、production runner は構造的に refuse される設計だった。

F062-R3 では:

- 実 LINE API 経路 (`production_send_callable`) を **別 module
  (`agents/line_production_sender.py`)** に隔離して追加
- F062-R2 router を最小拡張し、`_test_send_stub` と
  `production_send_callable` を排他で受け取る形に
- 実送信は HQ 承認 flag (`--hq-approved-send`) と recipient ID 明示と
  token 設定の三点が揃ったときだけ発火
- 最初は 1 chunk / test message only に限定

を達成する。

実 LINE API への本送信は本タスク内では実施しない。HQ (Fujiwara) の
承認後、別途手動で 1 chunk / test message を送る運用。

## 実装ファイル

| ファイル | 行数 | 役割 |
|---|---|---|
| `agents/line_production_sender.py` | 254 | LineBotClient 隔離 module。`ProductionSendConfig` + `assert_production_safe` + `build_production_send_callable` (max_chunks attr 付与) + chunk 単位 defense-in-depth 検査 + status verify + structured logging |
| `agents/line_advisory_send.py` | +69 / -19 | F062-R2 router 拡張: `production_send_callable` 引数 / 排他 / preflight max_chunks / outcome status verify / partial state preserve / `partial_delivery` field / `dry_run` 厳密反映 |
| `scripts/jobs/run_f062_line_production_send_smoke.py` | 397 | CLI runner: --send + --hq-approved-send + --recipient-id + --max-chunks 1 default + partial delivery 警告表示 |
| `tests/agents/test_line_production_sender.py` | 47 PASS | sender unit + router 統合 + Codex CRITICAL 対応 (partial / status / preflight) + source 検証 |
| `tests/scripts/jobs/test_run_f062_line_production_send_smoke.py` | 14 PASS | runner CLI 各 path |
| `tests/agents/test_line_advisory_send.py` | (修正) | F062-R3 注記反映 + dry_run 厳密反映 |
| `tests/scripts/jobs/test_run_f062_line_advisory_send.py` | (修正) | refuse 文言の F062-R3 形式に追従 |

## Codex pre-commit CRITICAL 5 件と修正内容

| # | 指摘 | 修正 |
|---|---|---|
| 1 | `client.send_text` の retry / durable log なしで transient failure で notification miss | sender 内で `logger.error` / `logger.info` で structured log を残す。LineBotClient 自体が default `retry=2` の internal retry を持ち、終局失敗は `{"status": "error", ...}` で返す形を活用 |
| 2 | `send_text` 戻り Mapping を success と即断、`status="error"` を sent 扱いしてしまう | sender 内で status が `"ok"` / `"dry_run"` でなければ `F062R3ProductionRefused` raise + log。router 側でも outcome `send_text_status` を verify (defense-in-depth) |
| 3 | `max_chunks=1` default で payload が 2+ chunks のとき 1 件目送信 → 2 件目で raise → partial 通知 + retry 重複 | callable に `max_chunks` 属性を attach、router 側で **preflight** で `payload chunks > max_chunks` なら 0 件送信で refuse |
| 4 | multi-chunk send が non-atomic、途中失敗で `F062R2SendRefused` raise → caller が partial 状態を観測できない、retry で重複 | router 側で chunk loop を try/except で wrap、失敗時 break + partial state 保持。`SendResult.partial_delivery: bool` field で「retry 厳禁」を caller に明示 |
| 5 | `dry_run = (mode == DRY_RUN or not send_allowed)` で partial delivery 後 dry_run=True になり「実 LINE 呼んでない」と誤解釈リスク | `dry_run = (mode == MODE_DRY_RUN)` に厳密反映。mode=send なら send_allowed=False でも `dry_run=False` を保つ |

## 多段 guard 設計

### F062-R2 router (拡張)

```
G-1 payload chunks 各々が safety footer 8 行を含み forbidden phrase 0
G-2 selected_rows 全件で manual_review_required=True / auto_order_allowed=False
G-3 gate_result.overall_status == "pass" (warning は --allow-warning のみ)
G-4 gate_result.line_send_allowed is True
G-5 mode == "send"
G-6 _test_send_stub と production_send_callable は **排他**
    両方 None だと send_allowed=False に強制 (notification-miss 防止)
```

### F062-R3 production sender (新規)

```
P-1 ProductionSendConfig 生成時に bool 厳密 / channel_token 非空 /
    recipient_id 非空 / hq_approved is True / max_chunks >= 1
P-2 recipient_id 先頭 1 文字を 'U' (user) / 'C' (group) / 'R' (room)
    に限定 (= broadcast / multicast 防御)
P-3 chunk-level defense-in-depth: 各 chunk で safety footer 8 行 +
    forbidden phrase 13 種を **本 module 内で再検査**
    (= F062-R2 を bypass された場合でも素通りしない)
P-4 max_chunks 超過は F062R3ProductionRefused (default = 1)
P-5 LineBotClient(channel_token, dry_run) は本関数内で **遅延 import**
    (= 本 module を import しない限り linebot SDK / token は読まれない)
```

## CLI 仕様

```
usage: run_f062_line_production_send_smoke
       --payload-json PAYLOAD_JSON
       --gate-json GATE_JSON
       [--send]
       [--hq-approved-send]
       [--recipient-id RECIPIENT_ID]
       [--channel-token-env LINE_CHANNEL_TOKEN]
       [--max-chunks 1]
       [--test-message-only]
       [--dry-run-line-api]
       [--allow-warning]
       [--output-json PATH]
       [--completion-report PATH]
```

- default: dry-run (= --send 不在で必ず実 API 呼ばない)
- --send 単独: refuse (= --hq-approved-send / --recipient-id 不在)
- --send + --hq-approved-send + --recipient-id + token 揃って初めて
  build_production_send_callable() が走る
- --max-chunks 1 default + --test-message-only で「最初は 1 chunk /
  test message only に限定する」F062-R3 タスク要件を満たす
- --dry-run-line-api: LineBotClient(dry_run=True) で構築し、実
  push_message を構造的に呼ばない段階導入モード

### exit code

- 0: pass (dry-run pass / send pass)
- 2: validation 失敗 (F062R2SendRefused / F062R3ProductionRefused)
- 3: 想定外 exception
- 4: send_allowed=False (= multi-stage guard 未達で 0 件送信)

## smoke 結果 (7 phase)

| Phase | 入力 | exit | sent_count | line_api_call_count | token_read | 主要 refuse 理由 |
|---|---|---|---|---|---|---|
| 1 dry-run | F062-R1 payload + DATA-R2 pass | 0 | 0 | 0 | 0 | (送信なし、send_allowed=True) |
| 2 --send only | (no --hq-approved-send) | 4 | 0 | 0 | 0 | --send requires --hq-approved-send |
| 3 no token | env unset, --hq-approved-send | 4 | 0 | 0 | 1 | token env var not set |
| 4 no recipient | token あり、--recipient-id 無 | 4 | 0 | 0 | 1 | --send requires --recipient-id |
| 5 broadcast | recipient="all" | 4 | 0 | 0 | 1 | recipient_id must start with U/C/R |
| 6 mock send | --dry-run-line-api + --test-message-only | 0 | 1 | 1 | 1 | (送信成功、LineBotClient.dry_run=True) |
| 7 refuse gate | gate.overall_status="refuse" | 4 | 0 | 0 | 1 | gate refuse / line_send_allowed=False |

## 実 LINE API 未呼び出し確認

- Phase 6 で送信成功したが LineBotClient(dry_run=True) で構築したため
  `_client.push_message()` は **構造的に** 呼ばれない (= LineBotClient.send_text
  が `{"status": "dry_run", ...}` を即返却)
- Phase 1-5 / 7 では production_send_callable が build されないか、
  build されても guard で refuse されて send_text 自体呼ばれない
- LineBotClient のログ (logs/notifications/notifications_line.log) に
  Phase 6 の dry_run 行が 1 行だけ追記される (DRY mode = 実 API 未送信
  の証跡)
- 全 smoke で実 LINE API への push_message / reply_message は呼ばれない

## token leak 確認

- production_outcomes / SendResult.to_dict / completion_report / output_json
  に channel_token は **平文では** 含まれない
- 含まれるのは `channel_token_length` (整数) のみ
- smoke 7 phase 全出力 (.json + .txt) に `DUMMY_TOKEN_FOR_SMOKE` 文字列
  は 0 件 (= grep -l で 0 ヒット)

## DB 不変確認

| DB | smoke 前 mtime | smoke 後 mtime | 不変 |
|---|---|---|---|
| data/fire.db | 5月 7 16:12 | 5月 7 16:12 | ✅ |
| data/fire.develop.db | 5月 7 18:14 | 5月 7 18:14 | ✅ |
| data/fire.staging.db | 5月 10 18:22 | 5月 10 18:22 | ✅ |

## 構造的隔離

- `agents/line_advisory_send.py` (= F062-R2 router) は AST 検査で
  `linebot` / `notifications.line_bot` / `agents.line_production_sender`
  / `dotenv` のいずれも import しない (test で実装側 source を検証)
- `agents/line_production_sender.py` (= F062-R3 sender) は LineBotClient
  を **関数内で遅延 import**、module top-level には書かない (test で検証)
- runner は `agents.line_production_sender` を `--send` path のときのみ
  関数内 import (top-level に書かない)

## 排他性検証

- `_test_send_stub` と `production_send_callable` を両方渡すと
  F062R2SendRefused (= test と production を混同させない)
- production_send_callable が non-Mapping を返すと F062R2SendRefused
- production_send_callable が raise すると F062R2SendRefused に wrap

## tests 結果

- F062-R3 新規 tests:    61 PASS (sender 47 + runner 14)
- F062-R2 既存 tests:    52 PASS (refuse 文言 + safety_notes + dry_run
                          厳密反映を F062-R3 形式に追従)
- full pytest:           3,192 PASS (= 3,131 baseline + 61 新規)
- regression:            0 件失敗

## partial delivery 仕様

| 状態 | sent_count | line_api_call_count | partial_delivery | send_allowed | retry 可否 |
|---|---|---|---|---|---|
| dry-run pass | 0 | 0 | False | True | (送信なし) |
| 0/N 失敗 (1 件目即失敗) | 0 | 0 | False | False | retry 可 (= 全失敗、未送信) |
| N/M 失敗 (途中失敗) | N>0 | N | True | False | **retry 厳禁** (重複送信リスク) |
| 全成功 | M | M | False | True | (再送不要) |
| preflight refuse (chunks > max) | 0 | 0 | False | False | retry 可 (= 0 件送信) |
| gate refuse | 0 | 0 | False | False | retry 可 |
| token missing | 0 | 0 | False | False | retry 可 (= 0 件送信) |

partial_delivery=True 検出時の運用:
1. completion_report の "PARTIAL DELIVERY WARNING" section に N 件
   送信の事実が記録される
2. `logs/notifications/notifications_line.log` を grep して Fujiwara
   LINE 側の実受信履歴と突き合わせ
3. 残りの chunk を **絶対に同 payload で再送しない**。retry したい
   場合は payload chunks を切り出し直して max_chunks 内に収まる形
   で実施

## 次の運用ステップ (HQ 承認後の実 API 送信)

1. HQ (Fujiwara) が:
   - LINE_CHANNEL_TOKEN 環境変数 を本物の channel access token に設定
   - `--recipient-id` に本物の userId/groupId/roomId を指定
2. 最初は必ず以下の組み合わせで 1 通だけ送る:
   ```
   .venv/bin/python -m scripts.jobs.run_f062_line_production_send_smoke \
     --payload-json /tmp/f062_r1_line_preview_payload.json \
     --gate-json /tmp/f286_data_r1_3_gate_after.json \
     --send --hq-approved-send \
     --recipient-id <YOUR_LINE_USER_ID> \
     --max-chunks 1 --test-message-only \
     --output-json /tmp/f062_r3_first_real_send.json \
     --completion-report /tmp/f062_r3_first_real_send_report.txt
   ```
   `--dry-run-line-api` を **付けない** ことで実 push_message が走る。
   `--test-message-only` で内容を 1 通の固定文に固定。
3. 受信確認 → ログ確認 → exit code 0 を確認後、payload chunks の
   実送信を段階的に解禁する (= --max-chunks を増やすかどうかは別判断)。
