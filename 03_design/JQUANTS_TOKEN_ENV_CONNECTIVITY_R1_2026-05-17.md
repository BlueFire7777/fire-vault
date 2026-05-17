# FIRE J-Quants token/env connectivity recovery R1 (2026-05-17)

doc_id: FIRE-JQUANTS-TOKEN-ENV-CONNECTIVITY-R1-2026-05-17
status: 完了 / 値非表示維持 / DB write 0 / API call 0 / git push 0
HQ marker: HQ_APPROVE_JQUANTS_TOKEN_ENV_CONNECTIVITY_NO_VALUE
related:
- [[DATA_R1_STAGING_REFRESH_FINANCIALS_2026-05-17]] — 起点 (= JQuantsAuthError 失敗)
- [[DATA_R1_FINANCIALS_REFRESH_ENABLEMENT_2026-05-17]] — enablement (本 wave で gate 追加)
- [[STAGING_DB_RESTORE_FROM_BACKUP_RECENT_2026-05-17]] — restore 状態維持

---

## §1 目的

financials refresh が JQuantsAuthError で停止した。原因は
`JQUANTS_API_KEY` が shell env に export されていないこと。

retry の前に:
1. token/env 読み込み経路を値非表示で audit
2. runner に env missing precheck gate を実装
3. tests で gate 動作を検証
4. Fujiwara が設定すべき経路を明確化

---

## §2 承認範囲 (= 全て遵守)

| 項目 | 内容 |
|---|---|
| HQ marker | HQ_APPROVE_JQUANTS_TOKEN_ENV_CONNECTIVITY_NO_VALUE |
| 監査範囲 | env 経路 (= shell / .env / dotenv / runner code) 読み取り |
| 実装範囲 | precheck gate + tests のみ |
| 禁止守備 | DB write 0 / API call 0 / token-env-secret 値表示 0 / .env 内容表示 0 / SSH key 表示 0 / financials retry 0 / LINE 0 / launchctl 0 / cron 0 / workflow 0 / git add/commit/push 0 / sudo/rm -rf 0 / auto-order 0 / 楽天/iSPEED 0 / Computer Use 0 / TODO Excel 0 |

---

## §3 git 状態

| 項目 | 値 |
|---|---|
| branch | develop |
| HEAD | 2918cb4 |
| working tree | 1 untracked (= rollback artifact 想定) |
| ahead/behind | 0/0 |

---

## §4 env 存在確認 (= boolean only、値非表示)

| 項目 | 結果 |
|---|---|
| `$JQUANTS_API_KEY` in shell | **NOT SET** |
| `.env` ファイル | EXISTS (= 1,127 bytes、~/fire/.env) |
| `.env` 内 `JQUANTS_API_KEY=` 行 | **PRESENT** (= 21 件の key の 1 つ、値非表示) |
| python-dotenv installed | yes (.venv) |
| runner で load_dotenv() | **NONE** (= grep 0 件) |
| `os.environ.get("JQUANTS_API_KEY")` 経路 | market_data/client.py L53 |

→ **原因確定**: `.env` には key 存在するが、runner は load_dotenv 呼ばない。
   shell env への export がない → `os.environ.get()` で None → JQuantsAuthError。

---

## §5 token 値非表示確認

| 操作 | 結果 |
|---|---|
| `printenv JQUANTS_API_KEY` | **実行せず** |
| `cat .env` | **実行せず** |
| `.env` parse | `^([A-Z_]+)=` で key 名のみ抽出、右辺は読まず |
| precheck コマンド | `[ -n "$VAR" ]` のみ (= boolean echo) |
| precheck gate error_msg | "JQUANTS_API_KEY is missing" のみ (= 値含まれず) |
| test assertion | "dummy_token" / "x-api-key" / "value=" を assert not in |

→ 全 path で値非表示維持。

---

## §6 実装範囲

### §6.1 agents/jquants_daily_refresh.py 追加

| 追加項目 | 内容 |
|---|---|
| 定数 `JQUANTS_API_KEY_ENV_NAME = "JQUANTS_API_KEY"` | market_data.client の定数と同期 (= test で整合性検証) |
| 関数 `assert_jquants_env_for_financials_write(env=None)` | env (= dict 注入可) から key 存在を boolean check、未設定で F286DataR1RunRefused、message に key 値含めず |
| `execute_refresh` 内 precheck gate logic | `has_financials_plan and fr is default_financials_refresh and plan.target_from/to is not None` で env check 発火 |

### §6.2 scripts/jobs/run_jquants_daily_refresh.py
変更なし (= execute_refresh 内 gate で十分、CLI level の追加 check は冗長)

### §6.3 既存挙動への影響 (= 全て不変)

| 状況 | gate 発火 |
|---|---|
| prices/index のみ | No (= 既存通り) |
| financials + DI fake callable | No (= test 経路、API 到達不能) |
| financials + target=None (no_existing_data) | No (= API 到達不能) |
| financials + default callable + env 不在 | **Yes** (= 新 behavior、refresh 前停止) |
| financials + default callable + env 存在 | No (= 通常進行) |
| production/develop | (assert_staging_write_safe で先に refuse) |

