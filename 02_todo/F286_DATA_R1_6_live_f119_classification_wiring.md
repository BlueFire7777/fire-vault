---
id: F286-DATA-R1.6
phase: P5: ResearchLane R1 / 本番運用準備
priority: 最優先 (= F062-R5.2 advisory 文面に F119 boost/avoid/caution を出すため)
status: 完了 ★ (2026-05-11、live F119 wiring + production payload で「🟢 結論: 買い検討候補あり」表示)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-DATA-R1.5 (= r2f4_baseline_live_v1 / 2026-05-09 signal 生成)
  - F119 historical evaluation (= /tmp/f119_eval_*.{json,csv})
  - F062-R5.1 (= production message mode + compact + freshness guard)
chapter: 第 13 章 (F119) / 第 14 章 (LINE) / 第 26 章
---

# F286-DATA-R1.6: Live F119 Classification Wiring for Production Advisory

最終更新: 2026-05-11

## ★ 状態: 完了

F286-DATA-R1.5 で生成した r2f4_baseline_live_v1 / 2026-05-09 candidates
に F119 historical artifacts (= /tmp/f119_eval_*) を **既存 F111-R4
runner の `--f119-*` 引数経由** で適用。non_neutral_count=**30/30**、
boost flag=30 / avoid flag=6 / caution flag=24 を獲得。F062-R5.2
production + compact payload で **「🟢 結論: 買い検討候補あり」**
冒頭表示 + freshness guard natural pass を確認。実 LINE 送信なし。

## 1. 使用 F119 artifact 正本

| artifact | path | size 概要 |
|---|---|---|
| F119 summary JSON          | `/tmp/f119_eval_summary.json` | source_version=**r2f4_baseline_v1** / rule_version=**r2g3_recommended_v2** / base_dates=22 件 (2024-06 〜 2026-03) |
| F119 insights JSON          | `/tmp/f119_eval_insights.json` | thresholds + overall_mean_h_primary + strong/avoid/caution candidates lists |
| F119 strong candidates CSV  | `/tmp/f119_eval_strong_candidates.csv` | cut_type / group_key / count / mean_return_20d / win_rate_20d 等 |
| F119 avoid candidates CSV   | `/tmp/f119_eval_avoid_candidates.csv` | 同 |
| F119 caution candidates CSV | `/tmp/f119_eval_caution_candidates.csv` | 同 |

これらは「**過去 22 base_dates の検証で蓄積された cut_summaries
(= sector/月/regime 等の group_key で見た強い／弱い特徴の集合)**」。
F111-R4 が live candidates の特徴 (= base_date / sector / interpretation
/ month 等) と group_key を照合し、該当 cut が strong/avoid/caution
リストにあれば対応 flag を付与する。

★ 「live base_date の未来 return は評価しない」を遵守:
- F119 summary は historical 22 base_dates (= 2024-06 〜 2026-03) の
  ものを使用、live 2026-05-09 の future return は **計算していない**
- live candidates には `f119_expected_h20_return` / `_h5_return` =
  None (= 0 件)、metric_present_count=0
- 付与されるのは「historical 検証で boost/avoid/caution と判定された
  group_key に該当するか」の flag のみ

## 2. F111-R4 結果

実行:
```
.venv/bin/python -m scripts.jobs.run_f111_research_advisory_real_wiring_r4_smoke \
  --db staging --db-path data/fire.staging.db \
  --source-version r2f4_baseline_live_v1 \
  --rule-version r2g3_recommended_v2 \
  --base-date 2026-05-09 \
  --top-n 30 \
  --f119-summary-json /tmp/f119_eval_summary.json \
  --f119-insights-json /tmp/f119_eval_insights.json \
  --f119-strong-csv /tmp/f119_eval_strong_candidates.csv \
  --f119-avoid-csv /tmp/f119_eval_avoid_candidates.csv \
  --f119-caution-csv /tmp/f119_eval_caution_candidates.csv \
  --output-json /tmp/f286_data_r1_6_advisory_preview.json \
  --output-csv /tmp/f286_data_r1_6_advisory_preview.csv \
  --summary-json /tmp/f286_data_r1_6_advisory_summary.json
```

結果:
- `f119_artifact_status: json` (= artifact 経由で適用、WIRING ANOMALY 解消)
- candidate_count=30 / **non_neutral=30** ★
- boost=30 / avoid=6 / caution=24
- expected_h20=0 (= live は future return 未計算、設計通り)
- auto_order_allowed_true=0 / manual_review_required=30
- DB last_modified before / after 一致 (= read-only 維持)

### advisory_label 分布 (= 30 candidate)

