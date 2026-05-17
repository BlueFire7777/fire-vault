# FIRE DATA-R1 staging prices+index refresh (2026-05-16 起票 / 2026-05-17 JST 実行)

doc_id: FIRE-DATA-R1-STAGING-REFRESH-PRICES-INDEX-2026-05-16
status: 実行成功 (= prices 4,443 行 + index 1 行 staging upsert、production/develop 不変、HQ 承認下で 1 回のみ実行)
related:
- F111_DATA_FRESHNESS_R1_2026-05-16.md (= 鮮度ギャップ監査の起点)
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2B_momentum_sector_2026-05-16.md (= W2-B sim、本 refresh で再実行可)


## §1 目的

W2-B momentum / sector_relative_strength sim が利用する staging market_prices_daily が
2026-05-14 で停止 (= 一部 W2-B 対象銘柄は 2026-05-08 止まり) と F111-DATA-FRESHNESS-R1 で判明。
最小範囲で staging DB の **prices + index のみ** を refresh し、W2-B label の鮮度を回復する。

★ 承認範囲は staging DB の prices + index のみ。listings / signals / W2-B 再実行は別 wave。


## §2 承認範囲

| 項目 | スコープ |
|---|---|
| HQ marker | `HQ_APPROVE_DATA_R1_STAGING_REFRESH_PRICES_INDEX` |
| 対象 DB | `data/fire.staging.db` 限定 (= URI / basename / FIRE_ENV 3 段 guard) |
| 対象 datasets | `prices, index` のみ (= WRITE_ENABLED_DATASETS と一致) |
| 対象期間 | --max-days 3 (= 直近 3 営業日、5-13 火 / 5-14 水 / 5-15 木 を埋める想定) |
| 実行回数 | **1 回のみ** |
| 禁止 | production/develop DB write、listings write、signals write、schema migration、LINE、launchctl/cron、git push、--no-verify |


## §3 実行コマンド

```
FIRE_ENV=staging .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
  --db-path data/fire.staging.db \
  --db-label staging \
  --datasets prices,index \
  --max-days 3 \
  --write \
  --output-json /tmp/fire_data_r1_refresh_prices_index/refresh_result.json \
  --completion-report /tmp/fire_data_r1_refresh_prices_index/completion_report.txt
```

3 段 staging-only guard:
1. `--db-label staging` 明示
2. `--db-path` basename = `fire.staging.db`
3. `FIRE_ENV=staging` 環境変数


## §4 pre-refresh freshness (audit_jquants_freshness.py read-only)

詳細は `/tmp/fire_data_r1_refresh_prices_index/pre_freshness.json` 参照。要約:

| table | latest | rows |
|---|---|---|
| market_prices_daily | 2026-05-14 | 2,085,332 (4,452 codes / 495 dates) |
| index_data | 2026-05-08 | 59 |

W2-B 9 codes: 9247=5-14、残り 8 件=5-08。


## §5 refresh 実行結果

| 項目 | 値 |
|---|---|
| started_at | 2026-05-17 00:05 JST |
| ended_at | 2026-05-17 00:51 JST |
| elapsed | ~46 minutes (= rate limit 429 retry 含む) |
| dataset prices | executed=True / write_invoked=True / inserted=4,443 / status=ok / error=None |
| dataset index | executed=True / write_invoked=True / inserted=1 / status=ok / error=None |
| rows_inserted_total | 4,444 |
| write_invoked_count | 2 |
| error_count | 0 |
| 429 rate limit | 11 件観測、全 retry 成功 |


## §6 post-refresh freshness

詳細は `/tmp/fire_data_r1_refresh_prices_index/post_freshness.json` + `freshness_delta.md` 参照。要約:

| table | pre | post | delta |
|---|---|---|---|
| market_prices_daily latest | 2026-05-14 | **2026-05-15** | +1 trading day |
| market_prices_daily rows | 2,085,332 | 2,089,775 | +4,443 |
| index_data latest | 2026-05-08 | **2026-05-11** | +3 calendar days |
| index_data rows | 59 | 60 | +1 |


## §7 W2-B 対象 9 銘柄 latest 改善

| code | pre | post | delta |
|---|---|---|---|
| 92470 (9247 ＴＲＥ HD) | 2026-05-14 | **2026-05-15** | +1 trading day |
| 96280 (9628 燦 HD) | 2026-05-08 | **2026-05-15** | **+5 trading days** |
| 44040 (4404 ミヨシ油脂) | 2026-05-08 | **2026-05-15** | **+5 trading days** |
| 90080 (9008 京王電鉄) | 2026-05-08 | **2026-05-15** | **+5 trading days** |
| 31340 (3134 Hamee) | 2026-05-08 | **2026-05-15** | **+5 trading days** |
| 75950 (7595 アルゴグラフィックス) | 2026-05-08 | **2026-05-15** | **+5 trading days** |
| 94170 (9417 スマートバリュー) | 2026-05-08 | **2026-05-15** | **+5 trading days** |
| 39620 (3962 チェンジ HD) | 2026-05-08 | **2026-05-15** | **+5 trading days** |
| 61960 (6196 ストライク) | 2026-05-08 | **2026-05-15** | **+5 trading days** |

