---
id: F033
phase: P1: Pattern/Feature Store
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F029]
chapter: "21"
created: 2026-04-26
updated: 2026-04-26
---

# F033: DEATH NOTE 3 段階判定独立モジュール

## 概要

`patterns/death_note.py` 新規作成。劣化パターンの DEATH NOTE 入りを 3 段階で判定する独立モジュール。

## 実装内容

### 主要モジュール

- `patterns/death_note.py`: 3 段階の劣化判定ロジック

### キーポイント

- F034 (REHAB Queue 管理) と対をなす劣化系パイプライン
- F038 ApprovalManager の DEATH_NOTE イベント発火源

## 関連リンク

- 要件書: [[FIRE_要件書_第21章_Pattern_Store___Feature_Store_運用方針]]
- 関連: [[F034_REHAB_Queue管理]], [[F038_パターン採用承認フロー]]
- コード: `~/fire/patterns/death_note.py`
