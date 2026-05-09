---
id: F286-R2-E
phase: P5: Research Lane R2 (signal persistence + backtest foundation)
priority: 高 (時系列 backtest の基盤、R2-F backtest evaluation の前提)
status: 完了 (migration / dry-run / write / re-run UPSERT / read helper smoke、production/develop DB 完全無触)
owner: Fujiwara
depends_on: [F286-R2-D Watchlist Ranker]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R2_D_watchlist_ranker, F286_R2_C_factor_scoring_smoke
---

★ **F286 R2-E**: Watchlist Ranker (R2-D) の signal を base_date 単位で
   時系列保存する基盤を完成。staging のみへの atomic UPSERT、production
   / develop write 完全 block、read helpers 5 種で R2-F backtest 着手
   可能な状態。

# F286-R2-E Signal Persistence / Backtesting Foundation

## サマリ

R2-D Watchlist Ranker の出力を `research_watchlist_signals` table に
**base_date / code / source_version** の 3-key で永続化。default
dry-run、staging への明示 `--write` のみ DB 変更。production/develop
への --write は即エラー。

実装 4 module:
1. **`scripts/setup/migrate_research_watchlist_signals.py`** (schema)
2. **`simulation/research_lane/signal_persistence.py`** (純粋 + read helpers)
3. **`scripts/jobs/run_research_watchlist_signal_persistence.py`** (runner)
4. tests 3 ファイル

## table schema

```sql
CREATE TABLE research_watchlist_signals (
    base_date          TEXT NOT NULL,
    code               TEXT NOT NULL,
    source_version     TEXT NOT NULL,
    generated_at       TEXT NOT NULL,
    final_score        REAL,
    final_rank_label   TEXT,
    pre_cap_rank       INTEGER,
    post_cap_rank      INTEGER,
    sector_17_code     TEXT,
    sector_17_name     TEXT,
    sector_cap_status  TEXT NOT NULL,
    sector_cap_reason  TEXT,
    strongest_strategy TEXT,
    available_strategy_count INTEGER NOT NULL,
    quality_value_score   REAL,
    earnings_growth_score REAL,
    cyclical_value_score  REAL,
    quality_value_rank_label   TEXT,
    earnings_growth_rank_label TEXT,
    cyclical_value_rank_label  TEXT,
    strategy_signal_summary_json TEXT,
    exclusion_summary_json       TEXT,
    metadata_json                TEXT,  -- top_n_scope / weights / cap_ratio
    excluded_all_strategies      INTEGER NOT NULL,  -- 0/1
    watchlist_decision           TEXT NOT NULL,
    watchlist_decision_reason    TEXT,
    rank_reason                  TEXT,
    created_at                   TEXT NOT NULL,
    PRIMARY KEY (base_date, code, source_version)
);

-- 7 INDEX:
CREATE INDEX idx_..._base_date ON ...(base_date);
CREATE INDEX idx_..._code ON ...(code);
CREATE INDEX idx_..._final_rank_label ON ...(base_date, final_rank_label);
CREATE INDEX idx_..._post_cap_rank ON ...(base_date, post_cap_rank);
CREATE INDEX idx_..._sector_17_code ON ...(sector_17_code);
CREATE INDEX idx_..._strongest_strategy ON ...(base_date, strongest_strategy);
CREATE INDEX idx_..._generated_at ON ...(generated_at);
```

PK 3-key の意味:
- `base_date`: 信号生成基準日 (= price_date 由来)
- `code`: 銘柄
- `source_version`: ロジック / 重み調整の version 識別子
  (例: `r2d_v1`、`r2d_v2_qv_0_4`)

→ **同 base_date / code でも別 source_version で並列保存可能**、
  backtest で異なる weight 設計の比較に使う。

## persistence 設計

### SignalRow + validate

`SignalRow` (frozen dataclass) で table 1 row。`convert_watchlist_to_
signal_row` で R2-D `WatchlistEntry` から変換。validation 7 項目
(code/base_date/source_version/sector_cap_status/watchlist_decision
必須、final_rank_label が ALLOWED、excluded_all と final_score の
整合性、final_score 有りで rank いずれか必須、0/1 制約)。

### top_n filter

```
filter_signals_by_top_n(rows, top_n=N):
  None    → 全件 (ALL 保存)
  整数 N  → pre_cap_rank<=N OR post_cap_rank<=N
            (= deferred も backtest 用に残す)
```

### Atomic UPSERT (Codex CRITICAL #4 対応)

```python
upsert_signals(conn, rows, write_enabled=True):
  - validation skipped を pre-filter
  - BEGIN → 全 row 投入 → commit
  - 1 row でも sqlite3.Error なら **全 batch rollback**
  - 部分書込みを防止
  - dry-run (write_enabled=False) では DB に触れない
```

### Read helpers (5 種)

