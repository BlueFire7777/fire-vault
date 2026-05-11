---
id: F286-PNL-R3-staging-update-smoke-plan
phase: ガバナンス / Wave 7 W7-1 / staging UPDATE smoke 計画
priority: 最優先
status: 起票 ☆ 実行 unapproved (= HQ 別 approve 必須)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R3 (= W6-1+2 完了、compute_paper_pnl module + runner)
  - W4.1-A (= staging 5 row 既存、decision_label='場中監視')
  - HQ Wave 7 W7-1 起票 approve (= 2026-05-11)
chapter: ガバナンス / R-01-08 / Codex 運用拡張 / F286 PNL シリーズ
---

# F286-PNL-R3 Staging UPDATE Smoke Plan (= Wave 7 W7-1)

最終更新: 2026-05-11

## ★ 状態: 起票 (= 実行 unapproved、HQ 明示承認後 Wave 8 W8-1 で実行)

W6-1+2 で実装した `pnl/paper_pnl.py` + `run_f286_pnl_r3_compute_paper_pnl.py`
を **staging DB で初回 UPDATE smoke** する計画。HQ 制約により本書は **plan
のみ**、実行は別 HQ approve 後。

## ⚠️ W7-3 audit prerequisite (= 2026-05-11 追記)

W7-3 PNL-R3 audit (= Codex L4) で **HIGH #1 (TOCTOU race)** が検出された:

- 該当: `run_f286_pnl_r3_compute_paper_pnl.py` の `--db-path` 検査が
  `parse_args()` で実行されるが、`_connect_write()` での write open までに
  symlink / hardlink 差し替えできる窓がある
- 影響: staging-only write guard の保証が弱まる (= adversarial 前提)
- merge_recommendation: **「Wave 8 W8-1 staging UPDATE smoke 着手 NG」**

→ **W8-1 着手前に Wave 8 W8-0-fix で以下を実装**:
1. `_connect_write()` 内で `_assert_db_path_not_symlink_attack()` を再実行
2. write transaction 直前に再 stat (= st_dev/st_ino 検証)
3. 可能なら `os.open(O_NOFOLLOW)` 相当で fd 取得 → fd の st_dev/st_ino を
   forbidden DB と比較

詳細: [[../07_incidents/F286_PNL_R3_audit_2026-05-11|F286-PNL-R3 audit
report]] HIGH #1 参照。

加えて W7-3 audit MEDIUM #1/#2 も Wave 8 W8-0-fix 内で対応推奨:
- MEDIUM #1: output path も write 直前に再検査 / atomic create (`'x'` mode)
- MEDIUM #2: decision_label を `strip()` + NFKC normalize、もしくは smoke で
  silent skip 残件 audit query で確認

## HQ 制約 (= 2026-05-11 Wave 6 approve 受領時)

1. W4.1-A の staging rows (5 件) は `decision_label='場中監視'` で paper_pnl
   **対象外** → paper_pnl 対象ラベル行をどう用意するかを **本 plan に明記必須**
2. UPDATE 対象は **`paper_pnl` + `updated_at` のみ**
3. **既存 row 不触**: `fujiwara_decision` / `actual_trade` / `notes` /
   `created_at` / `entry_price` / `exit_price` / `qty` / 等を **touch 禁止**
4. 実行前に **HQ への明示承認** が必要

## 1. 対象ラベル行用意 (= 3 案比較)

### 案 a: 既存 W4.1-A 5 row の decision_label を一時的に「積極的買い推奨」に書換

```sql
-- smoke 前
UPDATE advisory_decisions
SET decision_label = '積極的買い推奨'
WHERE advisory_id LIKE 'production-advisory-2026-05-%-場中監視-%';

-- smoke 後 rollback
UPDATE advisory_decisions
SET decision_label = '場中監視'
WHERE advisory_id LIKE 'production-advisory-2026-05-%-場中監視-%';
```

**判定: ❌ 却下**
- 既存 row の decision_label を UPDATE する = HQ 制約 #3「既存 row 不触」に **明確に反する**
- rollback で戻すとしても、smoke 中の transient 状態が「既存 row 改変」を意味する
- updated_at も書き換わってしまう

### 案 b: 新規 advisory_decisions row を seed (= INSERT only、推奨)

