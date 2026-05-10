---
id: F062-R4
phase: P5: 通知 / 第 14 章 LINE 通知配信
priority: 最優先
status: 停止 (token に U+2028 混入で UnicodeEncodeError、token sanitize が必要)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R3 (LINE production send path 構造的安全性 PASS)
  - F286-DATA-R2 (freshness gate 全 5 段 PASS)
  - FIRE-TODO-R1 (pre_launch_required 0 件確認)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 26 章
---

# F062-R4: First Real LINE Send Smoke

最終更新: 2026-05-10 (3 回目試行)

## 試行履歴

| 試行 | 状態 | 主因 |
|---|---|---|
| 1 回目 | 停止 | env 未提供 |
| 2 回目 | 停止 | LINE API 401 invalid_token (旧 token) |
| 3 回目 | **停止 (本書)** | UnicodeEncodeError (token に U+2028 混入) |

## 状態: 停止 (token に U+2028 LINE SEPARATOR 混入)

3 回目試行で env から token / recipient は再度正しく読み込まれた:

| 項目 | 結果 |
|---|---|
| LINE_CHANNEL_TOKEN_SET     | True (length=518) ← 1 回目 197 から再発行で長くなった |
| FIRE_LINE_RECIPIENT_ID_SET | True (prefix='U', length=33) |
| LINE /v2/oauth/verify       | verify_status=200 (Fujiwara 既報告) |

しかし real send で **UnicodeEncodeError** が発生し sent=0 / partial=False
で停止。

### 文字構成検査 (= 値非表示、文字種統計のみ)

```
token length:                  518
is_ascii:                       False
encode_to_latin1_ok:            False
  bad char position:           172
  bad char: U+2028 (LINE SEPARATOR)
encode_to_utf8_ok:              True
non_ascii_char_count:           2 ← U+2028 が 2 文字混入
unicode_category top 5:
  Ll (lowercase letter):       222
  Lu (uppercase letter):       210
  Nd (decimal digit):           66
  Po (other punctuation):       12  ← U+2028 はここに含まれる
  Sm (math symbol):              6
```

### 何が起きたか

- LINE Developers Console から token を copy-paste した際、UI 構造によっては
  「不可視 Unicode 改行」(U+2028 LINE SEPARATOR、または U+2029 PARAGRAPH
  SEPARATOR) が **見えない形で混入** することがある
- token 値そのものは LINE 側に登録されているため `/v2/oauth/verify` は
  200 を返すが、HTTP ヘッダの **Authorization: Bearer ...** を送る際に
  Python の http.client / urllib3 は Latin-1 (ISO-8859-1) で encode する
  ため、U+2028 (= 0x2028 > 0xFF) は **ordinal 範囲外で encode 失敗**
- LineBotClient.send_text 内で `push_message(...)` を呼ぶ前段階で
  UnicodeEncodeError が raise → F062-R3 sender が logger.error 後に
  propagate → F062-R2 router が partial_delivery=False で SendResult
  返却 → exit 4

## 実施結果

### Step 1: env 確認 ✅
LINE_CHANNEL_TOKEN length=518 (= 1 回目 197 から再発行)、
FIRE_LINE_RECIPIENT_ID prefix='U' length=33。

### Step 2: gate ✅
overall=pass / line_send_allowed=True / 5 段全 PASS。

### Step 3: 事前 dry-run ✅
```
exit: 0 / mode: dry_run / send_allowed: True
sent=0 / api=0 / token_read=0 / production_callable_built: False
forbidden_phrase_count: 0 / safety_footer_present: True
manual_review_required_count: 0
```

### Step 4: real send ✗ (UnicodeEncodeError)
```
exit: 4
mode: send / dry_run: False / send_allowed: False
sent_count: 0
line_api_call_count: 0
partial_delivery: False  ← retry 可
token_read_count: 1
production_callable_built: True  ← config 構築 OK (= str / 非空 / U/C/R prefix チェック PASS)
hq_approved_send: True / max_chunks: 1 / test_message_only: True
forbidden_phrase_count: 0 / safety_footer_present: True
selected_row_count: 0 / payload_chunks_total: 1
production_outcomes: []
refused_reasons:
  - chunk send failed: 0/1 sent before failure
  - production_send_callable raised at chunk_index=0:
    UnicodeEncodeError: 'latin-1' codec can't encode character
    ' ' in position 179: ordinal not in range(256)
```

