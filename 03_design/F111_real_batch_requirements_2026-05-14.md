---
id: F111-real-batch-requirements
phase: 本番 v0 中核 / W1 集約 / F111-real-batch 要件
priority: 高 (= next wave 着手前提)
status: design v1.0
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W1_results.md
  - 03_design/F111_real_artifact_contract_2026-05-14.md (= W60-F111-pre)
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
---

# F111 Real Batch Requirements v1.0

## §1 目的

D1-D5 pilot で実 entry 0/5 (= 全 day skip) となった**最大の構造的 blocker**:
- D1/D2: synthetic_fixture
- D3/D4: manual_seed (= 藤原さん手作り)、値嵩株リスクで 100 株 entry 困難
- D5: f111_sample (= F111 SAMPLE_CANDIDATES、サンプル ticker で実 trade 不可)

これを解消するために **F111-real-batch wave** で staging real candidates から
**実在する日本株 ticker** の候補生成 path を実装する。

## §2 D1-D5 blockers 要約

| Day | blocker | 解消 wave |
|---|---|---|
| D1 | synthetic_fixture (= MVP 直渡し) | W60-integration ✓ (= f062_preview 化) |
| D2 | synthetic_fixture (= 同上) | W60-integration ✓ |
| D3 | manual_seed (= 手作り) / 値嵩株 100 株 上限超過 | **F111-real-batch (= 本 wave)** |
| D4 | manual_seed (= 同) / 値嵩株 / D3 同候補 | **F111-real-batch** |
| D5 | f111_sample / **サンプル ticker 構造的 trade 不可** | **F111-real-batch** |

## §3 F111-real-batch 必須要件

### §3.1 候補 universe

- ✅ **実在する日本株 ticker** のみ (= 楽天証券で発注可)
- ❌ サンプル prefix 除外 (= `_template_`、`サンプル`、`1234` 等の placeholder)
- ✅ **tradable universe filter**: 上場銘柄 / 整理銘柄除外 / 不停止 / 板厚 minimum
- ✅ ticker → name の解決可能 (= F286-DATA-R1.7 name enrichment 連携可)

### §3.2 risk filter

| 要件 | 値 |
|---|---|
| 1 株 想定リスク (= entry − stop_loss) | F062 row に明示 |
| **100 株 想定損失 ≤ 15,000 円** | risk_within_pilot_limit=True |
| 100 株 想定損失 > 15,000 円 | risk_within_pilot_limit=False / value_stock_flag=True |
| 出来高 minimum | 平均 5,000 株以上 想定 |
| 決算 / 重要 IR 直前 | event_risk_flag=True |
| 不停止 / 整理銘柄 | universe 段階で除外 |

risk filter で `risk_within_pilot_limit=False` なら **AFTER-R1 で suppress** または
risk_notes に「100 株単位リスク上限超過、pilot 不適」を明示。

### §3.3 F111 output 必須 fields

W60-F111-pre で F111 sample が 32 fields 出力。real_batch では追加:
- `tradable_universe`: True/False
- `realtime_market_cap`: 時価総額 (= staging から取得)
- `estimated_100_share_risk`: 1 株リスク × 100
- `risk_within_pilot_limit`: True/False
- `value_stock_flag`: True/False (= 株価 ≥ 10,000 円目安)
- `event_risk_flag`: True/False
- `f111_input_source`: "f111_real_batch"

### §3.4 自動推定 marker (= AFTER-R1 側)

```python
F111_REAL_BATCH_MARKERS = (
    "tradable_universe", "estimated_100_share_risk",
    "risk_within_pilot_limit", "realtime_market_cap",
)
```

これら 3 件以上含むなら `f111_input_source = f111_real_batch` 自動推定 (= W60-F111-pre
pattern 継承)。

### §3.5 8 invariants 拡張案 (= 9 invariants)

W60-F111-pre 8 invariants に追加:

| # | invariant | 説明 |
|---|---|---|
| 1-8 | (W60-launchd-pre 7 + W60-F111-pre 1) | 既存 |
| **9** | **all top candidates have `risk_within_pilot_limit=True`** | **NEW** |

`risk_within_pilot_limit=False` の候補が top に混入していたら **HOLD/skip 推奨**。

