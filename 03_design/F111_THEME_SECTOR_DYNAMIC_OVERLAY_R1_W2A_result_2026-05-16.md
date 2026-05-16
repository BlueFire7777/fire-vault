---
id: FIRE-F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2A-RESULT-2026-05-16
phase: 実装完了 doc (= W2-A 表示レイヤー consumer + MD + tests + smoke)
priority: 最高
status: implementation 完了 / 41 new tests PASS / D49 smoke = 9247-9628-4404-9008-3134
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
parent_design: 03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_2026-05-16.md
sibling_w1: 03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W1_result_2026-05-16.md
sibling_universe_w3: 03_design/F111_UNIVERSE_EXPANSION_R1_W3_result_2026-05-16.md
d49_morning_summary: 04_daily/2026-07-22_morning_advisory_summary.md
codex_lanes: 4
critical_high_count: 0 CRITICAL / 0 HIGH (= W2-A 4 file scope 内、Lane A/B/C MEDIUM 4 即時修正済)
audit_residual_high: Lane D HIGH 1 = W60 前 wave (W2/W3 universe expansion) 由来 uncommitted file 2 件、W2-A wave touch 0
---

# F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2-A 実装完了 (= 2026-05-16)

## §1 目的 (= W2-A scope)

W1 read-only simulation で確認した theme/sector dynamic overlay 表示レイヤーを
**consumer payload + morning advisory Markdown** に実装する。

重要 scope 限定 (= W2-A 専用、W2-B+ 領域は出さない):
- 表示レイヤーのみ (= raw score top5 と overlay top5 を併記)
- **theme_tags table 追加なし** (= DB schema 変更 0)
- **momentum / sector_relative_strength 計算なし** (= W2-B+ 領域)
- **継続上昇 vs 惰性失速 の数値判別なし** (= W2-B+ 領域)
- DB write 0 / API 0 / LINE 0 / token 0 / launchctl 0 / cron 0

「主役排除」ではなく「**継続上昇候補 vs 惰性固定の判別**」を
朝判断 UX に組み込むための表示レイヤー (= raw rank を常に保存しつつ、
overlay rank で role 分類後の朝判断候補を提示).

## §2 実装内容

### §2.1 consumer (= `scripts/jobs/_v1_4_1_consumer.py`)

新規追加 (= 純関数、DB / API / LINE / token / file I/O 一切なし):
- `OVERLAY_POLICY_VERSION = "1.0.0-w2a-display-only"` (= W2-A 表示専用 marker)
- `OVERLAY_ROLE_CONTINUING_CORE / RISK_WARNING / NEW_FACE_SECTOR / COMPARISON` (= role 定数)
- `OVERLAY_ROLE_LABELS_JP` (= 「主役継続枠」「risk warning 枠」「新顔+新sector 比較枠」「比較枠」)
- `_to_str_set()` (= set 正規化 helper)
- `_overlay_candidate_eligible()` (= safety guard: 100株 / standard_lot_ok / liquidity / letter-suffix / no_new_chase / chase_limit / excluded_recovery)
- `_build_overlay_entry()` (= overlay top5 1 要素 dict 組み立て)
- `build_overlay_top5()` (= 純関数 main entry、外部 input で prior_day_entry_codes / sectors / excluded_recovery_codes を受ける)

`build_consumer_payload()` 拡張:
- 新規 keyword 引数 `overlay_input: Optional[dict] = None`
- 指定時、payload に
  - `overlay_policy_version`
  - `current_score_top5` (= raw rank top5、role 無し)
  - `overlay_top5` (= overlay rank top5、role / role_label_jp 付き)
  - `overlay_summary` (= counts + new_face_codes / new_sectors / excluded_recovery)
  を追加
- 未指定なら backward compat (= overlay block 不出力)

### §2.2 morning advisory MD (= `scripts/jobs/_v1_4_1_morning_advisory_markdown.py`)

