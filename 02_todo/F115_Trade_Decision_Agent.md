---
id: F115
phase: P4: LINE/通知基盤
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F111]
chapter: "04,06,14"
created: 2026-05-02
updated: 2026-05-02
tags: [P4, agents, trade_decision]
---

# F115: Trade Decision Agent (株数計算 + 注文指示生成)

## 概要

F111 `DaytradeCandidate` を受けて、第 6 章 v3.3 R-06-03 の **必須 11 項目を満たす
TradeOrder** を生成。F130 (許容損失準攻撃型) は仮値 = equity × 0.5%、F140
(Execution Quality Gate) はフックポイントのみ (素通し)。

要件根拠:
- 第 4 章 (エージェント群: Trade Decision Agent)
- 第 6 章 v3.3 R-06-03 (通知必須 11 項目)
- 第 6 章 R-06-04/05 (200 株: 利確 2 段、100 株: 利確 1 段)
- 第 14 章 R-14-07 (FIRE はロット計算する、余力最終確認は Fujiwara)

## 実装内容

### 主要モジュール

- `agents/trade_decision.py` (新規)
  - `TakeProfitLeg` dataclass: 利確 1 段 (price/quantity/label)
  - `TradeOrder` dataclass: R-06-03 11 項目 + F115 計算結果 + F140 ゲート結果
  - `TradeDecisionResult` dataclass: 1 日分の発注指示集計
  - `InsufficientFundsError` 例外
  - `TradeDecisionAgent` クラス
    - `decide_orders(candidates, account_equity=None, as_of_dt=None)`: メイン
    - `_build_order(candidate, equity, as_of)`: 1 候補 → 1 TradeOrder
    - `_build_take_profits(qty, entry, tp, side)`: R-06-04/05 利確段数
    - `_allowable_loss_per_trade(equity)`: **F130 仮値** (equity × 0.5%、関数で隔離)
    - `_check_execution_gate(order)`: **F140 フック** (素通し、後で置換)
    - `_fetch_account_equity()`: 最新 paper_live_account.current_capital、無ければ FALLBACK 100 万円
    - `_get_entry_price`: candidate.entry_price > market_prices_daily 最新 close > 1000.0
    - `_get_tp_price` / `_get_sl_price`: F054 デフォルト (+5% / -2%)
    - `_get_side`: candidate.side > "long" (Phase 1)

### F111 DaytradeCandidate / F051 account.py の API 確認結果

| 観点 | 確認結果 |
|---|---|
| F111 DaytradeCandidate に entry_price / side / tp_price / sl_price | **無し** — F115 で属性ベースのフォールバック (`_get_entry_price` で market_prices_daily 最新 close、`_get_side` は属性 or "long"、TP/SL は F054 デフォルト %) |
| F051 account.py に `get_account_equity()` 関数 | **無し** — `AccountTracker(run_id, db_path).get_account()` のみ。F115 では SQL 直接で `paper_live_account.current_capital` 最新行を取得 (run_id 非依存)、無ければ `FALLBACK_EQUITY=1_000_000` |

→ F111 拡張で `entry_price` / `side` を持つ場合は自動的に活用される構造 (`getattr` 経由)。

### キーポイント

- **R-06-03 11 項目**: symbol / side+margin_type / action / entry_price / quantity /
  stop_loss_price (= 6 と 8 兼用) / take_profits / valid_until / invalid_condition /
  strategy_summary
- **株数計算**: `allowable_loss / risk_per_share` を 100 株単位丸め、最小 100 株
- **余力チェック**: 必要資本 > 余力なら自動で株数を絞る、それでも MIN_QUANTITY 不可なら `InsufficientFundsError`
- **R-06-04/05 利確段数**: 100 株 → 1 段 / 200 株+ → 2 段 (TP1=entry+(tp-entry)×0.6 で半分、TP2=tp_price で残り)
- **F130 / F140 のフック**: 関数として隔離、後で本格実装時に差し替え
- **余力共有**: 複数候補で順次余力消費、後発候補は前発分を引いた残余力で計算
- **0 件入力で空完走**: F058/F111 と同じフレームワーク先行

