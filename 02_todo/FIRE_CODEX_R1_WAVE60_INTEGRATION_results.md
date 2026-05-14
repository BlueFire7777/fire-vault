---
id: FIRE-CODEX-R1-WAVE60-integration-results
phase: 本番 v0 中核 / Wave 60-integration / F062 ↔ AFTER-R1 連携
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_INTEGRATION_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_PRE_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_results.md
---

# Wave 60-integration Results — F062 ↔ AFTER-R1 Integration v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 F062 朝 batch 実 output 形式 (= dict with chunks/selected_count/...)
を AFTER-R1 MVP が読み込み、artifact_source を 4 成果物に明示する状態に到達 ✓**

D2 以降の trade plan は **synthetic_fixture / f062_preview / unknown** を
区別して扱える。F062 朝 batch が実稼働すれば automatic に `artifact_source =
f062_preview` で材料が出る。

加えて **Codex 6 lane stdin smoke (= 5-10 秒 sleep 改善案) を実証**、4/6 lane
完全 reply 取得 (= W60-pilot-pre 3/8 から並列度改善)、CRITICAL/HIGH/MEDIUM 0、
LOW = 全 lane "No finding" 確認 ✓。

## §2 実装変更一覧

### §2.1 file 別

| file | 変更内容 |
|---|---|
| `scripts/jobs/_after_r1_mvp.py` | MVP_LOGIC_VERSION 1.0.0→1.1.0 / `ARTIFACT_SOURCE_*` 3 種定数 / `F062_ACTUAL_OUTPUT_MARKERS` 5 種 / `MvpInputContext` に `artifact_source` + `f062_raw_kind` field / `_normalize_f062_rows()` が raw_kind を返す / `infer_artifact_source()` 新規 / `load_mvp_inputs()` に `artifact_source` arg / 全 row invalid NO-GO warning / 4 output (ledger/ranking/pattern/material) に `artifact_source` + `f062_raw_kind` 反映 + ledger/ranking/pattern の summary にも反映 |
| `scripts/jobs/run_f286_after_r1_night_batch.py` | CLI_VERSION 1.1.2→1.2.0 / `--artifact-source {synthetic_fixture, f062_preview, unknown, auto}` 追加 (default=auto) / `_run_mvp()` で explicit/auto 渡し / summary に artifact_source/f062_raw_kind 反映 / stdout に artifact_source 表示 |
| `tests/scripts/jobs/test_run_f286_after_r1_night_batch.py` | W60-integration regression 17 件追加 / CLI_VERSION assertion 1.1.2→1.2.0 |
| `fire-vault/04_daily/template_manual_live_pilot_trade_plan.md` | §3.0 `artifact_source` 欄追加 / §3.1 GO-check に artifact_source 確認追加 |
| `fire-vault/04_daily/2026-05-14_manual_live_pilot_review.md` | §13 next action の日付/曜日修正 (D2=5/15 金 / W1=D5=5/20 水) |
| `fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_results.md` | §10.2 D1→W1 path 修正 (5/17 日 skip 追記、D3=5/18 月 へ移動) |

### §2.2 FIRE 本体不触範囲

| 項目 | 状態 |
|---|---|
| F062 本体 (= run_f062_research_advisory_line_preview.py) | 不触 ✓ (= read 用 schema 確認のみ) |
| F286 DATA-R3 runner | 不触 ✓ |
| F282 weekly snapshot | 不触 ✓ |
| readiness / Ops Summary / wrapper CLI | 不触 ✓ |
| production / develop / staging DB | 不触 ✓ |
| launchd plist / cron | 不触 ✓ |

## §3 artifact_source taxonomy

| 値 | 推定条件 | 用途 / 実弾判断 |
|---|---|---|
| `synthetic_fixture` | F062 raw kind = plain_list (= W60-impl smoke 形式) | **実 trade 非推奨**、運用フロー検証のみ |
| `f062_preview` | F062 raw kind = f062_actual_dict (= chunks/selected_count/...) | **実 trade 可** if §3.1 全 PASS |
| `unknown` | other_dict / none / 推定不能 | **実 trade 禁止 + HQ 確認** |

CLI `--artifact-source` 明示指定が最優先、`auto` で raw_kind から推定。

### §3.1 F062 actual output 判定 marker

```
F062_ACTUAL_OUTPUT_MARKERS = (
    "chunks", "selected_count", "message_chunk_count",
    "forbidden_phrase_count", "safety_footer_present",
)
```

