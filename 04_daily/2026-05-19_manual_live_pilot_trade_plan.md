---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-19
owner: BlueFire7777 (Fujiwara)
pilot_day: D4
input_chain: F111 synth (D3 と同 sample 流用) → F062 runner --dry-run → AFTER-R1 MVP
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: synthetic_sample (= D3 と同、real_batch 未稼働)
data_r3_freshness_verdict: OK (= 選択肢 A dry-run runner)
pilot_use: eligible_with_caveat
pilot_judgment: GO (with F111 synth caveat 継続)
related:
  - 04_daily/2026-05-18_manual_live_pilot_trade_plan.md (= D3)
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D4_results.md
  - 04_daily/template_d3_real_artifact_prep.md
---

# Manual Live Pilot — Trade Plan (2026-05-19 / D4)

## ✅ D4 判定: GO 候補 (= 7 invariants 全 PASS、F111 synth caveat 継続)

**hard check 全 PASS** ✓:
- artifact_source = `f062_preview` ✓
- f062_raw_kind = `f062_actual_dict` ✓
- DATA-R3 freshness verdict = `OK` (= 選択肢 A dry-run runner) ✓
- forbidden_check.passed = True ✓
- auto_order_allowed = False ✓
- manual_review_required = True ✓
- safety_flags 13 keys 全 False ✓

### ⚠ Caveat (= D3 から継続)

- **F111 朝 batch 未稼働** (= W60-pilot-D4 baseline で再確認、Wave 41/45 進捗待ち)
- **F111 input source = `synthetic_sample`** (= D3 と同 3 銘柄 sample 流用)
- artifact_source は構造的に f062_preview だが、上流 F111 data は synth
- pilot_use = `eligible_with_caveat`

### F111 朝 batch 稼働状況確認 (= D4 朝)

| 確認項目 | 結果 |
|---|---|
| F111 朝 batch launchd plist 配置 | **未配置** (= W60-pilot-D4 baseline で confirm) |
| F062 朝 batch launchd plist 配置 | **未配置** |
| morning-advisory launchd plist 配置 | **未配置** |
| Wave 41 進捗 (= HQ_APPROVE_LAUNCHD_DAILY) | 5/19-5/26 期間 (= D4 当日着手の可能性あり) |

→ D4 朝も synth 継続。**Wave 41 着手 後に実 F111 朝 batch 化を期待**。

---

## §1 基本情報

- **date**: 2026-05-19 (火曜日 / D4)
- **記入時刻**: claude code が 2026-05-14 02:30 JST に事前生成
- **本日の本業 / 監視可能時間**: ☐ 9:00-15:30 監視可 / ☐ 部分監視 / ☐ 監視不能
- **15:10 までに手動 close 可能か**: ☐ Yes / ☐ No
- **大型イベント** (FOMC / 雇用統計 / 重要 IR / 決算跨ぎ): ☐ なし / ☐ あり

## §2 市場コンディション (= 寄付き前 藤原さん記入)

- 日経平均 寄付き前: __,___ (前日比 +__/-__)
- TOPIX 寄付き前: ____
- 米国 引け 5/18 (月) JST: NYダウ +__/-__、S&P +__/-__、NASDAQ +__/-__
- ドル円: ___.__ 円
- 自分の地合い judgement: ☐ 強気 / ☐ 中立 / ☐ 弱気

## §3 FIRE report paths (= 5/19 当日 生成済 ✓)

- `reports/after_r1/morning_line_material_2026-05-19.md` ✓
- `reports/after_r1/good_candidate_ranking_2026-05-19.md` ✓
- `reports/after_r1/pattern_candidate_report_2026-05-19.md` ✓
- `reports/after_r1/paper_live_ledger_2026-05-19.md` ✓

### §3.0 artifact_source / chain 詳細

- **artifact_source**: ☑ **`f062_preview`** / ☐ `synthetic_fixture` / ☐ `unknown`
- **f062_raw_kind**: ☑ **`f062_actual_dict`** / ☐ `plain_list` / ☐ `other_dict` / ☐ `none`
- **f111_input_source**: ☑ **`synthetic_sample`** / ☐ `real_batch` / ☐ `unknown`
- **data_r3_freshness_verdict**: ☑ **`OK`** (= 選択肢 A dry-run runner)
- **pilot_use**: ☑ **`eligible_with_caveat`** / ☐ `eligible` / ☐ `skip_recommended` / ☐ `no_go`
- **pilot_judgment**: ☑ **GO** (= 7 invariants PASS + F111 synth caveat) / ☐ HOLD / ☐ NO-GO

### §3.0.1 input chain 詳細

