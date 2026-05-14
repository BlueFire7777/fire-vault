---
id: FIRE-CODEX-R1-WAVE60-F111-DAILY-REFRESH-results
phase: 本番 v0 中核 / Wave 60-F111-daily-refresh / staging signal 日次更新
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
hq_marker: HQ_APPROVE_STAGING_RESEARCH_SIGNAL_REFRESH=1
classification: A
---

# Wave 60-F111-daily-refresh Results — staging signal daily refresh + D8 chain

## §1 結論

**形式的 refresh: 成功 ✓ / 実質的 refresh: 限定的**

staging research_watchlist_signals に base_date=2026-05-13 / source_version=
w60_f111_daily_v1 の 35 row を新規 INSERT。production-develop unchanged。
ただし staging features は前 cap (2026-05-12) のまま律速されるため、新 signal
の top 8 は base_date=2026-05-12 と **完全同一** (final_score 全部 1e-12
まで一致、post_cap_rank 1-8 一致)。

→ **D8 候補 ≒ D6/D7 候補** (87470/57290/34890)、features rerun が真の更新
には別途必要 (= 次 wave 候補)。

## §2 経路 classification

| 観点 | 値 |
|---|---|
| Runner | `scripts/jobs/run_research_watchlist_signal_persistence.py` (568 行) |
| API import (requests/aiohttp/urllib/httpx) | 0 |
| 内部 deps API import | 0 (simulation.research_lane.*) |
| --write guard | 4 段 (label refuse + filename verify + FIRE_ENV=staging) |
| HQ marker | HQ_APPROVE_STAGING_RESEARCH_SIGNAL_REFRESH=1 |
| 既存衝突 | base_date=2026-05-13 staging 0 件 / source_version w60_f111_daily_v1 staging 0 件 |
| **分類** | **A** (staging-only write、token/API/network 不要、HQ marker 充足) |

## §3 実行 step ごと結果

### Step 1: dry-run smoke

```
$ FIRE_ENV=staging .venv/bin/python -m scripts.jobs.run_research_watchlist_signal_persistence \
    --db staging --base-date 2026-05-13 --source-version r2g3_recommended_v2 \
    --top-n 30 --output-json /tmp/fire_f111_refresh/dryrun_signal.json
```

- records loaded: 3708
- rows after top_n filter: 35 (= top_n=30 + sector cap 5)
- mode: DRY-RUN / write_enabled: False
- row_count: 13695 → 13695 (不変) ✓
- constraints_check: staging_only_write=True / production_db_untouched=True /
  develop_db_untouched=True / write_guard_active=True / atomic_transaction=True

### Step 2: production/develop md5 baseline

```
fire.db        : b1df4673e5c3645fbe2c5f490ffac043
fire.develop.db: 0eed4ad2ec7ed2edf8f640d97341c5ad
```

### Step 3: staging write smoke

```
$ FIRE_ENV=staging HQ_APPROVE_STAGING_RESEARCH_SIGNAL_REFRESH=1 \
  .venv/bin/python -m scripts.jobs.run_research_watchlist_signal_persistence \
    --db staging --base-date 2026-05-13 --source-version w60_f111_daily_v1 \
    --top-n 30 --write --output-json /tmp/fire_f111_refresh/write_signal.json
```

- inserted: 35 / replaced: 0 / skipped: 0 / failed: 0 ✓
- row_count: 13695 → 13730 (+35) ✓
- write_enabled: True

### Step 4: production/develop md5 post-write 検証

```
fire.db        : b1df4673e5c3645fbe2c5f490ffac043 (unchanged ✓)
fire.develop.db: 0eed4ad2ec7ed2edf8f640d97341c5ad (unchanged ✓)
```

### Step 5: 新 signal top 8 確認

| rank | code | final_score | label | dec |
|---|---|---|---|---|
| 1 | 87470 | 0.9298885895446503 | A1 | selected |
| 2 | 57290 | 0.9151622168418339 | A1 | selected |
| 3 | 34890 | 0.9079741603926976 | A1 | selected |
| 4 | 340A0 | 0.8922434919929426 | A1 | selected |
| 5 | 37980 | 0.8765130217865811 | A1 | selected |
| 6 | 137A0 | 0.8685089966320712 | A1 | selected |
| 7 | 79910 | 0.8668992665795182 | A1 | selected |
| 8 | 91300 | 0.8586158472438529 | A1 | selected |

→ base_date=2026-05-12 と **完全同一** (binary-level)

### Step 6: D8 (2026-05-25) F111-real-batch chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-05-25 --max-candidates 10 \
    --output-json /tmp/fire_f111_refresh/d8_f111_real_batch_2026-05-25.json
