---
id: F111-real-artifact-contract
phase: 本番 v0 中核 / Wave 60-F111-pre / F111 Real Artifact Contract
priority: 高
status: design v1.0
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_F111_PRE_results.md
  - 04_daily/template_d5_real_artifact_prep.md
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
---

# F111 Real Artifact Contract v1.0

## §1 目的

F111 朝 batch が生成する **real artifact の output 形式** を文書化し、
F062 LINE preview runner / AFTER-R1 MVP との **handoff path** を確定する。

W60-F111-pre で **F111 runner `--sample` モード** が F062 → AFTER-R1 chain
に直接渡せることが実証された (= adapter 不要、artifact_source=f062_preview /
f111_input_source=f111_sample 自動推定 ✓)。

## §2 F111 runner 概要

| 項目 | 値 |
|---|---|
| script | `scripts/jobs/run_f111_research_advisory_preview.py` |
| 名称 | F111-R2: ResearchAdvisory 付き F111 candidate runner |
| safety | LINE 送信 / order / broker / rakuten / Computer Use 未接続 / DB write なし |
| input | `--sample` / `--candidate-json` / `--candidate-csv` |
| output | `--output-json` / `--output-csv` / `--output-text` / `--summary-json` |
| default mode | `--dry-run` (= True) |

## §3 F111 output schema (= row 単位、32 fields)

```json
{
  "code": "1234",
  "name": "サンプル情報通信A",
  "side": "long",
  "score": 0.82,
  "rank": 1,
  "reason": "F119 normal × 情報通信 × 5月 boost",

  "research_advisory_version": "F111-R1-v1",
  "research_source_version": "r2f4_baseline_v1",
  "research_rule_version": "r2g3_recommended_v2",
  "research_base_date": "2026-05-08",
  "research_final_score": 0.842,
  "research_post_cap_rank": 18,
  "research_rank_label": "A2",
  "research_interpretation": "use_signal_normal",
  "research_interpretation_detail": "normal_default",
  "research_advisory_label": "boost_with_caution",

  "market_regime": "uptrend",
  "sector_flow": "moderate_inflow",
  "sector_17": "情報通信",
  "month_of_year": 5,
  "top_bucket": "top30",

  "f119_boost_flags": ["..."],
  "f119_avoid_flags": [],
  "f119_caution_flags": ["..."],
  "f119_expected_h20_return": 0.1142,
  "f119_expected_h20_win_rate": 0.88,
  "f119_expected_h5_return": 0.04,

  "position_sizing_note": "通常 (R-05-02 ルール準拠)",
  "advisory_comment": "F119上は強い (...). 短期売買判断には使わない。",

  "manual_review_required": true,
  "auto_order_allowed": false
}
```

### §3.1 F062 が読む必須 field (= subset)

F062 preview runner は以下の field を読む:
- `code` / `name`: ticker / 銘柄名
- `research_advisory_label`: label (= boost / boost_with_caution / ...)
- `expected_h20`: confidence (= F111 では `f119_expected_h20_return` または
  `research_final_score` から推定可、F111 sample で実装)
- `post_cap_rank`: rank (= F111 では `research_post_cap_rank`)
- `manual_review_required` / `auto_order_allowed`: safety flag
- `research_advisory_text` または `reason`: 判断理由
- `reason_tags`: 根拠 tag list

**F111 sample output は直接 F062 に渡せる** (= adapter 不要、W60-F111-pre 実証 ✓)。

## §4 source_type 分類 (= W60-F111-pre 導入)

AFTER-R1 MVP の `f111_input_source` taxonomy:

| 値 | 由来 | 用途 |
|---|---|---|
| `manual_seed` | 藤原さん手作り synth (= D3/D4 で使用、3 銘柄 sample) | minimal field set、pilot 形式検証用 |
| `f111_sample` | F111 runner `--sample` 由来 (= 組み込み SAMPLE_CANDIDATES 5 銘柄) | F111 own ロジック、組み込み test data |
| `f111_real_batch` | F111 runner `--candidate-json` (= staging real candidates) | **本番 朝 batch、将来の正規 path** |
| `unknown` | 推定不能 / 不在 | NO-GO + HQ 確認 |

### §4.1 自動推定 logic (= MVP `infer_f111_input_source`)

```python
1. explicit 指定 (= --f111-input-source) が VALID なら最優先
2. row[0].f111_input_source field が VALID ならそれを採用
3. row[0] が F111_SAMPLE_OUTPUT_MARKERS を 3 件以上含む → f111_sample
   (= research_advisory_version / research_source_version /
   research_rule_version / research_final_score / research_post_cap_rank /
   research_rank_label / f119_expected_h20_return / f119_expected_h20_win_rate)
4. それ以外 (= minimum field set のみ) → manual_seed
5. rows 空 → unknown
```

F111_SAMPLE_OUTPUT_MARKERS = 8 keys、3 件以上で確定的に F111 sample 判定可能。

## §5 F062 handoff path (= W60-F111-pre 確立)

### §5.1 chain (= adapter なし)

```
F111 runner (--sample または --candidate-json)
  → F111 output JSON (32 fields/row)
  → F062 LINE preview runner (--preview-json で読む)
  → F062 actual output (= dict with chunks/selected_count/...)
  → AFTER-R1 MVP (--f062-preview-json)
  → 4 種類成果物 (= artifact_source=f062_preview, f111_input_source=auto 推定)
```

### §5.2 D5 用 manual command sequence

詳細: [[../04_daily/template_d5_real_artifact_prep|template_d5_real_artifact_prep.md]]

