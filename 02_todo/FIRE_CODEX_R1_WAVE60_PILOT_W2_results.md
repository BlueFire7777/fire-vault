---
id: FIRE-CODEX-R1-WAVE60-PILOT-W2-results
phase: 本番 v0 中核 / Wave 60-pilot-W2 / D9-D13 集約 + D14 entry criteria 確定
priority: 最高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14 (実行) / 集約対象 D9-D13 (= 2026-05-26 〜 2026-06-01)
design_doc: ~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
codex_lanes: 4 / 4 YES ✓
next_wave: Wave 60-pilot-D14 (= 初回少額実弾候補 GO 判定 day)
---

# Wave 60-pilot-W2 Results — Manual Live Pilot W2 Aggregation / D14 Entry Criteria

## §1 結論

D9-D13 で **GO_CONDITIONAL 5 連続維持**、340A0/3798/137A0 が **5 営業日連続 top
4-6 選出** で signal 安定確認。5 review 全 blank (= Fujiwara 未記入) + 4 paper
PnL 全 pending (= staging max=5/14 で未来 horizon 無し) は **構造的問題**、ただし
chain は完全稼働。

**D14 entry criteria 確定** (= 4 段階)、初回少額実弾 GO の path 明示。Codex 4 lane 全 YES。

## §2 D9-D13 summary 表

| day | date | pilot_judgment | top 1-3 | entry #1 | review status | paper PnL status |
|---|---|---|---|---|---|---|
| D9 | 2026-05-26 | GO_CONDITIONAL | 340A0/3798/137A0 | 340A0 (close=398 stale) | blank | (未実装) |
| D10 | 2026-05-27 | GO_CONDITIONAL | 340A0/3798/137A0 | 340A0 (close=380) | blank | candidates=20 / evaluated=0 / review_missing=1 |
| D11 | 2026-05-28 | GO_CONDITIONAL | 340A0/3798/137A0 | 340A0 (close=380) | blank | 同 |
| D12 | 2026-05-29 | GO_CONDITIONAL | 340A0/3798/137A0 | 340A0 (close=380) | blank | 同 |
| D13 | 2026-06-01 | GO_CONDITIONAL | 340A0/3798/137A0 | 340A0 (close=380) | blank | 同 |

→ **GO_CONDITIONAL 5 連続 + 5 review blank + 4 paper PnL pending**。

## §3 candidate overlap 分析

### 3.1 340A0 ジグザグ (= 5 連続 top 4)

| day | close | risk_yen | score | label |
|---|---|---|---|---|
| D9 | 398 stale | 1,990 | 0.892 | boost_with_caution |
| D10 | 380 refreshed | 1,900 | 0.892 | boost_with_caution |
| D11 | 380 | 1,900 | 0.892 | boost_with_caution |
| D12 | 380 | 1,900 | 0.892 | boost_with_caution |
| D13 | 380 | 1,900 | 0.892 | boost_with_caution |

→ D10 以降 4 連続で **完全同一** (close/risk/score)。

### 3.2 signal 安定 vs features cap 律速

- **signal 安定** (= entry 候補としての価値): research_final_score=0.892 が 5 日連続
  維持、A1 rank で boost_with_caution。Fujiwara にとって entry 候補性は維持。
- **features cap 律速** (= 新規発掘力の限界): staging market_prices_daily max=5/14、
  features rerun 無い、signal rerun 無い → 当然同候補。

→ **両方真**。D14 で同候補が出ても entry 候補として残す価値あり、ただし真の発掘力
   向上には features rerun + 全銘柄 daily refresh + DATA-R3 active job が必要 (= 別 wave)。

### 3.3 D14 で 340A0/3798/137A0 を残すか?

**残す**。理由:
- signal 安定 (= 5 日連続 score 不変、A1 rank)
- 7991 マミヤ・オーピー (機械) を多様化候補として併記、Fujiwara が sector
  選択肢を持てる
- 同候補連続 = recently_seen に入れる場合は別判断 (= entry した銘柄のみ)

## §4 review missing 整理

### 4.1 D10-D13 全 blank

| day | status | final_decision | actual entry/exit/PnL |
|---|---|---|---|
| D10 | blank | unknown | 全 None |
| D11 | blank | unknown | 全 None |
| D12 | blank | unknown | 全 None |
| D13 | blank | unknown | 全 None |

→ **4 連続 review missing**。FIRE-CODEX-R1 設計 § 6.2 で「捏造禁止」明記、未記入は
   未記入として記録。

### 4.2 D14 で review missing を GO 条件から外すか?

