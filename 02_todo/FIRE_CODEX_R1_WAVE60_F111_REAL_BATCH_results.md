---
id: FIRE-CODEX-R1-WAVE60-F111-real-batch-results
phase: 本番 v0 中核 / Wave 60-F111-real-batch / 実 ticker 候補生成
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_plan.md
  - 03_design/F111_real_batch_requirements_2026-05-14.md (= W1 集約で要件確定)
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W1_results.md
---

# Wave 60-F111-real-batch Results — F111 Real Batch Candidate Generation v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 F111_REAL_BATCH_MARKERS 自動推定 + sample exclusion + risk filter + 9th
invariant 強制 完全実装 ✓ — safe fixture chain で実 ticker 候補が top に出る
状態を確立**

D1-D5 で残った主要 blocker (= sample ticker / 値嵩株 100 株 上限超過 /
manual_seed caveat) を **AFTER-R1 MVP の filter logic** で解消。

実 chain smoke (= 4 候補):
- 信越化学 (4063 / 実在 / risk 12,000 円) → **top 採用 ✓**
- 三井住友FG (8316 / 実在 / risk 7,500 円) → **top 採用 ✓**
- レーザーテック (6920 / 実在だが risk 150,000 円 超過) → **top 抑制 ✓**
- サンプル情報通信A (1234 / サンプル) → **top 抑制 ✓**

→ **f111_input_source = f111_real_batch** 自動推定、exclusions_summary に
sample 1 / risk_above 1 を集計。

staging real SELECT 実装は本 wave スコープ外 (= 別 wave へ持ち越し)。
本 wave は **AFTER-R1 側 filter + safe fixture 実証** で完結。

## §2 実装内容

### §2.1 新規定数 (= _after_r1_mvp.py)

```python
F111_REAL_BATCH_MARKERS = (
    "tradable_universe", "estimated_100_share_risk",
    "risk_within_pilot_limit", "realtime_market_cap",
)  # 4 keys、≥3 hit で f111_real_batch 推定

SAMPLE_TICKER_NAME_PREFIXES = (
    "サンプル", "_template_", "サンプル銘柄", "TEST_", "DUMMY_",
)

PILOT_PER_TRADE_RISK_LIMIT_YEN = 15000  # operation_plan §5 / R-39-02 整合
```

### §2.2 推定 logic 拡張 (`infer_f111_input_source`)

5 step priority (= W60-F111-pre から拡張):
1. explicit 指定 (= `--f111-input-source`) が VALID なら最優先
2. row[0].f111_input_source field VALID → 採用
3. row が **F111_REAL_BATCH_MARKERS ≥ 3 件** → `f111_real_batch` ← **NEW**
4. row が F111_SAMPLE_OUTPUT_MARKERS ≥ 3 件 → `f111_sample`
5. それ以外 → `manual_seed` / rows 空 → `unknown`

### §2.3 helper 関数

| 関数 | 役割 |
|---|---|
| `_row_has_sample_ticker(row)` | name prefix / all-same-digit code 検出 |
| `_row_tradable_universe(row)` | field 不在は True (= 後方互換) |
| `_row_risk_within_pilot_limit(row)` | explicit field → estimated_100_share_risk 自動計算 → None |

### §2.4 score penalty 拡張 (`_score_row`)

| 条件 | score 影響 | why_selected 注記 |
|---|---|---|
| sample ticker 検出 | **-100.0** | ⚠ サンプル ticker (= 実 trade 不可) |
| tradable_universe=False | **-100.0** | ⚠ 取引不可 |
| risk_within_pilot_limit=False | **-30.0** | ⚠ 100 株 risk が pilot 上限超過 |
| risk_within_pilot_limit=True | **+5.0** | ✓ risk 上限内 |

### §2.5 ranking output 拡張

各 row に追加 field:
- `tradable_universe`
- `risk_within_pilot_limit`
- `estimated_100_share_risk`
- `sample_ticker_detected`

`risk_notes` に exclusion 理由を追記。

### §2.6 9th invariant 強制 (= morning_line_material top filter)

`build_morning_line_material()` で 3 種 excluded 候補を **top から強制除外**:
- `sample_ticker_detected = True`
- `tradable_universe = False`
- `risk_within_pilot_limit = False`

material に新 field:
- `exclusions_summary = {sample_ticker, tradable_universe_false, risk_above_pilot_limit}`
- `pilot_per_trade_risk_limit_yen = 15000`

### §2.7 version bump

- MVP_LOGIC_VERSION: 1.2.0 → **1.3.0**

