# FIRE staging DB restore from backup_recent (2026-05-17)

doc_id: FIRE-STAGING-DB-RESTORE-BACKUP-RECENT-2026-05-17
status: 復元成功 / 全受容判定 PASS / production-develop 不変 / git push 0
HQ marker: HQ_APPROVE_STAGING_DB_RESTORE_FROM_BACKUP_RECENT
related:
- [[STAGING_MARKET_FINANCIALS_V2_RECOVERY_AUDIT_2026-05-17]] — 本 wave の audit 起点 / Option A atomic flow 採用
- [[DATA_R1_FINANCIALS_REFRESH_ENABLEMENT_2026-05-17]] — 次 wave の enablement
- [[DATA_R1_STAGING_REFRESH_PRICES_INDEX_2026-05-16]] — backup の元となった DATA-R1 wave

---

## §1 目的

現 staging.db (= 5/17 02:00 で develop に上書きされた状態) を、DATA-R1 完了直後の backup_recent (= 5/17 00:51、md5 71a63a19...) へ atomic flow で復元。
これにより market_financials_v2 / research_derived_indicators / research_watchlist_signals / market_prices_daily latest 2026-05-15 を一括復旧する。

★ 本 wave スコープ: staging.db restore のみ。financials refresh / derived regen / signal regen / W2-B rerun は別 wave。

---

## §2 承認範囲

| 項目 | 内容 |
|---|---|
| HQ marker | HQ_APPROVE_STAGING_DB_RESTORE_FROM_BACKUP_RECENT |
| restore source | ~/fire-backups/fire.staging.db.bak.20260517_020005 |
| restore target | data/fire.staging.db |
| atomic flow | tmp copy → quick_check + md5 → atomic mv (= POSIX rename(2)) |
| 退避先 | data/fire.staging.db.before_restore_20260517_134908 |
| 禁止 | production/develop write、financials refresh、API、token-env、LINE、launchctl、plist、cron、workflow、git add/commit/push、PR、merge、--no-verify、sudo、rm -rf、auto-order、楽天、iSPEED、Computer Use |

---

## §3 git 状態 (pre)

| 項目 | 値 |
|---|---|
| branch | develop |
| HEAD | 2918cb4 |
| working tree | clean |
| ahead/behind origin/develop | 0/0 |
| fire-vault | main clean / 0/0 |

---

## §4 Precheck (= 全 PASS)

| 項目 | 結果 |
|---|---|
| current staging md5 | 085799da... (= develop と同一、復旧対象 confirmed) |
| production md5 (pre) | b1df4673... |
| develop md5 (pre) | 085799da... |
| F282 plist 3 file (pre) | 全 md5 記録済 |
| WAL/SHM 不在 | ✓ |
| lsof data/fire.staging.db | 空 (= 他 process 不在) |
| restore source 存在 | ✓ 4.8 GB / May 17 00:51 |
| restore source quick_check | ok |
| restore source 6 tables 期待値一致 | ✓ |
| restore source md5 | 71a63a19694385db4344246be60a2f91 (= DATA-R1 staging post と一致) |

---

## §5 atomic restore 実行 (5 step)

| step | action | 結果 |
|---|---|---|
| 1 | cp source → data/fire.staging.db.restore_tmp_20260517_134908 | OK / 0.9 sec / 4.8 GB |
| 2 | quick_check on tmp | ok |
| 3 | md5 tmp (71a63a19...) == src md5 (71a63a19...) | MATCH ✓ |
| 4 | mv current → data/fire.staging.db.before_restore_20260517_134908 | OK / 353 MB 退避 |
| 5 | mv tmp → data/fire.staging.db (= POSIX rename(2)、同一 FS 内 atomic) | OK / 4.8 GB |

→ 全 step 成功、中断 0、半端 file 0。

---

## §6 post-restore 受容判定 (= 全 PASS)

詳細: `/tmp/fire_staging_db_restore_from_backup/post_restore_acceptance.md`

| table | 期待 rows | 実 rows | 期待 latest | 実 latest | OK |
|---|---|---|---|---|---|
| market_financials_v2 | 164,678 | 164,678 | 2026-05-08 | 2026-05-08 | ✓ |
| market_prices_daily | 2,089,775 | 2,089,775 | 2026-05-15 | 2026-05-15 | ✓ |
| research_derived_indicators | 3,750 | 3,750 | 2026-05-08 | 2026-05-08 | ✓ |
| research_watchlist_signals | 13,839 | 13,839 | 2026-07-22 | 2026-07-22 | ✓ |
| index_data | 60 | 60 | 2026-05-11 | 2026-05-11 | ✓ |
| market_listings | 4,449 | 4,449 | 2026-05-01 | 2026-05-01 | ✓ |
| market_financials (v1) | (bonus) | 1,723 | - | 2026-05-08 | ✓ |

