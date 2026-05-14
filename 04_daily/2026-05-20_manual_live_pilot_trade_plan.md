---
template: manual-live-pilot-trade-plan
version: 1.0
date: 2026-05-20
owner: BlueFire7777 (Fujiwara)
pilot_day: D5
input_chain: F111 runner --sample → F062 runner --dry-run → AFTER-R1 MVP
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_sample
data_r3_freshness_verdict: OK (= 選択肢 A dry-run runner)
pilot_use: eligible_with_minor_caveat
pilot_judgment: GO (with minor caveat = F111 sample / staging real batch ではない)
related:
  - 04_daily/2026-05-19_manual_live_pilot_trade_plan.md (= D4)
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D5_results.md
  - 04_daily/template_d5_real_artifact_prep.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_F111_PRE_results.md
---

# Manual Live Pilot — Trade Plan (2026-05-20 / D5)

## ✅ D5 判定: GO 候補 (= 8 invariants 全 PASS、minor caveat 付き)

**hard check 全 PASS** ✓:
- artifact_source = `f062_preview` ✓
- f062_raw_kind = `f062_actual_dict` ✓
- **f111_input_source = `f111_sample`** ✓ (= D3/D4 `manual_seed` から進展)
- DATA-R3 freshness verdict = `OK` ✓
- forbidden_check.passed = True ✓
- auto_order_allowed = False ✓
- manual_review_required = True ✓
- safety_flags 13 keys 全 False ✓

### ✅ D3/D4 → D5 caveat 強度比較

| Day | f111_input_source | caveat |
|---|---|---|
| D3 (5/18) | manual_seed (= 藤原さん手作り 3 銘柄) | **強 caveat** |
| D4 (5/19) | manual_seed (= D3 と同) | **強 caveat** |
| **D5 (5/20)** | **f111_sample (= F111 runner own ロジック、SAMPLE_CANDIDATES)** | **中 caveat ★** |
| 将来 | f111_real_batch (= staging real candidates) | 最弱 caveat |

D5 caveat 内容: F111 SAMPLE_CANDIDATES は **F111 own logic 経由** (= research_advisory
版本管理、F119 score 連携、market_regime / sector_flow など多面的 evaluation 込み)。
ただし **staging DB の実 candidate ではない** ため caveat 残る。

---

## §1 基本情報

- **date**: 2026-05-20 (水曜日 / D5)
- **記入時刻**: claude code が 2026-05-14 02:50 JST に事前生成
- **本日の本業 / 監視可能時間**: ☐ 9:00-15:30 監視可 / ☐ 部分監視 / ☐ 監視不能
- **15:10 までに手動 close 可能か**: ☐ Yes / ☐ No
- **大型イベント** (FOMC / 雇用統計 / 重要 IR / 決算跨ぎ): ☐ なし / ☐ あり

## §2 市場コンディション (= 寄付き前 藤原さん記入)

- 日経平均 寄付き前: __,___
- TOPIX 寄付き前: ____
- 米国 引け 5/19 (月) JST: NYダウ / S&P / NASDAQ
- ドル円: ___.__ 円
- 自分の地合い judgement: ☐ 強気 / ☐ 中立 / ☐ 弱気

## §3 FIRE report paths (= 5/20 当日 生成済 ✓)

- `reports/after_r1/morning_line_material_2026-05-20.md` ✓
- `reports/after_r1/good_candidate_ranking_2026-05-20.md` ✓
- `reports/after_r1/pattern_candidate_report_2026-05-20.md` ✓
- `reports/after_r1/paper_live_ledger_2026-05-20.md` ✓

### §3.0 artifact_source / chain 詳細

- **artifact_source**: ☑ **`f062_preview`** / ☐ `synthetic_fixture` / ☐ `unknown`
- **f062_raw_kind**: ☑ **`f062_actual_dict`** / ☐ `plain_list` / ☐ `other_dict` / ☐ `none`
- **f111_input_source**: ☑ **`f111_sample`** / ☐ `manual_seed` / ☐ `f111_real_batch` / ☐ `unknown`
- **data_r3_freshness_verdict**: ☑ **`OK`** (= 選択肢 A dry-run runner)
- **pilot_use**: ☑ **`eligible_with_minor_caveat`** / ☐ `eligible_with_caveat` / ☐ `skip_recommended` / ☐ `no_go`
- **pilot_judgment**: ☑ **GO** (= 8 invariants PASS + minor caveat) / ☐ HOLD / ☐ NO-GO

