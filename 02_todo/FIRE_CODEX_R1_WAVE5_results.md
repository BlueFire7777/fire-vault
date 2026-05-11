---
id: FIRE-CODEX-R1-WAVE5-results
phase: ガバナンス / Codex 並列実装 Wave 5 完了 / R-01-08 整合
priority: 最優先
status: 完了 ★ Wave 5 3 lane 完了 (2026-05-11、CRITICAL 0、3,579 PASS)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 4 / Wave 4.1-A
  - HQ Wave 5 approve (= 2026-05-11)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 5: DATA-R3 sub-D2 dry-run + PNL-R3 設計

最終更新: 2026-05-11

## ★ 状態: 完了 (= 3 lane Codex 完了、CRITICAL 0、3 split commit)

HQ Wave 5 approve 受領後、Codex 3 lane を順次投入。W4.1-B は保留継続、
cron 凍結継続を遵守して、subprocess `--dry-run` 経路のみで実装。

## Wave 5 投入結果 (= 3 lane)

| sub | lane | task | 状態 |
|---|---|---|---|
| W5-1+2 | L3 Impl + L2 Test | DATA-R3 sub-D2 dry-run plan subprocess routing | ✓ 27 PASS |
| W5-3 | L4 Audit | DATA-R3 sub-D2 audit | ✓ CRITICAL 0 |
| W5-4 | L1 Design | F286-PNL-R3 Paper PnL Simulator Hook 設計 | ✓ 416 行 draft |

## fire develop split commit (= 3 件)

| commit | 内容 |
|---|---|
| 46cabe1 | feat(F286-DATA-R3-D2): integrate dry-run plan subprocess routing (W5-1+2) |
| d9b3c2f | docs(F286-PNL-R3): paper PnL simulator hook design draft (W5-4) [vault] |
| e18cb4a | docs(FIRE-CODEX-R1): add Wave 5 sub-task completion table entries |

Codex pre-commit hook: 全 OK 通過 (= W4 と異なり CRITICAL 検出なし)。

## W5-1+2 主要内容

### 新規 / 拡張機能

- `list_daily_refresh_jobs()`: 既存 sub-runner を実 module / dry_run
  _args で定義
  - f100_historical (= scripts.jobs.fetch_historical_market_data)
  - f111_research_watchlist_signal_persistence
  - placeholder 2 件 (= f101_announcements, f119_evaluation、sub-D2.2
    で確認後追加)
- `dry_run_each_job()`: subprocess.run(args, env=最小, timeout=300)
  で各 job の --dry-run を順次実行、結果集約
- `plan_daily_refresh(execute_dry_run_subprocesses=True)`: 各
  sub-runner の dry-run を実 subprocess で呼ぶ拡張
- CLI: `--execute-dry-run-subprocesses` (= default off)

### 安全制約 (= 構造的に実装)

- subprocess args に **`--write` を絶対に渡さない** (= dry_run_args
  list 内 literal で `--dry-run` のみ)
- subprocess env に **token / channel_token / secret を含めない**
  (= 最小 env、PYTHONUNBUFFERED=1 のみ)
- subprocess timeout 必須 (= default 300 秒)
- subprocess shell=False (= shell injection 防止)

### tests 拡張

- TestListDailyRefreshJobsContent (= 4 entry、real_fetch_approved 全 False)
- TestDryRunEachJob (= placeholder skip、success / failed / timeout)
- TestPlanDailyRefreshDryRunWithSubprocess
- TestPlanDailyRefreshWriteNotImplemented (= read_only=False refuse)
- TestSubprocessSafety (= env / args 検査)
- 27 PASS (= 既存 16 + 新規 11)

## W5-3 audit 結果

| 観点 | 結果 |
|---|---|
| subprocess args 安全性                   | 1 call site / 条件付き OK |
| env 注入安全性                           | 1 call site / OK |
| timeout 設定                             | OK |
| forbidden import                         | なし |
| 既存 sub-runner 本体 touched              | なし |
| dry_run_args 構成                        | 1 件改善余地 (= sub-D2.2 で対応) |
| placeholder 取扱                          | OK |
| エラー伝搬                                | 継続実行 OK / exit code 集約改善余地 |
| F286-PNL-R1/R2 module 衝突                | なし (= pnl.storage import ありだが DB write は三段ガード経由のみ) |
| sub-D3 整合性                             | 2 件申送り (= cwd / WorkingDirectory 整合等) |

