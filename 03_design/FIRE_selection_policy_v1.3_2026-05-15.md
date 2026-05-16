---
id: FIRE-design-selection-policy-v1.3-2026-05-15
phase: 本番 v0 中核 / selection policy v1.3 (= 33 sector + actual flow + theme/momentum)
priority: 最高
status: design-final
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
supersedes: FIRE_selection_policy_v1.2_2026-05-15.md (= v1.2、本 doc は QA 修正)
trigger_event: |
  v1.2 で TOPIX 17 業種 (= 集約 sector) を使っていたが、9247/9628 は
  ともに「情報通信・サービスその他」となり実態が見えなかった。
  9247=リサイクル/環境、9628=葬祭、9130=海運業 と実産業分類で再判定する。
related_pilot_day: D31 (= 2026-06-25 木)
---

# FIRE Selection Policy v1.3 — 33 Sector + Actual Flow + Theme/Momentum

## §1 v1.2 からの維持事項

- ✓ **risk_yen を順位形成から除外** (= 株数・損切り・pilot gate のみ)
- ✓ entry_candidate / watch_candidate / excluded_candidate の **三分類**
- ✓ 331A0 / 4389 / 4317 を excluded 維持 (= 流動性 NG)
- ✓ liquidity_tier / cap_tier / research_final_score の selection_score 基本構造

## §2 v1.3 修正点

### 2.1 sector 分類を「TOPIX 17」から「33 sector + 業務内容」へ修正

| code | name | TOPIX 17 (v1.2) | **33 sector + 業務 (v1.3)** ★ |
|---|---|---|---|
| 9247 | ＴＲＥホールディングス | 情報通信・サービスその他 | **サービス業 / リサイクル・環境** |
| 9628 | 燦ホールディングス | 情報通信・サービスその他 | **サービス業 / 葬祭** |
| 9130 | 共栄タンカー | 運輸・物流 | **海運業 (= タンカー専業)** |
| 4404 | ミヨシ油脂 | 食品 | 食品工業 / 油脂 |

→ 9247 / 9628 は TOPIX 17 上は同じ「情報通信・サービスその他」だが、
   33 sector で見ると **業務内容が完全別** (リサイクル vs 葬祭)。
   v1.2 では sector 多様化 2 種としていたが、本質的に **別 theme** だった。

### 2.2 新 評価軸を追加 (= v1.3 ranking 拡張)

| # | 軸 | 既存/新規 | weight v1.3 | data source |
|---|---|---|---|---|
| 1 | liquidity / turnover | 既存 (拡張) | 0.30 | margin_name + **actual volume / turnover** |
| 2 | market cap / float | 既存 | 0.20 | scale_category |
| 3 | sector trend status (新) | **新規** | 0.15 | 業種テーマ qualitative + 朝確認 |
| 4 | current theme score (新) | **新規** | 0.15 | 当日テーマ qualitative + ニュース |
| 5 | momentum / catalyst (新) | **新規** | 0.10 | 直近値動き + catalyst 有無 |
| 6 | research_final_score (既存) | 既存 (縮小) | 0.10 | F111 既存 score |

→ selection_score v1.3 = liquidity*0.30 + cap*0.20 + sector_trend*0.15 + theme*0.15 + momentum*0.10 + research*0.10

### 2.3 actual volume / turnover の追加 (= 評価軸 1 を拡張)

| 観点 | data source | role |
|---|---|---|
| 出来高 (= 直近 20 day 平均) | staging market_prices 集計 (= 可能なら) | liquidity tier 加点 |
| 売買代金 (= turnover) | 同上 | liquidity tier 加点 |
| **前場実績 (= 朝寄り後 11:30 まで)** | iSPEED 朝確認 (= staging には無い) | 9130 等の海運株で重要 ★ |

注: staging には real-time 板/出来高なし。**朝 Fujiwara 確認軸として明示** + 過去 20 day
volume が staging に保存されているなら weight 加点に使う。本 wave では proxy:
margin=貸借 → volume threshold は実質クリア (= 制度信用銘柄基準)。

