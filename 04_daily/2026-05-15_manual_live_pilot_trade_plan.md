---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-15
owner: BlueFire7777 (Fujiwara)
pilot_day: D2
input_source: synthetic_fixture (= /tmp/fire_w60_smoke/、実 F062 朝 batch 未稼働)
artifact_source: synthetic_fixture
pilot_use: skip_recommended
pilot_judgment: HOLD
related:
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_results.md
  - 04_daily/2026-05-14_manual_live_pilot_trade_plan.md (= D1)
---

# Manual Live Pilot — Trade Plan (2026-05-15 / D2)

## ⚠ D2 判定: HOLD (= SKIP_RECOMMENDED)

**artifact_source = `synthetic_fixture`** (= 4 成果物全て、自動推定)。
実 F062 朝 batch は 5/15 時点でも未稼働 (= launchd 未配置、W44-pre / Wave 41/45
進行中)。

**判定: HOLD** — 形式は OK だが **実 F062 batch 由来ではない** ため、
**実 trade は非推奨**。運用フロー検証のみ。
迷いなく **skip** が原則的選択。

`artifact_source = f062_preview` が実現するのは:
- F062 朝 batch が実稼働 + AFTER-R1 MVP に F062 actual output を渡した時
- 現状 (= 5/15 朝) 実稼働なし → synthetic 入力での運用フロー検証

参考: `/tmp/fire_w60_pilot_d2_ref/output/` に F062 actual format (= 6920 レーザーテック
/ 4063 信越化学) fixture を渡した場合の試走 artifact あり (= artifact_source=
f062_preview 判定実証用、ただし内容は手作り)。これは正本ではない。

---

## §1 基本情報

- **date**: 2026-05-15 (金曜日 / D2)
- **記入時刻**: 自動生成 (= claude code が 02:xx JST に当日朝向け生成)
- **本日の本業 / 監視可能時間**: ☐ 9:00-15:30 監視可 / ☐ 部分監視 / ☐ 監視不能
  → 藤原さん記入
- **15:10 までに手動 close 可能か**: ☐ Yes / ☐ No
- **大型イベント (FOMC / 雇用統計 / 重要 IR / 決算跨ぎ)**: ☐ なし / ☐ あり
  (= 5/15 金 は SQ 週終了直後、特殊需給注意)

## §2 市場コンディション (= 寄付き前 藤原さん記入)

- 日経平均 寄付き前: __,___ (前日比 +__/-__)
- TOPIX 寄付き前: ____ (前日比 +__/-__)
- 米国 引け: NYダウ +__/-__、S&P +__/-__、NASDAQ +__/-__
- ドル円: ___.__ 円
- 自分の地合い judgement: ☐ 強気 / ☐ 中立 / ☐ 弱気 / ☐ 不明

## §3 FIRE report paths (= 5/15 当日 生成済)

- `reports/after_r1/morning_line_material_2026-05-15.md` ✓
- `reports/after_r1/good_candidate_ranking_2026-05-15.md` ✓
- `reports/after_r1/pattern_candidate_report_2026-05-15.md` ✓
- `reports/after_r1/paper_live_ledger_2026-05-15.md` ✓

### §3.0 artifact_source (= W60-integration 必須)

- **artifact_source**: ☑ `synthetic_fixture` / ☐ `f062_preview` / ☐ `unknown`
- **f062_raw_kind**: ☑ `plain_list` / ☐ `f062_actual_dict` / ☐ `other_dict` / ☐ `none`
- **data_r3_freshness_verdict**: ☑ `OK` / ☐ `FAILED` / ☐ `MISSING`
- **pilot_use**: ☑ **`skip_recommended`** / ☐ `eligible` / ☐ `no_go`
- **pilot_judgment**: ☑ **HOLD** / ☐ GO / ☐ NO-GO

artifact_source = `synthetic_fixture` の根拠:
- 4 成果物 root field `artifact_source` = `synthetic_fixture`
- `f062_raw_kind` = `plain_list` (= W60-impl smoke 用 hardcoded 6 銘柄 list)
- 実 F062 朝 batch output 形式 (= dict with chunks/selected_count/...) ではない

### §3.1 共通 GO-check

