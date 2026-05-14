---
id: F062-no-send-wrapper-preview-cli-v1-0
phase: 本番 v0 Launch / Wave 52-pre / no-send wrapper / marker / DB labels preview
priority: 高
status: 実装 v1.0 (= Wave 52-pre、新規 CLI + marker 統合 + DB labels contract)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40.8 (= F282 post-run inspection CLI)
  - Wave 42-pre (= F062 freshness consumer / no-send runner)
  - Wave 44.5-pre (= cutover/token/rollback runbook v1.0 final)
  - Wave 44.5-post (= adversarial audit)
  - Wave 51 (= readiness CLI v1.2 CRITICAL fix)
  - Wave 44.6-pre/post (= Ops Summary CLI v1.0.1 + audit)
related:
  - 03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md
  - 03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14.md
  - 03_design/Production_v0_ops_summary_cli_v1_0_2026-05-14.md
chapter: production-v0 / Wave 52-pre / no-send preparation
---

# F062 No-Send Wrapper / Send Marker / DB Labels Preview CLI v1.0 — Wave 52-pre

最終更新: 2026-05-13

## §1 目的

Production v0 D-Day (= 2026-06-09 火) 本番 send 手前の wrapper chain と
HQ marker semantics、DB record-decisions labels の preview contract を、
**完全 read-only** で固定する。

実 wrapper 配置 / 実 LINE 送信 / token 値参照 / DB write / launchctl は **0**。
本 wave で固めた preview を **Wave 52 本番**で参照する。

## §2 W52-pre で確定した事項

### §2.1 HQ marker semantics 統合 (= 曖昧名廃止)

W44.5-pre 時点では `HQ_APPROVE_SEND_MARKER` という曖昧名が runbook §6.2 / §7.4 /
§10.2 / §14 で使われていた。意味が:
- token 投入承認 (= marker 6) なのか
- production send 開始承認 (= marker 7) なのか
- 日次 rotate token なのか
が曖昧で、HQ marker 7 段固定 (= W44.5-pre §4) と整合しなかった。

**W52-pre で統合 (= cutover runbook minimal edit 適用済)**:
- `HQ_APPROVE_SEND_MARKER` を廃止
- send 承認は **marker 7 (`HQ_APPROVE_PRODUCTION_V0_LAUNCH`) 値を直接渡す**
- 別 daily-rotating token は不要 (= 過剰設計だった)
- wrapper Stage 2 では `--hq-approved-send "${HQ_APPROVE_PRODUCTION_V0_LAUNCH:-}"` で渡す

W52-pre CLI は `FORBIDDEN_MARKER_ALIASES` constant で以下を refuse:
- `HQ_APPROVE_SEND_MARKER`
- `HQ_SEND_MARKER`
- `SEND_MARKER`

### §2.2 7 段 HQ marker 名

| # | marker name | wave / 時期 | 役割 |
|---|---|---|---|
| 1 | F282 GO 判定 (= 自然遷移) | 5/19 | Wave 41 着手前提 |
| 2 | `HQ_APPROVE_LAUNCHD_DAILY` | Wave 41 | DATA-R3 plist 配置 + load |
| 3 | DATA-R3 no-write 試走 GO (= 自然遷移) | W41 完了 | Wave 45 着手前提 |
| 4 | `HQ_APPROVE_LAUNCHD_MORNING_ADVISORY` | Wave 45 | F062 plist 配置 + load |
| 5 | F062 no-send trial GO (= 自然遷移) | W45 完了 | Wave 52 着手前提 |
| 6 | `HQ_APPROVE_LINE_TOKEN_PRODUCTION` | Wave 52 | LINE token 投入許可 |
| 7 | `HQ_APPROVE_PRODUCTION_V0_LAUNCH` | Wave 53 (= 6/8 月夕方) | **production send 開始許可 + 日次 send 承認** |
| 補 | `HQ_APPROVE_PRODUCTION_V0_RECOVERY` | rollback 後 | 再起動許可 |

