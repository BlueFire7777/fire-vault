---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-15
owner: BlueFire7777 (Fujiwara)
pilot_day: D2
status: blank (= 15:30 以降 / 終日 skip 時の確認用)
artifact_source: synthetic_fixture
pilot_use: skip_recommended
related:
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/2026-05-15_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_results.md
---

# Manual Live Pilot — Trade Review (2026-05-15 / D2)

## ⚠ D2 特記: artifact_source = synthetic_fixture → HOLD/skip 推奨

本日 (2026-05-15) の入力は **synthetic fixture** (= W60-impl smoke 用 hardcoded
6 銘柄 plain list)。実 F062 朝 batch / 実 DATA-R3 batch ではない。

trade plan (§13 final decision) の推奨は **skip** (= 運用フロー検証のみ)。

**skip 選択した場合**:
- 「synthetic 入力のため実 trade なし、運用フロー検証 + template 記入練習のみ」
- §2-§13 は最小記入で OK、§9 §13 next action に重点

**enter 選択した場合** (= 自己責任で):
- §2 以降を 15:30 以降に記入

---

## §1 基本情報

- **date**: 2026-05-15 (金曜日 / D2)
- **記入時刻**: __:__ JST (= 15:30 以降推奨)
- **trade_plan path**: `04_daily/2026-05-15_manual_live_pilot_trade_plan.md`
- **入力 source**: synthetic_fixture (= D2 特記)
- **artifact_source**: synthetic_fixture
- **pilot_use**: skip_recommended
- **ticker / name**: ____ / __________ (= skip の場合は "skip")
- **label**: __________

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

(= skip の場合は全行 "skip")

## §3 PnL

- **総 PnL**: ______ 円
- **本日累計 PnL**: ______ 円
- **週累計 PnL** (= D1+D2): ______ 円
- **1 日上限到達?**: ☐ No / ☐ Yes / ☐ skip 該当なし
- **2 連敗?** (= D1 enter + 負け & D2 enter + 負け): ☐ No / ☐ Yes / ☐ skip

## §4 Reason for entry / skip

- FIRE の why_selected: __________
- 自分の追加判断: __________
- 採用 pattern: __________
- 寄付きで見た指標: __________
- 後から見て妥当だった点: __________

skip の場合の標準理由: "artifact_source = synthetic_fixture、実 F062 朝 batch 由来でないため運用フロー検証のみ"

## §5 Reason for exit (= 該当時)

- exit trigger: ☐ stop_loss / ☐ take_profit / ☐ 15:10 cutoff / ☐ 手動判断 /
  ☐ skip 該当なし
- 判断理由: __________

## §6 Rule followed?

| ルール | 遵守? | 備考 |
|---|---|---|
| 事前に stop_loss を決めた | ☐ Yes / ☐ No / ☐ skip | __________ |
| 損切りを後ろにずらさなかった | ☐ Yes / ☐ No / ☐ skip | __________ |
| ナンピンしなかった | ☐ Yes / ☐ No / ☐ skip | __________ |
| 追加買いしなかった | ☐ Yes / ☐ No / ☐ skip | __________ |
| 15:10 までに手動 close した (信用) | ☐ Yes / ☐ No / ☐ skip | __________ |
| 持ち越ししなかった | ☐ Yes / ☐ No / ☐ skip | __________ |
| 自動発注を使わなかった | ☐ Yes (= FIRE 構造的) | (= 保証) |
| 楽天 / iSPEED 手動操作のみ | ☐ Yes / ☐ No / ☐ skip | __________ |
| 1 トレード最大損失 ≤ 15,000 円 | ☐ Yes / ☐ No / ☐ skip | __________ |
| 1 日最大損失 ≤ 30,000 円 | ☐ Yes / ☐ No / ☐ skip | __________ |
| **入力が synthetic だと認識 + skip 推奨理解** | ☐ Yes (= D2 特記) | (= 保証) |
| **artifact_source = synthetic_fixture を確認** | ☐ Yes (= D2 特記) | (= 保証) |

## §7 Pattern matched? (= pattern_candidate_report との突合)

採用 pattern (= D1 と同じ synthetic fixture):
- `freshness_ok_high_confidence` (= 7203, 6758): ☐ matched / ☐ NA (= skip)
- `manual_review_active_label` (= 7203, 6758, 9984): ☐ matched / ☐ NA
- `multi_reason_basis` (= 7203, 6758): ☐ matched / ☐ NA
- `low_risk_note` (= 7203, 6758, 9984): ☐ matched / ☐ NA

