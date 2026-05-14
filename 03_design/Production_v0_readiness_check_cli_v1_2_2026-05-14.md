---
id: Production-v0-readiness-check-cli-v1-2
phase: 本番 v0 Launch / readiness CLI v1.2 CRITICAL fix
priority: 高
status: 実装 v1.2 (= Wave 51、W44.5-post CRITICAL 3 件解消)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40.7 (= readiness CLI v1.0)
  - Wave 43-pre (= readiness CLI v1.1、25 check / 32 test)
  - Wave 44.5-post (= adversarial audit、CRITICAL 3 件指摘)
related:
  - 03_design/Production_v0_readiness_check_cli_v1_1_2026-05-14.md
  - 03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md
chapter: production-v0 / readiness CLI v1.2 / CRITICAL fix
---

# Production v0 Readiness Check CLI v1.2 — Wave 51 CRITICAL Fix

最終更新: 2026-05-13

## §1 目的

W44.5-post adversarial audit で発見された readiness CLI v1.1 CRITICAL 3 件を
修正し、Production v0 D-Day の GO/NO-GO 判定を **安全側に倒す**。

## §2 W44.5-post CRITICAL 3 件 (= 監査結果)

| # | 件名 | 影響 |
|---|---|---|
| C1 | HQ marker phase gating bug | pre-v0-launch で Wave 41/45/52 marker SKIP され silent GO リスク |
| C2 | D-Day weekday Tuesday 固定 WARN | W44.5-pre で Tuesday lock 確定後も WARN、strict mode で false NO-GO |
| C3 | F282 post-run report 設計 doc vs 実装 gap | post-f282 以降 phase 別の WARN/FAIL 切替が無く、設計 doc とズレ |

## §3 v1.2 修正内容

### §3.1 CLI_VERSION = "1.2"

```python
CLI_VERSION = "1.2"
```

JSON output `cli_version` も 1.2。後方互換 (= check_id schema / 25 check 構成) 維持。

### §3.2 C1 修正: HQ marker cumulative phase gating

#### 新規定数

```python
PHASE_ORDER: tuple[str, ...] = (
    PHASE_PRE_F282,      # idx=0
    PHASE_POST_F282,     # idx=1
    PHASE_PRE_WAVE41,    # idx=2
    PHASE_PRE_WAVE45,    # idx=3
    PHASE_PRE_WAVE52,    # idx=4
    PHASE_PRE_V0_LAUNCH, # idx=5
)

HQ_MARKER_FIRST_REQUIRED_PHASE: dict[str, str] = {
    HQ_MARKER_WAVE41: PHASE_PRE_WAVE41,
    HQ_MARKER_WAVE45: PHASE_PRE_WAVE45,
    HQ_MARKER_WAVE52: PHASE_PRE_WAVE52,
    HQ_MARKER_V0_LAUNCH: PHASE_PRE_V0_LAUNCH,
}
```

#### `_check_hq_marker` の挙動変更

| condition | v1.1 | v1.2 |
|---|---|---|
| `phase < target_phase` | SKIP | SKIP (= 同じ) |
| `phase == target_phase` + marker 取得済 | PASS | PASS (= 同じ) |
| `phase == target_phase` + marker 不足 | **WARN** | **FAIL** ← 強化 |
| `phase > target_phase` + marker 取得済 | **SKIP (bug)** | **PASS** ← 修正 |
| `phase > target_phase` + marker 不足 | **SKIP (silent GO bug)** | **FAIL** ← 修正 |

→ **cumulative gating** で「target_phase 以降は必ず marker 必須」を強制。

### §3.3 C2 修正: D-Day Tuesday lock PASS

#### 新規定数

```python
D_DAY_HQ_LOCKED: bool = True
D_DAY_HQ_LOCKED_AS_OF: date = date(2026, 5, 13)  # W44.5-pre 確定日
D_DAY_LOCKED_WEEKDAY: str = "Tuesday"
```

#### `check_d_day_weekday_hq_decision_pending` の挙動変更

