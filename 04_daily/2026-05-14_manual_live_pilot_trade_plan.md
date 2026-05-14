---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-14
owner: BlueFire7777 (Fujiwara)
pilot_day: D1
input_source: synthetic_fixture (= /tmp/fire_w60_smoke/、実 F062 朝 batch 未稼働)
related:
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_PRE_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_results.md
---

# Manual Live Pilot — Trade Plan (2026-05-14 / D1)

## ⚠ 重要前提 (= D1 初日特記)

本 trade plan の入力 (= F062 preview / DATA-R3 freshness / Ops Summary) は
**synthetic fixture** (= W60-impl smoke で 5/13 23:56 に作成した hardcoded data) で生成。
**実 F062 朝 batch / 実 DATA-R3 batch / 実 TDnet 開示 ではない**。

そのため、**本 plan で藤原さんが実発注するかどうかは別判断**。
本 wave (W60-pilot-D1) は **plan 作成と運用フロー検証** までで、
実 trade は藤原さんが「synthetic でなく実 batch が稼働するまで待つ」「synthetic で
合致するなら追って実値で再判断」等を選択する。

---

## §1 基本情報

- **date**: 2026-05-14 (木曜日)
- **記入時刻**: 寄付き前 (= 本 plan は claude code が AM 0:48 JST に自動生成、
  藤原さんは寄付き前に確認)
- **本日の本業 / 監視可能時間**: ☐ 9:00-15:30 監視可 / ☐ 部分監視 / ☐ 監視不能
  → **藤原さんが当日朝に記入**
- **15:10 までに手動 close 可能か**: ☐ Yes / ☐ No (= 藤原さん記入)
- **大型イベント (FOMC / 雇用統計 / 重要 IR / 決算跨ぎ)**: ☐ なし / ☐ あり
  (= 藤原さんが TDnet / 日経でチェック)

## §2 市場コンディション (= 藤原さんが寄付き前に記入)

- 日経平均 寄付き前: __,___ (前日比 +__/-__)
- TOPIX 寄付き前: ____ (前日比 +__/-__)
- 米国 引け: NYダウ +__/-__、S&P +__/-__、NASDAQ +__/-__
- ドル円: ___.__ 円
- 自分の地合い judgement: ☐ 強気 / ☐ 中立 / ☐ 弱気 / ☐ 不明

## §3 FIRE report paths (= 生成済、claude code が AM 0:48 JST に生成)

- `reports/after_r1/morning_line_material_2026-05-14.md` ✓
- `reports/after_r1/good_candidate_ranking_2026-05-14.md` ✓
- `reports/after_r1/pattern_candidate_report_2026-05-14.md` ✓
- `reports/after_r1/paper_live_ledger_2026-05-14.md` ✓

### §3.1 共通 GO-check (= 全 PASS 確認済 ✓)

- [x] morning_line_material の `forbidden_phrases_check.passed = True` ✓
- [x] paper_live_ledger 全 entry で `auto_order_allowed = False` ✓
- [x] paper_live_ledger 全 entry で `manual_review_required = True` ✓
- [x] DATA-R3 freshness verdict = `OK` ✓ (= ただし synthetic fixture 由来)
- [x] ranking と morning_line_material の top ticker が **整合** (= 7203/6758/9984) ✓
- [x] 出力 path が `reports/after_r1/` 配下 ✓

**結論: GO 条件 (= claude code 側) 全 PASS。藤原さんの監視可否判断のみ残り。**

## §4 Top candidates (= morning_line_material から自動転記)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | **7203** | **トヨタ自動車** | 積極的買い推奨 | 🟢 | 0.85 | 194.0 |
| 2 | **6758** | **ソニーG** | 積極的買い推奨 | 🟢 | 0.72 | 179.0 |
| 3 | **9984** | **ソフトバンクG** | 条件付き買い推奨 | 🟡 | 0.65 | 138.0 |

### §4.1 全 candidates (= alternative_labels_recap)

ranking 全 6 件:

| rank | ticker | name | label | score |
|---|---|---|---|---|
| 1 | 7203 | トヨタ自動車 | boost 🟢 | 194.0 |
| 2 | 6758 | ソニーG | boost 🟢 | 179.0 |
| 3 | 9984 | ソフトバンクG | boost_with_caution 🟡 | 138.0 |
| 4 | 4502 | 武田薬品 | caution ⚠️ | (推定) |
| 5 | 8316 | 三井住友FG | avoid 🔴 | (推定) |
| 6 | 4543 | テルモ | suppress 🔴 | (推定) |

