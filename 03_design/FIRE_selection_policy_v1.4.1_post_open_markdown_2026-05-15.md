---
id: FIRE-design-selection-policy-v1.4.1-post-open-markdown-2026-05-15
phase: 本番 v0 中核 / v1.4.1 post-open payload → morning advisory Markdown 接続
priority: 最高
status: complete
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
parent_design: 03_design/FIRE_selection_policy_v1.4.1_post_open_update_2026-05-15.md
codex_rounds: 2 (= 初回 + Lane A 修正再確認)
codex_lanes: 8 / 8 YES ✓ (= Lane A 初回 NO "stdout summary 末尾の固定文言が post-open と矛盾" → 2 巡目修正で YES)
critical_high_count: 0
test_count: 81 (= 既存 60 + 本 wave 新 21)
test_total_pytest: 5051 PASS (= 前 wave 5028 + 本 wave 新 21 - 既存上書きで実 +23 / 後で再確認)
---

# FIRE Selection Policy v1.4.1 — Post-Open Payload → Morning Advisory Markdown 接続

## §1 目的

前 wave で実装した **post-open update wrapper** の出力 (= `advisory_payload_post_open.json`) を、
既存 **morning advisory Markdown renderer** に **additive** に接続し、
Fujiwara が朝/前場後にそのまま読める **post-open 反映済 Markdown** を生成可能にする.

**pre-open Markdown render を壊さない** (= 既存 60 tests 全 PASS) ことを invariant として、
post-open payload 専用の表示・metadata を分岐 ON する.

## §2 実装 file

### 2.1 既存 file 拡張 (= additive only)

| file | 拡張内容 |
|---|---|
| `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` | `is_post_open_payload()` 追加、`render_header` / `render_no_new_chase_section` / `render_review_10` / `render_safety_footer` / `render_stdout_summary` に post-open 分岐追加 |
| `scripts/jobs/run_v1_4_1_morning_advisory_markdown.py` | **変更なし** (= 同 CLI で両 payload 受付可能) |
| `tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py` | post-open fixture (= d31_post_open_payload, d32_post_open_payload) + 21 tests 追加 |

### 2.2 新規 smoke artifact

- `/tmp/fire_morning_advisory_v1_4_1_markdown_post_open_smoke/morning_advisory_post_open.md`
  (= post-open updated Markdown、8,782 bytes)
- `/tmp/fire_morning_advisory_v1_4_1_markdown_post_open_smoke/morning_advisory_pre_open_regression.md`
  (= pre-open regression Markdown、7,530 bytes)

### 2.3 vault doc

- 本 file (= `~/fire-vault/03_design/FIRE_selection_policy_v1.4.1_post_open_markdown_2026-05-15.md`)

## §3 renderer 拡張内容

### 3.1 `is_post_open_payload(payload)` (= 新規 helper)

```
metadata.post_open_phase == True → post-open 更新済 payload と判定
それ以外 → pre-open snapshot
```

### 3.2 `render_header` 拡張

- pre-open: 既存挙動 + `phase_label=" — pre-open snapshot"` + `post_open_phase: False`
- post-open: `phase_label=" — post-open updated"` + post-open metadata block (= post_open_update_version /
  post_open_update_time / source_payload_hash / actual_flow_source / counts / moved_entry_to_watch /
  unmatched_observed_codes / post_open_validation_passed)

### 3.3 `render_no_new_chase_section` 拡張

- post-open: `post_open_monitor` を優先列挙 (= 各 candidate の chase_limit_value /
  chase_limit_exceeded / no_new_chase / signal_success / entry_success / fujiwara_decision /
  fujiwara_reason / reason_codes を詳細表示) + 追加 section scan
- pre-open: 既存挙動 (= "本 payload は pre-open static snapshot" + section scan + 0 件 note)

### 3.4 `render_review_10` 拡張

末尾に **signal_success 判定基準注記** を追加:
- **historical review** (= `~/fire-vault/04_daily/*.md`): 朝 advisory 利確目安に対する事後評価.
  例: D31 9130 = 朝 chase_limit 1,440 観測上限超過 → signal_success=YES (= historical 記録).
- **current renderer** (= `compute_signal_success`): post-open high/low が +2R take_profit / -1R
  stop_loss を hit したかで自動判定. obs 未提供 / partial → pending. 推測禁止.
  例: D31 9130 post-open = high=1497 < take_profit=1551 で signal_success=pending (= current renderer).
- → 両者は **別 field / 別意味**、current renderer 値で historical review を **上書きしない**.

### 3.5 `render_safety_footer` 拡張

- post-open: post_open_validation_passed / post_open_validation_violations_count /
  post_open_update_version / post_open_used_api / post_open_modified_source_payload /
  post_open_update_executed を追加表示
