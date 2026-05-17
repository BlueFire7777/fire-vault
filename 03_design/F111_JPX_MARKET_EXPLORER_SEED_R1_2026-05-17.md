# FIRE F111-JPX-MARKET-EXPLORER-SEED-R1 (2026-05-17)

doc_id: FIRE-F111-JPX-MARKET-EXPLORER-SEED-R1-2026-05-17
status: 実装完了 / 56 PASS / smoke OK (= candidate_count=7、safety_flags 全 0)
HQ marker: (= 純実装 + read-only smoke + vault doc commit のみ別 HQ)
related:
- [[F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17]]
- [[F111_DATA_FRESHNESS_GATE_R1_2026-05-17]]
- [[F062_OPS_SUMMARY_FRESHNESS_ADAPTER_DESIGN_2026-05-17]]
- [[NIGHT_R0_READ_ONLY_BATCH_DESIGN_2026-05-17]]

---

## §1 目的

JPX Market Explorer 株式スクリーナー (= 公式 site の screener) を FIRE
の銘柄探索入口として採用する。本 wave R1 では:

1. **JPX Market Explorer の API / export / CSV download 可否** を調査
   項目として整理し、将来の R2 接続方針を vault doc に明記
2. **CSV / manual / local text を FIRE seed JSON へ変換する受け皿** を
   pure helper + runner として実装

JPX サイトへの自動アクセス / スクレイピング / Computer Use / 外部 HTTP は
本 wave R1 では行わない (= 別 wave 別 HQ marker 必須)。

---

## §2 成果物

### §2.1 fire repo (= develop branch、PR 別 wave)

| path | 役割 | LOC |
|---|---|---|
| `scripts/jobs/_f111_jpx_market_explorer_seed.py` | pure helper (= normalize / parse / dedupe / build / render / output guard) | ~310 |
| `scripts/jobs/run_f111_jpx_market_explorer_seed_preview.py` | CLI runner (= --input-csv / --input-text / output dir guard) | ~135 |
| `tests/scripts/jobs/test_run_f111_jpx_market_explorer_seed_preview.py` | unit + integration tests (= 56 PASS) | ~480 |

合計 production ~445 + tests ~480 = ~925 LOC

### §2.2 fire-vault (= 本 doc、別 commit)

`~/fire-vault/03_design/F111_JPX_MARKET_EXPLORER_SEED_R1_2026-05-17.md`

---

## §3 API / export 可否 調査設計 (= 本 wave 整理、R2 で実行)

### §3.1 公式 API 有無 確認項目 (= 全て fire repo / fire-vault 外で手動確認)

| 項目 | 確認方法 | 結果記入欄 |
|---|---|---|
| 公式 REST API の有無 | JPX 公式 site (= jpx.co.jp) を operator 手動確認 | (= 未確認) |
| Market Explorer 専用 API | https://gw.market-explorer.jpx.co.jp 等の endpoint 公開有無 | (= 未確認) |
| API 利用規約 | 個人利用許可 / レート制限 / 認証要否 | (= 未確認) |
| API レスポンス形式 | JSON / XML / CSV | (= 未確認) |
| API 認証 | API key 必要か (= secrets 管理対象になるか) | (= 未確認) |

→ 公式 API 確認は **operator の手動 web 閲覧 + 利用規約読了** で行う
   (= FIRE 自動 HTTP 取得は R1 範囲外)

### §3.2 CSV export / download 可否 確認項目

| 項目 | 確認方法 | 結果記入欄 |
|---|---|---|
| screener 画面に CSV export ボタンの有無 | operator 手動 UI 確認 | (= 未確認) |
| CSV header の format / encoding (= UTF-8 vs Shift_JIS) | 試しに 1 件 download して確認 | (= 未確認) |
| 1 export あたりの 上限件数 | (= e.g. 1000 件 等) | (= 未確認) |
| 認証要否 | 無料 / 会員 / 有料 | (= 未確認) |

→ CSV export ある場合、本 wave R1 の `--input-csv` で即対応可
   (= operator が download → CLI 渡し)

