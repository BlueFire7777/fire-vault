---
id: F286-DATA-R3-wave41-pre-runner-enhancement
phase: 本番 v0 Launch / Wave 41 Foundation
priority: 高
status: 実装 v1.0 (= Wave 41-pre、runner 強化 + freshness gate + 15 新 test)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40-pre (= DATA-R3 launchd 設計 v1.0)
  - Wave 40.5/40.6/40.7/40.8 (= readiness/cutover/CLI 群)
related:
  - 03_design/F286_DATA_R3_daily_refresh_launchd_2026-05-13.md
chapter: production-v0 / DATA-R3 runner enhancement
---

# F286-DATA-R3 Wave 41-pre Runner Enhancement — 設計 v1.0

最終更新: 2026-05-13

## §1 目的

F282 GO 後 (= 5/19) に Wave 41 本番 (= plist 配置 + launchctl load
+ 1 週間 no-write 試走) で即着手できるよう、DATA-R3 daily refresh
runner の強化 + freshness gate + test 強化 + vault 記録を完了。

**本 wave は実装 + test、plist 配置 / launchctl 0 / DB write 0**。

## §2 現在地

- W40.5 56 項目 checklist + 40.6 cutover runbook + 40.7 readiness CLI
  + 40.8 post-run CLI 完備
- DATA-R3 既存 runner 707 行、test 660 行 (= W11 まで)
- **本 wave で**: runner 強化 + freshness gate report + 15 新 test 追加

### Wave 41-pre 構成 (= 本線 + Codex 8 lane)

| lane | task_id | elapsed | verdict |
|---|---|---|---|
| L5 本線 | runner 強化 + 新 test 実装 + 動作確認 | 全 wave | — |
| Lane A | runner enhancement plan | 59s | READY |
| Lane B | freshness gate report design | 33s | READY |
| Lane C | subprocess sequence order design | 33s | READY |
| Lane D | 6 mandatory test enhancement | 54s | READY |
| Lane E | F282 non-interference test | 27s | READY |
| Lane F | source-level safety audit | 54s | CONCERN 9 (= 監査不能、本線で補完) |
| Lane G | morning advisory contract | 32s | READY |
| Lane H | vault doc 12 章 outline | 43s | READY |

並列 max 59s vs 直列 335s = 約 82% 短縮。全 lane CRITICAL 0 / HIGH 0。

## §3 実装内容

### sub-runner 順序整列 (= Lane C 反映)

`list_daily_refresh_jobs` return list の順序を整列:

```
旧: f100 → f111 → f101 → f119
新: f100 → f101 → f111 → f119  (= 価格 → 材料 → signal → 評価)
```

定数 `EXPECTED_JOB_SEQUENCE` を新規追加、test で順序固定を保証。

### freshness gate report (= Lane A + B 反映)

新規関数:
- `build_freshness_summary(payload, *, stale_threshold_hours=24) -> dict`
- `write_freshness_report(path, summary) -> None`

新規 const:
- `EXPECTED_JOB_SEQUENCE`
- `FRESHNESS_REPORT_SCHEMA_VERSION = "1.0"`
- `DEFAULT_STALE_THRESHOLD_HOURS = 24`

新規 CLI option:
- `--freshness-report-json PATH` (default `/tmp/f286_data_r3_freshness.json`)
- `--stale-threshold-hours INT` (default 24)

`main()` で freshness summary を build + write、結果 verdict を stdout に出力。

### freshness gate report schema (= Lane B 反映)

```json
{
  "report_id": "data_r3_freshness_YYYY-MM-DD",
  "schema_version": "1.0",
  "generated_at": "ISO 8601 UTC",
  "base_date": "YYYY-MM-DD",
  "source_version": "...",
  "aggregate_exit_code": int,
  "verdict": "OK" | "STALE" | "MISSING" | "FAILED",
  "stale_threshold_hours": 24,
  "subrunners": [
    {"job_id": "...", "status": "...", "exit_code": int, "elapsed_sec": float}
  ],
  "missing_subrunners": [...],
  "expected_sequence": [...],
  "f282_safety": {
    "checked": true,
    "f282_touched": false,
    "note": "..."
  }
}
```

### verdict 判定ルール

| 条件 | verdict |
|---|---|
| aggregate_exit_code = 0 + dry_run_results 充実 + missing 0 | OK |
| dry_run_results 空 or 全 missing | MISSING |
| missing_subrunners 存在 | MISSING |
| aggregate_exit_code != 0 | FAILED |

(stale 判定は morning advisory 側で生成 mtime と比較、本 wave 範囲外)

## §4 freshness gate 仕様 (= morning advisory contract)

morning advisory (= F062) が読む形:

```python
import json
from pathlib import Path

report = json.loads(Path("/tmp/f286_data_r3_freshness.json").read_text())
if report["verdict"] != "OK":
    abort_advisory(f"DATA-R3 freshness {report['verdict']}")
# stale 判定 (= advisory 側)
generated_at = parse_iso(report["generated_at"])
if now() - generated_at > timedelta(hours=24):
    abort_advisory("DATA-R3 stale")
```

実 morning advisory consumer 実装は Wave 45 で。本 wave は producer 側のみ。

## §5 subprocess sequence (= Lane C 反映)

