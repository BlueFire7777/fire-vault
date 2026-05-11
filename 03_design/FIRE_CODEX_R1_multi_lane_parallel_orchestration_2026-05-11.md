---
type: design_doc
id: FIRE-CODEX-R1
title: Multi-Lane Parallel Development Orchestration
version: v1.1
date: 2026-05-11
owner: BlueFire7777 (Fujiwara)
status: 確定 (= v1.1、Codex を実装部隊化、運用開始)
chapter: ガバナンス / R-01-08 補強 / Codex レビュー運用拡張
related:
  - "[[../CLAUDE]] (vault)"
  - "[[../タスク運用ルール]]"
  - "[[../log]]"
  - F062-R5 (= 本番 Advisory 配信ループ)
  - F286-PNL-R1 (= 三段ガードを Codex CRITICAL で 2 度修正した実例)
---

# FIRE-CODEX-R1: Multi-Lane Parallel Development Orchestration Design

最終更新: 2026-05-11

## 0. 一行サマリ

**Claude Code 本線を PM / Architect / Integrator / Final Reviewer に
固定し、Codex を 5 種類の限定レーンとして並列活用する**。
LINE 本番送信 / production / develop DB write / token 参照は
Codex に絶対委譲しない。1 Codex task = 1 branch / 1 目的 / 1 成果物
/ 1 file ownership グループ、で衝突を構造的に防ぐ。

**v1.1 改訂**: Codex を **review 専用ではなく実装部隊** として
積極利用する方向に明確化。3-5 本の独立 task を並列で走らせて本線
開発速度を底上げする。実装レーン (L3) の標準運用 + safety scanner /
forbidden operation audit を初回投入候補に追加 (= §15-18)。

## 1. 背景と動機

### 1.1 これまでの実例から見えた論点

F286-PNL-R1 では Codex pre-commit が **2 件の CRITICAL** を即座に
指摘し、本線が即修正してから commit した。これは Codex を「最後の
2 つの目」として活かす運用が機能している実証。

一方で、複数の Codex を並列に走らせる場面はまだ無く、以下のリスクが
未明文化:

1. **同一ファイルへの同時編集競合** — 2 lane が同じ template に
   触ったらどちらの diff が正本か不明。
2. **責任境界の曖昧化** — Codex が本線判断 (LINE 本番送信 / DB
   write / merge 判断) を勝手に下す可能性。
3. **CRITICAL 検出の二重化漏れ** — 1 Codex で見落とした問題を別の
   Codex が拾う仕組みが無い。

### 1.2 本設計の目的

- **本線 (Claude Code) の指揮系統を明文化**:
  PM / Architect / Integrator / Final Reviewer の 4 役を本線が独占。
- **Codex の役割を 5 レーンに分割**:
  Design / Test / Implementation / Audit / Docs。
- **file ownership 表** で衝突を構造的に防ぐ。
- **プロンプト雛形 + 完了報告テンプレート** で運用を再現可能にする。

## 2. Claude Code 本線の役割定義

本線は **常時** 次の 4 役を兼ねる。Codex はいずれも代行しない。

| 役割 | 責務 | 権限 |
|---|---|---|
| **PM**             | 全体スケジュール / 並列タスクの起票 / 投入順序 / 中断判断 | タスク起票、Codex 投入、HQ 報告 |
| **Architect**      | 設計レビュー / 三段ガード / scope-A 値域 / schema 確定 | 設計 doc 承認、Schema 変更承認 |
| **Integrator**     | 複数 Codex 成果物の統合 / merge 順序 / コンフリクト解決 | 本線 merge 判断 |
| **Final Reviewer** | pre-commit / 回帰 / CRITICAL 最終判断 / 安全要件最終チェック | merge 拒否、commit 拒否 |

### 2.1 本線が **絶対** に独占する判断

| 判断 | 説明 |
|---|---|
| **HQ 報告作成**          | Fujiwara への完了報告は本線が必ず書く。Codex 出力をそのままコピーしない。 |
| **本線 merge 判断**       | feat/test/docs commit を develop に取り込む最終判断。Codex 出力は staging branch 経由を推奨。 |
| **pre-commit / regression 維持** | `.venv/bin/pytest` 全件 PASS を最終確認するのは本線。 |
| **LINE 本番送信判断**     | `--send --hq-approved-send` を渡す runner 起動は本線のみ。 |
| **CRITICAL 受領判断**     | Codex CRITICAL の即修正可否 / 設計差し戻し可否を本線が判断。 |
| **Workflow / GitHub 設定** | `.github/workflows/` / Actions / Secrets / Webhooks 触らない。Codex にも禁止。 |

## 3. Codex レーン定義 (5 種)

| Lane ID | 名前 | 主タスク | 典型成果物 | 並列度 |
|---|---|---|---|---|
| L1 | **Codex-Design**         | 設計案 / schema 提案 / 三段ガード提案 | design draft md (vault に置く前のもの) | 1 同時 |
| L2 | **Codex-Test**           | tests 追加 / regression 検査 | `tests/.../test_*.py` 単一ファイル | 2-3 並列可 |
| L3 | **Codex-Implementation** | 実装本体 (= isolated module 限定) | `module/*.py` 単一 module | 1 同時 (= 重要、merge 衝突防止) |
| L4 | **Codex-Audit**          | 既存コードレビュー / forbidden import 検査 | audit report md | 並列可 (= read-only) |
| L5 | **Codex-Docs**           | docs draft / vault 雛形 | `02_todo/*.md` / `03_design/*.md` 雛形 | 並列可 |

### 3.1 レーン別の典型タスクサイズ

| Lane | 1 task の上限目安 | 想定所要 (Codex) |
|---|---|---|
| L1 Design        | 設計 doc 1 本 (= 500-1000 行)        | 5-15 分 |
| L2 Test          | test ファイル 1 本 (= 100-300 行)    | 3-8 分 |
| L3 Implementation | module 1 本 / runner 1 本 (= 200-400 行) | 10-20 分 |
| L4 Audit         | 既存コードの 1 観点レビュー         | 3-10 分 |
| L5 Docs          | vault doc 1 本 (= 100-300 行)        | 3-8 分 |

複数レーンを跨ぐ大規模実装 (例: F286-PNL-R1 全体) は本線が **PM 視点**
で 5-10 個の sub-task に分割し、それぞれを別 Codex に投げる。

## 4. Codex に任せてよい作業