### §3.3 URL share / filter parameter 有無 (= R2 で URL 再現性検討)

| 項目 | 確認方法 | 結果記入欄 |
|---|---|---|
| 検索条件 URL の share 可能性 | screener で条件指定後 URL bar 確認 | (= 未確認) |
| URL parameter で条件再現可能か | URL コピー → 別 browser で開く | (= 未確認) |
| URL の安定性 (= 日付依存 / session 依存) | 翌日同じ URL で同条件再現 | (= 未確認) |

→ URL share 可能なら、conditions 文字列を screen_conditions field に保存し
   再現性 audit 用とする (= 本 R1 で受け皿 field あり)

### §3.4 4 案の優先順位 (= 採用方針)

| 優先 | 案 | scope | HQ marker |
|---|---|---|---|
| 1 (推奨) | 公式 API / export を operator が手動取得 → CSV file → 本 R1 runner | R1 範囲内 (= 既実装) | (= 不要、本 R1 で稼働) |
| 2 | URL share / parameter で条件再現 → operator 手動取得 → CSV | R1 範囲内 (= 同上、URL を screen_conditions に保存) | (= 不要) |
| 3 | 公式 API を **FIRE 自動取得** (= aiohttp / requests 等) | R2 / 別 wave | HQ_APPROVE_JPX_MARKET_EXPLORER_AUTOFETCH (= 利用規約確認後) |
| 4 (最終手段) | OpenClaw 画面操作 / Computer Use / Playwright スクレイピング | 最終手段、別 wave | HQ_APPROVE_JPX_BROWSER_AUTOMATION (= 利用規約 + 倫理確認後) |

### §3.5 API / export が無い場合の代替案

| 代替案 | 内容 | 制約 |
|---|---|---|
| local text 案 | operator が手で銘柄コードを テキストファイル に貼り付け → 本 R1 runner | 速い、手間中、件数制限なし |
| 画面 manual copy → CSV 整形 | operator が画面の表をコピペ → spreadsheet → CSV export | 1 件取りこぼし注意、本 R1 で対応 |
| 別 source 経由 (= 別 screener / 別 site) | 例: SBI screener / 株探 等 | source field を別値で記録、別 doc 化 |

### §3.6 スクレイピング R1 対象外

- BeautifulSoup / lxml / Selenium / Playwright は R1 production code に含めない
- tests の forbidden imports list に明記 (= `selenium / playwright / webdriver / urllib.request / http.client` を refuse)

---

## §4 seed JSON schema (= 1.0.0-f111-jpx-market-explorer-seed-r1)

```json
{
  "schema_version": "1.0.0-f111-jpx-market-explorer-seed-r1",
  "helper_version": "1.0.0-f111-jpx-market-explorer-seed-r1",
  "source": "jpx_market_explorer_screener",
  "screen_name": "出来高急増 + W2-B 強気",
  "screen_conditions": "vol_mom>+0.05 & srs>0",
  "import_method": "manual|csv|local_text",
  "base_date": "2026-05-17",
  "created_by": "fire-operator",
  "generated_at_jst": "2026-05-17T21:30:00+09:00",
  "candidates": [
    {
      "code": "92470",
      "display_code": "9247",
      "name": "TRE Holdings",
      "market": null,
      "sector": null,
      "source_rank": 1,
      "source_reason": null,
      "memo": null,
      "notes": null
    }
  ],
  "invalid_candidates": [
    { "raw_code": "ABCD", "reason": "non-digit: 'ABCD'", "name": null }
  ],
  "summary": {
    "candidate_count": 7,
    "invalid_count": 0,
    "duplicate_count": 0,
    "total_input": 7
  },
  "safety_flags": {
    "db_write": 0,
    "api_call": 0,
    "line_send": 0,
    "token_read": 0,
    "external_http": 0,
    "scraping": 0,
    "browser_automation": 0
  }
}
```

---

## §5 code normalization 仕様 (= 本 R1 確定)

