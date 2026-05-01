# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## このリポジトリについて

`~/fire-vault` は Obsidian Vault であり、日本株 AI エージェントシステム **FIRE** の設計・開発・運用記録を管理する場所。コードリポジトリは別途 `~/fire`（GitHub: BlueFire7777/fire, Private）に存在する。

## Vault 構造と役割

| ディレクトリ | 内容 |
|---|---|
| `01_requirements/` | 要件書 v3.3（第01〜41章の .md 分割版） |
| `02_todo/` | タスクごとの実装メモ（F001〜F999） |
| `03_design/` | 設計書（Pattern Store, Feature Store, Agent 仕様など） |
| `04_daily/` | 日次メモ |
| `05_weekly_review/` | 週次レビュー |
| `06_monthly/` | 月次レビュー |
| `07_incidents/` | 障害・事故・気づき |
| `08_reference/` | 参考資料 |
| `タスク運用ルール.md` | タスク管理・Git・Obsidian の運用規約 |

## FIRE システムの全体アーキテクチャ（v3.3）

**3層ハイブリッド構造**（詳細: `01_requirements/FIRE_要件書_第04章_*`）:

```
[第1層] OpenClaw on Mac mini M4 Pro
         常駐・スケジュール・並列実行・自動復旧・セッション永続化
         ↓
[第2層] Claude Code + 10 エージェント
         Market Intelligence / Daytrade Selection / Swing / Long-term /
         Portfolio Balance / Trade Decision / Monitoring & Alert /
         Review & Journal / Goal Tracking / Evaluation
         ↓
[LINE通知 / Dashboard]
         ↓
[第3層] 人間（Fujiwara さん）が手動発注・手動クローズ
         ↓
[約定結果] M+2 以降は Gmail API 自動取込
```

**重要な設計判断**:
- Computer Use による自動発注は**採用しない**（v3.3 で撤回）
- 強制クローズは LINE 緊急アラート（14:45/14:55/15:05/15:10/15:15）→ 人間対応
- Evaluation Agent は**提案のみ**（自動反映禁止）
- 崩してはならない前提 9 点: `タスク運用ルール.md` § 4 を参照

## 実装の着手順（絶対原則）

1. **OpenClaw 基盤が最初**（全ての土台）
2. **Pattern Store / Feature Store が最優先**（エッジの本質）
3. LINE 通知基盤は Phase 2 終盤から並行開発可
4. 約定取込は M+2 以降（初期は LINE 手動報告）
5. **Stage 飛ばし禁止**（Stage 0 → 1 → 2 → 3 を順守）

詳細は `01_requirements/FIRE_要件書_第40章_*`（実装ロードマップ）を参照。

## タスク管理

- **正本**: [Google Sheets TODO_Master](https://docs.google.com/spreadsheets/d/1Zf0rmQHxuBLWsBriV3hKH51eGiJ5u96b/edit?gid=1782363065#gid=1782363065)
- ステータス更新（未着手 → 実装中 → 完了）は Google Sheets のみで行う
- Obsidian の `02_todo/` ノートは詳細メモ・実装ログ用（ステータスの正本ではない）
- タスク ID（F001〜F999）は Git commit・Branch・PR タイトル・Obsidian ノート名すべてで引用する

## Git 運用（コードリポジトリ `~/fire` 側）

コミットメッセージ形式: `<type>(<task_id>): <subject>`（例: `feat(F242): OpenClaw 初期セットアップ`）

type: `feat` / `fix` / `docs` / `refactor` / `test` / `chore`

ブランチ: `main`（本番）/ `dev`（統合）/ `feature/F242-openclaw-setup`（タスクID含む）

FIRE リポジトリ専用の Git 設定（業務用と分離）:
```bash
cd ~/fire
git config user.name "BlueFire7777"
git config user.email "t.fujiwara199007@gmail.com"
```

## Obsidian ノートの書き方

- 要件書ノート（`01_requirements/`）は frontmatter に `type: requirement_chapter`, `chapter: "XX"`, `version: v3.3` を付与
- タスクノート（`02_todo/`）は frontmatter に `id`, `phase`, `priority`, `status`, `owner`, `depends_on`, `chapter` を付与
- ノート間リンクは `[[ファイル名|表示テキスト]]` の Wikilink 形式
- 各要件章末尾に `R-XX-XX` 形式の要件 ID を列挙（TODO 表の RTM と対応）

## 現在のフェーズ（P0: 基盤・環境構築）

- 開発マシン: Mac mini M4 Pro（64GB, 12core/16GPU, 1TB SSD）、macOS
- 次の着手タスク: F017 → F242（OpenClaw セットアップ）→ F022（FIRE Runner 骨組み）
- コードリポジトリ `~/fire` は F004 で初期化予定

## Obsidian Skills（導入済み）

`.claude/skills/` に kepano/obsidian-skills を導入済み。

- obsidian-markdown: Obsidian Flavored Markdown（wikilink, embed, callout, properties）
- obsidian-bases: .base ファイルでの集計・フィルタ・数式
- json-canvas: .canvas ファイル
- obsidian-cli: Obsidian CLI 操作
- defuddle: Web 記事のクリーン Markdown 抽出

ノート作成・編集時は上記 skill の記法に準拠すること。

## 開発ログ(log.md)

重要なイベントは vault 直下の `log.md` に追記する。

### 形式
`## [YYYY-MM-DD] type | summary` で見出し開始、本文で詳細を箇条書き。

### type 一覧
setup / decision / ingest / incident / lint / milestone

### 追記タイミング
- 設計判断が確定したとき
- 章を追加・大幅修正したとき
- 障害・想定外事象が発生したとき
- ツール導入・環境変更したとき
- フェーズ移行・大きなタスク完了時

### 検索
- 直近5件: `grep "^## \[" log.md | tail -5`
- type別: `grep "decision" log.md`

## Lint(健康チェック)

vault が育つにつれ発生する以下の問題を、定期的(月1回想定)に Claude Code に検出させる。

### チェック項目
1. **章間の矛盾**: 第04章と第40章で設計判断が食い違っていないか
2. **古いクレーム**: v3.2時点の記述が v3.3 に更新されていないか
3. **孤立ページ**: 02_todo/ にあるのに wikilink で参照されていないタスクメモ
4. **データギャップ**: 実装メモはあるのに対応する設計書が無い箇所
5. **相互参照漏れ**: 関連する章同士に wikilink が張られていない箇所

### 実行依頼の例
「fire-vault 全体に lint をかけて、上記5項目をチェックしてください。
 結果は log.md に `## [YYYY-MM-DD] lint | 結果サマリ` として追記し、
 詳細レポートを 07_incidents/lint_YYYY-MM-DD.md に出力してください」

### 結果の扱い
- 軽微な問題: その場で修正提案を受ける
- 設計判断レベルの矛盾: 必ず人間(Fujiwara)が判断、自動修正禁止
