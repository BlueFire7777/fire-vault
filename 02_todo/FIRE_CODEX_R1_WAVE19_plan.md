---
id: FIRE-CODEX-R1-WAVE19-plan
phase: ガバナンス / Wave 19 / F101 API 調査 + staging cleanup policy
priority: 高
status: 完了 ☆ 3 sub-task (= plan + W19-1 + W19-2)、staging write 0
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 18 (= 完了)
  - HQ Wave 19 W19-1/W19-2 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F101 + staging cleanup
---

# Wave 19: F101 API 403 investigation + staging smoke row cleanup policy

W18-2 の HTTP 403 別 issue 調査 + W17-3/W18-1 残置 row inventory + filter
policy。実 staging write 0、API call 0。

## Wave 19 sub-task

| sub | task | 結果 |
|---|---|---|
| W19-1 | F101 API behavior investigation | ✓ vault doc、推奨対応提示 |
| W19-2 | staging smoke row inventory + cleanup policy | ✓ vault doc、filter 方針確定 |

## 関連リンク

- [[../03_design/F101_API_403_investigation_2026-05-12|W19-1 F101 調査]]
- [[../03_design/F286_staging_smoke_row_inventory_cleanup_policy_2026-05-12|W19-2 cleanup policy]]
