---
id: FIRE-CODEX-R1-WAVE60-pilot-D4-results
phase: 本番 v0 中核 / Wave 60-pilot-D4 / D4 f062_preview Manual Live Pilot
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-19
pilot_day: D4
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D4_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D3_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_results.md
  - 04_daily/template_d3_real_artifact_prep.md
  - 04_daily/2026-05-19_manual_live_pilot_trade_plan.md
  - 04_daily/2026-05-19_manual_live_pilot_review.md
---

# Wave 60-pilot-D4 Results — D4 f062_preview Manual Live Pilot v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 D4 (= 2026-05-19 火) で artifact_source=f062_preview / 7 invariants 全 PASS /
Pilot GO 候補 (= F111 synth caveat 継続) ✓**

D3 で確立した 4 step manual sequence を再実行、**2 営業日連続で f062_preview
判定 + invariants check PASS** を実証。

F111 朝 batch / F062 朝 batch / morning-advisory **launchd plist 未配置** を baseline
で再確認、F111 input source = `synthetic_sample` (= D3 と同 sample 流用) で運用。

## §2 D4 4 step manual sequence 実行結果

### §2.1 Step 1: F111 advisory preview 用意

D3 の 3 銘柄 synth (= 6920 レーザーテック / 4063 信越化学 / 7203 トヨタ neutral)
を `/tmp/fire_d4_prep/f111_preview_2026-05-19.json` に複製。

### §2.2 Step 2: DATA-R3 freshness (= 選択肢 A dry-run runner)

```bash
.venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh \
  --dry-run --execute-dry-run-subprocesses \
  --base-date 2026-05-19 \
  --freshness-report-json /tmp/fire_d4_prep/data_r3_freshness_2026-05-19.json \
  --include-placeholder-skipped
```

結果:
- mode=dry-run / db_label=staging / **db_row_writes=0** ✓
- base_date=2026-05-19 / verdict=**OK** ✓
- 4 sub-runner (f100/f101/f111/f119) 全 ok

### §2.3 Step 3: F062 runner --dry-run --require-freshness-ok

結果:
- freshness gate: freshness OK ✓
- input=3 / selected=2 / chunks=1 / **forbidden=0** ✓
- safety_footer=True / auto_order_allowed_true_count=0 /
  manual_review_required_count=3 / **line_send_count=0** ✓

### §2.4 Step 4: AFTER-R1 MVP

結果:
- artifacts=4 / ranking_size=2
- **artifact_source=f062_preview** ✓
- **f062_raw_kind=f062_actual_dict** ✓
- 8 file (= JSON+Markdown × 4) in `reports/after_r1/`

### §2.5 Step 5: 7 invariants hard check (= 全 PASS ✓)

| # | invariant | 結果 |
|---|---|---|
| 1 | artifact_source = `f062_preview` | ✓ |
| 2 | f062_raw_kind = `f062_actual_dict` | ✓ |
| 3 | DATA-R3 freshness verdict = `OK` | ✓ |
| 4 | forbidden_check.passed = True | ✓ |
| 5 | material.auto_order_allowed = False | ✓ |
| 6 | material.manual_review_required = True | ✓ |
| 7 | safety_flags 13 keys 全 False | ✓ (13/13 keys) |

**✓ Pilot GO check PASSED — all 7 invariants satisfied**

## §3 Top candidates (= D3 と同、F111 同 sample 流用のため)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | **6920** | **レーザーテック** | 積極的買い推奨 | 🟢 | 0.78 | 187.0 |
| 2 | **4063** | **信越化学** | 条件付き買い推奨 | 🟡 | 0.60 | 133.0 |

### §3.1 D3 → D4 連続性の意味

F111 が D3 と同 sample なので候補も同じ:
- D3 で skip 選択 → D4 でも同候補が再提示 → 同判断軸で再評価可
- D3 で entry 選択 → D4 同候補は **ナンピン抵触**、watch / skip 推奨
- D3 で 4063 entry → D4 6920 検討 (= 1-2 銘柄ルール内)

## §4 F111 input source 判定 (= 新規 D4 観点)

| 確認項目 | 結果 |
|---|---|
| F111 朝 batch launchd plist | **未配置** ✓ |
| F062 朝 batch launchd plist | **未配置** |
| morning-advisory launchd plist | **未配置** |
| Wave 41 (= HQ_APPROVE_LAUNCHD_DAILY) 進捗 | 5/19-5/26 期間 (= 着手の可能性) |
| **F111 input source 結論** | **synthetic_sample** (= D3 と同) |

