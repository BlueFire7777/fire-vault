# F111-R4 Multi base_date Smoke + F119 Built-in Insights

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: F111-R3 real_wiring (44b9764 / 4fb3176 / 7b47114) +
> F119 evaluate_signals_with_interpretation
> **Mode**: 完全 read-only (URI mode=ro + PRAGMA query_only=ON +
> output Path 衝突 guard の三重防御)
> **Result**: ★★★ staging 4 base_dates × 100 = 400 候補に F119
> insights / cut_summaries が実際に接続され、boost 250 / avoid 116
> / caution 284 / expected_h20 全 400 / DB unchanged。
> Stage 3 Live Advisory dry run の最終形。★★★

---

## タスク名

F111-R4 Multi base_date Smoke + F119 Built-in Insights

---

## 背景

F111-R3 で staging 実 signal から ResearchAdvisory を read-only で
build できるようになったが、F119 insights / cut_summaries 入力なしで
実行したため、advisory_label_counts は **neutral 100** になっていた
(boost / avoid / caution の判定材料がなかった)。

F111-R4 では F119 evaluation を read-only で接続し、real staging
candidate に以下が実際に付くことを検証する:

- boost flags (= F119 strong_candidates 該当)
- avoid flags (= F119 avoid_candidates 該当)
- caution flags (= F119 caution_candidates 該当)
- f119_expected_h20_return / win_rate
- f119_expected_h5_return

---

## F111-R3 からの差分

| 観点 | F111-R3 | F111-R4 |
|---|---|---|
| base_date | 単一 (latest 既定) | 複数 (--base-dates カンマ区切り) |
| F119 接続 | 無し (advisory neutral) | inline / artifact JSON / 3 CSV から選択 |
| evaluate-f119 | ❌ | ✅ runner 内で read-only 実行 |
| summary metric | candidate_count + label_counts | + per_base_date_counts / non_neutral / boost/avoid/caution_count / expected_h20/h5_present |
| preview text | 単一 list | base_date ごとに分割 |
| 出力 Path safety | runner 経由のみ | + DB Path 衝突 guard (Codex CRITICAL 対応) |
| exit code | 0/2/3/4 | + 5 (= F119 接続済みなのに non_neutral=0 異常検出) |

---

## F119 artifacts / evaluate-f119 接続仕様

### 接続経路 (resolve_f119_artifacts の優先順)

1. **inline (`--evaluate-f119`)** — staging から signal/regime/
   sector_flow を読み、F119 既存ロジックを read-only で再 orchestrate
   して insights / cut_summaries を生成
2. **JSON (`--f119-summary-json` / `--f119-insights-json`)** —
   既存 F119 runner 出力を読み込み
3. **CSV (`--f119-strong-csv` / `--f119-avoid-csv` /
   `--f119-caution-csv`)** — 3 CSV を合成して insights のみ提供
4. **none** — 何も接続しない (= advisory neutral になる、F111-R3 と
   同等の挙動)

### cut_summaries 形式

evaluate_signals_with_interpretation 出力と互換:

    {
      "interpretation": [{"cut_type": "interpretation",
                          "group_key": "use_signal_normal",
                          "count": 1500,
                          "mean_return": {20: 0.029, 5: 0.005},
                          "win_rate": {20: 0.591, 5: 0.5}, ...}, ...],
      "interpretation_sector_17_month": [...],
      "month_of_year": [...],
      "top_bucket": [...],
      ...
    }

### insights 形式

extract_f119_insights 出力と互換:

    {
      "strong_candidates": [{"cut_type": ..., "group_key": ...,
                              "count": ..., "mean_return_20d": ...,
                              "win_rate_20d": ...}, ...],
      "avoid_candidates": [...],
      "caution_candidates": [...]
    }

### fallback 順 (F111-R1 build_advisory が cut_summaries から
expected_h20 を探す優先順)

1. interpretation_sector_17_month (3-key 最 specific)
2. interpretation_month
3. interpretation
4. overall

