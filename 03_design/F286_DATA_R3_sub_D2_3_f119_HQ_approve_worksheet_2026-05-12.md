---
id: F286-DATA-R3-sub-D2.3.f119-HQ-approve-worksheet
phase: Wave 11 W11-3 / sub-D2.3.f119 HQ approve worksheet
priority: 高
status: 起票 ☆ 実 staging write は HQ 別 明示承認必須
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W10-5 sub-D2.3.f119 smoke plan
  - HQ Wave 11 W11-3 起票承認 (= 2026-05-12)
chapter: F286 DATA / R-01-08
---

# F286-DATA-R3 sub-D2.3.f119 HQ Approve Worksheet (= Wave 11 W11-3)

最終更新: 2026-05-12

## ★ 状態: 起票 (= HQ 別 明示承認必須、他 runner と必ず分離実行)

## 対象 runner

`scripts/jobs/run_f119_interpretation_evaluation.py` (= F119 Evaluation Agent)

## staging write 対象

| table | 操作 |
|------|------|
| evaluation_reports | INSERT (= F119 Phase 3 既存) |
| approval_requests | INSERT (= F038、R-13-08 承認制) |
| proposals 等 | 必要なら INSERT |

★ R-13-08 で本来 **LINE REPORT 部屋通知が含まれる** が、本 smoke では
**LINE 通知 disable で実行**。

## 完全 HQ approve template

```
=== HQ 承認依頼 / W12-X-f119 sub-D2.3.f119 staging fetch + write 実行 ===

date:                (実行予定日)
parent_task:         F286-DATA-R3-sub-D2.3.f119
runner:              scripts/jobs/run_f119_interpretation_evaluation.py
plan source:         [[F286_DATA_R3_sub_D2_3_f119_smoke_plan_2026-05-12]]

予定 evaluation:
  base_date:         (例: 2026-05-08)
  source_version:    smoke-2026-05-12

予定 staging write:
  tables:            evaluation_reports + approval_requests
  INSERT row 数:    集計結果次第 (= 数件)

予定 production write:  0
予定 develop write:     0
予定 LINE 送信:         0 (= R-13-08 通知を **disable で実行**)
予定 secret 値露出:     0
予定 課金:              0

LINE disable の手段:
  - env F286_LINE_DISABLE=1 設定 (= 実装上、本起票で確認要)
  - もしくは smoke 用 wrapper runner を用意
  - F119 Phase 3 orchestrator (submit_proposals) が LINE REPORT 通知を
    skip するかの実装確認 + HQ 承認時提示

rollback 計画:
  - DELETE FROM evaluation_reports WHERE source_version='smoke-2026-05-12'
  - DELETE FROM approval_requests WHERE proposed_by LIKE 'smoke%' (= 必要なら)

失敗時対応:
  - LINE 送信トリガー検出 → 即停止
  - 不要 approval_requests row 残置 → rollback

排他確認:
  - F119 通常実行と並走していないこと

承認希望: 上記 W12-X-f119 staging fetch + write smoke 実行
         (= LINE 通知 disable の手段を別途明示)
```

## LINE disable 検討メモ

F119 Phase 3 orchestrator は `submit_proposals` → LINE REPORT 通知が設計
組込。smoke 用の disable 手段が必要:

候補:
1. env `FIRE_LINE_DISABLE=1` (= F119 内部で参照、実装要)
2. config 経由で notification_recipient を空に設定
3. monkeypatch / mock 経由 (= 実 runner では不可)
4. smoke 用 wrapper runner (= notification_recipient を強制 None)

実装方針は HQ 承認時に具体化。本 worksheet では「LINE disable 手段が
未確定」を明示。

## 関連リンク

- [[F286_DATA_R3_sub_D2_3_f119_smoke_plan_2026-05-12|f119 plan]]
- [[F286_DATA_R3_sub_D2_3_f100_HQ_approve_worksheet_2026-05-12|f100 worksheet (= pattern)]]
