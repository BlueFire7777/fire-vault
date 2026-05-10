# F062-R1 LINE Advisory Notification Template

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: F111-R4 real wiring artifacts
> (74e2276 / fe8d170 / 8dee93a)
> **Mode**: 完全 dry-run / template only
> (LINE 本番送信 / token / DB 全て未接続)
> **Result**: ★★★ F111-R4 400 候補から avoid 5 / boost_with_avoid 5
> / caution 5 / boost_with_caution 5 = 20 件を選抜、3000 文字制約で
> 5 chunks に分割した LINE preview text を生成。forbidden 0 / safety
> footer 全 chunk / line_send_count 0 / token_read_count 0 / DB
> unchanged。次の F062-R2 (本番送信導線) 接続準備完了。★★★

---

## タスク名

F062-R1 LINE Advisory Notification Template

---

## 背景

F111-R4 で staging 実 400 候補に F119 由来の boost / avoid / caution
+ expected_h20 が実際に付くことが確認できた。

advisory_label_counts (F111-R4 smoke):

    boost_with_caution: 217
    avoid:               83
    caution:             67
    boost_with_avoid:    33

これら advisory metadata を Fujiwara が **LINE で読みやすい候補レビュー
文面** に変換するのが F062-R1 の目的。

★ ただし本タスクでは **LINE 本番送信は行わない**。
- LineBotClient / channel token / send_text / push_message を一切
  使わない
- F062-R2 で dry-run / production 分離した送信導線を別途検討

---

## F111-R4 からの接続内容

| F111-R4 (input) | F062-R1 (本タスク) |
|---|---|
| /tmp/f111_r4_real_wiring_preview.json | --preview-json (= advisory rows list) |
| /tmp/f111_r4_real_wiring_summary.json | --summary-json (optional) |
| advisory_label / boost/avoid/caution_flags / expected_h20 / win | LINE 候補ブロック整形に使用 |
| manual_review_required=True / auto_order_allowed=False | 起動時に厳密 bool 検査 (= refuse if 違反) |
| F119 strong/avoid/caution 分類 | LABEL_PRIORITY (avoid 先頭、boost_with_avoid は警告寄り) |

---

## LINE preview template 仕様

### Label 分類と表示優先順

    LABEL_PRIORITY (= 重要 / 注意ほど上):
      1. avoid              【見送り候補】
      2. boost_with_avoid   【要注意】           ← boost あっても警告寄り
      3. suppress           【抑制候補】
      4. caution            【注意候補】
      5. boost_with_caution 【有望・注意あり】
      6. boost              【有望候補】
      7. neutral            【参考候補】
      8. missing_advisory   【advisory 未付与・参考】

DEFAULT_INCLUDE_LABELS は 1-6 (neutral / missing は default 除外)。

### chunk 分割

- max_chars default 3000
- 1 chunk に収まらない場合は複数 chunk に分割
- 各 chunk 先頭に "{title} (i/N)" prefix
- 各 chunk に header + body + safety footer を必ず付与

### safety footer (全 chunk 末尾に強制付与)

    Safety
    - LINE 送信なし (dry-run / template only)
    - 自動発注なし
    - 楽天操作なし
    - Computer Use なし
    - 注文価格・数量・執行指示は生成しない
    - F119 の優位は 20d 寄り (daytrade 即時判断には使わない)
    - Fujiwara manual review required

### forbidden phrases (= 出力 text に絶対含めない)

13 種:
- 買え / 発注せよ / 発注しろ / 発注して
- 自動で注文 / そのまま約定 / そのまま発注
- 楽天で実行
- 確実に勝てる / 必ず勝てる / 確実に儲かる / 必ず儲かる / 必勝

`assert_safety_invariants` が全 chunk を検査、検出すれば
`AdvisoryLineSafetyViolation` raise。

### bool 厳密検査 (Codex CRITICAL #2 対応)

`assert_row_safety_flags`:
- `isinstance(v, bool)` を要求
- str "true" / "false" / int 1 / 0 / list [] / dict / None / 欠落
  全て refuse
- True bool のみ通過

