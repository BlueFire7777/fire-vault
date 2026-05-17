# FIRE DATA-R1 financials refresh enablement / guard 確認 (2026-05-17)

doc_id: FIRE-DATA-R1-FIN-ENABLE-2026-05-17
status: enablement 完了 (= 実 write 0、実 API 0、token 表示 0)
HQ marker: HQ_APPROVE_DATA_R1_FINANCIALS_ENABLEMENT_NO_WRITE
related:
- [[F111_DERIVED_FRESHNESS_AUDIT_R1_2026-05-16]] — 鮮度 audit + Lane D HIGH 起点
- [[DATA_R1_STAGING_REFRESH_PRICES_INDEX_2026-05-16]] — 案 A (prices+index) 完了 wave
- 次 wave: HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS

---

## §1 目的

F111-DERIVED-FRESHNESS-AUDIT-R1 で `market_financials_v2` が 2026-05-08 stale と判明 (Lane D HIGH: 5 月決算シーズン直撃)。 `research_derived_indicators` 再生成 (= 案 A.5) の前に financials を `run_jquants_daily_refresh.py` で安全に staging refresh 対象に追加できるか確認 + 実装。

★ 本 wave は **enablement only**: 実 financials write / API / token / env 参照は行わず、コード経路と guard と tests のみ整備。

---

## §2 承認範囲

| 項目 | スコープ |
|---|---|
| HQ marker | HQ_APPROVE_DATA_R1_FINANCIALS_ENABLEMENT_NO_WRITE |
| 対象 file | agents/jquants_daily_refresh.py / scripts/jobs/run_jquants_daily_refresh.py / tests/agents/test_jquants_daily_refresh.py |
| 対象 DB | data/fire.staging.db (read-only / 実 write 0) |
| 禁止 | 実 DB write / staging write / production-develop write / J-Quants API 実行 / token/env/secret 参照 / LINE 送信 / launchctl / plist / cron / workflow 変更 / PR 作成 / merge / --no-verify / sudo / rm -rf / auto-order / 楽天/iSPEED / Computer Use / TODO Excel 更新 / git add / commit / push |

---

## §3 git 状態

| 項目 | 値 |
|---|---|
| branch | develop |
| working tree | (本 wave で 3 file 変更、commit 未実施) |
| ahead/behind origin/develop | 0/0 |
| HEAD | 00c1e38f |

---

## §4 実装

`/tmp/fire_data_r1_financials_enablement/implementation_summary.md` 参照。
要点:

- `DATASET_FINANCIALS = "financials"` 追加
- `WRITE_ENABLED_DATASETS` に `DATASET_FINANCIALS` 追加 (prices/index と同等扱い)
- `DATASET_TABLE_INFO[DATASET_FINANCIALS] = ("market_financials_v2", "disclosure_date", "code")`
- `FinancialsRefreshCallable` Protocol 追加
- `default_financials_refresh` 関数: `backfill_market_financials.run_smoke` を delayed import で delegate (= 既存 4 段 staging guard / smoke_type 制限 / pagination hard limit / duplicate audit を再利用、新規 API 経路を作らない)
- CLI `--financials-smoke-type` (choices=five_codes/mini_100)、`full_5year` は argparse 拒否 + 二重 guard
- `execute_refresh` に `financials_refresh` / `financials_smoke_type` kwarg + DATASET_FINANCIALS branch

---

## §5 guard 防御 (= 多層)

| 層 | 場所 | 拒否対象 |
|---|---|---|
| L1 argparse choices | CLI `--financials-smoke-type` | `full_5year` 等の未許可 smoke_type |
| L2 execute_refresh early check | agents | invalid smoke_type を build_plans 後すぐ拒否 |
| L3 default_financials_refresh | agents | 直接呼び出された場合も `full_5year` 拒否 |
| L4 assert_staging_write_safe | agents | db_label / basename / FIRE_ENV / symlink |
| L5 backfill_market_financials._check_staging_guard | scripts/jobs | 4 段 (FIRE_ENV / DB 存在 / symlink / basename) |
| L6 backfill_market_financials hq_approved | scripts/jobs | `SMOKE_TYPE_FULL_5YEAR + hq_approved=False` で refuse、本 runner は常に `hq_approved=False` で delegate |
| L7 schema validation | backfill_market_financials._verify_v2_schema_or_raise | market_financials_v2 三キー PK 不在で refuse |

