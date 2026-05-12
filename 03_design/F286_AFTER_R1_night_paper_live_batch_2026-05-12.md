---
id: F286-AFTER-R1-night-paper-live-batch
phase: After-Close Night Paper Live Batch Runner (= F286 系)
priority: 高
status: 設計 v1.0 (= Wave 35-pre、impl は別 wave + HQ 明示承認後)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-12
depends_on:
  - F286 PNL R1/R2/R3 (= advisory_decisions / paper_pnl 既存)
  - F286 REPORT-R1 (= daily/weekly/monthly 既存)
  - F282 (= 試走待機中、AFTER-R1 と read source 共通)
  - HQ Wave 35-pre 起票承認 (= 2026-05-12)
---

# F286-AFTER-R1 / After-Close Night Paper Live Batch Runner — 設計 v1.0

後場終了後 〜 夜間に Paper Live / Replay / Simulation / Lane 評価を
大量実行し、勝ち / 負けパターン + Lane 昇格降格候補 + 翌営業日 Advisory
改善材料を生成する。**本 doc は設計のみ**、impl / DB write / cron / LINE
/ token は別 wave + HQ 明示承認後。

## 0. 概要

| 項目 | 内容 |
|---|---|
| ID | F286-AFTER-R1 |
| 種別 | After-Close 夜間バッチ runner |
| 実行時間帯 | 15:00 JST 以降 (= 後場終了後) / 23:00 (= 夜間) / 08:00 (= 翌朝 Advisory 前) |
| 入力 | advisory_decisions / paper_pnl / market_prices_daily / etc |
| 出力 | win/loss patterns / Lane 候補 / ML feature 候補 / REPORT 連携 |
| 安全 | default dry-run、production read-only、staging-only write (= 別 wave) |
| 関連 | PNL R1/R2/R3 / REPORT-R1 / LANE-R0/R1/R2 / DASH-R1 / RISK-R1 / SIM-R1 |

## 1. 責務定義 (= L1a 反映)

### MVP scope (= Phase 1)

- 既存 advisory_decisions / paper_pnl / market_prices_daily から **集約 read のみ**
- Markdown report + json summary 出力
- DB write **0** (= read-only first)
- REPORT-R1 に 1 section 追加 or 別 file 出力

### Phase 2 拡張 (= 別 wave)

- Paper Live 再計算 (= 仮想 entry/exit を horizon 別に再演算)
- Lane 評価 (= win rate / max drawdown / sample size)

### Phase 3 (= 将来、cron 化)

- Replay (= 過去 6 ヶ月 historical price + decisions 再計算)
- Simulation (= 仮想 decisions + parameter swap)
- launchd 登録 (= 別 wave + HQ approve)

### Paper Live / Replay / Simulation / Lane 評価 責務分解

| 種別 | 入力 | 処理 | 出力 |
|---|---|---|---|
| Paper Live | 当日 advisory_decisions + market_prices | 仮想 PnL 再計算 | paper_pnl 集約 |
| Replay | 過去 N 日 historical price + decisions | 同 logic で再計算 | 過去 PnL |
| Simulation | 仮想 decisions (= 別 parameter) | scenario 模擬 | 仮想 PnL |
| Lane 評価 | 全 PnL data + decisions metadata | aggregate by lane/label | win rate / drawdown |

## 2. 入力データ (= L1b 反映)

### read source (= 既存、read-only)

