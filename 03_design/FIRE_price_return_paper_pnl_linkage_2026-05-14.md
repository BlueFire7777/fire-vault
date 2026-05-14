---
id: FIRE-design-price-return-paper-pnl-linkage-2026-05-14
phase: 本番 v0 中核 / Wave 61-pre 設計 doc
priority: 高
status: design-draft
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
wave: Wave 61-pre (= 設計 only)
implementation_wave: Wave 61-impl (= 次 wave で実装、read-only / no DB write)
---

# FIRE Price / Return / Paper PnL Linkage Design v1.0

## §1 目的

FIRE を「候補を出す」だけから「**候補が利益につながったか**を検証して改善する」
段階へ進める。D10 で出た 340A0 / 3798 / 137A0 / 7991 などの候補について:

1. **price / return**: refresh 後の値動きを market_prices_daily (= staging
   read-only) で追跡
2. **paper PnL**: 翌営業日寄付き entry + h20 close exit の paper ledger を JSON 生成
3. **manual review 接続**: review.md の actual entry / exit / pnl を読み込み、
   未記入なら review_missing warning
4. **pattern outcome**: freshness_ok / multi_reason / low_risk / refreshed_price_ok
   等の pattern と outcome (= win/loss/flat) を結びつけ
5. **promote / suppress / watch**: D10 単発で promote しない、複数 D-day 累積で評価

本 wave は **設計 only** (= DB write 0 / API 0 / token 0)。実装は Wave 61-impl。

## §2 D10 追跡対象 candidates

### 2.1 重点追跡対象 (= AFTER-R1 top_candidates)

| rank | ticker | name | label | score (F062 経由) |
|---|---|---|---|---|
| 1 | 340A0 | ジグザグ | 条件付き買い推奨 | 175.22 |
| 2 | 3798 | ＵＬＳグループ | 条件付き買い推奨 | 172.65 |
| 3 | 137A0 | Cocolive | 条件付き買い推奨 | 170.85 (= refresh 復活) |

### 2.2 拡張追跡対象 (= F111 boost_with_caution、entry 候補 4-12)

| rank | ticker | name | sector | close (5/14) | risk_yen |
|---|---|---|---|---|---|
| 4 | 7991 | マミヤ・オーピー | 機械 | 1,177 | 5,885 |
| 5 | 9130 | 共栄タンカー | 運輸 | 1,410 | 7,050 |
| 6 | 331A0 | メディックス | 情報通信 | 482 | 2,410 |
| 7 | 4389 | プロパティデータバンク | 情報通信 | 863 | 4,315 |
| 8 | 9247 | TRE ホールディングス | 情報通信 | 1,612 | 8,060 |
| 9 | 4317 | レイ | 情報通信 | 508 | 2,540 |

計 **9 銘柄** (= 重点 3 + 拡張 6)。前 wave (W60-jquants-daily-refresh-staging) で
価格更新済の 12 銘柄のうち 9 (= boost_with_caution + risk_within=True) が対象。

## §3 return horizon 定義

| horizon | 名称 | 説明 |
|---|---|---|
| **h0** | base_date close | F111-real-batch 生成時の close (= 2026-05-14 refreshed) |
| **h1** | next business day close | 翌営業日終値 |
| **h5** | 5 営業日後 close | 短期 trend |
| **h20** | 20 営業日後 close | 中期 trend (= 既存 F286-PNL-R3 既定の保有期間) |

return 計算:
```
hN_return = (close_hN - close_h0) / close_h0
```

intraday tick data が無い場合は daily close base で代替。intraday が必要に
なれば次 wave (= Wave 62?) で別途検討。

## §4 price source 設計

### 4.1 source

- table: `market_prices_daily` (= staging DB)
- access: **read-only URI mode=ro**
- production / develop DB 接続: **禁止**
- query pattern (= 既存 pnl/paper_pnl.py 流用):
  - `fetch_market_close_after_business_days(conn, code, base_date, n_days)` (= h1/h5/h20)
  - `fetch_market_open_next_business_day(conn, code, base_date)` (= 翌営業日寄付き)

### 4.2 freshness 判定

| state | 条件 | outcome_status |
|---|---|---|
| ok | close 取得済 + base_date 以降 row 存在 | 計算実行 |
| stale_price | close は取得済だが date < (today - 5 営業日) | caveat + warning 残し計算 |
| missing_price | 該当 code の row なし | outcome_status="missing_price"、skip 計算 |

## §5 paper PnL ledger JSON schema