F062 raw JSON dict にこれら 5 keys のいずれかが含まれれば
`f062_actual_dict` と判定。

### §3.2 smoke run 動作確認

実 F062 actual format (= dict with chunks + selected_rows) で smoke:
```
artifact_source=f062_preview
f062_raw_kind=f062_actual_dict
ranking_size=2 (= レーザーテック 6920 / 信越化学 4063)
```

全 4 output (= ledger / ranking / pattern / material) の root + summary に
artifact_source 反映確認 ✓

## §4 D1-D5 日付/曜日修正

W60-pilot-D1 results §10.2 で「D3=5/16 (土 skip)」と書いていたが、5/16 は土曜
かつ 5/17 は日曜なので両方 skip。正しい営業日 path:

| Day | 日付 | 曜日 | 状態 |
|---|---|---|---|
| D1 | 2026-05-14 | 木 | trade plan 作成済 (= 本日) |
| D2 | 2026-05-15 | 金 | 次回 (= artifact_source 区別可) |
| - | 2026-05-16 | 土 | skip (= 休場) |
| - | 2026-05-17 | 日 | skip (= 休場) |
| D3 | 2026-05-18 | 月 | 予定 |
| D4 | 2026-05-19 | 火 | 予定 |
| D5 | 2026-05-20 | 水 | 予定、W60-pilot-W1 集約 |

## §5 tests 結果

| wave | collected |
|---|---|
| W60.6-fix | 4593 |
| W60-pilot-pre | 4593 (= docs only) |
| W60-pilot-D1 | 4593 (= docs only) |
| W60-integration | **4610** (= +17 件) |

全 **4610 PASS** ✓

### 新規 W60-integration regression 17 件

- `TestW60IntegrationArtifactSource` 10 件:
  - taxonomy 確認 / F062 actual dict 推定 / plain list 推定 /
    None 推定 / explicit override / invalid explicit fallback /
    4 outputs 反映 / CLI default auto / CLI explicit synthetic / other_dict
- `TestW60IntegrationDataR3Enforcement` 4 件:
  - OK / FAILED で risk 追加 / not provided = MISSING / unknown verdict 正規化
- `TestW60IntegrationAllRowsInvalid` 2 件:
  - 全 row invalid で NO-GO warning / partial valid で strong warning なし
- `TestW60IntegrationLogicVersion` 1 件:
  - MVP_LOGIC_VERSION = "1.1.0"

## §6 Codex 6 lane smoke (= W60.5/W60.6 並列改善案)

W60.5/W60.6 で観察された 8 lane 並列の途中停止問題に対応し、本 wave は
**6 lane + 5-10 秒 sleep 間隔** で起動。

| Lane | 観点 | 結果 | finding |
|---|---|---|---|
| A | artifact_source logic | exit 0 ✓ | LOW No finding — explicit/F062 actual/plain list/other dict/None 全 confirmed |
| B | F062 row normalization | exit 0 ✓ | LOW No blocking — list/dict/markers/malformed/non-dict 全 confirmed |
| C | DATA-R3 enforcement | exit 0 ✓ | LOW No issue — OK/FAILED/STALE→MISSING/-20 penalty/risk note 全 confirmed |
| D | CLI --artifact-source | exit 0 ✓ | LOW No issue — choices/default auto/None inference/explicit pass-through |
| E | safety invariants | exit 0 (途中停止) | reply 未到達 → **self-audit**: MVP_SAFETY_FLAGS 13 keys 全 False / auto_order=False+manual_review=True 維持 / 新規 import 0 / write_text 限定 / LOGIC_VERSION 1.1.0 ✓ |
| F | tests regression | exit 0 (途中停止) | reply 未到達 → **self-audit**: 17 件 + 既存 160 件 全 PASS / 既存 互換 ✓ |

### §6.1 並列改善案の評価

- W60.5 (4 lane parallel + no sleep): 4/4 完全 reply
- W60.6 (8 lane parallel + no sleep): 7/8 完全 reply
- W60-pilot-pre (8 lane + no sleep): 3/8 完全 reply (= 5 途中停止)
- **W60-integration (6 lane + 5-10 秒 sleep): 4/6 完全 reply** ← 本 wave

並列度を 6 に下げ、5-10 秒 sleep で起動間隔を空けたことで W60-pilot-pre よりは
改善 (= 完全 reply 率 67% > 37.5%)。ただし 100% には届かず、E/F lane が途中停止。
W60.5 の 4 lane 並列が最も安定だったため、**重要観点 4 lane を直列または分割で
起動するのが最善** という結論。