## §3 chain smoke 結果

### §3.1 fixture (= 4 候補で全 path 検証)

| ticker | name | risk | tradable | 期待 |
|---|---|---|---|---|
| 4063 | 信越化学 | 12,000 円 | True | top OK |
| 6920 | レーザーテック | 150,000 円 | True | top 抑制 (= risk_above) |
| 1234 | サンプル情報通信A | 8,000 円 | True | top 抑制 (= sample) |
| 8316 | 三井住友FG | 7,500 円 | True | top OK |

### §3.2 実行結果

```
F286-AFTER-R1 MVP done: base_date=2026-05-21 artifacts=4 ranking_size=4
artifact_source=f062_preview f062_raw_kind=f062_actual_dict
f111_input_source=f111_real_batch
```

morning_line_material 結果:
- artifact_source = `f062_preview` ✓
- **f111_input_source = `f111_real_batch`** ✓ (= 自動推定、F111_REAL_BATCH_MARKERS
  4 件 hit)
- exclusions_summary = `{'sample_ticker': 1, 'tradable_universe_false': 0,
  'risk_above_pilot_limit': 1}`
- pilot_per_trade_risk_limit_yen = 15000

Top candidates (= 9th invariant 強制で excluded 候補なし):
- #1 🟢 4063 信越化学 score=189.0
- #3 🟡 8316 三井住友FG score=141.0
- (6920 = risk_above で top 抑制 / 1234 = sample で top 抑制)

Full ranking (= 全候補列挙、抑制理由を可視化):
- #1 4063 信越化学 score=189.0 excl=[]
- #2 6920 レーザーテック score=151.0 excl=[risk_above]
- #3 8316 三井住友FG score=141.0 excl=[]
- #4 1234 サンプル情報通信A score=82.0 excl=[sample]

## §4 作成 / 更新 file 一覧

### §4.1 新規 (2 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 02_todo/FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_results.md | 本 wave results (= 本 doc) |

### §4.2 更新 (2 件)

| path | 変更 |
|---|---|
| scripts/jobs/_after_r1_mvp.py | F111_REAL_BATCH_MARKERS / SAMPLE_TICKER_NAME_PREFIXES / PILOT_PER_TRADE_RISK_LIMIT_YEN / 3 helper / infer 拡張 / score penalty / ranking output 拡張 / morning_line_material 9th invariant / MVP_LOGIC_VERSION 1.2.0→1.3.0 |
| tests/scripts/jobs/test_run_f286_after_r1_night_batch.py | 21 件追加 (TestW60F111RealBatch{Inference, RankingPenalty, 9thInvariantEnforcement}) / logic_version assertion |

### §4.3 FIRE 本体 (= F111 / F062 / F282 / DATA-R3 / readiness / Ops / wrapper) 不触 ✓

scaffold (run_f286_after_r1_night_batch.py) 不触 ✓ — `--f111-input-source` CLI は
W60-F111-pre 既存で対応 (= `auto` で自動推定 / explicit `f111_real_batch` 受理)。

## §5 tests 結果

| wave | collected |
|---|---|
| W60-F111-pre | 4629 |
| **W60-F111-real-batch** | **4650** (= +21 件) |

全 **4650 PASS** ✓

### 新規 W60-F111-real-batch tests 21 件

- `TestW60F111RealBatchInference` 13 件:
  - F111_REAL_BATCH_MARKERS 定数 / PILOT_PER_TRADE_RISK_LIMIT_YEN 定数 /
    SAMPLE_TICKER_NAME_PREFIXES 定数
  - real_batch dict 推定 / sample ticker prefix 検出 / template prefix 検出 /
    all-same-digit code 検出 / 実 ticker 不誤検出 / tradable_universe 後方互換 /
    tradable explicit / risk explicit / risk from estimated / risk unknown
- `TestW60F111RealBatchRankingPenalty` 3 件:
  - sample heavily penalized / risk_above penalized / tradable=False penalized
- `TestW60F111RealBatch9thInvariantEnforcement` 5 件:
  - top excludes sample / top excludes risk_above / top includes eligible /
    exclusions_summary in material / pilot risk limit exposed

## §6 Codex 4 lane stdin audit 結果

W60-pilot-W1 の **短文 prompt + 単一観点 ≤80 words** 戦略を継続。