| 入力例 | code (= 5 桁、J-Quants 規約) | display_code (= 4 桁表示) | valid |
|---|---|---|---|
| `9247` (= 4 桁) | `92470` | `9247` | ✓ |
| `92470` (= 5 桁末尾 0) | `92470` | `9247` | ✓ |
| `13580` (= 5 桁末尾 0、ETF 等普通株扱い) | `13580` | `1358` | ✓ |
| `13579` (= 5 桁末尾非 0、旧 JASDAQ 等特殊形式) | `13579` | `13579` | ✓ |
| `999` (= 3 桁) | (= invalid) | - | × |
| `123456` (= 6 桁) | (= invalid) | - | × |
| `ABCD` (= 非数字) | (= invalid) | - | × |
| `9247X` (= 数字 + 文字) | (= invalid) | - | × |
| `""` (= 空) | (= invalid) | - | × |
| `9247` (= int) | (= invalid、non-string) | - | × |
| `"  9247  "` (= 前後空白) | `92470` (= strip 後 normalize) | `9247` | ✓ |

### §5.1 duplicate 判定

- normalize 後の `code` (= 5 桁) で同一判定
- `9247` (= 4 桁) と `92470` (= 5 桁) は同一扱い (= 初出を残す)
- invalid 同士は duplicate 判定対象外 (= raw 保持)
- summary.duplicate_count に削除件数を記録

---

## §6 runner CLI

```
.venv/bin/python -m scripts.jobs.run_f111_jpx_market_explorer_seed_preview
  --input-csv PATH        | --input-text PATH      (= 排他、いずれか必須)
  --screen-name STR       (= 必須)
  --screen-conditions STR (= 任意、CSV 内 columns で代替可)
  --base-date YYYY-MM-DD  (= 必須)
  --import-method csv|manual|local_text  (= 任意、入力種別から自動)
  --created-by STR        (= default "fire-operator")
  --output-dir PATH       (= default /tmp/fire_jpx_market_explorer_seed)
  --strict                (= invalid 1 件以上で exit 4)
```

exit code:
- 0: success (= 全 valid)
- 2: argparse error / 入力 file 不在 / 排他違反
- 4: --strict 指定 + invalid_count > 0

output path guard: `/tmp` / `/private/tmp` / `~/fire` 配下のみ許可
(= F282 / F111 と同じ Path.is_relative_to() 方式、prefix sibling refuse)

---

## §7 tests (= 56 PASS)

### §7.1 test class 別

| class | scope | 件数 |
|---|---|---|
| TestNormalizeCode | 4 桁 / 5 桁末尾 0 / 5 桁特殊 / invalid / 空白 strip | 11 |
| TestParseCsvText | English / Japanese alias / 必須 column 欠落 / 空 / 空行 / case-insensitive | 10 |
| TestParseManualText | 基本 / コメント / 空白 / 空行 | 6 |
| TestDedupe | 4-5 桁同一 / 重複なし / invalid 維持 / 複数重複 | 4 |
| TestBuildSeedPayload | 基本 / 必須欠落 / format 不正 / invalid segregation | 7 |
| TestOutputPathGuard | 許可 root / 拒否 / prefix sibling refuse | 3 |
| TestRenderMarkdown | 基本 / pipe escape / invalid section | 3 |
| TestRunnerE2E | manual / CSV / 排他 / 必須欠落 / --strict / CSV→cli screen / CSV→csv conditions / cli override | 9 |
| TestSourceSafety | 禁止 import 11 token / safety_flags 全 0 | 3 |
| **合計** | | **56 PASS / 0.05 sec** |

### §7.2 source safety 観点 (= 禁止 import 11 token、helper + runner 両方確認)

`requests`, `aiohttp`, `sqlite3`, `linebot`, `subprocess`,
`urllib.request`, `http.client`, `selenium`, `playwright`,
`webdriver`, `websockets`

→ いずれも source 内に出現しない (= test で保証)

---

## §8 smoke (= 5/17 21:xx JST)

