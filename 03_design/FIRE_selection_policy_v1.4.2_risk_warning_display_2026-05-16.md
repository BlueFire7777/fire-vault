---
id: FIRE-design-selection-policy-v1.4.2-risk-warning-display-2026-05-16
phase: 本番 v0 / v1.4.2 risk_yen_over_pilot_budget warning 表示
priority: 最高
status: complete
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
parent_design: 03_design/FIRE_selection_policy_v1.4.2_risk_cap_removal_2026-05-16.md
codex_rounds: 2 (= 初回 + Lane C/F 修正再確認)
codex_lanes: 8 / 8 YES ✓ (= Lane C/F 初回 NO → 2 巡目で YES)
critical_high_count: 0
test_count: 190 (= MD 87 + consumer 50 + selection_policy 51 + 新 2 regression)
test_total_pytest: 5071 PASS (= 前 wave 5060 + 本 wave 新 11、regression 0)
---

# FIRE Selection Policy v1.4.2 — risk_yen_over_pilot_budget warning 表示

## §1 目的

v1.4.2 で撤廃した pilot risk cap gate の代わりに、**risk_yen が大きい候補を「除外」せず「警告表示」**
するための表示・consumer・Markdown 連携 wave.

100 株標準 / 非 100 株調整禁止 / liquidity gate / no-new-chase / recently_seen demote は維持.

## §2 修正 file

### 2.1 code 修正 (= 2 file)

| file | 変更内容 |
|---|---|
| `scripts/jobs/_v1_4_1_consumer.py` | `format_candidate_summary()` に `risk_yen_over_pilot_budget` mapping 追加 / `is_entry_displayable()` から旧 `risk_within_pilot_limit` gate 撤廃 (= Codex Lane C HIGH 修正) |
| `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` | `render_entry_section()` / `render_order_advisory()` / `render_stdout_summary()` に warning 表示追加 |

### 2.2 tests 修正 (= 2 file)

| file | 追加内容 |
|---|---|
| `tests/scripts/jobs/test_v1_4_1_consumer.py` | TestV1_4_2_RiskYenOverPilotBudgetPropagation (= 3 propagation tests + 2 regression tests = 5 tests) |
| `tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py` | TestV1_4_2_RiskWarningDisplay (= 6 tests: entry section / order advisory / stdout summary / full md / validation / normal 無 warning) |

### 2.3 vault doc (= 本 file)

- `~/fire-vault/03_design/FIRE_selection_policy_v1.4.2_risk_warning_display_2026-05-16.md`

## §3 表示内容

### 3.1 entry section (= 4404 fixture 例)

```
### 3) 4404 ミヨシ油脂 (食品 / -)

- ⚠ **risk warning** (= 旧 pilot 上限超過): 100 株 risk_yen = 10,665 円
  (= entry 除外条件ではなく、損切り設計 / position awareness 用)
  → 板 / 出来高 / spread / RR / 許容損失を **手動確認** してから entry 判断
- entry 検討価格: 2,133 円
- 指値目安: 2,133 円 (= 寄付前 reference)
- 追わない上限: 2,176 円 (= +2.0% gap)
- 利確目安: 2,346 円 (= +2R reference)
- 損切り目安: 2,026 円 (= -1R reference)
- 株数: 100 株 (= 標準、非標準調整なし)
- risk_yen_for_100_shares: 10,665 円
  (= 100 株あたり最大損失 budget ⚠ v1.4.2 旧 pilot 上限超過警告)
```

### 3.2 注文完成形 advisory (= 4404 fixture 例)

```
### 3) 4404 ミヨシ油脂

- ⚠ **risk warning** (= 旧 pilot 上限超過): 100 株 risk_yen = 10,665 円
  → 損切り価格 / 許容損失 / 板 / 出来高 / spread / RR を **手動確認** してから発注
- 銘柄コード: 4404
- 株数: 100 株 (= 標準)
- 指値: 2,133 円 (= 寄付前 reference)
- 損切り: 2,026 円 (= -1R)
- 利確: 2,346 円 (= +2R)
- 追わない上限: 2,176 円 (= +2.0%)
```

### 3.3 stdout summary (= 4404 fixture 例)

```
  3) 4404 ミヨシ油脂 ⚠ risk_yen warning (sector=食品, price=2,133 円,
    chase_limit=2,176 円, stop=2,026 円, take=2,346 円, shares=100 株, risk=10,665 円)
```

