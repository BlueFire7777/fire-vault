---
id: FIRE-design-selection-policy-v1.4.1-runner-smoke-2026-05-15
phase: 本番 v0 中核 / F111 runner v1.4.1 smoke test
priority: 高
status: complete
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
parent_design: 03_design/FIRE_selection_policy_v1.4.1_runner_implementation_2026-05-15.md
codex_lanes: 4 / 4 YES ✓
critical_high_count: 0
test_count: 124 PASS (= 42 helper + 82 F111)
sixty_share_mention_count: 0 (= 非 100 株調整出力なし grep 検証)
---

# F111 Runner v1.4.1 Smoke Test 結果

## §1 smoke 実行

### 1.1 コマンド

    .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
      --base-date 2026-06-25 \
      --max-candidates 20 \
      --label-threshold-mode strict \
      --recently-seen-codes 8747,5729,3489,340A0,3798,137A0,7991 \
      --output-json /tmp/fire_f111_v1_4_1_smoke/f111_v1_4_1_d31_smoke.json

### 1.2 結果

    [F111-real-batch] db=/Users/bluefire/fire/data/fire.staging.db mode=read-only (URI mode=ro)
    [F111-real-batch] safety: no DB write / no LINE / no order / no token / no launchctl / no Computer Use
    [F111-real-batch] done: candidates=20 eligible=19 exclusions={'sample_ticker': 0, 'tradable_universe_false': 0, 'risk_above_pilot_limit': 1, 'risk_estimate_missing': 0} output=/tmp/fire_f111_v1_4_1_smoke/f111_v1_4_1_d31_smoke.json

### 1.3 artifact path

- `/tmp/fire_f111_v1_4_1_smoke/f111_v1_4_1_d31_smoke.json`

## §2 v1_4_1 namespace 確認

### 2.1 tuning_params

    {
      "label_threshold_mode": "strict",
      "recently_seen_codes": ["340A0", "7991", "5729", "8747", "3489", "137A0", "3798"],
      "demoted_count": 7,
      "selection_policy_version": "1.4.1",      ← 新規 ★
      "share_unit_standard": 100                ← 新規 ★
    }

### 2.2 必須 21 fields (= candidate[0] 8747 の v1_4_1)

全 21 必須 fields 出力確認 ✓ (= 欠落 0 件):

| field | sample value (8747) |
|---|---|
| policy_version | "1.4.1" ✓ |
| share_unit | 100 ✓ |
| standard_lot_ok | False |
| risk_yen_for_100_shares | 12100 |
| liquidity_status | "FAIL" |
| liquidity_tier | 1 |
| cap_tier | 3 |
| is_letter_suffix_new_listing | False |
| pre_open_score | 1.8299 |
| candidate_role | "excluded_candidate" |
| candidate_role_reason | "liquidity_status=FAIL (= 信用 + 非 TOPIX or letter-suffix 新規)" |
| chase_limit_exceeded | False |
| no_new_chase | False |
| entry_priority | None |
| signal_success | None |
| entry_success | None |
| review_required_fields_version | "v1.4.1" ✓ |
| review_required_fields_count | 10 ✓ |
| reason_codes | ["liquidity_fail", "share_unit_pilot_exceeded", "recently_seen_demoted"] |
| exclusion_reason | "liquidity_status=FAIL (= 信用 + 非 TOPIX or letter-suffix 新規)" |
| watch_reason | None |

## §3 candidate_role 集計

| role | 件数 | codes |
|---|---|---|
| **entry_candidate** | 3 | 9130, 9247, 9628 (= 全 貸借) |
| **watch_candidate** | 1 | 4404 (= 貸借 + 100 株 risk 上限超) |
| **excluded_candidate** | 16 | 8747, 5729, 3489, 340A0, 3798, 137A0, 7991 (= demote 7 件) + 331A0, 4389, 4317, 2700, 6149, 288A0, 2981, 8152, 339A0 (= 信用/非 TOPIX/letter-suffix) |

## §4 D31/D32 代表ケース検証

### 4.1 D31 reality (= 9130/331A0/4389) の運用整合

| code | name | role | liquidity | risk_yen | letter_suffix | reason_codes |
|---|---|---|---|---|---|---|
| **9130** 共栄タンカー | 海運業/タンカー | **entry_candidate** | PASS | 7050 | False | [] |
| **331A0** メディックス | 情報通信 | **excluded_candidate** | FAIL | 2410 | True | [liquidity_fail, letter_suffix_new_listing] |
| **4389** プロパティデータバンク | 情報通信 | **excluded_candidate** | FAIL | 4315 | False | [liquidity_fail] |
| **4317** レイ | 情報通信 | **excluded_candidate** | FAIL | 2540 | False | [liquidity_fail] |

→ 331A0/4389/4317 全 excluded 確認 (= D31 review 実績と整合)

### 4.2 D32 候補 (= 9247/9628/4404) の運用整合

| code | name | role | liquidity | risk_yen | standard_lot_ok | reason_codes |
|---|---|---|---|---|---|---|
| **9247** ＴＲＥ HD | サービス業/リサイクル | **entry_candidate** | PASS | 8060 | True | [] |
| **9628** 燦 HD | サービス業/葬祭 | **entry_candidate** | PASS | 6950 | True | [] |
| **4404** ミヨシ油脂 | 食品工業/油脂 | **watch_candidate** | PASS | 10665 | **False** | [share_unit_pilot_exceeded] |