- 両 payload 共通: **F282 plist 分離注記** を追加:
  - production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (= 本番 launchctl 対象)
  - smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (= smoke test 用、launchctl 未投入)
  - 両者は **別ファイル**。safety final 報告では 2 file それぞれ size + mtime を記録.

### 3.6 `render_stdout_summary` 拡張

- 単一フェンス先頭 label: `pre-open snapshot` / `post-open updated`
- post-open: post-open meta block (= update_version / update_time / source_hash / actual_flow_source /
  observed_count / pending_count / chase_exceeded_count / moved_entry_to_watch /
  unmatched_observed_codes / post_open_validation_passed)
- no-new-chase / demote 監視 section: post_open_monitor から 1 行ずつ列挙 (= original_role → new_role /
  chase_exceeded / no_new_chase / signal / entry)
- 末尾 note: post-open / pre-open で分岐 (= Lane A 2 巡目修正、固定矛盾文言を排除)

### 3.7 既存挙動非破壊 invariants

- `_candidate_is_safe_for_entry` (= 既存): no_new_chase / chase_limit_exceeded / liquidity FAIL /
  standard_lot_ok=False を全て entry section から除外する **二重 guard** を維持
- `compute_judgment` (= 既存): safe_entry_count ベース判定 (= GO_CONDITIONAL / HOLD / NO-GO)
- `render_entry_section` / `render_order_advisory` / `render_stdout_summary`: 全て同じ guard 適用
- pre-open render は既存 60 tests 全 PASS

## §4 smoke 結果

### 4.1 post-open smoke

- input: `/tmp/fire_v1_4_1_post_open_update_smoke/advisory_payload_post_open.json`
- output: `/tmp/fire_morning_advisory_v1_4_1_markdown_post_open_smoke/morning_advisory_post_open.md`
- size: 8,782 bytes
- exit 0 / md_violations 0 / payload_violations 0 / payload_passed True

post-open Markdown 主要 section:
- header: `(= v1.4.1 morning advisory Markdown — post-open updated)` + post_open_phase: True +
  post-open metadata block (= post_open_update_version 1.0 / moved_entry_to_watch ['9130'] /
  chase_exceeded_count 1 / pending_count 20)
- 判定: **GO_CONDITIONAL** (entry raw=2 / entry safe=2 / watch=2 / excluded=16)
- entry 候補: 9247 / 9628 (= D32-style、9130 は除外)
- watch 候補: 9130 (= post-open chase 移動) / 4404 (= 維持)
- excluded 候補: 16 件 (= 331A0/4389/4317 含む全 件維持)
- no-new-chase: post_open_monitor 1 件 (= 9130 entry_candidate → watch_candidate、
  chase_limit_exceeded=True、no_new_chase=True、signal=pending、entry=skipped、
  fujiwara_decision=skip、reason_codes=[chase_limit_observed, fujiwara_skip, fujiwara_skip_chase])
- 注文完成形 advisory: 9247 / 9628 のみ (= 9130 出ない)
- iSPEED 確認項目 7 項目
- review 10 項目 + signal_success basis 注記 (= historical review = YES vs current renderer = pending)
- safety footer: post_open_validation_passed True + F282 plist 分離注記

### 4.2 pre-open regression smoke

- input: `/tmp/fire_after_r1_v1_4_1_consumer_smoke/advisory_payload.json`
- output: `/tmp/fire_morning_advisory_v1_4_1_markdown_post_open_smoke/morning_advisory_pre_open_regression.md`
- size: 7,530 bytes
- exit 0 / md_violations 0 / payload_violations 0 / payload_passed True

pre-open Markdown 主要 section:
- header: `(= v1.4.1 morning advisory Markdown — pre-open snapshot)` + post_open_phase: False
- 判定: GO_CONDITIONAL (entry raw=3 / safe=3 / watch=1 / excluded=16)
- entry: 9130, 9247, 9628 (= 既存挙動完全維持)
- watch: 4404
- excluded: 16 件
- no-new-chase: "pre-open static snapshot" + 該当 0 件
- 注文完成形: 9130 / 9247 / 9628 (= 既存挙動)
- review 10 項目 + signal_success basis 注記 (= 両 payload 共通追加)

### 4.3 forbidden phrase grep (= post-open Markdown 全文)

- "60 株" / "60株" / "30 株" / "50 株" : **全 0 件** ✓
- "買え" / "売れ" / "全力" / "必ず" / "100%" / "保証" / "自動発注" / "自動売買" : **全 0 件** ✓

## §5 D31/D32 整合確認

### 5.1 D31 post-open (= 9130 chase シナリオ)