| label | count |
|---|---|
| boost_with_caution | 24 |
| boost_with_avoid   | 6 |
| boost              | 0 |
| caution            | 0 |
| avoid              | 0 |
| neutral            | 0 |

全 30 candidate が boost 系。F062-R5.1 _BUY_LABELS は
`boost / boost_with_caution / boost_with_avoid` を含むため、冒頭結論
は **"🟢 結論: 買い検討候補あり"** ★

### F119 flag count

| flag | count (30 candidate 中) |
|---|---|
| f119_boost_flags non-empty       | **30** |
| f119_avoid_flags non-empty       | 6 |
| f119_caution_flags non-empty     | 30 |
| f119_expected_h20_return present | **0** (= live future return 未計算) |
| f119_expected_h5_return present  | **0** |

## 3. F062-R1 production + compact payload

実行:
```
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/f286_data_r1_6_advisory_preview.json \
  --summary-json /tmp/f286_data_r1_6_advisory_summary.json \
  --message-mode production --compact \
  --max-candidates 8 --max-per-label 4 \
  --include-labels avoid,boost_with_avoid,caution,boost_with_caution,boost \
  --output-json /tmp/f286_data_r1_6_line_payload.json \
  --output-text /tmp/f286_data_r1_6_line_preview.txt \
  --output-summary-json /tmp/f286_data_r1_6_line_summary.json
```

結果:
| field | value |
|---|---|
| message_mode                | production ✅ |
| compact                     | True ✅ |
| dry_run                     | False |
| send_intent                 | production-advisory |
| chunks                      | 1 |
| chunk[0] length             | 955 (compact だが label section 増で +276) |
| forbidden_phrase_count      | 0 ✅ |
| safety_footer_present       | True ✅ |
| selected_count              | 8 |
| selected_label_counts       | **boost_with_avoid: 4, boost_with_caution: 4** |
| metadata.payload_base_date  | 2026-05-09 |
| metadata.source_version     | r2f4_baseline_live_v1 |
| metadata.rule_version       | r2g3_recommended_v2 |

### chunk[0] 文面 (先頭抜粋)

```
FIRE 本番 Advisory
本番 LINE 通知 / 自動発注なし / 手動レビュー必須
source: r2f4_baseline_live_v1 / r2g3_recommended_v2
base_dates: 2026-05-09
selected_candidates: 8

🟢 結論: 買い検討候補あり        ★
候補は手動レビュー前提。自動発注なし。

Summary
candidates: 30
boost: 30 / avoid: 6 / caution: 24
non-neutral: 30
auto_order_allowed_true: 0
manual_review_required: 30/30
...
- 本番 LINE 通知 (production send)
- 自動発注なし
- 楽天操作なし
- Computer Use なし
- 注文価格・数量・執行指示は生成しない
- F119 の優位は 20d 寄り (daytrade 即時判断には使わない)
- Fujiwara manual review required
```

- "dry-run" / "LINE 送信なし (dry-run / template only)" 不在 ✅
- "本番 LINE 通知" / "自動発注なし" / "手動レビュー必須" 含む ✅
- 冒頭結論 "🟢 結論: 買い検討候補あり" ★
- F119 historical 由来の boost/avoid/caution が反映

## 4. F062-R5.2 payload freshness guard result

### dry-run (--send 無し) ✅
| field | value |
|---|---|
| mode                       | dry_run |
| send_allowed               | True |
| sent_count                 | 0 |
| line_api_call_count        | 0 |
| token_read_count           | 0 |
| forbidden_phrase_count     | 0 |
| safety_footer_present      | True |
| manual_review_required_count | 8 |
| auto_order_allowed_true_count | 0 |

### simulated send (--send + --hq-approved-send + --dry-run-line-api) ✅
| field | value |
|---|---|
| send_allowed                | True ★ |
| sent_count                  | 1 |
| line_api_call_count         | 1 |
| partial_delivery            | False |
| production_callable_built   | True |
| dry_run_line_api            | True (= 実 push_message 未呼出) |
| max_chunks                  | 1 |
| production_outcomes         | 1 件 (chunk_index=0 / status=dry_run / chunk_length=955 / recipient_type=user / recipient_hash8=b344b213) |
| **payload_freshness_check** | max_lag_days=10 / payload_base_date=2026-05-09 / gate_signal_max_base_date=2026-05-09 / **lag_calendar_days=0** ✅ |

→ F062-R5.2 freshness guard 自然 pass。実 LineBotClient.push_message
は呼ばれていない (= dry_run_line_api=True 経路)。

## 5. token / recipient leak 検査

