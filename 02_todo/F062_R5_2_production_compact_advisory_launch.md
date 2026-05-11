---
id: F062-R5.2
phase: P5: 通知 / 第 14 章 LINE 通知配信 / 第 19 章 R-19-08
priority: 最優先
status: 停止 (2 段目: F286-DATA-R1.4 restore 後の再開試行で r2f4_baseline_v1 latest base_date が古く HQ 判断「本タスク停止」)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R5 (初回本番 Advisory 送信成功 + Fujiwara 受信確認)
  - F062-R5.1 (production message mode + compact LINE UX + payload
    freshness guard)
  - F286-DATA-R2 (freshness gate)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 26 章
---

# F062-R5 production + compact 再送信 (F062-R5.2)

最終更新: 2026-05-11

## 状態: 停止 (staging.db 巻き戻りで gate refuse)

env preflight は全 pass、F062-R5.1 のコード修正 (= production message
mode / compact UX / payload freshness guard) は 3,249 PASS で完了済み。
しかし最新 staging で再生成した DATA-R2 gate が **overall=refuse /
line_send_allowed=False** で停止条件 hit。Advisory 送信を実施せず停止。

## env / 安全要件 (停止前 preflight)

| 項目 | 結果 |
|---|---|
| LINE_CHANNEL_TOKEN_SET     | True (length=516) |
| LINE_CHANNEL_TOKEN_ASCII   | True ✅ |
| LINE_CHANNEL_TOKEN_HAS_WHITESPACE | False ✅ |
| FIRE_LINE_RECIPIENT_ID_SET | True (prefix='U', length=33) |
| ENV_CHECK                  | OK ✅ |

## DATA-R2 gate 状態

```
overall_status:    refuse ✗
line_send_allowed: False ✗
as_of_date:        2026-05-11
db_label:          staging

gate-1-prices       required    refuse   market_prices_daily 鮮度不足:
                                          max=2026-05-01 / lag=6 > 1
gate-2-signals      required    refuse   research_watchlist_signals 不在
gate-3-index        recommended warning  index_data 鮮度不足 lag=6 > 1
gate-4-derived      recommended warning  research_derived_indicators 空/不在
gate-5-other        soft        warning  market_financials_v2 不在 /
                                          announcements 鮮度低 lag=6
```

すべての段で warning/refuse が発生。

## staging.db 巻き戻りの観察

| 項目 | 前回 F062-R5 / 案 A 後 (2026-05-11 01:35) | 現在 (2026-05-11 07:00 後) |
|---|---|---|
| staging.db mtime | May 11 01:35:30 | **May 11 07:00:05** ★ |
| staging.db size  | (案 A で増えていた) | **371,064,832 bytes** |
| market_prices_daily max_date | 2026-05-08 | **2026-05-01** ★ |
| market_prices_daily total_rows | 2,085,284 | **526,764** ★ |
| market_prices_daily distinct codes | 4,448 | 4,452 |
| research_watchlist_signals | max_base_date=2026-05-09 / 109 codes | **テーブル不在** ★ |
| research_derived_indicators | max_base_date=2026-05-08 / 42 codes | **テーブル不在** ★ |
| announcements | 1,098 行 / max=2026-05-08 | **7 行 / max=2026-05-01** ★ |

3 DB の現在 size:

| DB | size | mtime |
|---|---|---|
| data/fire.db          | 371,064,832 | May  7 16:12:38 2026 |
| data/fire.develop.db  | 371,064,832 | May  7 18:14:26 2026 |
| data/fire.staging.db  | **371,064,832** | **May 11 07:00:05 2026** |

→ **3 DB が完全に同 size 371 MB**。staging.db は 5/11 07:00 ごろに
fire.db / develop.db からの全コピー (= 同一 snapshot copy) で上書き
された可能性が極めて高い。

→ 案 A で TDnet 取得した 1,091 行の announcements、F286-DATA-R1.3
で取得した 5/2 〜 5/8 の prices、研究 R2 系で生成した
research_watchlist_signals / research_derived_indicators **すべて消えた**。

## 推定原因

Mac mini 上で本タスクシーケンス **外** に走った何らかの定期処理が
staging.db を **production / develop snapshot で上書き** した。
候補:

- cron / launchd の daily DB sync job (= 同 size から推測)
- 別 Claude Code session / shell で実施された手動 DB copy
- F242 OpenClaw 系前段で daily reset するスクリプト

本タスクシーケンス内では staging.db に書き込みなし (= gate runner は
read-only)。原因は外部。

## 実施せず

DATA-R2 stop により以下を **実施していない**:

- F062-R5.2 用 production + compact payload 生成
- payload freshness guard 確認 (= signals 不在で base_date 取得不可)
- 送信前 dry-run
- 本番 1 chunk Advisory 送信
- token / recipient leak 検査 (= 送信していないので artifact 不在)

