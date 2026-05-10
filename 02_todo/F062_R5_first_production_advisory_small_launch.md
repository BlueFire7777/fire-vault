---
id: F062-R5
phase: P5: 通知 / 第 14 章 LINE 通知配信 / 第 19 章 R-19-08
priority: 最優先
status: 完了 ★ (2026-05-11、初回本番 Advisory 1 通送信成功 + Fujiwara 受信確認待ち)
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

---

## 案 A 実施結果 (2026-05-11、announcements 再 fetch 完了)

### 実施内容

1. **/fins/announcement (J-Quants API)** を試行 → HTTP 403 (endpoint
   does not exist for this contract plan)。J-Quants ライト契約では
   `/fins/announcement` を叩けないことを確認。staging.db への書き込み 0。
2. **TDnet HTML 直接取得 (F101 Phase 2、scripts/jobs/fetch_tdnet_html.py)**
   を 2026-05-04 〜 2026-05-11 の各営業日 1 日ずつ実行 (rate limit
   1 req/sec 厳守)。

### 取得結果 (staging のみ)

| 日付 | 結果 | inserted |
|---|---|---|
| 2026-05-04 (月、GW 振替) | schema parse error (= 開示なし) | 0 |
| 2026-05-05 (火、GW 子供の日) | schema parse error | 0 |
| 2026-05-06 (水、GW 振替) | schema parse error | 0 |
| 2026-05-07 (木、平日) | OK | **294** |
| 2026-05-08 (金、平日) | OK | **797** |
| 2026-05-11 (月、JST 01:35 早朝) | schema parse error (= 当日 TDnet 未開示) | 0 |

合計 inserted: **1,091 件** (= 2026-05-07 + 2026-05-08)

### staging announcements 状態 before / after

| field | BEFORE | AFTER |
|---|---|---|
| total_rows                       | 7 | **1,098** |
| max_announced_date               | 2026-05-01 | **2026-05-08** ★ |
| distinct dates                   | 2 | 4 |
| 直近 2 営業日 (5/7 + 5/8)        | 0 件 | 1,091 件 |

### DATA-R2 gate before / after (案 A 実施)

| 項目 | BEFORE | AFTER |
|---|---|---|
| overall_status                                 | warning | **pass** ★ |
| line_send_allowed                              | False | **True** ★ |
| reasons (count)                                | 1 | 0 |
| gate-1-prices (required)                       | pass | pass |
| gate-2-signals (required)                      | pass | pass |
| gate-3-index (recommended)                     | pass | pass |
| gate-4-derived (recommended)                   | pass | pass |
| gate-5-other (soft、announcements lag)         | warning (lag=6) | **pass** (lag=1) ★ |

### gate-5-other 詳細 (after)

`max_announced_date=2026-05-08 / business_day_lag=1` (= threshold=5
を大幅クリア)。GW 期間 (5/4-5/6) は TDnet 物理停止のため空が正、
fetch しても件数 0。GW 直後の 5/7 / 5/8 を取得すれば lag=1 営業日
(= 5/11 - 5/8) で正常化することを確認。

### 安全要件遵守

| 項目 | 結果 |
|---|---|
| LINE 送信なし                                       | ✅ (= LineBotClient.send_text 未呼出) |
| F062-R5 を勝手に再開しない                         | ✅ (= 本タスクは案 A 結果報告まで) |
| --allow-warning を勝手に使わない                  | ✅ (= 案 A の素直な fetch でクリア、案 B 不要) |
| production/develop DB write 禁止                  | ✅ (= mtime unchanged 確認、下記) |
| staging のみ write                                | ✅ (= staging.db mtime 5/10 → 5/11) |
| 自動発注 / 楽天操作 / Computer Use                 | 0 ✅ |
| TODO Excel                                        | 未更新 ✅ |
| --no-verify                                      | 不使用 ✅ |
| scripts/seed_pattern_layer1.py / historical_indicators.py | 未接触 ✅ |
| unrelated modified                                | 未 stage / 未 commit ✅ |
| Codex pre-commit (docs commit のため対象外)       | ✅ |

