---
id: F052
phase: P3: Paper Live
priority: 中
status: 完了
owner: Fujiwara
depends_on: [F044, F050, F051]
chapter: "19"
created: 2026-05-01
updated: 2026-05-01
tags: [P3, paper_live, fill_model]
---

# F052: Paper Live 厳しめ約定モデル標準化

## 概要

F050 で約定モデルは引数受け取りのみだった状態を、tick.py での **価格決定処理に統合**。
デフォルトを `realistic` → `strict` に変更し、Paper Live の標準を厳しめモデルに据える
(R-19-05 準拠)。約定品質情報を `paper_live_results.payload` に記録する。

## 実装内容

### 主要変更

- `simulation/paper_live/runner.py`:
  - `start_session(fill_model="strict")` がデフォルトに (`"realistic"` から変更)
- `simulation/paper_live/tick.py`:
  - `process_tick()` で `get_fill_model(ctx.fill_model)` でインスタンス化
  - `evaluate_virtual_entries(ctx, candidates, fill_model)`: `fill_model.fill_entry(req)` で約定価格・スリッページ・手数料・品質スコア取得 → payload に格納
  - `monitor_virtual_positions(ctx, fill_model)`: F051 PositionTracker から open_positions を取得、F100 後の TP/SL タッチ判定に備えたフレームワーク (現状はデータ 0 件で空動作)
  - `fill_model` 引数は Optional (後方互換、None なら約定計算スキップ)
- `simulation/paper_live/cli.py`:
  - `--fill-model` デフォルトも `strict` に揃える

### payload に記録される 4 項目 (R-19-05)

各 VIRTUAL_ENTRY イベントの `payload` に以下を保存:
- `fill_quality_score` (0.0-1.0、1.0 が理想)
- `slippage_pct` (スリッページ率、0.0 が滑りなし)
- `commission` (手数料、円単位)
- `fill_model` (使用された約定モデル名)

### キーポイント

- `filled=False` (約定不成立) は entries に追加せずスキップ。"rejected" イベント拡張は別タスク
- `fill_model=None` で後方互換 — F050 既存テストが直接 `evaluate_virtual_entries(ctx, [])` を呼んでも壊れない
- 強制クローズの fill_model 統合は F052 スコープ外 (時刻判定ベースで価格決定なし)
- TYPE_CHECKING 経由の forward reference で循環 import を回避

## テスト

- `tests/simulation/test_paper_live_strict_integration.py`: **15 ケース全 PASS**
  - process_tick fill_model 統合 (3) / evaluate_virtual_entries (3) / monitor (2) /
    runner デフォルト (2) / 統合 (3) / 永続化 (2)
- `tests/simulation/test_paper_live.py`: 1 ケース更新 (`fill_model == "realistic"` → `"strict"`)、24/24 PASS 維持
- `tests/simulation/test_paper_live_positions.py`: 変更なし、22/22 PASS 維持
- 累計 **209 PASS** (194 → 209)

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| start_session デフォルト = "strict" | ✅ |
| tick.py に fill_model 統合 | ✅ |
| payload に 4 項目記録 | ✅ |
| F050 既存 24 PASS 維持 (1 ケース更新済) | ✅ |
| F051 既存 22 PASS 維持 | ✅ |
| F052 新規 15 PASS | ✅ |
| 累計 209 PASS | ✅ |
| CLI デフォルト = strict | ✅ |

## 関連リンク

- 要件書: [[FIRE_要件書_第19章_Simulation___Paper_Live_方針]]
- 要件 ID: R-19-05 (3 段階約定モデル: 理想/現実寄り/厳しめ)
- 前タスク: [[F051_仮想建玉計算]]
- 次タスク: [[F053_Semi_Auto昇格基準]] (Paper Live → Live Advisory)
- コード:
  - `~/fire/simulation/paper_live/runner.py` (デフォルト変更)
  - `~/fire/simulation/paper_live/tick.py` (fill_model 統合)
  - `~/fire/simulation/paper_live/cli.py` (CLI デフォルト変更)
- テスト: `~/fire/tests/simulation/test_paper_live_strict_integration.py`

## スコープ外メモ

- 強制クローズの fill_model 統合 (将来別タスク)
- "rejected" イベント記録 (filled=False を別イベントで残す)
- F046 / F047 (Simulation Accuracy) との統合 (Replay 側で完結)
- スケジューラ実装 (別タスク)
