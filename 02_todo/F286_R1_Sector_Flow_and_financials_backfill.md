---
id: F286-R1
phase: P5: Research Lane R1 (Sector Flow Agent MVP + market_financials backfill)
priority: 高 (R1 設計 Vault 化、HQ 承認後実装着手)
status: R1 implementation plan v1.0 vault 化、HQ Q1-Q5 判断待ち
owner: Fujiwara
depends_on: [F286 R0 feasibility, F285 Research Lane 仕様 v1.1, F100 J-Quants V2 daily, F276 indices]
chapter: "27"
created: 2026-05-09
updated: 2026-05-09
related: F286_R1_Research_Lane_implementation_plan_2026-05-09, F286_Research_Lane_R0_feasibility_2026-05-09, F285_Research_Lane_requirements_and_spec_2026-05-08
---

★ **F286 Research Lane R1**: Sector Flow Agent MVP + market_financials
   /fins/summary backfill 設計確定タスク。R1 Vault 化が完了したら HQ
   承認後 commit 分割で実装着手。

# F286-R1 Sector Flow Agent MVP + market_financials backfill

## サマリ

R1 = (A) Sector Flow Agent MVP + (B) market_financials に /fins/summary
を取り込む設計確定。R0 で Premium 不要が確認できた前提で、
F285 Research Lane の根幹データ基盤 + 最初の Agent を実装する Phase。

R1 で R2 (Cyclical Value / Growth / Dividend Growth) が必要とする
財務 data + 派生指標前提を整える。

## 詳細

[[F286_R1_Research_Lane_implementation_plan_2026-05-09|F286-R1 implementation plan v1.0]]
を参照 (11 章、Sector Flow MVP 仕様 + mapping + commit 分割 + HQ 判断
5 項目)。

## R1 範囲

### A. Sector Flow Agent MVP

入力: 既存 staging DB のみ (market_listings + market_prices_daily)
追加 endpoint: なし (Premium 不要)

集計:
- 銘柄レベル daily_return / turnover / volume_ratio_sma20
- 業種レベル mean_return / total_turnover / mean_volume_ratio
- sector_score = 0.4 × return + 0.3 × momentum + 0.3 × breadth
- up-flow Top 5 / down-flow Bottom 5

出力 (R1): SectorFlowSnapshot dataclass + JSON serialize
DB 書込: R3 で feature table or 専用 table に保存 (R1 では計算のみ)
Morning Report 接続: R4 で実装

### B. market_financials backfill 設計

取得元: /v2/fins/summary (107 fields)
既存 schema: 13 columns (R0 確認済、変更なし、migration 最小)

mapping (連結優先 + 非連結 fallback):
- code → Code / disclosure_date → DiscDate / fiscal_year_end → CurFYEn
- type_of_document → DocType
- net_sales → Sales / NCSales (fallback)
- operating_profit → OP / NCOP
- ordinary_profit → OdP / NCOdP
- profit → NP / NCNP
- total_assets → TA / NCTA
- equity → Eq / NCEq
- cash_flow_operating → CFO
- payload_json → 全 107 fields raw JSON (R2 派生計算用)

主キー (R1 推奨案 A):
- `(code, disclosure_date, type_of_document)` UNIQUE
- 修正開示は INSERT OR REPLACE で最新 fetched_at の値

backfill 期間 (R1 推奨案 A):
- 過去 5 年 (約 88,000 件、6-8 時間想定)

incremental update:
- 日次 / 週次 / 月次 で差分 fetch (運用方針)

## R2 派生指標の前提 (R1 で整合性確保)

| 派生 | 必要 field | R1 保存方法 |
|---|---|---|
| PBR = close/BPS | BPS | payload_json |
| PER = close/EPS | EPS / DEPS | payload_json |
| ROE = NP/Eq | profit / equity (schema 直接) | schema |
| 営業利益率 = OP/Sales | operating_profit / net_sales | schema |
| 配当性向 = DivAnn/EPS or PayoutRatioAnn | DivAnn / PayoutRatioAnn | payload_json |
| 売上成長率 / EPS 成長率 | Sales / EPS 4 期 | schema (Sales) + payload_json (EPS) |
| 会社予想成長率 | FSales / NxFSales | payload_json |
| 上方修正フラグ | ChgByASRev / ChgNoASRev | payload_json |
| 連続増配 | DivAnn 5 期 | payload_json |

## R1 commit 分割 (HQ 承認後着手、推奨 7 commit)

| # | commit |
|---|---|
| R1-A1 | feat(F286-R1): add Sector Flow Agent MVP |
| R1-A2 | chore(F286-R1): add Sector Flow Agent runner |
| R1-B1 | feat(F286-R1): /fins/summary -> market_financials mapping |
| R1-B2 | chore(F286-R1): backfill_market_financials runner |
| R1-B3 | chore(F286-R1): vault Sector Flow / market_financials smoke |
| R1-B4 | chore(F286-R1): full backfill 5 years |
| R1-C1 | docs(F286-R1): vault R1 result + Phase R2 着手判断 |

各 commit Codex pre-commit / --no-verify 禁止 / 個別 commit 厳守。

## 受入基準 (R1)

A. Sector Flow Agent:
- 既存 DB (read-only) で動作 / 17 / 33 業種両対応 / up_flow / down_flow ranking
- smoke (5 営業日) で出力検証 / regression なし

B. market_financials backfill:
- mapping 確定 / 主キー設計確定 (HQ Q1) / payload_json で raw 保存
- 5 銘柄 smoke で insert/upsert 動作 / R2 派生指標 field 確認

## 制約 (HQ 厳守、R0 継承)

- 自動発注 / 楽天証券 / Computer Use 禁止
- production / develop DB 無触
- staging のみ (write は backfill 時のみ)
- 外部スクレイピング禁止
- migration / 本格 ETL は HQ 承認後
- いきなり全銘柄 backfill しない (5 銘柄 smoke 経由)
- Premium 課金禁止 (MVP 不要、確認済)
- Codex pre-commit / --no-verify 禁止 / 個別 commit 厳守

## HQ 判断ポイント (詳細は計画書 §9)

- Q1: 主キー案 A (シンプル) or B (disc_no 追加) 選択
- Q2: backfill 期間 5 / 3 / 10 年
- Q3: commit 分割 7 案で OK か
- Q4: F287 並行着手 可否
- Q5: R1 完了基準

## 関連 commit

vault 側:
- (本 commit) docs(F286-R1): R1 implementation plan + TODO

## 関連リンク

- [[F286_R1_Research_Lane_implementation_plan_2026-05-09|F286-R1 plan v1.0]]
- [[F286_Research_Lane_R0_feasibility_2026-05-09|F286 R0 feasibility]]
- [[F285_Research_Lane_requirements_and_spec_2026-05-08|F285 仕様 v1.1]]
- [[F286_Research_Lane_R0_feasibility|F286 R0 TODO]]
- [[F284_F105_c6_final_result_2026-05-09|c6 final result]]
