---
id: FIRE-CODEX-R1-WAVE23-plan
phase: ガバナンス / Wave 23 / Phase P1 試験 / 6 lane Codex 実起動 minimal probe
priority: 高
status: 進行中 ☆ 6 lane Codex 実起動、minimal task、output dir = /tmp 配下のみ
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 22 (= 完了、cron thaw design only)
  - HQ Wave 23 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / R2 prompt template v1.0 実証
---

# Wave 23: 6 lane Codex 実起動 minimal probe (= R2 prompt template v1.0 実証)

R2 prompt template v1.0 (= Wave 21 確立) を **実際に Codex 6 lane 並列起動**
で実証する。各 lane の task は **超 minimal** (= /tmp 配下のみ書込み、fire /
fire-vault には書込まない)。完全 dummy / no-op、実 system 変更 0。

## Wave 23 sub-task (= 6 lane Codex + 本線)

| sub  | lane | owner | task | allowed_files |
|------|------|-------|------|---------------|
| W23-1| L5   | 本線  | Wave 23 plan + 6 prompt files | vault, /tmp/codex_wave23/prompts/ |
| W23-2| L1a  | Codex | minimal probe design 要約 | /tmp/codex_wave23/output/l1a_design.md |
| W23-3| L1b  | Codex | 代替設計 / 比較案 | /tmp/codex_wave23/output/l1b_design.md |
| W23-4| L2   | Codex | dummy test 1 件 (= trivial PASS) | /tmp/codex_wave23/output/l2_dummy_test.py |
| W23-5| L3   | Codex | dummy impl / no-op patch 案 | /tmp/codex_wave23/output/l3_noop_patch.md |
| W23-6| L4   | Codex | R2 設計 doc audit | /tmp/codex_wave23/output/l4_audit.md |
| W23-7| L6   | Codex | pytest 全 PASS 確認 report | /tmp/codex_wave23/output/l6_regression.md |
| W23-8| 本線  | 本線  | merge + audit + commit + HQ 報告 | vault, log.md |

## file ownership 表 (= 全 lane disjoint、衝突 0)

| file | owner_lane | merge_owner |
|---|---|---|
| /tmp/codex_wave23/output/l1a_design.md | L1a | 本線 (= vault に merge する場合のみ、本 wave では merge せず参照のみ) |
| /tmp/codex_wave23/output/l1b_design.md | L1b | 同上 |
| /tmp/codex_wave23/output/l2_dummy_test.py | L2 | 同上 |
| /tmp/codex_wave23/output/l3_noop_patch.md | L3 | 同上 |
| /tmp/codex_wave23/output/l4_audit.md | L4 | 同上 |
| /tmp/codex_wave23/output/l6_regression.md | L6 | 同上 |
| 02_todo/FIRE_CODEX_R1_WAVE23_plan.md | 本線 | 本線 |
| 02_todo/FIRE_CODEX_R1_WAVE23_results.md | 本線 | 本線 |
| log.md | 本線 | 本線 |

## 実起動 Codex CLI 形式

```
codex exec \
  --sandbox read-only \
  --skip-git-repo-check \
  --cd /Users/bluefire/fire \
  --output-last-message /tmp/codex_wave23/output/{lane}_last_message.txt \
  -m sonnet \
  < /tmp/codex_wave23/prompts/{lane}_prompt.txt
```

`--sandbox read-only` で **file 書込みは codex の output area のみ**。
fire / fire-vault は read-only。

各 lane に「output を `/tmp/codex_wave23/output/{lane}_*.md` に書く」と
指示するが、`read-only` sandbox では codex 自身は file 書き込みできないため、
**output は stdout / --output-last-message 経由で本線が受領**。本線が
/tmp/codex_wave23/output/ に転記する。

## 成功条件 (= HQ 指示)

- 6 lane 全完了
- file ownership 衝突 0
- CRITICAL 0
- safety violation 0
- 全 pytest PASS or 対象 test PASS + regression plan 明記
- wave 実時間 150 分以内
- commit 6 件以内
- Codex 直接 commit 0
- HQ 報告 1 ブロック

## 失敗時 → 後退

- 1 lane 失敗 → Step 1 後退 (= 5 lane)、根本原因分析
- 2 lane 以上失敗 → R1 5 lane に後退

## 禁止 (= HQ Wave 23 指示)

- 実 cron / launchd / crontab 登録
- plist 配置
- launchctl load
- logrotate 設定適用
- 実 LINE 送信
- 実 API call
- DB write
- token/secret/channel_token 参照
- workflow 変更
- --no-verify
- TODO Excel 更新
- F101 staging probe (= 別 HQ approve)

## Hard abort 条件 (= R2 template v1.0 継承)

- audit CRITICAL 1 件以上
- safety violation
- 未承認 LINE 送信
- token / secret leak
- file ownership 衝突 ≥ 2
- HQ 明示 abort

## 関連リンク

- [[../03_design/FIRE_CODEX_R2_codex_prompt_template_v1_2026-05-12|R2 prompt template v1.0]]
- [[../03_design/FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 設計本体]]
- [[../03_design/cron_thaw_design_2026-05-12|cron thaw design]]
- [[FIRE_CODEX_R1_WAVE22_results|Wave 22 results]]
