---
id: F140
phase: P4: LINE/通知基盤
priority: 最優先
status: 完了
owner: Fujiwara
depends_on: [F025, F023, F115]
chapter: "17"
created: 2026-05-02
updated: 2026-05-02
---

# F140: Execution Quality Gate (R-17-07)

## 概要

第 17 章 R-17-07 の執行品質ゲート判定。F115 `_check_execution_gate()` フックの
中身を実装。板厚 / スプレッド / 出来高 / 部分約定リスク / 値飛びリスク /
値嵩スリッページの 6 観点で「致命的 reject / 警告 / 通過」の二段構え判定。

## スコープ

- **致命的 reject 閾値** (1 つでも該当で即 reject):
  - スプレッド > 50 bps (0.5%)
  - 出来高 ratio < 0.3 (平均の 30% 未満)
  - 板厚スコア < 0.3
  - 部分約定リスク > 0.7
  - 値嵩株 + スリッページ > 50 bps
- **警告閾値** (累積で記録):
  - スプレッド > 30 bps
  - 出来高 ratio < 0.5
  - 板厚 < 0.5
  - 部分約定 > 0.5
  - 値嵩 + スリッページ > 30 bps
  - forced_close リスク > 0.5
- **警告 ≥ 3 件で reject** (複合リスク)
- features 不在時は data_available=False で警告 + 通過 (Paper Live 空回り防止)
- F115 `_check_execution_gate()` シグネチャ非破壊 (戻り値 `(bool, str)`)

## 重要な決定事項

- **feature_key 名は実コードに合わせた**: 仕様書の推測名 (`spread_pct`,
  `volume_5min`, `depth_yen_min` 等) は実装と乖離。F025/F023 の実 key 名を採用:
  - `spread_estimate` (bps、F023 C4)
  - `volume_ratio` (倍率、F023 C1)
  - `board_thickness_score` (0.0-1.0、F023 C6)
  - `partial_fill_risk_score` (0.0-1.0、F025 E3)
  - `entry_slippage_estimate` (bps、F025 E1)
  - `forced_close_risk_score` (0.0-1.0、F025 E5)
  - `high_price_stock_flag` (1.0/0.0、F023 C5)
- **値嵩株判定**: F023 と同じ `EXPENSIVE_PRICE_THRESHOLD = 3000.0`
  (仕様書の 5,000 円ではなく実装と整合)。`high_price_stock_flag` 優先、
  なければ `entry_price >= 3000` で fallback。
- **閾値は初期値**: 運用後に F252 / F144 でチューニング想定
- **F115 既存テスト 1 件修正**: 旧 `test_hook_always_passes` (素通し前提) を
  `test_hook_delegates_to_f140` に書き換え。features 不在時の通過動作を検証。

## 状態遷移マトリクス (smoke 結果)

| シナリオ | passed | reason 抜粋 |
|---|---|---|
| 1) features 不在 | OK | features 不在のため通過 (warning) |
| 2) スプレッド 60 bps | NG | スプレッド致命的 (60.0bps > 50.0bps) |
| 3) 出来高 ratio 0.2 | NG | 出来高致命的 (ratio=0.20 < 0.3) |
| 4) 板厚 0.2 | NG | 板厚致命的 (0.20 < 0.3) |
| 5) 部分約定 0.8 | NG | 部分約定リスク致命的 (0.80 > 0.7) |
| 6) 値嵩 5,000 円+slip 60 bps | NG | 値嵩株+高スリッページ |
| 7) 単独警告 (spread 35 bps) | OK | 通過 (警告 1 件) |
| 8) 警告 2 件 (spread + volume) | OK | 通過 (警告 2 件) |
| 9) 警告 3 件 | NG | 複合警告 (3 件) |
| 10) 全指標 OK | OK | 通過 |

## 成果物

### 新規ファイル

- `risk/execution_gate.py`: `ExecutionGateResult` + `check_execution_gate` +
  `_fetch_execution_features` + 14 閾値定数 + `EXECUTION_FEATURE_KEYS`
- `tests/risk/test_execution_gate.py`: 21 ケース

### 変更ファイル

- `risk/__init__.py`: F140 シンボル export
- `agents/trade_decision.py`: `_check_execution_gate()` を F140 呼び出しに置換
  (シグネチャ `(bool, str)` 維持)
- `tests/agents/test_trade_decision.py`: `TestExecutionGate.test_hook_always_passes`
  を `test_hook_delegates_to_f140` に書き換え (F140 連携動作確認)

### テスト

- 新規 21 件 (F140 関連)
  - TestDataAvailability: 3 件 (空 / 部分 / 無関係キー)
  - TestCriticalRejects: 5 件 (5 つの致命的閾値)
  - TestSingleWarnings: 4 件 (単独警告)
  - TestCompoundWarnings: 3 件 (2件→通過 / 3件→reject / 5件→reject)
  - TestF115Integration: 3 件 (フック直呼 / 致命reject / decide_orders 連携)
  - TestExpensiveBoundary: 2 件 (値嵩株境界)
  - TestConstants: 1 件 (閾値順序整合性)
- 累計: 711 → **732 PASS** (+21)
- 既存 711 への影響: **1 件修正** (F115 旧素通しテストを F140 連携テストに更新)

## F115 内ガード階層 (F140 完成後)

```
1. F133 時刻ガード        (R-05-10/11、14:45/15:00 cutoff)
2. F132 損失制御          (R-05-08/09、日次・週次累計)
3. F140 執行品質ゲート     (R-17-07、6 観点二段構え) ← 新規完成
4. R-06-03 11 項目 TradeOrder 生成 (F130 × F131)
```

## スコープ外 (別タスク)

- 寄り直後制限 (R-17-02 9:00-9:05) → **F142**
- 特買売禁止 (R-17-03) → **F142**
- 価格乖離上限 (R-17-04 +0.3%/値嵩+20円) → **F142**
- ハイブリッド執行方針 (R-17-01/08 指値/成行) → **F141**
- 部分約定処理 (R-17-05) → **F143**
- 3 段階監視 (R-17-09) → **F144**
- LINE 通知連携 → **F063** (先送り)
- 楽天証券 API による実時間板情報取得 → 将来検討

## 関連リンク

- 要件書: 第 17 章 R-17-07
- 関連: [[F115_Trade_Decision_Agent]] (フック実装) /
  [[F025_執行品質E1-E5]] (E1-E5 データソース) /
  [[F023_出来高流動性C1-C6]] (C1/C4/C5/C6 データソース) /
  [[F141_ハイブリッド執行方針]] (未着手) /
  [[F142_価格乖離特買売寄り直後]] (未着手) /
  [[F143_部分約定未約定処理]] (未着手) /
  [[F144_執行品質3段階アラート]] (未着手)
- コード: `risk/execution_gate.py` / `agents/trade_decision.py`
