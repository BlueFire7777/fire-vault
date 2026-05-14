---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-22
owner: BlueFire7777 (Fujiwara)
pilot_day: D7
input_chain: F111-real-batch-staging + research enriched → F062 → AFTER-R1
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
research_enrichment_source: research_watchlist_signals (base_date 2026-05-12)
data_r3_freshness_verdict: OK
pilot_use: eligible_with_liquidity_check
pilot_judgment: HOLD (= GO 構造的 PASS だが D6 review missing + 同候補再現で慎重)
d6_review_status: missing (= D6 review.md 未記入)
d6_d7_overlap: 100% (= 3/3 銘柄同一)
related:
  - 04_daily/2026-05-21_manual_live_pilot_trade_plan.md (= D6)
  - 04_daily/2026-05-21_manual_live_pilot_review.md (= D6 review、blank)
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D7_results.md
---

# Manual Live Pilot — Trade Plan (2026-05-22 / D7)

## 判定: HOLD (= 9 hard invariants PASS だが D6 review missing + 同候補再現)

hard check 9 invariants 全 PASS:
- artifact_source = `f062_preview` ✓
- f062_raw_kind = `f062_actual_dict` ✓
- f111_input_source = `f111_real_batch` ✓
- DATA-R3 freshness = `OK` ✓
- forbidden_check.passed = True ✓
- auto_order_allowed = False ✓
- manual_review_required = True ✓
- safety_flags 13 keys 全 False ✓
- top_candidates count = 3 ✓

### ⚠ D7 特記: 2 つの caveat

**Caveat 1: D6 review missing**
- D6 (= 2026-05-21) review.md は **blank (未記入)**
- 藤原さんが D6 場後に記入する想定だったが、本 plan 生成時点で未確認
- → liquidity actual / entry/skip 理由 が不明
- → D7 で D6 学習を反映できない

**Caveat 2: D6 → D7 同候補 100% 再現**
- D6 top: 8747 / 5729 / 3489
- D7 top: 8747 / 5729 / 3489 (= **完全一致**)
- 原因: staging research_watchlist_signals の最新 base_date が 2026-05-12 で更新されていない
  (= F111 朝 batch 未稼働、signal は前日更新まで)
- 結果: D6 で entry した場合 → D7 同候補は **ナンピン抵触** → skip 推奨
- D6 で skip した場合 → D7 同候補で再評価可、liquidity check 結果次第

→ **pilot_use = `eligible_with_liquidity_check`** だが **判定 = HOLD**
(= 構造的 GO 条件は満たすが、運用上の caveat あり)

---

## §1 基本情報

- **date**: 2026-05-22 (金曜日 / D7)
- **記入時刻**: claude code が 2026-05-14 03:50 JST に事前生成
- **本日の本業 / 監視可能時間**: ☐ 9:00-15:30 監視可 / ☐ 部分監視 / ☐ 監視不能
- **15:10 までに手動 close 可能か**: ☐ Yes / ☐ No
- **大型イベント**: ☐ なし / ☐ あり (= 月末・五十日 注意)

## §2 D6 review recap (= 重要)

**D6 (= 2026-05-21) review.md = blank**:
- 実 entry: 不明
- liquidity actual: 不明
- spread / board / volume: 不明
- skip 理由: 不明
- D7 への反映項目: **抽出不能**

藤原さん action: D6 review を **最低でも §13 next action のみでも記入** すること。
D7 で **D6 学習を反映できない** ことが本 plan の最大 caveat。

## §3 市場コンディション (= 寄付き前 藤原さん記入)

- 日経平均 寄付き前: __,___ (前日比 +__/-__)
- TOPIX 寄付き前: ____
- 米国 引け 5/21 (木) JST: NYダウ / S&P / NASDAQ
- ドル円: ___.__ 円
- 自分の地合い judgement: ☐ 強気 / ☐ 中立 / ☐ 弱気
- **金曜 = 翌週リスク回避 ポジ整理 注意**

## §4 FIRE report paths (= 5/22 当日 生成済 ✓)

- `reports/after_r1/morning_line_material_2026-05-22.md` ✓
- `reports/after_r1/good_candidate_ranking_2026-05-22.md` ✓
- `reports/after_r1/pattern_candidate_report_2026-05-22.md` ✓
- `reports/after_r1/paper_live_ledger_2026-05-22.md` ✓

### §4.0 artifact_source / chain 詳細

- **artifact_source**: ☑ `f062_preview` ✓
- **f062_raw_kind**: ☑ `f062_actual_dict` ✓
- **f111_input_source**: ☑ `f111_real_batch` ✓ (= D6 から維持)
- **research_enrichment_source**: research_watchlist_signals (= base_date 2026-05-12、
  D6 と同 staging signal、**日次更新なし**)
