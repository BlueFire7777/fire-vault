---
title: F281 Lane C Phase C2 strict 60 営業日評価 結果 (C2-7)
date: 2026-05-09
phase: F281-Phase-C2 / C2-7 strict evaluation
status: 全 preset Stage gate FAIL、Phase C2 真評価結果、HQ 判断要請 (C2-8 進まず、改善候補要検討)
related: F281_Lane_C_Phase_C2_implementation_plan_2026-05-09, F281_Lane_C_Phase_C2_smoke_result_2026-05-09, F284_F105_c6_final_result_2026-05-09
trigger: HQ C2-7 着手指示 (2026-05-09、C2-6 smoke + force_close 修正 a1b3347 後)
---

# F281 Lane C Phase C2 strict 60 営業日評価 結果 (C2-7)

★ HQ Stage gate 8 基準で評価、**全 preset (A/B/C/D) FAIL**。
   preset C が最良 (gross_pnl -3,206 / PF 0.998、ほぼ break-even)
   だが基準未達。C2-8 (tick/order template 統合) には進まず、
   改善候補を提示して HQ 判断を仰ぐ。

## 1. 評価情報

| 項目 | 値 |
|---|---|
| 実施日時 | 2026-05-09 09:19:06 開始 〜 09:20:34 終了 |
| 走行時間 | **88.0 sec (= 1.47 min)** |
| 対象 commit (~/fire) | a1b3347 (force_close 修正) + C2-1〜C2-5 全 commit |
| 対象 commit (vault) | dea61d4 (Phase C2 plan v1.0) / 9b0f13f (smoke v1.1) |
| 実行環境 | Mac mini、FIRE_ENV=staging、~/fire/.venv/bin/python |
| DB | /Users/bluefire/fire/data/fire.staging.db (read-only URI) |

## 2. 評価条件

| 項目 | 値 |
|---|---|
| universe | Tier2 478 codes (load_tier2_universe) |
| 72030 除外 | True (excluded_codes=('72030',)) |
| 期間 | **2026-02-03 〜 2026-05-01** |
| 営業日数 | **60** (休場日 4 日 = 02-11/02-23/03-20/04-29 除外) |
| preset | A / B / C / D (E ATR 連動は後回し) |
| max_daily_trades | 5 (Active Light) |
| initial_capital_jpy | 10,000,000 |
| notional_per_trade_jpy | 1,000,000 |
| daily_halt_threshold_jpy | -50,000 |
| 評価規模 | candidate 30,592 = 478 × 64 (no_data 1,913 含む) / preset 4 = trade 評価 1,200 |

## 3. exit_time <= 15:10 検査 (HQ preflight 必須)

| 項目 | 値 |
|---|---|
| total trades | 1,200 (= 300 × 4 preset) |
| max exit_time | **15:10** |
| violations (> 15:10) | **0** |

★ **全 exit_time が 15:10 以下、HQ 仕様準拠 ✅** ★
(commit a1b3347 force_close 修正で全 trade 担保)

## 4. preset 別 summary

| 項目 | A | B | C | D |
|---|--:|--:|--:|--:|
| total_candidates | 30,592 | 30,592 | 30,592 | 30,592 |
| setup_detected_count | 25,838 | 25,838 | 25,838 | 25,838 |
| **trade_count** | 300 | 300 | 300 | 300 |
| win_count | 119 | 118 | 125 | 108 |
| loss_count | 181 | 182 | 174 | 191 |
| force_close_count | 4 | 12 | **33** | **47** |
| **win_rate** | 0.3967 | 0.3933 | **0.4167** | 0.3600 |
| **gross_pnl_jpy** | **-146,159** | -90,751 | **-3,206** | -110,453 |
| avg_gross_pnl_per_trade | -487.2 | -302.5 | **-10.7** | -368.2 |
| gross_profit | +934,980 | +1,349,469 | +1,662,223 | +1,710,947 |
| gross_loss | -1,081,139 | -1,440,220 | -1,665,429 | -1,821,401 |
| **profit_factor** | 0.8648 | 0.9370 | **0.9981** | 0.9394 |
| max_drawdown_jpy | 227,729 | 191,738 | **150,930** | 253,612 |
| max_drawdown_pct | 0.022615 | 0.019012 | **0.014924** | 0.025116 |
| daily_halt_count | 0 | 0 | 0 | 0 |
| duplicate_entry_count | 0 | 0 | 0 | 0 |
| no_negative_capital_flag | True | True | True | True |

