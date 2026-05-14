---
id: FIRE-CODEX-R1-Wave-40.6
phase: 本番 v0 Launch / D-Day Cutover / Rollback / Token Runbook
priority: 高
status: Runbook 設計のみ
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40-pre / 40-post / 40.5 (= Phase B-C-D-E foundation 完成)
  - HQ Wave 40.6 起票指示
related:
  - 03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md
chapter: production-v0 / D-Day / cutover / rollback / token
---

# FIRE-CODEX-R1 Wave 40.6 plan — Production v0 Cutover/Rollback/Token Runbook

## 目的

D-Day 想定に向けて、本番開始直前に詰まりやすい 6 高優先度リスク
(= rollback / token / env / cutover / migration / GO-NO-GO) を設計段階で解消。
**Runbook のみ**、実行系変更 0。

## /goal 完了条件 30 項目

(= HQ 起票文書通り、設計成果物 12 項目 + 安全 0 項目 13 + pytest 維持 + 報告 3)

## 5 lane 構成 (= L5 本線 + Codex 4 lane、8 lane 不採用)

| lane | task_id | 担当 |
|---|---|---|
| L5 (本線) | 統合 + 15 章 doc + vault | plan / 設計 doc / vault / log / HQ 報告 |
| Lane A | D-Day rollback runbook | Codex |
| Lane B | LINE token / env setup runbook | Codex |
| Lane C | no-send → production send cutover | Codex |
| Lane D | record-decisions staging→production migration | Codex |

### 8 lane 不採用理由

1. F282 本番試走 5/16 直前のため干渉リスクと注意分散を回避
2. Runbook 中心成果物、過剰分割で Integrator 負荷増
3. DB/LINE/token/plist/launchctl に触らない設計 wave、4 lane 十分
4. Wave 40-pre/post で 8/11 lane 構成実証済

### 第 2 陣 Codex 不採用理由

- 本 wave は Runbook 中心、DB/LINE/token 経路に直接触らない設計のみで第 2 陣
  4 安全 audit (= token 経路 / no-send / duplicate / smoke) の適用範囲は
  Wave 40-post で網羅済
- 軽量範囲維持のため第 2 陣未投入

## 想定成果物

- `~/fire-vault/03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14.md`
  (= 15 章、主成果)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_6_plan.md` (= 本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_6_results.md`
- `~/fire-vault/log.md` milestone 追記
- `/tmp/codex_wave40_6/results/lane_{a,b,c,d}.txt`

## 禁止事項

実 file 変更 (= 設計 doc + plan/results + log.md 以外 全 0) / plist 配置 /
launchctl / cron / launchd / crontab / DB write / LINE / token /
channel_token / secret / env 全体読出 / 実 API / F101 staging probe /
F282 手動実行 / VACUUM INTO / 本番 F282 plist 変更 / workflow /
--no-verify / TODO Excel / 楽天 / 自動発注 / Computer Use
