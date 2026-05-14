---
id: Production-v0-readiness-check-cli-v1-1
phase: 本番 v0 Launch / readiness CLI v1.1 統合
priority: 高
status: 実装 v1.1 (= Wave 43-pre、W40.8/W41-pre/W42-pre 統合)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40.7 (= readiness CLI v1.0)
  - Wave 40.8 (= F282 post-run inspection CLI)
  - Wave 41-pre (= DATA-R3 freshness producer)
  - Wave 42-pre (= F062 freshness consumer)
related:
  - 03_design/Production_v0_readiness_check_cli_2026-05-14.md
chapter: production-v0 / readiness CLI v1.1
---

# Production v0 Readiness Check CLI v1.1 — Wave 43-pre

最終更新: 2026-05-13

## §1 目的

W40.7 readiness CLI v1.0 を v1.1 に拡張。W40.8 F282 post-run / W41-pre
DATA-R3 freshness producer / W42-pre F062 freshness consumer の成果を
統合し、F282 → DATA-R3 → F062 → D-Day chain の機械的 GO/NO-GO 判定を
実現。後方互換性維持 (= 既存 15 test 全 PASS)。

## §2 現在地

- W40.5/40.6/40.7/40.8/41-pre/42-pre 完備
- 既存 W40.7 CLI: 1051 行 / 15 test / 6 phase / 19 check
- 本 wave で **v1.1 拡張**: 25 check (= 19 既存 + 6 新規) / 32 test

### Wave 43-pre 構成 (= 本線 + Codex 8 lane)

| lane | task_id | elapsed | verdict |
|---|---|---|---|
| L5 本線 | CLI v1.1 拡張 + 新 test + 動作確認 | 全 wave | — |
| Lane A | CLI v1.1 extension plan | 61s | READY |
| Lane B | DATA-R3 freshness consumer check | 40s | READY |
| Lane C | F282 post-run reference check | 37s | READY |
| Lane D | F062 no-send safety check | 41s | READY |
| Lane E | D-Day + HQ marker enhancement | 52s | READY |
| Lane F | pytest design | 90s | READY |
| Lane G | source safety audit | 74s | CONCERN 10 (= 監査不能、本線で補完) |
| Lane H | vault doc 12 章 outline | 65s | READY |

並列 max 90s vs 直列 460s = 約 80% 短縮。全 lane CRITICAL 0 / HIGH 0。

## §3 v1.1 拡張内容

### CLI_VERSION constant

```python
CLI_VERSION = "1.1"
```

JSON output に `cli_version` key 含める。

### 新規 6 check

1. `check_data_r3_freshness_report_schema_valid`
   - W41-pre /tmp/f286_data_r3_freshness.json を schema 1.0 で検証
   - pre-wave41 までは SKIP、pre-wave45+ で OK 必須
2. `check_data_r3_expected_sequence_matches`
   - expected_sequence = f100 → f101 → f111 → f119 を機械検証
3. `check_f062_freshness_consumer_capability`
   - F062 runner に W42-pre 関数 (load_freshness_report /
     check_freshness_verdict) + CLI option 存在確認
4. `check_f062_no_send_source_safety`
   - F062 runner AST 検証 (= linebot import 0、token literal 0)
   - docstring 説明文を誤検出しない (= AST Constant ノードのみ)
5. `check_f282_post_run_report_exists`
   - reports/f282/post_run_*.md 存在確認
   - post-f282 phase で PASS/WARN、それ以外 SKIP
6. `check_d_day_weekday_hq_decision_pending`
   - 2026-06-09 = Tuesday と HQ docs "Monday" の乖離を WARN
   - 3 候補 (6/8 Mon / 6/9 Tue / 6/15 Mon) を evidence に含める

### 新規定数

```python
DATA_R3_FRESHNESS_DEFAULT_PATH = Path("/tmp/f286_data_r3_freshness.json")
EXPECTED_DATA_R3_SEQUENCE = (
    "f100_historical",
    "f101_announcements",
    "f111_research_watchlist_signal_persistence",
    "f119_evaluation",
)
SUPPORTED_DATA_R3_SCHEMA_VERSIONS = ("1.0",)
F062_RUNNER_RELATIVE_PATH = "scripts/jobs/run_f062_research_advisory_line_preview.py"
D_DAY_CANDIDATE_OPTIONS = (
    ("2026-06-08", "Monday", "前倒し"),
    ("2026-06-09", "Tuesday", "現行 plan"),
    ("2026-06-15", "Monday", "1 週間延期"),
)
```

## §4 check 一覧 (= 25 check)

| カテゴリ | check 数 |
|---|---|
| F282 launchd/plist (既存) | 5 |
| DB safety (既存) | 3 |
| Plist absent/present (既存) | 2 |
| Token safety (既存) | 2 |
| No-send mode (既存) | 1 |
| HQ marker (既存) | 4 |
| Date (既存) | 2 |
| DATA-R3 (W43-pre) | 2 |
| F062 (W43-pre) | 2 |
| F282 post-run (W43-pre) | 1 |
| D-Day weekday HQ (W43-pre) | 1 |
| **合計** | **25** |

