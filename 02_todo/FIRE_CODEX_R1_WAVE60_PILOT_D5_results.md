---
id: FIRE-CODEX-R1-WAVE60-pilot-D5-results
phase: 本番 v0 中核 / Wave 60-pilot-D5 / D5 f062_preview + f111_sample
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-20
pilot_day: D5
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D5_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_F111_PRE_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D4_results.md
  - 04_daily/template_d5_real_artifact_prep.md
  - 04_daily/2026-05-20_manual_live_pilot_trade_plan.md
  - 04_daily/2026-05-20_manual_live_pilot_review.md
---

# Wave 60-pilot-D5 Results — D5 f062_preview + f111_sample v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 D5 (= 2026-05-20 水) で f062_preview + f111_sample / 8 invariants 全 PASS /
Pilot GO 候補 with minor caveat ✓ — D1-D5 = 5 営業日 pilot 完遂 ✓**

W60-F111-pre 確立の **F111 --sample → F062 → AFTER-R1** chain を実行、
**D3/D4 manual_seed 強 caveat → D5 f111_sample 中 caveat** へ進展。
8 invariants hard check (= W60-launchd-pre 7 + W60-F111-pre 追加 #8) 全 PASS。

ただし D5 candidate ticker は サンプル名 (= 1234 / 7203 サンプル...) のため
**実 trade は構造的に skip 必須**。**D5 = F111 own ロジック検証 wave**。
完全 `f111_real_batch` は次 wave (= F111-real-batch) で対応。

## §2 D5 5 step chain 実行結果

### §2.1 Step 1: F111 runner --sample (= W60-F111-pre 確立)

```bash
.venv/bin/python -m scripts.jobs.run_f111_research_advisory_preview \
  --sample --max-candidates 10 --dry-run \
  --output-json /tmp/fire_d5_prep/f111_real_sample_2026-05-20.json \
  --summary-json /tmp/fire_d5_prep/f111_summary_2026-05-20.json
```

結果:
- input=6 candidates / dry_run=True
- safety: no LINE / no order / no broker / no rakuten / no Computer Use / no DB write
- done: candidate_count=6 / with_advisory=5 / missing_advisory=1 /
  auto_order_allowed_true_count=0 / manual_review_required_count=6

### §2.2 Step 2: DATA-R3 freshness (= 選択肢 A)

```bash
.venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh \
  --dry-run --execute-dry-run-subprocesses \
  --base-date 2026-05-20 \
  --freshness-report-json /tmp/fire_d5_prep/data_r3_freshness_2026-05-20.json \
  --include-placeholder-skipped
```

結果:
- mode=dry-run / db_label=staging / db_row_writes=0 ✓
- base_date=2026-05-20 / **verdict=OK** ✓
- 4 sub-runner 全 ok

### §2.3 Step 3: F062 LINE preview runner

```bash
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/fire_d5_prep/f111_real_sample_2026-05-20.json \
  --output-json /tmp/fire_d5_prep/f062_actual_output_2026-05-20.json \
  --output-summary-json /tmp/fire_d5_prep/f062_summary_2026-05-20.json \
  --freshness-report-json /tmp/fire_d5_prep/data_r3_freshness_2026-05-20.json \
  --require-freshness-ok --dry-run
```

結果:
- freshness gate: freshness OK ✓
- input=6 / selected=4 / chunks=1 / forbidden=0 ✓ / line_send_count=0 ✓

### §2.4 Step 4: AFTER-R1 MVP

結果:
- artifacts=4 / ranking_size=4
- **artifact_source=f062_preview** ✓
- **f062_raw_kind=f062_actual_dict** ✓
- **f111_input_source=f111_sample** ✓ (= W60-F111-pre 自動推定)

### §2.5 Step 5: 8 invariants hard check (= 全 PASS ✓)

| # | invariant | 結果 |
|---|---|---|
| 1 | artifact_source = `f062_preview` | ✓ |
| 2 | f062_raw_kind = `f062_actual_dict` | ✓ |
| 3 | **f111_input_source ∈ `{f111_sample, f111_real_batch}`** | **✓ f111_sample** |
| 4 | DATA-R3 freshness verdict = `OK` | ✓ |
| 5 | forbidden_check.passed = True | ✓ |
| 6 | material.auto_order_allowed = False | ✓ |
| 7 | material.manual_review_required = True | ✓ |
| 8 | safety_flags 13 keys 全 False | ✓ (13/13 keys) |

**✓ Pilot GO check PASSED — all 8 invariants satisfied**

## §3 Top candidates (= F111 --sample 由来)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | **1234** | サンプル情報通信A | 条件付き買い推奨 | 🟡 | 0.82 | 136.0 |
| 2 | **7203** | サンプル輸送機D | 条件付き買い推奨 | 🟡 | 0.78 | 129.0 |

(= D3/D4 と異なる candidates、F111 own ロジック由来)

### §3.1 サンプル ticker = 実 trade 構造的不可

- candidate name に「サンプル」プレフィックス
- 楽天証券 / iSPEED で発注しても銘柄存在しない
- → **D5 実 trade は構造的に skip 必須**
- D5 = F111 own ロジック検証 wave、実 trade は次 wave

## §4 D5 Pilot judgment: GO with minor caveat (= 構造的 skip)

### §4.1 GO 構造的判定: 8 invariants 全 PASS ✓

### §4.2 minor caveat (= D3/D4 強 caveat より進展)

- F111 SAMPLE_CANDIDATES は F111 own ロジック経由
  (= research_advisory 版本管理、F119 score 連携、multi-面評価)
- D3/D4 manual_seed (= 藤原さん手作り) より評価品質高い
- ただし staging real candidates ではない → minor caveat

### §4.3 実 trade 構造的不可

候補が サンプル ticker → 実 trade 構造的に skip。
完全 `f111_real_batch` 化で初めて実 trade 可。

### §4.4 進化記録 (= D3 → D4 → D5)

| Day | f111_input_source | caveat 強度 | 実 trade 可? |
|---|---|---|---|
| D3 (5/18) | manual_seed | 強 | 実在 ticker、値嵩株なら可 |
| D4 (5/19) | manual_seed | 強 | D3 と同 |
| **D5 (5/20)** | **f111_sample** | **中** | **構造的不可 (= サンプル ticker)** |
| 将来 | f111_real_batch | 最弱 | 実 trade 可 |

## §5 作成 file 一覧

### §5.1 新規 (4 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 04_daily/2026-05-20_manual_live_pilot_trade_plan.md | D5 trade plan / 15 セクション / GO + minor caveat + サンプル skip 必須 |
| 04_daily/2026-05-20_manual_live_pilot_review.md | D5 review blank / 15 セクション / D5 = F111 own 検証 特記 |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D5_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D5_results.md | 本 wave results (= 本 doc) |

### §5.2 reports/after_r1/ (= 8 file 追加、D5 用 base_date=2026-05-20)

### §5.3 /tmp/fire_d5_prep/ (= 4 中間 artifact)

f111_real_sample_2026-05-20.json (= F111 32-field output) /
data_r3_freshness_2026-05-20.json / f062_actual_output_2026-05-20.json /
f062_summary_2026-05-20.json

### §5.4 FIRE 本体不触 ✓

- scripts/jobs/ / tests/ 全て 不触
- pytest collected: 4629 (= W60-F111-pre と同じ、不変)

## §6 W1 集約への引き継ぎ (= D1-D5 完遂)

### §6.1 D1-D5 path 完了 table

| Day | 日付 | 曜日 | artifact_source | f111_input_source | judgment | 実 entry |
|---|---|---|---|---|---|---|
| D1 | 5/14 木 | - | synthetic_fixture | (= MVP 直渡し) | HOLD | 0 |
| D2 | 5/15 金 | - | synthetic_fixture | (= MVP 直渡し) | HOLD | 0 |
| D3 | 5/18 月 | - | f062_preview | manual_seed | GO + 強 caveat | 0 (skip) |
| D4 | 5/19 火 | - | f062_preview | manual_seed | GO + 強 caveat | 0 (skip) |
| **D5** | **5/20 水** | - | **f062_preview** | **f111_sample** | **GO + 中 caveat** | **0 (構造的 skip)** |

### §6.2 W60-pilot-W1 集約 観点 (= 5/21 木以降)

1. **artifact_source 推移評価**: 2/5 (= 40%) synthetic_fixture → 3/5 (= 60%)
   f062_preview / 平均 caveat 弱化
2. **f111_input_source 進展評価**: 0/5 → manual_seed×2 → f111_sample×1
   (= 段階的進展)
3. **実 entry 0/5**: 全 day で entry なし、運用フロー検証 wave として完遂
4. **skip/watch 判断理由の質**: D3/D4 値嵩株 / D5 サンプル ticker、
   いずれも明確な構造的理由
5. **値嵩株リスク再確認**: D3/D4 で 6920 / 4063 値嵩株 100 株単位リスク管理
   困難確認
6. **pattern promote/suppress/watch 候補**: D1-D5 で 4 pattern 全 watched 想定、
   W1 で結論
7. **F111 朝 batch 化への次 wave 計画**: F111-real-batch wave (= staging real
   candidates 化)
8. **Codex 並列度傾向**: W60.5 4 lane 100% / W60-F111-pre 4 lane 0% / W60-pilot
   系 0/8 自然 / W1 集約で 改善試行
9. **8 invariants hard check 運用感覚**: 自動 fail 検出 = 安全に skip 判断可
10. **Stage 3 昇格条件 (= R-19-08) 突合**: trade 0/50 / 20 営業日進行 0/20 /
    1 週間で達成不可、次 段階で計画

## §7 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 (production/develop) | 0 / staging 1 (= DATA-R3 dry-run SELECT) |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| **実発注** | 0 |
| **楽天 / iSPEED 操作** | 0 |
| **Computer Use** | 0 |
| 自動発注 | 0 |
| sudo / git push / rm -rf / --no-verify | 0 |
| workflow / TODO Excel | 0 |
| FIRE 本体コード変更 | 0 |
| file write 範囲 | reports/after_r1/ (= 8 file) + fire-vault/* (= 4 file) + /tmp/fire_d5_prep/ (= 4 file) |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4629 (= 不変) |

## §8 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **0/8 = 0%** (= 本線主導、強い技術的理由) |
| 短縮率 | N/A |
| 採用率 | N/A |
| 差戻率 | 0 |
| Integrator 負荷 | 中 (= 5 step chain + 8 invariants + trade plan + review + W1 引き継ぎ + vault) |
| 安全事故 | **0** ✓ |

### Codex 0/8 の理由

- 日次 trade plan / 本線主導 (= user spec で明示)
- W60-F111-pre / D3 / D4 で sequence + invariants check を Codex audit 済
- W1 集約 (= W60-pilot-W1) で Codex 4 lane 以上を実施予定

本線 self-check:
- ✓ 5 step 全完遂、exit code 0
- ✓ 8 invariants 自動 hard check 全 PASS
- ✓ F111 sample marker 8 keys 自動検出 → f111_sample 推定
- ✓ サンプル ticker 構造的 skip 明示
- ✓ 命令形 phrase 0 / auto_order=False / manual_review=True 維持

## §9 今日 (5/20 水) の人間アクション

### §9.1 藤原さん が朝行うこと

1. `~/fire-vault/04_daily/2026-05-20_manual_live_pilot_trade_plan.md` 確認
2. §3.0 artifact_source=f062_preview / f111_input_source=f111_sample 確認
3. §3.0.1 input chain (= F111 --sample → F062 → AFTER-R1) 確認
4. §4 top candidates (= 1234 / 7203 サンプル...) が **サンプル ticker** と認識
5. **§13 final decision = skip** (= 標準推奨、サンプル ticker のため構造的不可)

### §9.2 場後 行うこと

1. `~/fire-vault/04_daily/2026-05-20_manual_live_pilot_review.md` 記入
2. §8.4 D5 運用フロー評価 (= W60-F111-pre 5 step / f111_sample 信頼性) 重要
3. §13 next action = **W60-pilot-W1 集約 wave 着手** ★

### §9.3 Claude Code 行わないこと (= 絶対禁止)

実発注 / 楽天 / iSPEED / Computer Use / 自動発注 / LINE / DB write /
token / API / launchctl / plist / cron / workflow / git push / sudo / rm

## §10 残課題 / 次 Wave

### §10.1 D5 → W1 集約

- **W60-pilot-W1 (= 5/21 木以降)**: D1-D5 5 営業日 集約 + Codex 4 lane audit +
  pattern promote/suppress/watch 結論
- **F111-real-batch (= 別 wave、優先度高)**: staging → F111 --candidate-json で
  f111_real_batch 化 → 実 trade 可能候補生成
- **W60-launchd-real**: 本番 launchd 配置 (= Wave 41 / 45 進捗後)

### §10.2 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
  → 5/19 Official GO 判定見込み (= D4 当日)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §10.3 D1-D5 5 営業日 pilot 完遂の意義

- AFTER-R1 MVP / good_candidate_ranking / pattern_candidate_report /
  morning_line_material の **運用フロー 5 日間検証** 完了
- artifact_source 識別 (= W60-integration) / f111_input_source 識別
  (= W60-F111-pre) の **2 重 traceability** 確立
- 8 invariants hard check の **自動化** 確立
- Pilot GO/HOLD/NO-GO 判定 **基準確立**
- 値嵩株リスク / サンプル ticker / F111 caveat 等 **実 entry 阻害要因の特定**
- Stage 3 (= R-19-08) 昇格には **F111-real-batch wave** + 実 trade 50 回 が必要

## §11 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_PILOT_D5_plan]]
- [[FIRE_CODEX_R1_WAVE60_F111_PRE_results|W60-F111-pre results]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D4_results|D4 results]]
- [[../03_design/F111_real_artifact_contract_2026-05-14|F111 contract v1.0]]
- [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- [[../04_daily/template_d5_real_artifact_prep|D5 manual sequence]]
- [[../04_daily/2026-05-20_manual_live_pilot_trade_plan|5/20 D5 trade plan]]
- [[../04_daily/2026-05-20_manual_live_pilot_review|5/20 D5 review (blank)]]
