# FIRE F282-POST-SNAPSHOT-STAGING-AUDIT-RUNNER-R1 (2026-05-17)

doc_id: FIRE-F282-POST-SNAPSHOT-AUDIT-RUNNER-R1-2026-05-17
status: 実装完了 / 28 PASS / pre-snapshot smoke = PROCEED_FINANCIALS_RETRY
HQ marker: (= 純実装 + read-only smoke、specific marker 不要)
related:
- [[JQUANTS_ENV_SUPPLY_PREFLIGHT_R1_2026-05-17]] — 前 wave、wrapper 方式 verified
- [[F111_DATA_FRESHNESS_GATE_R1_2026-05-17]] — 前 wave、freshness gate
- [[STAGING_DB_RESTORE_FROM_BACKUP_RECENT_2026-05-17]] — 現 staging 起点

---

## §1 目的

5/18 02:00 F282 weekly snapshot 後に staging DB の状態を read-only で
判定し、以下のいずれに該当するかを機械判定する:
- staging が backup_recent (= 71a63a19) 状態維持 → financials retry 可
- staging が production 同期で上書きされた → restore 必要
- 状態不明 / 異常 → 調査必要
- F282 未実行 → 待機

これにより、5/18 朝の判断を **手動 audit ではなく runner 1 コマンド**で
完了させる。

---

## §2 runner 設計

### §2.1 CLI
```
.venv/bin/python -m scripts.jobs.run_f282_post_snapshot_staging_audit
  --prod-db data/fire.db
  --develop-db data/fire.develop.db
  --staging-db data/fire.staging.db
  --restore-backup ~/fire-backups/fire.staging.db.bak.20260517_post_restore_71a63a19
  --reference-date 2026-05-18
  --f282-log logs/cron/weekly-snapshot.log
  --f282-err logs/cron/weekly-snapshot.err
  --output-json /tmp/.../audit.json
  --output-md /tmp/.../audit.md
  [--strict]
```

### §2.2 DB 接続方針
- URI `mode=ro` + `PRAGMA query_only=ON`
- **`immutable=1` 不使用** (= WAL stale read 回避、Codex 既指摘)
- 全 4 DB (prod/develop/staging/backup) の md5 + size を取得

### §2.3 read-only audit 項目
- 3 DB + backup の md5 / size
- staging quick_check
- WAL/SHM state (= wal_exists + wal_size)
- lsof (= optional、不在で graceful degrade)
- table freshness (= 6 table)
- F282 log/err mtime + 直近 fire 検出 + error keyword grep

---

## §3 decision (= 4 値)

| decision | 条件 | next_action |
|---|---|---|
| **PROCEED_FINANCIALS_RETRY** | staging md5 == backup md5 + mf_v2/derived 存在 + err patterns なし + WAL zero | HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY |
| **RESTORE_FROM_BACKUP_REQUIRED** | staging md5 == production md5 (= F282 で上書きされた) | atomic restore from backup_recent |
| **INVESTIGATE** | staging missing / quick_check NG / backup missing / md5 不一致 / err patterns 検出 / WAL non-zero | stop and investigate |
| **WAIT_FOR_SNAPSHOT** | F282 未実行 (= log_mtime None) | wait for next F282 fire |

### §3.1 decision logic 詳細
1. **WAL non-zero gate** (CRITICAL safety): staging.db-wal が non-zero
   → md5 比較が不完全 → 自動 INVESTIGATE
2. staging md5 == backup → table 存在 + err patterns チェック → PROCEED
3. staging md5 == production → RESTORE
4. staging md5 == develop + mf_v2 あり → PROCEED、無し → RESTORE
5. md5 全不一致 + log なし → WAIT
6. それ以外 → INVESTIGATE

---

## §4 pre-snapshot smoke (= 2026-05-17 18:50 JST)

