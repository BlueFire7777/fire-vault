---
id: FIRE-CODEX-R1-WAVE4
phase: ガバナンス / Codex 並列実装 Wave 4 plan / R-01-08 整合
priority: 最優先
status: 起票 ★ (= 2026-05-11、HQ approve 受領、Codex 投入は別 approve)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 (= 設計)
  - Wave 1 / Wave 2 / Wave 3 (= 連続 70-80% 短縮実証済)
  - Wave 3 audit クリア (= HQ 判定 2026-05-11)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 4: Plan

最終更新: 2026-05-11

## ★ 状態: 起票 (= Codex 投入準備完了、HQ approve 後に Wave 4 実装開始)

Wave 3 audit クリア (= HQ 判定 2026-05-11) を受け、Wave 4 で 4 sub-task
を並列起票。本タスクは **起票のみ**、Codex 投入は HQ approve 後に別途
実施。staging smoke は **Wave 4.1 として完全分離** (= HQ 別 approve 要)。

## Wave 4 sub-task 一覧 (= 4 件)

| sub | parent       | lane                       | branch                              | staging write |
|-----|--------------|----------------------------|-------------------------------------|---------------|
| W4-1 | F286-PNL-R2  | L3 Implementation          | codex/f286-pnl-r2-runner            | **しない**     |
| W4-2 | F286-PNL-R2  | L3 Implementation          | codex/f286-pnl-r2-ingest-helper     | **しない**     |
| W4-3 | FIRE-LABEL-R1 | L3 Impl + L2 Test (= 1 sub にまとめる) | codex/fire-label-r1-refresh    | しない         |
| W4-4 | F286-DATA-R3 | L1 Design + L3 Impl 設計のみ | codex/f286-data-r3-d2-design       | **しない**     |

### W4-1: F286-PNL-R2-runner (F062 --record-decisions 統合)

詳細: [[F286_PNL_R2_runner_record_decisions|F286-PNL-R2-runner 起票 doc]]

- 目的: F062 send_smoke runner に --record-decisions option を追加し、
  LINE 送信成功直後に SnapshotStore.save_snapshot を呼ぶ
- HQ 承認 #3 (= LINE 成功 / snapshot 失敗 → non-zero + 後追い helper)
- HQ 承認 #4 (= LINE 失敗時 → snapshot 残さない)
- HQ 承認 #7 (= --record-decisions は production-only)
- **dry-run / safety path 中心**、staging write は **しない**

allowed_files:
- scripts/jobs/run_f062_line_production_send_smoke.py (= 既存修正)
- scripts/jobs/run_f062_research_advisory_line_preview.py (= payload に
  send_id 埋め込み、必要なら)
- tests/scripts/jobs/test_run_f062_line_production_send_smoke.py (= 既存修正)