以下は **本線判断さえ正しければ** Codex 委譲して構造的に安全な
カテゴリ:

| カテゴリ | 例 | 理由 |
|---|---|---|
| **Isolated module**      | `notifications/listing_name_lookup.py` のような単一責務 module の新規作成 | 既存コードへの侵入なし、副作用は file 境界に閉じる |
| **Read-only helper**     | DB SELECT / 文字列整形 / 純関数 | DB write / 副作用なし |
| **Tests**                | `tests/.../test_*.py` 単一ファイル | 動作前に本線が `.venv/bin/pytest` で検証可能 |
| **Docs draft**           | `03_design/*.md` / `02_todo/*.md` の初稿 | 本線が最終 review / 修正してから merge |
| **Scanner**              | repo 横断 grep / ast 検査 / lint helper | 副作用なし、結果は report の md |
| **Schema proposal**      | DDL の提案 + CHECK 制約案 | 本線が DecisionStore 三段ガードに組み込む |
| **Dry-run runner**       | `default dry-run` + `--write` ガード付き runner | 副作用は --write 経路に限定、本線が --write 判断 |
| **Validation helper**    | pydantic / dataclass の `__post_init__` 値域チェック | DB / network / token に触れない |

## 5. Codex 禁止作業 (= 絶対委譲しない)

| 禁止項目 | 理由 |
|---|---|
| **LINE 本番送信**                  | R-19-08 / HQ 承認制。token 漏洩 + 誤通知リスク |
| **production / develop DB write**  | 正本破壊リスク。F286-PNL-R1 三段ガードと同じ原則 |
| **staging DB write (= 明示承認なし)** | 暗黙 write 禁止。本線が `--write` を明示渡す場合のみ可 |
| **token / secret / channel_token 参照** | `~/.fire_secrets/` / 環境変数 LINE_CHANNEL_TOKEN 等は本線のみ |
| **楽天証券操作**                    | R-19-08 / Computer Use 不採用方針 |
| **自動発注 / 注文価格 / 数量 / 執行指示の生成 helper** | F062-R5 で構造的禁止。Codex 実装にも禁止項目として明示 |
| **Computer Use**                   | v3.3 で撤回済方針 |
| **Playwright / Cron による強制クローズ** | v3.3 で不採用 |
| **GitHub Actions workflow 変更**   | R-01-08 / 月次確認チェックリスト対象。Codex にも禁止 |
| **`--no-verify` での hook bypass** | `scripts/hooks/pre-commit` 回避禁止 |
| **`scripts/seed_pattern_layer1.py` への変更** | 既存 modified 状態を維持、未触のまま PR 範囲外 |
| **`simulation/research_lane/historical_indicators.py` の既存 modified への干渉** | 同上、未触 |
| **TODO Excel 更新**                | 運用ルール上、Excel は本線が手動更新 |

これらは **プロンプト雛形に明示** し、Codex 投入時に毎回貼ること。

## 6. 並列タスク分解ルール

### 6.1 大原則

> **1 Codex task = 1 目的 / 1 成果物 / 触るファイル限定**

- **同じファイルを複数 Codex に触らせない** (= file ownership 表で衝突防止)
- **DB write / LINE send / secret が必要な作業は Codex 不可** (= 本線のみ)
- **仕様 / 実装 / test / docs / audit に分解する** (= 1 大タスクを 4-6 sub-task に)
- **file ownership 表を必ず作る** (= タスク投入前に本線が起票)

### 6.2 標準的な分解パターン (= 5 sub-task 型)

例: F286-PNL-R2 「Advisory Snapshot Auto-Ingest」を分解する場合:

| sub | lane | task | allowed_files |
|---|---|---|---|
| #1 | L1 Design       | snapshot schema + 三段ガード提案    | (vault draft only) |
| #2 | L3 Impl         | snapshot 保存 module (= isolated)   | `pnl/snapshot.py` (新規) |
| #3 | L2 Test         | snapshot module 単体 tests          | `tests/pnl/test_snapshot.py` (新規) |
| #4 | L4 Audit        | F062 payload → snapshot 経路 audit   | (report only) |
| #5 | L5 Docs         | vault doc draft                    | `02_todo/F286_PNL_R2_*.md` |

統合 (= runner 連携、merge) は **本線 Integrator** が行う (= 6 番目の sub-task)。

### 6.3 並列度の決め方

- **L3 Implementation は 1 同時** (= module 実装が同時に走ると merge 衝突)
- **L1 / L2 / L4 / L5 は 2-3 並列まで** (= read-only / file 別)
- **同じ design doc に 2 Codex を当てない** (= 設計衝突防止)

## 7. file ownership 表テンプレート

新規 task を Codex に投入する **前** に必ず本線が記入する。

```yaml
task_id: F286-XYZ-R1-sub1
lane: L3 Implementation
parent_task: F286-XYZ-R1
allowed_files:
  - pnl/example_new_module.py        # 新規作成のみ
  - tests/pnl/test_example.py        # (Test レーンの場合)
forbidden_files:
  - "*"                              # 上記 allowed 以外 全て
  - data/fire.db                     # production
  - data/fire.develop.db             # develop
  - .github/workflows/*              # workflow 絶対禁止
  - scripts/seed_pattern_layer1.py   # 既存 modified、未触必須
  - simulation/research_lane/historical_indicators.py
expected_outputs:
  - module ファイル 1 本
  - 完了報告 1 件 (= 後述テンプレート)
test_command: ".venv/bin/pytest tests/pnl/test_example.py -v"
report_required:
  - changed_files
  - tests result
  - safety checklist
  - merge recommendation
merge_owner: 本線 (Claude Code) Integrator
db_write_allowed: false                # staging も含めて false が default
line_send_allowed: false
token_read_allowed: false
```

### 7.1 file ownership 表の保管場所

- 大タスク 1 件につき `02_todo/<TASK_ID>_codex_lanes.md` を本線が起票
- 各 sub-task の file ownership を yaml ブロックで列挙
- Codex 投入時はこの yaml を プロンプトに丸ごとコピー

## 8. Codex 投入用プロンプト雛形 (5 種)

すべての雛形に **共通の禁止項目フッタ** を付ける (= 安全要件は毎回明示)。

### 8.1 共通フッタ (= 全レーン共通、毎回貼る)