★ **preset C が最良** (gross_pnl -3,206 円、PF 0.998、ほぼ break-even) ★

## 5. skipped_reason 集計 (全 preset 共通、setup_judge + extractor)

| reason | 件数 | 割合 |
|---|--:|--:|
| no_recovery_or_breakout | 1,974 | 41.5% |
| no_intraday_data | 1,913 | 40.2% (= 4 休場日 + 1 件) |
| no_pullback | 461 | 9.7% |
| outside_entry_time_window | 357 | 7.5% |
| insufficient_bars | 49 | 1.0% |
| **計** | **4,754** | 100.0% |

setup_detected_count + 計 = 25,838 + 4,754 = 30,592 ✅ candidates 整合

注目:
- **no_recovery_or_breakout 1,974**: state 2 (pullback 成立) 後に
  VWAP 回復も prior 再ブレイクも未達で skip → **setup の打率を制約**
- **outside_entry_time_window 357**: 14:45 cutoff 以降の entry 弾かれ

## 6. Stage gate 判定 (HQ 8 基準、全 preset FAIL)

| 基準 | 期待 | A | B | C | D |
|---|---|:-:|:-:|:-:|:-:|
| gross_pnl_jpy >= +300,000 | 利益 30 万円 | ❌ -146,159 | ❌ -90,751 | ❌ -3,206 | ❌ -110,453 |
| avg_per_trade >= +3,000 | 利益期待値 | ❌ -487 | ❌ -302 | ❌ -11 | ❌ -368 |
| PF >= 1.15 | 利益効率 | ❌ 0.86 | ❌ 0.94 | ❌ 0.998 | ❌ 0.94 |
| max_drawdown_pct <= 5% | drawdown | ✅ 2.3% | ✅ 1.9% | ✅ 1.5% | ✅ 2.5% |
| daily_halt <= 10 | halt 頻度 | ✅ 0 | ✅ 0 | ✅ 0 | ✅ 0 |
| trade_count >= 100 | 統計的有意性 | ✅ 300 | ✅ 300 | ✅ 300 | ✅ 300 |
| duplicate_entry_count == 0 | 重複なし | ✅ 0 | ✅ 0 | ✅ 0 | ✅ 0 |
| no_negative_capital_flag == True | 元本維持 | ✅ True | ✅ True | ✅ True | ✅ True |
| **総合判定** | | **❌ FAIL** | **❌ FAIL** | **❌ FAIL** | **❌ FAIL** |

★ **全 preset FAIL、C2-8 (tick/order template 統合) には進めない** ★

利益系 3 基準 (gross_pnl / avg_per_trade / PF) で全 preset FAIL、安全系
4 基準 (drawdown / halt / duplicate / no_negative_capital) は全 PASS。
trade_count 300 は max_daily_trades=5 × 60 営業日に対して 100% 採用、
統計的有意性は確保。

## 7. 理由分析

### 7.1 利益が出ない構造

1. **win_rate 36-42% (低)**: setup の打率不足
   - VWAP 回復 / prior 再ブレイク後の trade で 60% 近くが loss
   - setup 確認後も価格が反転する確率が高い (= momentum が短時間で消失)

2. **gross_loss が gross_profit を上回る** (preset A 以外)
   - preset B: profit +1,349,469 / loss -1,440,220 (差 -90,751)
   - preset C: profit +1,662,223 / loss -1,665,429 (差 -3,206)
   - 利益の幅 vs 損失の幅 がほぼ拮抗、win_rate 差で gross_pnl が
     決まる構造

