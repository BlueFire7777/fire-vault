---
id: FIRE-CODEX-R1-WAVE60-impl-plan
phase: 本番 v0 中核 / Wave 60-impl / Paper Live MVP & Pattern Candidate
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60-impl Plan — F286-AFTER-R1 Paper Live MVP / Pattern Candidate Implementation v1.0

## §1 目的

FIRE 本番 v0 の **中核** として、夜間に走る Paper Live MVP を実装する。
F062 推奨銘柄 preview / DATA-R3 freshness / Ops Summary / readiness を
read-only で取り込み、`reports/after_r1/` 配下に 4 種類の成果物を生成する:

1. `paper_live_ledger_<base_date>.{json,md}`
2. `good_candidate_ranking_<base_date>.{json,md}`
3. `pattern_candidate_report_<base_date>.{json,md}`
4. `morning_line_material_<base_date>.{json,md}`

これにより、翌朝 F062/LINE 通知に渡せる「買い候補 / 理由 / 当たりパターン候補 /
手動売買目安」を生成する。

## §2 v0 中核への繰り上げ (= HQ 方針変更)

旧 W59-pre 設計の L1-L5 段階構造で AFTER-R1 を v0 後拡張扱いしていたが、
HQ 方針変更により AFTER-R1 MVP を **v0 中核** に繰り上げ。

旧 (W59-pre):
- AFTER-R1 = v0 後拡張 (= D-Day 後の L1-L5 段階実装)

新 (W60-impl):
- AFTER-R1 MVP = v0 中核 (= 「夜間に Paper Live MVP が回り、当たりパターン候補
  抽出し、朝 LINE で材料を渡せる」状態を即時実現)
- Paper-Live Real (L3+) は引き続き将来 wave、変更なし

## §3 構成 (= 8 lane 第一候補)

| Lane | 内容 |
|---|---|
| A | input resolver 実装 (= F062 / DATA-R3 / Ops Summary / readiness read-only) |
| B | paper_live_ledger schema / 生成 |
| C | good_candidate_ranking scoring |
| D | pattern_candidate_report 抽出 |
| E | morning_line_material 生成 + forbidden_phrase check |
| F | pytest 設計 + 実装 (25+ tests) |
| G | safety AST + output guard 監査 |
| H | vault docs / v0 再定義反映 |

実装は本線で完了、Codex audit は別 lane (= W60-impl では本線完結を目指す)。

## §4 実施項目

1. baseline + 本番不変確認
2. F062 / DATA-R3 / Ops Summary 入力 schema 確認
3. 設計 doc 作成 (= `~/fire-vault/03_design/F286_AFTER_R1_paper_live_mvp_2026-05-14.md`)
4. MVP module 新規追加 (= `scripts/jobs/_after_r1_mvp.py`)
5. 既存 scaffold 拡張 (= `scripts/jobs/run_f286_after_r1_night_batch.py` に
   `--mode mvp` + `morning-material` task + CLI 引数追加)
6. tests 拡張 (= `tests/scripts/jobs/test_run_f286_after_r1_night_batch.py` に
   46+ MVP tests 追加)
7. synthetic fixture smoke run (= /tmp/fire_w60_smoke/ で 4 出力確認)
8. pytest 全体 PASS 確認
9. F282 / 3 DB 不変確認
10. vault plan / results 起票

## §5 安全境界 (= 全 0 維持)

DB write / DB sqlite 接続 (production/develop/staging) / LINE 実送信 /
line-bot-sdk import / token / channel_token / secret / .env / env 全体参照 /
API call (requests / urllib / aiohttp) / launchctl / plist / cron / VACUUM /
Paper Live Real 実行 / brokerage / 自動発注 / Computer Use / Playwright /
workflow / --no-verify / git push / sudo / rm -rf / TODO Excel 更新 全て **0**。

## §6 完了条件

- Paper Live MVP 実装完了 (= 4 種類生成可能)
- `--mode mvp --task all` 動作確認
- F062 preview を入力にした smoke run 成功
- output 4 種類が `reports/after_r1/` または tmpdir に生成
- tests 全 PASS (= 既存 + 新規 MVP)
- pytest collected の変化を報告
- 既存 v0 path 変更 0 (= F282/DATA-R3/F062/readiness/Ops/wrapper 不触)
- F282 plist mtime/size/next run 不変
- 3 DB mtime/size 不変
- safety_flags 12+ keys 全 false 保証
- auto_order_allowed=False / manual_review_required=True 保証
- vault plan + results + HQ 1 ブロック + 6 KPI

## §7 停止条件

以下を検知 → 即停止 + HQ:
- DB write / DB sqlite 接続が必要
- LINE 送信 / token / .env / env 全体参照が必要
- API call / launchctl / plist / cron 変更が必要
- Paper Live Real 実行 / brokerage / 自動発注 / Computer Use が必要
- VACUUM / workflow / --no-verify / git push / sudo / rm -rf が必要
- TODO Excel 更新が必要