W52-pre CLI で validate される marker は **marker 6 / 7 / 補** (= production send 経路の 3 種)。

### §2.3 no-send wrapper command (= W52-pre 確定、Wave 53 で実配置)

**Stage 1 のみ実行** (= preview gen + freshness gate)、Stage 2 は stub:

```bash
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --freshness-report-json /tmp/f286_data_r3_freshness.json \
  --require-freshness-ok \
  --dry-run \
  --base-date 2026-06-09 \
  --output-text /tmp/f062_no_send_preview_2026-06-09.txt
```

**禁止 flag** (= W52-pre safety):
- `--send`
- `--hq-approved-send`
- `--record-decisions`
- `--channel-token-env`
- `--allow-warning`

**必須 flag**:
- `--freshness-report-json`
- `--require-freshness-ok`
- `--dry-run`

### §2.4 production send wrapper stub (= W52 本番、本 wave 不実行)

```bash
.venv/bin/python -m scripts.jobs.run_f062_line_production_send_smoke \
  --payload-json /tmp/f062_no_send_preview_2026-06-09.txt \
  --hq-approved-send ${HQ_APPROVE_PRODUCTION_V0_LAUNCH} \
  --send \
  --record-decisions \
  --record-db-label production \
  --channel-token-env LINE_CHANNEL_TOKEN
```

env reference のみ (= `${...}`)、値は wrapper script source で初めて解決。
本 wave では stub command の文字列を preview するだけ。

### §2.5 DB labels preview contract (= record-decisions が書く列)

W52 本番で `advisory_decisions` table に書かれる label の **schema 固定**:

```json
{
  "schema_version": "1.0",
  "schema": {
    "base_date": "string YYYY-MM-DD",
    "run_id": "string (= f062_<send_mode>_<date>_<source_version>)",
    "decision_id": "string (= per-row UUID-like)",
    "ticker": "string (= 4-digit jp stock code)",
    "action_label": "string (= 積極/条件/場中/見送り 等)",
    "advisory_text_hash": "string (= sha256 of LINE payload)",
    "freshness_report_id": "string (= freshness JSON path + mtime)",
    "readiness_cli_version": "string (= 1.2)",
    "ops_summary_version": "string (= 1.0.1)",
    "send_mode": "string (no_send | production_send)",
    "send_status": "string (preview_only | skipped | sent | failed)",
    "marker_state": "object (= {marker_name: bool})"
  },
  "preview_values": {
    "send_mode": "no_send",
    "send_status": "preview_only",
    ...
  },
  "db_write": false,
  "db_sqlite_connect": false,
  "intent": "Preview only. Not written until W52/53 four-step guard pass."
}
```

DB write は **本 wave で 0**、`preview_values` の per-row fields
(= decision_id / ticker / action_label / advisory_text_hash) は null placeholder。

### §2.6 marker fixture 安全制約 (= 実 marker 作成抑止)

W52-pre で実 HQ marker を作成しない。test 用に必要な場合は:
- fixture は **tmpdir のみ許容** (= `/tmp/` / `/var/folders/` / `/private/tmp/` / `/private/var/folders/`)
- 危険 segment (= `/data/` / `/.git/` / `/.fire_secrets/` / `LaunchAgents/`) refuse
- `validate_marker_fixture(path, marker_name)` で 二重防御

CLI args:
- `--marker-fixture PATH` (repeatable)
- `--marker-required-name {HQ_APPROVE_LINE_TOKEN_PRODUCTION, HQ_APPROVE_PRODUCTION_V0_LAUNCH, HQ_APPROVE_PRODUCTION_V0_RECOVERY}` (= positional pair)

不正 fixture / 曖昧 alias は **必ず refuse**。

### §2.7 freshness gate chain (= W41-pre / W42-pre 整合)

- DATA-R3 freshness OK が **必須** (= verdict=OK)
- freshness NG → preview 生成しない (= Stage 1 abort)
- F062 preview runner の `--require-freshness-ok` で確実に enforce
- W52-pre CLI は freshness path の **stat 確認のみ** (= verdict 値は読まない)