### §3.6 safety 要件 (= W60-F111-pre 継承)

- F111 朝 batch **DB write 0** (= read-only staging access のみ)
- LINE 送信 / token / API / launchctl 0
- auto_order_allowed=False / manual_review_required=True 全 row 維持
- safety_flags 13 keys 全 False 維持

## §4 F062 handoff path

F111 real_batch output → F062 LINE preview runner `--preview-json` に直接渡し:

```bash
.venv/bin/python -m scripts.jobs.run_f111_research_advisory_preview \
  --candidate-json <staging から fetch した実 candidates>.json \
  --dry-run \
  --output-json /tmp/fire_realbatch_prep/f111_real_batch_$DATE.json

.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/fire_realbatch_prep/f111_real_batch_$DATE.json \
  ...
```

F111 `--candidate-json` 入力経路は既存実装あり。問題は **staging real candidates の
fetch path** (= W60-F111-real-batch wave で実装)。

### §4.1 staging fetch options

- Option A: 既存 staging DB の F111 candidate table (= 存在すれば SELECT のみ)
- Option B: 別 helper runner で staging から実 candidates を抽出 (= read-only)
- Option C: 朝 batch 化 (= launchd 連携、別 wave)

最小実装: **Option A** (= staging table 存在確認後、SELECT-only read)。

### §4.2 staging 接続安全要件

- staging DB のみ (= production-develop は接続禁止)
- read-only mode (`SQLITE_OPEN_READONLY` 相当)
- 書き込み API call 禁止
- 接続失敗時は safe-warn + f111_input_source=unknown で fallback (= NO-GO)

## §5 W61-pre 連携要件 (= price/return/paper_pnl)

### §5.1 W61-pre 内容

F111-real-batch で実候補生成後、**実 trade ↔ outcome 評価** path:
- 各 candidate の **actual close / next-day return / h1/h5/h20 return**
- **paper_pnl ledger** (= 仮想 entry/exit、real return ベース)
- manual review pnl (= 藤原さん実 trade pnl)
- **pattern outcome** (= D3-D5 で watched pattern の hit rate)
- pattern promote/suppress/watch **update**

### §5.2 優先順位

| Wave | 内容 | 優先度 |
|---|---|---|
| F111-real-batch (= 本 wave 起点) | 実 ticker 候補生成 | **最高** |
| W60-pilot-D6 | real_batch 由来 trade plan | 高 |
| W61-pre | price/return/paper_pnl 連携 | 中 |
| W60-launchd-real | 本番 launchd 配置 | 低 (= Wave 41/45 後) |

F111-real-batch 完了 → D6 で実 trade 可能候補生成 → W61-pre で実 trade 評価。

## §6 F111-real-batch wave 実装範囲 (= 提案)

### §6.1 minimum viable

1. F111 runner に `--from-staging` flag 追加 (= staging から candidates SELECT)
2. staging real candidates → F111 output (= 32+5 fields)
3. AFTER-R1 MVP に F111_REAL_BATCH_MARKERS 推定 logic 追加
4. CLI `--f111-input-source f111_real_batch` 強制 path
5. risk filter (= 100 株 上限) を F111 output に組み込み
6. tests 追加 (= 15+ 件)
7. Codex 4 lane audit

### §6.2 scope 外 (= 別 wave へ)

- F111 朝 batch launchd 化 (= W60-launchd-real)
- price/return/paper_pnl 連携 (= W61-pre)
- production_send (= Wave 53 D-Day)

## §7 残課題

- staging real candidates の table structure 確認 (= F111-real-batch wave Step 0)
- F111 runner 既存 `--candidate-json` の input format との互換性
- risk filter の閾値調整 (= 1 トレード 15,000 円固定 / 動的 / 銘柄別)
- value_stock_flag の閾値 (= 株価 10,000 円? 5,000 円? 板厚 base?)

## §8 安全境界

- 本 doc は read-only docs、F111 / F062 / AFTER-R1 / F282 本体不触
- F111-real-batch wave 着手時に staging read-only / no-write が前提
- 実装範囲は read-only / no-token / no-LINE / no-launchctl

---

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W1_results|W1 集約 results]]
- [[F111_real_artifact_contract_2026-05-14|F111 real artifact contract (W60-F111-pre)]]
- [[FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