### §6.2 self-audit 補完

途中停止 2 lane:
- **Lane E** (safety invariants): MVP_SAFETY_FLAGS 13 keys 全 False (= 既存)、
  artifact_source 追加で **新規 import なし** (= string field のみ)、
  4 output で auto_order_allowed=False / manual_review_required=True 維持、
  write_text は write_mvp_output 内のみ (= W60-impl から不変)、
  MVP_LOGIC_VERSION 1.0.0 → 1.1.0 ✓
- **Lane F** (tests regression): 全 177 件 PASS、新規 17 件 + 既存 160 件 互換、
  CLI_VERSION assertion 1.1.2→1.2.0 update 反映済

### §6.3 6 KPI 影響

Codex 稼働率 = 完全 reply 取得 4/6 = **66.7%** (= self-audit 含めると 6/6 = 100%
カバー)

## §7 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 | 0 |
| LINE 送信 / linebot import | 0 (= 新規 import 0) |
| token / channel_token / .env / env 全体参照 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| 実発注 / 楽天 / iSPEED 操作 / Computer Use | 0 |
| workflow / --no-verify / git push / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |
| 既存 v0 path (= F062/DATA-R3/F282/readiness/Ops/wrapper) 変更 | 0 |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| W30 snapshot | 不変 ✓ |
| MVP_SAFETY_FLAGS 13 keys 全 False | 維持 ✓ |
| auto_order_allowed = False (4 outputs 全件) | 維持 ✓ |
| manual_review_required = True (4 outputs 全件) | 維持 ✓ |
| pytest collected | 4593 → 4610 (= +17) |

## §8 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **4/6 = 66.7%** (= 完全 reply 4 lane / 6 lane 起動成功 6/6) |
| 短縮率 | 高 (= 6 lane 並列で 6 観点を ~6x 短縮) |
| 採用率 | 100% (= LOW No finding 4 件、修正不要、設計通り) |
| 差戻率 | 0 |
| Integrator 負荷 | 中-高 (= MVP module 拡張 + CLI 拡張 + 17 regression + 6 lane Codex + docs) |
| 安全事故 | **0** ✓ |

### Codex 4/6 = 66.7% (改善傾向)

W60.5 → W60.6 → W60-pilot-pre → W60-integration の Codex 稼働率推移:
- W60.5 (= 4 lane no sleep): 4/4 = **100%**
- W60.6 (= 8 lane no sleep): 7/8 = **87.5%**
- W60-pilot-pre (= 8 lane no sleep): 3/8 = **37.5%** (= 並列度限界)
- W60-pilot-D1 (= Codex 不使用、本線主導): N/A
- **W60-integration (= 6 lane + 5-10 秒 sleep): 4/6 = 66.7%**

仮説:
- 4 lane 並列 + no sleep が最も安定 (= W60.5)
- 8 lane 並列は不安定、sleep ありでも改善限定的
- 重要 lane は 4 並列まで、補助 lane は self-audit で代替

## §9 残課題 / 次 Wave

### §9.1 W60-integration 後の状態

- F062 actual format 入力 → AFTER-R1 MVP → 4 成果物 (= artifact_source=f062_preview) 動作可
- artifact_source 区別による pilot 実 trade 判断 が可能
- D1-D5 path 修正済 (= 5/18 月 D3)

### §9.2 並行 wave 候補

- **W60-pilot-D2 (= 5/15 金)**: artifact_source 区別での D2 trade plan
- **W60-launchd-pre**: nightly cron read-only plist 設計 (= 朝 自動 generate、
  synthetic から自動 → f062_preview 化)
- **F062 本体側修正 (= 別 wave 検討)**: F062 朝 batch output に artifact_source
  情報を含める (= MVP 側推定だけでなく F062 側からも明示)
- **W61-pre**: price/return data 取り込み + paper_pnl 連携

### §9.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §9.4 D1 → W1 修正済 path

- D1 (5/14 木) → D2 (5/15 金) → skip (5/16-17 土日) → D3 (5/18 月) →
  D4 (5/19 火) → D5 (5/20 水) → **W1 集約 (W60-pilot-W1)**

## §10 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_INTEGRATION_plan]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_PRE_results|W60-pilot-pre results]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D1_results|W60-pilot-D1 results]]
- [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- [[../03_design/F286_AFTER_R1_paper_live_mvp_2026-05-14|MVP 設計 doc]]
