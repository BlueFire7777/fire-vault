---
id: FIRE-design-manual-live-pilot-W3-hold-criteria-2026-06-09
phase: 本番 v0 中核 / Wave 60-pilot-W3 集約成果 + HOLD criteria 改訂案
priority: 最高
status: design-final
owner: BlueFire7777 (Fujiwara)
date: 2026-06-09 (D19)
related_waves:
  - Wave 60-pilot-D14 (= GO_CONDITIONAL 初運用)
  - Wave 60-pilot-D15 (= HOLD 初発動)
  - Wave 60-pilot-D16 (= 340A0 demote、HOLD #2 解消)
  - Wave 60-pilot-D17-D19 (= HOLD maintained 連続)
supersedes_design: FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
---

# FIRE Manual Live Pilot — W3 HOLD Criteria Revision + D20 Recovery Criteria v1.0

## §1 目的

W2 設計 doc §3.3 HOLD 条件を W3 (= D14-D19 6 day pilot) で実運用検証。
浮上した課題:

- HOLD #1 (review missing 5 連続超過): 5 連続維持で解除困難
- HOLD #2 (同 candidate 7 営業日連続): 340A0 demote で解消、ただし
  3798 が 4 連続で再発リスク

本 doc は **HOLD criteria 改訂案** + **D20 recovery criteria** を確定する。

## §2 D14-D19 集約

### 2.1 6 day summary 表

| day | date | pilot_judgment | top 1 (AFTER-R1) | review status | paper PnL | HOLD #1 | HOLD #2 |
|---|---|---|---|---|---|---|---|
| D14 | 2026-06-02 | GO_CONDITIONAL | 340A0 ジグザグ | blank | pending | (未発動、5 連続未満) | (未発動、6 連続) |
| D15 | 2026-06-03 | HOLD | (HOLD only) | blank | pending | **✓ 発動** (6 連続) | **✓ 発動** (340A0 7 連続) |
| D16 | 2026-06-04 | HOLD maintained | 3798 ＵＬＳ | blank | pending | ✓ 継続 | **✗ 解消** (340A0 demote) |
| D17 | 2026-06-05 | HOLD maintained | 3798 ＵＬＳ | blank | pending | ✓ 継続 | ✗ 解消継続 (2 連続) |
| D18 | 2026-06-08 | HOLD maintained | 3798 ＵＬＳ | blank | pending | ✓ 継続 | ✗ 解消継続 (3 連続) |
| D19 | 2026-06-09 | HOLD maintained | 3798 ＵＬＳ | blank | pending | ✓ 継続 (5 連続) | ✗ 解消継続 (4 連続) |

### 2.2 review missing 構造 (= D9-D14 6 連続 + D15-D19 5 連続 = 累積 11 day blank)

- D9: blank
- D10: blank
- D11: blank
- D12: blank
- D13: blank
- D14: blank (= 6 連続)
- D15-D19: 全 blank (= HOLD day も blank)

→ **11 day 累積 blank**、設計 doc §3.3 #1 既発動状態が継続。
   D14 review 必須 5 項目記入が完全解除の唯一の path。

### 2.3 paper PnL status (= 6 day 全 pending)

| day | candidates | evaluated | win/loss/flat | review_missing |
|---|---|---|---|---|
| D14 | 20 | 0 | 0/0/0 | 1 |
| D15 | 20 | 0 | 0/0/0 | 1 |
| D16 | 20 | 0 | 0/0/0 | 1 |
| D17 | 20 | 0 | 0/0/0 | 1 |
| D18 | 20 | 0 | 0/0/0 | 1 |
| D19 | 20 | 0 | 0/0/0 | 1 |

→ **全 pending は staging max=5/14 の限界、真の outcome は h20 後 (= 6/26〜30 頃)**。

## §3 340A0 demote 効果評価

### 3.1 demote 履歴

- **D9-D15 (= 7 営業日連続)**: 340A0 F111 rank 4 boost_with_caution + AFTER-R1 rank 1 → HOLD #2 発動
- **D16 (= demote 初実施)**: recently_seen に 340A0 追加 → label caution、AFTER-R1 top から除外
- **D16-D19 (= 4 連続維持)**: 340A0 caution、demoted_count=4 連続

### 3.2 評価結論

- ✅ **demote は完全有効**: HOLD #2 (= 同 candidate 7 連続) を 1 wave で解消
- ✅ **持続性**: 4 連続維持で解消継続、demote 効果定着
- ✅ **新 top 確立**: 3798/137A0/331A0 が D16-D19 で 4 連続安定

## §4 新 top 3 安定性評価 (= 3798/137A0/331A0)

### 4.1 D16-D19 連続性

| ranking layer | 連続性 |
|---|---|
| AFTER-R1 rank 1 (3798) | D16-D19 で **4 連続** |
| AFTER-R1 rank 2 (137A0) | 4 連続 |
| AFTER-R1 rank 3 (331A0) | 4 連続 |

### 4.2 signal 安定 vs features cap 律速

| 観点 | 評価 |
|---|---|
| signal 安定 | ✅ research_final_score 不変、A1 rank、boost_with_caution 維持 |
| features cap 律速 | ✅ staging max=5/14、refresh 無いので close/score 同じ |
| 両方真 | ✅ 両方 |

→ 4 連続 = 同 candidate signal の自然な持続、ただし新 HOLD #2 リスク。

## §5 ⚠ 3798 連続候補化リスク評価

### 5.1 リスク詳細

- 3798 AFTER-R1 rank 1 = **D16-D19 で 4 連続**
- 設計 doc §3.3 HOLD #2 = "同 candidate 7 営業日連続"
- **D20: 5 連続 / D21: 6 連続 / D22: 7 連続 → HOLD #2 再発動**

### 5.2 recently_seen に 3798 を追加するか?

**判断: D20 で 3798 を追加すべき** (= 推奨)。

理由:
1. **HOLD #2 再発動を未然防止** (= D22 を待たずに demote)
2. **設計 doc §3.3 解除 path 実証** (= 340A0 demote と同 logic)
3. **候補多様化** (= 3798 demote 後、137A0/331A0/7991 等が rank 1 候補化)
4. **W3 でこの判断を Fujiwara に明示し、D20 wave で実装**

### 5.3 3798 demote 後の予想

- 3798 → caution に demote
- recently_seen_codes = 8747, 5729, 3489, 340A0, 3798 (= 5 件、demoted_count=5)
- AFTER-R1 新 top 1: 137A0 (= rank 2 → rank 1 へ昇格)
- AFTER-R1 新 top 2: 331A0 (= rank 3 → rank 2)
- AFTER-R1 新 top 3: 7991 マミヤ・オーピー (= 機械、sector 多様化候補)

## §6 HOLD criteria 改訂案 (= 設計 doc §3.3 改訂)

### 6.1 既存 (W2 設計 doc §3.3)

| # | 条件 |
|---|---|
| 1 | review missing が 5 連続超過 |
| 2 | 同 candidate が 7 営業日以上連続 |
| 3 | actual price 確認できない |
| 4 | liquidity 確認できない |
| 5 | event リスク強い |

### 6.2 改訂案 (= W3 改訂)

| # | 条件 | 変更点 |
|---|---|---|
| **1** | review missing が 5 連続超過 (= 既存) | **解除条件追加**: review 最低 5 項目記入 |
| **2a** | **同 candidate が 5 営業日連続 → warning** | 新規追加 |
| **2** | 同 candidate が 7 営業日連続 → HOLD | 既存維持 |
| **2-exclusion** | **demote 済み candidate は連続カウント停止** | 新規追加 |
| 3 | actual price 確認できない | 既存維持 |
| 4 | liquidity 確認できない | 既存維持 |
| 5 | event リスク強い | 既存維持 |
| **6** | **override entry は review 必須** | 新規追加 |

### 6.3 改訂解釈

- **HOLD #2-exclusion**: 例えば 340A0 は D9-D15 で 7 連続発動 → demote 後は連続
  カウント停止 → D22 で 14 連続でも HOLD 再発動しない (= demote 状態維持なら)
- **HOLD #2a warning**: 5 連続で警告、Fujiwara に「recently_seen に追加検討」を示唆。
  これにより HOLD #2 (7 連続) 発動を未然防止可能。

## §7 D20 recovery criteria (= GO_CONDITIONAL 復帰)

### 7.1 D20 で GO_CONDITIONAL 復帰する条件

| # | 条件 | 必須 |
|---|---|---|
| 1 | D14 review 必須 5 項目記入 **または** D20 review gap 扱い明確化 | ✓ |
| 2 | 340A0 demote 維持 | ✓ |
| 3 | **3798 を recently_seen に追加** (= demote、W3 改訂で必須化) | ✓ |
| 4 | f111_real_batch | ✓ |
| 5 | top candidate ≥ 1 | ✓ |
| 6 | risk_within_pilot_limit=True | ✓ |
| 7 | actual price / liquidity / event 確認可能 | ✓ (= 朝確認) |
| 8 | safety_flags 13 keys all False | ✓ |
| 9 | paper PnL/review に重大問題なし | ✓ |

→ 9 条件 ✓ で **D20 = GO_CONDITIONAL** へ移行。

### 7.2 D20 で HOLD 継続する条件

- D14 review blank のまま、かつ
- 3798 を recently_seen に追加しない、かつ
- actual/liquidity/event 確認不可

→ いずれか 1 つ ✓ で HOLD 継続。

### 7.3 D20 NO-GO (= 設計 doc §3.4 維持)

- top candidates 0 / f111_input_source 不正 / risk_within 全 False / sample /
  forbidden_check failed / safety_flags True / price 不明 / 監視不能

## §8 D20 handoff

### 8.1 D20 で使う recently_seen_codes

```
8747, 5729, 3489, 340A0, **3798** (= 新追加、HOLD #2 未然防止)
```

→ demoted_count=**5**

### 8.2 D20 で見る候補

3798 demote 後の予想 top:

| rank | code | name | sector | 期待 |
|---|---|---|---|---|
| 1 (AFTER-R1) | 137A0 | Cocolive | 情報通信 | 新 #1 候補 |
| 2 | 331A0 | メディックス | 情報通信 | 新 #2 候補 |
| 3 | 7991 | マミヤ・オーピー | 機械 | 新 #3 候補 (= sector 多様化) |
| 4 | 9130 | 共栄タンカー | 運輸 | 多様化 |
| 5 | 4389 | プロパティデータバンク | 情報通信 | 候補 |

### 8.3 D20 で必ず確認する項目 (= W2 と同 + 改訂)

1. **D14 review 記入状況** (= W3 改訂で 5 項目最低限記入が GO 条件)
2. **3798 demote 動作確認** (= recently_seen 追加 → caution へ)
3. **新 top 1 = 137A0 確認** (= AFTER-R1 rank 1 切替)
4. 朝 J-Quants refresh (= 25 営業日 gap)
5. iSPEED actual price 確認 (= 5 銘柄)
6. 板厚 / 出来高 / spread
7. TDnet 開示 / 決算予定 / ニュース

### 8.4 D20 review 最低 5 項目 (= W2 設計 doc §5 維持)

1. ★ §1 記入時刻 + entry 銘柄
2. ★ §2 entry 価格 / 株数 / 時刻
3. ★ §3 総 PnL
4. ★ §5 Reason for entry / skip
5. ★ §6 final decision

### 8.5 paper PnL handoff (= D20 場後 / D21 翌朝 / h20 後)

- D20 当日: `--review-md` 付きで再 run
- D21 翌朝: h1 close 確認
- h20 後 (= 2026-07-06 頃): 真の win/loss/flat 判定

## §9 W3 改訂 vs W2 比較

| 項目 | W2 (D14 entry criteria) | W3 (D20 recovery criteria) |
|---|---|---|
| HOLD #1 解除 | (= 言及なし) | **review 最低 5 項目記入で解除** |
| HOLD #2 解除 | (= 言及なし) | **demote 済み candidate は連続カウント停止** |
| HOLD #2 warning | (= なし) | **5 連続で warning、7 連続で発動** |
| recently_seen 拡張 | 8747,5729,3489 default | **D20 で 3798 追加 = 5 件** |
| override entry | (= 言及なし) | **review 必須** |

## §10 next wave action (= D20 想定)

```
Wave 60-pilot-D20 (= GO_CONDITIONAL 復帰判定 + 3798 demote)
- design reference: 03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
- recently_seen: 8747,5729,3489,340A0,3798 (= 5 件)
- D14 review 5 項目記入確認
- 137A0 が新 rank 1 として確立か確認
- 4 段階判定 (= W3 改訂版 criteria)
```
