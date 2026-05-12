---
id: F286-DATA-R3-sub-D2.3.f111-preflight
phase: Wave 16 W16-3 / f111 preflight + final smoke plan
priority: 高
status: 起票 ☆ 実 staging write は W17 別 HQ approve
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W10-5 f111 smoke plan / W11-3 f111 HQ approve worksheet
  - W14-3 SCHEMA-R1 完了 (= advisory_decisions 全 環境同期、PK 衝突回避)
  - HQ Wave 16 W16-3 承認 (= 2026-05-12)
chapter: F286 DATA / R-01-08 / preflight
---

# F286-DATA-R3 sub-D2.3.f111 Preflight + Final Smoke Plan (= W16-3)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実 staging write は W17 別 HQ 明示承認後)

## HQ 必須 8 項目

### 1. target table

- `advisory_decisions` (= staging DB のみ)
- + 関連 (= signals 等、実 runner で確認)

★ **W14-3 SCHEMA-R1 統合派完了で全 環境に存在**、本 plan は staging 限定。

### 2. expected API call

- 外部 API なし (= Feature Store 内部処理、Pattern Research)
- 既存 features / 既存 advisory_decisions の SELECT のみ

### 3. expected inserted / updated row count

- INSERT: Pattern Research 結果次第 (= 0 〜 数十件)
- UPDATE: 0 (= INSERT only、INSERT OR IGNORE で衝突 skip)
- DELETE: 0

### 4. rollback 可否

可能、prefix 限定 DELETE。

```sql
DELETE FROM advisory_decisions
WHERE advisory_id LIKE 'f111-smoke-2026-05-12-%';
```

★ prefix `f111-smoke-2026-05-12-` を strict regex で validate。
W4.1-A (= f062-r5.8-... / production-advisory-...) や W9-1 (= staging-smoke-...)
と **PK 衝突しない** ことを smoke 前 SELECT で事前確認。

### 5. pre/post mtime 確認

```bash
ls -la ~/fire/data/fire.staging.db
# pre / post で size 増加
# production / develop mtime unchanged enforce
```

### 6. existing row 不触確認方法

```bash
# pre snapshot (= W14-3 で全 環境同期完了、staging に既存 10 row + W9-1
# smoke 後 0 row)
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT advisory_id, code, base_date, decision_label, \
            fujiwara_decision, actual_trade, paper_pnl, paper_reason \
     FROM advisory_decisions \
     WHERE advisory_id NOT LIKE 'f111-smoke-2026-05-12-%' \
     ORDER BY advisory_id;" \
    > /tmp/w17_X_f111/pre_existing.txt
sha256sum /tmp/w17_X_f111/pre_existing.txt

# post-smoke 同 SELECT、diff 0 行期待
```

### 7. failure 時の停止条件

| 条件 | 対応 |
|------|------|
| production / develop DB mtime 変化 | **即停止** + rollback |
| PK 衝突検出 (= INSERT OR IGNORE 想定外 skip) | smoke 中断、調査 |
| advisory_decisions 以外への INSERT (= 想定外 table) | **即停止** |
| Pattern Research crash | smoke 中断 |
| LINE 送信検出 | **即停止** |
| subprocess 起動検出 (= 外部実行) | **即停止** |
| token 値露出 | **即停止** |

### 8. HQ approve template

```
=== HQ 承認依頼 / W17-3 f111 staging write 実行 ===

date:                 (実行予定日)
plan source:          F286-DATA-R3-sub-D2.3.f111-preflight (W16-3)
runner:               scripts/jobs/run_research_watchlist_signal_persistence.py
target_db:            staging
target_table:         advisory_decisions
advisory_id prefix:   f111-smoke-2026-05-12-* (= strict、衝突回避済)
予定 API call:        0 (= 内部処理のみ)
予定 INSERT row 数:    Pattern Research 結果次第 (= 0 〜 数十)
予定 UPDATE row 数:    0
予定 production write: 0
予定 develop write:    0
予定 LINE 送信:        0
予定 secret 値露出:    0
PK 衝突回避:           W4.1-A / W9-1 と prefix 分離確認済
排他確認:              F012 / cron が staging touch なし
所要時間 (est):       5-15 分

承認希望: W17-3 f111 staging Pattern Research + write 実行
```

## preflight checklist (= W17-3 実行直前)

- [ ] HQ approve 受領
- [ ] PK 衝突事前 SELECT で確認 (= prefix 一意)
- [ ] pre-mtime + pre-snapshot 記録
- [ ] staging DB advisory_decisions 既存 10 row + W14 後 0 row + W9-1 後 0 row
      確認
- [ ] W14 SCHEMA-R1 完全 schema 適用済 (= 22 列 + paper_reason + PK + indexes)
- [ ] LINE / token / subprocess 安全枠維持

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE16_plan|Wave 16 plan]]
- [[F286_DATA_R3_sub_D2_3_f111_smoke_plan_2026-05-12|W10-5 元 plan]]
- [[F286_DATA_R3_sub_D2_3_f111_HQ_approve_worksheet_2026-05-12|W11-3 worksheet]]
- [[F286_PNL_SCHEMA_R1_advisory_decisions_full_migration_2026-05-12|W14 SCHEMA-R1]]