```
[input.txt]
# JPX Market Explorer 抽出例 (= 2026-05-17 想定)
9247 TRE Holdings
9628 燦HD
7595 アルゴグラフィックス
9008 京王
3134 Hammock
9417 スマートバリュー
4404 ミヨシ油脂

[runner output]
[jpx-seed] wrote JSON: /private/tmp/fire_jpx_market_explorer_seed_smoke/jpx_market_explorer_seed.json
[jpx-seed] wrote MD:   /private/tmp/fire_jpx_market_explorer_seed_smoke/jpx_market_explorer_seed.md
[jpx-seed] candidate_count=7 invalid=0 duplicate=0

[summary]
- source: jpx_market_explorer_screener
- screen_name: 出来高急増 + W2-B 強気
- screen_conditions: vol_mom>+0.05 & srs>0
- candidate_count: 7
- first candidate: { code: '92470', display_code: '9247', name: 'TRE Holdings', source_rank: 1 }
- safety_flags: 全 0 ✓
```

---

## §9 Codex 8 lane 相当 self-audit

(= 本 wave は CLI 並列実行ではなく self-audit 表で代替、merge 前に CLI 実 review 別 wave)

| lane | 観点 | verdict | 摘要 |
|---|---|---|---|
| A | seed schema | APPROVE | source / screen_name / import_method / base_date 必須、safety_flags 7 値、provenance 完備 |
| B | code normalization | APPROVE | 4 桁→5 桁 / 5 桁末尾 0→display 4 桁 / 5 桁末尾非 0→そのまま / invalid 4 種、11 testcase |
| C | CSV / manual edge | APPROVE | English/Japanese alias、case-insensitive、空白 strip、空行 / コメント skip、混合 dup detect |
| D | output clarity | APPROVE | JSON pretty + MD table、invalid section / safety_flags 明示、provenance line 3 種 |
| E | safety / no DB API token LINE | APPROVE | 禁止 11 import token を src で 0 確認 + tests 強制、safety_flags 全 0、output path guard is_relative_to |
| F | F111 / W2-B / Paper Live 連携 | APPROVE | seed JSON は将来 F111 candidate / W2-B classifier / Paper Live への共通入力 (= §10 提示) |
| G | API / export 調査設計 | APPROVE | 4 案 + 優先順位、スクレイピング R1 対象外明示、operator 手動取得 + 既 R1 runner で連携可 |
| H | next-wave sequencing | APPROVE | R1 完了→ operator 手動取得運用→ R2 API/URL share 検討→ R3 OpenClaw / 自動取得 (= 別 HQ) |

CRITICAL: 0 / HIGH: 0
MEDIUM 検出:
- M1: ETF/REIT 等の特殊 4 桁コード (= 1306, 1577) も 4 桁→5 桁正規化される
  (= 期待動作、display は 4 桁維持)
- M2: source_reason field は本 R1 では未充填、CSV alias 追加検討は別 wave

---

## §10 F111 / W2-B / Paper Live 連携案 (= 次 wave 候補)

### §10.1 F111 candidate smoke 連携

```
[本 R1 seed JSON]
   ↓ (= operator 手動 or NIGHT-R0 step 4 拡張)
[F111 candidate generator (= 既)]
   - seed.candidates[].code を whitelist として優先 candidate に入れる
   - raw_score 算出時に screen_conditions を notes に保持
   ↓
[overlay top5]
```

### §10.2 W2-B classification 連携

```
[本 R1 seed JSON candidates]
   ↓ (= staging DB read-only join)
[W2-B classifier (= 既 _f111_overlay_momentum_sector.py)]
   - 各 candidate code に vol_mom / srs / 5 label を算出
   ↓
[seed JSON へ enrichment 追加版を別 wave で出力可]
```

### §10.3 Paper Live 検証 連携

```
[本 R1 seed JSON]
   ↓ (= F054 Paper Live tick の input candidate)
[Paper Live virtual entry / exit]
   - seed の screen_name 別に PnL 集計
   - screen_conditions 別の hit ratio 算出
   ↓
[F119 Evaluation に screen 別 KPI 追加可]
```

