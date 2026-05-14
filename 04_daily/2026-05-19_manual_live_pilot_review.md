---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-19
owner: BlueFire7777 (Fujiwara)
pilot_day: D4
status: blank (= 15:30 以降記入)
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: synthetic_sample
pilot_use: eligible_with_caveat
pilot_judgment: GO (with F111 synth caveat 継続)
related:
  - 04_daily/2026-05-19_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D4_results.md
---

# Manual Live Pilot — Trade Review (2026-05-19 / D4)

## ✅ D4 特記: GO 候補 (= artifact_source=f062_preview)、F111 caveat 継続

input chain: F111 synth (= D3 と同 sample) → F062 → AFTER-R1
= artifact_source=f062_preview / 7 invariants PASS / F111 synth caveat 継続

---

## §1 基本情報

- **date**: 2026-05-19 (火 / D4)
- **記入時刻**: __:__ JST (= 15:30 以降推奨)
- **trade_plan**: `04_daily/2026-05-19_manual_live_pilot_trade_plan.md`
- **artifact_source**: f062_preview ✓
- **f062_raw_kind**: f062_actual_dict ✓
- **f111_input_source**: synthetic_sample (= D3 同)
- **pilot_use**: eligible_with_caveat
- **ticker / name**: ____ / __________ (= skip の場合は "skip")

## §2 計画 vs 実際

| 項目 | planned | actual | 差異 |
|---|---|---|---|
| entry 価格 | ____ 円 | ____ 円 | ____ 円 |
| entry 時刻 | __:__ | __:__ | ____ 分 |
| entry 株数 | ___ 株 | ___ 株 | __ 株 |
| stop loss | ____ 円 | ____ 円 / ☐ 未到達 | __ |
| take profit | ____ 円 | ____ 円 / ☐ 未到達 | __ |
| exit 価格 | ____ 円 | ____ 円 | __ 円 |
| exit 時刻 | __:__ | __:__ | __ 分 |
| 保有期間 | __ 分 | __ 分 | __ 分 |

## §3 PnL

- **総 PnL**: ______ 円
- **本日累計**: ______ 円
- **週累計 (= D1+D2+D3+D4)**: ______ 円
- **1 日上限到達?**: ☐ No / ☐ Yes / ☐ skip
- **2 連敗?**: ☐ No / ☐ Yes / ☐ skip

## §4 Reason for entry / skip

- FIRE の why_selected: __________
- 自分の追加判断 (= F111 caveat + 値嵩株 + D3 同候補): __________
- 採用 pattern: __________
- 寄付き指標: __________
- 後から見て妥当だった点: __________

skip 標準理由: "F111 synth 継続 / 値嵩株 100 株単位リスク超過 / D3 同候補で新規材料なし"

## §5 Reason for exit (= 該当時)

- exit trigger: ☐ stop_loss / ☐ take_profit / ☐ 15:10 cutoff / ☐ 手動判断 / ☐ skip
- 判断理由: __________

## §6 Rule followed?

| ルール | 遵守? | 備考 |
|---|---|---|
| 事前 stop_loss 決定 | ☐ Yes / ☐ No / ☐ skip | __________ |
| 損切り後ろずらさず | ☐ Yes / ☐ No / ☐ skip | __________ |
| ナンピンなし | ☐ Yes / ☐ No / ☐ skip | __________ |
| 追加買いなし | ☐ Yes / ☐ No / ☐ skip | __________ |
| 15:10 close (信用) | ☐ Yes / ☐ No / ☐ skip | __________ |
| 持ち越しなし | ☐ Yes / ☐ No / ☐ skip | __________ |
| 自動発注なし | ☐ Yes (= 構造的) | (= 保証) |
| 楽天 / iSPEED 手動のみ | ☐ Yes / ☐ No / ☐ skip | __________ |
| 1 トレード ≤ 15,000 円 | ☐ Yes / ☐ No / ☐ skip | __________ |
| 1 日 ≤ 30,000 円 | ☐ Yes / ☐ No / ☐ skip | __________ |
| **artifact_source=f062_preview 認識** | ☐ Yes | (= 保証) |
| **F111 synth caveat 認識** | ☐ Yes | (= 保証) |

