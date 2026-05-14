---
id: FIRE-CODEX-R1-WAVE60-launchd-pre-results
phase: 本番 v0 中核 / Wave 60-launchd-pre / Real Artifact Generation Path Preparation
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_INTEGRATION_results.md
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/template_d3_real_artifact_prep.md
  - 04_daily/template_f111_synthetic_preview.json
---

# Wave 60-launchd-pre Results — Real Artifact Generation Path Preparation v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 F111 synth → F062 runner → AFTER-R1 MVP chain で
`artifact_source = f062_preview` 取得 path を実証 + 文書化 ✓**

D3 (= 2026-05-18 月) 以降に藤原さんが朝に 1 sequence で 4 step 実行することで、
**synthetic_fixture でなく f062_preview 判定の 4 成果物**を `reports/after_r1/` に
生成できる手順が確定。

Codex 4 lane parallel smoke 全成功 (= 4/4 完全 reply 取得、W60.5 安定 pattern 再現)、
**HIGH 1 + MEDIUM 2 + LOW 4 発見 → HIGH/MEDIUM 全 修正完了** ✓。

本番 launchd 配置 / load は本 wave 範囲外 (= Wave 41 / 45 へ持ち越し、
本 wave は手動 manual command path のみ整備)。

## §2 chain smoke 動作確認

### §2.1 3 step chain

```
F111 advisory preview (= 藤原さん手作り or 朝 batch、現状 synthetic)
   ↓ --preview-json
F062 LINE preview runner (= --dry-run --require-freshness-ok)
   ↓ --f062-preview-json
AFTER-R1 MVP (= --mode mvp --task all)
   ↓ --output-dir reports/after_r1
4 成果物 (= artifact_source=f062_preview)
```

### §2.2 smoke 結果 (= 2026-05-18 base_date)

| step | command | 結果 |
|---|---|---|
| F062 runner | `--preview-json f111_synth_preview.json --dry-run --require-freshness-ok` | input=3 / selected=2 / chunks=1 / forbidden=0 / line_send=0 |
| AFTER-R1 MVP | `--mode mvp --task all --f062-preview-json f062_actual_output.json` | artifact_source=**f062_preview** / f062_raw_kind=f062_actual_dict / ranking_size=2 |
| 4 成果物 | reports/after_r1/{paper_live_ledger,...,morning_line_material}_2026-05-18.{json,md} | 8 file 生成 ✓ |

Top candidates (= F111 synth から F062 経由で抽出された 2 銘柄):
- 6920 レーザーテック (= boost、確信度 0.78)
- 4063 信越化学 (= boost_with_caution、確信度 0.60)

### §2.3 F062 actual output keys (= 16 keys、AFTER-R1 が読む)

```
['action_mode', 'buyability_mode', 'card_mode', 'chunks', 'compact', 'dry_run',
 'forbidden_phrase_count', 'message_chunk_count', 'message_mode', 'metadata',
 'safety_footer_present', 'safety_notes', 'selected_count',
 'selected_label_counts', 'selected_rows', 'send_intent']
```

AFTER-R1 は `chunks` / `selected_count` / `message_chunk_count` /
`forbidden_phrase_count` / `safety_footer_present` の 5 marker で
**`f062_raw_kind = f062_actual_dict`** 判定。余分 field (= action_mode /
buyability_mode / metadata / send_intent 等) は無視。

## §3 作成 / 更新 file 一覧

### §3.1 新規 (3 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 04_daily/template_d3_real_artifact_prep.md | D3 朝 manual command sequence v1.0 / 13 セクション |
| 04_daily/template_f111_synthetic_preview.json | F111 advisory rows synthetic 雛形 (= 3 行) |
| 02_todo/FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_results.md | 本 wave results (= 本 doc) |

### §3.2 更新 (1 件)

| path | 変更内容 |
|---|---|
| 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md | §10.1 を D3 用 4 step manual command 化 (= F062 + AFTER-R1 連携) |

### §3.3 tests (= +7 件)

| path | 変更内容 |
|---|---|
| tests/scripts/jobs/test_run_f286_after_r1_night_batch.py | `TestW60LaunchdPreF062RealOutput` 5 件 / `TestW60LaunchdPreD3CommandSequenceDocs` 2 件 追加 |

### §3.4 FIRE 本体 (= 不触 ✓)

- F062 / DATA-R3 / AFTER-R1 runner 本体 全て不触
- pytest collected: 4610 → **4617** (= +7 件)

## §4 Codex 4 lane parallel smoke 結果

W60.5 で実証された **4 lane parallel + no sleep** pattern を再現 (= 安定動作)。