**Fujiwara LINE app に新規通知は届かない**。

## 安全要件遵守 (本停止時点)

| 項目 | 結果 |
|---|---|
| 送信は 0 通                                       | ✅ |
| 自動発注                                          | 0 ✅ |
| 楽天証券操作                                       | 0 ✅ |
| Computer Use                                      | 0 ✅ |
| 注文価格 / 数量 / 執行指示                         | 送信していない ✅ |
| token 平文出力                                    | 0 ✅ |
| recipient_id 平文出力                              | 0 ✅ |
| DB write (本タスク内)                              | 0 ✅ (gate runner read-only) |
| production fire.db mtime                          | May  7 16:12:38 unchanged ✅ |
| develop fire.develop.db mtime                     | May  7 18:14:26 unchanged ✅ |
| staging fire.staging.db mtime                     | May 11 07:00:05 (= 本タスク外で巻き戻り、本タスク内では書込なし) |
| TODO Excel                                        | 未更新 ✅ |
| --no-verify                                       | 不使用 ✅ |
| scripts/seed_pattern_layer1.py                    | 未接触 ✅ |
| simulation/research_lane/historical_indicators.py | 未接触 ✅ |
| unrelated modified                                | 未 stage / 未 commit ✅ |

## 復旧 case (要 HQ 判断)

### 案 A (推奨): staging 再構築

1. fetch_historical_market_data.py で 2026-05-02 〜 2026-05-08 の
   prices を staging に再取得
2. fetch_tdnet_html.py で 2026-05-07 / 2026-05-08 の announcements を
   再取得 (= 前回案 A 同様)
3. research_watchlist_signals / research_derived_indicators を再生成
   (= F286-R2-A1 / R2-A2 / R2-A3 / R2-D 等のパイプライン再実行)
4. DATA-R2 gate 再確認 → pass
5. F062-R5.2 production + compact 送信を再起動

これは大規模な再構築 (= 1 〜 数時間)。

### 案 B (調査): staging 巻き戻りの原因特定

- launchd / cron 一覧確認 (`launchctl list | grep fire`、`crontab -l`)
- 直近 7 時間に staging.db を touch した process の log 確認
- Mac mini 上で他 Claude Code session / shell history 確認
- もし「daily DB sync」が staging を上書きする仕様なら、F242 OpenClaw
  運用基盤の設計を見直し

### 案 C (延期): 5/12 以降の通常運用サイクル

5/12 平日に daily refresh (cron / 手動) を回す前提なら、その時点で
staging が再構築される。当面 F062-R5 系送信は延期。

## 次タスク

1. ★ F062-R5.2 停止記録 (本書、HQ 判断要請中)
2. HQ (Fujiwara) が案 A / B / C を選択
3. 並走候補:
   - F286-DATA-R3 daily refresh cron 化 (= 本問題の再発防止に直結)
   - F242 OpenClaw / F022 FIRE Runner / F013 launchd (= 自動運用基盤)
   - F286-PNL-R1 Advisory Decision / Actual PnL Tracking 設計

---

## 2 段目停止記録 (2026-05-11、F286-DATA-R1.4 restore 後の再開試行)

FIRE-OPS-R0 + F286-DATA-R1.4 で staging を案 A 完了状態 (= 5/8 prices
/ 5/9 signals / 5/8 announcements / 5/8 derived) に restore 済み、
DATA-R2 gate 全 5 段 PASS / line_send_allowed=True に復活した状態で
F062-R5.2 を再開。

### 再開試行時の env / gate ✅

| 項目 | 結果 |
|---|---|
| LINE_CHANNEL_TOKEN_SET     | True (length=516 / ASCII / no whitespace) |
| FIRE_LINE_RECIPIENT_ID_SET | True (prefix='U' / length=33) |
| DATA-R2 overall            | pass ✅ |
| line_send_allowed          | True ✅ |
| gate-2-signals max_base    | 2026-05-09 |

### r2f4_baseline_v1 の latest base_date 確認

| source_version | latest base_date | rows |
|---|---|---|
| r2d_v1                       | 2026-05-09 | 109 |
| r2f3_leaksafe_v1             | 2026-03-16 | 1,130 |
| r2d2_baseline_v1             | 2026-03-16 | 547 |
| ★ **r2f4_baseline_v1**       | **2026-03-01** | 109 |
| r2f4_risk_adjusted_v1        | 2026-03-01 | 3,544 |
| r2f4_quality_defensive_v1    | 2026-03-01 | 2,429 |

