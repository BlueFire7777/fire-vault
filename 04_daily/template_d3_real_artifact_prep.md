---
template: d3-real-artifact-prep
version: 1.0
date: 2026-05-18 (D3 想定)
owner: BlueFire7777 (Fujiwara)
purpose: D3 で artifact_source=f062_preview を狙うための朝 manual command sequence
related:
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_results.md
  - 04_daily/template_manual_live_pilot_trade_plan.md
---

# D3 Real Artifact Preparation — Manual Command Sequence

## ⚠ 目的

D3 (= 2026-05-18 月) 以降の朝、藤原さんが 1 つの sequence で:
1. **F111 advisory preview** を取得 / 用意
2. **F062 LINE preview runner** を no-send / dry-run で実行 → F062 actual output 生成
3. **DATA-R3 freshness OK** を確認
4. **AFTER-R1 MVP** を実行 → reports/after_r1/ に **artifact_source=f062_preview** の
   4 成果物生成
5. **trade plan** を作成 → 藤原さんが判断

を完遂できる手順を docs 化。launchd / cron 未配置でも手動で実行可能。

**LINE 送信 0 / DB write 0 / token / API / launchctl / cron / 楽天 / iSPEED / 自動発注 全 0**。

---

## §1 前提条件 (= 朝の確認)

- 体調 / 本業都合で 寄付き〜15:10 通し監視可能
- 大型イベント (FOMC / 雇用統計 / 重要 IR / 決算跨ぎ) なし
- 当日朝 8:30 〜 9:00 にこの sequence を回す時間がある (= 10-15 分想定)

## §2 Step 0 — 環境変数 + path 確認

```bash
cd ~/fire
export DATE=$(date +%Y-%m-%d)        # 例: 2026-05-18
echo "DATE=$DATE"

# 出力先 dir 確保 (= /tmp 配下のため安全)
mkdir -p /tmp/fire_d3_prep
```

確認事項:
- `cd ~/fire` で repo root に移動済
- `.venv` 存在 (= `.venv/bin/python` 動作)
- DATE 変数 が当日 (= 営業日)

## §3 Step 1 — F111 advisory preview を用意

### §3.1 実 F111 朝 batch が稼働している場合 (= 将来)

W44-pre / Wave 41 進捗で F111 朝 batch が launchd 自動起動するようになれば、
`/tmp/f111_preview_$DATE.json` 等に自動出力される。本 step は skip 可。

### §3.2 F111 batch 未稼働の場合 (= D3 想定、藤原さん手作り)

藤原さん自身が当日の TDnet 開示 / 日経 / セクター状況を見て、**3-10 銘柄** の
advisory rows を作成。形式:

```json
[
  {
    "code": "<4 桁 ticker>",
    "name": "<銘柄名>",
    "research_advisory_label": "<boost / boost_with_caution / boost_with_avoid / caution / neutral / avoid / suppress>",
    "expected_h20": 0.0-1.0,
    "post_cap_rank": <1-N>,
    "manual_review_required": true,
    "auto_order_allowed": false,
    "research_advisory_text": "<判断理由>",
    "reason_tags": ["<根拠 1>", "<根拠 2>", "..."]
  },
  ...
]
```

保存先: `/tmp/fire_d3_prep/f111_preview_$DATE.json`

参考雛形 (= W60-launchd-pre smoke で使用):
- `/tmp/fire_w60_launchd_pre/f111_synth_preview.json` (= 3 銘柄、6920/4063/7203)

**注意**: 藤原さんが手作りした場合 artifact_source は **f062_preview** にはなるが、
**F062 朝 batch 由来ではない**ことを trade plan に明記。実 trade 判断は慎重に。

## §4 Step 2 — DATA-R3 freshness 用意

### §4.1 実 DATA-R3 batch が稼働している場合 (= 将来)

Wave 41 進捗で DATA-R3 dry-run batch が launchd 自動起動するようになれば、
`/tmp/f286_data_r3_freshness.json` に自動出力される。

### §4.2 DATA-R3 batch 未稼働の場合 (= D3 想定)

選択肢 A: **DATA-R3 runner を手動で dry-run 実行** (= staging DB read のみ、write 0)

```bash
.venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh \
  --dry-run --execute-dry-run-subprocesses \
  --base-date $DATE \
  --freshness-report-json /tmp/fire_d3_prep/data_r3_freshness_$DATE.json \
  --include-placeholder-skipped
```

選択肢 B (= **形式検証専用、実 trade では禁止**): **synthetic freshness**

⚠ **HIGH 警告 (W60-launchd-pre Codex Lane B 指摘)**:
synthetic freshness は **DATA-R3 OK を偽装** できるため、`--require-freshness-ok`
を **構造的に bypass** してしまう。実 freshness 未確認のまま F062 / AFTER-R1 が GO
経路に進む危険がある。

**運用ルール**:
- 実 trade では **選択肢 A (= 実 DATA-R3 dry-run runner) のみ許可**
- synthetic freshness を使った場合、その日の trade_plan は **必ず HOLD 扱い**
  (= synthetic_fixture の DATA-R3 版、artifact_source 由来とは別軸の HOLD 理由)
