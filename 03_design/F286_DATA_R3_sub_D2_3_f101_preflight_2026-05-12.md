---
id: F286-DATA-R3-sub-D2.3.f101-preflight
phase: Wave 16 W16-2 / f101 preflight + final smoke plan
priority: 高
status: 起票 ☆ 実 staging write は W17 別 HQ approve
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W10-5 f101 smoke plan / W11-3 f101 HQ approve worksheet
  - HQ Wave 16 W16-2 承認 (= 2026-05-12)
chapter: F286 DATA / R-01-08 / preflight
---

# F286-DATA-R3 sub-D2.3.f101 Preflight + Final Smoke Plan (= W16-2)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実 staging write は W17 別 HQ 明示承認後)

## HQ 必須 8 項目

### 1. target table

- `announcements` (= 開示 header)
- `parsed_metrics` (= XBRL パース結果、F101 Phase 3)

両 staging DB のみ。

### 2. expected API call

- J-Quants V2 `/fins/announcement` (= 開示一覧、1 営業日)
- TDnet HTML 取得 (= 各 announcement の HTML page)
- XBRL ZIP 取得 (= 各 announcement の XBRL)

期間 1 営業日で開示件数 0 〜 数十件、各件 + HTML + XBRL → 1 件あたり 3 API。

### 3. expected inserted / updated row count

| table | INSERT 期待 |
|-------|-----|
| announcements | 0 〜 数十件 |
| parsed_metrics | XBRL 解析成功件数 (= announcement の subset) |

UPDATE: 0、DELETE: 0。

### 4. rollback 可否

可能。

```sql
DELETE FROM announcements WHERE fetch_date='2026-05-08';
DELETE FROM parsed_metrics WHERE fetch_date='2026-05-08';
```

(実 schema 確認後、適切な date column を確認)

### 5. pre/post mtime 確認

```bash
ls -la ~/fire/data/fire.staging.db
# pre / post で size 変化、production/develop unchanged
```

### 6. existing row 不触確認方法

```bash
# pre snapshot (= 全 row、JSON 化が大きい場合は count + max id 等で代替)
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT count(*) FROM announcements; SELECT count(*) FROM parsed_metrics;"

# post で diff 確認 (= 増分のみ、既存 row 不変)
```

### 7. failure 時の停止条件

| 条件 | 対応 |
|------|------|
| production / develop DB mtime 変化 | **即停止** + rollback |
| J-Quants auth fail | smoke 中止 |
| TDnet rate limit | smoke 中止 |
| XBRL ZIP 破損 | 当該 announcement skip、続行 |
| announcements / parsed_metrics 以外への INSERT | **即停止** |
| LINE 送信検出 | **即停止** |
| subprocess 起動検出 (= python 内部以外) | **即停止** |
| token 値露出 | **即停止** |

### 8. HQ approve template

```
=== HQ 承認依頼 / W17-2 f101 staging write 実行 ===

date:                 (実行予定日)
plan source:          F286-DATA-R3-sub-D2.3.f101-preflight (W16-2)
runner:               scripts/jobs/fetch_announcements.py
target_db:            staging
target_tables:        announcements + parsed_metrics
予定 API call:        最大 N 回 (= 開示件数 × 3 + auth)
予定 INSERT row 数:    開示件数依存
予定 UPDATE row 数:    0
予定 production write: 0
予定 develop write:    0
予定 LINE 送信:        0
予定 secret 値露出:    0
排他確認:              F012 / cron が staging に touch なし
所要時間 (est):       10-30 分 (= 開示件数 + XBRL 取得時間)

承認希望: W17-2 f101 staging fetch + write 実行
```

## preflight checklist (= W17-2 実行直前)

- [ ] HQ approve 受領
- [ ] pre-mtime + pre row count 記録
- [ ] env JQUANTS 認証情報 存在確認
- [ ] staging DB announcements / parsed_metrics schema 確認
- [ ] 対象 日付の開示件数を J-Quants で事前確認 (= dry-run probe)
- [ ] rate limit 余裕確認
- [ ] LINE / token / subprocess 安全枠維持

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE16_plan|Wave 16 plan]]
- [[F286_DATA_R3_sub_D2_3_f101_smoke_plan_2026-05-12|W10-5 元 plan]]
- [[F286_DATA_R3_sub_D2_3_f101_HQ_approve_worksheet_2026-05-12|W11-3 worksheet]]
