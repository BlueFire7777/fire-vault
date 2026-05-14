---
id: F286-AFTER-R1-read-only-runner-design
phase: 本番 v0 後拡張 / Wave 54-pre / AFTER-R1 read-only scaffold
priority: 高
status: 実装 v1.0 (= Wave 54-pre、scaffold + tests + docs)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 35-pre (= F286_AFTER_R1_night_paper_live_batch design v1.0、既存)
  - Wave 51 (= readiness CLI v1.2)
  - Wave 44.6-post (= Ops Summary CLI v1.0.1)
  - Wave 52-post (= wrapper preview v1.0.1)
  - Wave 52.5-pre (= consistency cleanup)
related:
  - 03_design/F286_AFTER_R1_night_paper_live_batch_2026-05-12.md
  - 03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md
chapter: post-v0 / AFTER-R1 read-only scaffold
---

# F286-AFTER-R1 Read-Only Runner Scaffold v1.0 — Wave 54-pre

最終更新: 2026-05-13

## §1 目的

Production v0 後拡張である F286-AFTER-R1 (= After-Close Night Paper Live Batch)
の **read-only runner scaffold** を前倒し整備。

v0 開始前 (= 6/9 D-Day 前) なので **実 task 実行 / DB write / Paper Live 実行 /
LINE 送信 / token / API call / cron 登録は 0**。
v0 安定稼働後に各 task 実装へ安全に移れる contract / schema / tests を固定。

## §2 v0 後にやる理由

| 観点 | v0 中に必要か | 理由 |
|---|---|---|
| Paper Live nightly summary | **不要** | advisory_decisions row が蓄積されないと無意味 |
| Replay simulation | **不要** | Paper Live 動作確認後でないと検証 logic 不確実 |
| Simulation (= parameter swap) | **不要** | Replay 動作確認後 |
| Lane evaluation | **不要** | 最低 20 営業日蓄積後 (= Stage 2 Paper Live 昇格条件と同じ) |
| Win/Loss pattern extraction | **不要** | Lane evaluation + Replay 完了後 |
| Nightly summary report | **不要** | 他 task 出力統合先、並行進行 |

→ **v0 前に AFTER-R1 を動かす必要は無い**。ただし、設計 / runner contract /
test pattern / safety design を **v0 前に固める** ことで:
1. v0 開始後の Wave 55+ で迷わず実装着手
2. v0 安定中に scaffold だけ存在しても安全 (= 実行されない)
3. 既存 v0 path (= F282 / DATA-R3 / F062 / readiness / Ops Summary / wrapper) に
   触らず、独立 module として開発可能

## §3 v0 前にやらないこと

- 実 Paper Live tick replay
- DB write (= production / develop / staging いずれも)
- LINE / token / API call
- cron / launchd 登録
- 楽天証券操作 / 自動発注
- 過去 historical price 取得 (= F100 既存 launchd は触らない)

## §4 CLI 仕様 (= scaffold v1.0)

### §4.1 引数

```
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  [--base-date YYYY-MM-DD]          # default: today JST
  [--repo-root PATH]                 # default: ~/fire
  [--vault-root PATH]                # default: ~/fire-vault
  [--task TASK]                      # default: all
  [--dry-run]                        # default: True (= 本 wave 強制)
  [--read-only]                      # default: True (= 本 wave 強制)
  [--output-json PATH]               # optional, guarded
  [--markdown PATH]                  # optional, guarded
  [--json]                           # JSON to stdout
  [--strict]                         # verdict tightening (本 wave は preview only)
```

### §4.2 --task choices (= 6 + all)

| choice | meaning |
|---|---|
| `paper-live` | Paper Live nightly summary |
| `replay` | Replay simulation (= 過去 N 日) |
| `simulation` | Simulation (= 仮想 parameter swap) |
| `lane-eval` | Lane evaluation (= win rate / drawdown / sample size) + sub_items: lane promotion/demotion candidates |
| `pattern` | Win/Loss pattern extraction + sub_items: ML feature export |
| `report` | Nightly summary report + sub_items: advisory improvement / dashboard summary |
| `all` | 上記 6 task 全て |

