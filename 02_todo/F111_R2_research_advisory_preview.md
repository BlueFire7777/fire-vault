# F111-R2 Research Advisory Output Preview / Smoke Runner

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: F111-R1 ResearchAdvisory metadata (07453f7 / f4a63f1 / 1a8b03b)
> **Mode**: 完全 in-memory / read-only (DB write 一切なし、staging も未使用)
> **Result**: ★★★ DaytradeCandidate × ResearchAdvisory を JSON / CSV /
> preview text / summary に整形する純関数群 + 安全強制 read-only CLI runner
> 完成。--sample 6 ケース smoke で auto_order_allowed_true_count=0 /
> manual_review_required_count=6 / DB 3 件全 unchanged を確認。★★★

---

## タスク名

F111-R2 Research Advisory Output Preview / Smoke Runner

---

## 背景

F111-R1 で以下が完了済み:

- agents/research_advisory.py (ResearchAdvisory dataclass + builder)
- agents/daytrade_selection.py (DaytradeCandidate.advisory wiring +
  attach_advisories ヘルパ)
- ResearchAdvisory.__post_init__ で manual_review_required=True /
  auto_order_allowed=False を強制矯正

次は ResearchAdvisory 付き DaytradeCandidate を **Fujiwara が手動レビュー
できる preview output** に整形する。LINE 本番送信 / 注文生成 / 楽天証券
操作 / Computer Use は一切行わない。preview / smoke runner 専用。

---

## F111-R1 からの接続内容

| F111-R1 で用意した素 | F111-R2 で何をするか |
|---|---|
| ResearchAdvisory dataclass (21 フィールド) | row dict → JSON / CSV / preview text に変換 |
| __post_init__ 安全強制 | preview 出力でも再強制 (= 二重防御) |
| build_advisory 純関数 | 入力源として呼ばれるが本タスクでは使わない (= advisory 作成済み前提) |
| DaytradeCandidate.advisory フィールド | dataclass / dict 両対応で受ける |
| compute_advisory_label / boost / avoid / caution flags | label 8 種に分類 + summary に counts |

---

## 実装ファイル一覧

| 種別 | ファイル | 行数 |
|---|---|---|
| feat | `agents/research_advisory_preview.py` | 541 |
| chore (runner) | `scripts/jobs/run_f111_research_advisory_preview.py` | 546 |
| test | `tests/agents/test_research_advisory_preview.py` | (formatter 42 PASS) |
| test | `tests/scripts/jobs/test_run_f111_research_advisory_preview.py` | (runner 26 PASS) |

### モジュール構成

    agents/
    └── research_advisory_preview.py   (541 行、新規)
        ├── ROW_FIELDS (31 項目、JSON/CSV 順序固定)
        ├── ALL_ADVISORY_LABELS (8 種:
        │   boost / boost_with_caution / boost_with_avoid /
        │   neutral / caution / avoid / suppress / missing_advisory)
        ├── PREVIEW_HEADER_LINES / PREVIEW_SAFETY_LINES /
        │   SAFETY_NOTES (再利用可能な定数化)
        ├── _normalize_candidate (Mapping / dataclass 両対応)
        ├── _extract_advisory (None safe)
        ├── _coerce_str / _coerce_int / _coerce_float /
        │   _coerce_str_list (型ガード)
        ├── compute_advisory_label (suppress > avoid+boost > avoid >
        │   boost+caution > boost > caution > neutral / missing)
        ├── advisory_to_row (= 安全強制 row 化、★ 中核)
        ├── candidates_to_rows
        ├── row_to_csv_row (list → ; 連結、bool → "true"/"false")
        ├── render_candidate_block (multi-line preview)
        ├── build_preview_text (header + 全ブロック + safety lines)
        ├── _top_candidates_by_flag
        ├── build_summary (counts + top_*_candidates + safety_notes)
        └── AdvisoryPreviewSafetyViolation (caller fail 用例外)

    scripts/jobs/
    └── run_f111_research_advisory_preview.py   (546 行、新規)
        ├── F111R2RunRefused (CLI / loader 例外)
        ├── SAMPLE_CANDIDATES (6 ケース、F119 主要結果対応)
        ├── load_candidates_from_json (list 検証付き)
        ├── load_candidates_from_csv (advisory 子 dict 化)
        ├── run_preview (= 中核、safety violation で raise)
        ├── build_arg_parser (LINE / broker / rakuten / write
        │   option を意図的に作らない)
        ├── parse_args (1 入力源強制)
        └── main (refused → exit 2、violation → exit 3、success → 0)

    tests/  (68 PASS = 42 + 26)
    ├── tests/agents/test_research_advisory_preview.py (42)
    │   - compute_advisory_label / advisory_to_row safety /
    │     row_to_csv_row / build_preview_text / build_summary /
    │     safety violation / render_candidate_block
    └── tests/scripts/jobs/test_run_f111_research_advisory_preview.py (26)
        - parse_args / SAMPLE_CANDIDATES 構造 / run_preview 統合 /
          load_*_from_json/csv / main exit code /
          runner source に LINE / broker / sqlite3 が無いこと

