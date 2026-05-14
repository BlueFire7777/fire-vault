---
id: FIRE-CODEX-R1-WAVE60-pilot-D3-results
phase: 本番 v0 中核 / Wave 60-pilot-D3 / D3 f062_preview Manual Live Pilot
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-18
pilot_day: D3
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D3_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_results.md
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/template_d3_real_artifact_prep.md
  - 04_daily/2026-05-18_manual_live_pilot_trade_plan.md
  - 04_daily/2026-05-18_manual_live_pilot_review.md
---

# Wave 60-pilot-D3 Results — D3 f062_preview Manual Live Pilot v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 D3 (= 2026-05-18 月) で artifact_source=f062_preview の AFTER-R1 MVP 成果物を
4 step manual sequence で取得、7 invariants hard check 全 PASS、Pilot GO 候補 ✓**

W60-launchd-pre で確立した **F111 → F062 runner → AFTER-R1 MVP** chain を初の
本格運用 path として実行。D1/D2 の synthetic_fixture/HOLD から **D3 で f062_preview
判定に到達**。

**ただし F111 input は手作り synth** (= 実 F111 朝 batch 未稼働) のため、
**pilot_use = eligible_with_caveat** とし、藤原さんが TDnet / 日経で材料の真実性を
**再確認した上で実 trade 判断**。

## §2 D3 4 step manual sequence 実行結果

### §2.1 Step 1: F111 advisory preview 用意

W60-launchd-pre smoke の 3 銘柄 synth (= 6920 レーザーテック / 4063 信越化学 /
7203 トヨタ) を `/tmp/fire_d3_prep/f111_preview_2026-05-18.json` に複製。

### §2.2 Step 2: DATA-R3 freshness (= 選択肢 A dry-run runner)

```bash
.venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh \
  --dry-run --execute-dry-run-subprocesses \
  --base-date 2026-05-18 \
  --freshness-report-json /tmp/fire_d3_prep/data_r3_freshness_2026-05-18.json \
  --include-placeholder-skipped
```

結果:
- mode=dry-run / db_label=staging / **db_row_writes=0** ✓
- base_date=2026-05-18 / source_version=r2f4_baseline_live_v1
- aggregate_exit_code=0
- **verdict=OK** ✓
- subrunners: f100/f101/f111/f119 全 ok

### §2.3 Step 3: F062 LINE preview runner --dry-run --require-freshness-ok

```bash
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/fire_d3_prep/f111_preview_2026-05-18.json \
  --output-json /tmp/fire_d3_prep/f062_actual_output_2026-05-18.json \
  --output-summary-json /tmp/fire_d3_prep/f062_summary_2026-05-18.json \
  --freshness-report-json /tmp/fire_d3_prep/data_r3_freshness_2026-05-18.json \
  --require-freshness-ok --dry-run
```

結果:
- freshness gate: freshness OK ✓
- input=3 / selected=2 / chunks=1 / **forbidden=0** ✓
- safety_footer=True / auto_order_allowed_true_count=0 /
  manual_review_required_count=3 / **line_send_count=0** ✓

### §2.4 Step 4: AFTER-R1 MVP

```bash
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all --base-date 2026-05-18 \
  --f062-preview-json /tmp/fire_d3_prep/f062_actual_output_2026-05-18.json \
  --f062-preview-summary-json /tmp/fire_d3_prep/f062_summary_2026-05-18.json \
  --data-r3-freshness-json /tmp/fire_d3_prep/data_r3_freshness_2026-05-18.json \
  --output-dir /Users/bluefire/fire/reports/after_r1
```

結果:
- artifacts=4 / ranking_size=2
- **artifact_source=f062_preview** ✓
- **f062_raw_kind=f062_actual_dict** ✓
- 4 file × (JSON + Markdown) = **8 file 生成** in `reports/after_r1/`

### §2.5 Step 5: 7 invariants hard check (= W60-launchd-pre 確立)

| # | invariant | 結果 |
|---|---|---|
| 1 | artifact_source = `f062_preview` | ✓ |
| 2 | f062_raw_kind = `f062_actual_dict` | ✓ |
| 3 | DATA-R3 freshness verdict = `OK` | ✓ |
| 4 | forbidden_check.passed = True | ✓ |
| 5 | material.auto_order_allowed = False | ✓ |
| 6 | material.manual_review_required = True | ✓ |
| 7 | safety_flags 13 keys 全 False | ✓ (13/13 keys) |

**✓ Pilot GO check PASSED — all 7 invariants satisfied** ✓

## §3 Top candidates (= morning_line_material 自動抽出)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | **6920** | **レーザーテック** | 積極的買い推奨 | 🟢 | 0.78 | 187.0 |
| 2 | **4063** | **信越化学** | 条件付き買い推奨 | 🟡 | 0.60 | 133.0 |

F111 input の 7203 トヨタ (= neutral label) は morning_line_material の top には
含まれない (= TOP_INCLUDE_LABELS 制約)。

