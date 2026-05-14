---
id: FIRE-CODEX-R1-WAVE60-F111-real-batch-staging-results
phase: 本番 v0 中核 / Wave 60-F111-real-batch-staging / staging real candidate 供給
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_STAGING_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_results.md
  - 03_design/F111_real_batch_requirements_2026-05-14.md
---

# Wave 60-F111-real-batch-staging Results — staging Real Candidate Source v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 staging DB read-only → F111-real-batch artifact → F062 → AFTER-R1 完全 chain
成功 ✓ — 実在 JP 株 ticker (= 1925 大和ハウス工業 / 4063 信越化学 等) で
f111_input_source=f111_real_batch を自動推定**

D1-D5 で実 entry 0/5 だった主因 (= sample ticker / 値嵩株 / F111 朝 batch 不在)
の最後の解消 path 完成。**D6 以降は実在 ticker で trade plan が作成可能**。

Codex 4 lane stdin audit: **4/4 = 100% 完全 reply、LOW No finding 全件** ✓
(= W60-F111-real-batch 75% → 本 wave **100%** 改善実証)

## §2 staging DB schema 確認

read-only probe で確認した関連 table:
- `market_listings` (= 4,449 銘柄、code/company_name/sector_17_name/scale_category 等)
- `market_prices_daily` (= code/date/open/high/low/close/volume)
- `do_not_trade_list` (= 0 rows、全 tradable)
- `advisory_decisions` / `advisory_snapshots` / `research_watchlist_signals` (= 未使用、将来 wave)

scale_category 分布: `-` / `TOPIX Core30` / `TOPIX Large70` / `TOPIX Mid400` /
`TOPIX Small 1` / `TOPIX Small 2`。

## §3 実装 file (= 新規 1 件)

`scripts/jobs/run_f111_real_batch_staging.py` (= ~360 行):
- staging DB 読み取り (sqlite URI `mode=ro` で write 不可)
- production / develop DB path refuse (`is_safe_db_path`)
- output path guard (= W60.6-fix pattern 流用、`is_safe_output_path`)
- ticker 5 桁 → 4 桁 正規化 (= "40630" → "4063")
- sample / tradable / risk filter
- F062 actual format 互換 output (= chunks / selected_rows / selected_count /
  safety_footer_present / forbidden_phrase_count)
- 各 candidate に F111_REAL_BATCH_MARKERS (= tradable_universe /
  estimated_100_share_risk / risk_within_pilot_limit / realtime_market_cap) +
  `f111_input_source='f111_real_batch'` 明示

CLI:
```
--db-path /Users/bluefire/fire/data/fire.staging.db
--base-date YYYY-MM-DD
--max-candidates 10
--scale-categories {core30, large70, mid400, core_and_large, all_top}
--output-json <safe path>
--dry-run (default)
```

## §4 完全 chain smoke 結果 (= staging → F111 → F062 → AFTER-R1)

### §4.1 Step 1: staging real_batch 生成

```bash
.venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
  --db-path /Users/bluefire/fire/data/fire.staging.db \
  --base-date 2026-05-21 --max-candidates 10 \
  --scale-categories core_and_large \
  --output-json /tmp/fire_realbatch_staging/f111_real_batch_2026-05-21.json
```

結果: candidates=10 / eligible=5 / exclusions=`{sample:0, tradable_false:0,
risk_above:5, risk_estimate_missing:0}`

| ticker | name | close | risk | risk_within |
|---|---|---|---|---|
| 1925 | 大和ハウス工業 | 4,773 | 23,865 | False (超過) |
| 1928 | 積水ハウス | 3,407 | 17,035 | False |
| 2502 | アサヒグループHD | 1,529 | 7,645 | **True** ✓ |
| 2503 | キリンHD | 2,497 | 12,485 | **True** ✓ |
| 2802 | 味の素 | 5,030 | 25,150 | False |
| 2914 | JT | 5,769 | 28,845 | False |
| 3382 | セブン&アイHD | 1,899 | 9,495 | **True** ✓ |
| 3407 | 旭化成 | 1,537 | 7,685 | **True** ✓ |
| 4063 | 信越化学工業 | 7,480 | 37,400 | False |
| 4188 | 三菱ケミカル G | 922 | 4,610 | **True** ✓ |

