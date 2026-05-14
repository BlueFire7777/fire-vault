---
id: F062-wave42-pre-no-send-runner-enhancement
phase: 本番 v0 Launch / Wave 45 Foundation
priority: 高
status: 実装 v1.0 (= Wave 42-pre、F062 freshness consumer + 25 新 test)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 41-pre (= DATA-R3 freshness report producer schema 1.0)
  - Wave 40.5/40.6/40.7/40.8 (= readiness/cutover/CLI 群)
related:
  - 03_design/F286_DATA_R3_wave41_pre_runner_enhancement_2026-05-14.md
  - 03_design/F062_morning_advisory_launchd_2026-05-13.md
chapter: production-v0 / F062 no-send runner enhancement
---

# F062 Morning Advisory No-Send Runner Enhancement — Wave 42-pre

最終更新: 2026-05-13

## §1 目的

W41-pre で確立した DATA-R3 freshness report (schema_version "1.0") を
F062 preview runner が読む consumer を実装。verdict != OK で preview
生成を停止する **no-send safe abort** を確立し、Wave 45 morning advisory
no-send trial で即着手可能にする。

本 wave は **実装 + test**、launchd 配置 / load / 送信 0 / DB write 0。

## §2 現在地

- W40 group + W41-pre (= DATA-R3 freshness producer) 完備
- F062 preview runner: 756 行、test 1292 行 (= 既存)
- 本 wave で freshness consumer + 25 新 test 追加

### Wave 42-pre 構成 (= 本線 + Codex 8 lane)

| lane | task_id | elapsed | verdict |
|---|---|---|---|
| L5 本線 | runner 強化 + 新 test + 動作確認 | 全 wave | — |
| Lane A | F062 runner enhancement plan | 59s | READY |
| Lane B | freshness consumer design | 24s | READY |
| Lane C | send-guard / no-send integrity | 63s | READY |
| Lane D | preview payload chunk policy | 49s | READY |
| Lane E | pytest enhancement design | 36s | READY |
| Lane F | F062 source safety audit | 54s | CONCERN 10 (= 監査不能、本線で補完) |
| Lane G | vault doc 12 章 outline | 32s | READY |
| Lane H | Wave 45 cutover handoff | 27s | READY |

並列 max 63s vs 直列 344s = 約 82% 短縮。全 lane CRITICAL 0 / HIGH 0。

## §3 実装内容

### 新規関数

```python
def load_freshness_report(path: Optional[Path]) -> Optional[dict]:
    """schema_version 1.0 を読込、不在/不正は None (= 安全側)."""

def check_freshness_verdict(report: Optional[dict]) -> tuple[bool, str]:
    """(should_proceed, reason) を返す。
    None → False, schema!=1.0 → False, OK → True, それ以外 → False."""

def summarize_freshness_for_preview(report: Optional[dict]) -> dict:
    """preview 同梱用の主要 field 抽出 (= token/secret 不含)."""
```

### 新規 CLI option

- `--freshness-report-json PATH` (default None、optional)
- `--require-freshness-ok` (store_true、default False)
  - 渡されない時は完全 後方互換 (= 既存 test 影響 0)

### 新規定数

- `SUPPORTED_FRESHNESS_SCHEMA_VERSIONS = ("1.0",)`

### main() 統合

input load **前** に freshness gate check:
- gate NG → 早期 `return 0` (= safe abort、output file 不在)
- gate OK → 通常 preview 生成へ進行

## §4 freshness consumer contract (= W41-pre と整合)

| condition | should_proceed | reason |
|---|---|---|
| report None (= file 不在/parse 失敗) | False | unavailable |
| schema_version != "1.0" | False | unsupported |
| verdict == "OK" | True | freshness OK |
| verdict in {STALE, MISSING, FAILED} | False | aborting + details |

W41-pre runner との連携:
1. 06:30 DATA-R3 で `/tmp/f286_data_r3_freshness.json` 生成
2. 08:45 F062 で同 file 読込 → verdict 判定 → preview 生成 or abort

## §5 send-guard 整合性 (= Lane C 反映)

- 本 runner: LINE 送信機能 0 (= linebot 系 import なし、AST 検証済)
- 別 runner (= run_f062_line_production_send_smoke.py) で send 経路 + HQ marker
- **二重 gate 構造** (= W52-post で marker 7 store_true gate に統合):
  - (本 runner) freshness OK → preview 生成
  - (別 runner) `--hq-approved-send` store_true gate (= W52-pre/post 統合)
    + marker 7 (`HQ_APPROVE_PRODUCTION_V0_LAUNCH`) 存在で `SEND_FLAGS+=(--hq-approved-send)`
  - いずれか NG → chain-level safe (= 下流が input 不在 / marker 不在で abort)