### §4.3 verdict / exit code

- **DESIGN_PREVIEW** → exit 0 (= scaffold preview として常に提供)
- **NOT_READY** → exit 0 (本 wave では発生しない、将来 implementation phase で使う可能性)

scaffold CLI は preview-only なので **常に exit 0**。

## §5 task registry (= 6 task の v0 後実装計画)

各 task に以下 field を定義 (= scaffold contract):

```python
TASK_REGISTRY[task_id] = {
    "name": "...",
    "description": "...",
    "status": "planned",
    "skipped_reason": "v0 pending; W54-pre scaffold design only",
    "v0_dependency": "...",
    "planned_outputs": [...],
    "expected_wave": "Wave NN+",
    "depends_on_modules": [...],
    "sub_items": [...],
}
```

### §5.1 6 task の expected wave

| task | expected wave | v0 dependency |
|---|---|---|
| paper-live | Wave 55+ | v0 dual-run 完了 (= 最低 1 営業日) |
| report | Wave 55+ (並行) | 他 task の出力統合 |
| replay | Wave 56+ | Paper Live 安定後 |
| simulation | Wave 57+ | Replay 動作確認後 |
| lane-eval | Wave 58+ | 20 営業日蓄積後 |
| pattern | Wave 60+ | Lane eval + 6 ヶ月 Replay 完了 |

### §5.2 sub_items 明示

- `lane-eval` に lane promotion/demotion candidates 含む (= HQ 承認待ち提案生成)
- `pattern` に ML feature export 含む (= 仮 parquet 出力、要再設計)
- `report` に advisory improvement material + DASH-R1 dashboard summary 含む

## §6 output schema

### §6.1 JSON (= --json / --output-json)

```json
{
  "schema_version": "1.0",
  "cli_version": "1.0",
  "generated_at": "...",
  "base_date": "YYYY-MM-DD",
  "mode": ["dry_run", "read_only"],
  "strict": false,
  "verdict": "DESIGN_PREVIEW",
  "tasks": [{"task_id": "...", "status": "planned", ...}],
  "skipped_tasks": [...],
  "selected_tasks": [...],
  "planned_outputs": [...],
  "v0_dependencies": [{"task_id": "...", "v0_dependency": "...", "expected_wave": "..."}],
  "next_steps": [...],
  "safety_flags": {
    "db_write": false,
    "db_sqlite_connect": false,
    "line_send": false,
    "token_access": false,
    "api_call": false,
    "launchctl_call": false,
    "plist_modified": false,
    "cron_modified": false,
    "vacuum_executed": false,
    "order_automation": false,
    "production_data_modified": false,
    "paper_live_executed": false
  },
  "expected_upstream_versions": {
    "readiness_cli": "1.2",
    "ops_summary": "1.0.1",
    "wrapper_preview": "1.0.1"
  }
}
```

### §6.2 Markdown (= --markdown)

- `# F286-AFTER-R1 Nightly Batch Preview — DATE`
- Preview readiness verdict (= bold)
- Tasks 表 (= selected / task_id / status / name / expected_wave)
- Task details (selected only)
- Planned outputs aggregate
- V0 dependencies
- Next steps
- Safety flags (= 12 keys 全 false)
- Expected upstream versions

## §7 safety design

### §7.1 safety_flags (= 12 keys 全 false design 保証)

W44.6-post / W52-post の 8 / 10 keys から拡張:
- (W52-post 既存 10) db_write / db_sqlite_connect / line_send / token_access /
  launchctl_call / plist_modified / cron_modified / vacuum_executed /
  production_send_executed / real_hq_marker_created
- (W54-pre 新規 2) api_call / order_automation / production_data_modified /
  paper_live_executed

実 task が動かないので **常に false**。

### §7.2 output path guard

W52-post / W44.6-post パターン継承:
- 絶対 path 必須
- safe-prefix: `/tmp/` / `/var/folders/` / `/private/...` / `/Users/`
- forbidden segment: `/data/` / `/.git/` / `/.github/` / `/LaunchAgents/` /
  `/.fire_secrets/`
