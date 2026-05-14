---
id: FIRE-CODEX-R1-WAVE60-pilot-pre-plan
phase: 本番 v0 中核 / Wave 60-pilot-pre / Small Manual Live Pilot Operation
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60-pilot-pre Plan — Small Manual Live Pilot Operation Plan v1.0

## §1 目的

AFTER-R1 Paper Live MVP の 4 種類成果物 (paper_live_ledger /
good_candidate_ranking / pattern_candidate_report / morning_line_material) を
**藤原さんの少額手動実弾** に接続し、「候補生成 → 手動実弾 → 結果記録 → 改善」
ループを開始するため、運用ルール / リスク上限 / 記録テンプレ / 緊急停止条件を
正式化する。

## §2 構成 (= docs / template only、FIRE 本体不触)

| step | 内容 |
|---|---|
| 1 | baseline + scope 確認 |
| 2 | 設計 doc 作成 (~/fire-vault/03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md) |
| 3 | trade_plan template 作成 (= 04_daily) |
| 4 | trade_review template 作成 (= 04_daily) |
| 5 | Codex 8 lane audit (stdin pattern) |
| 6 | findings 集約 + CRITICAL/HIGH 修正 |
| 7 | vault plan + results + HQ 1 ブロック + 6 KPI |

## §3 安全境界

- docs / template のみ、FIRE 本体コード変更 0
- 実発注 / 自動発注 / LINE 送信 / DB write / token 参照 / API call 0
- 楽天証券 / iSPEED / Computer Use / Playwright 言及は **禁止項目として記述のみ**
- FIRE 本体不触
- F282 / 3 DB / W30 snapshot 完全不変
- Codex 8 lane は read-only (= stdin 経由 codex review)

## §4 完了条件

- operation plan 完成 (= 9 セクション以上)
- trade_plan template 完成
- trade_review template 完成
- pilot GO/NO-GO / risk limits / entry-stop-TP-close / no-trade / emergency stop
  明確
- Paper Live MVP artifact 連携手順明確
- 自動発注なし / 楽天手動 / Computer Use なし が明示
- Codex 8 lane audit 実施 (= stdin pattern)
- CRITICAL/HIGH 修正済 or HQ 判断待ち
- FIRE 本体コード変更 0
- F282 plist + 3 DB 不変
- vault に plan + results + HQ 1 ブロック + 6 KPI

## §5 停止条件

実発注 / 楽天証券操作 / Computer Use / DB write / LINE / token / API call /
launchctl / plist / cron / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
