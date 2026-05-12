---
id: F286-DATA-R3-sub-D2.3.f111-HQ-approve-worksheet
phase: Wave 11 W11-3 / sub-D2.3.f111 HQ approve worksheet
priority: 高
status: 起票 ☆ 実 staging write は HQ 別 明示承認必須
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W10-5 sub-D2.3.f111 smoke plan
  - HQ Wave 11 W11-3 起票承認 (= 2026-05-12)
chapter: F286 DATA / R-01-08
---

# F286-DATA-R3 sub-D2.3.f111 HQ Approve Worksheet (= Wave 11 W11-3)

最終更新: 2026-05-12

## ★ 状態: 起票 (= HQ 別 明示承認必須、他 runner と必ず分離実行)

## 対象 runner

`scripts/jobs/run_research_watchlist_signal_persistence.py` (= F111-R4
advisory signal persistence)

## staging write 対象

| table | 操作 |
|------|------|
| advisory_decisions | INSERT (= W4.1-A 10 row / W9-1 seed 5 row と PK 衝突しない prefix) |
| signals 等 | INSERT (= 既存 schema 確認要) |

⚠️ advisory_decisions は F286-PNL-R1/R2/R3 + W9-1 と **共通 table**。
**PK 衝突回避 必須**。

## 完全 HQ approve template

```
=== HQ 承認依頼 / W12-X-f111 sub-D2.3.f111 staging fetch + write 実行 ===

date:                (実行予定日)
parent_task:         F286-DATA-R3-sub-D2.3.f111
runner:              scripts/jobs/run_research_watchlist_signal_persistence.py
plan source:         [[F286_DATA_R3_sub_D2_3_f111_smoke_plan_2026-05-12]]

予定 fetch range:
  source_version:    f111-smoke-2026-05-12 (= 別 prefix で衝突回避)
  base_date:         (例: 2026-05-08)
  銘柄:              Pattern Research 結果次第

予定 staging write:
  tables:            advisory_decisions + signals (= 必要に応じて)
  advisory_id prefix: f111-smoke-2026-05-12-... (= W4.1-A や W9-1 と衝突なし)
  INSERT row 数:    候補抽出結果次第 (= 数件〜数十件)

予定 production write:  0
予定 develop write:     0
予定 LINE 送信:         0
予定 secret 値露出:     0
予定 課金:              0 (= 外部 API なし、Feature Store のみ)

PK 衝突回避:
  - W4.1-A 10 row (= f062-r5.8-... / production-advisory-...) と衝突なし
  - W9-1 seed (= staging-smoke-...) と衝突なし
  - f111-smoke- prefix 一意性 SELECT 前確認

rollback 計画:
  - DELETE FROM advisory_decisions WHERE advisory_id LIKE 'f111-smoke-%'

失敗時対応:
  - PK 衝突検出 → 即停止
  - Pattern Research 失敗 → 該当 batch skip

排他確認:
  - F012 FIRE Runner が advisory_decisions に書込んでいないこと
  - W9-1 paper_pnl smoke が並走していないこと

承認希望: 上記 W12-X-f111 staging fetch + write smoke 実行
```

## 関連リンク

- [[F286_DATA_R3_sub_D2_3_f111_smoke_plan_2026-05-12|f111 plan]]
- [[F286_DATA_R3_sub_D2_3_f100_HQ_approve_worksheet_2026-05-12|f100 worksheet (= pattern)]]
