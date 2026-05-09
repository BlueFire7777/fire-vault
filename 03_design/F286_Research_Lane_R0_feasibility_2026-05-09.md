---
title: F286 Research Lane R0 feasibility 結果
date: 2026-05-09
phase: F286 R0 (data feasibility precheck)
status: 4 endpoint precheck + 既存 DB schema 確認完了、MVP 即着手範囲特定済、HQ 判断要請
related: F285_Research_Lane_requirements_and_spec_2026-05-08, F284_F105_c6_final_result_2026-05-09, F101_TDnet, F100_J-Quants
trigger: HQ F286 着手指示 (2026-05-09、F293 Lane C Phase C2 保留後の方針転換)
---

# F286 Research Lane R0 feasibility 結果

★ J-Quants Standard で 4 endpoint 取得可否を実機検証。**重要発見**:
   /v2/fins/summary が 107 fields (BPS/EPS/Eq/Div*) を返し、**Premium
   契約なしで Cyclical Value / Dividend Growth も MVP 可能**な水準。
   /v2/fins/details と /v2/fins/dividend は Premium 必要だが、MVP では
   /fins/summary + 派生計算で代替可。

## 1. 調査目的

F285 で定義した Research Lane (中長期企業発掘エンジン) 本格実装前に、
必要データが取得可能か実機検証 (= Phase R0 feasibility precheck)。

確認項目 (HQ 指示):
1. Research Lane に必要なデータが既存 DB / J-Quants で取れるか
2. Standard プランで取得できるか
3. response field が MVP に十分か
4. 不足データは何か
5. 代替手段はあるか
6. Phase R1 以降で何を実装すべきか

## 2. F285 要求 7 項目との対応

| Fujiwara 初期要求 | F286 対応 endpoint / DB | precheck 結果 |
|---|---|---|
| 1. 今後資金循環セクター予想 | market_listings (sector_17/33) + market_prices_daily | **available**、既存 DB MVP 可 |
| 2. シクリカルバリュー株 | /fins/summary BPS/EPS/Eq + market_prices_daily | **available**、派生計算で MVP 可 |
| 3. 増収増益候補 | /fins/summary Sales/EPS/NCSales/NCEPS (4 期トレンド) | **available** |
| 4. 増配候補 | /fins/summary Div*/FDiv* fields | **available** (Premium /fins/dividend 不要) |
| 5. 決算先回り | /equities/earnings-calendar + /fins/summary | **available** |
| 6. 中長期企業リサーチ FIRE 組込 | F285 全体仕様 v1.1 で確立済 | (Phase R1+ で実装) |
| 7. Portfolio / Morning / LINE 接続 | F236 (5 部屋) + F287 計画あり | (Phase R4+ で実装) |

★ **要求 7 項目すべてに対し MVP 経路を確認、Premium 不要** ★

## 3. precheck 実施情報

| 項目 | 値 |
|---|---|
| 実施日時 | 2026-05-09 09:5X JST |
| 対象 commit (~/fire) | e718f1a (chore(F286): add Research Lane R0 precheck script) |
| sample 銘柄 (5 桁) | 72030 / 67580 / 80350 / 83060 / 16050 |
| sleep | 0.7 sec/req (rate limit 配慮) |
| 出力 | /tmp/f286_research_lane_r0_precheck_result.json + log |
| 制約 | DB 書込なし、production / develop DB 無触、staging read-only |

## 4. J-Quants V2 endpoint 確認結果

### 4.1 /v2/fins/summary — **★ available** ★

| 項目 | 値 |
|---|---|
| HTTP status | 200 (全 5 銘柄) |
| Standard 取得可否 | **可** |
| top-level key | `data`, `pagination_key` |
| 5 銘柄合計 data_count | **234 件** (= 1 銘柄 30-50 件、4 期 × 数年分) |
| 取得 field 数 | **107** |

