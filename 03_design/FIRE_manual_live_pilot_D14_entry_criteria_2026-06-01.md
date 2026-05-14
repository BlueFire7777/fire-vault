---
id: FIRE-design-manual-live-pilot-D14-entry-criteria-2026-06-01
phase: 本番 v0 中核 / Wave 60-pilot-W2 集約成果
priority: 高
status: design-final
owner: BlueFire7777 (Fujiwara)
date: 2026-06-01 (= D14 想定)
related_waves:
  - Wave 60-pilot-D9 〜 D13 (= 5 day GO_CONDITIONAL 連続)
  - Wave 61-impl (= paper PnL preview runner)
---

# FIRE Manual Live Pilot — D14 Entry Criteria Design v1.0

## §1 目的

D9-D13 で **GO_CONDITIONAL が 5 連続** 維持。本 design doc は D14 で **初回少額
実弾** に進めるための GO/GO_CONDITIONAL/HOLD/NO-GO 判定基準を明文化する。

## §2 D9-D13 lessons

- 候補生成 chain (F111 → F062 → AFTER-R1 → paper_pnl_preview) は安定稼働 ✓
- 340A0/3798/137A0 が 5 営業日連続 top 4-6 (= signal 安定)
- features cap 律速で同 candidates (= 真の更新には features rerun + API refresh が必要)
- review 5 連続 blank (= Fujiwara 未記入、構造的問題)
- DATA-R3 freshness 接続 path 確認 ✓、ただし実 verdict=OK は別 wave

→ 候補と chain は十分、**Fujiwara 自身が「初回 entry」の引き金を引く」段階**。

## §3 D14 entry criteria (= 4 段階)

### 3.1 GO 条件 (= 12 項目全 ✓ 必要)

| # | 条件 | 検証方法 |
|---|---|---|
| 1 | f111_input_source=f111_real_batch | F111 artifact 確認 |
| 2 | top candidate ≥ 1 | morning_line_material |
| 3 | risk_within_pilot_limit=True | F111 candidate field |
| 4 | sample でない (= name に サンプル/_template_/TEST_/DUMMY_ 含まない) | F111 sample exclusion |
| 5 | tradable_universe=True | F111 candidate field |
| 6 | forbidden_phrases_check.passed=True | morning_line_material |
| 7 | auto_order_allowed=False (= 構造的禁止) | morning_line_material |
| 8 | manual_review_required=True (= 構造的) | morning_line_material |
| 9 | safety_flags 13 keys all False | morning_line_material |
| 10 | **actual price confirmation passed** | iSPEED で actual 確認、±5% 以内 |
| 11 | **liquidity/spread/volume 確認 passed** | 板厚≥5,000株、出来高≥5,000株、spread≤0.5% |
| 12 | **event/earnings 確認 passed** | TDnet 確認、ネガティブニュース無し、決算予定外 |

**追加 require**:
- review missing が許容範囲 = D14 review.md に最低限 §1 (基本情報) + §11 (final decision) を記入予定

### 3.2 GO_CONDITIONAL 条件 (= GO の 12 項目のうち 9 項目以上 ✓ + 残り 1-3 項目 caveat)

許容される caveat:
- DATA-R3 freshness_verdict=MISSING (= 接続経路 OK、実 verdict=OK は別 wave)
- price refresh 未実施で actual confirmation 待ち
- D10-D13 review missing が残る (= D14 review で初記入推奨)
- sector 集中 (= 情報通信 100%)、ただし sector 多様化 candidate (= 7991) も併記

→ Fujiwara が朝寄付き前 3 確認 (= actual + liquidity + event) を実行後、全 ✓ で entry。

### 3.3 HOLD 条件 (= いずれか 1 つ ✓ で HOLD)

| # | 条件 | 判定根拠 |
|---|---|---|
| 1 | review missing が 5 連続超過 (= D9-D14 全 blank で 6 連続) | 学習反映不可、Fujiwara 確認必須 |
| 2 | 同 candidate が 7 営業日以上連続 (= signal 安定 vs 真の更新無しの臨界) | features rerun wave 必要 |
| 3 | actual price 確認できない (= iSPEED 接続不可 / 市場休場) | 朝 D-day 中止 |
| 4 | liquidity 確認できない (= 板気配無し / 出来高 0) | 寄付き未成立 |
| 5 | event risk 強い (= TDnet ネガ開示 / 決算予定 / 不祥事) | 個別銘柄 skip |

### 3.4 NO-GO 条件 (= いずれか 1 つ ✓ で NO-GO)

| # | 条件 | 判定根拠 |
|---|---|---|
| 1 | top_candidates count = 0 | F111 fallback も無し |
| 2 | f111_input_source ≠ f111_real_batch | synthetic / manual_seed 混入 |
| 3 | 全 candidate で risk_within_pilot_limit=False | 全銘柄 risk 上限超え |
| 4 | sample ticker (= サンプル/_template_/TEST_/DUMMY_) | F111 sample exclusion 失敗 |
| 5 | forbidden_check failed (= 命令形 / 強制表現混入) | LINE 発射絶対禁止 |
| 6 | safety_flags のいずれか True (= db_write / line_send / token_access / api_call / launchctl_call / order_automation 等) | 構造的安全違反 |
| 7 | price freshness 不明 + actual confirm 不可 (= 二重不明) | リスク判断材料無し |
| 8 | 監視不能 (= 市場休場以外で iSPEED 接続不能) | exit 手段無し |