```sql
-- smoke seed (= 新規 row INSERT、既存 5 row は完全不触)
INSERT INTO advisory_decisions (
    advisory_id, code, base_date, decision_label,
    fujiwara_decision, actual_trade, notes,
    created_at, updated_at,
    paper_pnl, paper_reason
) VALUES
    ('staging-smoke-2026-05-11-w7-1-001', '7203', '2026-05-08', '積極的買い推奨',
     NULL, NULL, 'W7-1 smoke seed',
     '2026-05-11T22:00:00+09:00', '2026-05-11T22:00:00+09:00',
     NULL, NULL),
    ('staging-smoke-2026-05-11-w7-1-002', '9984', '2026-05-08', '条件付き買い推奨',
     NULL, NULL, 'W7-1 smoke seed',
     '2026-05-11T22:00:00+09:00', '2026-05-11T22:00:00+09:00',
     NULL, NULL),
    ('staging-smoke-2026-05-11-w7-1-003', '6758', '2026-05-08', '注意つき買い候補',
     NULL, NULL, 'W7-1 smoke seed',
     '2026-05-11T22:00:00+09:00', '2026-05-11T22:00:00+09:00',
     NULL, NULL),
    ('staging-smoke-2026-05-11-w7-1-004', '7203', '2026-05-08', '場中監視',
     NULL, NULL, 'W7-1 smoke seed (= exclude label control)',
     '2026-05-11T22:00:00+09:00', '2026-05-11T22:00:00+09:00',
     NULL, NULL),
    ('staging-smoke-2026-05-11-w7-1-005', '7203', '2026-05-08', '見送り推奨',
     NULL, NULL, 'W7-1 smoke seed (= exclude label control)',
     '2026-05-11T22:00:00+09:00', '2026-05-11T22:00:00+09:00',
     NULL, NULL);

-- smoke 後 rollback (= seed した 5 row のみ DELETE、既存 5 row は不触)
DELETE FROM advisory_decisions
WHERE advisory_id LIKE 'staging-smoke-2026-05-11-w7-1-%';
```

**判定: ✅ 推奨 (= 主案)**
- 既存 5 row は完全不触
- 新規 5 row で全 decision_label を網羅 (= 3 計算対象 + 2 除外)
- rollback は DELETE のみで完結
- lineage 明確 (= advisory_id に "staging-smoke-2026-05-11-w7-1-" prefix)
- market_prices_daily に 7203 / 9984 / 6758 が無い可能性 → 計算結果が
  paper_reason='market_data_missing: ...' になる可能性、これは smoke 想定内

### 案 c: production / develop の advisory_decisions から「積極的買い推奨」row を選び staging に複製

```sql
-- production の積極的買い推奨 row を staging に複製
INSERT INTO advisory_decisions
SELECT * FROM <production-attached>.advisory_decisions
WHERE decision_label = '積極的買い推奨' AND base_date >= '2026-05-01'
LIMIT 5;
```

**判定: ⚠️ 採用しない (= 主案では却下、補助案として将来検討)**
- 本番に近いデータで smoke できる利点はある
- ただし staging に production data を引き寄せる ATTACH 経路は F282 環境分離
  ポリシーで明示的に禁止していないが、運用上のリスク (= 誤って production
  に書き戻し / WAL ハンドル衝突) が高い
- W7-1 smoke では 案 b を採用、必要であれば Wave 8+ で別 plan

## 2. 推奨案 (= 案 b) seed runner 仕様

### 新規 runner: `scripts/jobs/run_f286_pnl_r3_seed_staging_smoke.py`

(= W7-1 plan 起票時点では **実装しない**、Wave 8 W8-1 着手時に実装)

```
positional args:
  なし

required flags:
  --db-label staging                  (= W4-2 / W6-1+2 と同じ六段ガード)
  --db-path <path>                    (= basename='fire.staging.db' 必須)
  --output-json <path>                (= seed 結果記録)

optional flags:
  --completion-report <path>          (= Markdown 完了報告)
  --base-date 2026-05-08              (= デフォルト today、過去日推奨)
  --seed-prefix staging-smoke-2026-05-11-w7-1
                                      (= advisory_id prefix)
  --rollback                          (= 同 prefix の row を DELETE する mode)

ガード再利用:
  - SnapshotStore コンストラクタ read_only=False 強制 (= 既存)
  - --db-label staging 必須
  - --db-path basename='fire.staging.db' 必須
  - --output-json / --completion-report が DB/WAL/SHM/sqlite path 不可
  - --db-path symlink refuse + Path.resolve() basename 一致
  - --db-path inode が forbidden DB と一致 (= hardlink) 不可

INSERT 内容 (= 5 row 固定):
  ('staging-smoke-...-001', '7203', base_date, '積極的買い推奨', ...)
  ('staging-smoke-...-002', '9984', base_date, '条件付き買い推奨', ...)
  ('staging-smoke-...-003', '6758', base_date, '注意つき買い候補', ...)
  ('staging-smoke-...-004', '7203', base_date, '場中監視', ...)
  ('staging-smoke-...-005', '7203', base_date, '見送り推奨', ...)

idempotent:
  - 既存 advisory_id (= 同 seed-prefix) があれば skip (= INSERT OR IGNORE)
  - rerun 時は seed 件数 0 / skip 件数 5 のレポート

rollback:
  - --rollback flag で DELETE FROM advisory_decisions
    WHERE advisory_id LIKE :seed_prefix || '%'
  - DELETE 件数を output JSON に記録
```