新規追加:
- `render_overlay_section(payload) -> str` (= overlay 表示 section の Markdown)
  - raw score top5 table (= raw rank / code / name / sector / score / risk / warning)
  - overlay top5 table (= overlay rank / raw rank / code / name / sector / role / 新顔 / 新 sector / warning / 選定理由)
  - overlay の見方 (= 4 role 説明 + W2-B+ 範囲外注記)
  - 主役継続候補 vs 惰性固定の判別 (= W2-A 表示意図 + 数値判別は W2-B+)
- overlay block 不在時は空文字列を返す (= backward compat)

`render_full_markdown()` 拡張:
- `render_overlay_section(payload)` を judgment と entry section の間に挿入
- payload に overlay block があれば組み込み、なければ skip

### §2.3 overlay 選定 logic 詳細

```
入力: entry_candidates (= consumer payload sections.entry.candidates)
     prior_day_entry_codes (= 新顔判定基準 set)
     prior_day_entry_sectors (= 新 sector 判定基準 set)
     excluded_recovery_codes (= 9130 等の復帰禁止 set)

Step 1: raw rank 計算 (= research_final_score 降順、同点は code 昇順 deterministic)
Step 2: 各候補に is_new_face / is_new_sector / eligible flag 付与
Step 3: 枠分配 (= 順):
  (a) 主役継続枠 (continuing_core): raw rank 上位、最大 max_continuing_core=2
  (b) risk warning 枠 (risk_warning_track): risk_yen_over_pilot_budget=True 最上位、最大 max_risk_warning=1
  (c) 新顔+新sector 比較枠 (new_face_sector_compare): is_new_face AND is_new_sector の最上位、sector diversity 確保、最低 min_new_face_sector_compare=1
  (d) 比較枠 (comparison): 残り、raw rank 順 fill
Step 4: overlay_summary 集計 (= counts + new_face_codes / new_sectors / excluded_recovery 記録)
出力: dict(overlay_policy_version, current_score_top5, overlay_top5, overlay_summary)
```

## §3 D49 smoke 結果 (= /tmp/fire_theme_sector_overlay_w2a/)

入力:
- F111 raw output: `/tmp/fire_d49_prep/d49_f111.json` (= base_date 2026-07-22 / max=50 / policy=1.4.2)
- prior_day_entry_codes (= D48 推定 entry codes 14 件、W1 design 通り)
- prior_day_entry_sectors (= D48 推定 entry sectors 7 種、W1 design 通り)
- excluded_recovery_codes = `{"9130"}`

出力:
- `/tmp/fire_theme_sector_overlay_w2a/overlay_consumer_payload.json`
- `/tmp/fire_theme_sector_overlay_w2a/overlay_morning_advisory.md`

### §3.1 current raw score top5 (= 既存ロジック reference)

| raw | code | 銘柄 | sector | score | risk | warning |
|---|---|---|---|---|---|---|
| 1 | 9247 | ＴＲＥ HD | 情報通信 | 0.8546 | 8,060 | |
| 2 | 9628 | 燦 HD | 情報通信 | 0.8479 | 6,950 | |
| 3 | 4404 | ミヨシ油脂 | 食品 | 0.8413 | 10,665 | ⚠ |
| 4 | 2146 | ＵＴグループ | 情報通信 | 0.8305 | 920 | |
| 5 | 4828 | ビジネスエンジニアリング | 情報通信 | 0.8293 | 6,230 | |

### §3.2 overlay top5 (= role 分類後の朝判断候補)

| overlay | raw | code | 銘柄 | sector | role | 新顔 | 新sec | warning |
|---|---|---|---|---|---|---|---|---|
| 1 | 1 | 9247 | ＴＲＥ HD | 情報通信 | 主役継続枠 | | | |
| 2 | 2 | 9628 | 燦 HD | 情報通信 | 主役継続枠 | | | |
| 3 | 3 | 4404 | ミヨシ油脂 | 食品 | risk warning 枠 | | | ⚠ |
| 4 | 16 | 9008 | 京王電鉄 | 運輸・物流 | 新顔+新sector 比較枠 | ✓ | ✓ | |
| 5 | 19 | 3134 | Ｈａｍｅｅ | 小売 | 新顔+新sector 比較枠 | ✓ | ✓ | |