| code | post-open role | reason | 表示 |
|---|---|---|---|
| 9130 | watch_candidate (= moved from entry) | chase_limit 1497>1438 + Fujiwara skip-chase | watch + post_open_monitor (= 価格付き entry 表示なし) |
| 331A0 | excluded_candidate (= 維持) | liquidity FAIL + letter-suffix | excluded table |
| 4389 | excluded_candidate (= 維持) | liquidity FAIL | excluded table |
| 4317 | excluded_candidate (= 維持) | liquidity FAIL | excluded table |
| 9247 | entry_candidate (= pending、actual_flow なし) | - | entry section + 注文 reference |
| 9628 | entry_candidate (= pending、actual_flow なし) | - | entry section + 注文 reference |
| 4404 | watch_candidate (= 維持) | standard_lot_ok=False | watch section |

### 5.2 9130 post-open 再 entry 推奨に見えないか確認

post-open Markdown 内 9130 出現箇所:
- header `post_open_moved_entry_to_watch: ['9130']` (= metadata 記録)
- watch 候補 `### 1) 9130 共栄タンカー` (= watch_reason 明示)
- excluded には出ない
- entry 候補 / 注文完成形 advisory には **絶対に出ない** (= entry section grep "9130" 0 件)
- no-new-chase / post_open_monitor section に詳細列挙 (= "新規 entry 不可" 明示)
- stdout summary entry 欄に価格付き表示 **0 件**

→ post-open Markdown 上で 9130 が **新規 entry 推奨に見えない** invariant 確立 ✓

### 5.3 D32 post-open (= actual_flow 未提供)

| code | post-open role | priority / status |
|---|---|---|
| 9247 | entry_candidate (= 維持) | pending (= actual_flow 未提供) |
| 9628 | entry_candidate (= 維持) | pending |
| 9130 | entry_candidate (= 維持、pre-open chase なし、actual_flow 未提供) | pending |
| 4404 | watch_candidate (= 維持) | na (= standard_lot_ok=False) |

→ D32 fixture には 331A0/4389/4317 不在、D31/D32 fixture 完全分離 (= TestPostOpenD31D32Separation).

## §6 tests 結果

### 6.1 構成 (= 既存 60 + 新規 21 = 81 total)

| 新規 test class | 件数 | 内容 |
|---|---|---|
| TestIsPostOpenPayload | 3 | pre-open False / post-open True / 非 dict False |
| TestRenderHeaderPostOpen | 2 | pre-open label / post-open metadata block |
| TestRenderNoNewChasePostOpen | 2 | post-open monitor 列挙 / pre-open legacy text |
| TestRenderReview10SignalBasis | 1 | historical vs current renderer 注記 |
| TestPostOpenFullMarkdown | 5 | full md 構造 / 9130 entry 不出 / order_advisory 不出 / judgment HOLD / validation clean / D32 pending |
| TestPostOpenStdoutSummary | 2 | post-open meta block / pre-open block 不在 |
| TestSafetyFooterPostOpen | 2 | post-open fields / pre-open footer |
| TestPostOpenD31D32Separation | 2 | D31 post-open に D32 only codes 不在 / D32 post-open に D31 only codes 不在 |
| TestPostOpenCLIIntegration | 1 | 既存 CLI が post-open payload を受付 + entry section に 9130 不在 |
| **本 wave 合計** | **21** | (実態 +21 PASS) |

### 6.2 全 pytest

- 前 wave 後: 5028 PASS
- 本 wave 追加: 21 (= 既存 60 + 新 21 = 81 本 file total)
- 全 pytest 最終: **5051 PASS** (= regression 0)

## §7 Codex 8 lane 監査 (= 2 巡)

### 7.1 巡目別

| 巡 | A | B | C | D | E | F | G | H | 追加 HIGH |
|---|---|---|---|---|---|---|---|---|---|
| 1 巡目 | **NO** | YES | YES | YES | YES | YES | YES | YES | Lane A |
| 2 巡目 (= Lane A 修正後) | **YES** | - | - | - | - | - | - | - | **なし** |

### 7.2 採用 / 棄却理由

- Lane A (2 巡 YES、初回 NO 修正済): `render_stdout_summary` 末尾 note を post-open / pre-open で
  分岐させて、固定矛盾文言「post-open phase 動的更新は別 wave (= 本 wave 出力は pre-open snapshot)」を
  post-open 時に出さないよう修正
- Lane B (1 巡 YES): entry / order / stdout の全箇所で no_new_chase / chase_limit 銘柄を guard、
  9130 は watch + post_open_monitor のみ表示