## §4 risk_yen_over_pilot_budget mapping

### 4.1 consumer (= `format_candidate_summary()` 拡張)

```python
if rr.has_namespace:
    ns = rr.namespace
    base.update({
        ...
        # v1.4.2: risk 警告フラグ (= entry 除外には使わない、表示警告のみ)
        "risk_yen_over_pilot_budget": bool(ns.get("risk_yen_over_pilot_budget", False)),
        ...
    })
```

### 4.2 namespace 不在 fallback

- fallback では `risk_yen_over_pilot_budget` field 不在 (= None or False)
- 旧 v1.4.1 namespace payload (= field 不在) でも False default

### 4.3 is_entry_displayable() 変更 (= Codex Lane C HIGH 修正)

**旧 (v1.4.1)**: F111 top-level field `risk_within_pilot_limit` が True 必須

**新 (v1.4.2)**: `risk_within_pilot_limit` gate **撤廃**.
v1_4_1 namespace の `candidate_role=entry_candidate` を正本.
risk_yen 大きくても entry 表示.

## §5 影響を受けた候補例

### 5.1 4404 ミヨシ油脂

| 観点 | 旧 v1.4.1 | 新 v1.4.2 (= risk cap 撤廃) | 本 wave (= warning 表示) |
|---|---|---|---|
| candidate_role | watch | entry_candidate | entry_candidate (= 維持) |
| Markdown entry section | 不出 | 出力 | 出力 + ⚠ warning header |
| 注文完成形 | 不出 | 出力 | 出力 + ⚠ warning header |
| stdout summary | "watch" entry に | entry に price/risk 表示 | entry + " ⚠ risk_yen warning" marker |

### 5.2 9247 / 9628 (= 通常 entry)

- `risk_yen_over_pilot_budget=False` (= warning フラグなし)
- warning header 表示 **なし** (= 通常 entry 表示)

### 5.3 9130 / 331A0 / 4389 / 4317 (= 維持)

- 9130: recently_seen demoted → excluded 維持 (= warning 関係なし)
- 331A0 / 4389 / 4317: liquidity FAIL → excluded 維持
- 全て invariant 維持 ✓

## §6 smoke 結果

### 6.1 実行

```
.venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
  --base-date 2026-07-06 \
  --recently-seen-codes 8747,5729,3489,340A0,3798,137A0,7991,9130 \
  → /tmp/fire_v1_4_2_risk_warning_smoke/d38_f111.json

→ consumer payload → morning advisory MD
→ /tmp/fire_v1_4_2_risk_warning_smoke/morning_advisory_risk_warning.md (7,594 bytes)
```

### 6.2 結果

- entry: 3 (= 9247, 9628, **4404 + risk warning**) ✓
- watch: 0
- excluded: 17 (= 9130 demote + 331A0/4389/4317 等 liquidity FAIL)
- md_violations: 0 / payload_violations: 0 / validation_passed: True
- forbidden phrase 全 0 件 (= "60 株" / "買え" / "必ず" / "自動発注" 等)
- 4404 entry section + 注文完成形 + stdout 全 3 箇所に warning 表示 ✓

## §7 tests

### 7.1 新規 tests (= 計 11、`test_v1_4_1_morning_advisory_markdown.py` + `test_v1_4_1_consumer.py`)

**MD warning 表示 (= 6 tests in test_v1_4_1_morning_advisory_markdown.py)**:
1. `test_4404_entry_section_shows_warning`: entry section warning 確認
2. `test_4404_order_advisory_shows_warning`: 注文完成形 warning 確認
3. `test_4404_stdout_summary_shows_warning_marker`: stdout warning marker 確認
4. `test_normal_candidate_no_warning`: 通常候補で warning 表示なし
5. `test_v1_4_2_4404_full_markdown`: full markdown 経由で確認
6. `test_validate_markdown_output_clean`: validation violations 0

**consumer propagation + regression (= 5 tests in test_v1_4_1_consumer.py)**:
1. `test_risk_yen_over_pilot_budget_propagated_when_true`: True 伝播
2. `test_risk_yen_over_pilot_budget_default_false_when_absent`: 不在時 False
3. `test_fallback_no_namespace_no_warning`: fallback で warning なし
4. `test_is_entry_displayable_v1_4_2_risk_within_pilot_limit_gate_removed`: 旧 gate 撤廃確認
5. `test_is_entry_displayable_v1_4_2_high_risk_no_pilot_field`: field 不在でも entry 表示

