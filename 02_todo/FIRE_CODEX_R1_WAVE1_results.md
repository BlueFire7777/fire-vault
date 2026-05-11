---
id: FIRE-CODEX-R1-WAVE1
phase: ガバナンス / Codex 並列実装初回投入 / R-01-08 整合
priority: 最優先
status: 完了 ★ Wave 1 完了 (2026-05-11、CRITICAL false positive のみ)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 (= 設計確定)
  - F286-PNL-R1 (= 三段ガード正本)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 Wave 1: Audit ×2 + Design Proposal ×1 完了

最終更新: 2026-05-11

## ★ 状態: 完了 (Wave 2 進行可、CRITICAL は全件本線 context 確認済)

FIRE-CODEX-R1 v1.1 初回投入プラン Wave 1 (= 3 lane 並列) を実戦投入し、
全 3 タスクを完了。Codex 成果物は **本線 Integrator が context 確認**
してから採用 / 差し戻し判断を実施。

実 LINE 送信なし / DB write なし / fire リポ touched なし。

## 実施結果

### Wave 1 投入結果 (= 3 lane)

| sub | lane | status | 成果物 | Codex 検出 CRITICAL | 本線判定 |
|---|---|---|---|---|---|
| sub-A FIRE-AUDIT-R1 forbidden import scan | L4 Audit | ✓ 完了 | /tmp/fire_audit_r1_forbidden_imports_report.md (1253 行) | 502 件 | **全 false positive** (safety note の文字列マッチ) |
| sub-B FIRE-AUDIT-R1 SQL/token hardcode scan | L4 Audit | ✓ 完了 | /tmp/fire_audit_r1_sql_token_hardcode_report.md (80 行) | 16 件 | **既存設計課題、FIRE-OPS-R0 で対応予定、Wave 2 影響なし** |
| sub-1 F286-PNL-R2 design proposal | L1 Design | ✓ 完了 | /tmp/f286_pnl_r2_design_draft.md (300 行) | 0 | **設計案として採用可、Architect 確認事項 8 件を本線が判断要** |

### sub-A 詳細: forbidden import scan

Codex 検出 (= 5 観点で 502 件):

| 観点 | 件数 | 本線 context 検証結果 |
|---|---|---|
| LINE SDK (whitelist 除く)        | 221 件 | 例外型 import (`linebot.v3.messaging.exceptions`) が大半、設計上許可。**実害なし** |
| broker / 楽天 / Computer Use    | 103 件 | docstring 内 safety note (例「no order / broker / rakuten / Computer Use」)、**全て negation 表現** |
| token / secret hardcode         | 1 件   | `tests/notifications/test_line_bot.py:183` の `UsendModeSecretDoNotLeakAAAAAAAA` (= 明らかな dummy test fixture)、masking 検査 |
| workflow / --no-verify          | 2 件   | docstring 内「--no-verify 禁止」の safety note。**実害なし** |
| 自動発注 / 注文価格生成         | 275 件 | `auto_order_allowed=False` の構造的禁止フラグ、`force_close` は LINE 緊急アラート用語、**実害なし** |

**本線 Integrator 判断**:
- 502 件中 **真の CRITICAL は 0 件**
- Codex の literal 検出のため false positive 多数 → v1.2 改訂で「AST + docstring context 除外を必須化」を検討
- Wave 2 進行への影響: **なし**

推奨される将来改善 (= 別タスク候補):
- LINE SDK 例外型 import は専用 `notifications/line_exceptions.py` に集約 (= R2 scope 外)
- test fixture の dummy token を `"dummy-..."` prefix に統一 (= 軽微)

### sub-B 詳細: SQL/token hardcode scan

Codex 検出 (= 構造的検出 16 件):

| 観点 | 件数 | 本線 context 検証結果 |
|---|---|---|
| production/develop DB write 経路 | 7 件 | `sqlite3.connect(db_path)` で `db_path` のデフォルトが `DB_PATH = Path(os.getenv("DB_PATH", "data/fire.db"))` の **既存設計** |
| staging 偽装ベクタ              | 0 件 | ✓ クリーン |
| token/channel_token 直書き      | 0 件 | ✓ クリーン |
| 危険な db_path ハードコード     | 9 件 | 同上、既存 DB_PATH fallback パターン |
| WAL 違反候補 (情報)              | 2 件 | INFO 扱い、設計影響なし |