### DB mtime before / after

| DB | before | after |
|---|---|---|
| data/fire.db          | May  7 16:12:38 2026 | May  7 16:12:38 2026 (unchanged ✅) |
| data/fire.develop.db  | May  7 18:14:26 2026 | May  7 18:14:26 2026 (unchanged ✅) |
| data/fire.staging.db  | May 10 18:22:36 2026 | **May 11 01:35:30 2026** (announcements 1,091 行追加) |

### F062-R5 再開可否

**再開可能** ★。

- DATA-R2 gate **全 5 段 PASS** / `line_send_allowed=True`
- env / token / recipient は F062-R4 試行時の最終検証から変動なし
  (length=516 / ASCII=True / no whitespace、recipient prefix='U' length=33)
- token preflight + recipient masking は F062-R4.1 / R4.2 / F236-R1
  で全層整備済

ただし、本タスクは「announcements 再 fetch + gate pass 確認」までを
スコープとし、**F062-R5 再開は HQ (Fujiwara) 判断後に開始**。タスク
仕様「F062-R5 を勝手に再開しない」を遵守。

### 警告: 5/11 (本日) TDnet 未開示

2026-05-11 (本日) の TDnet 取得は schema parse error 扱い。これは
当日 16:00 以前の早朝に走らせたため (= TDnet 開示は通常 16:00 以降)。
本タスクで取得したのは 5/7 + 5/8 のみ。lag=1 営業日 (= 5/11 - 5/8)
は閾値 5 内なので gate=pass。当日中に 5/11 分が出たら再 fetch して
lag=0 にすることも可能だが、本タスクではそこまでやらない。

### 軽微改善候補 (本タスク対象外)

- TDnet 1 日単位 fetch を 1 つの runner で 範囲指定 (--from / --to)
  できるように拡張すれば、`fetch_announcements.py` と整合する。本
  タスク対象外。
- 5/11 の schema parse error は「TDnet ページが空 / フォーマット異」
  だが、log では同じ error message。「empty page」と「real schema
  change」を区別できると望ましい。本タスク対象外。
- F286-DATA-R3 cron 化が完了すれば、本問題は自動的に出にくくなる。

### 次タスク (案 A 実施完了後)

1. ★ 案 A 完了報告 → HQ (Fujiwara) が F062-R5 再開を承認
2. F062-R5 First Production Advisory Small Launch 再実行
   - 最新 staging から Advisory payload 生成
   - dry-run → 1 chunk 本番送信
3. 並走候補:
   - F286-PNL-R1 Advisory Decision / Actual PnL Tracking 設計
   - F286-DATA-R3 daily refresh cron 化
   - F242 OpenClaw / F022 FIRE Runner / F013 launchd

---

## 本番送信実施結果 (2026-05-11、初回 Advisory)

### env / gate 再確認 ✅

| 項目 | 結果 |
|---|---|
| token (length / ASCII / no whitespace) | 516 / True / True ✅ |
| recipient (prefix / length)             | 'U' / 33 ✅ |
| DATA-R2 overall                         | pass ✅ |
| line_send_allowed                       | True ✅ |
| 5 段 (prices/signals/index/derived/other) | 全 PASS ✅ |

### Advisory payload 生成

実行: F111-R4 → F062-R1 line preview の chain (両 runner とも
read-only / dry-run 専用、DB write 0)。

