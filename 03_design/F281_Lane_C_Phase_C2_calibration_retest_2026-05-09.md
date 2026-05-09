---
title: F281 Lane C Phase C2 C2-7.1 calibration retest 結果
date: 2026-05-09
phase: F281-Phase-C2 / C2-7.1 calibration retest
status: 30 パターン全 Stage gate FAIL、HQ 追加判断 3 基準すべて未達 → Lane C Phase C2 保留候補
related: F281_Lane_C_Phase_C2_strict_60days_result_2026-05-09, F281_Lane_C_Phase_C2_implementation_plan_2026-05-09, F281_Lane_C_design_2026-05-08
trigger: HQ C2-7.1 着手指示 (2026-05-09、C2-7 全 preset FAIL 後の calibration 検証)
---

# F281 Lane C Phase C2 C2-7.1 calibration retest 結果

★ HQ 指定 30 パターン (cutoff 3 × pullback 2 × preset 5) で grid 評価。
   全パターン Stage gate FAIL、HQ 追加判断 3 基準も全未達 →
   **Lane C Phase C2 を保留候補とし、HQ 判断要請**。

## 1. 走行情報

| 項目 | 値 |
|---|---|
| 実施日時 | 2026-05-09 09:40:21 開始 〜 09:48:50 終了 |
| 走行時間 | **508.5 sec (= 8.48 min)** |
| 対象 commit (~/fire) | 35f814f (calibration runner + 引数拡張) + a1b3347 (force_close fix) + C2-1〜C2-5 |
| 対象 commit (vault) | dea61d4 / 9b0f13f / 7001623 / 66dd985 |
| 実行環境 | Mac mini、FIRE_ENV=staging、~/fire/.venv/bin/python |
| DB | /Users/bluefire/fire/data/fire.staging.db (read-only URI) |
| evaluator | scripts/jobs/run_lane_c_calibration.py (新規) |

## 2. 評価条件

| 項目 | 値 |
|---|---|
| universe | Tier2 478 codes (72030 除外) |
| 期間 | 2026-02-03 〜 2026-05-01 (60 営業日) |
| max_daily_trades | 5 |
| initial_capital_jpy | 10,000,000 |
| notional_per_trade_jpy | 1,000,000 |
| daily_halt_threshold_jpy | -50,000 |

## 3. calibration grid

| 軸 | 値 |
|---|---|
| A. entry_time_cutoff | 14:45 (baseline) / 14:15 / 14:00 |
| B. pullback_threshold | 0.005 (baseline) / 0.008 |
| C. TP/SL preset (C 周辺 5 種) | C0 (1.0/0.8) / C1 (1.2/0.8) / C2 (1.5/0.8) / **C3 (1.5/1.0=baseline)** / C4 (1.8/1.0) |
| 合計 | **3 × 2 × 5 = 30 パターン** |

## 4. exit_time <= 15:10 検査 (HQ preflight 必須)

| 項目 | 値 |
|---|---|
| total trades | 9,000 (= 300 × 30 patterns) |
| **violations (> 15:10)** | **0** |

★ **全 9,000 trade で HQ 仕様準拠 ✅** ★ (commit a1b3347 force_close 修正で担保)

## 5. best 5 ranking (gross_pnl_jpy 降順)

| # | cutoff | pullback | preset | trades | gross_pnl | PF | WR | maxDD | gate |
|---|---|---|---|--:|--:|--:|--:|--:|:-:|
| 1 | 14:45 | 0.005 | **C3 (= baseline)** | 300 | **-3,206** | 0.998 | 41.7% | 1.49% | ❌ |
| 2 | 14:15 | 0.005 | C3 | 300 | -3,206 | 0.998 | 41.7% | 1.49% | ❌ |
| 3 | 14:00 | 0.005 | C3 | 300 | -3,206 | 0.998 | 41.7% | 1.49% | ❌ |
| 4 | 14:45 | 0.005 | C1 (1.2/0.8) | 300 | -90,751 | 0.937 | 39.3% | 1.90% | ❌ |
| 5 | 14:15 | 0.005 | C1 | 300 | -90,751 | 0.937 | 39.3% | 1.90% | ❌ |

★ **best 3 が baseline と完全同値**、cutoff 変化が結果に影響しない (§7 分析参照)
★ **pullback 0.008 は全 preset で悪化** (best 5 にも入らない)

## 6. baseline (cutoff 14:45 / pb 0.005 / preset C3) 詳細

| 項目 | 値 |
|---|--:|
| trade_count | 300 |
| win/loss/force_close | 125 / 174 / 33 |
| win_rate | 0.4167 |
| gross_pnl_jpy | -3,206 |
| avg_gross_pnl_per_trade | -10.7 |
| profit_factor | 0.998 |
| max_drawdown_pct | 1.49% |
| daily_halt | 0 / duplicate | 0 / no_negative_capital | True |
| Stage gate 全 PASS | False |