W1 design 完全一致 ✓

### §3.3 overlay_summary

| 項目 | 値 |
|---|---|
| total_entry | 20 |
| eligible_count | 20 |
| continuing_core_count | 2 |
| risk_warning_count | 1 |
| new_face_sector_compare_count | 2 |
| comparison_count | 0 |
| overlay_total | 5 |
| new_face_codes_in_entry | 3134, 3962, 6196, 7595, 9008, 9417 |
| new_sectors_in_entry | 小売, 運輸・物流 |
| excluded_recovery_codes_active | 9130 |
| prior_day_entry_codes_count | 14 |
| prior_day_entry_sectors_count | 7 |

### §3.4 validation

| check | 結果 |
|---|---|
| consumer.validation_passed | True ✓ |
| consumer.validation_violations | 0 件 ✓ |
| md.validation_violations | 0 件 ✓ |
| safety_flags 全 False | ✓ |

## §4 tests 結果

新規追加 tests:
- `tests/scripts/jobs/test_v1_4_1_consumer.py::TestBuildOverlayTop5W2A` (= 22 tests)
- `tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py::TestRenderOverlaySectionW2A` (= 12 tests)

tests カバー範囲:
1. overlay_policy_version W2-A marker
2. overlay top5 件数 (= 5 件)
3. current_score_top5 raw rank 1-5 / score 降順 / role 無し
4. D49 fixture で overlay top5 = 9247/9628/4404/9008/3134
5. overlay rank + raw rank 併記
6. 主役継続枠 = raw rank 1, 2
7. risk warning 枠 = warning=True 最上位 4404
8. 新顔+新sector 枠 ≥ 1 件 + 各候補 is_new_face=True かつ is_new_sector=True
9. D49 fixture で 新顔+新sector codes = 9008, 3134
10. role_label_jp 4 種 (主役継続枠 / risk warning 枠 / 新顔+新sector 比較枠 / 比較枠)
11. overlay 各要素に必須 fields 14 種完備
12. 9130 復帰禁止 (= excluded_recovery_codes)
13. 低流動性 (liquidity_status=FAIL) excluded
14. letter-suffix excluded
15. 非 100 株 (share_unit != 100 / standard_lot_ok=False) excluded
16. risk_yen 順位加点 0 / entry 除外 gate 0 (= warning は表示のみ)
17. overlay_summary counts (= 20/20/2/1/2/0/5)
18. deterministic sort (= 同 score は code 昇順)
19. consumer_payload に overlay block 追加 (overlay_input 指定時)
20. consumer_payload に overlay block 不追加 (overlay_input None 時) = backward compat
21. prior_day 未指定時 is_new_face / is_new_sector 全 False
22. reason_text に forbidden phrase なし

MD section tests:
23. overlay section 描画 (block 有時)
24. overlay section 空 (block 無時) = backward compat
25. raw score top5 table 描画
26. overlay top5 table に overlay rank + raw rank 両方
27. role label 日本語 4 種
28. overlay の見方 section 描画 + W2-B+ 範囲外注記
29. forbidden phrase なし (= 買え/売れ/必ず/100%/保証/自動発注/自動売買/auto_order_allowed=True)
30. overlay_summary counts 描画
31. excluded_recovery_codes_active 描画
32. render_full_markdown に overlay section 組み込み
33. render_full_markdown overlay 不在時 backward compat
34. validate_markdown_output 既存 invariant 全 PASS

```
.venv/bin/pytest tests/scripts/jobs/test_v1_4_1_consumer.py
                tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py -q
→ 185 passed in 0.18s (= W2-A 新規 34 + 既存 151)
```

## §5 safety gate 確認

