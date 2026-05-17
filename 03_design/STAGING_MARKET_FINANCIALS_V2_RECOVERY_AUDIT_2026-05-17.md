# FIRE staging market_financials_v2 schema/data recovery audit (2026-05-17)

doc_id: FIRE-STAGING-MF2-RECOVERY-AUDIT-2026-05-17
status: 監査完了 / 実 restore 0 / git push 0 (= read-only audit)
HQ marker: 不要 (= 純監査)
related:
- [[DATA_R1_FINANCIALS_REFRESH_ENABLEMENT_2026-05-17]] — enablement 完了 wave、Lane D HIGH の起点
- [[F111_DERIVED_FRESHNESS_AUDIT_R1_2026-05-16]] — 案 A.5 必須結論、Lane D HIGH 由来

---

## §1 目的

DATA-R1 financials enablement (= 2918cb4) 完了後、現 staging.db に `market_financials_v2` が不在。
financials refresh / 案 A.5 derived regen / 案 B signal regen を実行する前に、staging DB の schema/data 復旧方法を read-only で監査し、backup 復元か migration かを判断する。

---

## §2 git 状態

| repo | branch | HEAD | clean | sync |
|---|---|---|---|---|
| fire | develop | 2918cb4 | ✓ | 0/0 |
| fire-vault | main | 6b9be79 | ✓ | 0/0 |

---

## §3 現 staging DB 状態 (= 353 MB、md5 085799da... = develop 同一)

| table | rows | latest |
|---|---|---|
| **market_financials_v2** | **✗ ABSENT** | - |
| market_financials | 0 | - |
| market_prices_daily | 526,764 | 2026-05-01 (= 古い!) |
| **research_derived_indicators** | **✗ ABSENT** | - |
| **research_watchlist_signals** | **✗ ABSENT** | - |
| market_listings | 4,449 | 2026-05-01 |
| index_data | 58 | 2026-05-01 |
| total tables | 31 | - |

→ DATA-R1 完了直後の状態 (= 2,089,775 prices / 2026-05-15) が消失。

---

## §4 backup 候補 (= 両方とも integrity ok)

### §4.1 backup_recent (~/fire-backups/fire.staging.db.bak.20260517_020005)
- size: 4,804,788,224 bytes
- mtime: 2026-05-17 00:51:37 (= DATA-R1 完了直後)
- integrity: ok
- 37 tables

| table | rows | latest |
|---|---|---|
| **market_financials_v2** | **164,678** | 2026-05-08 |
| market_financials | 1,723 | 2026-05-08 |
| **market_prices_daily** | **2,089,775** | **2026-05-15** |
| **research_derived_indicators** | **3,750** | 2026-05-08 |
| **research_watchlist_signals** | **13,839** | 2026-07-22 (合成) |
| market_listings | 4,449 | 2026-05-01 |
| index_data | 60 | 2026-05-11 |

→ DATA-R1 完了直後の正常 staging snapshot ★

### §4.2 backup_older (data/fire.staging.db.bak.20260511_070004)
- size: 4,803,829,760 bytes
- mtime: 2026-05-11 01:35:30
- integrity: ok
- 34 tables

| table | rows | latest |
|---|---|---|
| market_financials_v2 | 164,678 | 2026-05-08 |
| market_prices_daily | 2,085,284 | 2026-05-08 (= DATA-R1 前) |
| research_derived_indicators | 3,750 | 2026-05-08 |
| research_watchlist_signals | 13,551 | 2026-05-09 |

→ DATA-R1 前の状態。prices+index 更新が無い分、recent より古い。

---

## §5 migration 候補 (= scripts/setup/migrate_market_financials_pk.py)

| 項目 | 内容 |
|---|---|
| 4 段 staging guard | ✓ (FIRE_ENV + 存在 + symlink + basename) |
| 作成 schema | market_financials_v2 (PK: code, disclosure_date, type_of_document) |
| 既存 v1 (market_financials) | **触らない** (drop しない、保持) |
| 既存 data 影響 | v1 行を v2 に INSERT (UNKNOWN doc_type 正規化) |
| 現 staging の v1 = 0 rows | → **v2 も 0 rows で作られる** |
| production / develop refusal | ✓ FIRE_ENV != staging で raise |
| 実行所要時間 | < 1 秒 |

