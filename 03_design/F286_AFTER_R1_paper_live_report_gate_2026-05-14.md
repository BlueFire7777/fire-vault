---
id: F286-AFTER-R1-paper-live-report-gate-and-contract
phase: 本番 v0 後拡張 / Wave 59-pre / AFTER-R1 Paper-Live & Report gate
priority: 高
status: 設計 v1.0 (= Wave 59-pre、impl は v0 後 + HQ 明示承認後)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 35-pre (= F286_AFTER_R1_night_paper_live_batch 設計 v1.0)
  - Wave 54-pre (= AFTER-R1 read-only scaffold v1.0)
  - Wave 54-post (= scaffold adversarial audit、SAFETY_FLAGS immutable / outputs reports/after_r1/)
  - Wave 58-temp (= Integrated Engineering GO、5/16 本番 F282 + Official GO 待ち)
related:
  - 03_design/F286_AFTER_R1_night_paper_live_batch_2026-05-12.md
  - 03_design/F286_AFTER_R1_read_only_runner_design_2026-05-14.md
  - scripts/jobs/run_f286_after_r1_night_batch.py (= v1.0.1 scaffold)
chapter: post-v0 / AFTER-R1 implementation gate
---

# F286-AFTER-R1 Paper-Live / Report Gate & Contract Design v1.0 — Wave 59-pre

最終更新: 2026-05-13

## §1 目的

v0 後拡張 F286-AFTER-R1 の Paper-Live / Report 機能を **実装する前** に
必要な gate / input dependency / output contract / safety policy /
future tests / roadmap を **read-only design** で固める。

実 Paper-Live 実行 / DB write / API call / LINE 送信 / token 参照 /
launchd/cron 登録は **本 wave で行わない**。本 wave は **docs のみ**。

## §2 なぜ今やるか / 今回やらないこと

### §2.1 なぜ今 (= 5/13 時点)
- 朝の LINE 通知 chain は Wave 58-temp で Integrated Engineering GO に到達
- Official F282 GO は 5/16 本番試走後 + 5/19 別判定待ち
- 待ち時間に v0 後拡張の最初の機能 (= Paper-Live / Report) を前倒し設計
- v0 安定後すぐに実装に入れる準備を整える
- 本 wave は **読み取り設計のみ** で v0 本線に干渉しない安全構成

### §2.2 今回やらないこと
- Paper Live 実実行
- DB write / DB sqlite 接続
- API call / 外部通信
- LINE 送信 / token 参照
- launchctl 実行 / plist 配置 / cron 登録
- W54-pre scaffold (= `run_f286_after_r1_night_batch.py`) の大改修
- v0 本線 (F282 / DATA-R3 / F062 / readiness / Ops Summary / wrapper) 変更
- TODO Excel 更新
- 楽天証券操作 / 自動発注 / Computer Use

## §3 Paper-Live Implementation Gate (= Lane A 設計)

Paper-Live 実装 phase に進む **必須条件** (= 全 9 件 PASS 必須):

| # | 条件 | 確認方法 |
|---|---|---|
| 1 | **Production v0 D-Day 完了** | 2026-06-09 火 D-Day 完了、log.md に milestone 記録 |
| 2 | **v0 dual-run 期間完了** | 6/9-6/15 月 5 営業日全 PASS、daily Ops Summary GO |
| 3 | **safety incidents 0** | `~/fire-vault/07_incidents/` 内に rollback / NO-GO 記録 0 |
| 4 | **DATA-R3 freshness 安定** | 5 営業日連続 verdict=OK (= W41 trial + dual-run) |
| 5 | **F062 advisory production/no-send 運用安定** | 5 営業日連続 production send OK / record-decisions row +1 |
| 6 | **readiness CLI v1.2 daily 判定安定** | 5 営業日連続 verdict=GO (= pre-v0-launch or daily phase) |
| 7 | **Ops Summary v1.0.1 daily 判定安定** | 5 営業日連続 verdict=GO or HOLD (= NO-GO 0) |
| 8 | **advisory_decisions 記録方式確定** | 12 keys schema (= W52-pre DB labels preview contract) + 四段ガード安定動作 |
| 9 | **`HQ_APPROVE_AFTER_R1_PAPER_LIVE` marker 取得** | env or vault file で明示承認、別 HQ 判断 |