| gate | 結果 |
|---|---|
| DB write | 0 ✓ (= write 0、build_overlay_top5 + render_overlay_section は純関数) |
| staging write | 0 ✓ |
| production / develop DB 接続 | 0 ✓ |
| schema migration | 0 ✓ |
| theme_tags table 追加 | 0 ✓ (= W2-B+ 領域) |
| LINE 送信 | 0 ✓ |
| API 実行 | 0 ✓ |
| token / secret / env 参照 | 0 ✓ |
| launchctl | 0 ✓ |
| plist 変更 | 0 ✓ |
| cron 登録 | 0 ✓ |
| VACUUM | 0 ✓ |
| workflow 変更 | 0 ✓ |
| git add / commit / push | 0 ✓ |
| `--no-verify` 使用 | 0 ✓ |
| sudo / rm -rf | 0 ✓ |
| auto-order / 楽天証券 / iSPEED 自動操作 | 0 ✓ |
| Computer Use | 0 ✓ |
| TODO Excel 更新 | 0 ✓ |
| forbidden phrase grep on output | 0 ✓ (= 60/30/50 株、買え/売れ/必ず/100%/保証/自動発注/自動売買) |
| 9130 復帰 | 0 ✓ (= excluded_recovery_codes={9130} で overlay 不出を確認) |
| 9247 / 9628 demote 本実行 | 0 ✓ (= 継続主役枠 1, 2 維持) |
| raw score 順位保存 | ✓ (= overlay rank と併記、current_score_top5 + overlay_top5[].raw_rank 全 attach) |

## §6 3 DB md5 invariant (= production / develop / staging)

| DB | md5 (本実装後) | 前回値 (= W3 後) | 一致 |
|---|---|---|---|
| production (`data/fire.db`) | b1df4673e5c3645fbe2c5f490ffac043 | b1df4673e5c3645fbe2c5f490ffac043 | ✓ unchanged |
| develop (`data/fire.develop.db`) | 0eed4ad2ec7ed2edf8f640d97341c5ad | 0eed4ad2ec7ed2edf8f640d97341c5ad | ✓ unchanged |
| staging (`data/fire.staging.db`) | f6fa86df9c22d3e5866c7d3f380286d5 | f6fa86df9c22d3e5866c7d3f380286d5 | ✓ unchanged |

3 DB すべて touch 0 (= W2-A は純関数表示レイヤー、DB 触らない).

## §7 F282 plist 3 files 別個確認

- **production plist** (= 本番 launchctl 対象): `docs/launchd/jp.fire.weekly-snapshot.plist`
  - size=1772 / mtime=1778592445
- **smoke plist** (= test 用、launchctl 未投入): `docs/launchd/jp.fire.weekly-snapshot-smoke.plist`
  - size=1844 / mtime=1778769277
- **LaunchAgents loaded plist** (= ユーザー直下): `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist`
  - size=1772 / mtime=1778593597

3 file unchanged (= W2-A で plist touch 0).

## §8 W2-B 切り分け (= 本 wave 範囲外、別 wave 起票候補)

W2-A は **表示レイヤーのみ**。下記は **W2-B+ で実装**:

| W2-B+ 領域 | 説明 |
|---|---|
| theme_tags table 追加 | seed list (半導体/AI/電線/防衛/インバウンド 等) を保持する DB schema |
| volume_momentum 計算 | 直近 N 日平均出来高 / 過去 N 日 (= 継続上昇 vs 失速判別) |
| turnover_momentum 計算 | 売買代金 momentum (= 規模考慮) |
| sector_relative_strength 計算 | sector 相対強度 (= 主役 sector vs 衰退 sector 判別) |
| consecutive_signal_success 集計 | 連続日数別 +2R hit rate (= 主役継続 vs 惰性失速の実績) |
| 9247 / 9628 数値判別 | momentum > 0 + 相対強度 > 0 → 主役継続 / 両 < 0 → 惰性固定注意 (= W1 §6.3 4 分類) |
| migration script | theme_tags + momentum テーブル idempotent CREATE TABLE IF NOT EXISTS |

W2-A 実装範囲は厳密に守られている (= 上記 7 項目すべて W2-A code に含まれていない).

## §9 関連 file

