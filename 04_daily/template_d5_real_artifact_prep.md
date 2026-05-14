---
template: d5-real-artifact-prep
version: 1.0
date: 2026-05-20 (D5 想定)
owner: BlueFire7777 (Fujiwara)
purpose: D5 で f111_input_source=f111_sample を狙う朝 manual command sequence
related:
  - 03_design/F111_real_artifact_contract_2026-05-14.md
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/template_d3_real_artifact_prep.md (= 比較用)
  - 02_todo/FIRE_CODEX_R1_WAVE60_F111_PRE_results.md
---

# D5 Real Artifact Preparation — Manual Command Sequence (= f111_sample 化)

## ⚠ 目的

D5 (= 2026-05-20 水) 朝で **f111_input_source = `f111_sample`** を達成。
D3/D4 の `manual_seed` (= 藤原さん手作り 3 銘柄) から **F111 runner --sample 経由
(= F111 own ロジックで研究 advisory 計算)** へ進展。

D5 chain (= 5 step):
1. **F111 runner --sample** (= W60-F111-pre 確立)
2. **DATA-R3 freshness 選択肢 A**
3. **F062 LINE preview runner --dry-run --require-freshness-ok**
4. **AFTER-R1 MVP --mode mvp --task all**
5. **8 invariants hard check** (= W60-launchd-pre 7 + W60-F111-pre 追加 #8)

**LINE 送信 0 / DB write 0 / token / API / launchctl / cron / 楽天 / iSPEED / 自動発注 全 0**。

---

## §1 前提条件

D3 manual command sequence と同条件:
- 体調 / 本業都合で 寄付き〜15:10 通し監視可能
- 大型イベント なし
- 当日朝 8:30 〜 9:00 にこの sequence を回す時間がある (= 10-15 分想定)

## §2 Step 0 — 環境変数 + path 確認

```bash
cd ~/fire
export DATE=$(date +%Y-%m-%d)
echo "DATE=$DATE"
mkdir -p /tmp/fire_d5_prep
```

## §3 Step 1 — F111 runner --sample (= W60-F111-pre 確立、D3 からの進化)

```bash
.venv/bin/python -m scripts.jobs.run_f111_research_advisory_preview \
  --sample --max-candidates 10 --dry-run \
  --output-json /tmp/fire_d5_prep/f111_real_sample_$DATE.json \
  --summary-json /tmp/fire_d5_prep/f111_summary_$DATE.json
```

確認事項:
- 出力末尾に `candidate_count=N` (= N > 0、組み込み SAMPLE_CANDIDATES)
- `with_advisory=N` (= 全件 research_advisory 付き)
- `auto_order_allowed_true_count=0` ✓
- `manual_review_required_count=N` (= 全件 manual_review)
- 出力 file: `/tmp/fire_d5_prep/f111_real_sample_$DATE.json` (= 32 fields/row)

**絶対禁止**: `--no-include-missing-advisory` 以外の 危険 flag (= 想定外、F111
runner には DB write / LINE / token はそもそも未配線)

### §3.1 F111 sample marker 確認

F111 output に以下 8 marker が含まれること:
- research_advisory_version
- research_source_version
- research_rule_version
- research_final_score
- research_post_cap_rank
- research_rank_label
- f119_expected_h20_return
- f119_expected_h20_win_rate

→ AFTER-R1 が `f111_input_source = f111_sample` 自動推定の根拠。

## §4 Step 2 — DATA-R3 freshness (= 選択肢 A dry-run runner、D3 と同)

```bash
.venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh \
  --dry-run --execute-dry-run-subprocesses \
  --base-date $DATE \
  --freshness-report-json /tmp/fire_d5_prep/data_r3_freshness_$DATE.json \
  --include-placeholder-skipped
```

確認事項:
- verdict=OK ✓
- aggregate_exit_code=0
- 4 sub-runner (f100/f101/f111/f119) 全 ok
- db_row_writes=0 / cron_install=none

選択肢 B (synthetic freshness) は **形式検証専用、実 trade では禁止** (= W60-launchd-pre
Codex Lane B HIGH 修正後の運用ルール継続)。

## §5 Step 3 — F062 LINE preview runner --dry-run --require-freshness-ok

```bash
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/fire_d5_prep/f111_real_sample_$DATE.json \
  --output-json /tmp/fire_d5_prep/f062_actual_output_$DATE.json \
  --output-summary-json /tmp/fire_d5_prep/f062_summary_$DATE.json \
  --freshness-report-json /tmp/fire_d5_prep/data_r3_freshness_$DATE.json \
  --require-freshness-ok --dry-run
```

確認事項:
- freshness gate: freshness OK ✓
- input=N (= F111 candidate_count と一致)
- selected=K (= F062 がフィルタリング)
- chunks=1+ / forbidden=0 / line_send_count=0 ✓ / safety_footer=True ✓
- 出力 file: `/tmp/fire_d5_prep/f062_actual_output_$DATE.json` (= F062 actual dict format)

**絶対禁止**: `--hq-approved-send` / `--message-mode production`

## §6 Step 4 — AFTER-R1 MVP

```bash
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all --base-date $DATE \
  --f062-preview-json /tmp/fire_d5_prep/f062_actual_output_$DATE.json \
  --f062-preview-summary-json /tmp/fire_d5_prep/f062_summary_$DATE.json \
  --data-r3-freshness-json /tmp/fire_d5_prep/data_r3_freshness_$DATE.json \
  --output-dir reports/after_r1
```

期待:
- `artifact_source=f062_preview` ✓
- `f062_raw_kind=f062_actual_dict` ✓
- **`f111_input_source=f111_sample`** ✓ (= W60-F111-pre 新規追加)
- 4 file × (JSON + Markdown) = 8 file 生成

## §7 Step 5 — 8 invariants hard check (= W60-launchd-pre 7 + W60-F111-pre 追加)

```bash
.venv/bin/python -c "
import json, sys
from pathlib import Path
d = Path('reports/after_r1')
DATE = '$DATE'
material = json.loads((d / f'morning_line_material_{DATE}.json').read_text(encoding='utf-8'))
ledger = json.loads((d / f'paper_live_ledger_{DATE}.json').read_text(encoding='utf-8'))

sf = material['safety_flags']
sf_all_false = all(v is False for v in sf.values())

# 表示
print(f'1. artifact_source: {material[\"artifact_source\"]}  (expect f062_preview)')
print(f'2. f062_raw_kind: {material[\"f062_raw_kind\"]}  (expect f062_actual_dict)')
print(f'3. freshness_verdict: {material[\"freshness_verdict\"]}  (expect OK)')
print(f'4. forbidden_check.passed: {material[\"forbidden_phrases_check\"][\"passed\"]}  (expect True)')
print(f'5. auto_order_allowed: {material[\"auto_order_allowed\"]}  (expect False)')
print(f'6. manual_review_required: {material[\"manual_review_required\"]}  (expect True)')
print(f'7. safety_flags all False: {sf_all_false} ({len(sf)} keys)')
print(f'8. f111_input_source: {material[\"f111_input_source\"]}  (expect f111_sample or f111_real_batch)')

# hard check
failures = []
if material['artifact_source'] != 'f062_preview': failures.append('(1) artifact_source ≠ f062_preview')
if material['f062_raw_kind'] != 'f062_actual_dict': failures.append('(2) raw_kind ≠ f062_actual_dict')
if material['freshness_verdict'] != 'OK': failures.append('(3) freshness ≠ OK')
if not material['forbidden_phrases_check']['passed']: failures.append('(4) forbidden_check failed')
if material['auto_order_allowed'] is not False: failures.append('(5) auto_order_allowed ≠ False')
if material['manual_review_required'] is not True: failures.append('(6) manual_review_required ≠ True')
if not sf_all_false: failures.append('(7) safety_flags not all False')
if material['f111_input_source'] not in ('f111_sample', 'f111_real_batch'):
    failures.append(f'(8) f111_input_source not f111_sample/f111_real_batch (got {material[\"f111_input_source\"]})')

if failures:
    print()
    print('⚠ Pilot GO check FAILED:')
    for f in failures: print(f'  - {f}')
    sys.exit(1)
print()
print('✓ Pilot GO check PASSED — 8 invariants all satisfied')
"
```

## §8 Step 6 — Pilot judgment matrix

| f111_input_source | artifact_source | pilot_use | judgment | 実 trade 推奨? |
|---|---|---|---|---|
| `f111_sample` (D5 期待) | `f062_preview` | `eligible_with_minor_caveat` | **GO with minor caveat** | 自己責任で entry 可 |
| `manual_seed` (D3/D4) | `f062_preview` | `eligible_with_caveat` | GO with strong caveat | 慎重判断 |
| `f111_real_batch` (将来) | `f062_preview` | `eligible` | GO (= 真の本番) | 推奨 |
| any | `synthetic_fixture` | `skip_recommended` | HOLD | 非推奨 |
| any | `unknown` | `no_go` | NO-GO + HQ 確認 | 禁止 |

D5 の **目標は f111_input_source = `f111_sample`**。

## §9 Step 7 — trade plan / review template 複製

```bash
cp ~/fire-vault/04_daily/template_manual_live_pilot_trade_plan.md \
   ~/fire-vault/04_daily/${DATE}_manual_live_pilot_trade_plan.md
cp ~/fire-vault/04_daily/template_manual_live_pilot_review.md \
   ~/fire-vault/04_daily/${DATE}_manual_live_pilot_review.md
```

§3.0 で f111_input_source 欄を **`f111_sample`** に check (= D3/D4 の `manual_seed`
から進展)。

## §10 Step 8 — iSPEED 手動発注 (= enter 時のみ)

- 楽天 Web / iSPEED で 藤原さん本人が手動発注
- 信用デイトレ は 15:10 までに必ず手動 close
- 持ち越し禁止

## §11 全 step の安全境界

| 項目 | 結果 |
|---|---|
| DB write | 0 (= F111/F062/AFTER-R1 全 file write、DB write なし) |
| DB sqlite 接続 | staging 1 (= DATA-R3 dry-run SELECT のみ) / production-develop 0 |
| LINE 送信 | 0 (= F062 --dry-run、--hq-approved-send 不渡し) |
| token / API | 0 |
| launchctl / plist / cron | 0 |
| 自動発注 / 楽天 / iSPEED / Computer Use | 0 |
| F282 plist | 不変 |
| 3 DB | 不変 |
| sudo / git push / rm -rf / --no-verify / workflow / TODO Excel | 0 |

## §12 緊急停止条件

D3 と同じ + W60-F111-pre 追加:
- F111 runner exit code != 0 / candidate_count = 0
- F111 output に F111_SAMPLE_OUTPUT_MARKERS が不足 (= f111_sample 判定不能)
- F062 output `line_send_count > 0`
- 8 invariants の 1 件でも fail
- **f111_input_source = `unknown` → NO-GO + HQ 確認**

## §13 関連リンク

- F111 contract: [[../03_design/F111_real_artifact_contract_2026-05-14|F111 real artifact contract v1.0]]
- D3 template: [[template_d3_real_artifact_prep|D3 manual sequence (= 比較)]]
- operation plan: [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- trade plan template: [[template_manual_live_pilot_trade_plan]]
- review template: [[template_manual_live_pilot_review]]
- W60-F111-pre results: [[../02_todo/FIRE_CODEX_R1_WAVE60_F111_PRE_results|W60-F111-pre results]]
