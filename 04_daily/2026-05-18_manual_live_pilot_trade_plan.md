---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-18
owner: BlueFire7777 (Fujiwara)
pilot_day: D3
input_chain: F111 synth (= 手作り) → F062 runner --dry-run → AFTER-R1 MVP
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
data_r3_freshness_verdict: OK (= 選択肢 A dry-run runner 出力)
pilot_use: eligible_with_caveat
pilot_judgment: GO (with F111 synth caveat)
related:
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D3_results.md
  - 04_daily/template_d3_real_artifact_prep.md
  - 04_daily/2026-05-15_manual_live_pilot_trade_plan.md (= D2)
---

# Manual Live Pilot — Trade Plan (2026-05-18 / D3)

## ✅ D3 判定: GO 候補 (= 7 invariants 全 PASS) — ただし F111 caveat あり

**hard check 全 PASS** ✓ (= W60-launchd-pre Step 5 strict check 経由):
- artifact_source = `f062_preview` ✓
- f062_raw_kind = `f062_actual_dict` ✓
- DATA-R3 freshness verdict = `OK` (= 選択肢 A dry-run runner) ✓
- forbidden_check.passed = True ✓
- auto_order_allowed = False ✓
- manual_review_required = True ✓
- safety_flags 13 keys 全 False ✓

### ⚠ Caveat: F111 input は手作り synth

- F111 advisory preview は **実 F111 朝 batch 由来ではない** (= W60-launchd-pre smoke
  の 3 銘柄 sample を流用、藤原さん手作り扱い)
- F062 runner 経由なので **artifact_source は f062_preview だが**、上流の F111
  data は **synthetic**
- 当日材料の **真実性は藤原さん本人が判断** (= TDnet / 日経 / セクター状況で確認)
- **pilot_use = eligible_with_caveat**: 構造的には GO 候補だが慎重判断推奨

### 実 F062 朝 batch が稼働する時期

- Wave 41 (5/19-5/26): HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2): HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 53 (6/9 D-Day): production_send 開始 + dual-run

それまで D3-D5 は F111 手作り synth 経由で GO 判定。

---

## §1 基本情報

- **date**: 2026-05-18 (月曜日 / D3)
- **記入時刻**: claude code が 2026-05-14 02:18 JST に当日朝向け事前生成
- **本日の本業 / 監視可能時間**: ☐ 9:00-15:30 監視可 / ☐ 部分監視 / ☐ 監視不能
  → **藤原さんが当日朝に記入**
- **15:10 までに手動 close 可能か**: ☐ Yes / ☐ No (= 藤原さん記入)
- **大型イベント (FOMC / 雇用統計 / 重要 IR / 決算跨ぎ)**: ☐ なし / ☐ あり
  (= 月曜は週初め、米国動向確認重要)

## §2 市場コンディション (= 寄付き前 藤原さん記入)

- 日経平均 寄付き前: __,___ (前日比 +__/-__)
- TOPIX 寄付き前: ____ (前日比 +__/-__)
- 米国 引け 5/16 (金) JST: NYダウ +__/-__、S&P +__/-__、NASDAQ +__/-__
- ドル円: ___.__ 円
- 自分の地合い judgement: ☐ 強気 / ☐ 中立 / ☐ 弱気 / ☐ 不明

## §3 FIRE report paths (= 5/18 当日 生成済 ✓)

- `reports/after_r1/morning_line_material_2026-05-18.md` ✓
- `reports/after_r1/good_candidate_ranking_2026-05-18.md` ✓
- `reports/after_r1/pattern_candidate_report_2026-05-18.md` ✓
- `reports/after_r1/paper_live_ledger_2026-05-18.md` ✓

### §3.0 artifact_source / chain 詳細 (= W60-integration / W60-launchd-pre)