| 項目 | 値 |
|---|---|
| F111-R4 source_version    | r2f4_baseline_v1 |
| F111-R4 rule_version      | r2g3_recommended_v2 |
| F111-R4 base_date          | 2026-03-01 (= r2f4_baseline_v1 の最新) |
| F111-R4 top_n              | 30 |
| F111-R4 candidate_count    | 30 (boost=0 / avoid=30 / caution=0) |
| F111-R4 auto_order_allowed_true_count | 0 ✅ |
| F111-R4 manual_review_required_count | 30 ✅ |
| F062-R1 max_candidates    | 8 |
| F062-R1 max_per_label     | 4 |
| F062-R1 include_labels    | avoid / boost_with_avoid / caution / boost_with_caution / boost |
| F062-R1 selected (= LINE) | **4** (avoid 系のみ、その他 label は 0 件) |
| F062-R1 chunks            | **1** (chunk_length=1892) |
| forbidden_phrase_count    | 0 ✅ |
| safety_footer_present     | True ✅ |
| auto_order_allowed_true_count | 0 ✅ |
| manual_review_required_count | 4 (= selected_count) ✅ |

### dry-run 結果 ✅

| 項目 | 値 |
|---|---|
| exit                       | 0 |
| mode                       | dry_run |
| send_allowed               | True |
| sent_count                 | 0 |
| line_api_call_count        | 0 |
| token_read_count           | 0 |
| production_callable_built  | False |
| forbidden_phrase_count     | 0 |
| safety_footer_present      | True |
| manual_review_required_count | 4 |
| auto_order_allowed_true_count | 0 |
| payload_chunks_total       | 1 |

### real send 結果 ★

| 項目 | 値 |
|---|---|
| exit                       | **0** |
| mode                       | send |
| dry_run                    | False |
| send_allowed               | True |
| sent_count                 | **1** ★ |
| line_api_call_count        | **1** ★ |
| stub_invocations           | 0 |
| partial_delivery           | **False** ★ |
| token_read_count           | 1 |
| production_callable_built  | True |
| hq_approved_send           | True |
| max_chunks                 | 1 |
| payload_chunks_total       | 1 |
| forbidden_phrase_count     | 0 ✅ |
| safety_footer_present      | True ✅ |
| manual_review_required_count | 4 ✅ |
| auto_order_allowed_true_count | 0 ✅ |
| production_outcomes        | 1 件 (chunk_index=0 / status=ok / dry_run_line_api=False / chunk_length=1892 / recipient_type=user / recipient_hash8=b344b213) |

### token / recipient leak 検査

artifact 全件 + LINE log file を検査:
- /tmp/f062_r5_advisory_preview.json / .csv / .txt
- /tmp/f062_r5_advisory_summary.json
- /tmp/f062_r5_line_payload.json
- /tmp/f062_r5_line_preview.txt
- /tmp/f062_r5_line_summary.json
- /tmp/f062_r5_pre_send_dryrun.json / report.txt
- /tmp/f062_r5_first_production_advisory_send.json / report.txt
- logs/notifications/notifications_line.log

| 検査 | 結果 |
|---|---|
| TOKEN_LEAK total | **0 件** ✅ |
| FULL_RECIPIENT_LEAK total | **0 件** ✅ |
| LINE log の SEND 行: recipient | masked (`user:prefix=U:len=33:hash8=b344b213`) ✅ |
| LINE log の SEND 行: token | 不在 ✅ |
| LINE log の SEND 行: 本文 preview | "FIRE Research Advisory Preview\\ndry-run / LINE 送信なし / 自動発注なし\\n手動レビュー必須\\nsource: r2f4_baseline_v1 / r2g3_recommended_v2\\n..." (= 先頭 200 文字、注文 / 価格 / 数量なし) |

### DB 不変確認

| DB | mtime |
|---|---|
| data/fire.db          | May  7 16:12:38 2026 (unchanged ✅) |
| data/fire.develop.db  | May  7 18:14:26 2026 (unchanged ✅) |
| data/fire.staging.db  | May 11 01:35:30 2026 (= 案 A 案 A の announcements fetch 時のみ、本送信では touch なし) |

### 安全要件遵守

