---
id: F062-R5.6
phase: P5: 通知 / 第 14 章 LINE 通知配信 / 第 19 章 R-19-08
priority: 最優先
status: 完了 ★ (2026-05-11、銘柄名付き buyability card 1 chunk 本番送信成功)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R5.4 (= card mode 本番送信、chunk_length=492)
  - F062-R5.5 (= buyability mode 実装)
  - F286-DATA-R1.7 (= code → company_name read-only enrichment)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 25 章 / 第 26 章
---

# F062-R5.6: Buyability Card Production Send with Names

最終更新: 2026-05-11

## ★ 状態: 完了

F286-DATA-R1.7 で実装した name enrichment と F062-R5.5 buyability
mode を組み合わせ、Fujiwara 個人 LINE app へ **銘柄名付きの実用判断
カード本番 Advisory を 1 通だけ送信成功**。chunk_length=1,120、
partial_delivery=False、leak 0、3 DB 全 unchanged。

## 実施結果

### Step 1: env 検査 ✓

| 項目 | 結果 |
|---|---|
| LINE_CHANNEL_TOKEN_SET   | True (length=516 / ASCII / no whitespace) |
| FIRE_LINE_RECIPIENT_ID_SET | True (prefix='U' / length=33 / ASCII) |

### Step 2: DATA-R2 gate 再取得 ✓

`scripts.jobs.run_data_freshness_gate --db-path data/fire.staging.db
--source-version r2f4_baseline_live_v1`

| 項目 | 結果 |
|---|---|
| overall_status                | pass ✓ |
| line_send_allowed             | True ✓ |
| reasons                       | [] |
| gate-1-prices                 | pass (max=2026-05-08 lag=1 / codes=4448) |
| gate-2-signals                | pass (max_base=2026-05-09 lag=0 / codes=109) |
| gate-3-index                  | pass |
| gate-4-derived                | pass |
| gate-5-other                  | pass |

### Step 3: name-enriched buyability payload 生成 ✓

```
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/f286_data_r1_6_advisory_preview.json \
  --summary-json /tmp/f286_data_r1_6_advisory_summary.json \
  --gate-json    /tmp/f062_r5_6_gate.json \
  --listings-db  data/fire.staging.db \
  --message-mode production --buyability-mode --card-top 5 \
  --output-json    /tmp/f062_r5_6_line_payload.json \
  --output-text    /tmp/f062_r5_6_line_preview.txt \
  --output-summary-json /tmp/f062_r5_6_line_summary.json
```

| field | value |
|---|---|
| message_mode                       | production ✓ |
| buyability_mode                    | True ✓ |
| metadata.card_mode (effective)     | True ✓ |
| metadata.card_top                  | 5 |
| metadata.source_version            | r2f4_baseline_live_v1 |
| metadata.rule_version              | r2g3_recommended_v2 |
| metadata.payload_base_date         | 2026-05-09 |
| metadata.name_enrichment.attempted | 30 |
| metadata.name_enrichment.enriched  | **30** ★ (= 全件成功) |
| metadata.name_enrichment.missing   | 0 |
| metadata.name_enrichment.cache_size | 4,449 |
| chunks                             | 1 |
| chunk[0] length                    | **1,120** ★ |
| forbidden_phrase_count             | 0 |
| safety_footer_present              | True |
| selected_count                     | 5 |
| selected_label_counts              | boost_with_avoid: 5 |

Top 5 candidates (= LINE 文面):
```
1. 57290 日本精鉱
2. 340A0 ジグザグ
3. 37980 ＵＬＳグループ
4. 137A0 Ｃｏｃｏｌｉｖｅ
5. 331A0 メディックス
```

### Step 4: dry-run smoke ✓

`run_f062_line_production_send_smoke --payload-json ... --gate-json ...
--recipient-id ... --max-chunks 1` (= `--send` なし)

| field | value |
|---|---|
| mode                       | dry_run |
| sent_count                 | 0 |
| line_api_call_count        | 0 |
| token_read_count           | 0 |
| production_callable_built  | False (= --send なしで callable 未構築) |

### Step 5: real send ★

```
source ~/.fire_secrets/line.env && .venv/bin/python -m \
  scripts.jobs.run_f062_line_production_send_smoke \
  --payload-json /tmp/f062_r5_6_line_payload.json \
  --gate-json    /tmp/f062_r5_6_gate.json \
  --send --hq-approved-send \
  --recipient-id "$FIRE_LINE_RECIPIENT_ID" \
  --max-chunks 1 \
  --output-json       /tmp/f062_r5_6_real_send.json \
  --completion-report /tmp/f062_r5_6_real_send_report.txt
```

| field | value |
|---|---|
| mode                       | send |
| dry_run                    | False |
| **sent_count**             | **1** ★ |
| **line_api_call_count**    | **1** ★ |
| stub_invocations           | 0 |
| **partial_delivery**       | **False** ★ |
| token_read_count           | 1 |
| production_callable_built  | True |
| hq_approved_send           | True |
| max_chunks                 | 1 |
| **dry_run_line_api**       | **False** (= 実 push_message 呼出) |
| production_outcomes        | 1 件 (label=f062_r3_smoke / chunk_index=0 / **send_text_status=ok** ★ / chunk_length=1120 / recipient_type=user / recipient_hash8=b344b213) |
| **payload_freshness_check** | max_lag_days=10 / payload_base_date=2026-05-09 / gate_signal_max_base_date=2026-05-09 / **lag_calendar_days=0** ✓ |

### Step 6: token / recipient leak 検査 ✓

artifact:
- `/tmp/f062_r5_6_*.{json,txt}`
- `logs/notifications/notifications_line.log`

結果:
- **TOKEN_LEAK: 0** ✓
- **FULL_RECIPIENT: 0** ✓

