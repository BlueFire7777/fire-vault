# FIRE-CODEX-R0 Parallel Development Workflow Design

## 目的

Claude Code本線開発とCodex並列開発を安全に併用するための運用ルールを定義する。

Codexは本線作業を加速する補助レーンとして扱い、独立した作業場とブランチで限定的な成果物を作成する。採用可否と本線統合はHQ判断を必須とし、Codex成果を自動的に本線へ混入させない。

## worktree構成

- `~/fire` = Claude Code本線
- `~/fire-codex` = Codex並列
- `~/fire-vault` = 記録

## ブランチ命名ルール

Codex作業ブランチは以下の形式に統一する。

```text
codex/<task-id>-<short-name>
```

例:

```text
codex/fire-safety-r1-artifact-leak-scanner
codex/f286-order-r1-spec-test-design
```

## Codexに渡してよいタスク

- docs
- tests
- read-only audit
- small helper
- safety scanner
- schema/design draft

## Codexに渡してはいけないタスク

- LINE本番送信
- token/secret
- DB write
- 楽天/証券/発注
- workflow/GitHub Actions
- Claude Codeと同一ファイルの同時編集
- 大規模refactor

## commit方針

- 個別commitにする
- まとめcommitは禁止
- `--no-verify`は禁止
- code変更がない場合はdocs commitのみ
- 変更対象ファイルをcommit前に確認する
- Codex成果は本線統合前提であっても、Codexブランチ上では独立した履歴として残す

## review方針

- Claude Code本線はCodex pre-commit reviewあり
- Codex側はCodex reviewなしでよい
- ただしHQ採用判断なしに本線mergeしない
- Codex成果は「採用候補」であり、HQまたはClaude Code本線が必要性、安全性、競合リスクを確認する

## merge/統合方針

- Codex成果はHQ部屋へ報告する
- HQが採用/保留/破棄を判断する
- 必要ならClaude Codeが本線へ統合する
- Codex側から`develop`、`production`、`staging`へ直接mergeしない
- 本線統合時はClaude Code側の作業状況を優先し、同時編集リスクがある場合は統合を後回しにする

## conflict防止

- 作業前に`git status`を確認する
- 作業対象ファイルを明示する
- 本線modifiedを触らない
- Claude Codeが作業中のファイルを触らない
- `fire-vault/log.md`更新は別commitにする
- `fire-vault/log.md`で衝突した場合は後回しにする
- `~/fire`のmodified/untrackedには触らない
- Codex作業開始時にbase branchとの差分範囲を把握する

## safety checklist

作業開始前:

- [ ] `pwd`が想定作業場である
- [ ] `git branch --show-current`が想定ブランチである
- [ ] `git status --short`で予期しない変更がない
- [ ] 作業対象ファイルを明示した
- [ ] Claude Code本線のmodified/untrackedに触らないことを確認した

作業中:

- [ ] LINE送信をしない
- [ ] DB writeをしない
- [ ] secret/envを読まない
- [ ] workflow/GitHub Actionsを変更しない
- [ ] 自動発注をしない
- [ ] 楽天証券操作をしない
- [ ] Computer Useをしない
- [ ] production/develop/staging DBに触らない
- [ ] TODO Excelを更新しない
- [ ] `--no-verify`を使わない

commit前:

- [ ] `git status --short`で変更対象が意図どおりである
- [ ] docsのみの作業ならdocs commitのみである
- [ ] 禁止ファイルを触っていない
- [ ] `fire-vault/log.md`を同一commitに混ぜていない

完了時:

- [ ] commit hashを記録した
- [ ] 作成/変更ファイルを記録した
- [ ] LINE送信なしを報告した
- [ ] DB writeなしを報告した
- [ ] secret/env未読を報告した
- [ ] workflow変更なしを報告した
- [ ] `--no-verify`未使用を報告した
- [ ] 本線modified未接触を報告した

## 完了報告テンプレート

```text
FIRE-CODEX task complete

- task:
- current_dir:
- branch:
- git status:
- commit hash:
- 作成/変更ファイル:
- log.md更新有無:
- LINE送信なし:
- DB writeなし:
- secret/env未読:
- workflow変更なし:
- --no-verify未使用:
- 本線modified未接触:
- HQ判断待ち:
- 次アクション候補:
```

## 初回Codex候補タスク案

- FIRE-SAFETY-R1 Artifact Leak Scanner
- F286-ORDER-R1 spec/test design
- F286-INTRA-R2 design draft
- F286-PNL-R1 schema draft

## 推奨運用

初回はdocs、tests、read-only audit、design draftに限定する。Codexに実装を渡す場合でも、DB write、外部送信、証券操作、workflow変更、secret/env参照を必要としない小さなhelperまたはscannerに限定する。

Codex成果は「そのままmergeするもの」ではなく「HQが採用判断する材料」として扱う。Claude Code本線の進行中ファイルと重なる場合は、Codex成果を破棄または後回しにする。
