---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-21
owner: BlueFire7777 (Fujiwara)
pilot_day: D6
input_chain: F111-real-batch-staging (= research_watchlist_signals enriched) → F062 → AFTER-R1 MVP
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
research_enrichment_source: research_watchlist_signals (base_date 2026-05-12)
data_r3_freshness_verdict: OK
pilot_use: eligible_with_liquidity_check
pilot_judgment: GO (= 9 hard invariants PASS、ただし中小型株の liquidity 手動確認必須)
related:
  - 04_daily/2026-05-20_manual_live_pilot_trade_plan.md (= D5)
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D6_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_RESEARCH_ADVISORY_STAGING_results.md
---

# Manual Live Pilot — Trade Plan (2026-05-21 / D6)

## ✅✅ D6 判定: GO (= 9 hard invariants 全 PASS、初の実 trade 可能 day)

**D1-D5 全 day 実 entry 0/5 → D6 で初の実 trade 候補登場** ✓

hard check 9 invariants 全 PASS:
- artifact_source = `f062_preview` ✓
- f062_raw_kind = `f062_actual_dict` ✓
- **f111_input_source = `f111_real_batch`** ✓ (= staging real_batch + research enriched)
- DATA-R3 freshness verdict = `OK` ✓
- forbidden_check.passed = True ✓
- auto_order_allowed = False ✓
- manual_review_required = True ✓
- safety_flags 13 keys 全 False ✓
- **top_candidates count = 3** ✓ (= D5 まで全 0 件 → 初の出現)

### ⚠ 重要 caveat: 中小型成長株 liquidity 手動確認必須

候補は **中小型成長株** (= Core30/Large70 大型株とは別 universe):
- 板薄・値動き荒い可能性高い
- 100 株 entry でも spread / 出来高 / 板厚 で skip 判断必要
- 寄付き直後の乱高下に注意

**藤原さんが §6-§10 で liquidity / event 手動確認必須**。確認できなければ skip。

---

## §1 基本情報

- **date**: 2026-05-21 (木曜日 / D6)
- **記入時刻**: claude code が 2026-05-14 03:30 JST に事前生成
- **本日の本業 / 監視可能時間**: ☐ 9:00-15:30 監視可 / ☐ 部分監視 / ☐ 監視不能
- **15:10 までに手動 close 可能か**: ☐ Yes / ☐ No
- **大型イベント** (FOMC / 雇用統計 / 重要 IR / 決算跨ぎ): ☐ なし / ☐ あり

## §2 市場コンディション (= 寄付き前 藤原さん記入)

- 日経平均 寄付き前: __,___ (前日比 +__/-__)
- TOPIX 寄付き前: ____
- 米国 引け 5/20 (水) JST: NYダウ +__/-__、S&P +__/-__、NASDAQ +__/-__
- ドル円: ___.__ 円
- 自分の地合い judgement: ☐ 強気 / ☐ 中立 / ☐ 弱気

## §3 FIRE report paths (= 5/21 当日 生成済 ✓)

- `reports/after_r1/morning_line_material_2026-05-21.md` ✓
- `reports/after_r1/good_candidate_ranking_2026-05-21.md` ✓
- `reports/after_r1/pattern_candidate_report_2026-05-21.md` ✓
- `reports/after_r1/paper_live_ledger_2026-05-21.md` ✓

### §3.0 artifact_source / chain 詳細

- **artifact_source**: ☑ **`f062_preview`** ✓
- **f062_raw_kind**: ☑ **`f062_actual_dict`** ✓
- **f111_input_source**: ☑ **`f111_real_batch` ★** (= D5 f111_sample から進化)
- **research_enrichment_source**: ☑ **`research_watchlist_signals`** (= base_date 2026-05-12)
- **data_r3_freshness_verdict**: ☑ **`OK`** (= 選択肢 A dry-run runner)
- **pilot_use**: ☑ **`eligible_with_liquidity_check`** (= 初の実 trade 可能、ただし liquidity 確認必須)
- **pilot_judgment**: ☑ **GO** (= 9 hard invariants PASS)