### min_count gate

F119Thresholds.min_count = 20 (default)
expected metrics は count >= min_count な cut から取り出す
(overall は count 0 でも採用)。

---

## 実装ファイル一覧

| 種別 | ファイル | 行数 |
|---|---|---|
| feat | `agents/research_advisory_f119_inline.py` | 373 |
| chore (runner) | `scripts/jobs/run_f111_research_advisory_real_wiring_r4_smoke.py` | 838 |
| test | `tests/agents/test_research_advisory_f119_inline.py` | 14 PASS |
| test | `tests/scripts/jobs/test_run_f111_research_advisory_real_wiring_r4_smoke.py` | 32 PASS |

### モジュール構成

    agents/
    └── research_advisory_f119_inline.py (373 行、新規)
        ├── F111R4ArtifactRefused (例外)
        ├── load_f119_insights_json (= strong/avoid/caution 必須キー検証)
        ├── load_f119_summary_json (= cut_summaries dict 必須)
        ├── extract_cut_summaries_from_summary
        ├── load_f119_insight_csv (= type coercion)
        ├── assemble_insights_from_csvs (3 CSV → insights dict)
        ├── InlineEvaluateResult (frozen dataclass)
        ├── evaluate_f119_inline (★ read-only conn から F119 実行)
        ├── F119Artifacts (frozen dataclass)
        └── resolve_f119_artifacts (優先順統合)

    scripts/jobs/
    └── run_f111_research_advisory_real_wiring_r4_smoke.py (838 行、新規)
        ├── F111R4RunRefused
        ├── parse_base_dates_arg / parse_horizons_arg
        ├── _stat_last_modified
        ├── _assert_output_paths_safe (★ Codex CRITICAL 対応 guard)
        ├── run_multi_base_date_wiring (per_base_date 集約)
        ├── build_r4_summary (R4 拡張 metric)
        ├── build_r4_preview_text (base_date 区切り)
        ├── write_completion_report
        ├── _save_inline_artifacts (--f119-output-dir 永続化)
        ├── build_arg_parser
        ├── parse_args (--base-date / --base-dates 排他)
        └── main (refused → 2 / safety → 3 / no candidates → 4 /
            non_neutral=0 anomaly → 5)

    tests/  (46 PASS = 14 + 32)
    ├── tests/agents/test_research_advisory_f119_inline.py (14 PASS)
    │   - load_f119_insights_json 6
    │   - load_f119_summary_json + extract_cut_summaries 5
    │   - load_f119_insight_csv + assemble 5
    │   - resolve_f119_artifacts 4
    │   - evaluate_inline validation 1
    │   - module source safety 2
    └── tests/scripts/jobs/
        test_run_f111_research_advisory_real_wiring_r4_smoke.py (32 PASS)
        - parsers 7
        - arg parser safety 1
        - build_r4_summary 2
        - build_r4_preview_text 1
        - main full pipeline 4 (inline / non_neutral=0 anomaly /
          db-missing / zero candidates)
        - _assert_output_paths_safe 5 (no_collision / exact /
          relative / wal-shm / main refuses DB as output)
        - runner source safety 2

---

## commit hash 一覧

| commit | type | 内容 |
|---|---|---|
| `74e2276` | feat | F111-R4: add F119 insights real wiring support |
| `fe8d170` | chore | F111-R4: add multi-base-date real wiring smoke runner |
| `8dee93a` | test | F111-R4: add F119 insights real wiring tests |
| (本 commit) | docs | F111-R4: vault |
| (次 commit) | docs | F111-R4: log milestone |

---

## smoke 条件

    DB:               data/fire.staging.db (label=staging)
    source_version:   r2f4_baseline_v1
    rule_version:     r2g3_recommended_v2 (= cautious_mixed_to_suppress)
    base_dates:       2025-05-01 / 2025-06-01 / 2025-08-01 / 2026-03-01
                      (= F119 で 5月強 / 6月強 / 8月強 / 3月弱)
    top_n:            100
    horizons:         1, 5, 20
    min_count:        20
    F119 経路:        --evaluate-f119 (inline)
    --f119-output-dir: /tmp/f111_r4_f119
    mode:             dry-run / read-only

