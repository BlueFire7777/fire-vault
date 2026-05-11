---
id: F286-PNL-R1
phase: P5 / 第 13 章 R-13-04 / 第 19 章 R-19-08 Phase 2
priority: 最優先
status: 完了 ★ (2026-05-11、advisory_decisions table + 118 tests + staging smoke)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R5 (= First Production Advisory Small Launch、受信確認済み)
  - F062-R5.8 (= action-first 文面、advisory_id 採用元)
chapter: 第 13 章 / 第 19 章 R-19-08 / 第 25 章 / 第 26 章
---

# F286-PNL-R1: Advisory Decision / Actual PnL Tracking

最終更新: 2026-05-11

## ★ 状態: 完了

FIRE 本番 Advisory で提示された候補について、Fujiwara の採用 / 見送り
判断、実際の売買、実現/含み損益を **staging DB に記録する評価基盤** を
新規実装。R-19-08 Phase 1 (Advisory 配信) → Phase 2 (Advisory →
判断 → 実約定 → PnL ループ) への基礎を整えた。

## 設計判断

### スキーマ (`advisory_decisions` table)

| 列 | 型 | 備考 |
|---|---|---|
| advisory_id        | TEXT NOT NULL    | F062 send_id (例: `f062-r5.8-2026-05-11T09:19:53Z`) |
| code               | TEXT NOT NULL    | 銘柄コード |
| base_date          | TEXT             | payload_base_date (2026-05-09 等) |
| source_version     | TEXT             | r2f4_baseline_live_v1 |
| rule_version       | TEXT             | r2g3_recommended_v2 |
| name               | TEXT             | 銘柄名 (F286-DATA-R1.7 enrichment) |
| advisory_label     | TEXT             | F119 ラベル (boost_with_avoid 等) |
| decision_label     | TEXT             | action mode ラベル (wait / buy_now 等) |
| fujiwara_decision  | TEXT NOT NULL    | adopted / watched / skipped / rejected / unknown (CHECK 制約) |
| decision_reason    | TEXT             | Fujiwara メモ |
| actual_trade       | TEXT NOT NULL    | none / buy / sell / partial (CHECK 制約) |
| entry_price        | REAL             | 約定価格 |
| entry_qty          | INTEGER          | 約定数量 |
| exit_price         | REAL             | 売却価格 |
| exit_qty           | INTEGER          | 売却数量 |
| realized_pnl       | REAL             | (exit-entry)*qty |
| unrealized_pnl     | REAL             | (current-entry)*held |
| paper_pnl          | REAL             | 「採用していたら」シミュ PnL |
| notes              | TEXT             | 自由記入 |
| created_at         | TEXT NOT NULL    | ISO timestamp |
| updated_at         | TEXT NOT NULL    | ISO timestamp |

PK: `(advisory_id, code)` (= 1 通知 / 1 銘柄 / 1 row)。
INDEX: `base_date`, `fujiwara_decision`。

### Module 構成

```
pnl/
├── __init__.py        - 公開 API (= ensure_schema は **export しない**)
├── models.py          - AdvisoryDecision + 値域定数 + __post_init__ validate
├── schema.py          - CREATE TABLE IF NOT EXISTS + 三段ガード ensure_schema
├── storage.py         - DecisionStore (read/write、三段ガード、atomic upsert)
├── pnl_calc.py        - realized / unrealized 純関数 (side='buy' 限定)
└── ingestion.py       - JSON / CSV ローダ (= row index 付き error 伝搬)

scripts/jobs/
└── run_f286_pnl_r1_record_decision.py  - runner (default dry-run)
```

### 三段ガード (= Codex CRITICAL 2 件対応の集大成)

| 段 | チェック | refuse 条件 |
|---|---|---|
| 1 | `read_only=False` 必須         | 構築時 read_only=True なら write 全て refuse |
| 2 | `db_label='staging'` 必須      | 'production' / 'develop' / 'other' は構築/parse で refuse |
| 3 | `db_path basename` 必須        | basename != 'fire.staging.db' なら parse/store/schema で refuse |

3 段とも独立してチェック (= caller がいずれを忘れても残り 2 段が
拾う)。CRITICAL #1 で staging 必須キーワード化、CRITICAL #2 で
basename ガード追加。

### runner の安全設計

- **default dry-run** (= --write 指定無しなら validation + schema 確認のみ)
- `--db-label staging` + `--db-path *fire.staging.db` 両方必須で --write が通る
- `--ensure-schema` で CREATE TABLE IF NOT EXISTS 冪等実行
- 注文価格 / 数量 / 執行指示の **生成 helper は含めない** (構造的禁止)
- argparse から forbidden options 全排除:
  `--send`、`--auto-order`、`--broker`、`--rakuten`、`--computer-use`、
  `--playwright`、`--token`、`--channel-token`

## 実施結果

### Step 1: dry-run smoke ✓

```
.venv/bin/python -m scripts.jobs.run_f286_pnl_r1_record_decision \
  --input-json /tmp/f286_pnl_r1_sample_input.json \
  --db-path data/fire.staging.db --db-label staging
```

| field | value |
|---|---|
| mode                | dry-run |
| decisions_loaded    | 5 |
| schema_info         | None |
| write_stats         | None |
| DB ファイル作成      | なし (= 未作成のまま) |
| 3 DB mtime          | 全 unchanged |

### Step 2: staging write smoke (= R5.8 受信 5 件) ✓

```
.venv/bin/python -m scripts.jobs.run_f286_pnl_r1_record_decision \
  --input-json /tmp/f286_pnl_r1_sample_input.json \
  --db-path data/fire.staging.db --db-label staging \
  --write --ensure-schema
```