### 7.2 tests 結果

- `test_v1_4_1_morning_advisory_markdown.py`: 87 PASS (= 81 既存 + 6 新規)
- `test_v1_4_1_consumer.py`: 52 PASS (= 47 既存 + 5 新規)
- `test_selection_policy_v1_4_1.py`: 51 PASS (= v1.4.2 既存)
- 全 pytest: **5071 PASS** (= 前 wave 5060 + 本 wave 新 11、regression 0) ✓

## §8 Codex 8 lane 監査

### 8.1 巡目別結果

| 巡 | A | B | C | D | E | F | G | H | 追加 HIGH |
|---|---|---|---|---|---|---|---|---|---|
| 1 巡目 | YES | YES | **NO**(HIGH) | YES | YES | **NO** | YES | YES | Lane C: risk_within_pilot_limit gate 残存、Lane F: regression test 未追加 |
| 2 巡目 (= 修正後) | - | - | **YES** | - | - | **YES** | - | - | **なし** |

### 8.2 採用 / 棄却理由

- Lane A: `format_candidate_summary()` で `risk_yen_over_pilot_budget` を `bool(..., False)` で伝播、
  不在時 False、fallback で None / False
- Lane B: entry section / 注文完成形 / stdout summary に "⚠ risk warning" + "手動確認" 文言、
  警告であって entry 除外していない
- Lane C (= 2 巡目 YES): `is_entry_displayable()` から旧 `risk_within_pilot_limit` gate を撤廃、
  v1_4_1 namespace の `candidate_role` を正本
- Lane D: 100 株標準維持、60/30/50 株文言 grep 0 件
- Lane E: liquidity FAIL / recently_seen_demoted / no-new-chase 維持
- Lane F (= 2 巡目 YES): regression tests 2 件追加 (= risk_within_pilot_limit=False / 不在
  両ケースで entry 表示維持確認)
- Lane G: DB / LINE / token / API / launchctl 全 0、出力先 /tmp/fire_v1_4_2_risk_warning_smoke/ 適正
- Lane H: D31-D37 invariant と矛盾なし

## §9 safety final

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
| forbidden phrase 全 0 件 | ✓ |
| 本 wave 修正 | code 2 file + tests 2 file + vault doc 1 file + smoke 1 file |

## §10 次 wave 提案

### Priority A
1. **D38 朝 pilot (= 2026-07-06 月、v1.4.2 + warning display 反映)**
   - 4404 が entry 候補に **⚠ warning 付き** で復帰した状態で実 pilot 運用
   - Fujiwara が warning 確認 → 損切り設計 / 板 / RR 確認後に entry 判断

### Priority B
2. **F111 max_candidates 拡張 baseline 化検討** (= D37 sector 多様化 sim follow-up)
3. **D32-D37 actual fill** (= Fujiwara verbatim review、J-Quants 待ち)

### Priority C
4. **demote 永続化** (= 環境変数 / config / runner default、別 wave)
5. **git commit + push 実行** (= 前 wave 計画済、HQ approve 後)
   - 新規追加: v1.4.2 risk warning display 修正 (= consumer + MD + tests)
6. **post-open update wrapper v1.4.2 対応検討** (= post-open 後の risk_yen_over_pilot_budget
   再評価が必要か、別 wave)

## §11 まとめ

v1.4.2 risk_yen_over_pilot_budget warning 表示 **完全実装** ✓.
consumer payload で警告フラグ伝播、Markdown renderer の entry section / 注文完成形 advisory /
stdout summary の **3 箇所**に warning 表示. 4404 が entry 候補に復帰しつつ、Fujiwara が
損切り設計 / 板 / RR / 許容損失を手動確認するよう促す UI を確立.

100 株標準 / 非 100 株調整禁止 / liquidity gate / no-new-chase / recently_seen demote 全 invariant
維持. Codex 8 lane 2 巡で全 YES、CRITICAL/HIGH 0、tests 全関連 190 PASS、全 pytest 5071 PASS、
safety 0 違反、3 DB md5 / F282 plist 3 file 全不変.