- **artifact_source**: ☑ **`f062_preview`** / ☐ `synthetic_fixture` / ☐ `unknown`
- **f062_raw_kind**: ☑ **`f062_actual_dict`** / ☐ `plain_list` / ☐ `other_dict` / ☐ `none`
- **data_r3_freshness_verdict**: ☑ **`OK`** (選択肢 A dry-run runner) / ☐ `FAILED` / ☐ `MISSING`
- **pilot_use**: ☑ **`eligible_with_caveat`** / ☐ `eligible` / ☐ `skip_recommended` / ☐ `no_go`
- **pilot_judgment**: ☑ **GO (= 7 invariants 全 PASS、F111 synth caveat 付き)** / ☐ HOLD / ☐ NO-GO

### §3.0.1 input chain 由来

```
F111 advisory preview (= /tmp/fire_d3_prep/f111_preview_2026-05-18.json、3 銘柄 synth)
  → F062 LINE preview runner (--dry-run --require-freshness-ok)
    → /tmp/fire_d3_prep/f062_actual_output_2026-05-18.json
    = chunks: 1 / selected_count: 2 / forbidden_phrase_count: 0 / line_send_count: 0
  → AFTER-R1 MVP (--mode mvp --task all)
    → reports/after_r1/*_2026-05-18.{json,md}
    = artifact_source=f062_preview / f062_raw_kind=f062_actual_dict
```

DATA-R3 freshness は 選択肢 A (= dry-run runner) で生成:
- `/tmp/fire_d3_prep/data_r3_freshness_2026-05-18.json`
- 4 sub-runner (f100/f101/f111/f119) 全 ok / aggregate_exit_code=0

### §3.1 共通 GO-check (= 全 PASS ✓)

- [x] morning_line_material の `forbidden_phrases_check.passed = True` ✓
- [x] paper_live_ledger 全 entry で `auto_order_allowed = False` ✓
- [x] paper_live_ledger 全 entry で `manual_review_required = True` ✓
- [x] DATA-R3 freshness verdict = `OK` ✓
- [x] ranking と morning_line_material の top ticker が **整合** ✓ (6920/4063)
- [x] 出力 path が `reports/after_r1/` 配下 ✓
- [x] **artifact_source = `f062_preview`** ✓

## §4 Top candidates (= morning_line_material 自動転記、2 候補)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | **6920** | **レーザーテック** | 積極的買い推奨 | 🟢 | 0.78 | 187.0 |
| 2 | **4063** | **信越化学** | 条件付き買い推奨 | 🟡 | 0.60 | 133.0 |

F111 input には 7203 (トヨタ neutral) も含んでいたが、neutral label のため
morning_line_material の top には含まれない。

## §5 D3 推奨選択肢 (= GO 候補だが F111 synth caveat、慎重 entry)

### §5.1 推奨パターン

**候補 A**: 🟢 6920 レーザーテック (= 最有力)
- score = 187.0 (= D1/D2 の 7203 トヨタ score=194 に次ぐ)
- conf = 0.78 (= 高 confidence)
- 根拠 3 件: 好決算 / 半導体堅調 / 出来高高位
- 大型株 (= 板厚い)、ただし半導体セクターは Volatility 高
- 4 pattern matched (= freshness_ok_high_conf / manual_review_active /
  multi_reason / low_risk_note 該当)

**候補 B**: 🟡 4063 信越化学 (= 慎重)
- score = 133.0 / conf = 0.60 (= 中 confidence)
- 根拠 1 件のみ (= 業績ポジティブ)
- 初日には **不向き** (= 根拠薄、「条件付き」label)

### §5.2 推奨選択

**D3 は 6920 レーザーテック 1 銘柄 慎重 entry** が標準推奨。

ただし **F111 input synthetic caveat** のため:
- 実材料の真実性を藤原さんが TDnet / 日経で **必ず再確認**
- 半導体セクターの当日地合い (= TSM / NVDA 動向) を確認
- 寄付き 5 分間は様子見推奨

### §5.3 GO 判定の根拠

7 invariants 全 PASS (= §3.0 / §3.1 確認)。
構造的には GO だが、F111 input が 手作り synth であることを認識した上で
**自己責任で実 trade 判断**。