CSV loader も "true" / "false" 以外は変換せず生値のまま、
`assert_row_safety_flags` で非 bool として refuse される設計。

---

## 実装ファイル一覧

| 種別 | ファイル | 行数 |
|---|---|---|
| feat | `notifications/templates/research_advisory.py` | 563 |
| chore (runner) | `scripts/jobs/run_f062_research_advisory_line_preview.py` | 588 |
| fix | (上記 2 file への bool tighten 修正) | +26/-4 |
| test | `tests/notifications/templates/test_research_advisory_line_template.py` | 45 PASS |
| test | `tests/scripts/jobs/test_run_f062_research_advisory_line_preview.py` | 20 PASS |

### モジュール構成

    notifications/templates/
    └── research_advisory.py (563 行、新規)
        ├── LABEL_AVOID / BOOST_WITH_AVOID / SUPPRESS / CAUTION /
        │   BOOST_WITH_CAUTION / BOOST / NEUTRAL / MISSING /
        │   LABEL_PRIORITY / LABEL_HEADING / LABEL_NOTE /
        │   DEFAULT_INCLUDE_LABELS
        ├── FORBIDDEN_PHRASES (13 種) / SAFETY_FOOTER_LINES /
        │   HEADER_LINES_FIXED
        ├── DEFAULT_MAX_CANDIDATES (20) / MAX_PER_LABEL (5) /
        │   MAX_CHARS (3000)
        ├── AdvisoryLineSafetyViolation (例外)
        ├── format_header / format_summary / format_candidate /
        │   format_footer
        ├── select_candidates (priority sort + max_per_label +
        │   include / exclude filter)
        ├── chunk_text (max_chars 分割 + title prefix + footer)
        ├── AdvisoryLinePreview (dataclass)
        ├── build_advisory_line_preview (★ Top-level、内部で
        │   assert_row_safety_flags を強制呼び出し)
        ├── assert_safety_invariants (forbidden / footer 検査)
        └── assert_row_safety_flags (bool 厳密 + 必須キー検査)

    scripts/jobs/
    └── run_f062_research_advisory_line_preview.py (588 行、新規)
        ├── F062R1RunRefused
        ├── load_preview_json / load_preview_csv (bool / list 復元) /
        │   load_summary_json
        ├── build_r1_summary (token_read_count=0 / line_send_count=0)
        ├── write_completion_report
        ├── build_arg_parser (--send / --line / --token / --write 等
        │   forbidden option を作らない)
        ├── parse_args (input 排他 + max > 0 検証)
        └── main: refused → 2 / safety violation → 3 / 0 件 → 4 / 0

    tests/  (65 PASS = 45 + 20)
    ├── tests/notifications/templates/
    │   test_research_advisory_line_template.py (45)
    │   - LABEL_PRIORITY constants 4
    │   - format_header/summary/footer 4
    │   - format_candidate 各 label 7
    │   - select_candidates 9
    │   - chunk_text 3
    │   - assert_safety_invariants forbidden 全種 1+1
    │   - assert_row_safety_flags 非 bool / 欠落 / list 7
    │   - build_advisory_line_preview pipeline 5
    │   - module source safety 2
    └── tests/scripts/jobs/
        test_run_f062_research_advisory_line_preview.py (20)
        - parse_args 5 / arg parser safety 1
        - load_preview_json/csv/summary 5
        - build_r1_summary 1
        - main full pipeline / exit codes 5
        - runner source safety 3

---

## commit hash 一覧

| commit | type | 内容 |
|---|---|---|
| `4ebf285` | feat | F062-R1: add research advisory LINE preview template |
| `7b6ad6c` | chore | F062-R1: add research advisory LINE preview runner |
| `0f1254f` | fix | F062-R1: tighten bool type check on safety flags |
| `4023577` | test | F062-R1: add LINE preview template tests |
| (本 commit) | docs | F062-R1: vault |
| (次 commit) | docs | F062-R1: log milestone |

---

## smoke 条件

### 入力

    --preview-json:  /tmp/f111_r4_real_wiring_preview.json
                     (= F111-R4 400 候補 advisory rows)
    --summary-json:  /tmp/f111_r4_real_wiring_summary.json