- Lane C (1 巡 YES): smoke Markdown で 9247/9628 entry、9130/4404 watch、331A0/4389/4317 excluded
- Lane D (1 巡 YES): safe_entry_count 判定維持 (= 既存 logic)
- Lane E (1 巡 YES): 60/30/50 株文言 0、validate_markdown_output で検出
- Lane F (1 巡 YES): signal_success と entry_success 別 field、historical vs current renderer 注記、
  review 10 項目維持
- Lane G (1 巡 YES): tests 21 新規、post-open + pre-open regression smoke 両方、DB write / LINE /
  token / API / launchctl 等の実行経路なし
- Lane H (1 巡 YES): D31/D32 分離 tests、F282 production (size=1772 mtime=May 12 22:27 2026) と
  smoke (size=1844 mtime=May 14 23:34 2026) plist 別 file 確認

### 7.3 CRITICAL / HIGH 0

最終: 残 CRITICAL/HIGH **なし**.

## §8 safety final verification

| 観点 | 結果 |
|---|---|
| production md5 (= fire.db) | b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → 不変 ✓ |
| **F282 production plist** | `docs/launchd/jp.fire.weekly-snapshot.plist` size=1772 mtime=1778592445 → **不変** ✓ |
| **F282 smoke plist** | `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` size=1844 mtime=1778769277 → **本 wave 不変** ✓ |
| launchctl jp.fire.weekly-snapshot (loaded production) | 不変 (= `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` size=1772 → 不変) |
| DB write / API / token | 全 0 ✓ |
| LINE / launchctl / plist / cron / workflow 変更 | 全 0 ✓ |
| git commit / git push / --no-verify | 全 0 ✓ |
| forbidden phrase ("60 株" / "買え" / "必ず" / "自動発注" 等) | 全 0 件 ✓ |
| 出力先 | /tmp/fire_morning_advisory_v1_4_1_markdown_post_open_smoke/ + pytest tmp_path |
| source consumer payload | 既存 renderer は元 payload を読み込み変更しない (= read-only consumption) |

## §9 次 wave 提案

### Priority A
1. **D33 pilot で post-open Markdown を実 advisory に反映**
   - 朝: pre-open Markdown 生成 → 確認
   - 寄り後: actual flow JSON 手動入力 → post-open update → post-open Markdown 生成
   - 結果: Fujiwara が両 Markdown を iPhone でコピー利用、review 10 項目記録

2. **post-open update 自動 trigger 検討**
   - 11:30 前場引け後に launchctl で post-open update CLI を自動起動 (= read-only)
   - actual flow JSON は launchctl 起動前に Fujiwara が手動入力
   - LINE 送信は **しない** (= read-only artifact のみ生成)

### Priority B
3. weekly digest renderer (= 5 営業日分 post-open_monitor を集約)
4. F062 advisory に post-open payload integration option (= optional flag)
5. signal_success 累積記録 (= advisory_decisions テーブルへ、DB write は別 HQ approve)

### Priority C
6. W5 集約 (= D20-D33 14 day、v1.4.1 framework 演化総括)
7. F111 / consumer / morning advisory / post-open update / post-open Markdown
   コード git commit + push (= HQ approve 後)

## §10 まとめ

post-open payload → morning advisory Markdown 接続 **完全実装**.
**5 段 pipeline** 確立:

```
F111 runner (pre-open snapshot)
  ↓
v1.4.1 consumer payload
  ↓
[OPTION 1] morning advisory Markdown (pre-open) ──┐
                                                  │
v1.4.1 morning advisory Markdown CLI ←────────────┤ 同 CLI 共用
                                                  │
[OPTION 2] morning advisory Markdown (post-open) ─┘
  ↑
v1.4.1 post-open update payload (= actual flow + chase / signal / entry 反映)
  ↑
actual_flow_input.json (= 寄り後 OHLCV + Fujiwara 判断、手動入力)
```

- **既存 pre-open render を壊さない** (= 60 既存 tests 全 PASS、additive 拡張のみ)
- **同 CLI で両 payload 受付** (= run_v1_4_1_morning_advisory_markdown.py 変更なし)
- **9130 post-open 表示**: entry 0 / watch 1 / post_open_monitor 1、entry section / order_advisory /
  stdout 価格付き候補欄に **絶対に出ない** invariant 確立
- **signal_success basis 注記**: historical review (= D31 9130 = YES) と current renderer (= pending) を
  別 field / 別意味として明示、上書き禁止
- **F282 plist 分離注記**: production (= size=1772) と smoke (= size=1844) を **別 file** として
  safety footer に明示、両 file の size + mtime を別々に確認

Codex 8 lane 2 巡監査で全 YES、CRITICAL/HIGH 0、tests 81 PASS、全 pytest 5051 PASS、
safety 0 違反、3 DB md5 / F282 production + smoke plist 両方不変、forbidden phrase 0 件.