---

## §7 precheck gate 詳細

```python
JQUANTS_API_KEY_ENV_NAME: str = "JQUANTS_API_KEY"

def assert_jquants_env_for_financials_write(
    env: Optional[dict[str, str]] = None,
) -> None:
    src = env if env is not None else os.environ
    value = src.get(JQUANTS_API_KEY_ENV_NAME)
    if not value:
        raise F286DataR1RunRefused(
            f"refusing: env {JQUANTS_API_KEY_ENV_NAME} is missing"
            f" (financials write requires API key、本 message は値"
            f"非表示)。 set the env var before --write."
        )
```

呼出位置 (= execute_refresh 内):
```python
if has_financials_plan and fr is default_financials_refresh:
    for p in plans:
        if (
            p.dataset == DATASET_FINANCIALS
            and p.write_supported
            and p.target_from is not None
            and p.target_to is not None
        ):
            assert_jquants_env_for_financials_write()
            break
```

---

## §8 tests (= 全 PASS)

新規 8 件 (= TestFinancialsEnablement 拡張):
1. test_env_gate_constant_matches_market_data_client
2. test_env_gate_helper_passes_when_env_set
3. test_env_gate_helper_refuses_when_env_missing
4. test_env_gate_helper_refuses_when_env_empty_string
5. test_execute_refresh_env_gate_blocks_financials_default_callable
6. test_execute_refresh_env_gate_skipped_for_di_callable
7. test_execute_refresh_env_gate_skipped_for_dry_skipped_financials
8. test_execute_refresh_env_gate_does_not_affect_prices_only

実行結果:
- tests/agents/test_jquants_daily_refresh.py: **71 PASS** (= 60 → 71、+11 を最終 count、本 wave で +8)
- wider tests/agents/ tests/scripts/jobs/: **2,662 PASS**
- regression: 0

---

## §9 Codex 6 lane 監査結果

| lane | verdict | CRITICAL | HIGH | 摘要 |
|---|---|---|---|---|
| A (env/token 経路) | **APPROVE** | 0 | 0 | shell env / .env / dotenv / runner code 独立検証、値非表示維持 |
| B (precheck gate 実装) | **APPROVE** | 0 | 0 | error_msg 値非漏洩、dict 注入 + os.environ fallback OK、既存挙動不変 |
| C (tests / edge cases) | **APPROVE** | 0 | 0 | 71 PASS / 2,662 PASS、test 3-8 全項目独立検証 |
| D (secret 漏洩防止) | **APPROVE** | 0 | 0 | /tmp + vault doc + agents/tests 全件 secret 漏洩 0 |
| E (retry readiness) | CONCERN | 0 | **1** | rollback 手順が `mv` 単体だと WAL/SHM 整合不十分 (= 前 wave doc で既反映済の指摘) |
| F (safety/next-wave) | CONCERN | 0 | **2** | (1) rollback artifact mismatch、(2) F282 weekly-snapshot 衝突可能性 |

### §9.1 Lane A APPROVE
- `[ -n "$JQUANTS_API_KEY" ]` exit=1 で shell env 非表示確認 ✓
- ~/fire/.env existence + size 確認、内容非読 ✓
- python-dotenv import OK in .venv
- agents/jquants_daily_refresh.py に load_dotenv なし
- market_data/client.py L53 os.environ.get、raise 文言値非表示

### §9.2 Lane B APPROVE
- secret 非表示 error_msg
- env 注入 + os.environ fallback 両方機能
- gate 発火条件 (staging guard 後、default callable、target range あり) 妥当
- DI fake / no target range skip 正しい
- 定数 drift 検知 test 有効
- prices/index/derived/signals 経路不変

### §9.3 Lane C APPROVE
- TestFinancialsEnablement 25 件、全ファイル 71 件 PASS
- wider tests/agents tests/scripts/jobs 2,662 PASS
- test 3-8 独立検証 + monkeypatch.delenv 確認
- edge case blocking なし

### §9.4 Lane D APPROVE
- agents に env value print/log/repr 経路なし
- error は key 名 + missing のみ
- test で "dummy_token" / "x-api-key" / "value=" 非混入 assert
- /tmp + vault doc に実 secret 漏洩なし
- echo/printenv/cat 実行経路なし

### §9.5 Lane E CONCERN (HIGH 1) — 採用、next_retry_readiness.md に反映済
**HIGH 指摘**: rollback `mv ... fire.staging.db` 単体では WAL/SHM 整合不十分。

**反映**: next_retry_readiness.md の rollback 手順を sidecar 退避対応版に修正
(= 前 wave STAGING_DB_RESTORE_FROM_BACKUP_RECENT で既に確立した手順):
1. writer 停止 (lsof 確認)
2. 現 staging.db + WAL/SHM を rollback_keep_<TS> に退避
3. 退避 backup を mv で本体位置に戻す
4. quick_check 確認

