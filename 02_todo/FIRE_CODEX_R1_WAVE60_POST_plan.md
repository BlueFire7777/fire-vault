---
id: FIRE-CODEX-R1-WAVE60-post-plan
phase: 本番 v0 中核 / Wave 60-post / AFTER-R1 MVP adversarial audit
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60-post Plan — AFTER-R1 Paper Live MVP / Candidate Ranking / Pattern Report Adversarial Audit v1.0

## §1 目的

W60-impl で実装した F286-AFTER-R1 Paper Live MVP (= 4 artifact 生成器) を
**少額手動実弾パイロット前** に 8 観点で敵対的監査し、
ランキング誤判定 / 危険表現 / 出力 namespace 逸脱 / 安全 flag 破壊 /
自動発注誤解 / pattern 過信リスクを潰す。

## §2 構成 (= 8 lane 原則)

| Lane | 観点 |
|---|---|
| A | input resolver / missing / malformed input |
| B | good_candidate_ranking scoring |
| C | paper_live_ledger safety / status |
| D | pattern_candidate_report overclaim / schema |
| E | morning_line_material wording / manual trade plan |
| F | output namespace / path guard / artifact schema |
| G | safety AST / no DB / LINE / token / API / order |
| H | docs / v0 product definition / live pilot readiness |

### Codex CLI lane 起動

本セッションでは `codex review` 起動が **permission denied** されたため、
8 lane の Codex 並列起動は不可。本線で各 lane を self-audit し、
HQ 報告に「Codex CLI 制約による本線監査」を明記する。

## §3 実施項目

1. baseline 取得 (F282 plist + 3 DB + pytest collected)
2. Codex CLI 起動可否確認 → 本線 self-audit に切替
3. Lane A-H 観点で監査 + finding 列挙
4. CRITICAL/HIGH 修正 (= 最小限)
5. MEDIUM/LOW は docs 注記 or 次 wave 候補
6. regression tests 追加
7. 全 pytest PASS 確認
8. F282 / 3 DB / W30 snapshot 不変確認
9. vault に plan + results + HQ 1 ブロック + 6 KPI

## §4 完了条件

- 8 観点 audit 完了
- CRITICAL/HIGH 0 (= 全 修正済) or 0 件発見
- 修正は v0 本線 (F282/DATA-R3/F062/readiness/Ops/wrapper) を不触
- tests 全 PASS
- pytest collected の変化を報告
- 8 Codex lanes 未使用理由を明記 (= 上記 §2)
- ranking 過剰推奨 0、pattern_candidate_only 維持、material 命令調・自動発注示唆 0、
  ledger 実取引誤認可能性 0
- reports/after_r1/ namespace 制約維持
- 安全 flag 13 keys 全 false / auto_order_allowed=False /
  manual_review_required=True 全 artifact で維持
- F282 plist + 3 DB 不変
- vault + HQ 報告

## §5 停止条件

DB write / DB sqlite 接続 / LINE 送信 / token / .env / env 全体 / API call /
launchctl / plist / cron 変更 / VACUUM / Paper Live Real / brokerage /
自動発注 / Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
