---
id: F062-R5.4
phase: P5: 通知 / 第 14 章 LINE 通知配信 / 第 19 章 R-19-08
priority: 最優先
status: 完了 ★ (2026-05-11、card mode 1 chunk 本番送信成功)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R5.2 (= 初回 production + compact 本番送信成功)
  - F062-R5.3 (= card mode 実装 + Codex CRITICAL 2 件対応)
  - F286-DATA-R1.5 (= r2f4_baseline_live_v1 / 2026-05-09 staging 書込)
  - F286-DATA-R1.6 (= F119 historical artifacts wiring)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 26 章
---

# F062-R5.4: Compact Production Advisory Send (Card Mode)

最終更新: 2026-05-11

## ★ 状態: 完了

F062-R5.3 で実装した card mode (= 判断カード化 / chunk_length 955→
491、48.6% 短縮 / 入力全体ベース結論 / Data Gate PASS 動的反映) を
使い、Fujiwara 個人 LINE app へ **判断カード形式の本番 Advisory を
1 通だけ送信成功**。chunk_length=492、partial_delivery=False、leak 0、
DB 全 unchanged。

## 実施結果

### Step 1: env / gate / staging 状態 ✅

| 項目 | 結果 |
|---|---|
| LINE_CHANNEL_TOKEN_SET     | True (length=516 / ASCII / no whitespace) |
| FIRE_LINE_RECIPIENT_ID_SET | True (prefix='U' / length=33) |
| DATA-R2 overall            | pass ✅ |
| line_send_allowed          | True ✅ |
| gate-1-prices              | pass (max=2026-05-08 lag=1 / codes=4448) |
| gate-2-signals             | pass (max_base=2026-05-09 lag=0 / codes=110) |
| gate-3-index               | pass |
| gate-4-derived             | pass |
| gate-5-other               | pass |
| staging r2f4_baseline_live_v1 / 2026-05-09 rows | 109 (= F286-DATA-R1.5 末尾状態維持) |

### Step 2: card mode payload 再生成 ✅

実行:
```
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/f286_data_r1_6_advisory_preview.json \
  --summary-json /tmp/f286_data_r1_6_advisory_summary.json \
  --gate-json /tmp/f062_r5_4_gate.json \
  --message-mode production --card-mode --card-top 5 \
  --output-json /tmp/f062_r5_4_line_payload.json \
  --output-text /tmp/f062_r5_4_line_preview.txt \
  --output-summary-json /tmp/f062_r5_4_line_summary.json
```

| field | value |
|---|---|
| message_mode                       | production ✅ |
| compact                            | False |
| card_mode                          | True ✅ |
| metadata.card_top                  | 5 |
| metadata.source_version            | r2f4_baseline_live_v1 |
| metadata.rule_version              | r2g3_recommended_v2 |
| metadata.payload_base_date         | 2026-05-09 |
| chunks                             | 1 |
| chunk[0] length                    | **492** ★ (F062-R5.2 の 955 から **48.5% 短縮**) |
| forbidden_phrase_count             | 0 |
| safety_footer_present              | True |
| selected_count                     | 5 |
| selected_label_counts              | boost_with_avoid: 5 (= LABEL_PRIORITY Top 5) |

chunk[0] 冒頭 8 行 (= 判断カードの「3 秒判断」レイアウト):
```
FIRE 本番Advisory
Data Gate PASS               ← gate JSON から動的反映 (CRITICAL #1 修正)
base_date: 2026-05-09
source: r2f4_baseline_live_v1 / r2g3_recommended_v2

🟢 結論: 買い検討候補あり    ★ 入力全体 30 候補中 boost 30 で反映
買い検討 30 / 注意 0 / 見送り 0  ← 入力全体ベース (CRITICAL #2 修正)
```

### Step 3: dry-run (--send 無し) ✅

| field | value |
|---|---|
| mode                       | dry_run |
| send_allowed               | True |
| sent_count                 | 0 |
| line_api_call_count        | 0 |
| token_read_count           | 0 |
| forbidden_phrase_count     | 0 |
| safety_footer_present      | True |
| manual_review_required_count | 5 |
| auto_order_allowed_true_count | 0 |

### Step 4: real send ★

実行:
```
source ~/.fire_secrets/line.env && .venv/bin/python -m \
  scripts.jobs.run_f062_line_production_send_smoke \
  --payload-json /tmp/f062_r5_4_line_payload.json \
  --gate-json    /tmp/f062_r5_4_gate.json \
  --send --hq-approved-send \
  --recipient-id "$FIRE_LINE_RECIPIENT_ID" \
  --max-chunks 1 \
  --output-json       /tmp/f062_r5_4_first_card_advisory_send.json \
  --completion-report /tmp/f062_r5_4_first_card_advisory_send_report.txt
```

| field | value |
|---|---|
| exit                       | 0 |
| mode                       | send |
| dry_run                    | False |
| send_allowed               | True |
| **sent_count**             | **1** ★ |
| **line_api_call_count**    | **1** ★ |
| partial_delivery           | **False** ★ |
| token_read_count           | 1 |
| production_callable_built  | True |
| hq_approved_send           | True |
| max_chunks                 | 1 |
| **dry_run_line_api**       | **False** (= 実 push_message 呼出) |
| forbidden_phrase_count     | 0 ✅ |
| safety_footer_present      | True ✅ |
| manual_review_required_count | 5 ✅ |
| auto_order_allowed_true_count | 0 ✅ |
| production_outcomes        | 1 件 (chunk_index=0 / **status=ok** / dry_run_line_api=False / chunk_length=492 / recipient_type=user / recipient_hash8=b344b213) |
| **payload_freshness_check** | max_lag_days=10 / payload_base_date=2026-05-09 / gate_signal_max_base_date=2026-05-09 / **lag_calendar_days=0** ✅ |

