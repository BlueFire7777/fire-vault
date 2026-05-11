---
id: FIRE-CODEX-R1-WAVE9-plan
phase: ガバナンス / Codex 並列実装 Wave 9 起票 / R-01-08 整合
priority: 最優先
status: 起票 ☆ W9-1 を最優先実行、W9-2 / W9-3 は別ターン起票 (2026-05-12)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 8 (= 完了)
  - HQ Wave 9 approve (= 2026-05-12、W8-5 TOCTOU 限定受容 + W9-1 staging
    smoke 条件付き承認 + REPORT-R1 weekly/monthly + sub-D2.3.x 起票承認)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 9: W9-1 staging UPDATE smoke 実行 + W9-2 + W9-3 起票

最終更新: 2026-05-12

## ★ 状態: 起票 (= W9-1 を最優先、結果次第で W9-2/W9-3 を別ターン)

HQ Wave 8 approve 受領後、W9-1 PNL-R3 staging UPDATE smoke を **条件付き
承認**として実行。W8-5 HIGH #1 residual TOCTOU は **限定受容範囲内** で
許容。staging DB のみ、production/develop touch 禁止。

W9-1 完了後 HQ 1 ブロック報告、HQ レスポンス待ちで W9-2/W9-3 着手判断。

## Wave 9 構成

| sub | task | 状態 |
|-----|------|------|
| W9-1 | PNL-R3 staging UPDATE smoke 実行 (= W7-1 plan を実行) | 着手 ← 本ターン |
| W9-2 | REPORT-R1 weekly / monthly impl | HQ 承認済、別ターン起票 |
| W9-3 | DATA-R3 sub-D2.3.x 起票 (= 4 sub 個別 plan) | 起票のみ承認、別ターン |

## W9-1 sub-task 構成 (= 本ターン)

| sub | lane | task | 成果物 |
|-----|------|------|--------|
| W9-1a | L3+L2 (Codex) | seed runner impl + tests | scripts/jobs/run_f286_pnl_r3_seed_staging_smoke.py |
| W9-1b | L4 (Codex) | seed runner audit | audit report |
| W9-1c | 本線 | staging UPDATE smoke 実行 (7 step) | HQ 報告 |

## W8-5 TOCTOU 限定受容範囲 (= HQ 明示)

W9-1 は以下 6 条件を **すべて満たす場合のみ** documented risk 受容下で実行可:
1. staging smoke のみ
2. Fujiwara 専用 Mac mini 上の単独実行のみ
3. HQ 明示承認あり (= 本起票後、smoke 直前に再度仰がない、本起票内で完結)
4. production/develop DB 対象外
5. cron/自動実行対象外
6. LINE/token を伴う処理対象外

→ 全 6 条件を満たす。W9-1c 実行で違反させない。

## W9-1 実行条件 (= HQ 明示)

| 項目 | 条件 |
|---|---|
| 案 | b: INSERT 5 row + rollback |
| 対象 DB | staging のみ (= fire.staging.db) |
| production/develop write | 禁止 |
| pre-record | production/develop/staging DB mtime 記録 |
| seed | 5 row INSERT |
| compute | paper_pnl 実行 |
| 既存 row 検証 | hash 一致確認 |
| UPDATE 列 | seed row の paper_pnl + updated_at のみ |
| 既存 column 不触 | fujiwara_decision / actual_trade / notes / created_at touch なし |
| rollback | seed row 全 DELETE |
| final 検証 | rollback 後 pre 状態完全一致 |
| LINE | 0 |
| token / secret / channel_token | 0 |
| cron | 0 |
| 報告 | 実行後 1 ブロック HQ 報告 |

## W9-1a seed runner 仕様

### 新規 runner: `scripts/jobs/run_f286_pnl_r3_seed_staging_smoke.py`

```
positional args:
  なし

required flags:
  --db-label staging                  (= staging のみ)
  --db-path <path>                    (= basename='fire.staging.db' 必須)
  --output-json <path>                (= seed 結果記録、必須)

optional flags:
  --completion-report <path>          (= Markdown 完了報告)
  --base-date YYYY-MM-DD              (= デフォルト today、過去日推奨)
  --seed-prefix staging-smoke-YYYY-MM-DD-w9-1
                                      (= advisory_id prefix)
  --rollback                          (= seed prefix row を DELETE する mode)
```

### 六段ガード再利用 (= W4-2 / W6-1+2 / W8-1 同 pattern)

1. SnapshotStore コンストラクタ read_only=False 強制
2. --db-label staging 必須 (= production / develop は refuse)
3. --db-path basename='fire.staging.db' 必須
4. --output-json / --completion-report が DB/WAL/SHM/sqlite path 不可
5. --db-path symlink refuse + Path.resolve() basename 一致
6. --db-path inode が forbidden DB と一致 (= hardlink) 不可
   + W8-1 で導入した os.open(O_NOFOLLOW) + fstat + post-connect verify

### INSERT 内容 (= 5 row 固定、idempotent)

