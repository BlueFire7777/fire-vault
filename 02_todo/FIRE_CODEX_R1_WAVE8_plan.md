---
id: FIRE-CODEX-R1-WAVE8-plan
phase: ガバナンス / Codex 並列実装 Wave 8 起票 / R-01-08 整合
priority: 最優先
status: 起票 ☆ Wave 8 7 sub-task 並列実行中 (2026-05-11)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 7 (= 完了)
  - HQ Wave 8 approve (= 2026-05-11、W8-0-fix + REPORT-R1 impl + DATA-R3 sub-D2.3 --dry-run 承認)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 8: W8-0-fix + REPORT-R1 impl + DATA-R3 sub-D2.3 --dry-run

最終更新: 2026-05-11

## ★ 状態: 起票 (= 7 sub-task 並列実行中、impl 4 + audit 3)

HQ Wave 8 approve 受領後、Architect 判断で 7 sub-task を起票。W8-0-fix
が W8-1 staging UPDATE smoke prerequisite として最優先、REPORT-R1 impl
と DATA-R3 sub-D2.3 --dry-run は並走。

**全 sub-task は read-only or staging-only**:
- W8-1 (= W8-0-fix): 既存 W6 runner の hardening、新規 DB write は発生せず
- W8-2/W8-3 (= REPORT-R1): read-only aggregators + Markdown renderer +
  dry-run runner (= LINE 送信なし、DB write なし)
- W8-4 (= DATA-R3 sub-D2.3 --dry-run): 4 runner に --dry-run option 追加、
  connection probe only、fetch/write 不発生

W4.1-B / W8-1 staging smoke 実行 / cron sub-D3 は引き続き凍結。

## Wave 8 7 sub-task 構成

| sub  | lane | task | 成果物 | HQ 承認状態 |
|------|------|------|--------|------------|
| W8-1 | L3+L2 (Codex) | W8-0-fix impl + tests | fire code change | 着手 approve |
| W8-2 | L3+L2 (Codex) | REPORT-R1 aggregators impl + tests | fire code change | 着手 approve |
| W8-3 | L3+L2 (Codex) | REPORT-R1 daily_report + markdown + runner + tests | fire code change | 着手 approve |
| W8-4 | L3+L2 (Codex) | DATA-R3 sub-D2.3 --dry-run × 4 runner + tests | fire code change | 着手 approve |
| W8-5 | L4 (Codex) | W8-1 (= W8-0-fix) audit | audit report | 推奨枠 |
| W8-6 | L4 (Codex) | REPORT-R1 audit | audit report | 推奨枠 |
| W8-7 | L4 (Codex) | DATA-R3 sub-D2.3 audit | audit report | 推奨枠 |

## W8-1 W8-0-fix 詳細

W7-3 audit で検出された HIGH #1 + MEDIUM #1 + MEDIUM #2 を解消:

### HIGH #1: TOCTOU race on `--db-path`

**現状**: `run_f286_pnl_r3_compute_paper_pnl.py` で `parse_args()` 内の
symlink / hardlink / basename 検査と `_connect_write()` の write open までに
window があり、検査後に DB path を差し替え可能。

**修正方針**:
1. `_connect_write(db_path)` 内で write open 直前に
   `_assert_db_path_not_symlink_attack(db_path)` を再実行
2. 可能なら `os.open(db_path, O_NOFOLLOW)` で fd 取得、fd の `st_dev/st_ino`
   を forbidden DB と比較 (= 最低でも staging basename 一致再確認)
3. write transaction 直前に再 stat (= cursor.execute('BEGIN') の直前)

### MEDIUM #1: output path TOCTOU

**現状**: `--output-json` / `--completion-report` も parse_args() で検査するが、
`Path.write_text()` 直前に再検査しない。

**修正方針**:
1. `_write_outputs()` 直前で `_assert_safe_output_path()` を再実行
2. atomic create (`open(path, 'x')` mode、O_EXCL) で既存 symlink を踏まない
3. または temp file + `os.replace()` パターン

### MEDIUM #2: decision_label exact match silent skip

**現状**: `should_compute_paper_pnl()` と runner SQL filter が include label と
完全一致のみ、whitespace / NFKC / 不可視文字混入で silent skip。

**修正方針**:
1. `pnl/paper_pnl.py` に `_normalize_decision_label(s) -> str` を追加:
   ```python
   import unicodedata
   def _normalize_decision_label(s: str) -> str:
       if s is None:
           return ""
       return unicodedata.normalize("NFKC", str(s).strip())
   ```
2. `should_compute_paper_pnl(decision_label)` / SQL filter で normalize 適用
3. runner で `decision_label NOT IN normalize(include) AND NOT IN
   normalize(exclude)` の残件 count を output JSON に記録 (= audit query)

### 期待 tests

