---
id: FIRE-CODEX-R1-WAVE60-F111-PRESET-TUNE-plan
phase: 本番 v0 中核 / Wave 60-F111-preset-tune / 候補多様化 短期改善
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
hq_marker: なし (= code 変更のみ、staging read-only、token/API/DB write 0)
---

# Wave 60-F111-preset-tune Plan — F111 Real Batch Preset / Label / Risk Tune for D9 v1.0

## §1 目的

W60-features-rerun-pre で features cap の真因 (= market_prices_daily 2026-05-08
止まり) を切り分けた。真の解決は J-Quants daily refresh (= classification C、
HQ marker 2 個必要) だが、API/token 承認前にできる **短期策** として
F111-real-batch の preset / label mapping / universe / risk filter を調整し、
D9 trade plan で D6/D7/D8 と完全同一候補になる問題を **緩和** する。

## §2 安全境界

- API call / token / .env / env 全体参照: **禁止**
- DB write (production / develop / staging): **禁止**
- production / develop DB 接続: **禁止**
- staging DB: **read-only のみ**
- LINE / workflow / cron / launchctl / plist / VACUUM / --no-verify /
  git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED / Computer Use: **禁止**
- TODO Excel 更新: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist / 3 DB md5 / pytest collect) |
| 2 | F111-real-batch + AFTER-R1 現行ロジック把握 |
| 3 | 調整方針: label 閾値 + max_candidates + recently_seen demote |
| 4 | implement: F111-real-batch / _after_r1_mvp 改修 |
| 5 | tests 追加 + 既存 PASS 維持 |
| 6 | D9 simulation (= 3 scenario) + D6-D9 overlap 計算 |
| 7 | Codex 4 lane factual-confirm |
| 8 | vault plan + results + HQ 1-block + 6 KPI |

## §4 改修 3 軸 (= 過剰品質低下なし)

| 軸 | 内容 | 効果 |
|---|---|---|
| **a. label_threshold_mode** | default (0.80/0.70/0.60) と strict (0.92/0.85/0.75) を CLI flag で切替 | boost / boost_with_caution / caution の label 分散 |
| **b. recently_seen_codes** | D6-D8 で見た銘柄をカンマ区切りで指定、boost/boost_with_caution → caution へデモート + risk_notes に警告 | 「最近見た銘柄」明示 |
| **c. max_candidates default 10 → 20** | top 3 同候補のままだが、4-N で多様化 | Fujiwara が 4 位以下を見る運用 |

## §5 完了条件

- D6/D7/D8 重複再確認 ✓
- F111 preset/label/risk 短期調整完了
- D9 candidate simulation (= 3 scenario) 完了
- D6-D9 overlap 率 (= ticker + label) 報告
- D9 で実弾判断に使える候補が出るか明確
- 短期策の限界 + J-Quants refresh が真の解決と明記
- tests PASS (= 既存 55 維持 + 新規追加)
- Codex 4 lane factual-confirm 実施
- DB write 0 / production-develop 接続 0 / staging write 0
- LINE 0 / token 0 / API 0 / launchctl 0 / plist 0 / cron 0
- 実発注 0 / 楽天 0 / iSPEED 0 / Computer Use 0
- F282 plist + 3 DB md5 不変
- fire-vault に plan + results
- HQ 1-block + 6 KPI

## §6 停止条件

16 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env / API /
LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED / Computer Use /
workflow / --no-verify / git push / sudo / rm -rf / TODO Excel

## §7 Codex 4 lane (factual-confirm)

- A: preset/universe (= --scale-categories choices, --max-candidates default 20)
- B: label mapping (= threshold_mode default vs strict, LABEL_THRESHOLDS dict)
- C: risk filter (= PILOT_PER_TRADE_RISK_LIMIT_YEN unchanged, demote = label only)
- D: TOP_INCLUDE_LABELS (= contains caution, demoted candidates surface as caution)

stdin pipe pattern: `echo "<≤80 words>" | codex review -`
