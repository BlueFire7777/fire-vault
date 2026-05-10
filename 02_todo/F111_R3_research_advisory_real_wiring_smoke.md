# F111-R3 Research Advisory Real Wiring Smoke

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: F111-R1 (07453f7 / f4a63f1 / 1a8b03b) + F111-R2 (c382a38
> / 16a3a07 / 424c5fb)
> **Mode**: 完全 read-only (URI mode=ro + PRAGMA query_only=ON 二重防御)
> **Result**: ★★★ staging 実 signal 100 件 → ResearchAdvisory build →
> preview JSON / CSV / text / summary 生成、auto_order_allowed_true_count=
> 0 / manual_review_required_count=100 / DB last_modified unchanged
> 全項目 PASS。Stage 3 Live Advisory dry run 経路完成。★★★

---

## タスク名

F111-R3 Research Advisory Real Wiring Smoke

---

## 背景

F111-R1 で ResearchAdvisory を DaytradeCandidate に接続済み:
- agents/research_advisory.py / build_advisory + ResearchAdvisory
- agents/daytrade_selection.py / DaytradeCandidate.advisory + attach_advisories
- ResearchAdvisory.__post_init__ で manual_review_required=True /
  auto_order_allowed=False 強制

F111-R2 で ResearchAdvisory 付き candidate を JSON / CSV / text preview /
summary に出力できるようになった:
- agents/research_advisory_preview.py / advisory_to_row / build_summary
- scripts/jobs/run_f111_research_advisory_preview.py / SAMPLE_CANDIDATES

F111-R3 では sample ではなく **staging 実データ** を read-only で読み、
Research Lane / F119 / F111-R1 / F111-R2 の流れを **dry run で連結** する。

---

## F111-R1 / F111-R2 からの接続内容

| F111-R1 | F111-R2 | F111-R3 |
|---|---|---|
| build_advisory (純関数) | advisory_to_row | run_real_wiring (= build_advisory を staging signal 行に対して大量呼び出し) |
| ResearchAdvisory dataclass | row dict | candidate dict (advisory 埋め込み) |
| __post_init__ 安全強制 | row 強制 | 三重防御の最終層 (build_summary で再検証) |
| DaytradeCandidate.advisory | candidates_to_rows | run_real_wiring → preview formatter に直接渡す |
| F119 evaluate (insights/cut_summaries) | optional file 入力 | optional `--insights-json` / `--cut-summaries-json` |

---

## 実装ファイル一覧

| 種別 | ファイル | 行数 |
|---|---|---|
| feat | `agents/research_advisory_real_wiring.py` | 476 |
| chore (runner) | `scripts/jobs/run_f111_research_advisory_real_wiring_smoke.py` | 450 |
| test | `tests/agents/test_research_advisory_real_wiring.py` | (32 PASS) |
| test | `tests/scripts/jobs/test_run_f111_research_advisory_real_wiring_smoke.py` | (17 PASS) |

### モジュール構成

    agents/
    └── research_advisory_real_wiring.py   (476 行、新規)
        ├── ALLOWED_DB_LABELS / RULE_VERSION_TO_CANDIDATE
        ├── F111R3RunRefused (例外)
        ├── resolve_db_path (F119 と同形式 / DB_PATH env 対応)
        ├── open_readonly_connection (URI mode=ro + PRAGMA
        │   query_only=ON の二重防御)
        ├── resolve_candidate_name (rule_version → candidate)
        ├── list_available_base_dates / resolve_latest_base_date
        ├── load_signal_rows (top_n + post_cap_rank int filter)
        ├── WiringContext (frozen dataclass)
        ├── build_wiring_context (regime / sector_flow features)
        ├── build_joined_rows (F119 _build_joined_rows と同形)
        ├── enrich_with_interpretation (apply_candidate)
        ├── _month_of_year (YYYY-MM-DD → 1..12)
        ├── build_candidate_dict (★ build_advisory 中継)
        ├── build_candidates_with_advisories
        ├── WiringResult (dataclass)
        └── run_real_wiring (主 orchestrator)

    scripts/jobs/
    └── run_f111_research_advisory_real_wiring_smoke.py   (450 行、新規)
        ├── _maybe_load_json (insights/cut_summaries optional)
        ├── _stat_last_modified
        ├── write_completion_report (= /tmp 完了報告 永続化)
        ├── build_arg_parser (--write / --line / --broker /
        │   --rakuten / --order / --auto-order / --computer-use
        │   / --playwright option を作らない)
        ├── parse_args (--source-version 必須)
        └── main (refused → exit 2 / 0 件 → exit 4 /
            safety violation → exit 3 / DB modified → exit 3)

    tests/
    ├── tests/agents/test_research_advisory_real_wiring.py (32 PASS)
    │   - resolve_db_path 4 / open_readonly_connection 4 (INSERT
    │     refuse / CREATE TABLE refuse 含む) / resolve_candidate_name 3
    │   - list_available_base_dates / resolve_latest 4 /
    │     load_signal_rows 4 (top_n filter)
    │   - build_joined_rows 3 / enrich_with_interpretation 2 /
    │     build_candidate_dict 3 / safety invariants 1
    │   - _month_of_year 2 / module source safety 2
    └── tests/scripts/jobs/
        test_run_f111_research_advisory_real_wiring_smoke.py (17 PASS)
        - arg parser 4 / _maybe_load_json 4 / _stat_last_modified 2
        - write_completion_report 1 / main 3 (db-missing / full /
          zero-signals)
        - runner source safety 3 (LINE/order strings, write SQL exec,
          help に forbidden option 不在)

