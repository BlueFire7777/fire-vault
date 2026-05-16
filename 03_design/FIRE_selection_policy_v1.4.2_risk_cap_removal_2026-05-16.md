---
id: FIRE-design-selection-policy-v1.4.2-risk-cap-removal-2026-05-16
phase: 本番 v0 / v1.4.2 initial pilot risk cap gate 撤廃
priority: 最高
status: complete
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
parent_design: 03_design/FIRE_selection_policy_v1.4.1_runner_implementation_2026-05-15.md
codex_lanes: 8 / 8 YES ✓ (= 1 巡で全 YES)
critical_high_count: 0
test_count: 51 (= 既存 44 + v1.4.2 新 7)
test_total_pytest: 5060 PASS (= 前 wave 5051 + v1.4.2 新 9 / regression 0)
---

# FIRE Selection Policy v1.4.2 — Initial Pilot Risk Cap Gate 撤廃

## §1 目的

v1.4.1 で risk_yen > pilot 上限 (= 8,500 円) を超える銘柄を **watch 降格** していたが、
**大型・中型・流動性十分・相場テーマに沿った銘柄が候補から脱落**する副作用が発生.
特に 4404 ミヨシ油脂 (= 食品/油脂、貸借、PASS、risk 10,665) が watch 降格されたことで、
中型・高 risk 銘柄を pilot 範囲から不必要に除外していた.

**v1.4.2 で risk cap gate を撤廃**し、risk_yen は **表示 / 警告 / 損切り設計用**に限定.
100 株標準は維持、非 100 株調整は引き続き禁止.

## §2 v1.4.1 から v1.4.2 への変更点

### 2.1 POLICY_VERSION

- 旧: `POLICY_VERSION = "1.4.1"`
- 新: `POLICY_VERSION = "1.4.2"`

### 2.2 `check_share_unit_pilot_fit()` logic 変更

**旧 v1.4.1**:
```python
if risk_int <= pilot_max_risk_yen:
    standard_lot_ok=True
else:
    standard_lot_ok=False  # ← 4404 がここで False → watch 降格
```

**新 v1.4.2**:
```python
if risk_int <= pilot_max_risk_yen:
    standard_lot_ok=True
    risk_yen_over_pilot_budget=False
else:
    standard_lot_ok=True  # ← v1.4.2: 100 株単位発注可能なら True 維持
    risk_yen_over_pilot_budget=True  # ← v1.4.2 新フィールド、警告のみ
```

### 2.3 新フィールド `risk_yen_over_pilot_budget`

| 値 | 意味 |
|---|---|
| `False` | risk_yen が pilot 警告 threshold 内、または risk_yen 不明 |
| `True` | risk_yen が pilot 警告 threshold を超過 (= 警告フラグ、entry 除外には使わない) |

**用途**:
- 表示用 (= morning advisory / trade plan に警告マーク表示)
- 損切り設計の参考値
- position awareness (= Fujiwara が pilot 余力管理に使う)

### 2.4 `classify_candidate_role()` 変更

`standard_lot_ok=False` watch 降格は維持するが、**意味が変わる**:

| 旧 v1.4.1 | 新 v1.4.2 |
|---|---|
| `standard_lot_ok=False` = risk_yen > pilot 上限 (= risk gate) | `standard_lot_ok=False` = risk_yen 不明 (= 100 株発注不可、損切り設計不能) |
| reason: "100 株 risk_yen が pilot 限度超、非 100 株調整不採用" | reason: "risk_yen 不明、100 株発注不可、損切り設計不能" |

### 2.5 reason_codes 変更

| 旧 v1.4.1 | 新 v1.4.2 |
|---|---|
| `share_unit_pilot_exceeded` | **削除** (= entry 除外 reason ではなくなった) |
| - | `risk_yen_warning` (= 新規、警告のみ、entry 除外しない) |
| - | `risk_yen_unknown` (= 新規、risk_yen None 時、watch 降格) |

### 2.6 維持された invariants

- `share_unit = 100` 固定
- 非 100 株調整禁止 (= 60-share / 非標準 lot 案出力なし)
- `liquidity_status` FAIL (= 信用 + 非 TOPIX or letter-suffix 新規) → entry 除外
- `recently_seen demoted` → entry 除外
- `chase_limit_exceeded` → watch 降格
- `liquidity_status` WARN (= 信用 + TOPIX Small) → watch
- `compute_pre_open_score()` で risk_yen 非加点 (= 順位形成に使わない)
- `review_required_fields_version = "v1.4.1"` 維持 (= consumer/review 互換性)
- namespace 名も `v1_4_1` 維持 (= consumer 互換性)

## §3 修正 file

### 3.1 修正済 (= 2 file)

