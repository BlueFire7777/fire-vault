---
id: FIRE-CODEX-R1-WAVE60-pilot-D6-results
phase: 本番 v0 中核 / Wave 60-pilot-D6 / D6 real_batch + research enriched
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-21
pilot_day: D6
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D6_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_RESEARCH_ADVISORY_STAGING_results.md
  - 04_daily/2026-05-21_manual_live_pilot_trade_plan.md
  - 04_daily/2026-05-21_manual_live_pilot_review.md
---

# Wave 60-pilot-D6 Results — D6 Real Batch Research-Enriched Manual Live Pilot v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉🎉🎉 D6 (= 2026-05-21 木) で初の実 trade 可能 day 達成 ✓** —
D1-D5 全 day 実 entry 0/5 から **構造的飛躍**。

9 hard invariants 全 PASS:
- artifact_source = f062_preview ✓
- f062_raw_kind = f062_actual_dict ✓
- **f111_input_source = f111_real_batch** ✓ (= staging real_batch + research enriched)
- DATA-R3 freshness verdict = OK ✓
- forbidden_check.passed = True ✓
- auto_order_allowed = False ✓
- manual_review_required = True ✓
- safety_flags 13 keys 全 False ✓
- **top_candidates count = 3** ✓ (= D5 まで 0 件 → 初の出現)

Top candidates:
- 🟢 8747 豊トラスティ証券 score=221.99 conf=0.93
- 🟢 5729 日本精鉱 score=219.52 conf=0.92
- 🟢 3489 フェイスネットワーク score=217.8 conf=0.91

ただし候補は **中小型成長株** = liquidity 手動確認必須。藤原さんが
trade_plan §7 で entry/skip 判断。

## §2 D6 5 step chain 実行結果

### §2.1 Step 1: F111-real-batch enriched (= W60-research-advisory-staging 成果物 reuse)

```bash
# W60-research-advisory-staging の output を D6 prep へ複製
cp /tmp/fire_research_advisory/f111_enriched_v2.json \
   /tmp/fire_d6_prep/f111_real_batch_2026-05-21.json
```

15 candidates、全 boost、eligible 14、exclusions risk_above 1 (Cocolive 137A0)

### §2.2 Step 2: DATA-R3 dry-run runner

verdict=OK / aggregate_exit_code=0 / db_row_writes=0 / 4 sub-runner 全 ok

### §2.3 Step 3 ~ Step 4: F062 + AFTER-R1 → reports/after_r1/

```bash
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all --base-date 2026-05-21 \
  --f062-preview-json /tmp/fire_d6_prep/f111_real_batch_2026-05-21.json \
  --data-r3-freshness-json /tmp/fire_d6_prep/data_r3_freshness_2026-05-21.json \
  --output-dir /Users/bluefire/fire/reports/after_r1
```

結果:
- artifacts=4 / ranking_size=15
- artifact_source=**f062_preview** ✓
- f062_raw_kind=**f062_actual_dict** ✓
- **f111_input_source=f111_real_batch** ✓
- 4 file × (JSON+Markdown) = **8 file 生成** in `reports/after_r1/`

### §2.4 Step 5: 9 invariants hard check (= 全 PASS ✓)

詳細: 本 doc §1 + trade plan §3.0

**✓ Pilot D6 GO check PASSED — all 9 hard invariants satisfied**

exclusions_summary: sample_ticker 0 / tradable_universe_false 0 /
risk_above_pilot_limit 1 (= Cocolive 137A0)

## §3 D6 Pilot judgment: GO (= eligible_with_liquidity_check)

### §3.1 GO 構造的判定: 9 hard invariants 全 PASS ✓

### §3.2 ただし caveat (= 中小型成長株 liquidity)

- 候補は **中小型成長株 universe** (Core30/Large70 大型株とは別)
- 板薄・値動き荒い可能性
- 100 株 entry でも spread / 出来高 / 板厚 で skip 判断必要
- 寄付き直後 5 分は様子見推奨

→ **pilot_use = `eligible_with_liquidity_check`** (= 構造的 GO だが §7 手動確認必須)

### §3.3 推奨フロー

1. 朝 8:30: TDnet / 日経で当日材料 / 大型イベント確認
2. 朝 8:55: iSPEED で top 3 候補の 板 / 気配 / 出来高目処 確認
3. 寄付き 9:00 直前: 気配 ±2% / 板 5 本 / 出来高 5,000 株目処
4. 寄付き 5 分後 (= 9:05): 1 銘柄選択 entry または skip
5. 15:10 までに手動 close (= 信用)

