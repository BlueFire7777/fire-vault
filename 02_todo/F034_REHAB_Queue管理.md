---
id: F034
phase: P1: Pattern/Feature Store
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F033, F038]
chapter: "21"
created: 2026-04-26
updated: 2026-04-26
---

# F034: REHAB Queue 管理 (DEATH_NOTE → REHAB → READOPT)

## 概要

`patterns/rehab.py` 新規作成。DEATH_NOTE 入りパターンの REHAB (リハビリ) → READOPT (再採用) 提案フロー。

## 実装内容

### 主要モジュール

- `patterns/rehab.py`
  - `RehabConditionType` Enum 4 種: 決算期 / 強地合い / セクター限定 / 執行改善
  - `get_rehab_conditions` / `set_rehab_conditions`: metadata 経由で `rehab_conditions` JSON 保存・取得
  - `list_rehab_queue`: REHAB 状態のパターン一覧
  - `evaluate_sector_specific_recovery` / `evaluate_regime_specific_recovery`: 復活シナリオ判定 (セクター/銘柄限定、強地合いのみ)
  - `propose_rehab_for_death_note_pattern` + `submit_rehab_request_with_conditions`: DEATH_NOTE → REHAB 提案
  - `propose_readopt_for_rehab_pattern` + `submit_readopt_request`: REHAB → READOPT 提案
  - `scan_all_death_note_for_rehab_proposals` / `scan_all_rehab_for_readopt_proposals`: 一括スキャン
- CLI: `python -m patterns.rehab --mode scan_rehab --auto-submit`

### キーポイント

- マイグレーション不要: `rehab_conditions` は `patterns.metadata` に保存
- F038 ApprovalManager 経由で状態遷移、自動復帰禁止 (R-21-09)

## テスト

- `tests/patterns/test_rehab.py`: 全 PASS

## 関連リンク

- 要件書: [[FIRE_要件書_第21章_Pattern_Store___Feature_Store_運用方針]]
- 要件 ID: R-21-08, R-21-09
- 関連: [[F033_DEATH_NOTE_3段階判定]], [[F038_パターン採用承認フロー]]
- コード: `~/fire/patterns/rehab.py`
