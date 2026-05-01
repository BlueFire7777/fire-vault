---
id: F243
phase: P1: Pattern/Feature Store
priority: 中
status: 完了
owner: Fujiwara
depends_on: [F035, F036, F039]
chapter: "41"
created: 2026-04-26
updated: 2026-04-26
---

# F243: Sector/Symbol 特化パターン研究機能

## 概要

`agents/sector_research.py` 新規作成。銘柄/セクター個別性検出 + フィルタ提案。穴 12 対策 (第 41 章)。

## 実装内容

### 主要モジュール

- `agents/sector_research.py`
  - `sector_master` テーブル (任意・無くても機能、将来 J-Quants 統合用)
  - `aggregate_by_symbol` / `aggregate_by_sector`: 成績集計
  - `research_pattern_individuality`: 高成績/低成績の銘柄・セクター検出
  - `propose_filters_from_individuality`: sector_filter / symbol_whitelist / symbol_blacklist 提案 (F039 連携)
  - `scan_all_patterns_for_individuality`: 全パターン一括スキャン
- `agents/pattern_research.py`: `PatternResearchAgent` に 3 メソッド追加
  - `research_sector_individuality`
  - `propose_symbol_filter_for_pattern`
  - `scan_individuality_for_active_patterns`

### キーポイント

- 自動適用なし: F039 sector_filter 適用は F038 ApprovalManager 経由

## テスト

- `tests/agents/test_sector_research.py`: 22 ケース全 PASS

## 関連リンク

- 要件書: [[FIRE_要件書_第21章_Pattern_Store___Feature_Store_運用方針]], [[FIRE_要件書_第41章_セクター_銘柄個別性管理方針]]
- 要件 ID: R-21-15, 第 41 章 (穴 12 対策)
- 関連: [[F035_Pattern_Research_Agent]], [[F036_Live_Research_Log]], [[F039_セクター銘柄フィルタ]]
- コード: `~/fire/agents/sector_research.py`