```
F111 advisory preview (= /tmp/fire_d4_prep/f111_preview_2026-05-19.json、3 銘柄 synth、D3 と同)
  → F062 LINE preview runner (--dry-run --require-freshness-ok)
    → /tmp/fire_d4_prep/f062_actual_output_2026-05-19.json
    = chunks: 1 / selected_count: 2 / forbidden_phrase_count: 0 / line_send_count: 0
  → AFTER-R1 MVP (--mode mvp --task all)
    → reports/after_r1/*_2026-05-19.{json,md}
    = artifact_source=f062_preview / f062_raw_kind=f062_actual_dict
```

DATA-R3 freshness:
- `/tmp/fire_d4_prep/data_r3_freshness_2026-05-19.json` (= 選択肢 A dry-run runner)
- 4 sub-runner (f100/f101/f111/f119) 全 ok / aggregate_exit_code=0

### §3.1 共通 GO-check (= 全 PASS ✓)

- [x] morning_line_material `forbidden_phrases_check.passed = True` ✓
- [x] paper_live_ledger 全 entry で `auto_order_allowed = False` ✓
- [x] paper_live_ledger 全 entry で `manual_review_required = True` ✓
- [x] DATA-R3 freshness verdict = `OK` ✓
- [x] ranking と morning_line_material の top ticker 整合 ✓ (6920/4063)
- [x] 出力 path = `reports/after_r1/` 配下 ✓
- [x] **artifact_source = `f062_preview`** ✓

## §4 Top candidates (= D3 と同じ、F111 同 sample 流用のため)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | **6920** | **レーザーテック** | 積極的買い推奨 | 🟢 | 0.78 | 187.0 |
| 2 | **4063** | **信越化学** | 条件付き買い推奨 | 🟡 | 0.60 | 133.0 |

D3 から **2 営業日連続同候補**。これは F111 が同 sample のため。実 F111 朝 batch
稼働後は日々変化が期待される。

### §4.1 D3 → D4 連続性の意味

- D3 で skip 選択した場合、D4 でも同候補が再提示される → 同じ判断軸で再評価可
- D3 で enter 選択した場合、D4 は既保有銘柄あり → 連続 entry は **ナンピン禁止
  ルール抵触**、watch / skip 推奨
- D3 で 4063 entry した場合、D4 は別候補 (= 6920) を新規 entry 検討 (= 最大建玉
  1-2 銘柄ルール内)

## §5 D4 推奨選択肢 (= D3 と同じ caveat 継続)

### §5.1 D3 と同じ推奨

**候補 A**: 🟢 6920 レーザーテック (= 最有力、ただし値嵩株リスク)
**候補 B**: 🟡 4063 信越化学 (= 慎重、根拠 1 件)

### §5.2 値嵩株リスク (= D3 から再確認)

- 6920 30,000 円帯: 100 株必要資金 300 万円、5% SL で想定リスク 150,000 円
  → 1 トレード上限 15,000 円 大幅超過 → **100 株 entry 不可**
- 4063 3,000-5,000 円帯: 100 株 30-50 万円、5% SL で 25,000 円
  → 1 トレード上限超過 → **100 株 entry 困難**
- **両候補とも本 pilot 100 株 単位リスク管理 困難** → **skip も自然選択**

### §5.3 D4 で skip が自然な理由

1. F111 input が D3 と同 synth (= 新規材料なし)
2. F111 朝 batch 未稼働継続 (= 朝の真の材料把握不可)
3. 値嵩株リスクで 100 株 entry 困難
4. D3 で同候補を check 済 → D4 で同候補を新たに entry する理由が弱い

→ **D4 推奨**: **skip** または **watch** (= D5 / Wave 41 進捗待ち)

## §6 Selected ticker

- **ticker**: ____ (= 推奨: skip)
- **name**: __________
- **label**: __________
- **rank**: #__
- **score**: _____
- **confidence**: __.__

### §6.1 Why selected (= 候補選択理由、自分の言葉で / skip 理由)

- FIRE の why_selected 要約: __________
- 自分の追加判断 (= F111 caveat 継続 + 値嵩株 + D3 同候補): __________
- pattern_tags: __________
- risk_notes: __________

### §6.2 採用 pattern (= D3 と同)

候補 6920:
- [ ] `freshness_ok_high_confidence`
- [ ] `manual_review_active_label`
- [ ] `multi_reason_basis`
- [ ] `low_risk_note`

(= 全 pattern `status=candidate_only` / `validation_status=unvalidated`)

## §7 Entry plan

