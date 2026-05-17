# FIRE DATA-R1 staging financials refresh (2026-05-17、five_codes)

doc_id: FIRE-DATA-R1-STAGING-REFRESH-FINANCIALS-2026-05-17
status: **FAILED (= fetch 段階で JQuantsAuthError、DB write 0、安全側停止)**
HQ marker: HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS
related:
- [[STAGING_DB_RESTORE_FROM_BACKUP_RECENT_2026-05-17]] — restore 直後の状態起点
- [[DATA_R1_FINANCIALS_REFRESH_ENABLEMENT_2026-05-17]] — 本 wave で使用した enablement code
- [[F111_DERIVED_FRESHNESS_AUDIT_R1_2026-05-16]] — 案 A.0 提案起点

---

## §1 目的と結果

staging restore で復旧した market_financials_v2 (= 164,678 rows /
2026-05-08) を five_codes smoke で最新化する予定だったが、
**JQuantsAuthError (API key not provided) で fetch 段階で停止**、
DB write 0 で完全に安全側で終了。

---

## §2 承認範囲 (= 全て遵守)

| 項目 | 内容 |
|---|---|
| HQ marker | HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS |
| 対象 DB | data/fire.staging.db のみ |
| 対象 dataset | financials のみ |
| smoke_type | five_codes のみ |
| max-days | 3 |
| write 回数 | 1 回 (= 試みた、ただし auth で失敗) |
| 禁止守備 | production/develop write 0 / derived/signal regen 0 / W2-B 0 / schema migration 0 / LINE 0 / launchctl 0 / cron 0 / workflow 0 / git add/commit/push 0 / PR/merge 0 / --no-verify 0 / sudo/rm -rf 0 / auto-order 0 / 楽天/iSPEED 0 / Computer Use 0 / TODO Excel 0 / token-env 表示 0 |

---

## §3 precheck (= 全 PASS)

| 項目 | 結果 |
|---|---|
| git | develop / clean (1 untracked = rollback artifact 想定) / 0/0 |
| DB md5 pre | prod b1df4673... / dev 085799da... / stg 71a63a19... |
| F282 plist 3 file | 全 md5/size/mtime 記録 |
| lsof data/fire.staging.db* | 空 (= no process holding) |
| WAL | 0 bytes ✓ (= 続行可) |
| SHM | 32 KB (= 前 wave Codex 検査由来、害なし、記録) |
| JQUANTS_API_KEY | not in shell env (= runner 内部 load 想定だった、これが後の失敗原因) |
| staging pre quick_check | ok |
| staging mf_v2 rows | 164,678 / latest 2026-05-08 / PK dup 0 / null 0 |
| production/develop quick_check | ok / ok |

---

## §4 pre-refresh freshness

| table | rows | latest |
|---|---|---|
| market_financials_v2 | 164,678 | 2026-05-08 |
| market_prices_daily | 2,089,775 | 2026-05-15 |
| research_derived_indicators | 3,750 | 2026-05-08 |
| research_watchlist_signals | 13,839 | 2026-07-22 |

---

## §5 dry-run (= PASS)

```
FIRE_ENV=staging .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
  --db-path data/fire.staging.db --db-label staging \
  --datasets financials --financials-smoke-type five_codes \
  --max-days 3 --dry-run --output-json /tmp/.../pre_dry_run.json
```

結果:
- plan: financials / market_financials_v2 / before_max=2026-05-08 /
  target=2026-05-09〜2026-05-11 / span=3 / reason=capped_to_max_days_3
- write_invoked_count=0, error_count=0
- API call 0, token 表示 0
- full_5year 経路ではない (= argparse choices で full_5year 拒否済)
- production/develop refusal guard 維持

---

## §6 refresh 実行結果 (= FAILED)

```
FIRE_ENV=staging .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
  --db-path data/fire.staging.db --db-label staging \
  --datasets financials --financials-smoke-type five_codes \
  --max-days 3 --write \
  --output-json /tmp/.../refresh_result.json \
  --completion-report /tmp/.../completion_report.txt
```

実行ログ抜粋:
- started_at: 2026-05-17 14:28:11
- five_codes: ['72030', '67580', '80350', '83060', '16050']
- row_count_before: 164,678 / db_size_before: 4,582 MB
- `--- /fins/summary fetch (sleep 0.7 sec/req) ---`
- **status: error:JQuantsAuthError**
- **error: API key not provided. Set JQUANTS_API_KEY env var or pass api_key argument.**
- inserted=0, write_invoked=True, executed=True
- exit code 4 (= PARTIAL/ERROR FAIL、明示的 non-zero 停止)

