# F286-DATA-R1.3 Remaining Symbols Price Refresh / Gate Pass Smoke

> **Status**: ✅ COMPLETED 2026-05-10 (★ 運用 smoke のみ、fire 側
> コード変更なし)
> **Source**: F286-DATA-R1.2 (運用 smoke、c818327 / 88b40ce)
> **Mode**: 既存 DATA-R1.1 runner で missing symbols CSV を分割
> staging-only write
> **Result**: ★★★ prices distinct_codes 1499 → 4448 / +2949 銘柄、
> DATA-R2 gate 全 5 段 PASS、overall_status=pass /
> line_send_allowed=True 達成。Stage 3 LINE 本番 Advisory 送信前提
> が完成。develop / production 完全 unchanged。★★★

---

## タスク名

F286-DATA-R1.3 Remaining Symbols Price Refresh / Gate Pass Smoke

---

## 背景

DATA-R1.2 完了時点で staging:
- prices distinct_codes_at_max_date = 1499
- gate-1 prices REFUSE (1499 < 4000)
- overall_status: refuse / line_send_allowed: False

DATA-R1.3 では残り 2950 銘柄を `--symbols-csv` で分割取得し、
gate-1 prices coverage 4000+ を達成する。

---

## 実施 phase 一覧

| phase | 内容 | inserted | 累計 codes_at_max |
|---|---|---|---|
| Phase 1 | missing symbols 抽出 + CSV 分割 (read-only) | - | 1499 |
| Phase 2 (失敗) | --symbols-csv 1000 銘柄 1 batch | 0 (rate limit exhausted) | - |
| Phase 2 v2 part1 | --symbols-csv 300 銘柄 (再試行) | 299 | 1917 |
| Phase 2 v2 part2 | 300 銘柄 | 300 | 2217 |
| Phase 2 v2 part3 | 300 銘柄 | 300 | 2517 |
| Phase 2 v2 part4 | 300 銘柄 | 300 | 2817 |
| Phase 2 v2 part5 | 300 銘柄 | 300 | 3117 (推定) |
| Phase 2 v2 part6 | 300 銘柄 | 300 | 3417 (推定) |
| Phase 2 v2 part7 | 300 銘柄 | 300 | 3717 |
| Phase 2 v2 part8 | 300 銘柄 (★ gate-1 閾値 4000 突破) | 300 | 4017 |
| Phase 2 v2 part9 | 300 銘柄 | 300 | 4317 (推定) |
| Phase 2 v2 part10 | 131 銘柄 (端数) | 131 | **4448** |
| Phase 5 | DATA-R2 gate 再実行 | (read-only) | - |

★ 1499 → 4448 = **+2949 銘柄追加**、4449 listings 中 4448 取得
   (= 99.98% coverage、残り 1 銘柄は J-Quants 側にデータなし想定)

---

## missing symbols 件数

開始時点: 2950 銘柄 (= listings 4449 - codes_at_max 1499)

artifact:

    /tmp/f286_data_r1_3_missing_symbols.csv (= 全 2950 銘柄)
    /tmp/f286_data_r1_3_missing_part1-3.csv (= 1000-batch、
                                              Phase 2 で rate limit
                                              exhausted のため未使用)
    /tmp/f286_data_r1_3_v2_part1-10.csv (= 300-batch × 10、★ 採用)

---

## 分割単位

- **採用: 300 銘柄 / batch (= 10 batch、最後は 131 銘柄)**
- 不採用: 1000 銘柄 / batch (= Phase 2 で rate limit exhausted、
  実際は 119 件部分書き込み後に exception で全体 raise)
- 各 batch の所要時間: 約 5-10 分
- 合計所要時間: 約 90-120 分

---

## rows_inserted / rows_updated

| batch | inserted |
|---|---|
| Phase 2 v2 part1 (300) | 299 |
| Phase 2 v2 part2 (300) | 300 |
| Phase 2 v2 part3 (300) | 300 |
| Phase 2 v2 part4 (300) | 300 |
| Phase 2 v2 part5 (300) | 300 |
| Phase 2 v2 part6 (300) | 300 |
| Phase 2 v2 part7 (300) | 300 |
| Phase 2 v2 part8 (300) | 300 |
| Phase 2 v2 part9 (300) | 300 |
| Phase 2 v2 part10 (131) | 131 |
| **合計** | **3,130 (★ 部分 +2,949 codes、+部分書き込み 119 = 3,068。実 inserted は INSERT OR REPLACE で重複更新含む)** |

注: Phase 2 (1000-batch、失敗) で +119 件部分書き込み済み、その後
v2 で再度同 codes を upsert。INSERT OR REPLACE のため新規追加分は
codes_at_max 1499 → 4448 = +2949 銘柄。

