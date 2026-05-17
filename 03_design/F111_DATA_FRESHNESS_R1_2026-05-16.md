# FIRE F111 DATA FRESHNESS R1 audit (2026-05-16)

doc_id: FIRE-F111-DATA-FRESHNESS-R1-2026-05-16
status: read-only audit 完了 (= DB write 0 / API 0 / token 0 / production code 変更 0)
related:
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2B_momentum_sector_2026-05-16.md (= W2-B 実装、as_of_date 2026-05-14 の根拠調査)
- F286-DATA-R0 audit / R1 daily refresh / R3 cron skeleton (= 既存 refresh infrastructure)


## §1 監査背景

W2-B momentum / sector_relative_strength sim 実行時に as_of_date が **2026-05-14** となり、
candidate base_date 2026-07-22 と **約 70 calendar days のギャップ**が発生。

本 wave では:
- 鮮度ギャップの原因究明 (= read-only audit)
- F111 / W2-B / Universe Expansion 入力経路の調査
- 既存 DATA-R3 / J-Quants refresh runner の存在確認
- W2-B 再実行可否判断
- W2-C / Universe W4 / data refresh の優先順位提案

★ DB write 0 / staging write 0 / API 0 / token 参照 0 / launchctl / cron 0。


## §2 git 状態

- branch = develop (= 監査開始時 main にいたため checkout 完了、working tree clean)
- ahead/behind = 0/0
- 本 wave 中の git add / commit / push = 0


## §3 DB 鮮度監査 (staging URI mode=ro)

| table | latest | 件数 | 候補 base_date 2026-07-22 との差 | 今日 2026-05-16 との差 |
|---|---|---|---|---|
| **market_prices_daily** | **2026-05-14** | 2,085,332 行 / 4,452 codes / 495 distinct dates | **+69 calendar days** (signal forward-dated) | 2 calendar days |
| research_watchlist_signals | 2026-07-22 | 13,839 / latest 109 件 | 0 (= signal 側 forward) | +67 days (synthetic future) |
| market_listings | 2026-05-01 (fetched_at) | 4,449 / fetched_at 1 種 | n/a (静的 snapshot) | 15 days |
| market_financials_v2 | (disclosure_date based) | 164,678 | 該当なし | 該当なし |
| research_derived_indicators | (base_date based) | 3,750 | n/a | n/a |
| announcements | (announced_date based) | 1,098 | n/a | n/a |
| features | (dt based) | 1,131,331 / 504 symbols | n/a | n/a |

**W2-B 対象 9 codes の market_prices_daily 最新日**:
- 9247: 2026-05-14
- 残り 8 件 (9628, 4404, 9008, 3134, 7595, 9417, 3962, 6196): **2026-05-08** (= さらに 4 営業日 stale)
- **2026-05-15 以降の行 = 0 件**

→ staging の price data は実質 2026-05-08〜2026-05-14 で停止、signal はそれ以降の synthetic future date


## §4 F111 入力経路

### §4.1 F111 staging runner (= `scripts.jobs.run_f111_real_batch_staging`)

- `--base-date` default = `datetime.now(JST).date()` (= 今日)
- 但し signal lookup は `MAX(base_date) FROM research_watchlist_signals` (= 自動的に 2026-07-22)
- candidate listing: `INNER JOIN market_listings ML ON ML.code = WS.code`
- 各 ticker の latest close: `SELECT close FROM market_prices_daily WHERE code=? ORDER BY date DESC LIMIT 1`

**output base_date と内部 source date は分離**:
- output base_date = `--base-date` (= 今日 / 引数)
- signal source date = 2026-07-22 (forward synthetic)
- price source date = 2026-05-14 (= 約 2 日 stale)

### §4.2 W2-B sim (= `scripts.jobs.run_f111_overlay_momentum_sector_sim`)

- `--as-of-date` default = `_latest_date(conn)` = `MAX(date) FROM market_prices_daily`
- staging では **2026-05-14 が自動選択**
- 5-15 以降の price データが 0 件のため、これが唯一の選択肢

### §4.3 signal persistence (= `scripts.jobs.run_research_watchlist_signal_persistence`)

- `--base-date` 明示指定で signal 再生成
- 価格 / financials を入力に signal を組み立てる
- 更新には staging write 必須、本 audit 範囲外


## §5 W2-B が 2026-05-14 を使った根本理由

W2-B runner は技術的に正しい挙動:
1. `--as-of-date` 引数なし → `_latest_date(conn)` 呼び出し
2. `SELECT MAX(date) FROM market_prices_daily` → 2026-05-14 を返却
3. 2026-05-14 を起点に過去 20 営業日の price / volume series を取得
4. これが唯一利用可能な「最新データ」だった

→ **staging に 2026-05-15 以降の price data が存在しない**という単純な事実が根本原因。
   W2-B sim 実装の問題ではなく、staging DB の refresh が滞っていることが問題。


## §6 DATA-R3 / J-Quants refresh 経路

### §6.1 既存 runner (= 利用可能)

| runner | 用途 | 状態 |
|---|---|---|
| `audit_jquants_freshness.py` | read-only audit | 利用可能 (= 本 audit と同等の処理) |
| `run_jquants_daily_refresh.py` | DATA-R1 daily refresh, --dry-run / --write | 利用可能 / --write は `FIRE_ENV=staging` + token 必須 |
| `run_f286_data_r3_daily_refresh.py` | DATA-R3 cron skeleton | 半実装 (= write 未実装、sub-D2.2 別 approve 待ち) |
| `fetch_historical_market_data.py` | F100 過去データ取得 | 利用可能 / J-Quants token 必須 |
| `backfill_market_financials.py` | 財務 backfill | 利用可能 |

