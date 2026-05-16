---
id: FIRE-design-selection-policy-v1.4.1-consumer-implementation-2026-05-15
phase: 本番 v0 中核 / AFTER-R1 / morning advisory v1.4.1 consumer 実装
priority: 最高
status: complete
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
parent_design: 03_design/FIRE_selection_policy_v1.4.1_runner_implementation_2026-05-15.md
codex_lanes: 8 / 8 YES ✓ (= Lane H 初回 NO → 追加 fixture 3 tests + scope split 明示で YES)
critical_high_count: 0
test_count: 47 (= 44 unit/fixture + 3 post-open chase scope split)
test_total_pytest: 4898 + 3 = 4901 (確認後)
---

# FIRE Selection Policy v1.4.1 — Consumer 実装

## §1 目的

前 wave で F111 runner output に追加された `v1_4_1` namespace を、
AFTER-R1 / morning advisory 側で **正式に読み取り**、朝通知サマリ・trade plan・
review template への反映可能な consumer payload を構築する。

## §2 実装 file

### 2.1 新規

| file | 役割 |
|---|---|
| `scripts/jobs/_v1_4_1_consumer.py` | 純関数 helper (= read_namespace / candidate_role / is_*_displayable / format / validate / build_consumer_payload) |
| `scripts/jobs/run_v1_4_1_advisory_render.py` | CLI runner (= F111 json → consumer payload) |
| `tests/scripts/jobs/test_v1_4_1_consumer.py` | 47 tests (= 44 helper + 3 post-open chase scope) |

### 2.2 修正 (= なし、additive only)

F111 runner / advisory 既存 runner には **手を加えていない** (= consumer 側で
F111 output を読む形で実装、上流改変なし)。

## §3 Consumer field mapping

### 3.1 read_v1_4_1_namespace

`v1_4_1` namespace の存在を安全に判定し、ReadResult で:
- `has_namespace`: bool
- `namespace`: Optional[dict]
- `fallback_applied`: bool
- `warnings`: list[str] (= 不在時に「v1_4_1 namespace missing for code=X, fallback to existing fields」記録)

namespace 不在時:
- `candidate_role()` → `excluded_candidate` (= 安全側 default)
- `get_policy_version()` → None
- `format_candidate_summary()` → `_v1_4_1_fallback=True` 明示

### 3.2 Section 分類

| section | 条件 |
|---|---|
| **entry** | candidate_role==entry_candidate + share_unit==100 + standard_lot_ok==True + risk_within_pilot_limit==True + liquidity_status != FAIL + no_new_chase != True + chase_limit_exceeded != True |
| **watch** | candidate_role==watch_candidate (= helper の上位 priority で判定済) |
| **excluded** | candidate_role==excluded_candidate OR namespace なし (= 安全側) |

### 3.3 consumer payload 構造

```json
{
  "metadata": {
    "consumer_version": "1.4.1",
    "expected_policy_version": "1.4.1",
    "f111_policy_version": "1.4.1",
    "share_unit_standard": 100,
    "review_required_fields_version": "v1.4.1",
    "review_required_fields_count": 10,
    "base_date": "2026-06-25",
    "recently_seen_codes": [...],
    "demoted_count": 7
  },
  "sections": {
    "entry": {"role": "entry_candidate", "count": N, "candidates": [...]},
    "watch": {"role": "watch_candidate", "count": N, "candidates": [...]},
    "excluded": {"role": "excluded_candidate", "count": N, "candidates": [...]}
  },
  "warnings": [...],
  "safety_flags": {全 False},
  "validation_passed": true,
  "validation_violations": []
}
```

## §4 consumer smoke 結果

### 4.1 実行

    .venv/bin/python -m scripts.jobs.run_v1_4_1_advisory_render \
      --input /tmp/fire_f111_v1_4_1_smoke/f111_v1_4_1_d31_smoke.json \
      --output /tmp/fire_after_r1_v1_4_1_consumer_smoke/advisory_payload.json \
      --strict

### 4.2 結果

- entry=3 (= 9130, 9247, 9628)
- watch=1 (= 4404 standard_lot_ok=False)
- excluded=16 (= demote 7 + liquidity FAIL 9、331A0/4389/4317 含む)
- warnings=0
- validation_violations=0
- validation_passed=True
- "60 株" / "60株" grep = **0 件** ✓

## §5 D31/D32 整合性 + scope split (= Lane H 解消)

### 5.1 D31 review (= 9130/331A0/4389) と D32 候補 (= 9247/9628/4404) 分離