→ 9 件全 PASS → Paper-Live 実装 phase 着手可。
→ 実行前に **再度 HQ 承認** が必要 (= 着手と実行の二段階承認)。

### §3.1 Gate の段階分け

| Gate Level | 必要条件 | 進行可能 phase |
|---|---|---|
| **L0 Design** | 本 wave (W59-pre) | docs / schema / contract |
| **L1 Read-Only Aggregator** | L0 + v0 D-Day (= 条件 1) | reports/after_r1/ への集計 (= DB read のみ可) |
| **L2 Paper-Live Dry-Run** | L1 + 条件 2-5 + dual-run 完了 | dry-run paper PnL 計算、DB write 0 |
| **L3 Paper-Live Real** | L2 + 条件 6-9 + 二段階 HQ 承認 | 実 Paper Live PnL 計算 + advisory_decisions 紐付け書込 |
| **L4 Replay / Simulation / Lane** | L3 安定後 + 別 HQ 承認 | 過去 6 ヶ月 replay / parameter swap / lane eval |
| **L5 Pattern / ML / Dashboard** | L4 + 別 HQ 承認 | pattern extraction / ML feature / DASH-R1 連携 |

各 level 間で **別 HQ 承認** が必要。skip 不可。

## §4 Report Implementation Gate (= Lane B 設計)

Paper-Live **実実行前** に作れる report 範囲。

### §4.1 許可範囲 (= L1 Read-Only Aggregator level)
- **`reports/after_r1/` namespace 限定** (= W54-post で確定)
- **v0 運用 artifact の read-only 集計**:
  - F062 advisory preview / send result (= advisory_decisions read)
  - DATA-R3 freshness report (= read-only)
  - readiness CLI output (= read-only)
  - Ops Summary output (= read-only)
  - 04_daily checklist (= read-only)
  - W282 snapshot (= read-only)
- **DB sqlite 接続は read-only mode** (= `mode=ro` URI / `PRAGMA query_only=ON`)
- **DB write 0**
- **API call 0**
- **LINE 送信 0**
- **token / env 全体読出 0**

### §4.2 禁止範囲 (= L2 以降)
- Paper Live 実実行 (= 仮想 PnL 計算)
- DB write (= report_aggregation 等の新 table 含む)
- production / develop / staging DB への write 接続
- token / API / channel_token / .env 参照
- brokerage / order automation / 楽天 / iSPEED
- LINE 送信
- launchd / cron 登録

### §4.3 Report 出力 path 規約 (= Lane B + Lane G 整合)
- 主 output: `reports/after_r1/<task>_<base_date>.md` / `.json`
- 副 output: `reports/after_r1/<task>_<base_date>_summary.json`
- W54-post で確定: `patterns/extracted/` / `reports/dashboard/` への直書きは scaffold 段階で禁止、
  代わりに `reports/after_r1/<task>_extracted_<base_date>/` / `reports/after_r1/dashboard_<base_date>.json`

## §5 Input Dependency Map (= Lane C 設計)

### §5.1 依存分類 (= 12 source)
| source | 役割 | 必要 level | read-only OK |
|---|---|---|---|
| F062 advisory preview (= W57-temp / W52-pre / production) | 入力 advisory rows | L1+ | ✓ |
| F062 send result / advisory_decisions DB | 実 send / record | L1+ | ✓ (= read-only) |
| DATA-R3 freshness report | freshness gate 入力 | L1+ | ✓ |
| readiness CLI output JSON | chain integration check | L1+ | ✓ |
| Ops Summary output JSON | daily verdict | L1+ | ✓ |
| 04_daily checklist md | 人間判断 trace | L1+ | ✓ |
| price / return data (= market_prices_daily) | PnL 計算入力 | L2+ | ✓ (= read-only) |
| PnL snapshot (= paper_pnl table) | 仮想 PnL 既存 | L2+ | ✓ (= read-only) |
| lane evaluation result | Lane 別 metric | L4+ | ✓ |
| simulation result | what-if 分析 | L4+ | ✓ |
| replay result | 過去 PnL | L4+ | ✓ |
| pattern extraction output | パターン Store 連携 | L5+ | ✓ |