### §3.0.1 D6 input chain (= 初の完全 enriched path)

```
staging DB (read-only URI mode=ro)
  ↓ research_watchlist_signals (最新 base_date 2026-05-12)
F111-real-batch-staging (--require-research-signal)
  ↓ INNER JOIN market_listings × signal + label mapping
F111-real-batch enriched (= 15 candidates、全 boost、eligible 14)
  ↓ --f062-preview-json
F062 (= F062 actual dict format 互換 output として渡し)
  ↓ --f062-preview-json
AFTER-R1 MVP (--mode mvp --task all)
  ↓ output-dir reports/after_r1
reports/after_r1/*_2026-05-21.{json,md} (= artifact_source=f062_preview /
  f111_input_source=f111_real_batch / top_candidates 3 件)
```

### §3.1 共通 GO-check + D6 追加 (= 全 PASS ✓)

- [x] morning_line_material `forbidden_phrases_check.passed = True` ✓
- [x] paper_live_ledger 全 entry で `auto_order_allowed = False` ✓
- [x] paper_live_ledger 全 entry で `manual_review_required = True` ✓
- [x] DATA-R3 freshness verdict = `OK` ✓
- [x] ranking と morning_line_material の top ticker 整合 ✓
- [x] 出力 path = `reports/after_r1/` 配下 ✓
- [x] **artifact_source = `f062_preview`** ✓
- [x] **f111_input_source = `f111_real_batch`** ✓ (= D6 進化)
- [x] **top_candidates count ≥ 1** ✓ (= 3 件、D5 まで 0 件から飛躍)
- [x] 全 top candidate が **sample_ticker_detected = False** ✓
- [x] 全 top candidate が **risk_within_pilot_limit = True** (= 9th invariant 強制) ✓

## §4 Top candidates (= D6 初の実在 ticker 3 件)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | **8747** | **豊トラスティ証券** | 積極的買い推奨 | 🟢 | 0.93 | 221.99 |
| 2 | **5729** | **日本精鉱** | 積極的買い推奨 | 🟢 | 0.92 | 219.52 |
| 3 | **3489** | **フェイスネットワーク** | 積極的買い推奨 | 🟢 | 0.91 | 217.80 |

### §4.1 D5 → D6 進化記録

| Day | f111_input_source | top_candidates | 実 trade 可? |
|---|---|---|---|
| D5 | f111_sample (= サンプル ticker) | 0 件 | No (構造的) |
| **D6** | **f111_real_batch (= 実在 + research enriched)** | **3 件** | **Yes** ✓ |

### §4.2 各候補 詳細 (= AFTER-R1 + F111 staging from staging signal)

**候補 #1: 🟢 8747 豊トラスティ証券** (= research_final_score 0.93、A1、selected)
- セクター: 金融
- reason_tags: rank_reason=quality_value boost / strategy=quality_value /
  rank=A1
- risk_notes: (= research signal でリスクフラグなし)
- estimated_100_share_risk: ~5,000 円台 (= 上限内 ✓)
- 中小型株 = 流動性 手動確認必須

**候補 #2: 🟢 5729 日本精鉱** (= 0.92、A1、selected)
- セクター: 鉄鋼・非鉄
- 中小型株 = 流動性 確認必須

**候補 #3: 🟢 3489 フェイスネットワーク** (= 0.91、A1、selected)
- セクター: 不動産
- 中小型株 = 流動性 確認必須

## §5 D6 推奨選択肢

### §5.1 推奨

**D6 で初の実 trade 候補登場** だが、**中小型成長株 = liquidity 不確実**。
慎重 entry が原則:
- 候補 A: 🟢 8747 豊トラスティ証券 (= 最高 score / 金融)
- 候補 B: 🟢 5729 日本精鉱 (= 鉄鋼)
- 候補 C: 🟢 3489 フェイスネットワーク (= 不動産)