検出された 16 件はすべて **既存コード全体の DB_PATH fallback 設計**:
- features/base.py, features/regime.py, market_data/index_fetcher.py,
- patterns/{labels,lanes,store}.py, scripts/jobs/{fetch_intraday_data,
  lane_c_universe_precheck}.py, scripts/setup/init_db.py, utils/config.py

これらは **FIRE 全体の DB ハンドリング設計** であり、三段ガード (=
F286-PNL-R1 で確立) と矛盾する旧来の運用。修正には全 module の
ライフサイクル変更が必要 → **FIRE-OPS-R0 案 1 (= production write
統一)** と整合する別タスクで対応推奨。

**本線 Integrator 判断**:
- pnl/ 配下 (= F286-PNL 系) は三段ガード遵守、**Wave 2 進行に影響なし**
- 16 件は **既存設計課題** として FIRE-OPS-R0 で扱う
- Wave 2 で新規追加する pnl/snapshot.py には三段ガード再利用を要求済 (= 設計 §7)

### sub-1 詳細: F286-PNL-R2 Design Proposal

Codex 設計案 (= 300 行 Markdown):
- §3 データモデル: 案 P (= 別テーブル `advisory_snapshots` +
  `advisory_snapshot_rows`) を推奨。`advisory_decisions` は判断 + 取引
  のみ持つ正規化方針。
- §5 mapping 表: F062 payload (`selected_rows[]`) → snapshot rows の
  完全 mapping 完備
- §6 advisory_id 採番: `<send_intent>-<base_date>-<sha8(payload)>`
  例 `production-advisory-2026-05-09-a1b2c3d4`
- §7 三段ガード再利用: `pnl.storage.WRITE_ALLOWED_DB_LABELS` +
  `pnl.schema.DDL_ALLOWED_DB_BASENAMES` 完全再利用
- §8 R1 共存: 既存 Fujiwara 判断 / actual_trade / PnL は **上書きしない**
  (= snapshot 由来列のみ UPDATE)
- §9 F062 runner 改修: `--record-decisions` option、production-advisory
  + LINE 送信成功後のみ保存
- §11 実装計画: sub-2 Impl / sub-3 Test への引き渡し順序
- §12 本線 Architect への確認事項 8 件 (= 後述)

**本線 Architect 判断 (= 8 件)**:

| # | 確認事項 | 本線判断 (= Architect 仮承認) |
|---|---|---|
| 1 | `advisory_id` と `send_id` は同一でよいか | **同一推奨** (= シンプル、後で send_attempt 概念追加可) |
| 2 | hash input に `generated_at_utc` を含めるか | **含める** (= 同 base_date 再送を別 ID にする、運用上の追跡性優先) |
| 3 | LINE 送信成功・snapshot 保存失敗時の exit code | **non-zero + stderr に後追い ingest command** (= LINE は不可逆) |
| 4 | LINE 送信失敗時に failed snapshot を残すか | **残さない** (= 「Fujiwara が受信した候補」のみ追跡) |
| 5 | `advisory_decisions.created_at` の意味 | **decision row 作成時刻** (= Fujiwara 判断時刻が必要なら R3 で `decided_at` 追加) |
| 6 | seed upsert 時に `notes` を触らない方針 | **触らない** (= snapshot 詳細は `raw_row_json` に保持) |
| 7 | `--record-decisions` を production-only に制限 | **production-only** (= preview 記録は別 option) |
| 8 | `ensure_schema()` に snapshot DDL を含めるか | **含める** (= F286-PNL 系の一括冪等作成として扱う) |

これら 8 件は **本線 Architect として仮承認**、HQ が異論あれば差し戻し可。

## 安全要件遵守 (= 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信 (Wave 1 中)                       | 0 通 ✓ |
| DB write                                       | 0 ✓ |
| token / channel_token / secret 参照            | 0 ✓ |
| fire リポ source ファイル変更                  | 0 ✓ (Codex は read-only audit + design proposal のみ) |
| .github/workflows/ 変更                       | 0 ✓ |
| --no-verify                                   | 不使用 ✓ |
| scripts/seed_pattern_layer1.py                | 未接触 ✓ |
| simulation/research_lane/historical_indicators.py | 未接触 ✓ |
| TODO Excel                                    | 未更新 ✓ |
| Codex 直接 commit                              | なし ✓ (= 本線が成果物を review してから記録) |
| 3 DB mtime (production/develop/staging)       | 全 unchanged ✓ |

## Codex 完了報告 (= §9 テンプレ準拠、3 件)

### sub-A 完了報告