```

- candidates: 10 / eligible: 9
- exclusions: risk_above_pilot_limit=1 (137A0) + risk_estimate_missing=1
- artifact: /tmp/fire_f111_refresh/d8_f111_real_batch_2026-05-25.json

### Step 7: D8 top candidates

| # | code | name | adv | risk_within |
|---|---|---|---|---|
| 1 | 8747 | 豊トラスティ証券 | boost | True |
| 2 | 5729 | 日本精鉱 | boost | True |
| 3 | 3489 | フェイスネットワーク | boost | True |
| 4 | 340A0 | ジグザグ | boost | True |
| 5 | 3798 ＵＬＳグループ | boost | True |
| 6 | 137A0 | Ｃｏｃｏｌｉｖｅ | boost | **False** (= excluded) |
| 7 | 7991 | マミヤ・オーピー | boost | True |
| 8 | 9130 | 共栄タンカー | boost | True |
| 9 | 331A0 | メディックス | boost | True |
| 10 | 4389 | プロパティデータバンク | boost | True |

## §4 D6/D7/D8 overlap

| day | top 3 |
|---|---|
| D6 (2026-05-21) | 8747 / 5729 / 3489 |
| D7 (2026-05-22) | 8747 / 5729 / 3489 |
| **D8 (2026-05-25)** | **8747 / 5729 / 3489** |

→ **100% overlap 継続** (= staging features cap 律速)

## §5 効果評価

| 観点 | 評価 |
|---|---|
| staging-only write | ✓ |
| production/develop md5 不変 | ✓ |
| 既存 13695 row 完全保護 | ✓ (replaced=0) |
| 新 base_date 列追加 (= 鮮度 watermark) | ✓ |
| F111-real-batch chain 動作 (= 新 base_date を JOIN) | ✓ |
| top candidate 内容更新 | ✗ (features cap 律速) |
| **真の意味の "日次更新"** | **△** (= 形式的 OK、実質要 features rerun) |

## §6 D8 推奨

D6 review が 2026-05-14 14:45 時点で **blank** (= 未記入)。staging signal
形式的 refresh は完了したが top 3 同候補。D8 推奨判定は D7 と同方針:

- **HOLD 推奨** (= 月曜 D8 = 2026-05-25 朝寄付き)
- 理由: D6 review missing + D6/D7/D8 top 3 100% overlap + features cap 律速
- ただし D6/D7 で skip した場合 → D8 で entry 評価可
- 同候補で 3 営業日 (D6/D7/D8) 連続 = signal 安定性確認の意味あり

## §7 D8 alternative path (= features rerun を待たない場合)

| option | 説明 |
|---|---|
| A. HOLD 継続 | D6 review 完了 + features rerun wave 着手まで pilot pause |
| B. 同候補で entry 試行 | 8747/5729/3489 のいずれかで実 entry (= signal 安定性確認) |
| C. liquidity filter 強化 | 板厚/スプレッド actual で 3 候補絞込み |
| D. features rerun wave 着手 | 別 wave で staging features を最新 J-Quants で update (= API 必要、別 HQ approve) |

**推奨: A + D** (= HOLD で D6 review + features rerun を待ち、D9 以降で新候補へ)

## §8 9 hard invariants 遵守

| invariant | D8 chain 結果 |
|---|---|
| artifact_source=f062_preview | (F111 raw raw、AFTER-R1 通過時に付与) |
| f062_raw_kind=f062_actual_dict | (AFTER-R1 通過時) |
| f111_input_source=f111_real_batch | ✓ (per candidate に付与) |
| freshness_verdict=OK | (AFTER-R1 通過時) |
| forbidden_check.passed=True | (AFTER-R1 通過時) |
| auto_order_allowed=False | ✓ (F111 raw に明示) |
| manual_review_required=True | ✓ (F111 raw に明示) |
| safety_flags 13 keys all False | (AFTER-R1 通過時) |
| top_candidates count ≥1 | ✓ (9 eligible) |

## §9 安全境界 final verification

- production/develop md5: 不変 ✓
- staging write: 35 row insert / 既存 13695 完全保護 ✓
- LINE 0 / token 0 / API 0 / launchctl 0 / plist 0 / cron 0 ✓
- 実発注 0 / 楽天 0 / iSPEED 0 / Computer Use 0 ✓
- F282 plist 不変 (1778593597/1772) ✓
- pytest 4705 collected 不変 ✓

## §10 Next action 候補

- ☐ D6 review 記入 (= Fujiwara 本人) → D7/D8 trade plan の review-aware 更新
- ☐ features rerun wave (= staging features を最新 J-Quants で update、API 必要、別 HQ approve)
- ☐ D8 (= 2026-05-25 月) HOLD or 同候補 entry の最終判断
- ☐ liquidity filter 強化 wave (= 板厚/spread actual)
- ☐ W60-pilot-D8 plan + review template 作成