---

## §6 復旧案比較

詳細は `/tmp/fire_financials_v2_recovery_audit/recovery_options.md`。
要約:

| 案 | 復旧範囲 | API | 所要 | 推奨度 |
|---|---|---|---|---|
| **A. backup 復元** | v2 164,678 + derived + signals + 5-15 prices すべて | 0 | 30 秒 | ★★★ |
| B. migration | v2 空 (0 rows)、derived/signals 不変 | 0 | < 1 秒 | × |
| C. table 移植 | 任意 table 選択可、複雑 | 0 | 10-30 分 | △ |
| D. refresh 任せ | 機能しない (schema gate で refuse) | - | - | × |

---

## §7 推奨案 = A. backup 復元

詳細は `/tmp/fire_financials_v2_recovery_audit/recommended_next_wave.md`。

| 項目 | 内容 |
|---|---|
| HQ marker | `HQ_APPROVE_STAGING_DB_RESTORE_FROM_BACKUP_RECENT` |
| 対象 backup | `~/fire-backups/fire.staging.db.bak.20260517_020005` |
| 退避先 | `data/fire.staging.db.before_restore_<TIMESTAMP>` |
| API call | 0 |
| 復旧時間 | ~30 秒 (= 4.8 GB SSD cp) |
| rollback | `mv 退避先 → data/fire.staging.db` (即実行可) |
| 受容判定 | quick_check ok / md5 一致 / 164,678 rows / 5-15 prices / production-develop 不変 |

---

## §8 必要 HQ 承認 (= 次 wave)

1. `HQ_APPROVE_STAGING_DB_RESTORE_FROM_BACKUP_RECENT` (= 本 audit の推奨次 wave)
2. (= 1 完了後) `HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS`
3. `HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH` (= 案 A.5)
4. `HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH` (= 案 B)
5. `HQ_APPROVE_W2B_RE_RUN_FRESH_DATA` (= 案 C)

---

## §9 rollback 方針

復元失敗 / 受容判定 NG 検出時:
```bash
mv data/fire.staging.db.before_restore_<TIMESTAMP> data/fire.staging.db
```
即実行可能、1 ファイル mv のみ。原本退避は復元前必須。

---

## §10 Codex 4 lane 監査結果

| lane | verdict | CRITICAL | HIGH | 摘要 |
|---|---|---|---|---|
| A (current staging) | **APPROVE** | 0 | 1 (= 状況確認 HIGH) | DB read-only 独立検証、現 staging が develop と md5 完全一致、DATA-R1 成果物消失を再現、develop 上書き仮説強支持 |
| B (backup 候補) | (未完了) | - | - | full integrity_check (4.8 GB × 2) 中に出力上限到達、Lane A の独立検証で backup quick_check=ok 確認済のため致命的影響なし |
| C (migration/restore options) | CONCERN | 0 | **1** | **HIGH**: 直接 `cp backup data/fire.staging.db` は atomic でない、中断時に半端 size staging.db リスク。tmp → 検証 → mv に変更必須 |
| D (safety/rollback/next-wave) | CONCERN | 0 | **1** | **HIGH**: 同上 atomic 問題。RECOMMENDED_NEXT=option_a_restore 一致。next-wave 順序妥当、先行 safety (production md5 / lsof / launchd 停止) を提案 |

### §10.1 Lane A APPROVE 詳細
- staging: 353,128,448 bytes / mtime 2026-05-17 02:00:07 / 31 tables / quick_check=ok
- md5: staging/develop 完全一致 (= 085799daf117e59b05d4b9d9aeb7662d)
- market_financials_v2 / research_derived_indicators / research_watchlist_signals 不在、market_financials = 0 rows
- market_prices_daily: 526,764 rows / max(date)=2026-05-01 (DATA-R1 結果消失確認)
- staging独自 table なし (= develop と sqlite_master 完全一致)
- WAL/SHM 不在、journal_mode=delete (= 00:51 backup 後 02:00 上書きが clean cp である証拠)

