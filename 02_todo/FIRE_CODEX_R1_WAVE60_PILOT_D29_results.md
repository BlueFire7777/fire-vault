---
id: FIRE-CODEX-R1-WAVE60-PILOT-D29-results
phase: 本番 v0 中核 / Wave 60-pilot-D29 / 7991 rank 1 5 連続 warning + demote sim 完了
priority: 最高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-06-23 (D29) / wave 実行: 2026-05-15
pilot_day: D29
pilot_judgment: GO_CONDITIONAL maintained
hard_check_chain_level_passed: 11/12
top_continuity_5day: "7991/9130/331A0 D25-D29 5 連続安定"
sector_diversification_5day: "機械 + 運輸 + 情報通信 = 3 種維持 5 連続"
hold_2_avoidance: 完全継続 (= D15 以降 14 連続)
recurrence_warning_active: "7991 rank 1 5 連続 = 段階 1 warning 発動 ★"
demote_sim_completed: true
demote_sim_new_top: "9130 (運輸) → rank 1、sector 2 種一時縮退 (= 運輸+情報通信)"
d30_demote_recommendation: "★★★ D30 で 7991 demote 本実行強推奨 (= 案 B)"
---

# Wave 60-pilot-D29 Results — 7991 Recurrence Warning / Demote Simulation

## §1 結論

**D29 = GO_CONDITIONAL maintained 10 連続**、ただし **7991 rank 1 = 5 連続到達
= 段階 1 warning 発動** ★。demote sim 完了 = **新 top 1 = 9130 (運輸)、sector
一時 2 種縮退** path 確立。**D30 で 7991 demote 本実行強推奨** (= 案 B、
W4 demote policy §3 段階 3 厳守)。3 DB md5 全 不変、本 wave で code 変更 0。

## §2 chain 実行結果

### 2.1 D28 review 確認 (= 0/5 記入)

→ W3 §7.1 path 2 継続適用 (= 累積 21 day blank)

### 2.2 D28 paper PnL --review-md 再 run

- candidates=20 / evaluated=0 / review_missing=1
- output: /tmp/fire_d29_prep/d28_paper_pnl_reload.json + .md

### 2.3 D29 chain (= 現行 recently_seen 6 件)

- F111: candidates=20 / eligible=19 / demoted_count=6
- paper PnL: candidates=20 / evaluated=0 / review_missing=1
- artifact_source: f062_preview

## §3 D29 top 3 (= D25-D29 5 連続安定 ✓)

| rank | code | name | sector | close | risk_yen |
|---|---|---|---|---|---|
| 1 | **7991** | マミヤ・オーピー | **機械** | 1,177 | 5,885 |
| 2 | **9130** | 共栄タンカー | **運輸・物流** | 1,410 | 7,050 |
| 3 | **331A0** | メディックス | **情報通信** | 482 | 2,410 |

## §4 ⚠ 7991 5 連続 warning 段階 1 発動 (= 主要発見) ★★★

| day | 7991 rank 1 連続 | HOLD #2 まで残日数 | 段階 |
|---|---|---|---|
| D25-D28 | 1-4 連続 | 6-3 日 | 監視 |
| **D29** | **5 連続** | **2 日** | **warning 段階 1 ✓** |
| D30 (予想) | 6 連続 | 1 日 | demote 本実行 (= 案 B) |
| D31 (予想) | 7 連続 | 0 日 | **HOLD #2 再発動 リスク** |

## §5 7991 demote sim 結果 (= 参考 read-only chain)

### 5.1 sim 条件

```
recently_seen_codes: 8747, 5729, 3489, 340A0, 3798, 137A0, **7991** (= 追加)
demoted_count: 7
```

### 5.2 sim 後の新 top 5

