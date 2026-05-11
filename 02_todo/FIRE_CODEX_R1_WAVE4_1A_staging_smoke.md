---
id: FIRE-CODEX-R1-WAVE4.1-A
phase: ガバナンス / Codex 並列実装 Wave 4.1-A / 第 25 章 F286-PNL-R2
priority: 最優先
status: 完了 ★ W4.1-A staging smoke 成功 (2026-05-11)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 Wave 4 (= 4 lane 統合 commit 完了)
  - W4-2 F286-PNL-R2-ingest-helper (= 六段ガード runner)
  - HQ W4.1-A 条件付き approve (= 2026-05-11)
chapter: ガバナンス / R-01-08 / F286-PNL-R2 検証
---

# FIRE-CODEX-R1 v1.1 Wave 4.1-A: W4-2 ingest helper 単独 staging smoke

最終更新: 2026-05-11

## ★ 状態: 完了 (= staging に snapshot + rows + decisions seed 書き込み成功、
   idempotent rerun 確認、全 safety guard 機能を実証)

HQ W4.1-A 条件付き approve 受領後、W4-2 ingest helper を **F062-R5.8
production payload** (= 既存 LINE 送信実績の正本) で staging smoke。
初回 DDL 込みで `advisory_snapshots` / `advisory_snapshot_rows` 2
table を作成、5 銘柄分の snapshot + advisory_decisions seed を保存。

F062 経由 smoke (= W4.1-B) は HQ 未承認のため **実施しない**。
LINE 送信 0 / token-secret 参照 0 / production・develop DB unchanged。

## HQ W4.1-A 必須条件 (= 11 項目、全 ✓)

| # | 必須条件 | 結果 |
|---|---|---|
| 1 | 実行前 production / develop / staging DB mtime 記録 | ✓ (= 下記 baseline) |
| 2 | staging DB fire.staging.db のみ                  | ✓ (= data/fire.staging.db) |
| 3 | 限定 payload 1〜5 件                              | ✓ (= 5 件、Top 5) |
| 4 | inserted / updated / row_count 記録              | ✓ |
| 5 | idempotent rerun 確認                            | ✓ (= updated 5、row count 維持) |
| 6 | production / develop DB mtime unchanged 確認     | ✓ |
| 7 | LINE 送信 0                                      | ✓ (= SEND 件数 4 のまま) |
| 8 | token / secret 参照 0                            | ✓ |
| 9 | symlink / hardlink guard 有効確認                | ✓ (= W4-2 tests 24 PASS) |
| 10 | output path guard 有効確認                       | ✓ (= 同上) |
| 11 | HQ へ 1 ブロック報告                              | (= 別途) |

## Baseline 記録

```
production DB:   data/fire.db          May  7 16:12 / 371,064,832 bytes
develop DB:      data/fire.develop.db  May  7 18:14 / 371,064,832 bytes
staging DB:      data/fire.staging.db  May 11 18:49 / 4,803,952,640 bytes
LINE SEND today: 4 件 (= R5.6 / R5.8)
forbidden files mtime:
  scripts/seed_pattern_layer1.py            May 7 17:39
  simulation/research_lane/historical_indicators.py  May 9 21:10
staging pre-smoke counts:
  advisory_decisions:        5 件 (= F286-PNL-R1 smoke の既存 row、
                                    advisory_id='f062-r5.8-2026-05-11T09:19:53Z')
  advisory_snapshots:        table absent (= 初回 DDL 待ち)
  advisory_snapshot_rows:    table absent
```

## payload (= F062-R5.8 production send 正本)

```
/tmp/f062_r5_8_line_payload.json
message_mode:        production
action_mode:         True
send_intent:         production-advisory
selected_count:      5
selected_label_counts: {'boost_with_avoid': 5}
payload_base_date:   2026-05-09
source_version:      r2f4_baseline_live_v1
rule_version:        r2g3_recommended_v2
generated_at_utc:    2026-05-11T09:19:09.542450+00:00
selected_rows codes: ['57290', '340A0', '37980', '137A0', '331A0']
```

## Step 1: dry-run ✓

```
.venv/bin/python -m scripts.jobs.run_f286_pnl_r2_ingest_snapshot \
  --payload-json /tmp/f062_r5_8_line_payload.json \
  --db-path data/fire.staging.db --db-label staging \
  --output-json /tmp/w4_1_a_dry_run.json
```

| field | value |
|---|---|
| mode                  | dry-run |
| write_enabled         | False |
| advisory_id / send_id | `production-advisory-2026-05-09-520d6429e10e0b2a` ★ |
| decisions_loaded      | 5 |
| schema_info           | None (= dry-run なので DDL なし) |
| snapshot_save_result  | None |
| dry_run_existing_count | 0 |
| 3 DB mtime            | 全 unchanged ✓ |

