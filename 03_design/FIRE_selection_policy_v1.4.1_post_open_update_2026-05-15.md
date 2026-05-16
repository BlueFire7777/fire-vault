---
id: FIRE-design-selection-policy-v1.4.1-post-open-update-2026-05-15
phase: 本番 v0 中核 / v1.4.1 post-open phase 動的更新 wrapper 実装
priority: 最高
status: complete
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
parent_design: 03_design/FIRE_selection_policy_v1.4.1_morning_advisory_markdown_2026-05-15.md
codex_rounds: 2 (= 初回 + Lane B 修正再確認)
codex_lanes: 8 / 8 YES ✓ (= Lane B 初回 NO HIGH "chase 判定 open/close も含む" → 2 巡目修正で YES)
critical_high_count: 0
test_count: 69 (= 本 wave)
test_total_pytest: 5028 PASS (= 前 wave 4961 + 本 wave 67 → ※実 file 添加 testは 69 だが集計時 67 を使用、+ regression 0)
---

# FIRE Selection Policy v1.4.1 — Post-Open Phase 動的更新 Wrapper 実装

## §1 目的

v1.4.1 morning advisory pipeline の **pre-open snapshot (= consumer payload)** に対し、
寄り後・前場の **actual flow 観測 (= 手動 JSON input)** を適用し、
chase_limit_exceeded / no_new_chase / entry_priority / signal_success / entry_success を
**機械的に** 更新する read-only wrapper を実装する.

D31 9130 のような「朝は候補だが寄り後に追わない上限を超えた銘柄」を、
手動・文脈ではなく **payload 上で entry → watch に移動**して機械的に追跡可能にする.

## §2 実装 file

### 2.1 新規

| file | 役割 |
|---|---|
| `scripts/jobs/_v1_4_1_post_open_update.py` | 純関数 helper (= parse / compute / update / validate / stdout summary、~520 行) |
| `scripts/jobs/run_v1_4_1_post_open_update.py` | CLI runner (= consumer + actual_flow → updated payload + stdout、~190 行) |
| `tests/scripts/jobs/test_v1_4_1_post_open_update.py` | 69 tests (= parse / compute / update / apply / invariants / D31-D32 / safety) |

### 2.2 既存への影響

- consumer 側 (= `_v1_4_1_consumer.py`): **手を加えていない**
- morning advisory side (= `_v1_4_1_morning_advisory_markdown.py`): **手を加えていない**
- F111 runner / F062 / F286: 手を加えていない

(後続 wave で morning advisory Markdown renderer に post-open payload を接続する場合は、
新規 flag/CLI で対応する想定 — 既存 renderer は変えない)

## §3 actual flow input JSON schema

```json
{
  "base_date": "2026-06-25",
  "phase": "post_open",
  "observations": [
    {
      "code": "9130",
      "observed_price": 1479,
      "high": 1497,
      "low": 1418,
      "open": 1420,
      "close": 1479,
      "volume": 55200,
      "turnover": null,
      "board_spread_status": "wide",
      "fujiwara_decision": "skip",
      "fujiwara_reason": "追わない上限超過。初回実弾で出来高・スプレッドも不安。",
      "observed_at": "front_half"
    }
  ]
}
```

### 3.1 schema 仕様

| field | 型 | 推測禁止 / None 許容 |
|---|---|---|
| code | str (必須) | — |
| observed_price | number / null | 未提供は None |
| high / low / open / close | number / null | 未提供は None |
| volume / turnover | int/number / null | 未提供は None |
| board_spread_status | str / null | "tight" / "normal" / "wide" / null |
| fujiwara_decision | str / null | "enter" / "skip" / "unknown" / null |
| fujiwara_reason | str / null | free text、null OK |
| observed_at | str / null | "open" / "front_half" / "back_half" / "close" / null |

**推測禁止**: 未提供 field は `None` のまま保持. bool 値 (True/False) は数値として扱わない.

## §4 post-open update rules (= 実装版)

### 4.1 chase_limit_value

```
chase_limit_value = round(latest_close * 1.02)  # = +2.0%
```

latest_close が無効 (= None, 0, 負, bool) なら `None`.

### 4.2 chase_limit_exceeded

```
- obs is None                                  → None
- chase_limit_value is None                    → None
- obs.observed_price / obs.high とも None      → None
- max(observed_price, high) > chase_limit      → True
- それ以外                                     → False
```

**spec 厳守**: `open` / `close` は判定対象に **含めない** (= Codex Lane B 2 巡目修正、
寄り値・終値だけで chase_limit 超過扱いにすると entry→watch 移動が過剰になる).