---

## commit hash 一覧

| commit | type | 内容 |
|---|---|---|
| `44b9764` | feat | F111-R3: add research advisory real wiring module |
| `4fb3176` | chore | F111-R3: add real wiring smoke runner |
| `7b47114` | test | F111-R3: add real wiring smoke tests |
| (本 commit) | docs | F111-R3: vault |
| (次 commit) | docs | F111-R3: log milestone |

---

## real wiring smoke の流れ

    [staging DB (read-only)]
          ↓
    open_readonly_connection
      = sqlite3.connect("file:...?mode=ro", uri=True)
      + PRAGMA query_only=ON  ★ 二重防御
          ↓
    resolve_latest_base_date
      = SELECT DISTINCT base_date FROM research_watchlist_signals
          ↓
    load_signal_rows (top_n=100)
      = signal_persistence.load_signals_by_base_date
        + post_cap_rank int filter
          ↓
    build_wiring_context
      = build_market_regime_features +
        reapply_volatility_labels_leak_safe +
        build_sector_flow_features
          ↓
    build_joined_rows (F119 と同形)
      → market_regime_label / market_volatility_label /
        sector_flow_label / signal_sector_alignment 付与
          ↓
    enrich_with_interpretation
      = apply_candidate (R2-G3 candidate)
      → interpretation / interpretation_detail / rule_version 付与
          ↓
    build_candidate_dict (per row, ★ 中核)
      = ResearchAdvisory build_advisory(...)
        - month_of_year を base_date から派生
        - top_bucket を post_cap_rank から派生 (derive_top_bucket)
        - F119 insights / cut_summaries が optional 入力されれば
          advisory.f119_*_flags / expected_h20 が埋まる
          ↓
    [F111-R2 preview formatter]
      candidates_to_rows → row_to_csv_row / build_preview_text /
      build_summary
          ↓
    [output artifacts]
      preview.json / preview.csv / preview.txt / summary.json /
      completion_report.txt

---

## DB read-only 方式 (= 二重防御の構造)

    1. URI レベル: file:{db_path}?mode=ro
       (sqlite3 自体が write を refuse)
    2. PRAGMA レベル: PRAGMA query_only=ON
       (write SQL が SQLITE_READONLY エラーで refuse)
    3. source レベル: runner / module の source 文字列に
       INSERT/UPDATE/DELETE/DROP/CREATE TABLE 等の write SQL
       execute pattern が一切無い (= test で検証)
    4. CLI レベル: --write / --auto-order / --line-send 等の
       option を作らない (= argparse help で検証)

---

## smoke 条件

    DB:               data/fire.staging.db (label=staging)
    source_version:   r2f4_baseline_v1 (count 2,417, max base_date 2026-03-01)
    rule_version:     r2g3_recommended_v2
    candidate_name:   cautious_mixed_to_suppress
    base_date:        2026-03-01 (= source_version の最新 base_date)
    top_n:            100
    mode:             dry-run / read-only
    F119 insights:    なし (advisory.f119_*_flags は空)
    F119 cut_summaries: なし (advisory.f119_expected_* は None)

### 実行コマンド

    .venv/bin/python -m scripts.jobs.run_f111_research_advisory_real_wiring_smoke \
      --db staging \
      --db-path data/fire.staging.db \
      --source-version r2f4_baseline_v1 \
      --rule-version r2g3_recommended_v2 \
      --top-n 100 \
      --output-json /tmp/f111_r3_real_wiring_preview.json \
      --output-csv /tmp/f111_r3_real_wiring_preview.csv \
      --output-text /tmp/f111_r3_real_wiring_preview.txt \
      --summary-json /tmp/f111_r3_real_wiring_summary.json \
      --completion-report /tmp/f111_r3_completion_report.txt \
      --dry-run

---

## output artifacts 一覧