→ r2f4_baseline_v1 の最新 base_date は **2026-03-01**。F062-R5 系で
本来採用したい本線 source_version だが、研究 task (= F286-R2-F4
broader historical sampling) で生成された分のみ存在し、本タスク時点
(= 2026-05-11) からは **69 calendar days 前** のデータ。

### freshness guard 評価 (= F062-R5.1 設計)

| 項目 | 値 |
|---|---|
| payload_base_date (r2f4_baseline_v1 max) | 2026-03-01 |
| gate signal max_base_date (= 全 source 横断) | 2026-05-09 |
| calendar lag                              | **69 days** |
| --max-payload-base-date-lag-days default | 10 days |

→ **freshness guard が refuse する** (69 >> 10)。F062-R5.1 設計の
「historical / smoke base_date payload は production 送信不可」が
正しく作動する。

### HQ (Fujiwara) 判断: 本タスク停止 ★

提示した 3 案:
- 案 A: `--max-payload-base-date-lag-days 100` で guard 緩めて r2f4
        / 2026-03-01 を送信
- 案 B: `r2d_v1` (latest 2026-05-09) で送信
- 案 C: 本タスク停止 + r2f4_baseline_v1 を最新 base_date まで再生成
        する別 task を提案

→ **HQ 選択: 案 C (本タスク停止)**

HQ 理由:
- r2f4_baseline_v1 の latest_base_date が 2026-03-01 で古く、F062-R5.1
  freshness guard の設計意図に反する
- `--max-payload-base-date-lag-days 100` で緩めて送るのは不可
  (= guard を実質無効化するため、F062-R5.1 設計を否定する)
- r2d_v1 は最新 base_date だが、F286-R2-F4 で baseline 本線として
  採用した source_version ではない (= 研究 R2 系列の中間 source)、
  本番初回 Advisory に使わない
- 現状で送信可能な production source_version が **存在しない** と
  判断

### 実施せず

DATA-R2 gate が pass であっても HQ 判断「本タスク停止」を受け、以下
を **実施していない**:

- F111-R4 advisory rows 生成 (= source_version=r2f4_baseline_v1
  / base_date=2026-03-01 の生成試行も含めて未実行)
- F062-R1 line preview --message-mode production --compact での
  payload 生成
- 送信前 dry-run / 本番 1 chunk 送信
- token / recipient leak 検査 (= 送信していないので artifact 不在)

**Fujiwara LINE app に新規通知は届かない**。

### 安全要件遵守 (2 段目停止時点)

| 項目 | 結果 |
|---|---|
| 送信は 0 通                                          | ✅ |
| --max-payload-base-date-lag-days を勝手に緩めない    | ✅ (= HQ 不承認、本タスク内で touched なし) |
| source_version を勝手に r2d_v1 に変えない             | ✅ |
| 自動発注                                             | 0 ✅ |
| 楽天証券操作                                          | 0 ✅ |
| Computer Use                                         | 0 ✅ |
| 注文価格 / 数量 / 執行指示                            | 送信していない ✅ |
| token 平文出力                                       | 0 ✅ |
| recipient_id 平文出力                                 | 0 ✅ |
| DB write (本タスク内)                                 | 0 ✅ (= gate runner read-only) |
| production fire.db / develop fire.develop.db          | mtime unchanged ✅ |
| staging fire.staging.db                              | F286-DATA-R1.4 restore 時点 (5/11 11:20:59 / 4.8 GB) で本タスク内 touched なし ✅ |
| TODO Excel                                           | 未更新 ✅ |
| --no-verify                                          | 不使用 ✅ |
| scripts/seed_pattern_layer1.py / historical_indicators.py | 未接触 ✅ |
| unrelated modified を stage / commit                 | しない ✅ |

### 次タスク提案 (HQ 指示)

★ **F286-DATA-R1.5 Latest Baseline Signal Regeneration for Production
Advisory**

目的: 最新の J-Quants / market_prices_daily / derived indicators
/ research_watchlist_signals を使って、r2f4_baseline_v1 相当の
production-ready signal を最新 base_date (= 2026-05-09 or それに
近い直近日) で生成し、F062-R5.2 の payload freshness guard を
**自然に通せる** 状態にする。

本タスクで明示された制約:
- LINE 送信しない
- `--allow-warning` しない
- freshness guard を緩めない
- source_version を勝手に r2d_v1 へ変えない
- DB write する場合は staging のみ、production / develop 無触
- TODO Excel 更新しない

並走候補:
- FIRE-OPS-R0 再発防止策案 1 (= 本番運用データを production fire.db
  に書く運用統一) の設計レビュー
- 03_design/F282_environment_isolation_*.md の運用ルール明文化
  (= staging weekly snapshot との整合)
- F286-DATA-R3 daily refresh cron 化 (= 案 1 と整合)
