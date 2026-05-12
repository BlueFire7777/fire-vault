---
id: FIRE-CODEX-R1-WAVE36-plan
phase: ガバナンス / Wave 36 / F286-AFTER-R1 read-only smoke
priority: 高
status: 進行中 ☆ /goal モード / 8 lane (本線 1 + Codex 7) / F282 試走干渉 0
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 35-pre (= 完了、F286-AFTER-R1 設計 v1.0)
  - HQ Wave 36 起票承認 + HQ_APPROVE_AFTER_R1_READ=1 (= 2026-05-13)
chapter: ガバナンス / R-01-08 / F286-AFTER-R1 / read-only smoke
---

# Wave 36: F286-AFTER-R1 read-only smoke 実施

W35-pre で確定した F286-AFTER-R1 設計 v1.0 に基づき、**production /
develop / staging を read-only で確認** し、AFTER-R1 が参照する入力
データの存在 / 件数 / 欠損 / filter 方針 / artifact 出力を検証する。

**F282 試走 (= 5/16 土曜 02:00 待機中) に一切干渉しない**。

## lane 数選定理由 (= HQ 必須追加項目)

**8 lane 第一候補採用** (= HQ 補足方針 2026-05-12)。本 wave は次の 7 sub
に自然分割可能:

- L1a read-only smoke design (= 接続方式 / query 設計)
- L1b data coverage / filter design (= 件数閾値 / W17 smoke 除外)
- L2a smoke test plan (= test 観点)
- L2b artifact validation plan (= json + markdown 検証)
- L3 smoke command / runner interface (= 一次 CLI 案)
- L4 adversarial audit (= 7 観点)
- L6 regression / F282 interference (= 4,090 PASS + launchctl 不変)

各 sub disjoint、本線が read-only smoke 実行 + artifact 出力を並走。

**8 lane 不採用ケース**: 該当なし。HQ 補足方針通り 8 lane 採用。

## 配置前 baseline (= 必須確認 #1 + #2 取得済)

### F282 launchd 状態 (= 干渉 0 baseline、read-only 確認)

- launchctl list: jp.fire.weekly-snapshot PID=- LastExitStatus 0 維持
- launchctl list jp.fire.weekly-snapshot: Label / LastExitStatus 0 ✓
- 次 run: 2026-05-16 土曜 02:00 JST (= 約 3 日 2 時間後)

### 既存 DB mtime + size baseline

| file | size | mtime |
|---|---|---|
| /data/fire.db | 371,081,216 | 5/12 16:17:24 |
| /data/fire.develop.db | 371,081,216 | 5/12 16:11:43 |
| /data/fire.staging.db | 4,804,063,232 | 5/12 18:45:22 |