```
=== 必ず守ること (= 違反は本線 refuse 対象) ===
- LINE 本番送信なし (LineBotClient / linebot SDK 不 import)
- production / develop DB への write なし
- staging DB への write も、本指示で明示許可されていない限り行わない
- token / channel_token / secret / .env / ~/.fire_secrets を読まない
- 楽天証券操作 / 自動発注 / Computer Use / Playwright / Selenium 使わない
- 注文価格 / 数量 / 執行指示の生成 helper を **含めない**
- .github/workflows/ を一切変更しない
- --no-verify を含む git skip オプションを使わない
- scripts/seed_pattern_layer1.py に触らない
- simulation/research_lane/historical_indicators.py に触らない
- TODO Excel に触らない
- 上記 allowed_files の外に書き込まない / 触らない

完了したら、後述の「Codex 完了報告テンプレート」に従って報告すること。
```

### 8.2 Design 雛形 (L1)

```
あなたは FIRE Codex-Design レーン担当です。
タスク ID: <TASK_ID>
親タスク: <PARENT_TASK>

目的:
<1 段落で目的を書く>

設計対象スコープ:
<対象 module / schema / 三段ガード のいずれか>

期待成果物:
1. Markdown 設計 draft (= 後で本線が 03_design/ にマージする想定)
2. スキーマ提案 (= CREATE TABLE / dataclass) があれば draft で出す
3. 三段ガード (= read_only + db_label + basename) の必要可否

allowed_files:
<vault 内 draft のみ、コードファイル変更なし>

制約:
- DB write なし / LINE 送信なし / token 不参照
- 設計のみ、実装コードは書かない
- (共通フッタ貼り付け)
```

### 8.3 Test 雛形 (L2)

```
あなたは FIRE Codex-Test レーン担当です。
タスク ID: <TASK_ID>
親タスク: <PARENT_TASK>

テスト対象モジュール:
<対象 .py パス>

期待成果物:
1. `tests/<module>/test_<name>.py` 単一ファイル新規作成
2. parametrize / ast-based 検査 / 三段ガード違反 refuse の検証 含む
3. `.venv/bin/pytest <ファイルパス> -v` で全件 PASS

allowed_files:
- tests/<path>/test_<name>.py    # 新規作成

forbidden_files:
- 全ての src コードファイル (= test 対象 module を含む実装に触らない)
- (共通フッタの禁止項目すべて)

報告:
- 件数 PASS / 件数 FAIL
- ast-based safety 検査の有無
- (共通フッタ貼り付け)
```

### 8.4 Implementation 雛形 (L3)

```
あなたは FIRE Codex-Implementation レーン担当です。
タスク ID: <TASK_ID>
親タスク: <PARENT_TASK>

実装対象 module:
<module 名 / 責務 1 文>

期待成果物:
1. <module>.py 1 ファイル (= isolated)
2. (必要なら) 三段ガード組込 (read_only + db_label + basename)
3. default dry-run、副作用は --write / --opt-in にカプセル化

allowed_files:
- <module 名のみ>

forbidden_files:
- 他の既存 module
- runner / argparse 統合は本線がやるので、ここでは module のみ
- (共通フッタの禁止項目すべて)

制約:
- LineBotClient / channel_token / 楽天 / Playwright / Selenium 不 import
- 注文価格 / 数量 / 執行指示の生成 helper を含めない
- (共通フッタ貼り付け)

報告:
- 完了報告テンプレートに従う
```

### 8.5 Audit 雛形 (L4)

```
あなたは FIRE Codex-Audit レーン担当です。
タスク ID: <TASK_ID>
親タスク: <PARENT_TASK>

監査対象:
<既存ファイルパス or 観点>

監査観点:
1. forbidden import (linebot / playwright / selenium 等)
2. SQL DDL/DML literal の漏洩
3. token / secret hardcode の有無
4. 三段ガードの抜け穴
5. (タスクに応じて追加)

期待成果物:
- /tmp/<task_id>_audit_report.md (Markdown レポート、新規作成)
- CRITICAL があれば 'CRITICAL:' 行で先頭に記載

allowed_files:
- /tmp/<task_id>_audit_report.md      # 新規 report のみ

forbidden_files:
- 全ての既存ソースコードの **書き換え禁止** (= read-only audit)
- (共通フッタの禁止項目すべて)

制約:
- 既存コードに触らない (= read-only)
- (共通フッタ貼り付け)
```

### 8.6 Docs 雛形 (L5)

```
あなたは FIRE Codex-Docs レーン担当です。
タスク ID: <TASK_ID>
親タスク: <PARENT_TASK>

ドキュメント対象:
<タスク完了マーカー / 設計 doc 雛形 / vault note>

期待成果物:
1. vault md ファイル 1 本 (frontmatter + 本文)
2. 後で本線が最終 review / 微修正してマージする

allowed_files:
- 02_todo/<TASK_ID>_*.md (= 完了マーカー雛形)
- 03_design/<TASK_ID>_*.md (= 設計雛形)

forbidden_files:
- log.md (= 本線が直接管理)
- CLAUDE.md (= 本線が完了テーブル更新)
- 全ての fire リポ側のコードファイル
- (共通フッタの禁止項目すべて)

制約:
- (共通フッタ貼り付け)
```

## 9. Codex 完了報告テンプレート

Codex が完了したら **必ず** このフォーマットで本線に返す:

```
=== Codex 完了報告 ===
task_id:            <TASK_ID>
lane:               <L1-L5>
changed_files:
  - <file 1>
  - <file 2>
commits:
  - <commit hash + メッセージ> (本線 commit 前なら "本線で commit 予定")
tests:
  added:            <N 件>
  pytest result:    <PASS N / FAIL M>
  test_command:     <実行コマンド>
safety_checks:
  forbidden_import: <none / 検出あり>
  forbidden_sql:    <none / 検出あり>
  forbidden_phrase: <none / 検出あり>
  three_stage_guard: <該当なし / 適用済>
DB_writes:
  production:       0
  develop:          0
  staging:          <none / N 件 (= 本指示で明示許可された場合のみ)>
LINE_sends:         0
secrets_touched:    none
forbidden_files_touched: none
known_risks:
  - <検出した懸念点 / 既知の制約>
merge_recommendation:
  - <"本線 review 後 merge 可" / "差し戻し推奨 + 理由">
```

未記入欄があれば **本線は差し戻し** (= 完了扱いにしない)。

## 10. 次に並列化する候補比較

直近の次タスク候補 4 件を、並列化適性で評価する:

| 候補 | 概要 | 並列化適性 | 推奨 sub-task 数 | 重要 risk |
|---|---|---|---|---|
| **F286-PNL-R2** Advisory Snapshot Auto-Ingest | F062 payload を snapshot 化、advisory_decisions に自動 ingest | **★★★ 高** | 5 (Design / Impl / Test / Audit / Docs) | snapshot schema を間違えると R1 破壊。Audit lane で守る |
| **F286-INTRA-R2** Intraday Advisory Trigger Engine | 「待ち」候補の場中監視 (VWAP/出来高) | **★ 低** | 1-2 (Design / PoC) | 場中監視は LINE 2 通目 trigger に直結 → 本線判断必須 |
| **F286-ORDER-R1** Manual Order Draft Generator | 注文 draft (価格/数量/OCO) を LINE に追加 | **× 不可** | 0 (Codex 不可) | 構造的禁止 (注文価格・数量・執行指示の生成 helper を含めない) と直接衝突 |
| **F286-DATA-R3** Daily Refresh Cron 化 | staging データの日次更新 cron 化 | **★★ 中** | 3 (cron 設計 / runner Impl / Test) | cron 起動は本線確認、staging write が必要 → 明示許可運用 |

### 10.1 並列化適性の判断基準

- **★★★ 高**: 5+ sub-task に分解可能、5 レーン全てを使える、副作用ゼロ
- **★★ 中**: 3-4 sub-task、staging write が必要だが構造ガード可
- **★ 低**: 1-2 sub-task、本線判断比率が高い
- **× 不可**: タスク自体が Codex 禁止項目と衝突

## 11. 初回の推奨並列投入プラン (= F286-PNL-R2 を題材)

次タスクの最有力候補 **F286-PNL-R2 Advisory Snapshot Auto-Ingest**
を 5 本の Codex レーンで並列実行する詳細プラン。

### 11.1 全体構造

```
本線 PM (タスク分解 + 投入順序)
    ↓
[並列] Codex-Design (sub-1)
    ↓
本線 Architect (設計 approve)
    ↓
[並列] Codex-Impl (sub-2) + Codex-Test (sub-3) + Codex-Docs (sub-5)
    ↓
[並列] Codex-Audit (sub-4) ← Impl 完了後、read-only audit
    ↓
本線 Integrator (merge order: Impl → Test → Docs)
    ↓
本線 Final Reviewer (pre-commit + 回帰 + CRITICAL 最終判断)
    ↓
本線 HQ 報告 (= 1 block)
```

### 11.2 sub-1: Codex-Design

```
あなたは FIRE Codex-Design レーン担当です。
タスク ID: F286-PNL-R2-sub1
親タスク: F286-PNL-R2 Advisory Snapshot Auto-Ingest

目的:
F062 が LINE 送信時に生成する payload を「Advisory Snapshot」として
保存し、F286-PNL-R1 の advisory_decisions に **手入力なしで** ingest
できる経路を設計する。

設計対象スコープ:
1. Snapshot schema (= テーブル設計、PK、CHECK、INDEX)
2. F062 payload → snapshot 変換ルール (= advisory_id 採番、銘柄 1 行に分割)
3. 三段ガード (= staging-only、basename ガード)
4. F286-PNL-R1 との互換性 (= 既存 advisory_decisions PK と衝突しない)

期待成果物:
- Markdown 設計 draft (本線が 03_design/F286_PNL_R2_*.md に整える)
- スキーマ提案 (CREATE TABLE) draft
- F062 payload からの mapping 表

allowed_files:
- 設計 draft のみ (= vault draft、コードファイル変更なし)

(共通フッタを貼り付け)
```

### 11.3 sub-2: Codex-Implementation

```
あなたは FIRE Codex-Implementation レーン担当です。
タスク ID: F286-PNL-R2-sub2
親タスク: F286-PNL-R2 Advisory Snapshot Auto-Ingest

実装対象 module:
pnl/snapshot.py — F062 payload から AdvisorySnapshot を構築する純関数 +
staging への保存 helper (= 三段ガード再利用)。

期待成果物:
1. pnl/snapshot.py (= 新規 1 ファイル、isolated)
2. F286-PNL-R1 の DecisionStore と同じ三段ガード設計
3. default dry-run、--write は本線が runner 側で渡す前提

allowed_files:
- pnl/snapshot.py    # 新規作成のみ

forbidden_files:
- pnl/storage.py      # 既存、触らない
- pnl/schema.py       # 既存、触らない
- scripts/jobs/*      # runner は本線が組む
- (共通フッタの禁止項目すべて)

制約:
- LineBotClient / channel_token / 楽天 / Playwright 不 import
- 注文価格 / 数量 / 執行指示の生成 helper を含めない
- DB write は staging のみ、basename ガード必須
- (共通フッタ貼り付け)
```

### 11.4 sub-3: Codex-Test

```
あなたは FIRE Codex-Test レーン担当です。
タスク ID: F286-PNL-R2-sub3
親タスク: F286-PNL-R2 Advisory Snapshot Auto-Ingest

テスト対象モジュール:
pnl/snapshot.py (= sub-2 で実装される module)

期待成果物:
1. tests/pnl/test_snapshot.py 1 ファイル
2. parametrize で fujiwara_decision/actual_trade 値域検査
3. 三段ガード違反 refuse 検証
4. F062 payload mock から snapshot 構築の round-trip test
5. ast-based forbidden import 検査

allowed_files:
- tests/pnl/test_snapshot.py    # 新規作成のみ

forbidden_files:
- pnl/snapshot.py 自体           # 実装は sub-2 担当、test は read-only 検査
- (共通フッタの禁止項目すべて)

(共通フッタを貼り付け)
```

### 11.5 sub-4: Codex-Audit

```
あなたは FIRE Codex-Audit レーン担当です。
タスク ID: F286-PNL-R2-sub4
親タスク: F286-PNL-R2 Advisory Snapshot Auto-Ingest

監査対象:
1. pnl/snapshot.py (sub-2 で実装される module、Impl 完了後にレビュー)
2. F286-PNL-R1 既存コードとの干渉ポイント (advisory_decisions スキーマ衝突等)

監査観点:
1. forbidden import (linebot / playwright / selenium 等)
2. SQL DDL/DML literal の漏洩
3. 三段ガードの抜け穴 (= staging-only の偽装ベクタ)
4. F286-PNL-R1 advisory_decisions との PK 衝突 / created_at 上書き
5. 注文価格・数量・執行指示の生成 helper の有無

期待成果物:
- /tmp/f286_pnl_r2_audit_report.md (Markdown レポート)
- CRITICAL 行があれば 'CRITICAL:' 先頭

allowed_files:
- /tmp/f286_pnl_r2_audit_report.md   # 新規 report

forbidden_files:
- 全てのソースコード書き換え禁止 (= read-only audit)
- (共通フッタの禁止項目すべて)

(共通フッタを貼り付け)
```

