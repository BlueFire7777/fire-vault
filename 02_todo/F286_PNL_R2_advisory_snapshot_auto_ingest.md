---
id: F286-PNL-R2
phase: P5 / 第 13 章 / 第 19 章 R-19-08 Phase 2
priority: 最優先
status: 完了 ★ (= Wave 2 で実装完了、HQ 統合 approve + 本線 merge 待ち)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R1 (= advisory_decisions 三段ガード正本)
  - FIRE-CODEX-R1 v1.1 (= Codex 実装部隊化、Wave 2 で実戦投入)
  - F062-R5 (= production Advisory 配信ループ)
chapter: 第 13 章 / 第 19 章 R-19-08 / 第 25 章 / 第 26 章
---

# F286-PNL-R2: Advisory Snapshot Auto-Ingest

最終更新: 2026-05-11

## ★ 状態: 完了 (= Wave 2 実装、本線 Integrator review 完了、HQ 統合 approve 待ち)

F062 本番 Advisory payload を **Advisory Snapshot** として保存し、
F286-PNL-R1 の `advisory_decisions` table に **手入力なしで** ingest
する pipeline を Codex 4 lane 並列で実装。本線 Integrator がすべての
成果物を review、回帰 3,515 PASS 達成。fire リポへの commit は HQ
統合 approve 後に本線が実施予定。

## 設計サマリ (= sub-1 設計 draft から要点抽出)

- **案 P 採用**: 別テーブル正規化
  - `advisory_snapshots` (= 通知単位 header)
  - `advisory_snapshot_rows` (= 銘柄単位、1 通知 N 行)
  - `advisory_decisions` は Fujiwara 判断 + 取引 + PnL のみ持つ
- **advisory_id 採番**: `<send_intent>-<base_date>-<sha16(payload)>`
  例: `production-advisory-2026-05-09-1e44f004ac168328`
- **send_id = advisory_id 同一**

## HQ 承認 8 件 × 実装ファイル × 検証 test の対応表

| # | HQ 承認内容 | 実装ファイル | 検証 test |
|---|---|---|---|
| 1 | `advisory_id = send_id` 同一                       | pnl/snapshot.py | tests/pnl/test_snapshot.py TestPayloadToSnapshot |
| 2 | hash input に `generated_at_utc` 含める             | pnl/snapshot.py | TestBuildAdvisoryId |
| 3 | LINE 成功・snapshot 失敗 → non-zero + 後追い helper | (caller 責務、F062 runner 改修は別 sub-task) | N/A |
| 4 | LINE 失敗時 → snapshot 残さない                    | (caller 責務 = F062 runner)             | N/A |
| 5 | `created_at = decision row` 作成時刻                | pnl/snapshot.py SnapshotStore.save_snapshot | TestSnapshotStoreSaveSnapshot |
| 6 | seed upsert で `notes` 触らない                     | pnl/snapshot.py                         | TestSeedAdvisoryDecisionsBehavior |
| 7 | `--record-decisions` は production-only             | pnl/snapshot.py `payload_to_snapshot` で refuse | TestPayloadToSnapshot |
| 8 | `ensure_schema()` に snapshot DDL                  | pnl/schema.py                           | TestEnsureSchemaSnapshotTables |

すべて Codex 実装 + 本線 Integrator 動作確認済。具体 test method 名は
最終 review 時に微修正の余地あり (= test file 内の表記と完全一致)。

## 実装ファイル一覧 (= Codex 直接生成、本線 Integrator review 済、commit 未)

| ファイル | 内容 | 行数 / tests | Codex lane |
|---|---|---|---|
| `pnl/snapshot.py`                                      | AdvisorySnapshot + Row + Store + payload_to_snapshot + build_advisory_id + compute_payload_hash | 772 行 | sub-2 (L3 Impl) |
| `pnl/schema.py`                                        | snapshot DDL 2 table + INDEX、ensure_schema 拡張 | +105 行 | sub-2 |
| `tests/pnl/test_snapshot.py`                          | payload→snapshot / 三段ガード / idempotent / malformed | 50 tests PASS | sub-3 (L2 Test) |
| `scripts/jobs/run_f286_data_r3_daily_refresh.py`      | F286-DATA-R3 daily refresh **skeleton** (= dry-run 中心、実 fetch なし) | 約 14 KB | sub-D1 (L3 Impl) |
| `tests/scripts/jobs/test_run_f286_data_r3_daily_refresh.py` | runner skeleton tests | 16 tests PASS | sub-D1 |

## 三段ガード適用

- **SnapshotStore コンストラクタ**: read_only=False + db_label='staging' +
  db_path basename='fire.staging.db' の三段検査
- **pnl.schema.ensure_schema**: db_label='staging' 必須キーワード +
  basename 'fire.staging.db' 必須