### §3.0.1 D5 input chain (= W60-F111-pre 確立)

```
F111 runner --sample (= /tmp/fire_d5_prep/f111_real_sample_2026-05-20.json)
  = input=6 / candidate_count=6 / with_advisory=5 / missing_advisory=1
  = manual_review_required_count=6 / auto_order_allowed_true_count=0 / DB write 0
  ↓ --preview-json
F062 LINE preview runner (= /tmp/fire_d5_prep/f062_actual_output_2026-05-20.json)
  = freshness gate OK / input=6 / selected=4 / chunks=1
  = forbidden=0 / safety_footer=True / line_send_count=0
  ↓ --f062-preview-json
AFTER-R1 MVP (= reports/after_r1/*_2026-05-20.{json,md})
  = artifact_source=f062_preview / f062_raw_kind=f062_actual_dict
  = f111_input_source=f111_sample / ranking_size=4
```

DATA-R3 freshness:
- `/tmp/fire_d5_prep/data_r3_freshness_2026-05-20.json` (= 選択肢 A dry-run runner)
- verdict=OK / aggregate_exit_code=0 / 4 sub-runner 全 ok

### §3.1 共通 GO-check (= 全 PASS ✓)

- [x] morning_line_material `forbidden_phrases_check.passed = True` ✓
- [x] paper_live_ledger 全 entry で `auto_order_allowed = False` ✓
- [x] paper_live_ledger 全 entry で `manual_review_required = True` ✓
- [x] DATA-R3 freshness verdict = `OK` ✓
- [x] ranking と morning_line_material の top ticker 整合 ✓
- [x] 出力 path = `reports/after_r1/` 配下 ✓
- [x] **artifact_source = `f062_preview`** ✓
- [x] **f111_input_source = `f111_sample`** ✓ (= W60-F111-pre 達成)

## §4 Top candidates (= F111 runner --sample 由来)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | **1234** | **サンプル情報通信A** | 条件付き買い推奨 | 🟡 | 0.82 | 136.0 |
| 2 | **7203** | **サンプル輸送機D** | 条件付き買い推奨 | 🟡 | 0.78 | 129.0 |

(= D3/D4 と異なる candidates、F111 own ロジック由来)

### §4.1 F111 sample candidates の特性

- F111 SAMPLE_CANDIDATES の 6 銘柄から advisory_label / 評価で 4 件 selected
- top 2 は両方 `boost_with_caution` (= 条件付き買い推奨 🟡)
- D3/D4 の boost / boost_with_caution と比べると **やや慎重な label set**
- F111 own ロジック (= research_final_score / f119_expected_h20_return) 由来で、
  単純な手作り synth より **多面評価**

## §5 D5 推奨選択肢

### §5.1 推奨

**候補 A**: 🟡 1234 サンプル情報通信A score=136.0 conf=0.82
- 高 conf (= 0.82)、F119 expected_h20_return ≈ +11.42%、win_rate ≈ 88%
- research_advisory_version: F111-R1-v1 / sector_17: 情報通信 / 5月 boost
- ただし「サンプル」名 = staging real 候補ではない (= **minor caveat**)

**候補 B**: 🟡 7203 サンプル輸送機D score=129.0 conf=0.78
- 中 conf、boost_with_caution
- 同様に「サンプル」名

### §5.2 D5 判定 (= GO 候補 + minor caveat)

8 invariants 全 PASS。F111 own ロジック経由のため D3/D4 強 caveat より **1 段強化**。

ただし候補が **サンプル名 (= 1234 / 7203 サンプル...)** であり、staging real な
ticker ではない。実 trade には適さない (= 楽天で発注しようとしても銘柄が存在しない)。

→ **D5 推奨**: **形式検証 OK / 実 trade は skip 必須**

### §5.3 D3 → D4 → D5 caveat 進展のまとめ

| Day | f111_input_source | candidate 内容 | 実 trade 可? |
|---|---|---|---|
| D3 | manual_seed | 6920/4063/7203 (= 実在 ticker、手作り材料) | リスク上限内なら可、ただし材料手作り |
| D4 | manual_seed | D3 と同 | 同上 |
| **D5** | **f111_sample** | 1234 / 7203 サンプル... (= 仮想 ticker) | **不可** (= ticker 存在しない) |

