---
id: FIRE-CODEX-R1
phase: ガバナンス / R-01-08 補強 / Codex 運用拡張
priority: 最優先
status: 完了 ★ (2026-05-11、v1.1 確定: Codex を実装部隊化)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R1 (= 三段ガードを Codex CRITICAL 2 回で確立した実例)
  - F062-R5 (= 本番 LINE Advisory Phase 1 完結)
chapter: ガバナンス / R-01-08 / Codex レビュー運用体制
---

# FIRE-CODEX-R1: Multi-Lane Parallel Development Orchestration

最終更新: 2026-05-11

## ★ 状態: 完了 (= 設計確定、運用開始)

Claude Code 本線を PM / Architect / Integrator / Final Reviewer に
固定し、Codex を 5 種類の限定レーン (Design / Test / Implementation /
Audit / Docs) として並列活用する運用設計を確定。本タスクはコード
変更なし、fire-vault docs / log のみ。

詳細は設計本体 [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 設計 doc v1.0]] を参照。

## 設計サマリ

### 本線 (Claude Code) の独占判断

| 判断 | 説明 |
|---|---|
| HQ 報告作成               | Fujiwara への完了報告は本線のみ |
| 本線 merge 判断           | feat/test/docs commit を develop に取り込む最終判断 |
| pre-commit / 回帰維持      | pytest 全件 PASS 最終確認 |
| LINE 本番送信判断          | --send --hq-approved-send は本線のみ |
| CRITICAL 受領判断          | 即修正 / 差し戻し / 改訂判断 |
| Workflow / GitHub 設定変更 | R-01-08 維持、本線も Codex も触らない |

### Codex 5 レーン

| Lane | 名前                   | 主タスク                      | 並列度       |
|---|---|---|---|
| L1   | Codex-Design           | 設計 / schema / ガード提案     | 1 同時 |
| L2   | Codex-Test             | tests 1 ファイル単位          | 2-3 並列可 |
| L3   | Codex-Implementation   | isolated module 1 本          | 1 同時 (merge 衝突防止) |
| L4   | Codex-Audit            | read-only 監査                | 並列可 |
| L5   | Codex-Docs             | vault draft 1 本              | 並列可 |

### Codex 禁止項目 (= 絶対委譲しない、13 項目)

1. LINE 本番送信
2. production / develop DB write
3. staging DB write (= 明示承認なし)
4. token / secret / channel_token 参照
5. 楽天証券操作
6. 自動発注 / 注文価格 / 数量 / 執行指示の生成
7. Computer Use
8. Playwright / Cron による強制クローズ
9. GitHub Actions workflow 変更
10. `--no-verify` での hook bypass
11. `scripts/seed_pattern_layer1.py` への変更
12. `simulation/research_lane/historical_indicators.py` の既存 modified への干渉
13. TODO Excel 更新

### 並列タスク分解大原則

> **1 Codex task = 1 目的 / 1 成果物 / 触るファイル限定**

- 同じファイルを複数 Codex に触らせない (= file ownership 表で衝突防止)
- DB write / LINE send / secret が必要な作業は Codex 不可
- 仕様 / 実装 / test / docs / audit に分解する

### file ownership 表テンプレ (= 設計 doc §7 参照)

```yaml
task_id: <ID>
lane: <L1-L5>
parent_task: <親タスク>
allowed_files: [...]
forbidden_files: [..., 共通禁止リスト含む]
expected_outputs: [...]
test_command: ...
report_required: [...]
merge_owner: 本線 (Claude Code) Integrator
db_write_allowed: false  # default
line_send_allowed: false
token_read_allowed: false
```

### プロンプト雛形 (= 5 種、設計 doc §8 参照)

- §8.1 共通フッタ (= 13 禁止項目を毎回貼り付け)
- §8.2 Design (L1)
- §8.3 Test (L2)
- §8.4 Implementation (L3)
- §8.5 Audit (L4)
- §8.6 Docs (L5)

### 完了報告テンプレート (= 設計 doc §9 参照)