- **F286-DATA-R3 runner**: 同じ三段ガードを再利用 (= pnl.storage の
  WRITE_ALLOWED_DB_LABELS / WRITE_ALLOWED_DB_BASENAMES を import)
- 偽装 path (`--db-path data/fire.db --db-label staging`) は refuse

## 動作確認 (= 本線 Integrator smoke)

```
advisory_id: production-advisory-2026-05-09-1e44f004ac168328
send_id == advisory_id: True
action_mode=True + bwa → decision_label='待ち'
idempotent same payload: True
generated_at_utc 変化 → 別 advisory_id: True
三段ガード: production refuse / develop refuse / staging+production-basename refuse / staging+staging-basename OK
回帰: 3,515 PASS (= 3,449 baseline + 66 新規)
```

## 安全要件 (= Wave 2 内で全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信 (Wave 2 中)                       | 0 通 ✓ |
| DB write (production / develop / staging)     | 0 ✓ (= 実装のみ、smoke なし) |
| token / channel_token / secret 参照            | 0 ✓ |
| 自動発注 / 楽天操作 / Computer Use / Playwright | なし ✓ |
| 注文価格 / 数量 / 執行指示の生成 helper       | 実装に含めない ✓ |
| .github/workflows/ 変更                       | 0 ✓ |
| --no-verify                                   | 不使用 ✓ |
| scripts/seed_pattern_layer1.py                | 未接触 ✓ (= mtime Wave 2 開始前のまま) |
| simulation/research_lane/historical_indicators.py | 未接触 ✓ |
| TODO Excel                                    | 未更新 ✓ |
| cron / launchd / crontab 本番登録              | 0 ✓ |
| Codex 直接 commit                              | なし ✓ |
| Codex sandbox 違反                            | sub-5 で fire-vault 書込 NG (= 設計通り、本線が代替) |
| allowed_files 違反                            | 0 ✓ (= Codex が触ったのは指定 5 ファイルのみ) |

## Known Limitations / 既知の制約

1. **F062 runner 改修は別 sub-task**: `--record-decisions` option 追加 +
   payload に send_id 埋め込みは Wave 3 候補 (F286-PNL-R2-sub-runner)。
2. **実 F062 → snapshot ingest smoke は別 sub-task**: 実際の staging
   write を伴う E2E smoke は HQ approve 後の Wave 3。
3. **F286-DATA-R3 は skeleton**: `list_daily_refresh_jobs` /
   `plan_daily_refresh` の骨格のみ、実 fetch / 実 write は sub-D2 候補。
4. **cron 本番登録は凍結**: launchd plist / crontab 変更は HQ approve
   付きの sub-D3 候補まで保留。
5. **Codex sub-5 sandbox 制約**: Codex は fire リポ外 (fire-vault) に
   書けず、本タスクの vault doc は本線が代替作成。HQ 報告 draft
   `/tmp/codex_wave2_hq_report_draft.md` は Codex が作成、本線が
   review してから HQ コピペ用に使用。

## HQ 判断論点

1. **F286-PNL-R2 snapshot 実装を本線へ統合してよいか** (= 推奨: approve)
2. **F286-DATA-R3 skeleton を同一 commit に含めるか、別 commit か** (= 別 commit 推奨)
3. **F062 runner `--record-decisions` 統合を Wave 3 前に進めるか**
4. **staging DB smoke をいつ許可するか** (= 推奨: HQ approve 必須)
5. **後追い ingest helper を runner 統合と同時に必須化するか**
6. **DATA-R3 の実 fetch / 実 write は sub-D2 として HQ approve 後に進めるか**
7. **cron 登録は sub-D3 まで凍結する方針で問題ないか**

## HQ 追加方針受領 (= 2026-05-11、新ラベル方針)

Wave 2 終盤に HQ から FIRE Advisory / LINE UX の表示ラベル方針変更を
受領 (= 5 段階の新ラベル):

| 新ラベル | 意味 |
|---|---|
| 🟢 積極的買い推奨   | 最上位、条件が崩れていなければ手動買い優先可 |
| 🟡 条件付き買い推奨 | VWAP 上 / 出来高増 / 地合い悪化なし 等の条件成立で買い候補 |
| 🟠 場中監視         | まだ買わない、初押し・再加速・出来高確認待ち |
| ⚠️ 注意つき買い候補 | 上昇余地はあるがボラ・決算・需給に注意 (新規 bucket) |
| 🔴 見送り推奨       | 今日は触らない方がよい |

旧 → 新 mapping:
- 「買い検討候補」「買い候補」「今すぐ買い」→ 「積極的買い推奨」or「条件付き買い推奨」
- 「待ち」→ 「場中監視」
- 「まだ買わない / 場中監視候補」→ 「場中監視」
- 「見送り」→ 「見送り推奨」
- 「買え」「今すぐ買え」命令口調は禁止