(C2-7 の preset C と完全一致、C2-7 結果が再現確認できた)

## 7. 全 30 パターン分析

### 7.1 cutoff (14:45 / 14:15 / 14:00) は結果に影響なし

| cutoff | preset C0 | C1 | C2 | C3 | C4 |
|---|--:|--:|--:|--:|--:|
| 14:45 (pb 0.005) | -146,897 | -90,751 | -126,575 | -3,206 | -123,749 |
| 14:15 (pb 0.005) | -146,897 | -90,751 | -126,575 | -3,206 | -123,749 |
| 14:00 (pb 0.005) | -146,897 | -90,751 | -126,575 | -3,206 | -123,749 |

cutoff を変えても全 preset で gross_pnl が完全一致。

**理由**: Lane C は寄付き直後から setup 形成、各日 setup_detected が
5 件以上ある日が多く、`max_daily_trades=5` で先に採用上限に達する。
14:00 以降の entry は既に max_daily_trades 制約で skip される (=
outside_entry_time_window で skip される前に max_daily_trades 制約で
別経路 skip)。

→ **cutoff 引数による改善は本評価では実質ゼロ**

### 7.2 pullback 0.008 で全 preset 悪化

| pullback | preset C0 | C1 | C2 | C3 | C4 |
|---|--:|--:|--:|--:|--:|
| 0.005 | -146,897 | -90,751 | -126,575 | **-3,206** | -123,749 |
| 0.008 | -206,864 | -205,827 | -176,120 | -153,480 | -367,764 |

pullback 0.005 → 0.008 で全 preset 大幅悪化:
- C3 : -3,206 → **-153,480** (-150,274)
- C0 : -146,897 → -206,864 (-59,967)
- C4 : -123,749 → **-367,764** (-244,015) 最悪

**理由**: pullback 0.8% で「浅い pullback の trade」を捨てると、
本来勝てた trade を取り逃し、setup 件数も減って統計が悪化。
浅い pullback でも勝率自体は悪くなく、捨てるとマイナス side が拡大。

→ **pullback 引き上げは逆効果**

### 7.3 preset C3 が最良 (0.005 only)

| preset | gross_pnl (pb 0.005) | PF | WR |
|---|--:|--:|--:|
| C0 (1.0/0.8) | -146,897 | 0.893 | 42.0% |
| C1 (1.2/0.8) | -90,751 | 0.937 | 39.3% |
| C2 (1.5/0.8) | -126,575 | 0.918 | 35.0% |
| **C3 (1.5/1.0)** | **-3,206** | **0.998** | 41.7% |
| C4 (1.8/1.0) | -123,749 | 0.931 | 37.0% |

C3 (TP 1.5% / SL 1.0%) が最良。SL を 0.8% に下げる (C0/C1/C2) と SL hit
率が上がり gross_loss 拡大、TP を 1.8% に上げる (C4) と TP 達成率が
下がり force_close 増 → どちらも C3 に劣る。

## 8. Stage gate 判定 (HQ 8 基準)

★ **30 / 30 パターン FAIL (PASS 0)** ★

| 基準 | 期待 | best (C3 / pb 0.005) | 達成 |
|---|---|--:|:-:|
| gross_pnl_jpy >= +300,000 | +30 万 | -3,206 | ❌ |
| avg_per_trade >= +3,000 | 期待値 | -10.7 | ❌ |
| PF >= 1.15 | 利益効率 | 0.998 | ❌ |
| max_drawdown_pct <= 5% | drawdown | 1.49% | ✅ |
| daily_halt <= 10 | halt 頻度 | 0 | ✅ |
| trade_count >= 100 | 統計 | 300 | ✅ |
| duplicate_entry == 0 | 重複なし | 0 | ✅ |
| no_negative_capital | 元本維持 | True | ✅ |

利益系 3 基準で全 preset FAIL、安全系 5 基準は全 PASS (= C2-7 と同様)。

## 9. HQ 追加判断 3 基準 (Lane C Phase C2 保留候補判定)

★ HQ 指示: 「best PF >= 1.05 / best gross_pnl >= +100,000 / avg_pnl/trade
   >= +1,000 のいずれも満たせない場合、Lane C Phase C2 を一旦保留候補
   にする」

| 基準 | 期待 | best (C3 / pb 0.005) | 達成 |
|---|---|--:|:-:|
| best PF | >= 1.05 | 0.998 | ❌ |
| best gross_pnl | >= +100,000 | -3,206 | ❌ |
| best avg_pnl/trade | >= +1,000 | -10.7 | ❌ |

★ **3 基準すべて未達 → Lane C Phase C2 を保留候補に位置付ける** ★

## 10. 結論

### 10.1 C2-7.1 結果

- **30 パターン全 Stage gate FAIL**
- **best = baseline (cutoff 14:45 / pb 0.005 / preset C3)**、calibration
  での改善は **見られなかった**
