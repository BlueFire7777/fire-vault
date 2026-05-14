---
id: F286-AFTER-R1-paper-live-mvp-design
phase: 本番 v0 中核 / Wave 60-impl / Paper Live MVP & Pattern Candidate
priority: 高
status: design v1.0
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 03_design/F286_AFTER_R1_paper_live_report_gate_2026-05-14.md (= W59-pre L0)
  - 03_design/F286_AFTER_R1_read_only_runner_design_2026-05-14.md (= W54-pre/post)
---

# F286-AFTER-R1 Paper Live MVP / Pattern Candidate Implementation Design v1.0

## §1 目的

FIRE 本番 v0 の **中核** として、夜間に走る Paper Live MVP を実装する。
F062 推奨銘柄 preview / DATA-R3 freshness / Ops Summary / readiness を
read-only で取り込み、`reports/after_r1/` 配下に **4 種類** の成果物を生成する:

1. `paper_live_ledger_<base_date>.{json,md}`
2. `good_candidate_ranking_<base_date>.{json,md}`
3. `pattern_candidate_report_<base_date>.{json,md}`
4. `morning_line_material_<base_date>.{json,md}`

これにより、翌朝 F062/LINE 通知へ渡せる「買い候補 / 理由 / 当たりパターン候補 /
手動売買目安」を生成する。

## §2 v0 中核への繰り上げ (= HQ 方針変更)

W59-pre までは AFTER-R1 を **v0 後拡張** として L1-L5 の段階的実装と位置付けて
いたが、HQ 方針変更により以下の通り再定義:

| 旧定義 (W59-pre) | 新定義 (W60-impl) |
|---|---|
| AFTER-R1 = v0 後拡張、L0-L5 段階 | AFTER-R1 MVP = v0 **中核** |
| L1 = Read-Only Aggregator (v0 D-Day 後) | MVP = 即実装、read-only / no-write |
| L3 = Paper-Live Real (= 二段階 HQ 承認) | Paper-Live Real は将来 wave、変更なし |

v0 中核 = 「**夜間に Paper Live MVP が回り、良銘柄を見つけ、当たりパターンを
抽出し、朝 LINE で材料を渡せる**」状態。

## §3 やらないこと (= 安全境界)

- DB write / DB sqlite 接続 (production/develop/staging)
- LINE 実送信 (line-bot-sdk import すらしない)
- token / channel_token / secret / .env 全体参照
- API call (requests / urllib / aiohttp 等)
- launchctl / plist 配置・変更 / cron / crontab 変更
- VACUUM / VACUUM INTO
- Paper Live **Real** 実行 (= MVP は read-only シミュレーションのみ)
- brokerage / 自動発注 / Computer Use
- workflow / --no-verify / git push / sudo / rm -rf

## §4 アーキテクチャ

### §4.1 入出力フロー

```
[入力 read-only]
  F062 preview JSON           → 候補 row 群 (= chunks + selected_rows)
  F062 preview summary JSON   → label_counts / forbidden / safety_footer
  DATA-R3 freshness JSON      → verdict (OK/FAILED/MISSING)
  Ops Summary JSON            → readiness phase / verdict
  readiness JSON (任意)       → cumulative phase / version
  optional ranking JSON       → existing rank score (fallback)
  optional watchlist JSON     → existing watchlist tickers (fallback)
                ↓
       [MVP entry: run_f286_after_r1_night_batch.py --mode mvp]
                ↓
       [load_mvp_inputs] read-only safe-warn for missing
                ↓
       [compute_good_candidate_ranking] score 付与 + sort
                ↓
       ┌─────────────────┬────────────────────┬──────────────────┐
       ↓                 ↓                    ↓                  ↓
  paper_live_ledger  good_candidate_     pattern_candidate_  morning_line_
                     ranking             report              material
       ↓                 ↓                    ↓                  ↓
  reports/after_r1/<artifact>_<base_date>.{json,md}
```

### §4.2 task 拡張

W54-pre TASK_REGISTRY (6 task: paper-live / replay / simulation / lane-eval /
pattern / report) に **`morning-material`** を追加し計 7 task。

