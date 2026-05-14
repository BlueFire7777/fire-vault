---
id: FIRE-CODEX-R1-WAVE60-F111-pre-results
phase: 本番 v0 中核 / Wave 60-F111-pre / F111 Real Artifact Path
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_F111_PRE_plan.md
  - 03_design/F111_real_artifact_contract_2026-05-14.md
  - 04_daily/template_d5_real_artifact_prep.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D4_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_results.md
---

# Wave 60-F111-pre Results — F111 Real Artifact Path v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 F111 runner `--sample` → F062 runner → AFTER-R1 MVP chain で
`f111_input_source = f111_sample` 自動推定成功 ✓**

D3/D4 で残った最大課題である **synthetic_sample (= 藤原さん手作り) 依存** を
**F111 runner own ロジック (= f111_sample)** へ進展。D5 以降は `f111_sample`
由来で trade plan を作成可能。

加えて AFTER-R1 MVP に **f111_input_source taxonomy + 自動推定 + 4 output 反映**
を実装、CLI に `--f111-input-source` 追加。

完全 `f111_real_batch` (= F111 staging real candidates) 化は次 wave 持ち越し。

## §2 F111 ↔ F062 chain 実証

### §2.1 F111 runner --sample 動作確認

```bash
.venv/bin/python -m scripts.jobs.run_f111_research_advisory_preview \
  --sample --max-candidates 5 --dry-run \
  --output-json /tmp/fire_f111_pre/f111_real_sample_output.json \
  --summary-json /tmp/fire_f111_pre/f111_summary.json
```

結果:
- input=6 candidates / dry_run=True
- safety: no LINE send / no order / no broker / no rakuten / no Computer Use / no DB write
- done: candidate_count=5 / with_advisory=4 / missing_advisory=1 /
  auto_order_allowed_true_count=0 / manual_review_required_count=5

→ F111 output = 32 fields / row、F111_SAMPLE_OUTPUT_MARKERS 8 keys 含む。

### §2.2 F111 → F062 直接渡し (= adapter なし)

```bash
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/fire_f111_pre/f111_real_sample_output.json \
  --output-json /tmp/fire_f111_pre/f062_output_from_f111_sample.json \
  --freshness-report-json /tmp/fire_d4_prep/data_r3_freshness_2026-05-19.json \
  --require-freshness-ok --dry-run
```

結果:
- freshness gate: freshness OK ✓
- input=5 / selected=4 / chunks=1 / forbidden=0 / safety_footer=True ✓
- auto_order_allowed_true_count=0 / manual_review_required_count=5 /
  line_send_count=0 ✓

→ **F111 output が F062 input 互換** ✓ (= adapter 不要)

### §2.3 F062 → AFTER-R1 MVP 全 chain

```bash
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all --base-date 2026-05-20 \
  --f062-preview-json /tmp/fire_f111_pre/f062_output_from_f111_sample.json \
  ...
```

結果:
- artifact_source=**f062_preview** ✓
- f062_raw_kind=**f062_actual_dict** ✓
- **f111_input_source=`f111_sample`** ✓ (= W60-F111-pre 新規追加)
- ranking_size=4 / 4 file × (JSON+md) = 8 file 生成

Top candidates (= F111 SAMPLE_CANDIDATES 由来):
- #1 🟡 1234 サンプル情報通信A score=136.0 conf=0.82
- #2 🟡 7203 サンプル輸送機D score=129.0 conf=0.78

## §3 AFTER-R1 MVP 実装

### §3.1 f111_input_source taxonomy

| 値 | 由来 | caveat 強度 |
|---|---|---|
| `manual_seed` | 藤原さん手作り synth (= D3/D4) | 強 |
| `f111_sample` | F111 runner `--sample` | 中 |
| `f111_real_batch` | F111 runner `--candidate-json` (= staging) | 最弱 |
| `unknown` | 推定不能 | NO-GO + HQ |

### §3.2 自動推定 logic (`infer_f111_input_source`)

5 step priority:
1. explicit 指定 (= `--f111-input-source`) が VALID → 最優先
2. `row[0].f111_input_source` field が VALID → 採用
3. row[0] が `F111_SAMPLE_OUTPUT_MARKERS` を **3 件以上**含む → `f111_sample`
4. それ以外 (= minimum field set のみ) → `manual_seed`
5. rows 空 → `unknown`

### §3.3 F111_SAMPLE_OUTPUT_MARKERS (= 8 keys)

```python
F111_SAMPLE_OUTPUT_MARKERS = (
    "research_advisory_version", "research_source_version",
    "research_rule_version", "research_final_score",
    "research_post_cap_rank", "research_rank_label",
    "f119_expected_h20_return", "f119_expected_h20_win_rate",
)
```

