# FIRE F111-DERIVED-FRESHNESS-AUDIT-R1 (2026-05-16 起票 / 2026-05-17 JST 実行)

doc_id: FIRE-F111-DERIVED-FRESHNESS-AUDIT-R1-2026-05-16
status: 監査完了 (= read-only、コード変更 0、DB write 0)
related:
- [[F111_DATA_FRESHNESS_R1_2026-05-16]] — 鮮度ギャップ監査の起点 wave
- [[DATA_R1_STAGING_REFRESH_PRICES_INDEX_2026-05-16]] — prices+index refresh 完了 wave
- [[F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2B_momentum_sector_2026-05-16]] — W2-B sim 起点

---

## §1 目的

DATA-R1 で staging `market_prices_daily` を 2026-05-15 まで refresh 成功した。
ただし Codex Lane D が「案 B (signal regen) は `research_derived_indicators × market_listings` を参照、fresh prices だけでは不十分」と HIGH 指摘。

本 audit は **read-only で** 以下を判定する:
- `research_derived_indicators` / `research_watchlist_signals` / `market_listings` の鮮度
- W2-B 9 銘柄の derived/listings 鮮度
- `research_derived_indicators` の生成 runner と入力依存
- 案 B (signal regen) 実行前に必要な前提 wave (= 案 A.5 derived regen 等)
- 案 A.5 / 案 B / 案 C の推奨実行順

★ コード変更 0、DB write 0、git push 0、merge 0、LINE 0、launchctl 0 (= 純監査)。

---

## §2 承認範囲

| 項目 | スコープ |
|---|---|
| HQ marker | (audit のみ、specific HQ marker 不要) |
| 対象 DB | `data/fire.staging.db` 限定 (= URI mode=ro + immutable=1) |
| read scope | research_derived_indicators / research_watchlist_signals / market_listings / market_financials_v2 / market_prices_daily / index_data |
| write scope | **0** (= read-only) |
| 出力 | /tmp/fire_derived_freshness_audit/ 配下 4 file + 本 vault doc |
| 禁止 | DB write、schema migration、git push/merge、LINE、launchctl/cron、production/develop 触、--no-verify |

---

## §3 git 状態 (read-only 確認)

| 項目 | 値 |
|---|---|
| branch | develop |
| working tree | clean |
| ahead / behind origin/develop | 0 / 0 |
| HEAD | 0f26f47 (= test(F111): cover overlay display and letter-suffix regressions) |

---

## §4 staging DB 鮮度監査結果 (詳細: derived_freshness.json)

| table | latest | rows | distinct codes | 鮮度評価 |
|---|---|---|---|---|
| market_prices_daily | **2026-05-15** | 2,089,775 | 4,452 | ✓ fresh (= DATA-R1 post) |
| index_data | 2026-05-11 | 60 | - | △ 6 日 stale |
| market_financials_v2 | 2026-05-08 | 164,678 | 3,787 | △ 9 日 stale |
| **research_derived_indicators** | **2026-05-08 部分** | 3,750 | 3,708 | × **16 日 stale (= 主データ 2026-05-01)** |
| research_watchlist_signals | 2026-07-22 (= W2-A/B 合成) | 13,839 | 910 | (= 31 base_dates、現在 sim 由来) |
| market_listings | 2026-05-01T11:38:33Z | 4,449 | 4,449 | △ 16 日 stale |

### §4.1 research_derived_indicators 詳細

| base_date | rows | distinct codes |
|---|---|---|
| 2026-05-01 | 3,708 | 3,708 (= 主データ、full_eligible 推定) |
| 2026-05-08 | 42 | 42 (= 部分 smoke の追加分) |

→ 2 distinct base_dates のみ、5-08 行は 42 件のみ。

### §4.2 W2-B 9 銘柄の derived / prices 鮮度差