**重要 field (Cyclical Value / Growth / Dividend MVP の根幹)**:
| field | 用途 |
|---|---|
| `Code` / `DiscDate` / `DiscTime` / `DocType` | 銘柄・開示識別 |
| `CurFYSt` / `CurFYEn` / `CurPerSt` / `CurPerEn` / `CurPerType` | 当期/対象期間 |
| `Sales` / `OP` / `OdP` / `NP` (※確認必要) | 売上 / 営業利益 / 経常利益 / 純利益 |
| `EPS` / `DEPS` / `BPS` | EPS / 希薄化後 EPS / BPS |
| `Eq` / `EqAR` | 自己資本 / 自己資本比率 |
| `CFO` / `CFI` / `CFF` / `CashEq` | 営業 / 投資 / 財務 CF / 期末現預金 |
| `Div1Q` / `Div2Q` / `Div3Q` / `DivAnn` / `DivTotalAnn` | 四半期 / 年間 / 累計 配当 |
| `FDiv1Q` / `FDivAnn` / `FDivTotalAnn` / `NxFDivAnn` | 会社予想配当 / 来期予想 |
| `FSales` / `FEPS` / `FSales2Q` / `FEPS2Q` | 会社予想 (上期 / 通期) |
| `NxFSales` / `NxFEPS` | 来期会社予想 |
| `FNCSales` / `FNCEPS` / `FNCNP` / `FNCOP` | 非連結予想 |
| `ChgAcEst` / `ChgByASRev` / `ChgNoASRev` | 業績修正・会計方針変更フラグ |
| `AvgSh` / `DivUnit` | 平均発行済株式数 / 配当単位 |

★ Cyclical Value (BPS/EPS/Eq) + Growth (Sales/利益 4 期トレンド) +
   Dividend (Div*/FDiv*) を **1 endpoint で網羅**。

サンプル値 (72030 トヨタ 2016 期):
  Sales: 28,403,118,000,000 (= 28.4 兆円)
  EPS: 741.36 / DEPS: 735.36
  NxFSales: 26,500,000,000,000 / NxFEPS: 490.51 (会社予想)
  NCSales: 11,585,822,000,000 / NCEPS: 581.08 (非連結)

### 4.2 /v2/fins/details — ❌ subscription_required (Premium 必要)

| 項目 | 値 |
|---|---|
| HTTP status | 403 (全 5 銘柄) |
| error message | `"This API is not available on your subscription."` |
| 用途 | Cyclical Value (詳細 BS/PL/CF) |

★ 影響: 詳細財務諸表は取得不能だが、§4.1 /fins/summary に主要数値
   (Sales/利益/BPS/EPS/Eq/CFO/CFI/CFF) が含まれるため **MVP には影響
   なし**。Phase R1 で派生計算 (PBR = 株価 / BPS、PER = 株価 / EPS、
   ROE = NP / Eq、営業利益率 = OP / Sales) で代替。

### 4.3 /v2/fins/dividend — ❌ subscription_required (Premium 必要)

| 項目 | 値 |
|---|---|
| HTTP status | 403 (全 5 銘柄) |
| 用途 | Dividend Growth Agent |

★ 影響: 詳細 DPS 履歴は取得不能だが、§4.1 /fins/summary の Div1Q /
   Div2Q / Div3Q / DivAnn / DivTotalAnn / FDiv* から **4 期程度の DPS
   推移と来期予想が判別可能**。連続増配判定はサンプル数によるが、5 期
   程度で MVP 可能。

### 4.4 /v2/equities/earnings-calendar — **★ available** ★

| 項目 | 値 |
|---|---|
| HTTP status | 200 |
| Standard 取得可否 | **可** |
| 取得 field | `Date`, `Code`, `CoName`, `FY`, `FQ`, `Section`, `SectorNm` |
| date 範囲 14 日前〜30 日後 (sample) | **144 件** |

★ Earnings Preview Agent (決算予定日先回り) の根幹データを取得可。
   全銘柄 / 業種別カバレッジは追加検証推奨だが、MVP には十分。

## 5. 既存 staging DB schema 確認 (read-only)

| table | exists | row_count | 備考 |
|---|:-:|--:|---|
| `market_listings` | ✅ | 4,449 | sector_17/33_code/name + scale_category + market + margin 完備 |
| `market_financials` | ✅ | **0** | schema あるが空、R1 で /fins/summary から fetch |
| `market_prices_daily` | ✅ | 526,764 | OHLCV + turnover_value + adj_* 完備 |
| `market_prices_intraday` | ✅ | 10,830,049 | F284/F105 c6 で完備、Lane C 用 (Research では低利用) |
| `announcements` | ✅ | **7** | F101 Phase 2/3 で追加、TDnet adhoc data |
| `features` | ✅ | 1,131,331 | F021 系で生成された feature 大量 |
| `index_data` | ✅ | 58 | F276 系、TOPIX 等 |