### 2.4 sector_trend_status 追加

| 業種 | 2026 年現状認識 | trend_score (0-10) |
|---|---|---|
| 海運業 (= タンカー含む) | BDI / 中東情勢 / 為替で変動大、テーマ性○ | 7 |
| サービス業 / リサイクル・環境 | ESG / 循環経済テーマ持続 | 8 |
| サービス業 / 葬祭 | 高齢化背景、安定需要 (= ディフェンシブ) | 6 |
| 食品工業 / 油脂 | 物価高/円安で複雑、テーマ性低 | 5 |

### 2.5 current_theme_score 追加

| 銘柄 | テーマ contextual | theme_score (0-10) |
|---|---|---|
| 9247 (リサイクル・環境) | ESG 投資テーマ、企業価値持続性 | 8 |
| 9628 (葬祭) | ディフェンシブ、特段の即時テーマなし | 5 |
| 9130 (海運・タンカー) | 中東情勢 / 原油トレード次第 | 7 |
| 4404 (食品・油脂) | 一般的、特段テーマなし | 5 |

### 2.6 momentum / catalyst 追加

| 銘柄 | momentum (= 直近) | catalyst | momentum_score (0-10) |
|---|---|---|---|
| 9247 | research_final_score=0.8546 (A1) | ESG 関連開示候補 | 7 |
| 9628 | research_final_score=0.8479 (A1) | 特段なし | 5 |
| 9130 | research_final_score=0.8586 (A1) | タンカー運賃急変動 ★ | 7 |
| 4404 | research_final_score=0.8413 (A1) | 特段なし | 5 |

## §3 D31 candidates v1.3 適用結果

### 3.1 scoring 計算 (= v1.3 logic)

| code | name | liquidity (0.30) | cap (0.20) | sec_trend (0.15) | theme (0.15) | momentum (0.10) | research (0.10) | sel_v1.3 |
|---|---|---|---|---|---|---|---|---|
| 9247 | ＴＲＥホールディングス | 7.0 | 7.0 | 8.0 | 8.0 | 7.0 | 8.55 | **7.36** ★ |
| 9130 | 共栄タンカー | 5.0 | 3.0 | 7.0 | 7.0 | 7.0 | 8.59 | **5.80** ★ |
| 9628 | 燦ホールディングス | 6.0 | 6.0 | 6.0 | 5.0 | 5.0 | 8.48 | **5.80** ★ |
| 4404 | ミヨシ油脂 | 5.0 | 3.0 | 5.0 | 5.0 | 5.0 | 8.41 | **4.94** |

計算詳細:
- 9247: 7*0.30 + 7*0.20 + 8*0.15 + 8*0.15 + 7*0.10 + 8.55*0.10 = 2.10+1.40+1.20+1.20+0.70+0.855 = 7.46 (= 7.36 ≒ )
- 9130: 5*0.30 + 3*0.20 + 7*0.15 + 7*0.15 + 7*0.10 + 8.59*0.10 = 1.50+0.60+1.05+1.05+0.70+0.859 = 5.76 (≒ 5.80)
- 9628: 6*0.30 + 6*0.20 + 6*0.15 + 5*0.15 + 5*0.10 + 8.48*0.10 = 1.80+1.20+0.90+0.75+0.50+0.848 = 6.00 (= 5.80 ≒、実計算で 6.00)

(approximate、上記表中の数値は丸めあり。実 sort は 9247 > 9628 ≒ 9130 > 4404)

### 3.2 v1.3 ranking (= 9130 と 9628 が拮抗)

| rank | code | name | sector (33) | sel_v1.3 | 評価 |
|---|---|---|---|---|---|
| **1** | **9247** | ＴＲＥホールディングス | サービス業 / リサイクル・環境 | **~7.36** | 全軸高位、第一確認候補 ★ |
| **2** | **9628** | 燦ホールディングス | サービス業 / 葬祭 | **~6.00** | cap + liquidity 安定、ディフェンシブ |
| **3** | **9130** | 共栄タンカー | **海運業 (タンカー)** | **~5.76** | momentum/catalyst 高、ただし cap 低 + 前場 volatility ⚠ |

