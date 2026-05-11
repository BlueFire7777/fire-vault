---
id: F286-REPORT-R1
phase: F286 シリーズ / 設計起票 / Daily PnL Report Generator
priority: 高
status: 起票 ☆ Implementation Wave 8+ (= HQ approve 後)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R1 (= advisory_decisions、W4-1 完了)
  - F286-PNL-R2 (= advisory_snapshots / rows、W4-2 完了)
  - F286-PNL-R3 (= paper_pnl、W6-1+2 完了)
  - F119 Evaluation Agent (= Markdown report 既存)
  - F210 3000 万円 2 年目標 (= 進捗 input)
  - F236 LINE 5 部屋 (= 配信先 REPORT 部屋)
chapter: 第 13 章 (Evaluation Agent と相補的) / 第 26 章 (FIRE のエッジ = 再現性優位)
---

# F286-REPORT-R1: Daily PnL Report Generator (= 設計起票)

最終更新: 2026-05-11

## ★ 状態: 設計起票 (= Wave 7 W7-4、Implementation は Wave 8+ で別 approve)

F286-PNL-R1/R2/R3 で advisory→snapshot→paper_pnl のデータ蓄積基盤が
完成。**蓄積データを集約して Fujiwara への定常 PnL レポートを生成する**
新規シリーズ。F119 Evaluation Agent (= 提案出力) と相補的に、**日次/週次/
月次 PnL 報告** を担当する。

## 1. SIM-R1 比較 + REPORT-R1 選定理由

HQ Wave 7 W7-4 推奨枠で「REPORT-R1 or SIM-R1 設計起票」と OR 指定。
Architect 判断で **REPORT-R1 を主案**、SIM-R1 は後続枠とする。

### 比較表

| 観点 | REPORT-R1 | SIM-R1 |
|------|-----------|--------|
| **データ基盤前提** | F286 PNL R1/R2/R3 で完成 | F040-F047 Backtest / F050-F053 Paper Live 既存 |
| **緊急性** | 高 (= F286 PNL 完了→集約 が自然な次手) | 低 (= 既存 sim 基盤で十分) |
| **Fujiwara 可視性** | 直接的 (= 毎日 LINE で受信) | 間接的 (= sim 結果は HQ レビュー用) |
| **3000 万円 2 年目標 寄与** | 直接的 (= 進捗トラッキング) | 間接的 (= 戦略検証) |
| **F119 との関係** | 相補的 (= 評価 vs 報告) | 補完的 (= sim → 評価) |
| **Stage 進捗影響** | Stage 2 → 3 移行で有用性増 | Stage 3 移行後の課題 |
| **データ蓄積期間** | 1 ヶ月程度で意味あるレポート | 数ヶ月〜数年の sim 結果蓄積前提 |

### REPORT-R1 主案選定理由

1. F286 PNL R1/R2/R3 が「advisory 出力 → snapshot 保存 → paper_pnl 計算」
   までを完成させた → 次は「読み出して集約レポート」が自然な順序
2. Fujiwara への日次/週次/月次 PnL レポートは兼業前提利益最大化に直結
   (= R-01-04/05、3000 万円 2 年目標の進捗確認)
3. F119 Evaluation Agent は「評価/提案」、REPORT-R1 は「定常報告」、
   機能が重ならず棲み分け明確
4. LINE 配信パイプ (F062/F236、5 部屋) 整備済み → REPORT 部屋への送信は
   既存仕様の延長
5. Stage 3 移行ゲート (F241) には Markdown report が組込まれているが、
   F286 PnL data 統合は未対応 → REPORT-R1 で補完

### SIM-R1 後回し理由

1. 既存 sim 基盤 (F040-F047 / F050-F053 / F241) で Stage 2 (Paper Live)
   は稼働中、新規 SIM の緊急性低い
2. F286 PNL data を活用した「過去 advisory pattern simulation」は有望だが、
   データ蓄積期間が短い (= R3 完了直後) → 1-2 ヶ月運用後に再検討
3. REPORT-R1 で「何を見たいか」が固まってから SIM-R1 の出力形式を決める
   方が手戻り少ない

## 2. REPORT-R1 概要

### 2.1 目的

advisory_decisions / advisory_snapshots / advisory_snapshot_rows /
paper_pnl の蓄積データを集約し、**日次 / 週次 / 月次の PnL レポート** を
生成。Fujiwara が毎日朝の最初に確認する基本資料 + 兼業判断材料を提供。

### 2.2 機能

| 機能 | 出力 format | 配信先 | 頻度 |
|------|-------------|--------|------|
| 日次 PnL レポート | Markdown | LINE REPORT 部屋 + ファイル | 毎営業日 18:00 JST |
| 週次 PnL レポート | Markdown | LINE REPORT 部屋 + ファイル | 毎週金曜 18:00 JST |
| 月次 PnL レポート | Markdown | LINE REPORT 部屋 + ファイル | 毎月最終営業日 18:00 JST |