| task | MVP mode 出力 | design preview mode |
|---|---|---|
| paper-live | paper_live_ledger | (W54-pre のまま) |
| report | good_candidate_ranking (= report の MVP 中身) | (W54-pre) |
| pattern | pattern_candidate_report | (W54-pre) |
| morning-material (**新規**) | morning_line_material | (= MVP 専用、設計 preview なし) |
| replay / simulation / lane-eval | (= MVP 未対応、preview のみ) | (W54-pre) |
| all | 上記 4 種全部 | 全 6 task preview (W54-pre) |

### §4.3 mode 拡張

- `--mode design-preview` (default): W54-pre/post 既存挙動 (= scaffold 維持)
- `--mode mvp`: 新規実装、MVP 出力生成

互換性: default は **design-preview** で W54-pre/post と完全互換。tests も互換維持。

## §5 input resolver (= Lane A)

### §5.1 入力 source 優先度

| 優先 | source | 必須? | 形式 |
|---|---|---|---|
| 1 | F062 preview JSON | 候補抽出に必須 | JSON list[row] or {chunks, selected_rows, ...} |
| 1 | F062 preview summary JSON | 任意 (= 補助) | JSON dict |
| 2 | DATA-R3 freshness JSON | 任意 | JSON dict {verdict, aggregate_exit_code, ...} |
| 2 | Ops Summary JSON | 任意 | JSON dict {phase, verdict, ...} |
| 2 | readiness JSON | 任意 | JSON dict {phase, version, ...} |
| 3 | ranking / watchlist (任意) | 任意 | JSON list/dict |

### §5.2 missing input handling

- F062 preview JSON が **完全に空** または missing → warn + empty MVP 出力
- F062 summary 不在 → 補助情報省略、safe-warn
- DATA-R3 freshness FAILED → risk_warning に降格、候補から積極推奨を外す
- DATA-R3 freshness MISSING → 候補生成は継続するが warning に明記
- Ops / readiness 不在 → 表示のみで、生成は継続

### §5.3 InputContext dataclass

```python
@dataclass(frozen=True)
class MvpInputContext:
    base_date: date
    generated_at: datetime
    f062_rows: tuple[dict, ...]
    f062_summary: Optional[dict]
    data_r3_freshness: Optional[dict]
    ops_summary: Optional[dict]
    readiness: Optional[dict]
    optional_ranking: Optional[dict]
    optional_watchlist: Optional[dict]
    source_artifacts: tuple[dict, ...]
    warnings: tuple[str, ...]
```

`source_artifacts` 各 element = `{type, path, status, file_size, mtime,
schema_version (if available)}`。

### §5.4 read-only guarantees

- JSON 読み込みは `Path.read_text()` + `json.loads()` のみ
- `open()` mode = `"r"` 固定 (= write/append 一切なし)
- sqlite3 import なし
- requests / urllib / aiohttp / linebot import なし
- 全 input artifact path は是 read で開く、write しない

## §6 good_candidate_ranking (= Lane C)

### §6.1 scoring formula (MVP)

各 row に score を付与:

```
score = (
    confidence_score * 100      # F062 expected_h20 / 評価 score (0.0-1.0)
    + reason_tag_count * 5      # reason の数 (= 多根拠 bonus)
    + freshness_bonus           # OK=10, FAILED=-20, MISSING=-5, None=0
    + action_label_priority     # boost=50, boost_with_caution=25,
                                # boost_with_avoid=10, caution=5, neutral=0,
                                # avoid=-30, suppress=-50, missing=0
    + manual_review_bonus       # manual_review_required=True → +5
                                # (= safety footprint)
    + existing_rank_score       # fallback ranking 由来 (= -post_cap_rank、0-30)
    + missing_data_penalty      # name 空 → -3, label missing → -10
)
```

### §6.2 出力 schema

```json
{
  "schema_version": "1.0",
  "artifact": "good_candidate_ranking",
  "generated_at": "2026-05-13T23:30:00+09:00",
  "base_date": "2026-05-14",
  "mode": "mvp",
  "source_artifacts": [...],
  "ranking": [
    {
      "rank": 1,
      "ticker": "7203",
      "name": "トヨタ自動車",
      "score": 145.5,
      "action_label": "boost",
      "why_selected": ["高 confidence", "freshness OK", "複数根拠"],
      "risk_notes": [],
      "pattern_tags": ["freshness_ok_high_confidence"],
      "manual_buy_checklist": [
        "場中の出来高傾向確認",
        "寄付き直前のニュース再確認",
        "stop_loss 水準の事前メモ"
      ]
    },
    ...
  ],
  "summary": {
    "total_candidates": 20,
    "boost_count": 3,
    "boost_with_caution_count": 5,
    "boost_with_avoid_count": 2,
    "avoid_count": 4,
    "suppress_count": 6
  },
  "safety_flags": {...},
  "warnings": [...]
}
```