## §6 no-send preview payload 規約 (= Lane D 反映)

- max_chunks=1 (= 既存 max_chars で間接制御、維持)
- partial_delivery=False (= 別 runner で enforce、本 runner 範囲外)
- DB write 0 (= preview のみ、record-decisions 連携は別 wave)
- preview output file は **freshness gate 通過時のみ** 生成

## §7 test 結果

新規 test class (= 25 test 追加):
- `TestFreshnessConsumerLoad` (5)
- `TestFreshnessVerdictCheck` (6)
- `TestSummarizeFreshnessForPreview` (2)
- `TestFreshnessGateMainIntegration` (5)
- `TestF062NoSendSourceSafety` (3)
- `TestF062F282NonInterference` (4)

既存 test 37 → 37 全 PASS (= 後方互換維持)
新規 test 25 → 25 全 PASS
**合計 62 test 0.11s 全 PASS**

pytest collected: 4141 → **4166** (= +25)

## §8 safety 確認

| 項目 | 結果 |
|---|---|
| linebot 系 import | 0 (= AST 検証で機械保証) |
| launchctl literal | 0 (= AST 検証) |
| LINE_CHANNEL_TOKEN / CHANNEL_TOKEN / JQUANTS_API_KEY / LINE_USER_ID 文字列 | 0 |
| F282 plist hardcode | 0 |
| F282 log hardcode | 0 |
| data/snapshot/ への write | 0 |
| freshness report file | read のみ、write 0 |
| DB write | 0 |
| production / develop DB 接続 | 0 |
| LINE API call | 0 |
| env 全体読出 | 0 |
| VACUUM SQL | 0 |
| subprocess launchctl / cron | 0 |

## §9 F282 不干渉確認

baseline (= W42-pre 開始、14:41 JST):
- F282 plist mtime=1778593597 size=1772
- 3 環境 DB / W30 snapshot 既知値

完了時 (= 14:50 JST):
- 全て baseline と完全一致 ✓
- F282 next run 5/16 02:00 維持 ✓
- daily-refresh / morning-advisory plist 不在維持 ✓
- pytest 4141 → 4166 (= +25 想定通り)

## §10 Wave 45 no-send trial 接続

```
5/16 02:00 F282 試走
5/16 03:00 post-run inspection
5/19 F282 GO/NO-GO 判定
GO 後:
  Wave 41: DATA-R3 plist 配置 + 1 週間 no-write 試走
    → 毎朝 /tmp/f286_data_r3_freshness.json 生成
Wave 45:
  HQ_APPROVE_LAUNCHD_MORNING_ADVISORY 取得後
  jp.fire.morning-advisory.plist 配置 (= W40-post 設計)
  ProgramArguments:
    --dry-run
    --freshness-report-json /tmp/f286_data_r3_freshness.json
    --require-freshness-ok
    --preview-json <input>
    --output-json <out>
  launchctl load → 月-金 08:45 起動 → 1 週間 no-send 試走
```

## §11 残課題

| # | 項目 | 対応 |
|---|---|---|
| 1 | ~~HQ_APPROVE_SEND_MARKER 仕様確定~~ → **W52-pre/post で曖昧名廃止 + marker 7 store_true gate に統合済** | 完了 (= W52-post) |
| 2 | send_guard 別 runner 側強化 | Wave 45 |
| 3 | record-decisions DB write 経路 | Wave 45 後半 |
| 4 | partial_delivery 別 runner test 強化 | Wave 45 |
| 5 | production env 移行 | Wave 52 |

## §12 6 KPI

- Codex 稼働率: 8/8 = 100%
- 本線短縮率: 約 82% (= 並列 63s vs 直列 344s)
- 採用率: 8/8 = 100%
- 差戻率: 0 (= 内製修正 3 件で吸収)
- Integrator 負荷: 中-高 (= 8 lane + runner +90 行 + test +25)
- 安全事故 0: ✓

---

## 関連リンク

- [[FIRE_production_v0_launch_plan_2026-05-13|v0 Launch Plan]]
- [[F286_DATA_R3_wave41_pre_runner_enhancement_2026-05-14|W41-pre DATA-R3 強化]]
- [[F062_morning_advisory_launchd_2026-05-13|W40-post morning advisory 設計]]
- [[F062_no_send_trial_checklist_and_v0_readiness_2026-05-14|W40.5 readiness checklist]]
- [[Production_v0_cutover_rollback_token_runbook_2026-05-14|W40.6 cutover runbook]]
- 実装: `~/fire/scripts/jobs/run_f062_research_advisory_line_preview.py`
- test: `~/fire/tests/scripts/jobs/test_run_f062_research_advisory_line_preview.py`
