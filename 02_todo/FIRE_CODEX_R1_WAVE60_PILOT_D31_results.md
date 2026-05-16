---
id: FIRE-CODEX-R1-WAVE60-PILOT-D31-results
phase: 本番 v0 中核 / Wave 60-pilot-D31 / post-7991-demote 2 連続 + GO_CONDITIONAL 12 連続
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-25 (D31) / wave 実行: 2026-05-15
pilot_day: D31
pilot_judgment: GO_CONDITIONAL maintained
hard_check_chain_level_passed: 11/12
top_continuity_2day: "9130/331A0/4389 D30-D31 2 連続安定 ✓"
sector_diversification: "2 種維持 2 連続 (運輸+情報通信、機械喪失継続)"
hold_2_avoidance: 完全継続 (= D15 以降 16 連続)
recently_seen_codes: ["8747", "5729", "3489", "340A0", "3798", "137A0", "7991"]
demoted_count: 7
recurrence_monitor_9130: "rank 1 = 2 連続、D34 (= 5 連続) warning 予想"
goal_status: PASS
codex_lanes: 4 / 4 YES (= A 一致 / B 9130 連続 / C 7991 demote 維持 / D sector 2 種 / E HOLD #2 16 連続)
---

# Wave 60-pilot-D31 Results — Post-7991-Demote Continuity / 9130 Monitor

## §1 結論

**D31 = GO_CONDITIONAL maintained 12 連続**。D30 で 7991 demote 本実行 →
D31 で post-demote 2 連続到達、新 top 5 (= 9130/331A0/4389/9247/4317) が
**完全に同一順序で安定**、7991 caution 維持、sector 2 種維持 2 連続、
HOLD #2 完全回避 **16 連続継続**。9130 rank 1 = 2 連続で次の demote 候補
monitor 段階に入る (= D34 warning 想定)。Codex 4 lane factual-confirm 全 YES、
CRITICAL/HIGH 0。3 DB md5 / F282 plist 不変、code 変更 0。
**/goal 13 項目全 PASS**。

## §2 chain 実行結果

### 2.1 D30 review 確認 (= 0/5 記入)

→ W3 §7.1 path 2 継続適用 (= 累積 23 day blank、status: blank YAML 維持)

### 2.2 D30 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- output: /tmp/fire_d31_prep/d30_paper_pnl_reload.json + .md

### 2.3 D31 chain (= recently_seen 7 件維持)

- F111: candidates=20 / eligible=19 / exclusions risk_above=1
- tuning_params: strict / recently_seen 7 件 / demoted_count=7 (= 維持)
- paper PnL: candidates=20 / evaluated=0 / review_missing=1
- artifact_source: f062_preview

## §3 D31 top 5 (= D30 と完全一致、2 連続安定 ✓)

| rank | code | name | sector | close | risk_yen | score | label |
|---|---|---|---|---|---|---|---|
| 1 | **9130** | 共栄タンカー | **運輸・物流** | 1,410 | 7,050 | 0.8586 | boost_with_caution |
| 2 | **331A0** | メディックス | **情報通信** | 482 | 2,410 | 0.8567 | boost_with_caution |
| 3 | **4389** | プロパティデータバンク | **情報通信** | 863 | 4,315 | 0.8560 | boost_with_caution |
| 4 (参考) | 9247 | ＴＲＥホールディングス | 情報通信 | 1,612 | 8,060 | 0.8546 | boost_with_caution |
| 5 (参考) | 4317 | レイ | 情報通信 | 508 | 2,540 | 0.8515 | boost_with_caution |

→ D30 top 5 と **100% 一致** (= Codex Lane A YES)

## §4 7991 demote 効果維持確認 ✓ (= Codex Lane C YES)

- D31 F111: 7991 research_advisory_label = **caution** ✓
- F111 内 rank: 7 (= top 5 完全除外)
- W3 §3.3 #2-exclusion (= demote 済み連続カウント停止) 機能維持
- AFTER-R1 top 候補から除外維持

## §5 9130 rank 1 連続性 monitor (= 次の demote 候補) ★

| day | 9130 rank | 連続 |
|---|---|---|
| D30 (初昇格) | 1 | 1 |
| **D31** | **1** | **2** |
| D32 (予想) | 1 | 3 |
| D33 (予想) | 1 | 4 |
| **D34 (予想)** | **1** | **5 = warning ★** |
| D35 (予想) | demoted | (= 段階 3 本実行) |

→ Codex Lane B YES (= 9130 rank 1 = 2 連続到達)
→ 三段階 policy: D34 warning + sim、D35 本実行 想定

## §6 sector diversification (= 2 種維持 2 連続) ⚠

| 観点 | D29 (= 3 種維持期) | D30 (= 7991 demote) | **D31 (= 本日)** |
|---|---|---|---|
| rank 1 | 機械 (7991) | 運輸 (9130) | **運輸 (9130)** |
| rank 2 | 運輸 (9130) | 情報通信 (331A0) | **情報通信 (331A0)** |
| rank 3 | 情報通信 (331A0) | 情報通信 (4389) | **情報通信 (4389)** |
| sector 種数 | 3 種 | 2 種 | **2 種** (= 機械喪失 2 連続) |

→ Codex Lane D YES (= sector 2 種維持、3 種未回復)
→ 機械 sector 候補登場まで縮退継続

## §7 HOLD #2 完全回避継続 (= D15 以降 16 連続) ✓ (= Codex Lane E YES)

- D15 HOLD #2 発動 (= 340A0 7 連続) 以降、HOLD 再発動 0
- D31 時点で **16 連続** HOLD 完全回避
- 9130 rank 1 = 2 連続 (= HOLD #2 7 連続まで残 5 wave)
- W4 demote policy §3 三段階で D34 sim → D35 本実行 が次の節目

## §8 Codex 4 lane factual-confirm (= 4/4 YES、CRITICAL 0)

| lane | 観点 | reply |
|---|---|---|
| **A** | D30 vs D31 top 5 + sector 構成 一致 | **YES** ✓ |
| **B** | 9130 rank 1 = 2 連続到達 | **YES** ✓ |
| **C** | 7991 demote 効果維持 (= caution、top 除外) | **YES** ✓ |
| **D** | sector 2 種維持、機械喪失継続 | **YES** ✓ |
| **E** | HOLD #2 完全回避 16 連続 | **YES** ✓ |

→ **5 lane 全 YES** (= 当初予定 4 lane を超過実施)、CRITICAL/HIGH 0

## §9 D31 hard check (= chain-level 11/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-11 | chain-level invariants | 全 ✓ |
| 12 | actual / liquidity / event | Fujiwara 朝確認待ち |

## §10 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

→ **GO_CONDITIONAL maintained 12 連続** ✓

## §11 D20-D31 12 day post-recovery 集約

| day | judgment | top 1 | sector 種数 | 主要 event |
|---|---|---|---|---|
| D20-D24 | 復帰 〜 5 連続 | 137A0 | 2 種 | 137A0 monitor → demote 直前 |
| D25 | 6 連続 | 7991 (新) | **3 種達成 ✓** | 137A0 demote 本実行 |
| D26-D28 | 7-9 連続 | 7991 | 3 種維持 | 7991 連続 monitor |
| D29 | 10 連続 | 7991 | 3 種維持 | 7991 5 連続 warning + sim |
| D30 | 11 連続 | 9130 (新) | **2 種** | **7991 demote 本実行 ★** |
| **D31** | **12 連続** | **9130** | **2 種** (2 連続) | **post-demote 2 連続安定** |

## §12 demote 効果総括 (= D31 時点)

| 銘柄 | demote 開始 | D31 連続 | 効果 |
|---|---|---|---|
| 8747/5729/3489 | 〜D1 | 26 連続 | ✅ 完全定着 |
| 340A0 | D16 | 16 連続 | ✅ HOLD #2 解消継続 |
| 3798 | D20 | 12 連続 | ✅ HOLD #2 未然防止 |
| 137A0 | D25 | 7 連続 | ✅ HOLD #2 完全回避 |
| **7991** | **D30** | **2 連続** | ✅ HOLD #2 完全回避継続 |

## §13 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → 不変 ✓ |
| F282 plist | size=1772 / mtime=1778593597 → 不変 ✓ |
| DB write / API / token | 全 0 ✓ |
| LINE / launchctl / plist / cron | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault 4 file 追加のみ) |
| git push / commit / --no-verify | 全 0 ✓ |
| workflow 変更 | 0 ✓ |

## §14 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-25_manual_live_pilot_trade_plan.md`
- `~/fire-vault/04_daily/2026-06-25_manual_live_pilot_review.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_results.md`

## §15 D32 / D33 / W5 引き継ぎ材料

### 15.1 D32 (= 2026-06-26 金) 確認事項
- 9130 rank 1 = 3 連続到達確認
- 331A0 / 4389 rank 上位継続
- sector 2 種維持 vs 機械 sector 再登場
- 7991 caution 継続

### 15.2 D33 (= 2026-06-29 月) 確認事項
- 9130 rank 1 = 4 連続到達確認
- W5 集約直前 day

### 15.3 D34 (= 2026-06-30 火) 想定
- **9130 rank 1 = 5 連続 warning 段階 1 発動**
- demote sim 検討開始

### 15.4 D35 (= 2026-07-01 水) 想定
- **9130 demote 本実行候補** (= W4 demote policy §3 段階 3)

### 15.5 W5 集約材料 (= D20-D33 14 day)
1. 340A0/3798/137A0/7991 demote 効果 14 day evaluation
2. 三段階 policy 完全実証 (= 137A0 + 7991 各々で warning → sim → 本実行)
3. 9130 demote タイミング検証 (= D34-D35)
4. sector 多様化 2 種 vs 3 種期間
5. HOLD 完全回避継続 (= D15 以降 16+ 連続)
6. review missing 累積 23+ day = 簡易 1 行 review 導入再検討
7. paper PnL 真の outcome (= 7/8-7/23)
8. sector warning policy 正式化 (= W4 §4.2 草案ベース)
9. liquidity filter 強化必要性
10. DATA-R3 active job 優先度

## §16 next action 候補 (= 優先順)

### 16.1 Fujiwara 本人
1. D31 朝 (= 2026-06-25 08:30 JST) J-Quants refresh + iSPEED actual + entry 判断
2. D31 推奨: 9130 共栄タンカー (= 運輸、2 連続 #1)
3. D31 review.md 必須 5 項目記入
4. D31 場後 paper PnL handoff

### 16.2 後続 wave
5. **W60-pilot-D32** (= 2026-06-26 金、9130 3 連続到達)
6. **W60-pilot-D33** (= 2026-06-29 月、9130 4 連続到達 + W5 集約直前)
7. **W60-pilot-W5 集約** (= D20-D33 14 day 総括)
8. **W60-pilot-D34** ★★★ (= 2026-06-30 火、9130 5 連続 warning + sim)
9. **W60-pilot-D35** ★★★ (= 2026-07-01 水、9130 demote 本実行候補)
10. **smoke plist 本番投入** / **DATA-R3 active** / **liquidity 強化** / **features rerun**
11. **paper PnL h1/h20 outcome 評価** (= 2026-07-08〜7/23 頃)

## §17 /goal 13 項目 PASS verification

| # | 条件 | 結果 |
|---|---|---|
| 1 | baseline 完全一致 (3 DB + F282) | ✓ PASS |
| 2 | D30 review status 明示確認 (= blank) | ✓ PASS |
| 3 | D30 paper PnL --review-md 再 run 完了 | ✓ PASS (review_missing=1) |
| 4 | D31 chain 実行 (= recently_seen 7 件、demoted_count=7) | ✓ PASS |
| 5 | 9130 rank 1 = 2 連続 | ✓ PASS (Codex Lane B YES) |
| 6 | 7991 caution 維持 | ✓ PASS (Codex Lane C YES) |
| 7 | D31 top 3 + D30 差分明示 | ✓ PASS (完全一致確認、Lane A YES) |
| 8 | sector 2 種 vs 3 種明示 | ✓ PASS (2 種維持、Lane D YES) |
| 9 | HOLD #2 完全回避 16 連続判定 | ✓ PASS (Lane E YES) |
| 10 | 4 段階判定 | ✓ PASS (= GO_CONDITIONAL maintained 12 連続) |
| 11 | vault 4 file 作成 | ✓ PASS |
| 12 | 安全境界 0 違反 | ✓ PASS |
| 13 | Codex 4 lane PASS + D32/D33/W5 引き継ぎ + 6 KPI | ✓ PASS (= 5 lane 全 YES) |

→ **/goal PASS** ✓ (= 13/13 全項目達成)

## §18 6 KPI

1. 稼働率: 100% (= 11 step + Codex 5 lane 全完走、blocker 0)
2. 短縮率: 高 (= D30 review + paper PnL + D31 chain + 連続性確認 + 7991 維持 +
   sector + HOLD #2 判定 + Codex 5 lane + 判定 + trade plan + review +
   W5 引き継ぎ + vault を 1 conversation で完結)
3. 採用率: 100% (= post-demote 2 連続安定、9130 rank 1 = 2 連続、7991 caution 維持、
   sector 2 種維持、HOLD #2 回避 16 連続、Codex 5 lane 全 YES、/goal 13/13 PASS)
4. 差戻率: 0% (= chain 1 発成功、D30 vs D31 完全一致)
5. Integrator 負荷: 低 (= 3 DB md5 不変、F282 plist 不変、code 変更 0、vault 4 file)
6. 安全事故: 0 (= DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0 /
   実発注 0 / git push 0 / commit 0 / --no-verify 0)
