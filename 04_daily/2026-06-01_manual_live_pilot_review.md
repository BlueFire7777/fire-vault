---
template: manual-live-pilot-review
version: 1.0
date: 2026-06-01
owner: BlueFire7777 (Fujiwara)
pilot_day: D13
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
d10_d11_d12_review_status: missing
d9_d13_overlap: 100%
related:
  - 04_daily/2026-06-01_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D13_results.md
---

# Manual Live Pilot — Trade Review (2026-06-01 / D13)

## D13 特記: **review gap closure + actual confirmation 強制 day**

- D10-D12 3 連続 review_missing → 本 D13 が 4 回目記入機会
- focused J-Quants refresh = skip (= already_up_to_date)、ただし朝再 refresh 必須
- DATA-R3 freshness 接続経路確認 (= JSON 渡し成功、verdict=MISSING)
- 340A0/3798/137A0 が **D9-D13 5 営業日連続選出** = signal 極めて安定
- 5/14 refresh → 6/1 = 18 営業日 gap、actual confirmation 強制

---

## §1 基本情報

- date: 2026-06-01 (月 / D13)
- 記入時刻: __:__ JST
- trade_plan: `04_daily/2026-06-01_manual_live_pilot_trade_plan.md`
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- ticker / name (entry 銘柄): ____ / __________
- final_decision: ☐ enter / ☐ watch / ☐ skip

## §2 計画 vs 実際

| 項目 | planned (refreshed 5/14) | actual (6/1) |
|---|---|---|
| 候補 #1 close (340A0) | 380 | ____ |
| 候補 #2 close (3798) | 505 | ____ |
| 候補 #3 close (137A0) | 739 | ____ |
| entry 銘柄 | __________ | __________ |
| entry 価格 | ____ | ____ |
| entry 時刻 | 09:00 | __:__ |
| entry 株数 | 100 | ___ |
| stop loss (5%) | ____ | ____ |
| take profit (2%) | ____ | ____ |
| exit 価格 | ____ | ____ |
| exit 時刻 | __:__ | __:__ |
| 保有期間 | __ 分 | __ 分 |

## §3 PnL

- 総 PnL: ______ 円
- 累積 (D1-D13): ______ 円
- 1 日上限到達?: ☐ No / ☐ Yes
- 連敗?: ☐ No / ☐ Yes

## §4 actual price vs latest available close

| code | latest_available (5/14) | actual (6/1) | 乖離率 | 判断 |
|---|---|---|---|---|
| 340A0 | 380 | ____ | __.__% | __________ |
| 3798 | 505 | ____ | __.__% | __________ |
| 137A0 | 739 | ____ | __.__% | __________ |

- 朝再 refresh 結果 (= 6/1 実行): __________
- 乖離 ±5% 以内 → entry 検討
- 乖離 ±5% 超 → skip 強制

## §5 Reason for entry / skip

- FIRE why_selected: __________ (例: 340A0 = F111 rank 4、boost_with_caution、5 連続選出、risk=1,900)
- D9-D12 残ポジ: __________
- D10/D11/D12 review 未記入の影響: __________ (= 3 連続 → 本 D13 で初記入を強く推奨)
- 自分の追加判断: __________
- 4 改善 path の効果 (= review gap closure + refresh skip 明示 + freshness 接続 + actual 強制): __________
- paper PnL preview chain 3 連続運用の感触: __________
- 5 連続同候補 → 6 連続前に entry すべきか / D14 持ち越しか: __________

skip 標準理由: "actual 6/1 と refresh 5/14 の乖離大 / 流動性不足 / event / 5 連続見送り判断"

## §6 Reason for exit

- exit trigger: ☐ stop_loss / ☐ take_profit / ☐ 15:10 cutoff / ☐ 手動判断 / ☐ skip
- 判断理由: __________

## §7 Rule followed?

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
| **refreshed price 認識** | ☐ Yes (= 5/14 cap) | |
| **18 営業日 gap 認識** | ☐ Yes | |
| **freshness MISSING 認識** | ☐ Yes (= 接続経路 OK) | |
| **5 連続同候補 認識** | ☐ Yes | |
| **3 review missing 認識** | ☐ Yes (= D10/D11/D12) | |
| **朝再 refresh 実施** | ☐ Yes / No | |
| **paper PnL handoff 実行予定** | ☐ Yes (= 場後 + 翌日) | |

## §8 Liquidity / Spread actual (= 寄付き 5 分後)

| 候補 | 板厚 5 本目処 | 寄付き 5 分 出来高 | 寄付き spread |
|---|---|---|---|
| 340A0 | ___ 株 | ___ 株 | __.__% |
| 3798 | ___ 株 | ___ 株 | __.__% |
| 137A0 | ___ 株 | ___ 株 | __.__% |

- 月曜朝の板厚印象: __________
- 中小型 (情報通信集中) の板薄印象: __________

