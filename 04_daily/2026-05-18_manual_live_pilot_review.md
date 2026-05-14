---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-18
owner: BlueFire7777 (Fujiwara)
pilot_day: D3
status: blank (= 15:30 以降記入)
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
pilot_use: eligible_with_caveat
pilot_judgment: GO (with F111 synth caveat)
related:
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/2026-05-18_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D3_results.md
---

# Manual Live Pilot — Trade Review (2026-05-18 / D3)

## ✅ D3 特記: GO 候補 (= artifact_source=f062_preview)、F111 caveat 付き

本日 (2026-05-18 月) の input chain:
```
F111 advisory preview (= 手作り synth 3 銘柄)
  → F062 runner --dry-run --require-freshness-ok
  → AFTER-R1 MVP --mode mvp --task all
  → reports/after_r1/ に artifact_source=f062_preview の 4 成果物生成
```

trade plan の判定: **GO 候補** (= 7 invariants 全 PASS、F111 synth caveat 付き)。
藤原さんが §13 final decision で:
- **enter**: 自己責任で 1 銘柄少額 entry (= 値嵩株注意)
- **watch**: 場中観察のみ
- **skip**: F111 caveat / 値嵩株リスク / その他理由で見送り

---

## §1 基本情報

- **date**: 2026-05-18 (月曜日 / D3)
- **記入時刻**: __:__ JST (= 15:30 以降推奨)
- **trade_plan path**: `04_daily/2026-05-18_manual_live_pilot_trade_plan.md`
- **artifact_source**: f062_preview ✓
- **f062_raw_kind**: f062_actual_dict ✓
- **pilot_use**: eligible_with_caveat (= F111 synth)
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

## §3 PnL

- **総 PnL**: ______ 円
- **本日累計 PnL**: ______ 円
- **週累計 PnL** (= D1+D2+D3): ______ 円
- **1 日上限到達?**: ☐ No / ☐ Yes / ☐ skip 該当なし
- **連敗?**: ☐ No / ☐ Yes (= 当日 2 銘柄 entry した場合のみ該当)

## §4 Reason for entry / skip

- FIRE の why_selected: __________
- 自分の追加判断 (= F111 caveat 考慮): __________
- 採用 pattern: __________
- 寄付きで見た指標: __________
- 後から見て妥当だった点: __________
- 半導体セクター動向 (= 6920 entry の場合): __________

skip 場合の標準理由: "F111 input synth caveat / 値嵩株リスク / 監視不能 / その他"

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
| **artifact_source = f062_preview を認識** | ☐ Yes (= D3 特記) | (= 保証) |
| **F111 input synth caveat を認識** | ☐ Yes (= D3 特記) | (= 保証) |

## §7 Pattern matched? (= pattern_candidate_report との突合)

採用 pattern (= 6920 レーザーテック / 4063 信越化学):
- `freshness_ok_high_confidence` (= 6920 のみ): ☐ matched / ☐ not / ☐ NA / ☐ skip
- `manual_review_active_label` (= 6920, 4063): ☐ matched / ☐ not / ☐ NA / ☐ skip
- `multi_reason_basis` (= 6920 のみ): ☐ matched / ☐ not / ☐ NA / ☐ skip
- `low_risk_note` (= 6920, 4063): ☐ matched / ☐ not / ☐ NA / ☐ skip

結果との相関 (= 主観):
- 良い結果に寄与した pattern: __________
- 良い結果に寄与しなかった pattern: __________
- 誤シグナル だった pattern: __________

## §8 What worked / what failed

### §8.1 What worked

- __________
- __________

### §8.2 What failed

- __________
- __________

### §8.3 Surprise

- __________

### §8.4 D3 運用フロー 評価 (= 特記)

- W60-launchd-pre 4 step manual sequence の使い勝手: __________
- artifact_source = f062_preview の信頼性 (= F111 synth ながら): __________
- 値嵩株 (= 6920 30,000 円帯) のロット計算: __________
- F062 runner --dry-run --require-freshness-ok の動作: __________
- DATA-R3 dry-run runner (= 選択肢 A) の動作: __________
- hard check 7 invariants の網羅性: __________
- D2 と比較した GO 候補の質感: __________