## §7 Pattern matched? (= D3 と同 pattern)

採用 pattern (= 6920 / 4063):
- `freshness_ok_high_confidence` (= 6920 のみ): ☐ matched / ☐ NA / ☐ skip
- `manual_review_active_label` (= 6920, 4063): ☐ matched / ☐ NA / ☐ skip
- `multi_reason_basis` (= 6920 のみ): ☐ matched / ☐ NA / ☐ skip
- `low_risk_note` (= 6920, 4063): ☐ matched / ☐ NA / ☐ skip

結果との相関 (= 主観):
- 良い結果に寄与した pattern: __________
- 良い結果に寄与しなかった pattern: __________
- 誤シグナル: __________

## §8 What worked / what failed

### §8.1 What worked

- __________
- __________

### §8.2 What failed

- __________

### §8.3 Surprise

- __________

### §8.4 D4 運用フロー 評価 (= 特記)

- D3 と同 sequence の再現性: __________
- F111 synth caveat 継続の感覚: __________
- 値嵩株リスクで skip 判断する感覚: __________
- D3 → D4 同候補の連続性 (= 2 営業日 連続表示): __________
- F111 朝 batch 稼働への期待 (= Wave 41 着手 5/19-5/26): __________

## §9 Improvement

- D5 (= 2026-05-20 水) で 変えること: __________
- D5 で 続けること: __________
- FIRE 候補生成への 要望 (例: 値嵩株 100 株単位対応 minilot): __________
- F111 朝 batch 化 期待: __________

## §10 Pattern promote / suppress / watch 提案 (= D4 = 4 サンプル目)

| pattern_id | 提案 | 理由 |
|---|---|---|
| freshness_ok_high_confidence | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |
| manual_review_active_label | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |
| multi_reason_basis | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |
| low_risk_note | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |

(= D5 W1 集約 (= W60-pilot-W1) で結論)

## §11 Emergency log (= 該当時)

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

## §12 Screenshot / 関連 path memo (= optional)

- iSPEED 画面 SS: __________
- FIRE artifact path: `reports/after_r1/*_2026-05-19.{json,md}`
- F062 actual output: `/tmp/fire_d4_prep/f062_actual_output_2026-05-19.json`
- DATA-R3 freshness: `/tmp/fire_d4_prep/data_r3_freshness_2026-05-19.json`
- 当日 TDnet (= 6920/4063 関連): __________

## §13 Next action

- ☐ D5 (= 2026-05-20 水) も pilot 継続
- ☐ D5 ロット半減
- ☐ D5 stop
- ☐ ルール 修正提案
- ☐ W60-pilot-W1 (= 2026-05-21 木以降) 集約準備
- ☐ Wave 41 進捗確認 (= F111 朝 batch 化期待)

## §14 Stage 3 昇格条件 突合 (= D4 = 4 日目)

| 項目 | 当日値 | 5 日累積 (D1-D4) |
|---|---|---|
| trade 回数 | __ | __ / 50 |
| 累積 PnL | ______ 円 | ______ 円 |
| ルール遵守率 | __% | __% (目標 ≥ 90%) |
| ルール違反件数 | __ | __ |
| pattern hit 率 | __% | __% |
| emergency stop 発動 | __ | __ |
| artifact_source f062_preview 比率 | 2/4 (= D3 D4) | __ / 5 |
| F111 朝 batch 稼働率 | 0/4 (= 全 synth) | __ / 5 |

## §15 安全 footer

- 手動運用補助 / FIRE 発注しない / 自動化なし / auto_order=False /
  manual_review=True / 楽天正本 / Computer Use 不採用 / 最終判断は藤原さん本人 /
  D4 input chain = F111 synth → F062 → AFTER-R1 = f062_preview (with caveat)

---

trade plan → [[2026-05-19_manual_live_pilot_trade_plan|D4 trade plan]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D3 (5/18) review → [[2026-05-18_manual_live_pilot_review|D3 review]]
W60-pilot-D4 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D4_results|W60-pilot-D4 results]]
