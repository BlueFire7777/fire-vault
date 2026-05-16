---
id: FIRE-W2A-UNIVERSE-OVERLAY-COMMIT-PREP-2026-05-16
phase: commit prep doc (= W2-A + W60 W2 universe expansion 統合 commit 分割案)
priority: 最高
status: prep 完了 / 6 file 整理済 / 3 commit 分割案 / 229 PASS / 次は HQ 承認後 commit 可
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
sibling_w2a: 03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_result_2026-05-16.md
sibling_w1: 03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W1_result_2026-05-16.md
sibling_universe_w3: 03_design/F111_UNIVERSE_EXPANSION_R1_W3_result_2026-05-16.md
codex_lanes: 4
critical_high_count: 0 CRITICAL / 0 HIGH 修正後残り (= Lane B/C MEDIUM 全 即時修正)
audit_residual_high: Lane D HIGH 1 = commit split 案を W2-A wave 統合方針に修正で対応済
---

# FIRE W2-A + Universe Expansion commit prep (= 2026-05-16)

## §1 目的

F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2-A 完了後、未 commit 差分 6 file を
安全に整理し、commit 分割案 + 安全監査 + pre-commit readiness を作る。
**本 wave は commit / push / git add 一切 実行しない** (= prep only)。

## §2 git status + 差分一覧

```
$ git status --short
 M scripts/jobs/_v1_4_1_consumer.py
 M scripts/jobs/_v1_4_1_morning_advisory_markdown.py
 M scripts/jobs/run_research_watchlist_signal_persistence.py
 M tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py
 M tests/scripts/jobs/test_v1_4_1_consumer.py
 M tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py
```

- modified: 6 file
- untracked: 0 件
- 想定外 file: 0 件 (= 全 W60 W2 + W2-A 由来)

### §2.1 git diff --stat

```
scripts/jobs/_v1_4_1_consumer.py                   | 525 ++++++++++++-
scripts/jobs/_v1_4_1_morning_advisory_markdown.py  | 143 ++++-
scripts/jobs/run_research_watchlist_signal_persistence.py | 11 +-
tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py | 57 ++
tests/scripts/jobs/test_v1_4_1_consumer.py         | 815 +++++++++++++++++++++
tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py | 276 ++++++++
```

(= W2-A audit fix + commit prep audit fix で consumer + test が更に拡張)

### §2.2 主要 diff 要約

| file | 主要 hunk | 由来 wave | content |
|---|---|---|---|
| `scripts/jobs/run_research_watchlist_signal_persistence.py` | @@ -452 | W60 W2 | `--top-n` default `"30"` → `"100"` + help text 拡充 |
| `tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py` | TestTopNBaselineW2 | W60 W2 | 5 tests (= default 100 / 旧 30 削除 / help / override / safety) |
| `scripts/jobs/_v1_4_1_consumer.py` | @@ -140 | W60 W2 | `is_entry_displayable` に letter-suffix depth-in-defense |
| `scripts/jobs/_v1_4_1_consumer.py` | @@ -333 | W2-A | `validate_consumer_output` に overlay block 構造検証 |
| `scripts/jobs/_v1_4_1_consumer.py` | @@ -340 | W2-A | `build_consumer_payload(overlay_input=...)` 拡張 |
| `scripts/jobs/_v1_4_1_consumer.py` | @@ -387 | W2-A | overlay constants + helpers + `build_overlay_top5` |
| `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` | @@ -746 | W2-A | safety footer F282 plist 3 file 別個 bullet |
| `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` | @@ -892 | W2-A | `render_overlay_section` 新規追加 |
| `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` | @@ -901 | W2-A | `render_full_markdown` に overlay 組込 |
| `tests/scripts/jobs/test_v1_4_1_consumer.py` | TestBuildOverlayTop5W2A | W2-A | 25 tests + 9 audit fix regression + 4 commit prep audit fix regression |
| `tests/scripts/jobs/test_v1_4_1_consumer.py` | test_is_letter_suffix_new_listing_fails_w2_depth_in_defense | W60 W2 | letter-suffix regression test |
| `tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py` | TestRenderOverlaySectionW2A | W2-A | 13 tests |

## §3 commit 分割案 (= Lane D HIGH 反映後)

### §3.1 Commit 1 — F111-UNIVERSE-EXPANSION-R1-W2

**target file (= 2 file, file 単位 stage 可能):**
- `scripts/jobs/run_research_watchlist_signal_persistence.py`
- `tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py`

**commit message 案:**