### §6.3 Markdown

iPhone コピー対応の **単一 fence** 形式 (= MEMORY.md feedback 準拠)。

## §7 paper_live_ledger (= Lane B)

### §7.1 schema

```json
{
  "schema_version": "1.0",
  "artifact": "paper_live_ledger",
  "generated_at": "...",
  "base_date": "2026-05-14",
  "mode": "dry_run_mvp",
  "source_artifacts": [...],
  "ledger_entries": [
    {
      "ledger_id": "PLL-2026-05-14-001",
      "ticker": "7203",
      "name": "トヨタ自動車",
      "action_label": "boost",
      "entry_policy_preview": "寄付き成行 (= 仮想)",
      "exit_policy_preview": "h20 終値時点 (= 仮想)",
      "stop_loss_preview": "5%下落",
      "take_profit_preview": "3%上昇 or h20",
      "confidence": 0.82,
      "reason_tags": ["決算ポジティブ", "出来高伴う上昇"],
      "source_rank": 1,
      "freshness_report_id": "data_r3_freshness_2026-05-13",
      "advisory_text_hash": "sha256:...",
      "manual_review_required": true,
      "auto_order_allowed": false,
      "paper_live_status": "planned_preview",
      "real_trade_status": "not_executed"
    },
    ...
  ],
  "summary": {
    "total_entries": 20,
    "boost_entries": 3,
    "high_confidence_entries": 5,
    "all_manual_review_required": true,
    "all_auto_order_disallowed": true
  },
  "safety_flags": {12 keys 全 false},
  "next_actions": [
    "朝 F062 preview で再確認",
    "DATA-R3 freshness 最新化確認",
    "Fujiwara 手動買い判断"
  ]
}
```

### §7.2 advisory_text_hash

各 ledger entry の advisory_text は SHA-256 で hash 化して `advisory_text_hash`
に保存。本文は保存しない (= privacy / size 軽減)。

## §8 pattern_candidate_report (= Lane D)

### §8.1 候補 pattern 群 (MVP)

| pattern_id | description | matching condition |
|---|---|---|
| `freshness_ok_high_confidence` | freshness OK + confidence ≥ 0.7 | F062 row & DATA-R3 OK |
| `manual_review_active_label` | manual_review_required + boost / boost_with_caution | F062 row 条件 |
| `multi_reason_basis` | reason_tags.length ≥ 3 | F062 row 条件 |
| `sector_concentration` | 同 sector で 3 銘柄以上 boost | F062 rows aggregate (sector 取得可能時) |
| `momentum_data_available` | (将来) price / return data 存在 | 別 artifact 依存 |
| `low_risk_note` | risk_notes 空 | F062 row 条件 |

### §8.2 schema

```json
{
  "schema_version": "1.0",
  "artifact": "pattern_candidate_report",
  "generated_at": "...",
  "base_date": "2026-05-14",
  "mode": "mvp",
  "source_artifacts": [...],
  "patterns": [
    {
      "pattern_id": "freshness_ok_high_confidence",
      "description": "freshness OK + high confidence (≥0.7)",
      "matched_tickers": ["7203", "9984", "6758"],
      "matched_count": 3,
      "input_fields": ["confidence", "data_r3_verdict"],
      "expected_edge": "再現性候補 (= validation 必要)",
      "required_future_data": ["historical_return_h20", "actual_pnl"],
      "status": "candidate_only",
      "next_validation_method": "Wave 60+ で Replay/Paper PnL 適用"
    },
    ...
  ],
  "summary": {
    "total_patterns": 6,
    "matched_patterns": 4,
    "all_candidate_only": true
  },
  "safety_flags": {...},
  "warnings": [...]
}
```

## §9 morning_line_material (= Lane E)

### §9.1 contents