| field | 1 回目 | 2 回目 (idempotent rerun) |
|---|---|---|
| schema_info.table_existed_before | False | True |
| write_stats.inserted             | 5 | 0 |
| write_stats.updated              | 0 | 5 |
| staging row count                | 5 | 5 (= 変化なし、idempotent) |

staging に書き込まれた 5 件:
```
('f062-r5.8-2026-05-11T09:19:53Z', '57290', '日本精鉱',     'watched', 'none')
('f062-r5.8-2026-05-11T09:19:53Z', '340A0', 'ジグザグ',       'watched', 'none')
('f062-r5.8-2026-05-11T09:19:53Z', '37980', 'ＵＬＳグループ',  'watched', 'none')
('f062-r5.8-2026-05-11T09:19:53Z', '137A0', 'Ｃｏｃｏｌｉｖｅ', 'watched', 'none')
('f062-r5.8-2026-05-11T09:19:53Z', '331A0', 'メディックス',     'watched', 'none')
```

### Step 3: 偽装 case refuse ✓

```
.venv/bin/python -m scripts.jobs.run_f286_pnl_r1_record_decision \
  --input-json /tmp/f286_pnl_r1_sample_input.json \
  --db-path data/fire.db --db-label staging --write --ensure-schema
```
→ `refused: F286-PNL-R1: --write には db_path basename が
  ('fire.staging.db',) のいずれか必須。 got --db-path 'data/fire.db'`
→ exit 2 (= parse_args 段で refuse)。production DB は touched なし。

### Step 4: 回帰 + Codex review

| 項目 | 結果 |
|---|---|
| 新規 R1 tests                | **118 件 PASS** |
| 全 pytest スイート            | **3,449 PASS** (= 3,331 baseline + 118) |
| Codex pre-commit feat        | CRITICAL #1 / #2 即修正後 OK |
| Codex pre-commit test        | OK |

## 安全要件遵守 (= 全 ✓)

| 項目 | 結果 |
|---|---|
| DB write は staging のみ                              | ✓ (= 三段ガード) |
| production / develop DB mtime                         | unchanged ✓ |
| 偽装 path (--db-path data/fire.db --db-label staging) | parse 段 refuse ✓ |
| default dry-run                                       | ✓ |
| 注文価格 / 数量 / 執行指示の生成                       | しない (構造的) ✓ |
| 自動発注 / 楽天操作 / Computer Use / Playwright       | なし ✓ |
| LINE 本番送信                                          | なし (LineBotClient 不 import) ✓ |
| token / channel_token / secret 読み込み               | なし ✓ |
| Workflow / GitHub Actions 変更                        | なし ✓ |
| scripts/seed_pattern_layer1.py の既存 modified         | 未接触 ✓ |
| simulation/research_lane/historical_indicators.py     | 未接触 ✓ |
| --no-verify                                          | 不使用 ✓ |
| TODO Excel                                            | 未更新 ✓ |

## Known Limitations (= Phase 2+ で対応)

1. **入力は手動 JSON / CSV のみ** — F062 payload からの自動 ingest は
   未実装 (= 別タスクで対応、F286-PNL-R2 候補)。
2. **PnL 計算は side='buy' (long) のみ** — 空売り (short) は将来別
   タスクで `side='sell'` を追加し、Codex review を経る。
3. **paper_pnl は手入力** — 「採用していたら」シミュレーションは
   F286-PNL-R3 で別実装 (= simulation/paper_live モジュール連携)。
4. **手数料 / 税金 計上なし** — gross PnL のみ。Stage 3 移行後に
   net PnL を別タスクで導入。
5. **三段ガードは structural** — `fire.staging.db.bak.*` のような
   バックアップファイルは basename 不一致で refuse される。
   将来 staging snapshot を作る場合は path normalization を追加する。

## HQ 判断が必要な論点

1. **入力データの正本管理** — advisory_id は F062 send_id ベースだが、
   現状 LINE log と payload metadata の間で UUID 統一無し。次タスク
   候補: F062 payload に send_id を埋め込み、F286-PNL-R1 入力と
   厳密一致させる。
2. **scope-A vs scope-B 拡張** — fujiwara_decision 値域は 5 値だが、
   将来「watch_extended」「partial_adopt」等を追加するかを判断
   (= 現状 CHECK 制約で固定値域)。
3. **paper_pnl の計算ソース** — Phase 2 で simulation/paper_live と
   連携するか、別 PoC pipeline を作るか。

## 次タスク候補

1. **F286-PNL-R2 Advisory Snapshot Auto-Ingest** — F062 payload を
   送信時に snapshot として保存し、advisory_decisions の入力として
   流す経路を整備 (= 手入力不要化)。
2. **F286-PNL-R3 Paper PnL Simulator Hook** — 「採用していたら」の
   paper_pnl を simulation/paper_live と連携して自動計算。
3. **F286-DATA-R3 Daily Refresh Cron** — staging データの日次更新
   cron 化 (= F282 weekly snapshot で消えないように)。
4. **F286-INTRA-R2 Intraday Advisory Trigger Engine** — 「待ち」候補
   が VWAP/出来高条件を満たした瞬間に 2 通目の LINE をトリガ。
5. **F286-ORDER-R1 Manual Order Draft Generator** — 手動発注の
   注文価格 / 数量 / OCO の draft を LINE 通知に追加 (= 自動発注
   ではなく「下書き」)。

### 並走候補

- F286-DATA-R1.8: F111-R4 persistence に `--listings-db` 追加
- FIRE-OPS-R0 再発防止策案 1: 本番運用データを production fire.db
  に書く運用統一
- 03_design/F282_environment_isolation 運用ルール明文化