→ **eligible 5 件 (= 2502 / 2503 / 3382 / 3407 / 4188)** が pilot 候補。

### §4.2 Step 2: F062 → AFTER-R1 chain

```bash
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all --base-date 2026-05-21 \
  --f062-preview-json /tmp/fire_realbatch_staging/f111_real_batch_2026-05-21.json \
  --data-r3-freshness-json <fresh.json> \
  --output-dir /tmp/fire_realbatch_staging/after_r1
```

結果:
- **artifact_source = `f062_preview`** ✓
- **f062_raw_kind = `f062_actual_dict`** ✓
- **f111_input_source = `f111_real_batch`** ✓ (= 自動推定)
- ranking_size = 10
- 4 file × (JSON + Markdown) = 8 file 生成

morning_line_material:
- 全候補 `research_advisory_label = neutral` のため top_candidates 0 件
  (= TOP_INCLUDE_LABELS = {boost, boost_with_caution, boost_with_avoid, caution})
- これは正しい挙動 (= staging real_batch は research_advisory 未統合のため
  neutral)
- 実 trade 候補にするには **research_advisory 統合 wave** が次に必要 (= 別 wave)

## §5 作成 / 更新 file 一覧

### §5.1 新規 (3 件)

| path | 内容 |
|---|---|
| scripts/jobs/run_f111_real_batch_staging.py | ~360 行 staging read-only runner |
| tests/scripts/jobs/test_run_f111_real_batch_staging.py | 37 件 regression |
| fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_STAGING_{plan,results}.md | 本 wave docs |

### §5.2 更新 (1 件)

| path | 変更 |
|---|---|
| tests/scripts/test_db_path_consistency.py | allow list に `run_f111_real_batch_staging.py` 追加 (= staging-only refuse guard で fire.db literal を持つため) |

### §5.3 FIRE 本体 (= F111 / F062 / F282 / DATA-R3 / AFTER-R1 MVP) 不触 ✓

W60-F111-real-batch の AFTER-R1 MVP に既存追加した F111_REAL_BATCH_MARKERS
推定 logic + 9th invariant 強制 が本 wave の staging output で完全動作。
AFTER-R1 本体は今回不触。

## §6 tests 結果

| wave | collected |
|---|---|
| W60-F111-real-batch | 4650 |
| **W60-F111-real-batch-staging** | **4687** (= +37 件) |

全 **4687 PASS** ✓

### 新規 W60-F111-real-batch-staging tests 37 件

- TestConstants 5 件 (PILOT_PER_TRADE_RISK_LIMIT_YEN / STOP_LOSS_RATIO /
  SAMPLE_NAME_PREFIXES / DEFAULT_SCALE_CATEGORIES / FORBIDDEN_DB_BASENAMES)
- TestDbPathGuard 3 件 (staging allow / production refuse / develop refuse)
- TestOutputPathGuard 3 件 (tmp allow / data refuse / relative refuse)
- TestReadOnlyConnection 2 件 (URI mode=ro で write 不可確認 / production refuse)
- TestTickerNormalize 3 件 (5 桁 trailing 0 → 4 桁 / 5 桁 末尾非 0 そのまま / 4 桁 passthrough)
- TestSampleDetection 3 件
- TestRiskEstimate 4 件 (close → risk / 値嵩株上限超過 / None / 0)
- TestFetchCandidates 4 件 (real ticker 抽出 / risk_within 判定 / tradable
  default / F111_REAL_BATCH_MARKERS 含む)
- TestArtifactBuild 2 件 (artifact 構造 / eligible_count)
- TestCliMain 3 件 (smoke / production refuse / unsafe output refuse)
- TestAstSafety 5 件 (no linebot / no requests/aiohttp/urllib.request /
  no subprocess / sqlite URI ?mode=ro 使用 / SAFETY_FLAGS immutable)

