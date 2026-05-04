# git ガバナンス整備計画 (F278 設計記録)

**作成日**: 2026-05-04
**起票タスク**: [[02_todo/F278_git_ガバナンス整備|F278]]
**位置付け**: F275 完了直後の整理作業として、Stage 3 開始前に必須となる git
リポジトリ状態の整備計画
**ステータス**: 設計議論済 / 実装未着手 / F275/F276/F277 完了後着手

---

## 1. 起票背景

### 現状の git リポジトリ状態 (2026-05-04 時点)

- **dev branch**: main から大幅乖離 (F267 / F273 / F274 / F275 を含む 6 commit ahead)
- **untracked ファイル多数**: agents/ / risk/ / notifications/ / evaluation/ /
  market_data/ / materials/ / scripts/jobs/ 等が dev branch に未 commit
- **modified ファイル**: simulation/paper_live/cli.py / models.py / position.py /
  tick.py / 他 8 ファイルが modified だが未 commit
- **危険ファイル**: `.env.backup.20260503_180400` が untracked、commit すれば
  漏洩リスク
- **Codex review 体制**: `523af4d` (2026-05-04) で導入、それ以前のコミットは
  Codex review 未実施

### 問題

- Stage 3 開始前 (= 実取引開始前) に git 状態を clean にする必要
- F267/F273/F274/F275 のリリース内容を main にマージする必要
- それ以前の重要コミット (F050/F051/F053/F141/F142) も Codex review 遡及推奨
- `.gitignore` 整備が不十分 (機密ファイル混入リスク)

---

## 2. 確定方針 (Fujiwara 確認済 2026-05-04)

### 2-1. 遡及レビュー対象 5 本 (F144 オプション)

| Commit / Task | 重要性 | 理由 |
|---|---|---|
| F050 (Paper Live Mode 本体) | 高 | Stage 2 の中核、Codex review 未実施 |
| F051 (仮想建玉計算) | 高 | 損益計算の正本、計算誤差は致命的 |
| F053 (Semi Auto 昇格基準) | 高 | Stage 3 移行ゲート、判定誤差は致命的 |
| F141 (ハイブリッド執行方針) | 高 | 実取引の執行モード、不具合は通知漏れに直結 |
| F142 (発注前フィルタ) | 高 | 異常入力の遮断、不具合は誤発注に直結 |
| F144 (執行品質 3 段監視) | 中 | スリッページ監視、F141/F142 で大半カバー、オプション扱い |

→ 5 本必須 + F144 オプションで遡及レビュー実施。

### 2-2. .gitignore 必須項目

```gitignore
# 機密 / 認証
.env
.env.*
!.env.example
*credential*
*token*
*.pem
*.key

# DB ロックファイル (本体 fire.db は track、WAL/SHM は ignore)
data/*.db-shm
data/*.db-wal
data/*.db-journal

# IDE / Editor / OS
.claude/
.vscode/
.idea/
.DS_Store
Thumbs.db

# Python
__pycache__/
*.pyc
*.pyo
.venv/
venv/
.pytest_cache/
.mypy_cache/

# ログ / 一時ファイル
*.log
/tmp/
.cache/
*.bak
```

### 2-3. untracked 危険ファイル

- `.env.backup.20260503_180400`: **commit 禁止** + .gitignore 追加 (`.env.*` ルール)
- 古い `.env.backup.*` 群: 同様に `.env.*` ルールでカバー、commit 禁止徹底

### 2-4. untracked 重要ファイル (Phase 2 で commit 対象)

