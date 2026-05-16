---
template: morning-advisory-summary
version: 1.1
hotfix_applied: liquidity_gate_v1.0
date: 2026-06-25
weekday: 木
owner: BlueFire7777 (Fujiwara)
linked_pilot_day: D31
linked_pilot_results: 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_results.md
linked_design: 03_design/FIRE_liquidity_gate_hotfix_2026-05-15.md
supersedes: 04_daily/2026-06-25_morning_advisory_summary.md (= v1.0、baseline 保持)
pilot_judgment: GO_CONDITIONAL
post_recovery: 12 連続 maintained
hold_2_avoidance: 完全継続 (= D15 以降 16 連続)
demoted_count: 7
purpose: "v1.1 = liquidity gate 適用後の entry top 3。Fujiwara が iSPEED で手動確認するための朝サマリ。発注指示ではない。"
qa_consistency:
  date_matches_filename: true (= 2026-06-25 統一)
  date_matches_pilot_day: true (= D31 = 2026-06-25 木)
---

# 【FIRE 朝の銘柄通知サマリ v1.1】 — 2026-06-25 (木) / D31 + liquidity gate hotfix

## 判定

**GO_CONDITIONAL** (= pilot D31 で 12 連続維持、HOLD #2 完全回避 16 連続)

⚠ hotfix 適用: v1.0 の 331A0 / 4389 (= 信用 + 非 TOPIX) は **流動性 FAIL**
で entry から除外、watch only に降格。

## entry_candidate top 3 (= liquidity_status = PASS のみ) ★

| # | code | name | sector | margin | scale | close | risk_yen | liquidity |
|---|---|---|---|---|---|---|---|---|
| **1** | **9130** | 共栄タンカー | **運輸・物流** | 貸借 | - | 1,410 | 7,050 | **PASS** (= 貸借 = 制度信用) |
| **2** | **9247** | ＴＲＥホールディングス | **情報通信** | 貸借 | TOPIX Small 1 | 1,612 | 8,060 | **PASS** (= 貸借 + TOPIX) |
| **3** | **9628** | 燦ホールディングス | **情報通信** | 貸借 | TOPIX Small 2 | 1,390 | 6,950 | **PASS** (= 貸借 + TOPIX) |

### 第一確認候補
**9130 共栄タンカー** (= 運輸 sector、liquidity PASS、D30-D31 で rank 1 2 連続安定)

(参考) #4: 4404 ミヨシ油脂 (食品、貸借) — 食品 sector で 3 種多様化候補

## watch_candidate (= 流動性 FAIL、原則 entry 不可)

| code | name | sector | margin | scale | letter-suffix? | status |
|---|---|---|---|---|---|---|
| 331A0 | メディックス | 情報通信 | 信用 | - | YES | **FAIL** (= 信用 + 非 TOPIX + 新規上場) |
| 4389 | プロパティデータバンク | 情報通信 | 信用 | - | no | **FAIL** (= 信用 + 非 TOPIX) |
| 4317 | レイ | 情報通信 | 信用 | - | no | **FAIL** |
| 2700 | 木徳神糧 | 商社 | 信用 | - | no | **FAIL** |
| 6149 | 小田原エンジニアリング | 機械 | 信用 | - | no | **FAIL** |
| 288A0 | ラクサス・テクノロジーズ | 情報通信 | 信用 | - | YES | **FAIL** |

→ これらは **flag のみ、entry 不可** (= 朝確認で例外的に基準満たした場合のみ Fujiwara 明示判断)

---

## 【注文完成形アドバイザリ v1.1】 (= entry_candidate のみ)

⚠ 発注指示ではない。Fujiwara が iSPEED で手動確認するための目安。

### 候補 1: 9130 共栄タンカー (運輸・物流、貸借)

- 銘柄: **9130 共栄タンカー**
- liquidity_status: **PASS** (= 貸借 = 制度信用銘柄、流動性十分)
- 基準株価: **1,410 円**
- エントリー検討価格: **1,395 - 1,425 円** (= 基準 ±1%)
- 指値目安: **1,410 円**
- 追わない上限: **1,440 円** (= 基準 +2.1%、寄り急騰時)
- 利確目安: **1,450 - 1,470 円** (= +3-4%)
- 損切り目安: **1,340 円** (= 基準 -70 円、risk_yen=7,050 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **7,050 円**
- 見送り条件:
  - 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,440 円超) / 悪材料

### 候補 2: 9247 ＴＲＥホールディングス (情報通信、貸借 + TOPIX Small 1)

- 銘柄: **9247 ＴＲＥホールディングス**
- liquidity_status: **PASS** (= 貸借 + TOPIX Small 1)
- 基準株価: **1,612 円**
- エントリー検討価格: **1,596 - 1,628 円** (= 基準 ±1%)
- 指値目安: **1,612 円**
- 追わない上限: **1,644 円** (= 基準 +2.0%)
- 利確目安: **1,660 - 1,693 円** (= +3-5%)
- 損切り目安: **1,532 円** (= 基準 -80 円、risk_yen=8,060 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **8,060 円**
- 見送り条件:
  - 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,644 円超) / 悪材料

### 候補 3: 9628 燦ホールディングス (情報通信、貸借 + TOPIX Small 2)

- 銘柄: **9628 燦ホールディングス** (= 新登場 entry 候補、v1.1 で promote)
- liquidity_status: **PASS** (= 貸借 + TOPIX Small 2)
- 基準株価: **1,390 円**
- エントリー検討価格: **1,376 - 1,404 円** (= 基準 ±1%)
- 指値目安: **1,390 円**
- 追わない上限: **1,418 円** (= 基準 +2.0%)
- 利確目安: **1,432 - 1,460 円** (= +3-5%)
- 損切り目安: **1,320 円** (= 基準 -70 円、risk_yen=6,950 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **6,950 円**
- 見送り条件:
  - 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,418 円超) / 悪材料

---

## 藤原が iSPEED で見ること (= 寄り 09:00 前後、liquidity 強化版)

### 必須確認 5 軸 (= 1 つでも FAIL なら見送り)

1. **daily volume** (= 20 day 平均): ≥ 10 万株 推奨
2. **turnover (= 売買代金)**: ≥ 1 億円 推奨
3. **opening volume (= 寄り 5 分)**: ≥ 1,000 株
4. **spread (= 板気配差)**: ≤ 5 円 (small-cap は ≤ 2 円)
5. **board depth (= best bid/ask)**: ≥ 100 株

### 追加確認

6. **寄り付きからの上昇率** (= 追わない上限を超えていないか)
7. **ニュース / 適時開示** (= 朝 8:30 JST TDnet で悪材料がないか)
8. **約定しやすさ** (= 指値が即時 fill するか、滑り懸念がないか)

---

## 今日の review 必須 5 項目

1. **入った / 見送った** (= enter / watch / skip)
2. **判断理由**
3. **実際に見た銘柄** (= ticker / name)
4. **板・出来高・スプレッド所感** (= 朝の流動性メモ、liquidity gate v1.0 観点)
5. **結果メモ**

review path: `~/fire-vault/04_daily/2026-06-25_manual_live_pilot_review.md`

---

## 安全確認

- 自動発注 0 / 楽天 0 / iSPEED 自動操作 0 / Computer Use 0
- LINE 送信 0 / token 0 / API 0 / DB write 0
- git commit 0 / git push 0 / --no-verify 0
- launchctl / plist / cron 変更 0
- F111 runner code 変更 0 (= 本 wave は design doc + advisory v1.1 のみ)

## safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
