---
id: F282-post-run-inspection-report-cli
phase: 本番 v0 Launch / F282 試走後検証 CLI
priority: 高
status: 実装 v1.0 (= Wave 40.8、read-only CLI + 21 test + 12 章 doc)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40.7 (= v0 readiness CLI 実装パターン参照)
  - W39-temp 教訓 (= PID=-/LastExit=0 単独判定誤りの auto memory)
related:
  - 03_design/Production_v0_readiness_check_cli_2026-05-14.md
  - 03_design/F282_weekly_snapshot_launchd_2026-05-12.md
  - 03_design/F282_temporary_launchd_smoke_2026-05-13.md
chapter: production-v0 / F282 post-run inspection CLI
---

# F282 Post-Run Inspection Report CLI — Wave 40.8

最終更新: 2026-05-13

## §1 目的

2026-05-16 02:00 JST 予定の F282 weekly snapshot 本番試走後、03:00 JST の
実行後チェックを **安全・機械的** に行う read-only CLI を実装。

- **Wave 39-temp 教訓を組み込み**: PID=- / LastExitStatus=0 単独では未起動と
  単発起動完了後の区別不可。`runs` field + log + snapshot output の
  **3 点同時確認** を必須化、警告を report に必ず含める。
- 実 file: `scripts/jobs/run_f282_post_run_inspection_report.py`
- 実 test: `tests/scripts/jobs/test_run_f282_post_run_inspection_report.py`
- 安全制約: launchctl 0 / DB write 0 / VACUUM 0 / LINE 0 / token 0

## §2 CLI 仕様

### 引数

| 引数 | 必須/任意 | 役割 |
|---|---|---|
| --repo-root PATH | 任意 | default ~/fire |
| --launchctl-print-file PATH | 任意 | 保存済 launchctl print 結果 text |
| --baseline-json PATH | 任意 | 不変比較 baseline JSON |
| --json | 任意 | JSON stdout |
| --markdown PATH | 任意 | Markdown report file 出力 |
| --strict | 任意 | WARN を NO-GO 扱い |
| --expected-run-date YYYY-MM-DD | 任意 | default = 直近土曜 |
| --expected-hour INT | 任意 | default 2 |
| --expected-minute INT | 任意 | default 0 |

### exit code

- `0` = verdict GO
- `1` = verdict NO-GO (= FAIL あり、または strict+WARN あり)
- `2` = invalid args (= argparse / baseline file 不在 / launchctl-print-file 不在)

## §3 check 一覧 (= 20 check 関数)

| カテゴリ | # | check_id | severity |
|---|---|---|---|
| F282 launchd/plist | 1 | F282_PLIST_EXISTS | high |
| | 2 | F282_PLIST_LABEL_CORRECT | high |
| | 3 | F282_PLIST_SCHEDULE_SATURDAY_0200 | high |
| | 4 | F282_PLIST_MTIME_BASELINE_MATCH | high / SKIP |
| | 5 | LAUNCHCTL_PRINT_FILE_PROVIDED | medium |
| | 6 | LAUNCHCTL_PRINT_HAS_RUN_EVIDENCE | high |
| | 7 | LAUNCHCTL_PRINT_NOT_TREATED_AS_PROOF_ALONE | medium (= 注意常時 PASS) |
| F282 logs | 8 | WEEKLY_SNAPSHOT_LOG_EXISTS | high |
| | 9 | WEEKLY_SNAPSHOT_ERR_HANDLED | low |
| | 10 | LOG_CONTAINS_RUN_WINDOW_EVIDENCE | high |
| | 11 | ERR_LOG_NO_CRITICAL_ERROR | medium |
| Snapshot output | 12 | DATA_SNAPSHOT_DIR_EXISTS | high |
| | 13 | SNAPSHOT_FILES_EXIST | high |
| | 14 | SNAPSHOT_FILES_NON_ZERO_SIZE | high |
| | 15 | SNAPSHOT_FILES_MTIME_AFTER_EXPECTED | high |
| | 16 | SNAPSHOT_INTEGRITY_CHECK_OK | high |
| DB safety | 17 | DB_PRODUCTION_STAT_READABLE | medium |
| | 18 | DB_DEVELOP_STAT_READABLE | medium |
| | 19 | DB_STAGING_STAT_READABLE | medium |
| | 20 | DB_BASELINE_UNCHANGED | high / SKIP |

