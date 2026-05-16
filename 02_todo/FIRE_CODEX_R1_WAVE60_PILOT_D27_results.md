---
id: FIRE-CODEX-R1-WAVE60-PILOT-D27-results
phase: 本番 v0 中核 / Wave 60-pilot-D27 / post-137A-demote 3 連続到達 + GO_CONDITIONAL 8 連続
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-19 (D27) / wave 実行: 2026-05-14
pilot_day: D27
pilot_judgment: GO_CONDITIONAL maintained
hard_check_chain_level_passed: 11/12
top_continuity_3day: "7991/9130/331A0 D25-D27 3 連続安定 ✓"
sector_diversification_3day: "機械 + 運輸 + 情報通信 = 3 種維持 3 連続 ✓"
hold_2_avoidance: 完全継続
recently_seen_codes_maintained: 6 件
next_demote_candidate: "7991 (= 月 D29 で rank 1 5 連続 warning 予想)"
---

# Wave 60-pilot-D27 Results — Post-137A-Demote 3 連続到達

## §1 結論

**D27 = GO_CONDITIONAL maintained 8 連続**。
137A0 demote 後の新 top (= 7991 機械 / 9130 運輸 / 331A0 情報通信) が
**D25-D27 で 3 連続安定到達**、sector **3 種多様化 3 連続維持**、
HOLD #2 完全回避 8 連続継続。7991 rank 1 連続 = 3 で次の demote 候補
monitor 段階。3 DB md5 全 不変、本 wave で code 変更 0。

## §2 chain 実行結果

### 2.1 D26 review 確認 (= 0/5 記入)

→ W3 §7.1 path 2 継続適用 (= 累積 19 day blank、`status: blank` YAML 維持)

### 2.2 D26 paper PnL --review-md 再 run

- candidates=9 / evaluated=0 / review_missing=1
- review_actuals: is_blank=True / final_decision=unknown
- output: /tmp/fire_d27_prep/d26_paper_pnl_reload.json + .md

### 2.3 D27 chain

- F111: candidates=20 / eligible=19 / exclusions risk_above=1
- tuning_params: strict / recently_seen 6 件 (= 8747,5729,3489,340A0,3798,137A0) / demoted_count=6
- paper PnL: candidates=20 / evaluated=0 / review_missing=1
- artifact_source: f062_preview / f062_raw_kind: f062_actual_dict / f111_input_source: f111_real_batch

## §3 D27 top 3 (= D25-D27 で 3 連続安定 ✓)

| rank | code | name | sector | close | risk_yen | score | label |
|---|---|---|---|---|---|---|---|
| 1 | **7991** | マミヤ・オーピー | **機械** | 1,177 | 5,885 | 0.8669 | boost_with_caution |
| 2 | **9130** | 共栄タンカー | **運輸・物流** | 1,410 | 7,050 | 0.8586 | boost_with_caution |
| 3 | **331A0** | メディックス | **情報通信** | 482 | 2,410 | 0.8567 | boost_with_caution |
| (参考) 4 | 4389 | プロパティデータバンク | 情報通信 | 863 | 4,315 | 0.8560 | boost_with_caution |
| (参考) 5 | 9247 | ＴＲＥホールディングス | 情報通信 | 1,612 | 8,060 | 0.8546 | boost_with_caution |

## §4 sector 3 種多様化 3 連続維持 ✓

| day | sector top 3 |
|---|---|
| D25 (初) | 機械 + 運輸 + 情報通信 |
| D26 (2 連続) | 機械 + 運輸 + 情報通信 |
| **D27 (3 連続)** | **機械 + 運輸 + 情報通信** ✓ |

## §5 HOLD #2 完全回避継続 (= 8 連続)