## §7 Codex 4 lane stdin audit 結果

**全 4 lane 完全 reply 取得、LOW No finding 全件 ✓**:

| Lane | 観点 | 結果 |
|---|---|---|
| A | is_safe_db_path / open_staging_readonly URI mode=ro | LOW Confirmed ✓ |
| B | fetch_candidates sector/scale filter / normalize_jp_ticker_code | LOW Confirmed ✓ |
| C | PILOT_PER_TRADE_RISK_LIMIT_YEN / STOP_LOSS_RATIO / estimate_100_share_risk_yen | LOW Confirmed ✓ |
| D | F111_REAL_BATCH_MARKERS / F062 handoff schema | LOW Confirmed (= 軽微 note: source duplicate) ✓ |

### §7.1 短文 prompt 戦略の効果 (= 累積実証)

| wave | parallel | 完全 reply 率 |
|---|---|---|
| W60.5/W60.6/W60-launchd-pre (= 4 lane short JSON read) | 100% |
| W60-pilot-pre (= 8 lane) | 37.5% |
| W60-integration (= 6 lane + sleep) | 66.7% |
| W60-F111-pre (= 4 lane long prompts) | 0% |
| W60-pilot-W1 (= 4 lane ≤100 words) | 25% |
| W60-F111-real-batch (= 4 lane ≤80 words factual confirm) | 75% |
| **W60-F111-real-batch-staging (= 4 lane ≤80 words factual confirm)** | **100%** ★ |

→ **factual confirm + ≤80 words = 100% 安定** が再現実証。Codex 並列度戦略確定。

## §8 安全確認

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
| 新規 import (= linebot/requests/aiohttp/subprocess 等) | 0 |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4650 → 4687 (= +37) |

## §9 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **4/4 = 100%** (= 短文 factual confirm + 単一観点 戦略確立) |
| 短縮率 | 高 (= 4 観点 並列カバー) |
| 採用率 | 100% (= LOW No finding 全件、設計通り実装) |
| 差戻率 | 0 |
| Integrator 負荷 | 中-高 (= 360 行 runner + 37 regression + chain smoke + docs + Codex 4 lane) |
| 安全事故 | **0** ✓ |

## §10 残課題 / 次 Wave

### §10.1 W60-F111-real-batch-staging で開ける道

- D6 以降 = 実在 JP 株 ticker (= 大型株 Core30+Large70) で trade plan 作成可能
- 値嵩株 (= 信越化学 / JT / 味の素 / 大和ハウス 等) は **構造的に top から自動除外**
- pilot 損失上限内 ticker (= キリン / 三菱ケミカル / アサヒ / 旭化成 / セブン&アイ /
  等) が top 候補

### §10.2 次 wave 候補

1. **research_advisory 統合** (= 別 wave): staging から research_advisory_label
   を取得、neutral 以外の **boost/caution 系 label を実在 ticker に紐付け** →
   morning_line_material top に出る → 実 trade 可能候補
2. **W60-pilot-D6**: 上記統合後の最初の trade plan
3. **W61-pre**: price/return/paper_pnl 連携
4. **W60-launchd-real**: 本番 launchd 配置 (= Wave 41/45 後)

### §10.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §10.4 D6 期待 caveat 強度

| wave | f111_input_source | caveat |
|---|---|---|
| D3/D4 | manual_seed | 強 |
| D5 | f111_sample | 中 |
| **D6 (= F111-real-batch-staging 後)** | **f111_real_batch** ✓ | **弱** (= staging real ticker / research_advisory 未統合) |
| 将来 | f111_real_batch + research_advisory 統合 | 最弱 (= 真の本番 path) |

## §11 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_STAGING_plan]]
- [[FIRE_CODEX_R1_WAVE60_F111_REAL_BATCH_results|W60-F111-real-batch (= safe fixture)]]
- [[../03_design/F111_real_batch_requirements_2026-05-14|F111-real-batch 要件 v1.0]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_W1_results|W1 集約 results]]
