---
id: FIRE-CODEX-R1-WAVE37-plan
phase: ガバナンス / Wave 37 / F286-AFTER-R1 staging coverage smoke
priority: 高
status: 進行中 ☆ /goal モード / 8 lane (本線 1 + Codex 7) / staging + develop 比較
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 36 (= 完了、AFTER-R1 production read-only smoke、production データ薄い発見)
  - HQ Wave 37 起票承認 + HQ_APPROVE_AFTER_R1_READ=1 (= 2026-05-13)
chapter: ガバナンス / R-01-08 / F286-AFTER-R1 / staging coverage smoke
---

# Wave 37: F286-AFTER-R1 staging coverage smoke

W36 で production fire.db に AFTER-R1 入力データ (= advisory_decisions /
paper_pnl / research_signals) **が薄い** ことを発見。本 wave で
**staging.db / develop.db** を read-only で確認し、AFTER-R1 が staging
主軸で評価可能か検証する。

**F282 試走 (= 5/16 土曜 02:00 待機中) 干渉 0** 継続。

## lane 数選定理由

**8 lane 第一候補採用** (= HQ 補足方針通り)。本 wave は 7 sub に
自然分割可能 (= W36 と同じ pattern):

- L1a staging smoke design
- L1b production vs staging vs develop 比較設計
- L2a smoke test plan
- L2b artifact validation (= staging/, develop/, comparison.md)
- L3 smoke command extension
- L4 audit
- L6 regression / F282 interference

**8 lane 不採用ケース**: 該当なし、HQ 補足方針通り採用。

## baseline (= W37 開始時、必須確認)

| 項目 | 状態 |
|---|---|
| F282 launchctl LastExitStatus | 0 維持 (= 干渉 0 baseline) |
| /data/fire.db | 371,081,216 / 5/12 16:17:24 |
| /data/fire.develop.db | 371,081,216 / 5/12 16:11:43 |
| /data/fire.staging.db | 4,804,063,232 / 5/12 18:45:22 |
| reports/after_r1/smoke/2026-05-13/ | W36 artifact 3 file 存在 |
| 現在 | 2026-05-13 水曜 00:28 JST |
| F282 次 run | 2026-05-16 土曜 02:00 JST |

## artifact 命名空間 (= W36 と分離、上書き 0)

```
reports/after_r1/smoke/2026-05-13/                  ← W36 既存
├── summary.json                                    (W36、不変)
├── coverage_details.json                           (W36、不変)
├── report.md                                       (W36、不変)
├── staging/                                        ← W37 新規
│   ├── summary.json
│   ├── coverage_details.json
│   └── report.md
├── develop/                                        ← W37 新規
│   ├── summary.json
│   ├── coverage_details.json
│   └── report.md
└── comparison.md                                   ← W37 新規 (3 環境 比較)
```

全 W37 artifact は **O_EXCL 'x'** で atomic create、W36 file には触らない。

## Wave 37 sub-task (= 8 lane = 本線 1 + Codex 7)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W37-1| L5   | 本線  | plan + baseline + 7 Codex prompt              |
| W37-2| L1a  | Codex | staging smoke design                          |
| W37-3| L1b  | Codex | 3 環境比較設計 (= production vs staging vs develop) |
| W37-4| L2a  | Codex | smoke test plan                               |
| W37-5| L2b  | Codex | artifact validation (= 3 環境 + comparison)    |
| W37-6| L3   | Codex | smoke command extension (= W36 script を 3 環境対応) |
| W37-7| L4   | Codex | adversarial audit (= 7 観点)                  |
| W37-8| L6   | Codex | regression / F282 interference                |
| W37-9| 本線 | 本線  | smoke 実行 (3 環境) + artifact + comparison + 報告 |

## /goal 完了条件 (= HQ Wave 37 明示)

- staging coverage 確認完了
- smoke row filter 方針確認 (= w17-3-smoke + W18 market_prices_daily 5/8)
- DB mtime unchanged (= 3 環境全)
- F282 試走干渉 0
- LINE 送信 0
- token / secret / channel_token 参照 0
- DB write 0
- L4 audit CRITICAL 0 / HIGH 0
- 6 KPI table 付き HQ 報告

## file lock 表

### 既存 / read-only

| file | 状態 | 区分 |
|---|---|---|
| reports/after_r1/smoke/2026-05-13/{summary,coverage,report}.* | NEW(W36) | **read-only 維持** |
| ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist | NEW(W33) | read-only |
| data/snapshot/fire.staging.db / fire.develop.db | NEW(W30) | read-only |

### 本 wave 書込み

| file | 状態 | owner | 区分 |
|---|---|---|---|
| reports/after_r1/smoke/2026-05-13/staging/* | NEW (3 file) | 本線 (L3) | allowed (O_EXCL) |
| reports/after_r1/smoke/2026-05-13/develop/* | NEW (3 file) | 本線 | allowed (O_EXCL) |
| reports/after_r1/smoke/2026-05-13/comparison.md | NEW | 本線 | allowed (O_EXCL) |
| 02_todo/FIRE_CODEX_R1_WAVE37_plan.md | NEW | 本線 | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE37_results.md | NEW | 本線 | allowed |
| log.md | MOD | 本線 | allowed |

fire develop: commit 0 (= artifact のみ、git untracked 想定)。

## 関連リンク

- [[FIRE_CODEX_R1_WAVE36_results|Wave 36 results (= production smoke)]]
- [[../03_design/F286_AFTER_R1_night_paper_live_batch_2026-05-12|F286-AFTER-R1 設計 v1.0]]
- [[../03_design/F282_weekly_snapshot_trial_monitoring_2026-05-12|F282 試走監視]]
