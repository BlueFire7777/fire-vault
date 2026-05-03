---
id: F119
phase: P3 (Evaluation)
priority: 最優先
status: 完了 (Phase 1, 2, 3 全完了)
owner: Fujiwara
depends_on: [F035, F036, F051, F236]
chapter: "13"
created: 2026-05-02
updated: 2026-05-02
---

# F119: Evaluation Agent (3 段階分割)

## 全体スコープ (第 13 章 R-13-02〜10)

候補抽出 / 執行 / リスク管理 / システム / エッジ別の 5 カテゴリで FIRE 全体を
自己評価するエージェント。改善提案書を作成 → 承認制で反映。

## 段階分割

| Phase | 内容 | 工数 | 状態 |
|---|---|---|---|
| **Phase 1** | データ集計レイヤ (5 カテゴリのメトリクス計算) | 1.5 日 | **完了 (2026-05-02)** |
| **Phase 2** | 評価ロジック + 提案書生成 (Markdown) | 1.5 日 | **完了 (2026-05-02)** |
| **Phase 3** | 承認フロー + LINE 通知 + Markdown 保存 | 1 日 | **完了 (2026-05-02)** |

## Phase 1 スコープ (今回完了)

### 5 カテゴリのメトリクス

1. **CandidateExtractionMetrics** (R-13-02): live_research_log から候補総数 /
   実行率 / 勝率
2. **ExecutionMetrics** (R-13-03): paper_live_positions から
   TP/SL/force_close 比率 + 平均保有時間
3. **RiskManagementMetrics** (R-13-04): 日次最大損失率 / 平均ロット率 /
   max_concurrent_positions
4. **SystemMetrics** (R-13-05): tick / features / patterns / log の件数 +
   簡易健全性スコア (1.0 - 各種減点)
5. **EdgeMetrics** (R-13-06): pattern_id 別の n_entries / win_rate /
   expectancy / max_drawdown

### 設計方針 (確定済み)

- **配置**: `evaluation/` パッケージ (agents/ ではなく独立)
- **純粋関数**: aggregators.py の関数群は副作用なし、テスト容易
- **dataclass のデフォルト値**: 全フィールドに default、部分集計でも動く
- **後知恵バイアス対策**: as_of_dt 引数でフィルタ (Phase 2 で活用予定、
  Phase 1 では skipped_pseudo_win_rate=None のまま)
- **テーブル不在時の堅牢性**: `_table_exists` ヘルパーで吸収、健全性スコアで
  減点
- **JST タイムゾーン**: as_of_dt を JST に変換してから期間決定

### 実テーブル確認結果 (仕様書との差分)

| 仕様書推測 | 実体 |
|---|---|
| `live_research_log.dt` | `live_research_log.observed_at` |
| `paper_live_positions.entry_price` | `paper_live_positions.avg_entry_price` |
| `paper_live_positions.exit_reason` | `paper_live_positions.close_reason` |
| `patterns.status` IN ('approved_active', 'paper_validating', 'watchlist') | 実 status: 'approved_active' / 'candidate' (他は将来追加) |
| `patterns.metadata` (TEXT) | F029 拡張で `pattern_name` `layer` `rank` `lane` 等が直接カラム化済 |

## 成果物

### 新規ファイル

- `evaluation/__init__.py`: パッケージ export
- `evaluation/metrics.py`: 7 dataclass (EvaluationMetrics + 5 カテゴリ + PatternEdgeMetrics)
- `evaluation/aggregators.py`: `aggregate_evaluation_metrics` +
  `_resolve_period` + 5 集計関数 + `_table_exists`
- `tests/evaluation/__init__.py`
- `tests/evaluation/test_aggregators.py`: 24 ケース

### テスト

- 新規 24 件 (Phase 1 関連)
  - TestPeriodResolution: 4 件 (daily/weekly/monthly + 不正値)
  - TestEmptyDB: 4 件 (空 DB / period / no account / 健全性 0)
  - TestCandidateMetrics: 2 件 (実行なし / 実行あり勝率)
  - TestExecutionMetrics: 4 件 (open のみ / TP/SL hit / force_close / 保有時間)
  - TestRiskMetrics: 3 件 (損失なし / 損失率 / 平均ロット)
  - TestSystemMetrics: 2 件 (full health / partial)
  - TestEdgeMetrics: 3 件 (単一 / 複数 / status filter)
  - TestSmoke: 2 件 (E2E + 全 period)
- 累計: 755 → **779 PASS** (+24)
- 既存 755 への影響: **0 件** (新規パッケージ、既存ファイル変更なし)

### Smoke 結果 (real DB)