## §9 paper PnL handoff 実行記録

### §9.1 D13 当日 (= 場後)

- 実行 cmd:
  ```
  .venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
    --base-date 2026-06-01 --evaluation-date 2026-06-01 \
    --f111-real-batch-json /tmp/fire_d13_prep/d13_f111_real_batch.json \
    --review-md ~/fire-vault/04_daily/2026-06-01_manual_live_pilot_review.md \
    --output-json /tmp/fire_d13_prep/d13_paper_pnl_ledger_eod.json
  ```
- 実行時刻: __:__ JST
- 結果: __________

### §9.2 翌営業日 (= 2026-06-02 火、朝)

- 実行時刻: __:__ JST
- staging max_date 進捗: 5/14 → __ 営業日
- 結果: __________

### §9.3 h20 後 (= 2026-06-26 頃)

- 実行予定: ____
- outcome 期待: __________

## §10 J-Quants refresh 効果 (= 5 連続選出 day としての評価)

- focused refresh skip (= already_up_to_date) は妥当だった?: __________
- 朝再 refresh (= D13 当日 6/1) 実施した?: __________
- 137A0 連続選出の妥当性: __________
- 全銘柄 daily refresh 自動化への期待: __________

## §11 DATA-R3 freshness 接続評価

- AFTER-R1 が --data-r3-freshness-json を受け取った: ☐ Yes / No
- verdict=MISSING のまま entry 判断に影響: __________
- 実 verdict=OK 達成への期待 (= DATA-R3 active job wave): __________

## §12 Rule violation / 反省点

- 寄付き actual price 確認は実施した?: ☐ Yes / No
- 出来高 / spread 確認は実施した?: ☐ Yes / No
- ネガティブニュース確認は実施した?: ☐ Yes / No
- 朝再 refresh 実施した?: ☐ Yes / No
- 5 連続同候補での判断疲労感: __________
- 3 review missing の自己反省: __________

## §13 Pattern matched

- 該当パターン (= 6 種): __________
- 実 entry した場合の pattern score: __________
- D9-D13 5 連続選出 pattern が promote 候補?: __________

## §14 What worked / failed

- What worked: __________
- What failed: __________
- D13 運用フロー評価 (= review gap closure + actual confirmation 強制 + DATA-R3 接続): __________

## §15 Improvement

- D14 で変えること: __________
- D14 で続けること: __________
- DATA-R3 active job wave への期待: __________
- 全銘柄 daily refresh 自動化への期待: __________
- liquidity filter 強化への期待: __________

## §16 Pattern promote/suppress/watch 提案

- 340A0 ジグザグ (= 5 連続) → promote / suppress / watch: __________
- 3798 ＵＬＳ (= 5 連続) → promote / suppress / watch: __________
- 137A0 Cocolive (= 5 連続) → promote / suppress / watch: __________
- 8747/5729/3489 (= 8 連続 demoted) → 引き続き suppress: __________

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
  - D13 F111: `/tmp/fire_d13_prep/d13_f111_real_batch.json`
  - D13 F062: `/tmp/fire_d13_prep/d13_f062_preview.json`
  - D13 morning_line_material: `/tmp/fire_d13_prep/after_r1/morning_line_material_2026-06-01.json`
  - D13 paper PnL ledger (pre-EOD): `/tmp/fire_d13_prep/d13_paper_pnl_ledger.json`
  - DATA-R3 freshness: `/tmp/f286_data_r3_freshness.json`
  - D12 paper PnL ledger (ref): `/tmp/fire_d12_prep/d12_paper_pnl_ledger.json`

## §19 Next action

- ☐ D14 (= 2026-06-02 火) も pilot 継続、recently_seen 再評価
- ☐ DATA-R3 active job wave (= verdict=OK 目標、別 HQ 承認)
- ☐ liquidity filter 強化 wave
- ☐ paper_pnl preview h1/h20 再 run (= D14 朝、6/26 頃)
- ☐ 全銘柄 daily refresh launchd 自動化 (= 別 HQ 承認)
- ☐ W60-pilot-W2 集約 (= D6-D13 8 営業日 review、KPI 算出)

## §20 Stage 3 突合

| 項目 | 当日 | D1-D13 累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 |
| 累積 PnL | __ 円 | __ 円 |
| ルール遵守率 | __% | __% |
| f111_real_batch 比率 | 1/1 | 7/13 |
| top_candidates ≥1 達成率 | 1/1 | 8/13 |
| DATA-R3 freshness 接続経路確認 | ✓ | 1 day (= 初) |
| 340A0 連続選出 | ✓ (D9-D13) | 5 days |

## §21 安全 footer

- 手動運用補助 / FIRE 発注しない / auto_order_allowed=False /
  manual_review_required=True / 楽天証券正本 / Computer Use 不採用 /
  最終判断は Fujiwara / D13 = review gap closure + DATA-R3 接続経路確認 + 5 連続 340A0