### §2.8 readiness CLI v1.2 / Ops Summary v1.0.1 整合

- `DB_LABELS_SCHEMA["readiness_cli_version"] = "1.2"` (= W51)
- `DB_LABELS_SCHEMA["ops_summary_version"] = "1.0.1"` (= W44.6-post)
- W52-pre CLI が `HQ_MARKER_TOKEN_PRODUCTION / V0_LAUNCH / RECOVERY` を validate
  → readiness CLI v1.2 の `HQ_MARKER_WAVE52 / V0_LAUNCH` と string 完全一致を test で確認
- Ops Summary v1.0.1 は readiness JSON 経由で missing_markers を blocking_issues に取り込み
  (= W44.6-post で確認済)

## §3 CLI 仕様

### §3.1 引数

```
.venv/bin/python -m scripts.jobs.run_f062_no_send_wrapper_preview \
  [--base-date YYYY-MM-DD]                # default: today JST
  [--repo-root PATH]                       # default: ~/fire
  [--freshness-report-json PATH]           # default: /tmp/f286_data_r3_freshness.json
  [--preview-output-path PATH]             # default: /tmp/f062_no_send_preview_<date>.txt
  [--marker-fixture PATH]                  # repeatable, tmpdir only
  [--marker-required-name NAME]            # repeatable, positional pair
  [--json]                                  # JSON to stdout
  [--markdown PATH]                         # write Markdown (guarded)
  [--strict]                                # WARN → NOT_READY
```

### §3.2 出力 3 format

#### text (default)
- Preview readiness verdict
- No-send wrapper command (= preview)
- Production send wrapper stub (= W52 only)
- HQ marker semantics + forbidden aliases
- DB labels preview values
- Marker fixture validations
- Checks
- Safety flags

#### JSON (= `--json`) ※ cli_version は W52-post patch bump 後 "1.0.1"
```json
{
  "schema_version": "1.0",
  "cli_version": "1.0.1",
  "preview_readiness": "READY|NOT_READY|UNKNOWN",
  "no_send_wrapper_command": [...],
  "production_send_wrapper_stub_command": [...],
  "hq_marker_semantics": {...},
  "marker_fixture_validations": [...],
  "db_labels_preview": {...},
  "freshness_state": {...},
  "checks": [...],
  "safety_flags": {...},
  "expected_upstream_versions": {
    "readiness_cli": "1.2",
    "ops_summary": "1.0.1"
  }
}
```

#### Markdown (= `--markdown PATH`)
人間レビュー用、output path guard を経由 (= data/ / .git/ / .fire_secrets/ /
LaunchAgents/ refuse)。

### §3.3 verdict (= preview readiness)

| verdict | 条件 | exit code |
|---|---|---|
| **READY** | 全 PASS、W52 本番進行可 | 0 |
| **UNKNOWN** | SKIP のみ / WARN あり (non-strict) | 0 |
| **NOT_READY** | FAIL あり / strict + WARN | 1 |