- **data_r3_freshness_verdict**: ☑ `OK` ✓
- **pilot_use**: ☑ `eligible_with_liquidity_check` (= 構造的 GO + D6/D7 同候補 caveat)
- **pilot_judgment**: ☑ **HOLD** (= 9 invariants PASS だが運用上の caveat 2 件)

### §4.0.1 D6 → D7 chain 再現性

```
staging signal (= base_date 2026-05-12 未更新)
  → F111-real-batch (= 同 candidate set)
  → F062 → AFTER-R1
  → top_candidates 100% 同一 (= 8747 / 5729 / 3489)
```

これは **chain 自体の出力安定性 ✓** だが **staging signal 日次更新がない** ことを意味する。
真の本番化には F111 朝 batch (= 別 wave) で daily signal 更新が必要。

### §4.1 共通 GO-check + D7 追加 (= 9 hard invariants 全 PASS ✓)

- [x] morning_line_material `forbidden_phrases_check.passed = True` ✓
- [x] paper_live_ledger 全 entry で `auto_order_allowed = False` ✓
- [x] paper_live_ledger 全 entry で `manual_review_required = True` ✓
- [x] DATA-R3 freshness verdict = `OK` ✓
- [x] ranking と morning_line_material の top ticker 整合 ✓
- [x] 出力 path = `reports/after_r1/` 配下 ✓
- [x] artifact_source = `f062_preview` ✓
- [x] f111_input_source = `f111_real_batch` ✓
- [x] top_candidates ≥ 1 件 (= 3 件) ✓
- [x] sample / tradable / risk filter (= 9th invariant 強制) ✓

## §5 Top candidates (= D6 と 100% 同一、再現性確認)

| rank | ticker | name | label | confidence | score | D6 vs D7 |
|---|---|---|---|---|---|---|
| 1 | **8747** | 豊トラスティ証券 | 積極的買い推奨 🟢 | 0.93 | 221.99 | **同一** |
| 2 | **5729** | 日本精鉱 | 積極的買い推奨 🟢 | 0.92 | 219.52 | **同一** |
| 3 | **3489** | フェイスネットワーク | 積極的買い推奨 🟢 | 0.91 | 217.80 | **同一** |

### §5.1 D6 → D7 同一の意味

- 良い面: chain 出力 安定 (= deterministic)、再現性確認 ✓
- 悪い面: staging signal 日次更新なし、真の朝 batch 化が次の課題

## §6 D7 推奨選択肢

### §6.1 D6 entry / skip 別 推奨

**D6 で entry した場合** (= 推測):
- D7 同候補は **ナンピン抵触** → skip 推奨
- 既保有 銘柄を 15:10 までに 手動 close (= 信用なら確実に)
- 新規 entry は **しない**

**D6 で skip した場合** (= 推測):
- D7 同候補で **再評価可**
- ただし D6 で skip した理由 (= liquidity 等) が D7 でも該当する可能性高い
- D7 で **改めて liquidity check** 実行、問題なければ entry

**D6 review missing** で entry/skip 不明:
- 藤原さんが **D6 ポジション残存有無** を確認
- 残存あり → 15:10 close
- 残存なし → D7 で再評価

### §6.2 推奨 Final decision

**HOLD/skip** が安全側選択:
- D6 review missing で学習反映できない
- 同候補で新規 entry の追加学習価値が低い
- 翌週 月曜 (= D8 想定) で staging signal 更新を期待 (= 5/19 or 5/23 base_date)

## §7 Selected ticker (= 藤原さん が手動選択 / skip)

- **ticker**: ____ (= 推奨: skip)
- **name**: __________
- **label**: __________
- **rank**: #__

### §7.1 Why selected / skipped

- D6 entry / skip 状況: __________ (= 藤原さん 当日朝に確認)
- D6 残存ポジション: __________ (= iSPEED で確認)
- D7 推奨判断: __________

## §8 Liquidity / Event manual check (= D7 必須 ★)

### §8.1 Liquidity check (= D6 と同候補なので同様の特性想定)

- 出来高 平均比: ☐ ≥ 100% / ☐ 50-100% / ☐ < 50% (= skip 推奨)
- 板 気配本数: ☐ ≥ 5 本 / ☐ 3-5 本 / ☐ < 3 本 (= skip 推奨)
- spread 寄付き想定: ☐ ≤ 0.5% / ☐ 0.5-1% / ☐ > 1% (= skip 推奨)
- **金曜 翌週リスク回避で 中小型 出来高 細る可能性**

### §8.2 Event / Earnings check

- 決算 / 重要 IR 跨ぎ: ☐ なし / ☐ あり (= 月末週 注意)
- セクター材料 (= 金融 / 鉄鋼 / 不動産): __________
- 米国 同セクター 動向: __________
- **金曜午後 ポジ整理 注意** (= 板薄になる時間帯)

### §8.3 共通 entry 条件 (= D6 同)