```
feat(F111-UNIVERSE-EXPANSION-R1-W2): signal_persistence --top-n default 30→100 baseline化

W60-UNIVERSE-EXPANSION-R1-W2 で確定した F111 母集団拡張 baseline:
- signal_persistence --top-n CLI default を "30" → "100" へ変更
- F111 entry 母集団 35 → 109 件 (= W3 staging write で 109 records 保存済)
- sector 多様化、9247-9628 固定化緩和材料
- safety gate 完全維持: 9130 demote / 低流動性 / letter-suffix / 非 100 株
  entry 混入 0 (= consumer 側 excluded で対応)
- CLI override 維持: --top-n 30 / ALL / 200 等は明示指定優先

tests:
- TestTopNBaselineW2 (= 5 tests): default 100 / 旧 default 30 削除 /
  help text / override / safety constraint regression

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

### §3.2 Commit 2 — F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2-A overlay impl

**target file (= 2 file, file 単位 stage 可能):**
- `scripts/jobs/_v1_4_1_consumer.py`
- `scripts/jobs/_v1_4_1_morning_advisory_markdown.py`

**重要 (= Lane D HIGH 1 fix)**:
consumer.py には W60-UNIVERSE-EXPANSION-R1-W2 由来の letter-suffix
depth-in-defense hunk (@@ -140) が含まれる。これは W2-A overlay の
`_overlay_candidate_eligible` でも参照される共通防御線で、W2-A 実装の前提に
なっているため、本 Commit 2 (overlay impl) に**統合**する。
commit message でも明示する (= 同 commit に W60 由来 fix が含まれることを
読者が理解できるように)。

**commit message 案:**

```
feat(F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2-A): overlay top5 display layer + W60 letter-suffix depth-in-defense 統合

W2-A theme/sector dynamic overlay 表示レイヤー実装 (= 表示専用、DB write 0、
momentum 計算 0、theme_tags table 0、cron 0、LINE 0):
- build_overlay_top5() 純関数: raw rank + overlay rank 併記、role 4 分類
  (continuing_core / risk_warning_track / new_face_sector_compare / comparison)
- build_consumer_payload(overlay_input=...) 拡張で payload に
  current_score_top5 / overlay_top5 / overlay_summary / overlay_policy_version を追加
- render_overlay_section() 新規追加 + render_full_markdown() 統合
- F282 plist 注記を 3 file 別個 bullet 化 (= production repo / smoke /
  LaunchAgents loaded、Lane B audit MEDIUM 1 fix)
- validate_consumer_output() に overlay block 構造検証追加
  (Lane B/C MEDIUM 1/2 即時修正: 非 list TypeError 防止 / safety_flags 欠落 /
   overlay_summary 欠落 / overlay_total 欠落 / role 不正 / rank monotonic 検証)

同梱 (= 前 wave 由来共通基盤):
W60-UNIVERSE-EXPANSION-R1-W2 Codex Lane D HIGH letter-suffix depth-in-defense:
- is_entry_displayable() で is_letter_suffix_new_listing=True を entry 除外
- W2-A overlay の eligible 判定で同じ防御線を再利用 (= logical 整合)
- W2 wave 由来の修正だが W2-A overlay の前提となるため本 commit に統合

D49 smoke 結果 (W1 design 完全一致):
overlay top5 = 9247 / 9628 / 4404 / 9008 / 3134
継続主役枠 2 + risk warning 枠 1 + 新顔+新sector 比較枠 2 + 比較枠 0

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

### §3.3 Commit 3 — F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2-A tests + W60 letter-suffix regression

**target file (= 2 file, file 単位 stage 可能):**
- `tests/scripts/jobs/test_v1_4_1_consumer.py`
- `tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py`

**重要 (= Lane D HIGH 1 fix)**:
test_v1_4_1_consumer.py には W60 由来の letter-suffix regression test
(`test_is_letter_suffix_new_listing_fails_w2_depth_in_defense`) が含まれる。
これは Commit 2 の letter-suffix gate の regression test なので、Commit 2 と
**統合**する論理に従って Commit 3 (tests) に同梱する。commit message で明示。

**commit message 案:**