total_tables: 31 → **37** (復旧)
PRAGMA quick_check: ok

---

## §7 DB md5 (= production/develop 不変、staging 変化想定)

| DB | pre md5 | post md5 | 結果 |
|---|---|---|---|
| data/fire.db (production) | b1df4673... | b1df4673... | **不変 ✓** |
| data/fire.develop.db | 085799da... | 085799da... | **不変 ✓** |
| data/fire.staging.db | 085799da... | 71a63a19... | 変化 (= 復元成功の証拠) |

---

## §8 F282 plist 全 3 file 不変

| file | size | mtime | md5 | 不変 |
|---|---|---|---|---|
| ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist | 1,772 | 5月 12 22:46 | e4ea05e8... | ✓ |
| docs/launchd/jp.fire.weekly-snapshot.plist | 1,772 | 5月 16 21:29 | e4ea05e8... | ✓ |
| docs/launchd/jp.fire.weekly-snapshot-smoke.plist | 1,844 | 5月 16 21:29 | 0667c02e... | ✓ |

→ launchctl / plist 編集 0 ✓

---

## §9 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| staging DB write (intentional restore) | ✓ 1 回のみ (= atomic mv) |
| production DB write | 0 (md5 不変) |
| develop DB write | 0 (md5 不変) |
| schema migration | 0 |
| financials refresh / API call | 0 |
| token / .env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push / PR | 0 |
| sudo / rm -rf / --no-verify | 0 |
| auto-order / 楽天 / iSPEED / Computer Use | 0 |
| TODO Excel 更新 | 0 |

---

## §10 rollback 方針 (= Lane D HIGH 反映、sidecar 対応版)

問題検出時の手順 (= 本 wave スコープ外、明示指示要):
```bash
# 1. staging 利用 process がないこと確認
lsof data/fire.staging.db*

# 2. 現在の (= 復旧後) staging.db を退避
TS_RB=$(date +%Y%m%d_%H%M%S)
mv data/fire.staging.db data/fire.staging.db.rollback_keep_${TS_RB}

# 3. WAL/SHM sidecar も退避 (= 不整合防止)
[ -f data/fire.staging.db-wal ] && \
  mv data/fire.staging.db-wal data/fire.staging.db-wal.rollback_keep_${TS_RB}
[ -f data/fire.staging.db-shm ] && \
  mv data/fire.staging.db-shm data/fire.staging.db-shm.rollback_keep_${TS_RB}

# 4. retained artifact を本体位置に rename
mv data/fire.staging.db.before_restore_20260517_134908 data/fire.staging.db

# 5. quick_check 確認
sqlite3 "file:data/fire.staging.db?mode=ro" "PRAGMA quick_check;"
md5 -q data/fire.staging.db
# (= expected: 085799daf117e59b05d4b9d9aeb7662d、353,128,448 bytes)
```

- retained artifact `data/fire.staging.db.before_restore_20260517_134908` (353 MB)
  は次 wave (= financials refresh) で正常動作確認まで削除しない
- 受容判定 NG が出れば本 wave 内追加操作せず停止、HQ 報告
- ★ Lane D HIGH 反映: rollback 時に WAL/SHM sidecar も退避する (= 旧 sidecar が
  新 (= 戻した) DB と不整合状態になることを防止)

---

## §11 Codex 4 lane 監査結果

| lane | verdict | CRITICAL | HIGH | 摘要 |
|---|---|---|---|---|
| A (restore source / current / precheck) | **APPROVE** | 0 | 0 | 独立 md5 + 6 tables + quick_check 完全一致確認、precheck PASS 妥当 |
| B (atomic restore flow / md5 / quick_check) | **APPROVE** | 0 | 0 | atomic flow 順序妥当、同一 device 16777233 で rename atomic 確認、tmp 不在、md5 完全一致 |
| C (post-restore acceptance / table freshness) | **APPROVE** | 0 | 0 | quick_check ok、6 tables 一致、PK 検証 OK、PK duplicate/null 0、財務 2026-05-08 が refresh baseline として妥当 |
| D (safety / rollback / next-wave) | CONCERN | 0 | 0 | WAL/SHM が Codex 検査中に出現 (= read-only query 由来)、rollback 手順を sidecar 対応版に補強推奨 |

### §11.1 Lane A APPROVE
- current staging md5 71a63a19... + size 4,804,788,224 + 6 tables 全件 source/precheck と一致
- pre 退避 backup (= 085799da... / 353 MB) は pre staging と一致
- restore source quick_check ok + 6 tables 一致
- git HEAD/branch/ahead-behind 一致、working tree 未追跡は expected before_restore のみ
- precheck PASS 判定妥当