```bash
cd ~/fire && export DATE=$(date +%Y-%m-%d)
mkdir -p /tmp/fire_d5_prep

# Step 1 - F111 runner --sample (= F111-R2 本物化、組み込み SAMPLE_CANDIDATES)
.venv/bin/python -m scripts.jobs.run_f111_research_advisory_preview \
  --sample --max-candidates 10 --dry-run \
  --output-json /tmp/fire_d5_prep/f111_real_sample_$DATE.json \
  --summary-json /tmp/fire_d5_prep/f111_summary_$DATE.json

# Step 2 - DATA-R3 freshness (= 選択肢 A dry-run runner)
.venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh \
  --dry-run --execute-dry-run-subprocesses \
  --base-date $DATE \
  --freshness-report-json /tmp/fire_d5_prep/data_r3_freshness_$DATE.json \
  --include-placeholder-skipped

# Step 3 - F062 no-send LINE preview runner
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/fire_d5_prep/f111_real_sample_$DATE.json \
  --output-json /tmp/fire_d5_prep/f062_actual_output_$DATE.json \
  --output-summary-json /tmp/fire_d5_prep/f062_summary_$DATE.json \
  --freshness-report-json /tmp/fire_d5_prep/data_r3_freshness_$DATE.json \
  --require-freshness-ok --dry-run

# Step 4 - AFTER-R1 MVP
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all --base-date $DATE \
  --f062-preview-json /tmp/fire_d5_prep/f062_actual_output_$DATE.json \
  --f062-preview-summary-json /tmp/fire_d5_prep/f062_summary_$DATE.json \
  --data-r3-freshness-json /tmp/fire_d5_prep/data_r3_freshness_$DATE.json \
  --output-dir reports/after_r1
# 期待: artifact_source=f062_preview, f111_input_source=f111_sample

# Step 5 - 7+1 invariants hard check (= W60-F111-pre 拡張)
# 既存 7 invariants + f111_input_source ∈ {f111_sample, f111_real_batch}
```

### §5.3 hard check 拡張 (= 7 → 8 invariants)

| # | invariant | W60-launchd-pre | W60-F111-pre |
|---|---|---|---|
| 1 | artifact_source = `f062_preview` | ✓ | ✓ |
| 2 | f062_raw_kind = `f062_actual_dict` | ✓ | ✓ |
| 3 | DATA-R3 freshness verdict = `OK` | ✓ | ✓ |
| 4 | forbidden_check.passed = True | ✓ | ✓ |
| 5 | material.auto_order_allowed = False | ✓ | ✓ |
| 6 | material.manual_review_required = True | ✓ | ✓ |
| 7 | safety_flags 13 keys 全 False | ✓ | ✓ |
| **8** | **f111_input_source ∈ `{f111_sample, f111_real_batch}`** | - | **✓ 新規** |

invariants #8 fail (= manual_seed / unknown) → **HOLD/skip 推奨**。

## §6 pilot_use 判定 matrix (= D3 → D5 推移)

| f111_input_source | artifact_source | pilot_use | judgment |
|---|---|---|---|
| manual_seed | f062_preview | eligible_with_caveat | GO with strong caveat |
| **f111_sample** | **f062_preview** | **eligible_with_minor_caveat** | **GO with minor caveat** ← D5 |
| f111_real_batch | f062_preview | eligible | **GO** (= 真の本番 path) |
| any | synthetic_fixture | skip_recommended | HOLD |
| any | unknown | no_go | NO-GO |

### §6.1 caveat 強度

- `manual_seed`: 藤原さん手作り、研究 advisory も手作り = 強 caveat
- `f111_sample`: F111 runner 自身が研究 advisory 計算 = 中 caveat (= 銘柄は組み込み)
- `f111_real_batch`: staging から実 candidates = 最弱 caveat (= 残るのは staging
  data 自身の真実性)

D5 で `f111_sample` を達成 = D3/D4 から **大きな前進**。
完全 `f111_real_batch` 化は次 wave (= F111 朝 batch staging input + launchd 化)。

## §7 残課題 / 次 Wave

| Wave | 内容 |
|---|---|
| **W60-F111-pre (= 本 wave)** | F111 ↔ F062 contract docs + AFTER-R1 f111_input_source 識別 |
| W60-pilot-D5 (= 5/20 水) | F111 --sample で f111_input_source=f111_sample 達成、D1-D5 集約準備 |
| W60-pilot-W1 | D1-D5 集約 + Codex 4+ lane audit + pattern 分類結論 |
| F111-real-batch (= 別 wave) | staging → F111 --candidate-json で f111_real_batch 化 |
| W60-launchd-real | 本番 launchd 配置 (= F111 / F062 / morning-advisory) |
| W61-pre | price/return data + paper_pnl 連携 |

## §8 安全境界

- 本 doc は **read-only docs**、F111 / F062 / AFTER-R1 本体不触
- F111 runner は既存仕様 (= W60-F111-pre baseline で動作確認)
- AFTER-R1 MVP の f111_input_source 追加は **string field のみ**、
  新規 import / 新規 API call なし
- 全 sequence で LINE / DB write / token / API / launchctl / cron / 楽天 / iSPEED /
  Computer Use 0

---

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE60_F111_PRE_results|W60-F111-pre results]]
- [[../04_daily/template_d5_real_artifact_prep|D5 manual sequence]]
- [[../04_daily/template_d3_real_artifact_prep|D3 manual sequence (= 比較用)]]
- [[FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
