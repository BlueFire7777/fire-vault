---
id: Production-v0-readiness-check-cli
phase: 本番 v0 Launch / Phase D-E readiness CLI
priority: 高
status: 実装 v1.0 (= Wave 40.7、read-only CLI + 15 test + 10 章 doc)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40.5 (= 56 項目 checklist + HQ marker 7 段)
  - Wave 40.6 (= cutover/rollback/token runbook、D-Day 6/9 火曜確定)
related:
  - 03_design/F062_no_send_trial_checklist_and_v0_readiness_2026-05-14.md
  - 03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md
chapter: production-v0 / readiness check CLI
---

# Production v0 Readiness Check CLI — Wave 40.7

最終更新: 2026-05-13

## §1 目的

Production v0 D-Day (= 2026-06-09 火曜) 直前の readiness を **read-only**
で支援する CLI を実装。Wave 41 / Wave 45 / Wave 52 / Wave 53 の GO/NO-GO
判定で逐次使用。手動 checklist (= W40.5 56 項目) を機械的検証可能化。

- 実 file: `scripts/jobs/run_production_v0_readiness_check.py`
- 実 test: `tests/scripts/jobs/test_run_production_v0_readiness_check.py`
- 安全制約: DB write 0 / LINE 0 / token 値 0 / env 全体読 0 / launchctl 0

## §2 CLI 仕様

### 必須引数

- `--phase` (= choices 6 種)

### 任意引数

- `--json` (= JSON stdout)
- `--strict` (= WARN を NO-GO 扱い)
- `--repo-root PATH` (= default ~/fire)
- `--vault-root PATH` (= default ~/fire-vault)
- `--output-json PATH` (= JSON file 出力)

### exit code

- `0` = verdict GO (= 全 required check PASS)
- `1` = verdict NO-GO (= FAIL あり、または strict+WARN あり)
- `2` = argparse 失敗 / invalid phase / unsafe operation

## §3 phase 一覧

| phase | 想定使用日 | scope |
|---|---|---|
| `pre-f282` | 5/14-5/15 | F282 試走前 baseline |
| `post-f282` | 5/16 03:00 | F282 試走後確認 |
| `pre-wave41` | 5/19 | Wave 41 着手前 (= DATA-R3 plist 配置) |
| `pre-wave45` | 5/26 | Wave 45 着手前 (= F062 plist 配置) |
| `pre-wave52` | 6/7 | Wave 52 着手前 (= LINE token 投入) |
| `pre-v0-launch` | 6/8 月曜 | D-Day 6/9 火曜 前日最終確認 |

## §4 check 一覧 (= 19 check 関数)

| # | check_id | severity | scope |
|---|---|---|---|
| 1 | F282_PLIST_EXISTS | high | F282 plist 存在 |
| 2 | F282_PLIST_MTIME_UNCHANGED | high | F282 baseline 不変 |
| 3 | F282_NEXT_RUN_EXPECTED | high | Weekday=7/Hour=2/Minute=0 |
| 4 | F282_LOG_EXISTS | medium | post-f282 で PASS 期待 |
| 5 | F282_OUTPUT_SNAPSHOT_EXISTS | medium-high | snapshot file size > 0 |
| 6 | DB_PRODUCTION_STAT_READABLE | medium | fire.db stat |
| 7 | DB_DEVELOP_STAT_READABLE | medium | fire.develop.db stat |
| 8 | DB_STAGING_STAT_READABLE | medium | fire.staging.db stat |
| 9 | DAILY_REFRESH_PLIST_ABSENT_BEFORE_WAVE41 | high | phase 別 expectation |
| 10 | MORNING_ADVISORY_PLIST_ABSENT_BEFORE_WAVE45 | high | phase 別 expectation |
| 11 | TOKEN_FILE_NOT_REQUIRED_BEFORE_WAVE52 | high | token 値非読 |
| 12 | TOKEN_FILE_EXISTS_ONLY_AFTER_WAVE52 | high | chmod 600 + 存在 |
| 13 | NO_SEND_MODE_REQUIRED_BEFORE_V0 | high | ProgramArguments 検証 |
| 14 | HQ_MARKER_REQUIRED_FOR_WAVE41 | high | env / vault file 存在 |
| 15 | HQ_MARKER_REQUIRED_FOR_WAVE45 | high | 同上 |
| 16 | HQ_MARKER_REQUIRED_FOR_WAVE52 | high | 同上 |
| 17 | HQ_MARKER_REQUIRED_FOR_V0_LAUNCH | high | 同上 |
| 18 | D_DAY_DATE_IS_2026_06_09_TUESDAY | high | 静的曜日検証 |
| 19 | READINESS_CHECK_DATE_2026_06_08_MONDAY | medium | 月曜検証 |

→ phase によらず全 check 関数を実行、関数内で SKIP/PASS/WARN/FAIL 判定。

## §5 safety 設計

| 項目 | 実装 |
|---|---|
| token 値 read | 0 (= path / exists / size / mode のみ参照) |
| secret 値 read | 0 |
| env 全体読出 | 0 (= os.environ.get で個別 key の有無のみ) |
| DB write | 0 (= sqlite3.connect しない、stat のみ) |
| LINE API call | 0 (= line-bot-sdk import 0) |
| launchctl 呼出 | 0 (= subprocess 0、plistlib のみ) |
| plist 配置/変更 | 0 |
| --no-verify | 0 |
| TODO Excel | 0 |

### evidence dict の安全性

