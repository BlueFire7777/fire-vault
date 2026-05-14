---
id: FIRE-CODEX-R1-WAVE60-FEATURES-RERUN-PRE-results
phase: 本番 v0 中核 / Wave 60-features-rerun-pre / 候補更新力強化 設計 wave
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
hq_marker: なし (= 本 wave は設計のみ、staging write 0 / API 0)
---

# Wave 60-features-rerun-pre Results — Research Features / Signal Rerun Approval Design

## §1 結論

**features cap の真因 = market_prices_daily が 2026-05-08 で止まっている** (= 6 営業日欠落)。
真の日次候補更新には **J-Quants daily refresh による market_prices 更新** が必要であり、
これは **API call + token 参照** を必須とする (= **classification C**)。

staging 内のみで完結する rerun 経路 (= persist_derived_indicators の --hq-approved
full_eligible) は **token 不要** だが、入力 market_prices が cap されているため
**内容は同じ rank** を出す (= **classification B、効果限定**)。

短期で候補多様化を狙うなら **F111-real-batch 側の preset / label mapping / risk filter
調整** (= classification A、code 変更のみ、API/DB 不要) が即効性高い。

## §2 重複原因分類 (A〜G)

| 候補原因 | 該当 | 根拠 |
|---|---|---|
| **A. research_watchlist_signals 自体が古い** | △ (= 部分) | latest base_date=2026-05-13 は W60-F111-daily-refresh で新規追加、ただし score = 2026-05-12 と完全同一 (1e-12 一致) |
| **B. input features が古い** | **◎ (= 真因)** | research_derived_indicators latest base_date=2026-05-08 (partial 42 row)、full coverage は 2026-05-01 (3708 row) |
| **C. universe が固定** | × | market_listings 4449 row、scale_categories=all_top で 3000+ tradable_universe ✓ |
| **D. ranking cap で同候補上位固定** | × | post_cap_rank 1-30 全て A1、sector_cap 機能はしているが top の sector 構成は dynamic 想定 |
| **E. F111-real-batch preset 制約** | △ | --scale-categories all_top default、これ自体は問題なし、ただし top 3 が同候補なのは input 律速 |
| **F. label mapping が硬い** | △ | selected+A1+score≥0.80 → boost、現状 top 30 全 A1 → 全 boost、label の差別化なし |
| **G. risk filter で残少** | × | eligible=9/10 (= 1 risk_above 137A0)、十分 |

→ **真因 = B (features cap = market_prices_daily 2026-05-08 stale)**。
   表面的因子は A/F が補強。

## §3 research_watchlist_signals upstream source map

```
[外部] J-Quants API (refresh_token → id_token → /daily_quotes / /financials)
   │ ← classification C: scripts/jobs/run_jquants_daily_refresh.py (--write 時 API + staging upsert、default dry-run)
   │ ← classification C: scripts/jobs/fetch_historical_market_data.py (requests + JQUANTS_REFRESH_TOKEN)
   ▼
[staging.db] market_prices_daily (latest=2026-05-08、4448 row)
[staging.db] market_financials_v2 (164,678 row、決算 disclosure、static)
   │
   │ ← classification A: scripts/jobs/compute_derived_indicators.py (純 SQL compute、API 0、DB write 0)
   │ ← classification B: scripts/jobs/persist_derived_indicators.py
   │     args: --smoke-type {five, partial, full_eligible}
   │           --preflight (= A: API/DB write 0、stats のみ)
   │           --hq-approved (= full_eligible 用 HQ flag、R2-A3)
   │           --json-out / --db-path
   │     guard: FIRE_ENV='staging' 必須 (--preflight 以外で)
   ▼
[staging.db] research_derived_indicators (latest base_date=2026-05-08 partial 42 row / 2026-05-01 full 3708)
   │     schema: code/base_date/disclosure_date/type_of_document/fiscal_year_end/
   │             close_price/per/pbr/roe/operating_margin/net_margin/
   │             sales_growth_yoy/profit_growth_yoy/skip_reasons_json/payload_json
   │
   │ ← classification A: scripts/jobs/run_research_watchlist_ranker.py
   │     args: --top-n / --cap-ratio / --output / --db-path
   │     deps: stdlib + simulation.research_lane.* (= API 0)
   │ ← classification B: scripts/jobs/run_research_watchlist_signal_persistence.py
   │     guard: FIRE_ENV='staging' + fire.staging.db basename + 4 段 write guard
   │     既使用 HQ marker: HQ_APPROVE_STAGING_RESEARCH_SIGNAL_REFRESH (W60-F111-daily-refresh)
   ▼
[staging.db] research_watchlist_signals (latest base_date=2026-05-13 w60_f111_daily_v1 35 row、ただし 2026-05-12 完全同 rank)
   │
   │ ← classification A: scripts/jobs/run_f111_real_batch_staging.py (read-only、URI mode=ro)
   ▼
/tmp/.../f111_real_batch_*.json (= F062 preview 互換 format)
   │
   │ ← classification A: scripts/jobs/_after_r1_mvp.py (read-only chain、9 hard invariants 付与)
   ▼
reports/after_r1/morning_line_material_*.json (= D6/D7/D8 top_candidates)
```