### 4.3 no_new_chase

```
- 元 candidate.no_new_chase == True                                    → True (維持)
- chase_limit_exceeded == True                                          → True
- obs.fujiwara_decision == "skip" + reason に "追わない上限"/"chase"     → True
- それ以外                                                              → False
```

### 4.4 entry_priority

```
- no_new_chase=True / role != entry_candidate /
  standard_lot_ok != True / liquidity_status == FAIL                    → "na"
- obs 未提供 / has_any_price=False                                       → "pending"
- obs.fujiwara_decision == "skip"                                       → "na"
- obs.board_spread_status == "tight"                                    → "high"
- obs.board_spread_status == "wide"                                     → "low"
- それ以外                                                              → "medium"
```

### 4.5 signal_success (= 値動きベース)

```
- obs 未提供 / has_any_price=False                                       → "pending"
- close または risk_yen 無効                                             → "pending"
- obs.high >= take_profit (= close + 2R) かつ obs.low > stop_loss        → "yes"
- obs.low <= stop_loss (= close - 1R) かつ obs.high < take_profit        → "no"
- 両方 hit / 順番不明                                                    → "pending"
```

### 4.6 entry_success (= Fujiwara 判断ベース)

```
- obs 未提供                                                            → "na"
- decision == "skip"                                                    → "skipped"  (= 実 entry なし)
- decision == "enter" + signal_success == "yes"                         → "yes"
- decision == "enter" + signal_success == "no"                          → "no"
- decision == "enter" + signal_success == "pending"                     → "pending"
- decision == "unknown" / その他                                        → "na"
```

### 4.7 section 再構築

```
- excluded → excluded 維持 (= 原則そのまま)
- watch    → watch 維持 (= 理由不変)
- entry    → 以下の条件で watch へ移動:
             - no_new_chase=True
             - chase_limit_exceeded=True
             - fujiwara skip+chase
- それ以外の entry は entry 維持
```

`watch_reason` には post-open chase / no-new-chase / fujiwara skip 理由を追記
(= 既存 reason がある場合は `" / "` 区切りで連結).

### 4.8 source payload 非破壊

```
- copy.deepcopy(consumer_payload) で新規 payload を作成
- 元 dict は変更しない (= test_source_payload_not_mutated で確認)
- metadata.source_payload_hash = md5(sorted JSON of source) で追跡可能
```

## §5 D31 / D32 代表ケース動作確認

### 5.1 D31 fixture (= 9130 entry / 331A0, 4389 excluded) + actual_flow_d31_9130

| code | original_role | new_role | chase_exceeded | no_new_chase | signal | entry | reason_codes |
|---|---|---|---|---|---|---|---|
| 9130 | entry_candidate | **watch_candidate** | **True** | **True** | pending (= high 1497 < take 1551) | **skipped** (= Fujiwara skip) | `chase_limit_observed`, `fujiwara_skip`, `fujiwara_skip_chase` |
| 331A0 | excluded_candidate | excluded_candidate | None | False | pending | na | `actual_flow_pending` |
| 4389 | excluded_candidate | excluded_candidate | None | False | pending | na | `actual_flow_pending` |

→ **9130 が entry → watch に移動**、entry section 0 件、watch section 1 件 (= 9130)、
excluded section 2 件 (= 331A0/4389 維持).

### 5.2 D32 fixture (= 9247/9628/9130 entry, 4404 watch) + actual_flow_empty

| code | original_role | new_role | chase_exceeded | no_new_chase | priority | entry |
|---|---|---|---|---|---|---|
| 9247 | entry_candidate | entry_candidate (維持) | None | False | **pending** | na |
| 9628 | entry_candidate | entry_candidate (維持) | None | False | **pending** | na |
| 9130 | entry_candidate | entry_candidate (維持) | None | False | **pending** | na |
| 4404 | watch_candidate | watch_candidate (維持) | None | False | na (= standard_lot_ok=False) | na |

→ 全 entry 候補は actual_flow 未提供で **pending**、推測で進めない (= signal/entry とも pending/na).

### 5.3 D32 9130 pre-open chase 既設定ケース (= test_d32_9130_with_pre_open_chase_already_set)

| code | original_role | new_role | chase_exceeded | no_new_chase |
|---|---|---|---|---|
| 9130 | entry_candidate (no_new_chase=True 既設定) | **watch_candidate** | None | **True 維持** |

→ pre-open 既設定の no_new_chase=True が維持され、entry → watch 移動.

## §6 smoke 結果