`/tmp/f286_data_r1_6_*.{json,csv,txt}` 全件を grep:
- TOKEN_LEAK: **0**
- FULL_RECIPIENT: **0**

production_outcomes / production_config / LINE log file (= F236-R1
masked 形式) すべて leak free。

## 6. DB unchanged

| DB | mtime / size |
|---|---|
| data/fire.db          | May  7 16:12:38 2026 / 371,064,832 (unchanged ✅) |
| data/fire.develop.db  | May  7 18:14:26 2026 / 371,064,832 (unchanged ✅) |
| data/fire.staging.db  | May 11 11:48:57 2026 / 4,803,936,256 (本タスク内 unchanged ✅、F286-DATA-R1.5 末尾の状態を維持) |

本タスクは F111-R4 / F062-R1 / F062-R3 共に read-only / dry-run
mode。DB write 0、staging 含む 3 DB 全て本タスク内 unchanged。

## 7. 安全要件遵守

| 項目 | 結果 |
|---|---|
| LINE 送信なし (本タスク内、--dry-run-line-api で実 API 未呼出)        | ✅ |
| live base_date の未来 return を計算していない (= h20/h5 metric_present=0) | ✅ |
| F119 historical cut_summaries / insights を live に適用              | ✅ (artifact 経由) |
| freshness guard を緩めない (= default 10 のまま natural pass)         | ✅ |
| r2d_v1 へ勝手に切り替えない                                          | ✅ (= r2f4_baseline_live_v1 維持) |
| production / develop DB write                                       | 0 ✅ |
| staging DB write                                                    | 0 ✅ (本タスク内、F286-DATA-R1.5 末尾の +109 行で停止) |
| 自動発注 / 楽天操作 / Computer Use                                    | 0 ✅ |
| 注文価格 / 数量 / 執行指示                                            | 送信していない ✅ |
| TODO Excel                                                          | 未更新 ✅ |
| --no-verify                                                         | 不使用 ✅ |
| scripts/seed_pattern_layer1.py / historical_indicators.py           | 未接触 ✅ |
| unrelated modified を stage / commit                                 | しない ✅ |
| token / recipient 平文出力                                            | 0 ✅ |

## 8. tests 結果

本タスクはコード変更 0 件 (= 既存 F111-R4 runner の `--f119-*` 引数を
活用した artifact 注入のみ)。pytest 追加なし、既存ベースライン
**3,249 PASS** は維持 (= 別途検証不要)。

## 9. Codex pre-commit 結果

docs commit のみ → Codex review 対象外 (= scripts/hooks/pre-commit
の skip 判定)。fire コード変更なし。

## 10. 次に F062-R5.2 を再開してよいか

★ **再開可能**。

- DATA-R2 gate 全 5 段 PASS / line_send_allowed=True ✅
- payload freshness guard natural pass (lag=0) ✅
- F119 wiring 復活 (non_neutral=30、boost/caution/avoid 反映) ✅
- production + compact 文面が「🟢 結論: 買い検討候補あり」+ 候補 8 件
  + Safety footer 8 行 (production version) で揃っている ✅
- token / recipient leak 0、DB 全 unchanged ✅
- 本タスク内で実 LINE 送信していない ✅

F062-R5.2 を `--send + --hq-approved-send + --max-chunks 1` で
再起動すれば、Fujiwara LINE app に 1 通の本番 Advisory (= F119 反映、
冒頭結論 "🟢 結論: 買い検討候補あり"、length 955 文字) が届く構造。

ただし最終起動は HQ (Fujiwara) 判断後 (= 本タスクは検証まで)。

## 11. 注意: F282 weekly snapshot リスク

次回月曜 (2026-05-18 07:00 JST) に F282 weekly staging snapshot が
再実行され、本タスクで生成した r2f4_baseline_live_v1 / 2026-05-09
の 109 行は production fire.db snapshot で **再び消失** する見込み。
F062-R5.2 本起動は 5/11 〜 5/17 の 1 週間以内に完了させるか、
FIRE-OPS-R0 再発防止策案 1 (= 本番運用データを production fire.db
に書く運用統一) を並行で実装する必要あり。

## 12. 次タスク

1. ★ F286-DATA-R1.6 完了 (= 本書、live F119 wiring + payload 検証)
2. HQ (Fujiwara) が F062-R5.2 本起動を判断 (= 1 chunk 実 LINE 送信)
3. 並走候補:
   - FIRE-OPS-R0 再発防止策案 1 (= production write 運用統一) 設計
     レビュー
   - 03_design/F282_environment_isolation_*.md の運用ルール明文化
   - F286-DATA-R3 daily refresh cron 化 (= 案 1 と整合)