LINE log tail (recipient masked、token 平文なし):
```
2026-05-11T08:02:32  SEND  user:prefix=U:len=33:hash8=b344b213
   FIRE 本番Advisory\nData Gate PASS\nbase_date: 2026-05-09\n
   source: r2f4_baseline_live_v1 / r2g3_recommended_v2\n\n
   🟠 結論: 買い検討候補あり。ただし慎重寄り\n...
```

### Step 7: DB 不変 ✓

| DB | mtime / size | 不変 |
|---|---|---|
| data/fire.db          | May  7 16:12 / 371 MB     | ✓ |
| data/fire.develop.db  | May  7 18:14 / 371 MB     | ✓ |
| data/fire.staging.db  | May 11 11:48 / 4.8 GB     | ✓ (= R1.5 末尾維持) |

## 安全要件遵守 (= 全 ✓)

| 項目 | 結果 |
|---|---|
| 送信は 1 通だけ (--max-chunks 1)                              | ✓ |
| --buyability-mode + --card-top 5                              | ✓ |
| --listings-db data/fire.staging.db (read-only)                | ✓ |
| --send + --hq-approved-send                                   | ✓ |
| DATA-R2 gate pass                                             | ✓ |
| token / recipient 平文出力                                     | 0 ✓ |
| partial_delivery=True 時 retry なし                            | False (retry 不要) ✓ |
| 自動発注 / 楽天操作 / Computer Use                              | 0 ✓ |
| 注文価格 / 数量 / 執行指示                                      | 送信していない ✓ |
| name 既存値の上書き                                            | 0 (already_has_name=0) ✓ |
| 3 DB 全 mtime unchanged                                       | ✓ |
| TODO Excel                                                    | 未更新 ✓ |
| --no-verify                                                  | 不使用 ✓ |
| scripts/seed_pattern_layer1.py                                | 未接触 ✓ |
| simulation/research_lane/historical_indicators.py             | 未接触 ✓ |
| unrelated modified を stage / commit                          | しない ✓ |

## Fujiwara LINE app 受信確認 (依頼)

- 送信時刻: **2026-05-11T08:02:32 UTC** (= 2026-05-11 17:02 JST)
- 送信件数: **1 件のみ** (= 重複なし)
- chunk 内容: buyability mode (= 銘柄名付き判断カード)、length=1,120

冒頭 (= LINE log preview):
```
FIRE 本番Advisory
Data Gate PASS
base_date: 2026-05-09
source: r2f4_baseline_live_v1 / r2g3_recommended_v2

🟠 結論: 買い検討候補あり。ただし慎重寄り
boost はあるが avoid 条件も混在。
寄り後の値動きで判断。

買い検討 30 / 注意 0 / 見送り 0

🟠 強弱混在・慎重 1. 57290 日本精鉱
判定: 強弱混在・慎重
理由: F119 boost month_of_year / avoid sector_17 一部該当
見る点: 寄り後の値動き優先 / avoid 影響度

🟠 強弱混在・慎重 2. 340A0 ジグザグ
判定: 強弱混在・慎重
理由: F119 boost interpretation_sector_17_month / avoid top_bucket_interpretation_sector_17 一部該当
見る点: 寄り後の値動き優先 / avoid 影響度

🟠 強弱混在・慎重 3. 37980 ＵＬＳグループ ...
🟠 強弱混在・慎重 4. 137A0 Ｃｏｃｏｌｉｖｅ ...
🟠 強弱混在・慎重 5. 331A0 メディックス ...

Safety (= production marker 含む 8 行)
```

Fujiwara の LINE app に **1 通だけ** 届いていることを確認後、
銘柄名 + 判定 + 理由 + 見る点 の読みやすさ / 重複なし をご報告ください。

## F062-R5 シリーズ送信履歴

| 日時 (JST) | 内容 | chunk_length |
|---|---|---|
| 2026-05-10 22:49 (F062-R4)    | test-message-only             | 234 |
| 2026-05-11 01:50 (F062-R5)    | F119 未 wired neutral only    | 1,892 |
| 2026-05-11 13:01 (F062-R5.2)  | F119 wired compact            | 955 |
| 2026-05-11 14:27 (F062-R5.4)  | F119 wired card               | 492 |
| **2026-05-11 17:02 (F062-R5.6)** | **buyability + names enrichment** ★ | **1,120** |

## 注意: F282 weekly snapshot リスク

次回月曜 (2026-05-18 07:00 JST) に F282 weekly staging snapshot が
再実行され、`r2f4_baseline_live_v1 / 2026-05-09` の 109 行は再消失
予定。`market_listings` は production 側でも保持されるため、name
enrichment は失敗しない (= production fire.db に切り替えれば lookup
も同じ結果)。advisory_preview の再生成のみが必要。

定常運用には FIRE-OPS-R0 再発防止策案 1 (= 本番運用データを
production fire.db に書く運用統一) の実装が推奨。

## 次タスク

1. ★ F062-R5.6 完了 (= 本書、銘柄名付き buyability 1 chunk 送信成功)
2. **Fujiwara が LINE app で 1 通受信確認** → 銘柄名 + 判定 + 理由 +
   見る点 の読みやすさレビュー
3. 受信成功確認後:
   - F286-PNL-R1 Advisory Decision / Actual PnL Tracking 設計
4. 並走候補:
   - F286-DATA-R1.8: F111-R4 persistence に --listings-db 追加 (=
     上流で name を埋める根本対応)
   - FIRE-OPS-R0 再発防止策案 1 設計 (= production write 統一)
   - 03_design/F282_environment_isolation_*.md の運用ルール明文化
   - F286-DATA-R3 daily refresh cron 化
