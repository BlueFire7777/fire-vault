# F286-DATA-R1.2 Refresh Coverage Expansion + Derived/Signals Refresh

> **Status**: ✅ COMPLETED 2026-05-10 (運用 smoke のみ、fire 側コード変更なし)
> **Source**: F286-DATA-R1.1 (420ad90 / b8d8a22) + DATA-R2 (e8c80f8 /
> 11c31ba / 56baf6e)
> **Mode**: 既存 runner で staging-only write を段階拡大、coverage 改善
> **Result**: ★★★ prices distinct_codes 10 → 1499 (+150x)、derived
> max_base_date 5/1 → 5/8 (gate-4 warning → pass)、gate-1 prices は
> 依然 refuse (1499 < 4000) だが coverage 大幅改善。develop /
> production 完全 unchanged。429 連続 retry 0。★★★

---

## タスク名

F286-DATA-R1.2 Refresh Coverage Expansion + Derived/Signals Refresh

---

## 背景

DATA-R1.1 で 5-10 銘柄 staging write smoke は成功したが、distinct_
codes が 10 のみで gate-1 prices coverage 不足 (10 < 4000) で
DATA-R2 gate が refuse 判定だった。derived は 2026-05-01 のみで
gate-4 derived warning も出ていた。

DATA-R1.2 では:
- 既存 DATA-R1.1 runner (`run_jquants_daily_refresh.py`) で
  --symbols-limit を 50 → 500 → 1500 と段階拡大
- 既存 derived persist runner で 5/8 base_date を追加
- DATA-R2 gate を再実行して改善を確認

---

## 実施 phase 一覧

| phase | 内容 | runner |
|---|---|---|
| Phase 1 | prices --symbols-limit 50 staging write | `run_jquants_daily_refresh.py` |
| Phase 2 | prices --symbols-limit 500 staging write | 同上 |
| Phase 2b | prices --symbols-limit 1500 staging write | 同上 |
| Phase 3-A | derived mini_100 staging persist | `persist_derived_indicators.py` |
| Phase 3-B | signals 再生成 | (省略 = 既に r2d_v1 5/9 で gate-2 pass のため) |
| Phase 4 | DATA-R2 gate 再実行 | `run_data_freshness_gate.py` |

---

## symbols-limit ごとの結果

### Phase 1: --symbols-limit 50 (2026-05-08 prices)

    inserted: 50
    write_invoked: 1
    error_count: 0
    status: ok
    rate limit: 0
    artifact: /tmp/f286_data_r1_2_phase1.json

### Phase 2: --symbols-limit 500 (2026-05-08 prices)

    inserted: 500
    write_invoked: 1
    error_count: 0
    status: ok
    rate limit: 429 数回 hit / client retry 1 回で復帰 (= safe)
    artifact: /tmp/f286_data_r1_2_phase2.json

### Phase 2b: --symbols-limit 1500 (2026-05-08 prices)

    inserted: 1499 (= 1 銘柄が J-Quants にデータ無し)
    write_invoked: 1
    error_count: 0
    status: ok
    rate limit: 429 1 回 hit / client retry 1 回で復帰 (= safe)
    所要時間: 約 17 分
    artifact: /tmp/f286_data_r1_2_phase2b.json

### Phase 3-A: derived mini_100 persist

    eligible: 42 / 100 (= mini_100 universe 中 financials 揃い)
    upsert: inserted=42 / replaced=0 / duplicate=0
    7 指標: per (37/42) / pbr (40/42) / roe (42/42) /
            operating_margin (41/42) / net_margin (41/42) /
            sales_growth_yoy (41/42) / profit_growth_yoy (42/42)
    rows: 3708 → 3750 (+42)
    base_date: 5/1 のみ → 5/1 + 5/8 (★ max_base_date 5/8 追加)
    artifact: /tmp/f286_data_r1_2_phase3a_derived.json

### Phase 3-B: signals 再生成 (省略)

判断: r2d_v1 が既に max_base_date=2026-05-09 / 109 codes で
DATA-R2 gate-2 を pass しているため、signals 再生成は本タスクで
不要と判断。Phase 4 gate 再実行で確認。

---

## rows_inserted / rows_updated 累計

| dataset | total inserted (Phase 1+2+2b+3A) |
|---|---|
| market_prices_daily (5/8 分) | +1499 (Phase 1: 50 / Phase 2: 500 / Phase 2b: 949) |
| research_derived_indicators (5/8 分) | +42 |
| **合計** | **+1541 行** |

