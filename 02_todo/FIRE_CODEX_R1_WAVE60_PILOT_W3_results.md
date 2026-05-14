---
id: FIRE-CODEX-R1-WAVE60-PILOT-W3-results
phase: 本番 v0 中核 / Wave 60-pilot-W3 / HOLD 期間集約 + D20 recovery criteria
priority: 最高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14 (実行) / 集約対象 D14-D19 (= 2026-06-02 〜 2026-06-09)
aggregation_range: D14-D19 6 営業日
design_doc: ~/fire-vault/03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
codex_lanes: 4 / 4 YES ✓
next_wave: Wave 60-pilot-D20 (= GO_CONDITIONAL 復帰判定 + 3798 demote)
---

# Wave 60-pilot-W3 Results — HOLD Period Aggregation / D20 Recovery Criteria

## §1 結論

D14-D19 6 day pilot を集約、**HOLD criteria 改訂案完成**、**D20 recovery criteria
確定**、**3798 demote (= recently_seen 拡張) を D20 で実施推奨**。Codex 4 lane 全 YES、
3 DB md5 全 不変、本 wave で code 変更 0。

**主要成果**:
- ✅ D14-D19 全 status: blank + 全 paper PnL pending 確認
- ✅ 340A0 demote 4 連続維持 (= HOLD #2 解消継続)
- ✅ 3798 が AFTER-R1 rank 1 で 4 連続 → D22 で 7 連続 HOLD 再発動懸念明示
- ✅ HOLD criteria 改訂案 (= 5 連続 warning、demote 例外、override review 必須)
- ✅ D20 recovery criteria 確定 (= 9 条件、3798 recently_seen 追加必須化)
- ✅ design doc 完成 (= W3 改訂、W2 設計 doc を supersede)

## §2 D14-D19 6 day summary

| day | date | pilot | top 1 (AFTER-R1) | review | paper PnL | HOLD #1 | HOLD #2 |
|---|---|---|---|---|---|---|---|
| D14 | 6/2 | GO_CONDITIONAL | 340A0 | blank | pending | (未発動) | (未発動) |
| D15 | 6/3 | **HOLD** | (HOLD only) | blank | pending | **✓ 発動** | **✓ 発動** (340A0 7 連続) |
| D16 | 6/4 | HOLD maintained | 3798 (= 新 #1) | blank | pending | ✓ 継続 | **✗ 解消** (340A0 demote) |
| D17 | 6/5 | HOLD maintained | 3798 | blank | pending | ✓ 継続 (2 連続) | ✗ 解消継続 (2 連続) |
| D18 | 6/8 | HOLD maintained | 3798 | blank | pending | ✓ 継続 (3 連続) | ✗ 解消継続 (3 連続) |
| D19 | 6/9 | HOLD maintained | 3798 | blank | pending | ✓ 継続 (4 連続) | ✗ 解消継続 (4 連続) |

## §3 review missing 構造分析

- D9-D14: 6 連続 blank → HOLD #1 発動
- D15-D19: 5 連続 blank (= HOLD 期間も blank)
- **累積 11 day blank**
- D14 review 必須 5 項目記入が完全解除の唯一の path

→ Codex Lane A: **YES** ✓ "D14 status: blank with 0/6 filled, D15-D19 are blank, all six D14-D19 paper PnL ledgers have review_missing_count=1"

## §4 paper PnL status (= 6 day 全 pending)

| day | candidates | evaluated | review_missing |
|---|---|---|---|
| D14-D19 | 全 20 | 全 0 | 全 1 |

→ staging max=5/14 の限界、真の outcome は h20 後 (= 2026-06-26〜30 頃)。

## §5 340A0 demote 効果評価 (= ✅ 完全有効)

| 項目 | 結果 |
|---|---|
| D9-D15 (= 7 連続 boost_with_caution) | HOLD #2 発動 |
| D16 (= demote 初実施) | label caution、AFTER-R1 top 除外 |
| D16-D19 (= 4 連続維持) | demote 持続、HOLD #2 解消継続 |
| 新 top 確立 | 3798/137A0/331A0 |

→ Codex Lane B: **YES** ✓ "D16 demotion shifts AFTER-R1 top 1 from 340A0 to 3798, top 3 = 3798/137A0/331A0 for D16-D19"

## §6 ⚠ 3798 連続候補化リスク評価

### 6.1 現状

- 3798 AFTER-R1 rank 1 = **D16-D19 で 4 連続**
- 設計 doc §3.3 HOLD #2 = "同 candidate 7 営業日連続"
- **D20 で 5 連続 / D21 で 6 連続 / D22 で 7 連続 → HOLD #2 再発動**

→ Codex Lane C: **YES** ✓ "D16-D22 で同一候補 3798 が 7 連続 rank 1 なら、D22 で HOLD 再発動条件を満たします"

### 6.2 判断: **D20 で 3798 を recently_seen に追加** ★ 必須

理由:
- HOLD #2 再発動未然防止
- 設計 doc §3.3 解除 path 実証 (= 340A0 同 logic)
- 候補多様化 (= 137A0/331A0/7991 が rank 1 候補化)

### 6.3 3798 demote 後の予想

| rank | code | name | sector |
|---|---|---|---|
| 1 (AFTER-R1) | **137A0** | Cocolive | 情報通信 (= 新 #1) |
| 2 | **331A0** | メディックス | 情報通信 |
| 3 | **7991** | マミヤ・オーピー | 機械 (= sector 多様化) |

## §7 HOLD criteria 改訂案 (= W2 → W3)

| # | W2 (既存) | W3 (改訂) | 変更点 |
|---|---|---|---|
| 1 | review missing 5 連続超過 | + **解除条件**: review 最低 5 項目記入 | 新 解除 path |
| **2a** | (= なし) | **同 candidate 5 連続 → warning** | 新規 |
| 2 | 同 candidate 7 連続 → HOLD | (同) | 維持 |
| **2-exclusion** | (= なし) | **demote 済み candidate は連続カウント停止** | 新規 |
| 3-5 | actual / liquidity / event | (同) | 維持 |
| **6** | (= なし) | **override entry は review 必須** | 新規 |

## §8 D20 recovery criteria (= GO_CONDITIONAL 復帰、9 条件)

| # | 条件 | 必須 |
|---|---|---|
| 1 | D14 review 5 項目記入 OR D20 review gap 明確化 | ✓ |
| 2 | 340A0 demote 維持 | ✓ |
| 3 | **3798 を recently_seen に追加** | ✓ (= 新) |
| 4 | f111_real_batch | ✓ |
| 5 | top candidate ≥ 1 | ✓ |
| 6 | risk_within_pilot_limit=True | ✓ |
| 7 | actual / liquidity / event 確認可能 | ✓ (朝確認) |
| 8 | safety_flags 13 keys all False | ✓ |
| 9 | paper PnL / review に重大問題なし | ✓ |

→ Codex Lane D: **YES** ✓ "D20 GO_CONDITIONAL requires D14 review ≥5 fields or justified override, keeps 340A0 demote, considers 3798 connectivity"

## §9 D20 handoff

### 9.1 D20 で使う recently_seen_codes

```
8747, 5729, 3489, 340A0, **3798** (= W3 新追加)
demoted_count=5
```

### 9.2 D20 で見る候補 (= 3798 demote 後)

| rank | code | name | sector | close (5/14) | risk_yen |
|---|---|---|---|---|---|
| 1 (期待) | 137A0 | Cocolive | 情報通信 | 739 | 3,695 |
| 2 | 331A0 | メディックス | 情報通信 | 482 | 2,410 |
| 3 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 (= 多様化 #4) |
| 4 | 9130 | 共栄タンカー | 運輸 | 1,410 | 7,050 |
| 5 | 4389 | プロパティデータバンク | 情報通信 | 863 | 4,315 |

### 9.3 D20 で必ず確認する項目

1. **D14 review 記入状況** (= W3 改訂で必須)
2. **3798 demote 動作確認** (= recently_seen 追加 → caution へ)
3. **新 top 1 = 137A0 確認**
4. 朝 J-Quants refresh (= 25 営業日 gap、staging 5/14 → 6/10 想定)
5. iSPEED actual price 確認 (= 137A0/331A0/7991/9130/4389 = 5 銘柄)
6. 板厚 / 出来高 / spread
7. TDnet 開示 / 決算予定 / ニュース

### 9.4 D20 review 最低 5 項目 (= W2 設計 doc §5 維持)

1. ★ §1 記入時刻 + entry 銘柄
2. ★ §2 entry 価格 / 株数 / 時刻
3. ★ §3 総 PnL
4. ★ §5 Reason for entry / skip
5. ★ §6 final decision

### 9.5 D20 paper PnL handoff

- D20 当日 (= 場後): `--review-md` 付きで再 run
- D21 翌朝 (= 6/11 木): h1 close 確認
- h20 後 (= 2026-07-08 頃): 真の win/loss/flat 判定

## §10 Codex 4 lane factual-confirm

| lane | 観点 | reply |
|---|---|---|
| **A** | D14-D19 review/PnL (= 全 blank、ledger 6 件 review_missing=1) | **YES** ✓ |
| **B** | 340A0 demote 効果 (= D16 切替、3798 rank 1) | **YES** ✓ |
| **C** | 3798 連続性リスク (= D22 7 連続到達) | **YES** ✓ |
| **D** | D20 recovery criteria (= 9 条件、3798 demote、review 必須) | **YES** ✓ |

→ 4 lane 全 事実確認、改訂案実装可能。

## §11 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| DB write / API / token | 全 0 ✓ |
| LINE / launchctl / plist / cron | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault 3 file 追加のみ) |

## §12 vault docs (= 本 wave 追加)

- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W3_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W3_results.md`
- `~/fire-vault/03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md` (= W3 改訂、W2 supersede)

## §13 next wave instruction (= W60-pilot-D20)

```
Wave 60-pilot-D20 (= GO_CONDITIONAL 復帰判定 + 3798 demote 初実施)
- design reference: 03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
- recently_seen_codes: 8747,5729,3489,340A0,3798 (= 5 件)
- demoted_count: 5
- D14 review 5 項目記入確認 ★最優先 (= HOLD #1 解除 path)
- 137A0 が新 rank 1 として確立か確認
- 4 段階判定 (= W3 改訂版 criteria 9 条件)
- D20 で:
  - 全 9 条件 ✓ → GO_CONDITIONAL 復帰
  - D14 review blank / 3798 demote 未実施 → HOLD 維持
```

## §14 6 KPI

1. 稼働率: 100% (= 12 step 全完走、blocker 0)
2. 短縮率: 高 (= D14-D19 集約 + Codex 4 lane + criteria 改訂 + design doc + vault を 1 conversation で完結)
3. 採用率: 100% (= 改訂案全採用、Codex 4 lane 全 YES、D20 handoff 確定)
4. 差戻率: 0% (= chain read-only、Codex 全 YES、想定通り)
5. Integrator 負荷: 低 (= read-only 集約、3 DB 全 md5 不変、code 変更 0、vault 3 file)
6. 安全事故: 0 (= DB write 0 / API 0 / token 0 / launchctl 0 / 実発注 0)

## §15 設計成果総括

| 項目 | 達成 |
|---|---|
| D14-D19 集約 | ✅ 6 day 全 trade plan / review / paper PnL 確認 |
| HOLD #1 (review missing) 構造分析 | ✅ 11 day 累積 blank、解除 path 明示 |
| HOLD #2 (同 candidate) 解除事例 | ✅ 340A0 demote 4 連続維持 |
| 3798 連続性懸念 | ✅ D22 7 連続到達リスク明示、D20 demote 推奨 |
| HOLD criteria 改訂案 | ✅ W3 改訂 (= 5 連続 warning、demote 例外、override review 必須) |
| D20 recovery criteria | ✅ 9 条件確定、handoff 完成 |
| design doc (W2 supersede) | ✅ FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md |

→ FIRE pilot 運用の **HOLD criteria 自己改訂 path** が実運用で機能、
   FIRE が「設計 → 実運用 → 改訂」のサイクルを回せる状態に到達。