| 関数 | 用途 |
|---|---|
| `load_signals_by_base_date(conn, base_date)` | 指定日の全 signal |
| `load_top_signals(conn, base_date, top_n)` | post_cap_rank <= N |
| `load_signals_by_rank_label(conn, base_date, labels)` | A1/A2 等 |
| `load_signal_history(conn, code)` | code の全 base_date 履歴 |
| `compare_signal_dates(conn, date_from, date_to)` | 2 日 比較 (common / from_only / to_only / rank_changes) |

JSON-text fields (`strategy_signal_summary_json` 等) は **parse 済 dict
で返す** ので、呼び出し側で JSON parse 不要。

## Runner 仕様

```
FIRE_ENV=staging python -m scripts.jobs.run_research_watchlist_signal_persistence \
    --db {staging|develop|production} \
    --base-date YYYY-MM-DD \
    --source-version <id>           # 必須
    --top-n {N|ALL} \                # default 30
    --dry-run | --write              # default dry-run
    [--db-path /path] [--output-json] [--output-csv]
```

**Safety guard**:
- `--db production` + `--write` → 即 `PersistenceRunRefused`
- `--db develop` + `--write` → 即 `PersistenceRunRefused`
- `--db staging` + `--write` → 4 段 staging guard (FIRE_ENV /
  basename / symlink / 不在) 通過必須
- default は dry-run、`--write` を明示しない限り DB に書かない

## 実行結果 — migration

```
=== F286-R2-E research_watchlist_signals migration ===
target_table: research_watchlist_signals
table_existed_before: False
row_count: 0 -> 0
schema_verified: True
rollback_strategy: DROP TABLE research_watchlist_signals
```

## 実行結果 — dry-run (top_n=100)

```
records loaded: 3,708
rows after top_n filter: 109 (= post_cap<=100 100 件 + deferred で
                                pre_cap<=100 9 件)
upsert: inserted=109 / replaced=0 / skipped=0 / failed=0
write_enabled: False
row_count: 0 -> 0  ★ DB 書き込みなし
```

## 実行結果 — staging write (top_n=100)

```
mode:           WRITE
base_date:      2026-05-09
source_version: r2d_v1
top_n:          100
records loaded: 3,708
rows after top_n filter: 109
upsert: inserted=109 / replaced=0 / skipped=0 / failed=0
write_enabled: True
row_count: 0 -> 109
```

## 実行結果 — UPSERT 再実行検証

同 (base_date=2026-05-09, source_version=r2d_v1, top_n=100) で再 run:

```
upsert: inserted=0 / replaced=109 / skipped=0 / failed=0
row_count: 109 -> 109  ★ 増えない (= 全件 PK で UPSERT、idempotent)
```

→ **PK 3-key (base_date, code, source_version) で安全 UPSERT**。

## 実行結果 — production write 拒否確認

```
$ FIRE_ENV=staging .venv/bin/python -m scripts.jobs.run_research_...
    --db production --base-date 2026-05-09 --source-version r2d_v1 \
    --top-n 30 --write

PersistenceRunRefused: refusing: --write is forbidden for
db='production' (production / develop must not be modified by R2-E
persistence runner)
```

→ guard 動作確認 OK。

## read helper 動作確認

```python
load_signals_by_base_date(conn, '2026-05-09')      # 109 rows
  top: 87470 score=0.9299 rank=A1
load_top_signals(conn, '2026-05-09', top_n=30)     # 30 rows
  top30 sectors: IT 13 / 不動産 5 / 商社 3 / 金融 2 / 機械 2
  ※ top_n=100 で計算保存したので cap=30 件、IT が pre_cap rank
    上位に集中して 13 件に
load_signals_by_rank_label(conn, '2026-05-09',
                           labels=['A1', 'A2'])    # 109 rows (全 A1)
load_signal_history(conn, '87470')                 # 1 row (1 base_date のみ投入)
compare_signal_dates(conn, '2026-05-09', '2026-05-09')
  common=109 / from_only=0 / to_only=0
strategy_signal_summary (parsed): {quality_value:'A1',
  earnings_growth:'A1', cyclical_value:'A1'}
metadata: {top_n_scope:100, weights:{...},
           cap_ratio:0.3, cap_top_n_used_for_post_cap:100}
```

→ JSON-text fields も dict 形式で access 可能、metadata で
   再現性情報を保存。

## DB 安全性

| DB | last_modified | 状態 |
|---|---|---|
| production (`fire.db`) | 2026-05-07 16:12 | 完全無触 |
| develop (`fire.develop.db`) | 2026-05-07 18:14 | 完全無触 |
| staging (`fire.staging.db`) | 2026-05-09 20:21 | research_watchlist_signals に 109 row 書込 |

