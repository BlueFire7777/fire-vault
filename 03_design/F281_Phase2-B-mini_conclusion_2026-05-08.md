---
title: F281-Phase2-B-mini 完了総括 (Lane A1 broad strategy 不採用)
date: 2026-05-08
phase: F281-Phase2-B-mini
status: 完了 (Lane A1 broad strategy は Stage 3 候補から外す、approved_active 化なし)
related: F281 v1.2 §5-bis Active Light, eval_mode_strict v0.2, B-3 旧 eval, B-strict-2a/2b/2c, B-strict full run
trigger: HQ B-strict-4 受入後の F281-Phase2-B-mini 完了処理指示
---

# F281-Phase2-B-mini 完了総括

★ **Lane A1 broad strategy は Stage 3 候補として不採用** ★
   approved_active 化なし / Death Note 化はまだしない / memory 改訂なし。
   期待値プラス 32 patterns の事後抽出採用も実施せず (過学習 / データ
   スヌーピング懸念)。次工程は Lane B/C/D 等の複数戦略レーン評価 or
   F271 品質改善ブロック第 5 弾 (本部判断待ち)。

## 1. Phase2-B-mini の目的

F281 (複数戦略レーン管理型資産運用 OS) の Phase2-B-mini として、
Lane A1 (既存 patterns 4,708 件 / status='candidate') を Stage 3
Active Light 制約下で評価し、Lane A1 採用 / 不採用 / 中間判断を取得
することが本 Phase の目的。

最終ゴール:
- Lane A1 broad strategy が Stage 3 想定の Active Light 制約 (1 日
  最大 5 trade / max 3 OPEN / risk_per_trade_pct=0.002 / daily halt
  -0.5% / TP/SL preset) 下で実用的な期待値を持つかを実測
- 旧 eval_mode (B-3) の制約なし結果と比較可能な数値を取得

## 2. B-3 旧 eval 結果の扱い

run_id: PL-20260507134317-3859
60 営業日 / 全銘柄 / 制約なし
- closed: 171,859 件 / 期待値 -13,239 円/件 / current_capital -22.65 億

evaluation_status: **invalid_for_strategy_judgment**
理由: simulation_engine_invalid_under_active_light_constraints
- 同 pattern × symbol で 114 回 entry 累積 (5,700 株 × 高値株 137K
  円で単件 -2.3M 円損失)
- max_daily_trades / max_open_positions / daily_halt 機構不在
- 100 株固定 lot / buying_power check なし

→ B-3 結果は参考記録として Vault 化済 ([[F281_Phase2-B-mini_stageB_lane_a1_eval_result_2026-05-08]])、
   戦略判定には使わない。

## 3. B-strict 実装内容

### 3-1. B-strict-2a (schema + models + migration)

- `simulation/paper_live/strict_config.py` 新規 (ActiveLightConstraints +
  TPSLPreset + StrictCostModel)
- `simulation/paper_live/models.py` 拡張 (VirtualPosition / Account /
  Result + SKIPPED_ENTRY)
- `scripts/setup/migrate_strict_columns.py` 新規 (staging 専用 16 列追加、
  production / develop reject + traversal / symlink reject + 冪等)
- 13 migration tests PASS、staging DB に migration 適用済

### 3-2. B-strict-2b (account / position / tick の制約ロジック)

- `account.py` に None 補完 + reset_for_date + halt 判定 + buying_power
  check + max_daily_trades / max_open_positions check
- `position.py` に try_strict_entry (1 conn + BEGIN IMMEDIATE で
  position INSERT + account UPDATE + paper_live_results INSERT を全部
  atomic) + close_strict (gross/fees/slippage/net 分離 + account
  daily_pnl 更新)
- `tick.py` に StrictTickContext + extract_strict_candidates +
  metric_value 分離 helper + force_close exit_price_source
- 34 unit tests PASS (active light / risk qty / buying power / net pnl /
  skipped_entry / metric separation)
- Codex CRITICAL 5 件すべて atomic transaction で解消 (race / 不整合 /
  二重保存)

### 3-3. B-strict-2c (runner / cli 統合 + smoke test)

- `runner.py` に strict context (start_session strict 引数 + 4 段 guard
  + process_tick 分岐 + close dispatcher + daily reset + run_mode='eval_strict')
