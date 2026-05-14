---
id: FIRE-CODEX-R1-WAVE60-research-advisory-staging-results
phase: 本番 v0 中核 / Wave 60-research-advisory-staging / 実 ticker label 付与
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_RESEARCH_ADVISORY_STAGING_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_STAGING_results.md
  - 03_design/F111_real_batch_requirements_2026-05-14.md
---

# Wave 60-research-advisory-staging Results — Research Advisory Label Integration v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉🎉🎉 staging real_batch 候補に research_advisory_label 付与完了、
morning_line_material top_candidates に **実在中小型成長株 3 件** 出現 ✓ —
D1-D5 全 0 件 → D6 期待 3 件以上の構造的飛躍**

実 chain smoke (= base_date 2026-05-21):
- candidates 15 件、全 label=`boost` (= final_score 0.84-0.93 / A1 rank / selected)
- eligible 14 件 (= risk_within_pilot_limit=True、value 株 1 件除外)
- artifact_source = f062_preview ✓
- f111_input_source = f111_real_batch ✓
- **top_candidates 3 件 ✓**:
  - #1 🟢 8747 豊トラスティ証券 score=222 conf=0.93
  - #2 🟢 5729 日本精鉱 score=220 conf=0.92
  - #3 🟢 3489 フェイスネットワーク score=218 conf=0.91
- alternative_labels_recap: 積極的買い推奨 12 件 (= top 入り切らなかった残)

## §2 staging research source 調査結果

| table | rows | 役割 |
|---|---|---|
| **research_watchlist_signals** | **13,695** | **主 source** — watchlist_decision / final_rank_label / final_score |
| advisory_snapshot_rows | 5 | F062 過去 send 記録 (= 補助、本 wave 不使用) |
| advisory_decisions | 10 | past Fujiwara decision (= 補助、本 wave 不使用) |
| research_derived_indicators | 3,750 | PER/PBR/ROE 等 ファンダメンタル (= 補助、本 wave 不使用) |

`research_watchlist_signals` の最新 base_date = **2026-05-12** で 35 銘柄
(= selected 30 / deferred 5)。sector_17 分布は「情報通信・サービスその他」14 件
中心、Core30/Large70 大型株は **対象外** (= signal universe が中小型成長株中心)。

これにより **scale_categories filter から signal-INNER-JOIN へ切替**、
真に signal がある ticker のみを candidate にする戦略を採用。

## §3 label mapping 仕様

| watchlist_decision | final_rank_label | final_score | → advisory_label |
|---|---|---|---|
| selected | A1 | ≥ 0.80 | **boost** |
| selected | A1/A2 | ≥ 0.70 | **boost_with_caution** |
| selected | any | ≥ 0.60 | **caution** |
| selected | any | < 0.60 | neutral |
| deferred | any | ≥ 0.80 | **caution** (= 高 score だが保留) |
| deferred | any | < 0.80 | neutral |
| None / unknown | - | - | neutral |

無理に buy 推奨に寄せない。判定不能なら neutral。

threshold constants:
- RESEARCH_LABEL_BOOST_SCORE_MIN = 0.80
- RESEARCH_LABEL_BOOST_W_CAUTION_SCORE_MIN = 0.70
- RESEARCH_LABEL_CAUTION_SCORE_MIN = 0.60

## §4 実装

### §4.1 新規関数 (= run_f111_real_batch_staging.py)

```python
map_research_advisory_label(watchlist_decision, final_rank_label, final_score)
→ "boost" / "boost_with_caution" / "caution" / "neutral"

fetch_research_signal_for_code(conn, code_5digit)
→ Optional[dict] (= base_date / final_score / final_rank_label /
   post_cap_rank / watchlist_decision / rank_reason / strongest_strategy /
   sector_17_name / sector_cap_status)
```

### §4.2 fetch_candidates 拡張