### §5.2 依存グラフ
```
[v0 production]
F062 production send → advisory_decisions DB row +1
                    + freshness report (= DATA-R3 read)
                    + readiness CLI / Ops Summary daily
                    + 04_daily checklist
   ↓ (= 1 営業日 dual-run 後)
[L1 Read-Only Aggregator]
reports/after_r1/{daily_summary, weekly_summary, monthly_summary}_<date>.md
   ↓
[L2 Paper-Live Dry-Run]
+ market_prices_daily read
+ paper_pnl read
→ reports/after_r1/paper_live_<date>.md (= dry-run、書込 0)
   ↓
[L3 Paper-Live Real]
+ paper_pnl write (= staging-only first)
→ reports/after_r1/paper_live_<date>.md (= real PnL)
+ paper_pnl table row +N (= 候補数分)
   ↓
[L4 Replay / Simulation / Lane Eval]
+ historical price 6 months read
+ Lane logic 再計算
→ reports/after_r1/{replay,simulation,lane_eval}_<date>.{md,json}
   ↓
[L5 Pattern / ML / Dashboard]
+ patterns/extracted/ read
→ reports/after_r1/{patterns_extracted,ml_features_dataset,dashboard}_<date>/
```

### §5.3 W54-pre TASK_REGISTRY との対応

| W54-pre task | 必要 level | 必要 input |
|---|---|---|
| paper-live | L2 (dry-run) → L3 (real) | F062 send / advisory_decisions / market_prices_daily / paper_pnl |
| report | L1 | F062 / DATA-R3 / readiness / Ops Summary / 04_daily |
| replay | L4 | 過去 6 ヶ月 historical price + decisions |
| simulation | L4 | replay 安定 + 仮想 parameter |
| lane-eval | L4 | 20 営業日蓄積 + decisions + paper_pnl |
| pattern | L5 | lane_eval + replay |

## §6 Output Contract (= Lane B + Lane D 設計)

### §6.1 JSON schema (= scaffold v1.0.1 + L1 拡張)

```json
{
  "schema_version": "1.0",
  "cli_version": "1.0.1",
  "generated_at": "...",
  "base_date": "YYYY-MM-DD",
  "mode": ["dry_run", "read_only"],
  "strict": false,
  "verdict": "DESIGN_PREVIEW",
  "source_artifacts": [
    {"category": "F062_PREVIEW", "path": "...", "mtime_iso": "...", "size": N},
    {"category": "DATA_R3_FRESHNESS", "path": "...", "verdict": "OK"},
    {"category": "READINESS_CLI", "path": "...", "verdict": "GO"},
    {"category": "OPS_SUMMARY", "path": "...", "verdict": "GO"},
    {"category": "ADVISORY_DECISIONS", "row_count_today": N}
  ],
  "paper_live_status": "planned|dry_run|real|skipped",
  "report_status": "planned|aggregated|preview_only",
  "skipped_reason": "...",
  "readiness_summary": {...},
  "ops_summary": {...},
  "safety_flags": {
    "db_write": false,
    "db_sqlite_connect": false,
    "line_send": false,
    "token_access": false,
    "api_call": false,
    "launchctl_call": false,
    "plist_modified": false,
    "cron_modified": false,
    "vacuum_executed": false,
    "order_automation": false,
    "production_data_modified": false,
    "paper_live_executed": false
  },
  "implementation_gate": {
    "current_level": "L0_design",
    "next_level": "L1_read_only_aggregator",
    "gate_conditions": [...],
    "passed": [...],
    "missing": [...]
  },
  "next_actions": [...]
}
```

### §6.2 Markdown schema

```markdown
# F286-AFTER-R1 Paper-Live / Report Gate — YYYY-MM-DD

## V0 dependency
## Available inputs
## Missing prerequisites
## Implementation gate (= L0-L5)
## Report preview scope (= L1)
## Safety gate
## Next wave candidates
## Safety flags (= 12 keys 全 false)
```