3. **force_close 増加で取りこぼし** (preset C/D で顕著)
   - preset A: 4 件 / B: 12 件 / C: **33 件** / D: **47 件**
   - TP を広げる (1.5%/2.0%) と「TP 未達 + 当日 exit 強制」で利益
     確定機会を逃す

4. **持ち越し禁止 = エッジ希薄化**
   - intraday only で 5 時間以内に決着しない trend は force_close
   - 翌日まで持ち越せれば +5% / -2% 等を取れる場面でも当日決着強制

### 7.2 setup_judge 自体は機能している

- **state machine 動作正常**: state 0 → 1 → 2 遷移は smoke + strict
  両方で確認、no_recovery_or_breakout が 41.5% で「pullback 後に recovery
  しない pattern」を識別できている
- 25,838 / 30,592 = **84.5% setup_detected** で抽出は十分動作
- 問題は「setup 後に勝てる trade に絞れるか」 = 確度の calibration 必要

### 7.3 preset 比較から見える示唆

| preset | TP/SL | force | gross_pnl | 観察 |
|---|---|--:|--:|---|
| A | 0.8/0.6 | 4 | -146,159 | 最も TP/SL タイト、force 少ないが loss 主導 |
| B | 1.2/0.8 | 12 | -90,751 | 標準、A より改善 |
| C | 1.5/1.0 | 33 | **-3,206** | 最良、TP 拡大で勝率↑ + 力士損失↓ |
| D | 2.0/1.0 | 47 | -110,453 | TP 大きすぎで force 多発、損失拡大 |

→ **TP 1.0% 〜 1.5% / SL 0.8% 〜 1.0% 圏が optimum 候補**
→ ATR 連動 (preset E) を試す価値あり (= volatility 適応で C の改善余地)

## 8. 改善候補 (HQ 判断要請)

### 8.1 setup 確度を上げる方向

1. **PULLBACK_THRESHOLD 引き上げ** (0.5% → 0.7-1.0%): 浅い pullback を
   弾いて確度の高い setup のみ採用
2. **EPS_VWAP / EPS_BREAK 引き上げ** (0.1% → 0.2-0.3%): ノイズ閾値拡大
3. **出来高継続条件強化** (0.7 → 1.0 倍 or sma60 比較): 出来高弱い
   setup 弾く
4. **過熱除外閾値引き下げ** (10% → 5-7%): 高騰銘柄の遅れ entry 排除

### 8.2 universe / 時刻フィルタ

5. **Tier2 内の流動性別サブセット**: turnover 50 億超のみで再評価 (=
   流動性の確実な銘柄に絞る)
6. **entry_time cutoff 前倒し** (14:45 → 13:30 / 14:00): trend 残時間
   を確保、force_close 削減

### 8.3 preset / TP/SL 調整

7. **preset E (ATR 連動) 実装**: volatility 適応で C の利益確定機会
   増 (HQ 補正 3 で「後回し可」とした E の実装着手)
8. **TP/SL 範囲再検討**: preset C 周辺 (1.0-1.5% / 0.8-1.0%) を細分化、
   smoke + 部分 strict で再 calibration

### 8.4 サンプル拡張

9. **対象期間拡張**: c6 backfill 完了済の 60 営業日 + 過去 60 営業日
   (要 c6 期間延長 backfill) で 120 営業日評価
10. **業種 (sector) 別評価**: skipped_reason / win_rate を業種別に分解、
    機能する業種の特定

### 8.5 戦略本体の見直し

11. **持ち越し許容化検討**: 翌日に flat → close する 1 day swing 化
    (HQ 仕様「持ち越し禁止」を変更する場合は要 HQ 判断)
12. **別 trigger 候補**: VWAP 回復 / prior 再ブレイク以外の entry
    pattern を追加 (例: bull flag、cup-and-handle)