→ `full_5year` は本 runner からは **構造的に到達不可** (= L1 で argparse refuse、L1 を通っても L6 で hq_approved=False のため refuse)。

---

## §6 tests (= 全 PASS、`/tmp/.../test_results.txt` 参照)

### §6.1 既存 (44 件)
prices/index/derived/signals 既存テスト全 PASS。

### §6.2 新規 (12 (= 13 Lane A regression 追加後) 件、TestFinancialsEnablement)

1. `test_financials_in_available_and_write_enabled` — 定数登録確認
2. `test_financials_table_info_registered` — DATASET_TABLE_INFO 登録確認
3. `test_financials_smoke_type_defaults_five_codes` — default + allowed list 確認、full_5year not in allowed
4. `test_financials_plan_marks_write_supported` — build_plans で write_supported=True + before_state 取得
5. `test_execute_financials_via_di_no_api` — fake_financials DI、smoke_type / symbols 伝播
6. `test_execute_financials_custom_smoke_type` — --financials-smoke-type mini_100 伝播
7. `test_execute_financials_invalid_smoke_type_refused` — full_5year を execute_refresh で refuse、token/API_KEY 不漏洩
8. `test_execute_financials_partial_marks_status` — failed_codes → status=partial、後続 plan は aborted_after_partial
9. `test_financials_dry_run_no_call` — build_plans は fetcher を呼ばない
10. `test_financials_production_refused` — production DB で refuse
11. `test_financials_develop_refused` — develop DB で refuse
12. `test_financials_fire_env_non_staging_refused` — FIRE_ENV=develop で refuse
13. `test_default_financials_refresh_rejects_unknown_smoke_type` — 直接呼び出しでも full_5year refuse、token 不漏洩
14. test_all_datasets (更新): 4→5、financials.write_supported=True 検証
12 (= 13 Lane A regression 追加後). (既存 TestModuleSourceSafety 群、`load_dotenv` / `subprocess` / `LINE` / `place_order` 等の禁止 import 不在再検証)

### §6.3 全体 regression
`pytest tests/agents/ tests/scripts/jobs/` で **2,650 PASS** (= 既存 47 warnings は GoalConfig fallback、本 wave 由来なし)。

---

## §7 dry-run smoke (= 実 API/DB write 0)

### §7.1 financials 単独
```
FIRE_ENV=staging .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
  --db-path data/fire.staging.db --db-label staging \
  --datasets financials --max-days 3 --dry-run
```
→ plan: `[financials] table=market_financials_v2 reason=no_existing_data_use_full_backfill`、`write_invoked_count=0`、`error_count=0`

### §7.2 全 5 datasets dry-run
→ 5 plans 全て生成、prices/index/financials は write_supported=True、derived/signals は False

### §7.3 --financials-smoke-type full_5year 拒否
→ argparse: `error: argument --financials-smoke-type: invalid choice: 'full_5year' (choose from five_codes, mini_100)`

---

## §8 staging.db 状態 (= 観察、要対応事項)

| table | 状態 |
|---|---|
| market_financials (v1) | 0 rows |
| **market_financials_v2** | **存在しない** |
| market_prices_daily | 存在 / 353 MB |
| market_listings | 存在 |

★ F111-DERIVED-FRESHNESS-AUDIT-R1 は `market_financials_v2: 164,678 rows / latest 2026-05-08` と記録していたが、現 staging.db には v2 table が無い。staging.db が wave 間で reset された可能性。