| code | name | prices latest | derived latest | gap |
|---|---|---|---|---|
| 31340 | Ｈａｍｅｅ | 2026-05-15 | 2026-05-01 | 14 営業日 stale |
| 39620 | チェンジホールディングス | 2026-05-15 | 2026-05-01 | 14 営業日 stale |
| 44040 | ミヨシ油脂 | 2026-05-15 | 2026-05-01 | 14 営業日 stale |
| 61960 | ストライクグループ | 2026-05-15 | 2026-05-01 | 14 営業日 stale |
| 75950 | アルゴグラフィックス | 2026-05-15 | 2026-05-01 | 14 営業日 stale |
| 90080 | 京王電鉄 | 2026-05-15 | 2026-05-01 | 14 営業日 stale |
| 92470 | ＴＲＥホールディングス | 2026-05-15 | 2026-05-01 | 14 営業日 stale |
| 94170 | スマートバリュー | 2026-05-15 | 2026-05-01 | 14 営業日 stale |
| 96280 | 燦ホールディングス | 2026-05-15 | 2026-05-01 | 14 営業日 stale |

→ W2-B 9 銘柄全件、prices fresh / derived stale (= 5-08 部分にも含まれず)。

---

## §5 生成経路監査結果

### §5.1 research_derived_indicators の writer

- **唯一の writer**: `scripts/jobs/persist_derived_indicators.py` (F286-R2-A2 smoke runner)
- 入力 (= read-only): market_financials_v2 + market_prices_daily + market_listings
- 書込: `INSERT OR REPLACE INTO research_derived_indicators` (line 227 付近)
- smoke モード:
  - `five_codes` (= 72030/67580/80350/83060/16050 5 銘柄)
  - `mini_100` (= market_listings 先頭 100 から eligible)
  - `full_eligible` (= 全件 ~3,752 銘柄、R2-A3 で初実行想定)
- 4 段 staging guard: FIRE_ENV + db_path basename + db_label + smoke_type
- 計算指標 7 種: per / pbr / roe / operating_margin / net_margin / sales_growth_yoy / profit_growth_yoy
- 上流: J-Quants API (= 既 ingest 済 tables のみから JOIN、本 wave で API call なし想定)

### §5.2 research_watchlist_signals の writer

- **writer**: `scripts/jobs/run_research_watchlist_signal_persistence.py`
- 入力チェーン: research_derived_indicators × market_listings (JOIN by code) → score_factor_strategies → watchlist_ranker → upsert_signals
- `--base-date` arg は **signal 行の出力ラベル**、入力 cut ではない (= 重要)
- `score_factor_strategies._fetch_records()` は `ORDER BY r.code ASC, r.base_date DESC` で **各 code の最新 base_date 行のみ採用**

### §5.3 --base-date セマンティクス (= 致命的注意点)

```
signal_persistence --base-date 2026-05-17 を実行しても、
入力 research_derived_indicators 内の各 code 最新 base_date 行が SELECT される。
現状 W2-B 9 銘柄全件、derived latest = 2026-05-01。
→ 案 B 単独実行で fresh signal は作れない (= "fresh-labeled but stale-input")
→ 案 A.5 (= derived regen 全件) を案 B の前提に挟む必要あり
```

---

## §6 次 wave 推奨順序 (= Codex Lane D HIGH 反映後)

```
[完了] 案 A   DATA-R1 prices+index refresh (= 2026-05-17 完了)

[次 (新規追加)] 案 A.0  financials_v2 refresh ★Lane D HIGH 反映
       marker: HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS (新規)
       runner: scripts/jobs/run_jquants_daily_refresh.py --datasets financials (要 WRITE_ENABLED_DATASETS 拡張別 wave)
       注: 現状 WRITE_ENABLED_DATASETS = {prices, index} のため、financials 追加は scope 拡張別 HQ 承認必要
       前提: 5 月中決算シーズンで roe/margin/growth 直撃懸念ありの場合のみ
       write: market_financials_v2 (最新分のみ)

[案 A.0 後] 案 A.5  derived indicators regen ★必須
       marker: HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH (新規)
       runner: persist_derived_indicators.py --smoke-type full_eligible
       所要時間: 約 1.9 分 (= preflight 推定、過大見積 5-15 分から修正)
       write: research_derived_indicators ~3,700+ rows

[案 A.5 後] 案 B  signal regeneration
       marker: HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH
       runner: run_research_watchlist_signal_persistence.py
       所要時間: 1-3 分
       write: research_watchlist_signals

[案 B 後] 案 C  W2-B re-execution
       marker: HQ_APPROVE_W2B_RE_RUN_FRESH_DATA
       runner: run_f111_overlay_momentum_sector_sim.py --as-of-date 2026-05-17
       所要時間: <1 分
       write: 0 (= sim only)
```