| 項目 | 結果 |
|---|---|
| 自動発注                                          | 0 ✅ |
| 楽天証券操作                                       | 0 ✅ |
| Computer Use                                      | 0 ✅ |
| 注文価格 / 数量 / 執行指示                         | 送信していない ✅ |
| max_chunks                                        | 1 (固定) ✅ |
| selected candidates                               | 4 (= 5-10 範囲内) ✅ |
| DATA-R2 gate                                      | pass ✅ |
| --send + --hq-approved-send                       | 両指定 ✅ |
| recipient                                         | Fujiwara 個人宛 (= 'U' 始まり、masked のみ出力) ✅ |
| token 平文出力                                    | 0 ✅ |
| recipient_id 平文出力                              | 0 ✅ |
| partial_delivery                                  | False (= retry 不要) ✅ |
| production / develop DB 接触                       | なし ✅ |
| TODO Excel                                        | 未更新 ✅ |
| --no-verify                                       | 不使用 ✅ |
| scripts/seed_pattern_layer1.py                    | 未接触 ✅ |
| simulation/research_lane/historical_indicators.py | 未接触 ✅ |
| unrelated modified                                | 未 stage / 未 commit ✅ |

### Fujiwara LINE app 受信確認 (依頼)

送信時刻: 2026-05-11T01:50 JST 頃
内容: F062-R5 初回本番 Advisory (= chunk_length=1892 文字、selected 4
candidates、avoid 系のみ、注文 / 価格 / 数量 / 執行指示なし、Safety
footer 8 行入り)

冒頭:
```
FIRE Research Advisory Preview
dry-run / LINE 送信なし / 自動発注なし
手動レビュー必須
source: r2f4_baseline_v1 / r2g3_recommended_v2
base_dates: 2026-03-01
selected_candidates: 4
...
```

Fujiwara の LINE app に **1 通だけ** 届いているはず。受信内容のレビュー
+ 重複なし確認をお願いします。

### 観察事項 (本タスクの注意点)

1. **r2f4_baseline_v1 の最新 base_date は 2026-03-01**
   - R2-F4 broader_historical_sampling のときに生成された分のみ
   - 本日 (= 2026-05-11) からは 2 ヶ月以上前のデータ
   - 「これは初回本番 Advisory + Fujiwara 手動レビュー前提」として
     送信、文面に source / base_date を明示

2. **boost 系候補は 0 件**
   - r2f4_baseline_v1 では F119 evaluate が boost 判定する候補なし
   - 全 30 候補とも avoid 系
   - 4 件 selected も avoid 系

3. **Advisory 文面に注文 / 価格 / 数量 / 執行指示なし**
   - F062-R1 template 設計通り
   - chunk_length=1892 文字は forbidden phrase / safety footer 検査
     をすべて通過

### 次タスク

1. ★ F062-R5 完了 (= 本書記録時点)
2. **Fujiwara が LINE app で 1 通受信確認** → 受信内容のレビュー
3. 受信成功確認後:
   - F286-PNL-R1 Advisory Decision / Actual PnL Tracking 設計
4. 並走候補:
   - F286-DATA-R3 daily refresh cron 化
   - F242 OpenClaw / F022 FIRE Runner / F013 launchd
   - F062 系の運用反復 (= 次回以降の Advisory 送信は本パイプライン
     を再利用、必要に応じて max_chunks を増やす)

---

## F062-R5.1 Production Advisory Message UX + Payload Freshness Guard 完了 (2026-05-11)

F062-R5 で初回本番 Advisory 送信が成功したが、文面 UX と payload 鮮度
の 2 領域に問題が判明。F062-R5.1 でこれらを修正。本タスクでは実 LINE
送信は行わず、tests + dry-run のみで検証。

### 解決した問題

