---
id: F286-DATA-R1.5
phase: P5: ResearchLane R1 / 本番運用準備
priority: 最優先 (= F062-R5.2 freshness guard を自然に通すための前提)
status: 完了 ★ (2026-05-11、staging 限定 baseline live signal 生成 + dry-run 検証 PASS)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-DATA-R1.4 (= staging restore で gate pass 復活)
  - F062-R5.1 (= payload freshness guard 設計)
  - F286-R2-D (= watchlist ranker、DEFAULT_WEIGHTS QV/EG/CV)
  - F286-R2-E (= signal persistence runner、--db staging --write)
chapter: 第 10 章 (F282 環境分離) / 第 26 章
---

# F286-DATA-R1.5: Latest Baseline Signal Regeneration for Production Advisory

最終更新: 2026-05-11

## ★ 状態: 完了

r2f4_baseline_v1 相当の baseline ロジック (QV 0.35 / EG 0.35 / CV 0.30)
を最新 base_date=**2026-05-09** で再生成し、source_version=
**r2f4_baseline_live_v1** として staging のみに保存。DATA-R2 gate 全
5 段 PASS / line_send_allowed=True を維持、F062-R5.2 production
compact payload を作って **payload freshness guard が自然に pass**
(lag_calendar_days=0) することを simulated send (--dry-run-line-api)
で確認。実 LINE 送信は行わず。

## 1. 使用 runner / logic

| Step | runner | mode | 役割 |
|---|---|---|---|
| signal 生成 | `scripts/jobs/run_research_watchlist_signal_persistence.py` | `--db staging --write` (FIRE_ENV=staging) | F286-R2-E persistence runner。DEFAULT_WEIGHTS (QV 0.35 / EG 0.35 / CV 0.30) を `simulation/research_lane/watchlist_ranker` から取り込み、staging に書込 |
| gate 検証 | `scripts/jobs/run_data_freshness_gate.py` | read-only | DATA-R2 5 段 PASS / line_send_allowed=True |
| advisory rows | `scripts/jobs/run_f111_research_advisory_real_wiring_r4_smoke.py` | read-only / dry-run | source/rule/base_date 指定で advisory rows 生成 (= F119 evaluate は WIRING ANOMALY のため `--evaluate-f119` 無しで実行、neutral only) |
| LINE payload | `scripts/jobs/run_f062_research_advisory_line_preview.py` | dry-run / `--message-mode production --compact` | F062-R5.1 仕様 |
| freshness 検証 | `scripts/jobs/run_f062_line_production_send_smoke.py` | dry-run + simulated send (--dry-run-line-api) | freshness guard pass 確認 |

新規コード変更: **なし**。既存 runner の組み合わせで完結。

## 2. source_version / base_date

| 項目 | 値 |
|---|---|
| source_version       | **r2f4_baseline_live_v1** ★ |
| rule_version         | r2g3_recommended_v2 |
| base_date            | **2026-05-09** ★ |
| top_n                | 100 |
| 採用 weights         | QV 0.35 / EG 0.35 / CV 0.30 (= ranker DEFAULT_WEIGHTS) |
| sector cap           | 30% (= DEFAULT_SECTOR_CAP_RATIO) |

理由:
- r2f4_baseline_v1 (latest 2026-03-01) は研究 task (F286-R2-F4) で
  生成された古い分のみ → F062-R5.1 freshness guard refuse
- r2d_v1 (latest 2026-05-09) は baseline 本線でない研究 R2 中間 source
  → F062-R5.2 本番初回 Advisory には不採用
- 新規 source_version `r2f4_baseline_live_v1` で **baseline logic そのまま** /
  **最新 base_date** に統一

## 3. generated signal count

```
inserted: 109 / replaced: 0 / skipped: 0 / failed: 0 / total: 109
```

- top_n=100 指定だが実際は 109 行 (= ranker 内部の sector cap + 重複
  処理での実選出 109)
- 全 109 行に final_rank_label=A1 (= 上位ランク、本タスク時点での
  weighted score 順)

## 4. research_watchlist_signals before / after (staging)

| field | BEFORE | AFTER |
|---|---|---|
| total_rows                                                    | 13,551 | **13,660** (+109) |
| distinct source_version                                       | 12 | **13** (+1) |
| r2f4_baseline_live_v1 / 2026-05-09 rows                       | 0 | **109** ★ |
| 他 source_version の rows                                     | (変化なし) | (変化なし) |

新規 source は他の研究 source と並列で共存 (= 既存データ無触)。

## 5. DATA-R2 gate result (post-write)

```
overall_status:    pass ✅
line_send_allowed: True ✅
as_of_date:        2026-05-11
db_label:          staging
reasons:           []

gate-1-prices    required    pass (max=2026-05-08 lag=1 / codes=4448)
gate-2-signals   required    pass (max_base=2026-05-09 lag=0 / codes=110) ← 109→110 codes
gate-3-index     recommended pass (max=2026-05-08 lag=1)
gate-4-derived   recommended pass (max_base=2026-05-08 lag=1 / codes=42)
gate-5-other     soft        pass (financials / announcements / listings OK)
```