→ 次 wave (HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS) 着手前に schema 確認 (= `scripts/setup/migrate_market_financials_pk.py` 実行有無、market_financials_v2 schema 作成必要か) を別 wave で実施推奨。本 wave (= enablement) には影響なし (= 実 write 0)。

---

## §9 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| 実 DB write | 0 |
| schema migration | 0 |
| 実 J-Quants API call | 0 |
| token / .env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push / PR | 0 (= 本 wave 禁止) |
| auto-order / 楽天 / iSPEED | 0 (= module 内全 negation/safety_note) |
| Computer Use / Playwright / Selenium | 0 |
| --no-verify / sudo / rm -rf / subprocess | 0 |

---

## §10 Codex 4 lane 監査結果

(§10.1-§10.4 に追記。CRITICAL/HIGH 検出時は §12 結論で REJECT)

---

## §11 financials staging refresh 可否 (= 次 wave 判定)

| 条件 | 状態 |
|---|---|
| WRITE_ENABLED_DATASETS に financials 追加済 | ✓ |
| 4 段 staging guard + 二重 financials guard 実装 | ✓ |
| full_5year 構造的到達不能 | ✓ |
| backfill_market_financials.run_smoke 既存 4 段 guard + schema validation 再利用 | ✓ |
| 既存 prices/index regression 0 | ✓ (44 既存 test 全 PASS) |
| 新規 financials 専用 test 12 (= 13 Lane A regression 追加後) 件 PASS | ✓ |
| dry-run で API/DB write 0 確認 | ✓ |
| staging.db に market_financials_v2 schema 存在 | **未確認** (= 別 wave で schema check 必要) |

→ **schema 確認後 OK** で次 wave (`HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS`) を切れる。

---

## §12 結論 / 次アクション

### §12.1 結論
- enablement 完了、guard 多層化、tests 全 PASS、API/DB write 0
- staging.db の market_financials_v2 schema 確認が次 wave 前 prerequisite
- コード変更は本 wave に閉じる (= 別 wave で commit / push)

### §12.2 次 wave 推奨

★ Lane D HIGH の扱い: **本 wave スコープ外、別 wave 必須**:
  - 現 staging.db に market_financials_v2 不在 (= F111-DERIVED-FRESHNESS-AUDIT 記録の 164,678 rows と矛盾、rebuild 跡)
  - backup: `~/fire-backups/fire.staging.db.bak.20260517_020005` / `data/fire.staging.db.bak.20260511_070004`
  - 本 wave の enablement は復旧の有無に依存せず正しく動作 (= 復旧後即 write 可能、未復旧時は schema gate で安全に refuse)

1. staging.db market_financials_v2 schema/データ復旧 wave (= 別 marker、Lane D HIGH)
2. HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS で実 write (= --financials-smoke-type five_codes 推奨開始)
3. HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH (案 A.5)
4. HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH (案 B)
5. HQ_APPROVE_W2B_RE_RUN_FRESH_DATA (案 C)

---

## §13 関連 file

```
modified (= 本 wave、commit 未実施):
  agents/jquants_daily_refresh.py
  scripts/jobs/run_jquants_daily_refresh.py
  tests/agents/test_jquants_daily_refresh.py

unchanged (= 既存 delegate 先):
  scripts/jobs/backfill_market_financials.py
  simulation/research_lane/financials_mapping.py
  market_data/client.py

output:
  /tmp/fire_data_r1_financials_enablement/implementation_summary.md
  /tmp/fire_data_r1_financials_enablement/test_results.txt
  /tmp/fire_data_r1_financials_enablement/dry_run_smoke.json
  /tmp/fire_data_r1_financials_enablement/all_datasets_dry_run.json
  /tmp/fire_data_r1_financials_enablement/codex_lane_*_prompt.txt + result.txt
  ~/fire-vault/03_design/DATA_R1_FINANCIALS_REFRESH_ENABLEMENT_2026-05-17.md (本 doc)
```
