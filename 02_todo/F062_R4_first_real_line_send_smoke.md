---
id: F062-R4
phase: P5: 通知 / 第 14 章 LINE 通知配信
priority: 最優先
status: 完了 ★ (2026-05-10、初回 real LINE send 成功 + Fujiwara LINE app 受信確認済み)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R3 (LINE production send path 構造的安全性 PASS)
  - F286-DATA-R2 (freshness gate 全 5 段 PASS)
  - FIRE-TODO-R1 (pre_launch_required 0 件確認)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 26 章
---

# F062-R4: First Real LINE Send Smoke

最終更新: 2026-05-10 (4 回目試行、成功)

## ★ 状態: 完了 (実 LINE API 1 通送信成功 + Fujiwara 受信確認済み)

| 項目 | 結果 |
|---|---|
| sent_count                       | **1** ✅ |
| line_api_call_count              | **1** ✅ |
| partial_delivery                 | False ✅ |
| send_text_status                 | ok ✅ |
| dry_run_line_api                 | False (= 実 push_message) |
| token leak (4 artifact)           | **0** 件 ✅ |
| DB write                         | 0 (3 DB 全 mtime unchanged) ✅ |
| 通常 Advisory                     | 未開始 (= test-message-only 固定) ✅ |
| Codex pre-commit                 | docs commit のため対象外 |
| **Fujiwara LINE app 受信確認**    | **★ 確認済み (2026-05-10)** ✅ |
| 追加 LINE 送信                    | 0 (= 受信確認後の追加送信なし) ✅ |

## 試行履歴

| 試行 | 日時 (JST) | 状態 | 主因 |
|---|---|---|---|
| 1 回目 | 2026-05-10 21:30 頃 | 停止 | env 未提供 |
| 2 回目 | 2026-05-10 21:53 頃 | 停止 | LINE API 401 invalid_token (旧 token) |
| 3 回目 | 2026-05-10 22:35 頃 | 停止 | UnicodeEncodeError (token に U+2028 混入、length=518) |
| **4 回目** | **2026-05-10 22:49** | **★ 成功** | **token sanitize 後 (length=516, is_ascii=True, latin1_ok=True)** |

## 実施結果

### Step 1: env ✅
```
LINE_CHANNEL_TOKEN_SET:        True
LINE_CHANNEL_TOKEN_LENGTH:     516       ← sanitize 後 (518 - U+2028 × 2)
LINE_CHANNEL_TOKEN_IS_ASCII:   True      ← 不可視 Unicode 改行除去済
LATIN1_OK:                     True      ← Authorization ヘッダ encode 可能
FIRE_LINE_RECIPIENT_ID_PREFIX: 'U'
FIRE_LINE_RECIPIENT_ID_LENGTH: 33
```

### Step 2: gate ✅
```
overall_status:    pass
line_send_allowed: True
5 段 (prices/signals/index/derived/other): 全 pass
reasons:           []
allow_warning:     False
```

### Step 3: 事前 dry-run ✅
```
exit: 0
mode: dry_run / send_allowed: True
sent=0 / line_api=0 / token_read=0 / production_callable_built: False
forbidden_phrase_count: 0 / safety_footer_present: True
manual_review_required_count: 0 (= test-message-only で空 list 化)
```

### Step 4: real send ★ 成功
```
exit: 0
mode: send / dry_run: False / send_allowed: True
sent_count:                    1
line_api_call_count:           1
stub_invocations:              0
partial_delivery:              False
token_read_count:              1
production_callable_built:     True
hq_approved_send:              True
max_chunks:                    1
test_message_only:             True
payload_chunks_total:          1
forbidden_phrase_count:        0
safety_footer_present:         True
selected_row_count:            0
production_outcomes:
  - chunk_index=0
    status=ok
    dry_run_line_api=False  ← 実 push_message
    chunk_length=234
refused_reasons: []
```

### Step 5: token leak / DB unchanged ✅
- TOKEN_LEAK_HITS: 0 (4 artifact 内 grep) ★
- RECIPIENT_HITS_IN_ARTIFACTS: 2 (= production_config.recipient_id /
  production_outcomes.to に full 記録、F062-R3 設計通り)
- DB mtime: data/fire.db / fire.develop.db / fire.staging.db **全 unchanged**
- LINE log (logs/notifications/notifications_line.log) に
  `SEND <recipient> [FIRE] LINE 送信導通テスト (F062-R3) ...` を 1 行追記

## 安全要件遵守 (本 4 回目試行 = 成功時)

