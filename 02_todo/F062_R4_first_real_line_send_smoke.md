---
id: F062-R4
phase: P5: 通知 / 第 14 章 LINE 通知配信
priority: 最優先
status: 停止 (LINE API 401 Unauthorized、token 再確認が必要)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R3 (LINE production send path 構造的安全性 PASS)
  - F286-DATA-R2 (freshness gate 全 5 段 PASS)
  - FIRE-TODO-R1 (pre_launch_required 0 件確認)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 26 章
---

# F062-R4: First Real LINE Send Smoke

最終更新: 2026-05-10

## 状態: 停止 (LINE API 401 Unauthorized)

initial real send で **LINE API が 401 Unauthorized で拒否** したため、
タスク仕様の停止条件「real sendが 1 通以外」(= 0 通) に該当し停止。
ただし以下の安全要件はすべて維持され、partial delivery / DB write /
token leak は発生していない。

LineBotClient の F278-Pre Q1 実装どおり、401 / 403 は構造的エラー
として retry せず即 raise → F062-R3 sender が `logger.error` 後に
propagate → F062-R2 router が `F062R2SendRefused` に wrap せず
partial state 保持で SendResult 返却 → exit 4。

> ★ 重要: partial_delivery=False / sent_count=0 のため、**retry 可能**
> な状態 (= 重複送信リスクなし)。

## env / gate 検証

| 項目 | 結果 |
|---|---|
| LINE_CHANNEL_TOKEN_SET     | True (length=197) ✅ |
| FIRE_LINE_RECIPIENT_ID_SET | True (prefix='U', length=33) ✅ |
| ~/.fire_secrets/line.env mode | 600 ✅ |
| DATA-R2 overall_status     | pass ✅ |
| DATA-R2 line_send_allowed  | True ✅ |
| DATA-R2 5 段 gate          | 全 pass ✅ |

env から token / recipient は **構造的に正しく** 読み込まれた
(= length / prefix 整合)。401 は token の値そのものが LINE 側で
無効と判定されたことを意味する (= 期限切れ / 貼り付けミス / 別 channel
の token / production と test の取り違え等)。

## 実施結果

### 事前 dry-run (Step 3) ✅

```
mode: dry_run
send_allowed: True
sent_count: 0
line_api_call_count: 0
stub_invocations: 0
partial_delivery: False
token_read_count: 0  ← dry-run 経路で env を読まない
production_callable_built: False
forbidden_phrase_count: 0
safety_footer_present: True
manual_review_required_count: 0  ← test-message-only で selected_rows=[] 化
exit code: 0
```

### Real send (Step 4) ✗ (401)

```
mode: send
dry_run: False             ← Codex CRITICAL #5 対応で厳密反映
send_allowed: False
sent_count: 0
line_api_call_count: 0     ← 構造的に increment されていない
stub_invocations: 0
partial_delivery: False    ← retry 可
token_read_count: 1        ← --send + --hq-approved-send で env 読みに行った
production_callable_built: True  ← config 構築は成功 (= length / prefix 検査 PASS)
hq_approved_send: True
max_chunks: 1
test_message_only: True
forbidden_phrase_count: 0
safety_footer_present: True
selected_row_count: 0
payload_chunks_total: 1
exit code: 4
production_outcomes: []
```

refuse 内訳:
- `chunk send failed: 0/1 sent before failure`
- `production_send_callable raised at chunk_index=0: UnauthorizedException: (401)`
- LINE response: `Authentication failed. Confirm that the access token in the authorization header is valid.`
- LINE www-authenticate ヘッダ: `error="invalid_token"`

### 安全要件遵守

| 項目 | 結果 |
|---|---|
| 送信は 0 通                                         | ✅ (= 1 通も到達せず) |
| partial_delivery                                   | False ✅ (= retry 可) |
| token leak 0 (4 artifact 内)                        | ✅ (`grep token /tmp/f062_r4_*.{json,txt}` → 0 件) |
| LineBotClient response body 内に token 含有        | ✅ (= 401 response に token は含まれない、本物 token は外部に流出していない) |
| 通常 Advisory payload (銘柄候補 / 注文価格 / 数量 / 執行指示) | 送ろうとしていない ✅ |
| 自動発注                                          | 0 ✅ |
| 楽天証券操作                                       | 0 ✅ |
| Computer Use                                       | 0 ✅ |
| DB write                                           | 0 ✅ |
| data/fire.db / fire.develop.db / fire.staging.db    | 全 mtime unchanged ✅ |
| TODO Excel                                         | 未更新 ✅ |
| scripts/seed_pattern_layer1.py                     | 未接触 ✅ |
| simulation/research_lane/historical_indicators.py  | 未接触 ✅ |
| unrelated modified                                 | 未 stage / 未 commit ✅ |
| --no-verify                                        | 不使用 ✅ |
| Codex pre-commit (docs commit のため対象外)        | ✅ |