### selection options

    --max-candidates: 20
    --max-per-label:  5
    --max-chars:      3000
    --dry-run:        true (default)
    --include-labels: default (avoid / boost_with_avoid / suppress /
                                caution / boost_with_caution / boost)

### 実行コマンド

    .venv/bin/python -m \
      scripts.jobs.run_f062_research_advisory_line_preview \
      --preview-json /tmp/f111_r4_real_wiring_preview.json \
      --summary-json /tmp/f111_r4_real_wiring_summary.json \
      --output-json /tmp/f062_r1_line_preview_payload.json \
      --output-text /tmp/f062_r1_line_preview.txt \
      --output-summary-json /tmp/f062_r1_line_preview_summary.json \
      --completion-report /tmp/f062_r1_completion_report.txt \
      --max-candidates 20 --max-per-label 5 --max-chars 3000 --dry-run

---

## output artifacts 一覧

| ファイル | サイズ |
|---|---|
| /tmp/f062_r1_line_preview_payload.json | 74 KB |
| /tmp/f062_r1_line_preview.txt | 16 KB |
| /tmp/f062_r1_line_preview_summary.json | 20 KB |
| /tmp/f062_r1_completion_report.txt | 1.8 KB |

---

## smoke 結果

### 集約結果

    input_candidate_count:           400
    selected_candidate_count:        20
    message_chunk_count:             5
    selected_label_counts:
      avoid:               5
      boost_with_avoid:    5
      caution:             5
      boost_with_caution:  5
      (boost: 0 = R4 入力に boost 単独 label が無く、boost 系は全て
       boost_with_caution / boost_with_avoid に分類されているため)
    label_counts (= 入力 400 件全体):
      boost_with_caution: 217
      avoid:               83
      caution:             67
      boost_with_avoid:    33
    auto_order_allowed_true_count:   0    ★
    manual_review_required_count:    400  (= input_candidate_count)
    forbidden_phrase_count:          0    ★
    forbidden_matches:               []
    safety_footer_present:           True
    token_read_count:                0    ★
    line_send_count:                 0    ★

### 出力 text の例 (avoid 候補ブロック先頭)

    【見送り候補】96280
    label: avoid
    sector/month: 情報通信・サービスその他 / 3月
    rank: A1 / post_cap_rank 11
    F119: h20 -8.58% / win 13.3%
    flags:
    - avoid: sector_17_month:情報通信・サービスその他 × 03
      (n=30, h20=-8.58%/win=13.3%), ...
    - caution: overall:overall (n=400, h20=+3.66%/win=60.9%), ...
    note: F119 上 avoid 条件が該当。原則見送り、手動レビューでも
    慎重扱い。

### 各 label の見出し動作確認

✅ 【見送り候補】 (avoid)
✅ 【要注意】 (boost_with_avoid) ← boost ありでも警告寄り表示
✅ 【注意候補】 (caution)
✅ 【有望・注意あり】 (boost_with_caution)
✅ 【有望候補】 (boost、test ケースで検証)
✅ 【参考候補】 (neutral、test ケースで検証)
✅ 【advisory 未付与・参考】 (missing、test ケースで検証)

---

## 安全要件遵守

### auto_order_allowed_true_count == 0 確認

✅ summary.json: 0
✅ build_advisory_line_preview が assert_row_safety_flags を強制
   呼び出し (= caller が runner 経由でなくとも違反 row を refuse)
✅ runner main も二重防御で再検査

### manual_review_required_count == candidate_count 確認

✅ summary.json: 400 = input_candidate_count
✅ assert_row_safety_flags で True 以外を refuse

### LINE 送信なし確認

- ✅ source に `from notifications.line_bot` / `import linebot` /
  `LineBotClient(` / `.send_text(` / `.push_message(` /
  `.reply_message(` 不在 (= test で source 文字列検証)
- ✅ argparse help に `--send` / `--send-line` / `--line-token` /
  `--channel-token` / `--line` option 不在
- ✅ summary に `line_send_count: 0` を出力

### token 読み込みなし確認

