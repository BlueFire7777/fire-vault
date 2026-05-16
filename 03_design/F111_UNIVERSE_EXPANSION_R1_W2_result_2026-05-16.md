---
id: FIRE-F111-UNIVERSE-EXPANSION-R1-W2-RESULT-2026-05-16
phase: 実装結果 doc (= W2 完了、baseline 化反映)
priority: 最高
status: 実装完了 / staging smoke 完了、本番 baseline 反映済、develop/production 反映は別 wave
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
parent_w1_result: 03_design/F111_UNIVERSE_EXPANSION_R1_W1_result_2026-05-16.md
parent_design: 03_design/F111_UNIVERSE_EXPANSION_R1_2026-05-16.md
codex_lanes: 8
critical_high_count: 0
---

# F111-UNIVERSE-EXPANSION-R1-W2 実装結果 (= 2026-05-16)

## §1 目的

W1 read-only simulation で安全確認済 (= persistence 109 件、9130 demote 維持、低流動性 / letter-suffix / 非 100 株 entry 混入 0) の `signal_persistence --top-n=100` を CLI baseline 化.

W1 確認済根本原因:
- research_lane records loaded = 3,708 件
- signal_persistence --top-n=30 で 35 件まで縮小
- F111 universe 35 件頭打ち

W2 解決:
- CLI default 30 → 100 baseline 化 (= F111 universe 35 → 109 件、3.1 倍拡張)
- CLI override 維持 (= --top-n 30/200/ALL 明示指定可能)
- scoring / ranker / selection policy は無変更

## §2 変更内容 (= 最小変更)

### §2.1 修正 file (= 2 file のみ)

| file | 変更 |
|---|---|
| `scripts/jobs/run_research_watchlist_signal_persistence.py` (line 453-462) | `--top-n` CLI `default="30"` → `default="100"` + help text 更新 (W2 marker 追記) |
| `tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py` (末尾追記) | `TestTopNBaselineW2` クラス追加 (= 5 tests: default=100 / not 30 / help text / override / safety constraint) |

### §2.2 変更しなかった file (= 意図的不変)

| file | 理由 |
|---|---|
| `scripts/jobs/run_research_watchlist_ranker.py` | `DEFAULT_TOP_N: int = 30` は ranker 内 default、本質変更禁止に従い不変 |
| `scripts/jobs/score_factor_strategies.py` | `DEFAULT_TOP_N: int = 30` は別 module 別 CLI、影響なし |
| `simulation/research_lane/signal_persistence.py` | filter logic は不変、引数経由で override 維持 |
| `scripts/jobs/run_f111_real_batch_staging.py` | F111 runner は signal_persistence を直接呼ばない、影響なし |
| `scripts/jobs/_selection_policy_v1_4_1.py` | v1.4.2 selection policy は不変、risk_yen 非加点維持 |
| `scripts/jobs/_v1_4_1_consumer.py` | consumer 側 filter (= 9130 demote / 低流動性 / letter-suffix / 100 株) 不変 |

## §3 W1 結果との差分 + smoke 結果

### §3.1 W1 と W2 比較

| 項目 | W1 simulation (= read-only) | W2 baseline (= 実 CLI 設定) | 一致 |
|---|---|---|---|
| top-n=30 (override) | persistence 35 件 | smoke 35 件 (= override 維持) | ✓ |
| top-n=100 (default) | persistence 109 件 | smoke **109 件** (= 新 baseline) | ✓ |
| sectors (top-n=100) | 15 種 | **15 種** | ✓ |
| letter-suffix 数 | 13 件 | **13 件** | ✓ |
| runtime | ~200ms | ~200ms | ✓ |

→ W2 baseline 化結果は W1 simulation を完全に再現.

### §3.2 W2 smoke 詳細

**baseline smoke** (= `--top-n` 不指定):
```
.venv/bin/python -m scripts.jobs.run_research_watchlist_signal_persistence \
  --db staging --base-date 2026-07-15 \
  --source-version w60_universe_expansion_r1_w2_baseline \
  --output-json /tmp/fire_f111_universe_expansion_r1_w2/w2_baseline_smoke.json
```