- 実装 (= consumer): `scripts/jobs/_v1_4_1_consumer.py` (= build_overlay_top5 追加)
- 実装 (= MD): `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` (= render_overlay_section 追加)
- tests (= consumer): `tests/scripts/jobs/test_v1_4_1_consumer.py::TestBuildOverlayTop5W2A`
- tests (= MD): `tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py::TestRenderOverlaySectionW2A`
- smoke output: `/tmp/fire_theme_sector_overlay_w2a/`
  - `overlay_consumer_payload.json`
  - `overlay_morning_advisory.md`
- sibling W1: `~/fire-vault/03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W1_result_2026-05-16.md`
- sibling universe W3: `~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_W3_result_2026-05-16.md`

## §10 D50 handoff

- 朝 pilot top5 は overlay 案で運用検討可 (= 9247/9628 主役継続 + 4404 warning + 9008/3134 新顔/新sector)
- 朝判断 UX は overlay rank 順で見つつ raw rank も併記され、Fujiwara が iSPEED で板 / 出来高 / spread を見てから entry 判断
- W2-B 起票候補 (= HQ approve 後):
  - theme_tags table schema 追加 (= staging で別 wave migration、9 章 §8 参照)
  - volume_momentum / turnover_momentum / sector_relative_strength 計算 module
  - consecutive_signal_success 集計 (= 主役継続 vs 惰性失速の実績判別)
  - 9247 / 9628 数値判別 (= W1 §6.3 4 分類)

## §11 Codex 4 lane adversarial audit 結果

### §11.1 全 lane 集計

| lane | 範囲 | CRITICAL | HIGH | MEDIUM | LOW | verdict |
|---|---|---|---|---|---|---|
| A | overlay builder logic | 0 | 0 | 2 | 3 | concern (= MEDIUM 2 + LOW 1 即時修正済) |
| B | MD 表示 / 安全文言 | 0 | 0 | 1 | 1 | concern (= MEDIUM 1 即時修正済) |
| C | tests / fixture / consumer validation | 0 | 0 | 1 | 3 | concern (= MEDIUM 1 即時修正済) |
| D | safety gate + W2-B 切り分け | 0 | 1 | 0 | 0 | REJECT (= W60 前 wave 由来、§11.3 で説明) |

合計: **CRITICAL 0 / HIGH 1 (= 前 wave 由来 説明可能) / MEDIUM 4 全 即時修正 / LOW 7**

### §11.2 即時修正反映 (= MEDIUM 4 件 + Lane A LOW 1 件、本 wave 内で対応)

| ID | 修正内容 | file:line |
|---|---|---|
| Lane A MEDIUM 1 | `build_overlay_top5(None)` で `TypeError` → 入口で `entry_candidates = list(entry_candidates or [])` | `_v1_4_1_consumer.py` |
| Lane A MEDIUM 2 | `_to_str_set("9130")` → scalar string も `{values}` 単要素 set として扱う (= 9130 復帰禁止の単一指定対応) | `_v1_4_1_consumer.py` |
| Lane A LOW 1 | `reason_text` の `:.4f` で `TypeError` 防止 → `_fmt_score_safe()` 経由に統一 | `_v1_4_1_consumer.py` |
| Lane B MEDIUM 1 | safety footer F282 plist 注記が 2 bullet → **3 file 別個 bullet** に修正 (= production repo / smoke / LaunchAgents loaded) | `_v1_4_1_morning_advisory_markdown.py` |
| Lane C MEDIUM 1 | `validate_consumer_output()` が overlay block を構造検証していない → overlay_top5 safety_flags / role / overlay_rank monotonic / overlay_total 整合性を検証追加 | `_v1_4_1_consumer.py` |

修正後 regression tests 追加:
- `test_build_overlay_top5_none_input_no_typeerror`
- `test_build_overlay_top5_empty_list_input`
- `test_to_str_set_scalar_string_kept_intact`
- `test_excluded_recovery_codes_scalar_string_works`
- `test_validate_consumer_output_detects_bad_overlay_block`
- `test_validate_consumer_output_detects_invalid_role`
- `test_validate_consumer_output_overlay_rank_monotonic`
- `test_score_missing_does_not_typeerror_in_reason_text`
- `test_safety_footer_f282_plist_3_files_separately`