★ 各 wave 別 marker + 別 wave 推奨 (= 1 wave 統合しない)。
★ listings refresh は別 wave で後追い (= 16 日 stale 許容、致命的依存なし)。
★ 案 A.0 は決算シーズン (4-5 月、10-11 月) 中の判断、シーズン外は skip 可。
★ index_data 6 日 gap (5-11 → 5-17) は W2-B 直接依存外だが regime/指数特徴量系で別途 refresh 対象。

---

## §7 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| staging DB write | **0** (= mode=ro + immutable=1) |
| production DB write | **0** |
| develop DB write | **0** |
| listings write | 0 |
| signals write | 0 |
| derived write | 0 |
| schema migration | 0 |
| LINE 送信 | 0 |
| token / .env / secret 値表示 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push / PR | 0 |
| VACUUM / sudo / rm -rf / --no-verify | 0 |
| auto-order / 楽天 / iSPEED / Computer Use | 0 |
| API call (J-Quants/TDnet) | 0 |

---

## §8 Codex 4 lane 監査結果

| lane | verdict | CRITICAL | HIGH | 摘要 |
|---|---|---|---|---|
| A (DB freshness) | **APPROVE** | 0 | 0 | 5 table 主張値 1:1 一致、derived 内訳 3,708+42=3,750 算術一致、W2-B 9 codes 全件 derived=5-01 確認 |
| B (signal regen dep) | **APPROVE** | 0 | 0 | `--base-date` 出力ラベル限定確認、`_fetch_records` 各 code 最新 base_date 採用確認、A.5 必須結論妥当 |
| C (derived runner) | CONCERN | 0 | 0 | INSERT OR REPLACE のみ DELETE/DROP/VACUUM 0、guard 詳細修正、所要時間 1.9 分目安 (5-15 分は過大) |
| D (sequencing) | CONCERN | 0 | **1** | **HIGH**: A.5 前に market_financials_v2 refresh 推奨 (= 決算シーズン中の財務 stale 懸念) |

### §8.1 Lane A (DB freshness 主張値の独立確認) — **APPROVE**

DB 再 query で 5 table 主張値 1:1 一致:
- derived 3,750 / watchlist 13,839 / listings 4,449 / financials_v2 164,678 / prices 2,089,775
- derived 内訳: 2026-05-01: 3,708 + 2026-05-08: 42 = 3,750 (= 算術一致)
- W2-B 9 codes 全件 `derived_latest_base_date=2026-05-01`、5-08 行は 0、prices latest 全件 5-15
- 2026-05-08 derived 42 件 sector 分布: 建設業 19 / 水産・農林業 12 / 情報･通信業 4 / サービス業 2 / 不動産業 2 / 小売業 2 / 医薬品 1
- listings 16 日 stale 主張妥当

### §8.2 Lane B (signal regen 入力依存の正確性) — **APPROVE**

1. `_fetch_records` は `ORDER BY r.code ASC, r.base_date DESC` 後、`seen_codes` 済みなら `continue` で各 code 最新 1 行のみ採用
2. `run_research_watchlist_signal_persistence.py` `--base-date` は `convert_watchlist_to_signal_row(... base_date=base_date)` と結果読戻しに使用、入力 `_fetch_records(ro)` には不渡し
3. `run_research_watchlist_ranker.py` も `_fetch_records(conn)` をそのまま使用、全 `base_date` ではなく code 最新のみ
4. `--base-date` が `research_derived_indicators` の入力 cut / WHERE 条件として機能する経路なし
5. signal regen 単独では出力日付ラベルだけ fresh 化、入力指標 stale のまま
6. 案 A.5 derived regen 先行必須結論妥当

### §8.3 Lane C (derived 生成 runner / staging guard 検証) — CONCERN

1. `persist_derived_indicators.py` DB 書込 SQL は `INSERT OR REPLACE INTO research_derived_indicators` のみ。DELETE/DROP/VACUUM/UPDATE 経路なし ✓
2. 入力 read SQL は market_financials_v2 / market_prices_daily / market_listings のみ、HTTP/API 経路なし ✓
3. **修正**: staging guard 実装は `FIRE_ENV` / DB 存在 / symlink 拒否 / ファイル名 `fire.staging.db` で、本 doc 記載「db_label guard」は実際は **basename guard** が正しい
4. `smoke_type` guard と `full_eligible` の `--hq-approved` 必須 guard 実装あり
5. PK = (code, base_date, disclosure_date, type_of_document)、別 base_date は別行、5-01 既存行は同 4 キー一致なしで上書きされない ✓
6. **修正**: full_eligible 所要時間「5-15 分」は安全側過大、コード preflight 推定 約 0.03 sec/code × 3,752 件 ≈ **1.9 分**
7. migration DROP TABLE 文字列は rollback 説明のみ、実行経路なし ✓

