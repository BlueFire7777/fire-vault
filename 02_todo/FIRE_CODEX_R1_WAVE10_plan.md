---
id: FIRE-CODEX-R1-WAVE10-plan
phase: ガバナンス / Codex 並列実装 Wave 10 起票 / R-01-08 整合
priority: 最優先
status: 起票 ☆ Wave 10 5 Codex lane + 本線 5 計画並列実行中 (2026-05-12)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 9 W9-1 (= 完了)
  - HQ Wave 10 approve (= W10-1 MIG-R1 / W10-2/3 REPORT-R1 weekly+monthly /
    W10-5 sub-D2.3.x 起票、2026-05-12)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 10: MIG-R1 + REPORT-R1 weekly/monthly + sub-D2.3.x 起票

最終更新: 2026-05-12

## ★ 状態: 起票 (= 5 Codex lane impl/audit + 本線 4 plan vault doc)

HQ Wave 10 approve 受領後、3 タスクを並列着手:
- **W10-1**: F286-PNL-R3-MIG-R1 (= W9-1 で残課題化、production/develop の
  paper_reason 不在を構造化解消)
- **W10-2/W10-3**: REPORT-R1 weekly + monthly impl (= W8-3 daily の延長)
- **W10-5**: W9-3 DATA-R3 sub-D2.3.x 4 plan vault doc (= f100/f101/f111/f119)

## Wave 10 構成 (= 9 sub-task)

| sub | lane | task | 成果物 |
|-----|------|------|--------|
| W10-1 | L3+L2 (Codex) | F286-PNL-R3-MIG-R1 impl + tests | fire code |
| W10-1b | L4 (Codex) | MIG-R1 audit | audit report |
| W10-2 | L3+L2 (Codex) | REPORT-R1 weekly impl + tests | fire code |
| W10-3 | L3+L2 (Codex) | REPORT-R1 monthly impl + tests | fire code |
| W10-4 | L4 (Codex) | REPORT-R1 weekly+monthly 統合 audit | audit report |
| W10-5 | 本線 | W9-3 sub-D2.3.x 4 plan vault doc | vault docs ×4 |

## W10-1 F286-PNL-R3-MIG-R1 詳細

W9-1 で staging のみ ALTER TABLE で paper_reason 緊急 migrate。
production/develop は別途 migration script で対応する HQ 明示要件。

### 仕様

新規 script: `scripts/setup/migrate_NN_advisory_decisions_paper_reason.py`

- **idempotent**: paper_reason column 存在チェック → 不在の場合のみ ALTER
- **--db-label**: production / develop / staging を choices で受付
- **--db-path basename** チェック (= label と一致)
- **--dry-run**: schema 状態確認のみ、ALTER 実行なし
- **--write**: 実 ALTER 実行 (= HQ approve 必須 wording、自動で実行しない)
- **safety**: column 追加 (TEXT) のみ、UPDATE / DROP / DELETE 一切なし
- **logging**: pre/post PRAGMA、追加/skip の判定

### 既存 W9-1 staging への idempotent 動作

W9-1 で既に staging に paper_reason 追加済。本 script を staging で
--dry-run 実行 → 「already exists」 出力、ALTER skip。

### test 改善 (= tmp DB schema 乖離解消)

tests/conftest.py 等で、tmp DB 作成時に migration script を apply する
helper を導入。これにより:
- tmp DB schema = 実 DB schema 一致
- W9-1c で発見した「test PASS だが実 DB で失敗」を再発防止
- 既存 W9-1a/W8-1/W6 等の test も improve 候補

### production/develop 適用は別 HQ approve

migration script は **存在するが、staging 以外への実行は別 approve 必須**。
script の --db-label production / develop 経路は **HQ approve marker** を
要求 (= 環境変数 / config 等で明示承認)、または手動 SQL に委ねる。

本 task ではあくまで script を整備、実 production/develop 適用は実施しない。

## W10-2 / W10-3 REPORT-R1 weekly / monthly impl

W8-3 daily の pattern 延長。同一 module 構成、period が weekly / monthly。

### weekly_report.py (= W10-2)

```python
def aggregate_weekly_advisory(conn, base_date) -> WeeklyAdvisoryAggregate:
    """週次集計 (= base_date を含む週、月曜開始).

    例: base_date='2026-05-12' (火) → 2026-05-11 (月) 〜 2026-05-17 (日)
    """

def generate_weekly_report(conn, base_date, goal_config=None) -> WeeklyReport:
    """週次 PnL レポート."""

def render_weekly_markdown(weekly_agg, paper_agg, actual_agg,
                            progress, latest_f119_report_path=None) -> str:
    """iPhone コピー対応 Markdown."""
```