### Step 5: token leak / DB unchanged
- TOKEN_LEAK_HITS: 0 (4 artifact 内 grep) ✅
- DB mtime: data/fire.db / fire.develop.db / fire.staging.db 全 unchanged ✅

## 安全要件遵守

| 項目 | 結果 |
|---|---|
| 送信は 0 通                                         | ✅ |
| partial_delivery=False (= retry 可)                | ✅ |
| token leak 0 (4 artifact)                           | ✅ |
| 通常 Advisory payload 送ろうとしていない            | ✅ |
| 自動発注 / 楽天操作 / Computer Use                  | 0 ✅ |
| DB write 0、3 DB 全 mtime unchanged                 | ✅ |
| TODO Excel                                          | 未更新 ✅ |
| scripts/seed_pattern_layer1.py / historical_indicators.py | 未接触 ✅ |
| unrelated modified                                  | 未 stage / 未 commit ✅ |
| --no-verify                                         | 不使用 ✅ |
| Codex pre-commit (docs commit のため対象外)         | ✅ |

## token sanitize 修正案 (要 Fujiwara 判断)

### 案 1 (推奨、最も安全): Fujiwara が手動で再貼り付け

1. LINE Developers Console から token を改めて copy
2. **plain text editor** (= TextEdit 「フォーマット → 標準テキストにする」、
   または `nano ~/.fire_secrets/line.env` で直接編集) を使う
3. 貼り付け時に「Paste and Match Style」(= ⌥⇧⌘V) で書式なし貼り付け
4. shell に直接貼るのが最も確実:
   ```
   nano ~/.fire_secrets/line.env
   # LINE_CHANNEL_TOKEN='...' の行を全て選択して再貼り付け
   ```
5. 貼り付け後、文字数検査で 516 (= 518 - 2 個の U+2028) になることを確認

### 案 2: Claude が tr で U+2028 を除去 (Fujiwara 承認後のみ)

```
# U+2028 (UTF-8: 0xe2 0x80 0xa8) を削除
tr -d $'\xe2\x80\xa8' < ~/.fire_secrets/line.env > /tmp/line.env.cleaned
# 値を画面に出さず、length 比較で sanitize 効果を確認
mv /tmp/line.env.cleaned ~/.fire_secrets/line.env
chmod 600 ~/.fire_secrets/line.env
```

これは「Claude が secret file を加工する」ことになるため、Fujiwara の
明示承認後のみ実施。本タスクでは **未実施**。

### 検証方法 (両案共通)

```
source ~/.fire_secrets/line.env && .venv/bin/python - <<'PY'
import os
t = os.environ.get("LINE_CHANNEL_TOKEN", "")
print("length:", len(t))
print("is_ascii:", t.isascii())
try:
    t.encode("latin-1")
    print("latin1_ok: True")
except UnicodeEncodeError as e:
    print(f"latin1_ok: False (bad pos={e.start})")
PY
```

期待: `is_ascii: True` / `latin1_ok: True`。

## 次タスク

1. Fujiwara が token 再貼り付け (案 1) か、Claude による tr sanitize 承認
   (案 2) を選択
2. token sanitize 後、length=516 (= 518 - 2) と is_ascii=True を確認
3. F062-R4 を再実行
4. sent_count=1 / line_api_call_count=1 / partial_delivery=False が出れば
   LINE app で Fujiwara 受信確認
5. 受信成功時: F062-R5 First Production Advisory Small Launch
   (max_chunks 1〜2、少数候補、手動レビュー前提)

## 観察された軽微改善候補 (本タスクでは修正しない、F062-R5 後検討)

- `assert_production_safe(cfg)` で channel_token に **ASCII / latin-1
  encode 可能** の事前検査を追加すれば、HTTP 層に到達する前に refuse
  できる。今回のような U+2028 混入は build_production_send_callable で
  検出されず、push_message 時に初めて検出された
- production_config.recipient_id を output_json に full 記録している件
  (前回試行でも記録済) は引き続き F062-R5 後の改善候補