- DATA-R3 朝 batch (= Wave 41 進捗) が稼働するまでは synthetic 経路は **形式検証のみ**

```bash
# ⚠ 形式検証専用、実 trade では使わない
python3 -c "
import json
from pathlib import Path
Path('/tmp/fire_d3_prep/data_r3_freshness_$DATE.json').write_text(json.dumps({
    'schema_version': '1.0',
    'verdict': 'OK',  # 形式検証のみ、実 trade では選択肢 A 必須
    'aggregate_exit_code': 0,
    'freshness_report_id': 'data_r3_freshness_$DATE',
    'base_date': '$DATE',
    'source': 'SYNTHETIC_FORMAT_VERIFICATION_ONLY',  # 偽装抑止 marker
}, ensure_ascii=False), encoding='utf-8')
print('⚠ DATA-R3 synthetic ready (= 形式検証専用、実 trade では HOLD 必須)')
"
```

**verdict = OK が必須** (= 実 trade では選択肢 A の runner 出力で確認)。
FAILED / MISSING / STALE / unknown なら D3 は **NO-GO**。
**synthetic で OK にした場合 → trade_plan は HOLD (= 実 trade 不可)**。

## §5 Step 3 — F062 LINE preview runner (= no-send dry-run)

```bash
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/fire_d3_prep/f111_preview_$DATE.json \
  --output-json /tmp/fire_d3_prep/f062_actual_output_$DATE.json \
  --output-summary-json /tmp/fire_d3_prep/f062_summary_$DATE.json \
  --freshness-report-json /tmp/fire_d3_prep/data_r3_freshness_$DATE.json \
  --require-freshness-ok \
  --dry-run
```

確認事項:
- 出力末尾に `line_send_count=0` ✓
- `forbidden=0` ✓
- `safety_footer=True` ✓
- `auto_order_allowed_true_count=0` ✓
- `manual_review_required_count=<n>` (= input row 数と一致)
- exit code 0

**絶対禁止**: `--hq-approved-send` / `--message-mode production` (= LINE 送信 trigger)

F062 output (= dict with chunks / selected_count / message_chunk_count /
forbidden_phrase_count / safety_footer_present / selected_rows / ...) が生成される。
これが **F062 actual format** で、AFTER-R1 MVP が `artifact_source=f062_preview` と
判定する。

## §6 Step 4 — AFTER-R1 MVP (= 4 成果物生成)

```bash
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all \
  --base-date $DATE \
  --f062-preview-json /tmp/fire_d3_prep/f062_actual_output_$DATE.json \
  --f062-preview-summary-json /tmp/fire_d3_prep/f062_summary_$DATE.json \
  --data-r3-freshness-json /tmp/fire_d3_prep/data_r3_freshness_$DATE.json \
  --output-dir /Users/bluefire/fire/reports/after_r1
```

確認事項:
- stdout に **`artifact_source=f062_preview`** ✓
- `f062_raw_kind=f062_actual_dict` ✓
- `ranking_size=<n>` (= F062 selected_count と概ね一致)
- exit code 0
- 4 file × (JSON + Markdown) = 8 file 生成 in `reports/after_r1/`

## §7 Step 5 — artifact_source 判定 + Pilot GO check

⚠ **本 step は exit code で hard check** (= W60-launchd-pre Codex Lane C MEDIUM 修正):
artifact_source / forbidden_check / auto_order_allowed / manual_review_required が
期待通りでない場合 **exit 1** とし、本 sequence を中断 + HQ 確認。

```bash
.venv/bin/python -c "
import json
import sys
from pathlib import Path
d = Path('reports/after_r1')
material = json.loads((d / f'morning_line_material_$DATE.json').read_text(encoding='utf-8'))
ledger = json.loads((d / f'paper_live_ledger_$DATE.json').read_text(encoding='utf-8'))

# 表示 (= 藤原さん視認用)
print(f'artifact_source: {material[\"artifact_source\"]}')
print(f'f062_raw_kind: {material[\"f062_raw_kind\"]}')
print(f'freshness_verdict: {material[\"freshness_verdict\"]}')
print(f'forbidden_check.passed: {material[\"forbidden_phrases_check\"][\"passed\"]}')
print(f'auto_order_allowed: {material[\"auto_order_allowed\"]}')
print(f'manual_review_required: {material[\"manual_review_required\"]}')
print(f'all_auto_order_disallowed: {ledger[\"summary\"][\"all_auto_order_disallowed\"]}')
print(f'all_manual_review_required: {ledger[\"summary\"][\"all_manual_review_required\"]}')
print('--- Top candidates ---')
for c in material['top_candidates']:
    print(f'  #{c[\"rank\"]} {c[\"label_emoji\"]} {c[\"label\"]} {c[\"ticker\"]} {c[\"name\"]} score={c[\"score\"]} conf={c[\"confidence\"]:.2f}')

# hard check (= 構造的 invariant + GO 必須条件)
failures = []
if material['auto_order_allowed'] is not False:
    failures.append('material.auto_order_allowed must be False')
if material['manual_review_required'] is not True:
    failures.append('material.manual_review_required must be True')
if not material['forbidden_phrases_check']['passed']:
    failures.append('material.forbidden_phrases_check.passed must be True')
if not ledger['summary']['all_auto_order_disallowed']:
    failures.append('ledger.summary.all_auto_order_disallowed must be True')
if not ledger['summary']['all_manual_review_required']:
    failures.append('ledger.summary.all_manual_review_required must be True')
# GO 必須条件: f062_preview 由来 + DATA-R3 OK
if material['artifact_source'] != 'f062_preview':
    failures.append(f'GO requires artifact_source=f062_preview, got {material[\"artifact_source\"]} (= HOLD)')
if material['freshness_verdict'] != 'OK':
    failures.append(f'GO requires freshness OK, got {material[\"freshness_verdict\"]} (= NO-GO)')

if failures:
    print()
    print('⚠ Pilot GO check failed:')
    for f in failures:
        print(f'  - {f}')
    sys.exit(1)
print()
print('✓ Pilot GO check passed (= f062_preview + freshness OK + safety invariants)')
"
```

