---
id: FIRE-CODEX-R1-WAVE60-F111-DAILY-REFRESH-plan
phase: 本番 v0 中核 / Wave 60-F111-daily-refresh / staging signal 日次更新
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
hq_marker: HQ_APPROVE_STAGING_RESEARCH_SIGNAL_REFRESH=1
---

# Wave 60-F111-daily-refresh Plan — F111 Morning Batch / Research Signal Daily Refresh for D8 v1.0

## §1 目的

D7 (2026-05-22) で HOLD 推奨となった 2 因 (D6 review missing + staging
research_watchlist_signals base_date 2026-05-12 stale) のうち、後者
**staging signal の日次更新経路** を確立し、D8 (2026-05-25 月) GO 候補生成
の基盤を整える。

## §2 経路特定 + 分類

| 観点 | 結果 |
|---|---|
| 既存 runner | `scripts/jobs/run_research_watchlist_signal_persistence.py` (568 行) |
| API/token | requests / aiohttp / urllib / httpx 全 import なし → 0 |
| 内部依存 | run_research_watchlist_ranker / score_factor_strategies / migrate_research_watchlist_signals / simulation.research_lane.* |
| --write 安全 guard | 4 段 (WRITE_GUARD_LABELS refuse / ALLOWED_DB_LABELS / fire.staging.db filename / FIRE_ENV='staging') |
| 衝突 | base_date=2026-05-13 × source_version=w60_f111_daily_v1 既存 0 件 |
| 分類 | **A**: staging-only write、API 不要、HQ marker 充足、4 段 guard PASS |

## §3 構成

| step | 内容 |
|---|---|
| 1 | 経路特定 + classification A/B/C/D 判定 |
| 2 | dry-run smoke (base_date=2026-05-13、source_version=w60_f111_daily_v1) |
| 3 | production/develop md5 baseline 確定 |
| 4 | staging write smoke (= 4 段 guard + FIRE_ENV=staging + HQ marker) |
| 5 | 新 signal verify (= row_count + top 8 比較) |
| 6 | D8 (2026-05-25) F111-real-batch chain 再 run |
| 7 | D6/D7/D8 candidate overlap 評価 |
| 8 | vault plan/results + HQ 1-block + 6 KPI |

## §4 安全境界

- staging DB write 許可: HQ_APPROVE_STAGING_RESEARCH_SIGNAL_REFRESH=1 + FIRE_ENV=staging
- production-develop DB 接続: **禁止** (= md5 不変確認)
- LINE / token / API / launchctl / plist / cron / 実発注 / 楽天 /
  iSPEED / Computer Use / workflow / --no-verify / git push / sudo /
  rm -rf / TODO Excel: **禁止**
- 既存 row (= 13695 件) 完全保護 (= inserted=35, replaced=0 で確認)

## §5 完了条件

- 経路 classification 完了 (= A 確定)
- dry-run smoke + staging write smoke 完走
- production/develop md5 不変
- staging row_count delta = +35 (= top_n=30 + sector cap 5)
- D8 chain 動作 + top candidates 出力
- D6/D7/D8 overlap 評価 + 効果限界の正直記述
- vault plan + results + HQ 1-block 報告 + 6 KPI

## §6 停止条件

- API/token/network 接続が必要 → 即停止
- production/develop md5 変化 → 即停止
- guard fail → 即停止
- 既存 base_date/source_version 衝突 → 即停止
