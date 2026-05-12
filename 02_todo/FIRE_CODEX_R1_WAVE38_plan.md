---
id: FIRE-CODEX-R1-WAVE38-plan
phase: ガバナンス / Wave 38 / 本番 v0 Launch Plan 起票
priority: 最高 (= 本番 v0 最優先)
status: 進行中 ☆ /goal モード / 選択肢 B 採用 / 8 lane (本線 1 + Codex 7)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 37 (= 完了、AFTER-R1 staging coverage smoke)
  - HQ Wave 38 起票承認 + 本番 v0 最優先方針 (= 2026-05-13)
chapter: ガバナンス / R-01-08 / 本番 v0 / Production Launch
---

# Wave 38: Production v0 Launch Plan 起票 (= 選択肢 B)

## ★ 判断: 選択肢 B (Production v0 Launch Plan 起票) 採用

### 採用理由 (= HQ「本番 v0 優先」方針照らし)

1. **AFTER-R1 入力データ不足** (= W36/W37 で発覚)
   - staging.advisory_decisions: 10 row のみ
   - paper_pnl: 全 3 環境で table 不在
   - 10 row では win rate / sample size / pattern 抽出が成立しない
   - AFTER-R1 MVP 実装しても「意味ある統計」を出せない

2. **AFTER-R1 は本番 v0 後の拡張**
   - HQ 明示「本番 v0 後に回すもの」に AFTER-R1 が含まれる
   - 本番 v0 必須要素 (= DATA / freshness / Advisory / LINE / record-decisions /
     F282 / log / 失敗停止 / no-send) と AFTER-R1 は独立

3. **HQ 明示「迷う場合は v0 優先」**
   - HQ 優先順位: 1. v0 Launch Plan、2. v0 を遅らせない範囲の AFTER-R1 MVP、
     3. AFTER-R1 本格は v0 後
   - 「迷うなら 1」を明示採用

4. **ROI 比較**
   - AFTER-R1 MVP impl: 100-150 分本線負荷、データ不足で価値半減
   - v0 Launch Plan: 残タスク棚卸し + 依存整理 + 最短 Wave 順 → 即時 v0 加速

→ **選択肢 B 採用、Wave 38 で Production v0 Launch Plan を起票**

### 選択肢 A 不採用理由

AFTER-R1 MVP は本番 v0 を **間接的に遅らせる risk**:
- 本線 100-150 分占有
- Codex 7 lane 占有
- F282 試走中の並走として実施可能だが、v0 タスク並走の方が高 ROI

## lane 数選定理由 (= HQ 必須追加項目)

**8 lane 第一候補採用** (= HQ 補足方針)。本 wave は 7 sub に自然分割可能:

- L1a 残タスク棚卸し (= DATA / F062 / PNL / F282 / LINE 系)
- L1b 依存整理 (= upstream / downstream relation)
- L2a no-send 試走計画
- L2b v0 safety check plan
- L3 v0 までの最短 Wave 順
- L4 audit
- L6 regression / F282 干渉

**8 lane 不採用ケース**: 該当なし。HQ 補足方針通り採用。

## baseline

- F282: launchctl LastExitStatus 0 維持 (= 干渉 0 baseline)
- 3 環境 DB mtime + size unchanged (= W37 後と同じ)
- 現在: 2026-05-13 水曜 00:44 JST
- F282 次 run: 2026-05-16 土曜 02:00 JST

## Wave 38 sub-task (= 8 lane = 本線 1 + Codex 7)

| sub  | lane | owner | task                                        |
|------|------|-------|---------------------------------------------|
| W38-1| L5   | 本線  | plan + 選択肢 B 採用根拠 + 7 Codex prompt    |
| W38-2| L1a  | Codex | 本番 v0 残タスク棚卸し                       |
| W38-3| L1b  | Codex | 依存整理 (= DATA-R3 / F062 / PNL / F282 / LINE) |
| W38-4| L2a  | Codex | no-send 試走計画                             |
| W38-5| L2b  | Codex | v0 safety check plan                         |
| W38-6| L3   | Codex | v0 までの最短 Wave 順                        |
| W38-7| L4   | Codex | adversarial audit (= 7 観点)                  |
| W38-8| L6   | Codex | regression / F282 干渉確認                    |
| W38-9| 本線 | 本線  | Launch Plan doc 統合 + 19 条件 + 6 KPI + 報告 |

## 作成 vault doc

- 02_todo/FIRE_CODEX_R1_WAVE38_plan.md (本 doc)
- 02_todo/FIRE_CODEX_R1_WAVE38_results.md (= 後)
- **03_design/FIRE_production_v0_launch_plan_2026-05-13.md (= 主成果物)**
- log.md milestone

## /goal 19 条件達成計画

選択肢 B 採用:
1. ✓ 判断済 (= 本 doc)
2. ✓ 判断理由明記 (= 本 doc § 採用理由)
3-6. **該当なし** (= 選択肢 B 採用)
7. ✓ v0 Launch Plan doc (= 残タスク + 依存 + 最短 Wave 順)
8. ✓ test plan 明記 (= 設計 wave、test code 0)
9-13. 全 0 (= 設計のみ)
14. ✓ L4 audit CRITICAL/HIGH 0
15. ✓ 4,090 PASS 維持
16. ✓ docs/vault/log 更新
17. ✓ 6 KPI table 付き HQ 報告
18-19. ✓ lane 数理由明記 (= 上記 § lane 数選定理由)

## 関連リンク

- [[FIRE_CODEX_R1_WAVE37_results|Wave 37 results (= AFTER-R1 staging smoke)]]
- [[FIRE_CODEX_R1_WAVE36_results|Wave 36 results (= AFTER-R1 production smoke)]]
- [[../03_design/F282_weekly_snapshot_trial_monitoring_2026-05-12|F282 試走監視]]
- [[../03_design/F286_AFTER_R1_night_paper_live_batch_2026-05-12|F286-AFTER-R1 設計 v1.0 (= v0 後拡張)]]