- 順序: f100 → f101 → f111 → f119
- 各 sub-runner は `--dry-run` で subprocess 実行
- 1 つでも失敗 → `aggregate_dry_run_exit_code = max(各)` で non-zero
- 後続 sub-runner は **継続実行** (= 全体 diagnosis 目的)
- 並列実行禁止 (= ThreadPool / asyncio 不使用)

`test_subprocess_dry_run_each_job_preserves_order` で順序機械検証。

## §6 test 結果

新規 test class (= 15 test 追加):
- `TestSubprocessSequenceOrder` (3 test)
- `TestFreshnessReport` (5 test)
- `TestF282NonInterferenceSource` (3 test)
- `TestF282NonInterferenceRuntime` (2 test)
- `TestFreshnessReportArgparseIntegration` (2 test)

既存 test class:
- 37 → 37 (= TestListDailyRefreshJobsContent 1 件のみ assert を新順序に更新)

**pytest 結果**:
- 既存 test 37 → 37 全 PASS
- 新規 test 15 → 15 全 PASS
- **合計 52 test 全 PASS (0.07s)**
- pytest collected: 4126 → **4141** (= +15)

## §7 safety 確認 (= 全 ✓)

| 項目 | 結果 |
|---|---|
| production / develop DB write | 0 |
| staging DB write | 0 (= dry-run only) |
| launchctl 呼出 | 0 (= AST 検証 + 既存 TestArgParserSafety) |
| plist 配置 / 変更 | 0 |
| cron / crontab 登録 | 0 |
| LINE 送信 | 0 |
| token / channel_token / secret 値 | 0 (= log/json 出力に含まれない) |
| env 全体 dump | 0 |
| F101 staging probe | 0 |
| VACUUM SQL | 0 |
| F282 plist / log / snapshot 触り | 0 (= test source + runtime 両方で verify) |
| workflow / --no-verify / TODO Excel | 0 |

## §8 F282 不干渉確認

| 項目 | baseline (= W41-pre 開始) | 完了時 | 結果 |
|---|---|---|---|
| F282 plist mtime/size | 1778593597 / 1772 | 同 | ✓ unchanged |
| data/fire.db | 1778570244 / 371081216 | 同 | ✓ |
| data/fire.develop.db | 1778569903 / 371081216 | 同 | ✓ |
| data/fire.staging.db | 1778579122 / 4804063232 | 同 | ✓ |
| W30 snapshot | 全 mtime/size | 同 | ✓ |
| F282 next run | 5/16 02:00 (Weekday 7) | 同 | ✓ |

## §9 Wave 41 本番への接続

```
5/16 02:00  F282 本番試走 (= launchd 自動)
5/16 03:00  post-run inspection (= W40.8 CLI + W40.7 CLI)
5/19        F282 GO/NO-GO 判定
GO 後:
  1. W40.7 readiness CLI --phase pre-wave41 で着手前確認
  2. HQ_APPROVE_LAUNCHD_DAILY 取得
  3. ~/fire/docs/launchd/jp.fire.daily-refresh.plist 配置
  4. ~/Library/LaunchAgents/jp.fire.daily-refresh.plist 配置
  5. plutil -lint 検証
  6. launchctl load 実行
  7. launchctl list / print 確認
  8. 1 週間 no-write 試走 (= 月-金 5 営業日)
  9. 毎朝 06:30 起動 → freshness report 生成 → daily check
  10. 5 営業日全 PASS → Wave 41 完了 → Wave 45 へ
```

## §10 残課題

| # | 項目 | 対応 |
|---|---|---|
| 1 | production write 経路 | 別 wave、HQ_APPROVE_PRODUCTION_V0_LAUNCH 必須 |
| 2 | freshness gate consumer (= morning advisory) | Wave 45 |
| 3 | daily check 自動化 (= 翌朝確認) | 別 wave |
| 4 | stale 判定の producer 側強化 | 別 wave (= 現在は consumer 側) |
| 5 | freshness report 多日 retention | 別 wave |

## §11 6 KPI

- Codex 稼働率: 8/8 = 100% (= Lane A-H 全 exit 0)
- 本線短縮率: 約 82% (= 並列 59s vs 直列 335s)
- 採用率: 8/8 = 100% (= 全 lane 成果が runner/test/doc に反映)
- 差戻率: 0 (= 内製修正 3 件で吸収、HQ 差戻 0)
- Integrator 負荷: 中-高 (= 8 lane + runner 順序整列 + 新規 130 行 + test 15 個)
- 安全事故 0: ✓

## §12 関連リンク

- [[FIRE_production_v0_launch_plan_2026-05-13|v0 Launch Plan]]
- [[F286_DATA_R3_daily_refresh_launchd_2026-05-13|W40-pre DATA-R3 launchd 設計]]
- [[F062_morning_advisory_launchd_2026-05-13|W40-post F062 morning advisory]]
- [[F062_no_send_trial_checklist_and_v0_readiness_2026-05-14|W40.5 readiness]]
- [[Production_v0_cutover_rollback_token_runbook_2026-05-14|W40.6 cutover runbook]]
- [[Production_v0_readiness_check_cli_2026-05-14|W40.7 readiness CLI]]
- [[F282_post_run_inspection_report_cli_2026-05-14|W40.8 post-run CLI]]
- 実装 runner: `~/fire/scripts/jobs/run_f286_data_r3_daily_refresh.py`
- 実装 test: `~/fire/tests/scripts/jobs/test_run_f286_data_r3_daily_refresh.py`