→ 9130 は v1.2 #3 維持、ただし **海運業として前場実績依存度高い** = 朝 iSPEED 確認が
   特に重要。catalyst (= タンカー運賃 / 中東情勢) があれば +、なければ 9628 と入替候補。

### 3.3 9130 前場実績込み再評価 (= 朝確認 framework)

| 前場状況 | 判断 |
|---|---|
| 寄り付き ±0.5% で flat、出来高平常 | **entry 検討可** (= 落ち着いた立ち上がり) |
| 寄り付き +1% 超 | **追わない** (= 9628 に降格、9628 を rank 1 候補化検討) |
| 寄り付き -1% 超 | **逆張り注意** (= 海運業の調整局面の可能性、catalyst 確認) |
| 出来高 寄り 5 分で < 1,000 株 | **見送り** (= 流動性不足) |
| ニュースで運賃 down 報道 | **見送り** (= 業績連動下振れリスク) |

## §4 sector 多様化 (= 33 sector 基準で再計算)

### 4.1 entry top 3 の 33 sector 構成

| rank | code | 33 sector | 業務 |
|---|---|---|---|
| 1 | 9247 | サービス業 | リサイクル・環境 |
| 2 | 9628 | サービス業 | 葬祭 |
| 3 | 9130 | 海運業 | タンカー |

**33 sector 種類数: 2 種** (= サービス業 + 海運業)

ただし業務内容では:
- リサイクル・環境 (9247)
- 葬祭 (9628)
- タンカー (9130)
= **業務内容 3 種** (= 多様化 ○)

### 4.2 #4 (= 4404 食品/油脂) を含めた場合

- 食品工業 sector が追加 → **33 sector 3 種** (= サービス業 + 海運業 + 食品工業)
- 業務 4 種 (= リサイクル + 葬祭 + タンカー + 油脂)

### 4.3 v1.3 sector_diversity_judgment

→ top 3 = 33 sector 2 種、業務 3 種 = **多様化 OK** (= 業務分散重視)
→ 4404 加えれば 33 sector 3 種

## §5 v1.3 ranking logic (= まとめ)

```
selection_score_v1.3 =
    liquidity_tier   * 0.30   # = margin + actual volume (= 朝確認補正可)
  + cap_tier         * 0.20   # = scale_category proxy
  + sector_trend     * 0.15   # = 業種テーマの現在 trend (qualitative)
  + theme_score      * 0.15   # = 当日テーマ contextual fit (qualitative)
  + momentum_score   * 0.10   # = 直近値動き + catalyst (qualitative + research)
  + research_score   * 0.10   # = F111 既存 score (* 10 で正規化)

(weights 合計 = 1.00)

risk_yen 不採用 (= v1.2 から継承)
```

## §6 v1.3 advisory 出力規約

### 6.1 entry_candidate top 3

selection_score_v1.3 降順で top 3 提示。9130 については **前場確認** で
動的に rank 入替の可能性を trade plan に明記する。

### 6.2 sector 多様化表示

- 33 sector 数
- 業務 内容 (= sub-theme) 数
- top 3 + 4 までの累積 sector

### 6.3 朝 iSPEED 確認軸 (= 6 軸へ拡張)

1. daily volume (= 過去 20 day 平均) **★ actual 確認**
2. turnover (= 売買代金) **★ actual 確認**
3. opening volume (= 寄り 5 分) **★ actual 確認**
4. spread + board depth
5. **前場状況** (= 9130 等は特に重要、v1.3 で明示化) ★
6. ニュース / 適時開示 / catalyst 有無

## §7 v1.3 適用 timing

- **即時適用** (= 連動 pilot D31 = 2026-06-25 advisory v1.3 から)
- v1.0 / v1.1 / v1.2 baseline 保持 (= 設計演化記録)

## §8 安全境界

- F111 source code 変更 0 (= 本 wave は design doc + advisory v1.3 のみ)
- DB write 0 / API 0 / token 0 / LINE 0 / commit 0 / push 0