(cron 登録は sub-D3 凍結中、Wave 9+ で別 approve)

### 2.3 出力 (= Markdown sample skeleton)

```markdown
# FIRE 日次 PnL レポート (2026-05-11)

## 1. 当日 advisory 集計

| 区分 | 件数 |
|---|---|
| 🟢 積極的買い推奨 | 3 |
| 🟡 条件付き買い推奨 | 5 |
| ⚠️ 注意つき買い候補 | 2 |
| 🟠 場中監視 | 7 |
| 🔴 見送り推奨 | 4 |

## 2. paper_pnl (= 仮想 PnL)

| 銘柄 | label | 寄付き | h20 後終値 | qty | paper_pnl |
|---|---|---|---|---|---|
| 7203 | 🟢 積極的買い推奨 | 2,500 | 2,550 | 200 | +10,000 |
| ... |

合計 paper_pnl: +N,NNN 円 (N 件計算 / N 件未計算)

## 3. 実 PnL (= 楽天約定経由、M+2 以降)

| 銘柄 | 建値 | 決済値 | qty | 実 PnL |
|---|---|---|---|---|
| (M+2 以前は空欄) |

## 4. 進捗 (= F210 3000 万円 2 年目標)

- 当月累計 paper_pnl: +N,NNN 円
- 当月累計 実 PnL: +N,NNN 円
- 月次目標 (= 125,000 円/月) との比 (paper): N%
- 月次目標 との比 (実): N%
- 年次目標 (= 1,500,000 円/年) との比: N%

## 5. F119 評価サマリ (= F119 Evaluation Agent 最新出力 link)

(= 最新 evaluation report への link を提示)

## 6. 注記

(= 異常 / 注意 / 翌日への申送り)
```

## 3. データソース

| Source | Table | 用途 |
|--------|-------|------|
| F286-PNL-R1 | advisory_decisions | 当日 advisory 一覧、label 集計、paper_pnl |
| F286-PNL-R2 | advisory_snapshots / advisory_snapshot_rows | 当日 send batch 単位 |
| F286-PNL-R3 | advisory_decisions (paper_pnl 列) | 仮想 PnL |
| F210 | (goal_progress 等、F210 既存) | 3000 万円 2 年目標 進捗 |
| 楽天約定メール (M+2 以降) | actual_trades (= 別 schema、F235 想定) | 実 PnL |
| F119 | evaluation_reports (Markdown path) | 最新評価リンク |

## 4. 集約ロジック (= 純関数群、F119 と同じ approach)

### 4.1 module 構成 (= 案)

```
fire/report/
├── __init__.py
├── aggregators.py       # 純関数: 集計
├── daily_report.py      # 日次レポート生成
├── weekly_report.py     # 週次
├── monthly_report.py    # 月次
├── markdown_renderer.py # Markdown 整形 (F119 と同じ style)
└── ...

fire/scripts/jobs/
└── run_f286_report_r1_daily.py  # CLI runner

fire/tests/report/
└── test_*.py
```

### 4.2 集計関数 (= signature 案)

```python
def aggregate_daily_advisory(conn, base_date) -> DailyAdvisoryAggregate:
    """advisory_decisions の当日集計 (= label 別件数)"""

def aggregate_daily_paper_pnl(conn, base_date) -> DailyPaperPnlAggregate:
    """paper_pnl の当日集計"""

def aggregate_daily_actual_pnl(conn, base_date) -> DailyActualPnlAggregate:
    """actual_trades の当日集計 (= M+2 以降)"""

def fetch_goal_progress(conn, base_date) -> GoalProgress:
    """F210 進捗 (= 月次 / 年次目標との比)"""

def render_daily_markdown(daily_agg, paper_agg, actual_agg, progress) -> str:
    """Markdown 整形"""
```

すべて **read-only**、DB write しない。

## 5. 三段+六段ガード (= 既存 pattern 再利用)

REPORT-R1 runner は **read-only 専用** のため、F286-PNL-R3 と同等の
ガードは不要だが、以下を堅持:

1. **read-only DB open**: `file:?mode=ro` + `PRAGMA query_only=ON`
2. **--db-label / --db-path basename チェック** (= production / develop /
   staging を明示)
3. **--output-path** が DB / WAL / SHM path 不可 (= output 上書き防止)
4. **--output-path** symlink refuse + hardlink inode check
5. **LINE 配信は別 sub-task** (= 本 runner は Markdown ファイル生成のみ、
   LINE 送信は F062/F236 経由の別 runner)

