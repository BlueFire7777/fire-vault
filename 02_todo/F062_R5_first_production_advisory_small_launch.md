---
id: F062-R5
phase: P5: 通知 / 第 14 章 LINE 通知配信 / 第 19 章 R-19-08
priority: 最優先
status: 停止 (DATA-R2 gate=warning、announcements 鮮度低、HQ 判断要請中)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R4 (実 LINE API 1 通送信成功 + Fujiwara 受信確認)
  - F062-R4.1 (token ASCII preflight)
  - F062-R4.2 (output recipient masking)
  - F236-R1 (LineBotClient log mask)
  - F236-R1.1 (legacy LINE log sanitize)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 26 章
---

# F062-R5: First Production Advisory Small Launch

最終更新: 2026-05-11

## 状態: 停止 (DATA-R2 gate=warning)

実 Advisory 文面 (= 少数候補・注意点・データ鮮度の手動レビュー用)
を Fujiwara 個人宛に LINE で 1 chunk 送信する **初回本番 Advisory**
として企画。env preflight / git 状態 / token 検査は全て pass した
が、最新 staging で再実行した DATA-R2 gate が **gate-5-other (soft)
の announcements 鮮度低** で warning となり、`line_send_allowed=False`。

タスク仕様の停止条件「DATA-R2 gateがpassでない」「line_send_allowed
がtrueでない」に該当するため、**Advisory 送信を実施せず停止**。

**Fujiwara 受信通知は本タスクで発生しない**。

## 実施結果

### Step 1-2: env preflight ✅

| 項目 | 結果 |
|---|---|
| LINE_CHANNEL_TOKEN_SET     | True (length=516) |
| LINE_CHANNEL_TOKEN_ASCII   | True ✅ |
| LINE_CHANNEL_TOKEN_HAS_WHITESPACE | False ✅ |
| FIRE_LINE_RECIPIENT_ID_SET | True (prefix='U', length=33) |
| ENV_CHECK                  | OK ✅ |

### Step 3: DATA-R2 gate 再生成 ✗ (warning)

実行: `run_data_freshness_gate --db-path data/fire.staging.db
--db-label staging --output-json /tmp/f062_r5_gate.json --output-text
/tmp/f062_r5_gate.txt`

| field | value |
|---|---|
| overall_status | **warning** ✗ |
| line_send_allowed | **False** ✗ |
| as_of_date | 2026-05-11 |
| db_label | staging |
| allow_warning | False |
| reasons | `[gate-5-other/soft/warning] announcements 鮮度低: max_date=2026-05-01 / lag=6 営業日` |

| gate | level | status | metrics |
|---|---|---|---|
| gate-1-prices    | required    | pass    | max_date=2026-05-08 (lag=1) / distinct_codes=4448 ✅ |
| gate-2-signals   | required    | pass    | max_base_date=2026-05-09 (lag=0) / distinct_codes=109 ✅ |
| gate-3-index     | recommended | pass    | max_date=2026-05-08 (lag=1) ✅ |
| gate-4-derived   | recommended | pass    | max_base_date=2026-05-08 (lag=1) / distinct_codes=42 ✅ |
| gate-5-other     | soft        | **warning** ✗ | announcements lag=6 (上限 5) / financials lag=1 / listings 4449 |

threshold (announcements_max_date_lag_business_days: 5) を 1 営業日
超過。announcements `max_announced_date=2026-05-01` で、これは
**GW (Golden Week) 直前**。GW 期間中は TDnet 適時開示自体が物理的に
ほぼ停止するため、本鮮度低下は **予期される運用期間** とも解釈できる。

### Step 4-7: 未実施

DATA-R2 stop により以下を **実施していない**:

- F062-R5 用 Advisory payload 生成 (= F111-R4 / F062-R1 runner 未呼出)
- 送信前 dry-run (= F062-R3 runner 未呼出)
- 本番 1 chunk Advisory 送信 (= LineBotClient.send_text 未呼出)
- token / recipient leak 確認 (= 送信していないので artifact 不在)

## 安全要件遵守 (本停止時点)

| 項目 | 結果 |
|---|---|
| 送信は 0 通 (= gate stop で send 未到達) | ✅ |
| 自動発注                                        | 0 ✅ |
| 楽天証券操作                                     | 0 ✅ |
| Computer Use                                    | 0 ✅ |
| 注文価格 / 数量 / 執行指示                       | 送信していないため発生しない ✅ |
| token 平文出力なし                              | ✅ |
| recipient_id 平文出力なし                        | ✅ (= 検査で件数 / prefix のみ) |
| DB write 0 / 3 DB 全 mtime unchanged             | ✅ (gate runner は read-only) |
| TODO Excel                                       | 未更新 ✅ |
| --no-verify                                     | 不使用 ✅ |
| scripts/seed_pattern_layer1.py                   | 未接触 ✅ |
| simulation/research_lane/historical_indicators.py | 未接触 ✅ |
| unrelated modified                               | 未 stage / 未 commit ✅ |
| Codex pre-commit (docs commit のため対象外)       | ✅ |

## 解決案 (要 HQ 判断)

### 案 A (推奨、原則的解決): announcements 再 fetch

F101 / F286-DATA-R0 の TDnet announcement 取得 job を回し、最新 6 営業日
分を埋める。GW 期間中の停止だった場合、再取得しても max_announced_date
は 2026-05-01 のままになる可能性が高い (= 物理的に新規 announcement
が無いだけ)。

実行例 (要確認):
```
.venv/bin/python -m scripts.jobs.audit_jquants_freshness  # まず鮮度確認
# announcements 再 fetch jobs を別途実行
```

### 案 B (HQ 承認後のみ): --allow-warning で送信

GW 等の特殊事情で「**announcements 鮮度低は予期される運用条件**」と
HQ が判断した場合、`run_f062_line_production_send_smoke` ではなく
gate runner 側で `--allow-warning` を指定して新 gate JSON を生成し、
それを payload-json として渡す。

ただし本タスクの仕様上「DATA-R2 gate pass必須」「line_send_allowed
が true 必須」と明記されており、warning を pass 扱いにすることは
タスクスコープ拡大となる。Fujiwara が **明示承認** した場合のみ実施。

### 案 C: 本タスクを延期

GW 明けに TDnet announcement が再開されてから F062-R5 を再実施。
`max_announced_date` が直近 5 営業日以内に戻れば自動で gate=pass に。

## 注意

- 本タスクで Advisory 送信は **0 通**。Fujiwara LINE app に新規通知
  は届かない想定。
- gate runner は read-only (= staging.db に書き込みなし)。本タスク
  内で 3 DB すべて mtime unchanged。

## 次タスク

1. ★ F062-R5 停止記録 (本書、HQ 判断要請中)
2. HQ (Fujiwara) が案 A / B / C を選択:
   - 案 A: TDnet announcement 再 fetch → 鮮度回復後 F062-R5 再実行
   - 案 B: GW 例外で `--allow-warning` 承認 → 即 F062-R5 再実行
   - 案 C: GW 明け待ち → 後日 F062-R5 再実行
3. 並走候補 (= F062-R5 と独立に進められる):
   - F286-PNL-R1 Advisory Decision / Actual PnL Tracking 設計
   - F286-DATA-R3 daily refresh cron 化
   - F242 OpenClaw / F022 FIRE Runner / F013 launchd