実行コマンド:
```
.venv/bin/python -m scripts.jobs.run_f282_post_snapshot_staging_audit
  --prod-db data/fire.db
  --develop-db data/fire.develop.db
  --staging-db data/fire.staging.db
  --restore-backup ~/fire-backups/fire.staging.db.bak.20260517_post_restore_71a63a19
  --reference-date 2026-05-17
  --output-json /tmp/.../pre_snapshot_audit.json
  --output-md /tmp/.../pre_snapshot_audit.md
```

結果:
- **decision: PROCEED_FINANCIALS_RETRY**
- reason: staging md5 == restore-backup md5 (= 71a63a19 状態維持)
- staging quick_check: ok
- market_financials_v2: 164,678 rows
- market_prices_daily: 2,089,775 / latest 2026-05-15
- WAL: 0 bytes (= safe、journal_mode=delete)
- F282 err: error_patterns 検出なし
- next_action: HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY

DB md5:
| db | md5 |
| prod | b1df4673... |
| develop | 085799da... |
| staging | 71a63a19... ★ |
| backup_recent | 71a63a19... ★ |

→ pre-snapshot 時点で safe、5/18 朝 retry へ進める判定確定。

---

## §5 Codex 8 lane + commit 前 CRITICAL 結果

| lane | verdict | HIGH | 摘要 |
|---|---|---|---|
| A (decision logic) | CONCERN | 1 | error_patterns ignored → 採用、PROCEED 前 gate 追加 |
| B (DB md5 / freshness) | CONCERN | 0 (MEDIUM) | signals future synthetic未gated → 別 wave |
| C (F282 log audit) | CONCERN | 1 | A と同根 |
| D (Freshness Gate / WAL) | CONCERN | 1 | md5 WAL fingerprint 不足 → commit 前 CRITICAL に格上げ、採用 |
| E (tests / strict) | CONCERN | 1 | startswith → is_relative_to 採用 |
| F (no-write safety) | CONCERN | 0 | D + E と同根 |
| G (post-2am readiness) | CONCERN | 1 | WAIT 日曜固定 → log_mtime ベースに拡張 |
| H (next-wave) | APPROVE | 0 | 整合確認 |

採用 + 修正 (commit 前):
1. Lane A/C HIGH: PROCEED 前に F282 err error_patterns 確認、検出時
   INVESTIGATE 振替
2. Lane G HIGH: WAIT_FOR_SNAPSHOT を log_mtime 不在で発火
3. Lane E HIGH: _validate_output_path を is_relative_to に変更
4. **Codex pre-commit CRITICAL (= Lane D 格上げ)**: staging.db-wal が
   non-zero なら md5 比較不完全 → 自動 INVESTIGATE 振替 (= safety
   override)、regression test 2 件追加

deferred (= 別 wave):
- Lane B MEDIUM: signals future synthetic decision gating
- F062/Ops Summary adapter (= 元 F111 Lane F+H schema mismatch)

---

## §6 tests (= 28 PASS)

tests/scripts/jobs/test_run_f282_post_snapshot_staging_audit.py:

class 別:
- TestDecideLogic (11): 純粋 decision logic、error_patterns regression、
  WAIT non-Sunday regression、WAL non-zero regression、WAL zero regression
- TestF282LogState (3): log/err mtime + error keyword grep
- TestWalShmState (2): WAL/SHM present/absent
- TestRunnerIntegration (8): PROCEED / RESTORE / symlink refused /
  outside allowed / prefix sibling / JSON+MD output / strict mode
- TestReadonlyEnforcement (1): write attempt raise
- TestSourceSafety (3): forbidden imports / PRAGMA query_only /
  immutable 不使用

regression 0、実行時間 0.66 sec

---

## §7 5/18 02:00 後の使い方

5/18 02:00 F282 自動実行後、operator が以下を 1 コマンドで判定:

```
.venv/bin/python -m scripts.jobs.run_f282_post_snapshot_staging_audit
  --prod-db data/fire.db
  --develop-db data/fire.develop.db
  --staging-db data/fire.staging.db
  --restore-backup ~/fire-backups/fire.staging.db.bak.20260517_post_restore_71a63a19
  --reference-date 2026-05-18
  --f282-log logs/cron/weekly-snapshot.log
  --f282-err logs/cron/weekly-snapshot.err
  --output-json /tmp/.../post_snapshot_audit.json
  --output-md /tmp/.../post_snapshot_audit.md
```

期待される 3 パターン:

### パターン A: PROCEED
- staging が 71a63a19 状態維持 (= F282 が staging を skip した、または
  WAL non-zero でも本体 md5 一致)
- → HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY 即実行可
- 受容判定後: derived regen → signal regen → W2-B re-run

### パターン B: RESTORE
- staging が production 同期で上書きされた (= md5 一致)
- → 別 wave で atomic restore from backup_recent
- 受容判定後 → financials retry へ

### パターン C: INVESTIGATE
- 想定外状態 (= md5 不一致、quick_check NG、err patterns、WAL non-zero)
- → 即停止、log 確認、HQ 判断、必要なら手動 audit

strict mode (--strict) で non-PROCEED → exit 4、launchd/cron 連携時の
失敗判定に使用可能。

---

## §8 financials retry / restore への分岐

```
[5/18 02:00 F282 自動実行]
   ↓
[5/18 朝 post-snapshot audit 実行]
   ↓
┌──────────────────────────────────────┐
│ decision = PROCEED_FINANCIALS_RETRY  │ → HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY
│                                      │   (wrapper 方式で env 供給)
└──────────────────────────────────────┘   ↓ (PROCEED smoke)
                                          [derived regen wave (案 A.5)]
                                          ↓ → [signal regen (案 B)] → [W2-B (案 C)]

┌──────────────────────────────────────┐
│ decision = RESTORE_FROM_BACKUP       │ → 別 wave: atomic restore from
│                                      │   ~/fire-backups/...post_restore_71a63a19
└──────────────────────────────────────┘   ↓ → 再度 audit → PROCEED で retry

┌──────────────────────────────────────┐
│ decision = INVESTIGATE               │ → 即停止、log 確認、HQ 判断
└──────────────────────────────────────┘

┌──────────────────────────────────────┐
│ decision = WAIT_FOR_SNAPSHOT         │ → F282 fire 待ち
└──────────────────────────────────────┘
```

---

## §9 安全 gate

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB restore / overwrite | 0 |
| schema migration | 0 |
| 実 J-Quants API call | 0 |
| token / .env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl 実行 | 0 (= subprocess lsof のみ optional) |
| plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push | 0 (= 本 doc commit のみ別途) |
| financials refresh / retry / regen | 0 |
| auto-order / 楽天 / iSPEED | 0 |
| Computer Use | 0 |
| sudo / rm -rf / --no-verify | 0 |
| TODO Excel 更新 | 0 |

---

## §10 関連 file

```
本 wave 出力:
  /tmp/fire_f282_post_snapshot_audit_runner/
    pre_snapshot_audit.json (= 5/17 現状判定、PROCEED)
    pre_snapshot_audit.md
    codex_lane_{A-H}_prompt.txt + result.txt

modified (= 本 wave、commit 済):
  fire repo:
    scripts/jobs/run_f282_post_snapshot_staging_audit.py
    tests/scripts/jobs/test_run_f282_post_snapshot_staging_audit.py
    (commit: 4fcceb5)

設計 doc (= 本 doc、commit 済):
  ~/fire-vault/03_design/F282_POST_SNAPSHOT_STAGING_AUDIT_RUNNER_R1_2026-05-17.md

retained:
  ~/fire-backups/fire.staging.db.bak.20260517_post_restore_71a63a19 (4.8 GB)
  data/fire.staging.db.before_restore_20260517_134908 (353 MB、別 wave 削除推奨)
```
