---
title: F281 Lane C「前日強銘柄初押し戦略」設計記録 (HQ 確定版、intraday 前提)
date: 2026-05-08
phase: F281-Lane-C
status: Phase C0 (Intraday Feasibility & Universe Precheck) 設計確定、Phase C1 着手前
related: F281 v1.2 §8 Lane 構造, eval_mode_strict v0.2, B-strict full run, F104 (指数 daily) 別系列
trigger: HQ 判断 (Lane A1 broad 不採用後の narrow 戦略評価)、案 A daily 近似 MVP 不採用、intraday 前提で進む
---

# F281 Lane C「前日強銘柄初押し戦略」設計記録

★ 本設計は **intraday data 前提**。daily 近似 MVP (案 A) は HQ 不採用、
   持ち越し禁止ルールは緩和しない、TPM を VWAP 代替に使わない。

## 1. 戦略仮説

前日強かった銘柄のうち、翌日に初押しを作り、VWAP 回復または高値再
ブレイクで再加速する銘柄を狙う。margin 日計り、当日中 exit 厳守、
持ち越し禁止。

## 2. universe 定義 (HQ 初期閾値)

前日 t-1 close 時点で評価:

| 項目 | HQ 閾値 |
|---|---|
| 前日上昇率 | prev_change_pct >= +3% |
| ストップ高近辺除外 | prev_change_pct < +15% (= +15% 以上は除外候補) |
| 前日出来高増加率 | prev_volume / SMA20(volume) >= 1.5 |
| 前日売買代金 (3 段階比較) | turnover_value_jpy: 10 億 / 30 億 / 50 億 |
| t 日 open gap | prev_close 比 -2% 〜 +4% |
| 信用取引可 | margin_code != NULL **必須** |
| 流動性 | TOPIX Small 除外固定せず、流動性フィルタで代替 |

scripts/jobs/lane_c_universe_precheck.py で 60 営業日分の universe
size + 閾値感度を Phase C0 で実測予定。

## 3. setup 定義 (intraday 必須)

★ HQ 仕様: intraday 動きでの判定、daily 近似 (TPM = VWAP 代替) は
   不採用。

判定要素 (intraday data 必要):
1. **当日初押し**: 当日 intraday 高値到達後、当該 timestamp までに
   pullback_pct (= (intraday_high - current_close) / intraday_high)
   が一定値 (例 0.5% 〜 1%) 以上
2. **VWAP 回復**: current_close >= intraday_vwap_at_dt (累積 VWAP)
3. **高値再ブレイク**: current_close > intraday_high_at_dt ×
   (1 + ε)、ε=0.001 (0.1% 以上ブレイク) 等
4. **出来高継続**: 当該 timestamp までの累積 volume / SMA20(daily
   volume) >= 0.7
5. **過熱除外**: 当該時点で intraday_open 比上昇 >= 10% の場合は
   reject (乗り遅れ)

setup 充足 = 「universe pass + 初押し + (VWAP 回復 OR 高値再ブレイク)
 + 出来高継続 + 過熱でない」を満たす状態。

intraday VWAP の正確な計算:
  vwap_cumulative_at_dt = sum(typical_price × volume) / sum(volume)
                          (09:00 〜 当該 dt の累積)
  typical_price = (high + low + close) / 3

## 4. entry 定義

| 項目 | 値 |
|---|---|
| entry day | t 日 (前日 = t-1、当日 = t) |
| entry 時間帯 | 09:30 〜 14:00 (no_new_entry_after_hour=14 準拠) |
| entry trigger | setup 充足 timestamp で limit 指値発火 |
| entry price | 当該 5 min/15 min bar の close (intraday 価格) |
| entry_price_source | 'lane_c_pullback_intraday' |
| 成行 | **禁止** (HQ 指示)、必ず limit / trigger 相当で評価 |

## 5. exit 定義

| 項目 | 値 |
|---|---|
| TP/SL preset (HQ 案) | A baseline (TP +1.5% / SL -0.8%) / Lane C standard (TP +1.8% / SL -0.9%) / Lane C runner (TP +2.5% / SL -1.0%) |
| 時間切れ exit | t 日 15:10 force_close (HQ 「持ち越し禁止」厳守) |
| 持ち越し | **完全禁止** (margin 日計り前提) |
| daily_halt | 既存 -0.5% halt 機構流用 (entry 直前 gate) |
| close 経路 | strict close_strict (gross/fees/slippage/net 分離) 流用 |

intraday TP/SL ヒット判定:
  long TP: 当該以降 bar の high >= tp_price → TP 約定
  long SL: 当該以降 bar の low <= sl_price → SL 約定 (TP/SL 同 bar
            両ヒット時は SL 優先、保守判定)
  当日 15:10 まで未ヒット → force_close (15:10 直前 bar の close 価格)

## 6. strict 評価連携

eval_mode_strict 流用:
  - try_strict_entry (atomic transaction): 流用、lane_id='C1' 渡す
  - close_strict (gross/fees/slippage/net): 流用
  - account.maybe_reset_for_date: 流用
  - Active Light 制約 6 機構: 流用
  - SKIPPED_ENTRY 機構: 流用 + 新 reason 追加
  - StrictTickContext: lane_id 拡張

新規実装:
  - extract_lane_c_candidates (rule-based filter、intraday 評価)
    SimilarityEngine 経由ではない、intraday data + features 直接参照
  - tick.py / runner.py に lane 切替経路
  - cli.py に --lane C1 引数 + guard

新 SkippedReason (Lane C 専用):
  - universe_filter_failed
  - setup_signal_failed
  - gap_excessive
  - volume_insufficient
  - stop_high_near_excluded
  - intraday_overheated (過熱除外)