distinct_codes が 109 → **110** に増加 (= r2f4_baseline_live_v1 で
1 件追加 unique code) は ranker 出力の差。gate-2 PASS 維持。

## 6. payload freshness guard result

simulated send (`--send --hq-approved-send --dry-run-line-api
--max-chunks 1`) で freshness guard を通した:

| field | value |
|---|---|
| max_lag_days                                | 10 |
| payload_message_mode                        | production ✅ |
| payload_base_date                           | 2026-05-09 |
| gate_signal_max_base_date                   | 2026-05-09 |
| **lag_calendar_days**                       | **0** ★ |

→ F062-R5.2 freshness guard が **自然に pass** (= guard を緩めることなく)。

## 7. F062 production compact payload dry-run 結果

`--send` 無し dry-run + simulated send (--dry-run-line-api) の両方で
確認:

### dry-run (--send 無し) ✅
| field | value |
|---|---|
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
| payload_freshness_check    | None (= dry-run path で実行されない) |

### simulated send (--dry-run-line-api、実 API 呼ばず) ✅
| field | value |
|---|---|
| mode                       | send |
| send_allowed               | True ★ |
| sent_count                 | 1 (= LineBotClient(dry_run=True) で _log 1 行追加) |
| line_api_call_count        | 1 |
| partial_delivery           | False |
| token_read_count           | 1 |
| production_callable_built  | True |
| hq_approved_send           | True |
| dry_run_line_api           | **True** (= 実 push_message 未呼出) |
| max_chunks                 | 1 |
| production_outcomes        | 1 件 (chunk_index=0 / status=dry_run / chunk_length=679 / recipient_type=user / recipient_hash8=b344b213) |
| payload_freshness_check    | max_lag_days=10 / lag_calendar_days=**0** ✅ |

### payload chunk[0] 文面 (= 抜粋、preview)

```
FIRE 本番 Advisory
本番 LINE 通知 / 自動発注なし / 手動レビュー必須
source: r2f4_baseline_live_v1 / r2g3_recommended_v2
base_dates: 2026-05-09
selected_candidates: 4

⚪ 結論: 該当候補なし
neutral / 未付与のみ。手動レビュー必須。

Summary
candidates: 30
boost: 0 / avoid: 0 / caution: 0
non-neutral: 0
auto_order_allowed_true: 0
manual_review_required: 30/30

⚪ 参考 (4)
⚪ 参考 87470  金融（除く銀行） / F119 h20 n/a / win n/a
⚪ 参考 57290  鉄鋼・非鉄 / F119 h20 n/a / win n/a
...

Safety
- 本番 LINE 通知 (production send)
- 自動発注なし
- 楽天操作なし
- Computer Use なし
- 注文価格・数量・執行指示は生成しない
- F119 の優位は 20d 寄り (daytrade 即時判断には使わない)
- Fujiwara manual review required
```

- 「dry-run」「LINE 送信なし」非含 ✅
- 「本番 LINE 通知」「自動発注なし」「手動レビュー必須」含む ✅
- 結論行 "⚪ 結論: 該当候補なし" (= neutral only のため正)
- chunk_length 679 (compact、短い)

## 8. LINE 送信なし

- `--dry-run-line-api` を必ず付与し、LineBotClient(dry_run=True) で
  構築 → push_message **未呼出** ✅
- 実 LINE API へ HTTP リクエスト送信 0
- LineBotClient._log には DRY 行が 1 行追記された (= F236-R1 masked
  形式で recipient_type / hash8 のみ、token / full recipient 不在)

本タスクで実 LINE API を呼んでいない (= production send 完了は
F062-R5.2 再起動時に HQ 判断後)。

## 9. production / develop 無触

| DB | mtime | size | 変化 |
|---|---|---|---|
| data/fire.db          | May  7 16:12:38 2026 | 371,064,832 | unchanged ✅ |
| data/fire.develop.db  | May  7 18:14:26 2026 | 371,064,832 | unchanged ✅ |
| data/fire.staging.db  | May 11 11:48:57 2026 | 4,803,936,256 | **+106 KB** (signal persistence write) |

production / develop に対する DB write / cp / stat write 等 **一切なし**。

## 10. staging write 対象

- `research_watchlist_signals` table のみ
- 109 行 INSERT (= source_version='r2f4_baseline_live_v1' /
  base_date='2026-05-09')
- 他 source_version の既存行は UPDATE / DELETE 一切なし (= replaced=0)
- F282 weekly snapshot で次回月曜 (2026-05-18 07:00 JST) に消える運命

## 11. tests 結果

本タスクはコード変更 0 件 (= 既存 runner 実行のみ)。pytest 追加なし。
既存 tests は変更なしで通過する想定 (= 別途検証不要)。