→ D4 朝も **synth 継続**、pilot_use = `eligible_with_caveat`

## §5 D4 Pilot judgment

### §5.1 構造的判定: GO 候補 (= 7 invariants PASS)

### §5.2 F111 synth caveat 継続

- F111 朝 batch 未稼働 (= D3 から不変)
- f111_input_source = `synthetic_sample`
- pilot_use = `eligible_with_caveat`

### §5.3 値嵩株リスク (= D3 から再確認)

- 6920 100 株 ≈ 300 万円 / 5% SL ≈ 150,000 円 損失 → **上限 15,000 円 超過**
- 4063 100 株 ≈ 30-50 万円 / 5% SL ≈ 25,000 円 損失 → **超過**
- 両候補 100 株 entry リスク管理困難 → **skip も自然選択**

### §5.4 D4 推奨

**skip** または **watch** が藤原さんの自然選択肢:
1. F111 input が D3 と同 synth (= 新規材料なし)
2. F111 朝 batch 未稼働継続
3. 値嵩株リスクで 100 株 entry 困難
4. D3 で同候補 check 済 → D4 で新規 entry 理由が弱い

ただし「7 invariants 全 PASS、構造的 GO」は claude code 側判定。

## §6 作成 file 一覧

### §6.1 新規 (4 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 04_daily/2026-05-19_manual_live_pilot_trade_plan.md | D4 trade plan / 14 セクション / GO 候補 + F111 caveat 継続 + 値嵩株リスク再確認 |
| 04_daily/2026-05-19_manual_live_pilot_review.md | D4 review blank / 15 セクション / D3 → D4 連続性 特記 |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D4_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D4_results.md | 本 wave results (= 本 doc) |

### §6.2 reports/after_r1/ 追加 (= 8 file、D4 用)

paper_live_ledger / good_candidate_ranking / pattern_candidate_report /
morning_line_material × (JSON + Markdown) = 8 file、artifact_source=f062_preview。

### §6.3 /tmp/fire_d4_prep/ (= sequence 中間 artifact 4 件)

f111_preview / data_r3_freshness / f062_actual_output / f062_summary
の 2026-05-19 版。

### §6.4 FIRE 本体不触 ✓

- scripts/jobs/ 不触 / tests/ 不触
- pytest collected: 4617 (= W60-launchd-pre と同じ、不変)

## §7 D4 リスク上限 (= operation_plan §5 / m=0 R-39-02 整合)

| 項目 | 上限 | D4 想定 |
|---|---|---|
| 1 日銘柄数 | 最大 1-2 / D4 = 1 銘柄推奨、skip 自然 | __ |
| 1 トレード最大損失 | 5,000-15,000 円 | (= 値嵩株 100 株超過) |
| 1 日最大損失 | 20,000-30,000 円 | __ |
| ナンピン / 追加買い / 持ち越し | 禁止 | — |
| 信用 15:10 close | 必須 | — |
| 自動発注 | 0 (= 構造的) | ✓ |
| Computer Use | 不採用 | ✓ |

## §8 Codex 8 lane 判断: 不使用 (= 本線主導)

ユーザー spec で「日次 trade plan は本線主導」明示。
W1 集約 (= W60-pilot-W1) で 4 lane 以上を使用。

判断:
- D1 / D2 / D3 で同 pattern (= 本線主導)
- W60-launchd-pre / D3 で sequence + invariants check を Codex audit 済
- D4 は同 sequence の再実行、追加 Codex 価値が低い
- W1 で集約 audit 効率的

本線 self-check:
- ✓ 4 step 全完遂、exit code 0
- ✓ 7 invariants 自動 hard check 全 PASS
- ✓ F111 朝 batch 稼働状況 確認 (= 未配置 → synth 継続)
- ✓ 命令形 phrase 0 / auto_order=False / manual_review=True 維持

