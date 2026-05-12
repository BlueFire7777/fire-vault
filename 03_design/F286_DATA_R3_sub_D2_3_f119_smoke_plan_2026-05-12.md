---
id: F286-DATA-R3-sub-D2.3.f119-smoke-plan
phase: Wave 10 W10-5 / sub-D2.3 f119 個別 plan
priority: 高
status: 起票 ☆ 実 staging write は HQ 別 approve 必須
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W7-2 DATA-R3 sub-D2.3 plan
  - W8-4 + W8a-fix-2 (= f119 --dry-run option 実装済)
  - HQ Wave 10 W10-5 起票承認 (= 2026-05-12)
chapter: F286 DATA シリーズ / R-01-08
---

# F286-DATA-R3 sub-D2.3.f119 Fetch/Write Smoke Plan (= Wave 10 W10-5)

最終更新: 2026-05-12

## ★ 状態: 起票 (= 実 staging write は別 HQ approve)

W7-2 sub-D2.3 plan の **f119 (F119 解釈評価 / Evaluation Agent)** 個別
smoke plan。

## 対象 runner

`scripts/jobs/run_f119_interpretation_evaluation.py`

## smoke 目的

F119 Evaluation Agent の解釈評価結果 (= Markdown report + DB)
を staging に保存。第 13 章 R-13-02〜10 の完全閉ループ (= W6 で実装済)
を staging 環境で smoke。

## staging write 対象 table

- `evaluation_reports` (= F119 Phase 3 既存)
- `approval_requests` (= F038 既存、R-13-08 承認制)
- `proposals` (= F119 提案、未確認)

実 schema は smoke 前に確認。

## smoke 実行手順 (= 7 step、f100 と同 pattern)

### Step 1-2: pre + dry-run
### Step 3: 実 staging write
```bash
.venv/bin/python -m scripts.jobs.run_f119_interpretation_evaluation \
    --base-date 2026-05-08 \
    --source-version smoke-2026-05-12 \
    --db-path ~/fire/data/fire.staging.db
# 期待: F119 集計 + Markdown 生成 + evaluation_reports / approval_requests INSERT
```

### Step 4-7: 検証 + rollback + 報告

## 安全制約

- production / develop DB write 禁止
- staging DB のみ
- evaluation_reports / approval_requests への INSERT のみ
- 既存 row 完全不触
- LINE 送信 0 (= R-13-08 で LINE REPORT 部屋通知が設計上含まれるが、本 smoke
  では LINE 通知 disable で実行)
- subprocess 起動 0
- cron 登録 0

## LINE 送信 disable の手段

F119 Phase 3 orchestrator (= submit_proposals → LINE REPORT 通知) を
smoke 中は通知 mock or env flag で disable。実装上の対応:
- env `FIRE_LINE_DISABLE=1` 等の dry-run flag を実装に追加 (= 別 sub-task)
- または smoke 用 wrapper runner を別途作成
- いずれの場合も HQ 承認時に手段を明示

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE10_plan|Wave 10 plan]]
- [[F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|W7-2 元 plan]]
- HQ W10-5 起票承認 (= 2026-05-12)