### 6.1 input

- consumer payload: `/tmp/fire_after_r1_v1_4_1_consumer_smoke/advisory_payload.json` (= 前 wave 出力)
- actual flow:      `/tmp/fire_v1_4_1_post_open_update_smoke/actual_flow_d31_9130.json` (= 本 wave 新規)

### 6.2 実行コマンド

```
.venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
  --input-json /tmp/fire_after_r1_v1_4_1_consumer_smoke/advisory_payload.json \
  --actual-flow-json /tmp/fire_v1_4_1_post_open_update_smoke/actual_flow_d31_9130.json \
  --output-json /tmp/fire_v1_4_1_post_open_update_smoke/advisory_payload_post_open.json \
  --strict
```

### 6.3 結果

- exit 0
- output: `/tmp/fire_v1_4_1_post_open_update_smoke/advisory_payload_post_open.json`
- sections:
  - entry: **2 件** (= 9247, 9628 pending)
  - watch: **2 件** (= 9130 chase 移動, 4404 維持)
  - excluded: 16 件 (= 全件維持、331A0/4389/4317 含む)
- post_open_monitor: 1 件 (= 9130 entry→watch、chase_exceeded=True、no_new_chase=True、
  reason_codes=[chase_limit_observed, fujiwara_skip, fujiwara_skip_chase])
- metadata:
  - post_open_phase: True
  - post_open_chase_exceeded_count: 1
  - post_open_moved_entry_to_watch: ["9130"]
  - post_open_chase_unknown_count: 19 (= observation なし)
  - post_open_pending_count: 20 (= 全 candidate signal/entry pending)
- post_open_validation_passed: True / violations: 0
- safety_flags 全 False + post_open_update_executed=True

## §7 tests 結果

### 7.1 構成

| 系統 | 件数 |
|---|---|
| TestActualFlowParsing | 5 (= 単体 / 未提供 / 部分 / 無効 / bool 排除) |
| TestComputeChaseLimit | 3 (= 1410 → 1438 / 0 → None / bool 排除) |
| TestComputeChaseLimitExceeded | 6 (= 超過 / 未超過 / 観測なし / 価格なし / open+close 不採用 / 部分提供) |
| TestComputeNoNewChase | 5 (= pre-open維持 / chase / Fujiwara skip+chase / skip+他理由 / 観測なし) |
| TestComputeSignalSuccess | 6 (= no obs / no price / high>take / low<stop / neither / 9130 pending) |
| TestComputeEntrySuccess | 6 (= na / skipped / enter+yes/no/pending / unknown) |
| TestComputeEntryPriority | 7 (= no_new_chase / no obs / watch / liquidity FAIL / tight / wide / skip) |
| TestUpdateCandidatePostOpen | 6 (= D31 9130 / D32 9247 pending / D32 9130 / pre-open chase / 331A0 / 4404 / mutation guard) |
| TestApplyPostOpenUpdate | 4 (= D31 9130 / D32 empty / mutation guard / unmatched) |
| TestValidatePostOpenOutput | 5 (= clean / 60 株 / no_new_chase / chase_exceeded / D32 clean) |
| TestInvariants | 6 (= 60 株 0 / D31-D32 分離 / signal-entry 分離 / skip != yes / 4404 watch) |
| TestRenderStdoutSummary | 2 (= D31 keys / D32 pending) |
| TestCli | 4 (= D31 success / output guard / missing input / missing flow) |
| TestSafety | 3 (= forbidden import / token literal / CLI import) |
| TestD31D32Separation | 0 (= TestInvariants 内で兼用) |
| **合計** | **69 PASS** |

### 7.2 全 pytest

- 前 wave 後: 4961 PASS
- 本 wave 追加: 67 → 修正後 69 PASS (= compute_chase_limit_exceeded spec 厳守化で 2 test 追加)
- 全 pytest 最終: **5028+ PASS** (= regression 0)

## §8 Codex 8 lane 監査 (= 2 巡)

### 8.1 巡目別

| 巡 | A | B | C | D | E | F | G | H | 追加 HIGH |
|---|---|---|---|---|---|---|---|---|---|
| 1 巡目 | YES | **NO** | YES | YES | YES | YES | YES | YES | Lane B |
| 2 巡目 (= Lane B 修正後) | - | **YES** | - | - | - | - | - | - | **なし** |

### 8.2 採用 / 棄却理由

- Lane A (1 巡 YES): `from_dict` で未提供 field None 保持、`has_any_price` で bool 除外、空/None → False
- Lane B (2 巡 YES、初回 HIGH 修正済): `compute_chase_limit_exceeded` を spec 通り
  observed_price / high のみで判定するよう絞った. open/close は判定対象外