skip の場合: "synthetic 段階のため real pattern check 不可、validation_status=unvalidated 維持"

## §8 What worked / what failed

### §8.1 What worked

- __________
- __________

### §8.2 What failed

- __________
- __________

### §8.3 Surprise

- __________

### §8.4 D2 運用フロー 評価 (= 特記)

- artifact_source 区別 (= W60-integration 機能) の使い勝手: __________
- skip 判定の明確さ: __________
- D1 と比較した改善点: __________
- F062 朝 batch 稼働への期待: __________
- F062 actual format reference (= /tmp/fire_w60_pilot_d2_ref/) との見比べ: __________

## §9 Improvement

- 翌 D3 (= 2026-05-18 月) で 同様の場面で **変えること**: __________
- D3 で **続けること**: __________
- FIRE 候補生成への **要望**: __________
- 設計 doc への **要望**: __________

## §10 Pattern promote / suppress / watch 提案 (= D2 = 2 サンプル目)

| pattern_id | 提案 | 理由 |
|---|---|---|
| freshness_ok_high_confidence | ☐ promote / ☐ suppress / ☐ watch / ☐ skip (D2) | __________ |
| manual_review_active_label | ☐ promote / ☐ suppress / ☐ watch / ☐ skip (D2) | __________ |
| multi_reason_basis | ☐ promote / ☐ suppress / ☐ watch / ☐ skip (D2) | __________ |
| low_risk_note | ☐ promote / ☐ suppress / ☐ watch / ☐ skip (D2) | __________ |

(= D5 W1 集約で結論、本日は memo)

## §11 Emergency log (= 該当時のみ記入)

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
- 楽天 Web 画面 SS: __________
- FIRE artifact path: `reports/after_r1/*_2026-05-15.{json,md}`
- F062 actual format reference: `/tmp/fire_w60_pilot_d2_ref/output/`
- 当日 TDnet 開示: __________

## §13 Next action

- ☐ D3 (= 2026-05-18 月) も pilot 継続 (= 引き続き artifact_source 確認)
- ☐ D3 ロット半減
- ☐ D3 stop (= §11 該当時)
- ☐ ルール 修正提案 (= 詳細 §9)
- ☐ Pilot 1 週間目 review (= D5 / 5 営業日目 = 2026-05-20 水)
- ☐ **F062 朝 batch 稼働確認**:
  - W60-launchd-pre 設計確認
  - Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY 進捗
  - Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY 進捗

(= D1-D5 営業日 path: D1=5/14 木 / D2=5/15 金 / skip 5/16 土 5/17 日 /
D3=5/18 月 / D4=5/19 火 / D5=5/20 水)

## §14 Stage 3 昇格条件との 突合 (= D2 = 2 日目)

| 項目 | 当日値 | 5 日累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 (R-19-08 目標) |
| 累積 PnL | ______ 円 | ______ 円 |
| ルール遵守率 | __% | __% (目標 ≥ 90%) |
| ルール違反件数 | __ | __ |
| pattern hit 率 | __% (= D2 = サンプル不足) | __% |
| emergency stop 発動 | __ | __ |
| artifact_source f062_preview 比率 | 0/2 (= synthetic のみ) | __ / 5 |

## §15 安全 footer

- **本 review は学習記録** (= 投資助言ではない、**手動運用補助** のみ)
- **FIRE は何も発注しない、何も自動化しない**
- **`auto_order_allowed = False` / `manual_review_required = True`** (= MVP 不変量、本 review 対象 trade も該当)
- **建玉の正本は 楽天証券** (= FIRE は補助記録のみ)
- **改善提案は 自動反映しない** (= 承認制、§10 は memory のみ)
- **Computer Use / Playwright / 楽天 RIT API 不採用** (= 自動発注なし)
- **最終判断と発注責任は 藤原さん本人**
- **本 D2 入力は synthetic fixture** (= 実 F062 朝 batch / 実 DATA-R3 batch ではない)
- **artifact_source = synthetic_fixture → pilot_use=skip_recommended → 判定 HOLD**

---

trade plan → [[2026-05-15_manual_live_pilot_trade_plan|2026-05-15 trade plan]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D1 (5/14) review → [[2026-05-14_manual_live_pilot_review|D1 review]]
W60-pilot-D2 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_results|W60-pilot-D2 results]]
