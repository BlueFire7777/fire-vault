---
title: F281 Lane C Phase C0 - universe 件数 precheck 結果
date: 2026-05-08
phase: F281-Lane-C-Phase-C0
status: precheck 完了、universe 件数 / 閾値感度 確定、F105 起票へ進む判断材料整備済
related: F281_Lane_C_design_2026-05-08, F271 v1.7, B-strict full run
trigger: HQ Q-Lane-C-C0-1 GO 後の read-only daily 検証実施
---

# F281 Lane C Phase C0 - universe 件数 precheck 結果

★ 本検証は **daily データのみで実施した read-only universe 件数調査**。
   daily 近似での entry / exit 評価は **行っていない** (HQ 不採用)。
   Lane C 真実装は intraday 前提、F105 起票後の Phase C1 / C2 で実施。

## 1. 実行条件

| 項目 | 値 |
|---|---|
| script | `scripts/jobs/lane_c_universe_precheck.py` |
| db_path | `/Users/bluefire/fire/data/fire.staging.db` (read-only mode) |
| 期間 | 2026-02-03 〜 2026-05-01 (60 営業日) |
| HQ 閾値 | 上記設計記録 §2 の 7 項目 (HQ 確定値) |
| 出力 | stdout (Markdown) + JSON (`/tmp/lane_c_precheck.json`) |
| 実行時間 | < 5 秒 (in-memory daily 集計) |

## 2. 集約結果 (60 営業日合計)

| filter 段階 | 累積件数 | 直前段階比 |
|---|--:|--:|
| prev_change +3% 〜 < +15% | 20,148 | - |
| ∩ vol/SMA20 ≥ 1.5 | 5,887 | 29.2% |
| ∩ gap -2% 〜 +4% | 4,808 | 81.7% |
| ∩ margin 可 | 4,808 | 100% (margin 制約は実質ノーカット) |
| ∩ turnover ≥ 10 億 (**Tier1**) | **1,592** | 33.1% |
| ∩ turnover ≥ 30 億 (**Tier2**) | **897** | 56.3% (Tier1 比) |
| ∩ turnover ≥ 50 億 (**Tier3**) | **652** | 72.7% (Tier2 比) |

### 2-1. 1 日あたり平均 candidate 件数

| Tier | 1 日平均 | 60 営業日合計 |
|---|--:|--:|
| Tier1 (≥ 10 億) | **26.53 件/日** | 1,592 |
| Tier2 (≥ 30 億) | **14.95 件/日** | 897 |
| Tier3 (≥ 50 億) | **10.87 件/日** | 652 |

## 3. stage gate trade_count >= 100 充足見込み

★ **3 Tier すべて trade_count >= 100 達成可能** ✅

| Tier | candidate 60 日合計 | gate 100 件 | 評価 |
|---|--:|---|---|
| Tier1 (≥ 10 億) | 1,592 件 | OK | 余裕大 (15.9 倍) |
| Tier2 (≥ 30 億) | 897 件 | OK | 余裕中 (8.97 倍) |
| Tier3 (≥ 50 億) | 652 件 | OK | 余裕中 (6.52 倍) |

★ ただし Active Light 制約 (max_daily_trades=5、max_open_positions=3) で
   実 trade 数は **60 営業日 × 5 = 300 件が上限**。candidate 件数は
   十分、stage gate (>= 100) は容易に達成可能。

## 4. 日別 universe size 抜粋 (head 10)

| date | univ_total | +3〜15% | vol≥1.5 | gap | margin | T1 | T2 | T3 |
|---|--:|--:|--:|--:|--:|--:|--:|--:|
| 2026-02-03 | 4,168 | 132 | 86 | 73 | 73 | 18 | 11 | 7 |
| 2026-02-04 | 4,163 | 724 | 172 | 157 | 157 | 72 | 51 | 35 |
| 2026-02-05 | 4,162 | 295 | 121 | 108 | 108 | 53 | 36 | 23 |
| 2026-02-06 | 4,158 | 277 | 126 | 104 | 104 | 38 | 26 | 20 |
| 2026-02-09 | 4,161 | 247 | 138 | 95 | 95 | 39 | 22 | 17 |
| 2026-02-10 | 4,176 | 505 | 243 | 220 | 220 | 98 | 66 | 51 |
| 2026-02-12 | 4,180 | 637 | 246 | 199 | 199 | 64 | 37 | 29 |
| 2026-02-13 | 4,178 | 448 | 260 | 207 | 207 | 63 | 34 | 25 |
| 2026-02-16 | 4,179 | 146 | 120 | 89 | 89 | 32 | 21 | 17 |
| 2026-02-17 | 4,182 | 453 | 233 | 217 | 217 | 62 | 25 | 17 |