### 実行コマンド

    .venv/bin/python -m \
      scripts.jobs.run_f111_research_advisory_real_wiring_r4_smoke \
      --db staging \
      --db-path data/fire.staging.db \
      --source-version r2f4_baseline_v1 \
      --rule-version r2g3_recommended_v2 \
      --base-dates 2025-05-01,2025-06-01,2025-08-01,2026-03-01 \
      --top-n 100 \
      --evaluate-f119 \
      --f119-output-dir /tmp/f111_r4_f119 \
      --horizons 1,5,20 --min-count 20 \
      --output-json /tmp/f111_r4_real_wiring_preview.json \
      --output-csv /tmp/f111_r4_real_wiring_preview.csv \
      --output-text /tmp/f111_r4_real_wiring_preview.txt \
      --summary-json /tmp/f111_r4_real_wiring_summary.json \
      --completion-report /tmp/f111_r4_completion_report.txt \
      --dry-run

---

## output artifacts 一覧

| ファイル | サイズ |
|---|---|
| /tmp/f111_r4_real_wiring_preview.json | 1.0 MB |
| /tmp/f111_r4_real_wiring_preview.csv | 668 KB |
| /tmp/f111_r4_real_wiring_preview.txt | 702 KB |
| /tmp/f111_r4_real_wiring_summary.json | 18 KB |
| /tmp/f111_r4_completion_report.txt | 2.2 KB |
| /tmp/f111_r4_f119/f119_insights.json | 131 KB |
| /tmp/f111_r4_f119/f119_summary.json | 403 KB |

---

## smoke 結果 (= F119 接続が機能している証拠)

### 集約結果

    candidate_count:                  400
    with_advisory_count:              400
    missing_advisory_count:           0
    manual_review_required_count:     400  (= candidate_count) ★
    auto_order_allowed_true_count:    0    ★
    non_neutral_count:                400  (★ 全件が boost/avoid/
                                            caution に分類)
    boost_candidate_count:            250  (= boost + boost_with_*)
    avoid_candidate_count:            116  (= avoid + boost_with_avoid)
    caution_candidate_count:          284  (= caution + boost_with_caution)
    expected_h20_metric_present_count: 400  (★ 全件)
    expected_h5_metric_present_count:  400  (★ 全件)
    f119_artifact_status:             inline
    f119_summary_loaded:              True
    f119_insights_loaded:             True
    f119_cut_summaries_loaded:        True

### advisory_label_counts (4 base_dates 集約)

    boost_with_caution: 217
    avoid:               83
    caution:             67
    boost_with_avoid:    33

### per_base_date_counts (label 内訳)

    2025-05-01 (5月、F119 強):
      boost_with_caution: 99 / boost_with_avoid: 1
      → 100 件全てが boost 系 (5月の F119 強さを反映)

    2025-06-01 (6月、F119 強):
      boost_with_caution: 26 / caution: 67 /
      boost_with_avoid: 2 / avoid: 5
      → boost+caution 主体、混在

    2025-08-01 (8月、F119 強):
      boost_with_caution: 92 / boost_with_avoid: 8
      → 100 件全てが boost 系 (8月の F119 強さを反映)

    2026-03-01 (3月、F119 弱):
      avoid: 78 / boost_with_avoid: 22
      → 100 件全てが avoid 系 (3月の F119 弱さを反映)

★ F119 で確認された月別 strong/avoid 傾向が、real staging candidate
の advisory_label に **そのまま反映** されている。Stage 3 Live
Advisory のロジックとして適切に機能。

---

## 安全要件遵守

### 三重防御 (二重 + Codex CRITICAL 対応)

