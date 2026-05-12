---
id: F286-DATA-R3-sub-D2.3.f100-smoke-plan
phase: Wave 10 W10-5 / sub-D2.3 f100 個別 plan
priority: 高
status: 起票 ☆ 実 staging write は HQ 別 approve 必須 (= Wave 11+)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W7-2 DATA-R3 sub-D2.3 plan (= 元 plan)
  - W8-4 + W8a-fix-2 (= f100 --dry-run option 実装済、external API call no-op)
  - HQ Wave 10 W10-5 起票承認 (= 2026-05-12)
chapter: F286 DATA シリーズ / R-01-08
---

# F286-DATA-R3 sub-D2.3.f100 Fetch/Write Smoke Plan (= Wave 10 W10-5)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実 staging write は別 HQ approve)

W7-2 sub-D2.3 plan の **f100 (J-Quants 過去価格取得)** 個別 smoke plan。
HQ W10-5 で 4 sub 個別分離承認 + 「runner 別に個別 HQ approve」「まとめて
write しない」指示。

## 対象 runner

`scripts/jobs/fetch_historical_market_data.py` (= F100 historical fetch)

- W8-4 で `--dry-run` option 実装 (= connection probe only)
- W8a-fix-2 で external API call **no-op 化** (= probe は env + DB のみ)
- 本 plan は `--write` 渡し時の staging fetch + staging write smoke

## smoke 目的

J-Quants V2 `/prices/daily_quotes` から **過去 N 日分の市場データ**を取得、
staging DB `market_prices_daily` table に INSERT。本 plan は smoke 実行 plan、
**実行は HQ 別 approve**。

## staging write 対象 table + 列

```
market_prices_daily
├─ symbol (TEXT)            (例: '7203')
├─ trade_date (TEXT)        (= YYYY-MM-DD)
├─ open (REAL)
├─ high (REAL)
├─ low (REAL)
├─ close (REAL)
├─ volume (INTEGER)
├─ adjustment_factor (REAL)  (= 既存 schema 確認要)
└─ created_at / updated_at (TEXT)
```

(実 schema は smoke 実行前に `PRAGMA table_info` で確認)

## smoke 実行手順 (= 9 step)

### Step 1: pre-smoke 状態キャプチャ
```bash
# DB mtime + row count
ls -la ~/fire/data/fire.staging.db > /tmp/sub_d2_3_f100/pre_mtime.txt
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT count(*) FROM market_prices_daily;" \
    > /tmp/sub_d2_3_f100/pre_count.txt

# schema 確認
sqlite3 ~/fire/data/fire.staging.db \
    "PRAGMA table_info(market_prices_daily);" \
    > /tmp/sub_d2_3_f100/pre_schema.txt

# 既存 row hash (= symbol/trade_date 指定の subset)
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT * FROM market_prices_daily ORDER BY symbol, trade_date;" \
    > /tmp/sub_d2_3_f100/pre_snapshot.txt
sha256sum /tmp/sub_d2_3_f100/pre_snapshot.txt
```

### Step 2: dry-run 再確認
```bash
.venv/bin/python -m scripts.jobs.fetch_historical_market_data \
    --from 2026-05-08 --to 2026-05-08 \
    --db-path ~/fire/data/fire.staging.db \
    --dry-run
# 期待: exit 0 (= env + DB のみ verify)
```

### Step 3: 実 staging write 実行 (= HQ approve 後)
```bash
.venv/bin/python -m scripts.jobs.fetch_historical_market_data \
    --from 2026-05-08 --to 2026-05-08 \
    --symbols 7203,9984,6758 \
    --db-path ~/fire/data/fire.staging.db
# 期待: J-Quants から 3 銘柄 × 1 営業日分 fetch、staging に INSERT
```

⚠️ ここで J-Quants 認証情報 (= JQUANTS_REFRESH_TOKEN 等) が env 経由
読み出される。secret 値露出は最小化、ログに値出力禁止。