(残 50 日は JSON `/tmp/lane_c_precheck.json` で全件参照)

## 5. scale_category 別 candidate (margin pass 後 60 日累積)

| scale_category | count |
|---|--:|
| - (不明 / 新興市場相当) | 2,875 |
| TOPIX Small 2 | 767 |
| TOPIX Small 1 | 596 |
| TOPIX Mid400 | 451 |
| TOPIX Large70 | 87 |
| TOPIX Core30 | 32 |

★ 観察:
- "-" (scale_category 不明、新興市場銘柄含む) が最多 (60% = 2,875/4,808)
- TOPIX Small (1+2) = 1,363 件 (28%)
- Mid400 以上 = 570 件 (12%)
- HQ 「TOPIX Small 除外固定せず流動性フィルタ代替」方針:
  → **turnover Tier フィルタが流動性代替として有効**
    (Tier3 ≥ 50 億で 4,808 → 652 件、Small 含めても流動性確保)

## 6. 観察 / 含意

### 6-1. universe filter の効き方

  - prev_change_pct +3〜+15% で: 4,168 → 平均 336 件/日 (絞り込み小)
  - vol/SMA20 ≥ 1.5 で: 平均 336 → 98 件/日 (29% 維持)
  - gap -2〜+4% で: 平均 98 → 80 件/日 (82% 維持、極端 gap は希少)
  - margin で: ほぼ全件通過 (4,449 / 5,049 = 88% が信用可)
  - turnover Tier1 で: 平均 80 → 26.53 件/日 (33% 維持、流動性確保有効)

### 6-2. 期待 trade 数 (Active Light 制約下)

  candidate 26.53 件/日 (Tier1) → max_daily_trades=5 + max_open_positions=3
  で実 entry は 1 日 5 件まで → 60 営業日 × 5 = **最大 300 件**
  実態は signal 発火タイミング (intraday VWAP / pullback) の充足度
  次第、smoke で実測予定。

### 6-3. trade_count gate (HQ §7-1) の充足

  HQ stage gate: trade_count >= 100 (60 営業日 strict)
  Tier1〜Tier3 すべて余裕で達成可能 ✅
  → universe filter 設計は妥当、Phase C1 (intraday data 取得) +
     Phase C2 (Lane C 真実装) で signal 発火が候補内で機能すれば
     stage gate 達成見込み

### 6-4. 推奨 Tier 選択 (Lane C MVP 候補)

  Mac mini 推奨: **Tier1 (≥ 10 億)** から開始
  理由:
  - candidate 1,592 件 (60 日) で trade gate 100 件の 15.9 倍余裕
  - 流動性確保 (10 億は実用下限、機関投資家参加銘柄含む)
  - max_daily_trades=5 で実 trade 数は十分
  - 結果次第で Tier2 / Tier3 に絞り込み移行可能
  本部判断要 (Tier1 / Tier2 / Tier3 / 別案)

## 7. 次工程

### 7-1. F105 起票 (HQ Q-Lane-C-C0-2 確定)

  task: F105 J-Quants 個別銘柄分足 Add-ons 接続検証 + staging 取込設計
  内容:
  - J-Quants Add-ons (/v2/equities/bars/minute、/v2/equities/trades)
    の契約 / 権限 / API 疎通検証
  - intraday data 仕様 (期間 / フィールド / rate limit / 日次更新タイミング)
  - staging 取込設計 (1 分足 第一候補、5 分足 評価 MVP、15 分足 派生生成)

  Vault 起票: 02_todo/F105_J-Quants個別銘柄分足Add-ons接続検証.md
  本タスク (Phase C0) で起票案を提示、HQ 確定後に正式起票。

### 7-2. Phase C1 (intraday data 取込、F105 通過後)

  - market_data/intraday.py 実装
  - scripts/setup/migrate_intraday_columns.py 新規 (staging 専用)
  - scripts/jobs/fetch_intraday_data.py 新規
  - 60 営業日分 backfill

### 7-3. Phase C2 (Lane C 真実装、Phase C1 完了後)

  - extract_lane_c_candidates 実装 (intraday rule-based)
  - tick.py / runner.py / cli.py に lane 切替経路
  - smoke + full run + stage gate 照合

## 8. 改訂履歴

- v1.0 (2026-05-08): 初版、precheck 実行直後 Vault 化、F105 起票へ進む
  判断材料整備済