```json
{
  "schema_version": "1.0",
  "generated_at": "<ISO 8601 JST>",
  "base_date": "2026-05-27",
  "evaluation_date": "<ISO 8601 date>",
  "source_trade_plan": "fire-vault/04_daily/2026-05-27_manual_live_pilot_trade_plan.md",
  "source_review_md": "fire-vault/04_daily/2026-05-27_manual_live_pilot_review.md",
  "source_after_r1_artifacts": {
    "morning_line_material": "/tmp/fire_d10_prep/after_r1/morning_line_material_2026-05-27.json",
    "f111_real_batch": "/tmp/fire_d10_prep/d10_f111_real_batch.json"
  },
  "entries": [
    {
      "ticker": "340A0",
      "name": "ジグザグ",
      "rank": 1,
      "action_label": "boost_with_caution",
      "research_advisory_label": "boost_with_caution",
      "label_emoji": "🟡",
      "score": 0.892,
      "after_r1_morning_score": 175.22,
      "research_final_score": 0.892,
      "reason_tags": ["staging market_listings 由来", "top_N + sector cap 通過"],
      "pattern_tags": ["refreshed_price_ok", "low_risk", "manual_review_active"],
      "estimated_100_share_risk": 1900,
      "risk_within_pilot_limit": true,
      "tradable_universe": true,
      "artifact_source": "f062_preview",
      "f111_input_source": "f111_real_batch",
      "planned_entry": 380.0,
      "planned_stop_loss": 361.0,
      "planned_take_profit": 388.0,
      "actual_entry_optional": null,
      "actual_exit_optional": null,
      "close_price_base_date": 380.0,
      "close_price_eval": null,
      "next_close": null,
      "h1_return": null,
      "h5_return": null,
      "h20_return": null,
      "paper_entry_price": null,
      "paper_exit_price": null,
      "paper_pnl_yen": null,
      "paper_pnl_pct": null,
      "real_pnl_optional": null,
      "entry_decision": "unknown",
      "outcome_status": "pending",
      "freshness_status": "ok",
      "warnings": []
    }
  ],
  "summary": {
    "candidate_count": 9,
    "evaluated_count": 0,
    "missing_price_count": 0,
    "review_missing_count": 0,
    "paper_win_count": 0,
    "paper_loss_count": 0,
    "paper_flat_count": 0,
    "average_h20_return": null,
    "average_paper_pnl_yen": null
  },
  "safety_flags": {
    "db_write": false,
    "line_send": false,
    "token_access": false,
    "api_call": false,
    "launchctl_call": false,
    "plist_modified": false,
    "cron_modified": false,
    "vacuum_executed": false,
    "order_automation": false,
    "production_data_modified": false
  }
}
```

## §6 manual review 接続設計

### 6.1 review.md → ledger 接続

| review.md field | ledger field |
|---|---|
| §2 計画 vs 実際 → entry 価格 actual | `actual_entry_optional` |
| §2 計画 vs 実際 → exit 価格 actual | `actual_exit_optional` |
| §3 PnL 総 PnL | `real_pnl_optional` |
| §8 final decision (enter/watch/skip) | `entry_decision` |

### 6.2 review_missing warning

- review.md `status: blank` または該当 field 全て空 → `warnings: ["review_missing"]`
- entry_decision = "unknown"
- 既存の F111-real-batch + AFTER-R1 morning は **そのまま** 評価続行
- paper_pnl は計算する (= "入っていたらどうだったか" 評価可能)

### 6.3 review 内容捏造禁止

- 空欄は **空のまま記録** (= null / "unknown" / "review_missing")
- runner は自動推測しない (= Fujiwara 本人の記入を待つ)

## §7 pattern outcome 設計

### 7.1 pattern_tags 候補 (= 初期 9 種)

| pattern | 意味 | 候補例 |
|---|---|---|
| `freshness_ok_high_conf` | DATA-R3 freshness=OK + 高 score | (D10 = freshness MISSING で該当なし) |
| `multi_reason` | reason_tags ≥ 3 件 | watchlist_reason + sector_cap + rank + ... |
| `low_risk` | risk_yen ≤ 5,000 円 | 340A0 (1,900) / 3798 (2,525) / 137A0 (3,695) / 331A0 / 4317 |
| `manual_review_active` | 全 candidate 共通 | 全 9 銘柄 |
| `research_boost` | label in {boost, boost_with_caution} | 全 9 銘柄 |
| `refreshed_price_ok` | latest_close date ≥ today - 14 営業日 | 全 9 銘柄 (5/14 refresh) |
| `liquidity_ok` | 板厚 / spread / 出来高 OK (= review 記入後 算出) | (review 待ち) |
| `sector_concentration` | top 3 同 sector | 340A0/3798/137A0 全て情報通信 ✓ |
| `risk_within_pilot_limit` | risk_yen ≤ 15,000 円 | 全 9 銘柄 |

### 7.2 outcome aggregation

```json
{
  "pattern": "refreshed_price_ok",
  "matched_tickers": ["340A0", "3798", "137A0", "7991", "9130", "331A0", "4389", "9247", "4317"],
  "outcome_count": 0,
  "positive_count": 0,
  "negative_count": 0,
  "average_return": null,
  "max_drawdown_proxy": null,
  "decision": "watch",
  "evidence_level": "low (= D10 単発)"
}
```

## §8 promote / suppress / watch 初期基準

### 8.1 基本原則

- **D10 単発では原則 promote しない** (= sample 不足、unvalidated)
- candidate_only / unvalidated を維持
- 断定表現を避ける (= "可能性あり" "傾向ある" 等の慎重 wording)
- 複数 D-day 累積で promote 判定

### 8.2 promote 条件 (= 全 ✓ 必要、初期案)

