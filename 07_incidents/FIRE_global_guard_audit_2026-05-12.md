# FIRE Global Guard Audit Report (= Wave 11 W11-4)

date: 2026-05-12
lane: L4 Audit (= 横断 / 全体監査)
scope: Wave 1-10 全 F286 シリーズ production code

CRITICAL: 0 / HIGH: 1

static_review_only: pytest 実行 0 / DB write 0 / LINE 送信 0 / code change 0

## CRITICAL findings

なし。

## HIGH findings

### HIGH-1: LINE preview の複数 chunk 時に「全 chunk が単一コードフェンス」を満たせない

- 対象: `fire/report/line_delivery.py`, `scripts/jobs/run_f286_report_r1_line_preview.py`
- 根拠:
  - `fire/report/markdown_renderer.py` は daily/weekly/monthly で全体 Markdown を ` ```text ... ``` ` の単一 outer fence に包み、内部 fence は `replace("```", "'''")` で潰している。
  - しかし `fire/report/line_delivery.py` の `_generate_preview()` は、fence 済み Markdown 全体を `_chunk_text()` で単純分割している。
  - `scripts/jobs/run_f286_report_r1_line_preview.py` は `preview.chunks` をそのまま stdout / JSON に出す。
- 影響:
  - `total_chunks == 1` の場合は OK。
  - `total_chunks > 1` の場合、先頭 chunk だけが opening fence を持ち、末尾 chunk だけが closing fence を持つ形になり得る。各 chunk 単位では「単一コードフェンス + 内部 fence なし」を満たさない。
- 既存テストの穴:
  - `tests/report/test_line_delivery.py` は `body = "".join(preview.chunks)` で全体結合後の fence 数だけを見ており、chunk ごとの fence 健全性を検査していない。
- 判定:
  - F 観点は NG。
  - LINE 送信は存在しないため即時の secret / send incident ではないが、iPhone コピー format の受入条件違反として HIGH。

## MEDIUM findings

### MEDIUM-1: `run_f286_data_r3_daily_refresh.py` の staging write helper は六段ガードではない

- 対象: `scripts/jobs/run_f286_data_r3_daily_refresh.py`
- 根拠:
  - `_assert_staging_write_allowed()` は `db_label in WRITE_ALLOWED_DB_LABELS` と `db_path.name in WRITE_ALLOWED_DB_BASENAMES` を確認する。
  - CLI の `--write` 経路は `plan_daily_refresh(... read_only=False)` で `NotImplementedError` となり、現状は実 write に到達しない。
  - ただし `ensure_daily_refresh_schema()` は import 経由で直接呼ぶと staging label/basename のみで `CREATE TABLE IF NOT EXISTS` を実行でき、symlink / resolved basename / hardlink inode guard はない。
- 影響:
  - 現 CLI では write 不到達なので CRITICAL/HIGH ではない。
  - 将来 `--write --ensure-schema` を有効化する前に、W4-2/W6/W9-1a と同じ symlink / resolved basename / forbidden inode / post-connect inode verify を再利用すべき。

## LOW findings / note

- `requests` import は `scripts/jobs/fetch_historical_market_data.py` と `scripts/jobs/fetch_announcements.py` のみで検出。audit 条件の fetch 系例外に該当。
- `subprocess.run` は `scripts/jobs/run_f286_data_r3_daily_refresh.py` のみで検出。F286-DATA-R3 で承認済みの dry-run subrunner 実行用途に限定され、`--write` と同時実行しない guard がある。
- LINE token / channel secret の直接参照は F286 production scope では検出なし。`fire/report/line_delivery.py` は `F286_LINE_HQ_APPROVE` の label 値のみ確認する。
- `fetch_historical_market_data.py` / `fetch_announcements.py` は `JQUANTS_*` env key の存在確認を行うが、dry-run probe では値を読まず、stdout/stderr に値を出さない。
- `pnl/paper_pnl.py` に `compute_paper_entry_qty()` がある。これは F130 許容損失ベースの paper PnL 用仮想数量で、docstring 上も「実注文数量や執行指示ではない」と明記されている。`execute_order` / broker / Rakuten SDK 経路は検出なし。

## 観点別 verdict (= A-G)

| 観点 | verdict | 詳細 |
|---|---|---|
| A. forbidden import 完全排除 | OK | F286 production scope で LINE SDK / aiohttp / Playwright / pyautogui / Rakuten SDK は検出なし。`requests` は fetch 系 2 file の許容例外のみ。`subprocess.run` は daily_refresh の承認済み例外のみ。 |
| B. token / secret 不参照 | OK | LINE token / channel secret / Rakuten secret の直接参照なし。HQ marker は `F286_MIG_R1_HQ_APPROVE` と `F286_LINE_HQ_APPROVE` の label 値確認のみ。J-Quants fetch 系は env key 存在確認のみ。 |
| C. read-only enforcement | OK | report runner は `mode=ro` + `PRAGMA query_only=ON`。F286 write SQL は staging-only / dry-run / HQ marker のいずれかに閉じる。daily_refresh helper は MEDIUM note あり。 |
| D. staging-only / refuse 整合性 | OK | W4-2/W6/W9-1a 系 runner は `--db-label staging` / `fire.staging.db` / symlink refuse / resolved basename / forbidden inode / post-connect inode verify を確認。daily_refresh は対象外寄りだが将来 write 有効化前に補強推奨。 |
| E. HQ approve marker 運用 | OK | MIG-R1 は `F286_MIG_R1_HQ_APPROVE == db_label`、LINE send guard は `F286_LINE_HQ_APPROVE == production`。marker bypass 経路は検出なし。 |
| F. iPhone コピー format | NG | renderer 単体は単一 outer fence OK。ただし LINE preview chunk が複数になると chunk 単位の単一 fence 条件を満たせない。HIGH-1。 |
| G. 注文 / 自動発注 helper 不在 | OK | `execute_order` / `order_price` / Rakuten SDK / Computer Use / Playwright 経路なし。`compute_paper_entry_qty()` は paper PnL 仮想数量であり自動発注 helper ではない。 |

## 総合判断

- 本 audit verdict: 要対応
- merge_recommendation: HIGH-1 を修正してから merge 推奨。LINE preview の各 chunk を独立した ` ```text ... ``` ` に包む、または chunk 出力を 1 chunk に制限して partial を refuse する方針が必要。
- 次フェーズ提言:
  - `tests/report/test_line_delivery.py` に chunk 単位で `chunk.startswith("```text\n")`, `chunk.endswith("\n```")`, `chunk.count("```") == 2` を検査するテストを追加。
  - `run_f286_data_r3_daily_refresh.py` の将来 write 有効化前に staging DB guard を W4-2/W6/W9-1a と同じ helper へ寄せる。
  - `fire/report/line_delivery.py` の `env = dict(os.environ)` は現状 secret 出力なしだが、長期 governance では `os.environ.get("F286_LINE_HQ_APPROVE")` の単一 key 読みへ絞ると監査面が明確。