## §4 feature freshness (staging read-only 確認)

| table | latest | rows | gap to 2026-05-14 | 評価 |
|---|---|---|---|---|
| **market_prices_daily** | 2026-05-08 | 4448 | **6 営業日** | **stale (= 真因)** |
| market_financials_v2 | (disclosure 経由、static) | 164,678 | static | OK (= 決算依存) |
| research_derived_indicators (full) | 2026-05-01 | 3708 | **9 営業日** | **stale (= 派生)** |
| research_derived_indicators (partial latest) | 2026-05-08 | 42 | 6 営業日 | partial (= 真因継承) |
| research_watchlist_signals | 2026-05-13 (w60_f111_daily_v1 35) | 13,730 | 1 日 | 形式 OK / 内容 stale (= 派生) |

market_prices_daily が cap source。これより上流の更新には J-Quants API call 必須。

## §5 rerun runner 一覧 + 安全引数

| runner | API/token | DB write | safety args | classification |
|---|---|---|---|---|
| `audit_jquants_freshness.py` | 0 | 0 | "no API call / no token" docstring 明記 | **A** |
| `compute_derived_indicators.py` | 0 | 0 (= --output 経由で JSON) | --base-date / --output-json / --db-path | **A** |
| `persist_derived_indicators.py` --preflight | 0 | 0 | --preflight (= stats only) | **A** |
| `persist_derived_indicators.py` --smoke-type five | 0 | 5 row staging | FIRE_ENV=staging | **B** |
| `persist_derived_indicators.py` --smoke-type full_eligible --hq-approved | 0 | full staging | FIRE_ENV=staging + --hq-approved | **B (HQ flag 必要)** |
| `run_research_watchlist_ranker.py` | 0 | 0 (= --output JSON) | --top-n / --cap-ratio / --output / --db-path | **A** |
| `run_research_watchlist_signal_persistence.py` (--dry-run / 既 default) | 0 | 0 | 4 段 guard | **A** |
| `run_research_watchlist_signal_persistence.py` (--write) | 0 | 35 row staging | FIRE_ENV=staging + 4 段 guard | **B (既 HQ marker)** |
| `run_jquants_daily_refresh.py` (default dry-run) | 0 | 0 | "default dry-run / no API call" docstring 明記 | **A** |
| `run_jquants_daily_refresh.py` --write | **必要** | staging | --db-label staging / --write 必須 / refresh token 必要 | **C** |
| `fetch_historical_market_data.py` | **必要** | DB write | JQUANTS_REFRESH_TOKEN / JQUANTS_USER+PASSWORD / JQUANTS_API_KEY 必要 | **C** |
| `backfill_features.py` (旧 F021-F026 features 系) | 0 | features 旧 schema write | scripts.jobs.extract_features 経由 | B (不関与) |

## §6 rerun 経路 A〜E 分類