### §6.2 staging-only 安全 guard (= run_jquants_daily_refresh)

3 段 guard:
1. `--db-label staging` 明示必須
2. `--db-path` basename `fire.staging.db` 一致必須
3. `FIRE_ENV=staging` 環境変数必須

3 つ全て揃わないと write refuse (= production / develop 誤書込み構造的防止)。


## §7 更新が必要な table

| 優先 | table | 更新方法 | HQ approve scope |
|---|---|---|---|
| ★★★ | market_prices_daily | `run_jquants_daily_refresh --datasets prices --write --max-days 3` | API + staging write |
| ★★ | market_listings | `run_jquants_daily_refresh --datasets listings --write` | API + staging write |
| ★★ | research_watchlist_signals | `run_research_watchlist_signal_persistence --base-date <today>` | staging write + signal logic 確認 |
| ★ | research_derived_indicators | research lane runner | staging write |
| - | features / financials | 既存 backfill / extract runner | 必要時 |


## §8 必要 HQ approve

| 案 | HQ marker (推奨) | scope |
|---|---|---|
| A. staging refresh | `HQ_APPROVE_DATA_R1_STAGING_REFRESH_PRICES_LISTINGS` | run_jquants_daily_refresh --write (= API + staging write + token + FIRE_ENV=staging) |
| B. signal regeneration | `HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH` | run_research_watchlist_signal_persistence --base-date today (要 案 A 完了後) |
| C. W2-B 再実行 | `HQ_APPROVE_W2B_RE_RUN_FRESH_DATA` | run_f111_overlay_momentum_sector_sim --as-of-date today (= 案 A+B 完了後) |
| D. DATA-R3 cron 本登録 | `HQ_APPROVE_DATA_R3_CRON_INSTALL` | sub-D2.2 write 実装 + sub-D3 cron 登録 + launchd plist (長期、別 wave 群) |


## §9 W2-B 再実行可否

**結論**: 条件付き使用可、**朝判断の絶対基準にはしない**

- 使ってよい: 大局 trend (= vol_mom_20d、leader_inertia の極端例 = 9417, 3134) は 2 営業日 stale でも有効
- 使わない: 細かい boundary 判定、特に「9247 主役継続 vs 惰性固定」の判別は fresh data 待ち推奨
- 9130 復帰禁止 confirm は静的、freshness 影響なし
- **fresh data 取得後の再実行** (= 案 A → B → C) が朝判断の正規根拠を整える唯一の道


## §10 次 wave 推奨

優先順位:
1. **案 A** (= staging prices refresh) — `HQ_APPROVE_DATA_R1_STAGING_REFRESH_PRICES_LISTINGS`
   最も small scope、最も高効用 (= W2-B label の鮮度を直接回復)
2. **案 B** (= signal regen) — `HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH`
   案 A 完了後、current date base の signal を生成
3. **案 C** (= W2-B 再実行) — `HQ_APPROVE_W2B_RE_RUN_FRESH_DATA`
   案 A+B 完了後、9247/9628/3134 等の label を fresh data で再評価
4. **W2-C 着手** (= overlay 選定 logic 改修) は 案 C 完了 + 数日の安定確認後
5. **案 D** (= DATA-R3 cron) は最後、長期運用化


## §11 Codex 4 lane 監査結果

(別途 §11.1〜§11.4 に追記される)


## §12 安全 gate (本 wave 行為)

| 項目 | 結果 |
|---|---|
| DB write (production / develop / staging) | 0 |
| schema migration | 0 |
| LINE 送信 | 0 |
| API call | 0 |
| token / channel / secret / .env / os.environ 参照 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push | 0 |
| PR 作成 / merge | 0 |
| VACUUM / sudo / rm -rf | 0 |
| --no-verify | 0 |
| auto-order / 楽天 / iSPEED / Computer Use | 0 |
| TODO Excel 更新 | 0 |
| production code 変更 | 0 |
| 出力先 | /tmp/fire_f111_data_freshness_r1/ + ~/fire-vault/03_design/ (= 安全 prefix 限定) |


## §13 関連 file

```
新規 (= 本 wave 出力):
  /tmp/fire_f111_data_freshness_r1/data_freshness_tables.json
  /tmp/fire_f111_data_freshness_r1/f111_input_freshness_audit.md
  /tmp/fire_f111_data_freshness_r1/refresh_path_candidates.md
  /tmp/fire_f111_data_freshness_r1/w2b_reexecution_readiness.md
  ~/fire-vault/03_design/F111_DATA_FRESHNESS_R1_2026-05-16.md (= 本 doc)

既存 (= 監査参照):
  scripts/jobs/run_f111_real_batch_staging.py (= F111 staging runner)
  scripts/jobs/run_f111_overlay_momentum_sector_sim.py (= W2-B sim)
  scripts/jobs/run_research_watchlist_signal_persistence.py (= signal persist)
  scripts/jobs/audit_jquants_freshness.py (= DATA-R0)
  scripts/jobs/run_jquants_daily_refresh.py (= DATA-R1)
  scripts/jobs/run_f286_data_r3_daily_refresh.py (= DATA-R3 cron skeleton)
  scripts/jobs/fetch_historical_market_data.py (= F100)
  agents/jquants_daily_refresh.py (= DATA-R1 agent layer)
```
