---
id: FIRE-CODEX-R1-WAVE60-JQUANTS-DAILY-REFRESH-STAGING-results
phase: 本番 v0 中核 / Wave 60-jquants-daily-refresh-staging / 価格鮮度真の解決
priority: 最高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
hq_markers:
  - HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1
  - HQ_APPROVE_JQUANTS_TOKEN_READ=1
rows_inserted: 48
write_invoked_count: 1
error_count: 0
market_prices_daily_max_date_before: 2026-05-08
market_prices_daily_max_date_after: 2026-05-14
---

# Wave 60-jquants-daily-refresh-staging Results — Price Freshness Recovery

## §1 結論

🎉 **真の解決完了** — J-Quants V2 daily refresh で staging market_prices_daily
を 2026-05-08 → **2026-05-14** (= 6 営業日分の catch-up) へ進めた。production /
develop DB 完全不変、token 値非表示維持、4 段 guard 全 PASS、Codex 4 lane 全 YES。

**D10 chain で 137A0 Cocolive が close=0 → 739 で entry 候補化 (rank 3)**、
**5729 日本精鉱 -18.5% 急落判明** (= caution で見送り推奨だったので影響軽微)、
**340A0 ジグザグ -4.5% 緩やか下落** (= entry 候補 #1 維持、risk=1,990→1,900 円)。

## §2 chain 実行サマリ

### 2.1 token 存在確認 (値非表示)

```
JQUANTS_REFRESH_TOKEN: NOT_SET
JQUANTS_USER:          NOT_SET
JQUANTS_PASSWORD:      NOT_SET
JQUANTS_API_KEY:       SET (len=43)   ← V2 認証はこれ 1 つで完結
JQUANTS_ID_TOKEN:      NOT_SET
```

→ J-Quants V2 では `x-api-key` header で API key 認証 (= Codex Lane C 確認済み)。
refresh_token → id_token の 2 段交換は不要。

### 2.2 dry-run smoke (= API 0 / DB write 0)

```
$ .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
    --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
    --datasets prices --max-days 6 \
    --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
    --dry-run --output-json /tmp/fire_d10_prep/jq_dryrun_d9.json
```

| 項目 | 値 |
|---|---|
| symbols | csv:12 件 (= D9 entry 候補 9 + recently_seen 3) |
| datasets | prices (= market_prices_daily 限定) |
| target span | 2026-05-09〜2026-05-14 (= 6 営業日) |
| dry_run | True / write: False |
| rows_inserted | 0 / write_invoked_count: 0 |

### 2.3 staging refresh 実行

```
$ set -a; source /Users/bluefire/fire/.env; set +a
$ FIRE_ENV=staging \
  HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 \
  HQ_APPROVE_JQUANTS_TOKEN_READ=1 \
  .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
    --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
    --datasets prices --max-days 6 \
    --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
    --sleep-seconds 0.3 --write \
    --output-json /tmp/fire_d10_prep/jq_refresh.json \
    --completion-report /tmp/fire_d10_prep/jq_completion.txt
```

| 項目 | 結果 |
|---|---|
| rows_inserted | **48** (= 12 銘柄 × 4 営業日、5/9 と 5/10 はデータなし or 取引日外) |
| write_invoked_count | 1 |
| error_count | **0** ✓ |
| status | ok |
| after_state.max_date | **2026-05-14** ✓ |
| after_state.row_count | 2,085,332 (= 2,085,284 + 48) |
| db_mtime_before | 2026-05-14T14:47:55.904245 |
| db_mtime_after | 2026-05-14T17:14:38.205599 |

### 2.4 production / develop md5 (= 不変であるべき)

```
fire.db        : b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓
fire.develop.db: 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓
```

## §3 D9 12 銘柄: stale vs actual close 比較

| code | name | stale (5/8) | actual (5/14) | 変化率 |
|---|---|---|---|---|
| 87470 | 豊トラスティ証券 | 2,490 | 2,420 | -2.8% |
| 57290 | 日本精鉱 | 2,202 | 1,795 | **-18.5% (急落)** |
| 34890 | フェイスネットワーク | 734 | 699 | -4.8% |
| **340A0** | **ジグザグ (= D9 entry #1)** | **398** | **380** | **-4.5%** |
| 37980 | ＵＬＳグループ | 519 | 505 | -2.7% |
| **137A0** | **Cocolive** | **0 (excluded)** | **739** | **新 entry 化** |
| 79910 | マミヤ・オーピー | 1,246 | 1,177 | -5.5% |
| 91300 | 共栄タンカー | 1,532 | 1,410 | -8.0% |
| 331A0 | メディックス | 480 | 482 | +0.4% |
| 43890 | プロパティデータバンク | 770 | 863 | **+12.1% (上昇)** |
| 92470 | TRE ホールディングス | 1,621 | 1,612 | -0.6% |
| 43170 | レイ | 507 | 508 | +0.2% |

### 重要観察

1. **5729 日本精鉱が -18.5% 急落** — caution で見送り推奨だったので影響軽微。
   ただし stale price だと察知できなかった重大変動。**真の解決の意義**。
2. **137A0 Cocolive が close=0 → 739** — 前 wave で risk_above 除外、now entry
   候補化。AFTER-R1 で rank 3 へ。
3. **4389 プロパティデータ +12.1% 上昇** — 急騰、entry 候補 #6 だったが
   stale ベース判断のリスク大。
4. **340A0 ジグザグ -4.5%** — entry 候補 #1 維持、risk 計算が緩やかに減少
   (= 1,990 → 1,900 円)。

## §4 downstream chain (refresh 後 D10)

### 4.1 F111-real-batch chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-05-27 --max-candidates 20 \
    --label-threshold-mode strict \
    --recently-seen-codes 8747,5729,3489 \
    --output-json /tmp/fire_d10_prep/d10_f111_real_batch.json
```

- eligible_count: **18 → 19** (+1 = 137A0 が close=0 → 739 で risk_within=True に)
- exclusions risk_above_pilot_limit: **1 → 0** (= 137A0 が除外解除)
- tuning_params: strict / recently_seen 3 / demoted 3

### 4.2 D10 top 12 (refresh 後)

| rank | code | name | close (actual) | score | label | risk_yen | 変化 |
|---|---|---|---|---|---|---|---|
| 1 | 8747 | 豊トラスティ証券 | 2,420 | 0.930 | caution | 12,100 | (demoted) |
| 2 | 5729 | 日本精鉱 | 1,795 | 0.915 | caution | 8,975 | (demoted、急落判明) |
| 3 | 3489 | フェイスネットワーク | 699 | 0.908 | caution | 3,495 | (demoted) |
| **4** | **340A0** | **ジグザグ** | **380** | 0.892 | **boost_with_caution** | **1,900** | **entry #1** |
| 5 | 3798 | ＵＬＳグループ | 505 | 0.877 | boost_with_caution | 2,525 | entry #2 |
| **6** | **137A0** | **Cocolive** | **739** | 0.869 | **boost_with_caution** | **3,695** | **新 entry 化** |
| 7 | 7991 | マミヤ・オーピー | 1,177 | 0.867 | boost_with_caution | 5,885 | entry #3 |
| 8 | 9130 | 共栄タンカー | 1,410 | 0.859 | boost_with_caution | 7,050 | entry #4 |
| 9 | 331A0 | メディックス | 482 | 0.857 | boost_with_caution | 2,410 | entry #5 |
| 10 | 4389 | プロパティデータバンク | 863 | 0.856 | boost_with_caution | 4,315 | entry #6 (急騰) |
| 11 | 9247 | TRE ホールディングス | 1,612 | 0.855 | boost_with_caution | 8,060 | entry #7 |
| 12 | 4317 | レイ | 508 | 0.852 | boost_with_caution | 2,540 | entry #8 |

### 4.3 AFTER-R1 morning_line_material (= F062 → AFTER-R1 chain)

```
artifact_source      : f062_preview ✓
f062_raw_kind        : f062_actual_dict ✓
f111_input_source    : f111_real_batch ✓
freshness_verdict    : MISSING (= DATA-R3 gate 未渡し、本 wave 意図的)
auto_order_allowed   : False ✓
manual_review_required: True ✓
forbidden_check.passed: True ✓
safety_flags 13 keys all-false: True ✓
exclusions_summary   : {sample: 0, tradable: 0, risk_above: 0}
```

**top_candidates (= refresh 後)**:

| rank | ticker | name | label | score (F062 経由) |
|---|---|---|---|---|
| 1 | 340A0 | ジグザグ | 条件付き買い推奨 | 175.22 |
| 2 | 3798 | ＵＬＳグループ | 条件付き買い推奨 | 172.65 |
| **3** | **137A0** | **Ｃｏｃｏｌｉｖｅ** | **条件付き買い推奨** | **170.85 (= 新 entry)** |

→ refresh 前 (D9) は rank 3 = 331A0 メディックスだったが、refresh で 137A0 が
復活して rank 3 に昇格。

## §5 9 hard invariants (refresh 後)

| invariant | D10 result |
|---|---|
| artifact_source | f062_preview ✓ |
| f062_raw_kind | f062_actual_dict ✓ |
| f111_input_source | f111_real_batch ✓ |
| freshness_verdict | MISSING (= DATA-R3 gate 未渡し、次 wave で解消候補) |
| forbidden_check.passed | True ✓ |
| auto_order_allowed | False ✓ |
| manual_review_required | True ✓ |
| safety_flags 13 keys all False | True ✓ |
| top_candidates count | 3 ✓ (340A0/3798/137A0) |

→ 8/9 PASS (freshness_verdict のみ意図的に MISSING)。

## §6 Codex 4 lane factual-confirm

| lane | 観点 | reply | 一致 |
|---|---|---|---|
| **A** | runner args (= --db-path required, --db-label staging default, WRITE_ENABLED prices+index, DEFAULT_MAX_DAYS=3, HARD=14, dry-run default, no LINE/order/broker) | **YES** ✓ |
| **B** | 4-guard (db_label staging + path exists+non-symlink + resolved.name fire.staging.db + FIRE_ENV staging、production/develop refuse) | **YES** ✓ |
| **C** | V2 auth (= JQUANTS_API_KEY env + x-api-key header、refresh/id-token 不要) | **YES** ✓ |
| **D** | risk recalc impact (= estimate_100_share_risk_yen が ORDER BY date DESC LIMIT 1 で最新 close を読む、filter 不変) | **YES** ✓ |

→ 全 lane が事実確認。実装に問題なし、設計通り。

## §7 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | a8663a07... → **変化** ✓ (= 48 row insert、想定通り) |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| token 値表示 | **0 回** (= 存在のみ len で確認) |
| env 全体表示 | **0 回** (= 個別 key の存在のみ確認) |
| LINE / launchctl / plist / cron / VACUUM / workflow | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| API call | **J-Quants V2 daily_quotes のみ** (= 必要範囲内) |
| git tracked file | 変更 0 (= vault docs 追加のみ) |

## §8 D10 への影響 (= 価格鮮度状態)

- D10 base_date = **2026-05-27 (水)** 想定
- market_prices_daily latest = **2026-05-14** (= D10 当日前 → 約 13 営業日 gap、要再 refresh)
- ただし本 wave は **12 銘柄限定** での smoke。残り 4,400+ 銘柄は依然 2026-05-08 cap。
- D10 直前に「12 銘柄限定再 refresh」または「全銘柄 daily refresh」が必要 (= 次 wave)

## §9 short-term limits + Next action 候補 (= 優先順)

### 9.1 本 wave で達成
- ✓ J-Quants V2 daily refresh 経路確立 (= classification C 真の解決)
- ✓ 12 銘柄 staging-only write smoke 成功
- ✓ production/develop 不変、token 値非表示、9 invariants 8/9 PASS
- ✓ D9 entry candidates の真の price + risk recalc
- ✓ 137A0 復活 (= 0 → 739 で entry 候補化)
- ✓ 5729 急落察知 (= 真の解決の意義実証)

### 9.2 残課題 (= 次 wave 候補)

1. **D10 直前 daily refresh の自動化** (= W60-jquants-daily-refresh-launchd?)
   - 朝 7:30 JST 自動実行、staging market_prices_daily を毎営業日 update
   - launchd plist 必要 (= 別 HQ 承認 + plist 配置承認)
2. **全銘柄 (= 4,448) daily refresh** (= 1 invocation で全 catch-up)
   - 6 営業日 × 4,448 = 26,688 row、HARD_MAX_DAYS=14 + HARD_MAX_SYMBOLS=4500 内
   - rate limit + token quota 検討要
3. **AFTER-R1 DATA-R3 freshness gate JSON 連携** (= freshness_verdict MISSING → OK)
4. **research_derived_indicators rerun** (= 新 price → 新 derived → 新 ranking)
   - persist_derived_indicators full_eligible --hq-approved 経路 (= 別 HQ marker)
5. **research_watchlist_signals rerun** (= 内容更新)
6. **W60-pilot-D10 plan + review template** (= 朝寄付き再 refresh 前提)

## §10 next session 手順 (= D10 朝寄付き)

```bash
# D10 = 2026-05-27 水曜 08:30 JST 想定
# 1. token load
set -a; source /Users/bluefire/fire/.env; set +a

# 2. 12 銘柄再 refresh (= 最新化)
FIRE_ENV=staging \
HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 \
HQ_APPROVE_JQUANTS_TOKEN_READ=1 \
.venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
  --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
  --datasets prices --max-days 3 \
  --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
  --write --output-json /tmp/d10_jq_refresh.json

# 3. D10 chain 再 run
.venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
  --base-date 2026-05-27 --max-candidates 20 \
  --label-threshold-mode strict \
  --recently-seen-codes 8747,5729,3489 \
  --output-json /tmp/d10_f111_real_batch.json
# ↓ F062 → AFTER-R1 chain
```
