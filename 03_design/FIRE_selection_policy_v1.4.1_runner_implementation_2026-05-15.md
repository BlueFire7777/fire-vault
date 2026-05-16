---
id: FIRE-design-selection-policy-v1.4.1-runner-implementation-2026-05-15
phase: 本番 v0 中核 / selection policy v1.4.1 runner-level 正式実装
priority: 最高
status: design-final + impl-complete
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
parent_design: 03_design/FIRE_selection_policy_v1.4.1_2026-05-15.md
implementation_scope: F111 runner output に v1.4.1 fields を additive 追加
codex_lanes: 8 / 8 YES ✓ (= A/B/C/D/E/F/G/H 全 PASS)
critical_high_count: 0
test_count: 42 (= 新 helper module tests) + 既存維持
test_total_pytest: 4854 (= 4812 既存 + 42 新)
---

# FIRE Selection Policy v1.4.1 — Runner-Level 正式実装

## §1 目的

D31/D32 実弾 pilot で固まった selection policy v1.4.1 を、
**advisory/design layer ではなく F111 runner output に正式実装**。
これにより、毎朝の F111 候補生成時点で以下が runner 出力に含まれる:

- risk_yen 非加点 (= 順位形成に使わない)
- 100 株標準 (= 非 100 株調整不採用)
- entry / watch / excluded 三分類 (= candidate_role)
- pre_open_score / entry_priority 分離
- signal_success / entry_success 分離 (= review 接続 placeholder)
- chase_limit_exceeded / no_new_chase (= post-open 動的)
- review 10 項目接続 (= review_required_fields_version="v1.4.1")
- 低流動性・小型・非 TOPIX・letter-suffix 新規銘柄除外
- 連続上位の demote 準備 (= recently_seen_codes 連携)

## §2 実装 file

### 2.1 新規

| file | 役割 | 行数 |
|---|---|---|
| `scripts/jobs/_selection_policy_v1_4_1.py` | 純関数 helper module (= classify / compute / apply) | ~340 行 |
| `tests/scripts/jobs/test_selection_policy_v1_4_1.py` | unit + D31/D32 fixture tests | ~330 行、42 tests |

### 2.2 修正 (= 最小 additive)

| file | 修正内容 |
|---|---|
| `scripts/jobs/run_f111_real_batch_staging.py` | import + apply_v1_4_1_policy 呼出 (= 各 candidate に `v1_4_1` namespace 追加) + tuning_params に policy_version / share_unit_standard 追加 |

## §3 Field Contract (= V14_1FieldContract dataclass)

F111 candidate に追加される 21 fields:

| field | 型 | 役割 |
|---|---|---|
| policy_version | str | "1.4.1" 固定 |
| share_unit | int | 100 (= 標準) |
| standard_lot_ok | bool | 100 株 entry が pilot 限度内か |
| risk_yen_for_100_shares | Optional[int] | 100 株 risk_yen |
| liquidity_status | str | PASS / WARN / FAIL |
| liquidity_tier | int | 0-10 score |
| cap_tier | int | 0-10 score |
| is_letter_suffix_new_listing | bool | XXXAY 形式 (= 2025+ IPO) 判定 |
| pre_open_score | float | 静的 proxy score (= risk_yen 非加点) |
| candidate_role | str | entry_candidate / watch_candidate / excluded_candidate |
| candidate_role_reason | str | 役割判定理由 |
| chase_limit_exceeded | bool | post-open 動的、初期 False |
| no_new_chase | bool | post-open 動的、初期 False |
| entry_priority | Optional[int] | post-open 動的、初期 None |
| signal_success | Optional[str] | Y/N/NA、review 接続用、初期 None |
| entry_success | Optional[str] | Y/N/NA、review 接続用、初期 None |
| review_required_fields_version | str | "v1.4.1" |
| review_required_fields_count | int | 10 |
| reason_codes | list[str] | liquidity_fail / share_unit_pilot_exceeded / recently_seen_demoted / letter_suffix_new_listing 等 |
| exclusion_reason | Optional[str] | excluded の場合の詳細理由 |
| watch_reason | Optional[str] | watch の場合の詳細理由 |

## §4 helper module 関数群