- **entry 価格目安**: ____ 円
- **entry 株数**: ___ 株 (= 100 株 entry は両候補ともリスク上限超過)
- **必要資金**: ______ 円
- **entry 時刻目安**: ☐ 寄付き成行 / ☐ 寄付き 5 分後 / ☐ skip
- **entry 条件**:
  - [ ] 出来高 平均比 ≥ 100%
  - [ ] 気配 寄付き ±2% 以内
  - [ ] 板 気配本数 ≥ 5 本
  - [ ] 同セクター上位が上昇
  - [ ] 日経 / TOPIX が逆風でない
  - [ ] **artifact_source = f062_preview** ✓
  - [ ] **F111 caveat (= synth) 認識** ✓
  - [ ] **D3 trade との連続性問題なし** (= ナンピン回避)

## §8 Stop loss

- **stop loss 価格**: ____ 円 (= entry − 5%)
- **想定リスク**: 1 株 ___ 円 × 100 株 = ______ 円
- **1 トレード最大損失 (= 5,000-15,000 円) 内**: ☐ Yes / ☐ No

## §9 Take profit

- **take profit 価格**: ____ 円
- **想定利益**: 1 株 ___ 円 × 100 株 = ______ 円
- **リスク・リワード比**: ___ : ___

## §10 No-trade (= skip) 条件

- [ ] `reason_tags` 薄い (0-1 件) — 4063 該当
- [ ] `risk_notes` 重大 — top 2 全 空 ✓
- [ ] 出来高弱い
- [ ] 寄付き荒い
- [ ] 指数逆風
- [ ] セクター弱い
- [ ] 板薄い
- [ ] 決算 / 重要 IR / FOMC 直前
- [ ] FIRE 候補と裁量不一致
- [ ] 監視時間取れない
- [ ] レポート不整合 — 本日整合 ✓
- ☑ **F111 input は synth、D3 と同候補、新規材料なし** — 該当
- ☑ **値嵩株 100 株単位でリスク上限超過** — 該当

## §11 Manual buy checklist

- [x] §3.1 共通 GO-check 全 PASS ✓
- [x] §3.0 artifact_source = f062_preview ✓
- [x] §3.0 F111 caveat (= synth) 認識 ✓
- [ ] §7 entry plan 確定
- [ ] §8 stop loss 確定
- [ ] §9 take profit 確定
- [x] §10 no-trade 条件 該当 (= F111 synth + 値嵩株) → skip 自然
- [ ] 1 トレード最大損失 ≤ 15,000 円
- [ ] 当日累計損失 + 想定 stop loss ≤ 30,000 円
- [ ] **D3 で entry した場合は ナンピン禁止 ルール認識**
- [ ] ナンピン / 追加買い しない
- [ ] 15:10 までに手動 close できる
- [x] **auto_order_allowed = False** ✓
- [x] **manual_review_required = True** ✓
- [ ] 楽天証券 / iSPEED 操作 は 自分の手動作業
- [x] **本日 F062 actual format dict 経由、F111 input は synth (= D3 同) を認識**

## §12 D4 リスク上限

| 項目 | 上限 | D4 想定 |
|---|---|---|
| 1 日銘柄数 | 最大 1-2 / **D4 = 1 銘柄推奨、skip も自然** | __ |
| 1 トレード最大損失 | 5,000-15,000 円 | (= 100 株で超過注意) |
| 1 日最大損失 | 20,000-30,000 円 | __ |
| 連敗停止 | 2 連敗 = 当日停止 | (= D3 enter なし想定 / 不該当) |
| 最大建玉 | 1-2 銘柄 (= D3 既保有 + D4 = 制限内) | __ |
| ナンピン / 追加買い / 持ち越し | 禁止 | — |
| 信用 15:10 close | 必須 | — |

## §13 Final decision

☐ **enter** (= 構造的 GO + F111 synth caveat 継続 + 値嵩株、自己責任)

☐ **watch** (= 場中観察のみ、D5 / Wave 41 進捗で再判断)

☐ **skip** (= F111 synth + 値嵩株 + D3 同候補で見送り推奨)

### §13.1 enter / watch / skip 詳細

(藤原さん記入)

## §14 安全 footer

- 手動運用補助 / FIRE 発注しない / iSPEED 手動 / Computer Use 不採用 /
  最終判断は藤原さん本人 / D4 input chain = F111 synth → F062 → AFTER-R1 =
  artifact_source=f062_preview / 7 invariants PASS / F111 caveat 継続

---

review → [[2026-05-19_manual_live_pilot_review|D4 review (blank)]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D3 trade plan → [[2026-05-18_manual_live_pilot_trade_plan|D3 trade plan]]
D3 manual sequence → [[template_d3_real_artifact_prep|template_d3_real_artifact_prep.md]]
W60-pilot-D4 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D4_results|W60-pilot-D4 results]]