## 9. C2-8 進む判定

★ **C2-8 (tick/order template 統合) には進めない** ★

理由: HQ Stage gate 8 基準のうち利益系 3 基準 (gross_pnl / avg_per_trade /
PF) で全 preset FAIL。HQ 仕様 v1.0 §11「C2-8 は strict 評価が基準を満た
した後の判断」に従い、本格 Live 接続には進まない。

次工程: HQ 判断要請 (= 改善候補から方針選定)。

## 10. 制約遵守確認

| 制約 | 結果 |
|---|---|
| daily 近似禁止 | ✅ intraday 1 分足 + vwap_cumulative |
| TPM を VWAP 代替にしない | ✅ vwap_cumulative 直接使用 |
| 持ち越し禁止 | ✅ 全 trade 当日 exit (force_close 含む) |
| force_close 15:10 厳守 | ✅ 全 1,200 trades exit_time <= 15:10 |
| Tier2 478 codes 固定 | ✅ universe_loader、smoke 残 72030 除外 |
| market_prices_intraday 全体無条件参照禁止 | ✅ extract_candidates 経由 |
| FIRE_ENV=staging guard | ✅ 4 段 guard 通過 |
| production / develop DB 無触 | ✅ read-only URI、staging のみ |
| staging DB read-only | ✅ `file:?mode=ro` |
| 自動発注禁止 | ✅ 純粋評価、注文発行なし |
| 楽天証券操作自動化禁止 | ✅ |
| Computer Use 禁止 | ✅ |
| C2-8 tick/order template 統合に進まない | ✅ 本評価まで、Live 接続せず |

## 11. 関連 commit

~/fire 側 (本評価で動作した実装):
| commit | task |
|---|---|
| c94fb16 | C2-1 Tier2 universe loader |
| 6ce8f49 | C2-2 setup judge state machine |
| ca43370 | C2-3 candidate extraction |
| 22023a4 | C2-4 intraday entry/exit simulator |
| 94bc75f | C2-5 strict evaluator + preset comparison |
| **a1b3347** | force_close 15:10 厳守 (HQ preflight 修正) |

vault 側:
| commit | task |
|---|---|
| dea61d4 | C2-0 Phase C2 plan v1.0 |
| e777899 | C2-6 smoke result v1.0 |
| 9b0f13f | C2-6 smoke result v1.1 (force_close 修正反映) |
| 本 commit | C2-7 strict 60 営業日評価結果 Vault 化 |

## 12. 次工程提案

★ **C2-8 進まず、HQ 判断要請** ★

優先度高 (低コストで再評価可能):
- 案 A1 (§8.1 #1): PULLBACK_THRESHOLD を 0.5% → 0.8% に引き上げて再評価
- 案 A4 (§8.1 #4): 過熱除外閾値を 10% → 7% に引き下げて再評価
- 案 B6 (§8.2 #6): entry_time cutoff を 14:45 → 14:00 に前倒しして再評価

優先度中 (実装少々必要):
- 案 D7 (§8.3 #7): preset E (ATR 連動) 実装 + 評価
- 案 D8 (§8.3 #8): preset C 周辺 (1.0-1.5% / 0.8-1.0%) で細分化 calibration

優先度低 (大幅改修):
- 案 E11 (§8.5 #11): 持ち越し許容化検討 (HQ 仕様変更要)
- 案 E12 (§8.5 #12): 別 trigger pattern 追加

提案順序:
1. 優先度高 3 案を並行検討、HQ 判断
2. 採用案で setup 閾値再 calibration
3. 改善版 strict 60 営業日評価 → Stage gate 再判定
4. PASS → C2-8 着手判断 / FAIL → 次の改善候補

## 13. 改訂履歴

- v1.0 (2026-05-09): 初版、C2-7 strict 60 営業日評価完了直後の
  Vault 化、全 preset Stage gate FAIL、C2-8 進まず HQ 判断要請。
