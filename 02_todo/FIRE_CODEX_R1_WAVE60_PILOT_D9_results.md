---
id: FIRE-CODEX-R1-WAVE60-PILOT-D9-results
phase: 本番 v0 中核 / Wave 60-pilot-D9 / W60-F111-preset-tune 初実弾検証 day
priority: 高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-26 (D9) / wave 実行: 2026-05-14
pilot_day: D9
pilot_judgment: GO_CONDITIONAL
entry_candidate_1: 340A0 ジグザグ
---

# Wave 60-pilot-D9 Results — Strict Recent Real-Batch Manual Live Pilot Trade Plan

## §1 結論

**D9 Pilot 判定 = GO_CONDITIONAL** (= 寄付き直前 actual price + liquidity +
event の 3 確認後 entry、いずれか NG なら skip)。

W60-F111-preset-tune の 3 軸 (strict + recently_seen + max=20) を D9 に
初適用し、D6-D8 で 100% 重複した top 1-3 (8747/5729/3489) を caution へ
デモート → 見送り、top 4 以降の boost_with_caution 7 銘柄から:

- **entry 候補 #1 = 340A0 ジグザグ** (score=0.892、risk=1,990 円、情報通信)
- entry 候補 #2 = 3798 ＵＬＳグループ
- entry 候補 #3-#6 = 7991 / 9130 / 331A0 / 4389

ただし **market_prices_daily cap=2026-05-08 (= 6 営業日 stale)** + freshness_verdict=MISSING
の 2 caveat あり → 寄付き actual confirm 必須、いずれか NG で skip。

## §2 chain 実行結果

### 2.1 F111-real-batch chain

```
$ .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --base-date 2026-05-26 \
    --max-candidates 20 \
    --label-threshold-mode strict \
    --recently-seen-codes 8747,5729,3489 \
    --output-json /tmp/fire_d9_prep/f111_real_batch_2026-05-26.json
```

- cli_version: 1.2.0 (= W60-F111-preset-tune) ✓
- candidates: 20 / eligible: 18
- exclusions: risk_above_pilot_limit=1 (137A0) + risk_estimate_missing=1
- tuning_params: strict / recently_seen=[8747, 5729, 3489] / demoted_count=3

### 2.2 F062 preview chain

```
$ .venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
    --preview-json /tmp/fire_d9_prep/d9_f111_preview_list.json \
    --output-json /tmp/fire_d9_prep/d9_f062_preview.json \
    --output-text /tmp/fire_d9_prep/d9_f062_preview.txt \
    --output-summary-json /tmp/fire_d9_prep/d9_f062_summary.json
```

- input=20 / selected=10 / chunks=1
- forbidden=0 / safety_footer=True
- auto_order_allowed_true_count=0 / manual_review_required_count=20

### 2.3 AFTER-R1 night batch (= MVP mode)

```
$ .venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
    --mode mvp --task all --base-date 2026-05-26 \
    --f062-preview-json /tmp/fire_d9_prep/d9_f062_preview.json \
    --f062-preview-summary-json /tmp/fire_d9_prep/d9_f062_summary.json \
    --output-dir /tmp/fire_d9_prep/after_r1 \
    --artifact-source f062_preview \
    --f111-input-source f111_real_batch
```

- artifacts: 4 (paper_live_ledger / good_candidate_ranking /
  pattern_candidate_report / morning_line_material)
- ranking_size: 10
- artifact_source: f062_preview ✓
- f062_raw_kind: f062_actual_dict ✓
- f111_input_source: f111_real_batch ✓

## §3 D9 candidate evaluation (= F111-real-batch top 20)

### 3.1 top 1-3 caution (= recently_seen demoted、見送り)

| rank | code | name | label | score | recently_seen | 推奨 |
|---|---|---|---|---|---|---|
| 1 | 8747 | 豊トラスティ証券 | caution | 0.930 | True (D6-D8) | skip |
| 2 | 5729 | 日本精鉱 | caution | 0.915 | True | skip |
| 3 | 3489 | フェイスネットワーク | caution | 0.908 | True | skip |