## 12. Codex pre-commit 結果

本タスクは docs commit のみ → Codex review 対象外 (= scripts/hooks
/pre-commit の判定で *.md / docs / vault repo は code review skip)。
fire コード変更なし。

## 13. 安全要件遵守

| 項目 | 結果 |
|---|---|
| LINE 送信なし                                       | ✅ (= --dry-run-line-api で実 API 呼ばず) |
| freshness guard を緩めない                          | ✅ (= --max-payload-base-date-lag-days 触らず、default 10 で natural pass) |
| r2d_v1 へ勝手に切り替えない                          | ✅ (= 新規 source_version r2f4_baseline_live_v1 を作成) |
| production / develop DB write                       | 0 ✅ (= mtime/size/inode 全 unchanged) |
| staging のみ write                                   | ✅ (= research_watchlist_signals に 109 行 INSERT のみ) |
| 自動発注 / 楽天操作 / Computer Use                   | 0 ✅ |
| 注文価格 / 数量 / 執行指示                           | 送信していない (= compact / default 共生成しない構造) ✅ |
| TODO Excel                                          | 未更新 ✅ |
| --no-verify                                         | 不使用 ✅ |
| scripts/seed_pattern_layer1.py / historical_indicators.py | 未接触 ✅ |
| unrelated modified を stage / commit                 | しない ✅ |
| token / recipient 平文出力                           | 0 ✅ |

## 14. 観察事項

### F119 WIRING ANOMALY

`--evaluate-f119` 付きで F111-R4 を実行すると以下の警告:

```
[F111-R4] WIRING ANOMALY: f119_artifact_status=inline だが
non_neutral_count == 0。接続失敗の可能性。caller は原因
(cut_summary group_key 不一致 等) を確認すること。
```

これは F119 evaluate の cut_summary が r2f4_baseline_live_v1 用に
蓄積されていないため、新規 source_version では cut_summary の
group_key と一致せず全 candidate が neutral 扱いになる現象。

→ 本タスクでは `--evaluate-f119` を **外して** F111-R4 を実行
   (= raw signals advisory、全 neutral)。F062-R5.2 production payload
   は freshness guard 自体は通るが、Advisory 文面は「⚪ 結論: 該当
   候補なし」表示になる (= F119 boost/avoid が出ない)。

### F062-R5.2 を natural に通せる状態には到達

タスクの主目的 (= F062-R5.2 freshness guard を自然に通す) は達成。
ただし、F119 結合できないため Advisory 文面は neutral only。
Fujiwara が「neutral のみの本番 Advisory を送るか / F119 cut_summary
を r2f4_baseline_live_v1 用に蓄積する別 task を先に回すか」を判断。

## 15. 次に F062-R5.2 再開してよいか

**★ freshness guard 観点では再開可能**。

ただし以下を HQ (Fujiwara) が判断:

- 案 X1 (即時): r2f4_baseline_live_v1 / 2026-05-09 (= F119 neutral only)
  で F062-R5.2 を再起動。文面は "⚪ 結論: 該当候補なし" になる。
- 案 X2 (F119 結合): 別 task で F286-R2-H orthogonal_cuts 等の F119
  cut_summary を r2f4_baseline_live_v1 用に蓄積 → F119 boost/avoid
  判定を出せる状態にしてから F062-R5.2 を再起動。
- 案 X3 (延期): F282 weekly snapshot で次回月曜 (5/18 07:00 JST)
  に staging リセット予定なので、再発防止策案 1 (= production DB
  に write する運用統一) を先に設計してから本格運用へ。

## 16. 退避ファイル / 一時 artifact

- 退避: 既存 data/fire.staging.db.pre_restore_20260511_112053 維持
- 新規 artifact (= /tmp 配下、token/full recipient 不在検証済):
  - f286_data_r1_5_gate.json / .txt
  - f286_data_r1_5_signal_persistence.json / .csv
  - f286_data_r1_5_advisory_preview.json / .csv
  - f286_data_r1_5_advisory_summary.json
  - f286_data_r1_5_line_payload.json
  - f286_data_r1_5_line_preview.txt
  - f286_data_r1_5_line_summary.json
  - f286_data_r1_5_dry_run.json / report.txt
  - f286_data_r1_5_send_simulated.json

## 17. 次タスク

1. ★ F286-DATA-R1.5 完了 (= 本書、staging baseline live signal 生成
   + freshness guard pass 確認)
2. HQ (Fujiwara) が F062-R5.2 再起動方針を判断 (案 X1/X2/X3)
3. 並走候補:
   - FIRE-OPS-R0 再発防止策案 1 (= 本番運用データを production に書く
     運用統一) の設計レビュー
   - F286-R2-H orthogonal_cuts を r2f4_baseline_live_v1 用に再実行
     (= F119 結合復旧)
   - 03_design/F282_environment_isolation_*.md の運用ルール明文化
   - F286-DATA-R3 daily refresh cron 化