runner: `scripts/jobs/run_f286_report_r1_weekly.py`
- daily と同じ三段+六段ガード再利用
- --output-md / --output-json / --completion-report safety
- --dry-run / --base-date / read-only DB

### monthly_report.py (= W10-3)

```python
def aggregate_monthly_advisory(conn, base_date) -> MonthlyAdvisoryAggregate:
    """月次集計 (= base_date を含む月、月初〜月末)."""

def generate_monthly_report(conn, base_date, goal_config=None) -> MonthlyReport:
    """月次 PnL レポート."""

def render_monthly_markdown(...) -> str:
    """iPhone コピー対応 Markdown."""
```

runner: `scripts/jobs/run_f286_report_r1_monthly.py`

## W10-5 W9-3 sub-D2.3.x 4 plan vault doc (= 本線)

各 sub runner の staging write smoke plan vault doc:

| plan | runner | staging write 内容 |
|------|--------|--------------------|
| sub-D2.3.f100 | fetch_historical_market_data | market_prices_daily への過去価格 INSERT |
| sub-D2.3.f101 | fetch_announcements | announcements + parsed_metrics への INSERT |
| sub-D2.3.f111 | run_research_watchlist_signal_persistence | advisory_decisions / signals への INSERT |
| sub-D2.3.f119 | run_f119_interpretation_evaluation | evaluation_reports / proposals への INSERT |

各 plan に含めるべき項目 (= W7-1 / W7-2 plan の pattern 踏襲):
- staging write の具体 SQL 列
- 対象 row 用意方法 (= INSERT new / UPDATE existing)
- pre-smoke 状態キャプチャ
- 実 fetch + write 手順
- 既存 row 不触検証
- rollback 手順
- HQ approve template

各 plan は **起票のみ**、実行は別 HQ approve。

## 安全制約 (= Wave 10 全 ✓ 維持)

- 実 LINE 送信 0
- W4.1-B F062 経由 smoke 保留継続
- production DB write 0
- develop DB write 0
- 未承認 staging DB write 0 (= W10-1 MIG-R1 は --dry-run のみ、--write は別
  HQ approve required)
- token / channel_token / secret 参照 0
- 楽天 / 自動発注 / Computer Use / Playwright なし
- cron / launchd / crontab 本番登録 0 (= sub-D3 凍結継続)
- .github/workflows/ 変更 0
- --no-verify 不使用
- scripts/seed_pattern_layer1.py 未接触
- simulation/research_lane/historical_indicators.py 未接触
- TODO Excel 未更新
- Codex 直接 commit 0

## 並列実行方針

**Phase 1**: W10-1 + W10-2 + W10-3 を Codex 並列起動 (= 3 lane、独立)
**Phase 2**: W10-5 sub-D2.3.x 4 plan を本線並走 (= Phase 1 と同時)
**Phase 3**: Phase 1 完了後、W10-1b + W10-4 audit を並列起動 (= 2 lane)
**Phase 4**: 本線 Integrator review + split commits + Wave 10 results +
  log + HQ 報告

## 受入基準

- [ ] W10-1 MIG-R1 impl + tests + audit 完了
- [ ] W10-2 weekly impl + tests 完了
- [ ] W10-3 monthly impl + tests 完了
- [ ] W10-4 REPORT-R1 weekly/monthly 統合 audit CRITICAL 0
- [ ] W10-1b MIG-R1 audit CRITICAL 0
- [ ] W10-5 4 plan vault doc 完成
- [ ] fire repo の commit + fire-vault の commit
- [ ] CLAUDE.md 完了 table 追加
- [ ] 3,785 PASS baseline 維持 + Wave 10 新規分追加
- [ ] HQ 1 ブロック報告

## Wave 11 候補

- W11-1: W10-1 MIG-R1 を production / develop に適用 (= HQ 別 approve)
- W11-2: REPORT-R1 LINE 配信 (= F062/F236 経由、別 HQ approve)
- W11-3: DATA-R3 sub-D2.3.x の staging write 実行 (= 4 sub 個別 HQ approve)
- W11-4: F271 v1.7 / FIRE-AUDIT-R1 v1.2 等の運用基盤改善

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE9_results|Wave 9 results]]
- [[../03_design/F286_REPORT_R1_daily_pnl_report_generator_2026-05-11|REPORT-R1 design (= daily)]]
- [[../03_design/F286_PNL_R3_schema_gap_paper_reason_2026-05-12|schema gap plan]]
- [[../log]]