---

## commit hash 一覧

| commit | type | 内容 |
|---|---|---|
| `c382a38` | feat | F111-R2: add research advisory preview formatter |
| `16a3a07` | chore | F111-R2: add research advisory preview runner |
| `424c5fb` | test | F111-R2: add research advisory preview tests |
| (本 commit) | docs | F111-R2: vault |
| (次 commit) | docs | F111-R2: log milestone |

---

## preview output 仕様

### 1. JSON (--output-json)

`list[dict]` 形式。各 row は ROW_FIELDS の 31 項目を固定順で持つ。

    [
      {
        "code": "1234",
        "name": "サンプル情報通信A",
        "side": "long",
        "score": 0.82,
        "rank": 1,
        "reason": "F119 normal × 情報通信 × 5月 boost",
        "research_advisory_version": "F111-R1-v1",
        "research_source_version": "r2f4_baseline_v1",
        "research_rule_version": "r2g3_recommended_v2",
        "research_base_date": "2026-05-08",
        "research_final_score": 0.842,
        "research_post_cap_rank": 18,
        "research_rank_label": "A2",
        "research_interpretation": "use_signal_normal",
        ...,
        "f119_boost_flags": ["interpretation_sector_17_month:..."],
        "f119_avoid_flags": [],
        "f119_caution_flags": ["interpretation_sector_17_month:..."],
        ...,
        "research_advisory_label": "boost_with_caution",
        "advisory_comment": "F119上は強い... 短期売買判断には使わない。",
        "manual_review_required": true,    ← 必ず true
        "auto_order_allowed": false        ← 必ず false
      },
      ...
    ]

### 2. CSV (--output-csv)

ヘッダは ROW_FIELDS 順。list は ";" 連結、bool は "true"/"false"、None は空文字。

    code,name,side,score,rank,reason,research_advisory_version,
    research_source_version,...,manual_review_required,auto_order_allowed
    1234,サンプル情報通信A,long,0.82,1,...,true,false
    ...

### 3. preview text (--output-text)

Fujiwara が読みやすい multi-line 形式 (Markdown 風 / 純テキスト):

    F111 Research Advisory Preview
    mode: preview only / no LINE send / no order
    manual review required: true
    auto order allowed: false
    generated_at: 2026-05-10T04:55:00.247731+00:00
    candidate_count: 6

    [1] 1234 サンプル情報通信A
    - advisory: boost_with_caution
    - rank: A2 / post_cap_rank 18 / final_score 0.842
    - interpretation: use_signal_normal (normal_default)
    - sector/month: 情報通信 / 5月 / bucket=top30
    - F119 expected: h20 +11.42% / win 88.0% / h5 +4.00%
    - boost: interpretation_sector_17_month:use_signal_normal × 情報通信 × 05 ...
    - avoid: none
    - caution: interpretation_sector_17_month:use_signal_normal × 情報通信 × 05 ...
    - sizing: 通常 (R-05-02 ルール準拠)
    - comment:
        F119上は強い (h20 +11.42%/win 88%)。ただし 20d 寄りの示唆。短期売買判断には使わない。
    - ★ 候補 (= レビュー対象 / 参考情報)。手動発注前提、自動発注禁止。

    ...

    安全:
    - 自動発注なし
    - 楽天操作なし
    - broker 接続なし
    - LINE 送信なし (preview text のみ)
    - Computer Use / Playwright 未使用
    - DB write なし
    - F119 の優位は 20d 寄り (daytrade 即時判断には使わない)
    - Fujiwara manual review required