### 11.6 sub-5: Codex-Docs

```
あなたは FIRE Codex-Docs レーン担当です。
タスク ID: F286-PNL-R2-sub5
親タスク: F286-PNL-R2 Advisory Snapshot Auto-Ingest

ドキュメント対象:
F286-PNL-R2 タスク完了マーカー雛形

期待成果物:
1. 02_todo/F286_PNL_R2_advisory_snapshot_auto_ingest.md (= 完了マーカー雛形)
2. frontmatter + 本文 (= 設計サマリ / smoke 予定 / 安全要件 / 次タスク)

allowed_files:
- 02_todo/F286_PNL_R2_advisory_snapshot_auto_ingest.md   # 新規 draft

forbidden_files:
- log.md (= 本線が直接管理)
- CLAUDE.md (= 本線が完了テーブル更新)
- 03_design/ 配下 (= sub-1 Design 担当)
- 全ての fire リポ側のコードファイル
- (共通フッタの禁止項目すべて)

(共通フッタを貼り付け)
```

### 11.7 投入順序

| ステップ | 並列性 | 担当 | 想定所要 |
|---|---|---|---|
| 1. file ownership 表起票           | (本線のみ) | 本線 PM | 5 分 |
| 2. sub-1 Design 投入              | 1 lane | Codex-Design | 10-15 分 |
| 3. 本線 Architect で design approve | (本線のみ) | 本線 Architect | 5-10 分 |
| 4. sub-2 Impl + sub-3 Test + sub-5 Docs 並列投入 | 3 lane | Codex 並列 | 10-15 分 |
| 5. sub-4 Audit 投入 (= Impl 完了後) | 1 lane | Codex-Audit | 5-10 分 |
| 6. 本線 Integrator が merge 統合    | (本線のみ) | 本線 Integrator | 5-10 分 |
| 7. 本線 Final Reviewer (= pre-commit + 回帰) | (本線のみ) | 本線 | 5 分 |
| 8. 本線 HQ 報告 (= 1 block)        | (本線のみ) | 本線 PM | 5 分 |

合計: 約 50-75 分 (= 単独本線実装で 90-120 分のタスクを並列で 30-40%
短縮できる試算)。

## 12. ガバナンス / リスク

### 12.1 失敗時のロールバック

- Codex 成果物が CRITICAL 含有なら **本線が即修正、または差し戻し**
- 複数 Codex の merge で衝突した場合は **Integrator が単独で resolve**
- staging DB に意図せず書き込みが発生したら **本線が即 rollback**
  (= F282 snapshot or pre_restore .bak から復元)

### 12.2 監査ログ

- 各 Codex 投入時に **task_id + lane + 投入時刻** を log.md に記録
- 完了報告も log.md に記録 (= 並列化の効果測定にも使う)

### 12.3 R-01-08 整合

本設計は CI/CD 不採用方針 (R-01-08) を **強化** する方向:

- Codex は Mac mini 上のローカル CLI で動作 (= GitHub Actions 不使用)
- `.github/workflows/` 変更は Codex にも禁止 (= 月次 5 項目チェックリスト維持)
- pre-commit hook は本線 commit でのみ走る (= Codex 直接 commit はしない、
  本線が staging branch でテスト → develop へ最終 commit する想定)

## 13. 改訂履歴

| 日付 | 版 | 内容 |
|---|---|---|
| 2026-05-11 | v1.0 | 初版 (= F286-PNL-R1 完了直後の HQ 承認で運用開始) |
| 2026-05-11 | v1.1 | HQ 補足受領: Codex を実装部隊として積極利用する方向に明確化。実装レーン (L3) の標準運用 / 1 branch / 統合手順 / safety scanner を初回候補に追加 / 3-5 本並列タスクの分解と投入プロンプトを §15-18 で詳細化 |

---

## 14. v1.1 補足: Codex を実装部隊化する方針

### 14.1 動機

v1.0 では Codex を 5 レーン (Design / Test / Implementation / Audit /
Docs) に分けたが、運用初期は **review 中心** の印象が強かった。
HQ 補足受領 (2026-05-11) で「Codex を実装部隊として積極利用、本線
開発速度を底上げする」方針に明確化した。

これは v1.0 の **§3 (5 レーン)** と **§4 (委譲可カテゴリ 8 件)** を
書き換えるのではなく、**実装レーン (L3) を主軸**に位置付け直し、
他レーンを支援役にする運用シフト。禁止項目 13 件 (§5) は完全維持。

### 14.2 v1.0 → v1.1 の運用シフト

| 観点 | v1.0 | v1.1 |
|---|---|---|
| Codex の主用途      | レビュー + 限定実装         | **実装本体 + tests + scanner + docs 並列** |
| 並列度の上限         | L3 は 1 同時                | **L3 を 3-5 本並列** (= 1 branch / 1 ownership で衝突回避) |
| 本線の業務比率       | レビュー中心                | **PM + Integrator + Final Reviewer 比率 ↑** |
| 初回候補            | F286-PNL-R2 単独 5 sub      | **F286-PNL-R2 / F286-DATA-R3 / F286-INTRA-R2 PoC / safety scanner / F286-ORDER-R1 dummy** を並列 |
| Branch 戦略         | (未定義)                    | **1 Codex task = 1 feature branch** (= 衝突防止 + rollback 容易) |

## 15. Codex 実装レーンの標準運用 (= v1.1 中核)

### 15.1 1 Codex task の標準形

```yaml
# 全 Codex 実装 task が必ず守る形
task_id:        <ID>          # 例: F286-PNL-R2-impl
parent_task:    <親タスク>
lane:           L3 Implementation
branch:         codex/<task_id>     # ★ 1 task = 1 branch
allowed_files:                       # 触ってよいファイル列挙
  - <file 1>
  - <file 2>
forbidden_files:                     # 触らないファイル (= 共通禁止 + 個別)
  - "*"                              # allowed 以外
  - data/fire.db
  - data/fire.develop.db
  - .github/workflows/*
  - scripts/seed_pattern_layer1.py
  - simulation/research_lane/historical_indicators.py
expected_outputs:                    # ★ 必須
  - <成果物 1>
  - <成果物 2>
test_command:                        # ★ 必須
  ".venv/bin/pytest <path> -v"
report_required: true                # ★ §9 テンプレ厳守
merge_owner: 本線 (Claude Code) Integrator
db_write_allowed: false              # default、明示許可なければ false
line_send_allowed: false
token_read_allowed: false
```

