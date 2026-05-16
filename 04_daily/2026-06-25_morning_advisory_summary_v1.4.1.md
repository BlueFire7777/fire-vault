---
template: morning-advisory-summary
version: 1.4.1
policy_applied: selection_policy_v1.4.1 (= pre_open_score と entry_priority 分離 + 追い見送り logic)
date: 2026-06-25
weekday: 木
owner: BlueFire7777 (Fujiwara)
linked_pilot_day: D31
linked_pilot_results: 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_results.md
linked_design: 03_design/FIRE_selection_policy_v1.4.1_2026-05-15.md
supersedes:
  - 04_daily/2026-06-25_morning_advisory_summary_v1.4.md (= v1.4)
pilot_judgment: GO_CONDITIONAL
post_recovery: 12 連続 maintained
hold_2_avoidance: 完全継続 (= D15 以降 16 連続)
demoted_count: 7
actual_flow_observation: "前場 9130 高値 +4.89% (= ~1,479 円)、追わない上限 1,440 円超過"
qa_consistency:
  date_matches_filename: true (= 2026-06-25 統一)
  date_matches_pilot_day: true (= D31 = 2026-06-25 木)
  numerical_consistency: true (= pre_open と entry_priority を分離、数値矛盾解消)
---

# 【FIRE 朝の銘柄通知サマリ v1.4.1】 — 2026-06-25 (木) / D31 + QA 修正

## 判定