- TestDBPathTOCTOURecheck: write 直前再検査が機能
- TestOutputPathAtomicCreate: 'x' mode で既存 symlink refuse
- TestDecisionLabelNormalize: whitespace / NFKC 入力で計算対象を正しく拾う
- TestDecisionLabelAuditCount: include/exclude 以外の残件 count を返す
- 既存 test 全 PASS 維持

### 安全制約

- `pnl/paper_pnl.py` の API 互換性維持 (= 既存 caller 影響なし)
- `run_f286_pnl_r3_compute_paper_pnl.py` の UPDATE 範囲は変えない
  (= paper_pnl + updated_at のみ)
- staging-only ガード強化、production / develop refuse 動作は変更なし
- LINE / 楽天 / Computer Use 不使用、token 不参照

## W8-2 REPORT-R1 aggregators

### scope

- `fire/report/__init__.py` 作成
- `fire/report/aggregators.py` 作成
  - `aggregate_daily_advisory(conn, base_date) -> DailyAdvisoryAggregate`
  - `aggregate_daily_paper_pnl(conn, base_date) -> DailyPaperPnlAggregate`
  - `aggregate_daily_actual_pnl(conn, base_date) -> DailyActualPnlAggregate`
    (= actual_trades テーブル未実装の場合は空 dataclass 返却)
  - `fetch_goal_progress(conn, base_date) -> GoalProgress` (= F210 既存連携)
- `fire/tests/report/test_aggregators.py` 作成
- すべて **read-only** (= SQL は SELECT のみ)

### 安全制約

- DB write 0
- LINE 不 import
- requests / aiohttp / 楽天 / Playwright 不 import
- token / channel_token / secret 不参照
- F286 R1/R2/R3 既存 table の column を **read のみ**

### 期待 tests

- aggregate_daily_advisory: label 別件数集計 OK
- aggregate_daily_paper_pnl: paper_pnl 集計 + None count
- aggregate_daily_actual_pnl: actual_trades 未実装時の空 dataclass
- fetch_goal_progress: F210 GoalConfig fallback
- 15-25 件追加見込

## W8-3 REPORT-R1 daily_report + runner

### scope

- `fire/report/markdown_renderer.py` 作成 (= W7-4 設計 §2.3 skeleton)
- `fire/report/daily_report.py` 作成 (= aggregators を組み合わせて Markdown 生成)
- `fire/scripts/jobs/run_f286_report_r1_daily.py` 作成
  - read-only DB open (= `file:?mode=ro` + `PRAGMA query_only=ON`)
  - `--db-label` / `--db-path` / `--base-date` / `--output-md` 必須
  - `--dry-run` flag (= 集計値計算のみ、Markdown ファイル生成しない)
  - 出力 path は DB / WAL / SHM 不可、symlink refuse + hardlink inode check
  - 'x' mode で atomic create (= W8-1 で導入する safety pattern を流用)
- `fire/tests/report/test_daily_report.py` + `test_markdown_renderer.py` +
  `tests/scripts/jobs/test_run_f286_report_r1_daily.py` 作成

### 安全制約

- DB write 0 (= read-only enforcement)
- LINE 送信 0 (= 本 runner は Markdown ファイル生成のみ、送信は別 sub-task)
- Markdown format: iPhone コピー対応 (= 単一コードフェンス + 内部 fence
  禁止、Fujiwara feedback memory 参照)
- 出力 path 安全 (= DB/WAL/SHM/secret path 不可)

### 期待 tests

- markdown_renderer: skeleton format 正しい
- daily_report: aggregators integration、normal / no-data / partial data
- runner: --dry-run / --base-date 過去日 / --output-md 妥当 path
- 20-35 件追加見込

## W8-4 DATA-R3 sub-D2.3 --dry-run × 4 runner

### scope

4 runner 各々に **統一仕様 --dry-run option** を追加:

| runner | --dry-run 挙動 | exit 0 条件 |
|--------|-------------|---|
| `scripts/jobs/fetch_historical_market_data.py` | env + DB path + J-Quants V2 auth probe ping | auth ping 成功 |
| `scripts/jobs/fetch_announcements.py` | env + DB path + TDnet/J-Quants V2 ping | ping 成功 |
| `scripts/jobs/run_daytrade_selection.py` | env + DB path + Feature Store 接続 | DB read 成功 |
| `scripts/jobs/run_f119_interpretation_evaluation.py` | env + DB path + 必要 table 存在確認 | DB read 成功 |

### 統一仕様

```python
parser.add_argument(
    "--dry-run",
    action="store_true",
    help="Connection probe only (no fetch / no write). "
         "Validates config, DB path, env vars, and API auth without "
         "executing actual fetch or write. "
         "Exit 0 = probe success / 1 = config error / 2 = connectivity fail",
)
# 本体:
if args.dry_run:
    perform_connection_probe()
    return DRY_RUN_OK_EXIT  # = 0
```

### 安全制約