### 15.2 1 task = 1 branch (= v1.1 で新規追加)

| 項目 | 詳細 |
|---|---|
| branch 命名         | `codex/<task_id>` (例: `codex/f286-pnl-r2-impl`) |
| base 元             | `develop` (= 本線開発ブランチ) |
| 並列時の衝突回避     | 同じ allowed_files を持つ branch を作らない (= ownership 表で防ぐ) |
| 完了後の merge       | Codex は **直接 develop に merge しない**。本線 Integrator が cherry-pick or PR merge |
| pre-commit hook     | Codex の commit でも fire リポの pre-commit が走る (= Codex 自身が `codex exec` review を通す) |
| 失敗時の rollback    | branch ごと破棄で済む (= develop 汚染なし) |

### 15.3 必須必要項目チェックリスト (= 本線が起票時に埋める)

- [ ] task_id / parent_task 命名済
- [ ] lane 明示 (L1-L5)
- [ ] branch 名決定 (= `codex/<id>`)
- [ ] allowed_files 列挙 (= 最小限)
- [ ] forbidden_files に共通禁止リスト + 個別 file が入っている
- [ ] expected_outputs 列挙
- [ ] test_command 指定 (= 動作確認の単一行)
- [ ] 完了報告テンプレ (§9) を Codex プロンプトに添付済
- [ ] db_write_allowed / line_send_allowed / token_read_allowed が
      `false` (= 明示許可された task のみ true)
- [ ] 共通フッタ (§8.1) を Codex プロンプトに貼付済

埋まっていなければ **Codex 投入禁止**。

### 15.4 本線 merge は本線 / HQ のみ

Codex が完成しても、最終的に develop に取り込むのは **本線 Integrator
+ HQ 判断**:

1. 本線 Integrator が Codex branch を取得
2. `.venv/bin/pytest` 全件 PASS を確認
3. 既存 develop との衝突確認 (= file ownership 表で予防済のはず)
4. HQ (Fujiwara) に「merge 可否」の 1 block 報告
5. HQ approve → 本線が develop に merge
6. log.md に merge 履歴を記録

Codex 自身が develop に push することは **構造的に禁止**。

## 16. Codex に実装させる初回候補 (= v1.1 で再評価)

### 16.1 候補一覧 (= 5 件)

| 候補 | 概要 | Codex 実装適性 | sub-task 数 | branch |
|---|---|---|---|---|
| **F286-PNL-R2** Advisory Snapshot Auto-Ingest | F062 payload → advisory_decisions 自動 ingest | **★★★ 高** | 5 | `codex/f286-pnl-r2-*` |
| **F286-INTRA-R2** Intraday Advisory Trigger Engine | 「待ち」候補の場中 VWAP / 出来高監視 | **★★ 中** (= PoC 部分のみ) | 2-3 | `codex/f286-intra-r2-*` |
| **F286-ORDER-R1** Manual Order Draft Generator | 注文 draft (価格/数量/OCO) を LINE 通知 (= **dummy / placeholder のみ**) | **★ 条件付き可** | 2 | `codex/f286-order-r1-*` |
| **F286-DATA-R3** Daily Refresh Cron 化 | staging データの日次更新 cron | **★★ 中** | 3 | `codex/f286-data-r3-*` |
| **FIRE-AUDIT-R1** Safety Scanner / Forbidden Op Audit | repo 横断で forbidden import / SQL DDL / token hardcode を AST 走査 | **★★★ 高** (= read-only) | 2-3 | `codex/fire-audit-r1-*` |

### 16.2 F286-ORDER-R1 を「条件付き可」に再分類した根拠

v1.0 では「× 不可」だったが、v1.1 で **dummy / placeholder のみ実装**
ならば Codex 委譲可と再評価:

| 段階 | Codex 可否 | 内容 |
|---|---|---|
| Step 1: draft 表示の **骨組みのみ** (= placeholder 関数 + 文字列 template) | **★ 可**  | 値は dummy / 0、計算ロジック未実装、本線が後で実 logic を埋める前提 |
| Step 2: 実 draft 価格 / 数量 / OCO の **生成 logic 本体** | × 不可 | 「注文価格・数量・執行指示の生成 helper」と直接衝突、本線単独実装 |
| Step 3: LINE 統合 + 本番送信                          | × 不可 | LINE 本番送信は本線のみ |

つまり F286-ORDER-R1 は Step 1 だけ Codex に出し、Step 2-3 は本線
担当。これで「Codex を実装部隊化」と「自動発注禁止」の両立を実現。

### 16.3 FIRE-AUDIT-R1 (= safety scanner) を初回候補に追加した根拠

- repo 横断の **read-only AST 走査** であり、副作用ゼロ
- 既存コードの forbidden import / SQL DDL / token hardcode を
  **早期発見** して F286-PNL-R1 のような CRITICAL を予防
- Codex の典型強み (= 大量パターン認識) を活かせる
- 並列度: 観点別に 2-3 lane で同時走査可能

## 17. v1.1 初回投入プラン (= 3-5 本並列、20-40 分短縮想定)

### 17.1 全体構造

```
本線 PM
  └─ file ownership 表 5 件起票 (= 1 task / 1 branch)
       │
       ├─ Codex-Design (sub-1)    F286-PNL-R2 設計
       ├─ Codex-Audit (sub-A)     FIRE-AUDIT-R1 forbidden import scan
       └─ Codex-Audit (sub-B)     FIRE-AUDIT-R1 SQL/token hardcode scan
                            (3 lane 並列、約 15 分)
              ↓
       本線 Architect (sub-1 approve)
              ↓
       ├─ Codex-Impl (sub-2)      F286-PNL-R2 pnl/snapshot.py
       ├─ Codex-Test (sub-3)      F286-PNL-R2 tests/pnl/test_snapshot.py
       ├─ Codex-Impl (sub-D1)     F286-DATA-R3 cron runner (= staging write 明示許可)
       └─ Codex-Docs (sub-5)      F286-PNL-R2 vault draft
                            (4 lane 並列、約 15-20 分)
              ↓
       Codex-Audit (sub-4)        F286-PNL-R2 audit report
              ↓
       本線 Integrator が 5 branch を統合
              ↓
       本線 Final Reviewer (pre-commit + 回帰)
              ↓
       本線 HQ 報告 (= 1 block)
```

