---
id: F286-DATA-R3-sub-D2.3.f101-HQ-approve-worksheet
phase: Wave 11 W11-3 / sub-D2.3.f101 HQ approve worksheet
priority: 高
status: 起票 ☆ 実 staging write は HQ 別 明示承認必須
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W10-5 sub-D2.3.f101 smoke plan
  - HQ Wave 11 W11-3 起票承認 (= 2026-05-12)
chapter: F286 DATA / R-01-08
---

# F286-DATA-R3 sub-D2.3.f101 HQ Approve Worksheet (= Wave 11 W11-3)

最終更新: 2026-05-12

## ★ 状態: 起票 (= HQ 別 明示承認必須、他 runner と必ず分離実行)

## 対象 runner

`scripts/jobs/fetch_announcements.py` (= F101 TDnet 開示)

## staging write 対象

| table | 操作 |
|------|------|
| announcements | INSERT (= 開示 header) |
| parsed_metrics | INSERT (= XBRL パース結果) |

## 完全 HQ approve template

```
=== HQ 承認依頼 / W12-X-f101 sub-D2.3.f101 staging fetch + write 実行 ===

date:                (実行予定日)
parent_task:         F286-DATA-R3-sub-D2.3.f101
runner:              scripts/jobs/fetch_announcements.py
plan source:         [[F286_DATA_R3_sub_D2_3_f101_smoke_plan_2026-05-12]]

予定 fetch range:
  期間:             (例: 2026-05-08 単日)
  銘柄 filter:      (= 通常 filter なし、TDnet 全件)
  API 呼び出し:     最大 N 回 (= J-Quants /fins/announcement)

予定 staging write:
  tables:           announcements + parsed_metrics
  INSERT row 数:    開示件数依存 (= 0 〜 数百件)
  UPDATE row 数:    0
  DELETE row 数:    0

予定 production write:  0
予定 develop write:     0
予定 LINE 送信:         0
予定 secret 値露出:     0
予定 課金:              J-Quants API 課金
予定 XBRL fetch:        各 announcement の XBRL ZIP 取得 (= 別 fetch)

rollback 計画:
  - 削除する場合: DELETE FROM announcements / parsed_metrics WHERE
    fetch_date='YYYY-MM-DD'

失敗時対応:
  - rate limit 抵触 → 即停止
  - XBRL ZIP 破損 → 該当 announcement skip、次へ
  - DB write 失敗 → transaction rollback、全停止

排他確認:
  - F012 FIRE Runner が announcements に書込んでいないこと

承認希望: 上記 W12-X-f101 staging fetch + write smoke 実行
```

## 実行前 checklist + 中断条件 + 完了報告 template

(= f100 worksheet と同 pattern)

## 関連リンク

- [[F286_DATA_R3_sub_D2_3_f101_smoke_plan_2026-05-12|f101 plan]]
- [[F286_DATA_R3_sub_D2_3_f100_HQ_approve_worksheet_2026-05-12|f100 worksheet (= pattern)]]