| Lane | 観点 | 結果 | finding | 修正 |
|---|---|---|---|---|
| A | F062 real preview path | exit 0 ✓ 完全 reply | LOW: 「forbidden=0」vs 「forbidden_phrase_count=0」 wording | docs 注記レベル |
| B | DATA-R3 freshness handoff | exit 0 ✓ 完全 reply | **HIGH**: 選択肢 B (= synthetic freshness) が DATA-R3 OK 偽装、--require-freshness-ok bypass 可能 | ✓ **本 wave 修正** |
| C | AFTER-R1 manual command | exit 0 ✓ 完全 reply | **MEDIUM**: Step 5 が `print` のみ、assert/exit nonzero がない | ✓ **本 wave 修正** |
| D | safety + tests audit | exit 0 ✓ 完全 reply | **MEDIUM**: sudo / git push 禁止 明示なし / LOW × 2 | ✓ **本 wave 修正 (MEDIUM)** |

**Codex 4/4 = 100% 完全 reply**、CRITICAL 0 / HIGH 1 / MEDIUM 2 / LOW 4。

### §4.1 修正実施

**Lane B HIGH 修正** (= template_d3_real_artifact_prep.md §4.2):
- 選択肢 B (synthetic freshness) を **「形式検証専用、実 trade では禁止」** に格下げ
- 「synthetic で OK にした場合 → trade_plan は HOLD 必須」を明示
- 偽装抑止 marker (`source: SYNTHETIC_FORMAT_VERIFICATION_ONLY`) を JSON に追記
- 「実 trade は選択肢 A (= 実 DATA-R3 dry-run runner) のみ許可」を明示

**Lane C MEDIUM 修正** (= template_d3_real_artifact_prep.md §7):
- Step 5 (= artifact_source 判定) に **hard check 化**
- 7 invariant を `failures` list で集積、1 件でも fail なら `sys.exit(1)`
  - `material.auto_order_allowed = False`
  - `material.manual_review_required = True`
  - `material.forbidden_phrases_check.passed = True`
  - `ledger.summary.all_auto_order_disallowed = True`
  - `ledger.summary.all_manual_review_required = True`
  - `material.artifact_source = "f062_preview"` (= GO 必須条件)
  - `material.freshness_verdict = "OK"` (= GO 必須条件)
- 通過時 `✓ Pilot GO check passed` 表示

**Lane D MEDIUM 修正** (= template_d3_real_artifact_prep.md §11):
- 安全境界 table に **sudo / git push / rm -rf / --no-verify / workflow** を明示追加
- 各「0 (= 本 sequence で 不要、検知時即停止 + HQ)」と明記

### §4.2 LOW 扱い

- Lane A LOW (= wording 「forbidden=0」): F062 runner 出力例の引用、docs 注記レベルで本 wave 未修正、次 wave 改善候補
- Lane D LOW × 2 (= tests コメント / metadata-absent 確認): 軽微、次 wave 改善候補

### §4.3 6 KPI への影響

Codex 稼働率 = 4/4 = **100%** ✓ (= W60.5 と同等)

## §5 tests 結果

| wave | collected |
|---|---|
| W60-integration | 4610 |
| W60-launchd-pre | **4617** (= +7 件) |

全 **4617 PASS** ✓

### 新規 W60-launchd-pre regression 7 件

- `TestW60LaunchdPreF062RealOutput` 5 件:
  - F062 runner 実 output (16 keys) → f062_preview 推定
  - 余分 field (action_mode / metadata / send_intent 等) 無視
  - F062 runner → AFTER-R1 end-to-end pipeline で 4 成果物全件 f062_preview
  - F062 runner output 経由でも auto_order=False / manual_review=True 維持
  - zero chunks edge case で AFTER-R1 が落ちない
- `TestW60LaunchdPreD3CommandSequenceDocs` 2 件:
  - template_d3_real_artifact_prep.md 存在 + 必須 step 文書化確認
  - template_f111_synthetic_preview.json 存在 + 必須 field 確認

## §6 D3 (= 2026-05-18 月) 朝 4 step (= 確定手順)

詳細: [[../04_daily/template_d3_real_artifact_prep|template_d3_real_artifact_prep.md]]

```bash
cd ~/fire && export DATE=$(date +%Y-%m-%d)
mkdir -p /tmp/fire_d3_prep

# Step 1 - F111 advisory preview を用意 (= 朝 batch 未稼働なら藤原さん手作り)
#   雛形: ~/fire-vault/04_daily/template_f111_synthetic_preview.json
#   出力: /tmp/fire_d3_prep/f111_preview_$DATE.json

# Step 2 - DATA-R3 freshness (= 選択肢 A: dry-run runner 必須)
.venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh \
  --dry-run --execute-dry-run-subprocesses \
  --base-date $DATE \
  --freshness-report-json /tmp/fire_d3_prep/data_r3_freshness_$DATE.json \
  --include-placeholder-skipped

# Step 3 - F062 no-send LINE preview runner
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/fire_d3_prep/f111_preview_$DATE.json \
  --output-json /tmp/fire_d3_prep/f062_actual_output_$DATE.json \
  --output-summary-json /tmp/fire_d3_prep/f062_summary_$DATE.json \
  --freshness-report-json /tmp/fire_d3_prep/data_r3_freshness_$DATE.json \
  --require-freshness-ok --dry-run

# Step 4 - AFTER-R1 MVP
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all --base-date $DATE \
  --f062-preview-json /tmp/fire_d3_prep/f062_actual_output_$DATE.json \
  --f062-preview-summary-json /tmp/fire_d3_prep/f062_summary_$DATE.json \
  --data-r3-freshness-json /tmp/fire_d3_prep/data_r3_freshness_$DATE.json \
  --output-dir reports/after_r1

# Step 5 - hard check (= W60-launchd-pre 修正後の Lane C strict check)
#   詳細は template_d3_real_artifact_prep.md §7

# Step 6 - trade plan / review template 複製
#   詳細は template_d3_real_artifact_prep.md §8 §9
```