```
task_id / lane / changed_files / commits / tests / safety_checks /
DB_writes / LINE_sends / secrets_touched / forbidden_files_touched /
known_risks / merge_recommendation
```

未記入欄があれば本線は差し戻し。

## 次に並列化する候補比較

| 候補 | 並列化適性 | 推奨 sub-task 数 |
|---|---|---|
| **F286-PNL-R2** Advisory Snapshot Auto-Ingest        | ★★★ 高 | 5 (Design/Impl/Test/Audit/Docs) |
| **F286-INTRA-R2** Intraday Advisory Trigger Engine   | ★ 低    | 1-2 (Design / PoC) |
| **F286-ORDER-R1** Manual Order Draft Generator       | × 不可  | 0 (構造的禁止と直接衝突) |
| **F286-DATA-R3** Daily Refresh Cron 化               | ★★ 中  | 3 (cron 設計 / runner / Test) |

## 初回の推奨並列投入プラン (= F286-PNL-R2 / 5 lane 並列)

設計 doc §11 にコピペ用プロンプト 5 本を完備:

1. sub-1 Codex-Design       — snapshot schema + 三段ガード提案
2. sub-2 Codex-Implementation — pnl/snapshot.py (= isolated module)
3. sub-3 Codex-Test         — tests/pnl/test_snapshot.py
4. sub-4 Codex-Audit        — /tmp/f286_pnl_r2_audit_report.md
5. sub-5 Codex-Docs         — 02_todo/F286_PNL_R2_*.md draft

統合 (sub-6) は本線 Integrator が単独実施。

予想時間: 約 50-75 分 (= 単独本線実装 90-120 分の 30-40% 短縮)。

## 安全要件遵守 (= 全 ✓)

| 項目 | 結果 |
|---|---|
| コード変更                              | なし ✓ (= 設計のみ) |
| DB write                                | 0 ✓ |
| LINE 送信                                | 0 ✓ |
| token / secret 参照                     | 0 ✓ |
| Workflow / GitHub Actions 変更          | なし ✓ |
| --no-verify                            | 不使用 ✓ |
| 個別 commit (= まとめ commit 禁止)       | docs 1 + log 1 の 2 commit 構成 ✓ |
| pre-commit 通過                          | vault に hook 無し / fire リポは変更なし ✓ |
| scripts/seed_pattern_layer1.py          | 未接触 ✓ |
| simulation/.../historical_indicators.py | 未接触 ✓ |
| TODO Excel                              | 未更新 ✓ |

## v1.1 補足 (= HQ 受領、Codex を実装部隊化)

v1.0 確定直後に HQ 補足受領 (= 同日 2026-05-11):

> Codex を review 専用ではなく、実装レーンとして積極利用する前提で
> 設計してください。Claude Code 本線の開発速度を上げることを目的に、
> 仕様整理だけでなく、実装・test・scanner・docs 草案も並列で任せます。

これを受け、設計 doc §14-19 に **v1.1 補足セクション** を追加 (= v1.0
を書き換えず、運用シフトを上乗せ):

| 観点 | v1.0 | v1.1 |
|---|---|---|
| Codex 主用途        | レビュー + 限定実装         | **実装本体 + tests + scanner + docs 並列** |
| 並列度上限           | L3 は 1 同時                | **L3 を 3-5 本並列** (1 branch / 1 ownership) |
| 本線業務比率         | レビュー中心                | **PM + Integrator + Final Reviewer 比率 ↑** |
| 初回候補            | F286-PNL-R2 単独 5 sub      | **F286-PNL-R2 / F286-DATA-R3 / FIRE-AUDIT-R1 / F286-ORDER-R1 dummy / F286-INTRA-R2 PoC** 並列 |
| Branch 戦略         | (未定義)                    | **1 Codex task = 1 feature branch (`codex/<id>`)** |

### Codex 実装レーンの標準運用 (= 設計 doc §15)

1 task の標準形は yaml で定義 (= §15.1):

```yaml
task_id / parent_task / lane / branch / allowed_files /
forbidden_files / expected_outputs / test_command /
report_required / merge_owner / db_write_allowed /
line_send_allowed / token_read_allowed
```

