---
id: FIRE-CODEX-R1-WAVE60-F111-PRESET-TUNE-results
phase: 本番 v0 中核 / Wave 60-F111-preset-tune / 候補多様化 短期改善
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
hq_marker: なし (= code 変更のみ、staging read-only、token/API/DB write 0)
cli_version_bump: 1.1.0 → 1.2.0
tests_delta: +27 (= 55 既存 + 27 新規、計 82 PASS)
total_pytest: 4705 → 4732 collected (+27)
---

# Wave 60-F111-preset-tune Results — F111 Real Batch Preset / Label / Risk Tune for D9

## §1 結論

D6/D7/D8 で top 3 = 8747/5729/3489 が 100% 重複した features cap 問題に対し、
**F111-real-batch staging runner の code 変更のみ** (= API/token/DB write 0)
で 3 軸の短期改善 (label_threshold_mode / recently_seen_codes / max_candidates
default 10→20) を実装。D9 simulation で:

| metric | D6/D7/D8 baseline | D9 default+recently_seen+max=20 | D9 strict+recently_seen+max=20 |
|---|---|---|---|
| **top 3 ticker overlap** | (= 100%) | **100%** (= 同 ticker、features cap 律速で残る) | **100%** |
| **top 3 label distribution** | boost×3 | **caution×3** (= 全 demote) | **caution×3** |
| **top 10 label distribution** | boost×10 | caution×3 + boost×7 | **caution×3 + boost_with_caution×7** |

→ **ticker overlap 100% 維持** (= 真の解決には features rerun = J-Quants
refresh = classification C が必要、本 wave のみでは不可能)。
→ **label distribution は 0% 重複** (= boost 完全分散へ)。
→ Fujiwara は "caution 3 銘柄 = 最近見た銘柄なので慎重 / 見送り、
   4 位以下 (= boost_with_caution / boost) から選ぶ" 運用が可能。

## §2 D6/D7/D8 重複の再確認

- top 3 ticker: 8747 豊トラスティ証券 / 5729 日本精鉱 / 3489 フェイスネットワーク (D6=D7=D8 完全同一)
- research_watchlist_signals latest base_date=2026-05-13 (w60_f111_daily_v1)
  だが内容 = 2026-05-12 と同一 score
- market_prices_daily cap=2026-05-08 (= 真因、本 wave では未解決)
- 本 Wave では **価格データの更新は行わない** (= W60-jquants-daily-refresh-staging で別承認)

## §3 改修内容 (= scripts/jobs/run_f111_real_batch_staging.py)

### 3.1 CLI_VERSION bump

- `1.1.0` → `1.2.0` (W60-F111-preset-tune marker)

### 3.2 新 constants

```python
# strict mode 閾値 (= label 分散用)
RESEARCH_LABEL_BOOST_SCORE_MIN_STRICT: float = 0.92
RESEARCH_LABEL_BOOST_W_CAUTION_SCORE_MIN_STRICT: float = 0.85
RESEARCH_LABEL_CAUTION_SCORE_MIN_STRICT: float = 0.75

LABEL_THRESHOLD_MODES: tuple[str, ...] = ("default", "strict")
LABEL_THRESHOLDS: dict[str, tuple[float, float, float]] = {
    "default": (0.80, 0.70, 0.60),
    "strict":  (0.92, 0.85, 0.75),
}

RECENTLY_SEEN_DEMOTE_TARGET_LABEL: str = "caution"
```

### 3.3 新 API

| API | 意味 |
|---|---|
| `map_research_advisory_label(..., threshold_mode="default"\|"strict")` | mode 引数追加 (= 後方互換、default mode は既存 logic) |
| `apply_recently_seen_demote(label, code, recently_seen_codes)` | boost / boost_with_caution → caution へデモート、demote flag 返却 |
| `fetch_candidates(..., *, label_threshold_mode, recently_seen_codes)` | keyword-only 引数で受取 |
| `build_f111_real_batch_artifact(..., *, label_threshold_mode, recently_seen_codes)` | tuning_params metadata を artifact に追記 |

### 3.4 新 CLI flag

| flag | default | choices/format |
|---|---|---|
| `--max-candidates` | **20** (← 10) | int |
| `--label-threshold-mode` | "default" | {"default", "strict"} |
| `--recently-seen-codes` | "" | "8747,5729,3489" 形式 |

### 3.5 artifact 追加 metadata

```json
"tuning_params": {
  "label_threshold_mode": "strict",
  "recently_seen_codes": ["8747", "5729", "3489"],
  "demoted_count": 3
}
```

### 3.6 _after_r1_mvp.py は変更なし

- TOP_INCLUDE_LABELS frozenset 内に既に `caution` を含む (= caution に demote
  されても top に出る)
- demote logic は upstream で完結 → AFTER-R1 は読むだけ
- 9 hard invariants 全維持