| function | 役割 |
|---|---|
| `is_letter_suffix_code(code)` | 331A0/137A0 形式判定 |
| `classify_liquidity_status(margin, scale, code)` | PASS/WARN/FAIL 3 段階 |
| `compute_liquidity_tier(margin, scale)` | 0-10 tier (= pre_open_score 用) |
| `compute_cap_tier(scale)` | 0-10 tier |
| `compute_pre_open_score(liquidity_tier, cap_tier, ..., research_score, ...)` | v1.4.1 weighting で score 算出、**risk_yen 引数なし** |
| `check_share_unit_pilot_fit(estimated_100_share_risk, pilot_max_risk_yen=8500)` | 100 株 entry 適性判定、非 100 株調整不採用 |
| `classify_candidate_role(liquidity_status, standard_lot_ok, is_recently_seen_demoted, chase_limit_exceeded)` | 3 分類 + 理由 |
| `apply_v1_4_1_policy(candidate, recently_seen_codes, pilot_max_risk_yen)` | F111 candidate に additive で v1_4_1 namespace 追加 |

## §5 v1.4.1 設計原則の runner 実装確認

### 5.1 risk_yen 非加点 (= Lane B YES)

- `compute_pre_open_score` signature に **risk_yen 引数なし** (= test `test_pre_open_score_signature_no_risk_yen`)
- `apply_v1_4_1_policy` 内で risk_yen は **share_unit gate のみ**に使用 (= 順位形成に使われない)
- 4404 risk 10665 > 9628 risk 6950 でも、9628 が #1 になるのは **cap_tier 差**によるもの (= risk 加点ではない)
- standard_lot_ok=False は watch 降格、entry 候補リストから除外

### 5.2 100 株標準 (= Lane B YES)

- `SHARE_UNIT_STANDARD = 100` 固定
- `check_share_unit_pilot_fit` は 100 株 risk_yen を pilot_max_risk_yen と比較
- 超過時は standard_lot_ok=False → watch 降格 (= 非 100 株調整は不採用)
- test `test_share_unit_check_no_60_in_reason` / `test_apply_policy_no_60_in_any_field` で "60 株" 文言が出力に出ないこと検証

### 5.3 entry / watch / excluded 分類 (= Lane A/C YES)

```
priority order:
  liquidity FAIL                  → excluded_candidate
  recently_seen demoted           → excluded_candidate
  standard_lot_ok = False         → watch_candidate
  chase_limit_exceeded = True     → watch_candidate (= no-new-chase)
  liquidity WARN                  → watch_candidate
  else (= PASS + lot_ok + no demote + no chase) → entry_candidate
```

### 5.4 pre_open_score / entry_priority 分離 (= Lane A YES)

- `pre_open_score`: 静的 proxy (= 寄り前 staging データのみ)
- `entry_priority`: post-open 動的 (= 寄り後 actual flow override で更新)
- 初期値 entry_priority=None、apply_v1_4_1_policy 後の post-open phase で更新想定

### 5.5 signal_success / entry_success 分離 (= Lane E YES)

- 2 つの独立 Optional[str] field
- review 受領時に Y/N/NA で更新
- D31 9130 = signal Y / entry N、D31 4389 = signal Y / entry NA を表現可能

### 5.6 no-new-chase (= Lane D YES)

- `chase_limit_exceeded` / `no_new_chase` fields
- 初期 False、post-open phase で動的更新
- True 設定時 → candidate_role=watch_candidate に自動降格

### 5.7 review 10 項目接続 (= Lane E YES)

- `review_required_fields_version="v1.4.1"`
- `review_required_fields_count=10`
- review.md 10 項目構造 (= §1-§10) と version 整合

## §6 D31/D32 fixture tests (= Lane F YES)

| test | 検証 |
|---|---|
| test_d32_9247_entry_candidate | 9247 ＴＲＥ (貸借+TOPIX Small 1、risk 8060) → entry |
| test_d32_9628_entry_candidate | 9628 燦 (貸借+TOPIX Small 2、risk 6950) → entry |
| test_d32_9130_recently_seen_demoted | 9130 demote 想定 → excluded |
| test_d32_9130_no_demote_entry | 9130 = demote なし → entry (= chase_limit 別判定) |
| test_d32_4404_share_unit_exceeded_watch | 4404 risk 10665 → standard_lot_ok=False → watch |
| test_d31_331A0_excluded | 331A0 (信用+非TOPIX+letter-suffix) → excluded |
| test_d31_4389_excluded | 4389 (信用+非TOPIX) → excluded (= watch_signal_success 観察) |
| test_additive_only_existing_fields_preserved | 既存 21 field 保持確認 |
| test_review_required_fields_version | v1.4.1 metadata 出力 |
| test_signal_entry_success_initial_none | 初期 None (= review 提供時更新) |
| test_no_new_chase_initial_false | 初期 False (= post-open 動的) |
| TestRiskYenNonRanking (2 件) | risk_yen 順位加点ではない検証 |
| TestNo60ShareMention (2 件) | "60 株" 文言出力されない |
| TestSafety (2 件) | sqlite3/subprocess/aiohttp/linebot/launchctl import なし、token literal なし |