### §8.4 Lane D (次 wave 順序 A.5→B→C の妥当性 + 漏れ監査) — CONCERN (HIGH 1)

**HIGH**: `market_financials_v2` 2026-05-08 latest のまま 5 月中決算シーズンに A.5 を回すと、`base_date=2026-05-17` だが財務入力 stale な derived 7 指標になるリスク。

1. A.5→B→C 依存順自体は妥当
2. **ただし A.5 前に `market_financials_v2` refresh 挟むのを推奨** (= 7 指標中 `roe/op_margin/net_margin/sales_growth_yoy/profit_growth_yoy` に直撃)
3. `market_listings` は A.5 前必須ではない (= 16 日 stale 別 wave 後追いで十分)
4. A.5+B 1 wave 統合非推奨 (= A.5 row count / failed_codes / stale financials 検証後に B 切るのが安全)
5. 漏れ監査: `index_data` 2026-05-11 latest の 6 日 gap は W2-B 直接依存外だが regime/指数特徴量系で別途 refresh 対象
6. `research_watchlist_signals` 13,839 rows / 31 base_dates / latest 2026-07-22 合成日付は latest 参照系混入リスクの別監査推奨
7. marker `HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH` は既存 marker と衝突なし、A.5 専用で妥当

→ **Lane D 推奨 RECOMMENDED_NEXT = OTHER** (= 案 A.0 financials_v2 refresh → A.5 → B → C)

---

## §9 案 A.5 / B / C 詳細

詳細は `/tmp/fire_derived_freshness_audit/next_wave_recommendation.md` 参照。

要点:
- 案 A.5 = derived regen full_eligible: 推定 5-15 分、staging write のみ
- 案 B = signal regen: 1-3 分、staging write のみ
- 案 C = W2-B re-run: <1 分、write 0

---

## §10 結論 / 次アクション

### §10.1 結論

- staging `research_derived_indicators` は 16 日 stale (= 2026-05-01 主データ)
- W2-B 9 銘柄全件、derived stale
- signal_persistence `--base-date` は出力ラベルであり入力 cut ではないため、案 B 単独では fresh signal にならない
- **案 A.5 (derived regen) を案 B の前提 wave として実施が必須**
- DB write 0 / production-develop 不変 / コード変更 0

### §10.2 次 wave 推奨 (= Lane D HIGH 反映後)

1. **案 A.0** (Lane D HIGH 反映): 決算シーズン中なら HQ 承認後 `HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS` で financials_v2 refresh 先行
2. **案 A.5**: HQ 承認後 `HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH` で実行
3. **案 B**: 案 A.5 完了確認後 `HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH` で実行
4. **案 C**: 案 B 完了確認後 `HQ_APPROVE_W2B_RE_RUN_FRESH_DATA` で実行
5. listings refresh は別 wave で後追い (= 任意、月次運用想定)
6. research_watchlist_signals 合成日付 (= 2026-07-22) latest 参照系混入リスクは別 wave で独立 audit 推奨

---

## §11 関連 file

```
本 wave 出力 (= /tmp 限定):
  /tmp/fire_derived_freshness_audit/derived_freshness.json
  /tmp/fire_derived_freshness_audit/signal_input_dependency.md
  /tmp/fire_derived_freshness_audit/next_wave_recommendation.md
  /tmp/fire_derived_freshness_audit/codex_lane_A_prompt.txt + result.txt
  /tmp/fire_derived_freshness_audit/codex_lane_B_prompt.txt + result.txt
  /tmp/fire_derived_freshness_audit/codex_lane_C_prompt.txt + result.txt
  /tmp/fire_derived_freshness_audit/codex_lane_D_prompt.txt + result.txt

設計 doc:
  ~/fire-vault/03_design/F111_DERIVED_FRESHNESS_AUDIT_R1_2026-05-16.md (本 doc)
```