合計: 約 50-70 分 (= 本線単独 120-180 分のタスクを **30-50% 短縮**)。

### 17.2 file ownership 表 (= 初回投入 5 件分の正本)

```yaml
# ========================================
# 投入順: A → B → 1 (並列)、続いて 2 → 3 → D1 → 5 (並列)、最後に 4
# ========================================

- task_id: FIRE-AUDIT-R1-sub-A
  parent_task: FIRE-AUDIT-R1
  lane: L4 Audit
  branch: codex/fire-audit-r1-imports
  allowed_files:
    - /tmp/fire_audit_r1_forbidden_imports_report.md
  forbidden_files: ["*"]   # report 以外 全て read-only
  expected_outputs:
    - Markdown レポート (linebot / playwright / selenium / requests.post
      の import を repo 横断で走査、見つかった場所を列挙)
  test_command: "(N/A、read-only audit)"
  report_required: true
  merge_owner: 本線 Integrator
  db_write_allowed: false
  line_send_allowed: false
  token_read_allowed: false

- task_id: FIRE-AUDIT-R1-sub-B
  parent_task: FIRE-AUDIT-R1
  lane: L4 Audit
  branch: codex/fire-audit-r1-sql-token
  allowed_files:
    - /tmp/fire_audit_r1_sql_token_hardcode_report.md
  forbidden_files: ["*"]
  expected_outputs:
    - Markdown レポート (SQL DDL/DML literal の漏洩 + token / channel_token
      / FIRE_LINE_RECIPIENT_ID / secret 系の hardcode を AST + grep で走査)
  test_command: "(N/A、read-only audit)"
  report_required: true
  merge_owner: 本線 Integrator
  db_write_allowed: false
  line_send_allowed: false
  token_read_allowed: false

- task_id: F286-PNL-R2-sub-1
  parent_task: F286-PNL-R2
  lane: L1 Design
  branch: codex/f286-pnl-r2-design
  allowed_files:
    - /tmp/f286_pnl_r2_design_draft.md
  forbidden_files: ["*"]   # 設計のみ、コード触らない
  expected_outputs:
    - Snapshot schema CREATE TABLE 案
    - F062 payload → snapshot mapping 表
    - 三段ガード適用方針
  test_command: "(N/A、設計のみ)"
  report_required: true
  merge_owner: 本線 Architect
  db_write_allowed: false
  line_send_allowed: false
  token_read_allowed: false

- task_id: F286-PNL-R2-sub-2
  parent_task: F286-PNL-R2
  lane: L3 Implementation
  branch: codex/f286-pnl-r2-snapshot-module
  allowed_files:
    - pnl/snapshot.py    # 新規 1 ファイル
  forbidden_files:
    - pnl/storage.py
    - pnl/schema.py
    - pnl/models.py
    - "*"                # 上記 allowed 以外
  expected_outputs:
    - pnl/snapshot.py (= isolated module、F286-PNL-R1 三段ガード再利用)
    - default dry-run、--write は本線側 runner で渡す前提
  test_command: ".venv/bin/python -c 'import pnl.snapshot; print(\"ok\")'"
  report_required: true
  merge_owner: 本線 Integrator
  db_write_allowed: false
  line_send_allowed: false
  token_read_allowed: false

- task_id: F286-PNL-R2-sub-3
  parent_task: F286-PNL-R2
  lane: L2 Test
  branch: codex/f286-pnl-r2-snapshot-tests
  allowed_files:
    - tests/pnl/test_snapshot.py    # 新規 1 ファイル
  forbidden_files:
    - pnl/snapshot.py               # 実装は sub-2 担当
    - "*"
  expected_outputs:
    - parametrize 値域 / 三段ガード違反 refuse / round-trip 含む test
    - ast-based forbidden import 検査
  test_command: ".venv/bin/pytest tests/pnl/test_snapshot.py -v"
  report_required: true
  merge_owner: 本線 Integrator
  db_write_allowed: false
  line_send_allowed: false
  token_read_allowed: false

- task_id: F286-DATA-R3-sub-D1
  parent_task: F286-DATA-R3
  lane: L3 Implementation
  branch: codex/f286-data-r3-cron-runner
  allowed_files:
    - scripts/jobs/run_f286_data_r3_daily_refresh.py    # 新規 runner
  forbidden_files:
    - scripts/jobs/run_f111_*                            # 既存 wiring
    - scripts/jobs/run_data_freshness_gate.py            # 既存 gate
    - "*"
  expected_outputs:
    - daily refresh 用 runner (= default dry-run、--write でのみ staging
      に書き込み)
    - F286-PNL-R1 と同じ三段ガード (db_label='staging' + basename)
  test_command: ".venv/bin/pytest tests/scripts/jobs/test_run_f286_data_r3_daily_refresh.py -v"
                # (test も同じ branch で書く、または別 sub-task で分割)
  report_required: true
  merge_owner: 本線 Integrator
  db_write_allowed: true   # ★ 例外: staging のみ、basename ガード越し
  line_send_allowed: false
  token_read_allowed: false

- task_id: F286-PNL-R2-sub-5
  parent_task: F286-PNL-R2
  lane: L5 Docs
  branch: codex/f286-pnl-r2-docs
  allowed_files:
    - 02_todo/F286_PNL_R2_advisory_snapshot_auto_ingest.md  # vault のみ
  forbidden_files:
    - log.md
    - CLAUDE.md
    - 03_design/*       # 設計は sub-1 担当
    - "*"
  expected_outputs:
    - 02_todo タスク完了マーカー雛形 (本線が最終 review してマージ)
  test_command: "(N/A、docs)"
  report_required: true
  merge_owner: 本線 Integrator
  db_write_allowed: false
  line_send_allowed: false
  token_read_allowed: false

- task_id: F286-PNL-R2-sub-4
  parent_task: F286-PNL-R2
  lane: L4 Audit
  branch: codex/f286-pnl-r2-audit
  allowed_files:
    - /tmp/f286_pnl_r2_audit_report.md
  forbidden_files: ["*"]   # read-only audit
  expected_outputs:
    - Markdown レポート (forbidden import / SQL DDL/DML literal /
      三段ガード抜け穴 / F286-PNL-R1 advisory_decisions PK 衝突
      / 注文価格・数量・執行指示生成 helper 有無)
  test_command: "(N/A、read-only audit)"
  report_required: true
  merge_owner: 本線 Integrator
  db_write_allowed: false
  line_send_allowed: false
  token_read_allowed: false
```

