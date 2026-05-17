# FIRE F111-DATA-FRESHNESS-GATE-R1 (2026-05-17)

doc_id: FIRE-F111-DATA-FRESHNESS-GATE-R1-2026-05-17
status: 実装完了 / 36 PASS / 値非表示維持 / DB write 0 / API 0
HQ marker: (= 純実装 + read-only smoke、specific marker 不要)
related:
- [[JQUANTS_ENV_SUPPLY_PREFLIGHT_R1_2026-05-17]] — 前 wave、retry 前提
- [[STAGING_DB_RESTORE_FROM_BACKUP_RECENT_2026-05-17]] — 現 staging 起点

---

## §1 目的

F111 / W2-B / Universe Expansion / F062 daily flow が古いデータを使って
候補を出さないように、staging DB 主要 6 table の鮮度を read-only で
判定する freshness gate を実装。

★ 本 wave スコープ: helper + runner + tests のみ。F111 候補生成への
本組み込みは別 wave。

---

## §2 承認範囲

| 項目 | 内容 |
|---|---|
| 対象 file | scripts/jobs/_f111_data_freshness_gate.py (helper)<br>scripts/jobs/run_f111_data_freshness_gate.py (runner)<br>tests/scripts/jobs/test_run_f111_data_freshness_gate.py |
| 対象 DB | staging (URI mode=ro のみ) |
| 禁止守備 | DB write 0 / API call 0 / token-env 0 / LINE 0 / launchd-related 0 / git add/commit/push 0 / PR/merge 0 / sudo/rm -rf 0 / auto-order 0 / 楽天/iSPEED 0 / Computer Use 0 / TODO Excel 0 |

---

## §3 freshness gate 設計

### §3.1 主要 6 table

| table | latest 列 | status 判定根拠 | blocks |
|---|---|---|---|
| market_prices_daily | `date` | OK: lag <= 0 / WARN: 1-3 / FAIL: >= 4 (= bdays) | w2b_live_use, signal_regen, line_send (FAIL 時) |
| index_data | `date` | regime/指数系専用、W2-B 直接依存外 | regime_analysis |
| market_financials_v2 | `disclosure_date` | OK: <= 5 bdays / WARN: > 10 / FAIL: absent | signal_regen, universe_expansion (absent 時) |
| research_derived_indicators | `base_date` ★hardening | derived < prices で signal_regen FAIL | signal_regen (FAIL 時) |
| research_watchlist_signals | `base_date` | future synthetic / stale-derived 検出 | (= 警告のみ、blocking なし) |
| market_listings | `fetched_at` | 7 日 OK / 8-14 INFO / 15+ WARN | universe_expansion (absent 時) |

### §3.2 status 値

- **OK**: fresh、blocks なし
- **WARN**: 一定 stale だが致命的でない、blocks 限定
- **FAIL**: 致命的 (= retry/restore/regen 要)、blocks 多数
- **INFO**: future synthetic 等の参考情報

### §3.3 blocks 値 (= 後続 wave 判定用 string)

- `blocks_w2b_live_use`
- `blocks_signal_regen`
- `blocks_universe_expansion`
- `blocks_line_send`
- `blocks_regime_analysis`

### §3.4 previous_trading_day (= 土日 fallback)

| 基準日 weekday | 前営業日 |
|---|---|
| Mon (0) | Fri (-3 days) |
| Tue-Fri (1-4) | 前日 (-1 day) |
| Sat (5) | Fri (-1 day) |
| Sun (6) | Fri (-2 days) |

★ **祝日対応は本 wave 未実装** (= 別 wave TODO、helper の `previous_trading_day` docstring に明記)

---

## §4 current staging 判定結果 (= reference 2026-05-17)

| table | status | rows | latest | lag_bdays | blocks |
|---|---|---|---|---|---|
| market_prices_daily | **OK** | 2,089,775 | 2026-05-15 | 0 | - |
| index_data | WARN | 60 | 2026-05-11 | 4 | blocks_regime_analysis |
| market_financials_v2 | OK | 164,678 | 2026-05-08 | 5 | - |
| **research_derived_indicators** | **FAIL** | 3,750 | 2026-05-08 | 5 (vs prices) | **blocks_signal_regen** |
| research_watchlist_signals | INFO | 13,839 | 2026-07-22 (合成) | - | - (= future synthetic) |
| market_listings | WARN | 4,449 | 2026-05-01 | 16 cdays | - |

**overall_status: FAIL**
**aggregated_blocks: [blocks_regime_analysis, blocks_signal_regen]**

→ 解釈:
- prices fresh、financials は WARN 域内 (5 bdays、許容)
- **derived が prices より 5 bdays 古い** → signal_regen 不可
- 案 A.5 (derived regen) wave を実行すれば解消
- index/listings は WARN だが W2-B 直接依存外

---

## §5 W2-B 対象 9 銘柄 coverage