注意点:
- 注文価格 / 数量 / 執行指示は一切作らない
- 「買え」「発注せよ」は使わず「候補」「レビュー対象」「参考情報」表現
- F119 が h20 寄りであることを明記
- 1 候補ごとに「手動発注前提、自動発注禁止」reminder を付与

### 4. summary JSON (--summary-json)

    {
      "candidate_count": 6,
      "with_advisory_count": 5,
      "missing_advisory_count": 1,
      "manual_review_required_count": 6,
      "auto_order_allowed_true_count": 0,         ← > 0 で runner fail
      "advisory_label_counts": {
        "boost_with_caution": 2,
        "avoid": 2,
        "missing_advisory": 1,
        "neutral": 1
      },
      "boost_flag_counts": {...},
      "avoid_flag_counts": {...},
      "caution_flag_counts": {...},
      "top_boost_candidates": [...],
      "top_avoid_candidates": [...],
      "top_caution_candidates": [...],
      "safety_notes": [
        "no auto order",
        "no broker connection",
        "no rakuten operation",
        "no Computer Use / Playwright",
        "no LINE send (preview only)",
        "no DB write",
        "F119 edge is h20-centric (not for daytrade immediate decision)",
        "manual review required"
      ]
    }

---

## JSON / CSV / text 全 31 field 一覧 (ROW_FIELDS 順)

    1.  code
    2.  name
    3.  side
    4.  score
    5.  rank
    6.  reason
    7.  research_advisory_version
    8.  research_source_version
    9.  research_rule_version
    10. research_base_date
    11. research_final_score
    12. research_post_cap_rank
    13. research_rank_label
    14. research_interpretation
    15. research_interpretation_detail
    16. market_regime
    17. sector_flow
    18. sector_17
    19. month_of_year
    20. top_bucket
    21. f119_boost_flags
    22. f119_avoid_flags
    23. f119_caution_flags
    24. f119_expected_h20_return
    25. f119_expected_h20_win_rate
    26. f119_expected_h5_return
    27. position_sizing_note
    28. research_advisory_label   (★ F111-R2 で算出)
    29. advisory_comment
    30. manual_review_required    (= 常に true 強制)
    31. auto_order_allowed        (= 常に false 強制)

candidate に存在しないフィールドは:

- 数値: None (CSV では空文字)
- 文字列: "" / "missing" (advisory_version 専用)
- list: [] (CSV では空文字)
- ★ manual_review_required / auto_order_allowed は None ではなく
  常に true / false を保証

---

## sample smoke 条件

### 入力 (SAMPLE_CANDIDATES、6 ケース)

| code | 想定ラベル | 内容 |
|---|---|---|
| 1234 | boost_with_caution | normal × 情報通信 × 5月 (h20 +11.42%/win 88%) |
| 8801 | avoid | 不動産 × 3月 (h20 -5.36%/win 33.3%) |
| 9984 | avoid | cautious × 10月 (h20 -3.36%/win 23.3%) |
| 7203 | boost_with_caution | 8月 normal h20 強 / h5 弱 |
| 0000 | missing_advisory | advisory なし → 安全側に倒す |
| 9999 | neutral | caller が auto_order=True を渡しても矯正 |

### 実行コマンド

    .venv/bin/python -m scripts.jobs.run_f111_research_advisory_preview \\
      --sample \\
      --output-json /tmp/f111_research_advisory_preview.json \\
      --output-csv /tmp/f111_research_advisory_preview.csv \\
      --output-text /tmp/f111_research_advisory_preview.txt \\
      --summary-json /tmp/f111_research_advisory_preview_summary.json \\
      --dry-run

### 出力 artifacts

| ファイル | サイズ |
|---|---|
| /tmp/f111_research_advisory_preview.json | 7,757 B |
| /tmp/f111_research_advisory_preview.csv | 3,151 B |
| /tmp/f111_research_advisory_preview.txt | 4,032 B |
| /tmp/f111_research_advisory_preview_summary.json | 3,036 B |

### summary 結果 (= 安全強制が機能した証拠)

    candidate_count:                  6
    with_advisory_count:              5
    missing_advisory_count:           1
    manual_review_required_count:     6   (= candidate_count)
    auto_order_allowed_true_count:    0   (★ caller の True 入力も矯正)
    advisory_label_counts: {
      boost_with_caution: 2,
      avoid:              2,
      missing_advisory:   1,
      neutral:            1
    }