翌朝 F062 + LINE 通知に渡せる「材料」。LINE 送信は **しない**。

```json
{
  "schema_version": "1.0",
  "artifact": "morning_line_material",
  "generated_at": "...",
  "base_date": "2026-05-14",
  "mode": "mvp",
  "source_artifacts": [...],
  "top_candidates": [
    {
      "rank": 1,
      "ticker": "7203",
      "name": "トヨタ自動車",
      "label": "積極的買い推奨",
      "label_emoji": "🟢",
      "score": 145.5,
      "entry_guideline": "寄付き成行 or 始値±0.5%以内",
      "stop_loss_guideline": "5%下落で撤退",
      "take_profit_guideline": "3%上昇 or h20 終値",
      "skip_conditions": ["寄付き直前にネガティブニュース", "出来高薄"],
      "caution_notes": ["材料の再現性は別途検証中"]
    }
  ],
  "alternative_labels_recap": {
    "条件付き買い推奨": 5,
    "場中監視": 2,
    "見送り推奨": 10
  },
  "safety_footer": "本通知は手動売買判断補助のみ。自動発注なし。最終判断は Fujiwara。",
  "manual_review_required": true,
  "auto_order_allowed": false,
  "safety_flags": {...},
  "forbidden_phrases_check": {
    "passed": true,
    "forbidden_terms_checked": ["買え", "売れ", "全力", "確実"]
  }
}
```

### §9.2 命令形 / 強制表現の禁止 (= FORBIDDEN_PHRASES 拡張)

| 拒否 | 採用 |
|---|---|
| 「買え」「売れ」 | 「積極的買い推奨」「条件付き買い推奨」「見送り推奨」 |
| 「全力で」「確実に」 | 「再現性候補」「目安」「参考」 |
| 「絶対」「必ず」 | 「推奨」「想定」 |

morning_line_material 生成時に **必ず** forbidden check を通す。違反検出 → exit 3 / 警告。

### §9.3 label mapping (= F062 label → 朝 LINE 材料 label)

| F062 label | 朝 LINE label |
|---|---|
| `boost` | 🟢 積極的買い推奨 |
| `boost_with_caution` | 🟡 条件付き買い推奨 |
| `boost_with_avoid` | 🟠 場中監視 |
| `caution` | ⚠️ 注意つき買い候補 |
| `neutral` | (= top 候補に含めない) |
| `avoid` | 🔴 見送り推奨 |
| `suppress` | 🔴 見送り推奨 |

## §10 output guard (= Lane G)

許可 prefix:
- `reports/after_r1/`
- `/tmp/` / `/var/folders/` (= tests)
- `/Users/<user>/fire/reports/after_r1/`

拒否 segment (W52-post / W44.6-post 既存):
- `/data/`
- `/.git/`
- `/.github/`
- `/LaunchAgents/`
- `/.fire_secrets/`
- `/reports/dashboard/` (= 別 wave)
- `/patterns/` (= 別 wave)

is_safe_output_path() は W54-pre 既存実装をそのまま流用。

## §11 safety_flags (= 12 keys 全 false)

| flag | 値 |
|---|---|
| `db_write` | false |
| `db_sqlite_connect` | false |
| `line_send` | false |
| `token_access` | false |
| `api_call` | false |
| `launchctl_call` | false |
| `plist_modified` | false |
| `cron_modified` | false |
| `vacuum_executed` | false |
| `order_automation` | false |
| `production_data_modified` | false |
| `paper_live_executed` | false (= MVP は Real 実行しない) |

MappingProxyType で immutable 化済 (= W54-post Lane C fix)。

## §12 tests 設計 (= Lane F)

### §12.1 安全系 (= 必須、12 件以上)

1. default mode = design-preview (= W54-pre/post 互換維持)
2. `--mode mvp --task all --dry-run` → 4 種類生成
3. DB write 0 (= ファイル open mode "r" のみ)
4. sqlite3 import なし (= ast scan)
5. LINE/linebot import なし
6. requests / urllib / aiohttp import なし
7. token / channel_token / secret 参照 0
8. launchctl / subprocess.run / VACUUM call なし
9. output path guard (= safe-prefix 限定、forbidden 拒否)
10. safety_flags 12 keys 全 false 保証
11. brokerage / order automation 0
12. forbidden phrase 検出 (= morning_line_material)