| 経路 | 内容 | 分類 | 効果 | HQ marker |
|---|---|---|---|---|
| **a. F111-real-batch preset/label/risk 調整** | code 変更のみ、ranking 本体は staging から read | **A** | 短期、top 3 同候補は変わらないが 4-N で多様化 | 不要 (= code 変更) |
| **b. compute_derived_indicators --output JSON** | 純 read-only artifact 生成、新 base_date で indicator 計算 | **A** | 0 (= rerun 再現性確認のみ) | 不要 |
| **c. persist_derived_indicators --preflight** | stats 確認、DB write 0 | **A** | 確認のみ | 不要 |
| **d. persist_derived_indicators --smoke-type five** | 5 銘柄分 staging write | **B** | smoke のみ、内容律速 | (新規) HQ_APPROVE_STAGING_FEATURES_RERUN |
| **e. persist_derived_indicators --smoke-type full_eligible --hq-approved** | full staging write | **B** | research_derived_indicators の新 base_date 行追加、ただし market_prices cap 律速で rank 変化僅か | (新規) HQ_APPROVE_STAGING_FEATURES_RERUN + コード内 --hq-approved flag |
| **f. run_jquants_daily_refresh.py (dry-run)** | API call 0、stats のみ | **A** | 確認のみ | 不要 |
| **g. run_jquants_daily_refresh.py --write** | J-Quants API call + staging write、market_prices_daily 更新 | **C** | **真の解決**、新 price → 新 derived → 新 signal → 新 candidate | (新規) HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH + HQ_APPROVE_JQUANTS_TOKEN_READ |
| **h. fetch_historical_market_data.py** | バッチ historical fetch、token 必要 | **C** | 同上、ただし日次より広範 (= 1 ヶ月単位) | (新規) HQ_APPROVE_MARKET_DATA_API_READ + HQ_APPROVE_JQUANTS_TOKEN_READ |
| **i. schema migration** | 不要 (= 現 schema 十分) | **D** | - | (将来) HQ_APPROVE_FEATURES_SCHEMA_MIGRATION |

→ **本 wave は a〜f までを設計のみ把握** (= write 0 / API 0)。g〜h は次 wave 用 HQ approve 経由。

## §7 必要 HQ marker 案 (= 次 Wave 用)

| marker | 用途 | 既存 |
|---|---|---|
| `HQ_APPROVE_STAGING_RESEARCH_SIGNAL_REFRESH` | run_research_watchlist_signal_persistence --write | **既使用** (W60-F111-daily-refresh) |
| `HQ_APPROVE_STAGING_FEATURES_RERUN` | persist_derived_indicators --hq-approved + staging write | 新規、推奨 |
| `HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH` | run_jquants_daily_refresh.py --write (J-Quants API call + staging write) | 新規、推奨 |
| `HQ_APPROVE_JQUANTS_TOKEN_READ` | JQUANTS_REFRESH_TOKEN / JQUANTS_API_KEY env 参照許可 | 新規、必須 (C 経路前提) |
| `HQ_APPROVE_MARKET_DATA_API_READ` | fetch_historical_market_data.py 等で J-Quants API call 許可 | 新規、optional |
| `HQ_APPROVE_FEATURES_SCHEMA_MIGRATION` | scripts/setup/migrate_*.py | 不要 (= 現 schema 十分) |

各 marker は **environment 変数 = 1 + そのまま .env で永続化しない** (= ad-hoc 承認 only) ように運用想定。

## §8 D8 / D9 候補更新方針

| option | 内容 | 必要承認 | 効果 | 推奨度 |
|---|---|---|---|---|
| **D8 HOLD 継続** | D6 review 未完了 + features cap、当面 pilot pause | 不要 | 安全、ただし pilot ペース低下 | ★★ |
| **D9: 経路 a (F111 preset/label 調整)** | code 変更のみ、`--scale-categories mid400` / label 閾値緩和 / risk filter 緩和 | 不要 (= code review のみ) | top 3 同候補は変わらず、4-20 で発掘 | **★★★** |
| **D9: 経路 e (persist_derived full_eligible)** | features 新 base_date 追加、内容律速で rank 変化僅か | HQ_APPROVE_STAGING_FEATURES_RERUN | 形式 OK、効果限定 | ★ |
| **D9: 経路 g (jquants daily refresh)** | 真の price 更新 → 真の derived 更新 → 真の rank 更新 | HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH + HQ_APPROVE_JQUANTS_TOKEN_READ | **真の解決** | **★★★** (= 中期推奨) |
| **D9: 経路 h (historical batch fetch)** | g より広範、1 ヶ月単位の price + financials catch-up | HQ_APPROVE_MARKET_DATA_API_READ + HQ_APPROVE_JQUANTS_TOKEN_READ | 6 営業日 backfill | ★★ |
| **D9: 経路 g + a 併用 (推奨)** | jquants refresh → 新 derived → 新 signal → F111 preset/label も同時調整 | 全 marker 必要 | 最大効果 | **★★★ (中期 1 番手)** |

### 推奨 next wave sequence

