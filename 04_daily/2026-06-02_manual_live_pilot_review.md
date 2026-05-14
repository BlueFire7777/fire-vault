---
template: manual-live-pilot-review
version: 1.0
date: 2026-06-02
owner: BlueFire7777 (Fujiwara)
pilot_day: D14
status: blank
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
preset_tune_applied: true
label_threshold_mode: strict
recently_seen_codes: ["8747", "5729", "3489"]
jquants_refresh_applied: true
market_prices_daily_max_date: 2026-05-14
data_r3_freshness_verdict: MISSING
pilot_judgment: GO_CONDITIONAL
hard_check_passed: 12/12
wave_purpose: 初回少額実弾開始判定 (= W2 設計 doc 初運用)
d10_d11_d12_d13_review_status: missing
d9_d14_overlap: 100%
related:
  - 04_daily/2026-06-02_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D14_results.md
  - 03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
---

# Manual Live Pilot — Trade Review (2026-06-02 / D14)

## D14 特記: **初回少額実弾開始判定 day**

- W2 設計 doc 初運用、hard check 12/12 PASS
- 判定 = GO_CONDITIONAL (= 朝寄付き 3 確認後 GO 寄り)
- 340A0 が D9-D14 6 営業日連続 entry 候補 #1
- D10-D13 5 review missing → 本 D14 が **初の本格記入機会** (= 最低 5 項目必須)
- ⚠ 5/14 refresh → 6/2 = 19 営業日 gap、朝再 refresh + actual confirmation 必須

---

## §1 [必須] 基本情報 (= 設計 doc §5 #1)

- date: 2026-06-02 (火 / D14)
- **記入時刻**: __:__ JST ★ 必須記入
- trade_plan: `04_daily/2026-06-02_manual_live_pilot_trade_plan.md`
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- **ticker / name (entry 銘柄)**: ____ / __________ ★ 必須記入

## §2 [必須] 計画 vs 実際 (= 設計 doc §5 #2)

| 項目 | planned (refreshed 5/14) | actual (6/2) |
|---|---|---|
| 候補 #1 close (340A0) | 380 | ____ |
| 候補 #2 close (3798) | 505 | ____ |
| 候補 #3 close (137A0) | 739 | ____ |
| 候補 #4 close (7991) | 1,177 | ____ |
| **entry 銘柄** ★ 必須 | __________ | __________ |
| **entry 価格** ★ 必須 | ____ | ____ |
| **entry 時刻** ★ 必須 | 09:00 | __:__ |
| **entry 株数** ★ 必須 | 100 | ___ |
| stop loss (5%) | ____ | ____ |
| take profit (2%) | ____ | ____ |
| exit 価格 | ____ | ____ |
| exit 時刻 | __:__ | __:__ |
| 保有期間 | __ 分 | __ 分 |

## §3 [必須] PnL (= 設計 doc §5 #3)

- **総 PnL** ★ 必須: ______ 円 (= 円単位 or "skip")
- 累積 (D1-D14): ______ 円
- 1 日上限到達?: ☐ No / ☐ Yes
- 連敗?: ☐ No / ☐ Yes

## §4 [必須] actual price check result (= W2 設計 §3.1 GO 条件 #10)

| code | latest_available (5/14) | actual (6/2) | 乖離率 | 判断 |
|---|---|---|---|---|
| 340A0 | 380 | ____ | __.__% | __________ |
| 3798 | 505 | ____ | __.__% | __________ |
| 137A0 | 739 | ____ | __.__% | __________ |
| 7991 | 1,177 | ____ | __.__% | __________ |

- 乖離 ±5% 以内 → entry 検討
- 乖離 ±5% 超 → skip 強制
- ★ **actual price check result 必須記入**: __________ (例: "全 ±3% 以内、entry OK")

## §5 [必須] Reason for entry / skip (= 設計 doc §5 #4)

★ 必須記入 (= 1 文以上):

- FIRE why_selected: __________
- 自分の追加判断: __________
- 6 連続同候補 → 7 連続前に entry すべきか / D15 持ち越しか: __________
- **理由 1 文** ★: __________

skip 標準理由: "actual 6/2 と refresh 5/14 の乖離大 / 流動性不足 / event / 6 連続見送り判断"

## §6 [必須] final decision checkbox (= 設計 doc §5 #5)

★ 1 つ選択 必須:

- ☐ **enter 340A0** (= ★ entry candidate #1)
- ☐ enter 3798
- ☐ enter 137A0
- ☐ enter 7991 (= sector 多様化)
- ☐ watch (= 寄付き後再判断)
- ☐ skip (= D15 持ち越し)

## §7 liquidity / spread result (= 寄付き 5 分後)

| 候補 | 板厚 5 本目処 | 寄付き 5 分 出来高 | 寄付き spread | 採否 |
|---|---|---|---|---|
| 340A0 | ___ 株 | ___ 株 | __.__% | ☐ OK / ☐ NG |
| 3798 | ___ 株 | ___ 株 | __.__% | ☐ OK / ☐ NG |
| 137A0 | ___ 株 | ___ 株 | __.__% | ☐ OK / ☐ NG |
| 7991 | ___ 株 | ___ 株 | __.__% | ☐ OK / ☐ NG |

- ★ liquidity 確認結果: __________

## §8 event / earnings result (= 寄付き前)

| 候補 | TDnet 開示 | 決算予定 | ニュース | 採否 |
|---|---|---|---|---|
| 340A0 | __________ | __________ | __________ | ☐ OK / ☐ NG |
| 3798 | __________ | __________ | __________ | ☐ OK / ☐ NG |
| 137A0 | __________ | __________ | __________ | ☐ OK / ☐ NG |
| 7991 | __________ | __________ | __________ | ☐ OK / ☐ NG |

- ★ event 確認結果: __________

## §9 Reason for exit

- exit trigger: ☐ stop_loss / ☐ take_profit / ☐ 15:10 cutoff / ☐ 手動判断 / ☐ skip
- 判断理由: __________

## §10 Rule followed?

| ルール | 遵守? | 備考 |
|---|---|---|
| 事前 stop_loss 設定 | ☐ Yes / No / skip | 5% |
| 損切ずらさず | ☐ Yes / No / skip | |
| ナンピンなし | ☐ Yes / No / skip | |
| 追加買いなし | ☐ Yes / No / skip | |
| 15:10 close | ☐ Yes / No / skip | |
| 持ち越しなし | ☐ Yes / No / skip | |
| 自動発注なし | ☐ Yes (= 構造的) | |
| 楽天 / iSPEED 手動 | ☐ Yes / No / skip | |
| 1 トレード ≤ 15,000 円 | ☐ Yes / No / skip | |
| 1 日 ≤ 30,000 円 | ☐ Yes / No / skip | |
| 朝再 refresh 実施 | ☐ Yes / No | |
| 6 連続同候補認識 | ☐ Yes | |
| W2 設計 doc 12 項目認識 | ☐ Yes | |
| paper PnL handoff 実行予定 | ☐ Yes (= 場後 + 翌日) | |

## §11 J-Quants refresh 効果 (= D14 朝)

- 朝再 refresh (= 6/2 当日) 実施: ☐ Yes / No
- staging max_date 進捗: 5/14 → __ 営業日
- 12 銘柄 actual close 更新: __________
- 340A0 actual close vs 5/14 close: ____ vs 380 (= __% 乖離)

## §12 paper PnL handoff 実行記録

### §12.1 D14 当日 (= 場後)

- 実行 cmd:
  ```
  .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-06-02 --evaluation-date 2026-06-02 \
    --f111-real-batch-json /tmp/fire_d14_prep/d14_f111_real_batch.json \
    --review-md ~/fire-vault/04_daily/2026-06-02_manual_live_pilot_review.md \
    --output-json /tmp/fire_d14_prep/d14_paper_pnl_ledger_eod.json
  ```
- 実行時刻: __:__ JST
- 結果: __________

### §12.2 D15 翌朝 (= 2026-06-03 水)

- 実行時刻: __:__ JST
- h1 close 結果: __________

### §12.3 h20 後 (= 2026-06-26〜30 頃)

- 実行予定日: ____
- outcome 期待: __________

## §13 Pattern matched

- 該当パターン (= 6 種: refreshed_price_ok / manual_review_active / research_boost / risk_within_pilot_limit / multi_reason / low_risk): __________
- 実 entry した場合の pattern score: __________
- D9-D14 6 連続選出 pattern が promote 候補?: __________

## §14 What worked / failed

- What worked: __________
- What failed: __________
- D14 = 初回少額実弾開始判定 day の評価: __________

## §15 Improvement

- D15 で変えること: __________
- D15 で続けること: __________
- W2 設計 doc 12 項目 hard check の使い勝手評価: __________
- DATA-R3 active job wave への期待: __________

## §16 Pattern promote/suppress/watch 提案

- 340A0 ジグザグ (= 6 連続) → promote / suppress / watch: __________
- 3798 ＵＬＳ (= 6 連続) → promote / suppress / watch: __________
- 137A0 Cocolive (= 6 連続) → promote / suppress / watch: __________
- 8747/5729/3489 (= 9 連続 demoted) → 引き続き suppress: __________

## §17 Emergency log

該当しなければ skip。

```
- trigger: __________
- 時刻: __:__ JST
- 対応: __________
```

## §18 Screenshot / 関連 path memo

- iSPEED SS: __________
- FIRE artifact:
  - D14 F111: `/tmp/fire_d14_prep/d14_f111_real_batch.json`
  - D14 F062: `/tmp/fire_d14_prep/d14_f062_preview.json`
  - D14 morning_line_material: `/tmp/fire_d14_prep/after_r1/morning_line_material_2026-06-02.json`
  - D14 paper PnL ledger (pre-EOD): `/tmp/fire_d14_prep/d14_paper_pnl_ledger.json`
  - DATA-R3 freshness: `/tmp/f286_data_r3_freshness.json`
  - W2 設計 doc: `~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md`

## §19 Next action

- ☐ D15 (= 2026-06-03 水) も pilot 継続、D14 entry 銘柄を recently_seen 追加検討
- ☐ paper PnL handoff (= D14 場後 / 翌朝 / h20 後)
- ☐ DATA-R3 active job wave (= verdict=OK 目標、別 HQ 承認)
- ☐ liquidity filter 強化 wave
- ☐ W60-pilot-W3 集約 (= D14-D18 後)

## §20 Stage 3 突合

| 項目 | 当日 | D1-D14 累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| ルール遵守率 | __% | __% |
| f111_real_batch 比率 | 1/1 | 8/14 |
| top_candidates ≥1 達成率 | 1/1 | 9/14 |
| W2 設計 doc 初運用 | ✓ | 1 day (= 初) |
| D14 hard check 12/12 | ✓ | 1 day (= 初) |
| 340A0 連続選出 | ✓ (D9-D14) | 6 days |

## §21 安全 footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D14 = 初回少額実弾開始判定 / W2 設計 doc 初運用 /
  hard check 12/12 / 6 連続 340A0