全 9 銘柄で:
- price_latest = 2026-05-15 (= 全件 fresh)
- derived_latest = 2025 年の値 (= 各 code の最新だが古い)
- listing_exists / financials_exists = True 全件
- status = OK (= 必要 data が揃っている)

★ per-code coverage は「データ存在」判定で OK だが、**table level の derived
が prices より古いため signal_regen は block される** (= §4 参照)。

W2-B 候補生成自体 (= price + listing 揃いで実行可) は可能。
ただし signal_regen を含む downstream pipeline は derived regen 後。

---

## §6 tests (= 全 PASS、hardening 反映後 42 PASS)

tests/scripts/jobs/test_run_f111_data_freshness_gate.py: **42 PASS** (= R1 36 + hardening +6)

内訳:
1. TestPreviousTradingDay (6): 土日 fallback / business_days_between
2. TestJudgePrices (5): OK/WARN/FAIL/table missing/zero rows
3. TestJudgeDerived (4): OK / signal_regen block / table absent / prices unknown
4. TestJudgeSignals (3): future synthetic / stale-derived / table absent
5. TestJudgeListings (4): 7 days OK / 8-14 INFO / 15+ WARN / absent
6. TestW2BCoverage (4): all OK / price absent / derived absent / financials INFO
7. TestRunnerIntegration (7): full run / signals future / URI mode=ro / symlink refused / outside allowed / JSON+MD output / strict mode FAIL exit 4
8. TestSourceSafety (2): forbidden imports なし / PRAGMA query_only ON

実行時間: 0.04 sec、regression 0。

---

## §7 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| DB write (prod/dev/staging) | 0 (= 全 md5 不変) |
| 実 J-Quants API call | 0 |
| token / .env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| schema migration | 0 |
| git add / commit / push / PR | 0 |
| auto-order / 楽天 / iSPEED | 0 |
| Computer Use | 0 |
| sudo / rm -rf / --no-verify | 0 |
| TODO Excel | 0 |
| financials refresh / retry | 0 (= 別 wave) |

---

## §8 5/18 post-snapshot 再利用性

5/18 02:00 F282 weekly-snapshot で staging が production 同期される。
直後に本 runner を実行することで:
- 同期後の staging に対し OK/WARN/FAIL を機械判定可能
- production-同期で prices/index/derived/signals/listings がどうなるか
  即座に audit
- financials refresh retry の前提条件 (= derived/signals/prices 状態)
  を再確認できる

実行例 (= 5/18 朝):
```
.venv/bin/python -m scripts.jobs.run_f111_data_freshness_gate \
  --db-path data/fire.staging.db --db-label staging \
  --reference-date 2026-05-18 \
  --output-json /tmp/.../post_snapshot_gate.json \
  --output-md /tmp/.../post_snapshot_gate.md
```

---

## §9 Codex 8 lane 監査結果

| lane | verdict | CRITICAL | HIGH | 摘要 |
|---|---|---|---|---|
| A (gate 仕様 / status 設計) | APPROVE | 0 | 0 | 仕様と実装一致、status 境界整合 |
| B (table freshness / read-only) | APPROVE | 0 | 0 | mode=ro+immutable+query_only 確認、write attempt raise |
| C (W2-B coverage / edge) | CONCERN | 0 | **1** | W2-B per-code derived stale 見逃し edge case |
| D (tests / fixtures / strict) | CONCERN | 0 | **1** | immutable=1 で WAL frame 読み損ねリスク |
| E (no-write/token/API/LINE) | APPROVE | 0 | 0 | safety gate 確認 |
| F (F111/W2-B/F062 integration) | CONCERN | 0 | **1** | F062 consumer schema mismatch (= adapter 要) |
| G (next-wave / 5/18 snapshot) | APPROVE | 0 | 0 | gate 再利用性確認 |
| H (docs / handoff / risk) | CONCERN | 0 | **1** | 同 F062 schema mismatch + WAL 運用注記 |

### §9.5 Lane C HIGH (= 採用 + ★ hardening wave で完全反映済)
**指摘**: W2-B per-code coverage は derived の存在のみ判定、絶対鮮度
未チェック → derived stale が global level FAIL で遮断されないと
表示上 OK 扱いになる。

**対応 (hardening wave 完了)**:
- helper の judge_w2b_code_coverage に reference_date kwarg 追加
- per-code derived 絶対鮮度判定 (OK: 0-5 bdays / WARN: 6-20 / FAIL: > 20)
- price_status / derived_status / derived_lag_bdays / derived_reason
  field 追加 (= W2BCodeCoverage)
- render_markdown 10 列に拡張、price/derived 別 status 表示
- tests +6 件 (per-code derived fresh/WARN/FAIL/missing 等)

→ current staging で 9 codes 全件 status=WARN
  (price OK / derived WARN 10 bdays = base_date 5-01 vs ref 5-17)