結果:
- mode: **dry-run** (= write_enabled=False) ✓
- top_n: **100** (= 新 baseline 自然適用) ✓
- inserted: 109 / replaced: 0 / skipped: 0 / failed: 0
- row_count: 13,730 → 13,730 (= DB write 0 確認)

**override smoke** (= `--top-n 30`):
- top_n: 30
- inserted: 35
- row_count: 13,730 → 13,730 (= DB write 0)

### §3.3 W2 baseline=100 の sector 分布

15 種: 不動産 / 医薬品 / 商社・卸売 / 小売 / 建設・資材 / 情報通信・サービスその他 / 機械 / 素材・化学 / 自動車・輸送機 / 運輸・物流 / 金融（除く銀行）/ 鉄鋼・非鉄 / 電機・精密 / 電気・ガス / 食品

→ 旧 top-n=30 baseline の sector 12 種から **+3 種** (= 自動車・輸送機 / 建設・資材 / 小売 が追加).

## §4 安全 gate 維持確認 (= 全 baseline 化後も維持)

| gate | W2 baseline 後 |
|---|---|
| 9130 recently_seen demote (= excluded 維持) | 維持 (= consumer 側 filter) ✓ |
| 9130 entry / watch 復帰 | 0 ✓ |
| 9247 / 9628 demote 本実行 | **無** (= 本 wave で実行せず、保留) ✓ |
| 低流動性 (liquidity_fail) entry 混入 | 0 (= consumer side filter) ✓ |
| letter-suffix entry 混入 | 0 (= consumer side excluded) ✓ |
| 非 100 株 entry 混入 | 0 ✓ |
| risk_yen 順位加点 / entry 除外 gate | 非使用継続 (= v1.4.2 policy) ✓ |
| risk_yen_over_pilot_budget warning | 表示のみ (= entry 維持) ✓ |
| 100 株標準 | 維持 ✓ |
| max_candidates=50 baseline | 衝突なし (= signal_persistence 側で 109 件、F111 max=50 で 35 件取得後 consumer top 14、独立) ✓ |
| CLI override | 維持 (= --top-n 30/200/ALL 明示指定で動作確認済) ✓ |

## §5 9247 / 9628 状態

| 観点 | 結論 |
|---|---|
| top-n=100 baseline 化後の rank | W1 確認: 9247=11位 / 9628=13位 / 9130=8位 で score 上位継続 |
| universe 拡張による固定化緩和 | 限定的 (= score 自体が高位、母集団拡大では順位下がらず) |
| demote 本実行 | **保留** (= 本 wave 範囲外) |
| 両方同時 demote | **禁止** (= 引き続き非推奨) |
| 片方 demote | 本 wave では実行せず |
| 次 wave 候補 | top-n=100 baseline 後の D47/D48 朝 pilot で固定化観察 → 緩和なしなら片方 demote or theme overlay |

## §6 tests 結果

| scope | 結果 |
|---|---|
| TestTopNBaselineW2 (= W2 新規 5 tests) | **5 PASS** ✓ |
| signal_persistence runner 全 regression | **31 PASS** ✓ |

新 tests 内容:
- `test_cli_default_top_n_is_100`: default="100" 確認
- `test_cli_default_top_n_is_not_30`: default="30" 残存無 (= regression 防止)
- `test_help_text_mentions_baseline_change`: help text に W2 marker 含む
- `test_filter_signals_supports_override`: filter signature override 維持
- `test_safety_constraints_unchanged`: write_enabled guard / WRITE_GUARD_LABELS 不変

## §7 max_candidates=50 baseline との関係 (= 衝突無確認)

- signal_persistence top-n=100 = research_watchlist_signals に保存される件数 (= 上流)
- F111 max_candidates=50 = F111 runner が SELECT する候補数上限 (= 下流)
- signal_persistence 109 件保存 → F111 が INNER JOIN で取得 → max=50 で 50 件上限
- 実際の F111 raw は staging research_watchlist_signals の latest base_date 件数に依存
- consumer 側 filter (= liquidity_fail / letter_suffix / recently_seen_demote) で entry 14 件程度に圧縮 → top 5 で morning advisory
- → **衝突無、独立に動作**