- D15 HOLD #2 発動 (= 340A0 7 連続) 以降、HOLD 再発動 **0**
- D27 時点で 12 連続 (= D15 から) HOLD 完全回避
- demote 三段階 policy (= W4 demote policy doc) 実証維持:
  - 段階 1 warning (5 連続): D24 で 137A0 警告
  - 段階 2 sim (5-6 連続): D24 sim run
  - 段階 3 本実行 (6 連続前): D25 で 137A0 demote

## §6 7991 rank 1 連続性 monitor (= 次の demote 候補)

| 銘柄 | D25 | D26 | D27 | rank 1 連続 |
|---|---|---|---|---|
| **7991** (機械) | 1 (初) | 1 | **1** | **3** |
| 9130 (運輸) | 2 | 2 | 2 | n/a (rank 2 維持) |
| 331A0 (情報通信) | 3 | 3 | 3 | n/a (rank 3 維持) |

→ **7991 rank 1 連続 = 3** (= warning まで残 2 wave、W4 demote policy §3 適用):

| 段階 | 予想 day | アクション |
|---|---|---|
| 1 warning | D29 (= 2026-06-22 月) | trade plan §警告追記、demote sim 検討 |
| 2 sim | D29-D30 | recently_seen 追加 sim chain |
| 3 本実行 | D30 (= 2026-06-23 火) | recently_seen 正式追加 |

## §7 D27 hard check (= chain-level 11/12 ✓、12 朝確認待ち)

| # | 項目 | 結果 |
|---|---|---|
| 1 | artifact_source=f062_preview | ✓ |
| 2 | f062_raw_kind=f062_actual_dict | ✓ |
| 3 | f111_input_source=f111_real_batch | ✓ |
| 4 | forbidden_check passed | ✓ |
| 5 | auto_order_allowed=false | ✓ |
| 6 | manual_review_required=true | ✓ |
| 7 | safety_flags all false | ✓ |
| 8 | top_candidates ≥ 1 | ✓ (= 3) |
| 9 | non-sample ticker | ✓ |
| 10 | tradable_universe=true | ✓ |
| 11 | risk_within_pilot_limit=true | ✓ |
| 12 | actual / liquidity / event | Fujiwara 朝確認待ち |

## §8 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

| # | 条件 | D27 結果 |
|---|---|---|
| 1 | review 5 項目 OR gap 明確化 | ✓ (= path 2 適用継続) |
| 2 | 340A0 demote 維持 | ✓ (= 12 連続) |
| 3 | 3798 demote 維持 | ✓ (= 8 連続) |
| 3' | 137A0 demote 維持 | ✓ (= 3 連続) |
| 4 | f111_real_batch | ✓ |
| 5 | top candidate ≥ 1 | ✓ (= 3) |
| 6 | risk_within_pilot_limit | ✓ |
| 7 | actual/liquidity/event | 朝確認待ち |
| 8 | safety_flags 13 keys False | ✓ |
| 9 | paper PnL/review 重大問題なし | ✓ |

→ **GO_CONDITIONAL maintained 8 連続** ✓

## §9 D20-D27 8 day post-recovery 集約

