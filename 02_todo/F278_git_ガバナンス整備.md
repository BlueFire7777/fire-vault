---
id: F278
phase: P9: 運用・保守
priority: 最優先
status: 未着手
owner: Fujiwara
depends_on: [F275, F276, F277]
chapter: "なし (運用整備)"
created: 2026-05-04
updated: 2026-05-04
---

# F278: git ガバナンス整備

## 概要

Stage 3 開始前に必須となる git リポジトリ状態の整備。Codex review 体制
(523af4d、2026-05-04) より前のコミット遡及レビュー、untracked ファイル整理、
.gitignore 整備、dev → main マージを段階的に実施する。

## タスク詳細

設計議論記録は [[../03_design/git_governance_2026-05-04]] に集約済み。

### Phase 構造

| Phase | 内容 | 工数 |
|---|---|---|
| 1 | untracked 仕分け + .gitignore 整備 | 0.5 日 |
| 2 | 重要モジュール段階 commit (Codex pre-commit 通過) | 0.5 日 |
| 3 | 過去 5 本コミット遡及レビュー (F050/F051/F053/F141/F142、F144 オプション) | 0.5 日 |
| 4 | dev → main PR + Codex 全体レビュー + git tag (v0.1-stage3-ready) | 0.5 日 |
| 5 | .gitignore 仕上げ | 0.25 日 |
| 6 | F271 v1.2 への次回改訂検討 (§ 6-9 / 6-10 候補記録) | 0.25 日 |

合計 **2〜3 日** (Codex 致命的指摘で +0.5〜1 日延長可能性)。

### 確定方針 (詳細は設計記録参照)

- **遡及レビュー対象**: F050 / F051 / F053 / F141 / F142 (F144 オプション)
- **.gitignore 必須項目**: `.env*` / `*credential*` / `data/*.db-shm` /
  `.claude/` / `__pycache__/` / `.venv/` / `.DS_Store` / 他
- **untracked 危険ファイル**: `.env.backup.*` (.gitignore + commit 禁止)
- **untracked 重要ファイル**: agents/ / risk/ / notifications/ /
  evaluation/ / market_data/ / materials/ / scripts/jobs/ / etc
- **dev → main マージ戦略**: 全整理完了後一括 PR、Codex 全体レビュー、
  `git tag v0.1-stage3-ready`
- **F271 仕様書改訂**: v1.1 (Codex review 結果項目追加) は F275 で対応済 (`aab0aac`)、
  F278 では v1.2 への次回改訂候補 (§ 6-9 / 6-10) を vault に記録するのみ

### F271 v1.2 改訂候補 (Phase 6 で記録)

- **§ 6-9 候補**: 「コード読解せず仮説を立てる」(F273/F274 で「target_patterns
  1 件のみ」を誤検知した経緯。F275 Phase 1 で実体調査して撤回)
- **§ 6-10 候補**: 「git untracked を放置して長期作業」(F267 で scripts/jobs
  全体が untracked のまま F273/F274/F275 を進めた経緯)

## 成果物

(F278 着手時に詳細化)

予定:
- ~/fire/.gitignore (必須項目反映)
- ~/fire dev branch: 全 untracked ファイルを段階 commit (10〜15 commit 想定)
- ~/fire main branch: dev からマージ、`v0.1-stage3-ready` tag
- ~/fire-vault/03_design/F278_implementation_<date>.md (新規、Phase 別実施記録)
- ~/fire-vault/02_todo/F278_*.md (本ファイル更新)

## 進捗チェックリスト

(F278 着手時に詳細化)

- [ ] Phase 1: untracked 仕分け + .gitignore 整備
- [ ] Phase 1: `.env.backup.*` の物理削除 or 安全な場所へ移動
- [ ] Phase 2: agents/ + risk/ + notifications/ + evaluation/ + market_data/ +
      materials/ + scripts/jobs/ 等を段階 commit (Codex pre-commit 通過)
- [ ] Phase 2: simulation/paper_live/{batch_replay,live_advisory_check,scheduler}.py +
      cli.py / models.py / position.py / tick.py 等を commit
- [ ] Phase 3: F050 / F051 / F053 / F141 / F142 の遡及 Codex review
- [ ] Phase 3: 致命的指摘あれば fix commit (リバートはしない)
- [ ] Phase 4: dev → main PR (タイトル `Stage 3 ready: F267-F275 系統合`)
- [ ] Phase 4: Codex `/codex:review --base main` + 重要領域 adversarial-review
- [ ] Phase 4: main マージ + `git tag v0.1-stage3-ready` + `git push --tags`
- [ ] Phase 5: 仕上げ (取り違え commit があれば `git rm --cached`)
- [ ] Phase 6: F271 v1.2 改訂候補 (§ 6-9 / 6-10) を vault に記録

## 完了条件

- [ ] `git status -uno` で modified / untracked が clean
- [ ] dev branch が main にマージされ、`git tag v0.1-stage3-ready` が存在
- [ ] 過去 5 本コミット (F050/F051/F053/F141/F142) の Codex review 完了、
      致命的指摘 0 件 (or 修正済)
- [ ] `.gitignore` に必須項目すべて反映 (機密ファイル混入リスクなし)
- [ ] F271 v1.2 改訂候補が vault に記録済 (実改訂は次回事故時)
- [ ] Stage 3 開始 (F266 ゲート) の git 側前提条件すべて満たす

## 着手時期

**F275 / F276 / F277 完了後、Stage 3 開始 (F266) 前必須**

理由:
- F276 (positions seeding + F104) で更なるコード追加が見込まれる
- F277 (paper_live 例外伝播 + マルチプロセス cache) で patterns/ +
  simulation/paper_live/ の大幅修正が見込まれる
- F278 を先に実施しても、その後の F276/F277 修正で再度整理が必要
- → 実装系 3 タスク完了後に一括整理が効率的

ただし、F277/F276 が untracked ファイル参照に依存する場合、F278 を先行する
選択肢も Fujiwara 判断あり。

## 関連リンク

- 設計記録: [[../03_design/git_governance_2026-05-04]]
- 完了基準 v1.1: [[../03_design/task_completion_criteria]]
- 前提タスク: [[F275_SimilarityEngine_最適化]] /
  [[F276_events達成_positions_seeding]] /
  [[F277_paper_live例外伝播_マルチプロセスcache]]
- 後続: Stage 3 移行 (F266)
- 関連既存タスク: [[F271_タスク運用ルール改訂]] (v1.1 で同期済、v1.2 改訂候補は
  本タスク Phase 6 で記録)
- 運用ルール: [[../タスク運用ルール]]
