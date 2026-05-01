---
id: F047
phase: P2: Simulation/Backtest
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F046]
chapter: "19"
created: 2026-05-01
updated: 2026-05-01
---

# F047: Optimistic Bias / Realistic Fill Score

## 概要

F046 の 5 生指標を組み合わせて 2 派生スコア (Optimistic Bias Score / Realistic Fill Score) を算出。両スコアとも 0.0〜1.0 にクランプ。これで P2 (Simulation/Backtest) フェーズが完成。

## 実装内容

### 主要モジュール

- `simulation/accuracy/bias_score.py` (新規)
  - `clip01(value)`: 0.0〜1.0 閉区間クランプ
  - `compute_optimistic_bias_score(optimism_gap, range_width, ideal_pnl)`: IDEAL 過信度 (0-1)
  - `compute_realistic_fill_score(fill_quality_avg, robustness_score)`: 実環境信頼度 (0-1)
  - `compute_both_scores(...)`: 2 スコアまとめて辞書で返す
- `simulation/accuracy/models.py`: `AccuracyMetricKey` Enum に 2 項目追加 (`OPTIMISTIC_BIAS_SCORE` / `REALISTIC_FILL_SCORE`)
- `simulation/accuracy/runner.py`: `SimulationAccuracyRunner.run()` に統合 (5 指標 + 2 スコア = 7 指標を simulation_accuracy_results に保存)
- `simulation/accuracy/report.py`: Markdown 7 列対応、派生 2 スコアを `**` で太字強調

### 計算式

```
optimistic_bias_score = clip01(
    optimism_gap_normalized * 0.6 + range_width_normalized * 0.4
)
realistic_fill_score = clip01(
    fill_quality_avg * 0.5 + robustness_score_normalized * 0.5
)
```

重み付け定数はファイル冒頭で `OPTIMISTIC_BIAS_OPTIMISM_WEIGHT` 等として定数化、後で調整可能。

### 閾値の目安 (将来の判定用、F050 以降で実装)

- Optimistic Bias: 0.0-0.3 安全 / 0.3-0.7 注意 / 0.7-1.0 危険
- Realistic Fill: 0.7-1.0 高信頼 / 0.4-0.7 中信頼 / 0.0-0.4 低信頼

### キーポイント

- range_width 正規化: `ideal_pnl` で割って 0〜1 に。`ideal_pnl=0` のゼロ除算は明示ガード
- robustness_score 正規化: 理論上 -∞〜+∞、`clip01` で 0〜1 にクランプ
- F046 既存テストの `len == 5` ハードコード 1 件を `== 7` に更新 (派生スコア追加に伴う調整)

## テスト

- `tests/simulation/test_bias_score.py`: 12 ケース全 PASS
  - clip01 (2) / optimistic_bias_score (3) / realistic_fill_score (3) / compute_both_scores (2) / runner integration (2)
- 累計 148 PASS (F040 20 + F041 20 + F042-F044 50 + F045 28 + F046 18 + F047 12)

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| bias_score.py 新規 (3 関数 + clip01) | ✅ |
| AccuracyMetricKey に 2 項目追加 | ✅ |
| runner 統合 (7 指標保存) | ✅ |
| report Markdown 7 列対応 | ✅ |
| 12 PASS 全成 | ✅ |
| 148 PASS 達成 | ✅ |
| **P2 フェーズ完成** | 🎉 |

## 関連リンク

- 要件書: [[FIRE_要件書_第19章_Simulation___Paper_Live_方針]]
- 要件 ID: R-19-07 (Optimistic Bias / Realistic Fill Score), R-19-09
- 前タスク: [[F046_Simulation_Accuracy指標算出]]
- 次タスク: F050 (Paper Live, Stage 2)
- コード: `~/fire/simulation/accuracy/bias_score.py`
- テスト: `~/fire/tests/simulation/test_bias_score.py`

## P2 フェーズ完成記念

完了スタック:
- Stage 0 (Backtest) 基盤
- Stage 1 (Replay) 基盤
- 約定モデル 3 種 (IDEAL/REALISTIC/STRICT)
- 並列リプレイ実行
- 3 モデル比較精度指標 (5 指標)
- 派生スコア (2 指標)

→ F050 (Paper Live, Stage 2) への移行準備完了