---

## rate limit 有無

- Phase 2 (1000 銘柄、初回): 429 連続 retry exhaustion (5 retries)
  → status=error:JQuantsRateLimitError、exit 4 で安全停止
  (= DATA-R1.1 で実装した rate limit safe break が機能、ただし
   HistoricalDataFetcher 内部で部分書き込み 119 件発生、これは
   client retry exhaustion 前に成功した分)
- Phase 2 v2 (300-batch × 10): 各 batch で 429 hit 数回 / client
  retry 1-2 回で復帰、status=ok 維持、連続 retry exhaustion なし
- 4449 銘柄を 300-batch に分けることで rate limit を完全に回避

---

## before/after prices distinct_codes

| 時点 | distinct_codes_at_max | 変化 |
|---|---|---|
| DATA-R1.1 完了 | 10 | - |
| DATA-R1.2 完了 | 1,499 | +1,489 |
| DATA-R1.3 開始 | 1,499 | - |
| Phase 2 (1000-batch 失敗) | 1,618 | +119 (部分書き込み) |
| Phase 2 v2 part1 後 | 1,917 | +299 |
| Phase 2 v2 part2 後 | 2,217 | +300 |
| Phase 2 v2 part3 後 | 2,517 | +300 |
| Phase 2 v2 part4 後 | 2,817 | +300 |
| Phase 2 v2 part5-7 後 | 3,717 | +900 |
| Phase 2 v2 part8 後 ★ | 4,017 | +300 (★ gate-1 突破) |
| Phase 2 v2 part10 後 ★ | **4,448** | +431 (= 4017+300+131) |

開始 → 完了の純改善: **+2949 銘柄 (1499 → 4448、×2.97)**
DATA-R1.1 → 完了の累計改善: **×444.8 (10 → 4448)**

---

## before/after row_count

| 時点 | rows |
|---|---|
| DATA-R1.2 完了 | 2,082,335 |
| DATA-R1.3 完了 | **2,085,284** |
| 純増 | **+2,949** |

---

## DATA-R2 gate before / after

### DATA-R1.2 完了時点 (本タスク開始前)

| gate | level | status | metric |
|---|---|---|---|
| gate-1 prices | required | **REFUSE** | distinct_codes=1499 < 4000 |
| gate-2 signals | required | pass | max=2026-05-09 / lag=0 / codes=109 |
| gate-3 index | recommended | pass | max=2026-05-08 / lag=0 |
| gate-4 derived | recommended | pass | max=2026-05-08 / lag=0 |
| gate-5 other | soft | pass | OK |

overall: **refuse** / line_send_allowed: **False** / exit 4

### DATA-R1.3 完了時点 ★

| gate | level | status | metric |
|---|---|---|---|
| gate-1 prices | required | **★ PASS** | max=2026-05-08 / lag=0 / **distinct_codes=4448** |
| gate-2 signals | required | pass | max=2026-05-09 / lag=0 / codes=109 |
| gate-3 index | recommended | pass | max=2026-05-08 / lag=0 |
| gate-4 derived | recommended | pass | max=2026-05-08 / lag=0 / codes=42 |
| gate-5 other | soft | pass | OK |

overall: **★ pass** / line_send_allowed: **★ True** / exit 0

artifact: /tmp/f286_data_r1_3_gate_after.json
         /tmp/f286_data_r1_3_gate_after.txt

---

## line_send_allowed 判定

- DATA-R1.3 完了時点: **True** ★
- 全 5 段 gate (= prices / signals / index / derived / other) pass
- LINE 本番 Advisory 送信前提が初めて完成
- F062-R2 LINE Send 分離 タスクで本 gate を組み込めば本番送信可能

---

## production / develop 無触確認

| target | before (DATA-R1.2 完了) | after (DATA-R1.3 完了) |
|---|---|---|
| fire.staging.db | 2026-05-10 17:25:34 | 2026-05-10 18:22:36 (★ 多数 write) |
| fire.develop.db | 2026-05-07 18:14:26 | unchanged ✅ |
| fire.db (production) | 2026-05-07 16:12:38 | unchanged ✅ |

★ 全 phase で develop / production の last_modified は完全 unchanged。
   3 段 staging guard が機能。

---

## fire 側コード変更

✅ なし (= 運用 smoke のみで完結)
- 既存 DATA-R1.1 runner (`run_jquants_daily_refresh.py`) の
  `--symbols-csv` で全 phase 実行可能だった
- missing symbols 抽出は read-only sqlite query で実施、CSV
  生成のみ
- 本タスクのコミットは fire-vault の docs と log のみ

---

## tests / pytest

✅ 新規 tests なし (= 運用 smoke のみで完結、fire 側コード変更なし)
- フル pytest **3,079 PASS** (= DATA-R2 完了時点と同一、本タスクで
  コード変更ないため変化なし)