### 本 Wave 2 では scope 外として分離

HQ 方針: 「現在対応中タスクの scope を無理に拡張しない」。

Wave 2 で Codex 実装した `pnl/snapshot.py` の `_ACTION_VERDICT` と
`tests/pnl/test_snapshot.py` の expected 値は **旧ラベル**
(「買い候補」「待ち」「見送り」等) のまま。これは:

1. `pnl/snapshot.py` の `_ACTION_VERDICT` は F062 LINE 表示で使われる
   `notifications/templates/research_advisory.py` の `ACTION_LABEL_VERDICT`
   と **同一文字列を共有** する設計
2. snapshot 側だけ新ラベルに変更すると、LINE 表示 (F062-R5.8 既送信)
   と DB 保存 (snapshot) で文字列が乖離する
3. F062 側変更には:
   - `notifications/templates/research_advisory.py` の
     ACTION_LABEL_BADGE / ACTION_LABEL_VERDICT 更新
   - `tests/notifications/templates/test_research_advisory_line_template.py`
     の expected 文字列更新 (= 既に R5.8 で送信した文面と整合)
   - F062-R5.8 の既送信履歴との連続性 (= 旧 LINE 受信内容と新 LINE
     受信内容を区別する mode 切替 or migration)
   が必要

→ **Wave 2 scope を大幅超過するため分離**。

### 次タスク候補 (= 新ラベル方針対応の分離)

**FIRE-LABEL-R1: Advisory / LINE UX Label Refresh** を起票推奨:

- `notifications/templates/research_advisory.py`: ACTION_LABEL_BADGE /
  ACTION_LABEL_VERDICT / ACTION_FLIP_CONDITION / 結論行を新ラベルに更新
- `tests/notifications/templates/test_research_advisory_line_template.py`:
  新ラベル文字列で全 action mode tests 更新
- `pnl/snapshot.py`: `_ACTION_VERDICT` を更新 (= F062 側と完全同期)
- `tests/pnl/test_snapshot.py`: snapshot decision_label の expected 更新
- 関連 vault docs: F062-R5.7 / R5.8 / FIRE-CODEX-R1 Wave 1/2 結果 doc を
  「旧ラベル時代の記録」として残し、新ラベルへの切替時期を log.md に
  decision entry として記録
- 命令口調 (「買え」「今すぐ買え」) のテンプレ全件検査 → 既に
  FORBIDDEN_PHRASES で「買え」は禁止済を再確認 + 新ラベル本文に
  混入しないか単体検査追加

**FIRE-LABEL-R1 単体での scope**:
- 表示ラベルのみ、優位性算出 / 採番 / DB スキーマは変更しない
- F062 runner で `--label-version v2` のような mode 切替を導入する
  かは Architect 判断 (= 旧 LINE 履歴との整合が必要なら mode 化)
- Stage 3 移行前にラベル統一が完了することが望ましい

### 本 vault doc の表記方針

本 doc には Wave 2 実装時点の旧ラベル文字列が複数登場するが、これは
**Wave 2 実装時点の事実記録** として保持。FIRE-LABEL-R1 完了後に
本 doc を更新する場合は revision history を追記する。

## 次タスク (= Wave 3 候補 + FIRE-LABEL-R1)

1. **Wave 3 Audit** (= sub-4 適用、推奨):
   - snapshot schema / store の adversarial review
   - F062 runner integration design review
   - staging write smoke plan の安全確認
   - DATA-R3 skeleton safety review
   - cron 登録前 checklist 整備
2. **本線 Integrator commit** (= HQ approve 後):
   - feat(F286-PNL-R2): add advisory snapshot ingest module
   - test(F286-PNL-R2): add advisory snapshot tests
   - feat(F286-DATA-R3): add daily refresh skeleton runner
   - test(F286-DATA-R3): add daily refresh skeleton tests
   - docs(F286-PNL-R2): document advisory snapshot design
3. **FIRE-LABEL-R1: Advisory / LINE UX Label Refresh** ★ 新規 (= 上記
   HQ 追加方針対応の分離タスク):
   - 旧 5 label → 新 5 label (= 🟢 積極的買い推奨 / 🟡 条件付き買い推奨 /
     🟠 場中監視 / ⚠️ 注意つき買い候補 / 🔴 見送り推奨) への一括切替
   - F062 LINE template + pnl/snapshot decision_label + 関連 tests を
     一斉更新、旧/新 mode 切替の必要性は Architect 判断

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE1_results|Wave 1 結果]]
- [[F286_PNL_R1_advisory_decision_pnl_tracking|F286-PNL-R1 三段ガード正本]]
- [[F062_R5_receipt_confirmation|F062-R5 受信確認]]
- [[../log]]