```
Wave 60-F111-preset-tune (= A 分類のみ、推奨優先 #1)
   ├─ F111-real-batch --scale-categories 検証 (= read-only sweep)
   ├─ label mapping 閾値緩和 (= boost = score≥0.85 等に厳しく、deferred 増)
   ├─ risk filter 緩和検討 (= 100 株 risk_limit を 20000 円等で発掘)
   └─ max_candidates 10 → 20 / 30 で top 3 以外の候補出力
   → DB/API/token 0、code 変更のみ、効果は短期

Wave 60-features-rerun-staging (= B 分類、推奨優先 #2、中効果)
   ├─ HQ_APPROVE_STAGING_FEATURES_RERUN marker
   ├─ persist_derived_indicators --preflight (= 確認、A 内)
   ├─ persist_derived_indicators --smoke-type five (= 5 銘柄試行)
   ├─ persist_derived_indicators --smoke-type full_eligible --hq-approved (= full staging)
   ├─ run_research_watchlist_signal_persistence --write (= HQ_APPROVE_STAGING_RESEARCH_SIGNAL_REFRESH=1)
   └─ D9 chain 再 run
   → API 0、staging write、内容律速で rank 変化僅か想定

Wave 60-jquants-daily-refresh-staging (= C 分類、推奨優先 #3、最大効果、最大慎重)
   ├─ HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH + HQ_APPROVE_JQUANTS_TOKEN_READ
   ├─ run_jquants_daily_refresh.py --dry-run (= A 内、確認)
   ├─ token availability 確認 (= .env / env 参照、別 HQ approve 必要)
   ├─ run_jquants_daily_refresh.py --datasets prices --max-days 6 --write (= staging price 6 日 catch-up)
   ├─ persist_derived_indicators full_eligible --hq-approved (= 新 price → 新 derived)
   ├─ signal_persistence --write (= 新 signal)
   └─ D9 chain で新 candidate 確認
   → **真の日次更新解決**、ただし HQ marker 2 個 + token 参照 = 大きな承認境界
```

## §9 Codex 4 lane factual-confirm audit 結果

| lane | prompt 観点 | reply | 一致 |
|---|---|---|---|
| A | input source: persistence runner → ranker → research_derived_indicators、API import 0 | **YES** | ✓ |
| B | freshness header (line 3) + FIRE_ENV='staging' guard | **NO** — persist_derived_indicators には `--write` flag なし、`--preflight` pattern、FIRE_ENV='staging' は非 preflight write path で必須 | ✓ (= 私の解析を Lane B が訂正、args が --smoke-type / --preflight / --hq-approved である事実を確認) |
| C | run_jquants_daily_refresh default dry-run、API call 0 | **YES** — "dry-run does not import/call J-Quants fetchers or write DB; read-only DB planning only" | ✓ |
| D | HQ marker 4 種が repo 内未定義 | **NO** — fire/ Python/doc 内 grep 0 hit (= 既存 HQ_APPROVE_STAGING_RESEARCH_SIGNAL_REFRESH も fire/ コード内には 0、fire-vault 内のみ 2 hit) | ✓ (= "code 内に既存 marker 0" を確認) |

→ Codex 4 lane factual-confirm 完走、Lane B の指摘で `--preflight` pattern を把握 ✓、Lane D で「marker は HQ approve text の env 変数として運用 (= code 内 hardcode しない)」を確認。

## §10 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓ |
| staging md5 (本 wave 開始時) | a8663a07a730378387c050ebb1b612ef |
| staging md5 (本 wave 終了時) | (read-only 確認のみ、本 wave で write 0、後で再 md5 verify) |
| F282 plist size/mtime | 1751 / 1778602507 → 不変 ✓ |
| pytest 4705 collected | 不変 ✓ |
| LINE 0 / token 0 / API 0 / launchctl 0 / plist 0 / cron 0 ✓ |
| 実発注 0 / 楽天 0 / iSPEED 0 / Computer Use 0 ✓ |
| git tracked file 変更 0 (= vault doc 2 個追加のみ) ✓ |

## §11 Next action 候補 (= 優先順)

1. **W60-F111-preset-tune** (= 経路 a、code 変更のみ、DB/API 0、推奨 #1)
2. **D6 review 記入待ち** (Fujiwara 本人) → D7/D8 trade plan の review-aware 化
3. **W60-features-rerun-staging** (= 経路 e、HQ_APPROVE_STAGING_FEATURES_RERUN 承認後)
4. **W60-jquants-daily-refresh-staging** (= 経路 g、HQ marker 2 個 + token 承認後、**真の解決**)
5. **W60-pilot-D8 plan** + review template
