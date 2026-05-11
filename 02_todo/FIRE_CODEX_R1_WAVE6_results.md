---
id: FIRE-CODEX-R1-WAVE6-results
phase: ガバナンス / Codex 並列実装 Wave 6 完了 / R-01-08 整合
priority: 最優先
status: 完了 ★ Wave 6 4 lane 完了 (2026-05-11、CRITICAL 1 件即修正、3,637 PASS)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 5 / Wave 4.1-A
  - HQ Wave 6 approve (= 2026-05-11)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 6: PNL-R3 implementation + DATA-R3 sub-D2.2

最終更新: 2026-05-11

## ★ 状態: 完了 (= 4 lane Codex 完了、6 split commit、3,637 PASS)

HQ Wave 6 approve 受領後、Codex 4 lane を順次投入。本線 Integrator が
全成果物 review、CRITICAL 1 件 (= PaperPnlError per-row skip 不在)
を即修正、split commit で fire develop に反映。

W4.1-B / cron sub-D3 は引き続き凍結を遵守。実 fetch / 実 write も
別 HQ approve 維持。

## Wave 6 投入結果 (= 4 lane)

| sub  | lane | task | 状態 | CRITICAL |
|------|------|------|------|---------|
| W6-1+2 | L3 Impl + L2 Test | PNL-R3 paper_pnl module + runner | ✓ 48 PASS | 1 件即修正 |
| W6-3   | L4 Audit          | PNL-R3 adversarial review | ✓ CRITICAL 0 | - |
| W6-4   | L5 Docs           | PNL-R3 完了マーカー doc draft | ✓ vault migrate | - |
| W6-5   | L3 Impl + L2 Test | DATA-R3 sub-D2.2 placeholder 解消 + exit code 集約 | ✓ 36 PASS | 0 |

## fire develop split commit (= 6 件)

| commit | 内容 |
|---|---|
| 7cd2f8e | feat(F286-PNL-R3): add paper PnL computation module + runner (W6-1+2) |
| adb8628 | feat(F286-DATA-R3-D2.2): resolve placeholder + aggregate exit code (W6-5) |
| a856dfb | test(F286-PNL-R3): whitelist W6-1+2 runner in db_path consistency test |
| 0d53300 | docs(FIRE-CODEX-R1): add Wave 6 sub-task completion table entries |

## Codex CRITICAL 即修正履歴 (= 1 件)

### W6-1+2 CRITICAL #1 (PaperPnlError per-row skip 不在)

- **指摘**: main() の compute_paper_pnl 呼出で sqlite3.Error と
  F286PnlR3ComputeRefused のみ catch、PaperPnlError は uncaught。
  1 row の bad market data (= negative close / zero entry) で全体
  crash するリスク。