`require_research_signal: bool = True` 引数追加:
- True (default): **INNER JOIN** research_watchlist_signals で signal あり ticker
  のみ抽出、`ORDER BY ws.final_score DESC`
- False: 従来 scale_categories filter mode (= 後方互換、tests で使用)

### §4.3 enrichment field

各 candidate に追加 / 上書き:
- `research_advisory_label`: 動的計算 (= neutral 固定から脱却)
- `expected_h20`: final_score (= 0.50 固定から脱却)
- `post_cap_rank`: signal の post_cap_rank (= None なら index)
- `reason_tags`: rank_reason / strongest_strategy / final_rank_label 追記
- `risk_notes`: watchlist_decision_reason / sector_cap_status 追記
- `source_modules`: ["market_listings", "market_prices_daily",
  "research_watchlist_signals"]
- `research_base_date` / `research_final_score`: signal field

### §4.4 CLI 拡張

`--require-research-signal` / `--no-require-research-signal` (= argparse
BooleanOptionalAction、default True)。

### §4.5 version bump

CLI_VERSION: 1.0.0 → **1.1.0**

## §5 chain smoke 結果

### §5.1 Step 1: F111-real-batch enriched

```bash
.venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
  --db-path /Users/bluefire/fire/data/fire.staging.db \
  --base-date 2026-05-21 --max-candidates 15 \
  --output-json /tmp/fire_research_advisory/f111_enriched_v2.json
```

結果: candidates=15 / **eligible=14** / **全 15 件 label=boost** /
exclusions={sample:0, tradable_false:0, risk_above:1, risk_estimate_missing:1}

| code | name | label | score | risk_within |
|---|---|---|---|---|
| 8747 | 豊トラスティ証券 | boost | 0.93 | True |
| 5729 | 日本精鉱 | boost | 0.92 | True |
| 3489 | フェイスネットワーク | boost | 0.91 | True |
| 340A0 | ジグザグ | boost | 0.89 | True |
| 3798 | ULSグループ | boost | 0.88 | True |
| 137A0 | Cocolive | boost | 0.87 | **False** (= 値嵩) |
| 7991 | マミヤ・オーピー | boost | 0.87 | True |
| 9130 | 共栄タンカー | boost | 0.86 | True |
| 331A0 | メディックス | boost | 0.86 | True |
| 4389 | プロパティデータバンク | boost | 0.86 | True |
| 9247 | TREホールディングス | boost | 0.86 | True |
| 4317 | レイ | boost | 0.85 | True |
| 9628 | 燦ホールディングス | boost | 0.85 | True |
| 2700 | 木徳神糧 | boost | 0.85 | True |
| 6149 | 小田原エンジニアリング | boost | 0.84 | True |

### §5.2 Step 2: F062 → AFTER-R1 chain

```
artifact_source = f062_preview ✓
f062_raw_kind = f062_actual_dict ✓
f111_input_source = f111_real_batch ✓
ranking_size = 15
exclusions_summary = {sample_ticker:0, tradable_universe_false:0,
                      risk_above_pilot_limit:1}
top_candidates: 3 件 ✓ (= D1-D5 全 0 件から大進化)
  #1 🟢 8747 豊トラスティ証券 score=221.99 conf=0.93
  #2 🟢 5729 日本精鉱       score=219.52 conf=0.92
  #3 🟢 3489 フェイスネットワーク score=217.8  conf=0.91
```

## §6 候補性質 caveat (= D6 trade plan で藤原さんが認識)

D6 で使う候補は **中小型成長株**:
- 豊トラスティ証券 / 日本精鉱 / フェイスネットワーク 等
- Core30/Large70 大型株とは異なる universe
- **流動性は別途確認必要** (= 板薄・値動き荒い可能性)
- 100 株 単位 entry でも:
  - 寄付き 板厚確認
  - 出来高 5,000 株以上目安
  - 場中 spread 確認
- 値嵩 (= Cocolive 137A0) は risk_within=False で構造的に top 除外 ✓