## §7 Safety Gate (= Lane D 設計)

以下のいずれか **1 つでも必要なら NO-GO**:
- DB write
- DB sqlite 接続 (= read-only mode 以外)
- token / API call
- LINE 送信
- Paper Live 実実行 (= L3 未到達時)
- brokerage / order automation
- launchd / cron 登録
- production / develop DB への write 接続
- v0 未安定 (= D-Day 未完了、dual-run 未完了、incident > 0)
- safety incidents > 0

### §7.1 safety_flags は immutable (= W54-post 確定)
- `types.MappingProxyType` で runtime 書換不能
- 12 keys 全 false 維持
- L1+ 進行時にも flag は更新せず、新 wave で別 SAFETY_FLAGS_L1 / L2 等を別途定義

### §7.2 停止条件
runtime に下記が必要になった場合 → 即停止 + HQ 確認:
- DB write / DB sqlite 接続 (= 非 read-only)
- LINE 送信 / token 参照
- API call / 外部通信
- launchctl / plist / cron 変更
- Paper Live 実実行 (= L3 marker 未取得)
- 楽天証券操作 / 自動発注 / Computer Use

## §8 Future Tests Design (= Lane E 設計)

Paper-Live / Report 実装時に **必須** な test 集 (= 概念設計、実装は L1+ wave で):

### §8.1 安全系 (= 12 件)
- default dry-run / read-only mode
- no DB write (= AST 検証 + runtime stat 不変)
- no sqlite connect (= AST 検証)
- no LINE / token / API call (= AST 検証)
- no brokerage / order automation (= AST 検証 + literal 検証)
- output namespace `reports/after_r1/` 限定
- output path guard (= /data/ / .git/ / .fire_secrets/ / LaunchAgents/ refuse)
- safety_flags immutable (= MappingProxyType assertion)
- malformed input safe fail (= dict guard / parse error 安全側)
- missing v0 artifacts safe skip (= 不在で abort せず WARN)
- Paper Live marker `HQ_APPROVE_AFTER_R1_PAPER_LIVE` 不在なら L3 実行不可
- advisory_decisions schema 12 keys 互換性確認

### §8.2 機能系 (= 8 件)
- L1 aggregator: F062 / DATA-R3 / readiness / Ops Summary / checklist 集計
- L2 dry-run: paper PnL 計算 + market_prices_daily read + DB write 0
- L3 real: staging DB paper_pnl write + advisory_decisions 紐付け
- L4 replay: 過去 6 ヶ月 historical + decisions 再計算
- L4 simulation: 仮想 parameter swap
- L4 lane-eval: Lane 別 win rate / drawdown / sample size
- L5 pattern: パターン抽出 + Pattern Store 連携
- L5 ML feature: 仮 parquet export

### §8.3 chain integration 系 (= 5 件)
- readiness CLI v1.2 JSON 取り込み
- Ops Summary v1.0.1 JSON 取り込み
- F062 preview text + payload 取り込み
- DATA-R3 freshness report 取り込み
- 04_daily checklist md 取り込み

## §9 Future Wave Sequencing (= Lane F 設計)

W54-pre TASK_REGISTRY と整合させ、Wave 番号は **概念上** の番号として記述。
v0 D-Day (6/9 火) 完了後の dual-run 完了 (= 6/15 月) 以降を起点。

