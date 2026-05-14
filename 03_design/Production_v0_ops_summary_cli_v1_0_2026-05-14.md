---
id: Production-v0-ops-summary-cli-v1-0
phase: 本番 v0 Launch / Daily Command Center CLI
priority: 高
status: 実装 v1.0 (= Wave 44.6-pre、新規実装)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40.8 (= F282 post-run inspection CLI)
  - Wave 41-pre (= DATA-R3 freshness producer schema 1.0)
  - Wave 42-pre (= F062 no-send runner)
  - Wave 43-pre (= readiness CLI v1.1)
  - Wave 44-pre (= F282 baseline + drill pack)
  - Wave 44.5-pre (= cutover/token/rollback runbook v1.0 final)
  - Wave 44.5-post (= adversarial audit)
  - Wave 51 (= readiness CLI v1.2 CRITICAL fix)
related:
  - 03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14.md
  - 03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md
  - 03_design/F282_baseline_capture_and_post_run_drill_2026-05-14.md
  - 03_design/F286_DATA_R3_wave41_pre_runner_enhancement_2026-05-14.md
chapter: production-v0 / Daily Command Center
---

# Production v0 Ops Summary CLI v1.0 — Wave 44.6-pre 新規実装

最終更新: 2026-05-13

## §1 目的

Production v0 運用開始後、**毎朝 1 コマンド** で FIRE の状態を確認できる
read-only Daily Command Center を提供。

統合対象 5 source:
1. F282 post-run report (= W40.8 出力 / W44-pre drill)
2. DATA-R3 freshness report (= W41-pre schema 1.0)
3. F062 preview / no-send artifact (= W42-pre 出力)
4. Production v0 readiness CLI v1.2 JSON (= W51)
5. 04_daily GO/NO-GO checklist (= W44.5-pre template)

判定 4 段階 (= GO / NO-GO / HOLD / UNKNOWN) を機械的に出力、
phase 7 種 (= readiness CLI v1.2 phase + 新規 daily) に対応。

## §2 W44.6-pre 構成 (= 本線完結)

| lane | 内容 |
|---|---|
| L5 本線 | CLI v1.0 実装 + 73 test + docs + 不干渉確認 + 報告 |

Codex lane 投入: **0/8 = 0%**。理由は §11 で詳述。

## §3 CLI 仕様

### §3.1 引数

```
.venv/bin/python -m scripts.jobs.run_production_v0_ops_summary \
  --phase {pre-f282|post-f282|pre-wave41|pre-wave45|pre-wave52|pre-v0-launch|daily} \
  [--repo-root PATH]                     # default: ~/fire
  [--vault-root PATH]                    # default: ~/fire-vault
  [--date YYYY-MM-DD]                    # default: today JST
  [--readiness-json PATH]                # optional
  [--f282-report PATH]                   # optional
  [--data-r3-freshness-json PATH]        # optional (= /tmp/f286_data_r3_freshness.json default)
  [--f062-preview PATH]                  # optional (= /tmp/f062_morning_preview_*.txt glob default)
  [--json]                               # JSON to stdout
  [--markdown PATH]                      # write Markdown to PATH
  [--strict]                             # WARN/HOLD → NO-GO
```

### §3.2 phase 7 種

| phase | 想定タイミング | 期待される input |
|---|---|---|
| pre-f282 | 5/13-5/15 (= F282 試走前) | 全 SKIP、UNKNOWN verdict |
| post-f282 | 5/16 03:00 drill 時 | F282 report 期待、DATA-R3/F062 SKIP |
| pre-wave41 | 5/19 GO 判定 〜 W41 着手前 | F282 報告必須、DATA-R3/F062 SKIP |
| pre-wave45 | W41 完了 〜 W45 着手前 | F282 + DATA-R3、F062 SKIP |
| pre-wave52 | W45 完了 〜 W52 着手前 | F282 + DATA-R3 + F062 (no-send trial) |
| pre-v0-launch | 6/8 月 final strict | 全 PASS 必須、readiness CLI v1.2 GO 期待 |
| **daily** | **6/9 火 D-Day 以降 + dual-run** | **全 input 期待、毎朝の運用判定** |

### §3.3 verdict semantics (= 4 段階)

| verdict | 意味 | exit code |
|---|---|---|
| **GO** | 全 PASS、送信可能 | 0 |
| **HOLD** | WARN あり (= 観察、本日 action 不要) | 0 |
| **NO-GO** | FAIL あり (= 送信不可、要対応) | 1 |
| **UNKNOWN** | 判定不能 (= 全 SKIP、入力不足) | 1 |