D5 は **F111 own ロジック検証 wave** であり、実 trade は構造的に不可。
完全 `f111_real_batch` 化で初めて実 trade 可。

## §6 Selected ticker (= 藤原さん が確認 / skip)

- **ticker**: ____ (= 推奨: **skip** — サンプル ticker 不在のため)
- **name**: __________
- **label**: __________
- **rank**: #__

### §6.1 Why skipped

- f111_sample 由来の candidate ticker は サンプル名 (= 仮想)
- 実 trade 不可 (= 楽天 / iSPEED で発注しても銘柄存在しない)
- **形式検証としては成功** (= 8 invariants PASS、F111 own ロジック動作確認)
- 次 wave (= F111-real-batch、staging real candidates) で初めて実 trade 候補

## §7-§10 Entry plan / SL / TP / no-trade conditions (= skip 推奨のため簡素化)

skip 選択時は §7-§10 全 skip 可。

## §11 Manual buy checklist (= skip 選択時)

- [x] §3.1 共通 GO-check 全 PASS ✓
- [x] §3.0 artifact_source = f062_preview / f111_input_source = f111_sample ✓
- [x] §5.3 candidate ticker が **サンプル名** (= 実在しない) と認識
- [x] **本日 skip 推奨** (= 実 trade 不可、構造的)
- [x] auto_order_allowed=False / manual_review_required=True ✓
- [x] 楽天証券 / iSPEED 操作 0 (= skip)
- [x] **D5 = F111 own ロジック検証完了 wave 認識**

## §12 D5 リスク上限

skip 選択につき適用なし。

## §13 Final decision

☐ **enter** (= サンプル ticker 発注は **構造的に不可**、選択肢にならない)

☐ **watch** (= 形式検証として skip と同等)

☑ **skip** (= D5 = F111 own ロジック検証 wave、実 trade は次 wave 以降)

### §13.1 skip の場合 (= 標準推奨)

- skip 理由: F111 SAMPLE_CANDIDATES は仮想 ticker、実 trade 不可
- D5 wave 達成: 8 invariants 全 PASS / f111_sample 自動推定 / minor caveat 化 ✓
- W1 集約 (= W60-pilot-W1、5/21 木以降) へ移行

## §14 W1 集約への引き継ぎ

D1-D5 path 完了:

| Day | 日付 | artifact_source | f111_input_source | judgment | 実 entry |
|---|---|---|---|---|---|
| D1 | 5/14 木 | synthetic_fixture | (= MVP 直渡し) | HOLD | skip 想定 |
| D2 | 5/15 金 | synthetic_fixture | (= MVP 直渡し) | HOLD | skip 想定 |
| D3 | 5/18 月 | f062_preview | manual_seed | GO + 強 caveat | skip 想定 (値嵩) |
| D4 | 5/19 火 | f062_preview | manual_seed | GO + 強 caveat | skip 想定 (連続同) |
| **D5** | **5/20 水** | **f062_preview** | **f111_sample** ★ | **GO + 中 caveat** | **skip 必須 (= 仮想 ticker)** |

W1 集約で見るべき観点:
1. artifact_source 推移 (= 2/5 synthetic_fixture → 3/5 f062_preview)
2. f111_input_source 推移 (= 0/5 manual_seed/f111_sample → ... 進展)
3. 実 entry 0/5 (= 全 day で entry なし、運用フロー検証のみ)
4. skip/watch 判断理由の質
5. 値嵩株リスク (= D3/D4 100 株単位超過)
6. pattern promote/suppress/watch 候補
7. F111 朝 batch 化への次 wave 計画 (= F111-real-batch wave)

## §15 安全 footer

- 手動運用補助 / FIRE 発注しない / iSPEED 手動 / Computer Use 不採用 /
  最終判断は藤原さん本人 / D5 chain = F111 --sample → F062 → AFTER-R1 =
  artifact_source=f062_preview + f111_input_source=f111_sample / 8 invariants
  PASS / minor caveat / 実 trade は次 wave 以降

---

review → [[2026-05-20_manual_live_pilot_review|D5 review (blank)]]
operation plan → [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
D4 trade plan → [[2026-05-19_manual_live_pilot_trade_plan|D4]]
D5 manual sequence → [[template_d5_real_artifact_prep|template_d5_real_artifact_prep.md]]
W60-pilot-D5 results → [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D5_results|W60-pilot-D5 results]]