| Wave | 内容 | Implementation Gate Level | 想定時期 |
|---|---|---|---|
| **W59-pre (本 wave)** | gate + contract design | L0 | 5/13 (現在) |
| W60-pre | AFTER-R1 report-only read-only aggregator 設計 + tests | L1 design | v0 D-Day 後 (6/9+) |
| W60-impl | AFTER-R1 report-only impl | L1 impl | v0 dual-run 完了後 (6/16+) |
| W61-pre | paper-live input resolver design | L2 design | W60 後 |
| W61-impl | paper-live dry-run runner impl | L2 impl | W61-pre 後 |
| W62-pre | paper-live dry-run fixture | L2 fixture | W61 後 |
| W62-impl | staging DB paper_pnl 試走 | L3 design + impl 着手前 HQ marker | W62-pre 後 |
| W63 | replay dependency design | L4 design | W62 後 |
| W64 | simulation dependency design | L4 design | W63 後 |
| W65 | lane evaluation contract | L4 design | W64 後 + 20 営業日蓄積 |
| W70+ | pattern / ML feature candidates | L5 design | W65 後 |
| W75+ | ML parquet export | L5 impl | W70 後 |
| W80+ | DASH-R1 dashboard 連携 | L5 design | W75 後 |

### §9.1 既存表記との対応 (= W54-pre EXPECTED_WAVE)
W54-pre TASK_REGISTRY の `expected_wave`:
- paper-live: Wave 55+ → 本 doc では W60+ (= 概念上の wave 番号、現行進行と整合)
- replay: Wave 56+ → W63+
- simulation: Wave 57+ → W64+
- lane-eval: Wave 58+ → W65+
- pattern: Wave 60+ → W70+

→ W54-pre 表記は **概念上の number** として残す。大規模 doc 書換はしない (= W52.5-pre 方針継承、minimal edit 中心)。
→ scaffold の `expected_wave` 文字列は将来 W60-pre/impl 着手時に最新番号に align (= 別 wave で minimal edit)。

## §10 v0 本線との衝突確認 (= Lane G + Lane H 監査)

### §10.1 v0 本線 file 変更 0
- `scripts/jobs/run_f282_weekly_snapshot.py` 不触
- `scripts/jobs/run_f286_data_r3_daily_refresh.py` 不触 (= W56.5-temp fix 後)
- `scripts/jobs/run_f062_research_advisory_line_preview.py` 不触
- `scripts/jobs/run_production_v0_readiness_check.py` 不触 (= W51 v1.2)
- `scripts/jobs/run_production_v0_ops_summary.py` 不触 (= W44.6-post v1.0.1)
- `scripts/jobs/run_f062_no_send_wrapper_preview.py` 不触 (= W52-post v1.0.1)

### §10.2 W54-pre scaffold 不変
- `scripts/jobs/run_f286_after_r1_night_batch.py` 不触 (= W54-post v1.0.1)
- 既存 tests 不変

### §10.3 v0 launch schedule 不変
- 5/16 02:00 F282 試走、5/19 GO 判定、Wave 41/45/52/53 順序、6/8 final strict、6/9 D-Day
  すべて維持

### §10.4 temp smoke と Official GO の分離維持
- W55/56/56.5/57/58-temp の Engineering GO は **Official GO の代替にしない**
- 本 wave も同様、L0 design は Official L1+ impl approval の代替にしない

## §11 Unresolved Items / HQ 判断事項

| # | 項目 | HQ 判断要素 |
|---|---|---|
| 1 | `HQ_APPROVE_AFTER_R1_PAPER_LIVE` marker の正式名 | 本 doc では仮称、Wave 53 後別途確定 |
| 2 | L1 → L2 / L2 → L3 進行の二段階承認 marker | L1 完了後別途設計 |
| 3 | staging DB への paper_pnl write 開始時期 | dual-run 完了後別 wave |
| 4 | replay 期間 (= 6 ヶ月 / 1 年 / 全期間) | W63 で別途決定 |
| 5 | ML feature export format (= parquet / csv / sqlite) | W75 別 wave |
| 6 | DASH-R1 dashboard 仕様 | W80 別 wave |
| 7 | Lane promotion / demotion 提案の承認フロー | W65 後 |
| 8 | advisory improvement material の Fujiwara 提示 format | W70 後 |

## §12 safety 確認 (= 本 wave 自体)

