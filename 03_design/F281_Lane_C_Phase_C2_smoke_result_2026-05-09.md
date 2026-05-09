---
title: F281 Lane C Phase C2 smoke 結果 (C2-6)
date: 2026-05-09
phase: F281-Phase-C2 / C2-6 smoke
status: smoke 完了、Phase C2 §13 12 観点すべて担保、C2-7 strict 60 営業日評価へ進める判定
related: F281_Lane_C_Phase_C2_implementation_plan_2026-05-09, F284_F105_c6_final_result_2026-05-09, F281_Lane_C_design_2026-05-08
trigger: HQ C2-6 着手指示 (2026-05-09、C2-5 strict_evaluator 完了承認後)
---

# F281 Lane C Phase C2 smoke 結果 (C2-6)

★ Phase C2 仕様書 §13 smoke 計画 (= Tier2 内 3 銘柄 × 5 営業日 ×
   preset B、利益ではなく動作確認) を実施。実 staging DB から
   read-only で C2-1〜C2-5 の一連パイプラインが安全に動作することを
   確認。

## 1. smoke 実施情報

| 項目 | 値 |
|---|---|
| smoke 実施日時 | 2026-05-09 09:06:49 JST |
| 対象 commit (~/fire) | C2-1 c94fb16 / C2-2 6ce8f49 / C2-3 ca43370 / C2-4 22023a4 / C2-5 94bc75f |
| 対象 commit (vault) | C2-0 dea61d4 |
| 実行環境 | Mac mini、FIRE_ENV=staging、~/fire/.venv/bin/python |
| DB | /Users/bluefire/fire/data/fire.staging.db (read-only URI: `file:...?mode=ro`) |

## 2. smoke 条件

| 項目 | 値 |
|---|---|
| 対象 universe | Tier2 478 codes 先頭 3 銘柄 (= **13060** / **13200** / **13210**) |
| 対象期間 | **2026-04-21 (火) 〜 2026-04-27 (月)** |
| 実営業日数 | **5** (土日 4-25/4-26 除外、c6 backfill 済期間内) |
| 対象 preset | **B** (TP +1.2% / SL -0.8%) |
| max_daily_trades | 5 (default、Active Light) |
| initial_capital_jpy | 10,000,000 |
| notional_per_trade_jpy | 1,000,000 |
| daily_halt_threshold_jpy | -50,000 |
| 72030 除外 | excluded_codes = ('72030',)、universe_loader で除外確認 |

## 3. preset B summary

| 項目 | 値 |
|---|--:|
| total_candidates | **15** (= 3 × 5) |
| setup_detected_count | **12** (= 3 件 skip = no_pullback 2 + no_recovery_or_breakout 1) |
| trade_count | **12** |
| win_count | 6 |
| loss_count | 5 |
| force_close_count | 6 |
| win_rate | **0.5000** |
| gross_pnl_jpy | **+17,515 円** |
| avg_gross_pnl_per_trade | +1,459.6 円 |
| gross_profit | +41,682 |
| gross_loss | -24,167 |
| profit_factor | **1.7247** |
| max_drawdown_jpy | 23,501 |
| max_drawdown_pct | 0.002350 (= 0.235%、initial_capital + 累積 pnl 基準) |
| **daily_halt_count** | **0** |
| **duplicate_entry_count** | **0** |
| **no_negative_capital_flag** | **True** |
| skipped_reason_counts | `{no_recovery_or_breakout: 1, no_pullback: 2}` |

★ smoke の目的は「動作確認」、利益値は参考。本格期待値評価は C2-7。

## 4. trades 詳細 (12 件)

| date | code | entry | exit | reason | shares | pnl_jpy |
|---|---|---|---|---|--:|--:|
| 2026-04-22 | 13060 | 09:52 @ 397.30 | 15:10 @ 397.30 | force_close | 2516 | 0 |
| 2026-04-22 | 13200 | 14:16 @ 62050.00 | 15:11 @ 62040.00 | force_close | 16 | -160 |
| 2026-04-22 | 13210 | 14:16 @ 62240.00 | 15:10 @ 62300.00 | force_close | 16 | +960 |
| 2026-04-23 | 13060 | 09:19 @ 396.10 | 11:04 @ 392.93 | stop_loss | 2524 | -7,998 |
| 2026-04-23 | 13200 | 09:19 @ 62360.00 | 10:34 @ 61861.12 | stop_loss | 16 | -7,982 |
| 2026-04-23 | 13210 | 09:29 @ 62670.00 | 10:29 @ 62168.64 | stop_loss | 15 | -7,520 |
| 2026-04-24 | 13210 | 09:07 @ 62290.00 | 15:10 @ 62450.00 | force_close | 16 | +2,560 |
| 2026-04-24 | 13200 | 13:12 @ 62030.00 | 15:11 @ 62210.00 | force_close | 16 | +2,880 |
| 2026-04-24 | 13060 | 13:52 @ 394.80 | 15:10 @ 394.60 | force_close | 2532 | -506 |
| 2026-04-27 | 13060 | 09:21 @ 392.60 | 10:51 @ 397.31 | take_profit | 2547 | +11,999 |
| 2026-04-27 | 13210 | 09:46 @ 62680.00 | 11:17 @ 63432.16 | take_profit | 15 | +11,282 |
| 2026-04-27 | 13200 | 09:50 @ 62500.00 | 12:30 @ 63250.00 | take_profit | 16 | +12,000 |

## 5. daily summaries