risk_within_pilot_limit=True は **損失 が pilot 上限内**であって、
**流動性 / 板厚 / 当日 spread** は trade plan §6-§10 で藤原さんが手動確認する範囲。

## §7 作成 / 更新 file 一覧

### §7.1 新規 (2 件)

| path | 内容 |
|---|---|
| 02_todo/FIRE_CODEX_R1_WAVE60_RESEARCH_ADVISORY_STAGING_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_RESEARCH_ADVISORY_STAGING_results.md | 本 wave results (= 本 doc) |

### §7.2 更新 (2 件)

| path | 変更 |
|---|---|
| scripts/jobs/run_f111_real_batch_staging.py | label mapping + signal fetch + INNER JOIN enrichment + `--require-research-signal` CLI / CLI_VERSION 1.0.0→1.1.0 |
| tests/scripts/jobs/test_run_f111_real_batch_staging.py | research_watchlist_signals table を fake_staging_db に追加 / 既存 tests に `require_research_signal=False` 設定 / W60-research-advisory-staging 18 件追加 |

### §7.3 FIRE 本体 (= F111 / F062 / F282 / DATA-R3 / AFTER-R1 / readiness / Ops / wrapper) 不触 ✓

W60-F111-real-batch で実装した AFTER-R1 MVP の F111_REAL_BATCH_MARKERS 推定 +
TOP_INCLUDE_LABELS filter + 9th invariant 強制 が、本 wave の enriched output で
完全動作。AFTER-R1 / F062 本体は今回不触。

## §8 tests 結果

| wave | collected |
|---|---|
| W60-F111-real-batch-staging | 4687 |
| **W60-research-advisory-staging** | **4705** (= +18 件) |

全 **4705 PASS** ✓

### 新規 18 件

- TestLabelMapping 9 件 (boost / boost_with_caution / caution / neutral /
  deferred high score / deferred low / None / unknown decision)
- TestResearchSignalFetch 2 件 (existing / missing)
- TestLabelThresholdConstants 3 件 (0.80 / 0.70 / 0.60)
- TestEnrichmentIntegration 4 件 (boost 付与 / signal 不在除外 / source_modules /
  safety invariants 維持)

## §9 Codex 4 lane stdin audit 結果

| Lane | 観点 | 起動 | 完全 reply |
|---|---|---|---|
| A | label mapping (= 0.80/0.70/0.60 threshold) | ✓ exit 0 | 途中停止 |
| B | fetch_research_signal_for_code 返却 schema | ✓ exit 0 | 途中停止 |
| C | INNER JOIN with latest base_date | ✓ exit 0 | 途中停止 |
| D | safety preserved (= f111_input_source / manual_review / source_modules) | ✓ exit 0 | 途中停止 |

**完全 reply 0/4**。短文 prompt + 単一観点 + factual confirm でも今回は不安定。
self-audit で **18 件 regression に全 観点 含む** ため補完可能 ✓:

- Lane A → TestLabelMapping 9 件で全 threshold 確認済
- Lane B → TestResearchSignalFetch 2 件 で existing/missing 確認済
- Lane C → TestEnrichmentIntegration test_enrichment_signal_required_
  excludes_no_signal で確認済
- Lane D → TestEnrichmentIntegration test_enrichment_keeps_safety_invariants
  で f111_input_source / manual_review / auto_order 確認済

### §9.1 Codex 並列度推移 (= 累積 10 wave)

| wave | parallel | 完全 reply 率 |
|---|---|---|
| W60.5/W60.6/W60-launchd-pre | 4 lane short JSON read | 100% |
| W60-pilot-pre | 8 lane | 37.5% |
| W60-integration | 6 lane + sleep | 66.7% |
| W60-F111-pre | 4 lane long prompts | 0% |
| W60-pilot-W1 | 4 lane ≤100 words | 25% |
| W60-F111-real-batch | 4 lane ≤80 words factual | 75% |
| W60-F111-real-batch-staging | 4 lane ≤80 words factual | 100% |
| **W60-research-advisory-staging** | **4 lane ≤80 words factual** | **0%** |