| file | 修正内容 |
|---|---|
| `scripts/jobs/_selection_policy_v1_4_1.py` | POLICY_VERSION 1.4.2、check_share_unit_pilot_fit logic 変更、新 field risk_yen_over_pilot_budget、classify_candidate_role 意味変更、reason_codes 更新、docstring v1.4.2 反映 |
| `tests/scripts/jobs/test_selection_policy_v1_4_1.py` | 既存 2 test を v1.4.2 用に更新、TestV1_4_2_RiskCapRemoval class 追加 (= 7 tests) |

### 3.2 変更なし (= 互換性維持)

- `scripts/jobs/run_f111_real_batch_staging.py` (= apply_v1_4_1_policy import 維持)
- `scripts/jobs/_v1_4_1_consumer.py` (= namespace v1_4_1、field standard_lot_ok 互換性)
- `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` (= 同上)
- `scripts/jobs/_v1_4_1_post_open_update.py` (= 同上)

## §4 影響を受けた候補例

### 4.1 4404 ミヨシ油脂 (= 食品/油脂、貸借、scale=-、close=2,133、risk=10,665)

| 観点 | v1.4.1 | v1.4.2 |
|---|---|---|
| standard_lot_ok | False | **True** ✓ |
| risk_yen_over_pilot_budget | (なし) | **True** (= 警告フラグ) |
| candidate_role | watch_candidate | **entry_candidate** ✓ |
| reason_codes | `['share_unit_pilot_exceeded']` | `['risk_yen_warning']` |
| Fujiwara 判断 | "100 株 risk 上限超で entry 不可" | "risk 警告あり、損切り設計時に注意" |

### 4.2 9130 / 9247 / 9628 / 331A0 / 4389 / 4317 等 (= 変化なし)

- 9130: recently_seen demoted → excluded 維持 (= v1.4.2 でも変化なし)
- 9247 / 9628: PASS + standard_lot_ok=True + 警告フラグなし → entry 維持
- 331A0 / 4389 / 4317: liquidity FAIL → excluded 維持
- demote 7 件 (= 8747/5729/3489/340A0/3798/137A0/7991): recently_seen → excluded 維持

### 4.3 D37 v1.4.2 smoke 結果 (= 9130 含む recently_seen 8 件版)

| section | 件数 | codes |
|---|---|---|
| entry | 3 | 9247, 9628, **4404** (= v1.4.2 新規昇格) |
| watch | 0 | (= 4404 が entry へ昇格、9130 は excluded) |
| excluded | 17 | demote 8 + liquidity FAIL 9 |

