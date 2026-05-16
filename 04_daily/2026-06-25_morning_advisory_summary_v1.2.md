---
template: morning-advisory-summary
version: 1.2
policy_applied: selection_policy_v1.2 (= risk_yen 非加点 + liquidity/cap/momentum 加重)
date: 2026-06-25
weekday: 木
owner: BlueFire7777 (Fujiwara)
linked_pilot_day: D31
linked_pilot_results: 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_results.md
linked_design: 03_design/FIRE_selection_policy_v1.2_2026-05-15.md
supersedes:
  - 04_daily/2026-06-25_morning_advisory_summary.md (= v1.0 baseline)
  - 04_daily/2026-06-25_morning_advisory_summary_v1.1.md (= v1.1 liquidity gate)
pilot_judgment: GO_CONDITIONAL
post_recovery: 12 連続 maintained
hold_2_avoidance: 完全継続 (= D15 以降 16 連続)
demoted_count: 7
purpose: "v1.2 = risk_yen 非加点 + liquidity/cap/momentum 加重で再選定した entry top 3。"
qa_consistency:
  date_matches_filename: true (= 2026-06-25 統一)
  date_matches_pilot_day: true (= D31 = 2026-06-25 木)
---

# 【FIRE 朝の銘柄通知サマリ v1.2】 — 2026-06-25 (木) / D31 + selection policy v1.2

## 判定