→ **silent_failure_detected ではなく明示的 error**
→ DB write 0 (= JQuantsClient init 段階で raise、_upsert_row_v2 未到達)

---

## §7 post-refresh 受容判定 (= 失敗側)

| 項目 | 期待 (= 失敗時) | 実 | OK |
|---|---|---|---|
| staging md5 | pre と一致 (= 71a63a19...) | 71a63a19... | ✓ |
| staging mtime | 2026-05-17 13:49:09 (= restore 時刻維持) | 同上 | ✓ |
| quick_check | ok | ok | ✓ |
| mf_v2 rows | 164,678 (= 不変) | 164,678 | ✓ |
| mf_v2 latest | 2026-05-08 (= 不変) | 2026-05-08 | ✓ |
| production md5 | b1df4673... | b1df4673... | ✓ |
| develop md5 | 085799da... | 085799da... | ✓ |
| F282 plist 3 file | 全 size/mtime/md5 不変 | 全 ✓ | ✓ |
| WAL post | 0 bytes (= 不変) | 0 bytes | ✓ |
| SHM post | 32 KB (= 不変) | 32 KB | ✓ |
| lsof post | empty | empty | ✓ |

→ 失敗時の安全側挙動として全項目 PASS。DB write 0 完全証明。

---

## §8 DB md5 (= production/develop 完全不変、staging も不変)

| DB | pre | post | 結果 |
|---|---|---|---|
| production | b1df4673e5c3645fbe2c5f490ffac043 | b1df4673e5c3645fbe2c5f490ffac043 | 不変 ✓ |
| develop | 085799daf117e59b05d4b9d9aeb7662d | 085799daf117e59b05d4b9d9aeb7662d | 不変 ✓ |
| staging | 71a63a19694385db4344246be60a2f91 | 71a63a19694385db4344246be60a2f91 | 不変 (= refresh 失敗で write 0) |

---

## §9 F282 plist (= 全 3 file 完全不変)

| file | size | mtime | md5 |
|---|---|---|---|
| ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist | 1,772 | 5月 12 22:46 | e4ea05e88625040d1f79151948420b4f |
| docs/launchd/jp.fire.weekly-snapshot.plist | 1,772 | 5月 16 21:29 | e4ea05e88625040d1f79151948420b4f |
| docs/launchd/jp.fire.weekly-snapshot-smoke.plist | 1,844 | 5月 16 21:29 | 0667c02e66049db46c209e55f3b9cf72 |

---

## §10 安全 gate

| 項目 | 結果 |
|---|---|
| 実 DB write (prod/dev/staging) | 0 (= 全 md5 完全不変) |
| 試みた API call | 1 (= auth 段階で即失敗、データ送信 0) |
| 成功した API call | 0 |
| token / .env / secret 値表示 | 0 (= error_msg は "not provided" のみ、key 値含まれず) |
| LINE 送信 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push / PR | 0 |
| schema migration | 0 |
| sudo / rm -rf / --no-verify | 0 |
| auto-order / 楽天 / iSPEED | 0 |
| Computer Use / Playwright / Selenium | 0 |
| TODO Excel 更新 | 0 |
| derived regen / signal regen / W2-B re-run | 0 (= 本 wave スコープ外) |

---

## §11 原因仮説

1. JQUANTS_API_KEY が runner プロセス環境変数に未供給
   - shell env で未設定状態だった (precheck で確認)
   - `FIRE_ENV=staging` は明示 export したが、JQUANTS_API_KEY は未 export
2. backfill_market_financials.run_smoke 内で `JQuantsClient()` init し、
   client が `JQUANTS_API_KEY` env を読む経路
3. .env auto-load 機構は本 runner に組み込まれていない (= load_dotenv 不在)
4. 既存 backfill smoke が単独で動く際は別経路 (= shell rc / wrapper) で
   env 供給されていた可能性

---

## §12 Codex 8 lane 監査結果