→ **全 9 銘柄が 2026-05-15 まで揃った** (= W2-B 再実行で fresh momentum / srs label 算出可能)


## §8 DB md5

| DB | pre | post | 変化 |
|---|---|---|---|
| data/fire.db (production) | b1df4673e5c3645fbe2c5f490ffac043 | b1df4673e5c3645fbe2c5f490ffac043 | **不変 ✓** |
| data/fire.develop.db | 0eed4ad2ec7ed2edf8f640d97341c5ad | 0eed4ad2ec7ed2edf8f640d97341c5ad | **不変 ✓** |
| data/fire.staging.db | f6fa86df9c22d3e5866c7d3f380286d5 | **71a63a19694385db4344246be60a2f91** | 変化 (想定) |

staging size: 4,804,222,976 → 4,804,788,224 (= +565,248 bytes、4,444 rows insert と整合)


## §9 F282 plist (全 3 file 不変)

| file | size | mtime | md5 |
|---|---|---|---|
| ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist (deployed) | 1,772 | 5月 12 22:46 | e4ea05e88625040d1f79151948420b4f |
| docs/launchd/jp.fire.weekly-snapshot.plist (source) | 1,772 | 5月 16 21:29 | (source) |
| docs/launchd/jp.fire.weekly-snapshot-smoke.plist (source smoke) | 1,844 | 5月 16 21:29 | (source smoke) |

全 3 file pre/post 完全一致 (= launchctl / plist 編集 0)。


## §10 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| staging DB write (intentional) | ✓ rows_inserted=4,444 / staging md5 変化 |
| production DB write | **0** (md5 不変) |
| develop DB write | **0** (md5 不変) |
| listings write | 0 (本 wave 対象外) |
| signals write | 0 (本 wave 対象外) |
| schema migration | 0 |
| LINE 送信 | 0 |
| token / .env / secret 値表示 | 0 (= 存在確認のみ、値非表示) |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push / PR | 0 |
| VACUUM / sudo / rm -rf / --no-verify | 0 |
| auto-order / 楽天 / iSPEED / Computer Use | 0 |
| 429 rate limit 検出 | 11 件 (全 retry 成功) |


## §11 Codex 4 lane 監査結果

(別途 §11.1-§11.4 に追記される。CRITICAL/HIGH 検出時は §13 結論で REJECT)


## §12 案 B signal regen 可否

**可能 (= fresh prices + listings 揃った段階で OK)**

ただし注意:
- listings は本 wave 対象外 (= 2026-05-01 fetched で 15 日前)
- 案 B (= run_research_watchlist_signal_persistence --base-date today) は fresh listings あった
  ほうが理想だが、listings 比較的低頻度更新で staging signal regen には致命的でない
- 推奨順: 案 B 先行 → fresh signals 確認 → 案 C (W2-B 再実行)


## §13 結論 / 次アクション

### §13.1 結論

- prices 4,443 行 + index 1 行 staging upsert 成功
- production / develop DB 完全不変
- W2-B 対象 9 銘柄 全件 2026-05-15 まで揃った (= W2-B label fresh data 化準備完了)
- F282 plist 全 3 file 不変、launchctl / cron 操作 0
- 安全 gate all 0 (= 想定 staging write 以外)

### §13.2 次 wave 推奨

1. **案 B**: signal regeneration (= `HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH`)
   - `run_research_watchlist_signal_persistence --base-date <today>`
   - fresh signals を current base_date で生成
2. **案 C**: W2-B 再実行 (= `HQ_APPROVE_W2B_RE_RUN_FRESH_DATA`)
   - `run_f111_overlay_momentum_sector_sim --as-of-date <today>`
   - 案 A (= 本 wave) + 案 B 完了後、fresh data で momentum/sector label を再評価
   - 9247/9628/3134/9417 の label が大きく変わる可能性 (= 2-3 ヶ月の時系列差)


## §14 関連 file

```
本 wave 出力 (= /tmp 限定):
  /tmp/fire_data_r1_refresh_prices_index/pre_freshness.json
  /tmp/fire_data_r1_refresh_prices_index/refresh_result.json
  /tmp/fire_data_r1_refresh_prices_index/completion_report.txt
  /tmp/fire_data_r1_refresh_prices_index/post_freshness.json
  /tmp/fire_data_r1_refresh_prices_index/freshness_delta.md
  (+ Codex 4 lane prompts/results)

設計 doc:
  ~/fire-vault/03_design/DATA_R1_STAGING_REFRESH_PRICES_INDEX_2026-05-16.md (本 doc)
```