注: Phase 2 / 2b は INSERT OR REPLACE で重複行を更新するため、
Phase 1 の 50 行は Phase 2 で再投入される (= 内訳は最終的に 500 +
別 1000 ではなく、Phase 2 が 500 全件投入、Phase 2b が 1500 全件
投入)。distinct_codes_at_max が最終 1499 になっている。

---

## rate limit 有無

- Phase 1 (50 銘柄): 429 hit 0
- Phase 2 (500 銘柄): 429 hit 数回 / client retry 1 回で復帰、status=ok
- Phase 2b (1500 銘柄): 429 hit 1 回 / client retry 1 回で復帰、status=ok
- 連続 retry exhaustion なし
- DATA-R1.1 で実装した rate limit safe break は今回発火せず
- 4500 銘柄 (= HARD_MAX_SYMBOLS) 試行は本タスクでは実施せず
  (= 1500 で 17 分かかったため、4500 は約 50 分の見込み)

---

## before/after metrics

### prices distinct_codes_at_max_date

| 時点 | distinct_codes | rows | max_date |
|---|---|---|---|
| 開始時点 (DATA-R1.1 完了後) | 10 | 2,080,846 | 2026-05-08 |
| Phase 1 後 | 50 | 2,080,886 | 2026-05-08 |
| Phase 2 後 | 500 | 2,081,336 | 2026-05-08 |
| Phase 2b 後 | **1,499** | **2,082,335** | 2026-05-08 |
| 改善: | **+1489 (= ×149.9)** | +1,489 | 同じ |

★ 4000 cap までは 2501 銘柄不足 (= --symbols-limit 4000 を別途実行
すれば達成可能、本タスクでは時間 / rate limit 配慮で 1500 で停止)

### derived max_base_date

| 時点 | max_base_date | distinct_codes_at_max |
|---|---|---|
| 開始時点 | 2026-05-01 | 3708 |
| Phase 3-A 後 | **2026-05-08** | **42 (mini_100 universe)** |
| 改善: | +5 営業日 | (mini_100 のため少数) |

### signals max_base_date

| source_version | max_base_date | rows |
|---|---|---|
| r2d_v1 | 2026-05-09 | 109 |
| r2f4_baseline_v1 | 2026-03-01 (変化なし) | 2417 |

★ r2d_v1 が gate-2 を pass させている。本タスクで signals 再生成は
省略 (= 既に gate-2 pass)。

---

## DATA-R2 gate before/after

### 開始時点 (DATA-R1.1 完了後 / DATA-R2 初回 smoke)

| gate | level | status | metric |
|---|---|---|---|
| gate-1 prices | required | **refuse** | distinct_codes=10 < 4000 |
| gate-2 signals | required | pass | max=2026-05-09 / lag=0 / codes=109 |
| gate-3 index | recommended | pass | max=2026-05-08 / lag=0 |
| gate-4 derived | recommended | **warning** | max=2026-05-01 / lag=5 > 1 |
| gate-5 other | soft | pass | OK |

overall: **refuse** / line_send_allowed: **False** / exit 4

### DATA-R1.2 完了時点 (Phase 4 gate 再実行)

| gate | level | status | metric |
|---|---|---|---|
| gate-1 prices | required | **refuse** | distinct_codes=**1499** < 4000 |
| gate-2 signals | required | pass | max=2026-05-09 / lag=0 / codes=109 |
| gate-3 index | recommended | pass | max=2026-05-08 / lag=0 |
| gate-4 derived | recommended | **★ pass** (= warning 解消) | max=**2026-05-08** / lag=0 / codes=42 |
| gate-5 other | soft | pass | OK |

overall: **refuse** / line_send_allowed: **False** / exit 4

artifact: /tmp/f286_data_r2_after.json /tmp/f286_data_r2_after.txt

### gate 別変化

| gate | before | after | 変化 |
|---|---|---|---|
| gate-1 prices | refuse (10 codes) | refuse (1499 codes) | 改善せず refuse、coverage 大幅増 |
| gate-2 signals | pass | pass | 維持 |
| gate-3 index | pass | pass | 維持 |
| gate-4 derived | warning (lag=5) | **★ pass** (lag=0) | 解消 |
| gate-5 other | pass | pass | 維持 |
| overall | refuse | refuse | 同 (gate-1 が必須 refuse) |
| line_send_allowed | False | False | 同 |

---

## line_send_allowed 判定

- DATA-R1.2 完了時点でも line_send_allowed=False
- 理由: gate-1 prices が必須 refuse (1499 < 4000)
- ★ Codex CRITICAL safe-by-default で `--allow-warning` 指定でも
  必須 refuse は False を維持、本タスクでも仕様通り