**GO_CONDITIONAL** (= pilot D31 で 12 連続維持、HOLD #2 完全回避 16 連続)

⚠ v1.4.1 QA 修正:
1. pre_open_score と entry_priority を **完全分離** (= 数値矛盾解消)
2. **9130 = シグナル成功** (= 前場 +4.89%、追わない上限超過、新規追い不可)
3. 「シグナル成功」と「実 entry 成功」を分離記録
4. review 項目を **10 項目** へ拡張 (= 実弾検証可能化)

## Pre-Open ranking (= 寄り前 staging proxy、参考のみ)

| rank | code | name | 33 sector | pre_open_score |
|---|---|---|---|---|
| 1 | 9247 | ＴＲＥ HD | サービス業 / リサイクル | 5.61 |
| 2 | 9628 | 燦 HD | サービス業 / 葬祭 | 4.65 |
| 3 | 9130 | 共栄タンカー | 海運業 / タンカー | 4.06 |
| 4 | 4404 | ミヨシ油脂 | 食品工業 | 3.74 |

→ **これは寄り前順位のみ**。実弾 entry の判断は **entry_priority** を参照。

## Post-Open Actual Flow Observation (= D31 前場実績、Fujiwara 観測)

| code | 寄り後上昇率 | 出来高 | catalyst | actual flow 判定 |
|---|---|---|---|---|
| **9130** | **+4.89% (= 1,479 円)** | 強 | 海運/タンカー関連 | **追わない上限超過** ⚠ |
| 9247 | (未観測) | 普通 | ESG 未確認 | actual flow 弱 |
| 9628 | (未観測) | 普通 | なし | actual flow 弱 |

## Entry Priority (= post-open override 適用、実弾運用順位 ★)

### case A: Fujiwara が朝 1,395-1,425 円で 9130 entry 済み

| entry_priority | code | 状態 |
|---|---|---|
| **#1 (= 利確段階)** | **9130** | 前場 +4.89% で利確目安 1,450-1,470 円到達、**実 entry 成功 ✓** |
| #2 (= 新規 entry 余地) | 9247 | pre-open #1、actual flow 弱、後場 iSPEED 確認後検討 |
| #3 | 9628 | pre-open #2、ディフェンシブ |

→ **9130 シグナル成功 + 実 entry 成功** → 利確 (= 1,450-1,470 円目安) に切替

### case B: Fujiwara が未 entry の場合

| entry_priority | code | 状態 |
|---|---|---|
| **#1 (= 新規 entry 検討)** | **9247** ★ | pre-open #1、追わない上限内、後場で検討余地 |
| #2 | 9628 | pre-open #2、ディフェンシブ |
| **NA (= skip 確定)** | **9130** ⚠ | **追わない上限 1,440 円超過、新規追い entry 不可**、**シグナル成功として記録のみ** |
| (参考) #3 | 4404 | 食品工業、後場検討余地 |

→ **9130 はシグナル成功、ただし新規 entry は skip**

## 9130 シグナル成功 vs 実 entry 成功の分離

| 評価軸 | 結論 |
|---|---|
| 9130 の selection logic 妥当性 | ✓ YES (= signal_success) |
| 9130 を post-open で **新規追い entry** 推奨か | **✗ NO** (= 追わない上限超過、entry_skip 確定) |
| 朝 entry 済みなら | ✓ 利確段階、実 entry 成功 |
| 未 entry なら | ✗ 9247 へスライド、9130 はシグナル記録のみ |

**「銘柄選定が正しかった」と「今から追える」は明確に分ける**

---

## 【注文完成形アドバイザリ v1.4.1】

⚠ 発注指示ではない。Fujiwara が iSPEED で手動確認するための目安。
**entry_priority に従い、追わない上限超過時は新規追い entry 不可**。

### 候補 1A: 9130 共栄タンカー (海運業 / タンカー) — case A 朝 entry 済の場合、利確段階

- 朝 entry 済 (= 1,395-1,425 円付近) なら:
  - 状態: **利確段階**、前場 +4.89% で利確目安 1,450-1,470 円到達
  - 利確判断: 利確目安到達 → exit、または trailing stop で追従
  - 損切り (=既存) は 1,340 円 (= 朝 entry 時設定値)
- 朝 entry なし なら:
  - **entry 不可** (= 追わない上限 1,440 円超過)
  - シグナル成功として記録のみ、新規追いは skip

### 候補 1B: 9247 ＴＲＥ HD (サービス業 / リサイクル) — case B 未 entry 時の #1

- liquidity_status: PASS (= 貸借 + TOPIX Small 1)
- pre_open_score: 5.61 (= 静的評価で最高位)
- actual flow: 弱 (= post-open uplift なし)、ただし追わない上限内
- 基準株価: **1,612 円**
- エントリー検討価格: **1,596 - 1,628 円**
- 指値目安: **1,612 円**
- 追わない上限: **1,644 円** (= +2.0%)
- 利確目安: **1,660 - 1,693 円** (= +3-5%)
- 損切り目安: **1,532 円** (= -80 円、risk_yen=8,060 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **8,060 円**
- 見送り条件:
  - 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,644 円超え)
  - ESG 後退ニュース
  - 後場で 上昇率 +2% 超え → 9628 へスライド

### 候補 2: 9628 燦 HD (サービス業 / 葬祭) — ディフェンシブ

- liquidity_status: PASS (= 貸借 + TOPIX Small 2)
- pre_open_score: 4.65
- actual flow: 弱、ディフェンシブ
- 基準株価: **1,390 円**
- エントリー検討価格: **1,376 - 1,404 円**
- 指値目安: **1,390 円**
- 追わない上限: **1,418 円** (= +2.0%)
- 利確目安: **1,432 - 1,460 円** (= +3-5%)
- 損切り目安: **1,320 円** (= -70 円、risk_yen=6,950 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **6,950 円**
- 見送り条件: 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,418 円超え) / 悪材料

---

## 追わない上限超過時の logic 明文化 (= v1.4.1 新規)

```
[追わない上限超過時の logic]

if 寄り後上昇率 > 追わない上限:
  if Fujiwara が朝 entry 済み:
    → 利確タイミング検討
      - 利確目安到達 → exit
      - 未到達 → hold + trailing stop
  else:
    → 新規追い entry 不可 (= "skip 確定")
    → 後場でも追わない (= 追い負けリスク)
    → 「シグナル成功」として記録、ただし「実 entry 成功」とは別
```

### D31 9130 の case 例

- 前場 +4.89% (= 1,479 円)、追わない上限 1,440 円 超過
- 朝 entry 済 → 利確 (= 1,450-1,470 円目安到達、実 entry 成功 ✓)
- 未 entry → skip 確定 (= シグナル成功記録のみ、9247 へスライド)

---

## 藤原が iSPEED で見ること (= v1.4 から継承、7 軸)

1. **daily volume** (= 過去 20 day 平均): ≥ 10 万株 推奨 ★ actual 確認
2. **turnover (= 売買代金)**: ≥ 1 億円 推奨 ★ actual 確認
3. **opening volume (= 寄り 5 分)**: ≥ 1,000 株 ★ actual 確認
4. **spread + board depth**
5. **寄り後 5 min 出来高 + 上昇率** (= post-open 評価の核) ★
6. **ニュース / 適時開示 / catalyst** (= TDnet で根拠確認)
7. **前場 09:00-11:30 全体観測** (= 9130 等海運株で必須、entry_priority 再評価) ★

---

## 今日の review 必須 10 項目 (= v1.4.1 で 5 → 10 項目拡張)

### 既存 5 項目 (= v1.0-v1.4 継承)

1. **入った / 見送った** (= enter / watch / skip)
2. **判断理由**
3. **実際に見た銘柄** (= ticker / name + 33 sector)
4. **板・出来高・スプレッド所感**
5. **結果メモ**

### v1.4.1 追加 5 項目 (= 実弾検証可能化)

6. **実 entry か skip か** (= enter [約定済] / skip [入らず])
7. **入った価格 or 見送った理由** (= 約定価格 / 理由)
8. **追わない上限を超えていたか** (= Yes / No、Yes なら超過率)
9. **pre-open 判断か post-open 判断か** (= どちらの phase で entry/skip 決定)
10. **シグナル成功と実 entry 成功の区別** (= signal_success: Y/N、entry_success: Y/N の 2 軸)

review path: `~/fire-vault/04_daily/2026-06-25_manual_live_pilot_review.md`

---

## watch_candidate

0 件 (= D31 staging に該当銘柄なし)

## excluded_candidate (= 8 件、entry 不可)

| code | name | 除外理由 |
|---|---|---|
| 4389 | プロパティデータバンク | 信用 + 非 TOPIX |
| 4317 | レイ | 信用 + 非 TOPIX |
| 2700 | 木徳神糧 | 信用 + 非 TOPIX |
| 6149 | 小田原エンジニアリング | 信用 + 非 TOPIX |
| 2981 | ランディックス | 信用 + 非 TOPIX |
| 331A0 ★ | メディックス | letter-suffix 新規 + 信用 + 非 TOPIX |
| 288A0 | ラクサス・テクノロジーズ | letter-suffix 新規 + 信用 + 非 TOPIX |
| 339A0 | プログレス・テクノロジーズ | letter-suffix 新規 + 信用 + 非 TOPIX |

→ 331A0 / 4389 / 4317 全 excluded 維持 ✓

---

## reference (= QA 整合性)

- 連動 pilot day: D31 (= 2026-06-25 木) ✓
- file 名 date: 2026-06-25 = 本文 date ✓
- linked design: FIRE_selection_policy_v1.4.1_2026-05-15.md (= 設計発行日) ✓
- v1.0-v1.4 baseline 全保持 ✓
- 数値矛盾解消: pre_open_score と entry_priority 分離 ✓
- pilot judgment: GO_CONDITIONAL maintained 12 連続
- HOLD #2 完全回避: D15 以降 16 連続

---

## 安全確認

- 自動発注 0 / 楽天 0 / iSPEED 自動操作 0 / Computer Use 0
- LINE 送信 0 / token 0 / API 0 / DB write 0
- git commit 0 / git push 0 / --no-verify 0
- launchctl / plist / cron 変更 0
- F111 runner code 変更 0

## safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