**CRITICAL なし**。軽微指摘 2 件は sub-D2.2 / sub-D3 で対応申送り。

report: `/tmp/f286_data_r3_d2_audit_report.md`

## W5-4 F286-PNL-R3 設計概要

設計 draft: `/Users/bluefire/fire-vault/03_design/F286_PNL_R3_paper
_pnl_simulator_hook_2026-05-11.md` (= 416 行)

### 推奨案 (= 本線 Architect 仮承認、HQ approve 待ち)

| 設計判断項目 | 案 | 推奨理由 |
|---|---|---|
| 仮想エントリ価格    | 案 a: 翌営業日寄付 | deterministic、market_prices_daily から取得可 |
| 仮想エグジット価格  | 案 x: h20 後の終値 | F119 model と整合 |
| 仮想数量            | 案 1: F130 許容損失で逆算 | 現実的、実取引と整合 |

### paper_pnl 計算対象

- 🟢 積極的買い推奨 / 🟡 条件付き買い推奨 / ⚠️ 注意つき買い候補 → 計算
- 🟠 場中監視 / 🔴 見送り推奨 / ⚪ 監視のみ → paper_pnl=None

### 実装計画 (= Wave 6+)

- W6-1 設計確定 (= 本書 Architect approve)
- W6-2 L3 Impl: scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py
- W6-3 L2 Test
- W6-4 L4 Audit
- W6-5 L5 Docs
- W7-1 cron 同時起動 (= sub-D3 凍結解除後)

## 安全要件 (= Wave 5 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信 (Wave 5 中)                       | 0 通 ✓ (= SEND 件数 4 のまま) |
| DB write                                       | 0 ✓ |
| production / develop / staging DB mtime         | 全 unchanged ✓ |
| token / channel_token / secret 参照            | 0 ✓ |
| 楽天 / 自動発注 / Computer Use                 | なし ✓ |
| .github/workflows/ 変更                       | 0 ✓ |
| --no-verify                                   | 不使用 ✓ |
| scripts/seed_pattern_layer1.py                | 未接触 ✓ (mtime May 7 17:39) |
| simulation/research_lane/historical_indicators.py | 未接触 ✓ (mtime May 9 21:10) |
| TODO Excel                                    | 未更新 ✓ |
| cron / launchd / crontab 本番登録              | 0 ✓ |
| Codex 直接 commit                              | 0 ✓ (= 本線 Integrator が全 commit) |
| subprocess --write 渡し                        | 0 ✓ (= 構造的に dry-run のみ) |
| subprocess env に secret                       | 0 ✓ (= 最小 env) |

## 並列効果計測 (= Wave 5)

- Wave 5 実時間: 約 30-35 分 (= 3 lane 順次 + Integrator review)
- 本線単独推定: 150-180 分
- 速度向上: **約 80% 短縮**

## W4.1-B 保留 / cron 凍結 (= 引き続き)

- W4.1-B (F062 経由 staging smoke): 保留継続、HQ 別 approve 要
- cron / launchd / crontab 本番登録: 凍結継続 (= sub-D3 で対応)

## HQ 判断が必要な論点 (= 4 件)

1. **Wave 5 完了 → 次フェーズ進行可否** (推奨: approve)
2. **F286-DATA-R3 sub-D2.2 着手判断** (= 実 fetch / 実 write 統合):
   - 現状 placeholder 2 件 (= f101 / f119) の sub-runner 確認 +
     dry-run args 追加
   - その後 `--write` 経路の実装を HQ 別 approve で sub-D2.2 起票
3. **F286-PNL-R3 sub-Impl 起票判断** (= Wave 6):
   - W5-4 設計 draft を Architect approve 後、Wave 6 として 5 sub-task
     並列で実装
4. **W4.1-B / cron sub-D3 凍結継続確認**

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE5_plan|Wave 5 plan]]
- [[../03_design/F286_DATA_R3_D2_real_fetch_write_2026-05-11|sub-D2 design]]
- [[../03_design/F286_PNL_R3_paper_pnl_simulator_hook_2026-05-11|F286-PNL-R3 design draft]]
- [[../log]]