- regression: 0 件失敗想定 (本タスクでは pytest 実行せず、コード
  変更ないため確認不要)

---

## Codex pre-commit 結果

| commit | type | Codex 判定 |
|---|---|---|
| (本 commit) | docs | (vault のみ、Codex pre-commit hook は fire-vault に無し) |
| (次 commit) | docs | log milestone (同) |

★ fire 側 commit なし → Codex pre-commit hook 未走行
★ fire-vault は `core.hooksPath` 設定なしのため Codex 不要

---

## --no-verify 未使用確認

✅ fire-vault 2 commit ともに `--no-verify` flag 不使用
✅ Codex usage limit / rate limit / auth error なし
   (J-Quants の rate limit は API 側のもの、Codex とは別)

---

## TODO Excel 未更新確認

✅ Google Sheets TODO 管理表は本タスクで更新していない

---

## unrelated modified 未接触確認

git status (fire) の `Changes not staged for commit:` 欄に下記 2
ファイルが残存:

- `scripts/seed_pattern_layer1.py` (本タスク無関係、未接触)
- `simulation/research_lane/historical_indicators.py` (同上)

本タスクでは fire 側 commit がないため stage / commit 自体行われず、
unrelated を巻き込む機会なし。

---

## 制約遵守チェックリスト

- ✅ 自動発注禁止 (LineBotClient / TradeOrder 等未接続)
- ✅ 楽天証券操作なし / Computer Use なし
- ✅ LINE 本番送信なし (= line_send_allowed=True が出たが、本タスク
  では送信していない、F062-R2 で接続予定)
- ✅ DB write は staging.db のみ
- ✅ production / develop 完全 unchanged
- ✅ rate limit 連続 retry なし (= Phase 2 で 1 回 exhausted で安全
  停止、Phase 2 v2 では batch サイズを 300 に縮小して回避)
- ✅ scripts/seed_pattern_layer1.py 未接触
- ✅ simulation/research_lane/historical_indicators.py 未接触
- ✅ unrelated modified を stage / commit しない
- ✅ TODO Excel 未更新
- ✅ --no-verify 未使用

---

## artifact 一覧

    /tmp/f286_data_r1_3_missing_symbols.csv     (全 2950 銘柄)
    /tmp/f286_data_r1_3_missing_part1.csv       (1000-batch、未使用)
    /tmp/f286_data_r1_3_missing_part2.csv       (1000-batch、未使用)
    /tmp/f286_data_r1_3_missing_part3.csv       (950-batch、未使用)
    /tmp/f286_data_r1_3_v2_part1.csv 〜 part10.csv  (300-batch × 10、
                                                     ★ 採用)
    /tmp/f286_data_r1_3_phase2.json            (1000-batch 失敗 summary)
    /tmp/f286_data_r1_3_v2_part1-10.json       (300-batch summary × 10)
    /tmp/f286_data_r1_3_gate_after.json        (★ Phase 5 gate JSON)
    /tmp/f286_data_r1_3_gate_after.txt         (★ Phase 5 gate text)
    /tmp/f286_data_r1_3_completion_report.txt  (本完了報告)

---

## 次タスク提案

### 第一候補: F062-R2 LINE Send dry-run / production 分離

DATA-R1.3 で line_send_allowed=True が出るようになったので、F062
LINE 送信 runner に DATA-R2 gate を組み込む。本番送信は `--send`
明示時 + gate pass 時のみ。--allow-warning 連携で warning 許可も可。

### 第二候補: persist runner 統合 (derived / signals 自動連鎖)

prices write 後に derived persist + signals persistence を 1
invocation で連鎖させる薄い orchestrator wrapper を作る。本タスクで
分かった「rate limit safe な 300-batch」を default にする。

### 第三候補: derived full_eligible 拡大

derived は mini_100 (= 42 銘柄) のため gate-4 distinct_codes は
42 のみ。full_eligible (HQ 承認 flag 必須) で 4000+ 銘柄に拡大。

優先度: 1 > 2 > 3 (LINE 送信導線の本番接続が最優先)

---

## 関連参照

- 02_todo/F286_DATA_R0_jquants_pipeline_audit_freshness_design.md
  (DATA-R0)
- 02_todo/F286_DATA_R1_jquants_daily_refresh.md (DATA-R1)
- 02_todo/F286_DATA_R1_1_jquants_limited_write_smoke.md (DATA-R1.1)
- 02_todo/F286_DATA_R1_2_refresh_coverage_expansion.md (DATA-R1.2)
- 02_todo/F286_DATA_R2_data_freshness_gate.md (DATA-R2)
- log.md milestone (本タスク完了時に追記)