- `--markdown` / `--output-json` 両方 guarded

### §7.3 AST safety (= 12 件)

- import: sqlite3 / subprocess / linebot / line_bot_sdk / requests /
  urllib.request / aiohttp / httpx / socket / asyncio / paramiko / fabric 全不在
- VACUUM SQL literal in Call arg 0
- --no-verify in Call arg 0
- os.environ direct iteration (= .items() / .keys() / .values()) 0
- eval / exec / compile 0
- destructive file ops (= unlink / remove / rmtree / rmdir / rename) 0
- LINE API call (= push_message / broadcast / pushMessage) 0
- brokerage / order automation references in Call arg 0

### §7.4 v0 前後の境界

| 観点 | v0 前 (= W54-pre 本 wave) | v0 後 (= Wave 55+) |
|---|---|---|
| scaffold runner 存在 | ✓ (= 本 wave で作成) | ✓ |
| task 実行 | 0 (= preview のみ) | 段階実装 |
| DB write | 0 | staging-only first、production 別 wave |
| LINE / token / API | 0 | 0 (= AFTER-R1 は LINE 送信しない設計) |
| cron / launchd 登録 | 0 | 別 wave (= LANE-R0 / DASH-R1 後) |
| safety_flags 全 false | 強制 | 設計次第で一部 true 可 (= DB write 等) |

## §8 test 結果 (= 56 test、CLI 単独)

| class | test 件数 | 内容 |
|---|---|---|
| TestVersionsAndConstants | 4 | SCHEMA / CLI / upstream / 7 task choices |
| TestTaskRegistry | 8 | 6 task 構造 / required keys / planned status / planned_outputs / sub_items 内容 |
| TestTaskSelection | 4 | select_tasks() / build_all_previews() |
| TestOutputPathGuard | 8 | tmpdir / Users / data/ / .fire_secrets / LaunchAgents / relative / main exit 2 |
| TestOutputFormats | 7 | text / JSON / mode / safety_flags / Markdown / output-json / task filter |
| TestDryRunReadOnlyDefaults | 3 | default mode / strict 不変 / exit 0 |
| TestAstSafety | 12 | sqlite3 / subprocess / linebot / HTTP / socket / VACUUM / --no-verify / environ / eval / destructive / LINE / brokerage |
| TestF282PlistAndDbUnchanged | 2 | F282 plist / 3 DB 不変 |
| TestV0DependencyDocumented | 4 | v0_dependency / expected_wave / aggregate / next_steps |
| TestVerdictAggregation | 3 | empty / design preview / strict 不変 |

合計 **56 test、0.09s、全 PASS**。
pytest 全体 collected: 4423 → **4479** (= +56)。

## §9 safety 確認 (= 本 wave 自体)

| 項目 | 結果 |
|---|---|
| 実 file 変更 | 4 file (= CLI 新規 + test 新規 + 設計 doc + plan/results) |
| 実 task 実行 | 0 (= preview-only design) |
| DB write / DB sqlite 接続 | 0 |
| LINE 送信 / API call | 0 |
| token / channel_token / secret / .env 値参照 | 0 |
| env 全体読出 | 0 |
| launchctl 実行 | 0 |
| plist 配置 / 変更 | 0 |
| cron / crontab 変更 | 0 |
| F282 試走干渉 | 0 |
| F282 manual run | 0 |
| Paper Live 実実行 | 0 |
| VACUUM / VACUUM INTO | 0 |
| 楽天証券操作 / 自動発注 | 0 |
| workflow / --no-verify / sudo / rm -rf / git push | 0 |
| TODO Excel 更新 | 0 |
| Codex 直接 commit | 0 |
| 既存 v0 path に変更 | 0 (= F282 / DATA-R3 / F062 / readiness / Ops Summary / wrapper 不触) |

## §10 F282 不干渉確認

baseline (= W54-pre 開始 20:36 JST):
- F282 plist mtime=1778593597 / size=1772
- 3 DB 既知値
- pytest collected 4423

