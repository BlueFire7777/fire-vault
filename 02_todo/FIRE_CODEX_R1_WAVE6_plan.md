---
id: FIRE-CODEX-R1-WAVE6
phase: ガバナンス / Codex 並列実装 Wave 6 / R-01-08 整合
priority: 最優先
status: 起票 ★ (= 2026-05-11、HQ Wave 6 approve 受領、4 lane 並列投入予定)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 5 (= sub-D2 dry-run subprocess routing 完成)
  - F286-PNL-R3 W5-4 design draft (= HQ 仮承認)
  - HQ Wave 6 approve (= 2026-05-11)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 6: F286-PNL-R3 implementation + DATA-R3 sub-D2.2

最終更新: 2026-05-11

## ★ 状態: 起票 (= 4 lane 並列投入準備完了)

HQ Wave 6 approve 受領内容:
- PNL-R3 implementation / tests / audit / docs 進行可
- DATA-R3 sub-D2.2 dry-run enhancement / test 進行可
- ただし **実 fetch / 実 write / staging DB write はまだ未承認**
- LINE 送信 / W4.1-B / cron 登録は引き続き禁止

PNL-R3 設計の HQ 仮承認内容:
- 仮想エントリ価格: 翌営業日寄付
- 仮想エグジット: h20 後の終値
- 仮想数量: F130 許容損失で逆算
- 計算対象: 🟢 積極的買い推奨 / 🟡 条件付き買い推奨 / ⚠️ 注意つき買い候補
- 対象外: 🟠 場中監視 / 🔴 見送り推奨 / ⚪ 監視のみ

## Wave 6 sub-task 一覧 (= 4 lane)

| sub  | parent | lane | branch | DB write |
|------|--------|------|--------|----------|
| W6-1+2 | F286-PNL-R3 | L3 Impl + L2 Test (一括) | codex/f286-pnl-r3-impl-tests | **しない** |
| W6-3 | F286-PNL-R3 | L4 Audit | codex/f286-pnl-r3-audit | **しない** (= read-only) |
| W6-4 | F286-PNL-R3 | L5 Docs | codex/f286-pnl-r3-docs | **しない** |
| W6-5 | F286-DATA-R3-D2.2 | L3 Impl + L2 Test (一括) | codex/f286-data-r3-d2-2-enhancement | **しない** |

### W6-1+2: F286-PNL-R3 impl + tests

新規 module + runner + tests:
- `pnl/paper_pnl.py` (新規): 純関数 `compute_paper_pnl()` + `compute
  _entry_qty()` + helper、F130 ステージ別許容損失 import (= 既存 risk
  module から?)、market_prices_daily からの価格取得は db read-only
- `scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py` (新規 runner):
  default dry-run + --write で staging UPDATE (= advisory_decisions
  .paper_pnl のみ)
- 六段ガード再利用 (= W4-2 と同じ pattern)
- 新規 ingest しない、既存 advisory_decisions row の paper_pnl のみ UPDATE
- 既存 Fujiwara 判断 / 実取引列は touched なし

allowed_files:
- pnl/paper_pnl.py
- scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py
- tests/pnl/test_paper_pnl.py
- tests/scripts/jobs/test_run_f286_pnl_r3_compute_paper_pnl.py

### W6-3: F286-PNL-R3 audit

- W6-1+2 完成後の adversarial audit
- 計算ロジック / 六段ガード / forbidden import / 注文 helper 不在
- read-only AST 走査

allowed_files:
- /tmp/f286_pnl_r3_audit_report.md

### W6-4: F286-PNL-R3 完了マーカー docs

- 02_todo/F286_PNL_R3_paper_pnl_simulator_hook.md (= 完了マーカー、
  frontmatter + summary)
- Codex の sandbox 制約があれば本線が代替作成 (= W4-5 と同様)

allowed_files:
- /tmp/f286_pnl_r3_completion_marker_draft.md (= Codex は /tmp/ 経由、
  本線が vault へ migrate)

### W6-5: DATA-R3 sub-D2.2 dry-run enhancement

W5-3 audit 軽微指摘 2 件対応 (= sub-D2.2 申送り):
1. **placeholder 2 件解消**:
   - `f101_announcements`: scripts/jobs/fetch_announcements.py (= 既存
     確認済) の dry-run args を追加
   - `f119_evaluation`: scripts/jobs/run_f119_interpretation_evaluation.py
     (= 既存) の dry-run args を追加
2. **exit code 集約ルール**:
   - 各 sub-job exit code を集約、全 OK → 0、いずれか failed → 1、
     timeout → 2 等の明確化
3. **dry-run tests 拡張**:
   - 全 4 sub-runner (= f100 / f111 / f101 / f119) の dry-run 経路 test

★ **実 fetch / 実 write / staging write は本タスクで実装しない**
(= HQ approve 必要)。subprocess は **--dry-run / --check のみ** で実
コマンドは呼ぶ (= 既存 sub-runner の dry-run 経路を試して動作確認)。
ただし tests では mocked subprocess を使用、test 中の実コマンド実行は
mock or controlled tmp_path 内のみ。

allowed_files:
- scripts/jobs/run_f286_data_r3_daily_refresh.py (= 既存修正、W5-1+2 続き)
- tests/scripts/jobs/test_run_f286_data_r3_daily_refresh.py (= 既存修正)

## 並列度 (= 4 sub 並行投入予定、ファイル衝突なし)

- W6-1+2: pnl/paper_pnl.py + scripts/jobs/run_f286_pnl_r3_* + tests
- W6-3: /tmp/ audit report
- W6-4: /tmp/ completion marker draft
- W6-5: scripts/jobs/run_f286_data_r3_daily_refresh.py + tests
  (= W6-1+2 の pnl/* と独立、衝突なし)

実時間想定: 約 35-50 分 (= 4 lane 順次 + 本線 Integrator)。

## 共通安全要件 (= Wave 6 全 sub に適用)

- 未承認 LINE 送信: 禁止
- production / develop DB write: 禁止
- 未承認 staging DB write: 禁止 (= W6-1+2 のみ将来 --write 経路実装、
  本 Wave で実行はしない)
- token / channel_token / secret 参照: 禁止
- 楽天証券操作 / 自動発注 / Computer Use / Playwright: 禁止
- GitHub Actions workflow 変更: 禁止
- --no-verify: 禁止
- scripts/seed_pattern_layer1.py: 未接触必須
- simulation/research_lane/historical_indicators.py: 未接触必須
- TODO Excel 更新: 禁止
- cron / launchd / crontab 本番登録: 禁止 (= 凍結継続)
- 注文価格 / 数量 / 執行指示の生成 helper: 含めない (= PNL-R3 は
  paper_pnl 計算のみ、注文系は別 task で常時除外)

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE5_results|Wave 5 results]]
- [[../03_design/F286_PNL_R3_paper_pnl_simulator_hook_2026-05-11|F286-PNL-R3 design draft]]
- [[../log]]