1. **DB URI レベル**: `file:{db}?mode=ro` で sqlite write を refuse
2. **PRAGMA レベル**: `PRAGMA query_only=ON` で write SQL を refuse
3. **出力 Path レベル** (★ Codex CRITICAL 対応):
   _assert_output_paths_safe で `--output-json` / `--output-csv` /
   `--output-text` / `--summary-json` / `--completion-report` /
   `--f119-output-dir` 配下の各 Path が `db_path` と完全一致 / 同名
   同ディレクトリ / -shm / -wal と衝突しないことを起動前に検証して
   F111R4RunRefused で停止
4. **source 文字列**: write SQL execute pattern (.execute("INSERT,
   UPDATE, DELETE, DROP, CREATE TABLE, executescript,
   executemany("INSERT) が runner / module source に一切無いことを
   test 検証

### auto_order_allowed_true_count == 0 確認

✅ summary.json: auto_order_allowed_true_count = 0
✅ build_advisory → ResearchAdvisory.__post_init__ →
   advisory_to_row → run_preview safety 検査の四重防御
✅ runner main 内の build_r4_summary 検査で >0 → exit 3

### manual_review_required_count == candidate_count 確認

✅ summary.json: manual_review_required_count = 400 = candidate_count
✅ run_preview / build_summary 内で equal を確認 (mismatch → exit 3)

### LINE 送信なし確認

✅ runner / module source に `notifications.line_bot` / `linebot` /
   `LineBotClient` / `send_text` / `push_message` の文字列なし
✅ argparse help に `--line` / `--send-line` option なし

### order / broker / rakuten / Computer Use 未接続確認

✅ runner / module source に `TradeOrder` / `TradeDecisionAgent` /
   `place_order` / `send_order` / `submit_order` の文字列なし
✅ `playwright` / `selenium` / `subprocess.run` の文字列なし
✅ argparse help に `--broker` / `--rakuten` / `--order` /
   `--computer-use` / `--playwright` option なし

### DB write なし / staging timestamp unchanged 確認

    target              before                       after                        status
    fire.staging.db     2026-05-09T22:40:35.385124   2026-05-09T22:40:35.385124   ✅
    fire.develop.db     May  7 18:14:26              unchanged                    ✅
    fire.db             May  7 16:12:38              unchanged                    ✅

✅ runner main 内で smoke 前後の last_modified 比較 (mismatch → exit 3)

### production / develop 無触

✅ smoke は --db staging / --db-path data/fire.staging.db のみ
✅ develop.db / fire.db (production 想定) の last_modified 完全 unchanged

---

## tests 結果

### 新規 46 PASS

    tests/agents/test_research_advisory_f119_inline.py (14)
    tests/scripts/jobs/
    test_run_f111_research_advisory_real_wiring_r4_smoke.py (32)

### regression

- F111 / F115 / F140 / F133 / F132 / F119 / F286 全テスト 0 件回帰
- フル pytest **2,874 PASS** (= 2,828 baseline + 46 新規)

---

## Codex pre-commit 結果

| commit | Codex 判定 |
|---|---|
| 74e2276 (feat module) | ✅ OK |
| fe8d170 (chore runner) | ✅ OK (★ CRITICAL 1 件 / 即修正後 OK) |
| 8dee93a (test)         | ✅ OK |

### CRITICAL 1 件と修正

最初の chore commit で Codex から下記 CRITICAL を受領:

> --output-json / --output-csv / --output-text / --summary-json /
> --completion-report が任意 Path に書けるため、
> --db-path data/fire.staging.db --output-json data/fire.staging.db
> のように指定すると read-only DB を通常ファイル書き込みで上書き
> 破壊できます。mode=ro と PRAGMA query_only=ON は sqlite 接続経由
> の write しか防げず、出力先 Path の衝突検査が必要です。

→ 即対応:
- `_assert_output_paths_safe` を runner 冒頭に追加
- output Path のいずれかが db_path / db_path-shm / db_path-wal /
  同名同ディレクトリと衝突する場合 F111R4RunRefused で exit 2
- 5 ケースの test (no_collision / exact / relative / wal-shm /
  main で --output-json=db_path 指定 → exit 2) を追加

✅ 全 3 commit で Codex review 通過
✅ 致命的指摘 1 件 → 修正 → 再 commit で OK
✅ pre-commit hook 正常通過 (`scripts/hooks/pre-commit`)

---

## --no-verify 未使用確認

✅ 全 3 commit で `--no-verify` flag 不使用
✅ Codex usage limit / rate limit / auth error なし
✅ 連続 retry なし (CRITICAL 後 1 回で修正完了)

---

## unrelated modified 未接触確認

git status の `Changes not staged for commit:` 欄に下記 2 ファイルが
F111-R4 commit 前後で残存:

- `scripts/seed_pattern_layer1.py` (本タスク無関係、未接触)
- `simulation/research_lane/historical_indicators.py` (同上、未接触)

3 commit すべて `git add <specific files>` で個別 stage。

---

## TODO Excel 未更新確認

✅ Google Sheets TODO 管理表は本タスクで更新していない

---

## 制約遵守チェックリスト

- ✅ 自動発注禁止 (4 重防御)
- ✅ 楽天証券操作の自動化なし
- ✅ Computer Use 未使用
- ✅ Playwright / Selenium / subprocess 未使用
- ✅ LINE 本番送信なし
- ✅ 発注は Fujiwara 手動実行前提
- ✅ production / develop / staging DB に書き込みなし
- ✅ DB write 0 件 (URI mode=ro + PRAGMA query_only=ON +
  output Path 衝突 guard の三重防御)
- ✅ staging も read-only でのみアクセス
- ✅ Codex pre-commit (× 3 全件通過、1 CRITICAL 即修正)
- ✅ --no-verify 禁止 (× 3 全件で flag 不使用)
- ✅ 個別 commit 厳守 (3 個別 commit)
- ✅ scripts/seed_pattern_layer1.py 未接触
- ✅ simulation/research_lane/historical_indicators.py 未接触
- ✅ unrelated modified を stage / commit しない
- ✅ TODO Excel 未更新

---

## 次タスク提案

### 第一候補: F062-R1 LINE Advisory Notification Template

F111-R4 で実用的な advisory metadata (boost / avoid / caution +
expected_h20) が real staging candidate に付くようになったので、
次は LINE 5 部屋向け template module を追加する。**ただし本番送信
は接続しない**:

- `notifications/templates/research_advisory.py` 新規作成
- F111-R4 preview row → LINE 簡易整形 multi-line text
- 5月情報通信 boost / 3月不動産 avoid / 8月 5d_warning caution が
  実 Fujiwara 向け文面になることを確認
- 本番送信 (LineBotClient.send_text) は F062-R2 で別タスク化
- safety lines / 「自動発注禁止」reminder を必ず付与

### 第二候補: F111-R5 アンサンブル評価

複数 source_version (= r2f4_baseline_v1 / r2f4_quality_defensive_v1
/ r2f4_risk_adjusted_v1) で wiring を回し、共通 boost 候補 / 共通
avoid 候補を抽出する read-only smoke。エッジ強化の検証。

### 第三候補: R2-G4 5d 用 rule 設計

F119 で確認された h5 短期 vs h20 中期の乖離 (cautious h5 -3.06% /
h20 +0.59%) を踏まえ、5 日保有用 interpretation rule を別 candidate
として設計し、R2-H 同様の orthogonal cuts を回す。

優先度: 1 > 2 > 3 (LINE 通知接続準備の流れに合わせる、F062-R1 で
先に safety 文言の本番転用を慎重に確認)

---

## 関連参照

- 02_todo/F111_R1_research_advisory_signal_integration.md (前段)
- 02_todo/F111_R2_research_advisory_preview.md (前段)
- 02_todo/F111_R3_research_advisory_real_wiring_smoke.md (直前段)
- 02_todo/F119_interpretation_evaluation.md
- 02_todo/F286_R2_H_orthogonal_cuts.md
- log.md milestone (本タスク完了時に追記)
