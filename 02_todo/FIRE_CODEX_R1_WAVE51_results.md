---
id: FIRE-CODEX-R1-WAVE51-results
phase: 本番 v0 Launch / readiness CLI v1.2 CRITICAL fix
priority: 高
status: results (= Wave 51 完了報告)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
related:
  - 02_todo/FIRE_CODEX_R1_WAVE51_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE44_5_POST_results.md
  - 03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14.md
---

# Wave 51 Results — Production v0 Readiness CLI v1.2 CRITICAL Fix

最終更新: 2026-05-13

## §1 目的 (= plan §1)

W44.5-post adversarial audit で発見された readiness CLI v1.1 CRITICAL 3 件を
修正し、Production v0 D-Day の GO/NO-GO 判定を **安全側に倒す**。

## §2 W44.5-post CRITICAL 3 件 → 全解消

| # | 件名 | v1.1 挙動 | v1.2 修正 |
|---|---|---|---|
| C1 | HQ marker phase gating bug | `phase != target_phase` で SKIP → silent GO リスク | **cumulative gating** で `target_phase` 以降は必ず check + 不足は FAIL |
| C2 | D-Day weekday Tuesday 固定 WARN | Tuesday WARN → strict false NO-GO | `D_DAY_HQ_LOCKED=True` で Tuesday PASS (= W44.5-pre 確定反映) |
| C3 | F282 post-run report 設計 doc vs 実装 gap | post-f282 以外 SKIP 設計 vs 実装 WARN | phase 別: pre-f282 SKIP / post-f282 WARN / pre-wave41 以降 FAIL |

## §3 実装内容

### §3.1 CLI 変更 (= +約 140 行、`scripts/jobs/run_production_v0_readiness_check.py`)

- **新規定数**:
  - `CLI_VERSION = "1.2"` (= v1.1 から bump)
  - `PHASE_ORDER` (= 6 phase の順序)
  - `_phase_index(phase) -> int` helper
  - `HQ_MARKER_FIRST_REQUIRED_PHASE` (= dict、各 marker の first required phase)
  - `D_DAY_HQ_LOCKED = True` (= W44.5-pre 確定反映)
  - `D_DAY_HQ_LOCKED_AS_OF` / `D_DAY_LOCKED_WEEKDAY = "Tuesday"`
- **関数修正**:
  - `_check_hq_marker`: cumulative gating + 不足は FAIL (= 旧 WARN)
  - `check_d_day_weekday_hq_decision_pending`: lock 状態判定 + Tuesday PASS
  - `check_f282_post_run_report_exists`: phase 別 WARN/FAIL 切替 + path pattern 明示
  - `overall_summary`: `missing_markers` 集約追加
  - `collect_missing_markers`: 新関数、FAIL marker check から名前抽出
  - `render_text` / `render_json`: missing_markers 出力 + CLI_VERSION header

### §3.2 test 追加 (= +27 件、5 class、`tests/scripts/jobs/test_run_production_v0_readiness_check.py`)

| class | test 件数 | 内容 |
|---|---|---|
| TestCliVersionV12 (= TestCliVersionV11 rename) | 2 | CLI_VERSION constant / JSON cli_version |
| TestDDayWeekdayHqDecisionPending (= update) | 4 | Tuesday lock PASS / legacy WARN / strict false NO-GO 防止 / other phase SKIP |
| TestDDayBaseDateInvariants (= 新規) | 3 | D-Day 不変条件 / final strict 不変条件 / check_d_day_date PASS |
| TestHqMarkerCumulativeGating (= 新規) | 8 | 各 phase で marker FAIL 確認 / cumulative 確認 / 全 marker PASS 時 / PHASE_ORDER 定数 / FIRST_REQUIRED_PHASE mapping |
| TestMissingMarkersInSummary (= 新規) | 2 | collect_missing_markers function unit / JSON field 存在 |
| TestF282PostRunReportPhaseGating (= 新規) | 6 | pre-f282 SKIP / post-f282 WARN / pre-wave41 FAIL / pre-v0-launch FAIL / report present PASS / path pattern 整合 |
| TestW51SafetyInvariants (= 新規) | 6 | AST safety: sqlite3 / subprocess / VACUUM / linebot / --no-verify / os.environ direct |