→ 9247/9628 entry、4404 watch (= 100 株 risk 上限超) 確認 (= D32 advisory と整合)

### 4.3 9130 no-new-chase 関連 (= chase_limit_exceeded 初期 False)

- 9130 entry_candidate (= recently_seen に未追加 + 貸借 + lot_ok)
- chase_limit_exceeded=False / no_new_chase=False = post-open 動的更新前の placeholder
- post-open phase で actual flow + chase_limit 判定後に True 設定想定
- 現状の runner output は **「再 entry 推奨」に見えない** (= chase_limit 判定の logic は consumer 側 / post-open phase で実施)

## §5 非 100 株調整 (= 60 株案等) 出力なし検証

    grep -c "60 株\|60株" /tmp/fire_f111_v1_4_1_smoke/f111_v1_4_1_d31_smoke.json

結果: **0 件** ✓ (= 非 100 株調整出力されない)

## §6 risk_yen 順位加点ではない検証

| candidate | risk_yen | role |
|---|---|---|
| 9628 | 6950 | entry_candidate |
| 9130 | 7050 | entry_candidate |
| 9247 | 8060 | entry_candidate |
| 4404 | 10665 | **watch_candidate** (= standard_lot_ok=False) |

→ entry candidate の risk_yen は 6,950-8,060 で混在 (= risk_yen 順位加点ではない)
→ 4404 が high risk で watch なのは standard_lot_ok=False が原因 (= 順位ではなく gate)
→ entry / watch 分類は risk_yen 値ではなく PASS/NG 判定に対応

## §7 tests 結果

| test 系 | 件数 |
|---|---|
| test_selection_policy_v1_4_1.py | **42 PASS** |
| test_run_f111_real_batch_staging.py | **82 PASS** |
| 合計 (関連) | **124 PASS** |

実行時間: 0.09s。

## §8 Codex 4 lane factual-confirm

| Lane | 観点 | reply | 採用 |
|---|---|---|---|
| A | v1_4_1 namespace + 必須 21 field + version/lot/review metadata | **YES ✓** | 採用 |
| B | risk_yen 非加点 + 100 株標準 + 上限超過 watch + 60 株文言なし | **YES ✓** | 採用 |
| C | 低流動性/小型/信用/letter-suffix entry 復帰なし + no_new_chase | **YES ✓** | 採用 |
| D | safety/tests/docs + D31-D32 正本関係維持 | **YES ✓** | 採用 |

→ **4/4 YES、CRITICAL/HIGH 0、棄却理由なし**

### Codex 採用/棄却理由

- Lane A: 必須 21 field 過不足なく出力、tuning_params に selection_policy_version も明示
- Lane B: entry/watch 分類が risk_yen 昇順 (6950→7050→8060→10665) ではなく PASS/NG 判定に対応、60 株文言 grep 0 件
- Lane C: 331A0 = liquidity FAIL + letter_suffix で excluded、9130 = chase_limit_exceeded False で「再 entry 推奨」に見えない (= post-open 動的更新前の placeholder)
- Lane D: staging read-only のみ、forbidden 操作 0、D31/D32 review file 分離維持

## §9 safety final

- production md5: b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓
- develop md5:    0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓
- staging md5:    6cb3885cd9c90bd6fdabb127c1cd0d17 → 不変 ✓
- F282 plist (weekly-snapshot): size=1772 / mtime=1778593597 → 不変 ✓
- 自動発注 0 / 楽天 0 / iSPEED 自動操作 0 / Computer Use 0
- LINE 送信 0 / token 0 / API 0 / DB write 0
- production / develop DB 接続 0 / staging read-only URI mode=ro のみ
- git commit 0 / git push 0 / --no-verify 0
- launchctl / plist / cron 変更 0
- workflow 変更 0
- sudo / rm -rf 0
- 本 wave write: 1 file (= 本 design doc) のみ
- smoke 出力 = /tmp 配下のみ (= 一時 artifact)

## §10 次 wave 提案

### Priority A
1. AFTER-R1 / morning advisory runner 側で v1_4_1 namespace consumer 実装
   (= candidate_role/standard_lot_ok/liquidity_status を読んで entry/watch/excluded を実行)
2. paper PnL preview runner で v1_4_1 fields back-fill (= signal_success/entry_success を
   review から runner output へ反映する pipeline)

### Priority B
3. post-open phase 動的更新 wrapper (= chase_limit_exceeded / no_new_chase /
   entry_priority を actual flow ベースで更新)
4. D33 pilot で v1.4.1 runner output を実 advisory に反映、Fujiwara 確認
5. D34/D35 demote sim/本実行で recently_seen_codes 連携実検証

### Priority C
6. W5 集約 (= D20-D33 14 day 総括、v1.4.1 framework 演化記録)
7. F111 / advisory コードの git commit + push (= HQ approve 後)

## §11 まとめ

F111 runner v1.4.1 正式実装の **smoke test 完了**。
candidate output に v1_4_1 namespace 21 fields が過不足なく出力され、
D31 (= 331A0/4389/4317 excluded) + D32 (= 9247/9628 entry, 4404 watch) の
代表ケースが正しく分類される。risk_yen 非加点・100 株標準・非 100 株調整禁止が
runner レベルで再現可能になった。
Codex 4 lane 全 YES、CRITICAL/HIGH 0、safety 違反 0、tests 124 PASS。
