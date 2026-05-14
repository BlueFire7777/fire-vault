---
template: manual-live-pilot-trade-plan
version: 1.0
date: ____-__-__
owner: BlueFire7777 (Fujiwara)
related:
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/template_manual_live_pilot_review.md
---

# Manual Live Pilot — Trade Plan (= 朝 寄付き前 記入)

## §1 基本情報

- **date**: ____-__-__ (営業日)
- **記入時刻**: __:__ JST (= 寄付き前)
- **本日の本業 / 監視可能時間**: 例「9:00-15:30 在宅、寄付き〜15:10 通し監視可」
- **15:10 までに手動 close 可能か**: ☐ Yes / ☐ No
- **大型イベント (FOMC / 雇用統計 / 重要 IR / 決算跨ぎ)**: ☐ なし / ☐ あり (→ NO-GO)

## §2 市場コンディション

- 日経平均 寄付き前: __,___ (前日比 +__/-__)
- TOPIX 寄付き前: ____ (前日比 +__/-__)
- 米国 引け: NYダウ +__/-__、S&P +__/-__、NASDAQ +__/-__
- ドル円: ___.__ 円
- 自分の地合い judgement: ☐ 強気 / ☐ 中立 / ☐ 弱気 / ☐ 不明

## §3 FIRE report paths (= 当日朝 生成済 を記載)

- `reports/after_r1/morning_line_material_<date>.md`: __________
- `reports/after_r1/good_candidate_ranking_<date>.md`: __________
- `reports/after_r1/pattern_candidate_report_<date>.md`: __________
- `reports/after_r1/paper_live_ledger_<date>.md`: __________

### §3.0 artifact_source (= W60-integration 追加)

- **artifact_source**: ☐ `synthetic_fixture` (= 実 batch 未稼働、運用フロー検証のみ、**実 trade 非推奨**)
                       / ☐ `f062_preview` (= 実 F062 朝 batch 出力、**実 trade 可** if §3.1 全 PASS)
                       / ☐ `unknown` (= 推定不能、**実 trade 禁止 + HQ 確認**)
- **f062_raw_kind**: ☐ `f062_actual_dict` (= F062 標準 output) / ☐ `plain_list` (= synthetic 形式) /
                     ☐ `other_dict` / ☐ `none`
- (artifact_source / f062_raw_kind は 4 成果物 JSON の root field から読み取り)

### §3.1 共通 GO-check (= 全 PASS 必須)

- [ ] morning_line_material の `forbidden_phrases_check.passed = True`
- [ ] paper_live_ledger 全 entry で `auto_order_allowed = False`
- [ ] paper_live_ledger 全 entry で `manual_review_required = True`
- [ ] DATA-R3 freshness verdict = `OK` (= FAILED/MISSING/STALE なら NO-GO)
- [ ] ranking と morning_line_material の top ticker が **整合**
- [ ] 出力 path が `reports/after_r1/` 配下
- [ ] **artifact_source が `f062_preview` または `synthetic_fixture`**
  (= `synthetic_fixture` の場合は 実 trade 非推奨、運用フロー検証のみ)

1 件でも未 PASS → 本日 **NO-GO**、§9 へジャンプ。
artifact_source = `unknown` → **実 trade 禁止 + HQ 確認**。

## §4 Top candidates (= morning_line_material から転記)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | ____ | __________ | __________ | __ | __.__ | _____ |
| 2 | ____ | __________ | __________ | __ | __.__ | _____ |
| 3 | ____ | __________ | __________ | __ | __.__ | _____ |

## §5 Selected ticker (= 藤原さん が top 1-3 から手動選択)

- **ticker**: ____
- **name**: __________
- **label**: __________ (例: 🟢 積極的買い推奨)
- **rank**: #__
- **score**: _____
- **confidence**: __.__

### §5.1 Why selected (= 候補選択理由、自分の言葉で)

- FIRE の why_selected 要約: __________
- 自分の追加判断: __________
- pattern_tags: __________
- risk_notes: __________

### §5.2 採用 pattern (= pattern_candidate_report から)

- [ ] `freshness_ok_high_confidence`
- [ ] `manual_review_active_label`
- [ ] `multi_reason_basis`
- [ ] `low_risk_note`
- [ ] その他: __________

## §6 Entry plan

