---
id: F181
phase: P1: Pattern/Feature Store
priority: 中
status: 完了
owner: Fujiwara
depends_on: [F180]
chapter: "29"
created: 2026-04-26
updated: 2026-04-26
---

# F181: target_holding_window パターン別管理

## 概要

`patterns/labels.py` 改修。F180 の `DEFAULT_HOLDING_WINDOWS_MIN` キーを日本語→英語化 (F200 整合) し、5 種の `StrategyType` Enum を整備。

## 実装内容

### 主要モジュール

- `patterns/labels.py`
  - `DEFAULT_HOLDING_WINDOWS_MIN` キーを英語化 (F200 整合)
  - `StrategyType` Enum 5 種類: `material_initial` / `theme_lead` / `semi_scalp` / `swing` / `long_term`
  - `STRATEGY_TYPE_LABELS_JA`: 英語キー → 日本語表示名マッピング
  - 追加関数:
    - `get_target_holding_window`
    - `is_within_target_window`
    - `resolve_holding_window_for_classify`
    - `set_strategy_type`
    - `list_patterns_by_strategy_type`
    - `get_strategy_type_label_ja`

### キーポイント

- `DEFAULT_HOLDING_WINDOWS_MIN` のキー変更は F180 既存コードへの破壊的変更 (英語化で F200 と整合)
- F201 同時リファクタ: `StrategyType` / `STRATEGY_TYPE_LABELS_JA` / `get_strategy_type_label_ja` を `lanes.py` に移管 (`labels.py` の重複解消)

## テスト

- `tests/patterns/test_labels_F181.py`: 28 ケース全 PASS

## 関連リンク

- 要件書: [[FIRE_要件書_第29章_Pattern_Library_パターン管理方針]]
- 要件 ID: R-29-07
- 関連: [[F182_補助メトリクス計算]], [[F183_採用最重要指標]], [[F201_頻度監視]]
- コード: `~/fire/patterns/labels.py`
