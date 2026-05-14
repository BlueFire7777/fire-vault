---
template: manual-live-pilot-review
version: 1.0
date: 2026-05-20
owner: BlueFire7777 (Fujiwara)
pilot_day: D5
status: blank (= 15:30 以降記入)
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_sample
pilot_use: eligible_with_minor_caveat
pilot_judgment: GO (with minor caveat、ただし実 trade は構造的 skip)
related:
  - 04_daily/2026-05-20_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D5_results.md
---

# Manual Live Pilot — Trade Review (2026-05-20 / D5)

## ✅ D5 特記: GO + minor caveat、ただし実 trade は構造的 skip

input chain: F111 --sample → F062 → AFTER-R1
= artifact_source=f062_preview / f111_input_source=f111_sample / 8 invariants PASS

ただし candidate ticker が **サンプル名** (= 1234 / 7203 サンプル...) のため、
**実 trade 構造的不可** → skip 必須。**D5 = F111 own ロジック検証 wave**。

---

## §1 基本情報

- **date**: 2026-05-20 (水 / D5)
- **記入時刻**: __:__ JST (= 15:30 以降推奨)
- **trade_plan**: `04_daily/2026-05-20_manual_live_pilot_trade_plan.md`
- **artifact_source**: f062_preview ✓
- **f062_raw_kind**: f062_actual_dict ✓
- **f111_input_source**: **f111_sample ✓** (= D3/D4 manual_seed から進展)
- **pilot_use**: eligible_with_minor_caveat
- **ticker / name**: skip (= サンプル ticker のため構造的に skip)

## §2 計画 vs 実際

(= skip のため簡素化)

| 項目 | planned | actual |
|---|---|---|
| entry | skip | skip |
| exit | skip | skip |
| PnL | 0 円 | 0 円 |

## §3 PnL

- 総 PnL: 0 円
- 本日累計: 0 円
- 週累計 (= D1+D2+D3+D4+D5): 0 円 (= 全 day skip 想定)

## §4 Reason for skip

- f111_input_source = f111_sample は F111 SAMPLE_CANDIDATES (= 仮想 ticker)
- 楽天証券 / iSPEED で 1234 / 7203 サンプル... の発注は **構造的不可**
- D5 = F111 own ロジック検証 wave、実 trade は次 wave (= F111-real-batch) 以降

## §5 Reason for exit

skip のため該当なし。

## §6 Rule followed?

| ルール | 遵守? | 備考 |
|---|---|---|
| 事前 stop_loss | ☐ skip | skip 判断 |
| 損切りずらさず | ☐ skip | |
| ナンピンなし | ☐ skip | |
| 追加買いなし | ☐ skip | |
| 15:10 close (信用) | ☐ skip | |
| 持ち越しなし | ☐ skip | |
| 自動発注なし | ☐ Yes (= 構造的) | (= 保証) |
| 楽天 / iSPEED 手動のみ | ☐ Yes (= 操作なし) | |
| 1 トレード ≤ 15,000 円 | ☐ skip | |
| 1 日 ≤ 30,000 円 | ☐ skip | |
| **artifact_source=f062_preview 認識** | ☐ Yes | (= 保証) |
| **f111_input_source=f111_sample 認識** | ☐ Yes | (= W60-F111-pre 達成) |
| **サンプル ticker → 実 trade 不可 認識** | ☐ Yes | (= D5 特記) |

## §7 Pattern matched? (= F111 sample 由来 pattern)

採用 pattern (= 1234 サンプル情報通信A / 7203 サンプル輸送機D):
- `freshness_ok_high_confidence` (= ?): ☐ matched / ☐ NA
- `manual_review_active_label` (= ?): ☐ matched / ☐ NA
- `multi_reason_basis` (= ?): ☐ matched / ☐ NA
- `low_risk_note` (= ?): ☐ matched / ☐ NA

→ サンプル ticker のため pattern matching は **形式検証のみ**、
真の pattern hit 率は f111_real_batch 化後に集計。

## §8 What worked / what failed

### §8.1 What worked

- F111 --sample → F062 → AFTER-R1 chain 完全動作 ✓
- 8 invariants 全 PASS ✓
- f111_input_source 自動推定 (= manual_seed → f111_sample 進展)
- minor caveat 化 (= D3/D4 強 caveat より進展)