strict mode (= `--strict`) では WARN → NO-GO に昇格 (= HOLD 経由なし)。

### §3.4 verdict 集約 algorithm

```python
def aggregate_verdict(results, strict=False):
    if any FAIL: return NO-GO
    if any PASS:
        if any WARN:
            return NO-GO if strict else HOLD
        return GO
    # All SKIP
    return UNKNOWN
```

### §3.5 出力 3 種

#### text (default、stdout)
- ヘッダー (= CLI version / phase / date / generated_at)
- Overall verdict
- 各 check の status / category / check_id / message
- Blocking issues (= FAIL 列挙、next_action 含む)
- Warnings (= WARN 列挙)
- Latest artifacts (= path 一覧)
- Human next actions
- Safety flags (= 全 false)

#### JSON (= `--json`)
```json
{
  "schema_version": "1.0",
  "cli_version": "1.0.1",
  "generated_at": "...",
  "phase": "...",
  "base_date": "YYYY-MM-DD",
  "strict": false,
  "overall_verdict": "GO|NO-GO|HOLD|UNKNOWN",
  "checks": [{"check_id": "...", "category": "...", "status": "...", ...}],
  "artifacts": [{"category": "...", "path": "...", "status": "..."}],
  "blocking_issues": [{"check_id": "...", "message": "...", "next_action": "..."}],
  "warnings": [...],
  "next_actions": ["[CAT] action...", ...],
  "safety_flags": {"db_write": false, ...}
}
```

#### Markdown (= `--markdown PATH`)
- # FIRE Production v0 Ops Summary — DATE
- Overall verdict (= bold)
- Checks 表
- Blocking issues 箇条書き + next
- Warnings
- Latest artifacts
- Human next actions
- Safety flags

## §4 source 別 check 仕様

### §4.1 F282 post-run report (= `check_f282_post_run_report`)

- path: `reports/f282/post_run_YYYY-MM-DD.md` (= W40.8 CLI `--markdown` arg 規約)
- glob: `post_run_*.md` → latest by mtime
- phase 別:
  - pre-f282 → SKIP
  - post-f282 missing → WARN (= drill 進行中許容)
  - pre-wave41 以降 missing → FAIL

### §4.2 DATA-R3 freshness (= `check_data_r3_freshness`)

- path: `/tmp/f286_data_r3_freshness.json` (= W41-pre 出力 path)
- schema_version: "1.0"
- verdict: OK / STALE / MISSING / FAILED / その他
- phase 別:
  - pre-f282 / post-f282 / pre-wave41 → SKIP
  - pre-wave45 以降 missing → FAIL
  - verdict STALE/MISSING/FAILED → FAIL
  - verdict OK → PASS
  - 不明 verdict → WARN

### §4.3 F062 preview (= `check_f062_preview`)

- path: `/tmp/f062_morning_preview_*.txt` glob (= W42-pre 出力 path)
- phase 別:
  - pre-wave45 以前 → SKIP
  - pre-wave52 missing → WARN (= F062 trial 未稼働許容)
  - pre-v0-launch / daily missing → FAIL

### §4.4 readiness CLI v1.2 JSON (= `check_readiness_cli_json`)

- input: `--readiness-json PATH` (= optional)
- 取り込み: cli_version / verdict / missing_markers / fail_count / warn_count / phase
- verdict GO → PASS
- verdict NO-GO → FAIL + missing_markers が message に含まれる
- 不明 verdict → WARN
- path 未指定 → SKIP

### §4.5 GO/NO-GO checklist (= `check_go_no_go_checklist`)

- dir: `<vault_root>/04_daily/`
- 候補 4 file:
  - `{date}_f282_post_run.md` (= W44-pre template)
  - `{date}_v0_final_strict_check.md` (= W44.5-pre template, 6/8 用)
  - `{date}_v0_d_day_morning_check.md` (= W44.5-pre template, 6/9 用)
  - `{date}_v0_daily_ops.md` (= 6/10 以降の dual-run / 定常運用)
- 1 件以上存在 → PASS
- 全 missing → WARN (= 記入推奨)

## §5 safety_flags (= 常に false、design 保証)

```python
SAFETY_FLAGS = {
    "db_write": False,
    "db_sqlite_connect": False,
    "line_send": False,
    "token_access": False,
    "launchctl_call": False,
    "plist_modified": False,
    "cron_modified": False,
    "vacuum_executed": False,
}
```