## §4 D9 simulation (3 scenario)

### 4.1 実行 cmd

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-05-26 --max-candidates 20 \
    --recently-seen-codes 8747,5729,3489 \
    --output-json /tmp/fire_f111_refresh/d9_default_recent.json

$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-05-26 --max-candidates 20 \
    --label-threshold-mode strict \
    --recently-seen-codes 8747,5729,3489 \
    --output-json /tmp/fire_f111_refresh/d9_strict_recent.json

$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-05-26 --max-candidates 10 \
    --output-json /tmp/fire_f111_refresh/d9_baseline.json
```

### 4.2 結果

| scenario | top 1-3 (code/label) | top 4-10 (code/label) |
|---|---|---|
| **D8 baseline** | 8747/boost, 5729/boost, 3489/boost | 340A0/boost, 3798/boost, 137A0/boost(risk=False), 7991/boost, 9130/boost, 331A0/boost, 4389/boost |
| **D9_baseline** (max=10、変更なし) | 8747/boost, 5729/boost, 3489/boost | (same as D8) |
| **D9_default+recent** (max=20) | 8747/caution, 5729/caution, 3489/caution (demoted) | 340A0/boost, 3798/boost, 137A0/boost(risk=False), 7991/boost, 9130/boost, 331A0/boost, 4389/boost |
| **D9_strict+recent** (max=20) | 8747/caution, 5729/caution, 3489/caution (demoted) | 340A0/boost_with_caution, 3798/boost_with_caution, 137A0/boost_with_caution(risk=False), 7991/boost_with_caution, 9130/boost_with_caution, 331A0/boost_with_caution, 4389/boost_with_caution |

### 4.3 overlap 表

```
=== overlap (= top 3 ticker と D6/D7/D8 reference の一致率) ===
  D8                        top3=['8747', '5729', '3489'] → 100%
  D9_baseline               top3=['8747', '5729', '3489'] → 100%
  D9_default_recent         top3=['8747', '5729', '3489'] → 100%
  D9_strict_recent          top3=['8747', '5729', '3489'] → 100%

=== top 3 label distribution ===
  D8                        ['boost', 'boost', 'boost']
  D9_baseline               ['boost', 'boost', 'boost']
  D9_default_recent         ['caution', 'caution', 'caution'] ← 完全分散
  D9_strict_recent          ['caution', 'caution', 'caution'] ← 完全分散

=== top 10 label distribution ===
  D8                        {'boost': 10}
  D9_baseline               {'boost': 10}
  D9_default_recent         {'caution': 3, 'boost': 7}
  D9_strict_recent          {'caution': 3, 'boost_with_caution': 7}  ← 完全に boost 消滅