## §9 Improvement

- 翌日 D4 (= 2026-05-19 火) で **変えること**: __________
- D4 で **続けること**: __________
- FIRE 候補生成への **要望**: __________
- 設計 doc / template への **要望**: __________
- F111 朝 batch 化への **期待**: __________

## §10 Pattern promote / suppress / watch 提案 (= D3 = 3 サンプル目)

| pattern_id | 提案 | 理由 |
|---|---|---|
| freshness_ok_high_confidence | ☐ promote / ☐ suppress / ☐ watch / ☐ skip (D3) | __________ |
| manual_review_active_label | ☐ promote / ☐ suppress / ☐ watch / ☐ skip (D3) | __________ |
| multi_reason_basis | ☐ promote / ☐ suppress / ☐ watch / ☐ skip (D3) | __________ |
| low_risk_note | ☐ promote / ☐ suppress / ☐ watch / ☐ skip (D3) | __________ |

(= D5 W1 集約 (= W60-pilot-W1) で結論)

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
- FIRE artifact path: `reports/after_r1/*_2026-05-18.{json,md}`
- F062 actual output: `/tmp/fire_d3_prep/f062_actual_output_2026-05-18.json`
- DATA-R3 freshness: `/tmp/fire_d3_prep/data_r3_freshness_2026-05-18.json`
- 当日 TDnet 開示 (= 6920 / 4063 関連): __________

## §13 Next action

- ☐ D4 (= 2026-05-19 火) も pilot 継続 (= F111 朝 batch 状況によって GO 候補)
- ☐ D4 ロット半減
- ☐ D4 stop (= §11 該当時)
- ☐ ルール 修正提案 (= 詳細 §9)
- ☐ Pilot 1 週間目 review (= D5 / 5 営業日目 = 2026-05-20 水)
- ☐ **F062 朝 batch 稼働確認**:
  - W60-launchd-real wave 設計
  - Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY 進捗
  - Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY 進捗
- ☐ F111 朝 batch 設計 (= synth 脱却の本丸)

## §14 Stage 3 昇格条件との 突合 (= D3 = 3 日目)

| 項目 | 当日値 | 5 日累積 (D1+D2+D3 = 0 trade) |
|---|---|---|
| trade 回数 | __ | __ / 50 (R-19-08 目標) |
| 累積 PnL | ______ 円 | ______ 円 |
| ルール遵守率 | __% | __% (目標 ≥ 90%) |
| ルール違反件数 | __ | __ |
| pattern hit 率 | __% (= サンプル不足) | __% |
| emergency stop 発動 | __ | __ |
| artifact_source f062_preview 比率 | 1/3 (= D3 ✓ / D1 D2 synthetic) | __ / 5 |
| F111 朝 batch 稼働率 | 0/3 (= 全て手作り synth) | __ / 5 |

## §15 安全 footer

- **本 review は学習記録** (= 投資助言ではない、**手動運用補助** のみ)
- **FIRE は何も発注しない、何も自動化しない**
- **`auto_order_allowed = False` / `manual_review_required = True`** (= MVP 不変量)
- **建玉の正本は 楽天証券** (= FIRE は補助記録のみ)
- **改善提案は 自動反映しない** (= 承認制、§10 は memory のみ)
- **Computer Use / Playwright / 楽天 RIT API 不採用** (= 自動発注なし)
- **最終判断と発注責任は 藤原さん本人**
- **本 D3 input chain は F111 synth → F062 → AFTER-R1**
  (= artifact_source=f062_preview だが F111 朝 batch 未稼働、手作り synth caveat 付き)

---

trade plan → [[2026-05-18_manual_live_pilot_trade_plan|2026-05-18 trade plan]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D2 (5/15) review → [[2026-05-15_manual_live_pilot_review|D2 review]]
D3 manual command sequence → [[template_d3_real_artifact_prep|template_d3_real_artifact_prep.md]]
W60-pilot-D3 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D3_results|W60-pilot-D3 results]]
