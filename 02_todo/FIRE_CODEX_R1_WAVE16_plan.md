---
id: FIRE-CODEX-R1-WAVE16-plan
phase: ガバナンス / Wave 16 起票 / R-01-08
priority: 最優先
status: 起票 ☆ Wave 16 5 sub-task 並列実行中 (2026-05-12)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 15 (= 完了)
  - HQ Wave 16 一括承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / DATA-R3 sub-D2.3.x preflight
---

# Wave 16: DATA-R3 sub-D2.3.x 4 runner preflight + final smoke plan + audit

最終更新: 2026-05-12

## ★ 状態: 起票 (= 5 sub-task、preflight + plan + audit only、staging write 0)

W10-5 sub-D2.3.x smoke plan + W11-3 HQ approve worksheet を統合し、実 write
直前の preflight checklist + final smoke plan として再整備。実 write は
W17+ で runner 別 HQ 別 approve 後。

## Wave 16 構成

| sub | lane | task |
|-----|------|------|
| W16-1 | 本線 | f100 preflight + final smoke plan |
| W16-2 | 本線 | f101 preflight + final smoke plan |
| W16-3 | 本線 | f111 preflight + final smoke plan |
| W16-4 | 本線 | f119 preflight + final smoke plan |
| W16-5 | L4 (Codex) | DATA-R3 write guard audit |

## HQ 必須項目 (= 各 plan に含める)

1. target table
2. expected API call
3. expected inserted / updated row count
4. rollback 可否 + 手順
5. pre/post mtime 確認
6. existing row 不触確認方法
7. failure 時の停止条件
8. HQ approve template

## 安全制約 (= 全 ✓)

実 LINE / production / develop / staging DB write / token / cron / 楽天 /
自動発注 / Computer Use / workflow / --no-verify / TODO Excel 全 0。

## 関連リンク

- [[FIRE_CODEX_R1_WAVE15_results|Wave 15 results]]
- [[../03_design/F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|W7-2 元 plan]]
- [[../log]]