**自動発注禁止 / 楽天証券自動化禁止 / Computer Use 禁止 / Playwright
禁止** などの HQ 必須制約は本タスクで関与せず (= 完全 read/write
ロジック層、UI 通知系も触らない、staging のみ)。

## constraints_check

```json
{
  "staging_only_write": true,
  "production_db_untouched": true,
  "develop_db_untouched": true,
  "write_guard_active": true,
  "atomic_transaction": true,
  "default_dry_run": true
}
```

## tests

| 対象 module | tests |
|---|---|
| scripts/setup/migrate_research_watchlist_signals | 15 PASS |
| simulation/research_lane/signal_persistence | 28 PASS |
| scripts/jobs/run_research_watchlist_signal_persistence | 26 PASS |
| **R2-E 合計** | **69 PASS** |
| 関連 regression (research_lane / scripts) | **676 PASS** |

**HQ 必須テスト 全項目クリア**:
- signal row mapping ✅
- validation ✅
- source_version required ✅
- unique key / duplicate handling ✅ (re-run replace)
- dry-run does not write ✅
- production/develop write guard ✅
- staging write allowed only with --write ✅
- top_n filtering ✅
- ALL 保存 option ✅
- read helper by base_date / rank_label / code / compare_dates ✅
- runner smoke ✅

## commit ログ

| # | commit | hash |
|---|---|---|
| R2-E-1 | feat(F286-R2): add research signal persistence schema | 7d195e8 |
| R2-E-2 | feat(F286-R2): add watchlist signal persistence | 9d1399a |
| R2-E-3 | chore(F286-R2): add signal persistence runner | 939ab38 |
| R2-E-4 | test(F286-R2): add signal persistence tests | 6aea83c |
| R2-E-5 | docs(F286-R2): vault signal persistence smoke result | (本 commit) |
| R2-E-6 | docs(F286-R2): log milestone — signal persistence completed | (次 commit) |

Codex pre-commit 通過 × 4 (feat schema / feat persistence / chore /
test)。CRITICAL 1 件発生 → 即修正 (atomic transaction、commit 9d1399a)。
--no-verify 不使用。

## Go/No-Go 判定

判定: **PASS、実用 OK、R2-F backtest evaluation 進行可能**

| 条件 | 結果 |
|---|---|
| migration 冪等性 | ✅ |
| dry-run / write 切替が安全 | ✅ |
| production / develop write block | ✅ |
| atomic UPSERT で部分書込みなし | ✅ |
| read helpers が R2-F で必要な access pattern を網羅 | ✅ 5 種 |
| metadata 保存で再現性確保 | ✅ |
| validation で不正 row を skip | ✅ |
| HQ 必須テスト 全項目 PASS | ✅ |

## 次のアクション (HQ 判断)

R2-E 完了 → 候補:

1. **R2-F return / backtest evaluation**: ある base_date の A1/A2
   銘柄が、その後 N 営業日 / N 週間でどう動いたかを `market_prices_daily`
   と join して return / sharpe / win rate を計算。time-series での
   strategy 検証の本格化。
2. **R2-D2 tuning**: 重み (QV/EG/CV)、cap_ratio (30%)、strongest
   閾値 (0.01) などの parameter 調整。複数 source_version で並列
   保存して比較。
3. **Lane integration**: 既存 Daytrade Selection / Swing Selection /
   Long-term Selection に research_watchlist_signals の output を
   入力。Fujiwara 手動レビューの効率化。
4. **R1-B5 (v1/v2 swap)**: 既存 v1 (1,723 row) を drop / rename して
   v2 を正本化、別途 HQ 承認後。

R2-F が最も次工程として重要 (= 本タスクの目的「将来 backtest 用基盤」
の本来の利用)。

## 関連リンク

- [[F286_R2_D_watchlist_ranker|F286-R2-D Watchlist Ranker]]
- [[F286_R2_C_factor_scoring_smoke|F286-R2-C 3 戦略 scoring]]
- [[F286_R2_B_indicator_distribution_analysis|F286-R2-B 分布検証]]
- [[F286_R2_A3_full_eligible_indicators|F286-R2-A3 全 eligible 永続化]]

## 評価 (タスク完了基準 v1.2)

- **動いた**: ✅ migration / dry-run / write / re-run / read helpers
  全て exit 0、atomic UPSERT 動作、production write block 動作
- **機能した**: ✅ HQ 必須テスト 11 項目 全 PASS、再 run で
  inserted 109 → replaced 109 と PK 動作確認、read helpers 5 種で
  期待 result、production/develop DB 完全無触、Codex CRITICAL 修正
  (atomic transaction)
- **期待値達成**: ✅ R2-F backtest 着手可能な基盤完成、source_version
  で並列保存可、metadata で再現性、tests 69 PASS / regression 676 PASS、
  Codex pre-commit 通過 × 4、--no-verify 不使用