## 3. smoke 実行手順 (= Wave 8 W8-1 で実行)

### Step 1: pre-smoke 状態キャプチャ

```bash
# staging DB の現在の row 数
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT count(*) FROM advisory_decisions;"
# → 期待値: 5 (= W4.1-A 5 row)

# staging DB の mtime
ls -la ~/fire/data/fire.staging.db

# 既存 5 row の hash (= diff 検証用)
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT advisory_id, code, base_date, decision_label,
            fujiwara_decision, actual_trade, notes,
            created_at, updated_at, paper_pnl, paper_reason
     FROM advisory_decisions ORDER BY advisory_id;" \
    > /tmp/w8_1_pre_smoke_rows.txt
sha256sum /tmp/w8_1_pre_smoke_rows.txt
```

### Step 2: seed 実行 (= 新規 5 row INSERT)

```bash
.venv/bin/python -m scripts.jobs.run_f286_pnl_r3_seed_staging_smoke \
    --db-label staging \
    --db-path ~/fire/data/fire.staging.db \
    --base-date 2026-05-08 \
    --seed-prefix staging-smoke-2026-05-11-w7-1 \
    --output-json /tmp/w8_1_seed_result.json \
    --completion-report /tmp/w8_1_seed_report.md
```

期待結果:
- exit 0
- seed 件数 5 / skip 件数 0
- staging DB row 数 5 → 10
- 既存 5 row mtime / hash unchanged

### Step 3: paper_pnl compute 実行 (= W6-1+2 runner)

```bash
.venv/bin/python -m scripts.jobs.run_f286_pnl_r3_compute_paper_pnl \
    --db-label staging \
    --db-path ~/fire/data/fire.staging.db \
    --advisory-id-prefix staging-smoke-2026-05-11-w7-1 \
    --base-date 2026-05-08 \
    --allowable-loss-amount 5000 \
    --assumed-stop-pct 0.02 \
    --output-json /tmp/w8_1_compute_result.json \
    --completion-report /tmp/w8_1_compute_report.md
```

期待結果:
- exit 0
- compute 対象 3 row (= 積極的買い推奨 / 条件付き買い推奨 / 注意つき買い候補)
- exclude 2 row (= 場中監視 / 見送り推奨、paper_pnl=None + paper_reason='label_excluded')
- 各 row の paper_pnl が None または数値 (= market_prices_daily 有無で変動)

### Step 4: post-smoke 検証 (= UPDATE 範囲確認 + 既存 row 不触確認)

```bash
# 既存 5 row が一切変更されていないことを確認
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT advisory_id, code, base_date, decision_label,
            fujiwara_decision, actual_trade, notes,
            created_at, updated_at, paper_pnl, paper_reason
     FROM advisory_decisions
     WHERE advisory_id NOT LIKE 'staging-smoke-2026-05-11-w7-1-%'
     ORDER BY advisory_id;" \
    > /tmp/w8_1_post_smoke_existing_rows.txt
sha256sum /tmp/w8_1_post_smoke_existing_rows.txt

# pre と post で sha256 一致確認
diff /tmp/w8_1_pre_smoke_rows.txt \
    <(sqlite3 ~/fire/data/fire.staging.db \
        "SELECT * FROM advisory_decisions
         WHERE advisory_id NOT LIKE 'staging-smoke-2026-05-11-w7-1-%'
         ORDER BY advisory_id;")
# → 期待値: diff 0 行 (= 完全一致)
```

### Step 5: 新規 5 row の paper_pnl / updated_at 確認

```sql
SELECT advisory_id, decision_label, paper_pnl, paper_reason,
       created_at, updated_at
FROM advisory_decisions
WHERE advisory_id LIKE 'staging-smoke-2026-05-11-w7-1-%'
ORDER BY advisory_id;
```

期待結果:
- 全 5 row の `created_at` は seed 時刻 (Step 2)
- 計算対象 3 row の `updated_at` は compute 時刻 (Step 3) (= seed 時刻と差がある)
- exclude 2 row の `updated_at` は seed 時刻のまま (= compute で touch されない)

注意: 案 b の現状仕様では exclude row も runner で `paper_pnl=None +
paper_reason='label_excluded'` を **UPDATE する** か **しない** かは設計
判断。W6-1+2 の仕様では WHERE `paper_pnl IS NULL` でフィルタするため、
exclude row も初回は対象になる。**smoke で確認**:
- exclude row の updated_at が touch されるなら、UPDATE 範囲が `paper_pnl
  + updated_at` のみであることを diff で再確認
- updated_at は touch されるが、それ以外の column は不触であれば OK

### Step 6: rollback (= seed 5 row を DELETE)