### §8.2 What failed

- candidate ticker がサンプル名 (= 1234 / 7203 サンプル...) で実 trade 構造的不可
- F111 朝 batch (= staging real candidates) はまだ稼働なし → 次 wave で対応

### §8.3 Surprise

- F111 --sample が想定以上に完全な 32 fields/row output を生成 (= 多面評価)
- F062 への adapter 不要で直接 handoff 可能

### §8.4 D5 運用フロー 評価 (= 特記)

- W60-launchd-pre 5 step → D5 8 step (= +F111 step) の使い勝手: __________
- f111_input_source = f111_sample の信頼性: __________
- minor caveat の運用感覚: __________
- サンプル ticker → skip 判断の自然さ: __________
- F111-real-batch 期待 (= 次 wave): __________

## §9 Improvement

- W1 集約 (= 5/21 以降) で 変えること: __________
- W1 で 続けること: __________
- F111 朝 batch 化 (= F111-real-batch wave) への 要望: __________
- 設計 doc / template への 要望: __________

## §10 Pattern promote / suppress / watch 提案 (= D5 = 5 サンプル目)

| pattern_id | 提案 | 理由 |
|---|---|---|
| freshness_ok_high_confidence | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |
| manual_review_active_label | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |
| multi_reason_basis | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |
| low_risk_note | ☐ promote / ☐ suppress / ☐ watch / ☐ skip | __________ |

(= W60-pilot-W1 で結論)

## §11 Emergency log

該当しなければ skip。

## §12 Screenshot / 関連 path memo

- FIRE artifact: `reports/after_r1/*_2026-05-20.{json,md}`
- F111 sample: `/tmp/fire_d5_prep/f111_real_sample_2026-05-20.json`
- F062 output: `/tmp/fire_d5_prep/f062_actual_output_2026-05-20.json`
- DATA-R3: `/tmp/fire_d5_prep/data_r3_freshness_2026-05-20.json`

## §13 Next action

- ☑ **W60-pilot-W1 (= 5/21 木以降) D1-D5 集約 wave 着手**
- ☐ F111-real-batch wave 着手 (= synth 脱却の本丸)
- ☐ W60-launchd-real wave 着手 (= Wave 41/45 進捗後)
- ☐ ルール 修正提案

(= D1-D5 = 5 営業日完了、W1 集約へ)

## §14 Stage 3 昇格条件 突合 (= D5 = 5 日目 / 1 週間完了)

| 項目 | 当日 | 5 日累積 (D1-D5) |
|---|---|---|
| trade 回数 | 0 | 0 / 50 (R-19-08 目標) |
| 累積 PnL | 0 円 | 0 円 |
| ルール遵守率 | N/A (= skip) | N/A (= 0 entry) |
| ルール違反件数 | 0 | 0 |
| pattern hit 率 | N/A | N/A (= 0 entry) |
| emergency stop 発動 | 0 | 0 |
| **artifact_source f062_preview 比率** | 1/1 | **3/5** (= D3/D4/D5) |
| **f111_input_source 進展** | f111_sample | manual_seed×2 + f111_sample×1 |
| F111 朝 batch 稼働率 | 0 | 0/5 (= synth or sample のみ) |

W60-pilot-W1 で:
- artifact_source 推移評価 (= synthetic_fixture 40% → f062_preview 60% へ)
- f111_input_source 推移評価 (= 0 → manual_seed → f111_sample 進展)
- pattern promote/suppress/watch 結論
- F111-real-batch wave 必要性確認

## §15 安全 footer

- 手動運用補助 / FIRE 発注しない / 自動化なし / auto_order=False /
  manual_review=True / 楽天正本 / Computer Use 不採用 / 最終判断は藤原さん本人 /
  D5 chain = F111 --sample → F062 → AFTER-R1 = f062_preview + f111_sample /
  minor caveat / 実 trade は次 wave 以降 / W1 集約 へ

---

trade plan → [[2026-05-20_manual_live_pilot_trade_plan|D5 trade plan]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D4 review → [[2026-05-19_manual_live_pilot_review|D4 review]]
D5 manual sequence → [[template_d5_real_artifact_prep|template_d5_real_artifact_prep.md]]
W60-pilot-D5 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D5_results|W60-pilot-D5 results]]