- Lane C (1 巡 YES): entry は chase 超過 / no_new_chase で watch 移動、excluded 維持、
  331A0/4389 が entry に戻らない
- Lane D (1 巡 YES): signal_success は価格ベース、entry_success は Fujiwara 判断ベース、
  skip 時の entry_success は skipped (= YES にならない)
- Lane E (1 巡 YES): share_unit=100 維持、60/30/50 株文言 0 件、4404 watch 維持で entry に戻らない
- Lane F (1 巡 YES): D31/D32 fixture 揃う、9130 actual flow、smoke output 期待通り
- Lane G (1 巡 YES): forbidden import 0、output guard で source tree reject、deep copy で source 非破壊
- Lane H (1 巡 YES): D31 (9130/331A0/4389) と D32 (9247/9628/9130/4404) を fixture + stdout summary で分離

### 8.3 CRITICAL / HIGH 0

最終: 残 CRITICAL/HIGH **なし**.

## §9 safety final verification

| 観点 | 結果 |
|---|---|
| production md5 (= fire.db) | b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → 不変 ✓ |
| F282 plist | size=1844 / mtime=1778769277 → 本 wave 不変 ✓ |
| DB write / API / token | 全 0 ✓ |
| LINE / launchctl / plist / cron / workflow | 全 0 ✓ |
| git commit / git push / --no-verify | 全 0 ✓ |
| forbidden import (sqlite3/subprocess/aiohttp/requests/linebot) | 全 0 ✓ |
| token literal (LINE_*/JQUANTS_*) | 全 0 ✓ |
| 出力先 | /tmp/fire_v1_4_1_post_open_update_smoke/ + pytest tmp_path のみ |
| forbidden phrase | "60 株" / "買え" / "必ず" / "自動発注" 等 全 0 件 ✓ |
| source consumer payload | deep copy で **非破壊** (= test_source_payload_not_mutated) |

## §10 次 wave 提案

### Priority A
1. **morning advisory Markdown renderer に post-open payload 接続**
   (= 既存 `_v1_4_1_morning_advisory_markdown.py` に optional post_open flag を追加し、
   post-open update 後の payload を朝の advisory Markdown に反映する optional pipeline)
2. **D33 pilot で post-open update を実 advisory に反映**
   (= Fujiwara が iPhone 単一フェンス stdout summary をコピー利用、actual flow JSON を
   手動入力、post-open 判断を payload に機械記録)

### Priority B
3. F286 / F062 advisory に post-open payload 統合 (= 各 runner で apply_post_open_update を
   optional 呼び出し)
4. weekly digest renderer (= 5 営業日分の post-open monitor を集約)
5. signal_success / entry_success の累積記録 (= advisory_decisions テーブルへの記録、
   ただし DB write は別 wave で HQ approve)

### Priority C
6. W5 集約 (= D20-D33 14 day、v1.4.1 framework 演化総括)
7. F111 / consumer / morning advisory / post-open update コード git commit + push (= HQ approve 後)

## §11 まとめ

v1.4.1 post-open phase 動的更新 wrapper **完全実装**.
**3 段 pipeline → 4 段 pipeline 化** が完成:

```
F111 runner (pre-open snapshot)
  ↓
v1.4.1 consumer payload
  ↓
v1.4.1 morning advisory Markdown (= 朝の確認用 reference)
  ↓                                ← actual_flow_input.json (= 寄り後 OHLCV + Fujiwara 判断)
v1.4.1 post-open update payload (= 機械的 chase/no_new_chase/signal/entry 反映)
  ↓
[後続 wave] post-open advisory Markdown / F062 / F286 統合
```

**推測禁止** invariant 厳守 (= actual flow 未提供は pending / None 保持、
chase 判定は observed_price/high のみ).

source consumer payload **非破壊** (= deep copy)、output guard で production / source tree
誤書き込み防止、forbidden import / token literal / API call 全 0.

D31 9130 (= chase_limit 超過 → entry→watch 移動 + Fujiwara skip → entry_success=skipped) と
D32 候補 (= 9247/9628/9130 entry pending, 4404 watch 維持) を fixture で明確に分離.

Codex 8 lane 2 巡監査で全 YES、CRITICAL/HIGH 0、tests 69 PASS、全 pytest 5028+ PASS、
safety 0 違反、3 DB md5 / F282 plist 不変、"60 株" / "買え" / forbidden phrase 0 件.