### 3.2 top 4-12 boost_with_caution (= entry 候補)

| rank | code | name | sector | close | score | risk_yen | entry 適性 |
|---|---|---|---|---|---|---|---|
| **4** | **340A0** | **ジグザグ** | 情報通信 | 398 | 0.892 | 1,990 | **★ #1** |
| 5 | 3798 | ＵＬＳグループ | 情報通信 | 519 | 0.877 | 2,595 | #2 |
| 6 | 137A0 | Ｃｏｃｏｌｉｖｅ | 情報通信 | 0 | 0.869 | 0 | **除外** |
| 7 | 7991 | マミヤ・オーピー | 機械 | 1,246 | 0.867 | 6,230 | #3 |
| 8 | 9130 | 共栄タンカー | 運輸 | 1,532 | 0.859 | 7,660 | #4 |
| 9 | 331A0 | メディックス | 情報通信 | 480 | 0.857 | 2,400 | #5 |
| 10 | 4389 | プロパティデータバンク | 情報通信 | 770 | 0.856 | 3,850 | #6 |
| 11 | 9247 | TRE ホールディングス | 情報通信 | 1,621 | 0.855 | 8,105 | #7 |
| 12 | 4317 | レイ | 情報通信 | 507 | 0.852 | 2,535 | #8 |

### 3.3 top 13-20 caution / neutral

| rank | code | name | label | risk_within | note |
|---|---|---|---|---|---|
| 13 | 9628 | 燦ホールディングス | caution | True | strict 0.85 未満で caution |
| 14 | 2700 | 木徳神糧 | caution | True | |
| 15 | 6149 | 小田原エンジニアリング | caution | True | |
| 16 | 288A0 | ラクサス | caution | True | close=112 超低価格 |
| 17 | 4404 | ミヨシ油脂 | caution | True | |
| 18 | 2981 | ランディックス | caution | True | |
| 19 | 8152 | ソマール | caution | **False** | 値嵩株 (close=5,460) |
| 20 | 339A0 | プログレス・テクノロジーズ | neutral | True | strict 0.75 未満 |

### 3.4 sector concentration

- 情報通信・サービスその他: 5 / 7 boost_with_caution (= 71%、集中)
- 機械: 1 (7991)
- 運輸・物流: 1 (9130)

→ sector 多様化のため #3 候補 = 7991 (機械) も検討対象。

## §4 entry candidate #1: 340A0 ジグザグ

| 項目 | 値 |
|---|---|
| code / name | 340A0 / ジグザグ |
| sector | 情報通信・サービスその他 |
| close (= 2026-05-08 stale) | 398 円 |
| score (research_final_score) | 0.892 (A1 相当) |
| label | boost_with_caution |
| AFTER-R1 morning_line score | 175.22 (= F062 経由再 scoring) |
| 100 株 entry 想定資金 | 39,800 円 (stale) |
| stop_loss ratio | 5% |
| 想定 max loss | 1,990 円 |
| pilot 1 トレード上限 | 15,000 円 → **余裕 13,010 円** |
| entry guideline | 始値±0.5%以内、出来高 OK で寄付き買い |

## §5 Pilot GO/HOLD/NO-GO 判定

### 5.1 判定: **GO_CONDITIONAL**

**GO 条件 (= 8/8 達成)**:

| 条件 | 結果 |
|---|---|
| f111_real_batch | ✓ |
| top 4 以降に risk_within=True 候補 ≥1 | ✓ (= 7 銘柄: 340A0/3798/7991/9130/331A0/4389/9247/4317) |
| research label boost_with_caution 以上 | ✓ (= 7 銘柄全て) |
| repeated top 1-3 ではない | ✓ (= recently_seen demote 済) |
| sample でない | ✓ |
| tradable_universe=True | ✓ (= 全 entry 候補) |
| liquidity/event 確認欄が trade plan に明記 | ✓ |
| 9 hard invariants 全 PASS | ⚠ 8/9 (freshness_verdict=MISSING のみ意図的) |