| ファイル | サイズ |
|---|---|
| /tmp/f111_r3_real_wiring_preview.json | 141 KB |
| /tmp/f111_r3_real_wiring_preview.csv | 58 KB |
| /tmp/f111_r3_real_wiring_preview.txt | 72 KB |
| /tmp/f111_r3_real_wiring_summary.json | 685 B |
| /tmp/f111_r3_completion_report.txt | 2.3 KB |

---

## smoke 結果

    candidate_count:                  100
    with_advisory_count:              100
    missing_advisory_count:           0
    manual_review_required_count:     100   (= candidate_count) ★
    auto_order_allowed_true_count:    0     ★
    advisory_label_counts:
      neutral: 100   (= F119 insights なし入力なので boost/avoid/
                       caution 判定材料がない、neutral に倒れる仕様)
    boost_flag_counts:    {}
    avoid_flag_counts:    {}
    caution_flag_counts:  {}
    safety_notes:
      - no auto order
      - no broker connection
      - no rakuten operation
      - no Computer Use / Playwright
      - no LINE send (preview only)
      - no DB write
      - F119 edge is h20-centric (not for daytrade immediate decision)
      - manual review required

---

## 安全要件遵守

### manual_review_required_count == candidate_count 確認

✅ summary.json: manual_review_required_count = 100 = candidate_count
✅ run_preview / build_summary 内の検査でも equal を確認 (mismatch
   なら exit 3)
✅ build_advisory → ResearchAdvisory.__post_init__ →
   advisory_to_row の三重防御

### auto_order_allowed_true_count == 0 確認

✅ summary.json: auto_order_allowed_true_count = 0
✅ build_advisory → ResearchAdvisory.__post_init__ →
   advisory_to_row の三重防御
✅ runner main 内の検査で >0 → exit 3

### LINE 送信なし確認

✅ runner / module source に `notifications.line_bot` / `linebot` /
   `LineBotClient` / `send_text` / `push_message` の文字列なし
✅ argparse help に `--line` / `--line-send` option なし
✅ source 文字列で test_no_line_or_order_strings 検証

### order / broker / rakuten / Computer Use 未接続確認

✅ runner / module source に `TradeOrder` / `TradeDecisionAgent` /
   `place_order` / `send_order` / `submit_order` の文字列なし
✅ `playwright` / `selenium` / `subprocess.run` の文字列なし
✅ argparse help に `--broker` / `--rakuten` / `--order` /
   `--computer-use` / `--playwright` option なし
✅ test_no_line_or_order_modules / test_no_write_option_in_argparser
   で source レベル / argparse レベル両方を検証

### DB write なし確認

✅ open_readonly_connection が URI mode=ro + PRAGMA query_only=ON
✅ runner module source に `.execute("INSERT` / `.execute("UPDATE` /
   `.execute("DELETE` / `.execute("DROP` / `.execute("CREATE TABLE` /
   `.executescript(` / `.executemany("INSERT` 文字列が含まれない
   (= test_no_write_sql_execution_in_runner / module で検証)
✅ INSERT 投入を試みると sqlite3.OperationalError で refuse される
   ことを test_insert_refused_by_query_only で検証
✅ CREATE TABLE も同様に refuse されることを test_create_table_refused
   で検証

### staging last_modified unchanged 確認

    target              before              after               status
    fire.staging.db     2026-05-09T22:40:35.385124  unchanged   ✅
    fire.develop.db     May  7 18:14:26              unchanged   ✅
    fire.db             May  7 16:12:38              unchanged   ✅

★ runner main 内で `DB last_modified が smoke 前後で変化したら exit 3`
  の最終 防御層も入れている (= 万一の OS 側書き込み防止)。

### production / develop 無触確認

✅ smoke は --db-path data/fire.staging.db のみで実行
✅ DB_PATH env を develop / production に向ける code 経路なし
✅ develop.db / fire.db (production 想定) の last_modified が前後で
   完全 unchanged

---

## tests 結果

### 新規 49 PASS

    tests/agents/test_research_advisory_real_wiring.py (32)
    ├── TestResolveDbPath (4)
    ├── TestOpenReadonlyConnection (4)
    │   - test_select_works
    │   - test_insert_refused_by_query_only ★
    │   - test_create_table_refused ★
    │   - test_missing_file_refused
    ├── TestResolveCandidateName (3)
    ├── TestBaseDates (4)
    ├── TestLoadSignalRows (4)
    ├── TestBuildJoinedRows (3)
    ├── TestEnrichWithInterpretation (2)
    ├── TestBuildCandidateDict (3)
    ├── TestSafetyInvariants (1)
    ├── TestMonthOfYear (2)
    └── TestModuleSourceSafety (2)

    tests/scripts/jobs/
    test_run_f111_research_advisory_real_wiring_smoke.py (17)
    ├── TestArgParser (4)
    ├── TestMaybeLoadJson (4)
    ├── TestStatLastModified (2)
    ├── TestWriteCompletionReport (1)
    ├── TestMain (3)
    └── TestRunnerSourceSafety (3)