```
agents/                              # Stage 3 必須エージェント実装
risk/                                # 第 5/17 章 リスク管理 + 執行品質
notifications/                       # F236 5 部屋構成
evaluation/                          # F119 Evaluation Agent
market_data/                         # F100 系
materials/                           # F101 系
scripts/jobs/                        # F267 / F100 historical / 他 batch
scripts/seed_pattern_layer1.py       # F273 seeder
scripts/e2e_smoke_test.py            # F058
scripts/emergency_alert.py           # F236 launchd 連携
scripts/wrapper/                     # CLI wrapper 系
scripts/setup/migrate_market_data.py # F100 schema
scripts/setup/migrate_paper_live_tp_sl.py # F054 schema
scripts/setup/seed_initial_patterns.py # 初期 patterns seed
simulation/paper_live/batch_replay.py # F230
simulation/paper_live/live_advisory_check.py # F241
simulation/paper_live/scheduler.py   # F057
docs/launchd/                        # 緊急アラート plist
docs/openclaw/agents/                # F261 IDENTITY.md 系
docs/e2e_test_report.md              # F058 結果
tests/agents/test_*.py               # 該当エージェントテスト
tests/evaluation/                    # F119 テスト
tests/market_data/                   # F100 テスト
tests/materials/                     # F101 テスト
tests/notifications/                 # F236 テスト
tests/patterns/conftest.py           # F029-F243 共通 fixture
tests/patterns/test_reproducibility.py # F032
tests/patterns/test_similarity.py    # F031
tests/risk/                          # 第 5/17 章テスト
tests/scripts/                       # F058 / F267 テスト
tests/simulation/test_*.py           # F050-F058 / F230 / F241
reports/                             # eod_*.md (eod レポート、機密性中)
requirements.txt                     # 依存定義
```

### 2-5. 段階 commit 順 (Phase 構造、§ 3 参照)

各 commit は **Codex pre-commit 必須通過 + Conventional Commits** 形式。

### 2-6. dev → main マージ戦略

- 全整理完了後に **一括 PR** (dev → main)
- Codex 全体レビュー (`/codex:review --base main` + 重要領域は adversarial-review)
- マージ後 `git tag v0.1-stage3-ready` で固定
- main マージ後に Stage 3 移行 (F266) を実施

### 2-7. F271 仕様書改訂 (v1.1 で完了済、F275 で対応)

[[task_completion_criteria]] v1.1 (commit `aab0aac`) で完了報告フォーマットに
「Codex review 結果」項目を追加済み。F278 では追加改訂不要 (v1.2 への次回
改訂は F278 Phase 6 で検討)。

---

## 3. F278 Phase 構造

### Phase 1: untracked 仕分け + .gitignore 整備 (0.5 日)

- 全 untracked ファイルを確認、commit 対象 / .gitignore 対象 / 削除対象に分類
- `.gitignore` を § 2-2 の必須項目で更新
- 危険ファイル (`.env.backup.*`) を `.gitignore` 対象 + 物理削除 or 移動
- **重要: dev branch で実施**、Codex pre-commit 通過

### Phase 2: 重要モジュール段階 commit (0.5 日)

§ 2-4 の untracked 重要ファイルを以下の順で commit:

1. agents/ + tests/agents/
2. risk/ + tests/risk/
3. notifications/ + tests/notifications/
4. evaluation/ + tests/evaluation/
5. market_data/ + tests/market_data/
6. materials/ + tests/materials/
7. simulation/paper_live/{batch_replay,live_advisory_check,scheduler}.py +
   tests/simulation/test_{batch_replay,live_advisory_check,scheduler}.py
8. その他 modified files (cli.py / models.py / position.py / tick.py / etc)

各 commit:
- Codex pre-commit 通過 (要 `--no-verify` の場合は理由を commit message に記載、
  ただし F278 では原則禁止)
- Conventional Commits (例: `feat(F236): notifications/ 5 部屋構成本体追加`)

### Phase 3: 過去 5 本コミット遡及レビュー (0.5 日)

§ 2-1 の F050/F051/F053/F141/F142 (オプションで F144) について:

- 各 commit に対して `git show <hash>` の差分を Codex `/codex:review` で読ませる
- 致命的指摘あり → fix commit を別途追加 (リバートはしない、追加修正で対応)
- 指摘なし → vault に「過去 commit 遡及 review 結果: 問題なし」を記録

### Phase 4: dev → main PR + Codex 全体レビュー + tag (0.5 日)