### §9.6 Lane D HIGH (= 採用、運用注記 + 別 wave 強化)
**指摘**: `immutable=1` 指定は SQLite WAL 運用時に *.db-wal frames を
読み損ねる (= checkpointed data のみ読む) ため、freshness 判定が
古い値になり得る。

現状: staging DB は journal_mode=delete (= 前 wave 確認済) のため
WAL frame は存在せず影響なし。ただし将来 WAL モード DB に対して
使う場合は問題。

**対策 (= 運用注記)**: docstring に「WAL モード DB では immutable=1
を外す or checkpoint 後に使用」を追記。本 wave では journal_mode=delete
staging に限り使用と明示。

### §9.7 Lane F HIGH + §9.8 Lane H HIGH (= 同源、別 wave で対応)
**指摘**: 既存 F062 / Ops Summary の freshness consumer は
`schema_version` + `verdict` (= pass/warning/refuse) 形式を期待。
本 gate は `overall_status` (= OK/WARN/FAIL) 形式 + `aggregated_blocks`
で出力。直接 handoff すると schema mismatch で安全停止 or 表示欠落。

現状: 本 wave スコープは「**read-only gate 実装・tests・smoke のみ。
既存 F111 候補生成ロジックへの本組み込みはまだしない**」(goal §16)
で明示的に integration 対象外。

**対策 (= 別 wave)**:
- 別 wave で F062/Ops Summary 向け adapter 実装
  (`overall_status` → `pass/warning/refuse` mapping)
- または本 helper に `to_f062_schema()` 互換 helper 追加
- 直接 handoff しない旨を本 doc § 10 next-actions に明記

---

## §10 次アクション

### Option B' (= 推奨、即進行可):
1. HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH (= 案 A.5)
   - 現 derived (= 5-08) → prices (= 5-15) 鮮度合わせ
   - 本 gate が **derived FAIL → signal_regen block** を明示
2. HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH (= 案 B、derived regen 後)
3. HQ_APPROVE_W2B_RE_RUN_FRESH_DATA (= 案 C)

### Option F282-Y (= 5/18 02:00 後):
1. 5/18 02:00 F282 weekly-snapshot 自動実行
2. post-snapshot gate 実行 → 鮮度再判定
3. financials refresh retry (= env 既整備済、wrapper 方式)
4. derived regen → signal regen → W2-B re-run

### 本 wave 成果物 (= 別 wave で commit/push):
- fire 3 file (= helper + runner + tests)
- fire-vault doc

---

## §10b hardening wave (2026-05-17 後追い) 追加 Codex 6 lane 結果

hardening wave で R1 の追加 audit を実施 (= 元 Lane C HIGH 修正済の
verify + 新規 HIGH 発見)。

| lane | verdict | HIGH | 摘要 |
|---|---|---|---|
| A (per-code derived logic) | APPROVE | 0 | 実装一致確認 |
| B (output clarity) | CONCERN | 1 | price MISSING で derived short-circuit → 修正済 |
| C (tests / edge) | APPROVE | 0 | 42 PASS 確認 |
| D (safety / no-write) | CONCERN | 1 | aggregated_blocks に per-code FAIL 未反映 → 修正済 |
| E (5/18 post-snapshot) | CONCERN | 1 | D と同根 + 4桁/5桁 fallback note → D 修正で blocks 解消、code fallback は別 wave |
| F (commit readiness) | **REJECT** | 1 | derived 鮮度は base_date 基準 (既存 contract 準拠) → **根本修正済** |

★ hardening wave で 3 HIGH + 1 REJECT 全件採用、本 commit 前修正:
- Lane B: price MISSING でも derived 絶対鮮度を計算するよう修正
- Lane D: aggregate_overall に BLOCK_W2B_LIVE_USE 追加 logic
- Lane F: judge_derived の引数を latest_base_date に変更、runner で
  MAX(base_date) を読むよう修正、tests 全件更新

詳細: /tmp/fire_f111_data_freshness_gate/codex_h_lane_{A,B,C,D,E,F}_result.txt

---

## §11 関連 file

```
本 wave 出力:
  /tmp/fire_f111_data_freshness_gate/
    freshness_gate.json (= current staging 判定結果)
    freshness_gate.md (= human-readable)
    codex_lane_{A,B,C,D,E,F,G,H}_prompt.txt + result.txt

modified (= 本 wave、commit 未実施):
  scripts/jobs/_f111_data_freshness_gate.py (+440 lines、helper)
  scripts/jobs/run_f111_data_freshness_gate.py (+280 lines、runner)
  tests/scripts/jobs/test_run_f111_data_freshness_gate.py (+520 lines、36 件)

設計 doc (= untracked):
  ~/fire-vault/03_design/F111_DATA_FRESHNESS_GATE_R1_2026-05-17.md
```