**GO_CONDITIONAL** (= pilot D31 で 12 連続維持、HOLD #2 完全回避 16 連続)

⚠ policy 改訂: v1.0/v1.1 は risk_yen の暗黙加点があった。v1.2 では
**risk_yen は順位形成に使わず**、liquidity / cap / momentum / entry feasibility
で順位を決める。

## v1.2 ranking 基準

```
selection_score = liquidity * 0.40 + cap * 0.25 + (research * 10) * 0.25 + feasibility * 0.10

  liquidity_tier:
    貸借 + TOPIX Core30:    10
    貸借 + TOPIX Large70:    9
    貸借 + TOPIX Mid400:     8
    貸借 + TOPIX Small 1:    7
    貸借 + TOPIX Small 2:    6
    貸借 + scale=-:          5
    信用 + TOPIX Small 1:    4
    信用 + TOPIX Small 2:    3
    信用 + scale=-:          1
    letter-suffix 新規 + 非 TOPIX: 0

  cap_tier:
    TOPIX Core30/Large70/Mid400/Small 1/Small 2/-:  10/9/8/7/6/3
```

risk_yen は **損切り設計とポジションサイズの制約のみ** (= 順位形成に使わない)。

## entry_candidate top 3 (= v1.2 selection_score 降順) ★

| # | code | name | sector | margin | scale | sel_score | close | risk_yen |
|---|---|---|---|---|---|---|---|---|
| **1** | **9247** | ＴＲＥホールディングス | **情報通信** | 貸借 | TOPIX Small 1 | **7.39** | 1,612 | 8,060 |
| **2** | **9628** | 燦ホールディングス | **情報通信** | 貸借 | TOPIX Small 2 | **6.62** | 1,390 | 6,950 |
| **3** | **9130** | 共栄タンカー | **運輸・物流** | 貸借 | - | **5.20** | 1,410 | 7,050 |

(参考) #4: 4404 ミヨシ油脂 (食品、貸借、sel=5.15) — 食品 sector で 3 種多様化候補

### 第一確認候補
**9247 ＴＲＥホールディングス** (= 貸借 + TOPIX Small 1、liquidity + cap 共に最上位)

## watch_candidate (= 信用 + TOPIX Small)

D31 staging F111 出力で該当銘柄 **0 件** (= 信用 + TOPIX Small 構成銘柄なし)

## excluded_candidate (= 信用 + 非 TOPIX or letter-suffix 新規上場 + 非 TOPIX)

| code | name | sector | margin | scale | letter-suffix? | 除外理由 |
|---|---|---|---|---|---|---|
| 4389 | プロパティデータバンク | 情報通信 | 信用 | - | no | 信用 + 非 TOPIX |
| 4317 | レイ | 情報通信 | 信用 | - | no | 信用 + 非 TOPIX |
| 2700 | 木徳神糧 | 商社 | 信用 | - | no | 信用 + 非 TOPIX |
| 6149 | 小田原エンジニアリング | 機械 | 信用 | - | no | 信用 + 非 TOPIX |
| 2981 | ランディックス | 不動産 | 信用 | - | no | 信用 + 非 TOPIX |
| **331A0** | **メディックス** | 情報通信 | 信用 | - | **YES** | **letter-suffix 新規 + 信用 + 非 TOPIX** ★ |
| 288A0 | ラクサス・テクノロジーズ | 情報通信 | 信用 | - | YES | letter-suffix 新規 + 信用 + 非 TOPIX |
| 339A0 | プログレス・テクノロジーズ・グループ | 情報通信 | 信用 | - | YES | letter-suffix 新規 + 信用 + 非 TOPIX |

→ **entry 不可 / watch も非推奨** (= 朝確認で例外的に流動性基準満たした場合のみ Fujiwara 明示判断)

---

## 【注文完成形アドバイザリ v1.2】 (= entry_candidate のみ)

⚠ 発注指示ではない。Fujiwara が iSPEED で手動確認するための目安。

### 候補 1: 9247 ＴＲＥホールディングス (情報通信、貸借 + TOPIX Small 1) ★ 第一確認候補

- 銘柄: **9247 ＴＲＥホールディングス**
- liquidity_status: **PASS** (= 貸借 + TOPIX Small 1)
- selection_score: **7.39** (= v1.2 最高位)
- 基準株価: **1,612 円**
- エントリー検討価格: **1,596 - 1,628 円** (= 基準 ±1%)
- 指値目安: **1,612 円**
- 追わない上限: **1,644 円** (= 基準 +2.0%)
- 利確目安: **1,660 - 1,693 円** (= +3-5%)
- 損切り目安: **1,532 円** (= 基準 -80 円、risk_yen=8,060 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **8,060 円**
- 見送り条件: 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,644 円超) / 悪材料

### 候補 2: 9628 燦ホールディングス (情報通信、貸借 + TOPIX Small 2)

- 銘柄: **9628 燦ホールディングス**
- liquidity_status: **PASS** (= 貸借 + TOPIX Small 2)
- selection_score: **6.62**
- 基準株価: **1,390 円**
- エントリー検討価格: **1,376 - 1,404 円** (= 基準 ±1%)
- 指値目安: **1,390 円**
- 追わない上限: **1,418 円** (= 基準 +2.0%)
- 利確目安: **1,432 - 1,460 円** (= +3-5%)
- 損切り目安: **1,320 円** (= 基準 -70 円、risk_yen=6,950 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **6,950 円**
- 見送り条件: 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,418 円超) / 悪材料

### 候補 3: 9130 共栄タンカー (運輸・物流、貸借)

- 銘柄: **9130 共栄タンカー** (= v1.1 #1 から v1.2 #3 へ降格、cap 低のため)
- liquidity_status: **PASS** (= 貸借)
- selection_score: **5.20**
- 基準株価: **1,410 円**
- エントリー検討価格: **1,395 - 1,425 円** (= 基準 ±1%)
- 指値目安: **1,410 円**
- 追わない上限: **1,440 円** (= 基準 +2.1%)
- 利確目安: **1,450 - 1,470 円** (= +3-4%)
- 損切り目安: **1,340 円** (= 基準 -70 円、risk_yen=7,050 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **7,050 円**
- 見送り条件: 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,440 円超) / 悪材料

---

## 藤原が iSPEED で見ること (= 寄り 09:00 前後、liquidity 強化版)

### 必須確認 5 軸 (= 1 軸でも FAIL なら見送り)

1. **daily volume** (= 20 day 平均): ≥ 10 万株 推奨
2. **turnover (= 売買代金)**: ≥ 1 億円 推奨
3. **opening volume (= 寄り 5 分)**: ≥ 1,000 株
4. **spread (= 板気配差)**: ≤ 5 円 (small-cap は ≤ 2 円)
5. **board depth (= best bid/ask)**: ≥ 100 株

### 追加確認 3 軸

6. **寄り付きからの上昇率** (= 追わない上限を超えていないか)
7. **ニュース / 適時開示** (= 朝 8:30 JST TDnet)
8. **約定しやすさ** (= 指値即時 fill、滑り懸念)

---

## 今日の review 必須 5 項目

1. **入った / 見送った** (= enter / watch / skip)
2. **判断理由**
3. **実際に見た銘柄** (= ticker / name)
4. **板・出来高・スプレッド所感** (= liquidity gate v1.2 観点、新方針順位への所感)
5. **結果メモ**

review path: `~/fire-vault/04_daily/2026-06-25_manual_live_pilot_review.md`

---

## reference (= QA 整合性)

- 連動 pilot day: D31 (= 2026-06-25 木 pilot 想定) ✓
- file 名 date: 2026-06-25 = 本文 date ✓
- linked design: FIRE_selection_policy_v1.2_2026-05-15.md (= 設計 doc 自体は今日 2026-05-15 発行) ✓
- v1.0 / v1.1 baseline 保持 (= 削除しない)
- pilot judgment: GO_CONDITIONAL maintained 12 連続
- HOLD #2 完全回避: D15 以降 16 連続
- demoted_count: 7 (= 8747,5729,3489,340A0,3798,137A0,7991)
- sector 多様化: top 3 で 2 種 (運輸+情報通信)、#4 で 食品 加わり 3 種

---

## 安全確認

- 自動発注 0 / 楽天 0 / iSPEED 自動操作 0 / Computer Use 0
- LINE 送信 0 / token 0 / API 0 / DB write 0
- git commit 0 / git push 0 / --no-verify 0
- launchctl / plist / cron 変更 0
- F111 runner code 変更 0 (= 本 wave は design doc + advisory v1.2 のみ)

## safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