```
=== F119 各 period での集計 ===
daily   : 2026-05-03 - 2026-05-03 | entries=0, patterns_active=1, features=0,  health=0.2
weekly  : 2026-04-27 - 2026-05-03 | entries=0, patterns_active=1, features=39, health=0.5
monthly : 2026-05-01 - 2026-05-03 | entries=0, patterns_active=1, features=0,  health=0.2
```

initial 9 patterns の per_pattern_metrics (n_entries=0) も全件出力確認。

## Phase 2 完了 (2026-05-02)

### 実装

- `evaluation/thresholds.py`: `EvaluationThresholds` (frozen=True) +
  5 カテゴリ閾値 dataclass + `DEFAULT_THRESHOLDS`
- `evaluation/judgments.py`: `JudgmentLevel` enum (良/通常/悪/重大) +
  `judge_metrics()` + 5 _judge_xxx + _judge_patterns (第 31 章劣化候補)
- `evaluation/proposals.py`: `Proposal` + `ProposalCategory` (R-13-08 6 カテゴリ)
  + `ProposalPriority` + `generate_proposals()` + 4 _propose_for_xxx
- `evaluation/report.py`: `render_markdown_report()` + 内部ヘルパ
  (3 行要約 / 良点 / 悪点 / 提案 / メトリクス表 / 劣化候補 / フッタ)

### スコープ達成

- R-13-02〜06: 5 カテゴリ判定 (4 レベル)
- R-13-07: NotebookLM 風 Markdown (3 行要約 + 良/悪/提案)
- R-13-08: 6 カテゴリの提案 (要承認マーク必須)
- R-13-09: 時間軸別 (日次/週次/月次) で features_per_day_min × 期間日数を適用
- R-13-10: as_of_dt 徹底 + フッタに「○○以前のデータのみ使用」明示

### 設計上の決定

- **`_max_level` の優先順位**: CRITICAL > BAD > GOOD > NORMAL
  (悪い側を優先しつつ、悪がなければ良があれば良を採用)
- **空 metrics 時の挙動**: candidates/entries=0 でも完走、各カテゴリ NORMAL
  (システムだけは空 DB なら CRITICAL になる、これは正常)
- **decay_candidate**: pattern_decay_threshold (0.30) 未満で True、第 31 章
  DEATH NOTE 検討対象として report に「劣化候補パターン」セクション出力
- **proposal sort**: HIGH > MEDIUM > LOW で並び替え
- **後知恵バイアス対策**: Phase 1 aggregator が as_of_dt で期間フィルタ済、
  Phase 2 は as_of_dt_iso を JudgmentResult / Markdown フッタに記録 (透明性)

### テスト

- 新規 31 件
  - test_thresholds.py: 5 件 (網羅 / 値整合 / frozen / カスタム / lot range)
  - test_judgments.py: 14 件 (5 カテゴリ × 良/悪/重大 + 期間別 + 全空 + as_of)
  - test_proposals.py: 7 件 (4 カテゴリ + 全 normal で 0 件 + 要承認 + 優先度)
  - test_report.py: 5 件 (必須セクション / 3 行 / 提案 0 件 / フッタ / 劣化)
- 累計: 779 → **810 PASS** (+31)
- 既存 779 への影響: **0 件** (新規ファイルのみ)

### Smoke 結果 (real DB、daily)

```
# FIRE 評価提案書 (日次 2026-05-04 〜 2026-05-04)

## 3 行要約
1. **重大判定 1 件** — 即時対応推奨
2. 候補抽出: 通常 / 執行: 通常 / リスク管理: 通常 / システム: 重大 / エッジ別: 通常
3. 改善提案なし

## 良かった点
- (該当なし)

## 悪かった点
- [システム] 🚨 健全性スコア低 (0.20 < 0.4)
- [システム] 🚨 features 件数不足 (0 < 5 = 1日 × 5件/日)

## 改善提案 (承認待ち 0 件)
(改善提案はありません)

## メトリクス詳細
| カテゴリ | 判定 | 主要メトリクス |
| 候補抽出 | 通常 | candidates_total=0, ... |
| ... |
| システム | 重大 | system_health_score=20.0%, ... |

---
*生成時刻: 2026-05-03T16:00:00+00:00*
*後知恵バイアス対策: 2026-05-03T16:00:00+00:00 以前のデータのみ使用 (R-13-10)*
```

## Phase 3 完了 (2026-05-02)

### 実装

- `evaluation/approval.py`: `EvaluationApprovalManager` + `StoredProposal` +
  `EvaluationDecision` enum + `ensure_eval_proposals_schema`
- `evaluation/orchestrator.py`: `run_evaluation` + `EvaluationRunResult` +
  `_format_line_message`