JSON output / text output / Markdown output 全てに表示。

## §6 output path guard (= 危険 path への書き込み refuse)

`--markdown` path が以下 segment を含む場合、refuse + exit 2:
- `/data/`
- `/.git/`
- `/.github/`
- `/LaunchAgents/`
- `/.fire_secrets/`

`is_safe_output_path(path)` 関数で実装、unit test で 5 種類とも検証。

## §7 v0 運用での使い方 (= phase 別)

### 5/16 03:00 (= post-f282 drill)

```bash
.venv/bin/python -m scripts.jobs.run_production_v0_ops_summary \
  --phase post-f282 --date 2026-05-16 --json \
  --markdown reports/ops/v0_summary_2026-05-16.md
```

- F282 post-run report 期待 PASS
- DATA-R3 / F062 SKIP (= まだ稼働前)
- GO/NO-GO checklist (= 04_daily/2026-05-16_f282_post_run.md) で WARN/PASS

### 5/19 (= F282 GO 判定 → W41 着手前)

```bash
.venv/bin/python -m scripts.jobs.run_production_v0_ops_summary \
  --phase pre-wave41 --date 2026-05-19 --strict
```

- F282 report 必須 (= 5/16 drill commit 済前提)
- DATA-R3 / F062 SKIP

### 6/8 月 (= final strict check 日)

```bash
.venv/bin/python -m scripts.jobs.run_production_v0_readiness_check \
  --phase pre-v0-launch --strict --json > /tmp/readiness_2026-06-08.json

.venv/bin/python -m scripts.jobs.run_production_v0_ops_summary \
  --phase pre-v0-launch --date 2026-06-08 --strict \
  --readiness-json /tmp/readiness_2026-06-08.json \
  --markdown reports/ops/v0_summary_2026-06-08.md
```

- 全 5 source PASS 必須 (= 1 件でも WARN/FAIL → NO-GO で延期)
- readiness CLI v1.2 missing_markers が見える化される

### 6/9 火 D-Day morning (= 08:30)

```bash
.venv/bin/python -m scripts.jobs.run_production_v0_readiness_check \
  --phase pre-v0-launch --strict --json > /tmp/readiness_2026-06-09.json

.venv/bin/python -m scripts.jobs.run_production_v0_ops_summary \
  --phase pre-v0-launch --date 2026-06-09 --strict \
  --readiness-json /tmp/readiness_2026-06-09.json \
  --markdown reports/ops/v0_summary_2026-06-09_morning.md
```

D-Day 当日朝の最終確認、08:30 までに GO 必須。

### 6/9 火 D-Day 09:30 以降 (= daily phase)

```bash
.venv/bin/python -m scripts.jobs.run_production_v0_ops_summary \
  --phase daily --date 2026-06-09 \
  --markdown reports/ops/v0_summary_2026-06-09_eod.md
```

D-Day 完了確認 + 翌朝の準備状況。

### 6/10 以降 (= dual-run + 定常運用、毎朝)

```bash
.venv/bin/python -m scripts.jobs.run_production_v0_ops_summary \
  --phase daily --date $(date +%F) \
  --markdown reports/ops/v0_summary_$(date +%F).md
```

毎朝 09:30 頃に 1 コマンドで GO/NO-GO/HOLD/UNKNOWN を確認、Markdown を vault commit。

## §8 test 結果 (= 73 test、CLI 単独)

| class | test 件数 | 内容 |
|---|---|---|
| TestCliVersionAndSchema | 4 | CLI_VERSION / SCHEMA_VERSION / phase / JSON 主要 key |
| TestF282PostRunReport | 7 | phase 別 SKIP/WARN/FAIL/PASS、custom_path |
| TestDataR3Freshness | 8 | phase 別、OK/STALE/MISSING/FAILED/UNKNOWN/parse error |
| TestF062Preview | 5 | phase 別 SKIP/WARN/FAIL、custom present |
| TestReadinessCliJson | 6 | SKIP/missing/GO/NO-GO with missing_markers/UNKNOWN/parse error |
| TestGoNoGoChecklist | 5 | pre-f282 SKIP / phase 別 4 file pattern |
| TestVerdictAggregation | 7 | PASS / SKIP / FAIL / WARN / strict / fail overrides |
| TestOutputFormats | 4 | text / JSON / Markdown file / safety_flags |
| TestStrictMode | 2 | WARN → NO-GO / HOLD exit 0 |
| TestOutputPathGuard | 4 | data/ / .git/ / .fire_secrets/ / LaunchAgents/ refuse + is_safe_output_path |
| TestMainIntegration | 4 | daily full GO / DATA-R3 STALE blocks / missing_markers in blocking / pre-f282 UNKNOWN |
| TestAstSafety | 7 | AST: sqlite3 / subprocess / linebot / VACUUM SQL / --no-verify / os.environ / LINE call |
| TestF282PlistBaselineUnchanged | 2 | F282 plist / 3 DB mtime/size 不変 |
| TestExitCode | 4 | GO=0 / HOLD=0 / NO-GO=1 / UNKNOWN=1 |
| TestPhaseGating | 3 | PHASE_ORDER_FOR_GATING / DATA-R3 / F062 phase 別 SKIP |