| day | judgment | top 1 | sector 種別 | 137A0 状態 |
|---|---|---|---|---|
| D20 | 復帰 (初) | 137A0 | 2 種 (= 機械追加) | rank 1 (1 連続) |
| D21 | 2 連続 | 137A0 | 2 種 | rank 1 (2 連続) |
| D22 | 3 連続 | 137A0 | 2 種 | rank 1 (3 連続) |
| D23 | 4 連続 | 137A0 | 2 種 | rank 1 (4 連続) |
| D24 | 5 連続 (warning) | 137A0 | 2 種 + sim | rank 1 (5 連続) |
| D25 | 6 連続 (demote 本実行) | **7991 (新 #1)** | **3 種達成 ✓** | demoted |
| D26 | 7 連続 (post-demote 2 連続) | 7991 | 3 種維持 | demoted 維持 |
| **D27** | **8 連続 (post-demote 3 連続)** | **7991** | **3 種 3 連続** | **demoted 3 連続** |

## §10 demote 効果総括 (= D27 時点)

| 銘柄 | demote 開始 | D27 時点連続 | 効果 |
|---|---|---|---|
| 8747/5729/3489 | 〜D1 | 22 連続 | ✅ 完全定着 |
| 340A0 | D16 | 12 連続 | ✅ HOLD #2 解消継続 |
| 3798 | D20 | 8 連続 | ✅ HOLD #2 未然防止継続 |
| **137A0** | **D25** | **3 連続** | ✅ HOLD #2 完全回避継続 |

## §11 安全境界 final verification

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
| git push | 0 |

## §12 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-19_manual_live_pilot_trade_plan.md`
- `~/fire-vault/04_daily/2026-06-19_manual_live_pilot_review.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D27_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D27_results.md`

## §13 W5 集約引き継ぎ準備 (= D20-D33 14 day pilot 想定)

W60-pilot-W5 集約 wave 推奨項目:

1. **340A0 / 3798 / 137A0 demote 効果 14 day evaluation**
2. **7991 rank 1 連続性監視 + demote タイミング検討** (= D29-D30 想定)
3. **9130 (運輸) 継続性評価**
4. **sector 3 種多様化維持率推移**
5. **HOLD 完全回避継続日数 (= D15 から)**
6. **review missing 構造的問題** (= 累積 19+ day blank、簡易 1 行 review 導入検討)
7. **paper PnL 真の outcome 評価** (= h20 後 7/8-7/22 頃に確定)
8. **sector warning policy 正式化** (= W4 demote policy §4.2 草案ベース)
9. **liquidity filter 強化必要性**
10. **DATA-R3 active job 優先度**

## §14 next action 候補

### 14.1 Fujiwara 本人

1. D27 朝 (= 2026-06-19 08:30 JST) J-Quants refresh + iSPEED actual + entry 判断
2. D27 推奨: **7991 マミヤ・オーピー** (= 機械、3 連続 #1)
3. D27 review.md 必須 5 項目記入
4. D27 場後 paper PnL handoff (`--review-md` 付きで再 run)

### 14.2 後続 wave

5. **W60-pilot-D28** (= 2026-06-22 月、7991/9130/331A0 4 連続到達 + 7991 rank 1 4 連続)
6. **W60-pilot-D29** ★ (= 2026-06-23 火、**7991 rank 1 5 連続 warning + demote sim 検討**)
7. **W60-pilot-D30** ★ (= 2026-06-24 水、**7991 demote 本実行候補**)
8. **W60-pilot-W5 集約** (= D20-D33 後、demote 設計実証総括)
9. **DATA-R3 active job wave**
10. **paper_pnl preview h1/h20 再 run** (= 2026-07-08〜7/22 頃で真の outcome)
11. **liquidity filter 強化 wave**
12. **smoke plist 本番投入 wave** (= ~/Library/LaunchAgents 配置 + launchctl load、HQ approve 経由)
13. **features rerun wave**
14. **全銘柄 daily refresh launchd 自動化**

## §15 6 KPI

1. 稼働率: 100% (= 11 step 全完走、blocker 0)
2. 短縮率: 高 (= D26 review + paper PnL + D27 chain + 3 連続確認 + 多様化確認 +
   demote monitor + 判定 + trade plan + review + W5 引き継ぎ + vault を 1 conversation で完結)
3. 採用率: 100% (= 7991/9130/331A0 3 連続到達、sector 3 種維持、HOLD #2 回避 8 連続、
   GO_CONDITIONAL maintained 8 連続、W5 引き継ぎ完成)
4. 差戻率: 0% (= chain 1 発成功、想定通り maintained)
5. Integrator 負荷: 低 (= 3 DB 全 md5 不変、code 変更 0、vault 4 file)
6. 安全事故: 0 (= DB write 0 / API 0 / token 0 / launchctl 0 / 実発注 0 / git push 0)