藤原さんが **§7 liquidity / event check** で 1 件選択、または skip。

### §5.2 D6 entry 推奨フロー

1. 朝 8:30: TDnet / 日経で当日材料 / 大型イベント 確認
2. 朝 8:55: iSPEED で top 3 候補の **板 / 気配 / 出来高目処** 確認
3. 寄付き 9:00 直前:
   - 気配 ±2% 以内
   - 板 5 本以上
   - 出来高 5,000 株以上 (大型と異なり中小型は基準 厳しめに)
   - 同セクター上位 動向確認
4. 寄付き 5 分後 (= 9:05):
   - 1 銘柄選択して entry (= 自己責任)
   - または skip 判断

## §6 Selected ticker (= 藤原さん が top 1-3 から手動選択)

- **ticker**: ____
- **name**: __________
- **label**: ☐ 積極的買い推奨
- **rank**: #__
- **score**: _____
- **confidence**: __.__

### §6.1 Why selected (= 候補選択理由)

- FIRE の why_selected: __________
- 自分の追加判断: __________
- セクター動向 (= 金融 / 鉄鋼 / 不動産): __________
- 当日材料 (TDnet 等で確認): __________
- 中小型 liquidity 印象 (= 板厚 / 出来高): __________

## §7 Liquidity / Event manual check (= D6 必須 ★)

### §7.1 Liquidity check

- 出来高 平均比: ☐ ≥ 100% / ☐ 50-100% / ☐ < 50% (= skip 推奨)
- 板 気配本数: ☐ ≥ 5 本 / ☐ 3-5 本 / ☐ < 3 本 (= skip 推奨)
- spread 寄付き想定: ☐ ≤ 0.5% / ☐ 0.5-1% / ☐ > 1% (= skip 推奨)
- 中小型 = **100 株 entry でも spread 1% 超なら skip**

### §7.2 Event / Earnings check

- 決算 / 重要 IR 跨ぎ: ☐ なし / ☐ あり (= skip)
- セクター固有材料 (= 金融政策 / コモディティ / 不動産規制): __________
- 米国 同セクター 動向: __________
- 板の異常な動き (= 寄付き直前 大量注文 等): ☐ なし / ☐ あり

### §7.3 共通 entry 条件

- [ ] 寄付き気配 ±2% 以内
- [ ] 出来高 平均比 ≥ 100%
- [ ] 同セクター上位が下落していない
- [ ] 日経 / TOPIX が -1% 超 ではない
- [ ] 板 5 本以上 + 出来高 5,000 株目処

## §8 Entry plan (= enter 選択時)

- **entry 価格目安**: ____ 円
- **entry 株数**: 100 株 (= pilot 標準)
- **必要資金**: ______ 円
- **estimated_100_share_risk** (= AFTER-R1 自動計算): _____ 円
- **entry 時刻目安**: ☐ 寄付き成行 / ☐ 寄付き 5 分後 / ☐ 場中 押し目 / ☐ skip

## §9 Stop loss

- **stop loss 価格**: ____ 円 (= entry − 5% 目安)
- **想定リスク**: 1 株 ___ 円 × 100 株 = ______ 円
- **1 トレード最大損失 (= 5,000-15,000 円) 内**: ☐ Yes / ☐ No

## §10 Take profit

- **take profit 価格**: ____ 円 (= entry + 3% 目安)
- **想定利益**: 1 株 ___ 円 × 100 株 = ______ 円
- **リスク・リワード比**: ___ : ___ (= ≥ 1.5 推奨)

## §11 No-trade (= skip) 条件 (= D6 中小型 特有)

- [ ] §7.1 liquidity 不足 (= 出来高 / 板 / spread)
- [ ] §7.2 event 跨ぎ
- [ ] §7.3 共通条件 不充足
- [ ] 監視時間 不足
- [ ] FIRE 候補と自分の裁量 不一致
- [ ] レポート不整合
- [ ] **D6 初日 = 慎重判断、迷ったら skip 推奨**