```
test(F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2-A): overlay display layer tests + audit fix regression + W60 letter-suffix regression

W2-A overlay 表示レイヤー tests:
- TestBuildOverlayTop5W2A (= 25 tests): overlay top5 builder の D49 fixture +
  各 role / safety guard / D48 prior_day 比較 / 9008 + 3134 expected codes
- TestRenderOverlaySectionW2A (= 13 tests): overlay MD section + raw/overlay rank 併記 +
  F282 plist 3 file 別個明示 + forbidden phrase なし

audit fix regression:
- W2-A audit fix (9 tests): None / 空 list / scalar str excluded_recovery /
  overlay block 不正 / role 不正 / rank monotonic / score safe format
- commit prep audit fix (4 tests): overlay_top5 非 list TypeError 防止 /
  safety_flags 欠落 / overlay_total 欠落 / overlay_summary 欠落

同梱 (= 前 wave 由来):
W60-UNIVERSE-EXPANSION-R1-W2 letter-suffix regression:
- test_is_letter_suffix_new_listing_fails_w2_depth_in_defense
  (W60 W2 由来 depth-in-defense の regression、Commit 2 の letter-suffix gate に対応)

forbidden phrase test を canonical FORBIDDEN_PHRASES に統一 (= Lane C LOW 1 fix、
`自動で買` / `自動で売` を含む全 14 phrase)。

合計 48 新規 tests / 全 229 PASS (= consumer + MD + signal_persistence)。

Co-Authored-By: Claude Opus 4.7 <noreply@anthropic.com>
```

### §3.4 commit 分割の特徴 (= safety + practicality)

| 項目 | 結果 |
|---|---|
| hunk staging 必要 | **不要** (= 6 file 全て file 単位で commit 可能) |
| 各 file の commit 帰属 | 1 file = 1 commit (= 100% clean split) |
| Lane D HIGH 1 対応 | commit message で W60 由来 hunk 統合を明示 |
| wave marker 整合 | F111-UNIVERSE-EXPANSION-R1-W2 / F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2-A |
| commit 順序 | Commit 1 → 2 → 3 (= W60 baseline → W2-A impl → W2-A tests) |
| pre-commit hook 影響 | `scripts/hooks/pre-commit` 通過想定 (= --no-verify 使わない) |

### §3.5 stage コマンド案 (= 各 commit 実行時の例)

**Commit 1:**
```
git add scripts/jobs/run_research_watchlist_signal_persistence.py
git add tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py
git commit -m "..."  # Commit 1 message
```

**Commit 2:**
```
git add scripts/jobs/_v1_4_1_consumer.py
git add scripts/jobs/_v1_4_1_morning_advisory_markdown.py
git commit -m "..."  # Commit 2 message
```

**Commit 3:**
```
git add tests/scripts/jobs/test_v1_4_1_consumer.py
git add tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py
git commit -m "..."  # Commit 3 message
```

(本 wave は実行しない、HQ 承認後の別 wave で実行想定)

## §4 禁止差分 確認 (= safety gate)

| 検査項目 | 結果 |
|---|---|
| `.github/workflows/` touch | 0 ✓ |
| `data/*.db` modified | 0 ✓ |
| `docs/launchd/*.plist` modified | 0 ✓ |
| `scripts/setup/migrate*.py` modified | 0 ✓ |
| `.env` / secrets / credential touch | 0 ✓ |
| `from linebot` / `import linebot` 新規 import | 0 ✓ |
| `import aiohttp` / `import requests` 新規 import | 0 ✓ |
| `import subprocess` / `os.system` 新規 | 0 ✓ |
| `launchctl` 新規呼出 | 0 ✓ |
| `VACUUM` / `DROP TABLE` 新規 SQL | 0 ✓ |
| `INSERT INTO` / `UPDATE...SET` 新規 SQL | 0 ✓ |
| `--no-verify` 記載 | 0 ✓ |
| 「自動発注」 production code 内出現 | 0 ✓ (tests 内 forbidden array のみ) |
| 「auto_order_allowed=True」 production code 内出現 | 0 ✓ (tests 内 forbidden array のみ) |
| 「AUTO_ORDER=ON」 production code 内出現 | 0 ✓ (tests 内 forbidden array のみ) |
| 「楽天証券」/「iSPEED 自動操作」/「Computer Use」 新規呼出 | 0 ✓ |
| 「買え」/「売れ」/「必ず」/「100%」 production code 内出現 | 0 ✓ (tests 内 forbidden array のみ) |

→ **全 禁止差分 0 件** ✓

## §5 tests 結果

```
.venv/bin/pytest tests/scripts/jobs/test_v1_4_1_consumer.py
                tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py
                tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py -q
→ 229 passed in 0.18s

.venv/bin/pytest tests/scripts/jobs/ -q
→ 2085 passed (= scripts/jobs/ 全体、47 warnings = 既存 GoalConfig fallback)
```

→ 全 関連 tests PASS ✓