| 観点 | D31 reality | D32 候補 |
|---|---|---|
| vault file | 2026-06-25_manual_live_pilot_review.md | 2026-06-26_manual_live_pilot_review.md |
| test fixture | d31_candidates (= 9130/331A0/4389) | d32_candidates (= 9247/9628/9130/4404) |
| 9130 | entry (= signal_success+) | entry_priority NA (= post-open no-new-chase) |

### 5.2 pre-open vs post-open scope split

F111 runner-level は **pre-open static snapshot のみ** を出力:
- 9130 chase_limit_exceeded = False (= 初期値)
- 9130 no_new_chase = False (= 初期値)
- 9130 candidate_role = entry_candidate (= pre-open 静的判定)

D32 vault advisory は **post-open 動的判定後** の状態:
- 9130 = no-new-chase 扱い、entry_priority NA

この差は **scope split** で説明:
- 本 wave (= consumer 実装) では F111 runner pre-open output を読み、
  consumer が chase_limit_exceeded=True を受け取った場合に entry から除外する
  機能を保証
- post-open phase 動的更新 wrapper は **別 wave** で実装 (= F111 runner output に
  chase_limit_exceeded=True を post-open phase 後に上書きする pipeline)

### 5.3 追加 test (= Lane H 初回 NO 解消)

`TestD32PostOpenChaseExclusion` 3 tests:
- `test_9130_with_chase_limit_excluded_from_entry`: post-open chase 設定 9130 fixture で entry から除外確認
- `test_9130_post_open_chase_displays_no_new_chase_note`: format_no_new_chase_note 動作確認
- `test_pre_open_static_snapshot_default_state`: pre-open F111 raw output 直後の 9130 は chase なし初期値で entry が正しい挙動と明示

## §6 不変条件 (= Codex 8 lane 全 YES)

| Lane | 観点 | reply | 採用 |
|---|---|---|---|
| A | v1_4_1 namespace mapping + fallback + WARN | YES ✓ | 採用 |
| B | candidate_role に応じた entry/watch/excluded 表示 | YES ✓ | 採用 |
| C | 100 株標準 + 60 株文言なし + risk_yen 順位加点なし | YES ✓ | 採用 |
| D | no_new_chase / chase_limit 銘柄 entry 除外 | YES ✓ | 採用 |
| E | review v1.4.1 10 項目 + signal/entry success 分離 | YES ✓ | 採用 |
| F | unit/runner/CLI tests 十分 + D31/D32 fixture + namespace なし fallback | YES ✓ | 採用 |
| G | safety/output guard 違反なし + 出力先適切 | YES ✓ | 採用 |
| H | D31/D32 正本関係維持 + pre-open/post-open scope split 明示 | YES ✓ (= 初回 NO → fixture 追加で解消) | 採用 |

## §7 tests 結果

- test_v1_4_1_consumer.py: **47 PASS** (= 44 helper + 3 post-open chase)
- 全 pytest: **4898 PASS** (= 前 wave smoke 後、新 3 test 追加で 4901 想定)

## §8 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → 不変 ✓ |
| F282 plist | size=1772 / mtime=1778593597 → 不変 ✓ |
| DB write / API / token | 全 0 ✓ |
| LINE / launchctl / plist / cron | 全 0 ✓ |
| git commit / git push / --no-verify | 全 0 ✓ |
| forbidden import | sqlite3/subprocess/aiohttp/linebot 等 0 ✓ |
| token literal | LINE_CHANNEL_TOKEN 等 0 ✓ |
| 出力先 | /tmp/fire_after_r1_v1_4_1_consumer_smoke/ + test tmp_path のみ |

## §9 次 wave 提案

### Priority A
1. **post-open phase 動的更新 wrapper** (= chase_limit_exceeded / entry_priority を actual flow ベースで F111 runner output に back-fill する pipeline)
2. **morning advisory CLI 実装** (= consumer payload から Fujiwara 朝サマリ Markdown 生成)

### Priority B
3. F062 advisory / paper PnL preview runner で consumer payload 利用
4. D33 pilot で v1.4.1 consumer payload を実 advisory に反映、Fujiwara 確認

### Priority C
5. W5 集約 (= D20-D33 14 day、v1.4.1 runner + consumer 演化総括)
6. F111 / consumer コード git commit + push (= HQ approve 後)

## §10 まとめ

AFTER-R1 / morning advisory v1.4.1 **consumer 側実装完了**。
F111 runner output の v1_4_1 namespace を安全に読み (= fallback + WARN 含む)、
entry/watch/excluded sectioned payload を構築する read-only consumer pipeline 確立。
Codex 8 lane 全 YES (= Lane H 初回 NO → fixture 追加で解消)、CRITICAL/HIGH 0、
safety 0 違反、3 DB md5 / F282 plist 不変、47 tests PASS。
post-open phase 動的更新 wrapper は scope split として後続 wave へ送り。