## §4 Top candidates 詳細

| rank | ticker | name | label | confidence | score | sector | scale |
|---|---|---|---|---|---|---|---|
| 1 | 8747 | 豊トラスティ証券 | 積極的買い推奨 🟢 | 0.93 | 221.99 | 金融 | (= 中小型) |
| 2 | 5729 | 日本精鉱 | 積極的買い推奨 🟢 | 0.92 | 219.52 | 鉄鋼・非鉄 | (= 中小型) |
| 3 | 3489 | フェイスネットワーク | 積極的買い推奨 🟢 | 0.91 | 217.80 | 不動産 | (= 中小型) |

reason_tags (= staging research_watchlist_signals 由来):
- final_rank_label = A1
- strongest_strategy = quality_value 等
- rank_reason = quality value boost 等
- watchlist_decision = selected

source_modules: market_listings + market_prices_daily +
**research_watchlist_signals** ★

## §5 作成 file 一覧

### §5.1 新規 (4 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 04_daily/2026-05-21_manual_live_pilot_trade_plan.md | D6 trade plan / 15 セクション / GO + liquidity caveat / §7 liquidity 手動 check 欄 |
| 04_daily/2026-05-21_manual_live_pilot_review.md | D6 review blank / 16 セクション / D6 初実候補 day 特記 |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D6_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D6_results.md | 本 wave results (= 本 doc) |

### §5.2 reports/after_r1/ (= 8 file 追加、D6 用 base_date=2026-05-21)

### §5.3 /tmp/fire_d6_prep/ (= 2 中間 artifact)

f111_real_batch_2026-05-21.json / data_r3_freshness_2026-05-21.json

### §5.4 FIRE 本体不触 ✓

scripts/jobs/ + tests/ 不触。pytest collected: 4705 (= 不変)

## §6 D1-D6 path artifact 推移 (= W2 集約 引き継ぎ)

| Day | 日付 | artifact_source | f111_input_source | top_candidates | 実 trade |
|---|---|---|---|---|---|
| D1 | 5/14 木 | (field なし) | (field なし) | 0 件 | 0 |
| D2 | 5/15 金 | synthetic_fixture | (field なし) | 0 件 | 0 |
| D3 | 5/18 月 | f062_preview | manual_seed | 0 件 (= 値嵩 skip) | 0 |
| D4 | 5/19 火 | f062_preview | manual_seed | 0 件 (= 同) | 0 |
| D5 | 5/20 水 | f062_preview | f111_sample | 0 件 (= サンプル) | 0 |
| **D6** | **5/21 木** | **f062_preview** | **f111_real_batch ★** | **3 件 ✓** | **可能 (= 藤原さん次第)** |

進化:
- artifact_source: (none)/synthetic → f062_preview 4/6 ✓
- f111_input_source: 0 → manual_seed → f111_sample → f111_real_batch 3 段階進化
- top_candidates: 0/5 → 3 件 (D6 初)
- 実 trade 可能性: D1-D5 構造的 No → D6 構造的 Yes ✓

## §7 W2 集約 (= D7-D10 後) への引き継ぎ材料

### §7.1 D6 達成事項

- 初の実 trade 可能 day
- 9 hard invariants 構造的 PASS pattern 確立
- 中小型成長株 universe 開放
- f111_real_batch path 本番運用化
- liquidity 手動 check 欄 整備

### §7.2 D7-D10 で観察すべき項目

1. 実 entry の有無 (= D6 で初の判断)
2. liquidity check 結果分布 (= 中小型 entry 可否)
3. 中小型 spread / 出来高 実測
4. pattern hit 率 (= 実 outcome 蓄積)
5. f111_real_batch 安定性 (= staging signal 更新タイミング)
6. value 株 exclusion 機能継続確認 (= 構造的 skip)
7. event 跨ぎ skip 発動率
8. research_advisory_label 分布 (= boost / boost_with_caution / caution)

### §7.3 W60-pilot-W2 で実施予定

- D6-D10 集約
- pattern promote/suppress/watch 結論 (= D1-D10 累積 10 サンプル)
- liquidity filter 強化要否判断
- W61-pre (= price/return/paper_pnl) 着手判断
- F062 朝 batch launchd 化要否

## §8 Codex 8 lane 判断: 不使用 (= 本線主導)