morning_line_material の top は #1-#3 (= 🟢🟢🟡)、#4 以降は別候補帯。

## §5 D1 推奨選択肢 (= 藤原さんが top 1-3 から手動選択)

### §5.1 推奨パターン (= 初日 1 銘柄 推奨)

**候補 A**: 🟢 7203 トヨタ自動車
- 最高 score (= 194.0) + 全 pattern matched (= freshness_ok_high_confidence /
  manual_review_active_label / multi_reason_basis / low_risk_note)
- conf = 0.85 (= 高)
- 根拠: 決算ポジティブ / 出来高伴う上昇 / 上方修正期待
- 流動性: 大型株 (= 板厚い、初日に適する)

**候補 B**: 🟢 6758 ソニーG
- score = 179.0、4 pattern matched
- conf = 0.72 (= 高)
- 根拠: 好材料連発 / 上昇トレンド / 出来高拡大
- 流動性: 大型株

**候補 C** (= 慎重): 🟡 9984 ソフトバンクG
- score = 138.0、2 pattern matched
- conf = 0.65 (= 中)、根拠 1 件のみ
- 初日には **不向き** (= 「条件付き」label、確信度低め)

### §5.2 推奨

**初日 D1 は 7203 (トヨタ自動車) 1 銘柄推奨** (= 最高 score / 4 pattern / 大型株流動性)。

ただし藤原さんが §6 以降を埋めて NO 判断したら **skip** で可。
**迷ったら見送り** (= operation_plan §11 原則)。

## §6 Selected ticker (= 藤原さん が top 1-3 から手動選択)

- **ticker**: ____ (= 7203 / 6758 / 9984 / skip)
- **name**: __________
- **label**: __________
- **rank**: #__
- **score**: _____
- **confidence**: __.__

### §6.1 Why selected (= 候補選択理由、自分の言葉で)

- FIRE の why_selected 要約: __________
- 自分の追加判断: __________
- pattern_tags: __________
- risk_notes: なし (= 全 top 候補 risk_notes 空)

### §6.2 採用 pattern (= pattern_candidate_report から)

- [ ] `freshness_ok_high_confidence` (= 7203, 6758)
- [ ] `manual_review_active_label` (= 7203, 6758, 9984)
- [ ] `multi_reason_basis` (= 7203, 6758)
- [ ] `low_risk_note` (= 7203, 6758, 9984)

## §7 Entry plan

- **entry 価格目安**: ____ 円 (= 寄付き ± __% 以内)
  - 7203 寄付き想定: 約 2,800-2,850 円帯 (= 過去推定、当日値で確認)
  - 6758 寄付き想定: 約 19,500-20,000 円帯
- **entry 株数**: ___ 株 (= 1 トレード最大損失 ÷ (entry − stop_loss) で算出)
  - 例: 7203 entry 2,800 / SL 2,660 (5% 下落) → 1 株 140 円リスク
  - 10,000 円 ÷ 140 = 71 株 → 100 株単位なので **100 株 (= 1 単元)** = 必要資金 28 万円
- **entry 時刻目安**: ☐ 寄付き成行 / ☐ 寄付き 5 分後 / ☐ 場中 押し目 / ☐ skip
- **entry 条件 (= 寄付き直前確認)**:
  - [ ] 出来高 平均比 ≥ 100%
  - [ ] 気配 寄付き ±2% 以内
  - [ ] 板 気配本数 ≥ 5 本
  - [ ] 同セクター上位が上昇
  - [ ] 日経 / TOPIX が逆風でない

## §8 Stop loss

- **stop loss 価格**: ____ 円 (= entry − 5% を目安、銘柄特性で調整)
- **想定リスク**: 1 株 ___ 円 × 100 株 = ______ 円
- **1 トレード最大損失 (= 5,000-15,000 円) 内に収まる**: ☐ Yes / ☐ No
  → No なら株数調整 or skip
- **逆指値発注**: ☐ 楽天逆指値 (= 発注は手動) / ☐ 価格 watch + 手動成行
- **損切ずらさない**: ☐ Yes (= 自分への約束)

## §9 Take profit

- **take profit 価格**: ____ 円 (= entry + 3% を目安、銘柄特性で調整)
- **想定利益**: 1 株 ___ 円 × 100 株 = ______ 円
- **リスク・リワード比**: ___ : ___ (= 利益 ÷ 損失、≥ 1.5 推奨)
- **利確 trigger**: ☐ 価格到達 / ☐ h20 終値 / ☐ 急騰時 手動判断 / ☐ 15:10 close