- ✅ source に `load_dotenv(` / `os.environ[` / `os.environ.get(` /
  `os.getenv(` / `from dotenv` / `import dotenv` /
  `LINE_CHANNEL_TOKEN` / `LINE_CHANNEL_SECRET` 不在
- ✅ summary に `token_read_count: 0` を出力

### order / broker / rakuten / Computer Use 未接続確認

- ✅ source に `TradeOrder(` / `.place_order(` / `.send_order(` /
  `.submit_order(` / `from agents.trade_decision` 不在
- ✅ source に `import playwright` / `import selenium` /
  `import subprocess` / `subprocess.run` 不在
- ✅ argparse help に `--broker` / `--rakuten` / `--order` /
  `--auto-order` / `--computer-use` / `--playwright` 不在

### DB access 0 / DB write 0 確認

- ✅ source に `import sqlite3` / `sqlite3.connect` 不在
- ✅ runner 本体は input file (--preview-json / --preview-csv /
  --summary-json) のみ読み、DB を一切開かない
- ✅ smoke 前後の staging.db / develop.db / fire.db last_modified
  全て unchanged

### forbidden phrases 検査結果

- ✅ FORBIDDEN_PHRASES 13 種を全 chunk に対して走査、smoke で 0 件
- ✅ test で各 forbidden phrase を含む chunk が refuse されることを
  検証 (parametrized 風 13 ケース)

### safety footer 確認

- ✅ smoke の全 5 chunks 末尾に "Safety" / "LINE 送信なし" /
  "自動発注なし" / "楽天操作なし" / "Computer Use なし" / "Fujiwara
  manual review required" が入る
- ✅ test で safety footer 欠落 chunk が refuse されることを検証

---

## DB unchanged / production・develop・staging 無触

    target              before                       after                        status
    fire.staging.db     2026-05-09T22:40:35.385124   unchanged                    ✅
    fire.develop.db     May  7 18:14:26              unchanged                    ✅
    fire.db             May  7 16:12:38              unchanged                    ✅

★ runner module / template module ともに sqlite3 を import せず、
   DB connect 経路が構造的に無い。

---

## tests 結果

### 新規 65 PASS

    tests/notifications/templates/
      test_research_advisory_line_template.py (45 PASS)
    tests/scripts/jobs/
      test_run_f062_research_advisory_line_preview.py (20 PASS)

### regression

- F062 / F111 / F115 / F140 / F133 / F132 / F119 / F286 全テスト
  0 件回帰
- フル pytest **2,939 PASS** (= 2,874 baseline + 65 新規)

---

## Codex pre-commit 結果

