---
id: FIRE-CODEX-R1-WAVE5
phase: ガバナンス / Codex 並列実装 Wave 5 / R-01-08 整合
priority: 最優先
status: 起票 ★ (= 2026-05-11、HQ approve 受領、3 lane 並列投入予定)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 4 結果
  - Wave 4.1-A staging smoke 成功
  - HQ Wave 5 approve (= 2026-05-11)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 5: DATA-R3 sub-D2 dry-run + PNL-R3 設計

最終更新: 2026-05-11

## ★ 状態: 起票 (= 3 lane 並列投入準備完了)

HQ Wave 5 approve 受領 (= 2026-05-11、W4.1-A 完了承認後):
- DATA-R3 sub-D2 dry-run / tests / audit の並列開発進行可
- PNL-R3 Paper PnL Simulator Hook 設計起票進行可
- W4.1-B (F062 経由 staging smoke) は **保留継続**
- cron / launchd / crontab 本番登録は **凍結継続**

## Wave 5 sub-task 一覧 (= 3 lane)

| sub  | parent           | lane                       | branch                            | DB write          |
|------|------------------|----------------------------|-----------------------------------|---------------------|
| W5-1+2 | F286-DATA-R3-D2 | L3 Impl + L2 Test (一括)   | codex/f286-data-r3-d2-dry-run     | **しない**           |
| W5-3 | F286-DATA-R3-D2  | L4 Audit                   | codex/f286-data-r3-d2-audit       | **しない** (= read-only) |
| W5-4 | F286-PNL-R3      | L1 Design                  | codex/f286-pnl-r3-design          | **しない** (= 設計のみ) |

### W5-1+2: DATA-R3 sub-D2 dry-run plan impl + tests

詳細:
- 既存 `scripts/jobs/run_f286_data_r3_daily_refresh.py` (= skeleton、
  commit e9c41c3) を **dry-run plan 強化** に拡張
- `plan_daily_refresh(read_only=True)` で各 sub-runner の --dry-run 経路を
  順次 subprocess 呼び出し、想定 row 数を集計
- **実 fetch / 実 write は実装しない** (= sub-D2.2 / sub-D2.3 で別 approve 後)
- subprocess 呼出は `--dry-run` 経路のみ (= 副作用ゼロ)

allowed_files:
- scripts/jobs/run_f286_data_r3_daily_refresh.py (= 既存修正)
- tests/scripts/jobs/test_run_f286_data_r3_daily_refresh.py (= 既存修正)

forbidden_files: 全 既存 sub-runner / pnl/* / notifications/* / 全
既存禁止リスト

### W5-3: DATA-R3 sub-D2 audit

- W5-1+2 完成後の audit
- subprocess 呼出 / env 注入の安全 audit
- forbidden import / token 漏洩 / 実 fetch なし確認

allowed_files:
- /tmp/f286_data_r3_d2_audit_report.md (= 新規 report)

forbidden_files: 全 source ファイル書き換え禁止 (= read-only audit)

### W5-4: F286-PNL-R3 設計起票

- Paper PnL Simulator Hook 設計
- 「もし採用していたら」シミュ PnL を simulation/paper_live と連携
- F286-PNL-R1 advisory_decisions.paper_pnl 列を自動計算で埋める設計
- 実装本体は Wave 6+

allowed_files:
- /tmp/f286_pnl_r3_design_draft.md (= 新規 design draft)

forbidden_files: 全 source ファイル (= 設計のみ、実装変更なし)

## 並列度 (= 3 sub 並行投入)

| Wave 5 phase | 並列 lane 数 | 想定時間 |
|---|---|---|
| 並列実行 (W5-1+2 + W5-3 + W5-4) | 3 lane 同時 (= 順次でも可) | 約 20-30 分 |
| 本線 Integrator review            | 1                          | 約 10-15 分 |
| Wave 5 完了 → HQ 報告           | 1                          | 約 5-10 分 |

合計: 約 35-55 分。

ファイル衝突: なし
- W5-1+2: scripts/jobs/run_f286_data_r3_daily_refresh.py + tests
- W5-3: /tmp/ audit report
- W5-4: /tmp/ design draft

## W4.1-B 保留 (= HQ 別 approve 要)

W4.1-B (F062 経由 staging smoke) は **保留継続**:
- LINE 1 通送信 (= token 実使用) を含む
- F062-R5.8 同一 payload 再送信のため Fujiwara LINE app への重複通知を
  許容する明示承認要

Wave 5 では着手しない。

## 共通安全要件 (= Wave 5 全 sub に適用)

- 未承認 LINE 送信: 禁止
- production / develop DB write: 禁止
- 未承認 staging DB write: 禁止 (= W5-1+2 / W5-3 / W5-4 全件)
- token / channel_token / secret 参照: 禁止
- 楽天証券操作 / 自動発注 / Computer Use / Playwright: 禁止
- GitHub Actions workflow 変更: 禁止
- --no-verify: 禁止
- scripts/seed_pattern_layer1.py: 未接触必須
- simulation/research_lane/historical_indicators.py: 未接触必須
- TODO Excel 更新: 禁止
- cron / launchd / crontab 本番登録: 禁止 (= 凍結継続)

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE4_results|Wave 4 results]]
- [[FIRE_CODEX_R1_WAVE4_1A_staging_smoke|Wave 4.1-A 結果]]
- [[F286_DATA_R3_D2_real_fetch_write_plan|F286-DATA-R3-D2 plan]]
- [[../03_design/F286_DATA_R3_D2_real_fetch_write_2026-05-11|F286-DATA-R3-D2 design draft]]
- [[../log]]