**変化**:
- 旧 v1.4.1: entry 2 (= 9247/9628) / watch 1 (= 4404)
- 新 v1.4.2: entry 3 (= 9247/9628/**4404**) / watch 0

→ **4404 が entry 候補に昇格** ✓、新候補劣化なし

## §5 仮想 large/mid cap シナリオ確認

v1.4.2 の効果を以下の仮想シナリオで検証:

| code (= hypothetical) | sector | scale | close | risk | role (v1.4.2) | reason_codes |
|---|---|---|---|---|---|---|
| 7203 大型仮想 | 自動車・輸送機 | TOPIX Large70 | 5,000 | 25,000 | **entry_candidate** ✓ | `['risk_yen_warning']` |
| 6098 中型仮想 | 情報通信 | TOPIX Mid400 | 4,500 | 22,500 | **entry_candidate** ✓ | `['risk_yen_warning']` |

→ 大型・中型・高 risk 銘柄も entry 候補に残れる ✓ (= test_large_cap_high_price_no_block)

## §6 tests

### 6.1 新規 test class (= TestV1_4_2_RiskCapRemoval、7 tests)

1. `test_policy_version_is_1_4_2`: POLICY_VERSION = "1.4.2"
2. `test_risk_yen_warning_does_not_block_entry`: 4404 entry 維持確認
3. `test_large_cap_high_price_no_block`: 大型・高 risk 銘柄 entry 維持
4. `test_risk_yen_none_still_blocks_lot_ok`: risk_yen None → watch (= 損切り設計不能)
5. `test_low_liquidity_still_excluded`: 信用 + 非 TOPIX 銘柄 (= 4317) excluded 維持
6. `test_demoted_still_excluded`: recently_seen demoted (= 9130) excluded 維持
7. `test_no_60_share_in_output`: 非 100 株調整文言 0 件確認

### 6.2 更新 test

1. `TestCheckShareUnitPilotFit.test_exceeds_limit_v1_4_2_still_lot_ok` (= 旧 test_exceeds_limit_not_ok 更新)
2. `TestApplyV141Policy.test_d32_4404_v1_4_2_risk_warning_entry` (= 旧 test_d32_4404_share_unit_exceeded_watch 更新)
3. `TestRiskYenNonRanking.test_higher_risk_does_not_lower_pre_open` (= 4404 role 期待値 watch → entry に変更)

### 6.3 tests 結果

- `test_selection_policy_v1_4_1.py`: **51 PASS** (= 既存 44 + v1.4.2 新 7)
- 全 pytest: **5060 PASS** (= 前 wave 5051 + v1.4.2 新 9、regression 0)

## §7 Codex 8 lane 監査

| Lane | 観点 | 結果 |
|------|------|------|
| A | risk cap gate 撤廃 | **YES ✓** |
| B | risk_yen 非加点 | **YES ✓** |
| C | 100 株標準 | **YES ✓** |
| D | liquidity gate 維持 | **YES ✓** |
| E | large/mid candidate 維持 | **YES ✓** |
| F | no-new-chase 維持 | **YES ✓** |
| G | tests 十分性 | **YES ✓** |
| H | safety / docs | **YES ✓** |

**実行 1 巡で全 YES、追加 CRITICAL/HIGH なし**.

採用理由:
- Lane A: risk_yen > 8500 でも standard_lot_ok=True、risk_yen_over_pilot_budget=True 警告のみ
- Lane B: compute_pre_open_score 引数 / logic に risk_yen 含まれず
- Lane C: SHARE_UNIT_STANDARD=100 固定、60/30/50 株 grep 0 件
- Lane D: classify_liquidity_status v1.4.1 と同一、331A0/4389/4317 excluded 維持
- Lane E: 4404 entry 復帰、大型仮想 7203/6098 想定も entry 候補維持
- Lane F: 9130 recently_seen_demoted → excluded 維持、chase_limit logic 不変
- Lane G: TestV1_4_2_RiskCapRemoval 7 tests + 既存 test 更新 3 件、全 5060 PASS
- Lane H: pure functions、DB / LINE / token / API / launchctl 全 0、plist 3 file 別個確認

## §8 safety final

| 観点 | 結果 |
|---|---|
| production md5 (= fire.db) | b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → 不変 ✓ |
| F282 production plist | size=1772 / mtime=1778592445 → 不変 ✓ |
| F282 smoke plist | size=1844 / mtime=1778769277 → 不変 ✓ |
| launchctl loaded plist | size=1772 / mtime=1778593597 → 不変 ✓ |
| DB write / API / token / LINE / launchctl | 全 0 ✓ |
| git commit / git push / --no-verify | 全 0 ✓ |
| sudo / rm -rf | 全 0 ✓ |
| forbidden phrase 全 0 件 (= "60 株" / "買え" / "必ず" / "自動発注" 等) | ✓ |
| 本 wave 修正: code 1 file + tests 1 file + 本 vault doc | 計 3 file |

## §9 次 wave 提案

### Priority A
1. **D38 朝 pilot (= 2026-07-06 月、v1.4.2 baseline 使用)**
   - 4404 が entry 候補に復帰する状態で実 pilot 運用
   - 9247 / 9628 / 4404 の 3 entry 候補から Fujiwara が選択判断
   - sector 集中化が改善 (= 情報通信 2 + 食品 1) するか確認

### Priority B
2. **morning advisory Markdown renderer に risk_yen_over_pilot_budget 警告表示**
   - entry 候補に "⚠ risk 警告 (= 損切り設計時注意)" マーク追加
   - trade plan / review template にも警告反映

3. **F111 max_candidates 拡張 baseline 化検討** (= D37 sector 多様化 sim の follow-up)
   - max=20 → max=50 で実 baseline 化
   - sector 集中化 + risk cap 撤廃の相乗効果評価

### Priority C
4. **D32-D37 actual fill** (= Fujiwara verbatim review、J-Quants 待ち)
5. **demote 永続化** (= 環境変数 / config / runner default)
6. **git commit + push 実行** (= 前 wave 計画済、HQ approve 後)
   - **新規追加**: v1.4.2 selection policy 修正 commit (= _selection_policy_v1_4_1.py + tests)

## §10 まとめ

v1.4.2 で **risk cap gate 完全撤廃** ✓.
4404 ミヨシ油脂 (= 中型・食品・貸借・risk 10,665) が entry 候補に復帰し、
大型・中型・高 risk 銘柄が pilot 範囲から不必要に除外されない構造を確立.

100 株標準 / 非 100 株調整禁止 / liquidity gate / no-new-chase / recently_seen demote 等の
invariants は全て維持 (= v1.4.1 から regression 0).

Codex 8 lane 1 巡で全 YES、CRITICAL/HIGH 0、tests 51 PASS、全 pytest 5060 PASS、
safety 0 違反、3 DB md5 / F282 plist 3 file 全不変.