## §7 Codex 8 lane factual-confirm 結果 (= 全 YES、CRITICAL/HIGH 0)

| Lane | 観点 | reply | 採用 |
|---|---|---|---|
| A | field contract (21 fields、pre_open/entry_priority 分離、三分類) | YES ✓ | 採用 |
| B | risk_yen 非加点、100 株標準、上限超過 watch 降格、非 100 株調整なし | YES ✓ | 採用 |
| C | 331A0/4389/4317 等低流動性 entry に戻らない、三分類妥当 | YES ✓ | 採用 |
| D | 9130 chase_limit 超過 → watch、recently_seen demote → excluded | YES ✓ | 採用 |
| E | signal_success と entry_success 別 field、review 10 項目接続 | YES ✓ | 採用 |
| F | unit + D31/D32 fixture tests 十分、42 tests + 既存 4812 全 PASS | YES ✓ | 採用 |
| G | safety / output guard 違反なし、forbidden import なし、token literal なし | YES ✓ | 採用 |
| H | D31 (9130/331A0/4389) と D32 (9247/9628/4404) 混同なし、正本関係維持 | YES ✓ | 採用 |

→ **8/8 YES、CRITICAL/HIGH 0、棄却理由なし**

## §8 tests 結果

- 新 tests: 42 PASS (= test_selection_policy_v1_4_1.py)
- 既存 F111 tests: 82 PASS 維持 (= test_run_f111_real_batch_staging.py)
- 全 pytest: **4854 PASS / 0 FAIL** (= 4812 既存 + 42 新)
- 警告: 48 (= GoalConfig fallback 既知、本 wave 由来なし)

## §9 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → 不変 ✓ |
| F282 plist (weekly-snapshot) | size=1772 / mtime=1778593597 → 不変 ✓ |
| DB write / API / token | 全 0 ✓ |
| LINE / launchctl / plist / cron | 全 0 ✓ |
| git commit / git push / --no-verify | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| 本 wave write | code 3 file (= helper.py 新規 / test.py 新規 / F111 runner.py 最小 additive) + vault doc 1 file (= 本 design doc) |
| forbidden import | sqlite3 / subprocess / requests / aiohttp / linebot / launchctl 全 0 ✓ |
| token literal | LINE_CHANNEL_TOKEN / CHANNEL_TOKEN / JQUANTS_API_KEY / LINE_USER_ID 全 0 ✓ |

## §10 D31 / D32 正本関係維持 (= Lane H YES)

- **D31 review 正本**: `04_daily/2026-06-25_manual_live_pilot_review.md` (= 9130/331A0/4389、Fujiwara 全 skip + 見送り理由 filled)
- **D32 review**: `04_daily/2026-06-26_manual_live_pilot_review.md` (= 9247/9628/9130/4404、Fujiwara 全 skip filled、OHLCV pending)
- 本実装 fixture tests は D31 (= 331A0/4389) と D32 (= 9247/9628/9130/4404) の両方を分離記載
- 9247/9628/4404 を D31 実績と混同していない (= post-D31 改善候補として扱う)

## §11 next wave action

### Priority A (= 即時)
1. F111 runner output で v1_4_1 namespace が実際に出力されるか smoke test (= staging chain 1 回実行で確認)
2. AFTER-R1 / morning advisory runner で v1_4_1 fields を consumer 側で読む実装

### Priority B (= 後続)
3. post-open phase 動的更新の実装 (= chase_limit_exceeded / entry_priority / no_new_chase の actual flow ベース更新)
4. review.md 10 項目から signal_success / entry_success を runner output へ back-fill する pipeline
5. D33 pilot で v1.4.1 runner output を実検証
6. D34/D35 demote sim/本実行で recently_seen_codes 連携を実検証

### Priority C (= 長期)
7. F062 advisory / paper PnL preview runner にも v1.4.1 namespace consumer 連携
8. W5 集約で v1.4.1 framework 14 day 総括

## §12 まとめ

v1.4.1 selection policy を **advisory/design layer から F111 runner-level へ正式実装**完了。
helper module + 42 tests + F111 runner 最小 additive 統合 + Codex 8 lane 全 YES。
D31/D32 fixture で実装妥当性検証済、CRITICAL/HIGH 0、safety 0 違反、3 DB md5 / F282 plist 不変。
runner output に v1_4_1 namespace (= 21 fields の field contract) が candidate ごとに付与される
状態に到達。advisory layer の policy が code レベルで再現可能になった。