## §4 D14 候補引き継ぎ

### 4.1 D14 で見る候補 (= F111-real-batch + research enriched chain)

```
推奨 cmd:
.venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
  --base-date 2026-06-02 --max-candidates 20 \
  --label-threshold-mode strict \
  --recently-seen-codes 8747,5729,3489 \
  --output-json /tmp/fire_d14_prep/d14_f111_real_batch.json
```

引き継ぎ候補 (= D13 と同じ高確率):

| rank | code | name | sector | close (5/14) | risk_yen | 推奨 |
|---|---|---|---|---|---|---|
| 4 | 340A0 | ジグザグ | 情報通信 | 380 | 1,900 | **★ #1 候補** |
| 5 | 3798 | ＵＬＳグループ | 情報通信 | 505 | 2,525 | #2 |
| 6 | 137A0 | Cocolive | 情報通信 | 739 | 3,695 | #3 |
| 7 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 | **#4 (= sector 多様化)** |

### 4.2 D14 で必ず確認する項目 (= 朝寄付き 08:30-09:00 JST)

1. **朝 J-Quants refresh** (= staging max → 5/29 まで)
   ```
   set -a; source /Users/bluefire/fire/.env; set +a
   FIRE_ENV=staging \
   HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1 \
   HQ_APPROVE_JQUANTS_TOKEN_READ=1 \
   .venv/bin/python -m scripts.jobs.run_jquants_daily_refresh \
     --db-path /Users/bluefire/fire/data/fire.staging.db --db-label staging \
     --datasets prices --max-days 14 \
     --symbols-csv /tmp/fire_d10_prep/d9_symbols.csv \
     --sleep-seconds 0.3 --write \
     --output-json /tmp/d14_jq_refresh.json
   ```

2. **iSPEED で actual price** (= 340A0 / 3798 / 137A0 / 7991 の 4 銘柄)
3. **板厚 5 本目処 + 寄付き 5 分出来高 + spread**
4. **TDnet 開示 + 決算予定 + ニュース**
5. **F111-real-batch + F062 + AFTER-R1 + paper PnL chain 再 run**

### 4.3 D14 GO 判定 quick check

- 全 12 項目 ✓ → **GO** (= 340A0 100 株 entry、resources 38,000 円、max loss 1,900 円)
- 9 項目以上 ✓ + 残り caveat → **GO_CONDITIONAL** (= D-day 初日 OK、ただし caveat 認識)
- HOLD 条件 1 個以上 ✓ → **HOLD** (= D15 持ち越し)
- NO-GO 条件 1 個以上 ✓ → **NO-GO** (= chain 再起 or 別 wave)

## §5 D14 review 必須記入項目 (= 最低限 5 つ)

D10-D13 で 4 review blank が続いた構造的問題への対応:

1. **§1 基本情報 entry 銘柄 + 記入時刻** (= 必ず記入)
2. **§2 計画 vs 実際 entry 価格 / 株数 / 時刻** (= entry した場合)
3. **§3 PnL 総 PnL** (= entry/skip いずれも円単位 or "skip" 記入)
4. **§5 Reason for entry / skip** (= 1 文以上)
5. **§11 (or §8) final decision の checkbox 選択** (= enter / watch / skip 1 つ)

これだけは記入を **強く推奨** (= 完全自由文ではなく、項目固定)。
記入率が D14 で改善すれば D15 以降の pattern_outcomes 評価が初めて意味を持つ。

## §6 paper PnL handoff (= D14 場後 + D15 朝 + h20 後)

### 6.1 D14 場後 (= 15:10 close 後)

```bash
.venv/bin/python -m scripts.jobs.run_after_r1_paper_pnl_preview \
  --base-date 2026-06-02 --evaluation-date 2026-06-02 \
  --f111-real-batch-json /tmp/fire_d14_prep/d14_f111_real_batch.json \
  --morning-line-material-json /tmp/fire_d14_prep/after_r1/morning_line_material_2026-06-02.json \
  --review-md ~/fire-vault/04_daily/2026-06-02_manual_live_pilot_review.md \
  --output-json /tmp/fire_d14_prep/d14_paper_pnl_ledger_eod.json
```

### 6.2 D15 朝 (= 翌営業日朝 refresh 後)

D14 entry 銘柄について h1 close 確認。

### 6.3 h20 後 (= 2026-06-26〜30 頃)

20 営業日後 outcome 判定で **真の win/loss/flat** 評価。pattern_outcomes が
初めて意味を持つタイミング。

## §7 D9-D13 で確立済の運用前提 (= D14 でも維持)

- F111-real-batch staging chain
- strict + recently_seen + max=20 設定
- recently_seen=[8747, 5729, 3489] 維持
- 100 株 entry / 5% stop_loss / 2% TP / 1 トレード上限 15,000 円
- 15:10 完全クローズ / ナンピンなし / 持ち越しなし
- 楽天 / iSPEED で手動発注 / Computer Use 不採用
- 最終判断は Fujiwara 本人