### Step 4: post-smoke 検証
```bash
# row 増加確認
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT count(*) FROM market_prices_daily WHERE trade_date='2026-05-08' \
     AND symbol IN ('7203','9984','6758');"
# 期待: 3 row (= 3 銘柄)

# 既存 row 不触確認 (= 2026-05-08 以外の hash 不変)
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT * FROM market_prices_daily \
     WHERE NOT (trade_date='2026-05-08' AND symbol IN ('7203','9984','6758')) \
     ORDER BY symbol, trade_date;" \
    > /tmp/sub_d2_3_f100/post_existing.txt
diff /tmp/sub_d2_3_f100/pre_snapshot.txt /tmp/sub_d2_3_f100/post_existing.txt
# 期待: diff (= 増えた 3 row 以外は不変)、より厳密には pre filter で比較
```

### Step 5: production / develop DB unchanged 確認
```bash
ls -la ~/fire/data/fire.db ~/fire/data/fire.develop.db
# 期待: pre と mtime 一致
```

### Step 6: rollback 必要なら
```sql
DELETE FROM market_prices_daily
WHERE trade_date='2026-05-08' AND symbol IN ('7203','9984','6758');
```
(= smoke では 3 row 追加が validation 目的、後続 W9-1 paper_pnl smoke で
活用するため rollback しない場合も検討。HQ 承認時に決定)

### Step 7: HQ 1 ブロック報告
- 各 step exit code
- INSERT row count
- staging mtime 変化
- production / develop unchanged
- LINE 0 / token 露出 0 / external API call 1 回 (= 実 fetch)
- 結論: 成功 / 失敗

## 安全制約

- production / develop DB write 禁止 (= mtime unchanged 確認)
- staging DB のみ INSERT
- 対象 table は `market_prices_daily` のみ (= 他 table UPDATE / DROP 禁止)
- 対象 row は --from / --to で明示指定した日付の symbols
- LINE 送信 0
- 注文価格 / 数量 / 執行指示 helper 不含
- subprocess 起動なし
- cron 登録 0
- 楽天 / 自動発注 / Computer Use 不使用

## 課金 / rate limit 配慮

J-Quants V2 API fetch は **課金対象**。本 smoke では最小範囲:
- 1 営業日 (= 2026-05-08)
- 3 銘柄 (= 7203 / 9984 / 6758)
- 合計 fetch: 1 day × 3 symbols = 3 row

rate limit 抵触リスクなし。

## 実行前 HQ approve template

```
=== HQ 承認依頼 / W11-X sub-D2.3.f100 staging fetch + write 実行 ===

date:                (実行予定日)
parent_task:         F286-DATA-R3-sub-D2.3.f100-smoke (= 本 plan 実行)
target_db:           staging (= ~/fire/data/fire.staging.db)
runner:              scripts/jobs/fetch_historical_market_data.py

予定 fetch:
  期間:              1 営業日 (= 2026-05-08)
  銘柄:              3 銘柄 (= 7203 / 9984 / 6758)
  API 呼び出し:      最大 3 回 (= J-Quants /prices/daily_quotes)

予定 staging write:
  table:             market_prices_daily
  INSERT:            3 row
  UPDATE / DELETE:   0

予定 production / develop write: 0 (= mtime unchanged 確認)
予定 LINE 送信:                  0
予定 secret 値露出:              0 (= token は env 経由、ログ出力なし)
予定 課金:                       J-Quants API 課金 (= 微少額)

rollback 計画:        smoke 後の row は paper_pnl smoke (W9-1 後続) で活用
                      も検討、削除要否は HQ 判断
失敗時対応:           sub-step ごとに exit code 検証、失敗時即停止
所要時間 (estimate): 5-10 分

承認希望: 上記 W11-X sub-D2.3.f100 staging fetch + write smoke 実行
```

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE10_plan|Wave 10 plan]]
- [[F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|W7-2 元 plan]]
- HQ W10-5 起票承認 (= 2026-05-12)
