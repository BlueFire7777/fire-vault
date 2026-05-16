---
id: FIRE-F111-MAX-CANDIDATES-BASELINE-50-2026-05-16
phase: D43+ baseline 化 / F111 max_candidates default 20 → 50
priority: 最高
status: 実装済 (= CLI default + tests + smoke 完了)、HQ approve 済 (= 本 wave 指示)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
parent_design: 03_design/FIRE_f111_max_candidates_expansion_sim_2026-05-16.md
codex_lanes: 8
critical_high_count: 0
---

# FIRE F111 max_candidates 20 → 50 baseline 化 (= 2026-05-16)

## §1 概要

D42 max_candidates expansion sim (= 親 doc 参照) の結果を受け、F111 CLI default
を **20 → 50** に baseline 化. D43 以降の F111 / morning advisory の標準探索幅.

**HQ approve 済** (= 本 wave 指示):
- max=50 を D43 以降の F111 baseline として使う
- max=100/200/300 は現時点では採用しない (= 母集団 35 件頭打ち)
- 9247/9628 demote 本実行は別 wave
- 本 wave は探索幅の baseline 化、実注文・auto-order ではない

## §2 変更内容 (= 最小変更)

### §2.0 本 wave 適用範囲注記

本 §2 で「変更」と記載するのは **本 wave (= F111 max_candidates 20→50 baseline 化) で新たに加えた差分のみ** を指す.
- 作業ツリー上に存在する `.gitignore` 修正 + v1.4.1 系未追跡ファイル群 (= `_selection_policy_v1_4_1.py` / `_v1_4_1_consumer.py` 等) は **過去 wave (= D38 以前の develop branch state) からの既存差分**であり、本 wave の追加変更ではない.
- 同 file 内の `_selection_policy_v1_4_1` import 等も **過去 wave で導入済**、本 wave で touch しない.
- 本 wave での **追加** git diff: `scripts/jobs/run_f111_real_batch_staging.py` (37 insertions / 8 deletions) + `tests/scripts/jobs/test_run_f111_real_batch_staging.py` (12 insertions / 1 deletion) のみ.

### §2.1 本 wave 追加修正 file path (= 2 file のみ、git diff 上の追加分)

| file | 変更箇所 | 内容 |
|---|---|---|
| `scripts/jobs/run_f111_real_batch_staging.py` | line 43 | CLI_VERSION "1.2.0" → **"1.3.0"** + comment 更新 |
| 同上 | line 597 | `default=20` → **`default=50`** |
| 同上 | line 599-602 | help text 更新 (= W60-F111-baseline-50 + D42 sim 結果反映) |
| `tests/scripts/jobs/test_run_f111_real_batch_staging.py` | line 884-889 | `test_max_candidates_default_is_20` → **`test_max_candidates_default_is_50`** + assert 50 |
| 同上 | line 891-898 (新規) | `test_max_candidates_comparison_explicit_20_still_works` (= regression 維持 test 追加) |

### §2.2 本 wave で touch しなかった file (= 意図的に残した max=20 + 過去 wave 既存差分)

| file | 理由 |
|---|---|
| `tests/scripts/jobs/test_run_f111_real_batch_staging.py:331` (`"--max-candidates", "10"`) | unit test の小さい母集団用、明示引数なので影響無 |
| `tests/scripts/jobs/test_run_f062_research_advisory_line_preview.py:97` (`args.max_candidates == 20`) | F062 LINE preview 別 pipeline (= F111 baseline 化と無関係) |
| `~/fire-vault/04_daily/2026-05-26 〜 2026-07-10` 内の D33-D42 pilot docs | historical 記録、comparison/simulation 用途として保持 |
| `scripts/jobs/run_f111_research_advisory_preview.py` | max_candidates default=None (= 制限なし)、別 runner |
| `.gitignore` + `scripts/jobs/_v1_4_1_*.py` + 関連 tests (= 未追跡ファイル) | **過去 wave (D38 以前) からの既存差分**、本 wave とは無関係 |
| `scripts/jobs/run_f111_real_batch_staging.py` の `_selection_policy_v1_4_1` import | 過去 wave で導入済、本 wave で touch なし |