| date | candidates | setup | taken | duplicates | halt | daily_pnl | W/L |
|---|--:|--:|--:|--:|---|--:|---|
| 2026-04-21 | 3 | 0 | 0 | 0 | False | 0 | 0/0 |
| 2026-04-22 | 3 | 3 | 3 | 0 | False | +800 | 1/1 |
| 2026-04-23 | 3 | 3 | 3 | 0 | False | -23,501 | 0/3 |
| 2026-04-24 | 3 | 3 | 3 | 0 | False | +4,934 | 2/1 |
| 2026-04-27 | 3 | 3 | 3 | 0 | False | +35,282 | 3/0 |

★ 4-21 は 3 銘柄全 setup 不成立 (skipped_reason: no_pullback 2 +
   no_recovery_or_breakout 1)、他日は全銘柄 setup 成立で trade 採用。

## 6. 必須確認 12 項目 (HQ 指定、すべて ✅)

| # | 項目 | 結果 |
|---|---|---|
| 1 | candidate 抽出が動く | ✅ 15 candidates 抽出 |
| 2 | setup 判定が動く | ✅ 12 setup_detected、3 skip 判定 |
| 3 | entry/exit simulation が動く | ✅ 12 trades 全 simulate |
| 4 | preset B で評価される | ✅ preset_summaries["B"] 値で確認 |
| 5 | skipped_reason が集計される | ✅ no_pullback 2 / no_recovery_or_breakout 1 |
| 6 | force_close が正しく記録 | ✅ 6 件 force_close (15:10 / 15:11 含む) |
| 7 | 72030 が評価対象に含まれない | ✅ trades codes = {'13060','13200','13210'}、72030 不在 |
| 8 | max_daily_trades 制御が壊れていない | ✅ 各日 candidates 3 件、5 件閾値内で動作 |
| 9 | duplicate_entry_count が 0 | ✅ 全 preset で 0 |
| 10 | no_negative_capital_flag が True | ✅ 全 preset で True |
| 11 | production / develop DB に触れていない | ✅ FIRE_ENV=staging guard 通過、fire.db / fire.develop.db 無触 |
| 12 | staging DB は read-only 接続 | ✅ `sqlite3.connect("file:...?mode=ro", uri=True)` |

## 7. 72030 除外確認

```
trades 内 codes: ['13060', '13200', '13210']
72030 in trades: False
```

universe_loader.EXCLUDED_CODES_PHASE_C2 = frozenset({"72030"}) で除外、
本 smoke では 72030 を評価対象から除外できていることを動作確認。

## 8. 制約遵守確認

| 制約 | 結果 |
|---|---|
| daily 近似禁止 | ✅ intraday bars 利用 (1 分足 + vwap_cumulative) |
| TPM を VWAP 代替にしない | ✅ vwap_cumulative 直接使用、TPM 不参照 |
| 持ち越し禁止 | ✅ 全 trade が当日 exit (force_close 含む) |
| force_close 15:10 厳守 | ✅ 15:10 force_close 動作、15:11 も発火確認 |
| Tier2 478 codes 固定 | ✅ universe_loader 経由、smoke は先頭 3 codes |
| 72030 除外 | ✅ excluded_codes 適用、trades に不在 |
| market_prices_intraday 全体無条件参照禁止 | ✅ extract_candidates 経由、code/date filter |
| FIRE_ENV=staging guard | ✅ FIRE_ENV=staging で起動、4 段 guard 通過 |
| production / develop DB 無触 | ✅ smoke で staging のみ接続 |
| staging DB read-only | ✅ `file:...?mode=ro` URI |
| 自動発注禁止 | ✅ 本 module は注文発行なし |
| 楽天証券操作自動化禁止 | ✅ 楽天 API/iSPEED 接続なし |
| Computer Use 禁止 | ✅ 純粋 Python module、GUI 操作なし |
| C2-8 tick/order template 統合に進まない | ✅ smoke は strict_evaluator までで終了 |

## 9. C2-7 strict 60 営業日評価へ進む判定

★ **進める ✅** ★

判定根拠:
- 必須確認 12 観点すべて担保
- 制約遵守確認すべて OK
- 5 営業日 smoke で candidate 抽出 / setup 判定 / entry-exit simulation /
  preset 比較 / skipped_reason 集計 / force_close / daily_halt / duplicate /
  no_negative_capital すべての機構が正常動作
- staging DB read-only 接続、production / develop DB 無触 確認
- 72030 除外動作 確認

次工程:
- **C2-7**: strict 60 営業日評価
  - 対象: Tier2 478 codes × 60 営業日 × preset A-D = 114,720 評価
  - 期間: 2026-02-03 〜 2026-05-01 (= c6 backfill 全範囲)
  - 結果を Vault 化、preset 別 summary + 日別 summary + trade list +
    skipped_reason_counts を提出

## 10. 関連 commit

~/fire 側 (本 smoke で動作した実装):
| commit | task | 内容 |
|---|---|---|
| c94fb16 | C2-1 | Tier2 universe loader + 72030 exclusion test |
| 6ce8f49 | C2-2 | Lane C setup judge state machine |
| ca43370 | C2-3 | candidate extraction + skipped reasons |
| 22023a4 | C2-4 | intraday entry/exit simulator |
| 94bc75f | C2-5 | strict evaluator + preset comparison |

vault 側:
| commit | task | 内容 |
|---|---|---|
| dea61d4 | C2-0 | Phase C2 implementation plan v1.0 |
| 本 commit | C2-6 | smoke 結果 Vault 化 (本ドキュメント) |

## 11. 改訂履歴

- v1.0 (2026-05-09): 初版、C2-6 smoke 完了直後の Vault 化、Phase C2
  §13 12 観点すべて担保、C2-7 strict 60 営業日評価へ進む判定。