### §3.4 safety_flags (= 10 keys 全 false)

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
    "production_send_executed": False,   # W52-pre 新規 (= W52 本番抑止確認)
    "real_hq_marker_created": False,     # W52-pre 新規 (= 実 marker 作成抑止)
}
```

## §4 test 結果 (= 72 test、CLI 単独)

| class | test 件数 | 内容 |
|---|---|---|
| TestVersionsAndConstants | 5 | SCHEMA_VERSION / CLI_VERSION / HQ marker constants / forbidden aliases / upstream versions |
| TestNoSendWrapperCommand | 6 | wrapper command flag 構成 / send 系不在 / preview runner module |
| TestProductionSendWrapperStub | 3 | send runner module / marker env reference / executable な値を含まない |
| TestMarkerFixtureValidation | 7 | tmpdir 限定 / 外部 path refuse / 曖昧 alias refuse / unknown marker / 不在 file |
| TestOutputPathGuard | 6 | safe paths / data/ / .git/ / .fire_secrets/ / LaunchAgents/ refuse + main exit 2 |
| TestDbLabelsPreview | 5 | schema 完備 / no_send mode / invalid send_mode raises / invalid send_status raises / run_id format |
| TestChecksAndVerdict | 13 | 各 check / verdict aggregation (READY/NOT_READY/UNKNOWN) / strict promotion |
| TestOutputFormats | 6 | text / JSON valid / safety_flags all false / wrapper no send flags / Markdown file / no real token in MD |
| TestMainIntegration | 4 | basic run / strict + missing freshness / marker fixture pair / outside tmpdir refuse |
| TestAstSafety | 10 | sqlite3 / subprocess / linebot / external HTTP / socket / VACUUM / --no-verify / os.environ / eval / destructive ops / LINE API call |
| TestF282PlistAndDbUnchanged | 2 | F282 plist / 3 DB mtime/size 不変 |
| TestUpstreamIntegration | 3 | readiness/ops 8mary version 整合 / marker constants が readiness CLI v1.2 と完全一致 |

**合計 72 test、0.10s、全 PASS**。
pytest 全体 collected: 4332 → **4404** (= +72)。

## §5 safety 確認 (= 本 wave 自体)

| 項目 | 結果 |
|---|---|
| 実 file 変更 | 5 file (= CLI 新規 + tests 新規 + 設計 doc + cutover runbook 4 箇所 minimal edit + W52-pre plan/results) |
| CLI AST: 10 safety test 全 PASS | ✓ |
| 実 wrapper 配置 | 0 |
| 実 LINE 送信 / API call | 0 |
| 実 HQ marker 作成 | 0 (= mock fixture は tmpdir 限定) |
| DB write / DB sqlite 接続 | 0 |
| token / channel_token / secret / .env 値参照 | 0 |
| env 全体読出 | 0 |
| launchctl 実行 | 0 |
| plist 配置 / 変更 | 0 |
| cron / crontab 変更 | 0 |
| F282 試走干渉 | 0 |
| F282 manual run | 0 |
| VACUUM / VACUUM INTO | 0 |
| workflow / --no-verify / sudo / rm -rf / git push | 0 |
| TODO Excel 更新 | 0 |
| 楽天証券操作 / 自動発注 / Computer Use | 0 |
| production send 経路到達 | 0 (= safety_flag production_send_executed=false) |

## §6 F282 不干渉確認

baseline (= W52-pre 開始、19:54 JST):
- F282 plist mtime=1778593597 / size=1772
- 3 DB 既知値
- pytest collected 4332

完了時:
- F282 plist mtime/size **完全一致 ✓**
- 3 DB mtime/size **完全一致 ✓**
- F282 next run 5/16 02:00 維持 ✓
- daily-refresh / morning-advisory plist 未配置維持 ✓
- pytest collected **4404** (= +72 想定通り)

## §7 Codex lane 0/8 = 0% の理由

HQ 補足では「8 lane 第一候補」だが、本 wave も **0/8 で本線完結**。
W44.6-pre / W51 で確立した「prescriptive spec + 局所新規実装」パターンを継承。

### 理由
1. user spec が完全 prescriptive (= CLI 引数 / marker 名 / DB labels keys / safety constraints 全列挙)
2. 既存 wave で marker 名 / source 仕様確定済 (= W44.5-pre 7 段 marker / W51 readiness JSON / W44.6-pre Ops Summary 構造)
3. 局所新規実装 (= 1 CLI + 1 test file)
4. test coverage 包括 (= 72 test + AST 10 safety)
5. W44.6-pre / W51 precedent (= 0/8 accepted)

### 代替手段
- AST safety test 10 件 = Lane F (source safety) 同等
- Marker semantics 統合は user spec で明示済、Codex の追加検討不要
- 将来 W52-post で adversarial audit 8 lane 投入の選択肢 (= HQ 判断)

## §8 6 KPI

- **Codex 稼働率**: **0/8 = 0%** (= §7 理由参照、HQ 補足からの逸脱明記)
- 短縮率 / 採用率: 該当なし
- 差戻率: 0 (= test 1 件 freshness path absent fix で吸収、新規実装本体は差戻 0)
- Integrator 負荷: 中-高 (= CLI 約 750 行 / test 約 750 行 / 設計 doc + cutover runbook 4 箇所 edit + plan/results)
- 安全事故: 0 ✓

## §9 cutover runbook minimal edit (= 4 箇所)

W52-pre で適用済 minimal edit:

1. §6.2 wrapper Stage 2 command を marker 7 統合版に
   - 旧: `--hq-approved-send "${HQ_APPROVE_SEND_MARKER:-}"`
   - 新: `--hq-approved-send "${HQ_APPROVE_PRODUCTION_V0_LAUNCH:-}"`
   - send runner 引数を `--payload-json` / `--send` / `--record-decisions` /
     `--record-db-label production` / `--channel-token-env LINE_CHANNEL_TOKEN` に確定
2. §6.2 注記: 「W52-pre 統合: 旧 HQ_APPROVE_SEND_MARKER は廃止、marker 7 に統一」
3. §6.2 要件 list: `HQ_APPROVE_PRODUCTION_V0_LAUNCH 未 export → ...` に置換
4. §16 残課題 #2: `HQ_APPROVE_SEND_MARKER 生成手順確定` → `marker 7 統合で確定` に更新

加えて §7.4 / §10.2 / §14 timeline / §16 一覧の `HQ_APPROVE_SEND_MARKER`
全 references (= 計 7 箇所) を `HQ_APPROVE_PRODUCTION_V0_LAUNCH` に統一。

## §10 残課題 (= 次 Wave 候補)

### Wave 52 本番で必要 (= W52-pre で解消できないもの)

- env `HQ_APPROVE_LINE_TOKEN_PRODUCTION` 投入 (= 人間作業、6/8 月以前)
- `~/.fire_secrets/line.production.env` 配置 + chmod 600 (= 人間作業)
- env `HQ_APPROVE_PRODUCTION_V0_LAUNCH` 投入 (= 人間作業、6/8 月夕方)
- wrapper script `~/fire/scripts/wrappers/run_f062_production_send.sh` 配置 + chmod 700
- `WRITE_ALLOWED_DB_LABELS_PRODUCTION_V0` / `WRITE_ALLOWED_DB_BASENAMES_PRODUCTION_V0`
  module 配置 (= record-decisions 四段ガード)
- staging→production schema 互換 shasum 確認

### W52-pre で開ける道
- W52-pre Ops Summary integration: send marker missing を Ops Summary blocking_issues に取り込み (= 現状 Ops Summary は readiness CLI 経由のみ)
- W52-pre 後の W52-post で adversarial audit (= HQ 判断)

### W44.5-post / W44.6-post から継続
- (Wave 52 / 53) markdown 上書き保護 (= LOW)
- (別 wave、低) launch plan §3 Phase D/E 1 日重複

---

## 関連リンク

- [[Production_v0_cutover_rollback_token_runbook_2026-05-14|cutover runbook v1.0 final (= W52-pre minimal edit 4 箇所適用済)]]
- [[Production_v0_readiness_check_cli_v1_2_2026-05-14|W51 readiness CLI v1.2]]
- [[Production_v0_ops_summary_cli_v1_0_2026-05-14|W44.6-pre Ops Summary CLI v1.0.1]]
- [[F062_wave42_pre_no_send_runner_enhancement_2026-05-14|W42-pre F062 freshness consumer]]
- [[../02_todo/FIRE_CODEX_R1_WAVE52_PRE_plan|W52-pre plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE52_PRE_results|W52-pre results]]
- 実装: `~/fire/scripts/jobs/run_f062_no_send_wrapper_preview.py`
- test: `~/fire/tests/scripts/jobs/test_run_f062_no_send_wrapper_preview.py`
