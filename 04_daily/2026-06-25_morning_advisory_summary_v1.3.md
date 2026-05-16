---
template: morning-advisory-summary
version: 1.3
policy_applied: selection_policy_v1.3 (= 33 sector + actual flow + theme/momentum)
date: 2026-06-25
weekday: 木
owner: BlueFire7777 (Fujiwara)
linked_pilot_day: D31
linked_pilot_results: 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_results.md
linked_design: 03_design/FIRE_selection_policy_v1.3_2026-05-15.md
supersedes:
  - 04_daily/2026-06-25_morning_advisory_summary.md (= v1.0)
  - 04_daily/2026-06-25_morning_advisory_summary_v1.1.md (= v1.1)
  - 04_daily/2026-06-25_morning_advisory_summary_v1.2.md (= v1.2)
pilot_judgment: GO_CONDITIONAL
post_recovery: 12 連続 maintained
hold_2_avoidance: 完全継続 (= D15 以降 16 連続)
demoted_count: 7
purpose: "v1.3 = 33 sector + 6 軸加重 (liquidity/cap/sector_trend/theme/momentum/research) で entry top 3 を再算出"
qa_consistency:
  date_matches_filename: true (= 2026-06-25 統一)
  date_matches_pilot_day: true (= D31 = 2026-06-25 木)
---

# 【FIRE 朝の銘柄通知サマリ v1.3】 — 2026-06-25 (木) / D31 + 33 sector + 6 軸加重

## 判定