- [ ] 寄付き気配 ±2% 以内
- [ ] 出来高 平均比 ≥ 100%
- [ ] 同セクター上位が下落していない
- [ ] 日経 / TOPIX が -1% 超 ではない
- [ ] 板 5 本以上 + 出来高 5,000 株目処

## §9 Entry plan (= enter 選択時、ただし D7 推奨は skip)

- **entry 価格目安**: ____ 円
- **entry 株数**: 100 株
- **必要資金**: ______ 円
- **estimated_100_share_risk**: _____ 円
- **entry 時刻目安**: ☐ 寄付き成行 / ☐ 寄付き 5 分後 / ☐ skip

## §10 Stop loss

- stop loss 価格: ____ 円 (= entry − 5%)
- 想定リスク: ___ 円 × 100 株 = ______ 円
- 1 トレード上限 (= 5,000-15,000 円) 内: ☐ Yes / ☐ No

## §11 Take profit

- take profit 価格: ____ 円 (= entry + 3%)
- 想定利益: ___ 円 × 100 株 = ______ 円
- リスク・リワード比: ___ : ___

## §12 No-trade (= skip) 条件 (= D7 強化)

- [ ] §8.1 liquidity 不足
- [ ] §8.2 event 跨ぎ / 月末 注意
- [ ] §8.3 共通条件 不充足
- [ ] 監視時間 不足
- [ ] D6 review missing で 学習不能
- [ ] D6 で同候補 entry 済 (= ナンピン抵触)
- [ ] 金曜午後 板薄 / 出来高細る
- [ ] **D7 = HOLD 推奨**

## §13 Manual buy checklist

- [x] §4.1 共通 GO-check 全 PASS ✓
- [x] §4.0 artifact_source / f111_input_source = f111_real_batch ✓
- [ ] §2 D6 review recap 確認 (= missing なら省略)
- [ ] §7.1 D6 残存ポジション 確認
- [ ] §8.1 liquidity check 全 PASS
- [ ] §8.2 event check 全 PASS
- [ ] §8.3 共通 entry 条件 全 PASS
- [ ] §9 entry plan 確定
- [ ] §10 stop loss 確定
- [ ] §11 take profit 確定
- [ ] §12 no-trade 条件 全 不該当
- [ ] 1 トレード ≤ 15,000 円
- [ ] 1 日累計 ≤ 30,000 円
- [ ] D6 entry した場合は ナンピン禁止 認識
- [ ] 15:10 までに手動 close できる
- [x] **auto_order_allowed = False** ✓
- [x] **manual_review_required = True** ✓
- [ ] 楽天 / iSPEED 手動操作のみ
- [x] **D6 同候補で再現、staging signal 日次更新なし caveat 認識**

## §14 D7 リスク上限

| 項目 | 上限 | D7 想定 |
|---|---|---|
| 1 日銘柄数 | 最大 1-2 / D7 = 1 銘柄推奨 (or skip) | |
| 1 トレード最大損失 | 5,000-15,000 円 | |
| 1 日最大損失 | 20,000-30,000 円 | |
| 連敗停止 | 2 連敗 = 当日停止 | (= D1-D5 trade なし、D6 不明) |
| 最大建玉 | 1-2 銘柄 | |
| ナンピン / 追加買い / 持ち越し | 禁止 | (D6 entry 場合は新規不可) |
| 信用 15:10 close | 必須 | |
| 自動発注 | 0 | ✓ |
| 楽天 / iSPEED 手動 | ✓ | |
| Computer Use 不採用 | ✓ | |
| **D6 同候補再現** | **skip 推奨** | |
| **D6 review missing** | **慎重判断** | |

## §15 Final decision

☐ **enter** (= D6 で skip / 残ポジなし + §8 全 PASS + 自己責任)

☐ **watch** (= 場中観察のみ、月曜 D8 で staging signal 更新後再判断)

☐ **skip** (= D7 推奨選択肢)

### §15.1 推奨

**skip** (= 標準推奨):
- D6 review missing で D6 学習反映できない
- 同候補で新規 entry の追加学習価値が低い
- 月曜 D8 で staging signal 更新を期待
- 金曜午後 板薄注意

## §16 安全 footer

- 手動運用補助 / FIRE 発注しない / 自動化なし / auto_order=False /
  manual_review=True / 楽天正本 / Computer Use 不採用 / 最終判断は藤原さん本人 /
  D7 chain = F111 real_batch + research enriched (= D6 と同 staging signal) /
  9 hard invariants PASS / 中小型 liquidity caveat / D6 review missing caveat /
  D6/D7 同候補 100% 再現 caveat / 推奨判定 HOLD

---

review → [[2026-05-22_manual_live_pilot_review|D7 review (blank)]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D6 trade plan → [[2026-05-21_manual_live_pilot_trade_plan|D6]]
D6 review → [[2026-05-21_manual_live_pilot_review|D6 review (blank)]]
W60-pilot-D7 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D7_results|W60-pilot-D7 results]]
