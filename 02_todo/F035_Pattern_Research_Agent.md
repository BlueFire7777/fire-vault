---
id: F035
phase: P1: Pattern/Feature Store
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F029]
chapter: "21"
created: 2026-04-26
updated: 2026-04-26
---

# F035: Pattern Research Agent

## 概要

`agents/pattern_research.py` 新規作成。パターンの新規発掘・既存パターンの調査を担当するエージェント。

## 実装内容

### 主要モジュール

- `agents/pattern_research.py`: `PatternResearchAgent` クラス
  - F038 連携: `submit_decay/rehab/adopt_approval_request()` (F038 で追加)
  - F039 連携: `research_sector/symbol_individuality()` (F039 で追加)
  - F243 連携: `research_sector_individuality` / `propose_symbol_filter_for_pattern` / `scan_individuality_for_active_patterns` (F243 で追加)

## 関連リンク

- 要件書: [[FIRE_要件書_第21章_Pattern_Store___Feature_Store_運用方針]]
- 関連: [[F036_Live_Research_Log]], [[F038_パターン採用承認フロー]], [[F039_セクター銘柄フィルタ]], [[F243_Sector_Symbol特化研究]]
- コード: `~/fire/agents/pattern_research.py`