| condition | v1.1 | v1.2 |
|---|---|---|
| `D_DAY_HQ_LOCKED=True` + Tuesday | **WARN (false NO-GO リスク)** | **PASS** ← 修正 |
| `D_DAY_HQ_LOCKED=True` + 他曜日 | (該当なし) | **FAIL** ← 新規 (= calendar 不変条件破綻検知) |
| `D_DAY_HQ_LOCKED=False` (= legacy) + Monday | PASS | PASS (= legacy 保持) |
| `D_DAY_HQ_LOCKED=False` + Tuesday | WARN | WARN (= legacy 保持) |

→ W44.5-pre 確定後は Tuesday → PASS、strict mode で false NO-GO にならない。
→ legacy mode (= W43-pre 時点の動作) は `D_DAY_HQ_LOCKED=False` で復元可能 (= 後方互換)。

### §3.4 C3 修正: F282 post-run report phase 別 WARN/FAIL

#### `check_f282_post_run_report_exists` の挙動変更

| phase | v1.1 (report missing) | v1.2 (report missing) | v1.1 / v1.2 (report present) |
|---|---|---|---|
| pre-f282 | SKIP | SKIP (= 同じ) | PASS |
| post-f282 | WARN | WARN (= 同じ、drill 進行中許容) | PASS |
| pre-wave41 以降 | **WARN** | **FAIL** ← 修正 (= mandatory) | PASS |

#### path pattern 明示

```python
expected_path_pattern = "reports/f282/post_run_YYYY-MM-DD.md"
```

evidence の `expected_report_path_pattern` に常に格納 (= W40.8 CLI `--markdown` arg と整合)。

### §3.5 missing_markers in JSON output

```python
def collect_missing_markers(results: list[CheckResult]) -> list[str]:
    """marker check の FAIL から missing marker 名を抽出."""
```

JSON output に `missing_markers: list[str]` を追加。text output も
`Missing HQ markers: NAME1, NAME2` 行を追加。

## §4 check 一覧 (= v1.1 と同じ 25 check、挙動のみ強化)

v1.2 で check の数 / id / name は変更なし。各 check の **判定 logic** のみ強化:

- HQ marker 系 4 件 (= W41/45/52/V0_LAUNCH) → cumulative gating + FAIL
- D-Day weekday 1 件 → lock 状態判定で Tuesday PASS
- F282 post-run report 1 件 → phase 別 WARN/FAIL

その他 19 check (= F282 plist / DB / token / DATA-R3 / F062 / Date) は v1.1 から
**完全互換** (= 既存 32 test 全 PASS 維持)。

## §5 phase 別判定 matrix (v1.2)

| phase | marker 必須 (FAIL if missing) | F282 post-run report | D-Day weekday |
|---|---|---|---|
| pre-f282 | 0 件 | SKIP | SKIP |
| post-f282 | 0 件 | WARN if missing | SKIP |
| pre-wave41 | W41 | FAIL if missing | SKIP |
| pre-wave45 | W41 + W45 | FAIL if missing | SKIP |
| pre-wave52 | W41 + W45 + W52 | FAIL if missing | SKIP |
| pre-v0-launch | W41 + W45 + W52 + V0_LAUNCH | FAIL if missing | PASS (= Tuesday lock) |

W43-pre v1.1 設計 doc §6 の matrix と比較すると、F282 post-run report が
post-f282 以外 SKIP だった点を v1.2 で「pre-wave41 以降 FAIL」に強化。

## §6 後方互換性

| 項目 | v1.1 | v1.2 | 後方互換 |
|---|---|---|---|
| CLI 引数 | `--phase / --json / --strict / --repo-root / --vault-root / --output-json` | 同じ | ✓ |
| check_id 名 | 25 件 | 25 件 (= 同じ id) | ✓ |
| check 数 | 25 | 25 | ✓ |
| `cli_version` JSON field | "1.1" | "1.2" | ◯ (= field 名は同じ、値のみ更新) |
| `missing_markers` JSON field | なし | 新規追加 | ◯ (= 既存 consumer は無視可) |
| `D_DAY_HQ_LOCKED` 定数 | (なし) | 新規追加 | ◯ (= False で v1.1 挙動復元可) |
| `PHASE_ORDER` / `HQ_MARKER_FIRST_REQUIRED_PHASE` | (なし) | 新規追加 | ◯ (= 内部定数) |
| `check_hq_marker_required_for_*` 関数 | WARN | FAIL | ✗ (= W44.5-post CRITICAL 1 fix の本質) |
| `check_d_day_weekday_hq_decision_pending` | WARN for Tuesday | PASS for Tuesday | ✗ (= CRITICAL 2 fix の本質) |
| `check_f282_post_run_report_exists` | WARN | FAIL post-wave41 | ✗ (= CRITICAL 3 fix の本質) |