★ 9999 で advisory に
`{"manual_review_required": False, "auto_order_allowed": True}` を
意図的に投入したが、advisory_to_row が両方を強制矯正したため
auto_order_allowed_true_count=0 / manual_review_required_count=6 に
収束。

---

## DB 安全確認 (smoke 前後)

    target              before              after               status
    fire.db             May  7 16:12:38     May  7 16:12:38     unchanged
    fire.develop.db     May  7 18:14:26     May  7 18:14:26     unchanged
    fire.staging.db     May  9 22:40:35     May  9 22:40:35     unchanged

write 対象 tables: なし (DB に一切接続しない設計)

★ runner module source に `sqlite3.connect` / `.connect(` / `DB_PATH`
の文字列が含まれない (= test_runner_does_not_open_db_connection で検証)。

---

## auto_order_allowed_true_count == 0 確認

✅ smoke summary.json に `"auto_order_allowed_true_count": 0`
✅ caller の True 入力 (9999) でも advisory_to_row が False に矯正
✅ ResearchAdvisory.__post_init__ (F111-R1) との二重防御
✅ build_summary が > 0 を検出した場合は AdvisoryPreviewSafetyViolation
   を raise → main() が exit 3 で停止

## manual_review_required_count == candidate_count 確認

✅ smoke summary.json に `"manual_review_required_count": 6`
   (= candidate_count: 6)
✅ caller の False 入力でも advisory_to_row が True に矯正
✅ build_summary が `manual_review_required_count != candidate_count`
   を検出した場合も violation で停止

---

## LINE 送信なし確認

✅ runner module source に `notifications.line_bot` / `linebot` /
   `LineBotClient` / `send_text` / `LINE_CHANNEL_TOKEN` の文字列が
   含まれない (= test_main_does_not_import_line_bot で検証)
✅ CLI に `--line` / `--line-send` / `--notify` option なし
   (= test_no_line_or_broker_options で help 文字列を検証)
✅ preview text にも実際の LINE push API 呼び出しコード絶無

## order / broker / rakuten / Computer Use 未接続確認

✅ runner module source に `TradeOrder` / `TradeDecisionAgent` /
   `place_order` / `send_order` / `submit_order` の文字列なし
   (= test_runner_does_not_send_order)
✅ runner module source に `playwright` / `selenium` / `subprocess.run`
   なし (= test_main_does_not_import_line_bot)
✅ CLI に `--broker` / `--rakuten` / `--order` / `--computer-use`
   option なし
✅ SAMPLE_CANDIDATES に broker / rakuten / ispeed / line_token /
   db_path / channel_token を含めない (= test_no_db_or_broker_keys_in_sample)

---

## DB write なし確認

✅ runner module は sqlite3 / psycopg2 / SQLAlchemy 不使用
✅ formatter module も DB ライブラリ不使用 (純関数のみ)
✅ smoke 前後で 3 DB の last_modified が完全 unchanged
✅ `--write` option を runner に作らない設計

---

## tests 結果

### 新規 68 PASS

    tests/agents/test_research_advisory_preview.py (42)
    ├── TestComputeAdvisoryLabel (8)
    ├── TestAdvisoryToRowSafety (10)
    ├── TestRowToCsvRow (4)
    ├── TestBuildPreviewText (9)
    ├── TestBuildSummary (6)
    ├── TestSafetyViolationDetection (2)
    ├── TestRenderCandidateBlock (2)
    └── TestAdvisoryPreviewSafetyViolation (1)

    tests/scripts/jobs/test_run_f111_research_advisory_preview.py (26)
    ├── TestParseArgs (5)
    ├── TestSampleCandidates (5)
    ├── TestRunPreviewCore (8)
    ├── TestLoadCandidates (4)
    └── TestMainCli (5)

### regression

- F111 / F115 / F140 / F133 / F132 / F119 / F286 全テスト 0 件回帰
- フル pytest **2,779 PASS** (= 2,711 baseline + 68 新規)

---

## Codex pre-commit 結果

| commit | Codex 判定 |
|---|---|
| c382a38 (feat formatter) | ✅ OK |
| 16a3a07 (chore runner)   | ✅ OK |
| 424c5fb (test)           | ✅ OK |