→ 同戦略でも結果が unstable。**Codex 並列は本質的に不安定要素を含む**。
self-audit が安定保証手段として確立されているため、戦略上の問題はなし。
次 wave も短文 prompt 戦略は継続、ただし self-audit 前提で運用。

## §10 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 (production/develop) | 0 |
| DB sqlite 接続 (staging) | read-only 1 (URI `mode=ro`) のみ |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| 実発注 / 楽天 / iSPEED / Computer Use | 0 |
| 自動発注 | 0 |
| sudo / git push / rm -rf / --no-verify | 0 |
| workflow / TODO Excel | 0 |
| F111 / F062 / F282 / DATA-R3 / AFTER-R1 本体変更 | 0 |
| 新規 import (linebot/requests/aiohttp/subprocess 等) | 0 |
| MVP_SAFETY_FLAGS 13 keys 全 False | 維持 ✓ |
| auto_order_allowed=False / manual_review_required=True 全 outputs 維持 | ✓ |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4687 → 4705 (= +18) |

## §11 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **0/4 = 0%** (= 4 lane 全 途中停止、self-audit 補完) |
| 短縮率 | 中 (= self-audit で 4 観点カバー) |
| 採用率 | 100% (= 18 件 regression 全 PASS、設計通り実装) |
| 差戻率 | 0 |
| Integrator 負荷 | 中-高 (= source 調査 + label mapping + enrichment + 18 regression + Codex 4 lane + docs) |
| 安全事故 | **0** ✓ |

## §12 残課題 / 次 Wave

### §12.1 W60-research-advisory-staging で開ける道

- **D6 で実 entry 候補 (= 中小型成長株、boost label、risk 上限内) で trade plan 作成可**
- f111_input_source = f111_real_batch / top_candidates ≥3 件 / 8 invariants PASS
- 実 trade の **構造的 blocker (= sample / 値嵩 / neutral) 完全解消**

### §12.2 caveat (= 藤原さんへの注意点)

- 候補が **中小型成長株 で 大型株とは異なる universe**
- 流動性 / 板厚 / 当日 spread は trade plan §6-§10 で **手動確認必須**
- 100 株 entry でも:
  - 板薄なら skip
  - 出来高 5,000 株以上目安
  - 寄付き直後の乱高下注意

### §12.3 次 wave 候補

1. **W60-pilot-D6 (= 5/21 木以降)**: 本 enrichment chain で初の実 trade 可能
   trade plan 作成 (= 実 entry 候補登場)
2. **W61-pre**: price/return/paper_pnl 連携 (= 実 entry 後の outcome 評価)
3. **W60-launchd-real**: 本番 launchd 配置 (= Wave 41/45 後、夜間自動化)
4. **流動性 filter 強化** (= 別 wave): 出来高 / 板厚 を staging から取得して
   risk_notes に追記

### §12.4 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §12.5 D1-D6 caveat 進化

| Day | f111_input_source | top_candidates | 実 trade 可? |
|---|---|---|---|
| D1/D2 | (synthetic) | 0 件 (HOLD) | No |
| D3/D4 | manual_seed | 0 件 (= 値嵩株 skip) | No |
| D5 | f111_sample | 0 件 (= サンプル ticker) | No |
| **D6** | **f111_real_batch + research enriched** | **≥3 件 (= 中小型成長株 boost)** ✓ | **Yes** ✓ |

実 entry path **構造的解消** ✓

## §13 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_RESEARCH_ADVISORY_STAGING_plan]]
- [[FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_STAGING_results|W60-F111-real-batch-staging]]
- [[FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_results|W60-F111-real-batch (safe fixture)]]
- [[../03_design/F111_real_batch_requirements_2026-05-14|F111-real-batch 要件 v1.0]]