- **entry 価格目安**: ____ 円 (= 寄付き ± __% 以内)
- **entry 株数**: ___ 株 (= 1 トレード最大損失 ÷ (entry − stop_loss) で算出)
- **必要資金**: ______ 円
- **entry 時刻目安**: ☐ 寄付き成行 / ☐ 寄付き 5 分後 / ☐ 場中 押し目 / ☐ その他: __________
- **entry 条件**:
  - [ ] 出来高 平均比 ≥ 100%
  - [ ] 気配 寄付き ±__% 以内
  - [ ] 板 気配本数 ≥ 5 本
  - [ ] 同セクター上位が上昇
  - [ ] 日経 / TOPIX が逆風でない

## §7 Stop loss

- **stop loss 価格**: ____ 円
- **想定リスク**: 1 株 ___ 円 × ___ 株 = ______ 円
- **1 トレード最大損失 (=5,000-15,000 円) 内に収まる**: ☐ Yes / ☐ No
- **逆指値発注**: ☐ 楽天逆指値 (= ただし発注は手動) / ☐ 価格 watch + 手動成行
- **損切ずらさない**: ☐ Yes (= 自分への約束)

## §8 Take profit

- **take profit 価格**: ____ 円
- **想定利益**: 1 株 ___ 円 × ___ 株 = ______ 円
- **リスク・リワード比**: ___ : ___ (= 利益 ÷ 損失)
- **利確 trigger**: ☐ 価格到達 / ☐ h20 終値 / ☐ 急騰時 手動判断 / ☐ その他: __________

## §9 No-trade (= skip) 条件

以下 1 件でも該当 → 当該銘柄 skip:

- [ ] `reason_tags` が薄い (0-1 件)
- [ ] `risk_notes` が重大
- [ ] 出来高弱い (平均比 < 50%)
- [ ] 寄付き荒い (気配 ±5% 以上)
- [ ] 指数逆風 (日経 -1% 超)
- [ ] セクター弱い
- [ ] 板薄い (< 5,000 株 / 5 本)
- [ ] 決算 / 重要 IR / FOMC 直前
- [ ] FIRE 候補と自分の裁量が一致しない
- [ ] 監視時間が取れない
- [ ] レポートに不整合

## §10 Manual buy checklist (= entry 前 最終確認)

- [ ] §3.1 共通 GO-check 全 PASS
- [ ] §6 entry plan 確定
- [ ] §7 stop loss 確定 (= 価格を 明文化)
- [ ] §8 take profit 確定
- [ ] §9 no-trade 条件 全 不該当
- [ ] 1 トレード最大損失 ≤ 15,000 円
- [ ] 当日累計損失 + 想定 stop loss ≤ 1 日上限 30,000 円
- [ ] 2 連敗していない (= 1 連敗なら慎重判断)
- [ ] ナンピン / 追加買い しない
- [ ] 15:10 までに手動 close できる
- [ ] **auto_order_allowed = False (= FIRE は発注しない)**
- [ ] **manual_review_required = True (= 自分が最終判断)**
- [ ] 楽天証券 / iSPEED 操作は **自分の手動作業** であることを認識

## §11 Final decision

☐ **enter** (= §10 全 PASS、手動発注へ)

☐ **watch** (= 一旦 skip、場中で再判断、entry 条件再評価)

☐ **skip** (= 当日 trade なし、明日に持ち越し)

### §11.1 enter の場合

- entry 発注時刻: __:__ JST
- 発注 channel: ☐ iSPEED / ☐ 楽天 Web
- 注文種別: ☐ 寄付き成行 / ☐ 指値 / ☐ 逆指値 stop
- 約定確認時刻: __:__ JST
- 約定価格: ____ 円
- 約定株数: ___ 株

### §11.2 watch の場合

- 再判断条件: __________
- 場中 trigger: __________
- 最終 cutoff: __:__ JST (= 14:00 等)

### §11.3 skip の場合

- skip 理由: __________
- 翌日の memory: __________

## §12 安全 footer

- **本 plan は手動運用補助** (= 投資助言ではない)
- **FIRE は何も発注しない**
- **楽天証券 / iSPEED は 藤原さん本人が手動操作**
- **Computer Use / Playwright / 自動売買 は使わない**
- **最終判断と発注責任は 藤原さん本人**

---

trade_review template → [[template_manual_live_pilot_review|template_manual_live_pilot_review.md]]