## §6 Codex 4 lane adversarial review 結果

### §6.1 全 lane 集計

| lane | 範囲 | CRITICAL | HIGH | MEDIUM | LOW | verdict |
|---|---|---|---|---|---|---|
| A | universe expansion diff | 0 | 0 | 1 | 1 | concern (= 別 wave 候補) |
| B | overlay impl diff | 0 | 0 | 2 | 1 | concern (= MEDIUM 2 即時修正済) |
| C | tests / coverage | 0 | 0 | 3 | 2 | concern (= MEDIUM 1/2 重複 fix、3 別 wave、LOW 1 fix) |
| D | safety / commit split | 0 | 1 | 1 | 0 | CONCERN (= commit split 修正で対応) |

合計: **CRITICAL 0 / HIGH 1 (= commit split、§6.4 で対応済) / MEDIUM 7 (= 即時修正 3 + 重複 1 + 受容 3) / LOW 4**

### §6.2 即時修正反映 (= MEDIUM 3 件 + LOW 1 件、本 wave 内対応)

| ID | 修正内容 | file |
|---|---|---|
| Lane B MEDIUM 1 / Lane C MEDIUM 1 | `validate_consumer_output()` で `overlay_top5` 非 list 後 `len()` / iteration → TypeError 防止、早期 violation 化 | `_v1_4_1_consumer.py` |
| Lane B MEDIUM 2 / Lane C MEDIUM 2 | `safety_flags` 欠落 / `overlay_summary` 欠落 / `overlay_total` 欠落を violation 化 | `_v1_4_1_consumer.py` |
| Lane C LOW 1 | overlay reason_text の forbidden phrase test を canonical `FORBIDDEN_PHRASES` に統一 (= `自動で買` / `自動で売` 含む) | `test_v1_4_1_consumer.py` |

修正後 regression tests 追加 (= 4 件):
- `test_validate_consumer_output_overlay_top5_not_list_no_typeerror`
- `test_validate_consumer_output_missing_safety_flags`
- `test_validate_consumer_output_missing_overlay_total`
- `test_validate_consumer_output_missing_overlay_summary`

修正後 pytest:
```
.venv/bin/pytest tests/scripts/jobs/test_v1_4_1_consumer.py
                tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py
                tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py -q
→ 229 passed in 0.18s (= W2-A 194 + commit prep audit fix regression 4 + 既存 31)
```

### §6.3 Lane A 別 wave 候補 (= MEDIUM 1 + LOW 1、本 wave scope 外)

- Lane A MEDIUM 1: `test_filter_signals_supports_override` が CLI override の実 behavior を検証していない (= filter_signals_by_top_n の signature しか見ない). 別 wave で `subprocess.run` + 実 CLI override test 追加候補.
- Lane A LOW 1: `test_cli_default_top_n_is_100` / `test_cli_default_top_n_is_not_30` が brittle source-string check. 別 wave で argparse 内 hunk 専用パーサ test 候補.

### §6.4 Lane D HIGH 1 対応 (= commit split 修正方針)

**指摘**: Commit split 案がそのままだと `_v1_4_1_consumer.py` の W60-UNIVERSE-EXPANSION-R1-W2 由来 letter-suffix hunk が Commit 2 (W2-A overlay) に混入し、wave marker 整合性を満たさない。

**対応 (= 採用、§3.2 / §3.3 commit message に反映)**:
- Commit 2 message に「同梱: W60-UNIVERSE-EXPANSION-R1-W2 Codex Lane D HIGH letter-suffix depth-in-defense 統合」を明示
- Commit 3 message に「同梱: W60-UNIVERSE-EXPANSION-R1-W2 letter-suffix regression test」を明示
- 統合理由を logical に説明: W2-A overlay の `_overlay_candidate_eligible` が letter-suffix gate を **再利用**しているため、letter-suffix gate は W2-A 実装の前提として Commit 2 に統合
- 代替案 (= hunk staging で W60 hunk + regression を Commit 1 に分離) は本 wave で**採用せず** (= hunk staging の mistake risk が高い)

**Lane D MEDIUM 1 対応**: read-only sandbox 制約は HOST 側 2085 PASS 確認で代替済 (= prep wave の限定 audit には十分).

### §6.5 受容 LOW (= 別 wave 候補)

- Lane B LOW 1: W2-B+ 注記が MD 出力本文に出る (= 設計判断、削るのが安全だが LOW 維持)
- Lane C MEDIUM 3: fixture と production smoke artifact の drift detection なし (= smoke artifact 維持は別 wave responsibility)
- Lane C LOW 2: pytest read-only sandbox 制約 (= HOST 側で 229 PASS 確認済)