## §3 baseline 化の根拠 (= D42 expansion sim 結果)

### §3.1 比較表 (= 親 doc から要約)

| max | raw | entry | sector | 推奨 |
|---|---|---|---|---|
| 20 | 20 | 3 | 8 | **旧 baseline (= 狭い)** |
| **50** | **35** | **14** | **12** | **★ 新 baseline ★** |
| 100 | 35 | 14 | 12 | 採用せず (= 50 と同一) |
| 200 | 35 | 14 | 12 | 採用せず |
| 300 | 35 | 14 | 12 | 採用せず |

### §3.2 max=50 採用理由

1. raw candidates: 20 → 35 (+75%)
2. entry: 3 → 14 (+11 件)
3. sector: 8 → 12 (+4: 医薬品 / 素材・化学 / 電機・精密 / 電気・ガス)
4. 安全条件 8 項目全 PASS:
   - 9130 demote 維持 (= recently_seen_demoted)
   - 低流動性 entry 混入 0
   - letter-suffix entry 混入 0
   - 非 100 株調整 0
   - 9247 / 9628 / 4404 entry 維持
   - 4404 risk_yen_warning 表示維持
   - risk_yen 順位加点 / entry 除外 gate なし維持
   - v1.4.2 policy 維持

### §3.3 max=100/200/300 不採用理由

- F111 出力母集団が **35 件で頭打ち** (= staging research_watchlist_signals 制約)
- max=100/200/300 全件 max=50 と完全同一結果
- baseline 化の意味なし

## §4 D43 baseline smoke 結果

### §4.1 実行内容
- input: `--base-date 2026-07-13 --recently-seen-codes 8747,5729,3489,340A0,3798,137A0,7991,9130`
- 出力 F111: `/tmp/fire_max_candidates_baseline_50/d43_baseline_smoke_f111.json`
- 出力 consumer: `/tmp/fire_max_candidates_baseline_50/d43_baseline_smoke_consumer.json`
- mode: staging read-only (sqlite URI mode=ro)

### §4.2 結果検証 (= 安全 8 項目 + 候補品質)

| 項目 | 結果 |
|---|---|
| cli_version | **1.3.0** ✓ |
| max_candidates | **50** ✓ |
| policy_version | 1.4.2 ✓ |
| demoted_count | 8 ✓ |
| candidates | 35 ✓ |
| entry / watch / excluded | 14 / 0 / 21 ✓ |
| sector | 12 ✓ |
| 9130 excluded 維持 | ✓ (recently_seen_demoted) |
| 9130 entry 戻り | **無** ✓ |
| 4404 / 9247 / 9628 entry 維持 | ✓ |
| liquidity_fail entry 混入 | **0** ✓ |
| letter-suffix entry 混入 | **0** ✓ |
| 100 株標準 (= 全 14 entry) | True ✓ |
| 4404 risk_yen_over_pilot_budget | True ✓ |
| risk_warning entry 件数 | 4 (4404, 3479, 4540, 8057) |
| forbidden_phrase_count | 0 ✓ |
| safety_footer_present | True ✓ |

### §4.3 entry top14 (= D43 baseline で浮上する候補)

| code | 銘柄 | sector_17 | risk_yen | risk_over |
|---|---|---|---|---|
| 9247 | ＴＲＥ HD | 情報通信・サービスその他 | 8,060 | False |
| 9628 | 燦 HD | 情報通信・サービスその他 | 6,950 | False |
| 4404 | ミヨシ油脂 | 食品 | 10,665 | **True** ⚠ |
| 2146 | ＵＴグループ | 情報通信・サービスその他 | 920 | False |
| 4828 | ビジネスエンジニアリング | 情報通信・サービスその他 | 6,230 | False |
| 8699 | ＨＳ HD | 金融（除く銀行） | 5,700 | False |
| 3089 | テクノアルファ | 商社・卸売 | 5,275 | False |
| 3712 | 情報企画 | 情報通信・サービスその他 | 5,195 | False |
| 7803 | ブシロード | 情報通信・サービスその他 | 1,305 | False |
| 9633 | 東京テアトル | 不動産 | 7,895 | False |
| 4914 | 高砂香料工業 | 素材・化学 | 5,900 | False |
| 3479 | ティーケーピー | 不動産 | 8,625 | **True** ⚠ |
| 4540 | ツムラ | 医薬品 | 17,910 | **True** ⚠ |
| 8057 | 内田洋行 | 商社・卸売 | 10,265 | **True** ⚠ |