### 17.3 本線統合手順 (= Integrator + Final Reviewer の手順)

```
[本線 Integrator]
1. 全 Codex branch (= codex/*) を確認
2. allowed_files 違反がないか git diff で確認 (= 範囲外ファイル変更検査)
3. 安全項目チェック:
   - LineBotClient / linebot / playwright 不 import 確認
   - SQL DDL/DML literal (INSERT/UPDATE/DELETE/DROP/CREATE) 漏洩確認
   - token / channel_token / FIRE_LINE_RECIPIENT_ID hardcode 確認
   - 三段ガード (= staging-only + basename) 適用確認
4. 各 branch を develop に rebase + cherry-pick (= 1 commit / 1 branch)
   または PR merge で 1 task 1 commit
5. 衝突発生時は **そのまま手 resolve**、--no-verify は使わない

[本線 Final Reviewer]
6. .venv/bin/pytest 全件 PASS 確認 (= 回帰 0 件)
7. Codex pre-commit が走った formal commit ログを確認
8. 本線が Codex CRITICAL 指摘を最終判断 (= 即修正 or 差し戻し)

[本線 PM]
9. HQ 報告 1 block を作成、Fujiwara へ報告
   - 完了 sub-task の task_id 一覧
   - 各 sub の commit hash
   - 統合後の pytest 回帰結果
   - DB write 結果 (staging 件数、production/develop unchanged)
   - LINE send 0
   - token-recipient leak 0
   - HQ 判断が必要な論点 (CRITICAL 残や設計判断)
```

### 17.4 投入時刻と運用ログ

各 Codex 投入時に log.md に 1 行記録:

```
## [YYYY-MM-DD] codex | <task_id> lane=<L#> branch=<codex/...> 投入
```

完了時にも 1 行:

```
## [YYYY-MM-DD] codex | <task_id> 完了 commits=<hash> tests=<PASS/FAIL>
```

これで並列化の効果測定 (= 想定時間 vs 実時間) + どの lane が詰まり
やすいか の運用データを蓄積。

## 18. Codex 投入用コピペプロンプト (= v1.1 で 3-5 本一括版)

§8 のプロンプト雛形を、§17.2 file ownership 表と組合せて初回投入する
コピペ用プロンプトを準備。本線はこれを **そのままコピペ** で Codex に
投入できる。

### 18.1 投入順序 (= 詳細)

**Wave 1 (= 並列 3 lane、約 15 分)**:
- FIRE-AUDIT-R1-sub-A (= forbidden import scan)
- FIRE-AUDIT-R1-sub-B (= SQL / token hardcode scan)
- F286-PNL-R2-sub-1 (= Design)

**Wave 2 (= 本線 Architect の design approve 後、並列 4 lane、約 15-20 分)**:
- F286-PNL-R2-sub-2 (= Impl)
- F286-PNL-R2-sub-3 (= Test)
- F286-DATA-R3-sub-D1 (= cron runner Impl + Test)
- F286-PNL-R2-sub-5 (= Docs draft)

**Wave 3 (= Impl 完了後、1 lane、約 5-10 分)**:
- F286-PNL-R2-sub-4 (= Audit)

**Wave 4 (= 本線統合 + HQ 報告、約 15-20 分)**:
- Integrator → Final Reviewer → PM 報告

### 18.2 各 sub のコピペプロンプト (= §8 雛形 + §17.2 ownership 表を結合)

各プロンプトは以下の構造で本線が組み立てる:

```
<§8 該当レーン雛形 (Design / Impl / Test / Audit / Docs)>

=== file ownership (= §17.2 から該当ブロックを貼り付け) ===
<task_id 等の yaml ブロック>

=== 完了報告フォーマット (= §9 を貼り付け) ===
<完了報告テンプレ>

=== 必ず守ること (= §8.1 共通フッタ) ===
<13 禁止項目>
```

3-5 個分のプロンプトは長くなるが、毎回テンプレが同じなので **本線が
1 度作って保存** すれば再利用できる (= 将来 `~/fire-vault/08_reference/
codex_prompts/` にテンプレ集を置く案を §19 で言及)。

## 19. Known Limitations / 将来拡張

### 19.1 既知の制約 (= v1.1 時点)

1. **Codex の自律完走可否は task サイズ依存** — 200 行超 の module 実装は
   分割推奨 (= sub-task をさらに細分化)。
2. **本線 Integrator が衝突を手 resolve** — file ownership 表が
   完璧に守られれば衝突ゼロ、運用初期は手動チェック必要。
3. **Codex の merge 直接 push は禁止** — 本線が必ず最終 commit。
   現状この強制は運用ルールベース (= 技術的強制は別タスク)。
4. **F286-ORDER-R1 の Step 2-3** — 注文 draft 本体は本線単独実装
   (= Codex Step 1 placeholder のみ)。
5. **dump 系の API key / channel token** — 完了報告でも値を出さない
   (= `*_SET: True / length=N` 形式のみ、§9 に追記候補)。

### 19.2 将来拡張案

1. **`08_reference/codex_prompts/` テンプレ集** — 5 レーン × 共通フッタ
   をディレクトリ化、初回 Codex 起票が `cat <template>` で済む。
2. **`scripts/codex/start_lane.sh`** — file ownership 表 yaml を
   読み込んで Codex プロンプトを自動組立する shell (= Phase 2)。
3. **本線 merge 自動化** — `gh pr` で Codex branch を PR 化、本線が
   approve → merge する web 経由運用 (= R-01-08 と整合する範囲で検討)。
4. **並列効果の計測** — log.md の codex entry から「想定時間 vs 実時間」
   を集計、週次レビューで並列度を最適化。

## 20. 関連リンク

- [[../CLAUDE]] vault
- [[../タスク運用ルール]]
- [[../log]]
- [[../02_todo/F286_PNL_R1_advisory_decision_pnl_tracking]] (= 三段ガード実例)
- [[../02_todo/F062_R5_receipt_confirmation]] (= 本番 LINE Phase 1 完結)
