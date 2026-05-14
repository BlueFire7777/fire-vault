---
id: FIRE-CODEX-R1-WAVE60-integration-plan
phase: 本番 v0 中核 / Wave 60-integration / F062 ↔ AFTER-R1 連携
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60-integration Plan — F062 Morning Batch ↔ AFTER-R1 Morning Material Integration v1.0

## §1 目的

F062 朝 batch の preview/payload/summary と DATA-R3 freshness report を
AFTER-R1 MVP に接続し、**synthetic fixture ではなく実 batch artifact 由来** で
4 種類成果物 (paper_live_ledger / good_candidate_ranking /
pattern_candidate_report / morning_line_material) を生成できる状態にする。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + F062 actual output format 確認 |
| 2 | MVP module に artifact_source 推定 + 4 output 反映 |
| 3 | scaffold CLI に `--artifact-source` (auto/synthetic/f062/unknown) 追加 |
| 4 | DATA-R3 freshness OK 強制 enforce 明確化 |
| 5 | malformed row + all rows invalid → NO-GO warning |
| 6 | trade_plan template + D1 doc に artifact_source 表示 |
| 7 | D1-D5 日付/曜日修正 (= 5/14 木 D1 / 5/15 金 D2 / skip 5/16 土 5/17 日 / 5/18 月 D3 / 5/19 火 D4 / 5/20 水 D5) |
| 8 | regression tests 15+ 件追加 |
| 9 | Codex 6 lane stdin smoke (= 5-10 秒 sleep 改善案、W60.5/W60.6 並列改善) |
| 10 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- DB write / sqlite / LINE / token / API / launchctl / plist / cron / VACUUM 全 0
- 実発注 / 楽天 / iSPEED / Computer Use 全 0
- ファイル write は scripts/jobs/* + tests/ + fire-vault/* に限定
- F282 / 3 DB / W30 snapshot 完全不変

## §4 完了条件

- F062 actual format (= dict with chunks/selected_count/etc) 対応
- artifact_source 4 output に反映
- `--artifact-source` CLI option 動作
- DATA-R3 OK 必須化 / FAILED safe handling
- malformed row skip + all rows invalid NO-GO warning
- tests 全 PASS、pytest collected の変化を報告
- D1-D5 日付修正済
- F282 plist + 3 DB 不変
- Codex 6 lane smoke 実施 (= 改善案)
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

実発注 / 楽天 / iSPEED / DB write / LINE / token / API call / launchctl /
plist / cron / VACUUM / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
