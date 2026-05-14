---
id: FIRE-CODEX-R1-WAVE60.6-fix-plan
phase: 本番 v0 中核 / Wave 60.6-fix / output path guard hardening
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60.6-fix Plan — AFTER-R1 Output Path Guard Hardening + Codex 8-Lane Recovery Smoke v1.0

## §1 目的

W60.5-codex で Codex Lane F が発見した `is_safe_output_path()` MEDIUM 問題を修正し、
terminal forbidden path (= 末尾 slash なし) を refuse する。
同時に W60.5 で復旧した stdin 経由 codex review pattern で **8 lane parallel
smoke** を実施し、Codex 並列開発の再運用を確認する。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + 既存 path guard / OUTPUT_FORBIDDEN_SEGMENTS 確認 |
| 2 | is_safe_output_path hardening 実装 (terminal forbidden + reports/after_r1 allowlist + traversal) |
| 3 | tests update + 新規 27+ 件追加 |
| 4 | 全体 pytest PASS 確認 |
| 5 | Codex 8 lane stdin smoke (= W60.5 標準 pattern) |
| 6 | vault docs + HQ 1 ブロック + 6 KPI |

## §3 修正方針

### §3.1 terminal forbidden path
- `OUTPUT_FORBIDDEN_TERMINALS` 新規追加 (= 末尾 slash なしの forbidden path 集合)
- `is_safe_output_path()` で `p_str.endswith(term)` check 追加

### §3.2 reports/after_r1 allowlist 厳格化
- `OUTPUT_REPORTS_REQUIRED_SUBPATH = "/reports/after_r1"`
- `/reports/` を含むなら after_r1/ 必須
- `/Users/` 配下なら after_r1/ 必須 (tmpdir 例外)

### §3.3 path 正規化
- `Path.resolve()` で symlink/`..` traversal 正規化
- 解決失敗 (OSError / RuntimeError) → refuse

## §4 完了条件

- is_safe_output_path の terminal forbidden 修正完了
- reports/after_r1/ 以外への危険出力拒否
- tests 全 PASS
- pytest collected の変化を報告
- Codex stdin 経由 8 lane smoke 実施
- Codex permission denied 再発なし、再発時は原因と代替明記
- DB write / sqlite / LINE / token / API / launchctl / plist / cron / VACUUM /
  workflow / --no-verify / git push / sudo / rm -rf / TODO Excel 全 0
- F282 plist + 3 DB 不変
- vault に plan + results + HQ 1 ブロック + 6 KPI

## §5 停止条件

DB write / production-develop-staging 接続 / LINE / token / API / launchctl /
plist / cron / brokerage / 自動発注 / VACUUM / workflow / --no-verify / git push /
sudo / rm -rf / TODO Excel / Computer Use が必要 → 即停止 + HQ