- **完全に外さない** (= 過去 review が無いと pattern_outcomes 評価できない)
- ただし **D14 review に最低限 5 項目記入** を D14 GO 条件に追加 (= 設計 doc §5)
  1. §1 基本情報 entry 銘柄 + 記入時刻
  2. §2 計画 vs 実際 entry 価格 / 株数 / 時刻 (= entry した場合)
  3. §3 PnL 総 PnL (= 円単位 or "skip")
  4. §5 Reason for entry / skip (= 1 文以上)
  5. §11 (or §8) final decision checkbox 選択

→ D14 から記入率改善が必要、D15 以降の pattern_outcomes 評価が初めて意味を持つ。

## §5 paper PnL status 整理

### 5.1 D10-D13 全 pending

| day | candidate_count | evaluated_count | win/loss/flat | review_missing | pattern_outcomes |
|---|---|---|---|---|---|
| D10 | 20 | 0 | 0/0/0 | 1 | 9 種全 watch / evidence=low |
| D11 | 20 | 0 | 0/0/0 | 1 | 同 |
| D12 | 20 | 0 | 0/0/0 | 1 | 同 |
| D13 | 20 | 0 | 0/0/0 | 1 | 同 |

→ **全 pending は設計通り** (= staging max=5/14、未来 horizon 無し)。

### 5.2 D14 判断に使える情報 / まだ使えない情報

| 情報 | D14 で使える? | 注記 |
|---|---|---|
| F111 candidate planned_entry / stop / TP | ✓ | refresh price 5/14 ベース |
| F111 pattern_tags (6 種) | ✓ | refreshed_price_ok / multi_reason etc. |
| AFTER-R1 morning_score | ✓ | F062 経由再 score、175.22 |
| paper PnL h1/h5/h20 outcome | ✗ | 全 pending (= 真の outcome は h20 後) |
| pattern_outcomes promote/suppress | ✗ | sample 不足、全 watch / evidence=low |
| review-based actual PnL | ✗ | 5 review blank |

→ D14 判断には **candidate-level evidence のみ使用可能**。pattern-level evidence
  は h20 後 (= 2026-06-26〜30 頃) の再 run で初めて意味を持つ。

## §6 price freshness / actual confirmation 条件

### 6.1 staging market_prices_daily 状況

- max date: **2026-05-14**
- D14 当日 (= 想定 2026-06-02 火) → **19 営業日 gap**
- 12 銘柄分は前 wave (W60-jquants-daily-refresh-staging) で 5/14 まで catch-up
- 残り 4,436 銘柄は 5/8 cap (= 26 営業日 gap)

### 6.2 D14 GO 条件: actual price confirmation

- 朝 J-Quants refresh (= D14 当日に max-days 14 で 5/14→5/29 catch-up)
- 寄付き直前 (= 08:55 JST) iSPEED で actual price 確認
- 乖離率 ±5% 以内 → entry 検討
- 乖離率 ±5% 超 → skip 強制

→ **actual confirmation passed** が GO 条件 12 項目の 10 番目 (= 設計 doc §3.1)。

## §7 DATA-R3 freshness MISSING 扱い

### 7.1 接続経路確認

- D13 wave で `--data-r3-freshness-json` 引数で渡せた ✓
- AFTER-R1 morning_line_material.freshness_verdict に反映 ✓
- ただし verdict=MISSING のまま (= dry-run + 実 fetch なし)

### 7.2 D14 で MISSING でも GO_CONDITIONAL?

- **YES**。理由:
  - 9 invariants のうち 8 (= freshness 以外) が PASS
  - DATA-R3 active job execution が別 wave (= HQ_APPROVE_FETCH_ACTIVE_JOB 想定)
  - 接続経路 OK = 将来 verdict=OK 達成時に GO 寄りへ自動移行
- **完全 GO にしない理由**: 真の verdict=OK が出ていない、material/announcement
  の新着検出が機能していない可能性

## §8 sector concentration 扱い

### 8.1 D9-D13 top 3 全 sector 100% 集中

| day | top 1 | top 2 | top 3 | sector |
|---|---|---|---|---|
| D9-D13 | 340A0 | 3798 | 137A0 | **全て情報通信・サービスその他** |

### 8.2 D14 で sector 集中を HOLD 要因にするか?

- **HOLD にしない** (= signal 安定 + multi_reason pattern 認識済)
- ただし **caveat として強く明示** + **多様化候補 #4 = 7991 マミヤ・オーピー (機械)**
  を併記 (= 設計 doc §4.1)
- Fujiwara が sector 選択肢を持てる構造を維持

## §9 D14 entry criteria 確定 (= 4 段階)

詳細は `~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md`
§3 に明文化。要約:

| 段階 | 条件 | 推奨アクション |
|---|---|---|
| **GO** | 12 項目全 ✓ (= 9 invariants 8/9 + actual + liquidity + event + review 記入予定) | 340A0 100 株 entry、max loss 1,900 円 |
| **GO_CONDITIONAL** | 9 項目以上 ✓ + 1-3 項目 caveat (= freshness MISSING / price refresh 未 / review missing / sector 集中) | 朝寄付き 3 確認後 entry、いずれか NG → skip |
| **HOLD** | review missing 5 連続超過 / 同 candidate 7 営業日連続 / actual 確認不可 / liquidity 不可 / event リスク | D15 持ち越し |
| **NO-GO** | top=0 / f111_input_source 不正 / risk_within 全 False / sample / forbidden_check failed / safety_flags True / price 二重不明 / 監視不能 | chain 再起 or 別 wave |

## §10 D14 handoff (= 候補 + 確認項目 + paper PnL 手順)

詳細は設計 doc §4-§6 に。要点:

1. **D14 で見る候補**: 340A0/3798/137A0 + sector 多様化 #4 7991
2. **D14 で必ず確認する項目**:
   - 朝 J-Quants refresh (= 5/14→5/29)
   - iSPEED で actual price (= 4 銘柄)
   - 板厚 / 出来高 / spread
   - TDnet 開示 / 決算予定 / ニュース
   - F111 + F062 + AFTER-R1 + paper PnL chain 再 run
3. **D14 review 必須記入** = 最低 5 項目
4. **paper PnL handoff** = 場後 + 翌朝 + h20 後 3 回

## §11 Codex 4 lane factual-confirm

| lane | 観点 | reply |
|---|---|---|
| **A** | D9-D13 candidate overlap (= 340A0/3798/137A0 5 連続、全 GO_CONDITIONAL) | **YES** ✓ — "D9-D13 5 ファイル全てで GO_CONDITIONAL、8747/5729/3489、340A0/3798/137A0 を確認 (D9 の 137A0 は除外表記付き)" |
| **B** | review/PnL (= D10-D13 全 blank、4 ledger 存在 review_missing_count=1) | **YES** ✓ — "D10-D13 review.md are status: blank, and each /tmp/fire_d{10,11,12,13}_prep ledger exists with review_missing_count: 1" |
| **C** | price/freshness (= max=2026-05-14、DATA-R3 接続 OK / verdict MISSING) | **YES** ✓ — "5/14 close is the D9-D13 base, and D13 verified DATA-R3 path while dry-run verdict remains MISSING" |
| **D** | D14 GO criteria (= 8/9 + actual + liquidity + event、single D-day candidate_only) | **YES** ✓ — "D14 GO requires 8/9 invariants plus actual price, liquidity, and event checks; single D-day remains candidate_only, and HOLD if review has <3 positives or sector concentration" |

→ 4 lane 全 事実確認 ✓。実装/設計通り、追加変更不要。

## §12 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ (= 本 wave 全 read-only) |
| F282 plist | size=1751 / mtime=1778602507 → **不変** ✓ |
| DB write | 0 ✓ |
| production / develop / staging 接続 | staging read-only のみ |
| LINE / token / API / launchctl / plist / cron | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault 3 file 追加のみ) |

## §13 vault docs (= 本 wave 追加)

- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W2_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W2_results.md`
- `~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md`

## §14 next wave instruction (= D14)

```
Wave 60-pilot-D14 (= 初回少額実弾候補 GO 判定 day)
- design reference: 03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
- 朝 J-Quants refresh 必須実行
- iSPEED で actual confirmation
- 4 段階判定 (= GO / GO_CONDITIONAL / HOLD / NO-GO)
- 全 12 項目 ✓ → GO で 340A0 100 株 entry 検討
- D14 review 最低 5 項目記入を強く推奨
- 場後 + 翌朝 + h20 後 で paper PnL handoff 3 回
```

## §15 次 Wave 候補 (= 優先順)

1. **D14 朝 (= 2026-06-02 08:30 JST) Wave 60-pilot-D14 着手**
2. **DATA-R3 active job wave** (= verdict=OK 目標、別 HQ 承認、HQ_APPROVE_FETCH_ACTIVE_JOB 想定)
3. **paper_pnl preview h1/h20 再 run** (= D14-D15 朝、6/26 頃)
4. **liquidity filter 強化 wave** (= 板厚 / spread を staging 内 read-only)
5. **全銘柄 daily refresh launchd 自動化** (= 別 HQ 承認 + plist 配置)
6. **features rerun wave** (= research_derived_indicators の persist、別 HQ 承認)
7. **W60-pilot-W3 集約** (= D14-D18 5 営業日 review、KPI 算出)