```python
SEED_ROWS = [
    {
        "advisory_id": f"{seed_prefix}-001",
        "code": "7203",
        "decision_label": "積極的買い推奨",
    },
    {
        "advisory_id": f"{seed_prefix}-002",
        "code": "9984",
        "decision_label": "条件付き買い推奨",
    },
    {
        "advisory_id": f"{seed_prefix}-003",
        "code": "6758",
        "decision_label": "注意つき買い候補",
    },
    {
        "advisory_id": f"{seed_prefix}-004",
        "code": "7203",
        "decision_label": "場中監視",
    },
    {
        "advisory_id": f"{seed_prefix}-005",
        "code": "7203",
        "decision_label": "見送り推奨",
    },
]
```

SQL:
```sql
INSERT OR IGNORE INTO advisory_decisions (
    advisory_id, code, base_date, decision_label,
    fujiwara_decision, actual_trade, notes,
    created_at, updated_at,
    paper_pnl, paper_reason
) VALUES (?, ?, ?, ?, NULL, NULL, ?, ?, ?, NULL, NULL)
```

- `INSERT OR IGNORE`: 既存 advisory_id (= 同 seed-prefix で再実行) で skip
- 既存 row (= W4.1-A 5 row + 任意 others) は **完全不触**

### rollback (= --rollback flag)

```sql
DELETE FROM advisory_decisions
WHERE advisory_id LIKE :seed_prefix || '%'
```

- 同 seed-prefix の row のみ DELETE
- DELETE 件数を output JSON / completion report に記録
- 既存 row は完全不触

### 安全制約

- INSERT 限定 (= UPDATE / DROP / ALTER 等不発行)
- staging-only 厳守
- LINE 不 import / 楽天 不 import / Computer Use 不使用
- token / secret 不参照
- subprocess 起動なし

### 期待 tests (= W9-1a Codex 実装、15-25 件追加見込)

- TestSeedFiveRowsHappyPath: 5 row INSERT 成功
- TestSeedIdempotentReRun: 2 回連続実行で row 数変わらず
- TestSeedDoesNotTouchExistingRows: 既存 row hash 不変
- TestSeedRequiresStagingLabel: production / develop で refuse
- TestSeedBasenameRefuse: 異 basename で refuse
- TestSeedSymlinkRefuse: symlink で refuse
- TestSeedHardlinkRefuse: forbidden DB hardlink で refuse
- TestRollbackDeletesOnlyPrefix: rollback で seed prefix の row のみ削除
- TestRollbackIdempotent: 2 回連続 rollback で safe
- TestRollbackPreservesOtherRows: rollback 後既存 row 不変
- TestOutputJsonStructure: seed_count / skip_count / advisories list
- TestAtomicCreateXMode: output_json の atomic create
- TestSymlinkOutputPathRefuse: dangling symlink で refuse
- TestForbiddenImports: linebot / requests / aiohttp 不 import
- TestSubprocessNotInvoked: subprocess.run 不呼出

## W9-1b audit 観点

- 六段ガード適用確認
- INSERT 限定 / DELETE 限定 (= UPDATE / DROP / ALTER 不発行)
- rollback の prefix LIKE が SQL injection 防止
- 既存 row 不触の SQL レベル保証
- atomic create 'x' mode + W8-1 inode verify 再利用
- forbidden import 不在 (= linebot / requests / aiohttp / 楽天 / Playwright)
- subprocess 起動 0
- token / secret 参照 0
- F286-PNL-R3 既存 contract (= W8 で実装した module) を破壊しない

## W9-1c 実行手順 (= 7 step、本線 Integrator)

### Step 1: pre-smoke 状態キャプチャ

```bash
# DB mtime 記録 (= production / develop / staging)
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db > /tmp/w9_1_pre_mtimes.txt

# staging row 数
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT count(*) FROM advisory_decisions;" \
    > /tmp/w9_1_pre_rows.txt

# 既存 row の SQL-level snapshot (= rollback 後 diff 検証用)
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT advisory_id, code, base_date, decision_label,
            fujiwara_decision, actual_trade, notes,
            created_at, updated_at, paper_pnl, paper_reason
     FROM advisory_decisions ORDER BY advisory_id;" \
    > /tmp/w9_1_pre_snapshot.txt
sha256sum /tmp/w9_1_pre_snapshot.txt > /tmp/w9_1_pre_hash.txt
```

### Step 2: seed 実行

```bash
.venv/bin/python -m scripts.jobs.run_f286_pnl_r3_seed_staging_smoke \
    --db-label staging \
    --db-path ~/fire/data/fire.staging.db \
    --base-date 2026-05-08 \
    --seed-prefix staging-smoke-2026-05-12-w9-1 \
    --output-json /tmp/w9_1_seed_result.json \
    --completion-report /tmp/w9_1_seed_report.md
```

期待: exit 0、seed_count=5、skip_count=0。

### Step 3: paper_pnl compute 実行 (= W8-1 既存 runner)