```bash
.venv/bin/python -m scripts.jobs.run_f286_pnl_r3_seed_staging_smoke \
    --db-label staging \
    --db-path ~/fire/data/fire.staging.db \
    --seed-prefix staging-smoke-2026-05-11-w7-1 \
    --rollback \
    --output-json /tmp/w8_1_rollback_result.json
```

期待結果:
- exit 0
- DELETE 件数 5
- staging DB row 数 10 → 5
- 既存 5 row mtime / hash unchanged

### Step 7: final 検証

```bash
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT count(*) FROM advisory_decisions;"
# → 期待値: 5

sqlite3 ~/fire/data/fire.staging.db \
    "SELECT * FROM advisory_decisions ORDER BY advisory_id;" \
    > /tmp/w8_1_final_rows.txt
diff /tmp/w8_1_pre_smoke_rows.txt /tmp/w8_1_final_rows.txt
# → 期待値: diff 0 行
```

## 4. 受容判定 (= smoke 成功条件)

| 項目 | 期待 |
|---|---|
| seed exit code | 0 |
| seed 件数 | 5 |
| compute exit code | 0 |
| compute 対象 row 数 | 3 (= 積極/条件/注意) |
| compute exclude row 数 | 2 (= 場中/見送り) |
| 既存 5 row の SQL-level hash | pre / post / final で完全一致 |
| 既存 5 row の mtime 不変性 | DB ファイル mtime は当然変わる、SQL-level diff 0 |
| compute 後 updated_at 変化 | 計算対象 3 row のみ更新、UPDATE 列は paper_pnl + updated_at のみ |
| paper_pnl 値 | 数値 or None (= market data 有無で変動、両方 OK) |
| paper_reason | 解釈可能なメッセージ (= 'ok' / 'label_excluded' / 'market_data_missing: ...') |
| rollback exit code | 0 |
| rollback DELETE 件数 | 5 |
| final row 数 | 5 |
| final diff | pre と完全一致 |

## 5. 実行前 HQ approve template

Wave 8 W8-1 着手前に HQ に以下 1 block で承認を仰ぐ。

```
=== HQ 承認依頼 / Wave 8 W8-1 PNL-R3 staging UPDATE smoke 実行 ===

date:                 (実行予定日)
parent_task:          F286-PNL-R3-staging-update-smoke (= W7-1 plan を実行)
db_label:             staging (= ~/fire/data/fire.staging.db)
seed_runner:          scripts/jobs/run_f286_pnl_r3_seed_staging_smoke
compute_runner:       scripts/jobs/run_f286_pnl_r3_compute_paper_pnl
                      (= W6-1+2 既存)

予定 INSERT 件数:    5 (= 案 b seed)
予定 UPDATE 件数:    0-5 (= compute 結果次第、最大 5)
予定 UPDATE 列:      paper_pnl + updated_at のみ
予定 DELETE 件数:    5 (= rollback)

既存 5 row の touch: 0 (= SQL-level hash で diff 0 検証)
production DB:       touch なし
develop DB:          touch なし
LINE 送信:           なし

rollback 計画:        --rollback で seed 5 row 全削除、final で pre と完全一致
失敗時対応:           sub-step ごとに output_json で検証、失敗時は即停止
                      + 手動 SQL で DELETE 補完
所要時間 (estimate): 5-10 分

承認希望: 上記 W8-1 staging UPDATE smoke の実行
```

## 6. リスク評価

| risk | 影響 | 対策 |
|---|---|---|
| market_prices_daily に 7203 / 9984 / 6758 の 2026-05 データなし | compute 全 None | 想定内、paper_reason で判別 |
| seed runner が 既存 row を誤って INSERT OR REPLACE する | 既存 row 改変 | INSERT OR IGNORE 厳守、test で UPSERT 拒否 確認 |
| compute runner の WHERE filter ミスで既存 5 row も対象になる | 既存 row UPDATE | --advisory-id-prefix で prefix 一致のみ厳守 |
| F282 weekly snapshot (Monday 07:00 JST) が smoke 中に走る | staging 上書き | smoke 開始時に F282 disable 確認、終了後 enable |
| rollback 失敗で seed row が staging に残留 | smoke 後 row 数 10 のまま | --rollback で DELETE、手動 SQL fallback |
| 別途 W7-2 / 並走タスクが staging に並列書込 | race condition | smoke 中は他 staging 書込タスク停止 |

## 7. 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE7_plan|Wave 7 plan]]
- [[F286_PNL_R3_paper_pnl_simulator_hook_2026-05-11|F286-PNL-R3 design]]
- [[../02_todo/F286_PNL_R3_paper_pnl_simulator_hook|F286-PNL-R3 完了マーカー]]
- HQ 制約 source: FIRE-CODEX-R1 v1.1 Wave 6 approve (= 2026-05-11)