### 観察された 1 点 (= 改善候補、本タスクでは修正しない)

`scripts/jobs/run_f062_line_production_send_smoke.py` の output_json
は `production_config.recipient_id` を full string で記録している
(= F062-R3 設計時に意図したもの、token は length のみ)。

artifact `/tmp/f062_r4_first_real_send.json` に recipient_id (= 'U...'
本人 ID) が full 形で残っている。本ファイルは `/tmp` 配下で git 管理
外、外部送信もしないため漏洩リスクは限定的だが、運用次第では
「length / prefix のみ記録」に絞ったほうが防御的。

→ 改善候補として **F062-R5 後** に検討。本タスクでは現状維持で記録。

## DB 不変確認

| DB | pre / post mtime | 不変 |
|---|---|---|
| data/fire.db          | May  7 16:12:38 / May  7 16:12:38 | ✅ |
| data/fire.develop.db  | May  7 18:14:26 / May  7 18:14:26 | ✅ |
| data/fire.staging.db  | May 10 18:22:36 / May 10 18:22:36 | ✅ |

## token leak 確認

```
TOKEN_LEAK_HITS: 0
RECIPIENT_HITS_IN_ARTIFACTS: 1  ← 設計通り (production_config.recipient_id full 記録)
```

token は 4 artifact のいずれにも平文で含まれていない。本物 token が
外部 (LINE API 以外) に流出した形跡なし。

## 通常 Advisory 本番送信は未開始である確認

- `--test-message-only` で payload chunks を 1 個の固定 test message
  に置換 (= selected_rows=[] / selected_count=0 / send_intent="f062-r3
  hq approved test message")
- 銘柄候補一覧 / 注文価格 / 数量 / 執行指示 は構造的に送れない
- そもそも sent_count=0 (= LINE API 受信前に 401 で停止)
- 通常 Advisory は **完全に未開始**

## LINE app での Fujiwara 受信確認

- 受信 0 通 (= sent_count=0、API レベルで拒否)
- Fujiwara の LINE app / iSPEED 通知に本タスク由来のメッセージは
  届かない

## Fujiwara への依頼 (= タスク再開条件)

LINE_CHANNEL_TOKEN を再確認してください。401 invalid_token は以下の
いずれかが原因:

1. token が LINE Developers コンソールで **regenerate** されている
   (= 旧 token が無効化された)
2. LINE Channel Type が違う (= Messaging API ではなく LINE Login や
   別 channel の token を貼ってしまった)
3. token に余分な空白 / 改行 / 引用符が混入した
4. 本番 / test channel の取り違え

確認手順:
- LINE Developers Console → 該当 Provider → Messaging API channel →
  「Channel access token (long-lived)」を再発行 or 既存値を確認
- `~/.fire_secrets/line.env` の `LINE_CHANNEL_TOKEN=...` を更新
- `chmod 600 ~/.fire_secrets/line.env` 維持
- 再度ターミナルで `source ~/.fire_secrets/line.env` 後に length 確認:
  ```
  python3 -c "import os; t=os.environ.get('LINE_CHANNEL_TOKEN'); print('len:',len(t) if t else 0)"
  ```
- 一般的な LINE Messaging API channel access token は **約 170-180
  文字または 200+ 文字** (= 197 文字は妥当範囲)。length 自体は問題
  なし。値 (= 内容) を再確認。

その後、本 F062-R4 タスクをそのまま再起動 (= 同じ vault doc / 同じ
artifact path) で再 send 試行。

## 次タスク

1. Fujiwara が LINE_CHANNEL_TOKEN を再確認 / 必要なら再発行
2. F062-R4 を再実行 (= 同 runner、同 artifact)
3. 401 が解消し sent_count=1 になれば LINE app で受信確認
4. 受信成功時: F062-R5 First Production Advisory Small Launch
   (max_chunks 1〜2、少数候補、手動レビュー前提)