```bash
.venv/bin/python -m scripts.jobs.run_f286_pnl_r3_compute_paper_pnl \
    --db-label staging \
    --db-path ~/fire/data/fire.staging.db \
    --advisory-id-prefix staging-smoke-2026-05-12-w9-1 \
    --base-date 2026-05-08 \
    --allowable-loss-amount 5000 \
    --assumed-stop-pct 0.02 \
    --output-json /tmp/w9_1_compute_result.json \
    --completion-report /tmp/w9_1_compute_report.md \
    --write
```

期待:
- exit 0
- candidates 3 (= 積極/条件/注意)
- audit_count_unmatched 0
- computed 3 (= market_prices_daily 有無で paper_pnl が数値 or None)
- updated は computed 件数のみ

### Step 4: 既存 row hash 一致確認 (= 既存 row 不触検証)

```bash
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT advisory_id, code, base_date, decision_label,
            fujiwara_decision, actual_trade, notes,
            created_at, updated_at, paper_pnl, paper_reason
     FROM advisory_decisions
     WHERE advisory_id NOT LIKE 'staging-smoke-2026-05-12-w9-1-%'
     ORDER BY advisory_id;" \
    > /tmp/w9_1_post_existing.txt
sha256sum /tmp/w9_1_post_existing.txt > /tmp/w9_1_post_existing_hash.txt
diff /tmp/w9_1_pre_snapshot.txt /tmp/w9_1_post_existing.txt
```

期待: diff 0 行 (= 既存 row は完全不触)。

### Step 5: seed row UPDATE 列限定確認

```sql
SELECT advisory_id, decision_label, paper_pnl, paper_reason,
       created_at, updated_at,
       fujiwara_decision, actual_trade, notes
FROM advisory_decisions
WHERE advisory_id LIKE 'staging-smoke-2026-05-12-w9-1-%'
ORDER BY advisory_id;
```

期待:
- 計算対象 3 row (= 積極/条件/注意): paper_pnl は数値 or None、updated_at は
  compute 時刻、fujiwara_decision / actual_trade / notes は NULL のまま
- 除外 2 row (= 場中監視/見送り): paper_pnl=None、updated_at は seed 時刻
  のまま (= compute で touch されない、include label 外)

### Step 6: rollback

```bash
.venv/bin/python -m scripts.jobs.run_f286_pnl_r3_seed_staging_smoke \
    --db-label staging \
    --db-path ~/fire/data/fire.staging.db \
    --seed-prefix staging-smoke-2026-05-12-w9-1 \
    --rollback \
    --output-json /tmp/w9_1_rollback_result.json
```

期待: exit 0、delete_count=5。

### Step 7: final 検証

```bash
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT * FROM advisory_decisions ORDER BY advisory_id;" \
    > /tmp/w9_1_final_snapshot.txt
diff /tmp/w9_1_pre_snapshot.txt /tmp/w9_1_final_snapshot.txt

# staging row 数
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT count(*) FROM advisory_decisions;" \
    > /tmp/w9_1_final_rows.txt
diff /tmp/w9_1_pre_rows.txt /tmp/w9_1_final_rows.txt

# DB mtime
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db \
    ~/fire/data/fire.staging.db > /tmp/w9_1_final_mtimes.txt
```

期待:
- pre vs final snapshot diff 0 行 (= 完全一致)
- pre vs final row 数 一致
- production / develop DB mtime 不変 (= staging のみ mtime が変わる可能性)

### Step 8: HQ 報告 (= 1 ブロック)

`/tmp/w9_1_hq_report.md` に Markdown で:
- 各 step exit code
- seed_count / compute_count / rollback_delete_count
- 既存 row hash diff (= 期待値「0 行」)
- final snapshot diff (= 期待値「0 行」)
- DB mtime production / develop / staging
- LINE 送信 0 / token 参照 0 / external API 0
- 結論: 成功 / 失敗

## 受入基準

- [ ] W9-1a seed runner 実装 + tests 全 PASS
- [ ] W9-1b audit CRITICAL 0
- [ ] W9-1c 7 step 全完了 + diff 0
- [ ] HQ 1 ブロック報告

## Wave 9 後続 (= W9-1 完了後別ターン)

- **W9-2** (= REPORT-R1 weekly / monthly impl、HQ 承認済):
  - W8-3 daily を延長、weekly / monthly aggregators + report + runner
  - read-only / dry-run / tests、LINE 送信なし / cron 登録なし
- **W9-3** (= DATA-R3 sub-D2.3.x 起票、HQ 起票のみ承認):
  - 4 sub 個別 plan: sub-D2.3.f100 / .f101 / .f111 / .f119
  - 各 sub の smoke plan を vault に書く
  - **実 write は個別 HQ approve 必要**

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE8_results|Wave 8 results]]
- [[../03_design/F286_PNL_R3_staging_update_smoke_plan_2026-05-11|W7-1 staging UPDATE smoke plan]]
- [[../07_incidents/F286_PNL_R3_audit_2026-05-11|W7-3 audit]]
- [[../log]]