- `cli.py` に --eval-mode-strict + --tp-sl-preset {A,B,C} + 3 段 CLI guard
- 22 integration / cli guard tests PASS

## 4. B-strict smoke test 結果 (3 営業日)

run_id: PL-20260507171222-56D8
3 営業日 / preset A / staging
- ticks 201、events 19,653、tick error 0、LINE 429 = 0
- closed: 15 (TP 2 / SL 8 / FC 5)、net -144,060、avg -9,604
- 合格条件 **15/15 クリア** (duplicate_entry_count=0、max_daily_trades<=5、
  max_open_positions<=3、current_capital マイナス化なし、Active Light
  6 制約全発火)

## 5. B-strict full run 結果 (60 営業日 × preset A 単独)

### 5-1. 走行サマリ

run_id: PL-20260507174223-3850
60 営業日 (2026-02-03 〜 2026-05-01) × 全銘柄 / preset A 単独
走行時間: **7 時間 46 分 30 秒** (HQ 想定 6-8h、ほぼ中央)

### 5-2. 主要指標

| 指標 | 値 |
|---|---|
| ticks | 4,020 / 4,020 |
| events | 469,035 |
| tick error | 0 ✅ |
| LINE 429 | 0 ✅ |
| DB 増分 | +245 MB |
| closed | **241 件** |
| TP / SL / force_close | 34 / 127 / 80 |
| gross_pnl | -1,058,520 円 |
| fees | 500,944 円 |
| slippage | 500,944 円 |
| **net_pnl** | **-2,060,408 円** |
| avg_net_pnl | **-8,549 円/trade** |
| **win_rate (TP/SL)** | **21.1%** (preset A 必要勝率 34.8%、13.7pt 不足) |
| **profit_factor** | **0.308** |
| max_drawdown | 2,060,408 円 (-20.6% / initial) |
| max single loss | -24,710 円 |
| max single profit | +31,573 円 |

### 5-3. 制約発動状況

- max_daily_trades_observed: 5 (上限到達多数)
- max_open_positions_observed: 3 (上限到達観測)
- duplicate_entry_count: **0** ✅
- daily_halt 発動日: **24 / 60 営業日 (40%)**
- skipped_entry: 468,071 (6 種類すべて発火)

### 5-4. account 状態

- initial: 10,000,000
- current_capital_final: 7,939,592 (∆ -2,060,408、-20.6%)
- マイナス化なし ✅

### 5-5. pattern 別

- trade 発生 pattern: 215 / 4,708 (4.6%)
- **期待値プラス pattern: 32 件 (14.9%)**
- 期待値マイナス/0 pattern: 183 件

## 6. Lane A1 broad strategy 不採用 判定 (HQ Q-2c-7 確定)

★ **不採用** ★

判定根拠:
1. **profit_factor 0.308 < 1.0**: 構造的に勝てない数値
2. **win_rate 21.1% << 必要勝率 34.8%** (preset A、R:R 1.875): 13.7pt
   不足、preset B/C で R:R 拡大しても勝率改善は期待困難
3. **avg_net_pnl -8,549 円/trade**: コスト (fees+slippage 1M = gross_loss
   33.7%) を含めた net で系統的にマイナス
4. **max_drawdown -20.6%**: Stage 3 想定の Active Light 仕様 (DD -2%
   で paper_only) と大幅乖離、実運用リスク過大
5. **期待値プラス pattern 14.9%**: 大半の pattern が構造的に勝てない、
   broad 採用は混在で全体マイナス

  ★ Active Light 制約は **正常に機能** (B-3 累積 entry 等の歪み排除済、
    本数値は実運用相当の真の評価)。strict 評価基盤の妥当性は確認済。

## 7. preset B/C 追加走行を行わない理由 (HQ Q-2c-8 確定)

判定: 追加走行 **実施しない**

理由:
1. preset A で win_rate 21.1% は構造的に低く、preset B/C で TP を
   遠くしても勝率改善は限定的
2. preset B (R:R 2.0、必要勝率 33.3%) / preset C (R:R 2.5、必要勝率
   28.6%) で **win_rate × RR > 1** を満たすには、win_rate が大幅
   改善する必要があるが、broad strategy では困難
3. 14 時間以上の追加走行コストに見合う情報利得が小さい
4. Lane B/C/D 新規戦略評価へリソース投下する方が効率的