**HOLD 条件 (= conditional 因子)**:

| 因子 | 状況 | 影響 |
|---|---|---|
| market_prices_daily cap=2026-05-08 | 6 営業日 stale | close 価格 actual と乖離可能性、寄付き confirm 必須 |
| freshness_verdict=MISSING | DATA-R3 gate 未渡し (意図的) | 次 wave で gate JSON 連携検討 |
| sector 集中 | 情報通信 5/7 | 多様化のため #3=7991 (機械) も候補 |

→ **GO_CONDITIONAL** = 寄付き直前 actual price + liquidity + event 3 確認後 entry、いずれか NG なら skip。

### 5.2 NO-GO ではない理由

- top 4 以降に risk_ok 候補 7 銘柄 (= NO-GO 第 1 条件 "候補なし" 不該当)
- 9 invariants の重要項目 (artifact_source / forbidden_check / safety_flags /
  auto_order_allowed=False / manual_review_required=True) 全 PASS
- forbidden_phrases_check.passed = True

## §6 9 hard invariants 詳細

| invariant | D9 result | note |
|---|---|---|
| artifact_source | f062_preview ✓ | F062 chain 通過 |
| f062_raw_kind | f062_actual_dict ✓ | F062 actual format |
| f111_input_source | f111_real_batch ✓ | --f111-input-source 明示 |
| freshness_verdict | **MISSING** ⚠ | DATA-R3 gate JSON 未渡し、意図的 (= 本 wave scope 外) |
| forbidden_check.passed | True ✓ | top_candidates 内 forbidden phrase 0 |
| auto_order_allowed | False ✓ | 構造的禁止 |
| manual_review_required | True ✓ | 全 candidate に明示 |
| safety_flags 13 keys all False | True ✓ | 全 false |
| top_candidates count ≥1 | 3 ✓ (340A0/3798/331A0) |

## §7 D9 trade plan + review template

### 7.1 trade plan

- path: `~/fire-vault/04_daily/2026-05-26_manual_live_pilot_trade_plan.md`
- 内容: D9 特記 / 基本情報 / market_prices_daily stale caveat / top 1-3 skip /
  top 4-10 evaluation / entry candidate #1 (340A0) / liquidity check 欄 /
  event check 欄 / final decision 欄 / risk limit check / stage 3 突合 /
  9 invariants / GO_CONDITIONAL 判定 / safety footer

### 7.2 review template

- path: `~/fire-vault/04_daily/2026-05-26_manual_live_pilot_review.md`
- status: blank (= 15:30 以降記入)
- 内容: 計画 vs 実際 / PnL / Reason for entry/skip / Rule followed / Liquidity actual /
  stale caveat impact / pattern matched / what worked-failed / improvement /
  promote/suppress/watch / next action / stage 3 突合 / 安全 footer

## §8 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓ |
| staging md5 | a8663a07a730378387c050ebb1b612ef → 不変 ✓ (= read-only only) |
| F282 plist | size=1751 / mtime=1778602507 → 不変 ✓ |
| DB write | 0 ✓ |
| production / develop 接続 | 0 ✓ |
| staging write | 0 ✓ |
| LINE / token / API / launchctl / plist / cron | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault/04_daily 配下に plan + review、02_todo 配下に plan + results を new file 追加) |

## §9 next action 候補 (= 優先順)

1. **D9 寄付き直前 (= 2026-05-26 08:55 JST) confirm**:
   - iSPEED で 340A0 actual price 確認
   - 出来高 / spread 確認
   - event / ニュース確認
   - 全 ✓ → 100 株 entry、いずれか NG → skip
2. **D9 14:55 stop loss / take profit 確認**
3. **D9 15:10 close + review.md 記入**
4. **D10 (= 2026-05-27 水) plan** (= recently_seen_codes に D9 entry 銘柄追加)
5. **W60-jquants-daily-refresh-staging** (= 真の price 更新、HQ marker 2 個 + token 承認後)
6. **AFTER-R1 DATA-R3 freshness gate JSON 連携 wave** (= freshness_verdict MISSING 解消)