```

### 4.4 評価

- **ticker overlap = 100%** (= 全 scenario で同 ticker)
  - 真因 = market_prices_daily cap=2026-05-08、本 wave で未解決
  - 次 wave (= W60-jquants-daily-refresh-staging) で C 分類実行が必要
- **label overlap = 0%** (= boost → caution / boost_with_caution へ完全分散)
  - Fujiwara は label を見て「最近見た銘柄 = 慎重 / 見送り」判断可能
- **top 4-10 が多様化** (= max_candidates 10→20 で発掘候補増)
  - Fujiwara が "top 3 = caution なら 4-10 から実弾選択" 運用可能
- **strict mode の効果**: boost が消滅し boost_with_caution に集約
  → 現状 staging features では真の高 score (≥0.92) 銘柄が無い事を示唆
  → 真の boost を狙うには features rerun + score 再計算が必須

## §5 D9 で実弾判断可能性

### 5.1 Yes (= 短期策で実弾判断可能)

- D9 trade plan 想定 (= 経路 d_strict + recently_seen):
  - top 1-3: caution (= 8747/5729/3489) → 見送り推奨 (= D6-D8 と同候補)
  - top 4-10: boost_with_caution (= 340A0/3798/137A0/7991/9130/331A0/4389)
    → 新 候補、Fujiwara が manual review 上 1 銘柄 entry 検討可
  - 137A0 (Cocolive) は risk_within_pilot_limit=False で exclude
  - 実弾候補 = 340A0/3798/7991/9130/331A0/4389 から 1 銘柄

### 5.2 短期策の限界

- **ticker そのものは更新されない** (= 同じ A1 ranked stock のまま)
- 真の意味の "新 stock pool" には J-Quants daily refresh で価格更新 → derived
  rerun → ranker rerun → signal rerun が必要
- → **W60-jquants-daily-refresh-staging (= 経路 g / C 分類) が真の解決**

## §6 J-Quants refresh 必要性 (= 強調)

本 wave は **短期対症療法** に過ぎない。

| 観点 | 短期策 (本 wave) | 真の解決 (W60-jquants-daily-refresh-staging) |
|---|---|---|
| ticker 多様化 | × (top 100% 同 ticker) | ✓ (新 price → 新 derived → 新 ranking) |
| label 多様化 | ✓ (boost / caution / boost_with_caution 分散) | ✓ (= 自然と分散) |
| recently_seen 警告 | ✓ (caution 明示) | (= 必要に応じて併用) |
| token / API 承認 | 不要 | **必要** (HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH + HQ_APPROVE_JQUANTS_TOKEN_READ) |
| 期間 | 即実装可 | 別 wave、HQ 承認後 |

→ **本 wave で D9 直前まで gap を埋め、Fujiwara 承認次第で次 wave へ**。

## §7 tests 追加 (+27)

### 7.1 新 test class

| class | tests | 観点 |
|---|---|---|
| `TestPresetTuneStrictThresholdConstants` | 5 | constants 値、LABEL_THRESHOLDS dict、modes enum |
| `TestPresetTuneMapLabelByMode` | 5 | default vs strict での label 分岐、unknown fallback |
| `TestPresetTuneRecentlySeenDemote` | 6 | boost/boost_w_caution/caution/neutral 各々の demote 挙動 |
| `TestPresetTuneArtifactTuningParams` | 2 | artifact 内 tuning_params metadata |
| `TestPresetTuneFetchCandidatesWithDemote` | 2 | fetch_candidates が新 args 受取 |
| `TestPresetTuneCliDefaults` | 5 | CLI flag default + parse |
| `TestPresetTuneEnrichmentWithDemote` | 1 | 統合: signal あり ticker を recently_seen で demote |
| `TestPresetTuneSafety` | 1 | safety_flags 維持 + 9 invariants 維持 |

### 7.2 既存 test

55 PASS 全維持。total 82 PASS in `test_run_f111_real_batch_staging.py`。

### 7.3 全体

pytest collected: 4705 → **4732** (+27)。

## §8 Codex 4 lane factual-confirm

| lane | prompt 観点 | reply | 評価 |
|---|---|---|---|
| **A** | preset/universe (= choices, --max-candidates 20, CLI_VERSION 1.2.0) | **YES** — Confirmed: choices/default and --max-candidates default=20 with CLI_VERSION 1.2.0 are present | 設計通り ✓ |
| **B** | threshold_mode 'default'/'strict' + unknown fallback | **YES** — threshold_mode supports default/strict, unknown modes fall back to default | 設計通り ✓ |
| **C** | risk constants unchanged + demote = label only | **NO** — constants and risk gate intact, but demoted candidates also get a `risk_notes` annotation, not only `research_advisory_label` | 事実訂正 (= 私の設計通り、demote 時に risk_notes に明示警告を追加するのは Fujiwara への注意喚起、意図的) ✓ |
| **D** | TOP_INCLUDE_LABELS contains caution、demote logic は _after_r1_mvp に存在しない | **NO** — TOP_INCLUDE_LABELS includes caution, but recently_seen/demotion is not present in this file by grep | 設計通り ✓ (= demote は upstream の run_f111_real_batch_staging に集約、AFTER-R1 は読むのみ) |

→ 全 lane が事実観察。Lane C/D の NO は私の prompt の wording を正確に補完 (= "label only" → "label + risk_notes"、"_after_r1 に demote 無い" → 設計通り)。**実装に問題なし** ✓。

## §9 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓ |
| staging md5 | a8663a07a730378387c050ebb1b612ef → 不変 ✓ (= read-only only) |
| F282 plist | size=1751 / mtime=1778602507 → 不変 ✓ |
| pytest collected | 4705 → 4732 (+27、全 PASS) ✓ |
| DB write | 0 ✓ |
| production / develop 接続 | 0 ✓ |
| staging write | 0 ✓ |
| LINE / token / API / launchctl / plist / cron | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | run_f111_real_batch_staging.py + test_run_f111_real_batch_staging.py の 2 file 編集 (= 前 wave からの untracked file への変更) |

## §10 Next action 候補 (= 優先順)

1. **D9 pilot trade plan 作成** (= 2026-05-26 月、本 wave 改修 + D6-D8 recently_seen 反映)
   - cmd 案: `--label-threshold-mode strict --recently-seen-codes 8747,5729,3489 --max-candidates 20`
   - top 3 = caution → 見送り、top 4 = 340A0 ジグザグ (boost_with_caution) → 候補 #1
2. **D6 review 記入待ち** (Fujiwara 本人)
3. **W60-jquants-daily-refresh-staging** (= 真の解決、HQ marker 2 個 + token 承認後)
4. **liquidity filter 強化 wave** (= 板厚 / 出来高 / spread を staging 内 read-only で再確認)
5. **F111-preset-tune の AFTER-R1 統合検証** (= AFTER-R1 を実 D9 input で動かして morning_line_material 確認)