Report 5 観点 (= 21-25 相当) は出力構造に組み込み:
- 21 OVERALL GO/NO-GO summary
- 22 evidence table (= Markdown report)
- 23 next_action 表示
- 24 THREE_POINT_WARNING
- 25 RETENTION_NOTE

## §4 launchctl print 保存テキストの扱い

`subprocess` で `launchctl` を直接呼ばない設計。保存済み text file を
`--launchctl-print-file` で入力。例:

```bash
# Step 1: 手動で launchctl print 出力を保存
launchctl print gui/$(id -u)/jp.fire.weekly-snapshot \
  > /tmp/f282_print_2026-05-16.txt

# Step 2: CLI で解析
.venv/bin/python -m scripts.jobs.run_f282_post_run_inspection_report \
  --launchctl-print-file /tmp/f282_print_2026-05-16.txt \
  --expected-run-date 2026-05-16
```

### parser 仕様

`parse_launchctl_print(text: str) -> dict` で以下を抽出:
- `label` / `last_exit_status` / `runs` / `state` / `last_exit_code`
- `descriptor_weekday` / `descriptor_hour` / `descriptor_minute`

`has_run_evidence(parsed)` で `runs >= 1` 判定 (= W39-temp 教訓)。

## §5 PID=- / LastExitStatus=0 注意 (= W39-temp 教訓)

Wave 39-temp 中間結論誤りの教訓:

> `launchctl list` の `PID=- / LastExitStatus=0` 単独は **未起動** と
> **単発起動完了後の終了状態** の **2 解釈** がある。区別には
> `launchctl print` の `runs` field + log + output file の **3 点同時確認** が必須。

本 CLI 全 report 出力 (text / json / markdown) に必ず含める warning:

```
WARNING: launchctl list PID=- / LastExitStatus=0 alone is INSUFFICIENT
to confirm successful execution. Always verify together:
(1) launchctl print 'runs' field;
(2) log content (logs/cron/weekly-snapshot.log);
(3) snapshot output file (data/snapshot/fire.staging.db + fire.develop.db)
    size > 0 and mtime after the expected run time.
```

auto memory feedback (= `feedback_launchd_diagnostic.md`) と整合。

## §6 baseline 仕様

baseline JSON example:

```json
{
  "f282_plist": {
    "path": "~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist",
    "mtime": 1778593597,
    "size": 1772
  },
  "db": {
    "data/fire.db": {"mtime": 1778570244, "size": 371081216},
    "data/fire.develop.db": {"mtime": 1778569903, "size": 371081216},
    "data/fire.staging.db": {"mtime": 1778579122, "size": 4804063232}
  }
}
```

### baseline 未指定時の挙動

- F282_PLIST_MTIME_BASELINE_MATCH → SKIP
- DB_BASELINE_UNCHANGED → SKIP
- stat readable check は PASS
- report に "baseline 未指定のため不変比較は SKIP" と注釈

### baseline path 解決

baseline JSON 内 key:
- 絶対 path はそのまま
- 相対 path (= `data/fire.db`) は `repo_root` から解決

## §7 snapshot integrity 設計

`SNAPSHOT_INTEGRITY_CHECK_OK` のみ `sqlite3.connect` を使用:

```python
conn = sqlite3.connect(f"file:{snapshot_path}?mode=ro", uri=True)
conn.execute("PRAGMA query_only=ON")
row = conn.execute("PRAGMA integrity_check").fetchone()
# row[0] == "ok" なら PASS
conn.close()
```

**重要**:
- 接続対象は `data/snapshot/fire.staging.db` と `data/snapshot/fire.develop.db` のみ
- `data/fire.db` / `data/fire.develop.db` / `data/fire.staging.db` (= production 3 環境)
  には **絶対接続しない** (= test 11 で機械的に verify)
- URI mode=ro + PRAGMA query_only=ON で write 不可能化

## §8 safety 設計