| Lane | 観点 | 起動 | 完全 reply | finding |
|---|---|---|---|---|
| A | F111_REAL_BATCH_MARKERS + inference | ✓ exit 0 | **✓ 取得** | LOW No finding |
| B | sample ticker detection | ✓ exit 0 | **✓ 取得** | LOW No finding |
| C | risk filter logic | ✓ exit 0 | **✓ 取得** | LOW No finding |
| D | 9th invariant enforcement | ✓ exit 0 | 途中停止 | self-audit: §3.2 smoke で実証 |

**Codex 3/4 = 75% 完全 reply** ← W60-pilot-W1 1/4 から **大幅改善**。

### §6.1 短文 prompt 戦略の効果

W60-pilot-W1 → W60-F111-real-batch:
- 1/4 → 3/4 = **+50%**
- prompt 長さ: 100 words → **≤80 words** で安定
- 観点 単一化 + factual confirm 系に絞る

**結論**: 4 lane + ≤80 words prompt + 単一観点 = **安定 75-100% 完全 reply**。
docs 長文 review は依然不安定だが、code factual confirm 系は安定。

## §7 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 (production/develop/staging) | 0 (= 本 wave は staging access スコープ外) |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| 実発注 / 楽天 / iSPEED / Computer Use | 0 |
| 自動発注 | 0 |
| sudo / git push / rm -rf / --no-verify | 0 |
| workflow / TODO Excel | 0 |
| FIRE 本体 (= F111/F062/F282/DATA-R3/readiness/Ops/wrapper) 変更 | 0 |
| 新規 import (= sqlite3/linebot/requests 等) | 0 |
| MVP_SAFETY_FLAGS 13 keys 全 False | 維持 ✓ |
| auto_order_allowed=False / manual_review_required=True 全 outputs 維持 | ✓ |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4629 → 4650 (= +21) |

## §8 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **3/4 = 75%** (= 短文 prompt 戦略で W60-pilot-W1 25% から大幅改善) |
| 短縮率 | 高 (= 4 観点 並列、3 観点で Codex confirm、1 観点 self-audit) |
| 採用率 | 100% (= LOW No finding 全件、設計通り実装) |
| 差戻率 | 0 |
| Integrator 負荷 | 中-高 (= 3 helper + score 拡張 + 9th invariant + 21 regression + Codex 4 lane + docs) |
| 安全事故 | **0** ✓ |

### §8.1 Codex 75% (= 改善実証)

| wave | parallel | 完全 reply 率 |
|---|---|---|
| W60.5 / W60.6 / W60-launchd-pre | 4 lane no sleep / short JSON read | 100% |
| W60-pilot-pre | 8 lane | 37.5% |
| W60-integration | 6 lane + sleep | 66.7% |
| W60-F111-pre | 4 lane no sleep (long prompts) | 0% |
| W60-pilot-W1 | 4 lane (≤100 words) | 25% |
| **W60-F111-real-batch** | **4 lane (≤80 words, factual confirm)** | **75%** |

→ **factual confirm + ≤80 words = 安定 75%+**、次 wave も同戦略で進める。

## §9 残課題 / 次 Wave

### §9.1 W60-F111-real-batch で開ける道

- D6 以降 = AFTER-R1 が f111_real_batch を自動認識、sample/risk 候補は top から
  自動除外、実 trade 可能候補のみが top に出る
- 「実在・取引可能・pilot 損失上限内」3 条件を **構造的に保証**

### §9.2 次 wave (= priority 順)

1. **W60-pilot-D6**: F111 real_batch 由来 (= safe fixture or staging real)
   候補で初の実 trade 可能 trade plan
2. **F111-real-batch-staging** (= 別 wave): staging DB read-only SELECT で
   実 candidates を F111 runner に流す path
3. **W61-pre**: price/return/paper_pnl 連携
4. **W60-launchd-real**: 本番 launchd 配置 (= Wave 41/45 後)

### §9.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §9.4 D6 期待

f111_input_source 進化 path:
- D1/D2: synthetic_fixture (= 直渡し)
- D3/D4: f062_preview + manual_seed (= 強 caveat)
- D5: f062_preview + f111_sample (= 中 caveat)
- **D6**: f062_preview + **f111_real_batch (= 最弱 caveat / 実 trade 可候補)**

実 trade 0/5 → **D6 で初の実 trade 候補出現** が現実的に視野へ。
(= staging real access は別 wave だが、safe fixture で D6 形式検証は可能)

## §10 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_plan]]
- [[../03_design/F111_real_batch_requirements_2026-05-14|F111-real-batch 要件 v1.0]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_W1_results|W1 集約 results]]
- [[FIRE_CODEX_R1_WAVE60_F111_PRE_results|W60-F111-pre results]]
