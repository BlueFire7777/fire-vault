---
id: F062-R4
phase: P5: 通知 / 第 14 章 LINE 通知配信
priority: 最優先
status: 停止 (HQ 環境変数未提供のため、real send 未実施。dry-run / real send 共に実行していない)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R3 (LINE production send path 構造的安全性 PASS)
  - F286-DATA-R2 (freshness gate 全 5 段 PASS)
  - FIRE-TODO-R1 (pre_launch_required 0 件確認)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 26 章
---

# F062-R4: First Real LINE Send Smoke

最終更新: 2026-05-10

## 状態: 停止 (HQ 環境変数未提供)

本タスクは「実 LINE API へ 1 通だけ test-message-only を手動発火する
初回 real send smoke」だが、必要な環境変数が **HQ (Fujiwara) から
セッションへ提供されていない** ため、タスク仕様の停止条件に従って
**dry-run / real send を共に実行せず停止** した。

## 環境変数チェック結果

|  | set? | length | prefix |
|---|---|---|---|
| LINE_CHANNEL_TOKEN     | False | 0 | (n/a) |
| FIRE_LINE_RECIPIENT_ID | False | 0 | (n/a) |

両方未設定のため、停止条件 (= token / recipient 未設定で即停止) に
該当。タスク仕様 (= 「チャット上で token を要求しない」「token を
ログに出さない」) を遵守し、Fujiwara への直接要求はしていない。

## 実施した検証

### 1. 必要 artifact 存在確認 ✅
- `/tmp/f062_r1_line_preview_payload.json`     74K  (F062-R1 payload)
- `/tmp/f286_data_r1_3_gate_after.json`         3.3K (DATA-R2 pass gate)
- `/tmp/f062_r3_completion_report.txt`          10K (F062-R3 完了報告)

### 2. DATA-R2 gate JSON 検証 ✅

| field | value |
|---|---|
| overall_status     | pass ✅ |
| line_send_allowed  | True ✅ |
| as_of_date         | 2026-05-08 |
| db_label           | staging |
| allow_warning      | False |
| reasons            | [] (= 全 PASS、refuse 理由 0 件) |
| gate-1 (prices)    | status=pass ✅ |
| gate-2 (signals)   | status=pass ✅ |
| gate-3 (index)     | status=pass ✅ |
| gate-4 (derived)   | status=pass ✅ |
| gate-5 (other)     | status=pass ✅ |

### 3. 環境変数チェック ✗ (停止条件 hit)

LINE_CHANNEL_TOKEN / FIRE_LINE_RECIPIENT_ID 両方未設定。

「停止して報告」の方針に従い、以下を **実行していない**:
- 事前 dry-run (`scripts/jobs/run_f062_line_production_send_smoke.py` の dry-run mode)
- real send smoke (`--send --hq-approved-send ...` 経路)

理由:
- タスク仕様「停止条件」: LINE_CHANNEL_TOKEN / FIRE_LINE_RECIPIENT_ID 未設定で即停止
- タスク仕様「最重要制約」: token を画面表示・要求しない / 画面に出さない
- token なしでも dry-run は構造的に可能だが、recipient 未設定の場合
  `--recipient-id ""` を渡してしまうと argparse で空文字列が通り、
  runner 側のガードに依存する。タスク仕様は「両方未設定なら停止」と
  明記しているため、最も保守的に dry-run も実施しない判断とした。

## 実施した安全要件遵守

| 要件 | 結果 |
|---|---|
| 送信は 0 通 (= token / recipient 未設定で停止) | ✅ |
| chat 上で token を要求しない | ✅ |
| token をログ / report / JSON に出力しない | ✅ (= そもそも送信していない) |
| LINE 本番送信なし | ✅ |
| 自動発注なし | ✅ |
| 楽天証券操作なし | ✅ |
| Computer Use なし | ✅ |
| DB write なし | ✅ |
| TODO Excel 未更新 | ✅ |
| scripts/seed_pattern_layer1.py 未接触 | ✅ |
| simulation/research_lane/historical_indicators.py 未接触 | ✅ |
| unrelated modified file を stage / commit しない | ✅ |
| --no-verify 不使用 | ✅ |
| Codex pre-commit (docs commit のため対象外) | ✅ |

