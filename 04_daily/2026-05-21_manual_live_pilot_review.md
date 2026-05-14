---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-21
owner: BlueFire7777 (Fujiwara)
pilot_day: D6
status: blank (= 15:30 以降記入)
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
research_enrichment_source: research_watchlist_signals
pilot_use: eligible_with_liquidity_check
pilot_judgment: GO
related:
  - 04_daily/2026-05-21_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D6_results.md
---

# Manual Live Pilot — Trade Review (2026-05-21 / D6)

## ✅✅ D6 特記: 初の実 trade 可能 day (= 9 hard invariants PASS、中小型 liquidity caveat)

D1-D5 全 day 実 entry 0/5 → **D6 で初の実 trade 候補 3 件 ★**

input chain: staging real_batch + research_watchlist_signals enriched →
F062 → AFTER-R1 MVP
= artifact_source=f062_preview / f111_input_source=f111_real_batch /
  top_candidates 3 件

候補: 🟢 8747 豊トラスティ証券 / 🟢 5729 日本精鉱 / 🟢 3489 フェイスネットワーク

藤原さんが §7 liquidity / event check 経由で:
- enter: 自己責任で 1 銘柄 100 株
- watch: 場中観察のみ
- skip: 中小型 liquidity 不確実で見送り

---

## §1 基本情報

- **date**: 2026-05-21 (木 / D6)
- **記入時刻**: __:__ JST (= 15:30 以降推奨)
- **trade_plan**: `04_daily/2026-05-21_manual_live_pilot_trade_plan.md`
- **artifact_source**: f062_preview ✓
- **f111_input_source**: **f111_real_batch ✓** (= D5 f111_sample から進化)
- **research_enrichment_source**: research_watchlist_signals
- **pilot_use**: eligible_with_liquidity_check
- **ticker / name**: ____ / __________

## §2 計画 vs 実際

| 項目 | planned | actual | 差異 |
|---|---|---|---|
| entry 価格 | ____ 円 | ____ 円 | ____ 円 |
| entry 時刻 | __:__ | __:__ | ____ 分 |
| entry 株数 | 100 株 | ___ 株 | __ |
| stop loss | ____ 円 | ____ 円 / ☐ 未到達 | __ |
| take profit | ____ 円 | ____ 円 / ☐ 未到達 | __ |
| exit 価格 | ____ 円 | ____ 円 | __ |
| exit 時刻 | __:__ | __:__ | __ |
| 保有期間 | __ 分 | __ 分 | __ |

## §3 PnL

- 総 PnL: ______ 円
- 本日累計: ______ 円
- 週累計 (= D6 のみで集計開始、D7 以降累積)
- **1 日上限到達?**: ☐ No / ☐ Yes
- 連敗: ☐ No / ☐ Yes (= D6 のみ 不該当)

## §4 Reason for entry / skip

- FIRE の why_selected: __________
- 自分の追加判断 (= 中小型 caveat 考慮): __________
- 採用 pattern: __________
- 寄付き / 場中 で見た指標: __________

skip 標準理由: "中小型 liquidity / 寄付き荒い / event / 監視不能"

## §5 Reason for exit

- exit trigger: ☐ stop_loss / ☐ take_profit / ☐ 15:10 cutoff / ☐ 手動判断 / ☐ skip
- 判断理由: __________

## §6 Rule followed?

| ルール | 遵守? | 備考 |
|---|---|---|
| 事前 stop_loss | ☐ Yes / ☐ No / ☐ skip | __________ |
| 損切りずらさず | ☐ Yes / ☐ No / ☐ skip | |
| ナンピンなし | ☐ Yes / ☐ No / ☐ skip | |
| 追加買いなし | ☐ Yes / ☐ No / ☐ skip | |
| 15:10 close (信用) | ☐ Yes / ☐ No / ☐ skip | |
| 持ち越しなし | ☐ Yes / ☐ No / ☐ skip | |
| 自動発注なし | ☐ Yes (= 構造的) | (= 保証) |
| 楽天 / iSPEED 手動のみ | ☐ Yes / ☐ No / ☐ skip | |
| 1 トレード ≤ 15,000 円 | ☐ Yes / ☐ No / ☐ skip | |
| 1 日 ≤ 30,000 円 | ☐ Yes / ☐ No / ☐ skip | |
| **artifact_source=f062_preview 認識** | ☐ Yes | (= 保証) |
| **f111_input_source=f111_real_batch 認識** | ☐ Yes | (= D6 進化) |
| **9 hard invariants PASS 認識** | ☐ Yes | (= 保証) |
| **中小型 liquidity caveat 認識** | ☐ Yes | (= D6 特記) |

## §7 Liquidity / Spread actual (= D6 中小型 特記)