★ Sector Flow Agent (sector + daily) は **既存 DB のみで MVP 可** ★

`market_financials` は Cyclical Value / Growth / Dividend の保存先想定:
- 13 columns (code / disclosure_date / fiscal_year_end / type_of_document /
  net_sales / operating_profit / ordinary_profit / profit / total_assets /
  equity / cash_flow_operating / payload_json / fetched_at)
- /fins/summary の主要数値と整合 (= R1 で fetch + insert ロジック実装)

## 6. Research Lane Agent 別 取得可否整理 (HQ 指定 6 Agent)

### 6.1 Sector Flow Agent

| 必要データ | 取得元 | 状況 |
|---|---|---|
| 業種分類 | `market_listings` (sector_17/33_code/name) | ✅ 既存 |
| daily 騰落率 | `market_prices_daily` | ✅ 既存 |
| 売買代金 | `market_prices_daily.turnover_value` | ✅ 既存 |
| 出来高変化 | `market_prices_daily.volume` (移動平均派生) | ✅ 既存 |
| 業種別集計 | listings + daily の JOIN | ✅ 派生 |

★ **MVP 即着手可能** (既存 DB + 集計ロジックのみ)
- 17 業種 / 33 業種どちらも sector_17_code / sector_33_code で識別可能
- TOPIX / 業種指数比較は `index_data` (58 行、F276) と組合せ

### 6.2 Cyclical Value Screener

| 必要 field | 直接 / 派生 | 取得元 |
|---|---|---|
| BPS | 直接 | /fins/summary `BPS` ✅ |
| EPS | 直接 | /fins/summary `EPS` / `DEPS` ✅ |
| 株価 | 直接 | `market_prices_daily.close` ✅ |
| **PBR** | 派生 | `close / BPS` |
| **PER** | 派生 | `close / EPS` |
| 自己資本 | 直接 | /fins/summary `Eq` ✅ |
| 純利益 | 直接 | /fins/summary `NP` ✅ |
| **ROE** | 派生 | `NP / Eq` |
| 売上 | 直接 | /fins/summary `Sales` ✅ |
| 営業利益 | 直接 | /fins/summary `OP` ✅ |
| **営業利益率** | 派生 | `OP / Sales` |
| 時価総額 | 派生 | `close × AvgSh` (発行済株式数) |

★ **MVP 可能、Premium /fins/details 不要** ★
派生計算で PBR / PER / ROE / 営業利益率 / 時価総額をすべて算出可。

### 6.3 Growth / Earnings Growth Screener

| 必要データ | 取得元 | 状況 |
|---|---|---|
| 売上成長 | /fins/summary `Sales` × 4 期 | ✅ 直接 |
| 営業利益成長 | /fins/summary `OP` × 4 期 | ✅ 直接 |
| 経常 / 純利益成長 | /fins/summary `OdP` / `NP` × 4 期 | ✅ 直接 |
| 会社予想 | /fins/summary `FSales` / `FEPS` | ✅ 直接 |
| 上方修正 | /fins/summary `ChgByASRev` / `ChgNoASRev` フラグ | ✅ 直接 |

★ **MVP 可能** (/fins/summary 単独で 4 期トレンド + revision 判定可)
1 銘柄 30-50 件返るので 5-10 年分の 4 期 trend が取れる。

### 6.4 Dividend Growth Agent

| 必要データ | 取得元 | 状況 |
|---|---|---|
| DPS 履歴 | /fins/summary `DivAnn` × 期数 | ✅ 直接 |
| 予想 DPS | /fins/summary `FDivAnn` / `NxFDivAnn` | ✅ 直接 |
| 増配履歴 | DivAnn 4-5 期比較 | ✅ 派生 |
| 配当性向 | `DivAnn / EPS` | ✅ 派生 |
| 自社株買い | `announcements` (TDnet) | ✅ 既存 (F101) |

★ **MVP 可能、Premium /fins/dividend 不要** ★
連続増配判定は DivAnn 5 期程度で実装可。

### 6.5 Earnings Preview Agent