## §5 tests 結果

| test scope | 結果 |
|---|---|
| F111 関連 (`test_run_f111_real_batch_staging.py`) | **83 PASS** ✓ |
| F062 関連 (`test_run_f062_research_advisory_line_preview.py`) | 68 PASS ✓ (= 別 pipeline 影響無) |
| v1_4_1 関連 (4 file) | 269 PASS ✓ |
| **合計** | **420 PASS** ✓ |

新規追加 test:
- `test_max_candidates_default_is_50`: baseline 化検証
- `test_max_candidates_comparison_explicit_20_still_works`: comparison 用途維持検証

## §6 D43 以降の運用方針

### §6.1 即時 (= D43 から適用)

- F111 / morning advisory baseline で max=50 が自然に使われる
- recently_seen_codes は引き続き 8 件版: `8747, 5729, 3489, 340A0, 3798, 137A0, 7991, 9130`
- policy_version 1.4.2 継続
- 9130 demote 効果継続観察 (= 8 日目)
- 4404 v1.4.2 entry 復帰継続 (= 6 日目、⚠ risk warning)
- 9247/9628 連続 entry 評価 (= 12 日目か リセットか)

### §6.2 別 wave 候補

| wave 候補 | 内容 | 優先度 |
|---|---|---|
| 9247/9628 単独 demote 本実行 | D41/D42 sim A or B (= 9247 or 9628 のみ demote) | 中 (= D44 以降検討) |
| F111-THEME-SECTOR-OVERLAY-R1 | 半導体/電線/AI/防衛/銀行 等 theme tag F111 field 追加 | 高 |
| research_watchlist_signals 母集団拡張 | 35 → 100+ | 中 |

### §6.3 theme overlay 不足

D42 expansion sim 結果: F111 出力に theme/business_label field が **不在**.
ユーザー要件 §8 で確認すべき theme 11 種 (semiconductor / electric_wire /
ai_datacenter / power_grid / optical_fiber / defense / bank / trading_company /
machinery / electric_equipment / nonferrous_metals) 全て **未実装**.

→ 別 wave F111-THEME-SECTOR-OVERLAY-R1 起票推奨.

## §7 CRITICAL/HIGH

- **CRITICAL: 0** (= 9130 entry 戻り無、低流動性 entry 浮上無)
- **HIGH: 0** (= letter-suffix 浮上無、非 100 株調整無、liquidity_fail 浮上無)

## §8 関連 file

- 親 design doc: `~/fire-vault/03_design/FIRE_f111_max_candidates_expansion_sim_2026-05-16.md`
- 修正 file 1: `scripts/jobs/run_f111_real_batch_staging.py` (line 43 / 597-602)
- 修正 file 2: `tests/scripts/jobs/test_run_f111_real_batch_staging.py` (line 884-898)
- smoke output: `/tmp/fire_max_candidates_baseline_50/d43_baseline_smoke_f111.json`
- smoke consumer: `/tmp/fire_max_candidates_baseline_50/d43_baseline_smoke_consumer.json`

## §9 safety footer

- 本 wave は **baseline 化 (= CLI default 値変更)**、実注文・auto-order ではない
- DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0 / cron 0 / plist 0 / workflow 0
- production / develop DB 接続なし、staging read-only URI mode=ro のみ
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- 100 株標準、要件 §5/6 表明: 100 株単位を標準、それ以外の株数調整は行わない
- forbidden phrase 全 0 件 (= grep 検証済)
- git add 0 / git commit 0 / git push 0 / --no-verify 0 (= 本 wave 内、HQ 別途決定)
- 9247/9628 demote 本実行なし
- max=100/200/300 baseline 化なし

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