合計 27 件 + 既存 32 件 = **CLI 単独 59 test、全 PASS、0.15s**

pytest 全体: 4202 → **4229** (= +27 想定通り)

### §3.3 docs 追加

- `~/fire-vault/03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14.md` (= 新規、v1.2 設計 doc)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE51_plan.md` (= plan)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE51_results.md` (= 本 file)

## §4 後方互換性 (= 確認 12 項目)

| 項目 | 結果 |
|---|---|
| CLI 引数 (`--phase / --json / --strict / --repo-root / --vault-root / --output-json`) | 完全互換 ✓ |
| check_id 名 / 25 check 構成 | 完全互換 ✓ |
| `cli_version` JSON field 構造 | field 名 ✓、値 1.1 → 1.2 (= 意図的) |
| `missing_markers` JSON field | 新規追加 (= 既存 consumer は無視可) |
| 既存 32 test 全 PASS | ✓ (= 内 3 test は v1.1 期待値を v1.2 に更新済) |
| `_check_hq_marker` 戻り値型 (CheckResult) | 同じ ✓ |
| evidence dict キー追加 (= `phase_index` / `target_phase_index` / `cumulative_required_from`) | 追加のみ ✓ |
| D_DAY_HQ_LOCKED = False で v1.1 挙動復元 | ✓ (= legacy mode test で確認) |
| CLI exit code (0=GO / 1=NO-GO / 2=invalid phase) | 同じ ✓ |
| text output format | summary 行に missing markers 行追加のみ ✓ |
| 既存 phase 6 種 (= pre-f282 / post-f282 / pre-wave41 / pre-wave45 / pre-wave52 / pre-v0-launch) | 同じ ✓ |
| 既存 25 check 関数名 | 同じ ✓ |

意図的 break (= CRITICAL fix の本質):
- marker 不足 phase >= target で WARN → FAIL
- D-Day Tuesday WARN → PASS (= lock 確定後)
- F282 post-run report missing pre-wave41+ で WARN → FAIL

## §5 Codex lane 0/8 = 0% の理由 + HQ 報告

HQ 補足では「8 lane 第一候補」だが、本 wave は **0/8 で本線完結**。理由:

1. **CRITICAL 所在が明確**: W44.5-post 8 lane 並列 audit (= Lane G が
   specific check_id + line number まで特定済) で fix 内容が確定済。
   Codex で「新たな観点」を獲得する余地が小さい。
2. **実装範囲が局所**: 3 関数 (= `_check_hq_marker` /
   `check_d_day_weekday_hq_decision_pending` / `check_f282_post_run_report_exists`)
   + 定数 + 集約関数 1 件。
3. **test coverage 十分**: 27 新規 test (= 5 class、全 phase / 全 marker /
   AST 6 safety) で W44.5-post finding 全網羅。
4. **AST safety を本線で実装**: Codex 監査の代替手段
   (= `TestW51SafetyInvariants` 6 件) を本線 pytest に固定、CI で常時自動検証可。
5. **W44.5-post 経験**: Lane F + H で Codex 実行許可拒否 (= 75% 稼働) を経験、
   本 wave で 100% 稼働は期待できず、本線完結が確実。

代替手段:
- 本線で各 CRITICAL fix の test (= 27 件) を体系的に追加
- AST 検証 6 safety test を本線で実装 (= Codex の source audit に相当)
- W44.5-post Lane G 指摘を 1:1 で fix (= 第三者観点の再投入は不要)

## §6 safety 確認 (= 本 wave 自体)