### §3.4 4 output 反映

paper_live_ledger / good_candidate_ranking / pattern_candidate_report /
morning_line_material **root + summary** (ledger/ranking/pattern のみ) に
`f111_input_source` 反映。

### §3.5 CLI 拡張

`run_f286_after_r1_night_batch.py`:
- `--f111-input-source {manual_seed, f111_sample, f111_real_batch, unknown, auto}`
  追加、default=auto
- stdout に `f111_input_source=<val>` 表示
- summary JSON に `f111_input_source` expose
- CLI_VERSION 1.2.0 → **1.3.0**

### §3.6 MVP_LOGIC_VERSION

1.1.0 → **1.2.0** (= W60-F111-pre f111_input_source 追加)

## §4 作成 / 更新 file 一覧

### §4.1 新規 (4 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 03_design/F111_real_artifact_contract_2026-05-14.md | F111 contract v1.0 / 8 セクション |
| 04_daily/template_d5_real_artifact_prep.md | D5 manual command sequence (= 5 step + 8 invariants) |
| 02_todo/FIRE_CODEX_R1_WAVE60_F111_PRE_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_F111_PRE_results.md | 本 wave results (= 本 doc) |

### §4.2 更新 (3 件)

| path | 変更 |
|---|---|
| scripts/jobs/_after_r1_mvp.py | f111_input_source taxonomy + 推定 + 4 output 反映 / MVP_LOGIC_VERSION 1.1.0→1.2.0 |
| scripts/jobs/run_f286_after_r1_night_batch.py | --f111-input-source CLI 引数追加 / stdout + summary 反映 / CLI_VERSION 1.2.0→1.3.0 |
| tests/scripts/jobs/test_run_f286_after_r1_night_batch.py | TestW60F111PreF111InputSource 12 件追加 / version assertions update |

### §4.3 F062 / F111 / F282 本体不触 ✓

F062 / F111 / F282 / DATA-R3 / readiness / Ops / wrapper 既存 runner は全て不触。
W60-F111-pre は **AFTER-R1 MVP 側拡張 + docs** のみ。

## §5 tests 結果

| wave | collected |
|---|---|
| W60-launchd-pre | 4617 |
| **W60-F111-pre** | **4629** (= +12 件) |

全 **4629 PASS** ✓

### 新規 TestW60F111PreF111InputSource 12 件

1. taxonomy 確認 (= 4 値)
2. F111_SAMPLE_OUTPUT_MARKERS 定数 (= 8 keys)
3. F111 sample dict → f111_sample 推定
4. minimum field set → manual_seed 推定
5. rows 空 → unknown 推定
6. explicit override (= manual_seed で f111_sample を上書き)
7. explicit f111_real_batch 受理
8. invalid explicit → infer fallback
9. row[0].f111_input_source field 受理 (= f111_real_batch)
10. 4 outputs 全て (= ledger/ranking/pattern/material) で反映
11. CLI default auto で f111_sample 自動推定
12. CLI explicit --f111-input-source f111_real_batch で override

## §6 Codex 4 lane stdin audit 結果

| Lane | 観点 | 起動 | 完全 reply |
|---|---|---|---|
| A | F111 input source inference logic | ✓ exit 0 | 途中停止 |
| B | F111 ↔ F062 handoff path | ✓ exit 0 | 途中停止 |
| C | F111 contract + D5 template docs | ✓ exit 0 | 途中停止 |
| D | safety + tests audit | ✓ exit 0 | 途中停止 |

### §6.1 self-audit 補完

完全 reply 0/4 だが、起動 4/4 成功 + permission denied 再発 0:
- **Lane A self-audit**: infer_f111_input_source の 5 step priority order
  (= explicit valid → row field → marker ≥3 → manual_seed → unknown) 確認 ✓
- **Lane B self-audit**: F111 output → F062 直接渡し (= §2.2 smoke で実証) ✓
- **Lane C self-audit**: contract docs 8 セクション + D5 template 5 step 完全 ✓
- **Lane D self-audit**: 12 件 regression 全 PASS / MVP_SAFETY_FLAGS 13 keys
  全 False / 新規 import 0 / CLI_VERSION 1.3.0 / MVP_LOGIC_VERSION 1.2.0 ✓

### §6.2 並列実行結果の傾向

W60-launchd-pre (4 lane no sleep): 4/4 完全 reply ✓
W60-F111-pre (4 lane no sleep): 0/4 完全 reply、途中停止