## §5 chain 全体像

```
F282 (W40.8 post-run inspection)
  → DATA-R3 (W41-pre daily refresh + freshness producer)
  → F062 (W42-pre preview + freshness consumer + safe abort)
  → D-Day v0 (HQ marker + 曜日確認)
```

各 phase で readiness CLI v1.1 が GO/NO-GO を機械判定。

## §6 phase 別判定マトリクス

| phase | DATA-R3 freshness | F282 post-run | F062 capability | D-Day weekday |
|---|---|---|---|---|
| pre-f282 | SKIP | SKIP | SKIP | SKIP |
| post-f282 | SKIP | PASS/WARN | SKIP | SKIP |
| pre-wave41 | SKIP | SKIP | SKIP | SKIP |
| pre-wave45 | PASS 必須 | SKIP | PASS 必須 | SKIP |
| pre-wave52 | PASS 必須 | SKIP | PASS 必須 | SKIP |
| pre-v0-launch | PASS 必須 | SKIP | PASS 必須 | WARN (= HQ 決定待ち) |

## §7 D-Day 曜日問題明示

HQ docs: D-Day = 月曜
実際: 2026-06-09 = 火曜 → 乖離

候補 3 案 (= HQ 判断事項):
- 2026-06-08 (Monday) = 1 日前倒し
- 2026-06-09 (Tuesday) = 現行 plan
- 2026-06-15 (Monday) = 1 週間延期

`check_d_day_weekday_hq_decision_pending` で WARN + 候補 evidence。

## §8 HQ marker 優先度

| phase | V0_LAUNCH marker |
|---|---|
| pre-wave41/45/52 | SKIP (= 該当 phase の marker のみ確認) |
| pre-v0-launch | WARN if absent (= 既存維持)、本 wave で FAIL 昇格は将来 |

既存 check_hq_marker_required_for_* 4 個は維持。

## §9 test 結果

新規 test class (= 17 test 追加):
- TestCliVersionV11 (= 2)
- TestDataR3FreshnessReportCheck (= 7)
- TestF062NoSendSafetyChecks (= 3)
- TestF282PostRunReferenceCheck (= 3)
- TestDDayWeekdayHqDecisionPending (= 2)

既存 15 test 全 PASS 維持 (= 後方互換)
新規 17 test 全 PASS
**合計 32 test 0.07s**

pytest collected: 4166 → **4183** (= +17)

## §10 safety 確認

- token / channel_token / secret 値 0 (= F062 source AST 検証で機械保証)
- env 全体読出 0
- DB write 0 / production/develop/staging DB sqlite 接続 0
- launchd/plist/cron 操作 0
- F282 plist mtime/size 完全不変
- LINE API call 0
- workflow / --no-verify / TODO Excel 0

## §11 v0 までの次アクション

```
5/14-5/15  baseline JSON 準備
5/16 02:00 F282 試走
5/16 03:00 W40.8 post-run inspection CLI 実行
           + readiness check CLI v1.1 --phase post-f282
5/19       F282 GO/NO-GO 判定
           + readiness check CLI v1.1 --phase pre-wave41
GO 後      Wave 41 (= DATA-R3 plist 配置)
           HQ_APPROVE_LAUNCHD_DAILY 必須
5/26+      Wave 45 (= F062 plist 配置 + 1 週間 no-send 試走)
           HQ_APPROVE_LAUNCHD_MORNING_ADVISORY 必須
           + readiness check CLI v1.1 --phase pre-wave45
6/7        readiness check CLI v1.1 --phase pre-wave52
           HQ_APPROVE_LINE_TOKEN_PRODUCTION 必須
6/8 月曜   readiness check CLI v1.1 --phase pre-v0-launch --strict
           HQ_APPROVE_PRODUCTION_V0_LAUNCH 必須
           D-Day 曜日 HQ 決定確認
6/9 火曜   D-Day v0 開始 (= HQ 決定済の場合)
```

## §12 6 KPI

- Codex 稼働率: 8/8 = 100%
- 本線短縮率: 約 80% (= 並列 90s vs 直列 460s)
- 採用率: 8/8 = 100%
- 差戻率: 0 (= 内製修正 1 件 = AST 検証移行 で吸収)
- Integrator 負荷: 中-高 (= 8 lane + CLI +300 行 + test +17)
- 安全事故 0: ✓

---

## 関連リンク

- [[FIRE_production_v0_launch_plan_2026-05-13|v0 Launch Plan]]
- [[Production_v0_readiness_check_cli_2026-05-14|W40.7 readiness CLI v1.0]]
- [[F282_post_run_inspection_report_cli_2026-05-14|W40.8 F282 post-run CLI]]
- [[F286_DATA_R3_wave41_pre_runner_enhancement_2026-05-14|W41-pre DATA-R3 強化]]
- [[F062_wave42_pre_no_send_runner_enhancement_2026-05-14|W42-pre F062 強化]]
- 実装: `~/fire/scripts/jobs/run_production_v0_readiness_check.py`
- test: `~/fire/tests/scripts/jobs/test_run_production_v0_readiness_check.py`