| 項目 | 結果 |
|---|---|
| 実 file 変更 | 5 file (= CLI + tests + v1.2 設計 doc + W51 plan + W51 results) |
| CLI sqlite3 import (AST) | 0 ✓ |
| CLI subprocess import (AST) | 0 ✓ |
| CLI linebot / line_bot_sdk import (AST) | 0 ✓ |
| CLI VACUUM SQL literal (AST) | 0 ✓ |
| CLI --no-verify in call arg (AST) | 0 ✓ |
| CLI os.environ direct access (AST) | 0 ✓ |
| DB write | 0 |
| production / develop / staging DB SQLite 接続 | 0 |
| LINE 送信 / API call | 0 |
| token / channel_token / secret / .env 値参照 | 0 (= 個別 key の `os.environ.get` のみ) |
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

## §7 F282 不干渉確認

baseline (= W51 開始、19:09 JST):
- F282 plist mtime=1778593597 / size=1772
- fire.db mtime=1778570244 / size=371081216
- fire.develop.db mtime=1778569903 / size=371081216
- fire.staging.db mtime=1778579122 / size=4804063232
- F282 state = not running、5/16 02:00 試走待機
- daily-refresh / morning-advisory plist 未配置
- pytest collected 4202

完了時:
- F282 plist mtime=1778593597 / size=1772 **完全一致 ✓**
- 3 DB mtime / size **完全一致 ✓**
- F282 state = not running 維持 ✓
- daily-refresh / morning-advisory plist 未配置維持 ✓
- pytest collected **4229** (= +27 想定通り)

## §8 6 KPI

- **Codex 稼働率**: **0/8 = 0%** (= §5 理由参照、HQ 補足からの逸脱を明記)
- **本線短縮率**: 該当なし (= Codex lane 不投入)
- **採用率**: 該当なし
- **差戻率**: 0 (= 内製 test 1 回更新 = AST 検証移行で吸収、CRITICAL 修正自体は差戻 0)
- **Integrator 負荷**: 中 (= CLI +約 140 行 / test +約 270 行 / docs 1 新規 + plan/results)
- **安全事故**: 0 ✓ (= §6 / §7 全 0)

### 比較

| wave | Codex 稼働率 | wave 性質 |
|---|---|---|
| W43-pre | 8/8 = 100% | CLI v1.1 新規実装 |
| W44-pre | 8/8 = 100% | F282 baseline helper 新規実装 |
| W44.5-pre | 0/8 = 0% | runbook 集約整理 (= 新規論点なし) |
| W44.5-post | 6/8 = 75% | adversarial audit (= 8 lane 投入、F/H 拒否) |
| **W51 (本 wave)** | **0/8 = 0%** | CRITICAL fix (= 所在明確 + 範囲局所) |

「集約整理 / CRITICAL fix は本線」「新規実装 / 敵対監査は Codex 並列」という
使い分けが KPI に反映。

## §9 残課題