- `git push origin dev` を最終確定
- GitHub で dev → main PR 作成 (タイトル: `Stage 3 ready: F267-F275 系統合`)
- Codex `/codex:review --base main` で全体レビュー
- 重要領域 (LINE 通知 / launchd / aiohttp / SQLite migration) は
  `/codex:adversarial-review` を追加
- 致命的指摘なし → main マージ
- マージ後 `git tag v0.1-stage3-ready` を作成、push origin v0.1-stage3-ready

### Phase 5: .gitignore 仕上げ (0.25 日)

main マージ後の状態で `.gitignore` を最終調整:
- 取り違えで commit してしまったファイルがあれば `git rm --cached` + .gitignore 追加
- `.gitignore` 自体に過剰なルールがあれば整理

### Phase 6: F271 v1.2 への次回改訂検討 (0.25 日)

F278 自体の経験を踏まえ、F271 v1.2 で追加すべきアンチパターン事例を vault に
記録 (実改訂は次の重要事故発生時に v1.2 へ反映する流れ):

- **§ 6-9 候補**: 「コード読解せず仮説を立てる」(F273/F274 で「target_patterns
  1 件のみ」を誤検知した経緯。F275 Phase 1 で実体調査して撤回)
- **§ 6-10 候補**: 「git untracked を放置して長期作業」(F267 で scripts/jobs
  全体が untracked のまま F273/F274/F275 を進めた経緯)

→ F278 完了時点で記録、改訂は次回事故時に。

---

## 4. 工数想定

合計 **2〜3 日** (Phase 1〜6 を連続実施した場合):
- Phase 1: 0.5 日
- Phase 2: 0.5 日
- Phase 3: 0.5 日
- Phase 4: 0.5 日
- Phase 5: 0.25 日
- Phase 6: 0.25 日

ただし Codex 全体レビュー (Phase 4) で致命的指摘があると修正サイクルが入り、
+0.5〜1 日延長の可能性。

---

## 5. F278 着手時期

**F275 / F276 / F277 完了後、Stage 3 開始前必須**。

理由:
- F276 (positions seeding + F104) で更なるコード追加が見込まれる
- F277 (paper_live 例外伝播 + マルチプロセス cache) で patterns/ +
  simulation/paper_live/ の大幅修正が見込まれる
- → F278 を先に実施しても、その後の F276/F277 修正で再度整理が必要
- → 実装系 3 タスク完了後に一括整理が効率的

→ F277 完了 → F276 完了 → **F278** → Stage 3 移行 (F266) の順。

---

## 6. リスク

### リスク 1: Phase 4 main マージで Codex 致命的指摘多発

- 過去 commit (F267/F273/F274/F275) で計 8 件以上の Codex CRITICAL を経験
- main マージ時の全体レビューで未検出の指摘が出る可能性
- → fix commit のサイクルで対応、最悪 +1〜2 日

### リスク 2: Phase 5 .gitignore 仕上げで重要ファイルが ignore 漏れ

- `.gitignore` で `*.log` を ignore したが、運用ログを残す経路があれば困る
- ⇒ Phase 5 で全 ignore ルールを 1 つずつレビュー、不必要なものは外す

### リスク 3: F277/F276 が F278 に依存する状況

- F277 で paper_live 例外伝播設計をするとき、現状の untracked simulation/
  paper_live/ ファイルを参照する必要
- → F278 で commit してから F277/F276 着手のほうが安全な可能性
- → F278 を先行実施する選択肢も Fujiwara 判断あり

---

## 7. 関連リンク

- 完了基準: [[task_completion_criteria]]
- 起点: [[F275_similarity_optimization_complete_2026-05-04]] (F275 で
  --no-verify を 1 回使用、F278 で改訂方針を再評価)
- スタブタスク: [[02_todo/F278_git_ガバナンス整備]]
- 関連タスク: [[02_todo/F276_events達成_positions_seeding]] /
  [[02_todo/F277_paper_live例外伝播_マルチプロセスcache]] /
  [[02_todo/F279_休日専業モード設計仕様書]]
- log.md: 2026-05-04 milestone エントリ (F275 完了 + 設計議論記録 Vault 化)

---

(以上、F278 着手時に本書を起点に Phase 1 から進めること)
