---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-22
owner: BlueFire7777 (Fujiwara)
pilot_day: D7
status: blank
artifact_source: f062_preview
f111_input_source: f111_real_batch
pilot_judgment: HOLD
d6_review_status: missing
d6_d7_overlap: 100%
related:
  - 04_daily/2026-05-22_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D7_results.md
---

# Manual Live Pilot — Trade Review (2026-05-22 / D7)

## D7 特記: HOLD 推奨 day

- 9 hard invariants 全 PASS だが D6 review missing + 同候補 100% 再現 caveat
- 推奨 = **skip**、ただし D6 で skip した場合は entry 検討可

---

## §1 基本情報

- date: 2026-05-22 (金 / D7)
- 記入時刻: __:__ JST
- trade_plan: `04_daily/2026-05-22_manual_live_pilot_trade_plan.md`
- artifact_source: f062_preview / f111_input_source: f111_real_batch
- ticker / name: ____ / __________

## §2 計画 vs 実際

| 項目 | planned | actual |
|---|---|---|
| entry 価格 | ____ | ____ |
| entry 時刻 | __:__ | __:__ |
| entry 株数 | 100 株 | ___ |
| stop loss | ____ | ____ |
| take profit | ____ | ____ |
| exit 価格 | ____ | ____ |
| exit 時刻 | __:__ | __:__ |
| 保有期間 | __ 分 | __ 分 |

## §3 PnL

- 総 PnL: ______ 円
- 週累計 (= D6+D7、または D6+D7+D6 ポジション): ______ 円
- 1 日上限到達?: ☐ No / ☐ Yes
- 連敗?: ☐ No / ☐ Yes

## §4 Reason for entry / skip

- FIRE why_selected: __________
- D6 残ポジ: __________
- 自分の追加判断: __________

skip 標準理由: "D6 同候補再現で新規 entry 価値低 / D6 review missing / 月曜 D8 待ち"

## §5 Reason for exit

- exit trigger: ☐ stop_loss / ☐ take_profit / ☐ 15:10 cutoff / ☐ 手動判断 / ☐ skip
- 判断理由: __________

## §6 Rule followed?

| ルール | 遵守? | 備考 |
|---|---|---|
| 事前 stop_loss | ☐ Yes / No / skip | |
| 損切ずらさず | ☐ Yes / No / skip | |
| ナンピンなし | ☐ Yes / No / skip | (D6 entry の場合特に重要) |
| 追加買いなし | ☐ Yes / No / skip | |
| 15:10 close | ☐ Yes / No / skip | |
| 持ち越しなし | ☐ Yes / No / skip | |
| 自動発注なし | ☐ Yes (= 構造的) | |
| 楽天 / iSPEED 手動 | ☐ Yes / No / skip | |
| 1 トレード ≤ 15,000 | ☐ Yes / No / skip | |
| 1 日 ≤ 30,000 | ☐ Yes / No / skip | |
| **D6 同候補再現 caveat 認識** | ☐ Yes | |
| **D6 review missing 認識** | ☐ Yes | |

## §7 Liquidity / Spread actual

- 寄付き 板 5 本目処合計: ___ 株
- 寄付き 5 分 出来高: ___ 株
- 寄付き spread: __.__ %
- D6 と比べて: __________ (= 金曜午後 板薄 印象)

## §8 D6 → D7 同候補 再現性 評価

- D6 で entry した場合 → D7 残ポジ処理: __________
- D6 で skip した場合 → D7 再評価判断: __________
- staging signal 日次更新がない caveat の感触: __________

## §9 What worked / failed

- What worked: __________
- What failed: __________
- D7 運用フロー評価 (= D6 → D7 連続 + HOLD 判定): __________

## §10 Improvement

- D8 (= 月曜 5/26 想定、ただし 5/26 は monday) で 変えること: __________
- D8 で 続けること: __________
- staging signal 日次更新 (= F111 朝 batch wave) への期待: __________

## §11 Pattern promote/suppress/watch 提案

- D6 → D7 同候補で signal 安定確認、ただし実 entry なら結論は W2 集約で

## §12 Emergency log

該当しなければ skip。

```
- trigger: __________
- 時刻: __:__ JST
```

## §13 Screenshot / 関連 path memo

- iSPEED SS: __________
- FIRE artifact: `reports/after_r1/*_2026-05-22.{json,md}`
- F111-real-batch: `/tmp/fire_d7_prep/f111_real_batch_2026-05-22.json`

## §14 Next action

- ☐ D8 (= 2026-05-26 月) も pilot 継続
- ☐ liquidity filter 強化 wave 着手
- ☐ W60-pilot-W2 集約 (= D6-D10 後)
- ☐ F111 朝 batch wave 着手 (= staging signal 日次更新)
- ☐ W61-pre (price/return/paper_pnl) 着手

## §15 Stage 3 突合

| 項目 | 当日 | D1-D7 累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 |
| 累積 PnL | ______ 円 | ______ 円 |
| ルール遵守率 | __% | __% |
| f111_real_batch 比率 | 1/1 (D7) | 2/7 (D6+D7) |
| top_candidates ≥1 達成率 | 1/1 (D7) | 2/7 |
| **D6/D7 同候補 overlap** | **100%** | (= staging signal 日次更新なし) |

## §16 安全 footer

- 手動運用補助 / FIRE 発注しない / auto_order=False / manual_review=True /
  楽天正本 / Computer Use 不採用 / 最終判断は藤原さん本人 / D7 chain =
  D6 と同 staging signal / 9 invariants PASS / HOLD 推奨 / 月曜 D8 へ