✅ 全 3 commit で Codex review 通過
✅ CRITICAL 指摘 0 件 (= 修正対応 0 件)
✅ pre-commit hook (`scripts/hooks/pre-commit`) 正常通過

---

## --no-verify 未使用確認

✅ 全 3 commit で `--no-verify` flag 不使用
✅ pre-commit を bypass する一切の手段なし
✅ FIRE_ALLOW_WORKFLOW_CHANGE も不要 (workflow 変更なし)

---

## unrelated modified 未接触確認

git status の `Changes not staged for commit:` 欄に下記 2 ファイルが
F111-R2 commit 前後で残存していることを確認:

- `scripts/seed_pattern_layer1.py` (本タスク無関係、touched せず)
- `simulation/research_lane/historical_indicators.py` (同上)

3 commit すべて `git add <specific files>` で個別 stage、`git add .`
や `git add -A` は不使用。

---

## TODO Excel 未更新確認

✅ Google Sheets TODO 管理表は本タスクで更新していない
✅ Fujiwara 側で別途更新する想定 (= AI 側の自動更新を作っていない)

---

## 制約遵守チェックリスト

- ✅ 自動発注禁止 (advisory_to_row が auto_order_allowed=False 強制)
- ✅ 楽天証券操作の自動化なし
- ✅ Computer Use 未使用
- ✅ Playwright / Cron 強制クローズ未接続
- ✅ LINE 送信なし (preview text 生成まで、push API 呼ばず)
- ✅ 発注は Fujiwara 手動実行前提 (manual_review_required=True 強制)
- ✅ production / develop / staging DB 不用意な変更なし (= 接続せず)
- ✅ 原則 DB write なし (= write は 1 件もなし)
- ✅ smoke は in-memory only (DB / staging も使わず)
- ✅ Codex pre-commit 必須 (× 3 全件通過)
- ✅ --no-verify 禁止 (× 3 全件で flag 不使用)
- ✅ 個別 commit 厳守 (3 個別 commit)
- ✅ scripts/seed_pattern_layer1.py の既存 modified 未接触
- ✅ simulation/research_lane/historical_indicators.py 未接触
- ✅ unrelated modified を stage / commit しない
- ✅ TODO Excel 未更新

---

## 次タスク提案

### 第一候補: F062-R1 LINE Advisory Notification Template (preview → 整形)

F111-R2 で作った preview text / row dict を LINE 5 部屋
(ENTRY 部屋メイン) のテンプレート用に整形する **template module**
を追加する。**ただし本番送信は接続しない**:

- `notifications/templates/research_advisory.py` 新規作成
- F062 既存 R-06-03 11 項目 TradeOrder 整形は touched せず、
  別系統の `research_advisory` テンプレートとして並列追加
- format(rows) → str (LINE 用 simple multi-line)
- `notifications/router.py` への登録は **F062-R2** で別タスク化
- preview module と同じく safety lines を強制

### 第二候補: F111-R3 Real Wiring Smoke (F119 evaluate → AdvisoryBuilder → preview)

F111-R1 build_advisory + F119 evaluate_signals_with_interpretation の
出力 → DaytradeSelectionAgent.attach_advisories → preview runner の
**実データで連結する read-only orchestrator**:

- 入力: F286-R2-F4 baseline signals + R2-G3 recommended_v2 +
  F119 evaluate 出力 + 仮想 F111 candidates
- 出力: preview JSON / CSV / text + smoke summary
- DB read のみ (= staging 経由)、write 一切なし

### 第三候補: R2-G4 5d 用 rule 設計 (F119 で確認した h5 警告対応)

F119 で確認された h5 短期 vs h20 中期の乖離 (cautious h5 -3.06% /
h20 +0.59%) を踏まえ、5 日保有用 interpretation rule を別 candidate
として設計し、R2-H 同様の orthogonal cuts を回す。

優先度: 1 > 2 > 3 (LINE 通知接続準備の流れに合わせる、ただし F062-R1
も safety 設計を慎重にやる必要あり)

---

## 関連参照

- 02_todo/F111_R1_research_advisory_signal_integration.md (前段)
- 02_todo/F119_interpretation_evaluation.md
- 02_todo/F286_R2_H_orthogonal_cuts.md
- 02_todo/F111_Daytrade_Selection_Agent.md (本体)
- log.md milestone (本タスク完了時に追記)