### §7.1 Pilot 判定

| artifact_source | pilot_use | 判定 |
|---|---|---|
| `f062_preview` | `eligible` | **GO** (= §3.1 全 PASS 確認後、実 trade 可) |
| `synthetic_fixture` | `skip_recommended` | **HOLD** (= 運用フロー検証のみ、実 trade 非推奨) |
| `unknown` | `no_go` | **NO-GO** (= 実 trade 禁止 + HQ 確認) |

§3 で F111 を **藤原さんが手作り**した場合、F062 runner 経由で
`artifact_source=f062_preview` とは判定されるが、**実 F062 朝 batch 由来ではない** ため
判断は GO/HOLD 中間。trade plan §3.0 に明記。

## §8 Step 6 — trade plan 作成

template から複製:
```bash
cp ~/fire-vault/04_daily/template_manual_live_pilot_trade_plan.md \
   ~/fire-vault/04_daily/${DATE}_manual_live_pilot_trade_plan.md
```

§3 / §3.0 / §3.1 / §4 / §5 を §6 の結果で埋める。
final decision は §7.1 の判定に従う。

## §9 Step 7 — review template 作成

template から複製:
```bash
cp ~/fire-vault/04_daily/template_manual_live_pilot_review.md \
   ~/fire-vault/04_daily/${DATE}_manual_live_pilot_review.md
```

15:30 以降に記入。

## §10 Step 8 — iSPEED 手動発注 (= enter 選択時のみ)

- 楽天 Web / iSPEED で **藤原さん本人が手動発注**
- FIRE / claude code は **何もしない**
- 信用デイトレ は **15:10 までに必ず手動 close**
- 持ち越し禁止

## §11 全 step の安全境界

| 項目 | 結果 |
|---|---|
| DB write | 0 (= F062/DATA-R3 dry-run、AFTER-R1 file write のみ) |
| DB sqlite 接続 | DATA-R3 dry-run で staging 1 接続のみ、production 0 |
| LINE 送信 | 0 (= F062 --dry-run、--hq-approved-send 不渡し) |
| token / API | 0 |
| launchctl / plist / cron | 0 (= 手動 command のみ) |
| 自動発注 | 0 |
| 楽天 / iSPEED | 0 (= 藤原さん手動のみ) |
| Computer Use | 0 |
| F282 plist | 不変 |
| 3 DB write | 0 |
| **sudo** | **0 (= 本 sequence で sudo 不要、検知時即停止 + HQ)** |
| **git push** | **0 (= 本 sequence で git 操作不要、検知時即停止 + HQ)** |
| **rm -rf** | **0 (= 本 sequence で破壊操作不要、検知時即停止 + HQ)** |
| **--no-verify** | **0 (= pre-commit/Codex review 回避禁止)** |
| **workflow / .github/** | **0 (= 本 sequence で .github 変更不要)** |

## §12 緊急停止条件 (= 本 sequence 中)

以下を検知 → 即停止 + HQ 確認:
- F062 runner exit code != 0
- F062 output に forbidden_phrase 検出
- F062 output `line_send_count > 0` (= 構造上 0 のはず)
- DATA-R3 freshness verdict != OK
- AFTER-R1 が unknown / NO-GO 判定
- artifact_source / forbidden_check / auto_order_allowed / manual_review_required
  のいずれかが期待と異なる

## §13 関連リンク

- operation plan: [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- trade plan template: [[template_manual_live_pilot_trade_plan]]
- review template: [[template_manual_live_pilot_review]]
- W60-launchd-pre results: [[../02_todo/FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_results|W60-launchd-pre results]]
- W60-integration results: [[../02_todo/FIRE_CODEX_R1_WAVE60_INTEGRATION_results|W60-integration results]]
- W60-pilot-D1 results: [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_results|W60-pilot-D1 results]]
- W60-pilot-D2 results: [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_results|W60-pilot-D2 results]]