| 項目 | 実装 |
|---|---|
| subprocess.run(launchctl) | 0 |
| F282 runner import | 0 (= test 16 で確認) |
| VACUUM SQL 発行 | 0 (= source 内 "VACUUM" 大文字 0、test 17) |
| production / develop / staging DB sqlite 接続 | 0 (= test 11) |
| snapshot DB 接続 | read-only URI のみ (= test 10) |
| token / secret / channel_token 読出 | 0 (= test 18) |
| LINE API call | 0 |
| plist 配置 / 変更 | 0 |
| env 全体読出 | 0 |
| launchctl load / unload / kickstart | 0 |

## §9 pytest 結果

新規 test file: `tests/scripts/jobs/test_run_f282_post_run_inspection_report.py`
- **21 test 全 PASS** (= HQ 起票文書 20 必須 + 補助 1)
- pytest collected: 4105 → **4126** (= +21)
- 実行時間: 0.05s
- 既存 regression 影響: 0

## §10 5/16 03:00 での使い方

```bash
# Step 1: launchctl print 結果を保存
launchctl print gui/$(id -u)/jp.fire.weekly-snapshot \
  > /tmp/f282_print_2026-05-16.txt

# Step 2: baseline JSON 作成
cat > /tmp/f282_baseline_2026-05-16.json <<'JSONEOF'
{
  "f282_plist": {
    "path": "~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist",
    "mtime": 1778593597,
    "size": 1772
  },
  "db": {
    "data/fire.db": {"mtime": 1778570244, "size": 371081216},
    "data/fire.develop.db": {"mtime": 1778569903, "size": 371081216},
    "data/fire.staging.db": {"mtime": 1778579122, "size": 4804063232}
  }
}
JSONEOF

# Step 3: post-run inspection report
.venv/bin/python -m scripts.jobs.run_f282_post_run_inspection_report \
  --launchctl-print-file /tmp/f282_print_2026-05-16.txt \
  --baseline-json /tmp/f282_baseline_2026-05-16.json \
  --markdown reports/f282/post_run_2026-05-16.md \
  --expected-run-date 2026-05-16
```

期待結果 (= F282 試走成功時):
- verdict = GO
- LAUNCHCTL_PRINT_HAS_RUN_EVIDENCE = PASS (= runs >= 1)
- WEEKLY_SNAPSHOT_LOG_EXISTS = PASS
- LOG_CONTAINS_RUN_WINDOW_EVIDENCE = PASS (= "F282 snapshot OK")
- SNAPSHOT_FILES_EXIST + NON_ZERO + MTIME_AFTER + INTEGRITY = 全 PASS
- DB_BASELINE_UNCHANGED = PASS

## §11 Wave 41 への接続

F282 試走 GO 判定 (= 5/19) 後 Wave 41 に進む。
本 CLI は F282 専用だが、Wave 41 で DATA-R3 daily refresh 用に拡張する
派生 CLI を別 wave で起票可。

Wave 40.7 readiness CLI の `--phase post-f282` と本 CLI を同日 5/16 03:00 で
両方実行することで、機械的判定と人手最終チェックの 2 段確認を実現。

## §12 制約

| 項目 | 結果 |
|---|---|
| F282 手動実行 | 0 |
| launchctl コマンド呼出 | 0 |
| launchctl load / unload / kickstart | 0 |
| DB write | 0 |
| production / develop / staging DB sqlite 接続 | 0 |
| LINE 送信 / LINE API | 0 |
| token / secret / channel_token 値読出 | 0 |
| env 全体読出 | 0 |
| plist 配置 / 変更 | 0 |
| VACUUM SQL 発行 | 0 |
| workflow 変更 / --no-verify / TODO Excel | 0 |
| F282 plist mtime/size 変化 | 0 (= W40.8 作業中検証) |
| 3 環境 DB mtime/size 変化 | 0 |

---

## 関連リンク

- [[FIRE_production_v0_launch_plan_2026-05-13|v0 Launch Plan v1.0]]
- [[Production_v0_readiness_check_cli_2026-05-14|W40.7 readiness CLI]]
- [[F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[F282_temporary_launchd_smoke_2026-05-13|F282 temporary smoke (= W39-temp)]]
- [[../02_todo/FIRE_CODEX_R1_WAVE40_8_plan|Wave 40.8 plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE40_8_results|Wave 40.8 results]]
- 実装: `~/fire/scripts/jobs/run_f282_post_run_inspection_report.py`
- test: `~/fire/tests/scripts/jobs/test_run_f282_post_run_inspection_report.py`