## §6 Selected ticker (= 藤原さん が top 1-2 から手動選択)

- **ticker**: ____ (= 推奨: 6920 / 4063 / skip)
- **name**: __________
- **label**: __________
- **rank**: #__
- **score**: _____
- **confidence**: __.__

### §6.1 Why selected (= 候補選択理由、自分の言葉で)

- FIRE の why_selected 要約: __________
- 自分の追加判断 (= F111 caveat 考慮): __________
- pattern_tags: __________
- risk_notes: 6920 は半導体セクター volatility、4063 は根拠薄

### §6.2 採用 pattern (= pattern_candidate_report から)

候補 6920:
- [ ] `freshness_ok_high_confidence` (= 6920 該当、4063 該当しない)
- [ ] `manual_review_active_label` (= 6920, 4063 両方該当)
- [ ] `multi_reason_basis` (= 6920 のみ該当、reason 3 件)
- [ ] `low_risk_note` (= 6920, 4063 両方該当)

(= 全 pattern `status=candidate_only` / `validation_status=unvalidated`)

## §7 Entry plan (= 6920 想定の場合)

- **entry 価格目安**: ____ 円 (= 6920 寄付き想定: 30,000-32,000 円帯、当日要確認)
- **entry 株数**: ___ 株
  - 例: entry 30,000 / SL 28,500 (5% 下落) → 1 株 1,500 円リスク
  - 1 トレード最大損失 10,000 円 → 10,000 ÷ 1,500 = 6 株 → 100 株単位制約で **100 株不可、ロット不足**
  - 6920 は値嵩株 (= 100 株単位で 300 万円規模)、本 pilot リスク上限超過
  - **6920 1 銘柄 100 株はリスク上限超過のため不採用、代替候補へ**
- **代替**: 4063 信越化学 (= 3,000-5,000 円帯、100 株で 30-50 万円規模、リスク管理可能)
  - 例: entry 5,000 / SL 4,750 (5%) → 1 株 250 円リスク
  - 10,000 ÷ 250 = 40 株 → 100 株単位なので 100 株、必要資金 50 万円、想定リスク 25,000 円
  - **25,000 円は 1 トレード上限 15,000 円 を超過** → 株数調整
  - 100 株不可なら **skip 推奨**
- **entry 時刻目安**: ☐ 寄付き成行 / ☐ 寄付き 5 分後 / ☐ 場中 押し目 / ☐ skip
- **entry 条件**:
  - [ ] 出来高 平均比 ≥ 100%
  - [ ] 気配 寄付き ±2% 以内
  - [ ] 板 気配本数 ≥ 5 本
  - [ ] 同セクター (= 半導体) 上位が上昇 / 反映
  - [ ] 日経 / TOPIX が逆風でない
  - [ ] **artifact_source = f062_preview** ✓
  - [ ] **F111 caveat (= synth 由来) を認識** ✓

## §8 Stop loss

- **stop loss 価格**: ____ 円 (= entry − 5%、銘柄特性で調整)
- **想定リスク**: 1 株 ___ 円 × 100 株 = ______ 円
- **1 トレード最大損失 (= 5,000-15,000 円) 内**: ☐ Yes / ☐ No (= 値嵩株なら超過注意)
- **株数調整**: 100 株単位で超過なら → **skip**
- **損切ずらさない**: ☐ Yes

## §9 Take profit

- **take profit 価格**: ____ 円
- **想定利益**: 1 株 ___ 円 × 100 株 = ______ 円
- **リスク・リワード比**: ___ : ___ (= ≥ 1.5 推奨)

## §10 No-trade (= skip) 条件 (= 1 件 該当 → skip)