修正後 pytest:
```
.venv/bin/pytest tests/scripts/jobs/test_v1_4_1_consumer.py
                tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py -q
→ 194 passed (= 185 既存 + 9 audit fix regression)
```

### §11.3 Lane D HIGH 1 の説明 (= 前 wave 由来、W2-A scope 外)

Lane D は「変更ファイルが指定 4 file に限定されていない」を HIGH として REJECT.
git status で確認される追加 2 file は:

- `scripts/jobs/run_research_watchlist_signal_persistence.py` (= `--top-n` default 30→100)
- `tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py` (= 同 baseline 化 test)

これらは **F111-UNIVERSE-EXPANSION-R1-W2 (universe top-n=100 baseline化)** wave で
実装した変更で、続く W3 wave で staging write を実施した時点で uncommitted のまま
残っている (= W3 wave は staging write のみ commit 対象を明示しなかったため).

W2-A wave (= 本 wave) は上記 2 file を **一切触っていない**:
- 本 wave の touch file は 4 file: consumer / MD / consumer test / MD test
- `git diff scripts/jobs/run_research_watchlist_signal_persistence.py` は本 wave 中 0 byte 増減
- 本 wave の追加 line は overlay_top5 builder + render_overlay_section + tests + audit fix のみ

つまり Lane D HIGH 1 は **W60 W2/W3 前 wave の uncommitted 状態が露呈したもの** で、
W2-A wave の implementation 不備ではない。次の commit wave (= W2-A + W2 + W3 uncommitted
一括 commit) で解消する想定。

### §11.4 LOW 件 (= 本 wave で対応せず、別 wave 候補)

- Lane A LOW 2 (`liquidity_status` 欠落で eligible): 仕様「FAIL 除外」前提で本 wave スコープ外
- Lane A LOW 3 (warning=True が raw rank 1-2 時の枠重複): 仕様明文化候補
- Lane C LOW 1 (`current_score_top5` が eligibility guard 前): production path は entry section 渡しのため実害限定
- Lane C LOW 2 (`test_no_9130_recovery_when_excluded` の単体 false positive): 別 test `test_excluded_recovery_codes_filter_9130` で 9130 復帰禁止 coverage あり
- Lane C LOW 3 (forbidden phrase test の hard-coded subset): regression test で `全力/確実/絶対/間違いなく/AUTO_ORDER=ON/勝手に買/勝手に売` を網羅追加済
- Lane B LOW 1 (table header の minor 差分 = `risk_yen` / `新 sector` 列名): 値・列順・意味は揃っているため LOW 維持

## §12 safety footer

- 本 wave は **実装レベル** (= consumer + MD + tests 追加、code 変更あり)
- DB write 0 / staging write 0 / production-develop 接続 0 / API 0 / token 0 / LINE 0
- launchctl 0 / plist 0 / cron 0 / VACUUM 0 / workflow 0
- git add 0 / commit 0 / push 0 / --no-verify 0
- TODO Excel 更新 0 / sudo 0 / rm -rf 0
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use 0
- 9247 / 9628 demote 本実行 0 (= 継続主役枠 1, 2 維持)
- 9130 demote 状態維持 (= excluded_recovery_codes={9130} で overlay 不出)
- fire repo コード変更: 2 file (= consumer / MD)、新規 file 0
- tests 追加: 2 file (= consumer test / MD test)、43 新規 tests / **194 全体 PASS** (= audit fix regression 9 含む)
- audit fix で本 wave 中に修正した production code 2 file (= consumer / MD) + tests 2 file (= consumer test / MD test) のみ
- W60 前 wave (= universe expansion W2/W3) 由来 uncommitted file 2 件は本 wave touch 0
- 一時 script + smoke output は /tmp/fire_theme_sector_overlay_w2a/ 配下のみ

### F282 plist 3 file 別個確認 (= safety final)

- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