| 項目 | 結果 |
|---|---|
| 送信は 1 通                                         | ✅ |
| --test-message-only / --max-chunks 1                | ✅ |
| --send / --hq-approved-send                         | ✅ |
| recipient_id は Fujiwara 個人宛 (先頭 'U')          | ✅ |
| token 平文出力なし (artifact / log / chat)          | ✅ |
| partial_delivery=False (= retry 不要、重複なし)     | ✅ |
| forbidden_phrase 0 / safety_footer 全 8 行          | ✅ |
| 通常 Advisory payload (銘柄候補 / 価格 / 数量 / 執行) 送らない | ✅ |
| 自動発注                                          | 0 ✅ |
| 楽天証券操作                                       | 0 ✅ |
| Computer Use                                       | 0 ✅ |
| DB write                                           | 0 ✅ |
| 3 DB mtime                                         | 全 unchanged ✅ |
| TODO Excel                                         | 未更新 ✅ |
| scripts/seed_pattern_layer1.py                     | 未接触 ✅ |
| simulation/research_lane/historical_indicators.py  | 未接触 ✅ |
| unrelated modified を stage / commit               | しない ✅ |
| --no-verify                                        | 不使用 ✅ |
| Codex pre-commit (docs commit のため対象外)        | ✅ |

## 送信内容 (= test-message-only の固定文面)

production_outcomes.send_text_preview に preview として記録された冒頭 50 文字:

```
[FIRE] LINE 送信導通テスト (F062-R3)
本メッセージは HQ 承認済みの単発疎通
```

実際の chunk 全文 (chunk_length=234) は `_TEST_MESSAGE_TEMPLATE`
(scripts/jobs/run_f062_line_production_send_smoke.py の固定文字列):

```
[FIRE] LINE 送信導通テスト (F062-R3)
本メッセージは HQ 承認済みの単発疎通テストです。
Safety
- LINE 送信なし (dry-run / template only)
- 自動発注なし
- 楽天操作なし
- Computer Use なし
- 注文価格・数量・執行指示は生成しない
- F119 の優位は 20d 寄り (daytrade 即時判断には使わない)
- Fujiwara manual review required
```

→ 銘柄候補 / 価格 / 数量 / 執行指示 を **構造的に含まない** test message。

## LINE app での Fujiwara 受信確認 ★ 確認済み (2026-05-10)

22:49 JST (= 2026-05-10T13:49:35 UTC) に送信した test message を
Fujiwara が LINE app で **1 通受信を確認済み**:

| 項目 | 結果 |
|---|---|
| Fujiwara LINE app 1 通受信       | ✅ 確認済み |
| 通常 Advisory 本番送信は未開始    | ✅ 確認済み |
| test-message-only の 1 通のみ     | ✅ 確認済み |
| 銘柄候補 / 注文価格 / 数量 / 執行指示は未送信 | ✅ 確認済み |
| 重複送信なし                     | ✅ 確認済み |

これにより **F062-R4 タスクは正式完了**。受信確認後の追加 LINE 送信
は **一切実行していない** (= runner を呼んでいない、文字数 0、
LineBotClient.send_text 未呼出)。

## DB 不変確認

| DB | pre / post mtime | 不変 |
|---|---|---|
| data/fire.db          | May  7 16:12:38 / May  7 16:12:38 | ✅ |
| data/fire.develop.db  | May  7 18:14:26 / May  7 18:14:26 | ✅ |
| data/fire.staging.db  | May 10 18:22:36 / May 10 18:22:36 | ✅ |

## 通常 Advisory 本番送信は未開始である確認

- `--test-message-only` で payload chunks を 1 個の固定 test message
  に置換 (= selected_rows=[] / selected_count=0 / send_intent=
  "f062-r3 hq approved test message")
- 銘柄候補 / 注文価格 / 数量 / 執行指示は構造的に含まれない
- production_outcomes 1 件の chunk_length=234 (= 上記固定文の長さ)
- 通常 Advisory 本番送信は **完全に未開始**

## 軽微改善候補 (F062-R5 後検討、本タスクでは修正しない)

1. **build_production_send_callable に ASCII / latin-1 encode 事前検査
   を追加**: 今回 3 回目試行で発覚した U+2028 混入問題を、HTTP 層に
   到達する前に refuse できるようにする
2. **production_config.recipient_id を output_json に full 記録している件**:
   token は length のみだが recipient は full。F062-R5 後に length /
   prefix のみへの絞り込み検討

## 次タスク

1. **F062-R4 正式完了** ★ (本書記録時点)
2. 次候補: **F062-R5 First Production Advisory Small Launch**
   (max_chunks 1〜2、少数候補、手動レビュー前提)
   ★ ただし F062-R5 は **HQ (Fujiwara) 判断後に開始**。本タスク
   完了報告だけで自動的に F062-R5 に進まない。
3. 並走候補 (= F062-R5 と独立に進行可能):
   - 軽微改善候補 1, 2 の解消 (channel_token ASCII guard /
     recipient_id length-only logging)
   - F286-DATA-R3 daily refresh の cron 化 (post_launch_high_priority)
   - F242 OpenClaw / F022 FIRE Runner / F013 launchd
     (post_launch_high_priority)