- [ ] `reason_tags` 薄い (0-1 件) — **4063 は根拠 1 件、要注意**
- [ ] `risk_notes` 重大 — top 2 全 空 ✓
- [ ] 出来高弱い
- [ ] 寄付き荒い (気配 ±5% 以上)
- [ ] 指数逆風 (日経 -1% 超)
- [ ] セクター弱い — **半導体セクター動向重要**
- [ ] 板薄い
- [ ] 決算 / 重要 IR / FOMC 直前
- [ ] FIRE 候補と自分の裁量が一致しない
- [ ] 監視時間が取れない
- [ ] レポート不整合 — 本日整合 ✓
- ☑ **F111 input は synth** — 慎重判断要、ただし NO-GO ではない (= GO with caveat)

## §11 Manual buy checklist (= entry 前 最終確認)

- [x] §3.1 共通 GO-check 全 PASS ✓
- [x] §3.0 artifact_source = f062_preview ✓
- [x] §3.0 F111 caveat 認識 ✓
- [ ] §7 entry plan 確定
- [ ] §8 stop loss 確定 (= 価格明文化)
- [ ] §9 take profit 確定
- [ ] §10 no-trade 条件 全 不該当
- [ ] 1 トレード最大損失 ≤ 15,000 円
- [ ] 当日累計損失 + 想定 stop loss ≤ 30,000 円
- [ ] D1 D2 trade なし → 連敗概念不該当
- [ ] ナンピン / 追加買い しない
- [ ] 15:10 までに手動 close できる
- [x] **auto_order_allowed = False** ✓
- [x] **manual_review_required = True** ✓
- [ ] 楽天証券 / iSPEED 操作 は 自分の手動作業
- [x] **本日 F062 actual format dict 経由、F111 input は synth を認識**

## §12 D3 リスク上限 (= operation_plan §5 / m=0 R-39-02)

| 項目 | 上限 | D3 想定 |
|---|---|---|
| 1 日銘柄数 | 最大 1-2 / **D3 = 1 銘柄推奨** | (= 6920 もしくは 4063、または skip) |
| 1 トレード最大損失 | 5,000-15,000 円 | (= 値嵩株は注意) |
| 1 日最大損失 | 20,000-30,000 円 | __ |
| 連敗停止 | 2 連敗 = 当日停止 | (= D1 D2 trade 0、不該当) |
| 最大建玉 | 1-2 銘柄 | __ |
| ナンピン / 追加買い / 持ち越し | 禁止 | — |
| 信用 15:10 close | 必須 | — |
| 迷ったら見送り | 原則 | __ |

## §13 Final decision

☐ **enter** (= 構造的 GO + F111 synth caveat 認識、自己責任で entry)

☐ **watch** (= 場中観察のみ、寄付き状況で再判断)

☐ **skip** (= F111 synth caveat / 値嵩株リスク / 監視不能 / その他理由で見送り)

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

- skip 理由: __________ (例: 「値嵩 6920 株 100 株はリスク上限超過、4063 は根拠
  1 件のみで慎重、本日は skip し D4 で実 batch 稼働待ち」)
- 翌日 D4 (5/19 火) memory: __________

## §14 安全 footer

- **本 plan は手動運用補助** (= 投資助言ではない)
- **FIRE は何も発注しない**
- **楽天証券 / iSPEED は 藤原さん本人が手動操作**
- **Computer Use / Playwright / 自動売買 は使わない**
- **最終判断と発注責任は 藤原さん本人**
- **本 D3 input chain**: F111 synth → F062 runner → AFTER-R1 MVP =
  **artifact_source = f062_preview** ✓
- ただし **F111 朝 batch は未稼働、F111 input は手作り synth (= caveat)**
- **artifact_source = f062_preview + 7 invariants 全 PASS → 構造的 GO 判定**

---

review template → [[2026-05-18_manual_live_pilot_review|review (= 当日 blank)]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D2 (5/15) trade plan → [[2026-05-15_manual_live_pilot_trade_plan|D2 trade plan]]
D3 manual command sequence → [[template_d3_real_artifact_prep|template_d3_real_artifact_prep.md]]
W60-pilot-D3 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D3_results|W60-pilot-D3 results]]
