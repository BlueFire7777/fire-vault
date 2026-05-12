---
id: F286-DATA-R3-sub-D2.3.f101-smoke-plan
phase: Wave 10 W10-5 / sub-D2.3 f101 個別 plan
priority: 高
status: 起票 ☆ 実 staging write は HQ 別 approve 必須
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W7-2 DATA-R3 sub-D2.3 plan (= 元 plan)
  - W8-4 + W8a-fix-2 (= f101 --dry-run option 実装済、external API call no-op)
  - HQ Wave 10 W10-5 起票承認 (= 2026-05-12)
chapter: F286 DATA シリーズ / R-01-08
---

# F286-DATA-R3 sub-D2.3.f101 Fetch/Write Smoke Plan (= Wave 10 W10-5)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実 staging write は別 HQ approve)

W7-2 sub-D2.3 plan の **f101 (TDnet 適時開示)** 個別 smoke plan。

## 対象 runner

`scripts/jobs/fetch_announcements.py` (= F101 TDnet 開示)

## smoke 目的

J-Quants V2 `/fins/announcement` から **過去 N 日分の TDnet 開示**を取得、
staging DB `announcements` + `parsed_metrics` table に INSERT。

## staging write 対象 table

- `announcements` (= 開示 header)
- `parsed_metrics` (= XBRL パース結果、F101 Phase 3 で導入)

(実 schema は smoke 前に PRAGMA table_info で確認)

## smoke 実行手順 (= 7 step、f100 と同 pattern)

### Step 1-2: pre + dry-run (f100 と同様)

### Step 3: 実 staging write
```bash
.venv/bin/python -m scripts.jobs.fetch_announcements \
    --from 2026-05-08 --to 2026-05-08 \
    --db-path ~/fire/data/fire.staging.db
# 期待: TDnet 開示 0-数件 fetch、announcements + parsed_metrics に INSERT
```

⚠️ TDnet は 1 営業日で開示件数が大幅に変動 (= 0 〜 数百件)。
rate limit に注意、最小範囲で。

### Step 4-7: 検証 + 報告 (f100 と同)

## 安全制約

- production / develop DB write 禁止
- staging DB のみ INSERT (= announcements + parsed_metrics のみ)
- LINE 送信 0
- TDnet HTML 取得は読み取り専用 (= TDnet 側に何も書かない)
- subprocess なし / cron なし

## rate limit 配慮

TDnet HTML は J-Quants 経由で取得。1 営業日に絞れば rate limit 抵触
回避可能。

## 実行前 HQ approve template

(= f100 plan と同 pattern、target が f101 になる)

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE10_plan|Wave 10 plan]]
- [[F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|W7-2 元 plan]]
- [[F286_DATA_R3_sub_D2_3_f100_smoke_plan_2026-05-12|f100 plan]]
- HQ W10-5 起票承認 (= 2026-05-12)