CRITICAL fix の本質である 3 つの挙動変化は **意図的な互換性 break** で、
W44.5-post audit の指摘解消が目的。

## §7 test 結果

新規 test class (= 27 test 追加):

- TestCliVersionV12 (= 2 件、v1.1 から rename + assert 値更新)
- TestDDayWeekdayHqDecisionPending (= 4 件、Tuesday PASS / legacy WARN / strict false NO-GO 防止)
- TestDDayBaseDateInvariants (= 3 件、D-Day / final strict の date 不変条件)
- TestHqMarkerCumulativeGating (= 8 件、phase 別 marker FAIL 確認 / cumulative 確認)
- TestMissingMarkersInSummary (= 2 件、collect_missing_markers 関数 / JSON field)
- TestF282PostRunReportPhaseGating (= 6 件、phase 別 WARN/FAIL)
- TestW51SafetyInvariants (= 6 件、AST safety re-verify)

既存 32 test 維持 (= 後方互換)。
**合計 59 test、0.15s、全 PASS**。

pytest collected: 4202 → **4229** (= +27)

## §8 safety 確認 (= 本 wave 自体)

| 項目 | 結果 |
|---|---|
| CLI 本体 sqlite3 import (= AST 検証) | 0 |
| CLI 本体 subprocess import (= AST 検証) | 0 |
| CLI 本体 linebot / line_bot_sdk import (= AST 検証) | 0 |
| CLI 本体 VACUUM SQL literal (= AST 検証) | 0 |
| CLI 本体 --no-verify in call arg (= AST 検証) | 0 |
| CLI 本体 os.environ direct items() / keys() / values() | 0 |
| 実 DB write / production・develop・staging DB SQLite 接続 | 0 |
| LINE 送信 / API call | 0 |
| token / channel_token / secret / .env 値参照 | 0 (= 個別 key の `os.environ.get` のみ) |
| launchctl 実行 | 0 |
| plist 配置 / 変更 | 0 |
| cron / crontab 変更 | 0 |
| F282 試走干渉 | 0 (= plist mtime/size 完全不変) |
| F282 manual run | 0 |
| pytest 実行 | 59 test in CLI 単独 / 4229 全体 collect 確認 |
| workflow / --no-verify / sudo / rm -rf / git push | 0 |
| TODO Excel 更新 | 0 |
| 楽天証券操作 / 自動発注 / Computer Use | 0 |
| Codex 直接 commit | 0 |

## §9 F282 不干渉確認

baseline (= W51 開始、19:09 JST):
- F282 plist mtime=1778593597 / size=1772
- 3 環境 DB mtime / size 既知値
- pytest collected 4202

完了時 (= W51 終了):
- F282 plist mtime / size **完全一致 ✓**
- 3 環境 DB mtime / size **完全一致 ✓**
- F282 next run 5/16 02:00 維持 ✓
- daily-refresh / morning-advisory plist 未配置維持 ✓
- pytest collected 4229 (= +27 W51 new tests、想定通り)

## §10 Codex lane 不投入 (= 0/8 = 0%)

HQ 補足では「8 lane 第一候補」だが、本 wave は **0/8 で本線完結**。

### 理由
1. **CRITICAL の所在が明確**: W44.5-post audit (= 8 lane 並列、Lane G で
   specific check_id + line number まで特定済) で fix 内容が事前確定。
   Codex 並列で「新たな観点」を獲得する余地が小さい。