## 6. LINE 配信 (= Wave 9+ 別 approve、本起票では含まない)

- 配信 runner: `scripts/jobs/run_f286_report_r1_send_line.py` (= 想定)
- 配信先: LINE REPORT 部屋 (= F236 既定)
- 配信内容: Markdown ファイルへの link / 重要 KPI 3 行サマリ
- 配信前 partial=False / sent_count >= 1 ガード (= W4.1-A pattern 再利用)
- 配信トリガー: cron (= sub-D3 凍結中、本起票では未定)

## 7. テスト方針 (= 3,637 baseline 維持)

| 種別 | 件数見込 |
|------|---------|
| aggregators 純関数 unit | 15-20 件 |
| daily_report 統合 | 5-10 件 |
| weekly_report | 5-10 件 |
| monthly_report | 5-10 件 |
| markdown_renderer | 5-10 件 |
| runner integration | 8-15 件 |
| **合計** | **45-75 件** |

実装 Wave 8+ で実 PR 後、tests 全 PASS 確認。

## 8. cron 連携 (= sub-D3 凍結中、Wave 9+ で別 approve)

```
# 想定 (= 凍結中、登録しない)
0 18 * * 1-5  cd ~/fire && .venv/bin/python -m scripts.jobs.run_f286_report_r1_daily ...
0 18 * * 5    cd ~/fire && .venv/bin/python -m scripts.jobs.run_f286_report_r1_weekly ...
0 18 * * *    cd ~/fire && .venv/bin/python -m scripts.jobs.run_f286_report_r1_monthly_if_last_biz_day ...
```

実 cron / launchd 登録は sub-D3 凍結解除後、HQ 別 approve で実施。

## 9. 受入基準 (= 3 段階)

### 動いた (= exit code 0 / Markdown ファイル生成成功)

- runner が exit 0 で完走
- 指定 path に Markdown ファイル生成

### 機能した (= 意味的成功)

- 当日 advisory 件数が正しく集計 (= advisory_decisions 件数と一致)
- paper_pnl 集計が compute_paper_pnl 結果と一致
- Markdown フォーマットが iPhone コピー対応 (= 単一コードフェンス形式
  推奨、内部 fence 禁止 — Fujiwara feedback)

### 期待値達成 (= 受入基準)

- Markdown レポートを Fujiwara が読んで「翌日の判断材料になる」と認定
- LINE REPORT 部屋への配信 (= Wave 9+ 後)
- 1 ヶ月運用後 F210 進捗トラッキングが機能していること

## 10. リスク評価

| risk | 影響 | 対策 |
|---|---|---|
| データ蓄積期間短く、月次レポートが空白 | 価値低 | 1 ヶ月後の本格運用開始まで日次のみ提示 |
| paper_pnl が大半 None (= market_prices_daily 不足) | レポート値薄い | paper_reason 別件数を表示、Fujiwara の透明性確保 |
| 実 PnL (= 楽天約定) が M+2 以降未連携 | 実 PnL 列空欄 | F235 完成まで「実 PnL 未連携」明記 |
| Markdown フォーマット改変で iPhone コピー破綻 | UX 悪化 | iPhone コピー対応形式厳守 (= 単一コードフェンス + 内部 fence 禁止) |
| F210 GoalConfig 未設定で進捗計算失敗 | レポート異常 | F210 既存 Phase 1A で対応済、Phase 1B 凍結中なので fallback |

## 11. 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE7_plan|Wave 7 plan]]
- [[F286_PNL_R3_paper_pnl_simulator_hook_2026-05-11|F286-PNL-R3 design]]
- [[F286_PNL_R3_staging_update_smoke_plan_2026-05-11|W7-1 staging UPDATE smoke plan]]
- [[F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|W7-2 sub-D2.3 smoke plan]]
- F119 Evaluation Agent: `~/fire/evaluation/`
- F210 GoalConfig: F210 vault doc
- F236 LINE 5 部屋: `~/fire/notifications/`
- 第 13 章: Evaluation Agent (= F119 と相補的)
- 第 26 章: FIRE のエッジ (= 再現性優位戦略、レポート蓄積もエッジの一部)

## 12. 次の起票候補 (= Wave 8+)

- **W8-impl-aggregators**: aggregators.py + tests (= Codex L3+L2 並列)
- **W8-impl-daily**: daily_report.py + Markdown renderer + runner + tests
- **W8-audit-daily**: REPORT-R1 daily の adversarial audit (Codex L4)
- **W9-impl-weekly**: 週次
- **W9-impl-monthly**: 月次
- **W9-line**: LINE REPORT 配信 (= F062/F236 既存 helper 経由)
- **W10-cron**: cron 登録 (= sub-D3 凍結解除前提)