## §9 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 | staging 1 (= DATA-R3 dry-run SELECT のみ) / production-develop 0 |
| 3 DB row_writes | 0 (= DATA-R3 出力で確認) |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| 実発注 / 楽天 / iSPEED / Computer Use | 0 |
| 自動発注 | 0 |
| sudo / git push / rm -rf / --no-verify | 0 |
| workflow / TODO Excel | 0 |
| FIRE 本体コード変更 | 0 |
| file write 範囲 | reports/after_r1/ (= 8) + fire-vault/* (= 4) + /tmp/fire_d4_prep/ (= 4) |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4617 (= 不変) |

## §10 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **0/8 = 0%** (= 本線主導、強い技術的理由) |
| 短縮率 | N/A |
| 採用率 | N/A |
| 差戻率 | 0 |
| Integrator 負荷 | 中 (= 4 step 実行 + 7 invariants + trade plan + review + vault) |
| 安全事故 | **0** ✓ |

### Codex 0/8 の理由

- 本線主導 wave (= user spec で明示)
- W60-launchd-pre / D3 で Codex audit 済の sequence の再実行
- W1 集約で Codex 4 lane 以上を使用予定

## §11 今日 (5/19 火) の人間アクション

### §11.1 藤原さん が朝行うこと

1. `~/fire-vault/04_daily/2026-05-19_manual_live_pilot_trade_plan.md` 確認
2. §3.0 artifact_source=f062_preview / f111_input_source=synthetic_sample 確認
3. §3.0.1 input chain 確認
4. F111 朝 batch 稼働状況確認 (= §4 結論: 未稼働)
5. §4 top candidates (= D3 と同) 確認 → 新規材料 / 値嵩株リスク
6. §6 ticker 選択 (= skip / watch / enter)
7. enter なら §7-§11 manual checklist → iSPEED 手動発注 / 15:10 close

### §11.2 場後 行うこと

1. `~/fire-vault/04_daily/2026-05-19_manual_live_pilot_review.md` 記入
2. §8.4 D4 運用フロー評価 (= D3 → D4 連続性 + F111 caveat 継続感) 重要
3. §13 next action 選択 (= D5 / W1 準備)

### §11.3 Claude Code 行わないこと (= 絶対禁止)

- 実発注 / 楽天 / iSPEED / Computer Use / 自動発注 / LINE / DB write /
  token / API / launchctl / plist / cron / workflow / git push / sudo / rm

## §12 残課題 / 次 Wave

### §12.1 D4 → D5 path

- D5 (= 2026-05-20 水): 同 sequence で trade plan + review
- W60-pilot-W1 (= 2026-05-21 木以降): D1-D5 集約 + Codex 8 lane

### §12.2 並行 wave

- F111 朝 batch 設計 (= 別 wave、synth 脱却の本丸)
- W60-launchd-real: 本番 launchd 配置 (= Wave 41 / 45 進捗後)
- W61-pre: price/return data 取り込み + paper_pnl 連携

### §12.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill ✓ (= 5/16-5/19 期間で完了見込み)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial **← D4 当日着手の可能性**
- Wave 45 (5/26-6/2)
- Wave 52 (6/2-6/8)
- Wave 53 (6/9 D-Day)

### §12.4 D1-D5 path artifact_source 推移

| Day | 日付 | 曜日 | artifact_source | F111 input | 判定 | 完了状況 |
|---|---|---|---|---|---|---|
| D1 | 2026-05-14 | 木 | synthetic_fixture | (= MVP 直渡し) | HOLD | ✓ |
| D2 | 2026-05-15 | 金 | synthetic_fixture | (= MVP 直渡し) | HOLD | ✓ |
| - | 2026-05-16 | 土 | - | - | skip | - |
| - | 2026-05-17 | 日 | - | - | skip | - |
| D3 | 2026-05-18 | 月 | f062_preview ✓ | synthetic_sample | GO + caveat | ✓ |
| **D4** | **2026-05-19** | **火** | **f062_preview ✓** | **synthetic_sample** | **GO + caveat 継続 ★** | **✓ 本 wave** |
| D5 | 2026-05-20 | 水 | f062_preview 期待 | 朝 batch 稼働期待 | GO 期待 | 予定 |

## §13 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_PILOT_D4_plan]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D3_results|D3 results]]
- [[FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_results|W60-launchd-pre results]]
- [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- [[../04_daily/template_d3_real_artifact_prep|D3 manual command sequence]]
- [[../04_daily/2026-05-19_manual_live_pilot_trade_plan|5/19 D4 trade plan]]
- [[../04_daily/2026-05-19_manual_live_pilot_review|5/19 D4 review (blank)]]