| source | DB / file | 用途 | 既存 module 再利用 |
|---|---|---|---|
| advisory_decisions | data/fire.db.advisory_decisions | advisory 集約 | pnl/aggregators.py |
| advisory_snapshots | data/fire.db.advisory_snapshots | snapshot 集約 | 同上 |
| advisory_snapshot_rows | 同上 | row 単位 | 同上 |
| paper_pnl | data/fire.db.paper_pnl | 仮想 PnL | pnl/paper_pnl.py |
| market_prices_daily | data/fire.db.market_prices_daily | 価格 | market_data/* |
| research_watchlist_signals | 同上 | signal | scripts/jobs/run_research_* |
| REPORT-R1 output | reports/eod_*.md / weekly / monthly | 既存 summary | fire/report/* |
| (将来) actual_trade | (= F235 後) | 実約定 | 別 wave |
| (将来) F101 announcements | data/fire.db.announcements | 材料 | F101 修正後 |

### read filter (= W17 smoke 除外)

- `SMOKE_SOURCE_VERSIONS_BLACKLIST = ("w17-3-smoke",)`
- W30 snapshot retention 由来 (= data/snapshot/) は本 module で **read しない**
- 既存 helper を再利用

## 3. 出力データ (= L1b 反映)

### 設計上の出力 (= 実 write は別 wave)

| output | format | 用途 |
|---|---|---|
| win_patterns | json + markdown | lane/sector/label 別 勝ちパターン |
| loss_patterns | json + markdown | 負けパターン |
| lane_promotion_candidates | json + markdown | 昇格候補 (= win rate 高 + sample 十分) |
| lane_demotion_candidates | 同上 | 降格候補 (= drawdown 大 / negative skew) |
| advisory_improvement | markdown | 翌営業日 Advisory hints |
| ml_feature_candidates | json | ML 用 feature 候補 (= 過学習防止前段) |
| summary_for_REPORT_R1 | markdown 1 section | REPORT-R1 連携 |
| summary_for_DASH_R1 | json | DASH-R1 (= 将来) 連携 |

### 出力 file 配置

```
reports/after_r1/
├── YYYY-MM-DD/
│   ├── summary.json
│   ├── report.md
│   ├── win_patterns.json
│   ├── loss_patterns.json
│   ├── lane_candidates.json
│   └── ml_features.json
└── smoke/                       (= staging smoke 専用 dir)
    └── YYYY-MM-DD/...
```

## 4. runner 設計 (= L3 反映)

### path 案

- `scripts/jobs/run_f286_after_r1_night_batch.py`
- 本 wave は **設計のみ**、impl 別 wave

### argparse 構成

```
--dry-run                  (default True)
--db-label production|staging|develop|snapshot
--db-path                  (default ~/fire/data/fire.db)
--output-dir               (default reports/after_r1/{YYYY-MM-DD}/)
--horizon                  choices ["7d","30d","90d"]、default "30d"
--emit-line                bool、default False
--filter-blacklist         (default ("w17-3-smoke",))
--output-json              path
--completion-report        path
--markdown-report          path
--lane-aggregation         choices ["lane","sector","label","regime"]
--min-sample-size          int、default 10
--walk-forward             bool、default False
```

### 関数 signature (= 別 wave で impl)

```python
def _check_run_window(now) -> bool: ...     # 後場終了確認
def _read_advisory_data(db, horizon, blacklist) -> pd.DataFrame: ...
def _read_paper_pnl(db, horizon, blacklist) -> pd.DataFrame: ...
def _read_market_prices(db, horizon) -> pd.DataFrame: ...
def _aggregate_by_lane(df_pnl, df_dec) -> dict: ...
def _aggregate_by_label(df_pnl, df_dec) -> dict: ...
def _detect_win_patterns(agg, min_sample) -> list: ...
def _detect_loss_patterns(agg, min_sample) -> list: ...
def _propose_lane_promotion(agg, threshold) -> list: ...
def _propose_lane_demotion(agg, threshold) -> list: ...
def _generate_ml_feature_candidates(df) -> dict: ...
def _render_markdown_report(...) -> str: ...
def _output_json(...) -> str: ...
def _emit_to_REPORT_R1(summary) -> None: ...
def _emit_to_DASH_R1(summary) -> None: ...  # 将来
def main() -> int: ...
```

### exception hierarchy

```python
class AfterR1Error(Exception): ...
class AfterR1RunWindowError(AfterR1Error): ...
class AfterR1DataInsufficientError(AfterR1Error): ...
class AfterR1WriteRefused(AfterR1Error): ...
```

### main() flow (= pseudo)

```
1. _check_run_window (= 後場終了確認)
2. _read_advisory_data + _read_paper_pnl + _read_market_prices (= read-only)
3. _aggregate_by_lane / _aggregate_by_label
4. _detect_win_patterns / _detect_loss_patterns
5. _propose_lane_promotion / _propose_lane_demotion
6. _generate_ml_feature_candidates
7. _render_markdown_report / _output_json
8. _emit_to_REPORT_R1 (= file 経由連携)
```

### exit code

- 0: run OK
- 1: arg error
- 2: data insufficient
- 3: safety guard violation
- 4: write attempted (= 想定外)

### completion report json format

```json
{
  "run_at": "ISO8601",
  "horizon": "30d",
  "summary": {
    "win_count": N, "loss_count": M,
    "promotion_candidates": [...],
    "demotion_candidates": [...]
  },
  "data_sources": {"advisory_decisions": N_rows, ...},
  "filters": {"blacklist": ["w17-3-smoke"]},
  "safety": {"db_write_count": 0, "line_send_count": 0}
}
```

## 5. 実行タイミング (= 設計のみ、cron 登録は別 wave)

| 時間帯 | 用途 | 想定 plist |
|---|---|---|
| 15:30 JST | 後場直後 quick check | (Phase 2 以降) |
| 23:00 JST | 夜間バッチ (= 主) | jp.fire.after-r1-night.plist |
| 翌 08:00 JST | Advisory 直前 hint | (Phase 3 以降) |

cron / launchd 登録は本 wave + 当面の wave で **行わない**。

## 6. Lane / Pattern 評価設計 (= L1a/L1b 反映)

### 集約 dimension

- **label** 別 (= F062-R5.7 4 段階結論: 積極的買い / 条件付き買い / 場中監視 / 見送り)
- **sector** 別 (= 銘柄 sector metadata)
- **regime** 別 (= 市場 regime、F110 連携)
- **horizon** 別 (= 7d / 30d / 90d)
- **source_version** 別 (= advisory_decisions の source_version、smoke 除外)

### metric

- win_rate (= positive PnL count / total)
- avg_return
- max_drawdown
- sample_size
- sharpe-like (= avg / std)
- skew

### Lane 昇格 / 降格判断 (= 候補生成、自動適用しない)

- **昇格候補**: win_rate > threshold + sample_size > min + sharpe > 0
- **降格候補**: max_drawdown > threshold or negative skew or win_rate < min
- 候補のみ出力、実 lane 変更は **LANE-R2 (= 別 module、将来) 経由 HQ approve**

## 7. ML 前段 feature 設計 (= L1b 反映)

### 本 wave + Phase 1/2 では **ML 本実装しない**

ML feature 候補だけ整理。本実装は将来 (= F286-ML-R1 候補)。

### feature 候補 (= 集約済みデータから)

- lane / label / sector / regime (= categorical)
- advisory_at_hour (= 14:30 / 14:45 / etc)
- material_type (= 材料種別、F101 経由)
- volume_z (= 出来高 z-score)
- price_change_pct (= 当日変化率)
- max_drawdown (= window 内)
- sample_size_in_lane (= 同 lane 過去 sample)

### 過学習防止

- `--min-sample-size` (default 10) 未満は除外
- walk-forward 評価 (= train / test 時系列分離) を **設計に含めるが本 wave で impl しない**
- feature engineering は手作業で開始 (= ML black box 化を防ぐ)

## 8. smoke plan (= L2b 反映)

### Step A: read-only smoke (= 別 wave、HQ approve 後)

- 既存 production DB / staging DB から read のみ
- 出力は reports/after_r1/smoke/ (= 別 dir、混在防止)
- F282 production 不変保証
- 本 wave (= 設計のみ) では実行 0

### Step B: staging-only write smoke (= 別 wave、HQ_APPROVE_AFTER_R1_STAGING_WRITE=1 必須)

- staging.db への write (= 新規 table or 既存 row insert)
- 三段ガード (= db_label / basename / 既存 inode refuse)

### Step C: production read-only confirm

- production.db は **read のみ**
- never write

### smoke filter

- `SMOKE_SOURCE_VERSIONS_BLACKLIST = ("w17-3-smoke",)` 再利用
- W30 snapshot 由来は読まない

### 想定 smoke wave (= Wave 36 候補)

AFTER-R1 read-only smoke (= production read のみ、reports/after_r1/smoke/
に出力、HQ approve marker 必須)。

## 9. safety 設計 (= L4 反映)

### 本 wave (= 設計のみ) 安全

- 実 DB write 0
- 実 LINE 送信 0
- token / channel_token / secret 参照 0
- cron / launchd 登録変更 0
- F282 試走干渉 0 (= source 共通だが read-only)

### 別 wave (= impl 時) 安全制約

- default --dry-run (= write 経路を default では取らない)
- write は HQ_APPROVE_AFTER_R1_WRITE=1 marker 必須
- F282 試走中 (= 5/16-5/19) は AFTER-R1 launchd 登録しない (= 並走干渉 risk 防止)
- write 経路で三段ガード (= db_label / basename / inode)
- LINE は default False、HQ_APPROVE_AFTER_R1_LINE=1 必須

### F282 試走に干渉しない設計

- F282 source: production fire.db (= read-only 接続)
- AFTER-R1 source: production fire.db (= read-only 接続) または staging
- 両者 source 共通だが **書込み 0** で論理干渉なし
- 物理 lock も SQLite WAL で並列 read 可、書込み 0 でロック不発

## 10. 関連 TODO / phase 連携 (= L1b 反映)

| TODO | 関係 | AFTER-R1 への入出力 |
|---|---|---|
| F286-SIM-R1 | 兄弟 (= Simulation 専門) | SIM-R1 結果を AFTER-R1 read |
| F286-LANE-R0 | 入力 (= Lane 定義) | Lane metadata read |
| F286-LANE-R1 | 兄弟 (= Lane 評価詳細) | LANE-R1 結果集約 |
| F286-LANE-R2 | (= 昇格降格自動化) | AFTER-R1 が候補生成、LANE-R2 が判断 |
| F286-REPORT-R1 | 出力先 | summary を 1 section 連携 |
| F286-DASH-R1 | 出力先 (= 将来) | json summary 連携 |
| F286-RISK-R1 | 入力 (= risk metric) | drawdown 等を RISK 渡し |
| F286-ORDER-R1 | 兄弟 (= 注文、将来) | 改善 hints 反映 |
| F101 | 入力 (= 材料、修正後) | material_type feature 補完 |
| F282 | 共通 source | read-only 接続のみ |

## 11. L4 audit verdict (= W35-pre W34-4 反映)

| 観点 | verdict |
|---|---|
| A. F282 試走干渉 0 | PASS |
| B. read-only 安全保証 (= mode=ro + query_only) | PASS |
| C. smoke filter (= w17-3-smoke blacklist) 確実適用 | PASS |
| D. min_sample_size 過学習防止 | PASS |
| E. ML feature 「本実装ではない」明示 | PASS |
| F. write path (= 将来) 三段ガード + HQ marker | PASS |
| G. REPORT-R1 / DASH-R1 / LANE-R1/R2 責務分離 | PASS |
| H. /goal 停止条件 13 件 該当なし | PASS |

CRITICAL 0 / HIGH 0、本 wave 設計範囲で full PASS。

## 12. test plan (= L2a 反映、別 wave 用)

別 wave (= AFTER-R1 impl 時) で 30-50 件 test 追加想定:

- TestArgValidation / TestRunWindowCheck / TestReadOnlyEnforcement
- TestAggregateByLane / TestAggregateByLabel
- TestWinPatternDetection / TestLossPatternDetection
- TestLanePromotionCandidates / TestLaneDemotionCandidates
- TestMlFeatureCandidates / TestMarkdownReportRendering / TestJsonOutput
- TestSmokeRowFiltering / TestEmptyDataHandling
- TestMinSampleSize / TestDryRunDefault

期待: 4,090 baseline + 30-50 = 4,120-4,140。

## 13. 本 wave 安全要件 (= 設計 doc 自体)

- 実 DB write 0 / 実 LINE 0 / 実 API 0
- 実 token 参照 0
- F282 試走干渉 0 ✓
- cron / launchd 登録変更 0
- code 修正 0 (= vault doc のみ)

## 14. 想定 next wave (= Wave 36+ 候補)

| Wave | 内容 | 必要 HQ approve |
|---|---|---|
| 36 候補 | AFTER-R1 read-only smoke (= production read のみ、reports/after_r1/smoke/) | HQ_APPROVE_AFTER_R1_READ=1 |
| 37 候補 | AFTER-R1 runner impl + test (30-50 件) | impl 着手承認 |
| 38 候補 | staging write smoke | HQ_APPROVE_AFTER_R1_STAGING_WRITE=1 |
| 39 候補 | launchd 登録 (= 夜間 23:00 自動実行) | HQ_APPROVE_AFTER_R1_LAUNCHD=1 |
| 40+ | Phase 2 拡張 (= Paper Live 再計算 / Replay / Simulation) | 各段階 HQ approve |

## 15. 関連リンク

- [[F282_weekly_snapshot_trial_monitoring_2026-05-12|F282 試走監視 (W34)]]
- [[F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[F286_PNL_R3_paper_pnl_simulator_hook_2026-05-11|PNL-R3 (= paper_pnl 既存)]]
- [[F286_REPORT_R1_daily_pnl_report_generator_2026-05-11|REPORT-R1]]
- [[F285_Research_Lane_requirements_and_spec_2026-05-08|F285 Research Lane]]
- [[../02_todo/FIRE_CODEX_R1_WAVE35_PRE_plan|Wave 35-pre plan]]