| 必要データ | 取得元 | 状況 |
|---|---|---|
| 決算予定日 | /equities/earnings-calendar | ✅ 直接 |
| 直近進捗率 | /fins/summary 期中 / 通期予想比較 | ✅ 派生 |
| 会社予想 | /fins/summary `FSales` / `FEPS` | ✅ 直接 |
| 上方修正 | /fins/summary フラグ + announcements | ✅ 直接 |
| 決算前の株価/出来高変化 | `market_prices_daily` | ✅ 既存 |

★ **MVP 可能** (calendar + summary + daily 既存で完結)

### 6.6 Watchlist Ranker

| 必要 | 状況 |
|---|---|
| 5 Agent 出力統合 | Phase R3 で着手、R1 完了後 |
| weighted_score | Phase R3 で実装 |
| A1/A2/B/C/D 分類 | F285 §7 仕様準拠 |
| Lane C / Morning / Portfolio 接続 | Phase R4 で着手 |

★ R1 (Sector Flow + Cyclical Value) 完了後に部分統合可。

## 7. 不足データ

| 項目 | 状況 | 代替案 |
|---|---|---|
| /v2/fins/details (詳細財務諸表) | ❌ Premium | /fins/summary 主要数値 + 派生計算で MVP 代替 |
| /v2/fins/dividend (詳細配当履歴) | ❌ Premium | /fins/summary Div* 4-5 期で代替 |
| アナリストコンセンサス | ❌ 商用 API 必要 | TDnet 上方修正 + 会社予想 + 進捗率で間接代用、F286 範囲外 |
| `market_financials` 実 data | row_count=0 | R1 で /fins/summary から fetch + insert 実装必要 |
| 自社株買い詳細 | TDnet 経由 | F101 announcements 既存、Phase R2 で連携 |

## 8. MVP 可能範囲 (即着手可能)

★ **F285 Research Lane の 5 Agent + Watchlist Ranker、すべて MVP 可** ★

優先度 (R1 即着手):
1. **Sector Flow Agent** (既存 DB のみで実装可、最も低コスト)
2. **Growth / Earnings Growth Screener** (/fins/summary fetch + 4 期 trend)
3. **Cyclical Value Screener** (/fins/summary BPS/EPS/Eq + 派生計算)
4. **Dividend Growth Agent** (/fins/summary Div* + 連続増配判定)
5. **Earnings Preview Agent** (earnings-calendar + summary 接続)
6. **Watchlist Ranker** (R3 で 5 Agent 統合)

Premium 課金は MVP では **不要**。詳細精度向上のため将来検討候補:
- /fins/details: BS/PL/CF の詳細項目 (= 棚卸資産、固定資産等)
- /fins/dividend: 長期 DPS 履歴 + 株主総会・基準日詳細

## 9. 推奨 Phase 分割 (R1 以降)

| Phase | 範囲 | データ依存 | 推定期間 |
|---|---|---|---|
| **R1** | Sector Flow + market_financials migration | 既存 DB のみ + /fins/summary smoke | 2-3 週間 |
| **R2** | /fins/summary backfill + Cyclical Value + Growth + Dividend | /fins/summary 全銘柄 | 2-3 週間 |
| **R3** | Earnings Preview Agent + Watchlist Ranker 統合 | earnings-calendar + summary 結合 | 2 週間 |
| **R4** | 朝レポート / LINE / Portfolio / Lane C 接続 | F236 既 + 配信機構 | 1-2 週間 |
| **R5** | shadow mode 受入評価 (3-6 ヶ月) | 全 Agent 稼働 | 3-6 ヶ月 |
| **R6** | Live 接続 (R5 合格後) | (継続) | (継続) |

合計: 7-10 週間 (R1〜R4) + R5 評価期間 3-6 ヶ月。

## 10. 次タスク案 (HQ 判断要請)

優先度高 (低コストで価値検証):
- **F286 R1-1**: Sector Flow Agent MVP 実装 (= sector 別資金循環 score)
  - 既存 DB のみで動く、API 追加不要、最速で価値検証
- **F286 R1-2**: /fins/summary backfill + market_financials 充填
  - migration 不要 (schema 既存)、insert ロジックのみ
  - 全銘柄 4,449 × 数十期 fetch (rate limit 配慮で 6-12h 推定)