### §3.1 Pattern matched (= pattern_candidate_report)

| pattern_id | matched | tickers |
|---|---|---|
| freshness_ok_high_confidence | 1 | 6920 |
| manual_review_active_label | 2 | 6920, 4063 |
| multi_reason_basis | 1 | 6920 |
| low_risk_note | 2 | 6920, 4063 |

全 pattern `status=candidate_only` / `validation_status=unvalidated`。

## §4 D3 Pilot judgment

### §4.1 構造的判定: GO 候補

7 invariants 全 PASS、**Pilot GO 候補**。

### §4.2 F111 synth caveat

- F111 朝 batch 未稼働 (= Wave 41 / 45 進捗待ち)
- F111 input は W60-launchd-pre smoke の sample を流用 (= 藤原さん手作り扱い)
- **pilot_use = eligible_with_caveat**
- 藤原さんが当日 TDnet / 日経で材料の真実性を **再確認した上で実 trade 判断**

### §4.3 値嵩株リスク (= 6920 レーザーテック特有)

- 6920 は 30,000-32,000 円帯 (= 値嵩株)
- 100 株単位で entry → 必要資金 300 万円規模
- 5% stop_loss → 1 株 1,500 円リスク × 100 株 = **150,000 円 想定リスク**
- **1 トレード最大損失 15,000 円 を大幅超過** → 100 株 entry 不可
- 代替: 4063 信越化学 (= 3,000-5,000 円帯、100 株 30-50 万円) も 1 株 250 円
  リスク × 100 株 = 25,000 円 → やはり超過 → 100 株 entry 困難
- **本日 100 株 entry は両候補ともリスク上限超過、skip 推奨も自然な選択**

## §5 作成 file 一覧

### §5.1 新規 (4 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 04_daily/2026-05-18_manual_live_pilot_trade_plan.md | D3 trade plan / 14 セクション / GO 候補 + F111 synth caveat 明示 |
| 04_daily/2026-05-18_manual_live_pilot_review.md | D3 review blank / 15 セクション / artifact_source=f062_preview 特記 |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D3_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D3_results.md | 本 wave results (= 本 doc) |

### §5.2 reports/after_r1/ 追加 (= 8 file、D3 用)

- paper_live_ledger_2026-05-18.{json,md}
- good_candidate_ranking_2026-05-18.{json,md}
- pattern_candidate_report_2026-05-18.{json,md}
- morning_line_material_2026-05-18.{json,md}

(= artifact_source=f062_preview / ranking_size=2)

### §5.3 /tmp/fire_d3_prep/ (= sequence 中間 artifact)

| file | 内容 |
|---|---|
| f111_preview_2026-05-18.json | F111 synth 3 銘柄 |
| data_r3_freshness_2026-05-18.json | DATA-R3 verdict=OK |
| f062_actual_output_2026-05-18.json | F062 runner output (= 16 keys dict) |
| f062_summary_2026-05-18.json | F062 summary |

### §5.4 FIRE 本体不触 ✓

- scripts/jobs/ 不触 / tests/ 不触
- pytest collected: 4617 (= W60-launchd-pre と同じ、不変)

## §6 D3 リスク上限 (= operation_plan §5 / m=0 R-39-02 整合)

| 項目 | 上限 | D3 想定 |
|---|---|---|
| 1 日銘柄数 | 最大 1-2 / **D3 = 1 銘柄推奨** | (= 6920 / 4063 / skip) |
| 1 トレード最大損失 | 5,000-15,000 円 | **6920 100 株は 150,000 円相当、超過** |
| 1 日最大損失 | 20,000-30,000 円 | __ |
| ナンピン / 追加買い / 持ち越し | 禁止 | — |
| 信用 15:10 close | 必須 | — |
| 自動発注 | 0 (= 構造的) | ✓ |
| 楽天 / iSPEED 操作 | 藤原さん手動のみ | ✓ |
| Computer Use | 不採用 | ✓ |

## §7 Codex 8 lane 判断: 不使用 (= 本線主導)

ユーザー spec で「日次 trade plan なので本線主導でよい」と明示。

判断:
- D1 / D2 / D3 で同 pattern (= 日次 trade plan)
- W60-launchd-pre で manual sequence + 7 invariants hard check を Codex audit 済
- 日次毎に Codex 8 lane は過剰、**W1 集約 (= W60-pilot-W1) で実施**

代わりに本線 self-check 実施:
- ✓ template_d3_real_artifact_prep.md の 4 step 全完遂
- ✓ 7 invariants hard check 全 PASS 自動確認
- ✓ artifact_source / f062_raw_kind 機械的確認
- ✓ 命令形 phrase 0
- ✓ auto_order_allowed=False / manual_review_required=True 全箇所明示

