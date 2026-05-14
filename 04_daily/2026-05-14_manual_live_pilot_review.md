---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-14
owner: BlueFire7777 (Fujiwara)
pilot_day: D1
status: blank (= 15:30 以降 / 終日 skip 時の確認用)
related:
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/2026-05-14_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_results.md
---

# Manual Live Pilot — Trade Review (2026-05-14 / D1)

## ⚠ D1 初日特記

本日 (2026-05-14) の入力は **synthetic fixture** (= /tmp/fire_w60_smoke/、
W60-impl smoke 用 hardcoded data)。実 F062 朝 batch / 実 DATA-R3 batch ではない。

trade plan (§13 final decision) で **skip 選択した場合**:
- 「synthetic 入力なので実 trade なし、運用フロー検証 + template 記入練習のみ」
- 翌日以降、実 F062 朝 batch が稼働してから本格 D1 として再開する選択肢

trade plan で **enter 選択した場合** (= 自己責任で):
- 以下 §2 以降を 15:30 以降に記入

---

## §1 基本情報

- **date**: 2026-05-14 (木曜日 / D1)
- **記入時刻**: __:__ JST (= 15:30 以降推奨)
- **trade_plan path**: `04_daily/2026-05-14_manual_live_pilot_trade_plan.md`
- **入力 source**: synthetic_fixture (= D1 初日特記)
- **ticker / name**: ____ / __________ (= skip の場合は "skip")
- **label**: __________ (= 例: 🟢 積極的買い推奨)

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

(= skip の場合は全行 "skip" で OK)

## §3 PnL

- **総 PnL**: ______ 円 (= 税・手数料込み)
- **(参考) entry vs exit 値幅**: ____ 円 × ___ 株 = ______ 円
- **(参考) 楽天証券での確認**: ☐ 確認済 / ☐ 未確認 / ☐ skip 該当なし
- **本日累計 PnL**: ______ 円
- **週累計 PnL** (= D1 = 当日値): ______ 円
- **1 日上限到達?**: ☐ No / ☐ Yes → 残りの日 trade 停止 / ☐ skip 該当なし

## §4 Reason for entry (= 何を見て買ったか)

- FIRE の why_selected: __________
- 自分の追加判断: __________
- 採用 pattern: __________
- 寄付きで見た指標: __________
- 後から見て妥当だった点: __________

(= skip の場合は "skip — synthetic fixture 段階のため見送り" 等)

## §5 Reason for exit

- exit trigger: ☐ stop_loss / ☐ take_profit / ☐ 15:10 cutoff / ☐ 手動判断 /
  ☐ その他 / ☐ skip 該当なし
- 判断理由: __________
- 後から見て妥当だった点: __________

## §6 Rule followed?

| ルール | 遵守? | 備考 |
|---|---|---|
| 事前に stop_loss を決めた | ☐ Yes / ☐ No / ☐ skip | __________ |
| 損切りを後ろにずらさなかった | ☐ Yes / ☐ No / ☐ skip | __________ |
| ナンピンしなかった | ☐ Yes / ☐ No / ☐ skip | __________ |
| 追加買いしなかった | ☐ Yes / ☐ No / ☐ skip | __________ |
| 15:10 までに手動 close した (信用) | ☐ Yes / ☐ No / ☐ skip | __________ |
| 持ち越ししなかった | ☐ Yes / ☐ No / ☐ skip | __________ |
| 自動発注を使わなかった | ☐ Yes (= FIRE は発注しない) | (= 構造的に保証) |
| 楽天 / iSPEED 手動操作のみ | ☐ Yes / ☐ No / ☐ skip | __________ |
| 1 トレード最大損失 ≤ 15,000 円 | ☐ Yes / ☐ No / ☐ skip | __________ |
| 1 日最大損失 ≤ 30,000 円 | ☐ Yes / ☐ No / ☐ skip | __________ |
| **入力が synthetic だと認識** | ☐ Yes (= D1 特記事項) | (= 構造的に保証) |

ルール違反が 1 件でも `No` → §11 emergency log 検討。

## §7 Pattern matched? (= pattern_candidate_report との突合)

採用 pattern (= trade_plan §6.2 から):