完了時:
- F282 plist / 3 DB **完全一致 ✓**
- F282 next run 5/16 02:00 維持 ✓
- daily-refresh / morning-advisory plist 未配置維持 ✓
- pytest collected **4479** (= +56 想定通り)

## §11 Codex lane 0/8 = 0% の HQ 報告整合性

HQ 補足では「8 lane 第一候補」だが、本 wave も **0/8 で本線完結**。
W44.6-pre / W51 / W52-pre / W52.5-pre precedent 継承。

### 理由
1. user spec が完全 prescriptive (= CLI 引数 11 + task 7 + safety 12 keys 全列挙)
2. 既存 W35-pre AFTER-R1 設計 doc + W51/W44.6-post/W52-post で marker / version 確定済
3. 局所新規実装 (= 1 CLI + 1 test、既存 v0 path に触らない)
4. test coverage 包括 (= 56 test + AST 12 safety)
5. preview-only CLI なので新規論点なし

### 代替手段
- AST safety test 12 件 = Codex Lane F 機能等価
- task registry 構造化 (= 6 task × 9 fields) で Lane A/B 機能等価
- 将来 Wave 54-post (or W55-pre) で adversarial audit 8 lane 投入の選択肢

## §12 6 KPI

- **Codex 稼働率**: **0/8 = 0%** (= prescriptive 実装、HQ 補足からの逸脱明記)
- 短縮率 / 採用率: 該当なし
- 差戻率: 0 (= test 1 件 freshness path 修正なし、新規実装本体は差戻 0)
- Integrator 負荷: 中-高 (= CLI 約 600 行 + test 約 600 行 + 設計 doc + plan/results)
- 安全事故: 0 ✓

### 過去 wave 比較

| wave | Codex 稼働率 | 性質 |
|---|---|---|
| W44.5-pre / W51 / W44.6-pre / W52-pre / W52.5-pre | 0/8 = 0% | prescriptive 実装 / cleanup |
| W44.5-post / W44.6-post / W52-post | 6-7/8 = 75-87.5% | adversarial audit |
| **W54-pre (本 wave)** | **0/8 = 0%** | **prescriptive scaffold 実装 (= v0 後拡張)** |

## §13 future wave 候補

### Wave 54-post (= HQ 判断)
- adversarial audit 8 lane (= scaffold が誤って v0 path に触れていないか、
  task registry に漏れがないか、AST safety 補強)

### v0 後実装 phase
- Wave 55+: paper-live + report task 実装着手
- Wave 56+: replay task 実装
- Wave 57+: simulation task 実装
- Wave 58+: lane-eval task 実装 (= 20 営業日蓄積後)
- Wave 60+: pattern task + ML feature export 実装
- Wave 65+: lane promotion / demotion candidates 実装
- Wave 70+: advisory improvement material 実装
- Wave 75+: ML feature parquet export 実装
- Wave 80+: DASH-R1 dashboard 連携実装

### v0 path から継続課題
- Wave 52 本番: HQ marker 6/7 env 投入 + wrapper 配置 + DB labels guard
- 別 wave (低): 履歴 v1.1 doc 整理

---

## 関連リンク

- [[F286_AFTER_R1_night_paper_live_batch_2026-05-12|W35-pre AFTER-R1 設計 v1.0 (= 既存)]]
- [[Production_v0_cutover_rollback_token_runbook_2026-05-14|cutover runbook]]
- [[Production_v0_readiness_check_cli_v1_2_2026-05-14|readiness CLI v1.2]]
- [[Production_v0_ops_summary_cli_v1_0_2026-05-14|Ops Summary CLI v1.0.1]]
- [[F062_no_send_wrapper_preview_cli_v1_0_2026-05-14|wrapper preview CLI v1.0.1]]
- [[../02_todo/FIRE_CODEX_R1_WAVE54_PRE_plan|W54-pre plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE54_PRE_results|W54-pre results]]
- 実装: `~/fire/scripts/jobs/run_f286_after_r1_night_batch.py`
- test: `~/fire/tests/scripts/jobs/test_run_f286_after_r1_night_batch.py`