### regression

- F111 / F115 / F140 / F133 / F132 / F119 / F286 全テスト 0 件回帰
- フル pytest **2,828 PASS** (= 2,779 baseline + 49 新規)

---

## Codex pre-commit 結果

| commit | Codex 判定 |
|---|---|
| 44b9764 (feat module) | ✅ OK |
| 4fb3176 (chore runner) | ✅ OK |
| 7b47114 (test)         | ✅ OK |

✅ 全 3 commit で Codex review 通過
✅ CRITICAL 指摘 0 件 (= 修正対応 0 件)
✅ pre-commit hook 正常通過

---

## --no-verify 未使用確認

✅ 全 3 commit で `--no-verify` flag 不使用
✅ Codex usage limit / rate limit / auth error なし

---

## unrelated modified 未接触確認

git status の `Changes not staged for commit:` 欄に下記 2 ファイルが
F111-R3 commit 前後で残存:

- `scripts/seed_pattern_layer1.py` (本タスク無関係、touched せず)
- `simulation/research_lane/historical_indicators.py` (同上)

3 commit すべて `git add <specific files>` で個別 stage。

---

## TODO Excel 未更新確認

✅ Google Sheets TODO 管理表は本タスクで更新していない

---

## 制約遵守チェックリスト

- ✅ 自動発注禁止 (advisory_to_row + ResearchAdvisory.__post_init__
  + run_preview safety 検査の三重防御)
- ✅ 楽天証券操作の自動化なし
- ✅ Computer Use 未使用
- ✅ Playwright / Selenium / subprocess 未使用
- ✅ LINE 本番送信なし
- ✅ 発注は Fujiwara 手動実行前提
- ✅ production / develop / staging DB に書き込みなし
- ✅ DB write 0 件 (URI mode=ro + PRAGMA query_only=ON 二重防御)
- ✅ staging も read-only でのみアクセス
- ✅ Codex pre-commit (× 3 全件通過)
- ✅ --no-verify 禁止 (× 3 全件で flag 不使用)
- ✅ 個別 commit 厳守 (3 個別 commit)
- ✅ scripts/seed_pattern_layer1.py の既存 modified 未接触
- ✅ simulation/research_lane/historical_indicators.py 未接触
- ✅ unrelated modified を stage / commit しない
- ✅ TODO Excel 未更新

---

## 次タスク提案

### 第一候補: F062-R1 LINE Advisory Notification Template

F111-R3 まで preview / wiring が固まったので、次は LINE 5 部屋
(ENTRY 部屋) 向け template module を追加する。**ただし本番送信は接続
しない**:

- `notifications/templates/research_advisory.py` 新規作成
- F111-R2 preview row → LINE 簡易整形 (multi-line text)
- 本番送信 (LineBotClient.send_text) は F062-R2 で別タスク化
- safety lines / 「自動発注禁止」reminder を必ず付与
- dry_run mode 専用、production token は使わない

### 第二候補: F111-R4 Multi base_date Smoke + F119 内蔵

F111-R3 では F119 insights / cut_summaries は optional 入力。次は
F119 evaluate を runner 内部で **実行** し、advisory に boost /
avoid / caution / expected_h20 が実際に埋まる smoke を回す。

- `--evaluate-f119` option 追加 (default false)
- staging で 1 base_date 分の F119 evaluate を実行 (read-only)
- advisory.f119_*_flags が boost/avoid/caution で分類される
- 複数 base_date 対応 (= F119 と同様の `--base-dates` カンマ区切り)

### 第三候補: R2-G4 5d 用 rule 設計

F119 で確認された h5 短期 vs h20 中期の乖離 (cautious h5 -3.06% /
h20 +0.59%) を踏まえ、5 日保有用 interpretation rule を別 candidate
として設計し、R2-H 同様の orthogonal cuts を回す。

優先度: 1 > 2 > 3 (LINE 通知接続準備の流れに合わせる、F062-R1 で
先に safety 文言の本番転用を確認)

---

## 関連参照

- 02_todo/F111_R1_research_advisory_signal_integration.md (前段)
- 02_todo/F111_R2_research_advisory_preview.md (前段)
- 02_todo/F119_interpretation_evaluation.md
- 02_todo/F286_R2_H_orthogonal_cuts.md
- 02_todo/F286_R2_G3_rule_finalization.md
- log.md milestone (本タスク完了時に追記)