## テスト

- `tests/agents/test_trade_decision.py`: **22 / 22 PASS**
  - TestAllowableLoss (3): F130 仮値 / 0 / 定数
  - TestExecutionGate (1): フック素通し
  - TestBuildTakeProfits (4): 100/200/300/99 株 R-06-04/05
  - TestBuildOrder (4): 11 項目 / 単元丸め / 0 リスク / 余力不足
  - TestDecideOrders (4): 空 / 単一 / 余力共有 / 余力不足→rejected
  - TestFetchEquity (2): フォールバック / DB 値
  - TestEndToEnd (4): F111→F115 連結 / TP 2 段 / to_dict JSON 化 / R-06-03 11 項目存在
- 累計 **622 PASS** (600 → 622、F040-F058 + F100 + F236 + F111 既存全 PASS 維持)

## CLI smoke 結果

### F054 → F111 → F115 連結 (実 DB)
```
F054 raw: 0 → F111 selected: 0 → F115 orders: 0 / rejected: 0
→ 0 件で空完走 (期待通り)
```

### TradeOrder サンプル (mock candidate, Fujiwara 視認用)

入力: 7203 / entry=1500 / pattern=guidance_upside-strong_market-... / score=0.78 / equity=100 万円

```
--- R-06-03 必須 11 項目 ---
1. 銘柄名         : 7203
2. 売買区分       : long (信用買建)
3. 行動区分       : 即発注
4. 新規価格       : 1500 円
5. 株数           : 100 株
6/8. 逆指値・損切 : 1470 円
7. 利確 (1 段):
     利確1: 1575 円 × 100 株
9. 有効期限       : 10:15 まで       (as_of 09:00 + 75 分)
10. 無効条件      : 1492 を割れたら見送り
11. 一言戦略      : 上方修正を起点とする前場高値ブレイクを狙う

--- F115 計算結果 ---
  許容損失       : 5000 円 (F130 仮値 = equity × 0.5%)
  期待利益 (TP) : 7500 円
  RR 比          : 2.50
  F140 ゲート    : passed=True (フック素通し)
```

→ R-06-03 全 11 項目埋まる、RR=2.50 で F053 promotion `min_avg_rr=1.5` をクリア可能。

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| agents/trade_decision.py 新規 | ✅ |
| TradeOrder / TakeProfitLeg / TradeDecisionResult | ✅ |
| decide_orders 動作 | ✅ |
| F130 仮値 株数計算 | ✅ |
| F140 フック素通し | ✅ |
| R-06-04/05 利確段数 | ✅ |
| 22 ケース全 PASS | ✅ |
| 既存 600 PASS 非破壊 | ✅ |
| 累計 622 PASS | ✅ |
| CLI smoke (F054→F111→F115) | ✅ |

## 関連リンク

- 要件書: 第 4 章 / 第 6 章 v3.3 R-06-03/R-06-04/R-06-05 / 第 14 章 R-14-07
- 関連: [[F111_Daytrade_Selection_Agent]] / [[F051_仮想建玉計算]] /
  [[F054_tick_py中身実装]]
- 次タスク候補:
  - **F062** LINE 注文完成形テンプレート (TradeOrder → LINE メッセージ整形)
  - **F130** 許容損失準攻撃型実装 (`_allowable_loss_per_trade` の本格化)
  - **F140** Execution Quality Gate (`_check_execution_gate` の中身)
- コード: `~/fire/agents/trade_decision.py`

## スコープ外メモ

- 許容損失準攻撃型の本格ルール → F130 (現在は equity × 0.5% 仮値)
- LINE メッセージ整形 → F062
- 執行品質ゲートの中身 → F140
- 注文タイプ細分化 (寄り指値/成行/逆指値) → F141 ハイブリッド執行方針
- 部分約定処理 → F143
- short / 空売り対応 → 将来 (現状 long メイン)
- OpenClaw 統合 → F260
