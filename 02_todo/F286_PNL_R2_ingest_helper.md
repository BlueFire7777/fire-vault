---
id: F286-PNL-R2-ingest-helper
phase: P5 / 第 13 章 / 第 19 章 R-19-08 Phase 2
priority: 高
status: 起票 (= 2026-05-11、Wave 4 W4-2、Codex 投入 HQ approve 待ち)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R2 (= snapshot module、commit cb17a7f)
  - FIRE-CODEX-R1 v1.1 Wave 3 sub-4B (= integration design audit)
chapter: 第 13 章 / 第 19 章 R-19-08
---

# F286-PNL-R2-ingest-helper: LINE 送信成功 / snapshot 失敗時の後追い ingest

最終更新: 2026-05-11

## ★ 状態: 起票 (= Wave 4 W4-2)

F062 LINE 送信は成功したが snapshot 保存が失敗した場合の **後追い
ingest helper**。F062 runner 統合 (W4-1) とは **分離** した独立 runner。

Wave 4 内では **dry-run / safety path 中心**、staging DB write は
**実施しない**。

## 目的

W4-1 で `--record-decisions` の経路で SnapshotStore が異常終了した
場合、F062 send_smoke は `exit 1 + stderr に後追い ingest command` を
出す。本 helper はその command を実行するための独立 runner。

実用上の動機:
- LINE 送信は外部 API への通信のため、一度成功すると取り消せない
- snapshot 保存失敗時は次回 send_smoke で同じ payload を送ると **同じ
  LINE が再送信** されて二重通知になる
- snapshot 保存だけ後追いで実行できる helper が必須

## 設計参照

- [[FIRE_CODEX_R1_WAVE3_audit_results|Wave 3 sub-4B integration audit]]
  「§4 後追い ingest helper API 設計」

## 実装内容 (= Codex 投入予定)

### scripts/jobs/run_f286_pnl_r2_ingest_snapshot.py (新規)

CLI:
```
--payload-json /path/to/f062_payload.json   # F062 が出した payload (= LINE 送信時の正本)
--db-path data/fire.staging.db               # 三段ガード、basename='fire.staging.db' 必須
--db-label staging
--dry-run                                     # default、--write を明示しない限り
--write                                       # staging 書き込み
--ensure-schema                               # snapshot tables 冪等作成
--output-json /tmp/f286_pnl_r2_ingest_payload.json
--completion-report /tmp/f286_pnl_r2_ingest_report.txt
--force                                       # 既存 advisory_id snapshot を上書きするか
                                                (default: idempotent UPDATE、--force なしでも同 PK 上書き OK)
```

動作:
1. payload_json 読み込み
2. `payload_to_snapshot(payload)` で snapshot 構築 (= message_mode
   != production なら refuse)
3. dry-run mode: 構築だけ確認、DB 接続なし
4. write mode: SnapshotStore(db_label='staging', read_only=False).
   save_snapshot(snap) を呼ぶ
5. 結果 JSON 出力 (= advisory_id / inserted / updated 件数)

LINE 送信は **絶対に行わない** (= LineBotClient / linebot SDK を
import しない、AST 検査で保証)。

### tests/scripts/jobs/test_run_f286_pnl_r2_ingest_snapshot.py (新規)

- TestParseArgs:
  - --payload-json 必須
  - --write + production / develop / 偽装 path refuse
- TestDryRunMain:
  - DB 未作成のまま dry-run、exit 0
- TestWriteWithStaging:
  - tmp_path 配下 fire.staging.db basename で --write + --ensure-schema
  - snapshot 1 件 INSERT、rows N 件、advisory_decisions seed N 件
- TestIdempotentRerun:
  - 同じ payload で 2 回 --write → row 数増えない
- TestPreviewPayloadRefused:
  - message_mode='preview' の payload は AdvisorySnapshotError で refuse
- TestArgParserSafety / TestRunnerSourceSafety:
  - --send / --token / --auto-order 不在
  - linebot / playwright / selenium 不 import

## allowed_files

- scripts/jobs/run_f286_pnl_r2_ingest_snapshot.py (= 新規)
- tests/scripts/jobs/test_run_f286_pnl_r2_ingest_snapshot.py (= 新規)

## forbidden_files

- pnl/* (= snapshot module 既存、import のみ可)
- scripts/jobs/run_f062_* (= W4-1 担当、衝突回避)
- notifications/* (= W4-3 FIRE-LABEL-R1 担当)
- scripts/seed_pattern_layer1.py
- simulation/research_lane/historical_indicators.py
- TODO Excel
- .github/workflows/*

## expected_outputs

1. 新規 runner (= 構造は run_f286_pnl_r1_record_decision.py pattern)
2. 新規 tests (= 既存 PNL-R1 / PNL-R2 tests と同じ pattern)
3. completion report テンプレ

## test_command

```
.venv/bin/pytest tests/scripts/jobs/test_run_f286_pnl_r2_ingest_snapshot.py -v
```

全 PASS。tmp_path のみ、本物の staging DB に書かない。

## 安全要件

- LINE 本番送信なし (= LineBotClient / linebot 完全不 import、AST 検査)
- production / develop / staging DB write なし (= 本タスク内、tests は tmp_path)
- token / channel_token / secret 参照なし
- 楽天 / Computer Use / Playwright なし
- 注文価格 / 数量 / 執行指示の生成 helper を追加しない
- workflow 変更なし / --no-verify 不使用
- scripts/seed_pattern_layer1.py / historical_indicators.py 未接触
- TODO Excel 未更新

## Codex 投入予定 (= HQ approve 後)

| 段階 | 内容 | lane | 想定時間 |
|---|---|---|---|
| Codex L3 Impl | runner + tests を生成 | L3 | 10-15 分 |
| 本線 Integrator review | smoke + 安全項目 + ガード検証 | (本線) | 5-10 分 |
| commit | feat(F286-PNL-R2): add ingest helper runner | (本線) | 5 分 |

## Wave 4.1 staging smoke 候補

本 helper 完成後、staging smoke の **最も安全な経路** (= LINE 送信
なし、staging 書き込みのみ) として使える:

```
.venv/bin/python -m scripts.jobs.run_f286_pnl_r2_ingest_snapshot \
  --payload-json /tmp/f062_r5_8_line_payload.json \
  --db-path data/fire.staging.db --db-label staging \
  --write --ensure-schema
```

ただし **HQ 別 approve 必須**、本タスクで実施しない。

## 関連リンク

- [[FIRE_CODEX_R1_WAVE4_plan|Wave 4 plan]]
- [[F286_PNL_R2_advisory_snapshot_auto_ingest|F286-PNL-R2 module]]
- [[F286_PNL_R2_runner_record_decisions|W4-1 F062 runner 統合]]
- [[FIRE_CODEX_R1_WAVE3_audit_results|Wave 3 sub-4B integration audit]]
