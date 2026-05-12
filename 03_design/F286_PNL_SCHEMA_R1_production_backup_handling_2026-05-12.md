---
id: F286-PNL-SCHEMA-R1-production-backup-handling
phase: Wave 14 完了後 / production backup 永続管理
priority: 高
status: 起票 ☆ 永続化済 (= 2026-05-12)、保存期間中
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W14-3 SCHEMA-R1 production apply 完了 (= 2026-05-12)
  - HQ Wave 14 backup 永続化指示
chapter: F286 PNL / R-01-08 / backup 管理
---

# F286-PNL-SCHEMA-R1 Production Backup Handling Plan

最終更新: 2026-05-12

## ★ 状態: 永続化済 (= /tmp/ から ~/fire-backups/ へ移動完了)

W14-3 SCHEMA-R1 production apply の前に取得した production DB backup を
HQ 指示に従い永続 backup 領域へ移動、保存期間 + 削除手順を明示。

## backup 情報

| 項目 | 値 |
|------|-----|
| 元 backup path | /tmp/w14_3/fire.db.pre_schema_r1_20260512_161611 |
| 永続 backup path | ~/fire-backups/fire.db.pre_schema_r1_20260512_161611 |
| size | 371,064,832 bytes (= 約 354 MB) |
| sha256 | 7068b959995eb612817abfdf4ae70952fb84544d170696dba7ac2cf44a4418e9 |
| 取得時刻 | 2026-05-12 16:16:11 JST |
| 取得契機 | W14-3 SCHEMA-R1 production apply 直前 (= W13-3 plan 必須 step 1) |
| 永続化時刻 | 2026-05-12 16:20 JST |
| sha256 保存 path | ~/fire-backups/fire.db.pre_schema_r1_20260512_161611.sha256 |

## 永続化の手順 (= 2026-05-12 実施)

```bash
mkdir -p ~/fire-backups
mv /tmp/w14_3/fire.db.pre_schema_r1_20260512_161611 \
   ~/fire-backups/
sha256sum ~/fire-backups/fire.db.pre_schema_r1_20260512_161611 \
   > ~/fire-backups/fire.db.pre_schema_r1_20260512_161611.sha256
```

移動後 sha256 確認: 元 sha256 (= 7068b9...) と完全一致 ✓ (= 整合性維持)。

## 保存期間 (= HQ 指示)

最低期間:
- **2 週間** (= 2026-05-26 まで保持)
- もしくは **REPORT-R1 / PNL 系 production dry-run 確認完了まで** (= 長い方)

削除条件:
- 上記保存期間経過
- **+ HQ 明示承認**
- 削除前に integrity 再確認 (= sha256)

## backup 復元手順 (= 万一)

```bash
# 1. application 全停止 (= F012 Runner / cron / OpenClaw 等)
# 2. 現 production DB を退避 (= 上書き前)
cp ~/fire/data/fire.db /tmp/fire.db.pre_restore_$(date +%Y%m%d_%H%M%S)
# 3. backup から復元
cp ~/fire-backups/fire.db.pre_schema_r1_20260512_161611 ~/fire/data/fire.db
# 4. integrity 確認
sha256sum ~/fire/data/fire.db ~/fire-backups/fire.db.pre_schema_r1_20260512_161611
# 期待: 完全一致
# 5. schema 確認
sqlite3 ~/fire/data/fire.db \
    "SELECT name FROM sqlite_master WHERE type='table' AND name='advisory_decisions';"
# 期待: 出力なし (= advisory_decisions table 復元前状態に戻る、不在)
# 6. application 再開
```

## 監視 / 健康チェック

定期 (= 週次推奨):
```bash
# 1. backup 存在確認
ls -la ~/fire-backups/fire.db.pre_schema_r1_20260512_161611

# 2. sha256 整合性確認
cd ~/fire-backups && sha256sum -c fire.db.pre_schema_r1_20260512_161611.sha256
# 期待: OK
```

## 削除時の HQ 承認 template

```
=== HQ 承認依頼 / W14-3 backup 削除 ===

date:                (実行予定日)
backup target:        ~/fire-backups/fire.db.pre_schema_r1_20260512_161611
size:                 354 MB
取得日:              2026-05-12
保存期間経過:        ✓ (= YYYY-MM-DD 経過、最低 2 週間 + REPORT-R1 PNL
                       系 production dry-run 確認完了)
delete reason:       backup 役目完了、永続保存不要
削除前 sha256 確認:  7068b9...4418e9 (= 元と一致 ✓)

承認希望: backup ファイル削除
```

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE14_results|Wave 14 results]]
- [[F286_PNL_SCHEMA_R1_production_apply_plan_2026-05-12|W13-3 production apply plan]]
- [[F286_PNL_SCHEMA_R1_advisory_decisions_full_migration_2026-05-12|SCHEMA-R1 起票]]
- W14-3 commit (fire develop、SCHEMA-R1 完成)