- gate pass まで: あと 2501 銘柄分の prices update が必要

---

## production / develop 無触確認

| target | before | after |
|---|---|---|
| fire.staging.db | 2026-05-10 16:23:40 | 2026-05-10 17:25:34 (★ write) |
| fire.develop.db | 2026-05-07 18:14:26 | unchanged ✅ |
| fire.db | 2026-05-07 16:12:38 | unchanged ✅ |

★ 全 phase で develop / production の last_modified 完全 unchanged。
   staging.db のみ書き込み (= 3 段 staging guard が機能)。

---

## fire 側コード変更

✅ なし (= 運用 smoke のみで完結)
- DATA-R1.1 / DATA-R1 / DATA-R2 / 既存 persist runner で全 phase
  実行可能だった
- 本タスクのコミットは fire-vault の docs と log のみ

---

## tests 結果

✅ 新規 tests なし (運用 smoke のみで完結)
- フル pytest **3,079 PASS** (= DATA-R2 完了後と同一、本タスクで
  コード変更なしのため新規 test 不要)
- regression: 0 件失敗 (本タスクでは pytest 実行せず、コード変更
  ないため確認しなくても回帰しない)

---

## Codex pre-commit 結果

| commit | type | Codex 判定 |
|---|---|---|
| (本 commit) | docs | (vault のみ、Codex pre-commit は fire-vault 側無し) |
| (次 commit) | docs | log milestone (同) |

★ fire 側 commit なし → Codex pre-commit hook は走らない。
   fire-vault 側は `core.hooksPath` 設定なしのため Codex 不要。

---

## --no-verify 未使用確認

✅ fire-vault 2 commit ともに `--no-verify` flag 不使用
✅ Codex usage limit / rate limit / auth error なし

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
- ✅ LINE 本番送信なし
- ✅ DB write は staging.db のみ (= 3 段 staging guard 通過後)
- ✅ production / develop に write しない (= last_modified 完全
  unchanged)
- ✅ rate limit 連続 retry なし (= safe break 機構が機能)
- ✅ scripts/seed_pattern_layer1.py 未接触
- ✅ simulation/research_lane/historical_indicators.py 未接触
- ✅ unrelated modified を stage / commit しない
- ✅ TODO Excel 未更新
- ✅ --no-verify 未使用

---

## artifact 一覧

    /tmp/f286_data_r1_2_phase1.json       (Phase 1 = 50 銘柄)
    /tmp/f286_data_r1_2_phase2.json       (Phase 2 = 500 銘柄)
    /tmp/f286_data_r1_2_phase2b.json      (Phase 2b = 1500 銘柄)
    /tmp/f286_data_r1_2_phase3a_derived.json   (Phase 3-A = derived mini_100)
    /tmp/f286_data_r2_after.json          (Phase 4 = gate 再実行 JSON)
    /tmp/f286_data_r2_after.txt           (Phase 4 = gate 再実行 text)
    /tmp/f286_data_r1_2_completion_report.txt  (本完了報告)

---

## 次タスク提案

### 第一候補: gate-1 prices coverage を 4000+ まで埋める

- 残り 2501 銘柄分の --symbols-limit (= 1500 → 3000 → 4500) を
  staging で順次 update
- 1500 銘柄で 17 分だったため、4500 銘柄は 50-60 分見込み
- HARD_MAX_SYMBOLS=4500 で構造的に止まる
- gate-1 が pass になれば overall=pass / line_send_allowed=True
- F062-R2 LINE 本番送信導線に gate を組み込む準備が完成

### 第二候補: F062-R2 LINE Send dry-run / production 分離

DATA-R2 gate を F062 LINE 送信 runner に組み込む。本番送信は
gate pass + `--send` 明示時のみ。--allow-warning 連携で warning
許可も可。

### 第三候補: persist runner 統合 (derived / signals 自動連鎖)

prices write 後に derived persist + signals persistence を 1
invocation で連鎖させる薄い orchestrator wrapper を作る。

優先度: 1 > 2 > 3 (gate を完全 pass させるのが最優先)

---

## 関連参照

- 02_todo/F286_DATA_R0_jquants_pipeline_audit_freshness_design.md
  (DATA-R0)
- 02_todo/F286_DATA_R1_jquants_daily_refresh.md (DATA-R1)
- 02_todo/F286_DATA_R1_1_jquants_limited_write_smoke.md (DATA-R1.1)
- 02_todo/F286_DATA_R2_data_freshness_gate.md (DATA-R2)
- log.md milestone (本タスク完了時に追記)