### §12.2 機能系 (= 8 件以上)

13. F062 preview から `good_candidate_ranking` 生成
14. F062 preview から `paper_live_ledger` 生成
15. F062 preview から `pattern_candidate_report` 生成
16. F062 preview から `morning_line_material` 生成
17. missing F062 → safe-warn + 空出力 (=空ledger / empty material)
18. DATA-R3 FAILED → risk_warning + boost 候補抑制
19. action_label mapping (= F062 label → 朝 LINE label)
20. ranking score 順序保証 (= sort stable)

### §12.3 chain integration 系 (= 5 件以上)

21. F062 + DATA-R3 + Ops Summary 統合 → 4 出力生成
22. JSON schema validation (= keys 存在確認)
23. Markdown output 存在確認
24. source_artifacts に全 input path / mtime 記録
25. all_manual_review_required = true / all_auto_order_disallowed = true 保証

合計 **25+ tests**。

## §13 CLI 仕様

```
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \\
  --mode mvp \\
  --task all \\
  --base-date 2026-05-14 \\
  --f062-preview-json /path/to/f062_preview.json \\
  --f062-preview-summary-json /path/to/f062_summary.json \\
  --data-r3-freshness-json /tmp/f286_data_r3_freshness.json \\
  --ops-summary-json /tmp/ops_summary.json \\
  --readiness-json /tmp/readiness.json \\
  --output-dir reports/after_r1 \\
  --dry-run --read-only --strict
```

引数:
- `--mode {design-preview, mvp}` default=design-preview
- `--task {paper-live, replay, simulation, lane-eval, pattern, report, morning-material, all}`
- `--base-date YYYY-MM-DD`
- `--f062-preview-json PATH` (任意、MVP mode で意味あり)
- `--f062-preview-summary-json PATH`
- `--data-r3-freshness-json PATH`
- `--ops-summary-json PATH`
- `--readiness-json PATH`
- `--output-dir PATH` default=`reports/after_r1`
- `--dry-run` (default true、上書き不可)
- `--read-only` (default true、上書き不可)
- `--strict`

## §14 v0 衝突確認

| file | 触る? |
|---|---|
| run_f286_after_r1_night_batch.py (= W54-pre scaffold) | **拡張** (互換維持) |
| test_run_f286_after_r1_night_batch.py | **拡張** (既存 tests 互換維持) |
| F062 preview runner | 不触 |
| F286 DATA-R3 runner | 不触 |
| readiness CLI / Ops Summary CLI / wrapper preview | 不触 |
| F282 weekly snapshot | 不触 |
| production / develop / staging DB | 不触 |
| launchd plist / cron | 不触 |
| notifications/ (= F062 templates) | 不触 |

## §15 future wave roadmap (= W59-pre §9 更新)

| Wave | 内容 |
|---|---|
| **W60-impl (= 本 wave)** | Paper Live MVP + Pattern Candidate impl |
| W61-impl | price/return 取り込み + paper_pnl 連携 (= MVP 拡張、read-only) |
| W62-impl | historical replay (= 過去 N 日 batch、read-only) |
| W63 | lane evaluation 拡張 (= advisory_decisions 蓄積後) |
| W64+ | Pattern Store 連携 + ML feature export |

L3 Paper-Live Real / L4 / L5 は W59-pre L3 design に従い継続待機。
本 wave は **L1 (Read-Only Aggregator) + 一部 L2 (Paper-Live Dry-Run preview)** の
**MVP 統合** 実装。

## §16 unresolved items

- price / return data の input artifact format (= 仮 schema、W61 で確定)
- pattern_candidate_report の expected_edge 計算 (= Replay 適用後に Wave 61+)
- morning_line_material の F062 直結 (= 朝 batch 内で F062 が読む形式、別 wave 設計)
- advisory_text の保存 vs hash のみ (= 現状 hash のみ、確認用 query で展開する設計は別 wave)

---

## 関連リンク

- [[F286_AFTER_R1_paper_live_report_gate_2026-05-14]] (W59-pre)
- [[F286_AFTER_R1_read_only_runner_design_2026-05-14]] (W54-pre/post)
- [[F286_AFTER_R1_night_paper_live_batch_2026-05-12]] (W35-pre)
