---
id: F286-DATA-R3-sub-D2.3.f100-preflight
phase: Wave 16 W16-1 / f100 preflight + final smoke plan
priority: 高
status: 起票 ☆ 実 staging write は W17 別 HQ approve
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W10-5 f100 smoke plan / W11-3 f100 HQ approve worksheet
  - W14-3 SCHEMA-R1 全 環境同期完了
  - W15 application path activation 完了
  - HQ Wave 16 W16-1 承認 (= 2026-05-12)
chapter: F286 DATA / R-01-08 / preflight
---

# F286-DATA-R3 sub-D2.3.f100 Preflight + Final Smoke Plan (= W16-1)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実 staging write は W17 別 HQ 明示承認後)

## HQ 必須 8 項目

### 1. target table

- `market_prices_daily` (= staging DB のみ)

### 2. expected API call

- J-Quants V2 `/prices/daily_quotes`
- 最大 3 銘柄 × 1 営業日 = 3 API call
- auth refresh: 1 回 (= JQUANTS_REFRESH_TOKEN → ID token)

### 3. expected inserted / updated row count

- INSERT: 最大 3 row (= 3 銘柄 × 1 日)
- UPDATE: 0
- DELETE: 0

### 4. rollback 可否

可能。

```sql
DELETE FROM market_prices_daily
WHERE trade_date='2026-05-08' AND symbol IN ('7203','9984','6758');
```

ただし W17 smoke で活用続行する場合は削除不要 (= HQ 判断)。

### 5. pre/post mtime 確認

```bash
ls -la ~/fire/data/fire.staging.db
# 期待: pre と post で mtime のみ変化、size 増加
```

production / develop DB mtime は **完全 unchanged enforce** (= W15 pattern)。

### 6. existing row 不触確認方法

```bash
# pre snapshot
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT * FROM market_prices_daily ORDER BY symbol, trade_date;" \
    > /tmp/w17_X_f100/pre_snapshot.txt
sha256sum /tmp/w17_X_f100/pre_snapshot.txt

# post-smoke: 新規 3 row 除外で hash 確認
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT * FROM market_prices_daily \
     WHERE NOT (trade_date='2026-05-08' AND symbol IN ('7203','9984','6758')) \
     ORDER BY symbol, trade_date;" \
    > /tmp/w17_X_f100/post_existing.txt
diff /tmp/w17_X_f100/pre_snapshot.txt /tmp/w17_X_f100/post_existing.txt
# 期待: diff 0 行
```

### 7. failure 時の停止条件

| 条件 | 対応 |
|------|------|
| production / develop DB mtime 変化検出 | **即停止** + rollback |
| J-Quants auth fail (= 401/403) | smoke 中止、env 確認 |
| rate limit / 課金超過 | smoke 中止 |
| market_prices_daily 以外への INSERT | **即停止** + rollback |
| LINE 送信検出 | **即停止** |
| subprocess 起動検出 | **即停止** |
| token 値 stdout / log 露出 | **即停止** + secret scan |

### 8. HQ approve template

```
=== HQ 承認依頼 / W17-1 f100 staging write 実行 ===

date:                 (実行予定日)
plan source:          F286-DATA-R3-sub-D2.3.f100-preflight (W16-1)
runner:               scripts/jobs/fetch_historical_market_data.py
target_db:            staging (= ~/fire/data/fire.staging.db)
target_table:         market_prices_daily
予定 API call:        最大 4 回 (= auth 1 + price 3)
予定 INSERT row 数:    最大 3
予定 UPDATE row 数:    0
予定 production write: 0 (= mtime unchanged 期待)
予定 develop write:    0
予定 LINE 送信:        0
予定 secret 値露出:    0 (= JQUANTS_REFRESH_TOKEN は env、ログ出力なし)
予定 課金:             J-Quants 微少額
排他確認:              F012 / cron が staging に touch なし
所要時間 (est):       5-10 分

承認希望: W17-1 f100 staging fetch + write 実行
```

## preflight checklist (= W17-1 実行直前)

- [ ] HQ approve 受領
- [ ] pre-mtime + pre-snapshot 記録
- [ ] env JQUANTS_REFRESH_TOKEN 存在確認 (= 値は読まず、key のみ)
- [ ] staging DB market_prices_daily schema 確認 (= 既存 column)
- [ ] 対象 3 銘柄 × 1 日が既存 row と衝突しない (= PK 一意)
- [ ] LINE 送信 0 / token 露出 0 maintain
- [ ] 失敗時 rollback 手順を読み返す

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE16_plan|Wave 16 plan]]
- [[F286_DATA_R3_sub_D2_3_f100_smoke_plan_2026-05-12|W10-5 元 plan]]
- [[F286_DATA_R3_sub_D2_3_f100_HQ_approve_worksheet_2026-05-12|W11-3 worksheet]]