- 同 ticker / 同 pattern が 3 D-day 以上で positive outcome
- max drawdown_proxy 軽微 (= h5 内 -3% 以内)
- manual review で「納得感あり」「ルール遵守」
- 寄り天 / liquidity 問題なし
- forbidden_check.passed 維持

### 8.3 suppress 条件 (= いずれか ✓ で suppress、初期案)

- 同 ticker / 同 pattern が 2 D-day 以上で negative outcome (= h20 -5% 以下)
- liquidity issue 連続 (= 板薄 / spread 大)
- risk_notes に重大 caveat (= 急騰急落 / event リスク)
- 寄り天傾向 (= 寄付き高 → 引け安) が複数回

### 8.4 watch 条件 (= sample 不足、方向感未定)

- D10 単発 (= 本 wave 想定の **デフォルト判定**)
- positive / negative outcome 混在
- まだ判断不能

### 8.5 D10 想定判定

| pattern | matched count | decision (初期) | evidence_level |
|---|---|---|---|
| refreshed_price_ok | 9 | **watch** (= D10 単発) | low |
| low_risk | 5 (340A0/3798/137A0/331A0/4317) | **watch** | low |
| sector_concentration | 3 (340A0/3798/137A0) | **watch + caveat** | low |
| manual_review_active | 9 | **watch** | low |

→ **全 watch + candidate_only 維持** (= sample 不足の正直記述)

## §9 optional helper 設計 (= 次 wave 実装案)

### 9.1 案 A: 新規 runner

```
scripts/jobs/run_after_r1_paper_pnl_preview.py
```

- args:
  - `--base-date YYYY-MM-DD` (required)
  - `--morning-line-material PATH` (= AFTER-R1 morning JSON)
  - `--f111-real-batch PATH` (= F111 artifact)
  - `--review-md PATH` (= optional、未指定 → review_missing)
  - `--db-path` (= staging.db、read-only)
  - `--output-json` (= ledger JSON path、guarded)
  - `--horizon-days` (= default 20、h20 用)
  - `--dry-run` (= default True、本 runner は read-only 専用)

- guards:
  - production/develop DB refuse (= `is_safe_db_path` 流用)
  - output path guard (= `is_safe_output_path` 流用)
  - DB write 0 (= URI mode=ro)
  - safety_flags 全 false

### 9.2 案 B: 既存 runner 拡張

```
scripts/jobs/run_f286_after_r1_night_batch.py に --task paper-pnl-preview 追加
```

- `--task` choices に `paper_pnl_preview` 追加
- AFTER-R1 chain と一緒に paper ledger 生成
- ただし既存 task との separation 不明確 → **案 A 推奨**

### 9.3 推奨

**案 A** (= 新規 runner)。既存 F286-PNL-R3 helpers (`pnl.paper_pnl`) を import
して流用、新規 logic は ledger build のみ。

## §10 tests 設計 (= 次 wave 実装時)

| test class | 観点 |
|---|---|
| `TestPaperPnlLedgerSchema` | ledger JSON schema valid (= keys + types) |
| `TestMissingPriceWarning` | market_prices_daily に row なし → outcome_status="missing_price" + safe warning |
| `TestReviewMissingWarning` | review.md status=blank → warnings=["review_missing"] |
| `TestSkippedDecisionStillEvaluated` | entry_decision=skip でも paper_pnl 計算する |
| `TestPatternOutcomeAggregation` | pattern_tags 集計、matched_tickers/decision/evidence_level |
| `TestNoDbWriteAfterLedgerBuild` | runner 終了後 staging md5 不変 |
| `TestProductionDevelopRefused` | `--db-path` に fire.db / fire.develop.db → refuse |
| `TestStagingReadOnlyMode` | URI mode=ro で write 不可確認 |
| `TestSafetyFlagsAllFalse` | ledger 内 safety_flags 全 false |
| `TestOutputPathGuard` | `--output-json` に /data/ や /.git/ 含む path → refuse |

最低 10 tests。次 wave 実装時に追加。

## §11 安全境界 (= 本 wave + Wave 61-impl 共通)

- DB write **0**
- production / develop DB 接続 **0**
- staging DB **read-only のみ** (= URI mode=ro)
- LINE 0 / token 0 / API 0
- launchctl 0 / plist 0 / cron 0
- 実発注 0 / 楽天 0 / iSPEED 0 / Computer Use 0
- F282 plist 不変
- 3 DB mtime/size 不変

## §12 next implementation wave 案

### Wave 61-impl: paper PnL preview runner 実装

- 推定工数: 1 wave (= 設計 → 実装 → tests → vault)
- 実装内容:
  - `scripts/jobs/run_after_r1_paper_pnl_preview.py` (= 案 A、新規)
  - `tests/scripts/jobs/test_run_after_r1_paper_pnl_preview.py` (= 10+ tests)
  - 既存 `pnl.paper_pnl` helpers を import (= DB write logic は import しない)
  - 設計 doc (本 doc) を Implementation reference として使用
- 安全境界: 同上 (= 全 0)
- Codex 4 lane: A=schema / B=guards / C=helpers / D=safety