- [x] morning_line_material の `forbidden_phrases_check.passed = True` ✓
- [x] paper_live_ledger 全 entry で `auto_order_allowed = False` ✓
- [x] paper_live_ledger 全 entry で `manual_review_required = True` ✓
- [x] DATA-R3 freshness verdict = `OK` ✓ (= ただし synthetic 由来)
- [x] ranking と morning_line_material の top ticker が **整合** ✓
- [x] 出力 path が `reports/after_r1/` 配下 ✓
- ☑ **artifact_source が `synthetic_fixture`** → **HOLD/skip_recommended** ⚠

## §4 Top candidates (= morning_line_material から自動転記)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | 7203 | トヨタ自動車 | 積極的買い推奨 | 🟢 | 0.85 | 194.0 |
| 2 | 6758 | ソニーG | 積極的買い推奨 | 🟢 | 0.72 | 179.0 |
| 3 | 9984 | ソフトバンクG | 条件付き買い推奨 | 🟡 | 0.65 | 138.0 |

(= D1 と同じ candidates、同じ synthetic fixture 使用)

## §5 D2 推奨選択肢 (= synthetic 段階のため skip 推奨)

### §5.1 推奨パターン: **skip**

artifact_source = `synthetic_fixture` のため、本 candidates は **形式検証用**。
実 F062 朝 batch 由来でないので、**実 trade は非推奨**。

藤原さんが「運用フロー検証として手動 trade を試したい」と判断した場合は
自己責任で 7203 1 銘柄 で entry も可だが、原則は **skip**。

### §5.2 GO 条件不充足の理由

- §3.0 artifact_source = synthetic_fixture
- 実 F062 朝 batch 未稼働 (= 5/14 D1 と同条件)
- 当日材料の **真実性が保証されない** (= 手作り data)

### §5.3 GO 判定にするためには

- 実 F062 朝 batch が稼働 (= W44-pre / Wave 41/45 完了後)
- artifact_source = `f062_preview` で生成
- 上記が満たせなければ Stage 3 移行までは synthetic で運用フロー検証

## §6 Selected ticker (= 藤原さん が top 1-3 から手動選択 / skip)

- **ticker**: ____ (= 推奨: **skip**)
- **name**: __________
- **label**: __________
- **rank**: #__
- **score**: _____
- **confidence**: __.__

### §6.1 Why selected (= 候補選択理由、自分の言葉で / skip 理由)

- FIRE の why_selected 要約: __________
- 自分の追加判断: __________
- pattern_tags: __________
- risk_notes: __________

### §6.2 採用 pattern (= pattern_candidate_report から)

- [ ] `freshness_ok_high_confidence` (= 7203, 6758)
- [ ] `manual_review_active_label` (= 7203, 6758, 9984)
- [ ] `multi_reason_basis` (= 7203, 6758)
- [ ] `low_risk_note` (= 7203, 6758, 9984)

(= D1 と同じ pattern matched、全件 `validation_status = unvalidated`)

## §7 Entry plan (= skip の場合は省略可)

- **entry 価格目安**: ____ 円
- **entry 株数**: ___ 株
- **必要資金**: ______ 円
- **entry 時刻目安**: ☐ 寄付き成行 / ☐ 寄付き 5 分後 / ☐ 場中 押し目 / ☐ skip
- **entry 条件** (= 寄付き直前確認):
  - [ ] 出来高 平均比 ≥ 100%
  - [ ] 気配 寄付き ±2% 以内
  - [ ] 板 気配本数 ≥ 5 本
  - [ ] 同セクター上位が上昇
  - [ ] 日経 / TOPIX が逆風でない
  - [ ] **artifact_source = f062_preview** (= **本日は不該当 ⚠**)

## §8 Stop loss

- **stop loss 価格**: ____ 円 (= entry − 5%)
- **想定リスク**: 1 株 ___ 円 × 100 株 = ______ 円
- **1 トレード最大損失 (= 5,000-15,000 円) 内**: ☐ Yes / ☐ No
- **損切ずらさない**: ☐ Yes (= 自分への約束)

## §9 Take profit

- **take profit 価格**: ____ 円 (= entry + 3%)
- **想定利益**: 1 株 ___ 円 × 100 株 = ______ 円
- **リスク・リワード比**: ___ : ___ (= ≥ 1.5 推奨)

