---
id: F286-DATA-R3-sub-D2.3.f111-smoke-plan
phase: Wave 10 W10-5 / sub-D2.3 f111 個別 plan
priority: 高
status: 起票 ☆ 実 staging write は HQ 別 approve 必須
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W7-2 DATA-R3 sub-D2.3 plan
  - W8-4 + W8a-fix-2 (= f111 --dry-run option 実装済)
  - HQ Wave 10 W10-5 起票承認 (= 2026-05-12)
chapter: F286 DATA シリーズ / R-01-08
---

# F286-DATA-R3 sub-D2.3.f111 Fetch/Write Smoke Plan (= Wave 10 W10-5)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実 staging write は別 HQ approve)

W7-2 sub-D2.3 plan の **f111 (advisory signal 永続化)** 個別 smoke plan。

## 対象 runner

`scripts/jobs/run_research_watchlist_signal_persistence.py` (= F111-R4)

## smoke 目的

Pattern Research / Daytrade Selection の advisory candidate を Feature
Store に persist、staging DB の `advisory_decisions` (= R-1 既存) + 関連
table に INSERT / UPDATE。

## staging write 対象 table

- `advisory_decisions` (= F286-PNL-R1 既存 table、R-1 で同 table 利用)
- `signals` 等 (= 既存 schema 確認要)

⚠️ advisory_decisions は **F286-PNL-R1/R2/R3 と共通 table**。f111 smoke で
INSERT する row は **W4.1-A 既存 10 row や W9-1 seed pattern と PK 衝突
させない** ことが重要。

## smoke 実行手順 (= 8 step、PK 衝突防止追加)

### Step 1-2: pre + dry-run (f100 と同)

### Step 3: 衝突確認 (= 追加 step)
```bash
# 予定 fetch / persist の advisory_id prefix を決定
# 例: f111-smoke-2026-05-12-<seq>
# 既存 row と prefix 衝突しないことを SELECT で事前確認
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT count(*) FROM advisory_decisions \
     WHERE advisory_id LIKE 'f111-smoke-2026-05-12-%';"
# 期待: 0
```

### Step 4: 実 staging write
```bash
.venv/bin/python -m scripts.jobs.run_research_watchlist_signal_persistence \
    --source-version smoke-2026-05-12 \
    --base-date 2026-05-08 \
    --db-path ~/fire/data/fire.staging.db
# 期待: pattern research → advisory candidate → INSERT
```

### Step 5-8: 検証 + rollback + 報告

特に既存 advisory_decisions row (= W4.1-A 10 row、W9-1 で確認済) の
hash 不変を必ず検証。

## 安全制約

- production / develop DB write 禁止
- staging DB のみ
- 既存 advisory_decisions row (= 10 row) 完全不触
- LINE 送信 0
- 既存 W9-1 seed prefix `staging-smoke-...` と衝突しない (= f111-smoke- prefix)

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE10_plan|Wave 10 plan]]
- [[F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|W7-2 元 plan]]
- HQ W10-5 起票承認 (= 2026-05-12)