forbidden_files: 全 pnl/* (= R2 module 既に commit 済、touched なし) /
全 既存禁止リスト

### W4-2: F286-PNL-R2-ingest-helper (後追い ingest)

詳細: [[F286_PNL_R2_ingest_helper|F286-PNL-R2-ingest-helper 起票 doc]]

- 目的: LINE 送信成功 / snapshot 保存失敗時の **後追い ingest** helper
- runner 統合 (W4-1) とは **分離** (= 独立タスク)
- 入力: F062 payload JSON path
- 動作: payload_to_snapshot → SnapshotStore.save_snapshot を staging に
  対して実行 (= LINE 送信なし)
- **dry-run / safety path 中心**、staging write は **しない**

allowed_files:
- scripts/jobs/run_f286_pnl_r2_ingest_snapshot.py (= 新規)
- tests/scripts/jobs/test_run_f286_pnl_r2_ingest_snapshot.py (= 新規)

forbidden_files: 全 pnl/* (= 既に commit 済) / 全既存禁止

### W4-3: FIRE-LABEL-R1 (新ラベル方針対応)

詳細: [[FIRE_LABEL_R1_advisory_label_refresh|FIRE-LABEL-R1 起票 doc]]

- 5 新ラベル: 🟢 積極的買い推奨 / 🟡 条件付き買い推奨 / 🟠 場中監視 /
  ⚠️ 注意つき買い候補 / 🔴 見送り推奨
- 影響範囲: notifications/templates/research_advisory.py +
  pnl/snapshot.py の `_ACTION_VERDICT` + 関連 tests
- mode 切替判断: 即時切替 (案 Y) 推奨
- 既存 advisory_decisions row は保持 (= migration なし)

allowed_files:
- notifications/templates/research_advisory.py
- pnl/snapshot.py (= `_ACTION_VERDICT` のみ修正)
- tests/notifications/templates/test_research_advisory_line_template.py
- tests/scripts/jobs/test_run_f062_research_advisory_line_preview.py
- tests/pnl/test_snapshot.py

forbidden_files: F062 runner (= W4-1 担当) / F062 send_smoke /
pnl/storage.py / pnl/schema.py / pnl/models.py / pnl/ingestion.py /
全既存禁止

### W4-4: F286-DATA-R3-D2 (実 fetch / 実 write 設計 + dry-run plan)

詳細: [[F286_DATA_R3_D2_real_fetch_write_plan|F286-DATA-R3-D2 起票 doc]]

- 目的: F286-DATA-R3 skeleton に **実 fetch / 実 write** 統合の設計案 +
  dry-run plan を作成
- 実装本体 (sub-D2 Impl) は HQ approve 後に Wave 5+ で実施
- 本 Wave 4 では **設計 doc のみ**
- cron 本番登録は **絶対に行わない** (= sub-D3 / 凍結継続)

allowed_files: 設計 doc 出力先のみ (= vault または /tmp/)

forbidden_files: scripts/jobs/run_f286_data_r3_daily_refresh.py (= 既存
skeleton、touched なし) / 全既存禁止

## 並列度 (= 4 sub 並行投入)

| Wave 4 phase | 並列 lane 数 | 想定時間 |
|---|---|---|
| 並列実行 (W4-1 + W4-2 + W4-3 + W4-4)        | 4 lane 同時 | 約 20-30 分 |
| 本線 Integrator review (= 全 4 成果物)       | 1 (本線)    | 約 15-20 分 |
| Wave 4 完了 → HQ 報告                       | 1 (本線)    | 約 5-10 分 |

合計: 約 40-60 分。

W4-1 と W4-2 は F062 / pnl/ の異なる経路、W4-3 は template + tests、
W4-4 は設計のみ → 衝突なし。

## Wave 4 完了後の HQ 判断項目

1. W4-1 (= F062 --record-decisions 統合) を develop へ commit
2. W4-2 (= ingest helper) を develop へ commit
3. W4-3 (= FIRE-LABEL-R1) を develop へ commit
4. W4-4 設計 doc を vault へ commit (= 03_design/F286_DATA_R3_D2_*.md)
5. **Wave 4.1 staging smoke** 着手判断
   - 初回 DDL 込みの ensure_schema 明示承認
   - F062 経由 + 後追い helper 経由の両方を smoke するか
   - 推奨実施日: 5/12〜5/17 内 (= F282 weekly snapshot 5/18 前)

## staging smoke 完全分離 (= HQ 明示再確認)

- ★ Wave 4 内では staging DB write は **しない**
- Wave 4 完了後、staging smoke は **Wave 4.1 として分離**
- 実施前に **「初回 DDL 込み staging write smoke」の HQ 明示承認** 要
- 現時点で staging smoke は **未承認**

## sub-D3 cron 本番登録 引き続き凍結

Wave 3 sub-4E CRITICAL 2 件への HQ 暫定方針:
- F282 月曜順序矛盾: **案 A 暫定採用** (= 月曜 daily refresh 07:30 に
  後ろ倒し、F282 weekly snapshot 見直しは将来 FIRE-OPS-R0 候補)
- launchd 日付付きログ: **固定ログ + rotation 設計優先** (= launchd
  StandardOutPath に日付変数を期待しない)

両方とも sub-D3 (= 将来の cron 本番登録) で適用、Wave 4 では cron 本番
登録を実施しない。

## 共通安全要件 (= Wave 4 全 sub に適用)

- LINE 本番送信: 禁止
- production / develop DB write: 禁止
- **未承認の staging DB write: 禁止** (= W4-1 / W4-2 / W4-3 / W4-4 全件)
- token / channel_token / secret 参照: 禁止
- 楽天証券操作 / 自動発注 / Computer Use / Playwright: 禁止
- GitHub Actions workflow 変更: 禁止
- --no-verify: 禁止
- scripts/seed_pattern_layer1.py: 未接触必須
- simulation/research_lane/historical_indicators.py: 未接触必須
- TODO Excel 更新: 禁止
- cron / launchd / crontab 本番登録: 禁止 (= 凍結継続)

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1 設計]]
- [[FIRE_CODEX_R1_WAVE1_results|Wave 1 結果]]
- [[F286_PNL_R2_advisory_snapshot_auto_ingest|Wave 2 結果]]
- [[FIRE_CODEX_R1_WAVE3_audit_results|Wave 3 結果]]
- [[F286_PNL_R2_runner_record_decisions|W4-1 F286-PNL-R2-runner 起票]]
- [[F286_PNL_R2_ingest_helper|W4-2 F286-PNL-R2-ingest-helper 起票]]
- [[FIRE_LABEL_R1_advisory_label_refresh|W4-3 FIRE-LABEL-R1 起票]]
- [[F286_DATA_R3_D2_real_fetch_write_plan|W4-4 F286-DATA-R3-D2 起票]]
- [[../log]]