duplicate guard 整合性:
  trading_date × lane_id × pattern_id × symbol × side で判定 →
  Lane A1 / Lane C1 別々にカウント、混在なし ✅

## 7. stage-gate (HQ 確定値)

### 7-1. strict 60 営業日

  | 条件 | 閾値 |
  |---|---|
  | after-cost net_pnl | >= +300,000 円 |
  | avg_net_pnl/trade | >= +3,000 円 |
  | profit_factor | >= 1.15 |
  | maxDD | -5% 以内 |
  | daily_halt | <= 10 / 60 営業日 |
  | trade_count | >= 100 |
  | duplicate_entry | = 0 |
  | current_capital マイナス化 | なし |

### 7-2. Historical Paper Live 20 営業日

  | 条件 | 閾値 |
  |---|---|
  | after-cost net_pnl | プラス |
  | profit_factor | >= 1.10 |
  | maxDD | -3% 以内 |
  | daily_halt | <= 3 / 20 営業日 |
  | duplicate_entry | = 0 |
  | current_capital マイナス化 | なし |

不採用条件: いずれか未達で Lane C 不採用、Death Note 化判定 (本部判断)。

## 8. data 取得経路の現状 + 必要追加実装

### 8-1. 現状

  - **個別銘柄 intraday data 取得経路は fire repo に未実装**
  - market_data/historical.py: 「分足: J-Quants V2 client に対応 API
    無いため対象外 (Phase 2 検討)」
  - F104 = 指数 (TOPIX/日経) daily OHLC、Lane C 不適合
  - 既存 features の "intraday_range_pct" は daily 近似値 (= (high-low)/
    open)、真の intraday data ではない

### 8-2. 必要新規実装 (Phase C1)

  仮タスク名: **F105 個別銘柄 intraday data 取得**
  data source 候補:
    - J-Quants Premium プラン (月額有料、5 分足 / Tick 取得可)
    - 楽天証券 API (個人プラン、historical 制限あり)
    - その他商用 (TradingView 等)

  schema (新規):
    market_prices_intraday (上記 §3 + Phase C0 報告参照)

  migration: scripts/setup/migrate_intraday_columns.py 新規 (staging
              専用、B-strict-2a 4 段 guard 同パターン)

  fetch ジョブ: scripts/jobs/fetch_intraday_data.py 新規

## 9. 実装 Phase 分割

### Phase C0 (本タスク範囲、調査 + 設計 + universe precheck)

  - C0-1: 本設計記録 Vault 化 (本ファイル)
  - C0-2: intraday data feasibility 調査結果 Vault 化 (F104 関係 +
          table 案 + Phase 分割)
  - C0-3: scripts/jobs/lane_c_universe_precheck.py 新規 (daily 検証
          read-only)
  - C0-4: precheck 実行 + 結果 Vault 化 (universe size + 閾値感度)

### Phase C1 (intraday data 取得経路の確立、HQ 承認後)

  - C1-1: intraday data source 調査 (J-Quants Premium / 楽天証券 /
          商用 API、コスト + 制約)
  - C1-2: Fujiwara コスト判断 (商用 API 契約有無)
  - C1-3: market_data/intraday.py 新規実装
  - C1-4: scripts/setup/migrate_intraday_columns.py 新規 (staging 専用)
  - C1-5: scripts/jobs/fetch_intraday_data.py 新規 + test
  - C1-6: 60 営業日分 backfill (staging only)

### Phase C2 (Lane C 真実装、Phase C1 完了後)

  - C2-1: features ETL 拡張 (intraday VWAP / pullback / high rebreak
          計算ジョブ)
  - C2-2: simulation/paper_live/lane_c.py 新規 (extract_lane_c_candidates、
          intraday rule-based)
  - C2-3: tick.py / runner.py / cli.py に lane 切替経路
  - C2-4: scripts/setup/seed_lane_c_patterns.py + Lane C patterns 投入
  - C2-5: unit tests
  - C2-6: staging smoke test (3 営業日)
  - C2-7: full run (60 営業日 strict)
  - C2-8: B-strict-4 同型集計 + Vault 化、HQ §7-1 stage gate 照合

### Phase C3 (Historical Paper Live、Phase C2 通過後)

  - C3-1: 20 営業日 Historical Paper Live 走行
  - C3-2: HQ §7-2 stage gate 評価
  - C3-3: approved_active 化 / Death Note 化判定 (本部判断)

## 10. 制約 (HQ 厳守)

- 自動発注禁止
- 楽天証券操作自動化禁止
- Computer Use 禁止
- Playwright/Cron 強制クローズ禁止
- LINE 通知は注文完成形の提示まで
- 発注は Fujiwara 手動
- production / develop DB への不用意な変更禁止
- Codex pre-commit 必須
- --no-verify 禁止
- 個別 commit 厳守
- daily 近似 (TPM = VWAP 代替) 不採用
- 持ち越し禁止ルール緩和不可

## 11. 関連 Vault リンク

- [[F281_strategy_portfolio_design_2026-05-07|F281 戦略レーン設計]]
- [[F281_Phase2-B-mini_conclusion_2026-05-08|Phase2-B-mini 完了総括 (Lane A1 不採用)]]
- [[F281_Phase2-B-mini_stageB_strict_fullrun_2026-05-08|B-strict full run 結果]]
- [[F104_指数四本値取得|F104 (指数 daily、Lane C 不適合)]]
- [[F100_市場データAPI|F100 J-Quants V2 daily]]

## 12. 改訂履歴

- v1.0 (2026-05-08): 初版、HQ 確定版仕様 (intraday 前提、daily 近似
  不採用、持ち越し禁止厳守、stage gate 数値 8 項目 + 6 項目)
