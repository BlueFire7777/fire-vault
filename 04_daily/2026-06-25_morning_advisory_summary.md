---
template: morning-advisory-summary
version: 1.0
date: 2026-06-25
weekday: 木
owner: BlueFire7777 (Fujiwara)
linked_pilot_day: D31
linked_pilot_results: 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_results.md
linked_pilot_trade_plan: 04_daily/2026-06-25_manual_live_pilot_trade_plan.md
linked_pilot_review: 04_daily/2026-06-25_manual_live_pilot_review.md
pilot_judgment: GO_CONDITIONAL
post_recovery: 12 連続 maintained
hold_2_avoidance: 完全継続 (= D15 以降 16 連続)
demoted_count: 7
purpose: "Fujiwara が iSPEED で手動確認するための朝サマリ。発注指示ではない。"
qa_consistency:
  date_matches_filename: true (= 2026-06-25 統一)
  date_matches_pilot_day: true (= D31 = 2026-06-25 木)
  reference_date_matches: true
---

# 【FIRE 朝の銘柄通知サマリ】 — 2026-06-25 (木) / D31

## 判定

**GO_CONDITIONAL** (= pilot D31 で 12 連続維持、HOLD #2 完全回避 16 連続)

## 本日の優先確認候補 (= D31 F111-real-batch top 3)

| # | code | name | sector | close | risk_yen |
|---|---|---|---|---|---|
| 1 | **9130** | 共栄タンカー | **運輸・物流** | 1,410 | 7,050 |
| 2 | **331A0** | メディックス | **情報通信** | 482 | 2,410 |
| 3 | **4389** | プロパティデータバンク | **情報通信** | 863 | 4,315 |

### 第一確認候補
**9130 共栄タンカー** (= 運輸 sector、D30-D31 で rank 1 2 連続安定)

---

## 【注文完成形アドバイザリ】

⚠ これは発注指示ではない。Fujiwara が iSPEED で手動確認するための目安。
auto_order_allowed=False / manual_review=True / Computer Use 不採用。

### 候補 1: 9130 共栄タンカー (運輸・物流)

- 銘柄: **9130 共栄タンカー**
- 基準株価: **1,410 円** (= 直近 staging close)
- エントリー検討価格: **1,395 - 1,425 円** (= 基準 ±1%)
- 指値目安: **1,410 円**
- 追わない上限: **1,440 円** (= 基準 +2.1%、寄り急騰時)
- 利確目安: **1,450 - 1,470 円** (= +3-4%)
- 損切り目安: **1,340 円** (= 基準 -70 円、risk_yen=7,050 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **7,050 円** (= F130 許容損失 0.4% 準拠、pilot 限度内)
- 見送り条件:
  - 板が薄い (= best bid/ask 100 株未満)
  - 出来高が少ない (= 寄り 5 分以内に 1,000 株未満)
  - スプレッドが広い (= 5 円以上)
  - 寄り直後に急騰しすぎ (= 1,440 円超え)
  - 悪材料/決算/ニュースあり

### 候補 2: 331A0 メディックス (情報通信)

- 銘柄: **331A0 メディックス**
- 基準株価: **482 円**
- エントリー検討価格: **478 - 487 円** (= 基準 ±1%)
- 指値目安: **482 円**
- 追わない上限: **492 円** (= 基準 +2.1%)
- 利確目安: **497 - 506 円** (= +3-5%)
- 損切り目安: **458 円** (= 基準 -24 円、risk_yen=2,410 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **2,410 円** (= pilot 最小 risk、entry 試行向け)
- 見送り条件:
  - 板が薄い (= small-cap 注意)
  - 出来高が少ない
  - スプレッドが広い (= 2 円以上で警戒)
  - 寄り直後に急騰しすぎ (= 492 円超え)
  - 悪材料/決算/ニュースあり

### 候補 3: 4389 プロパティデータバンク (情報通信)

- 銘柄: **4389 プロパティデータバンク** (= D30 新登場、D31 で 2 連続維持)
- 基準株価: **863 円**
- エントリー検討価格: **854 - 872 円** (= 基準 ±1%)
- 指値目安: **863 円**
- 追わない上限: **880 円** (= 基準 +2.0%)
- 利確目安: **889 - 906 円** (= +3-5%)
- 損切り目安: **820 円** (= 基準 -43 円、risk_yen=4,315 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **4,315 円**
- 見送り条件:
  - 板が薄い (= 新顔銘柄、流動性確認必須)
  - 出来高が少ない (= 1,000 株/5 分未満)
  - スプレッドが広い (= 3 円以上)
  - 寄り直後に急騰しすぎ (= 880 円超え)
  - 悪材料/決算/ニュースあり

---

## 藤原が iSPEED で見ること (= 2026-06-25 寄り 09:00 前後)

1. **板** (= 売り買い 5 本値の厚み、特に best bid/ask 100 株以上か)
2. **出来高** (= 寄り 5 分以内で 1,000 株以上の流れがあるか)
3. **スプレッド** (= 5 円以下、small-cap は 2 円以下)
4. **寄り付きからの上昇率** (= 追わない上限を超えていないか)
5. **ニュース / 適時開示** (= 朝 8:30 JST TDnet で悪材料がないか)
6. **約定しやすさ** (= 指値が即時 fill するか、滑り懸念がないか)

---

## 今日の review 必須 5 項目 (= 場後 review.md に必ず記入)

1. **入った / 見送った** (= enter / watch / skip)
2. **判断理由** (= なぜ entry したか、または skip したか)
3. **実際に見た銘柄** (= ticker / name)
4. **板・出来高・スプレッド所感** (= 朝の流動性メモ)
5. **結果メモ** (= entry 後の値動き、または skip 後の銘柄推移)

review path: `~/fire-vault/04_daily/2026-06-25_manual_live_pilot_review.md`
(= 既存 D31 review template、必須 5 項目欄あり)

---

## reference (= QA 整合性確認済)

- 連動 pilot day: **D31** (= 2026-06-25 木 pilot 想定) ✓
- file 名 date: **2026-06-25** = 本文 date ✓
- linked_pilot_day reference date: **2026-06-25** = 本サマリ date ✓
- pilot judgment: GO_CONDITIONAL maintained 12 連続
- HOLD #2 完全回避: D15 以降 16 連続
- demoted_count: 7 (= 8747,5729,3489,340A0,3798,137A0,7991)
- 7991 demote 効果: 維持 (= caution、rank 7、top 5 除外)
- sector 多様化: 2 種 (= 運輸 + 情報通信、機械喪失継続)
- 次の demote 候補: 9130 (= D34 = 2026-06-30 火 で 5 連続 warning、D35 = 2026-07-01 水 で本実行想定)

---

## 安全確認

- 自動発注: **0** (= 全 entry は Fujiwara 手動)
- 楽天証券 / iSPEED 操作自動化: **0** (= Fujiwara 手動のみ)
- LINE 送信: **0** (= 本サマリは file 保存 + 画面表示のみ)
- token / API / DB write: **0** (= read-only)
- git commit / push: **0**
- production / develop / staging DB write: **0**
- launchctl / plist / cron 変更: **0**

## safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