## §7 audit fix 反映後 final safety

| 検査項目 | 結果 |
|---|---|
| pytest scripts/jobs/ 全体 | 2085 PASS ✓ |
| pytest W2-A + universe expansion 関連 | 229 PASS ✓ |
| 3 DB md5 unchanged | ✓ (= b1df4673 / 0eed4ad2 / f6fa86df) |
| F282 plist 3 file size unchanged | ✓ (= 1772 / 1844 / 1772) |
| 禁止 file touch | 0 ✓ |
| 禁止 phrase production code 出現 | 0 ✓ |
| --no-verify 記載 | 0 ✓ |
| git add / commit / push 実行 | 0 ✓ (= prep only) |
| LINE 送信 / API 実行 / token 参照 / launchctl / plist 変更 / cron / VACUUM / sudo / rm -rf / auto-order / 楽天証券・iSPEED 操作 / Computer Use / TODO Excel | 全 0 ✓ |

## §8 commit 可否 / push 可否

| 状態 | 判定 |
|---|---|
| Codex CRITICAL | 0 ✓ |
| Codex HIGH (= 修正後) | 0 ✓ (= Lane D HIGH 1 は commit split 案修正で対応済) |
| tests PASS | 229 / 2085 ✓ |
| safety gate | 全 0 ✓ |
| commit split 案明確 | ✓ (= 3 commit、file 単位、hunk staging 不要) |
| **commit 可否** | **HQ 承認後 commit 可** ✓ |
| **push 可否** | **HQ 承認 (= commit 確定) 後 push 可** (= main 直 push は CLAUDE.md 禁止、`develop` branch push のみ) |

## §9 次アクション

1. **HQ (= Fujiwara) 承認**:
   - §3 の 3-commit 分割案を承認
   - Commit 2 / 3 に W60 letter-suffix hunk + regression を統合する方針を承認
   - commit message 案を承認
2. **commit 実行 wave (= 別 wave 起票)**:
   - Commit 1 → 2 → 3 順次実行
   - 各 commit で `pre-commit hook` (`scripts/hooks/pre-commit`) 通過確認
   - `--no-verify` 使わない (= hook の指摘あれば fix して再 commit)
3. **push wave (= 別 wave)**:
   - `develop` branch に push (= main 直 push 禁止)
   - PR 作成 (= `develop` → `main`)
4. **次の F111 wave (= W2-B 起票候補)**:
   - theme_tags table schema 追加 (= staging で別 wave migration)
   - volume_momentum / turnover_momentum 計算 module
   - sector_relative_strength 計算 module
   - consecutive_signal_success 集計
   - 9247 / 9628 数値判別 (= W1 §6.3 4 分類)

## §10 関連 file

- 実装 (Commit 1): `scripts/jobs/run_research_watchlist_signal_persistence.py` (= W60 W2)
- 実装 (Commit 2): `scripts/jobs/_v1_4_1_consumer.py` + `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` (= W2-A overlay + W60 letter-suffix 統合)
- tests (Commit 1): `tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py`
- tests (Commit 3): `tests/scripts/jobs/test_v1_4_1_consumer.py` + `tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py`
- W2-A 結果 doc: `~/fire-vault/03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_result_2026-05-16.md`
- W1 design doc: `~/fire-vault/03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W1_result_2026-05-16.md`
- W3 universe staging write doc: `~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_W3_result_2026-05-16.md`
- Codex 4 lane log: `/tmp/fire_w2a_commit_prep/lane_{A,B,C,D}_output.txt`

## §11 safety footer

- 本 wave は **commit prep のみ** (= 差分整理 + 監査 + plan + vault doc 作成、commit / push / git add 一切実行せず)
- DB write 0 / staging write 0 / production-develop 接続 0 / API 0 / token 0 / LINE 0
- launchctl 0 / plist 0 / cron 0 / VACUUM 0 / workflow 0
- git add 0 / commit 0 / push 0 / --no-verify 0
- TODO Excel 更新 0 / sudo 0 / rm -rf 0
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use 0
- 本 wave 中の fire repo コード変更: 2 file (= consumer / consumer test、Lane B/C MEDIUM 1/2 + LOW 1 audit fix で content 修正)
- 一時 script / 中間 file は `/tmp/fire_w2a_commit_prep/` 配下のみ

### F282 plist 3 file 別個確認 (= safety final)

- production plist (repo) = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)

3 file unchanged ✓