## DB 不変確認

| DB | mtime |
|---|---|
| data/fire.db          | 5月 7 16:12:38 (前回値と同じ) ✅ |
| data/fire.develop.db  | 5月 7 18:14:26 (前回値と同じ) ✅ |
| data/fire.staging.db  | 5月 10 18:22:36 (前回値と同じ) ✅ |

本タスクで一切 DB 書き込みなし。

## token leak 確認

- 本タスクで artifact 新規作成なし (= dry-run / real send 未実施)
- 既存 /tmp/f062_r3_*.{json,txt} は F062-R3 で生成済み、本タスクで触らない
- 過去 F062-R3 で `grep -l "DUMMY_TOKEN_FOR_SMOKE" /tmp/f062_r3_*.{json,txt}` → 0 件
  (= F062-R3 でも leak 0 確認済)

## 実送信内容の確認

- 送信していない (= 0 通)
- test-message-only でも本物 token / recipient が無いと発火不可
- 通常 Advisory payload (= 銘柄候補一覧 / 注文価格 / 数量 / 執行指示) を
  送ろうとしていない
- 本タスクで送信を試行する箇所はなく、通常 Advisory 送信は **未開始**

## HQ への提供依頼 (= タスク再開条件)

本タスクを再開するには、以下 2 つの環境変数を Fujiwara が **直接 shell
session に export** する必要がある (= チャット経由では受け取らない):

```
# .env や ~/.zshrc に書くのが最も安全
export LINE_CHANNEL_TOKEN='...'             # LINE Messaging API channel access token
export FIRE_LINE_RECIPIENT_ID='Uxxxxxxxx...' # Fujiwara 個人 LINE userId (先頭 'U')
```

設定確認方法 (値は表示しない):

```
python - <<'PY'
import os
t = os.environ.get("LINE_CHANNEL_TOKEN")
r = os.environ.get("FIRE_LINE_RECIPIENT_ID")
print("token_set:", bool(t), "len:", len(t) if t else 0)
print("recipient_set:", bool(r), "prefix:", r[:1] if r else "", "len:", len(r) if r else 0)
PY
```

その後、新しい Claude Code session で改めて F062-R4 タスクを起動する
ことで、本書の手順 5-6 (= dry-run + real send) を実行できる。

## 再実行コマンド (HQ 提供後)

### 5. 事前 dry-run

```
.venv/bin/python -m scripts.jobs.run_f062_line_production_send_smoke \
  --payload-json /tmp/f062_r1_line_preview_payload.json \
  --gate-json    /tmp/f286_data_r1_3_gate_after.json \
  --recipient-id "$FIRE_LINE_RECIPIENT_ID" \
  --max-chunks 1 --test-message-only \
  --output-json       /tmp/f062_r4_pre_real_send_dryrun.json \
  --completion-report /tmp/f062_r4_pre_real_send_dryrun_report.txt
```

期待: exit 0 / sent=0 / api=0 / token_read=0 / forbidden 0 / safety footer ✓

### 6. 初回 real LINE send smoke

```
.venv/bin/python -m scripts.jobs.run_f062_line_production_send_smoke \
  --payload-json /tmp/f062_r1_line_preview_payload.json \
  --gate-json    /tmp/f286_data_r1_3_gate_after.json \
  --send --hq-approved-send \
  --recipient-id "$FIRE_LINE_RECIPIENT_ID" \
  --max-chunks 1 --test-message-only \
  --output-json       /tmp/f062_r4_first_real_send.json \
  --completion-report /tmp/f062_r4_first_real_send_report.txt
```

期待: exit 0 / sent=1 / line_api_call_count=1 / partial_delivery=False
/ token leak 0 / DB unchanged

## 次タスク提案

1. **HQ (Fujiwara) が LINE_CHANNEL_TOKEN + FIRE_LINE_RECIPIENT_ID を
    shell session に export する** (= チャット送信ではなく `~/.zshrc` 等)
2. F062-R4 タスクを再起動 → 上記 5-6 を実行
3. real LINE 受信を Fujiwara が iSPEED / LINE app で確認
4. 成功時: F062-R5 First Production Advisory Small Launch
   (max_chunks 1〜2、少数候補、手動レビュー前提)