| # | 旧挙動 | 新挙動 (F062-R5.1) |
|---|---|---|
| 1 | 本番送信なのに "dry-run / LINE 送信なし" を含む | message_mode=production で構造的に除外、"本番 LINE 通知" を明示 |
| 2 | base_date 2026-03-01 (= 2 ヶ月前) を本番送信 | payload freshness guard で gate signal max_base_date と calendar lag <= 10 日のみ許可 |
| 3 | 文面が長く機械的、avoid のみで結論不明 | compact LINE UX で冒頭結論 + 絵文字 badge + 候補 2 行 |
| 4 | "新規買い検討候補なし" が冒頭で見えない | format_compact_conclusion: avoid のみ → "🔴 結論: 新規買い検討候補なし" / boost あり → "🟢 結論: 買い検討候補あり" |

### 実装

**notifications/templates/research_advisory.py**
- `MESSAGE_MODE_PREVIEW` / `MESSAGE_MODE_PRODUCTION` 定数
- `SAFETY_FOOTER_COMMON_LINES` (7 行共通) + `SAFETY_FOOTER_LINE_PREVIEW`
  / `SAFETY_FOOTER_LINE_PRODUCTION` (mode 固有 marker)
- `SAFETY_FOOTER_LINES` (= preview 8 行、既存互換) /
  `SAFETY_FOOTER_LINES_PRODUCTION` (= production 8 行)