### Step 5: token / recipient leak 検査 ✅

artifact 対象:
- `/tmp/f062_r5_4_*.{json,txt}`
- `/tmp/f286_data_r1_6_*.{json,csv}` (= advisory_preview 入力)
- `logs/notifications/notifications_line.log`

結果:
- **TOKEN_LEAK: 0** ✅
- **FULL_RECIPIENT: 0** ✅

LINE log tail (recipient masked):
```
2026-05-11T05:27:26.926695+00:00  SEND  user:prefix=U:len=33:hash8=b344b213
   FIRE 本番Advisory\nData Gate PASS\nbase_date: 2026-05-09\n
   source: r2f4_baseline_live_v1 / r2g3_recommended_v2\n\n
   🟢 結論: 買い検討候補あり\n買い検討 30 / 注意 0 / 見送り 0\n\n
   🟠 強弱混在・慎重 57290\n  鉄鋼・非鉄 / 月5\n\n🟠 強弱混在・慎重 340A...
```

### Step 6: DB 不変 ✅

| DB | mtime / size | 不変 |
|---|---|---|
| data/fire.db          | May  7 16:12:38 2026 / 371 MB     | ✅ |
| data/fire.develop.db  | May  7 18:14:26 2026 / 371 MB     | ✅ |
| data/fire.staging.db  | May 11 11:48:57 2026 / 4.8 GB     | ✅ (= F286-DATA-R1.5 末尾状態維持) |

## 安全要件遵守 (= 全 ✅)

| 項目 | 結果 |
|---|---|
| 送信は 1 通だけ                                              | ✅ |
| --card-mode + --card-top 5                                  | ✅ |
| --max-chunks 1                                              | ✅ |
| --send + --hq-approved-send                                 | ✅ |
| DATA-R2 gate pass                                           | ✅ |
| token / recipient 平文出力                                   | 0 ✅ |
| partial_delivery=True 時 retry なし                          | False (retry 不要) ✅ |
| 自動発注 / 楽天操作 / Computer Use                            | 0 ✅ |
| 注文価格 / 数量 / 執行指示                                    | 送信していない ✅ |
| TODO Excel                                                  | 未更新 ✅ |
| --no-verify                                                 | 不使用 ✅ |
| scripts/seed_pattern_layer1.py / historical_indicators.py    | 未接触 ✅ |
| unrelated modified を stage / commit                         | しない ✅ |

## Fujiwara LINE app 受信確認 (依頼)

- 送信時刻: **2026-05-11T05:27:26 UTC** (= 2026-05-11 14:27 JST)
- 送信件数: **1 件のみ** (= 重複なし)
- chunk 内容: card mode、length=492

冒頭 (= LINE log preview):
```
FIRE 本番Advisory
Data Gate PASS
base_date: 2026-05-09
source: r2f4_baseline_live_v1 / r2g3_recommended_v2

🟢 結論: 買い検討候補あり
買い検討 30 / 注意 0 / 見送り 0

🟠 強弱混在・慎重 57290
  鉄鋼・非鉄 / 月5
... (Top 5)

Safety
- 本番 LINE 通知 (production send)
- 自動発注なし / 楽天操作なし / Computer Use なし
- 注文価格・数量・執行指示は生成しない
- F119 の優位は 20d 寄り
- Fujiwara manual review required
```

Fujiwara の LINE app に **1 通だけ** 届いていることを確認後、文面の
読みやすさ / 重複なし確認をご報告ください。

## F062-R5 シリーズの送信履歴

| 日時 (JST) | 内容 | chunk_length |
|---|---|---|
| 2026-05-10 22:49 (F062-R4) | test-message-only           | 234 |
| 2026-05-11 01:50 (F062-R5) | F119 未 wired neutral only  | 1,892 |
| 2026-05-11 13:01 (F062-R5.2) | F119 wired compact         | 955 |
| **2026-05-11 14:27 (F062-R5.4)** | **F119 wired card** ★ | **492** |

## 注意: F282 weekly snapshot リスク

次回月曜 (2026-05-18 07:00 JST) に F282 weekly staging snapshot が
再実行され、r2f4_baseline_live_v1 / 2026-05-09 の 109 行は再消失予定。
定常運用には FIRE-OPS-R0 再発防止策案 1 (= 本番運用データを production
fire.db に書く運用統一) の実装が推奨。

## 次タスク

1. ★ F062-R5.4 完了 (= 本書、card mode 1 chunk 本番送信成功)
2. **Fujiwara が LINE app で 1 通受信確認** → 判断カードの読みやすさ
   レビュー
3. 受信成功確認後:
   - F286-PNL-R1 Advisory Decision / Actual PnL Tracking 設計
4. 並走候補:
   - FIRE-OPS-R0 再発防止策案 1 設計 (= production write 統一)
   - 03_design/F282_environment_isolation_*.md の運用ルール明文化
   - F286-DATA-R3 daily refresh cron 化