→ 4 lane 並列でも **完全 reply 取得が安定しない** ことが確認できた。
W60-launchd-pre は **4 lane 中 prompt が短く log read が浅い** ため安定だった可能性。
本 wave は **prompt が長く file read が複数** で各 lane の所要時間が長くなり、
途中停止に至った可能性。

**改善案**: 各 prompt を短く、file read を 1 つに絞る (= 次 wave で試行)。

## §7 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 | staging 1 (= DATA-R3 dry-run SELECT のみ) / production-develop 0 |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体参照 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| 実発注 / 楽天 / iSPEED / Computer Use | 0 |
| 自動発注 / brokerage | 0 |
| sudo / git push / rm -rf / --no-verify / workflow | 0 |
| TODO Excel 更新 | 0 |
| F111 / F062 / F282 / DATA-R3 / readiness / Ops / wrapper 本体 変更 | 0 |
| 新規 import (= sqlite3/linebot/requests 等) | 0 |
| MVP_SAFETY_FLAGS 13 keys 全 False | 維持 ✓ |
| auto_order_allowed=False / manual_review_required=True 全 outputs 維持 | ✓ |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4617 → 4629 (= +12) |

## §8 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **0/4 = 0%** (= 起動 4/4 だが 完全 reply 0、self-audit 補完) |
| 短縮率 | 中 (= self-audit で 4 観点カバー) |
| 採用率 | 100% (= 設計通り実装、self-audit で 4 観点全 confirm) |
| 差戻率 | 0 |
| Integrator 負荷 | 中-高 (= F111 chain smoke + AFTER-R1 MVP 拡張 + 4 output 反映 + CLI + 12 regression + docs 2 件 + Codex 4 lane) |
| 安全事故 | **0** ✓ |

### Codex 稼働率 0/4 = 0% の説明 + 強い技術的理由

- 4 lane 起動成功率: 4/4 = 100%、permission denied 再発 0
- 完全 reply 取得率: 0/4 = 0%、全 lane 途中停止
- 推定原因: 4 lane 同時 + 各 prompt 長文 + 複数 file read で OpenAI API session
  timeout または process kill

**改善案 (= 次 wave 候補)**:
1. 各 lane の prompt を短く、focus を 1 観点に絞る
2. file read を 1 つに絞る (= 30 lines 程度)
3. 並列起動間に 5-10 秒 sleep を挟む (= W60-integration で試行済、効果限定的)
4. または直列で 4 lane 順次起動 (= 時間倍増だが完全 reply 取得高確率)

本 wave では self-audit が十分にカバー (= 4 観点全 self-confirm)。

## §9 残課題 / 次 Wave

### §9.1 W60-F111-pre で開ける道

- D5 (= 2026-05-20 水) で `f111_input_source = f111_sample` 達成
- pilot_use = `eligible_with_minor_caveat` (= D3/D4 から caveat 強度減)

### §9.2 並行 wave

- **W60-pilot-D5 (= 5/20 水)**: F111 --sample chain で f111_sample 達成
- **W60-pilot-W1 (= 5/21 以降)**: D1-D5 集約 + Codex 4 lane (= 改善 prompt 短文化)
- **F111-real-batch (= 別 wave)**: F111 `--candidate-json` に staging real
  candidates 渡す path (= f111_real_batch 化、最終目標)
- **W60-launchd-real**: 本番 launchd 配置 (= Wave 41 / 45 進捗後)

### §9.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §9.4 D1-D5 path artifact 推移 (= W60-F111-pre 後 期待)

| Day | 日付 | 曜日 | artifact_source | f111_input_source | 完了 |
|---|---|---|---|---|---|
| D1 | 2026-05-14 | 木 | synthetic_fixture | (= MVP 直渡し) | ✓ HOLD |
| D2 | 2026-05-15 | 金 | synthetic_fixture | (= MVP 直渡し) | ✓ HOLD |
| D3 | 2026-05-18 | 月 | f062_preview | manual_seed | ✓ GO + 強 caveat |
| D4 | 2026-05-19 | 火 | f062_preview | manual_seed | ✓ GO + 強 caveat |
| **D5** | **2026-05-20** | **水** | **f062_preview** | **f111_sample (期待)** | **GO + 中 caveat ★** |
| W1 集約 | 2026-05-21 | 木 | - | - | 予定 |

## §10 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_F111_PRE_plan]]
- [[../03_design/F111_real_artifact_contract_2026-05-14|F111 contract v1.0]]
- [[../04_daily/template_d5_real_artifact_prep|D5 manual sequence]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D4_results|D4 results]]
- [[FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_results|W60-launchd-pre results]]