合計 **73 test、0.10s、全 PASS**。
pytest 全体 collected: 4229 → **4302** (= +73)。

## §9 safety 確認 (= 本 wave 自体)

| 項目 | 結果 |
|---|---|
| 実 file 変更 | 5 file (= CLI + tests + v1.0 設計 doc + W44.6-pre plan + W44.6-pre results) |
| CLI sqlite3 import (AST) | 0 ✓ |
| CLI subprocess import (AST) | 0 ✓ |
| CLI linebot / line_bot_sdk import (AST) | 0 ✓ |
| CLI VACUUM SQL literal in Call arg (AST) | 0 ✓ (docstring 言及は許容) |
| CLI --no-verify in Call arg (AST) | 0 ✓ |
| CLI os.environ direct iteration (AST) | 0 ✓ |
| CLI LINE API call (AST) | 0 ✓ |
| DB write | 0 |
| production / develop / staging DB SQLite 接続 | 0 |
| LINE 送信 / API call | 0 |
| token / channel_token / secret / .env 値参照 | 0 (= 個別 key の os.environ.get もしない) |
| launchctl 実行 | 0 |
| plist 配置 / 変更 | 0 |
| cron / crontab 変更 | 0 |
| F282 試走干渉 | 0 |
| F282 manual run | 0 |
| VACUUM / VACUUM INTO | 0 |
| workflow / --no-verify / sudo / rm -rf / git push | 0 |
| TODO Excel 更新 | 0 |
| 楽天証券操作 / 自動発注 / Computer Use | 0 |
| Codex 直接 commit | 0 |
| output path guard 動作確認 | ✓ (= 5 forbidden segments で refuse + exit 2) |

## §10 F282 不干渉確認

baseline (= W44.6-pre 開始、19:23 JST):
- F282 plist mtime=1778593597 / size=1772
- fire.db mtime=1778570244 / size=371081216
- fire.develop.db mtime=1778569903 / size=371081216
- fire.staging.db mtime=1778579122 / size=4804063232
- pytest collected 4229

完了時 (= 19:28 JST):
- F282 plist mtime=1778593597 / size=1772 **完全一致 ✓**
- 3 DB mtime / size **完全一致 ✓**
- F282 state = not running 維持、5/16 02:00 試走待機 ✓
- daily-refresh / morning-advisory plist 未配置維持 ✓
- pytest collected **4302** (= +73 想定通り)

## §11 Codex lane 0/8 = 0% の理由 + HQ 報告整合性

HQ 補足では「8 lane 第一候補」だが、本 wave も **0/8 で本線完結**。
W51 で確立した「CRITICAL fix / 局所新規実装は本線完結」パターンを継承。

### 理由

1. **user spec が完全 prescriptive**:
   - CLI 引数 11 個全て列挙
   - 入力 source 5 種類仕様明示
   - 出力 3 format (text / JSON / Markdown) 構造明示
   - safety_flags 8 keys 明示
   - test 要件 (= phase × input combination + AST + path guard) 明示
   - phase 別 verdict 期待値明示

2. **実装範囲が局所**:
   - 1 新規 CLI file + 1 新規 test file のみ
   - 既存 CLI / 既存 module への変更 0

3. **test coverage 包括的**:
   - 15 class / 73 test で全 phase × 全 input × 全出力 format × AST safety を網羅
   - W43-pre / W51 で確立した AST safety パターン継承