優先度中 (R2 範囲):
- **F286 R2-1**: Cyclical Value Screener (PBR/PER/ROE 派生計算)
- **F286 R2-2**: Growth Screener (4 期 trend + revision 判定)
- **F286 R2-3**: Dividend Growth Agent (連続増配 + 配当性向)

優先度後 (R3+):
- F286 R3: Earnings Preview Agent + earnings-calendar fetch
- F286 R3: Watchlist Ranker
- F286 R4: 朝レポート / LINE 通知

並行候補:
- **F287 開始**: 決算ダッシュボード (Earnings Preview と相互補完、Phase
  R0 仕様確定済 v1.1)

## 11. リスク・制約

### リスク

1. **/fins/summary field 名の確定**: 107 fields のうち主要数値の field
   名は確認済 (Sales/EPS/BPS/Eq/Div* 等)、ただし R2 実装時に field 仕様
   変更があれば追従必要。J-Quants V2 docs 参照 + sample 検証必須。
2. **会計基準差**: IFRS / JGAAP で field 名が変わる可能性 (例: NCSales
   は Non-Consolidated 連結除外)。連結優先 + 非連結 fallback の方針が
   必要。
3. **earnings-calendar カバレッジ**: 144 件 / 14 日前 〜 30 日後で十分
   そうだが、3 月期 / 9 月期偏重等の偏りあれば R2 で確認。
4. **アナリストコンセンサス代替の精度**: TDnet 上方修正 + 会社予想で
   代用、本格コンセンサス比較には商用 API 必要、Phase R5 受入評価で
   精度検証。

### 制約 (HQ 指示厳守)

- 自動発注 / 楽天証券操作自動化 / Computer Use 禁止
- DB migration / 本格 ETL は F286 R0 範囲外 (R1+ で別タスク)
- production / develop DB 無触
- staging DB read-only
- 外部スクレイピング実装禁止
- Premium 課金は Fujiwara 判断必須 (本 R0 では「不要」と判定済)
- Codex pre-commit / --no-verify 禁止 / 個別 commit 厳守

## 12. HQ 判断ポイント

Q1: **MVP 範囲が想定より広く取れた件、F286 R1 着手判断**
   /fins/summary が Premium 不要で BPS/EPS/Eq/CFO/Div* まで網羅、Cyclical
   Value / Dividend Growth も MVP 範囲内。R1 即着手 (Sector Flow Agent
   から) で OK か。

Q2: **F285 仕様 v1.1 §11 Phase 分割の見直し**
   仕様書では「Phase R2 = Earnings Preview + Dividend Growth」だったが、
   precheck 結果から R2 で 4 Agent (Cyclical Value / Growth / Dividend /
   Earnings Preview) を一括着手も可能。Phase 分割を §9 推奨に更新する?

Q3: **R1 着手順序**
   推奨: Sector Flow → /fins/summary backfill → 4 Agent 並行。HQ 順序
   選好あれば指示。

Q4: **F287 並行着手**
   F287 (決算ダッシュボード) と F286 R1 を並行進行は許容?
   F287 の earnings-calendar 連携が R3 と被るので、HQ 判断で順序設計。

Q5: **Premium 課金判断**
   /fins/details / /fins/dividend は Premium 必要だが、MVP 範囲では
   不要。詳細精度向上 (R5 評価結果次第) で再検討、現時点では「不要」
   判定で OK か。

## 13. 関連 commit

~/fire 側:
| commit | 内容 |
|---|---|
| **e718f1a** | chore(F286): add Research Lane R0 precheck script |

vault 側:
| commit | 内容 |
|---|---|
| 本 commit | docs(F286): add Research Lane R0 feasibility report |
| (続く) | docs(F286): update Research Lane R0 TODO |
| (続く) | docs(F286): log milestone — Research Lane R0 feasibility completed |

precheck 出力:
- /tmp/f286_research_lane_r0_precheck_result.json (JSON 詳細)
- /tmp/f286_research_lane_r0_precheck.log (走行 log)

## 14. 改訂履歴

- v1.0 (2026-05-09): 初版、F286 R0 precheck 完了直後の Vault 化、
  /fins/summary が Premium 不要で MVP 範囲を網羅する重要発見、
  R1 即着手判断材料を提供、HQ 判断 5 項目要請。