## §10 No-trade (= skip) 条件 (= 1 件 でも 該当 → 該当銘柄 skip)

- [ ] `reason_tags` が薄い (0-1 件) — **9984 は要注意 (= 1 件のみ)**
- [ ] `risk_notes` が重大 — **top 3 全 空 ✓**
- [ ] 出来高弱い (平均比 < 50%)
- [ ] 寄付き荒い (気配 ±5% 以上)
- [ ] 指数逆風 (日経 -1% 超)
- [ ] セクター弱い
- [ ] 板薄い (< 5,000 株 / 5 本)
- [ ] 決算 / 重要 IR / FOMC 直前
- [ ] FIRE 候補と自分の裁量が一致しない
- [ ] 監視時間が取れない
- [ ] レポートに不整合 — **本日 ranking と material 整合 ✓**

## §11 Manual buy checklist (= entry 前 最終確認)

- [x] §3.1 共通 GO-check 全 PASS ✓ (= claude code 側確認済)
- [ ] §7 entry plan 確定 (= 藤原さん記入)
- [ ] §8 stop loss 確定 (= 価格を明文化)
- [ ] §9 take profit 確定
- [ ] §10 no-trade 条件 全 不該当
- [ ] 1 トレード最大損失 ≤ 15,000 円
- [ ] 当日累計損失 + 想定 stop loss ≤ 1 日上限 30,000 円
- [ ] 2 連敗していない (= D1 なので不該当)
- [ ] ナンピン / 追加買い しない
- [ ] 15:10 までに手動 close できる
- [x] **auto_order_allowed = False (= FIRE は発注しない) ✓**
- [x] **manual_review_required = True (= 自分が最終判断) ✓**
- [ ] 楽天証券 / iSPEED 操作は **自分の手動作業** であることを認識
- [ ] **本日入力が synthetic fixture であることを認識** (= D1 初日特記)

## §12 D1 初日リスク上限 (= operation_plan §5 整合)

| 項目 | 上限 | 本日想定 |
|---|---|---|
| 1 日銘柄数 | **最大 1〜2 銘柄、D1 は 1 銘柄推奨** | __ |
| 1 トレード最大損失 | 5,000-15,000 円 | _______ 円 |
| 1 日最大損失 | 20,000-30,000 円 | _______ 円 |
| 連敗停止 | 2 連敗 = 当日停止 | (= D1 なので連敗概念なし) |
| 最大建玉 | 1-2 銘柄 | __ |
| ナンピン | 禁止 | — |
| 追加買い | 禁止 | — |
| 持ち越し | 禁止 (= デイトレ限定) | — |
| 信用 15:10 close | 必須 | — |
| 迷ったら見送り | 原則 | — |

## §13 Final decision

☐ **enter** (= §11 全 PASS、手動発注へ)

☐ **watch** (= 一旦 skip、場中で再判断)

☐ **skip** (= 当日 trade なし、特に synthetic fixture 段階のため見送り推奨)

### §13.1 enter の場合 (= 藤原さん が iSPEED で 手動発注)

- entry 発注時刻: __:__ JST
- 発注 channel: ☐ iSPEED / ☐ 楽天 Web
- 注文種別: ☐ 寄付き成行 / ☐ 指値 / ☐ 逆指値 stop
- 約定確認時刻: __:__ JST
- 約定価格: ____ 円
- 約定株数: ___ 株

### §13.2 watch の場合

- 再判断条件: __________
- 場中 trigger: __________
- 最終 cutoff: __:__ JST

### §13.3 skip の場合

- skip 理由: __________ (= 例: 「初日 synthetic fixture、実 batch 稼働後に再判断」)
- 翌日の memory: __________

## §14 安全 footer

- **本 plan は手動運用補助** (= 投資助言ではない)
- **FIRE は何も発注しない**
- **楽天証券 / iSPEED は 藤原さん本人が手動操作**
- **Computer Use / Playwright / 自動売買 は使わない**
- **最終判断と発注責任は 藤原さん本人**
- **本 D1 入力は synthetic fixture** (= 実 F062 朝 batch / 実 DATA-R3 batch ではない)

---

review template → [[2026-05-14_manual_live_pilot_review|review (= 当日 blank)]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
W60-pilot-D1 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_results|W60-pilot-D1 results]]
