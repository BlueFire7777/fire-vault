---
id: F282-baseline-capture-and-post-run-drill
phase: 本番 v0 Launch / F282 試走前準備
priority: 高
status: 実装 v1.0 (= Wave 44-pre、baseline helper + drill 手順 + GO/NO-GO テンプレ)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40.8 (= F282 post-run inspection CLI)
  - Wave 43-pre (= readiness CLI v1.1)
related:
  - 03_design/F282_post_run_inspection_report_cli_2026-05-14.md
  - 03_design/Production_v0_readiness_check_cli_v1_1_2026-05-14.md
chapter: production-v0 / F282 baseline + drill
---

# F282 Baseline Capture + Post-Run Drill Pack — Wave 44-pre

最終更新: 2026-05-13

## §1 目的

2026-05-16 02:00 JST F282 本番試走後、03:00 の post-run 確認を迷わず
実行できるよう以下を整備:
1. baseline JSON capture helper (= 完全 read-only)
2. W40.8 CLI / W43-pre CLI と連携する drill 6 step 手順
3. GO/NO-GO 報告テンプレ

本 wave は **準備のみ**。launchctl 実行 / F282 手動実行 / DB write / VACUUM 0。

## §2 現在地

- W40.5/40.6/40.7/40.8/41-pre/42-pre/43-pre 完備
- 本 wave で baseline helper + drill docs + GO/NO-GO テンプレ完成
- baseline JSON は 5/13 時点で /tmp/f282_baseline_2026-05-16.json 実 path に生成済

### Wave 44-pre 構成 (= 本線 + Codex 8 lane)

| lane | task_id | elapsed | verdict |
|---|---|---|---|
| L5 本線 | helper 実装 + test + vault docs | 全 wave | — |
| Lane A | baseline capture helper design | 60s | READY |
| Lane B | baseline JSON schema W40.8 整合 | 49s | READY |
| Lane C | 5/16 drill 6-step design | 54s | READY |
| Lane D | GO/NO-GO report template | 40s | READY |
| Lane E | pytest design | 56s | READY |
| Lane F | source safety audit | 80s | CONCERN 1 (= 軽微) |
| Lane G | vault doc 12 章 outline | 53s | READY |
| Lane H | chain design | 39s | READY |

並列 max 80s vs 直列 431s = 約 81% 短縮。全 lane CRITICAL 0 / HIGH 0。

## §3 実装内容

### 新規 file

- `~/fire/scripts/jobs/run_f282_baseline_capture.py` (= 約 280 行、read-only helper)
- `~/fire/tests/scripts/jobs/test_run_f282_baseline_capture.py` (= 19 test)
- `~/fire-vault/03_design/F282_baseline_capture_and_post_run_drill_2026-05-14.md` (= 本 doc)
- `~/fire-vault/04_daily/template_f282_post_run.md` (= GO/NO-GO テンプレ)

### CLI 仕様

```bash
.venv/bin/python -m scripts.jobs.run_f282_baseline_capture \
  --output /tmp/f282_baseline_2026-05-16.json \
  --repo-root ~/fire \
  --expected-run-date 2026-05-16 \
  --notes "Pre-F282 trial baseline" \
  --include-snapshot
```

- exit code: 0 (GO) / 1 (WARN = plist 不在) / 2 (invalid args or unsafe path)
- output path guard: data/ / .git/ / .github/ / LaunchAgents/ / .fire_secrets/ refuse

## §4 baseline JSON schema v1.0 (= W40.8 整合)

```json
{
  "schema_version": "1.0",
  "generated_at": "2026-05-13T09:24:07+00:00",
  "expected_run_date": "2026-05-16",
  "notes": "Pre-F282 trial baseline",
  "f282_plist": {
    "path": "/Users/bluefire/Library/LaunchAgents/jp.fire.weekly-snapshot.plist",
    "exists": true,
    "mtime": 1778593597,
    "size": 1772,
    "mtime_iso": "2026-05-12T22:46:37+09:00"
  },
  "db": {
    "data/fire.db": {"path": "...", "exists": true, "mtime": ..., "size": ..., "mtime_iso": "..."},
    "data/fire.develop.db": {...},
    "data/fire.staging.db": {...}
  },
  "snapshot": {
    "data/snapshot/fire.staging.db": {...},
    "data/snapshot/fire.develop.db": {...}
  },
  "safety_flags": {
    "sqlite_connect": false,
    "launchctl_call": false,
    "db_write": false,
    "vacuum_executed": false,
    "token_accessed": false,
    "line_send": false
  }
}
```

W40.8 `load_baseline()` で読込確認済 ✓。f282_plist.mtime/size + db.*.mtime/size
が W40.8 の必須 fields。

## §5 5/16 03:00 drill 手順 (= 6 step)

### Step 1: launchctl print 保存 (= 人間操作)

```bash
launchctl print gui/$(id -u)/jp.fire.weekly-snapshot \
  > /tmp/f282_print_2026-05-16.txt
wc -l /tmp/f282_print_2026-05-16.txt  # 数十行期待
```

**注意**: PID=- / LastExitStatus=0 単独で判定不可 (= W39-temp 教訓)。
runs / last exit code / state を確認。

### Step 2: baseline JSON 確認

```bash
ls -la /tmp/f282_baseline_2026-05-16.json
python -m json.tool /tmp/f282_baseline_2026-05-16.json | head -n 30
```

確認項目:
- schema_version = "1.0"
- f282_plist.mtime = 1778593597 / size = 1772
- db keys 3 個存在
- safety_flags 全 false

### Step 3: W40.8 post-run inspection CLI 実行