**GO_CONDITIONAL** (= pilot D31 で 12 連続維持、HOLD #2 完全回避 16 連続)

⚠ v1.3 改訂: sector を「TOPIX 17 集約」から **「33 sector + 業務内容」** へ修正。
9247=リサイクル、9628=葬祭、9130=海運。さらに評価軸を 6 軸に拡張
(= liquidity / cap / sector_trend / theme / momentum / research)。

## v1.3 ranking 基準

```
selection_score_v1.3 =
    liquidity_tier   * 0.30   # = margin + actual volume / turnover
  + cap_tier         * 0.20   # = scale_category proxy
  + sector_trend     * 0.15   # = 33 sector の現在 trend
  + theme_score      * 0.15   # = 当日テーマ contextual fit
  + momentum_score   * 0.10   # = 直近値動き + catalyst
  + research_score   * 0.10   # = F111 既存 score

(risk_yen は非加点、損切り設計のみ)
```

## entry_candidate top 3 (= v1.3 selection_score 降順) ★

| # | code | name | **33 sector / 業務** | margin | scale | sel_v1.3 | close | risk_yen |
|---|---|---|---|---|---|---|---|---|
| **1** | **9247** | ＴＲＥホールディングス | **サービス業 / リサイクル・環境** | 貸借 | TOPIX Small 1 | **~7.36** ★ | 1,612 | 8,060 |
| **2** | **9628** | 燦ホールディングス | **サービス業 / 葬祭** | 貸借 | TOPIX Small 2 | **~6.00** | 1,390 | 6,950 |
| **3** | **9130** | 共栄タンカー | **海運業 / タンカー** | 貸借 | - | **~5.76** | 1,410 | 7,050 |

(参考) #4: 4404 ミヨシ油脂 (= 食品工業 / 油脂、貸借、sel≒4.94) — 食品 sector で 3 種多様化候補

### 第一確認候補
**9247 ＴＲＥホールディングス** (= リサイクル・環境、ESG テーマ、貸借 + TOPIX Small 1)

### sector 多様化 (= 33 sector 基準で再計算)

- top 3 = **2 種 (サービス業 + 海運業)**、業務内容 **3 種** (リサイクル + 葬祭 + タンカー)
- #4 含めると 33 sector **3 種** (+ 食品工業)
- v1.2 で「同 sector 集中」と見えていた 9247/9628 は **業務内容で完全別**

## watch_candidate

D31 staging F111 出力に該当 0 件 (= 信用 + TOPIX Small 構成銘柄なし)

## excluded_candidate (= 8 件、entry 不可)

| code | name | sector | 除外理由 |
|---|---|---|---|
| 4389 | プロパティデータバンク | 情報通信 | 信用 + 非 TOPIX |
| 4317 | レイ | 情報通信 | 信用 + 非 TOPIX |
| 2700 | 木徳神糧 | 商社 | 信用 + 非 TOPIX |
| 6149 | 小田原エンジニアリング | 機械 | 信用 + 非 TOPIX |
| 2981 | ランディックス | 不動産 | 信用 + 非 TOPIX |
| 331A0 | メディックス | 情報通信 | letter-suffix 新規 + 信用 + 非 TOPIX ★ |
| 288A0 | ラクサス・テクノロジーズ | 情報通信 | letter-suffix 新規 + 信用 + 非 TOPIX |
| 339A0 | プログレス・テクノロジーズ・グループ | 情報通信 | letter-suffix 新規 + 信用 + 非 TOPIX |

→ 331A0 / 4389 / 4317 全 excluded 維持 ✓

---

## 【注文完成形アドバイザリ v1.3】 (= entry_candidate のみ)

⚠ 発注指示ではない。Fujiwara が iSPEED で手動確認するための目安。

### 候補 1: 9247 ＴＲＥホールディングス (サービス業 / リサイクル・環境) ★ 第一確認候補

- liquidity_status: PASS (= 貸借 + TOPIX Small 1)
- sector_trend: ○ (= ESG / 循環経済テーマ持続)
- theme_score: 高 (= 8/10、ESG 投資テーマ)
- momentum / catalyst: ESG 関連開示 catalyst あり
- 基準株価: **1,612 円**
- エントリー検討価格: **1,596 - 1,628 円** (= 基準 ±1%)
- 指値目安: **1,612 円**
- 追わない上限: **1,644 円** (= +2.0%)
- 利確目安: **1,660 - 1,693 円** (= +3-5%)
- 損切り目安: **1,532 円** (= -80 円、risk_yen=8,060 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **8,060 円**
- 見送り条件: 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,644 円超) / ESG 後退ニュース

### 候補 2: 9628 燦ホールディングス (サービス業 / 葬祭)

- liquidity_status: PASS (= 貸借 + TOPIX Small 2)
- sector_trend: 中 (= 高齢化背景、安定需要)
- theme_score: 中 (= ディフェンシブ、即時テーマなし)
- momentum / catalyst: 特段なし
- 基準株価: **1,390 円**
- エントリー検討価格: **1,376 - 1,404 円**
- 指値目安: **1,390 円**
- 追わない上限: **1,418 円** (= +2.0%)
- 利確目安: **1,432 - 1,460 円** (= +3-5%)
- 損切り目安: **1,320 円** (= -70 円、risk_yen=6,950 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **6,950 円**
- 見送り条件: 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,418 円超) / 悪材料

### 候補 3: 9130 共栄タンカー (海運業 / タンカー) ⚠ 前場確認必須

- liquidity_status: PASS (= 貸借、ただし scale=-)
- sector_trend: ○ (= 海運業、BDI / 中東情勢で変動大)
- theme_score: 中-高 (= タンカー運賃 / 原油トレード)
- momentum / catalyst: タンカー運賃急変動が catalyst 候補 ★
- 基準株価: **1,410 円**
- エントリー検討価格: **1,395 - 1,425 円**
- 指値目安: **1,410 円**
- 追わない上限: **1,440 円** (= +2.1%)
- 利確目安: **1,450 - 1,470 円** (= +3-4%)
- 損切り目安: **1,340 円** (= -70 円、risk_yen=7,050 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **7,050 円**
- 見送り条件:
  - 板薄 / 出来高小 / spread 広
  - **寄り付き +1% 超 (= 1,425 円超え)** → **追わない、9628 を rank 1 候補化検討**
  - **寄り付き -1% 超** → 逆張り注意、catalyst 確認 (= 海運業の調整局面リスク)
  - 寄り 5 分 < 1,000 株
  - タンカー運賃 down 報道

#### 9130 前場判定 framework (= v1.3 で明示化)

| 前場状況 | 判断 |
|---|---|
| 寄り付き ±0.5% で flat、出来高平常 | **entry 検討可** |
| 寄り付き +1% 超 | **追わない**、9628 promote |
| 寄り付き -1% 超 | **逆張り注意**、catalyst 確認 |
| 寄り 5 分 < 1,000 株 | **見送り** |
| 運賃 down ニュース | **見送り** |

---

## 藤原が iSPEED で見ること (= 寄り 09:00 前後、v1.3 で 6 軸に拡張)

1. **daily volume** (= 過去 20 day 平均): ≥ 10 万株 推奨 ★ actual 確認
2. **turnover (= 売買代金)**: ≥ 1 億円 推奨 ★ actual 確認
3. **opening volume (= 寄り 5 分)**: ≥ 1,000 株 ★ actual 確認
4. **spread + board depth**: spread ≤ 5 円 (small ≤ 2 円)、best bid/ask ≥ 100 株
5. **前場状況 (= 寄り上昇率 + catalyst)** ★ 9130 等海運株で特に重要
6. **ニュース / 適時開示 / catalyst 有無** (= 8:30 JST TDnet、9247 ESG 関連、9130 海運関連)

---

## 今日の review 必須 5 項目

1. **入った / 見送った** (= enter / watch / skip)
2. **判断理由** (= sector / theme / catalyst のどれが決め手か)
3. **実際に見た銘柄** (= ticker / name + 33 sector 認識)
4. **板・出来高・スプレッド所感 + 前場実績**
5. **結果メモ**

review path: `~/fire-vault/04_daily/2026-06-25_manual_live_pilot_review.md`

---

## reference (= QA 整合性)

- 連動 pilot day: D31 (= 2026-06-25 木) ✓
- file 名 date: 2026-06-25 = 本文 date ✓
- linked design: FIRE_selection_policy_v1.3_2026-05-15.md ✓
- v1.0 / v1.1 / v1.2 baseline 保持 ✓
- pilot judgment: GO_CONDITIONAL maintained 12 連続
- HOLD #2 完全回避: D15 以降 16 連続
- demoted_count: 7 (= 8747,5729,3489,340A0,3798,137A0,7991)

---

## 安全確認

- 自動発注 0 / 楽天 0 / iSPEED 自動操作 0 / Computer Use 0
- LINE 送信 0 / token 0 / API 0 / DB write 0
- git commit 0 / git push 0 / --no-verify 0
- launchctl / plist / cron 変更 0
- F111 runner code 変更 0 (= 本 wave は design doc + advisory v1.3 のみ)

## safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