- 寄付き 板 5 本目処合計: ___ 株
- 寄付き 5 分 出来高: ___ 株
- 寄付き spread: __.__ %
- 場中 spread 推移: __________
- 中小型 liquidity 印象 (= entry 場合): __________
- skip 場合: liquidity が原因だったか / 別理由か: __________

## §8 Pattern matched? (= research enriched)

候補の研究 advisory:
- research_advisory_label = boost
- final_score = 0.91-0.93
- final_rank_label = A1
- watchlist_decision = selected
- strongest_strategy = quality_value 等

実 outcome (= entry 場合):
- pattern 寄与: __________
- 中小型 boost 候補の信頼度: __________

## §9 What worked / what failed

### §9.1 What worked

- __________
- __________

### §9.2 What failed

- __________

### §9.3 Surprise

- __________

### §9.4 D6 運用フロー 評価 (= 初実 trade 可能 day 特記)

- W60-research-advisory-staging enrichment の使い勝手: __________
- staging real_batch → research_advisory → F062 → AFTER-R1 chain の信頼性: __________
- 中小型成長株 liquidity check 欄の有用性: __________
- 9 hard invariants 自動 PASS check の安心感: __________
- 8747 / 5729 / 3489 の実 trade 印象 (= entry 場合): __________

## §10 Improvement

- D7 (= 2026-05-22 金) で 変えること: __________
- D7 で 続けること: __________
- FIRE 候補生成への 要望: __________
- 流動性 filter 強化 (= 別 wave): __________
- price/return/paper_pnl 連携 (= W61-pre): __________

## §11 Pattern promote / suppress / watch 提案 (= D6 = 6 サンプル目、初実候補)

| pattern_id | 提案 | 理由 |
|---|---|---|
| freshness_ok_high_confidence | ☐ promote / ☐ suppress / ☐ watch | (= 初実候補で実 outcome 不明、watch 推奨) |
| manual_review_active_label | ☐ promote / ☐ suppress / ☐ watch | __________ |
| multi_reason_basis | ☐ promote / ☐ suppress / ☐ watch | __________ |
| low_risk_note | ☐ promote / ☐ suppress / ☐ watch | __________ |

(= W2 集約 (= D7-D10 後) で結論)

## §12 Emergency log

該当しなければ skip。

```
- trigger: __________
- 時刻: __:__ JST
- 状況詳細: __________
- 当日損益: ______ 円
- 残ポジ処理: __________
- 翌日対応: ☐ 停止 / ☐ ロット半減 / ☐ 通常再開
- HQ 報告先: __________
```

## §13 Screenshot / 関連 path memo

- iSPEED 画面 SS (= 寄付き / 場中 / 引け): __________
- FIRE artifact: `reports/after_r1/*_2026-05-21.{json,md}`
- F111-real-batch: `/tmp/fire_d6_prep/f111_real_batch_2026-05-21.json`
- DATA-R3 freshness: `/tmp/fire_d6_prep/data_r3_freshness_2026-05-21.json`
- 当日 TDnet (= 候補銘柄関連): __________

## §14 Next action

- ☐ D7 (= 2026-05-22 金) も pilot 継続
- ☐ D7 ロット半減
- ☐ D7 stop
- ☐ ルール 修正提案
- ☐ W60-pilot-W2 集約 (= D6-D10 後) で D6 結果 引き継ぎ
- ☐ liquidity filter 強化 wave 着手
- ☐ W61-pre (price/return/paper_pnl) 着手

## §15 Stage 3 昇格条件 突合 (= D6 = 6 日目、初実候補 day)

| 項目 | 当日 | D1-D6 累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 (R-19-08 目標) |
| 累積 PnL | ______ 円 | ______ 円 |
| ルール遵守率 | __% | __% (目標 ≥ 90%) |
| ルール違反件数 | __ | __ |
| pattern hit 率 | (= 初実候補) | __ |
| emergency stop 発動 | __ | __ |
| **f111_input_source f111_real_batch 比率** | **1/1 (D6) ✓** | **1/6 (= D1-D6 累積)** |
| **top_candidates ≥1 件達成率** | **1/1 (D6) ✓** | **1/6** |

## §16 安全 footer

- 手動運用補助 / FIRE 発注しない / 自動化なし / auto_order=False /
  manual_review=True / 楽天正本 / Computer Use 不採用 / 最終判断は藤原さん本人 /
  D6 chain = staging real_batch + research enriched / 9 hard invariants PASS /
  中小型 liquidity caveat / 初実 trade 可能 day

---

trade plan → [[2026-05-21_manual_live_pilot_trade_plan|D6 trade plan]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D5 review → [[2026-05-20_manual_live_pilot_review|D5 review]]
W60-pilot-D6 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D6_results|W60-pilot-D6 results]]