```bash
.venv/bin/python -m scripts.jobs.run_f282_post_run_inspection_report \
  --launchctl-print-file /tmp/f282_print_2026-05-16.txt \
  --baseline-json /tmp/f282_baseline_2026-05-16.json \
  --markdown reports/f282/post_run_2026-05-16.md \
  --expected-run-date 2026-05-16
```

期待: exit 0 (= GO) または 1 (= NO-GO) + markdown report 生成。

### Step 4: W43-pre readiness CLI **v1.2 (W51 fix 後)** 実行

```bash
.venv/bin/python -m scripts.jobs.run_production_v0_readiness_check \
  --phase post-f282 --json
```

期待: cli_version "1.2" + F282_POST_RUN_REPORT_EXISTS = PASS。
※ W51 で v1.1 → v1.2 に bump 済、HQ marker phase gating 修正反映。

### Step 4.5: W44.6-pre Ops Summary CLI smoke 実行

```bash
.venv/bin/python -m scripts.jobs.run_production_v0_ops_summary \
  --phase post-f282 --date 2026-05-16 \
  --markdown reports/ops/v0_summary_2026-05-16_post_f282.md
```

期待: overall_verdict GO (= F282 PASS) または HOLD (= checklist 未記入)。
NO-GO の場合は §6 GO/NO-GO 報告で詳細記録。

### Step 5: 成果物確認

```bash
cat reports/f282/post_run_2026-05-16.md
ls -la reports/f282/
```

### Step 6: GO/NO-GO 報告記入

`~/fire-vault/04_daily/template_f282_post_run.md` を参照。
`~/fire-vault/04_daily/2026-05-16_f282_post_run.md` に commit。

### 禁止事項 (= drill 当日も)

- launchctl unload / load / kickstart 0
- F282 plist 編集 0
- F282 手動実行 0
- DB write 0 / VACUUM INTO 0
- LINE 送信 0 / token 参照 0

## §6 GO/NO-GO 報告テンプレ

別 file `~/fire-vault/04_daily/template_f282_post_run.md` 参照 (= 12 section、
Fujiwara 記入用)。

## §7 baseline 事前作成タイミング

| 日時 | アクション |
|---|---|
| 2026-05-13 (本日) | baseline JSON 5/13 時点で生成済 (= /tmp/f282_baseline_2026-05-16.json) |
| 2026-05-14 or 5/15 | baseline JSON 再生成可 (= 直前 stat 反映) |
| 2026-05-16 02:00 | F282 試走 (= launchd 自動) |
| 2026-05-16 03:00 | 6-step drill 実行 |

## §8 test 結果

新規 test class (= 19 test):
- TestBaselineSchemaValid (= 3)
- TestMissingPlistHandled (= 1)
- TestDbStatOnly (= 1)
- TestNoLaunchctlCall (= 1)
- TestNoVacuumSql (= 1)
- TestNoTokenAccess (= 3、AST 検証)
- TestOutputPathGuard (= 3)
- TestNoDataWrite (= 1)
- TestW408Compatibility (= 1)
- TestF282PlistUnchangedRuntime (= 1)
- TestArgparseSafety (= 2)
- TestInvalidExpectedRunDate (= 1)

合計 **19 test 全 PASS** (= 0.04s)。
pytest collected: 4183 → **4202** (= +19)。

## §9 safety 確認

- sqlite3 import 0 (= AST 機械検証)
- launchctl call 0 (= subprocess.run 全 mock 確認)
- DB write 0 / production/develop/staging DB write 0
- VACUUM SQL 0 (= AST 機械検証)
- token / channel_token / secret literal 0 (= AST 機械検証)
- env 全体読出 0
- output path guard: data/ / .git/ / LaunchAgents/ / .fire_secrets/ refuse
- LINE API call 0
- workflow / --no-verify / TODO Excel 0

## §10 F282 不干渉確認

baseline (= W44-pre 開始、18:18 JST):
- F282 plist mtime=1778593597 size=1772
- 3 環境 DB 既知値

完了時 (= 18:24 JST):
- 全て baseline と完全一致 ✓
- F282 next run 5/16 02:00 維持 ✓
- W30 snapshot 完全保持 ✓
- pytest 4183 → 4202 (= +19 想定通り)

## §11 5/16 当日チェックリスト

- 事前 (5/14-5/15):
  - [ ] baseline JSON が /tmp/f282_baseline_2026-05-16.json に存在
  - [ ] schema_version = "1.0"
  - [ ] safety_flags 全 false
- 当日 03:00:
  - [ ] Step 1: launchctl print 保存
  - [ ] Step 2: baseline JSON 確認
  - [ ] Step 3: W40.8 CLI 実行
  - [ ] Step 4: W43-pre CLI 実行
  - [ ] Step 5: 成果物確認
  - [ ] Step 6: GO/NO-GO 記入

## §12 6 KPI

- Codex 稼働率: 8/8 = 100%
- 本線短縮率: 約 81% (= 並列 80s vs 直列 431s)
- 採用率: 8/8 = 100%
- 差戻率: 0 (= 内製修正 0)
- Integrator 負荷: 中-高 (= 8 lane + helper 280 行 + test 19 + 12 章 doc + GO/NO-GO テンプレ)
- 安全事故 0: ✓

---

## 関連リンク

- [[FIRE_production_v0_launch_plan_2026-05-13|v0 Launch Plan]]
- [[F282_post_run_inspection_report_cli_2026-05-14|W40.8 post-run CLI]]
- [[Production_v0_readiness_check_cli_v1_1_2026-05-14|W43-pre readiness CLI v1.1]]
- [[../04_daily/template_f282_post_run|GO/NO-GO テンプレ]]
- 実装: `~/fire/scripts/jobs/run_f282_baseline_capture.py`
- test: `~/fire/tests/scripts/jobs/test_run_f282_baseline_capture.py`