## §8 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 (= F062 + DATA-R3 + AFTER-R1 全 dry-run 系) |
| DB sqlite 接続 | staging 1 (= DATA-R3 dry-run の SELECT のみ) / production-develop 0 |
| 3 DB row_writes | 0 (= DATA-R3 出力で確認) |
| LINE 送信 / linebot import | 0 (= F062 --dry-run、--hq-approved-send 不渡し) |
| token / channel_token / .env / env 全体参照 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| **実発注** | 0 (= claude code は何も発注しない) |
| **楽天証券 / iSPEED 操作** | 0 |
| **Computer Use** | 0 |
| 自動発注 | 0 |
| **sudo / git push / rm -rf / --no-verify** | 0 |
| workflow / TODO Excel 更新 | 0 |
| FIRE 本体コード変更 | 0 |
| file write 範囲 | reports/after_r1/ (= 8 file) + fire-vault/04_daily,02_todo/ (= 4 file) + /tmp/fire_d3_prep/ (= 4 中間 file) |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4617 (= 不変) |

## §9 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **0/8 = 0%** (= 本線主導、強い技術的理由 §7) |
| 短縮率 | N/A |
| 採用率 | N/A |
| 差戻率 | 0 |
| Integrator 負荷 | 中-高 (= D3 4 step sequence 実行 + 7 invariants check + trade plan + review + vault) |
| 安全事故 | **0** ✓ |

### Codex 0/8 の理由

- 日次 trade plan / 本線主導 wave (= user spec で明示)
- W60-launchd-pre で manual sequence + invariants check を Codex audit 済
- W1 集約 (W60-pilot-W1) で Codex 8 lane を実施するほうが効率的

## §10 今日 (5/18 月) の人間アクション

### §10.1 藤原さん が朝行うこと

1. `~/fire-vault/04_daily/2026-05-18_manual_live_pilot_trade_plan.md` 確認
2. §3.0 artifact_source = f062_preview / pilot_use = eligible_with_caveat 確認
3. §3.0.1 input chain 由来確認 (= F111 synth → F062 → AFTER-R1)
4. §3.0.2 F111 synth caveat 認識
5. §4 top candidates (= 6920 レーザーテック / 4063 信越化学) を TDnet / 日経で
   材料の真実性 **再確認**
6. §6 ticker 選択 (= 値嵩株リスクで skip も自然)
7. enter なら §7-§11 entry plan / SL / TP / checklist 全 PASS → iSPEED 手動発注
8. 15:10 までに手動 close (信用)

### §10.2 藤原さん が場後 行うこと

1. `~/fire-vault/04_daily/2026-05-18_manual_live_pilot_review.md` 記入
2. §8.4 D3 運用フロー評価 (= W60-launchd-pre 4 step の使い勝手) 重要
3. §13 next action 選択

### §10.3 Claude Code 行わないこと (= 絶対禁止)

- 実発注 / 楽天 / iSPEED / Computer Use / 自動発注 / LINE / DB write /
  token / API / launchctl / plist / cron / workflow / git push / sudo / rm

## §11 残課題 / 次 Wave

### §11.1 D3 → D5 path

- D3 (5/18 月、本日): artifact_source=f062_preview / GO 候補 + F111 synth caveat
- D4 (5/19 火): 同 pattern で trade plan (= F111 朝 batch 進捗による)
- D5 (5/20 水): 同 pattern、W60-pilot-W1 集約準備
- W60-pilot-W1 (= 5/21 木以降): D1-D5 集約 + Codex 8 lane audit

### §11.2 並行 wave

- **F111 朝 batch 設計** (= 別 wave、synth 脱却の本丸): 真の f062_preview path 確立
- **W60-launchd-real**: 本番 launchd 配置 (= Wave 41 / 45 進捗後)
- **W61-pre**: price/return data 取り込み + paper_pnl 連携

### §11.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §11.4 D1-D5 path artifact_source 推移

| Day | 日付 | 曜日 | artifact_source | 判定 | 完了状況 |
|---|---|---|---|---|---|
| D1 | 2026-05-14 | 木 | synthetic_fixture | HOLD | ✓ 完了 |
| D2 | 2026-05-15 | 金 | synthetic_fixture | HOLD | ✓ 完了 |
| - | 2026-05-16 | 土 | - | skip | - |
| - | 2026-05-17 | 日 | - | skip | - |
| **D3** | **2026-05-18** | **月** | **f062_preview** ✓ | **GO 候補 (+ F111 caveat)** | **✓ 本 wave** |
| D4 | 2026-05-19 | 火 | f062_preview 期待 | GO 候補期待 | 予定 |
| D5 | 2026-05-20 | 水 | f062_preview 期待 | GO 候補期待 | 予定 |

## §12 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_PILOT_D3_plan]]
- [[FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_results|W60-launchd-pre results]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D2_results|W60-pilot-D2 results]]
- [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- [[../04_daily/template_d3_real_artifact_prep|D3 manual command sequence]]
- [[../04_daily/2026-05-18_manual_live_pilot_trade_plan|5/18 D3 trade plan]]
- [[../04_daily/2026-05-18_manual_live_pilot_review|5/18 D3 review (blank)]]