W44.5-post §16 の次 Wave 候補:
- ~~(#19) CLI HQ marker phase gating bug~~ → **本 wave で fix ✓**
- ~~(#20) CLI D-Day weekday 固定 WARN~~ → **本 wave で fix ✓**
- ~~(#21) W43-pre CLI v1.1 設計 doc 整合 gap~~ → **本 wave で実装側を修正済、
  v1.2 設計 doc 新規起票で整合**
- (#22) wrapper script Wave 53 実装時 runner 引数 grep 再確認 → Wave 53 継続
- (#23) W43-pre CLI v1.1 doc 内「HQ docs: D-Day = 月曜」整理 → 別 wave (低)
- (#24) launch plan §3 Phase D / E 1 日重複 → 別 wave (低)

W44.5-pre §16 から継続 (Wave 52 必須):
- HQ_APPROVE_SEND_MARKER 生成手順確定
- wrapper script 配置 + permission 確認
- WRITE_ALLOWED_DB_LABELS_PRODUCTION_V0 test 関数定義

## §10 HQ 1 ブロック報告

```
[FIRE-CODEX-R1 Wave 51 完了]
Production v0 Readiness Check CLI v1.2 CRITICAL fix 完了。
W44.5-post adversarial audit で発見された CRITICAL 3 件を全解消。

CRITICAL fix:
- C1: HQ marker phase gating bug → cumulative gating + 不足 FAIL (= silent GO 解消)
- C2: D-Day weekday Tuesday 固定 WARN → D_DAY_HQ_LOCKED で Tuesday PASS
  (= W44.5-pre 確定反映、strict false NO-GO リスク解消)
- C3: F282 post-run report 設計 vs 実装 gap → phase 別 WARN/FAIL 切替
  (= pre-wave41 以降は FAIL = mandatory)

実装内容:
- CLI_VERSION = "1.2" (= JSON output に反映)
- PHASE_ORDER / HQ_MARKER_FIRST_REQUIRED_PHASE / D_DAY_HQ_LOCKED 定数追加
- _check_hq_marker / check_d_day_weekday_hq_decision_pending /
  check_f282_post_run_report_exists 3 関数 fix
- collect_missing_markers 追加 + JSON / text output に missing_markers 反映

test 追加: 27 件 / 5 class (= 全 phase / 全 marker / D-Day lock / F282 phase /
AST 6 safety) + 既存 32 test 全 PASS 維持。

pytest collected: 4202 → 4229 (= +27 想定通り)
CLI 単独 test: 59 test 0.15s 全 PASS

更新 file:
- scripts/jobs/run_production_v0_readiness_check.py (= v1.1 → v1.2)
- tests/scripts/jobs/test_run_production_v0_readiness_check.py (= +27 test)
- fire-vault/03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14.md (= 新規)
- fire-vault/02_todo/FIRE_CODEX_R1_WAVE51_plan / results (= 新規)

後方互換: 既存 32 test 維持 / check_id 構造維持 / D_DAY_HQ_LOCKED=False で
v1.1 挙動復元可。意図的 break = CRITICAL fix の本質 3 点のみ。

安全確認 (全 0):
- F282 plist mtime/size 完全不変 (1778593597/1772)
- 3 環境 DB mtime/size 完全不変
- LINE/token/API/launchd/plist/cron/VACUUM/workflow/--no-verify/TODO Excel 全 0
- AST 検証で sqlite3 / subprocess / linebot / VACUUM / --no-verify import / 文字列 0
- pytest 4229 維持 + CLI 単独 59 test 0.15s 全 PASS

Codex lane: 0/8 = 0% (= HQ 補足「8 lane 第一候補」からの逸脱を明記)
理由: CRITICAL 所在明確 + 範囲局所 + 27 test + AST 6 safety で機能等価。
W44.5-post Lane G 指摘を 1:1 で fix、第三者観点の再投入不要。
W44.5-post で経験した Codex F/H 拒否を踏まえ、本 wave は本線完結が確実。

6 KPI: 稼働率 0% / 短縮率 N/A / 採用率 N/A / 差戻率 0 / Integrator 負荷 中 / 安全事故 0

次 Wave 候補:
- 5/16 02:00 F282 試走 → 5/16 03:00 W44-pre drill 6 step
- Wave 52: HQ_APPROVE_SEND_MARKER 生成手順確定、wrapper script 配置
- 別 wave (低): launch plan / W43-pre doc 履歴整理
```

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE51_plan|W51 plan]]
- [[FIRE_CODEX_R1_WAVE44_5_POST_results|W44.5-post audit (= CRITICAL 検出元)]]
- [[../03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14|CLI v1.2 設計 doc]]
- [[../03_design/Production_v0_readiness_check_cli_v1_1_2026-05-14|W43-pre CLI v1.1 (= 前版)]]
- [[../03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14|cutover runbook v1.0 final]]
- 実装: `~/fire/scripts/jobs/run_production_v0_readiness_check.py`
- test: `~/fire/tests/scripts/jobs/test_run_production_v0_readiness_check.py`