## §12 Manual buy checklist

- [x] §3.1 共通 GO-check + D6 追加 全 PASS ✓
- [x] §3.0 artifact_source / f111_input_source = f111_real_batch ✓
- [ ] §7.1 liquidity check 全 PASS
- [ ] §7.2 event check 全 PASS
- [ ] §7.3 共通 entry 条件 全 PASS
- [ ] §8 entry plan 確定
- [ ] §9 stop loss 確定 (= 価格 明文化)
- [ ] §10 take profit 確定
- [ ] §11 no-trade 条件 全 不該当
- [ ] 1 トレード最大損失 ≤ 15,000 円
- [ ] 当日累計損失 + 想定 stop loss ≤ 30,000 円
- [ ] ナンピン / 追加買い しない
- [ ] 15:10 までに手動 close できる
- [x] **auto_order_allowed = False** ✓
- [x] **manual_review_required = True** ✓
- [ ] 楽天 / iSPEED 操作 は 自分の手動作業
- [x] **D6 中小型成長株 caveat 認識** (= liquidity 不確実)

## §13 D6 リスク上限

| 項目 | 上限 | D6 想定 |
|---|---|---|
| 1 日銘柄数 | 最大 1-2 / D6 = 1 銘柄推奨 | 1 銘柄 if entry |
| 1 トレード最大損失 | 5,000-15,000 円 | (= estimated_100_share_risk 内、§9 で確認) |
| 1 日最大損失 | 20,000-30,000 円 | __ |
| 連敗停止 | 2 連敗 = 当日停止 | (= D1-D5 trade なし、不該当) |
| 最大建玉 | 1-2 銘柄 | 1 |
| ナンピン / 追加買い / 持ち越し | 禁止 | — |
| 信用 15:10 close | 必須 | — |
| 自動発注 | 0 (= 構造的) | ✓ |
| 楽天 / iSPEED 操作 | 藤原さん手動のみ | ✓ |
| Computer Use | 不採用 | ✓ |
| **中小型 liquidity 不足** | **skip** | (= §7.1 で判定) |

## §14 Final decision

☐ **enter** (= §7 全 PASS、自己責任で 1 銘柄 100 株 entry)

☐ **watch** (= 場中観察のみ、明日 D7 で再判断)

☐ **skip** (= liquidity 不確実 / event / 監視不能 等で見送り)

### §14.1 enter の場合 (= 藤原さん が iSPEED で 手動発注)

- entry 発注時刻: __:__ JST
- 発注 channel: ☐ iSPEED / ☐ 楽天 Web
- 注文種別: ☐ 寄付き成行 / ☐ 指値 / ☐ 逆指値 stop
- 約定確認時刻: __:__ JST
- 約定価格: ____ 円
- 約定株数: ___ 株
- 実 spread: __.__ %
- 実 出来高 (= 寄付き 5 分):___ 株

## §15 安全 footer

- **本 plan は手動運用補助** (= 投資助言ではない)
- **FIRE は何も発注しない**
- **楽天証券 / iSPEED は 藤原さん本人が手動操作**
- **Computer Use / Playwright / 自動売買 は使わない**
- **最終判断と発注責任は 藤原さん本人**
- **D6 input chain**: staging real_batch + research_watchlist_signals enriched →
  F062 → AFTER-R1 = artifact_source=f062_preview + f111_input_source=f111_real_batch
- **9 hard invariants 全 PASS** ✓ — 初の実 trade 可能 day
- ただし **中小型成長株 caveat**: liquidity 手動確認必須

---

review template → [[2026-05-21_manual_live_pilot_review|review (blank)]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D5 trade plan → [[2026-05-20_manual_live_pilot_trade_plan|D5]]
W60-pilot-D6 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D6_results|W60-pilot-D6 results]]