実行後 全 3 件 **完全 unchanged** 必須 (= 完了条件 #3)。

### reports/after_r1/ 状態

未作成 (= 本 wave で `reports/after_r1/smoke/2026-05-13/` を作成)。

### 現在日時

2026-05-13 水曜 00:05 JST。

## Wave 36 sub-task (= 8 lane、本線 1 + Codex 7)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W36-1| L5   | 本線  | plan + baseline + 7 Codex prompt              |
| W36-2| L1a  | Codex | read-only smoke design                        |
| W36-3| L1b  | Codex | data coverage / filter design                 |
| W36-4| L2a  | Codex | smoke test plan                               |
| W36-5| L2b  | Codex | artifact validation plan                      |
| W36-6| L3   | Codex | smoke command / runner interface              |
| W36-7| L4   | Codex | adversarial audit (= 7 観点)                  |
| W36-8| L6   | Codex | regression / F282 interference                |
| W36-9| 本線 | 本線  | **read-only smoke 実行 + artifact 出力 + 報告** |

## 実行内容 (= W36-9 本線)

### Step 1: reports/after_r1/smoke/2026-05-13/ 作成

```bash
mkdir -p /Users/bluefire/fire/reports/after_r1/smoke/2026-05-13/
```

### Step 2: production fire.db read-only smoke query

各 table の row count + filter 確認 (= URI mode=ro + PRAGMA query_only=ON):

- advisory_decisions: COUNT(*)
- advisory_snapshots: COUNT(*)
- advisory_snapshot_rows: COUNT(*)
- paper_pnl: COUNT(*) + WHERE paper_pnl IS NOT NULL count
- paper_pnl: COUNT(*) WHERE paper_reason IS NOT NULL
- market_prices_daily: MAX(date) + COUNT(*) per recent date
- research_watchlist_signals: GROUP BY source_version
- W17 smoke filter (= source_version='w17-3-smoke') 該当 count

### Step 3: artifact 出力

```
reports/after_r1/smoke/2026-05-13/
├── summary.json
├── report.md
└── coverage_details.json
```

atomic create (= O_EXCL or unlink → write)、既存 file 上書き禁止。

### Step 4: post-smoke 検証

- 全 production DB mtime + size unchanged
- launchctl list 状態 unchanged
- reports/after_r1/smoke/2026-05-13/ 配下に 3 file 作成確認

## /goal 完了条件 14 件

1. AFTER-R1 read-only smoke 設計完了 (= W36-2 + § 1)
2. production/develop/staging read-only coverage 確認完了
3. DB mtime before/after unchanged 確認
4. F282 試走干渉 0 確認
5. smoke row filter 確認 (= w17-3-smoke 除外)
6. artifact 出力 (= reports/after_r1/smoke/ 配下)
7. LINE 送信 0
8. token/secret/channel_token 参照 0
9. cron/launchd 登録変更 0
10. DB write 0
11. L4 audit CRITICAL 0 / HIGH 0
12. 4,090 PASS 維持
13. docs/vault/log 更新
14. 6 KPI table 付き HQ 報告

## /goal 停止条件 13 件 監視

DB write 必要 / LINE / token / cron 変更 / F282 手動 / VACUUM / launchctl /
production apply / artifact 既存 file 上書き / audit CRITICAL / safety
violation / file 衝突 / 150 分 / HQ abort

## file lock 表

### 既存 modified / 範囲外

| file | 状態 | 区分 |
|---|---|---|
| scripts/seed_pattern_layer1.py | M | forbidden |
| simulation/research_lane/historical_indicators.py | M | forbidden |
| .claude/ | ?? | 範囲外 |
| data/fire.staging.db.pre_restore_* | ?? | 範囲外 |
| data/snapshot/fire.staging.db | ?? | W30 retention 継続 |
| data/snapshot/fire.develop.db | ?? | W30 retention 継続 |
| ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist | NEW(W33) | read-only |

### 本 wave 書込み

| file | 状態 | owner | 区分 |
|---|---|---|---|
| /Users/bluefire/fire/reports/after_r1/smoke/2026-05-13/summary.json | NEW | 本線 (L3) | allowed (= HQ approve 下) |
| /Users/bluefire/fire/reports/after_r1/smoke/2026-05-13/report.md | NEW | 本線 | allowed |
| /Users/bluefire/fire/reports/after_r1/smoke/2026-05-13/coverage_details.json | NEW | 本線 | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE36_plan.md | NEW | 本線 (L5) | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE36_results.md | NEW | 本線 | allowed |
| log.md | MOD | 本線 | allowed |

fire develop: reports/after_r1/ は git untracked or commit (= ignore policy
要確認、本 wave では commit せず untracked のまま)。

## 関連リンク

- [[../03_design/F286_AFTER_R1_night_paper_live_batch_2026-05-12|F286-AFTER-R1 設計 v1.0 (W35-pre)]]
- [[../03_design/F282_weekly_snapshot_trial_monitoring_2026-05-12|F282 試走監視]]
- [[FIRE_CODEX_R1_WAVE35_PRE_results|Wave 35-pre results]]
