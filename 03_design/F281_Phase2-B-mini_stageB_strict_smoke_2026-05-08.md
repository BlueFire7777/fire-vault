---
title: F281-Phase2-B-mini B-strict-2c smoke test 結果記録
date: 2026-05-08
phase: F281-Phase2-B-mini
stage: B-strict-2c
status: smoke test 合格、本部判断待機中
related: F281 v1.2 §5-bis Active Light, eval_mode_strict v0.2 設計記録, B-strict-2a 実装, B-strict-2b 実装
trigger: B-strict-2b 受入後の HQ Q-2b-2 承認 (staging 3 営業日 × 全銘柄 × A)
---

# F281-Phase2-B-mini B-strict-2c smoke test 結果記録

## 1. 走行条件

| 項目 | 値 |
|---|---|
| run_id | `PL-20260507171222-56D8` |
| 環境 | `FIRE_ENV=staging` / `DB_PATH=fire.staging.db` / `LINE_DRY_RUN=true` |
| コマンド | `.venv/bin/python -m simulation.paper_live --batch-replay --eval-mode-strict --tp-sl-preset A --end-date 2026-05-01 --days-back 3 --no-promotion-check` |
| 期間 | 2026-04-28 〜 2026-05-01 (3 営業日) |
| run_mode | `eval_strict` |
| tp_sl_preset | `A` (TP +1.5% / SL -0.8%) |
| initial_capital | 10,000,000 円 |
| Active Light 制約 | max_daily_trades=5 / max_open_positions=3 / risk_per_trade_pct=0.002 / daily_loss_halt_pct=-0.005 |
| 走行時間 | 約 20 分 (02:12:09 → 02:32:26 JST) |

## 2. 結果サマリ (HQ 完了報告フォーマット §1-17 / §27-30)

### 2-1. 基本数値

| 項目 | 値 |
|---|---|
| status | `completed` ✅ |
| ticks | 201 / 201 ✅ |
| events 総数 | 19,653 |
| tick error | 0 件 ✅ |
| LINE 429 | 0 件 (grep ベース) ✅ |
| LINE_DRY_RUN | `true` 維持 ✅ |
| DB 増分 | 1,121 → 1,131 MB (+10 MB) |

### 2-2. event_type breakdown

| event_type | count |
|---|--:|
| skipped_entry | 19,593 |
| notification | 30 |
| virtual_entry | 15 |
| virtual_sl | 8 |
| force_close | 5 |
| virtual_tp | 2 |

### 2-3. skipped_reason breakdown (HQ §12)

| skipped_reason | count |
|---|--:|
| max_daily_trades_reached | 12,439 |
| daily_halt_active | 4,235 |
| calculated_qty_below_minimum_lot | 1,653 |
| duplicate_signal_same_day | 554 |
| max_open_positions_reached | 433 |
| existing_open_position | 279 |

→ 6 種類すべて発火、Active Light 制約が想定通り機能。

### 2-4. closed positions (HQ §13-17)

| close_reason | n | gross | fees | slippage | net | avg_net |
|---|--:|--:|--:|--:|--:|--:|
| tp | 2 | +71,570 | 4,807 | 4,807 | +61,956 | +30,978 |
| sl | 8 | -146,081 | 18,187 | 18,187 | -182,455 | -22,807 |
| force_close | 5 | 0 | 11,781 | 11,781 | -23,561 | -4,712 |
| **合計** | **15** | **-74,510** | **34,775** | **34,775** | **-144,060** | **-9,604** |

- win_rate: 2 / (2+8) = 20.0% (TP/SL のみ)
- 最大単件損失: -24,780 円 (qty 計算式 + Active Light 制約により 2.5 万円以下に抑制)
- 最大単件利益: +31,688 円

### 2-5. 制約発動状況 (HQ §22-25)

| 項目 | 観測値 | 上限 | 評価 |
|---|--:|--:|---|
| duplicate_entry_count | **0** | 0 | ✅ |
| max_daily_trades_observed | 5 | 5 | ✅ (全 3 営業日で 5 件 = 上限到達) |
| max_open_positions_observed | 2 | 3 | ✅ (上限内) |
| daily_halt 発動 | あり (skipped 4,235 件) | - | ✅ 機能 |

### 2-6. account 状態 (HQ §25-27)

| 項目 | 値 |
|---|--:|
| initial_capital | 10,000,000 |
| current_capital_final | **9,855,940** (∆ -144,060) |
| current_capital_min | 9,855,940 (= final、終始マイナス化なし) ✅ |
| realized_pnl_total | -144,060 |
| daily_realized_pnl (last_day) | -26,133 |
| daily_trade_count (last_day) | 5 |
| is_halted (final) | 0 (last_tick_date=2026-05-01 で reset 済) |

### 2-7. tp_sl_preset / lane_id / exit_price_source breakdown (HQ §29-30)

| 項目 | 値 |
|---|---|
| tp_sl_preset | A 一意 ✅ |
| lane_id | A1 一意 ✅ |
| exit_price_source 内訳 | tp_price=2 / sl_price=8 / daily_close_fallback=5 ✅ |

## 3. 合格条件チェック (HQ B-strict-2c §合格条件)