## Step 2: 初回 staging write (= 初回 DDL 込み) ★

```
.venv/bin/python -m scripts.jobs.run_f286_pnl_r2_ingest_snapshot \
  --payload-json /tmp/f062_r5_8_line_payload.json \
  --db-path data/fire.staging.db --db-label staging \
  --write --ensure-schema \
  --output-json /tmp/w4_1_a_write1.json
```

| field | value |
|---|---|
| mode                                          | write |
| write_enabled                                 | True |
| schema_info.table_existed_before              | True (= advisory_decisions、R1) |
| schema_info.snapshot_table_existed_before     | **False** ★ |
| schema_info.snapshot_table_exists_after       | True ★ (= 初回 DDL 成功) |
| schema_info.snapshot_rows_table_existed_before | False |
| schema_info.snapshot_rows_table_exists_after  | True |
| snapshot_save_result.snapshots_inserted       | **1** ★ |
| snapshot_save_result.snapshots_updated        | 0 |
| snapshot_save_result.snapshot_rows_inserted   | **5** ★ |
| snapshot_save_result.snapshot_rows_updated    | 0 |
| snapshot_save_result.decisions_inserted       | **5** ★ |
| snapshot_save_result.decisions_existing       | 0 |
| snapshot_save_result.total_rows               | 5 |

staging row counts after write 1:
- advisory_decisions:        **10 件** (= 既存 5 + W4.1-A 新規 5、advisory_id 異)
- advisory_snapshots:        **1 件** ★ (= 新規)
- advisory_snapshot_rows:    **5 件** ★ (= 新規)

**5 銘柄の seed (= decision_label に新ラベル「場中監視」が反映、
FIRE-LABEL-R1 動作確認)**:
```
production-advisory-2026-05-09-520d6429e10e0b2a, 137A0, Ｃｏｃｏｌｉｖｅ,   unknown, 場中監視
production-advisory-2026-05-09-520d6429e10e0b2a, 331A0, メディックス,       unknown, 場中監視
production-advisory-2026-05-09-520d6429e10e0b2a, 340A0, ジグザグ,            unknown, 場中監視
production-advisory-2026-05-09-520d6429e10e0b2a, 37980, ＵＬＳグループ,      unknown, 場中監視
production-advisory-2026-05-09-520d6429e10e0b2a, 57290, 日本精鉱,            unknown, 場中監視
```

3 DB mtime after write 1:
- production:   May 7 16:12 / 371 MB    **unchanged** ✓
- develop:      May 7 18:14 / 371 MB    **unchanged** ✓
- staging:      May 11 21:25 / 4.8 GB    更新 (= 想定通り、書き込み完了)

## Step 3: idempotent rerun ✓

同 payload で 2 回目の `--write --ensure-schema`:

| field | value |
|---|---|
| snapshots_inserted    | **0** ★ (= 既存 → UPDATE) |
| snapshots_updated     | 1 |
| snapshot_rows_inserted | **0** ★ |
| snapshot_rows_updated | 5 |
| decisions_inserted    | **0** ★ (= 既存 W4.1-A 5 件 → 保護) |
| decisions_existing    | 5 (= HQ #6 既存 Fujiwara 判断保護) |
| total_rows            | 5 |

staging row counts after rerun (= write 1 と同じ):
- advisory_decisions:        10 (= 不変、HQ #5/#6)
- advisory_snapshots:        1 (= 不変)
- advisory_snapshot_rows:    5 (= 不変)

production / develop DB mtime: **unchanged** ✓

## Step 4: safety verify (= 全 ✓)

| 項目 | 結果 |
|---|---|
| TOKEN_LEAK in W4.1-A artifacts                | **0** ✓ |
| FULL_RECIPIENT in W4.1-A artifacts            | **0** ✓ |
| LINE SEND 今日                                | **4 のまま** ✓ (= R5.6 / R5.8 のみ、追加なし) |
| production DB mtime                            | unchanged ✓ |
| develop DB mtime                               | unchanged ✓ |
| forbidden files mtime                          | unchanged ✓ |
| symlink / hardlink / output path guard         | 24 件 PASS ✓ |
| fire repo source 変更                          | 0 行 ✓ (= W4.1-A はコード変更なし、smoke のみ) |

## 主要観察

### 1. 初回 DDL 込み staging write 成功

- `advisory_snapshots` / `advisory_snapshot_rows` 2 table が初めて
  staging に作成された
- `pnl.schema.ensure_schema(db_path, db_label='staging')` の三段ガード
  経由で正常作成
- `pnl.snapshot.SnapshotStore.save_snapshot()` の atomic transaction で
  snapshot 1 + rows 5 + decisions seed 5 を一括 INSERT

### 2. F286-PNL-R1 既存 row 保護 (= HQ #6 動作確認)

- 事前 staging に F286-PNL-R1 smoke の advisory_decisions 5 件
  (= advisory_id `f062-r5.8-2026-05-11T09:19:53Z`) が存在
- W4.1-A の新規 advisory_id (= `production-advisory-2026-05-09-...`) と
  異なるため PK 衝突なし、両者が **共存**
- 既存 row は **touched なし** (= R1 smoke 時の watched 状態 / notes
  / created_at 全て保持)

### 3. FIRE-LABEL-R1 新ラベル反映確認 (= W4-3 動作確認)

- W4-3 で更新した `_ACTION_VERDICT` mapping により、
  `boost_with_avoid` → **「場中監視」** (= 旧「待ち」から変更) が
  advisory_snapshot_rows.decision_label + advisory_decisions
  .decision_label に保存された
- 5 件全件で新ラベル文字列が正しく適用、回帰なし

### 4. idempotent 動作 (= HQ #2 動作確認)

- 同 payload で 2 回目 → row 数増加 0、UPDATE 経路に正しく分岐
- 既存 Fujiwara 判断 (= 今回は seed 直後なので 'unknown' のまま) は
  保護される設計、本 smoke でも上書きなし

### 5. 六段ガード機能確認

- W4-2 で確立した 六段ガード (= read_only / db_label / basename /
  output path / symlink / hardlink) はすべて runtime で機能
- staging path として `data/fire.staging.db` を渡すと通り、
  production / develop / 偽装 path だと parse_args で refuse

## 安全要件 (= 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信                                  | 0 通 ✓ |
| LineBotClient / linebot SDK 呼出              | 0 ✓ (= helper 不 import) |
| token / channel_token / secret 参照            | 0 ✓ |
| .env / ~/.fire_secrets 読み込み                | 0 ✓ |
| production DB write                            | 0 ✓ (= mtime unchanged) |
| develop DB write                               | 0 ✓ (= mtime unchanged) |
| staging DB write                               | **1 snapshot + 5 rows + 5 decisions seed** ★ (= 想定通り) |
| 楽天証券操作 / 自動発注 / Computer Use         | 0 ✓ |
| .github/workflows/ 変更                       | 0 ✓ |
| --no-verify                                   | 不使用 ✓ |
| scripts/seed_pattern_layer1.py                | 未接触 (mtime May 7 17:39) ✓ |
| simulation/research_lane/historical_indicators.py | 未接触 (mtime May 9 21:10) ✓ |
| TODO Excel                                    | 未更新 ✓ |
| cron / launchd / crontab 本番登録              | 0 ✓ |

## 未承認項目 (= 引き続き禁止)

- W4.1-B F062 経由 staging smoke (= LINE 1 通送信あり)
- token / channel_token 実使用
- production / develop DB write
- cron / launchd / crontab 本番登録

## 注意: F282 weekly snapshot による消失リスク

- 2026-05-18 月曜 07:00 JST に F282 weekly staging snapshot が走り、
  staging を production fire.db で **上書き** する設計
- 今回作成した advisory_snapshots / advisory_snapshot_rows table と
  5 件分の seed は **5/18 で消える**
- 次回 staging に同じ snapshot を入れたい場合は本 helper で再 ingest 可
- FIRE-OPS-R0 案 1 (= production write 統一) が完成すれば恒久対策

## HQ 判断が必要な論点 (= 3 件)

1. **W4.1-A 完了 → 進行可否確認** (推奨: approve)
2. **W4.1-B F062 経由 staging smoke 着手判断**:
   - W4-1 F062 send_smoke + --record-decisions 統合経路で
     1 通 LINE + snapshot 保存を smoke
   - 必要承認: LINE 1 通送信 (= token 実使用) + 初回 DDL は既に W4.1-A
     で完了済のため不要
   - 推奨: 単独 approve、ただし F062-R5.8 既送信と同一 payload を
     再送信するため Fujiwara LINE app で「重複通知」を許容する確認が
     必要
3. **F286-PNL-R2 系の本格運用着手判断**:
   - W4.1-A で技術検証完了
   - 次は F062 経由運用 (= 毎回 Advisory 送信 + snapshot 自動保存) の
     継続運用判断

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE4_results|Wave 4 results]]
- [[F286_PNL_R2_advisory_snapshot_auto_ingest|F286-PNL-R2 module]]
- [[FIRE_LABEL_R1_advisory_label_refresh|FIRE-LABEL-R1 新ラベル]]
- [[../log]]