- cutoff 変更 (14:45 / 14:15 / 14:00) は max_daily_trades=5 制約により
  実質的な選別効果なし
- pullback 引き上げ (0.005 → 0.008) は全 preset で逆効果
- preset C 周辺の TP/SL 微調整 (C0/C1/C2/C4) は C3 (= baseline) を
  超えない

### 10.2 Lane C Phase C2 保留候補

- HQ 追加判断 3 基準 (PF>=1.05 / pnl>=+100k / avg>=+1k) すべて未達
- 浅い pullback と TP/SL 微調整では本質的なエッジを生み出せていない
- **C2-8 (tick/order template 統合) には進まない**

### 10.3 残された改善方向 (本タスク範囲外、HQ 判断要請)

C2-7 §8 で挙げた改善候補のうち、本 C2-7.1 で扱わなかった案:

| 案 | 内容 | 推定難度 | 推定インパクト |
|---|---|---|---|
| A1 (実施済) | PULLBACK 0.5% → 0.8% | 低 | **逆効果確認** |
| A4 | 過熱閾値 10% → 7% (前日比) | 中 (daily feature 安全取得が必要) | 中 |
| B6 (実施済) | entry_time cutoff 前倒し | 低 | **無効果確認** |
| D7 | preset E (ATR 連動) 実装 | 中 | 中 (volatility 適応) |
| E11 | 持ち越し許容化検討 | 大 (HQ 仕様変更要) | 大 (5h 制約解除) |
| E12 | 別 trigger pattern 追加 | 大 | 大 (新エッジ模索) |

最有望候補:
- **D7 preset E (ATR 連動)**: volatility 適応で基本構造は維持
- **A4 過熱閾値 7%**: 浅い entry の絞り込み (universe 内 trade 自体が
  3-4 倍に増えれば結果改善余地)
- **E11/E12 戦略本体見直し**: 持ち越し許容 / 別 trigger は HQ 判断必須

## 11. 制約遵守確認

| 制約 | 結果 |
|---|---|
| daily 近似禁止 | ✅ intraday 1 分足 + vwap_cumulative |
| TPM を VWAP 代替にしない | ✅ vwap_cumulative 直接使用 |
| 持ち越し禁止 | ✅ 全 9,000 trade 当日 exit |
| force_close 15:10 厳守 | ✅ violations 0 (全 9,000 trade) |
| Tier2 478 codes 固定 | ✅ universe_loader 経由 |
| 72030 除外 | ✅ excluded_codes 適用 |
| market_prices_intraday 全体無条件参照禁止 | ✅ extract_candidates 経由 |
| FIRE_ENV=staging guard | ✅ 4 段 guard 通過 |
| production / develop DB 無触 | ✅ read-only URI、staging のみ |
| C2-8 統合進まず | ✅ |

## 12. 関連 commit

~/fire 側:
| commit | 内容 |
|---|---|
| 35f814f | chore(F281-C2): add Lane C calibration retest runner |
| a1b3347 | fix(F281-C2): force_close 15:10 厳守 |
| 94bc75f | C2-5 strict evaluator |
| 22023a4 | C2-4 simulator |
| ca43370 | C2-3 candidate extraction |
| 6ce8f49 | C2-2 setup judge state machine |
| c94fb16 | C2-1 Tier2 universe loader |

vault 側:
| commit | 内容 |
|---|---|
| 7001623 | C2-7 strict 60 営業日評価結果 |
| 9b0f13f | C2-6 smoke v1.1 |
| dea61d4 | C2-0 Phase C2 implementation plan v1.0 |
| 本 commit | C2-7.1 calibration retest 結果 |

## 13. 次工程提案 (HQ 判断要請)

★ **C2-8 進まず、Lane C Phase C2 保留候補 → HQ 判断必須** ★

優先度提案:
1. **保留 + 別 Lane / F285 / F287 へ注力**: Lane C は本構造で限界、
   別 Lane (Lane B スイング 等) や F285 Research Lane / F287 決算
   ダッシュボードに優先度シフト
2. **戦略本体の見直し** (E11/E12): 持ち越し許容化 or 別 trigger 追加 →
   実装大幅、HQ 仕様レビュー必須
3. **preset E 実装 + 過熱閾値変更**: daily feature の安全取得が必要、
   F100 daily / F104 indices と統合検討

HQ 判断 Q:
- Q1: Lane C Phase C2 を保留にして別 Lane / F285 / F287 へ?
- Q2: 戦略本体見直し (持ち越し許容化 / 別 trigger) を検討?
- Q3: preset E + 過熱閾値変更で再評価する余地あり?

## 14. 改訂履歴

- v1.0 (2026-05-09): 初版、C2-7.1 calibration retest 完了直後の
  Vault 化、30 パターン全 Stage gate FAIL、HQ 追加判断 3 基準も
  全未達、Lane C Phase C2 保留候補と判定。
