---
id: FIRE-CODEX-R1-WAVE52-post-plan
phase: 本番 v0 Launch / Wave 52-post / wrapper / marker / DB labels audit
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 52-post Plan — Wrapper / Marker / DB Labels Contract Adversarial Audit

## §1 目的
W52-pre 実装 (no-send wrapper / marker / DB labels preview contract) を
敵対的観点で 8 lane 並列監査し、誤送信 / marker 誤判定 / DB write 化 /
曖昧 marker 復活リスクを潰す。

## §2 方針
監査 + tests 追加 + 最小バグ修正のみ。実 send / 実 marker / DB write 0。

## §3 構成 (= 8 Codex lane)

| lane | 観点 |
|---|---|
| L5 本線 | baseline + 8 lane 起動 + triage + minimal fix + tests + 報告 |
| Codex A | marker semantics / 7 marker 整合 |
| Codex B | wrapper args / no-send chain |
| Codex C | production send path 到達不能 |
| Codex D | DB labels schema / no-write |
| Codex E | tmpdir guard / path guard |
| Codex F | readiness v1.2 / Ops Summary v1.0.1 連携 |
| Codex G | source safety AST |
| Codex H | runbook / docs / failure matrix |

## §4 完了条件
- 8 観点 audit 完了
- CRITICAL / HIGH 0、修正済 or HQ 判断待ち
- pytest 全 PASS + collected 数報告
- F282 / 3 DB / launchctl / cron / VACUUM / workflow / TODO Excel 全 0
- vault に W52-post 報告 + HQ 1 ブロック + 6 KPI

## §5 停止条件
production send 実行 / DB write / DB sqlite 接続 / 実 marker 作成 / token 値参照 /
launchctl / plist / cron / VACUUM / F282 manual / workflow / --no-verify /
git push / sudo / TODO Excel / 楽天 / 自動発注 / Computer Use が必要 → 即停止 + HQ
