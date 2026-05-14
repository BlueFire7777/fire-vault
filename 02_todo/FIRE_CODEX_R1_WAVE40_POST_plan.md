---
id: FIRE-CODEX-R1-Wave-40-post
phase: 本番 v0 Launch / Phase C Foundation
priority: 高
status: 設計のみ (= 設計 doc / plist draft / no-send trial plan / duplicate prevention)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40-pre (= DATA-R3 設計 v1.0、HQ 承認済)
  - HQ 補足 (= Codex 余剰運用、第 2 陣追加投入許可)
  - F282 試走 5/16 02:00 待機 (= 干渉禁止)
  - F062-R5.8 (= 1 chunk send + send_guard + name_enrichment)
related:
  - 03_design/FIRE_production_v0_launch_plan_2026-05-13.md
  - 03_design/F286_DATA_R3_daily_refresh_launchd_2026-05-13.md
chapter: production-v0 / Phase C / F062 morning advisory
---

# FIRE-CODEX-R1 Wave 40-post plan — F062 Morning Advisory Launchd + No-Send Trial 設計

## 目的

Production v0 Phase C foundation として、F062 morning advisory launchd
+ no-send trial 設計を作成。月-金 08:45 JST、DATA-R3 完了 + freshness OK
を入力前提に preview / no-send 試走で 1 週間検証する Wave 46+ への基盤。
**本 wave は設計のみ**、実 plist 配置 / launchctl / DB write / API /
LINE / token 参照 0。

## /goal 完了条件 27 項目

1-10  設計 9 項目 + Wave 41 以降最短順 (= F062 設計 / 依存 / freshness /
      trial / send guard / duplicate / record-decisions / log / plist / 順)
11    F282 本番試走干渉 0
12-18 DB write / LINE / token / env 全体 / launchctl / plist / API 0
19    L4 audit CRITICAL 0 / HIGH 0
20    第 2 陣 safe-audit L7a-L7d 結果整理
21    4,090 PASS 維持
22    docs/vault/log 更新
23    6 KPI table 付き HQ 報告
24-26 lane 数選定理由 / 8 lane 不採用理由 / 第 2 陣採否理由
27    v0 寄与明記

## 12 lane 構成 (= 第 1 陣 8 + 第 2 陣 4 二段投入)

### 第 1 陣 8 lane

| lane | task_id | 担当 |
|---|---|---|
| L5  | 本線統合 | plan / 設計 doc / vault / HQ 報告 |
| L1a | F062 launchd architecture | Codex |
| L1b | DATA-R3/freshness/F282 dependency | Codex |
| L2a | no-send trial plan | Codex |
| L2b | duplicate prevention + record-decisions | Codex |
| L3  | plist XML draft | Codex |
| L4  | adversarial audit 8 観点 | Codex |
| L6  | regression + F282/DATA-R3 review | Codex |

### 第 2 陣 4 lane (= L4 GO 0 concerns 確認後)

| lane | task_id | 担当 |
|---|---|---|
| L7a | token/secret/LINE 経路 static audit | Codex |
| L7b | no-send trial audit | Codex |
| L7c | duplicate prevention audit | Codex |
| L7d | smoke plan 強化 | Codex |

### lane 数選定理由

- HQ 補足: Phase P2 = 8 lane 第一候補化 + Codex 余力時 第 2 陣追加投入
- 本 wave は分割可能な独立観点が第 1 陣 7 (= architecture / dependency / 
  trial / duplicate / draft / audit / regression) + 第 2 陣 4 安全 audit
- L4 = GO 0 concerns 確認後 第 2 陣 4 lane 全投入
- 同時最大 8 lane (= 第 1 陣 7 並列、第 2 陣 4 並列、二段投入)

### 第 2 陣 4 lane 選定理由

- L7a 採用: F062 は LINE 経路に最も近い、token 漏洩リスク最大、static audit 必須
- L7b 採用: no-send trial は v0 Phase D 直結、本 wave で audit
- L7c 採用: F286-PNL-R2 既存 + advisory_decisions schema との整合性確認
- L7d 採用: F062-R5.8 既存資産再利用方針の確実化

第 2 陣 8 候補中 4 採用、4 不採用:
- DB write / mtime risk audit → 本 wave DB 触らず不要
- adversarial audit 深掘り → L4 で十分カバー
- regression plan 強化 → L6 で十分
- docs / HQ report 改善 → 本線 (L5) でカバー

## 想定成果物

- `~/fire-vault/03_design/F062_morning_advisory_launchd_2026-05-13.md` (= 主成果)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_POST_plan.md` (= 本 file)
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE40_POST_results.md`
- `~/fire-vault/log.md` milestone 追記
- `/tmp/codex_wave40post/results/l{1a,1b,2a,2b,3,4,6,7a,7b,7c,7d}.txt` (= 11 lane stdout)

## 禁止事項

- 実 plist 配置 / launchctl load / unload
- DB write / LINE 送信 / token / secret / channel_token 参照
- env 全体読み取り / 実 API call
- F101 staging probe / F282 手動実行 / VACUUM INTO
- 本番 F282 plist 変更 / DATA-R3 plist 配置
- workflow / --no-verify / TODO Excel 更新
