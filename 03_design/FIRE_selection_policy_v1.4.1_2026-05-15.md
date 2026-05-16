---
id: FIRE-design-selection-policy-v1.4.1-2026-05-15
phase: 本番 v0 中核 / selection policy v1.4.1 QA 修正 (= pre-open score と entry_priority 分離)
priority: 最高
status: design-final
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
supersedes: FIRE_selection_policy_v1.4_2026-05-15.md (= v1.4、本 doc は数値矛盾解消 + 追い見送り logic 追加)
trigger_event: |
  v1.4 で 9130 を post_open_uplift +1.5 で promote したが、9247 (5.61) > 9130 (5.56) で
  「final_score 上は 9247 が #1」の矛盾が残った。さらに D31 前場で 9130 は +4.89%
  到達 (= 1,479 円推定) で「追わない上限 1,440 円」を超過、新規追い entry 不可 logic
  も明文化していなかった。本 v1.4.1 でこれらを QA 修正する。
related_pilot_day: D31 (= 2026-06-25 木)
actual_flow_observation: "前場 9130 高値 +4.89% (= ~1,479 円)、追わない上限 1,440 円超過"
---

# FIRE Selection Policy v1.4.1 — Pre-Open Score と Entry-Priority 分離 + 追い見送り Logic

## §1 v1.4 からの維持事項

- ✓ risk_yen 非加点 (= 株数・損切り・pilot gate のみ)
- ✓ entry / watch / excluded 三分類
- ✓ 331A0 / 4389 / 4317 を excluded 維持
- ✓ 33 sector 分類 (= 9247 リサイクル / 9628 葬祭 / 9130 海運業)
- ✓ theme / catalyst evidence-based (= 根拠なし加点 0)
- ✓ pre-open / post-open phase 分離
- ✓ safety 境界 (= DB write 0 / API 0 / etc.)

## §2 v1.4.1 QA 修正点

### 2.1 pre_open_score と entry_priority の **完全分離** (= 案 B 採用) ★

v1.4 では post_open_uplift を pre_open に加算して final_score を作っていたが、
9247 (5.61) > 9130 (5.56) で数値矛盾。v1.4.1 では:

| 用語 | 定義 | 使い道 |
|---|---|---|
| **pre_open_score** | staging proxy のみで算出 | 寄り前の候補順位 (= 参考) |
| **entry_priority** | post-open actual flow override 後の最終 entry 順位 | **実弾運用の判断軸** ★ |

→ **数値を 1 本化しない**。pre-open は static analysis 結果、entry_priority は
   actual flow override で別軸として明示。**矛盾なし**。

### 2.2 post-open actual flow override rule

```
[Override rule]

if post-open で 候補 X が以下のいずれかを満たす:
  - 寄り後 5 min 出来高 = 過去平均比 ≥ 150%
  - 寄り後上昇率 ∈ [+0.5%, +2.0%] (= 追わない上限内)
  - 板厚十分 + spread 狭 + catalyst 確認

  → entry_priority で X を **override 昇格** (= pre-open 順位無視)

if 候補 X が以下のいずれかを満たす:
  - 寄り後上昇率 > 追わない上限 (= +2.0% 超え)
  - 出来高急減 (= 過去平均比 < 50%)
  - 悪材料 / 適時開示で警戒

  → entry_priority から X を **降格 or 除外** (= 新規 entry 不可)
```

### 2.3 「シグナル成功」と「実 entry 成功」の分離 ★

D31 前場 9130 +4.89% を例に明文化:

| 観点 | 判定 |
|---|---|
| **シグナル成功** (= 候補抽出が正しかったか) | ✓ YES (= 9130 が前場最強、selection logic 正解) |
| **追わない上限内かどうか** | ✗ NO (= +4.89% は 1,440 円超過、上限内ではない) |
| **朝 1,395-1,425 円で entry 済みの場合** | ✓ 利確目安 1,450-1,470 円到達、**実 entry 成功** |
| **未 entry の場合** | ✗ 追わない (= 新規 entry 不可、追い負けリスク) |

→ **「銘柄選定が正しかった」と「今から追える」は分ける**

### 2.4 追わない上限超過時の logic 明文化

```
[追わない上限超過時の logic]

if 寄り後上昇率 > 追わない上限:
  if Fujiwara が朝 entry 済み:
    → 利確タイミング検討 (= 利確目安到達なら exit、未到達なら hold + trailing stop)
  else:
    → 新規追い entry 不可 (= "skip 確定"、後場でも追わない)
    → 「シグナル成功」として記録、ただし「実 entry 成功」とは別
```

### 2.5 review 項目強化 (= 実弾検証可能化)

v1.4 までは「入った / 見送った」「判断理由」「銘柄」「板出来高所感」「結果メモ」の 5 項目。
v1.4.1 で以下を必須追加:

| 追加項目 | 内容 |
|---|---|
| **6. 実 entry か skip か** | enter (= 約定済) / skip (= 入らず) |
| **7. 入った価格 or 見送った理由** | 約定価格 / 理由 (= 上限超過 / liquidity / event 等) |
| **8. 追わない上限を超えていたか** | Yes / No、超えていた場合は超過率も |
| **9. pre-open 判断か post-open 判断か** | どちらの phase で entry/skip 決定したか |
| **10. シグナル成功と実 entry 成功の区別** | signal_success: Y/N、entry_success: Y/N の 2 軸 |

→ 計 **10 項目**、実弾運用後の検証が可能な構造

## §3 D31 candidates v1.4.1 適用結果

### 3.1 pre_open_score (= v1.4 と同じ、参考)

| rank | code | name | 33 sector | pre_open |
|---|---|---|---|---|
| 1 | 9247 | ＴＲＥ HD | サービス業/リサイクル | 5.61 |
| 2 | 9628 | 燦 HD | サービス業/葬祭 | 4.65 |
| 3 | 9130 | 共栄タンカー | 海運業/タンカー | 4.06 |
| 4 | 4404 | ミヨシ油脂 | 食品工業 | 3.74 |

### 3.2 post-open actual flow observation (= D31 前場実績、Fujiwara 観測)

| code | 寄り後上昇率 | 出来高 | 板/spread | catalyst | actual flow 判定 |
|---|---|---|---|---|---|
| **9130** | **+4.89% (= 1,479 円)** ★ | 強 | 確認要 | 海運/タンカー関連 | **追わない上限超過** ⚠ |
| 9247 | (未観測 / proxy) | 普通 | 確認要 | ESG 未確認 | actual flow 弱 |
| 9628 | (未観測 / proxy) | 普通 | 確認要 | なし | actual flow 弱 |

### 3.3 entry_priority (= v1.4.1 final、override 適用)

**条件分岐**:

#### (A) Fujiwara が朝 1,395-1,425 円で 9130 を entry 済みの場合:

| entry_priority | code | 状態 |
|---|---|---|
| #1 (= 利確段階) | **9130** | 寄り後 +4.89% で利確目安 1,450-1,470 円到達、**実 entry 成功** ✓ |
| #2 (= 新規 entry 余地) | 9247 | pre-open #1、actual flow 弱、新規 entry 検討は朝 iSPEED 確認後 |
| #3 | 9628 | pre-open #2、actual flow 弱 |

→ **9130 シグナル成功 + 実 entry 成功**、利確に切替

#### (B) Fujiwara が未 entry の場合:

| entry_priority | code | 状態 |
|---|---|---|
| #1 (= 新規 entry 検討) | **9247** | pre-open #1、actual flow 弱だが追わない上限内、後場で検討余地 |
| #2 | 9628 | pre-open #2、ディフェンシブ |
| **NA** (= skip 確定) | **9130** ⚠ | **追わない上限 1,440 円超過 (= +4.89%)、新規追い entry 不可**、シグナル成功として記録のみ |

→ **9130 = シグナル成功、ただし実 entry は不可** (= 9247 へスライド)

### 3.4 「9130 が良かった」と「今から追う」の明確分離

| 評価軸 | 結論 |
|---|---|
| 9130 の selection logic 妥当性 | ✓ 妥当 (= signal_success) |
| 9130 を post-open で新規 entry 推奨か | **✗ 推奨せず** (= 追わない上限超過、entry_skip 確定) |
| 朝 entry 済みなら | 利確段階、実 entry 成功 |
| 未 entry なら | 9247 へスライド、9130 はシグナル記録のみ |

## §4 数値矛盾解消の検証 (= QA 観点 1)

| 観点 | v1.4 (= 矛盾あり) | v1.4.1 (= 矛盾解消) |
|---|---|---|
| 9247 final score | 5.61 | pre_open 5.61 のみ (= final score を作らない) |
| 9130 final score | 5.56 (uplift 加算後、9247 より低) | pre_open 4.06 のみ + entry_priority 別ランキング |
| 順位整合性 | 9247 > 9130 の数値矛盾 | **pre-open と entry_priority を分離**、各々独立 ✓ |
| post-open override | uplift 数値加算 | **entry_priority の override rule**、数値結合なし ✓ |

→ **数値矛盾完全解消** ✓

## §5 sector / liquidity / selection policy 既存修正維持

- ✓ risk_yen 非加点 (= v1.2 から維持)
- ✓ entry / watch / excluded 三分類 (= v1.2 から維持)
- ✓ 331A0 / 4389 / 4317 excluded (= v1.1 から維持)
- ✓ 33 sector 分類 (= v1.3 から維持)
- ✓ theme / catalyst evidence-based (= v1.4 から維持)
- ✓ pre-open / post-open 分離 (= v1.4 から維持、本 v1.4.1 で完全分離化)
- ✓ safety 境界 (= 全 0)

## §6 v1.4.1 適用 timing

- **即時適用** (= 連動 pilot D31 = 2026-06-25 advisory v1.4.1 から)
- v1.0-v1.4 baseline 全保持

## §7 安全境界

- F111 source code 変更 0 (= 本 wave は design doc + advisory v1.4.1 + Codex 4 lane)
- DB write 0 / API 0 / token 0 / LINE 0 / commit 0 / push 0
- v1.0-v1.4 baseline 全保持