## §7 safety 確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 | 0 (= staging dry-run 1 接続のみ、production 0) |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体参照 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 (= 本 wave は manual command path のみ) |
| VACUUM | 0 |
| 実発注 / 楽天 / iSPEED / Computer Use | 0 |
| 自動発注 | 0 |
| **sudo** | **0** ✓ |
| **git push** | **0** ✓ |
| **rm -rf** | **0** ✓ |
| **workflow / --no-verify** | **0** ✓ |
| TODO Excel 更新 | 0 |
| F062 / DATA-R3 / AFTER-R1 / F282 本体 変更 | 0 |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4610 → 4617 (= +7) |

## §8 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **4/4 = 100%** (= W60.5 と同等の安定 pattern 再現) |
| 短縮率 | 高 (= 4 lane 並列で 4 観点を ~4x 短縮) |
| 採用率 | 100% (= HIGH 1 + MEDIUM 2 全 修正、LOW 4 は次 wave 改善候補) |
| 差戻率 | 0 |
| Integrator 負荷 | 中-高 (= chain smoke + docs 3 件 + operation_plan update + 7 tests + Codex 4 lane + 修正) |
| 安全事故 | **0** ✓ |

### Codex 4/4 = 100% の評価

W60.5 (4 lane no sleep): 4/4 = 100% / W60.6 (8 lane): 7/8 / W60-pilot-pre
(8 lane): 3/8 / W60-pilot-D1 (Codex 不使用) / W60-pilot-D2 (Codex 不使用) /
W60-integration (6 lane + sleep): 4/6 / **W60-launchd-pre (4 lane no sleep):
4/4 = 100%**

**4 lane parallel pattern が最も安定** (= W60.5 / W60-launchd-pre で 100% 再現)。
**5 lane 以上は不安定、4 lane に絞るのが最善** という結論を再確認。

## §9 残課題 / 次 Wave

### §9.1 W60-launchd-pre で開ける道

- D3 (= 2026-05-18 月) で 4 step manual command 実行 → f062_preview 取得試行
- F111 朝 batch 未稼働でも、藤原さん手作り F111 で D3 D4 D5 を回せる
- W1 集約 (= W60-pilot-W1) で D1-D5 結果 + pattern 分類

### §9.2 並行 wave

- **W60-pilot-D3 (= 5/18 月)**: 本 sequence を実行、f062_preview GO 判定試行
- **W60-launchd-real (= 別 wave 検討)**: 本番 launchd 配置 + Wave 41/45 進捗
- **F111 朝 batch 設計**: 別 wave、実 朝 batch 化
- **W61-pre**: price/return data 取り込み + paper_pnl 連携

### §9.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §9.4 D1 → W1 path (= W60-integration 修正済 + W60-launchd-pre 後の状態)

| Day | 日付 | 曜日 | artifact_source 期待 | 判定 |
|---|---|---|---|---|
| D1 | 2026-05-14 | 木 | synthetic_fixture | HOLD ✓ (完了) |
| D2 | 2026-05-15 | 金 | synthetic_fixture | HOLD ✓ (完了) |
| - | 2026-05-16 | 土 | - | skip |
| - | 2026-05-17 | 日 | - | skip |
| **D3** | **2026-05-18** | **月** | **f062_preview (= 本 wave 経由) 期待** | **GO 候補 ★** |
| D4 | 2026-05-19 | 火 | f062_preview 期待 | GO 候補 |
| D5 | 2026-05-20 | 水 | f062_preview 期待 | GO 候補 |
| - | 2026-05-21 | 木 | - | W60-pilot-W1 集約 |

## §10 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_LAUNCHD_PRE_plan]]
- [[FIRE_CODEX_R1_WAVE60_INTEGRATION_results|W60-integration results]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D2_results|W60-pilot-D2 results]]
- [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- [[../04_daily/template_d3_real_artifact_prep|D3 manual command sequence]]
- [[../04_daily/template_f111_synthetic_preview|F111 synth template]]