## 8. 期待値プラス 32 patterns の扱い (HQ Q-2c-9 確定)

判定: **採用しない (subset 走行も実施しない)**

理由:
1. **過学習 / データスヌーピングのリスクが高い**: 同一期間内の事後
   抽出は in-sample bias、実運用での再現性は保証されない
2. walk-forward 検証 / holdout 検証なしに採用すると、選別バイアスで
   実運用で期待値マイナスになるリスク大
3. 32 patterns は参考候補として Vault に保存するが、approved_active
   化はしない

将来扱う場合の条件 (HQ 指示):
- 別タスクで walk-forward 検証または holdout 検証を実施
- 検証期間と評価期間を分離

## 9. approved_active 化しない理由

1. broad strategy 不採用 (PF 0.308、win_rate 21.1%)
2. 32 patterns subset の事後抽出は in-sample bias
3. R-13-08 (自動反映禁止・承認制) 維持、Fujiwara 手動承認なし

## 10. Death Note 化はまだしない理由

1. broad strategy 不採用 = 直ちに Death Note 化 (再採用永久禁止)
   ではなく、**Stage 3 初期候補から外す**段階
2. 将来 walk-forward 検証で期待値プラス subset が確認された場合、
   採用余地を残す
3. Death Note 化は F033 3 段階判定の最終 stage、慎重判断必要

## 11. 次に進むべき方向 (本部判断材料)

### 11-1. Lane B / Lane C / Lane D 等の複数戦略レーン評価

F281 v1.2 § 8 レーン構造に基づく:
- **Lane B**: 新規 patterns 抽出 (Pattern Research Agent 起動、F035)
- **Lane C**: 前日強銘柄初押し (4-15 営業日新規戦略、F281 §8 Lane C 候補)
- **Lane D**: スイング順張り (中期、F281 §8 Lane D 候補)
- **Lane G/H**: 短期決算プレイ / 板読み等 (F281 §8 拡張 Lane)

各 Lane で個別に Active Light 制約の eval_mode_strict 評価を実施可能
(B-strict 基盤再利用)。

### 11-2. F271 品質改善ブロック第 5 弾

ミス 21-26 候補を Vault 化 (HQ Q-2c-10 で B-strict 完了後と確定済)。

### 11-3. broad → narrow への構造改善

Lane A1 broad の代わりに sector / volume / time-of-day 等で narrow
化した sub-strategy 評価。

## 12. F281-Phase2-B-mini 完了判定

★ **完了** ★

完了根拠:
- B-strict-2a / 2b / 2c 実装完了 (44 新規 test PASS、Codex pre-commit
  全件 OK、--no-verify 不使用)
- staging smoke test 合格条件 15/15 クリア
- B-strict full run 60 営業日完了 (合格条件 12/12 クリア)
- B-strict-4 集計 + Vault 化完了
- Lane A1 broad strategy 不採用判定確定
- approved_active 化 / Death Note 化 / memory 改訂なし (HQ 厳守)
- production / develop DB 完全無触

未完了 / 保留:
- preset B/C 追加走行 → 実施しない (HQ 確定)
- 32 patterns subset 採用 → 実施しない (過学習リスク、HQ 確定)
- 32 patterns walk-forward 検証 → 別タスクで実施 (将来候補)
- ミス 21-26 Vault 化 → 完了処理後の品質改善ブロック第 5 弾で実施

## 13. 関連 Vault リンク

- [[F281_Phase2-B-mini_stageB_lane_a1_eval_2026-05-08|B-3 旧 eval 設計記録]]
- [[F281_Phase2-B-mini_stageB_lane_a1_eval_result_2026-05-08|B-3 旧 eval 結果記録 (戦略判定不可)]]
- [[F281_Phase2-B-mini_stageB_strict_eval_2026-05-08|eval_mode_strict v0.2 設計記録]]
- [[F281_Phase2-B-mini_stageB_strict_smoke_2026-05-08|B-strict smoke test 結果記録]]
- [[F281_Phase2-B-mini_stageB_strict_fullrun_2026-05-08|B-strict full run 結果記録]]
- [[F271_v1.3_2026-05-07|F271 本部品質改善ルール]]

## 14. 改訂履歴

- v1.0 (2026-05-08): 初版、F281-Phase2-B-mini 完了処理時点の総括