2. **実装範囲が局所**: scripts/jobs/run_production_v0_readiness_check.py
   の 3 関数 (= `_check_hq_marker` / `check_d_day_weekday_hq_decision_pending` /
   `check_f282_post_run_report_exists`) + 定数 + 集約関数 1 件のみ。
3. **test coverage 十分**: 27 新規 test (= 5 class、全 phase / 全 marker /
   AST 6 safety) で W44.5-post で言及された全 finding を網羅。
4. **AST safety test を本線実装**: Codex で source audit する代替手段
   (= `TestW51SafetyInvariants` 6 件) を本線 pytest で常時自動検証可。
5. **W44.5-post 経験**: Lane F + H で Codex 実行許可拒否 (= 6/8 = 75%) を
   経験済、本 wave でも 100% 稼働は期待できず、本線完結が確実。

### 代替手段
- 本線で各 critical fix に対応する test を 5 class / 27 件追加
- AST 検証 6 safety test を本線で実装 (= Lane F / H 同等)
- W44.5-post Lane G 指摘内容を 1:1 で fix (= 第三者観点の再投入は不要)

### KPI への反映 (= §11)
- Codex 稼働率 0/8 = 0% (= HQ 補足からの逸脱、本 results §10 で明記)
- 代替: 本線 27 test + AST 検証 6 件で機能等価カバレッジ

## §11 6 KPI

- Codex 稼働率: **0/8 = 0%** (= §10 理由参照、HQ 補足からの逸脱を明記)
- 本線短縮率: 該当なし (= Codex lane 不投入)
- 採用率: 該当なし
- 差戻率: 0 (= 内製 test 1 回更新 = 安全 test を string match → AST 検証に
  変更で吸収、CRITICAL 修正自体は差戻 0)
- Integrator 負荷: 中 (= CLI +約 140 行 / test +約 270 行 / docs 1 file 新規 +
  関連 docs 軽微更新)
- 安全事故: 0 ✓ (= §8 / §9 完全)

## §12 残課題

W44.5-post §16 の次 Wave 候補から本 wave で解消:
- ~~(#19) CLI HQ marker phase gating bug~~ → **本 wave で fix ✓**
- ~~(#20) CLI D-Day weekday 固定 WARN~~ → **本 wave で fix ✓**
- ~~(#21) W43-pre CLI v1.1 設計 doc 整合 gap~~ → **本 wave で実装側を修正、
  併せて本 doc (= v1.2 設計 doc) で整合確認**

継続課題:
- (#22) wrapper script Wave 53 実装時 runner 引数 grep 再確認 → Wave 53
- (#23) W43-pre CLI v1.1 doc 内「HQ docs: D-Day = 月曜」整理 → 別 wave (低優先)
- (#24) launch plan §3 Phase D `6/2-6/9` と Phase E `6/9` 1 日重複 → 別 wave (低優先)
- W44.5-pre §16 から継続 (Wave 52 必須):
  - ~~HQ_APPROVE_SEND_MARKER 生成手順確定~~ → **W52-pre / W52-post で曖昧名廃止 +
    marker 7 (HQ_APPROVE_PRODUCTION_V0_LAUNCH) store_true gate 統合で確定済**
  - wrapper script 配置 + permission 確認 → Wave 52 本番
  - WRITE_ALLOWED_DB_LABELS_PRODUCTION_V0 test 関数定義 → Wave 52 本番

---

## 関連リンク

- [[Production_v0_readiness_check_cli_v1_1_2026-05-14|W43-pre readiness CLI v1.1 (= 前版)]]
- [[Production_v0_cutover_rollback_token_runbook_2026-05-14|cutover runbook v1.0 final (audited)]]
- [[../02_todo/FIRE_CODEX_R1_WAVE51_plan|W51 plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE51_results|W51 results]]
- [[../02_todo/FIRE_CODEX_R1_WAVE44_5_POST_results|W44.5-post audit (= CRITICAL 検出元)]]
- 実装: `~/fire/scripts/jobs/run_production_v0_readiness_check.py`
- test: `~/fire/tests/scripts/jobs/test_run_production_v0_readiness_check.py`