```
task_id:            FIRE-AUDIT-R1-sub-A
lane:               L4 Audit
changed_files:      /tmp/fire_audit_r1_forbidden_imports_report.md (新規)
commits:            なし (= read-only audit、本線が記録)
tests:
  added:            0
  pytest result:    N/A (read-only audit)
safety_checks:
  forbidden_import: 502 件検出 (= 全 false positive、本線 context 確認済)
  forbidden_sql:    検査対象外 (sub-B 担当)
  forbidden_phrase: N/A
  three_stage_guard: N/A (= read-only)
DB_writes:          production: 0 / develop: 0 / staging: 0
LINE_sends:         0
secrets_touched:    none
forbidden_files_touched: none
known_risks:        Codex literal scan のため context 区別なし、本線 context 検証で false positive 判定
merge_recommendation: "本線 context 検証済、Wave 2 進行可"
```

### sub-B 完了報告

```
task_id:            FIRE-AUDIT-R1-sub-B
lane:               L4 Audit
changed_files:      /tmp/fire_audit_r1_sql_token_hardcode_report.md (新規)
commits:            なし
tests:
  added:            0
  pytest result:    N/A
safety_checks:
  forbidden_import: N/A (sub-A 担当)
  forbidden_sql:    DB_PATH fallback 16 件検出 (既存設計、FIRE-OPS-R0 で対応)
  forbidden_phrase: N/A
  three_stage_guard: pnl/ 配下は遵守済を確認、Wave 2 影響なし
DB_writes:          production: 0 / develop: 0 / staging: 0
LINE_sends:         0
secrets_touched:    none
forbidden_files_touched: none
known_risks:        既存 DB_PATH fallback 16 件は別タスク対応、Wave 2 範囲外
merge_recommendation: "本線 context 検証済、Wave 2 進行可"
```

### sub-1 完了報告

```
task_id:            F286-PNL-R2-sub-1
lane:               L1 Design
changed_files:      /tmp/f286_pnl_r2_design_draft.md (新規)
commits:            なし (= 設計のみ)
tests:
  added:            0
  pytest result:    N/A
safety_checks:
  forbidden_import: N/A (= 設計のみ)
  forbidden_sql:    DDL 案は staging のみ、三段ガード再利用方針
  forbidden_phrase: N/A
  three_stage_guard: §7 で再利用方針明示
DB_writes:          production: 0 / develop: 0 / staging: 0
LINE_sends:         0
secrets_touched:    none
forbidden_files_touched: none
known_risks:        send_id 衝突 / atomicity / PK 衝突 / decision_label drift 等 §10 で 8 件列挙
merge_recommendation: "本線 Architect が 8 件の確認事項を仮承認、HQ approve 後 Wave 2 sub-2 (Impl) へ"
```

## HQ 判断が必要な論点

1. **sub-A false positive 多数の取り扱い**:
   - 502 件中真の CRITICAL は 0 件、Codex の literal scan の制約
   - 推奨: v1.2 改訂で「AST + docstring context 除外を Audit lane の
     必須要件」に強化 (= 別 design task)
2. **sub-B の 16 件 DB_PATH fallback 課題**:
   - 既存全体設計の課題、本タスクで修正対象としない
   - 推奨: FIRE-OPS-R0 案 1 で staging-only 統一方針として対応
3. **sub-1 Architect 確認事項 8 件**:
   - 本線が仮承認、HQ が異論あれば差し戻し
   - 特に確認 #2 (hash input に generated_at_utc を含めるか) は
     再送時の挙動を決める重要判断
4. **Wave 2 進行可否**:
   - Wave 1 CRITICAL は実害ゼロ、Wave 2 への影響なし
   - 推奨: HQ approve で Wave 2 投入 (= sub-2 Impl + sub-3 Test +
     sub-D1 Cron + sub-5 Docs を並列で投入)

## 次タスク (= Wave 2 候補)

1. **Wave 2 並列投入** (= HQ approve 後):
   - sub-2 F286-PNL-R2 Impl (= pnl/snapshot.py)
   - sub-3 F286-PNL-R2 Test (= tests/pnl/test_snapshot.py)
   - sub-D1 F286-DATA-R3 cron runner
   - sub-5 F286-PNL-R2 Docs draft
2. Wave 2 完了後に Wave 3 (= sub-4 Audit) → Wave 4 (= 本線統合 + HQ 報告)

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_orchestration_design|FIRE-CODEX-R1 完了マーカー]]
- [[F286_PNL_R1_advisory_decision_pnl_tracking|F286-PNL-R1 (= 三段ガード正本)]]
- [[../log]]
