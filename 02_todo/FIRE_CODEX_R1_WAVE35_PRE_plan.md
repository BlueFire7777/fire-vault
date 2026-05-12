---
id: FIRE-CODEX-R1-WAVE35-PRE-plan
phase: ガバナンス / Wave 35-pre / F286-AFTER-R1 設計のみ
priority: 高
status: 進行中 ☆ /goal モード / 8 lane (本線 1 + Codex 7) / 設計 doc 1 件
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 34 (= 完了、F282 試走監視テンプレ + cleanup 設計)
  - HQ Wave 35-pre 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F286-AFTER-R1
---

# Wave 35-pre: F286-AFTER-R1 / After-Close Night Paper Live Batch Runner 設計

F282 1 週間試走 (= 5/16-5/19 待機中) **に一切干渉せず**、F286-AFTER-R1 の
設計を進める。コード実装 / DB write / cron 登録 / LINE 送信 は 0。

## lane 数選定理由 (= HQ 補足方針 2026-05-12)

**8 lane 第一候補採用** (= 分割可能な設計 task)。HQ 補足方針:
「分割可能な実装・test・audit・docs がある場合は 8 lane 投入を第一候補」。

本 wave の task は次の 7 sub に自然分割可能:
- architecture (L1a)
- data flow (L1b)
- test plan (L2a)
- smoke plan (L2b)
- runner interface (L3)
- audit (L4)
- regression / integration impact (L6)

各 sub は disjoint な領域なので 8 lane = 本線 1 + Codex 7 が最適。
逐次実行だと本線 4-6 時間想定 → 8 lane で 40-60 分に短縮見込み。

**8 lane 不採用なら**: 設計だけなら 4-5 lane でも完了可、ただし本線負荷
集中 + 並列効果薄。今回は HQ 補足方針通り 8 lane 第一候補で実施。

## Wave 35-pre sub-task (= 8 lane、本線 1 + Codex 7)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W35p-1| L5  | 本線  | plan + baseline + 7 Codex prompt + doc 統合   |
| W35p-2| L1a | Codex | AFTER-R1 overall architecture                 |
| W35p-3| L1b | Codex | data flow / dependencies / TODO 連携          |
| W35p-4| L2a | Codex | test plan                                     |
| W35p-5| L2b | Codex | smoke plan (= read-only + staging-only)       |
| W35p-6| L3  | Codex | runner interface / CLI design                 |
| W35p-7| L4  | Codex | adversarial audit (= 8 観点、HIGH 0)         |
| W35p-8| L6  | Codex | regression / integration impact review        |
| W35p-9| 本線 | 本線  | doc 統合 + 15 条件 + 6 KPI + lane 理由 + 報告 |

## /goal 完了条件 15 件

1. AFTER-R1 全体設計完了 (= § 1 architecture)
2. 入力 / 出力データ設計整理 (= § 2 + § 3)
3. runner interface 案 (= § 4)
4. Paper Live / Replay / Simulation / Lane 評価責務分解 (= § 1)
5. ML feature 前段設計 (= § 7)
6. smoke plan (= § 8)
7. safety 設計 (= § 9)
8. 関連 TODO 関係整理 (= § 10)
9. F282 試走干渉 0
10. DB write 0
11. LINE 送信 0
12. token / secret / channel_token 参照 0
13. cron / launchd 登録変更 0
14. docs / vault / log 更新
15. 6 KPI + lane 理由 HQ 報告

## /goal 停止条件 13 件 (= 監視)

- DB write 必要 / LINE 送信 / token / cron 変更 / F282 手動実行 /
  VACUUM INTO / launchctl / production apply / audit CRITICAL /
  safety violation / file 衝突 / 150 分 / HQ abort
→ 全 trigger 想定なし (= 設計のみ)

## 必須設計項目 10 件 (= F286_AFTER_R1 設計 doc に集約)

1. AFTER-R1 責務定義
2. 入力データ
3. 出力データ
4. runner 設計
5. 実行タイミング
6. Lane / Pattern 評価設計
7. ML 前段 feature
8. smoke plan
9. safety 設計
10. TODO / phase 連携

## file lock 表

### 既存 modified / 範囲外 / read-only

| file | 状態 | 区分 |
|---|---|---|
| scripts/seed_pattern_layer1.py | M | forbidden |
| simulation/research_lane/historical_indicators.py | M | forbidden |
| .claude/ | ?? | 範囲外 |
| data/fire.staging.db.pre_restore_* | ?? | 範囲外 |
| data/snapshot/fire.staging.db | ?? | W30 retention 継続 |
| data/snapshot/fire.develop.db | ?? | W30 retention 継続 |
| ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist | NEW (W33) | read-only |

### 本 wave 書込み (= vault のみ、本線)

| file | 状態 | owner | 区分 |
|---|---|---|---|
| 02_todo/FIRE_CODEX_R1_WAVE35_PRE_plan.md | NEW | 本線 (L5) | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE35_PRE_results.md | NEW | 本線 | allowed |
| 03_design/F286_AFTER_R1_night_paper_live_batch_2026-05-12.md | NEW | 本線 | allowed |
| log.md | MOD | 本線 | allowed |

fire develop: commit 0 (= 設計のみ)。

## 関連リンク

- [[../03_design/F282_weekly_snapshot_trial_monitoring_2026-05-12|F282 試走監視 (W34)]]
- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[../03_design/F285_Research_Lane_requirements_and_spec_2026-05-08|F285 Research Lane]]
- F286 PNL R1/R2/R3 / REPORT R1 / DATA R3 (= 既存実装)
