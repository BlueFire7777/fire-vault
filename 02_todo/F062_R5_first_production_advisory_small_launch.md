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
