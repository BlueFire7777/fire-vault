---
id: FIRE-CODEX-R1-WAVE60-PILOT-W4-plan
phase: 本番 v0 中核 / Wave 60-pilot-W4 / D20-D26 7 day recovery 集約 + demote policy 正式化
priority: 最高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-18 (W4 集約日 = D26 当日) / wave 実行: 2026-05-14
aggregation_range: D20 (2026-06-10) - D26 (2026-06-18) = 7 営業日
design_doc_in: ~/fire-vault/03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
design_doc_out: ~/fire-vault/03_design/FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md (= 本 wave で新規作成検討)
codex_mode: 4 lane factual-confirm (= 80 words/lane、self-audit)
---

# Wave 60-pilot-W4 Plan — Recovery Period Aggregation / Demote Policy Formalization v1.0

## §1 目的

D20-D26 7 営業日の GO_CONDITIONAL 連続維持を集約し、W3 設計 doc で導入した
HOLD criteria (= 5 連続 warning / demote 例外 / override review 必須) と
demote 運用 (= 340A0 / 3798 / 137A0 段階的拡張) の実運用妥当性を正式評価する。
D27 以降の運用方針 (= recently_seen 維持 / 7991/9130/331A0 連続許容 / 次の demote タイミング)
を確定し、次 wave 優先順位を決める。

## §2 安全境界

- 全 0 制約遵守: DB write 0 / production-develop 接続 0 / staging read-only /
  LINE 0 / token 0 / API 0 / launchctl 0 / plist 0 / cron 0 / 実発注 0 /
  楽天 0 / iSPEED 0 / Computer Use 0 / workflow 0 / --no-verify 0 /
  git push 0 / sudo 0 / rm -rf 0 / TODO Excel 0
- read-only 集約のみ、新 chain 実行なし

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline + D20-D26 全 artifact (= results / trade plan / review) 集約 |
| 2 | D20-D26 7 day summary 表 (= judgment / top / sector / recently_seen / review / paper PnL / caveat) |
| 3 | GO_CONDITIONAL 7 連続維持要因分析 (= 6 観点) |
| 4 | demote 効果評価 (= 340A0 / 3798 / 137A0 段階的、4 観点) |
| 5 | sector 多様化推移 (= 1 種 100% → 2 種 → 3 種) |
| 6 | HOLD criteria 機能性評価 (= W3 §3.3 5 条件 + W3 §7.1 9 条件) |
| 7 | review missing 構造分析 (= 累積 18 day blank、運用上の扱い検討) |
| 8 | paper PnL status 整理 (= 全 pending、h20 後 outcome 評価 timeline) |
| 9 | D27 運用方針 (= recently_seen 維持 / 連続許容 / 次の demote) |
| 10 | 次 Wave 優先順位 (= D27 / DATA-R3 / liquidity / paper PnL / features / launchd / LINE) |
| 11 | Codex 4 lane factual-confirm (= self-audit) |
| 12 | vault plan + results + demote policy design doc (任意) + HQ + 6 KPI |

## §4 完了条件

- D20-D26 集約完了
- 7 day summary 表完成
- GO_CONDITIONAL 維持要因分析完了 (= 6 観点)
- demote 効果評価完了 (= 3 銘柄 + 残効果)
- sector 多様化評価完了
- HOLD criteria 機能性評価完了
- review missing 整理完了
- paper PnL status 整理完了
- D27 運用方針完成
- 次 Wave 優先順位完成
- Codex 4 lane factual-confirm 実施
- 全 0 制約遵守
- F282 plist + 3 DB mtime/size/md5 不変
- vault plan + results + HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env /
API / LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED /
Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel

## §6 Codex 4 lane (= factual-confirm、80 words/lane、self-audit)

- **A**: D20-D26 judgment / top candidates / sector 推移 確認 (= 7 day 整合性)
- **B**: 340A0 / 3798 / 137A0 demote 効果 + 連続維持日数 確認
- **C**: review missing 7 day 全 blank / paper PnL 7 day 全 pending / outcome 制約 確認
- **D**: D27 方針 (= recently_seen 6 件維持 / 7991/9130/331A0 連続許容)
  + 次 wave 優先順位 確認

## §7 設計 doc 出力判断

W4 demote policy design doc (= `FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md`)
を新規作成するか:

- ✅ **作成推奨**: W3 改訂は HOLD criteria 中心、W4 では demote 運用そのものの
  正式化 (= 5 連続 warning → 6 連続 demote sim → 7 連続前 demote 実行 の三段階)、
  sector 多様化目標、D27 以降の運用方針を明文化する価値がある。
- 内容: §1 W4 集約成果 / §2 demote 三段階 policy / §3 sector 多様化目標 /
  §4 連続性 monitor 運用 / §5 D27 handoff / §6 W3 supersede 関係

## §8 W4 集約の特徴 (= W3 との差分)

| 観点 | W3 (= D14-D19) | W4 (= D20-D26) |
|---|---|---|
| 期間 | 6 営業日 | **7 営業日** |
| 主要 judgment | HOLD 連続 | **GO_CONDITIONAL 連続** |
| 主要 event | HOLD 初発動 + 340A0 demote 初 | **3798 demote 初 + 137A0 demote 本実行 + sector 3 種多様化** |
| design doc 主題 | HOLD criteria 改訂 | **demote policy 正式化** |
| Codex 4 lane | factual-confirm + reply 4/4 YES | **factual-confirm + self-audit** (= reply 不要) |