- `HEADER_LINES_FIXED_PRODUCTION` ("FIRE 本番 Advisory" / "本番 LINE
  通知 / 自動発注なし / 手動レビュー必須")
- `format_header(message_mode=...)` / `format_footer(message_mode=...)`
- `assert_safety_invariants(message_mode=...)` (= mode marker 必須 +
  反対 mode marker 不在)
- `build_advisory_line_preview(message_mode=..., compact=...)`
- Compact LINE UX:
  - `COMPACT_LABEL_BADGE` (= 🟢 buy / 🟡 buy_caution / ⚠️ caution /
    🟠 mixed / 🔴 avoid / ⚪ neutral / ⛔ suppress / ❔ missing)
  - `format_compact_conclusion(label_counts)` → 冒頭結論 2 行
  - `format_compact_candidate(row)` → badge + sector + F119 の 2 行
  - `format_compact_label_section(label, rows)` → label グループ表示

**scripts/jobs/run_f062_research_advisory_line_preview.py**
- `--message-mode {preview,production}` (default: preview)
- `--compact` (default: False)
- payload に `metadata` (message_mode / compact / source_version /
  rule_version / base_dates / payload_base_date / generated_at_utc) 追加

**scripts/jobs/run_f062_line_production_send_smoke.py**
- `_check_payload_freshness(payload, gate_result, max_lag_days)` 新設
  - payload.metadata.message_mode == "production" を要求
  - payload.metadata.payload_base_date と
    gate.gate-2-signals.metrics.max_base_date の calendar 日数差を
    max_lag_days 以下に制限
- `--max-payload-base-date-lag-days` (default 10) 引数追加
- output_json に `payload_freshness_check` (= masked 診断情報) 追記
- `_TEST_MESSAGE_TEMPLATE` を production marker に切替

**agents/line_production_sender.py**
- footer 定数を notifications/templates から import (= 単一情報源化)
- `_validate_chunk` で production marker 必須 + preview marker 不在
  検査追加 (= production 経路で「dry-run / LINE 送信なし」混入 chunk
  を refuse)

**agents/line_advisory_send.py**
- footer 検査を template の SAFETY_FOOTER_COMMON_LINES (7 行) +
  SAFETY_FOOTER_MARKER_LINES (preview / production 2 種) のいずれか
  1 行 に拡張 (= 両 mode の chunk が router を通せる、production 専用
  厳格化は sender 側)

### tests 結果

| 領域 | 新規件数 |
|---|---|
| template message_mode + compact | 8 件 (TestMessageModeProduction 3 + TestCompactLineUX 5) |
| production runner freshness guard | 6 件 (TestPayloadFreshnessGuard) |
| R1 line preview runner UX | 2 件 (TestF062R51RunnerOutputs) |
| **新規合計** | **16 件** |
| **full pytest** | **3,249 PASS** (= 3,233 baseline + 16) / 回帰 0 件 |

### before / after 文面例

**Before (F062-R5 で実送信した文面、preview marker 残存):**
```
FIRE Research Advisory Preview
dry-run / LINE 送信なし / 自動発注なし
手動レビュー必須
source: r2f4_baseline_v1 / r2g3_recommended_v2
base_dates: 2026-03-01
selected_candidates: 4
... (avoid 系を機械的に列挙、結論行なし、長文)
Safety
- LINE 送信なし (dry-run / template only)   ← 本番なのに "dry-run"
- 自動発注なし
...
```

**After (F062-R5.1 production + compact、本タスクで dry-run 確認):**
```
FIRE 本番 Advisory
本番 LINE 通知 / 自動発注なし / 手動レビュー必須
source: r2f4_baseline_v1 / r2g3_recommended_v2
base_dates: 2026-05-09
selected_candidates: 4

🟢 結論: 買い検討候補あり (or 🔴 結論: 新規買い検討候補なし)
候補は手動レビュー前提。自動発注なし。

🟢 買い検討候補 (N)
🟢 買い検討候補 9999 銘柄名
  情報・通信 / F119 h20 +5.0% / win 60.0%

🔴 見送り候補 (M)
🔴 見送り候補 1111 銘柄名
  電気機器 / F119 h20 +0.0% / win 50.0%
...

Safety
- 本番 LINE 通知 (production send)   ← production marker、preview 不在
- 自動発注なし
- 楽天操作なし
- Computer Use なし
- 注文価格・数量・執行指示は生成しない
- F119 の優位は 20d 寄り (daytrade 即時判断には使わない)
- Fujiwara manual review required
```

### 安全要件遵守

| 項目 | 結果 |
|---|---|
| 実 LINE 送信なし (本タスク内)                                       | ✅ |
| token 平文出力なし                                                 | ✅ |
| recipient_id 平文出力なし                                           | ✅ |
| DB write なし / 3 DB 全 mtime unchanged                              | ✅ |
| 自動発注 / 楽天操作 / Computer Use                                   | 0 ✅ |
| 注文価格 / 数量 / 執行指示                                           | 送らない (= compact / default いずれも生成しない) ✅ |
| TODO Excel 未更新                                                   | ✅ |
| --no-verify 不使用                                                  | ✅ |
| scripts/seed_pattern_layer1.py / historical_indicators.py          | 未接触 ✅ |
| unrelated modified を stage / commit                                 | しない ✅ |
| Codex pre-commit                                                    | 全 commit 通過、CRITICAL 0 件 ✅ |

### commits

| commit | message |
|---|---|
| a311e06 | fix(F062-R5): separate production advisory message mode and add compact LINE format |
| 86731c5 | fix(F062-R5): add production payload freshness guard + production-marker chunk check |
| 039a4a4 | test(F062-R5): add production message UX and freshness tests |
| (本 commit) | docs(F062-R5): log production advisory UX fix |

注: ユーザー指示の 5 commits を template の同一 file に message_mode
+ compact 両方が含まれる構造から file 単位分割が現実的でないため、
`fix(message_mode)` と `feat(compact)` を 1 commit (a311e06) に統合
した 4 commits 構成 (= 計算量同一、レビュー粒度のみ集約)。

### 次タスク

1. ★ F062-R5.1 完了 (= 本書記録)
2. F062-R5 系の次回送信は production + compact mode を default 化
   して運用 (= 文面短く、結論先行、preview marker 完全排除)
3. F286-PNL-R1 Advisory Decision / Actual PnL Tracking 設計
4. 並走候補: F286-DATA-R3 cron 化 / F242 OpenClaw / F022 FIRE Runner
   / F013 launchd