### §9.6 Lane F CONCERN (HIGH 2) — 採用、retry wave 前に対応必要

**HIGH 1**: rollback artifact `data/fire.staging.db.before_restore_20260517_134908`
は md5 `085799...` (= 353 MB、古い develop 同期状態) で、現 restored staging
`71a63a19...` (= 4.8 GB、market_financials_v2 含む) とは異なる。
documented `mv` rollback だと **古い状態に戻ってしまい market_financials_v2 が消失**。

**対策**:
- retry wave 前に **現 restored staging の新 backup を取得** (= md5 71a63a19... の snapshot)
- rollback path を「新 backup → atomic mv」へ変更
- old rollback artifact (= 353 MB) は別 wave で削除 or rename して用途明確化

**HIGH 2**: F282 weekly-snapshot plist が staging に書き込む経路あるかも不明。
loaded 状態未検証。retry 中に衝突可能性。

**対策**:
- retry wave 前に `launchctl list | grep jp.fire.weekly-snapshot` で load 状態確認
- loaded ならスケジュール時刻 (= 大概 weekly 1 回) と retry 実行時刻が重ならないことを確認
- 衝突可能性あれば `launchctl unload` で一時停止 (= 別 wave、HQ 承認必要)

**RECOMMENDED_NEXT** = `option_b_skip_to_derived` (= Option B 推奨継続)

---

## §10 安全 gate

| 項目 | 結果 |
|---|---|
| DB write (prod/dev/staging) | 0 |
| 実 J-Quants API call | 0 |
| token / .env / secret 値表示 | 0 |
| .env 内容表示 | 0 (= key 名のみ抽出) |
| SSH 秘密鍵表示 | 0 |
| LINE 送信 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push / PR | 0 |
| schema migration | 0 |
| financials refresh retry | 0 (= 本 wave 外) |
| auto-order / 楽天 / iSPEED | 0 |
| sudo / rm -rf / --no-verify | 0 |
| Computer Use | 0 |
| TODO Excel | 0 |

---

## §11 financials retry 可否

| 条件 | 状態 |
|---|---|
| precheck env gate 実装 | ✓ |
| env gate が値非表示で動作 | ✓ |
| 既存挙動 (prices/index/DI) 維持 | ✓ |
| production/develop refuse 維持 | ✓ |
| full_5year refuse 維持 | ✓ (= 別 layer guard) |
| **JQUANTS_API_KEY 供給** | **× 未対応** (= Fujiwara 設定要) |

→ **conditional ready**: env 設定後に retry 可能。
   ただし Lane H 推奨は Option B (= retry skip → derived へ)。

---

## §12 藤原側で必要な設定 (= 本 wave 外)

選択肢 (= どれか 1 つ実施):

### A. shell rc に export 追加 (★ 軽量推奨)
```
echo 'export JQUANTS_API_KEY="..."' >> ~/.zshrc  # 値非表示で運用
source ~/.zshrc
[ -n "$JQUANTS_API_KEY" ] && echo "OK"
```

### B. wrapper shell script
```
# scripts/wrappers/jquants_refresh.sh
set -a; source .env; set +a
FIRE_ENV=staging .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh "$@"
```

### C. python-dotenv を runner に組込 (= 別 wave 実装)
```python
# scripts/jobs/run_jquants_daily_refresh.py 先頭
try:
    from dotenv import load_dotenv
    load_dotenv()
except ImportError:
    pass
```

### D. launchd plist の `EnvironmentVariables` (= cron 経路で必要時)

---

## §13 次アクション

★ **HQ 判断要 (= Option A vs B)**:

### Option B (★ Lane H 推奨、即進行可)
- HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH (= 案 A.5)
- 現 mf_v2 (164,678 / 2026-05-08、9 days stale) で十分
- env 設定不要

### Option A (= retry を先行する場合)
1. Fujiwara が §12 のいずれかで env 整備
2. HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY
   - 本 wave の precheck gate が env 不在を早期検知
   - 整備後の retry で確実に動作

---

## §14 関連 file

```
本 wave 出力:
  /tmp/fire_jquants_token_env_recovery/
    env_connectivity_audit.md
    test_results.txt
    next_retry_readiness.md
    codex_lane_{A,B,C,D,E,F}_prompt.txt + result.txt

modified (= 本 wave、commit 未実施):
  agents/jquants_daily_refresh.py (= +35 lines、precheck gate)
  tests/agents/test_jquants_daily_refresh.py (= +130 lines、8 件 test)

unchanged:
  market_data/client.py (= 既存 JQUANTS_API_KEY 経路、参照のみ)
  scripts/jobs/run_jquants_daily_refresh.py (= CLI 変更なし)
  scripts/jobs/backfill_market_financials.py (= delegate 先、変更なし)

設計 doc (= untracked):
  ~/fire-vault/03_design/JQUANTS_TOKEN_ENV_CONNECTIVITY_R1_2026-05-17.md
```