| 項目 | 結果 |
|---|---|
| 実 file 変更 | 3 file (= 本 doc + W59-pre plan + results) |
| 既存 v0 path 変更 | 0 |
| W54-pre scaffold 変更 | 0 |
| 既存 tests 変更 | 0 |
| DB write / DB sqlite 接続 | 0 |
| LINE 送信 / API call | 0 |
| token / channel_token / secret / .env 値参照 | 0 |
| env 全体読出 | 0 |
| launchctl 実行 | 0 |
| plist 配置 / 変更 | 0 |
| cron / crontab 変更 | 0 |
| F282 試走干渉 | 0 |
| F282 manual run | 0 |
| Paper Live 実実行 | 0 |
| VACUUM / VACUUM INTO | 0 |
| workflow / --no-verify / sudo / rm -rf / git push | 0 |
| TODO Excel 更新 | 0 |
| 楽天証券操作 / 自動発注 / Computer Use | 0 |
| Codex 直接 commit | 0 |
| temp smoke 干渉 | 0 (= 既存 W55/56.5/57/58 成果物不触) |

## §13 F282 不干渉確認

baseline (= W59-pre 開始 23:13 JST):
- F282 plist mtime=1778593597 / size=1772
- 3 DB / W30 snapshot: 既知値
- DATA-R3 / F062 plist: 未配置
- temp label: 不在
- pytest collected: 4499

完了時:
- F282 plist / 3 DB / W30 snapshot **完全一致 ✓**
- F282 next run 5/16 02:00 維持 ✓
- DATA-R3 / F062 plist 未配置維持 ✓
- pytest collected **4499 維持**

## §14 6 KPI

- Codex 稼働率: **0/8 = 0%** (= prescriptive design wave、W44.5-pre / W52.5-pre / W54-pre / W55-temp 等 precedent 継承)
- 短縮率 / 採用率: 該当なし
- 差戻率: 0
- Integrator 負荷: 中 (= 設計 doc + plan/results、約 14 章)
- 安全事故: 0 ✓

### Codex 0/8 の理由
- 本 wave は **docs / design / contract のみ**、Codex 並列の追加価値小
- 既存 W35-pre / W54-pre / W54-post で AFTER-R1 設計の基盤は固まっており、本 wave はそれを
  「gate 段階化 + input dependency map + output contract + safety + future tests +
  wave sequencing + v0 衝突確認」の 9 section に整理するだけ
- 局所 docs work (= scaffold / v0 本線不触)
- W44.5-pre / W52.5-pre / W54-pre / W55-temp / W56.5-temp / W57-temp / W58-temp と同じ
  「prescriptive design は本線完結」パターン

代替: 将来 W59-post として adversarial audit 8 lane 投入の選択肢を HQ 判断に残す
(= L1 design 開始前に gate / contract / safety を Codex 並列で再検証する余地)

## §15 残課題 / 次 Wave

### W59-pre で開ける道 (= L0 → L1 design)
- **W60-pre**: AFTER-R1 report-only read-only aggregator 設計 + tests (= L1 design)
- 着手条件: v0 D-Day 完了 (= 6/9 火、ただし設計のみなら前倒し可)

### Official GO 取得経路 (= 5/16-5/19 帰結)
1. 2026-05-16 02:00 本番 F282 launchd 自動起動
2. 2026-05-16 03:00 W44-pre drill + Step 4.5 Ops Summary smoke
3. 2026-05-19 F282 Official GO 判定 → Wave 41 着手前提

### 本番 path 継続
- Wave 41 (5/19-5/26): HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2): HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8): HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day): production_send 開始

### W54-pre TASK_REGISTRY expected_wave 文字列更新 (= 将来 minimal edit)
- 「Wave 55+」「Wave 56+」等の表記は概念上、impl 着手 wave で各 task の expected_wave
  文字列を最新番号に minimal edit する (= 大規模置換禁止)

---

## 関連リンク

- [[F286_AFTER_R1_night_paper_live_batch_2026-05-12|W35-pre AFTER-R1 設計]]
- [[F286_AFTER_R1_read_only_runner_design_2026-05-14|W54-pre / W54-post scaffold]]
- [[../02_todo/FIRE_CODEX_R1_WAVE59_PRE_plan|W59-pre plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE59_PRE_results|W59-pre results]]
- 既存 scaffold: `~/fire/scripts/jobs/run_f286_after_r1_night_batch.py` (= 不触)