| rank | code | name | sector | close | risk_yen |
|---|---|---|---|---|---|
| **1** | **9130** | 共栄タンカー | **運輸・物流** (= 新 #1) | 1,410 | 7,050 |
| **2** | **331A0** | メディックス | **情報通信** (= 新 #2) | 482 | 2,410 |
| 3 | 4389 | プロパティデータバンク | 情報通信 | 863 | 4,315 |
| 4 | 9247 | ＴＲＥホールディングス | 情報通信 | 1,612 | 8,060 |
| 5 | 4317 | レイ | 情報通信 | 508 | 2,540 |

### 5.3 sector 多様化への影響 (= ⚠ 一時縮退)

| 観点 | D29 現行 (= 3 種) | D30 7991 demote 後 (= sim) |
|---|---|---|
| rank 1 | 機械 (7991) | 運輸 (9130) |
| rank 2 | 運輸 (9130) | 情報通信 (331A0) |
| rank 3 | 情報通信 (331A0) | 情報通信 (4389) |
| sector 種数 | **3 種** | **2 種** (= 機械喪失) |

→ **7991 demote で sector は 3 種 → 2 種へ一時縮退**、ただし HOLD #2 阻止が優先

## §6 D30 demote 判断 (= ★★★ 案 B 強推奨)

| 案 | 内容 | 推奨度 |
|---|---|---|
| A. D29 内即 demote | warning と同時に本実行 | ★★ (= 跳ばし、policy 非準拠) |
| **B. D30 で本実行** | 段階 3 (= 6 連続前) 正式 demote | **★★★ 推奨** (W4 policy 厳守) |
| C. D31 まで delay | HOLD #2 発動懸念 | ★ |

→ 案 B (= D30 で 7991 demote 本実行) 推奨

## §7 D29 hard check (= chain-level 11/12 ✓)

| # | 項目 | 結果 |
|---|---|---|
| 1-11 | chain-level invariants | 全 ✓ |
| 12 | actual / liquidity / event | 朝確認待ち |

## §8 W3 recovery criteria 9 条件判定 (= 9/9 ✓)

→ **GO_CONDITIONAL maintained 10 連続** ✓

## §9 D20-D29 10 day post-recovery 集約

| day | judgment | top 1 | 7991 rank 1 連続 | 状態 |
|---|---|---|---|---|
| D20-D24 | 復帰 〜 5 連続 | 137A0 | n/a | 137A0 監視 → demote 直前 |
| D25 | 6 連続 (137A0 demote) | **7991 (新 #1)** | 1 (初) | 3 種多様化達成 |
| D26 | 7 連続 | 7991 | 2 | 3 種維持 |
| D27 | 8 連続 | 7991 | 3 | 3 種維持 |
| D28 | 9 連続 | 7991 | 4 | 3 種維持 |
| **D29** | **10 連続** | **7991** | **5 (warning ★)** | **3 種維持 + sim 完了** |

## §10 demote 効果総括 (= D29 時点)

| 銘柄 | demote 開始 | D29 連続 | 効果 |
|---|---|---|---|
| 8747/5729/3489 | 〜D1 | 24 連続 | ✅ 完全定着 |
| 340A0 | D16 | 14 連続 | ✅ HOLD #2 解消 |
| 3798 | D20 | 10 連続 | ✅ HOLD #2 未然防止 |
| 137A0 | D25 | 5 連続 | ✅ HOLD #2 完全回避 |
| **(候補) 7991** | **(D30 想定)** | **(D29 sim 完了)** | **D30 本実行強推奨** |

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
| git tracked file | 変更 0 (= vault 4 file + demote sim artifact のみ) |
| git push | 0 |

## §12 vault docs (= 本 wave 追加)

- `~/fire-vault/04_daily/2026-06-23_manual_live_pilot_trade_plan.md`
- `~/fire-vault/04_daily/2026-06-23_manual_live_pilot_review.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D29_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D29_results.md`
- `/tmp/fire_d29_prep/demote_sim/` (= sim 参考 artifact)

## §13 W5 集約引き継ぎ材料 (= D20-D33 14 day)

1. **340A0/3798/137A0/(7991 想定) demote 効果 14 day evaluation**
2. **7991 demote 三段階 policy 実証** (= D24 137A0 sim → D25 本実行 → D29 7991 sim → D30 7991 本実行)
3. **9130 rank 1 昇格後の継続性**
4. **sector 多様化 3 種 → 2 種一時縮退期間**
5. **HOLD 完全回避継続 (= D15 から 14+ 連続)**
6. **review missing 累積 21+ day = 簡易 1 行 review 導入再検討**
7. **paper PnL 真の outcome** (= 7/8-7/22)
8. **sector warning policy 正式化** (= W4 demote policy §4.2 草案ベース)
9. **liquidity filter 強化必要性**
10. **DATA-R3 active job 優先度**

## §14 next action 候補

### 14.1 Fujiwara 本人

1. D29 朝 (= 2026-06-23 08:30 JST) J-Quants refresh + iSPEED actual + entry 判断
2. D29 推奨: **7991 マミヤ・オーピー** (= 機械、5 連続 warning #1、D30 demote 直前)
3. D29 review.md 必須 5 項目記入
4. D29 場後 paper PnL handoff

### 14.2 後続 wave

5. **W60-pilot-D30** ★★★ (= 2026-06-24 水、**7991 demote 本実行**、新 top = 9130/331A0/4389)
6. **W60-pilot-D31** (= 2026-06-25 木、post-7991-demote 1 連続確認)
7. **W60-pilot-W5 集約** (= D20-D33 後)
8. **DATA-R3 active job wave**
9. **paper_pnl preview h1/h20 再 run** (= 2026-07-08〜7/22 頃)
10. **liquidity filter 強化 wave**
11. **smoke plist 本番投入 wave**

## §15 6 KPI

1. 稼働率: 100% (= 10 step 全完走、blocker 0)
2. 短縮率: 高 (= D28 review + paper PnL + D29 chain + 5 連続確認 + demote sim +
   D30 判断 + 4 段階判定 + trade plan + review + W5 引き継ぎ + vault を
   1 conversation で完結)
3. 採用率: 100% (= demote sim 完了、新 top path 確立、D30 demote 本実行強推奨確定、
   GO_CONDITIONAL maintained 10 連続)
4. 差戻率: 0% (= chain 1 発成功、sim 想定通り)
5. Integrator 負荷: 低 (= 3 DB 全 md5 不変、code 変更 0、vault 4 file + sim artifact)
6. 安全事故: 0 (= DB write 0 / API 0 / token 0 / launchctl 0 / 実発注 0 / git push 0)
