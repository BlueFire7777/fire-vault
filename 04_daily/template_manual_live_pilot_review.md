---
template: manual-live-pilot-review
version: 1.0
date: ____-__-__
owner: BlueFire7777 (Fujiwara)
related:
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/template_manual_live_pilot_trade_plan.md
---

# Manual Live Pilot — Trade Review (= 15:30 以降 記入)

## §1 基本情報

- **date**: ____-__-__
- **記入時刻**: __:__ JST (= 15:30 以降推奨)
- **trade_plan path**: `04_daily/<YYYY-MM-DD>_manual_live_pilot_trade_plan.md`
- **ticker / name**: ____ / __________
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

## §3 PnL

- **総 PnL**: ______ 円 (= 税・手数料込みの実損益)
- **(参考) entry vs exit 値幅**: ____ 円 × ___ 株 = ______ 円
- **(参考) 楽天証券での確認**: ☐ 確認済 / ☐ 未確認 (= 建玉正本は楽天証券)
- **本日累計 PnL**: ______ 円
- **週累計 PnL**: ______ 円
- **1 日上限到達?**: ☐ No / ☐ Yes → 残りの日 trade 停止

## §4 Reason for entry (= 何を見て買ったか)

- FIRE の why_selected: __________
- 自分の追加判断: __________
- 採用 pattern: __________
- 寄付きで見た指標: __________
- 後から見て妥当だった点: __________

## §5 Reason for exit

- exit trigger: ☐ stop_loss / ☐ take_profit / ☐ 15:10 cutoff / ☐ 手動判断 / ☐ その他: __________
- 判断理由: __________
- 後から見て妥当だった点: __________

## §6 Rule followed?

| ルール | 遵守? | 備考 |
|---|---|---|
| 事前に stop_loss を決めた | ☐ Yes / ☐ No | __________ |
| 損切りを後ろにずらさなかった | ☐ Yes / ☐ No | __________ |
| ナンピンしなかった | ☐ Yes / ☐ No | __________ |
| 追加買いしなかった | ☐ Yes / ☐ No | __________ |
| 15:10 までに手動 close した (信用) | ☐ Yes / ☐ No / ☐ NA | __________ |
| 持ち越ししなかった | ☐ Yes / ☐ No / ☐ NA | __________ |
| 自動発注を使わなかった | ☐ Yes / ☐ No | __________ |
| 楽天 / iSPEED 手動操作のみ | ☐ Yes / ☐ No | __________ |
| 1 トレード最大損失 ≤ 15,000 円 | ☐ Yes / ☐ No | __________ |
| 1 日最大損失 ≤ 30,000 円 | ☐ Yes / ☐ No | __________ |

ルール違反が 1 件でも `No` → §11 emergency log 検討。

## §7 Pattern matched? (= pattern_candidate_report との突合)

- 採用 pattern (= trade_plan §5.2):
  - `freshness_ok_high_confidence`: ☐ matched / ☐ not / ☐ NA
  - `manual_review_active_label`: ☐ matched / ☐ not / ☐ NA
  - `multi_reason_basis`: ☐ matched / ☐ not / ☐ NA
  - `low_risk_note`: ☐ matched / ☐ not / ☐ NA

- **結果との相関 (= 主観)**:
  - 良い結果に **寄与した** pattern: __________
  - 良い結果に **寄与しなかった** pattern: __________
  - **誤シグナル** だった pattern: __________

## §8 What worked / what failed

### §8.1 What worked (= 続けたいこと)

- __________
- __________
- __________

### §8.2 What failed (= 改善したいこと)

- __________
- __________
- __________

### §8.3 Surprise (= 想定外、後で別 wave で検討)

- __________
- __________

## §9 Improvement (= 翌日に活かす)

- 翌日 同様の場面で **変えること**: __________
- 翌日 同様の場面で **続けること**: __________
- FIRE 候補生成への **要望**: __________

## §10 Pattern promote / suppress / watch 提案

→ pattern_candidate_report の各 pattern を以下に分類 (= 手動分類):

| pattern_id | 提案 | 理由 |
|---|---|---|
| __________ | ☐ promote / ☐ suppress / ☐ watch | __________ |
| __________ | ☐ promote / ☐ suppress / ☐ watch | __________ |
| __________ | ☐ promote / ☐ suppress / ☐ watch | __________ |

(= この分類は **memory 蓄積のみ**、 自動反映はしない、自動反映は次 wave 検討)

## §11 Emergency log (= 該当時のみ記入)

該当しなければ skip。該当条件:
- forbidden_check failed / auto_order_allowed True / 2 連敗 / 1 日上限到達 /
  system 矛盾 / 想定外 ticker / 発注ミス / 15:10 close 困難 / 監視不能

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
- FIRE artifact path: __________
- 当日 TDnet 開示: __________

## §13 Next action

- ☐ 翌日も pilot 継続
- ☐ 翌日 ロット半減
- ☐ 翌日 stop (= §11 該当時)
- ☐ ルール 修正提案 (= 詳細 §9)
- ☐ Pilot 1 週間目 review (= 5 営業日目に集約)

## §14 Stage 3 昇格条件との 突合 (= 5 営業日後 集約用)

5 営業日後の集約レビューで以下を check:

| 項目 | 当日値 | 5 日累積 |
|---|---|---|
| trade 回数 | __ | __ / 50 (R-19-08 目標) |
| 累積 PnL | ______ 円 | ______ 円 |
| ルール遵守率 | __% | __% (目標 ≥ 90%) |
| ルール違反件数 | __ | __ |
| pattern hit 率 | __% | __% |
| emergency stop 発動 | __ | __ |

## §15 安全 footer

- **本 review は学習記録** (= 投資助言ではない、**手動運用補助** のみ)
- **FIRE は何も発注しない、何も自動化しない**
- **`auto_order_allowed = False` / `manual_review_required = True`** (= MVP 不変量、本 review 対象 trade も該当)
- **建玉の正本は 楽天証券** (= FIRE は補助記録のみ)
- **改善提案は 自動反映しない** (= 承認制、§10 は memory のみ)
- **Computer Use / Playwright / 楽天 RIT API 不採用** (= 自動発注なし)
- **最終判断と発注責任は 藤原さん本人**

---

trade_plan template → [[template_manual_live_pilot_trade_plan|template_manual_live_pilot_trade_plan.md]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