- `evaluation_proposals` テーブル新設 (CREATE TABLE IF NOT EXISTS、冪等)
- `~/fire/reports/.gitkeep` + `.gitignore` に `reports/*` 追加

### スコープ達成

- F119 専用 evaluation_proposals テーブル (F038 と独立、pattern_id 不要)
- request_id `EP-YYYY-NNNN` 年内連番
- submit_proposals / approve / reject / defer / list_pending / get API
- pending → 他状態への遷移のみ許可 (再変更ガード)
- run_evaluation: Phase 1+2+3 統合エントリ (skip_db / save_report /
  send_line / reports_dir すべて引数で制御)
- LINE REPORT 部屋通知 (3 行要約 + カテゴリ別判定 + 提案数 + Markdown パス)
- LINE 送信失敗時は line_error に記録 (例外で止まらない)

### 設計上の決定

- **`aggregate_evaluation_metrics` の run_id**: spec が省略していたため
  `DEFAULT_RUN_ID = "EVAL-DEFAULT"` を導入。orchestrator から runtime で
  上書き可能。
- **proposals が空なら DB 書き込みなし**: `submit_proposals` が空リスト時
  no-op、`evaluation_proposals` テーブルが存在しない初期状態でも
  `skip_db=True` のテストが完走 (manager 自体を呼ばないため)。
- **report_path は絶対パス**: LINE メッセージにそのまま含めて Fujiwara が
  Mac mini で開ける。
- **R-13-08 厳守**: 承認 API は status を変更するだけ、設定への自動反映は
  Phase 3 では一切実装しない。

### テスト

- 新規 18 件
  - test_approval.py: 12 件 (schema / submit / id 連番 / 4 状態遷移 / ガード /
    list / get)
  - test_orchestrator.py: 6 件 (空 DB / skip_db / save_report=False /
    提案あり / LINE 失敗 / format ブランチ)
- 累計: 810 → **828 PASS** (+18)
- 既存 810 への影響: **0 件** (新規ファイルのみ)

### Smoke 結果 (real DB、daily)

```
period: 日次 (2026-05-04 - 2026-05-04)
提案数: 0 / request_id: []
report_path: /Users/bluefire/fire/reports/eod_2026-05-03.md  ← 生成済
line_sent: False
line_error: Room ID not configured: LINE_ROOM_REPORT  ← .env 未設定で正常

=== LINE メッセージ ===
【FIRE 評価レポート (日次)】
期間: 2026-05-04 〜 2026-05-04

⚠️ 重大判定 1 件 — 即時対応推奨
候補: 通常 / 執行: 通常 / リスク: 通常 / システム: 重大 / エッジ: 通常

改善提案: なし

詳細: /Users/bluefire/fire/reports/eod_2026-05-03.md
```

## 第 13 章 R-13-02〜10 達成状況

| 要件 | 達成 |
|---|---|
| R-13-02〜06 5 カテゴリ判定 | ✓ Phase 1+2 |
| R-13-07 NotebookLM 風 Markdown | ✓ Phase 2 |
| R-13-08 6 カテゴリ提案 (要承認) | ✓ Phase 2 + Phase 3 (DB 記録) |
| R-13-09 時間軸別判定 | ✓ Phase 2 (period 引数 + features_per_day_min × 期間日数) |
| R-13-10 後知恵バイアス対策 | ✓ as_of_dt 徹底 + フッタ明示 |

## 残: Dashboard 関連 (R-13-11〜19)
→ F090 Backend API で別タスク。F119 単体としては完全閉ループ完成。

## スコープ外 (Phase 2/3 へ)

- 評価ロジック (「勝率 < 30% は悪い」等の閾値判定) → Phase 2
- 改善提案書のフォーマット (Markdown 出力) → Phase 2
- 承認フロー連携 (F038 流用) → Phase 3
- LINE 通知 (REPORT 部屋) → Phase 3
- 自動反映 (Pattern 状態変更等) → Phase 3
- LLM 経由の自然文生成 → 将来検討
- Dashboard 連携 → F090 Backend API
- OpenClaw evaluation agent への登録 → F260
- F132/F133/F140 reject 履歴記録 → 別タスク (現状記録テーブル無し)

## 関連リンク

- 要件書: 第 13 章 R-13-02〜10、第 31 章 (Pattern 劣化判定)
- 関連: [[F035_Pattern_Research_Agent]] (パターン蓄積側) /
  [[F036_Live_Research_Log]] (主要データソース) /
  [[F051_仮想建玉]] (paper_live_account/_positions データソース) /
  [[F252_Pattern劣化判定ルール]] (1 ヶ月運用後)
- コード: `evaluation/metrics.py` / `evaluation/aggregators.py`