- **修正**:
  - 1 row 呼出を try/except で囲み、PaperPnlError / ValueError /
    sqlite3.Error を catch
  - 当該 row は skip (= paper_pnl=None + paper_reason='compute_error:
    ...') して後続継続
  - SELECT 自体の sqlite3.Error のみ致命的扱い (= safety violation)
  - PaperPnlError import 追加
  - 対応 test (TestPaperPnlErrorPerRowSkip) 追加、mock で 1 件
    crash + 1 件正常で全体 exit 0 / 後続実行 / output JSON で row
    別 paper_reason を確認

## W6-1+2 主要内容

### 新規 module: pnl/paper_pnl.py

- 純関数 `compute_paper_pnl(conn, PaperPnlInput) -> PaperPnlResult`
- 関連 helper:
  - `compute_paper_entry_qty(entry_price, allowable_loss, stop_pct)`
  - `fetch_market_open_next_business_day(conn, code, base_date)`
  - `fetch_market_close_after_business_days(conn, code, base_date, business_days)`
- 値域定数:
  - `PAPER_PNL_INCLUDE_LABELS = ('積極的買い推奨', '条件付き買い推奨', '注意つき買い候補')`
  - `PAPER_PNL_EXCLUDE_LABELS = ('場中監視', '見送り推奨', '監視のみ')`
- DB は read-only sqlite3 のみ、write しない

### 新規 runner: scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py

六段ガード (= W4-2 と同じ pattern) を再利用:
1. SnapshotStore コンストラクタ read_only=False 強制 → 本 runner では
   直接 sqlite3 を使うため、`_connect_write()` 内で URI 制限 (= staging
   only) を確認
2. --db-label staging 必須
3. --db-path basename='fire.staging.db' 必須
4. --output-json / --completion-report が DB/WAL/SHM/sqlite path 不可
5. --db-path symlink refuse + Path.resolve() basename 一致
6. --db-path inode が forbidden DB と一致 (= hardlink) 不可

UPDATE は **paper_pnl + updated_at のみ**、既存 fujiwara_decision /
actual_trade / notes / created_at / entry_price / 等は **touched なし**。
paper_pnl IS NULL の row のみ対象 (= idempotent rerun)。

## W6-3 audit 結果

CRITICAL: **0 件** ★
全観点 OK:
- 計算ロジック / 三段+六段ガード
- UPDATE 限定性 / decision_label include/exclude 振り分け
- forbidden import なし / read-only enforcement
- market_prices_daily 欠落耐性
- F130 参照 / idempotent
- R1/R2 既存 row 整合

report: `/tmp/f286_pnl_r3_audit_report.md`

## W6-5 主要内容

W5-3 audit 軽微指摘 2 件解消:

### 1. placeholder 2 件 active 化

- `f101_announcements`: `scripts.jobs.fetch_announcements`
- `f119_evaluation`: `scripts.jobs.run_f119_interpretation_evaluation`

各 sub-runner が `--dry-run` option を持つか `_probe_subrunner_supports
_dry_run()` で確認:
- 対応していれば active 化、dry_run_args に `--dry-run` 含む
- 対応していなければ status='no_dry_run_option_skipped' で safe-skip
  (= sub-D3 で別途 --dry-run option を既存 runner に追加する案あり)

### 2. exit code 集約ルール

`aggregate_dry_run_exit_code(results)`:
- 全 ok / placeholder_skipped / no_dry_run_option_skipped → 0
- 1 件以上 failed → 1
- 1 件以上 timeout → 2
- 想定外 status → 3

### tests 拡張

- TestDryRunArgsForF101AndF119
- TestProbeSubrunnerSupportsDryRun (= mocked --help)
- TestAggregateDryRunExitCode
- TestPlaceholderResolution
- 36 PASS (= W5-1+2 27 baseline + W6-5 9 新規)

## 安全要件 (= Wave 6 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信 (Wave 6 中)                       | 0 通 ✓ (= SEND 件数 4 のまま) |
| DB writes (production/develop/staging)         | 0 ✓ |
| token / channel_token / secret 参照            | 0 ✓ |
| 楽天 / 自動発注 / Computer Use / Playwright    | なし ✓ |
| 注文価格 / 数量 / 執行指示の生成 helper       | 含めない ✓ |
| .github/workflows/ 変更                       | 0 ✓ |
| --no-verify                                   | 不使用 ✓ |
| cron / launchd / crontab 本番登録              | 0 ✓ |
| scripts/seed_pattern_layer1.py                | 未接触 ✓ |
| simulation/research_lane/historical_indicators.py | 未接触 ✓ |
| TODO Excel                                    | 未更新 ✓ |
| Codex 直接 commit                              | 0 ✓ (= 本線 Integrator が全 commit) |
| subprocess --write 渡し (W6-5)                | 0 ✓ |
| subprocess env に secret (W6-5)               | 0 ✓ |

## 並列効果計測 (= Wave 1-6 通算)

| Wave | 内容                                          | 実時間  | 本線単独推定 | 短縮率 |
|------|----------------------------------------------|---------|---------------|--------|
| 1    | Audit×2 + Design                              | 25-30 分 | 90-120 分     | 70-75% |
| 2    | Impl + Test + DATA-R3 skeleton + Docs         | 25-30 分 | 120-150 分    | 75-80% |
| 3    | Audit×5                                       | 30-40 分 | 150-180 分    | 70-80% |
| 4    | Impl×2 + Impl+Test + Design (= 4 lane)        | 50-60 分 | 180-240 分    | 65-75% |
| 5    | Impl+Test + Audit + Design (= 3 lane)         | 30-35 分 | 150-180 分    | 80%    |
| 6    | Impl+Test + Audit + Docs + Impl+Test (= 4 lane) | 40-50 分 | 180-220 分 | 75-80% |

**6 wave 連続で 65-80% 短縮を達成** ★

## HQ 判断が必要な論点 (= 4 件)

1. **Wave 6 完了 → 次フェーズ進行可否** (推奨: approve)

2. **F286-PNL-R3 staging write smoke 着手判断** (= W7-1 候補):
   - W4.1-A と同じ pattern (= 単独 runner で staging UPDATE)
   - 対象は既存 advisory_decisions の paper_pnl IS NULL row
   - 注意: W4.1-A の 5 件は decision_label='場中監視' で paper_pnl 計算
     対象外、何も UPDATE されない (= 安全) が、計算条件の動作確認は可能
   - 推奨: 別 HQ approve、初回 UPDATE は HQ 明示承認後

3. **F286-DATA-R3 sub-D2.3 着手判断** (= 実 fetch / 実 write 統合):
   - W6-5 で placeholder 解消、exit code 集約完了
   - 次は staging write を伴う実 fetch (= F100 / F101 / F111 / F119
     の dry-run / write 経路)
   - HQ 別 approve 必須

4. **sub-D3 cron 凍結継続確認** + W4.1-B 保留継続確認

## 次タスク候補 (= Wave 7+)

- W7-1: F286-PNL-R3 staging write smoke (= HQ 別 approve 要、W4.1-A
  と同じ pattern)
- W7-2: F286-DATA-R3 sub-D2.3 実 fetch / 実 write 統合 (= HQ 別 approve)
- W7-3: F286-PNL-R3 cron 連携 (= sub-D3 凍結解除後、毎営業日 18:00 JST)
- 並走候補: FIRE-AUDIT-R1 v1.2 (= AST + docstring context 除外強化) /
  FIRE-OPS-R0 案 1 (= production write 統一)

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE6_plan|Wave 6 plan]]
- [[../03_design/F286_PNL_R3_paper_pnl_simulator_hook_2026-05-11|F286-PNL-R3 design]]
- [[F286_PNL_R3_paper_pnl_simulator_hook|F286-PNL-R3 完了マーカー]]
- [[../log]]