### §11.2 Lane B APPROVE
- atomic flow 順序 (cp → tmp quick_check → md5 → mv current → mv tmp) 妥当
- `data/` と source は同一 device 16777233、step 5 rename(2) は同一 FS 内 atomic
- tmp 不在、before_restore は存在、rollback artifact として有効
- 実測 md5 staging/source 完全一致 (= 71a63a19...)
- target 直接上書き経路の不在を log 上で確認
- rollback の論理経路 (= 現 target 退避 → before_restore rename) 機能確認

### §11.3 Lane C APPROVE
- data/fire.staging.db md5 71a63a19... 独立検証
- Python sqlite3 URI mode=ro で quick_check ok
- 6 tables rows/latest 全件主張一致
- market_financials_v2 PK = (code, disclosure_date, type_of_document)、key nulls/duplicates 0
- backfill_market_financials._verify_v2_schema_or_raise を通る schema 確認
- 37 tables 中 sqlite_sequence 含む = user tables 36、異常なし
- 財務 2026-05-08 が 9 日 stale で refresh baseline として妥当

### §11.4 Lane D CONCERN (HIGH 0)
**指摘**: Codex 検査中に `data/fire.staging.db-wal` 0B / `data/fire.staging.db-shm` 32KB が出現。
- DB 本体 md5 は不変 (= 71a63a19... 維持)、journal_mode=delete
- → Codex の read-only sqlite3 query (= 接続時) が一時 sidecar を作成しただけ
- DB 整合性に影響なし、journal_mode=delete のため次の clean shutdown で sidecar は除去される
- **本 wave への影響なし** (= データ破壊 / DB 不整合 0)
- ただし「WAL/SHM 不在」を主張する場合は明示記録が必要 → 本 doc §5 を更新

その他 Lane D APPROVE 項目:
- production/develop md5 不変 (= b1df4673... / 085799da...) 確認
- F282 plist 3 file md5/size/mtime 記録一致
- enablement commit 2918cb4 確認
- 次 wave (financials refresh) 前提充足
- 推奨: rollback 手順書を sidecar 対応版に補強、financials refresh の前に WAL/SHM 状態を再確認

---

## §12 financials refresh 可否

| 条件 | 状態 |
|---|---|
| market_financials_v2 schema 存在 + PK 3-key | ✓ (164,678 rows、PK 検証済) |
| current latest disclosure_date | 2026-05-08 (= 9 日 stale、refresh で 5-17 等へ) |
| backfill_market_financials._verify_v2_schema_or_raise が通る | ✓ (PK 一致) |
| 4 段 staging guard 適用条件 | ✓ (basename = fire.staging.db) |

→ **HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS を切れる状態 ✓**

---

## §13 結論 / 次アクション

### §13.1 結論
- atomic restore 完了 (= 全 5 step 成功)
- 全 6 主要 table 期待値一致
- production / develop DB 完全不変
- F282 plist 全 3 file 不変
- launchctl / cron / git push / LINE / API 0
- 復旧 retained artifact 退避済 (= rollback 容易)

### §13.2 次 wave 推奨

★ Lane D 推奨 RECOMMENDED_NEXT: other (= sidecar 整理を先行):

1. (推奨) WAL/SHM sidecar 整理 (= 軽量 wave、別 marker または financials refresh の precheck に組み込み)
2. HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS
   - 開始前: lsof data/fire.staging.db* / WAL/SHM 状態再確認
   - FIRE_ENV=staging .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh
     --db-path data/fire.staging.db --db-label staging
     --datasets financials --financials-smoke-type five_codes --max-days 3 --write
3. HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH (= 案 A.5、復旧後最新 derived 追加)
4. HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH (= 案 B)
5. HQ_APPROVE_W2B_RE_RUN_FRESH_DATA (= 案 C)
6. (任意) 復旧確認後に retained artifact 削除 wave
   (Lane D 推奨: financials refresh 完了後、quick_check + md5 記録 + v2 latest 更新 +
   smoke report PASS + rollback 不要 marker 作成後)

---

## §14 関連 file

```
本 wave 出力:
  /tmp/fire_staging_db_restore_from_backup/recovery_precheck.json
  /tmp/fire_staging_db_restore_from_backup/restore_result.json
  /tmp/fire_staging_db_restore_from_backup/post_restore_acceptance.md
  /tmp/fire_staging_db_restore_from_backup/restore_ts.txt
  /tmp/fire_staging_db_restore_from_backup/codex_lane_{A,B,C,D}_prompt.txt + result.txt

設計 doc:
  ~/fire-vault/03_design/STAGING_DB_RESTORE_FROM_BACKUP_RECENT_2026-05-17.md (本 doc)

retained artifact (= rollback 用):
  data/fire.staging.db.before_restore_20260517_134908 (353 MB)

restored DB:
  data/fire.staging.db (= 4,804,788,224 bytes、md5 71a63a19...)
```
