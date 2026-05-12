---
id: FIRE-CODEX-R1-WAVE14-results
phase: ガバナンス / Wave 14 完了 (= SCHEMA-R1 dry-run + develop apply + production apply) / R-01-08
priority: 最優先
status: 完了 ★ 3 sub-task / 3 環境全 schema 同期完了 (2026-05-12、production DB write 1 + develop DB write 1)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 13 (= 完了)
  - HQ W14-1 / W14-2 / W14-3 段階承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / Codex 運用拡張 / schema migration 実適用
---

# FIRE-CODEX-R1 v1.1 Wave 14: SCHEMA-R1 dry-run + develop apply + production apply

最終更新: 2026-05-12

## ★ 状態: 完了 (= 3 環境全 schema 同期、SCHEMA-R1 ライフサイクル完結)

W12-2 で発見した「production / develop に advisory_decisions table 不在」
問題を Wave 14 で **全 環境に完全 schema を構築** して解消。

production / develop / staging の advisory_decisions が 22 列 + paper_reason
+ PK + 2 indexes + CHECK 完全一致。F286-PNL-R1/R2/R3 + W7-4 REPORT-R1 の
前提が全 環境で成立。

## Wave 14 sub-task 結果 (= 3 件)

| sub | task | 結果 |
|-----|------|------|
| W14-1 | SCHEMA-R1 dry-run 3 環境 | ✓ 期待値完全一致 / DB write 0 |
| W14-2 | SCHEMA-R1 develop apply | ✓ 11 step、HQ 12 条件全 ✓ / CREATE 1 回成功 |
| W14-3 | SCHEMA-R1 production apply | ✓ 13 step、HQ 14 条件全 ✓ / backup 永続化 |

## W14-1 dry-run 結果 (= HQ 期待通り)

| 環境 | action | 結果 |
|------|--------|------|
| production | dry_run_would_create | ✓ |
| develop | dry_run_would_create | ✓ |
| staging | skip_already_exists | ✓ |

全 DB mtime unchanged、CREATE 0 回。

## W14-2 develop apply 結果

- action: created ★
- 22 列 + paper_reason 完全作成
- PK (advisory_id, code) + 2 indexes + CHECK constraints 全 ✓
- row count 0
- production / staging mtime unchanged
- idempotent 確認

## W14-3 production apply 結果

- production DB backup 事前作成
- sha256: 7068b959995eb612817abfdf4ae70952fb84544d170696dba7ac2cf44a4418e9
- action: created ★
- 22 列 + paper_reason 完全作成
- PK / 2 indexes / CHECK 全 ✓
- row count 0
- develop / staging mtime unchanged
- idempotent 確認
- backup 永続化 完了 (= ~/fire-backups/)

## 3 環境 schema 同期完了

production / develop / staging の advisory_decisions が 22 列 + paper_reason
+ PK + 2 indexes + CHECK + DEFAULT + NOT NULL で論理一致。

F286-PNL-R1/R2/R3 + W7-4 REPORT-R1 の前提が全 環境で成立。

## backup 永続化 (= HQ 指示)

[[../03_design/F286_PNL_SCHEMA_R1_production_backup_handling_2026-05-12|
backup 取扱 plan]]

- 移動先: ~/fire-backups/
- sha256 整合性 永続化後一致
- 保存期間最低 2 週間または REPORT-R1/PNL dry-run 確認完了まで
- 削除は HQ 明示承認後

## W11-1 paper_reason 単体 ALTER plan の取扱

HQ 指示: **superseded / completed mark、削除しない**。
3 plan vault doc に status 追記済。

## ガバナンス成果

「段階的 production 適用」枠組み完結:
- dry-run → develop → production の 3 段階
- 各 step で HQ 明示承認
- backup 必須 + sha256 整合性
- production DB write 初実行を安全に成功

FIRE プロジェクト史上初の production DB schema write を、26 必須確認 step
で完全制御。

## HQ 判断が必要な論点 (= 5 件、次フェーズ候補)

1. REPORT-R1 production/develop read-only dry-run 確認
2. REPORT-R1 LINE delivery preview / send guard 確認
3. DATA-R3 sub-D2.3.x runner 別 staging write
4. PNL-R2/F062 record-decisions 本番連携再評価
5. sub-D3 cron 凍結解除設計

各候補は HQ 別 approve 必要。

## 関連リンク

- [[FIRE_CODEX_R1_WAVE13_results|Wave 13 results]]
- [[../03_design/F286_PNL_SCHEMA_R1_advisory_decisions_full_migration_2026-05-12|SCHEMA-R1 起票]]
- [[../03_design/F286_PNL_SCHEMA_R1_production_backup_handling_2026-05-12|backup 取扱 plan]]
- [[../log]]