| lane | verdict | CRITICAL | HIGH | 摘要 |
|---|---|---|---|---|
| A (cmd/guard/scope) | APPROVE | 0 | 0 | financials only / five_codes / 3 段 guard / secret 露出 0 確認 |
| B (precheck/lsof/WAL/md5) | **CONCERN** | 0 | **1** | JQUANTS_API_KEY absent を「続行可」とした判断が不十分 (= refresh gate に組み込むべきだった) |
| C (dry-run plan/gating) | APPROVE | 0 | 0 | dry-run と write の plan 完全一致、JQuantsAuthError と silent-failure の区別正しい |
| D (refresh result/silent-failure) | APPROVE | 0 | 0 | JQuantsAuthError は exception 経路で明示 error、exit code 4 で正しく停止、retry なし |
| E (financials freshness delta) | APPROVE | 0 | 0 | pre/post 7 項目完全一致、derived regen 入力として基準値十分 |
| F (production/develop 不変 / F282 plist) | APPROVE | 0 | 0 | 全 DB md5 一致、plist 3 file 一致、weekly-snapshot.err 0 bytes |
| G (token/API/LINE/secret 安全) | APPROVE | 0 | 0 | /tmp 22 file 全件検査、secret 漏洩 0、LINE/launchd invoke 0 |
| H (rollback/next-wave) | APPROVE | 0 | 0 | RECOMMENDED_NEXT: option_b_skip_to_derived、rollback artifact 健全 |

### §12.1 Lane B HIGH (= 採用、私の判断ミスを認める)

**指摘**: precheck で「JQUANTS_API_KEY not in current shell env (will be loaded by runner from secret)」と楽観的に判定し続行したが、実際は runner にも `.env` auto-load なし → refresh 確定失敗。

**採用理由**: 妥当。env 存在確認は token 値を表示せず可能 (= `[ -n "$VAR" ]`)。precheck の「続行可」判定基準に env 存在 gate を追加すべきだった。

**今後の対応**:
- 次 retry wave (= Option A) では precheck で `[ -n "$JQUANTS_API_KEY" ]` を gate 化
- env 不在なら refresh 試行せず停止、wrapper / shell rc 設定後の retry へ誘導

### §12.2 Lane H 推奨 = option_b_skip_to_derived

**判定根拠**:
1. rollback artifact `data/fire.staging.db.before_restore_20260517_134908` は quick_check ok / 353 MB 完全 DB、必要時 mv で復旧可能
2. 現 staging `71a63a19...` / mtime 13:49:09 / mf_v2 164,678 / 2026-05-08 / PK dup 0 は健全
3. 9 calendar days stale (= 5 business days lag) は soft financials threshold 内
4. derived regen / signal regen / W2-B の前提は本 wave 失敗で崩れていない (= restore で揃った)
5. retry を選ぶなら env 整備 + 別 wave

---

## §13 推奨対応 (= HQ 判断要)

### Option A: env 整備 → financials refresh retry (= 別 wave)
- shell rc / wrapper / launchd plist で JQUANTS_API_KEY 供給設定 (本 wave 外)
- 別 wave で HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY 等の marker 提案
- 同 atomic flow + token 供給確認

### Option B: financials refresh skip → derived regen wave へ直接 (★ 推奨)
- 現 market_financials_v2 (164,678 / 2026-05-08) は derived regen 入力として十分
- F111-DERIVED-FRESHNESS-AUDIT Lane D: 案 A.5 は fresh financials なしでも実行可
- 9 日 stale は決算シーズン外なら許容範囲
- 次 wave: HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH (= 案 A.5) へ直接進む

---

## §14 rollback artifact / sidecar の扱い

- retained: `data/fire.staging.db.before_restore_20260517_134908` (353 MB) 保持
- sidecar `data/fire.staging.db-wal` (0 B) / `-shm` (32 KB) 保持
- 本 wave 失敗で staging DB 自体は restore 直後の状態のまま、
  rollback artifact は引き続き次 wave (= derived regen 等) 完了まで保持

---

## §15 関連 file

```
本 wave 出力:
  /tmp/fire_data_r1_financials_refresh/
    precheck.json
    pre_dry_run.json (= dry-run 結果)
    refresh_result.json (= 失敗結果)
    completion_report.txt (= runner 自動生成)
    post_refresh_audit.json
    financials_refresh_delta.md

設計 doc (= 本 wave、untracked):
  ~/fire-vault/03_design/DATA_R1_STAGING_REFRESH_FINANCIALS_2026-05-17.md (本 doc)

retained (= 前 wave restore 由来):
  data/fire.staging.db.before_restore_20260517_134908 (353 MB)
```