### §10.4 連携実装の wave 分解 (= 別 wave)

| wave 候補 | 連携対象 | 必要 HQ marker |
|---|---|---|
| JPX-SEED-R2-F111-HOOK | F111 candidate に whitelist 流入 | (= 別 wave、read-only) |
| JPX-SEED-R2-W2B-ENRICH | W2-B label を seed JSON に enrich | (= staging read-only) |
| JPX-SEED-R3-PAPER-LIVE | F054 入力に seed を渡す | (= Paper Live ramp-up 要、別 HQ) |
| JPX-SEED-R4-AUTOFETCH | 公式 API 自動取得 | HQ_APPROVE_JPX_MARKET_EXPLORER_AUTOFETCH |

---

## §11 安全 gate

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| staging write / production-develop | 0 |
| financials refresh / retry | 0 |
| 実 J-Quants API call | 0 |
| TDnet / XBRL 取得 | 0 |
| 実 JPX site HTTP call | 0 (= R1 範囲外、operator 手動取得想定) |
| スクレイピング | 0 (= 禁止 11 import token で enforce) |
| Computer Use / Playwright / Selenium | 0 (= 同上) |
| token / .env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| F111 本体への恒久組込 | 0 (= 本 R1 は受け皿のみ、F111 candidate generator 不変) |
| Paper Live 実行 | 0 |
| schema migration | 0 |
| auto-order / 楽天 / iSPEED | 0 |
| --no-verify / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |
| main 直 push (= fire repo) | 0 (= develop branch に commit、PR 別 wave) |

3 DB md5 (= wave 前後完全一致):
- `data/fire.db`: b1df4673... ✓
- `data/fire.develop.db`: 085799da... ✓
- `data/fire.staging.db`: 71a63a19... ✓

---

## §12 次 wave 候補 (= 順序)

| 優先 | wave | scope | HQ marker |
|---|---|---|---|
| 1 | 本 R1 doc + code commit/push (= 別 step) | docs / code / tests | HQ_APPROVE_JPX_SEED_R1_COMMIT_PUSH |
| 2 | PR + main merge | merge to main | HQ_APPROVE_JPX_SEED_R1_PR_MERGE |
| 3 | JPX 公式 API / export 調査 (= operator 手動) | web 閲覧 + 利用規約 + 試行 1 件 | (= 不要、手動作業) |
| 4 | JPX-SEED-R2-F111-HOOK | F111 candidate whitelist 連携 | (= 別 HQ、read-only) |
| 5 | JPX-SEED-R2-W2B-ENRICH | W2-B enrich (= staging read-only) | (= 別 HQ) |
| 6 | JPX-SEED-R3-PAPER-LIVE | Paper Live 入力連携 | (= 別 HQ、Paper Live ramp 要) |
| 7 | JPX-SEED-R4-AUTOFETCH | API 自動取得 (= 利用規約 OK 時) | HQ_APPROVE_JPX_MARKET_EXPLORER_AUTOFETCH |

---

## §13 success 判定

| 基準 | 結果 |
|---|---|
| JPX Market Explorer 候補 を FIRE seed JSON にできる | ✓ (= manual 7 件 / CSV 3 件 smoke、56 tests PASS) |
| API / export 有無の調査観点と R2 方針 明確化 | ✓ (= §3、4 案 + 優先順位、operator 手動が R1 主軸) |
| 既存 F111 本体は変更しない | ✓ (= 新規 3 file のみ、F111 candidate generator 不変) |
| DB / API / LINE / token | 0 (= §11 全 0) |
| tests PASS | ✓ (= 56 PASS / 0.05 sec) |
| F111 candidate smoke / Paper Live 検証への接続案 提示 | ✓ (= §10、4 wave 分解) |

→ **全成功基準 充足、本 R1 wave 完了 ✓**
   (= 残: fire repo develop commit/push + vault commit/push、次 step で実施)