- path / exists / size / mtime / mode のみ含む
- token / secret / channel_token 値、plist 本文、log 内容を含めない
- JSON serializable

## §6 json schema

```
{
  "phase": "pre-v0-launch",
  "started_at": "2026-06-08T08:00:00+09:00",
  "strict": false,
  "verdict": "GO" | "NO-GO",
  "pass_count": int,
  "warn_count": int,
  "fail_count": int,
  "skip_count": int,
  "checks": [
    {
      "check_id": "F282_PLIST_EXISTS",
      "phase": "pre-v0-launch",
      "title": "F282 weekly-snapshot plist exists",
      "status": "PASS" | "WARN" | "FAIL" | "SKIP",
      "severity": "high" | "medium" | "low",
      "message": "...",
      "evidence": {
        "path": "...",
        "exists": true,
        "size": 1772,
        "mtime_iso": "..."
      },
      "next_action": null | "..."
    }
  ]
}
```

CheckResult dataclass 8 fields:
- check_id / phase / title / status / severity / message / evidence / next_action

dataclass `frozen=True` で不変、test で schema 安定を verify。

## §7 pytest 結果

新規 test file: `tests/scripts/jobs/test_run_production_v0_readiness_check.py`
- **15 test 全 PASS** (= HQ 起票文書 15 項目)
- pytest collected: 4090 → **4105** (= +15、想定通り)
- 実行時間: 0.06s
- 既存 regression 影響: 0

### 15 test カバレッジ

1. --phase 必須 (= ArgumentError)
2. invalid phase exit 2
3. --json 有効 JSON 返却
4. pre-f282 で daily/morning plist 不在 PASS
5. pre-wave41 marker 未確認 WARN/FAIL
6. pre-wave45 marker 未確認 WARN/FAIL
7. pre-wave52 で token 値非読 (= 模擬 secret 漏出 0)
8. pre-v0-launch で D-Day Tuesday 検証
9. DB stat 確認で write 0 (= mtime before/after 一致)
10. CheckResult schema 安定 (= dataclass fields 固定)
11. strict mode で WARN → NO-GO
12. default text に GO/NO-GO summary
13. unsafe option 不在 (= write/load/unload/kickstart/send 等)
14. launchctl call 0 (= subprocess.run/Popen/check_output/call 全 mock)
15. token/secret 値が出力に含まれない

## §8 Wave 41 / 45 / 52 / 53 での使い方

### Wave 41 (= DATA-R3 plist 配置)

```
# 着手前 (= 5/19 GO 判定後)
python -m scripts.jobs.run_production_v0_readiness_check --phase pre-wave41
# → HQ_APPROVE_LAUNCHD_DAILY 取得確認 + F282 不変確認
```

### Wave 45 (= F062 morning advisory plist 配置)

```
# 着手前 (= 5/26 想定)
python -m scripts.jobs.run_production_v0_readiness_check --phase pre-wave45
# → HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + DATA-R3 完了確認
```

### Wave 52 (= LINE token 投入)

```
# 着手前 (= 6/7 想定)
python -m scripts.jobs.run_production_v0_readiness_check --phase pre-wave52
# → token 値非読、配置前 path 確認
```

### Wave 53 (= D-Day)

```
# 前日 (= 6/8 月曜) 最終確認
python -m scripts.jobs.run_production_v0_readiness_check --phase pre-v0-launch --strict
# strict mode で全 WARN を NO-GO 扱い、最も厳格に検証
```

### post-f282 (= 5/16 03:00)

```
# F282 試走実行後の整合性確認
python -m scripts.jobs.run_production_v0_readiness_check --phase post-f282
# → F282_LOG_EXISTS / F282_OUTPUT_SNAPSHOT_EXISTS 等を verify
```

## §9 制約

| 項目 | 結果 |
|---|---|
| 完全 read-only | ✓ |
| launchctl 直接呼出 | 0 (= subprocess 全 mock + production code 0) |
| F282 試走干渉 | 0 (= plist mtime 完全不変) |
| DB write | 0 (= stat のみ) |
| LINE 0 / token 0 / env 全体読 0 | ✓ |
| 既存 4090 test 影響 | 0 (= 新 15 test 追加で 4105 collected) |

## §10 次課題

| # | 項目 | 対応 wave |
|---|---|---|
| 1 | HQ marker file (~/fire-vault/02_todo/hq_markers/) 運用整理 | Wave 41 起票時 |
| 2 | launchctl print 出力 file の解析機能追加 | 別 wave |
| 3 | check 自動実行 cron 化 | D-Day +1 月運用後 |
| 4 | check 結果の log.md 自動追記機能 | 別 wave |
| 5 | production / develop DB の write detection 強化 | Wave 52 着手時 |

---

## 関連リンク

- [[FIRE_production_v0_launch_plan_2026-05-13|v0 Launch Plan v1.0]]
- [[F062_no_send_trial_checklist_and_v0_readiness_2026-05-14|W40.5 readiness 設計]]
- [[Production_v0_cutover_rollback_token_runbook_2026-05-14|W40.6 cutover/rollback/token runbook]]
- [[../02_todo/FIRE_CODEX_R1_WAVE40_7_plan|Wave 40.7 plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE40_7_results|Wave 40.7 results]]
- 実装: `~/fire/scripts/jobs/run_production_v0_readiness_check.py`
- test: `~/fire/tests/scripts/jobs/test_run_production_v0_readiness_check.py`