ユーザー spec で「日次 trade plan は本線主導でよい」明示。
W60-research-advisory-staging で chain logic を Codex audit 済。
W2 集約 (= W60-pilot-W2) で Codex 4 lane を実施予定。

本線 self-check:
- ✓ 5 step chain 全完遂、exit 0
- ✓ 9 hard invariants 自動 hard check 全 PASS
- ✓ artifact_source / f111_input_source 機械的確認
- ✓ top_candidates ≥ 1 件確認 (= 3 件 ✓)
- ✓ 中小型 liquidity caveat 明示

## §9 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 (production/develop) | 0 |
| DB sqlite 接続 (staging) | read-only 1 (= W60-research-advisory-staging で SELECT、本 wave 段階で reuse) |
| 3 DB row_writes | 0 (= DATA-R3 dry-run で確認) |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| 実発注 / 楽天 / iSPEED / Computer Use | 0 |
| 自動発注 | 0 |
| sudo / git push / rm -rf / --no-verify | 0 |
| workflow / TODO Excel | 0 |
| FIRE 本体コード変更 | 0 |
| file write 範囲 | reports/after_r1/ (= 8 file) + fire-vault/* (= 4 file) + /tmp/fire_d6_prep/ (= 2 file) |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4705 (= 不変) |

## §10 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **0/8 = 0%** (= 本線主導、強い技術的理由) |
| 短縮率 | N/A |
| 採用率 | N/A |
| 差戻率 | 0 |
| Integrator 負荷 | 中 (= chain re-run + 9 invariants check + trade plan + review + W2 引き継ぎ + vault) |
| 安全事故 | **0** ✓ |

## §11 今日 (5/21 木) の人間アクション

### §11.1 藤原さん が朝行うこと

1. `~/fire-vault/04_daily/2026-05-21_manual_live_pilot_trade_plan.md` 確認
2. §3.0 artifact_source/f111_input_source/research_enrichment 認識
3. §4 top candidates (= 8747 / 5729 / 3489) 確認
4. §5 中小型 caveat 認識
5. 8:30 朝 TDnet / 日経で材料 / イベント確認
6. 8:55 iSPEED で 3 候補 板 / 気配 確認
7. §7 liquidity check 実行 (= 出来高 / 板 / spread)
8. §14 final decision (= enter / watch / skip)
9. enter なら 1 銘柄選んで iSPEED 手動発注 / 15:10 close

### §11.2 場後 行うこと

1. `~/fire-vault/04_daily/2026-05-21_manual_live_pilot_review.md` 記入
2. §7 liquidity actual 記入 (= D6 特記)
3. §9.4 D6 運用フロー評価 (= 初実 trade 可能 day 印象) 重要
4. §14 next action 選択 (= D7 / W2 集約 / W61-pre 等)

### §11.3 Claude Code 行わないこと (= 絶対禁止)

実発注 / 楽天 / iSPEED / Computer Use / 自動発注 / LINE / DB write /
token / API / launchctl / plist / cron / workflow / git push / sudo / rm

## §12 残課題 / 次 Wave

### §12.1 D6 → D7 path

- D7 (= 2026-05-22 金): 同 sequence で trade plan、D6 結果 反映
- 場合により liquidity filter 強化判断
- 場合により W61-pre (= price/return) 着手判断

### §12.2 並行 wave 候補

| Wave | 内容 | 優先度 |
|---|---|---|
| W60-pilot-D7 | 同 chain で trade plan | 高 |
| W60-pilot-W2 (= D6-D10 後) | 集約 + Codex 4 lane | 高 |
| 流動性 filter 強化 | 出来高 / 板厚 を risk_notes に追記 (= 別 wave) | 中 |
| W61-pre | price/return/paper_pnl 連携 | 中 |
| W60-launchd-real | 本番 launchd 配置 (= Wave 41/45 後) | 低 |

### §12.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

## §13 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_PILOT_D6_plan]]
- [[FIRE_CODEX_R1_WAVE60_RESEARCH_ADVISORY_STAGING_results|W60-research-advisory-staging]]
- [[FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_STAGING_results|W60-F111-real-batch-staging]]
- [[../03_design/F111_real_batch_requirements_2026-05-14|F111-real-batch 要件 v1.0]]
- [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- [[../04_daily/2026-05-21_manual_live_pilot_trade_plan|5/21 D6 trade plan]]
- [[../04_daily/2026-05-21_manual_live_pilot_review|5/21 D6 review (blank)]]