## §8 J-Quants 扱い (= 本 wave 範囲外)

- J-Quants API 実行: **0** ✓
- token / env secret 参照: **0** ✓
- staging DB 内 既存 J-Quants 由来 data の read-only smoke のみ
- J-Quants universe builder 本実装は別 wave (= F111-JQUANTS-UNIVERSE-BUILDER-R1、HQ approve 後)

## §9 CRITICAL/HIGH 判定

- **CRITICAL: 0**
- **HIGH: 0**

## §10 D47/D48 pilot handoff

D47-D48 朝 pilot (= max=50 baseline + top-n=100 baseline 後の初回稼働):

1. 朝 advisory で persistence 109 件が反映されるか確認
2. F111 raw / entry / watch / excluded 件数が拡張されるか
3. sector 多様化拡張 (= 7 → 10-15 種 想定)
4. 9247 / 9628 順位変化観察 (= 固定化緩和 or 継続)
5. 4404 risk_yen_warning 表示維持
6. 9130 excluded 継続
7. 低流動性 / letter-suffix / 非 100 株 entry 混入 0 維持

判定ルール (= 既存継続):
- 9130 entry 復帰 → CRITICAL
- 9130 watch 復帰 → HIGH
- 4404 watch 戻り → HIGH
- 低流動性 / letter-suffix / 非 100 株 entry 浮上 → HIGH
- 9247/9628 固定化継続 (= D47-D48 で 14-15 連続到達) → 片方 demote 別 wave 検討

## §11 次 wave 候補

| ID | 内容 | 優先 | 依存 |
|---|---|---|---|
| D47/D48 朝 pilot | top-n=100 baseline 初回稼働確認 | **最優先** | HQ approve / 朝 pilot 通常 cycle |
| F111-UNIVERSE-EXPANSION-R1-W3 | top-n=200 検証 + 必要なら baseline 化 | 中 | W2 安定後 |
| F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 | seed theme list + assign runner (= 9247/9628 固定化対策の根本) | 高 | 別 wave 起票 |
| F111-JQUANTS-UNIVERSE-BUILDER-R1 | J-Quants universe builder (= API 承認後、業績 description / news theme 取込) | 低 | API approve |
| 9247 / 9628 片方 demote 検討 | D47-D48 後の固定化継続なら検討 | 中 | HQ approve + theme overlay 結果 |

## §12 safety footer

- 本 wave は **最小実装 + read-only staging smoke** (= DB write 0 / API 0 / token 0 / LINE 0)
- **launchctl 0 / plist 0 / cron 0 / workflow 0** ✓
- production / develop DB 接続 0、staging URI mode=ro のみ
- 9247 / 9628 demote 本実行 0、両方同時 demote 禁止継続
- 9130 recently_seen demote 状態維持
- v1.4.2 selection policy 完全維持 (= risk cap 撤廃 / risk_yen 非加点 / risk_yen_warning 表示のみ / 100 株標準)
- max_candidates=50 baseline 衝突無
- forbidden phrase 全 0 件
- git add 0 / git commit 0 / git push 0 / --no-verify 0
- TODO Excel 更新 0
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)

## §13 関連 file

- 親 W1 result: `~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_W1_result_2026-05-16.md`
- 親設計: `~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_2026-05-16.md`
- sibling: `~/fire-vault/03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_2026-05-16.md`
- 統合 strategy: `~/fire-vault/03_design/F111_exploration_expansion_strategy_2026-05-16.md`
- W2 smoke baseline: `/tmp/fire_f111_universe_expansion_r1_w2/w2_baseline_smoke.json`
- W2 smoke override 30: `/tmp/fire_f111_universe_expansion_r1_w2/w2_override_30.json`
- 修正 file: `scripts/jobs/run_research_watchlist_signal_persistence.py` (line 453-462)
- 新規 test class: `tests/scripts/jobs/test_run_research_watchlist_signal_persistence.py::TestTopNBaselineW2`