4. **入力 source は既存 wave で確定済**:
   - W40.8 F282 post-run report 仕様 → 既設計
   - W41-pre DATA-R3 freshness schema → 既設計
   - W42-pre F062 preview path → 既設計
   - W51 readiness CLI v1.2 JSON 構造 → 既設計
   - W44.5-pre 04_daily checklist 命名 → 既設計
   - **Codex で「新たな観点」を獲得する余地が小さい**

5. **W51 precedent**:
   - W51 (= CRITICAL fix) で 0/8 = 0% は HQ に accepted
   - 同様のパターン (= 局所実装 + 包括 test) で W44.6-pre も完結可

### 代替手段 (= Codex 不投入の補完)

- AST safety test 7 件 (= TestAstSafety class) で Lane F (source safety audit) 同等
- 73 test 包括カバレッジで Lane E (pytest design) 同等
- W44.6-post として adversarial audit 8 lane を将来別 wave で投入する選択肢 (= W44.5-post / W44.5-pre パターン)

### KPI への反映 (= §12)

- Codex 稼働率 0/8 = 0% (= HQ 補足からの逸脱、本 doc §11 で明記)
- 代替: 本線 73 test + AST 7 safety で機能等価カバレッジ

## §12 6 KPI

- **Codex 稼働率**: **0/8 = 0%** (= §11 理由参照、HQ 補足からの逸脱明記)
- **本線短縮率**: 該当なし
- **採用率**: 該当なし
- **差戻率**: 0 (= 内製 test 1 件更新 = VACUUM AST 検出を Call arg 限定に refine で吸収、新規実装自体は差戻 0)
- **Integrator 負荷**: 中-高 (= CLI 約 580 行 / test 約 530 行 / docs 1 file 新規 + W44.6-pre plan / results)
- **安全事故**: 0 ✓

### KPI 比較

| wave | Codex 稼働率 | wave 性質 |
|---|---|---|
| W43-pre | 8/8 = 100% | CLI v1.1 新規実装 (= 観点分割可能) |
| W44-pre | 8/8 = 100% | F282 baseline helper 新規実装 |
| W44.5-pre | 0/8 = 0% | runbook 集約整理 |
| W44.5-post | 6/8 = 75% | adversarial audit (= F/H Codex 拒否) |
| W51 | 0/8 = 0% | CRITICAL fix (= 所在明確 + 局所) |
| **W44.6-pre (本 wave)** | **0/8 = 0%** | **新規 CLI 実装 (= spec prescriptive + 局所)** |

W43-pre / W44-pre と異なり、W44.6-pre は user spec が完全に prescriptive
(= 11 引数 / 5 入力 / 3 出力 / 7 phase / 8 safety 全列挙) のため
Codex 並列の design exploration 価値が小さく、本線完結が効率的。

## §13 残課題

W44.5-post / W44.5-pre / W51 から継続:
- (Wave 52) HQ_APPROVE_SEND_MARKER 生成手順確定
- (Wave 53) wrapper script 配置 + permission 確認
- (Wave 52) WRITE_ALLOWED_DB_LABELS_PRODUCTION_V0 test 関数定義
- (別 wave、低) W43-pre CLI v1.1 doc 「HQ docs Monday」整理
- (別 wave、低) launch plan §3 Phase D / E 1 日重複

W44.6-pre で開ける道:
- (将来 W44.6-post) adversarial audit lane 投入 (= HQ 判断)
- (将来) Ops Summary CLI に過去 N 日の trend 表示 (= dual-run monitoring 強化)
- (将来) Ops Summary JSON を別 dashboard に消費させる integration

---

## 関連リンク

- [[Production_v0_readiness_check_cli_v1_2_2026-05-14|W51 readiness CLI v1.2]]
- [[Production_v0_cutover_rollback_token_runbook_2026-05-14|W44.5-pre cutover runbook v1.0 final (audited)]]
- [[F282_baseline_capture_and_post_run_drill_2026-05-14|W44-pre F282 baseline + drill]]
- [[F286_DATA_R3_wave41_pre_runner_enhancement_2026-05-14|W41-pre DATA-R3 freshness producer]]
- [[F062_wave42_pre_no_send_runner_enhancement_2026-05-14|W42-pre F062 freshness consumer]]
- [[../02_todo/FIRE_CODEX_R1_WAVE44_6_PRE_plan|W44.6-pre plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE44_6_PRE_results|W44.6-pre results]]
- 実装: `~/fire/scripts/jobs/run_production_v0_ops_summary.py`
- test: `~/fire/tests/scripts/jobs/test_run_production_v0_ops_summary.py`