### §10.2 Lane B 未完了
4.8 GB × 2 backup の full integrity_check が長時間化、Codex の token/出力上限到達で verdict 未出力。
ただし Lane A が backup_recent と current の quick_check=ok を独立確認、本 audit 主 query (= current_staging_audit.json) も backup を read-only で開けたため、致命的判断には影響なし。
別 wave で full integrity_check を実施推奨 (= 復旧前最終確認として)。

### §10.3 Lane C CONCERN (HIGH 1)
**HIGH 指摘**: Option A の `cp backup data/fire.staging.db` は atomic でない、中断時に半端 size の staging DB が残るリスクあり。tmp に copy → quick_check/md5 → 同一 FS 内 mv が安全。
- migration guard は主張通り (FIRE_ENV / 存在 / symlink / basename) 動作確認
- v2 schema 検証は PK 3-key + type_of_document NOT NULL のみ、全列・index は未検証 (= 部分検証)
- backup_recent の v2 schema は migration DDL と列・PK 一致 (= column/NOT NULL/PK が migration と整合)
- v1 = 0 rows なら migration 後 v2 = 0 rows の主張妥当
- Option C は .dump/restore で FK/trigger/index/restore 順序/部分 restore リスク大
- Option A 推奨は論理的に妥当、ただし **atomic flow への修正必須**

### §10.4 Lane D CONCERN (HIGH 1, RECOMMENDED_NEXT: option_a_restore)
**HIGH 指摘**: 同じく atomic でない cp は power loss / kill / disk full で半端 file リスク。

詳細:
1. HQ marker `HQ_APPROVE_STAGING_DB_RESTORE_FROM_BACKUP_RECENT` 既存 marker と衝突なし
2. 退避先 path 衝突なし
3. ディスク空き 826 GiB / data 配下、十分
4. **修正必須**: 直接 cp → tmp copy → verify → atomic mv に変更
5. backup_recent 検証一致 (quick_check ok / 164,678 rows / 2026-05-08 / 5-15 prices)
6. 復旧後 derived regen (案 A.5) は不要に近い (= backup_recent に既存)、ただし最新 base_date 追加で部分実行は妥当
7. 先行 safety 推奨:
   - production DB md5 backup 取得
   - `lsof data/fire.staging.db*` で他プロセス open 不在確認
   - launchd / cron / F282 weekly-snapshot で staging を上書きしうる job 一時停止
   - WAL/SHM 不在再確認

---

## §11 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| DB write (production/develop/staging) | 0 |
| DB restore / overwrite | 0 |
| schema migration | 0 |
| API call | 0 |
| token / .env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push | 0 |
| PR 作成 / merge | 0 |
| sudo / rm -rf / --no-verify | 0 |
| auto-order / 楽天 / iSPEED / Computer Use | 0 |
| TODO Excel 更新 | 0 |

---

## §12 結論

- 現 staging.db は develop ベースの再構築跡 (= md5 一致、market_financials_v2 等不在)
- backup_recent (= 5/17 00:51) が DATA-R1 完了直後の正常 staging で、復旧に最適
- migration (Option B) では v2 が空のままで「復旧」と言えない
- 推奨: **Option A backup 復元 (atomic flow 版)**、別 wave で HQ 承認下 1 回のみ実行
- ★ Codex Lane C/D HIGH 反映: 直接 cp → tmp copy → verify → atomic mv に変更必須
- 先行 safety (Lane D 推奨): production md5 取得 / lsof / launchd 一時停止 / WAL 不在再確認

---

## §13 関連 file

```
本 wave 出力 (read-only audit):
  /tmp/fire_financials_v2_recovery_audit/current_staging_audit.json
  /tmp/fire_financials_v2_recovery_audit/backup_audit.json
  /tmp/fire_financials_v2_recovery_audit/recovery_options.md
  /tmp/fire_financials_v2_recovery_audit/recommended_next_wave.md
  /tmp/fire_financials_v2_recovery_audit/codex_lane_{A,B,C,D}_prompt.txt + result.txt

設計 doc:
  ~/fire-vault/03_design/STAGING_MARKET_FINANCIALS_V2_RECOVERY_AUDIT_2026-05-17.md (本 doc)
```