| commit | Codex 判定 |
|---|---|
| 4ebf285 (feat template) | ✅ OK (★ CRITICAL #1 対応版で再 commit) |
| 7b6ad6c (chore runner)  | ✅ OK |
| 0f1254f (fix bool tighten) | ✅ OK (★ CRITICAL #2 対応) |
| 4023577 (test)           | ✅ OK |

### CRITICAL 2 件と修正

#### CRITICAL #1: build_advisory_line_preview が
assert_row_safety_flags を強制していない

> dry-run 安全テンプレートとしての不変条件が実際には強制されて
> いません。

→ `build_advisory_line_preview` 内で `assert_row_safety_flags` を
   強制呼び出し、template 単体で auto_order_allowed=True の row
   入力を refuse。test で 2 ケース追加検証。

#### CRITICAL #2: bool 型を厳密検査せず str truthy 値が
すり抜けうる

> `auto_order_allowed: "true"` のような文字列 truthy 値が
> `is True` 判定をすり抜けます。安全フラグは厳密に bool 型へ
> バリデーションし、不正型は refuse すべきです。

→ `assert_row_safety_flags` を `isinstance(v, bool)` 厳密検査に
   変更、CSV loader も "true" / "false" 以外を変換せず生値のまま
   残して非 bool として refuse させる。test で 4 ケース追加検証
   (str "false" / int 1 / list [] / 必須キー欠落)。

✅ 全 4 commit で Codex review 通過
✅ CRITICAL 2 件 → 即修正 → 再 commit で OK
✅ pre-commit hook 正常通過 (`scripts/hooks/pre-commit`)

---

## --no-verify 未使用確認

✅ 全 4 commit で `--no-verify` flag 不使用
✅ Codex usage limit / rate limit / auth error なし
✅ 連続 retry なし (CRITICAL は 1 回ずつで修正完了)

---

## 個別 commit 遵守確認

✅ feat (template) / chore (runner) / fix (bool tighten) / test の
   4 commit に分割
✅ 1 度 fix と test がまとまった commit (db2c08a) を作成してしまい、
   `git reset --soft HEAD^` で巻き戻し → tests を unstage → fix
   commit → test commit と再分割して個別 commit 規律を維持
✅ まとめ commit なし

---

## unrelated modified 未接触確認

git status の `Changes not staged for commit:` 欄に下記 2 ファイルが
F062-R1 commit 前後で残存:

- `scripts/seed_pattern_layer1.py` (本タスク無関係、未接触)
- `simulation/research_lane/historical_indicators.py` (同上、未接触)

4 commit すべて `git add <specific files>` で個別 stage、`git add .`
や `git add -A` は不使用。

---

## TODO Excel 未更新確認

✅ Google Sheets TODO 管理表は本タスクで更新していない

---

## 制約遵守チェックリスト

- ✅ 自動発注禁止 (LineBotClient / send_text 排除 + bool 厳密検査
  + assert_row_safety_flags 強制)
- ✅ LINE 本番送信なし (4 commit すべて source 文字列検証で確認)
- ✅ token / channel secret 読み込みなし
- ✅ .env / dotenv 読み込みなし
- ✅ 楽天証券操作なし
- ✅ Computer Use 未使用
- ✅ Playwright / Selenium / subprocess 未使用
- ✅ DB write 0 (sqlite3 unimported)
- ✅ DB access 0 (input file のみ読む、DB connect 経路なし)
- ✅ production / develop / staging DB 無触
  (smoke 前後で last_modified 完全 unchanged)
- ✅ Codex pre-commit (× 4 全件通過、CRITICAL 2 件即修正)
- ✅ --no-verify 禁止 (× 4 全件で flag 不使用)
- ✅ 個別 commit 厳守 (4 個別 commit)
- ✅ scripts/seed_pattern_layer1.py 未接触
- ✅ simulation/research_lane/historical_indicators.py 未接触
- ✅ unrelated modified を stage / commit しない
- ✅ TODO Excel 未更新

---

## 次タスク提案

### 第一候補: F062-R2 LINE Send dry-run / production 分離

F062-R1 で template / dry-run 文面が固まったので、次は **本番送信
導線** を慎重に追加する:

- `notifications/router.py` または新規 module で advisory 文面の
  ルーティング追加 (= LINE 5 部屋のうち ENTRY 部屋等)
- `--dry-run` (default) / `--send` の厳密分離
- production token は env から読むが、明示 `--send` 指定時のみ
- safety footer / forbidden phrases 検査を本番送信パスでも強制
- send 前に `assert_row_safety_flags` を再検査
- send 履歴を log に残す (notifications/notifications_line.log と
  整合)
- 既存 LineBotClient (notifications/line_bot.py) の dry_run mode
  を活用

### 第二候補: F111-R5 アンサンブル評価

複数 source_version (= r2f4_baseline_v1 / r2f4_quality_defensive_v1
/ r2f4_risk_adjusted_v1) で wiring を回し、共通 boost / 共通 avoid
候補を抽出する read-only smoke。エッジ強化検証。

### 第三候補: R2-G4 5d 用 rule 設計

F119 で確認された h5 短期 vs h20 中期の乖離を踏まえた 5 日保有用
interpretation rule を別 candidate として設計。

優先度: 1 > 2 > 3 (F062-R2 で本番送信を慎重に分離してから次へ)

---

## 関連参照

- 02_todo/F111_R1_research_advisory_signal_integration.md (前段)
- 02_todo/F111_R2_research_advisory_preview.md (前段)
- 02_todo/F111_R3_research_advisory_real_wiring_smoke.md (前段)
- 02_todo/F111_R4_multi_base_date_f119_insights_smoke.md (直前段)
- 02_todo/F119_interpretation_evaluation.md
- log.md milestone (本タスク完了時に追記)