- `--dry-run` 時 fetch / write 不発生
- probe 用 API call は 1 回のみ (= rate limit / 課金最小化)
- token / channel_token は probe でも env 経由で読まない (= 既存 auth flow
  維持、新規 secret 読出経路追加禁止)
- staging-only / production refuse 動作は変更なし

### 期待 tests

- 各 runner の --dry-run option パース
- --dry-run 時 fetch helper が呼ばれないこと (= mock で検証)
- exit code = 0 / 1 / 2 分岐
- 各 runner 10-15 件、合計 40-60 件追加見込

## W8-5 / W8-6 / W8-7 audit

各 impl の audit (= L4、Codex 独立視点):

- W8-5: W8-0-fix の TOCTOU 解消 + 既存 contract 維持確認
- W8-6: REPORT-R1 の read-only enforcement + Markdown safety
- W8-7: DATA-R3 sub-D2.3 --dry-run の fetch/write 不発生確認

各 audit は read-only static review、code change なし。

## 安全制約 (= Wave 8 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信 | 0 通維持 |
| W8-1 staging UPDATE smoke 実行 | 凍結 (= HQ 明示承認後 Wave 8b 別途) |
| production DB write | 0 |
| develop DB write | 0 |
| staging DB write | 0 (= Wave 8 内 hardening + impl only) |
| token / channel_token / secret 参照 | 0 |
| W4.1-B F062 経由 smoke | 保留継続 |
| 楽天 / 自動発注 / Computer Use / Playwright | なし |
| cron / launchd / crontab 本番登録 | 0 (= sub-D3 凍結継続) |
| .github/workflows/ 変更 | 0 |
| --no-verify | 不使用 |
| scripts/seed_pattern_layer1.py | 未接触 |
| simulation/research_lane/historical_indicators.py | 未接触 |
| TODO Excel | 未更新 |
| Codex 直接 commit | 0 (= 本線 Integrator が全 commit) |

## 並列実行方針

- **Phase 1 (= 並列起動)**:
  - Codex Lane 1: W8-1 W8-0-fix impl + tests
  - Codex Lane 2: W8-2 REPORT-R1 aggregators impl + tests
  - Codex Lane 3: W8-4 DATA-R3 sub-D2.3 --dry-run × 4 runner + tests
- **Phase 2 (= W8-2 完了後)**:
  - Codex Lane 4: W8-3 REPORT-R1 daily_report + markdown + runner + tests
- **Phase 3 (= impl 完了後、audit を並列起動)**:
  - Codex Lane 5: W8-5 W8-0-fix audit
  - Codex Lane 6: W8-6 REPORT-R1 audit
  - Codex Lane 7: W8-7 DATA-R3 sub-D2.3 audit
- **Phase 4 (= 本線 Integrator)**:
  - 全 review + CRITICAL 即修正 + split commits + log + HQ 報告

最大同時 Codex lane: **3 並列** (= Phase 1 / Phase 3)。Wave 6 の 4 lane 並列より
control しやすい。

## 受入基準 (= Wave 8 完了条件)

- [ ] 4 impl sub-task の code + tests 完成
- [ ] 各 audit で CRITICAL 検出が出たら即修正
- [ ] tests 全 PASS (= 3,637 baseline + 75-120 追加見込み、合計 ~3,750+)
- [ ] DB writes 0 / LINE sends 0 / secrets 0 / forbidden files 0
- [ ] fire develop split commit
- [ ] fire-vault main split commit (= plan + results + log + audit reports)
- [ ] CLAUDE.md 完了 table に Wave 8 entry 4+ 件追加
- [ ] HQ 1 block report 提示

## Wave 9 候補 (= HQ approve 後)

- W9-1: W8-1 staging UPDATE smoke 実行 (= HQ 明示承認後)
- W9-2: REPORT-R1 weekly / monthly impl + tests + audit
- W9-3: REPORT-R1 LINE 配信 helper (= F062/F236 経由、別 HQ approve)
- W9-4: DATA-R3 sub-D2.3.x staging write 個別 (= 4 runner × 個別 HQ approve)
- 並走候補: FIRE-AUDIT-R1 v1.2 (= AST + docstring context 除外強化)
- 凍結継続: sub-D3 cron 登録 / W4.1-B F062 経由 smoke

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE7_plan|Wave 7 plan]]
- [[FIRE_CODEX_R1_WAVE7_results|Wave 7 results]]
- [[../07_incidents/F286_PNL_R3_audit_2026-05-11|W7-3 audit report (= W8-0-fix source)]]
- [[../03_design/F286_PNL_R3_staging_update_smoke_plan_2026-05-11|W7-1 W8-1 smoke plan]]
- [[../03_design/F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|W7-2 W8-4 source]]
- [[../03_design/F286_REPORT_R1_daily_pnl_report_generator_2026-05-11|W7-4 REPORT-R1 design]]
- [[../log]]