`db_write_allowed` などは `false` が default。**埋まっていなければ
Codex 投入禁止** (= §15.3 チェックリスト)。

本線 merge は **本線 Integrator + HQ 判断のみ** (= §15.4)。Codex
自身が develop に push することは構造的禁止。

### Codex に実装させる初回候補 (= 設計 doc §16、5 件)

| 候補 | 実装適性 | sub | branch |
|---|---|---|---|
| F286-PNL-R2 Advisory Snapshot Auto-Ingest      | ★★★ 高 | 5 | `codex/f286-pnl-r2-*` |
| FIRE-AUDIT-R1 Safety Scanner / Forbidden Op    | ★★★ 高 (read-only) | 2-3 | `codex/fire-audit-r1-*` |
| F286-DATA-R3 Daily Refresh Cron                | ★★ 中 | 3 | `codex/f286-data-r3-*` |
| F286-INTRA-R2 Intraday Advisory Trigger Engine | ★★ 中 (PoC のみ) | 2-3 | `codex/f286-intra-r2-*` |
| F286-ORDER-R1 Manual Order Draft Generator     | ★ 条件付き可 (Step 1 placeholder のみ) | 2 | `codex/f286-order-r1-*` |

**F286-ORDER-R1 の再評価** (= v1.0 では「× 不可」だった):

| 段階 | Codex 可否 | 内容 |
|---|---|---|
| Step 1 placeholder / 文字列 template | **★ 可** | 値は dummy、計算ロジック未実装 |
| Step 2 実 draft 価格 / 数量 / OCO 生成 logic | × 不可 | 構造的禁止と直接衝突、本線単独 |
| Step 3 LINE 統合 + 本番送信 | × 不可 | LINE 本番送信は本線のみ |

**FIRE-AUDIT-R1 を初回候補に追加** (= read-only AST 走査、副作用ゼロ)。

## v1.1 初回投入プラン (= 設計 doc §17)

```
Wave 1 (3 lane 並列、~15 分):
  - FIRE-AUDIT-R1-sub-A (forbidden import scan)
  - FIRE-AUDIT-R1-sub-B (SQL / token hardcode scan)
  - F286-PNL-R2-sub-1 (Design)
        ↓ 本線 Architect approve
Wave 2 (4 lane 並列、~15-20 分):
  - F286-PNL-R2-sub-2 (Impl)
  - F286-PNL-R2-sub-3 (Test)
  - F286-DATA-R3-sub-D1 (cron runner Impl + Test)
  - F286-PNL-R2-sub-5 (Docs draft)
        ↓
Wave 3 (1 lane、~5-10 分):
  - F286-PNL-R2-sub-4 (Audit)
        ↓
Wave 4 (本線、~15-20 分):
  - Integrator → Final Reviewer → HQ 報告

合計: 約 50-70 分 (= 本線単独 120-180 分のタスクを 30-50% 短縮想定)
```

設計 doc §17.2 に **file ownership 表 5-8 件分の yaml** を完備、
§18 で **コピペ用プロンプト構造** を整理 (= §8 雛形 + §9 報告テンプレ
+ §17.2 yaml ブロック の結合)。

## 次タスク

1. ★ FIRE-CODEX-R1 完了 (= 本書、v1.1 設計確定 + 運用開始)
2. HQ (Fujiwara) が次の方向を判断:
   - 案 X1: **v1.1 初回投入プラン (= 設計 doc §17) を実行** (= Wave 1-4)
     - sub-A/B (FIRE-AUDIT-R1) で repo audit を先に取り、後続 task の
       品質を担保するのが推奨
   - 案 X2: F286-PNL-R2 を単独 5 sub で先行 (= v1.0 設計 §11)
   - 案 X3: F286-DATA-R3 を 3 lane で先行 (= 中規模)
   - 案 X4: F286-INTRA-R2 PoC を本線単独で進める
   - 案 X5: F286-ORDER-R1 Step 1 placeholder を Codex に出す

## 関連リンク

- 設計 doc 本体: [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[../CLAUDE]] vault
- [[../タスク運用ルール]]
- [[../log]]