## §10 No-trade (= skip) 条件 (= 1 件 該当 → skip)

- [ ] `reason_tags` 薄い (0-1 件) — 9984 は要注意
- [ ] `risk_notes` 重大 — top 3 全 空 ✓
- [ ] 出来高弱い (平均比 < 50%)
- [ ] 寄付き荒い (気配 ±5% 以上)
- [ ] 指数逆風 (日経 -1% 超)
- [ ] セクター弱い
- [ ] 板薄い (< 5,000 株 / 5 本)
- [ ] 決算 / 重要 IR / FOMC 直前
- [ ] FIRE 候補と自分の裁量が一致しない
- [ ] 監視時間が取れない
- [ ] レポート不整合 — 本日整合 ✓
- ☑ **artifact_source ≠ f062_preview** → **本日 該当 ⚠**

## §11 Manual buy checklist (= entry 前 最終確認 / skip では skip 可)

- [x] §3.1 共通 GO-check 全 PASS (= synthetic 由来除く構造的 check) ✓
- [ ] §7 entry plan 確定
- [ ] §8 stop loss 確定 (= 価格明文化)
- [ ] §9 take profit 確定
- ☑ §10 no-trade 条件 該当 (= artifact_source ≠ f062_preview)
- [ ] 1 トレード最大損失 ≤ 15,000 円
- [ ] 当日累計損失 + 想定 stop loss ≤ 30,000 円
- [ ] 2 連敗していない (= D2 連敗概念 NA)
- [ ] ナンピン / 追加買い しない
- [ ] 15:10 までに手動 close できる
- [x] **auto_order_allowed = False** ✓
- [x] **manual_review_required = True** ✓
- [ ] 楽天証券 / iSPEED 操作は 自分の手動作業
- ☑ **本日入力 が synthetic fixture を認識**

## §12 D2 リスク上限 (= operation_plan §5 / m=0 R-39-02)

| 項目 | 上限 | D2 想定 |
|---|---|---|
| 1 日銘柄数 | 最大 1-2 / **D2 は skip 推奨** | (= skip) |
| 1 トレード最大損失 | 5,000-15,000 円 | (= skip) |
| 1 日最大損失 | 20,000-30,000 円 | (= skip) |
| 連敗停止 | 2 連敗 = 当日停止 | (= D2 = D1 skip だった場合不該当) |
| 最大建玉 | 1-2 銘柄 | __ |
| ナンピン / 追加買い | 禁止 | — |
| 持ち越し | 禁止 | — |
| 信用 15:10 close | 必須 | — |
| 迷ったら見送り | 原則 | **本日該当** |

## §13 Final decision

☐ **enter** (= 自己責任で synthetic 候補に従い手動発注、原則非推奨)

☐ **watch** (= 場中観察のみ、トレード見送り)

☑ **skip** (= 推奨選択肢、synthetic fixture 段階のため見送り)

### §13.1 skip の場合

- skip 理由: artifact_source = `synthetic_fixture` (= 実 F062 朝 batch 未稼働)
- 翌日 D3 (= 2026-05-18 月) memory:
  - 5/16 土 / 5/17 日 skip
  - 5/18 月 朝も artifact_source を必ず確認
  - F062 朝 batch 稼働状態を W60-launchd-pre / Wave 41 進捗で確認

## §14 安全 footer

- **本 plan は手動運用補助** (= 投資助言ではない)
- **FIRE は何も発注しない**
- **楽天証券 / iSPEED は 藤原さん本人が手動操作**
- **Computer Use / Playwright / 自動売買 は使わない**
- **最終判断と発注責任は 藤原さん本人**
- **本 D2 入力は synthetic fixture** (= 実 F062 朝 batch / 実 DATA-R3 batch ではない)
- **artifact_source = synthetic_fixture → pilot_use=skip_recommended → 判定 HOLD**

---

review template → [[2026-05-15_manual_live_pilot_review|review (= 当日 blank)]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D1 trade plan → [[2026-05-14_manual_live_pilot_trade_plan|D1 (5/14 木)]]
W60-pilot-D2 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_results|W60-pilot-D2 results]]
