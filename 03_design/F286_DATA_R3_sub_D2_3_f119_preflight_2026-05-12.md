---
id: F286-DATA-R3-sub-D2.3.f119-preflight
phase: Wave 16 W16-4 / f119 preflight + final smoke plan
priority: 高
status: 起票 ☆ 実 staging write は W17 別 HQ approve + LINE disable 手段確定要
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W10-5 f119 smoke plan / W11-3 f119 HQ approve worksheet
  - HQ Wave 16 W16-4 承認 (= 2026-05-12)
chapter: F286 DATA / R-01-08 / preflight
---

# F286-DATA-R3 sub-D2.3.f119 Preflight + Final Smoke Plan (= W16-4)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実 staging write は W17 別 HQ 明示承認、LINE disable 手段確定要)

## ⚠️ 重要前提: LINE disable 手段が未確定

F119 Phase 3 orchestrator は R-13-08 で **LINE REPORT 部屋通知** を含む
設計。smoke 中の LINE 送信は禁止のため、disable 手段の確定が **W17-4 実行
の前提条件**。

候補:
1. env `FIRE_LINE_DISABLE=1` (= 実装上、F119 内部で参照、新規実装必要)
2. config 経由 notification_recipient を空に設定
3. monkeypatch / mock (= 実 runner では不可)
4. smoke 用 wrapper runner (= notification_recipient を強制 None)

HQ 明示判断が **W17-4 実行前に必要** (= 本 plan で別途確認)。

## HQ 必須 8 項目

### 1. target table

- `evaluation_reports` (= F119 Phase 3 既存)
- `approval_requests` (= F038 既存、R-13-08 承認制)
- + 関連 (= proposals 等、実 runner で確認)

staging DB のみ。

### 2. expected API call

- 外部 API なし (= internal aggregation + Markdown 生成)

### 3. expected inserted / updated row count

- INSERT (evaluation_reports): 1 件 (= 当 base_date 集計レポート)
- INSERT (approval_requests): 数件 (= proposal 件数次第)
- UPDATE: 0
- DELETE: 0

### 4. rollback 可否

可能、source_version 限定 DELETE。

```sql
DELETE FROM evaluation_reports WHERE source_version='smoke-2026-05-12';
DELETE FROM approval_requests
  WHERE proposed_by LIKE 'smoke%' OR source_version='smoke-2026-05-12';
```

(実 schema 確認後、適切 column 確認)

### 5. pre/post mtime 確認

```bash
ls -la ~/fire/data/fire.staging.db
# production / develop unchanged enforce
```

### 6. existing row 不触確認方法

```bash
# pre snapshot (= source_version 別)
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT * FROM evaluation_reports \
     WHERE source_version != 'smoke-2026-05-12' \
     ORDER BY id;" \
    > /tmp/w17_X_f119/pre_existing.txt
sha256sum /tmp/w17_X_f119/pre_existing.txt

# post で diff 0 行確認
```

### 7. failure 時の停止条件

| 条件 | 対応 |
|------|------|
| production / develop DB mtime 変化 | **即停止** + rollback |
| **LINE 通知トリガー検出** (= R-13-08 経路) | **即停止** ★最重要 |
| evaluation_reports / approval_requests 以外への INSERT | **即停止** |
| LINE disable 手段未確定 | smoke 開始**しない** |
| Markdown レポート 生成失敗 | smoke 中断 |
| subprocess 起動検出 | **即停止** |
| token 値露出 | **即停止** |

### 8. HQ approve template

```
=== HQ 承認依頼 / W17-4 f119 staging write 実行 ===

date:                 (実行予定日)
plan source:          F286-DATA-R3-sub-D2.3.f119-preflight (W16-4)
runner:               scripts/jobs/run_f119_interpretation_evaluation.py
target_db:            staging
target_tables:        evaluation_reports + approval_requests
LINE disable 手段:    (= HQ 確定後記入、例: FIRE_LINE_DISABLE=1 env)
予定 API call:        0
予定 INSERT row 数:    数件
予定 UPDATE row 数:    0
予定 production write: 0
予定 develop write:    0
予定 LINE 送信:        0 (= disable 手段経由で 確実 abort)
予定 secret 値露出:    0
排他確認:              F119 通常実行 / cron が staging に touch なし
所要時間 (est):       5-15 分

承認希望: W17-4 f119 staging evaluation + write 実行
         (= LINE disable 手段確定後)
```

## preflight checklist (= W17-4 実行直前)

- [ ] **LINE disable 手段 HQ 確定** (= 最重要 prerequisite)
- [ ] disable 手段の実装 / 確認 (= 新規 env flag 等が必要なら別 task)
- [ ] HQ approve 受領
- [ ] pre-mtime + pre-snapshot 記録 (= source_version 別)
- [ ] staging DB evaluation_reports / approval_requests schema 確認
- [ ] R-13-08 LINE 通知経路の disable 確認 (= test or dry-run で確実 abort)

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE16_plan|Wave 16 plan]]
- [[F286_DATA_R3_sub_D2_3_f119_smoke_plan_2026-05-12|W10-5 元 plan]]
- [[F286_DATA_R3_sub_D2_3_f119_HQ_approve_worksheet_2026-05-12|W11-3 worksheet]]
