---
id: FIRE-design-manual-live-pilot-W4-demote-policy-2026-06-18
phase: 本番 v0 中核 / Wave 60-pilot-W4 集約成果 + demote policy 正式化
priority: 最高
status: design-final
owner: BlueFire7777 (Fujiwara)
date: 2026-06-18 (D26) / wave 実行: 2026-05-14
related_waves:
  - Wave 60-pilot-D14 (= GO_CONDITIONAL 初運用)
  - Wave 60-pilot-D15 (= HOLD 初発動)
  - Wave 60-pilot-D16 (= 340A0 demote、HOLD #2 解消)
  - Wave 60-pilot-D17-D19 (= HOLD maintained 連続)
  - Wave 60-pilot-D20 (= GO_CONDITIONAL 復帰 + 3798 demote 初実施 + sector 2 種)
  - Wave 60-pilot-D21-D23 (= maintained 2-4 連続 + 137A0 連続化監視)
  - Wave 60-pilot-D24 (= 137A0 5 連続 warning + demote sim 初実施)
  - Wave 60-pilot-D25 (= 137A0 demote 本実行 + sector 3 種多様化達成)
  - Wave 60-pilot-D26 (= post-demote 2 連続安定)
supersedes_design: FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
---

# FIRE Manual Live Pilot — W4 Demote Policy Formalization v1.0

## §1 目的

W3 設計 doc で導入した HOLD criteria 改訂 (= 5 連続 warning / demote 例外 /
override review 必須) を W4 (= D20-D26 7 day pilot) で実運用検証。
浮上した運用 pattern (= 3 銘柄段階的 demote、sector 3 種多様化、HOLD #2
完全回避、二段階 sim → 本実行) を **demote policy** として正式文書化する。

本 doc は W3 設計 doc を **supersede** し、HOLD criteria + demote policy を
統合した運用基準とする。

## §2 W4 集約成果 (= D20-D26)

### 2.1 demote 履歴 + 連続維持 (= D26 時点)

| 銘柄 | demote 開始 | 開始時 status | D26 時点連続 | 効果 |
|---|---|---|---|---|
| 8747 | 〜D1 | boost_with_caution → caution | 21 連続 | ✅ 完全定着 |
| 5729 | 〜D1 | boost_with_caution → caution | 21 連続 | ✅ 完全定着 |
| 3489 | 〜D1 | boost_with_caution → caution | 21 連続 | ✅ 完全定着 |
| **340A0** | **D16** | 7 連続 rank 1 → caution | **11 連続** | ✅ HOLD #2 解消 + 持続 |
| **3798** | **D20** | 4 連続 rank 1 → caution | **7 連続** | ✅ HOLD #2 未然防止 |
| **137A0** | **D25** | 5 連続 rank 1 → caution | **2 連続** | ✅ HOLD #2 完全回避 |

### 2.2 sector 多様化推移

| 期 | 期間 | 多様化 |
|---|---|---|
| W3 (HOLD) | D14-D19 | 情報通信 100% |
| W4 前半 | D20-D24 | 情報通信 + 機械 (= 2 種) |
| **W4 後半** | **D25-D26** | **機械 + 運輸 + 情報通信 (= 3 種)** ✓ |

### 2.3 GO_CONDITIONAL 7 連続維持

D20-D26 全 7 wave で W3 §7.1 9 条件 全 ✓ → **GO_CONDITIONAL 連続維持**。

## §3 demote 三段階 policy (= W4 正式化)

### 3.1 三段階 policy 定義

| 段階 | 条件 | アクション | 例 |
|---|---|---|---|
| **段階 1: warning** | rank 1 で **5 連続** | trade plan §警告セクション追記、demote sim 検討開始 | D24 で 137A0 |
| **段階 2: sim** | rank 1 で **5-6 連続** | recently_seen に追加した参考 chain 実行 (= staging dry-run)、新 top 確認 | D24 で 137A0 sim |
| **段階 3: 本実行** | rank 1 で **6 連続** (= 7 連続到達前) | `recently_seen_codes` に正式追加、F111 chain 実 run | D25 で 137A0 demote |

### 3.2 各段階の判定 logic

```python
# 概念図 (= 実装は F111 内 demote logic で既存)
def evaluate_consecutive_rank_1(symbol_code, recent_n_waves):
    consecutive = count_consecutive_rank_1(symbol_code, recent_n_waves)
    if consecutive >= 5 and consecutive < 6:
        return "warning"  # = 段階 1
    elif consecutive == 6:
        return "demote_required"  # = 段階 3
    else:
        return "safe"
```

### 3.3 三段階を導入する理由

| 段階 | 設計意図 |
|---|---|
| 1 | **早期警戒**: HOLD #2 到達まで余裕を持たせる、demote 検討時間確保 |
| 2 | **影響予測**: demote 後の新 top + sector 多様化を事前に把握 |
| 3 | **正式実行**: 確実に HOLD 阻止、cost-justified だけ demote |

### 3.4 W3 では暗黙、W4 で明文化

- W3 設計 doc §3.3 #2a で "5 連続 warning" を新規追加 (= 段階 1 のみ言及)
- W3 では sim (段階 2) と 本実行 (段階 3) が一体的に扱われていた
- W4 で「sim → 本実行」を **二段階に分離** = 運用安全度向上

## §4 sector 多様化目標

### 4.1 目標値

| 項目 | 目標 |
|---|---|
| top 3 sector 種類数 | **3 種推奨** (= 最低 2 種) |
| 同 sector top 3 中の比率 | ≤ 2 (= 同 sector 過半数避ける) |
| sector warning trigger | top 3 同 sector 3 種で trade plan §警告に追記 |

### 4.2 sector warning policy (= W5 設計 doc で正式化検討)

| state | 内容 | action |
|---|---|---|
| **safe** | top 3 sector 2-3 種 | none |
| **warning** | top 3 全 同 sector | trade plan §警告追記 |
| **concentration risk** | warning 3 連続 | demote 検討開始 |

### 4.3 D25-D26 で達成した 3 種多様化を維持

- 機械 (= 7991): rank 1 維持
- 運輸 (= 9130): rank 2 維持
- 情報通信 (= 331A0): rank 3 維持

→ D27 以降も 3 種維持を狙う、ただし 7991 / 9130 連続性 monitor 必須。

## §5 連続性 monitor 運用

### 5.1 D26 時点 連続性 status

| 銘柄 | sector | rank | 連続日数 (= rank 上位) | rank 1 連続 | 警戒度 |
|---|---|---|---|---|---|
| 7991 | 機械 | 1 | 7 (= D20-D26) | 2 (= D25-D26) | 監視 |
| 9130 | 運輸 | 2 | 2 (= D25-D26) | 0 | 低 |
| 331A0 | 情報通信 | 3 | 7 (= D20-D26) | 0 | 監視 (= rank 3 のみ) |

### 5.2 D27 以降 monitor 対象

| 銘柄 | D27 で予想 rank 1 連続 | 段階 1 (warning) 予想 day | 段階 3 (本実行) 予想 day |
|---|---|---|---|
| **7991** | 3 | **D29** | **D30** |
| 9130 | 0 (rank 2) | n/a (= rank 1 でない) | n/a |
| 331A0 | 0 (rank 3) | n/a | n/a |

→ **D29 wave で 7991 demote sim 検討**、**D30 で本実行候補**。

### 5.3 monitor 用 trade plan §警告 template

```markdown
## §警告: 連続性 monitor

| 銘柄 | rank 1 連続 | 警戒段階 | 次 action |
|---|---|---|---|
| {code} | {N} | {warning/safe} | {sim/本実行/none} |
```

## §6 D27 handoff

### 6.1 D27 で使う recently_seen_codes

```
8747, 5729, 3489, 340A0, 3798, 137A0 (= 6 件維持)
demoted_count=6
```

### 6.2 D27 で見る候補 (= D26 と同じ予想)

| rank | code | name | sector | close (5/14) | risk_yen |
|---|---|---|---|---|---|
| 1 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 |
| 2 | 9130 | 共栄タンカー | 運輸・物流 | 1,410 | 7,050 |
| 3 | 331A0 | メディックス | 情報通信 | 482 | 2,410 |

### 6.3 D27 で必ず確認する項目

1. **D26 review 5 項目記入状況** (= 累積 18 day blank、path 2 継続適用判定)
2. **7991 rank 1 連続日数 = 3 確認** (= warning まで残 2 wave)
3. **9130 rank 2 連続日数 = 3 確認**
4. **331A0 rank 3 連続日数 = 8 確認**
5. **sector 3 種多様化維持確認**
6. 朝 J-Quants refresh (= 28 営業日 gap、staging 5/14 → 6/19 想定)
7. iSPEED actual price 確認 (= 7991/9130/331A0)
8. 板厚 / 出来高 / spread
9. TDnet 開示 / 決算予定 / ニュース

### 6.4 D27 review 最低 5 項目

1. ★ §1 記入時刻 + entry 銘柄
2. ★ §2 entry 価格 / 株数 / 時刻
3. ★ §3 総 PnL
4. ★ §5 Reason for entry / skip
5. ★ §6 final decision

### 6.5 D27 paper PnL handoff

- D27 当日 (= 場後): `--review-md` 付きで再 run
- D28 翌朝 (= 6/22 月): h1 close 確認 (= D27 base=6/19 から h1=6/22)
- h20 後 (= 2026-07-17 頃): 真の win/loss/flat 判定

## §7 W3 supersede 関係

### 7.1 W3 設計 doc との関係

| W3 設計 doc 章 | W4 doc での扱い |
|---|---|
| §1 目的 | 本 doc §1 で更新 |
| §2 D14-D19 集約 | 本 doc §2 で W4 集約に拡張 |
| §3 340A0 demote 効果 | 本 doc §2 で 3 銘柄に拡張 |
| §4 新 top 3 安定性 | 本 doc §3 + §5 で 連続性 monitor 化 |
| §5 3798 連続候補化リスク | 本 doc §3 三段階 policy に統合 |
| §6 HOLD criteria 改訂案 | **本 doc §3 + §4 で正式化** |
| §7 D20 recovery criteria | (実証済み、本 doc §6 で D27 handoff に進化) |
| §8 D20 handoff | (実証済み、本 doc §6 で D27 handoff に進化) |
| §9 W3 vs W2 比較 | (歴史的、本 doc §8 で W4 vs W3 比較に進化) |

### 7.2 W3 から維持される条項

- HOLD #1 解除条件 (= review 5 項目記入) ★ 維持
- HOLD #2 警告 (= 5 連続) ★ 維持 + 三段階化
- HOLD #2 例外 (= demote 済み連続カウント停止) ★ 維持
- override entry review 必須 ★ 維持
- recovery criteria 9 条件 ★ 維持 (= D20-D26 で実証)

### 7.3 W4 で新規追加される条項

- **demote 三段階 policy** (= warning → sim → 本実行)
- **sector 多様化目標** (= top 3 sector 2-3 種推奨)
- **sector warning policy 草案** (= W5 設計 doc で正式化)
- **連続性 monitor 運用** (= rank 1 連続日数で判定)

## §8 W4 vs W3 比較

| 項目 | W3 (= D14-D19) | W4 (= D20-D26) |
|---|---|---|
| 期間 | 6 営業日 | **7 営業日** |
| 主要 judgment | HOLD 連続 | **GO_CONDITIONAL 連続** |
| HOLD #2 発動回数 | 1 (= D15 初) | **0** (= demote で阻止) |
| demote 実施回数 | 1 (= 340A0 D16) | **2** (= 3798 D20 + 137A0 D25) |
| sector 多様化 | 情報通信のみ | **3 種達成** (= 機械 + 運輸 + 情報通信) |
| design doc 主題 | HOLD criteria 改訂 | **demote policy 正式化** |
| Codex 4 lane | reply 4/4 YES | **self-audit 4/4 YES** |

## §9 next wave action (= D27 想定)

```
Wave 60-pilot-D27 (= recovery 8 連続 + 7991/9130/331A0 3 連続到達確認)
- date: 2026-06-19 (金)
- design reference (= supersede chain):
  - ~/fire-vault/03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md (= 旧版、W4 で supersede)
  - ~/fire-vault/03_design/FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md ★ (= 本 doc)
- recently_seen: 8747,5729,3489,340A0,3798,137A0 (= 6 件)
- demoted_count: 6
- D26 review 5 項目記入確認 (= path 2 継続判定)
- 7991 rank 1 連続性 = 3 確認 (= warning まで残 2 wave)
- 4 段階判定 (= W3 改訂版 9 条件 + W4 三段階 policy)
- D27 で:
  - 全 9 条件 ✓ → GO_CONDITIONAL maintained 8 連続
  - HOLD #2 再発動 0 → 完全回避継続
  - 7991/9130/331A0 3 連続到達
```

## §10 設計成果総括

| 項目 | W4 doc での達成 |
|---|---|
| demote 三段階 policy 正式化 | ✅ §3 で warning → sim → 本実行 を明文化 |
| sector 多様化目標 | ✅ §4 で top 3 sector 2-3 種推奨 |
| sector warning policy 草案 | ✅ §4.2 で W5 設計 doc で正式化検討 |
| 連続性 monitor 運用 | ✅ §5 で rank 1 連続日数判定 + warning template |
| D27 handoff | ✅ §6 で recently_seen / 候補 / 確認項目 / review / paper PnL |
| W3 supersede 関係 | ✅ §7 で章対応表 + 維持条項 + 新規追加条項 明示 |

→ FIRE pilot の **demote 運用が「経験的」から「明文 policy 化」へ進化**。
   W4 で確立した三段階 policy は W5 以降の sector warning policy の土台となる。