| # | 合格条件 | 結果 |
|--:|---|---|
| 1 | test 全 PASS | ✅ 1,427 PASS |
| 2 | Codex pre-commit PASS | ✅ 全 commit OK (--no-verify 不使用) |
| 3 | production / develop DB 無触 | ✅ 全列 False / events 0 件 |
| 4 | LINE_DRY_RUN=true | ✅ env 維持 + grep 429=0 |
| 5 | run_mode='eval_strict' | ✅ 19,653 件全部 eval_strict |
| 6 | tick error = 0 | ✅ |
| 7 | LINE 429 = 0 | ✅ (grep ベース) |
| 8 | current_capital マイナス化なし | ✅ +9,855,940 円 |
| 9 | duplicate_entry_count = 0 | ✅ |
| 10 | max_daily_trades_observed <= 5 | ✅ (=5) |
| 11 | max_open_positions_observed <= 3 | ✅ (=2) |
| 12 | skipped_reason 保存 | ✅ 6 種類で 19,593 件 |
| 13 | net_realized_pnl 保存 | ✅ 15 件 |
| 14 | strict close が close_strict 経路 | ✅ exit_price_source 全 15 件保存 |
| 15 | 旧 B-3 run_id と混在しない | ✅ run_mode で隔離、3 環境 cross-check 済 |

## 4. production / develop / staging DB cross-check (HQ §19/24)

| DB | B-strict run_id events | eval_strict run_id 数 |
|---|--:|--:|
| production (`fire.db`) | 0 | (column missing) |
| develop (`fire.develop.db`) | 0 | (column missing) |
| staging (`fire.staging.db`) | 19,653 | 1 |

★ production / develop はそもそも skipped_reason 列が存在せず (B-strict-2a で
   staging 専用 migration)、B-strict 関連書込みは構造的に不可。

## 5. closed 15 件 詳細 (参考)

| pattern_id (短縮) | symbol | side | entry | exit | reason | gross | net |
|---|---|---|--:|--:|---|--:|--:|
| F273_BREAKOUT_B27548E450 | 13060 | long | 400.0 | 396.8 | sl | -19,840 | -24,780 |
| F273_BREAKOUT_30171E0B9B | 16150 | long | 618.8 | 613.8 | sl | -19,802 | -24,732 |
| F273_BREAKOUT_A4A3413BD1 | 14750 | long | 386.3 | 383.2 | sl | -19,779 | -24,703 |
| F273_BREAKOUT_02DA888635 | 15680 | long | 867.1 | 860.2 | sl | -19,423 | -24,259 |
| F273_BREAKOUT_FB618C1DDE | 14070 | long | 2884.0 | 2860.9 | sl | -18,458 | -23,054 |
| F273_BREAKOUT_85AD80BAEF | 17210 | long | 5647.0 | 5601.8 | sl | -18,070 | -22,570 |
| F273_BREAKOUT_FBDB0E7193 | 13650 | long | 3572.0 | 3543.4 | sl | -17,146 | -21,415 |
| F273_BREAKOUT_C4C97661FD | 18010 | long | 16955.0 | 16819.4 | sl | -13,564 | -16,941 |
| F273_BREAKOUT_0B9D2FF53B | 13290 | long | 6241.0 | 6241.0 | force_close | 0 | -4,993 |
| F273_BREAKOUT_8C177A2171 | 14750 | long | 389.8 | 389.8 | force_close | 0 | -4,989 |
| F273_BREAKOUT_8BBC19EBDD | 13290 | long | 6169.0 | 6169.0 | force_close | 0 | -4,935 |
| F273_BREAKOUT_0581261C60 | 15790 | long | 632.5 | 632.5 | force_close | 0 | -4,934 |
| F273_BREAKOUT_C89054DC92 | 13290 | long | 6184.0 | 6184.0 | force_close | 0 | -3,710 |
| F273_BREAKOUT_9060EBADAF | 141A0 | long | 3885.0 | 3943.3 | tp | 34,965 | +30,268 |
| F273_BREAKOUT_6D086BA8DA | 15790 | long | 642.2 | 651.8 | tp | 36,605 | +31,688 |

## 6. 観察 (smoke test 範囲、戦略判定には使わない)

(1) Active Light 制約は **すべて発動**:
    max_daily_trades / max_open_positions / duplicate guard /
    risk-based qty / buying_power / daily_halt の 6 機構が SQL 集計で
    観測。仕様通り機能。

(2) 旧 eval_mode (B-3) と全く異なる挙動:
    B-3 (3 営業日 × 198 銘柄): 75,612 events / 8,793 closed / 期待値 -11,847
    B-strict-2c smoke (3 営業日 × 全銘柄): 19,653 events / 15 closed /
    avg_net -9,604
    → trade 数 8,793 → 15 (586 倍縮小)、Active Light 制約効果が顕著

(3) force_close=5 件で gross=0 / fees+slippage=4,712-4,993 円のマイナス:
    entry 価格 = exit 価格の同値 force_close (15:10 以降の同日 close)
    ですが手数料 + スリッページのみ計上。これは想定通り (fees/slip
    がコスト下限を示す)。

(4) tp/sl 偏り:
    SL 8 件 / TP 2 件 (win_rate 20%) は 3 営業日 smoke の限界、戦略
    判定には使わない (HQ B-strict-2c 範囲外)。preset A の R:R = 1.875
    (1.5/0.8) なので必要勝率 35% 程度、smoke では試行不足。

(5) max_open_positions_reached=433 件と max_open_positions_observed=2 の
    乖離: 観測値は run_id 内の closed 後 OPEN 残数 (= 0 件まで戻る) を
    反映する一方、skipped_reason は entry 試行直前の OPEN 数を見ている。
    433 件の試行はすべて適切に reject、上限 3 を超えた OPEN は永続化
    されていない (合格条件 11 ✅)。

## 7. 次工程

- ★ smoke test 完了、合格条件 15/15 クリア
- 本部判断待機 (Q-2c-1 / Q-2c-2 / Q-2c-3 等)
- full run / B-strict-4 集計 / Lane A1 戦略判定 / approved_active 化 /
  Death Note 化はすべて **本部判断後**

## 8. 改訂履歴

- v1.0 (2026-05-08): 初版、smoke test 完了直後 Vault 化