- `freshness_ok_high_confidence` (= 7203, 6758): ☐ matched / ☐ not / ☐ NA / ☐ skip
- `manual_review_active_label` (= 7203, 6758, 9984): ☐ matched / ☐ not / ☐ NA / ☐ skip
- `multi_reason_basis` (= 7203, 6758): ☐ matched / ☐ not / ☐ NA / ☐ skip
- `low_risk_note` (= 7203, 6758, 9984): ☐ matched / ☐ not / ☐ NA / ☐ skip

**結果との相関 (= 主観)**:
- 良い結果に寄与した pattern: __________
- 良い結果に寄与しなかった pattern: __________
- 誤シグナル だった pattern: __________

(= skip の場合は "synthetic fixture 段階のため real pattern check 不可")

## §8 What worked / what failed

### §8.1 What worked

- __________
- __________
- __________

### §8.2 What failed

- __________
- __________
- __________

### §8.3 Surprise

- __________
- __________

### §8.4 D1 運用フロー 評価 (= 初日 特記)

- claude code の朝 batch (= manual run) は実用的だったか: __________
- artifact (= 4 種類) の見やすさ: __________
- trade plan template の使いやすさ: __________
- review template の記入時間: __________
- 翌日改善したい点: __________

## §9 Improvement

- 翌日 同様の場面で **変えること**: __________
- 翌日 同様の場面で **続けること**: __________
- FIRE 候補生成への **要望**: __________
- 設計 doc (operation_plan / templates) への **要望**: __________

## §10 Pattern promote / suppress / watch 提案

| pattern_id | 提案 | 理由 |
|---|---|---|
| freshness_ok_high_confidence | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |
| manual_review_active_label | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |
| multi_reason_basis | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |
| low_risk_note | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |

(= D1 = 1 サンプルのため、5 営業日後 W1 集約での判断推奨)

## §11 Emergency log (= 該当時のみ記入)

該当条件:
- forbidden_check failed / auto_order_allowed True / 2 連敗 / 1 日上限到達 /
  system 矛盾 / 想定外 ticker / 発注ミス / 15:10 close 困難 / 監視不能

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
- FIRE artifact path: `reports/after_r1/*_2026-05-14.{json,md}`
- 当日 TDnet 開示: __________

## §13 Next action (= W60-integration で日付/曜日修正)

- ☐ D2 (= 2026-05-15 金) も pilot 継続
- ☐ 翌日 ロット半減
- ☐ 翌日 stop (= §11 該当時)
- ☐ ルール 修正提案 (= 詳細 §9)
- ☐ Pilot 1 週間目 review (= D5 / 5 営業日目 = 2026-05-20 水)
- ☑ **synthetic fixture から実 F062 朝 batch 連携 wave** (= W60-integration) 着手済
  → D2 以降は artifact_source 区別可能

(= D1 〜 D5 営業日 path: D1=5/14 木 / D2=5/15 金 / skip 5/16 土 5/17 日 /
D3=5/18 月 / D4=5/19 火 / D5=5/20 水)

## §14 Stage 3 昇格条件との 突合 (= D1 = 1 日目)

| 項目 | 当日値 | 5 日累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 (R-19-08 目標) |
| 累積 PnL | ______ 円 | ______ 円 |
| ルール遵守率 | __% | __% (目標 ≥ 90%) |
| ルール違反件数 | __ | __ |
| pattern hit 率 | __% (= D1 = サンプル不足) | __% |
| emergency stop 発動 | __ | __ |

## §15 安全 footer

- **本 review は学習記録** (= 投資助言ではない、**手動運用補助** のみ)
- **FIRE は何も発注しない、何も自動化しない**
- **`auto_order_allowed = False` / `manual_review_required = True`** (= MVP 不変量、本 review 対象 trade も該当)
- **建玉の正本は 楽天証券** (= FIRE は補助記録のみ)
- **改善提案は 自動反映しない** (= 承認制、§10 は memory のみ)
- **Computer Use / Playwright / 楽天 RIT API 不採用** (= 自動発注なし)
- **最終判断と発注責任は 藤原さん本人**
- **本 D1 入力は synthetic fixture** (= 実 F062 朝 batch / 実 DATA-R3 batch ではない)

---

trade plan → [[2026-05-14_manual_live_pilot_trade_plan|2026-05-14 trade plan]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
W60-pilot-D1 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_results|W60-pilot-D1 results]]
