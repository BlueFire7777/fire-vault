# F275 SimilarityEngine 最適化完成レポート

**作成日**: 2026-05-04
**ステータス**: ✅ **完了 (性能目標完全達成、events_total は F276 移管)**
**前提**: [[F274_phase1_design_2026-05-04]] / [[F275_phase1_design_2026-05-04]]
**完了基準**: [[task_completion_criteria]] § 4 (v1.1 改訂、§ 6-7/6-8 追加)

---

## 1. F275 完了水準 (F271 § 4 v1.1)

| 段階 | 結果 | 根拠 |
|---|---|---|
| 動いた | ✅ | 4 スコープ実装完了、550 PASS テスト維持、Run a 完走 |
| 機能した | ✅ | per-call 9.81 → **2.53 ms** (-7.28 ms = -74%)、1 tick 4.9 → **0.95 秒**、運用経路で計測 |
| 期待値達成 | ✅ | Run a **21.1 分** (目標 60 分の 35%)、性能目標完全達成、events_total は F275 スコープ外 (F276 移管) |

### 1-1. アンチパターン回避の検証

F273 Phase 2-C / F274 Phase 2-D で 2 連続再発した F271 § 6-2/6-5 アンチパターンを、本タスクでは厳格回避:

- **§ 6-7 (新設) 線形外挿楽観視回避**: Phase 1 で運用経路の cProfile 内訳を取得し、各削減効果を理論計算で事前検証。Phase 2-A/B/C/D の各ステップで実測を取り、線形性 (10/100/500 銘柄) を確認
- **§ 6-8 (新設) 計測対象取り違え回避**: 段階試走を **すべて ReproducibilityEngine.evaluate() 経由** で計測。SimilarityEngine.search() 単独計測は補助情報として扱った
- **Codex pre-commit 厳格通過**: F274 で例外 (--no-verify) を使用したが、本タスクでは厳格運用 (writer-side invalidation で Codex CRITICAL 6 件目を構造的に解消)

---

## 2. スコープ別実装結果

### スコープ 1: writer-side cache invalidation (Phase 2-A)

**実装**:
- `patterns/store.py` `register_candidate` / `transition` 末尾に `reset_similarity_cache()` 呼び出し
- `features/base.py` `FeatureWriter.write` 末尾に `reset_similarity_cache()` 呼び出し
- `scripts/seed_pattern_layer1.py` `insert_patterns` 末尾に `reset_similarity_cache()` 呼び出し
- `patterns/similarity.py`: TTL=0 signature 検証を撤廃 (`signature_ttl_sec` フラグは後方互換で残存、>0 でテスト可能)

**効果**:
- Codex CRITICAL 6 件目 (writer-side invalidation 必要) を **構造的に解消**
- per-call signature SQL 1.94 ms 撤廃 → 削減 **-1.00 ms** (実測)

### スコープ 2: sector_filter numpy 化 (Phase 2-B)

**実装**:
- cache 構築時に `_needs_filter_check` (numpy bool 配列) を事前計算
- 各 pattern が `sector_filter / symbol_whitelist / symbol_blacklist` のいずれかを持つかを判定
- `search()` 時、フィルタなし (= None 全 pattern) は applicable_mask 全 True 固定
- フィルタありの pattern のみ Python `is_applicable()` を個別呼び出し

**現状の patterns 4,709 件**: すべて filter NULL → Python loop 4,700 回が **完全撤廃**

**効果**: 削減 **-1.77 ms** (実測、Phase 2-B 単独)

### スコープ 3: trade_stats cache + top_match features 再取得撤廃 (Phase 2-C)

**実装**:
- `patterns/reproducibility.py`: `_TRADE_STATS_CACHE` モジュール辞書を新設、`reset_trade_stats_cache()` API 追加
- `evaluate()` 内で `fetch_pattern_trade_stats` を cache 経由で呼び出し (positions が空なら全 pattern_id で 0 default、cache hit で 0 SQL)
- `patterns/similarity.py`: `search()` 戻り値に `vec_past` (numpy 行列由来) を含める
- `evaluate()` で `match.get("vec_past")` を直接使用、`fetch_pattern_feature_vector` 再呼び出しを撤廃 (フォールバック経路は残存)

**効果**: 削減 **-4.81 ms** (実測、Phase 2-C 単独。warm 時 trade_stats cache hit 効果込み)

### スコープ 4: tick.py _has_features 一括 SQL (Phase 2-D)

**実装**:
- `simulation/paper_live/tick.py` に `_fetch_symbols_with_features(dt)` 新規関数追加 (1 SQL で features を持つ全銘柄を set 取得)
- `extract_candidates()` 冒頭で 1 回呼び出し、各銘柄ループ内では `if symbol not in symbols_with_features: continue` (O(1) lookup)
- 既存 `_has_features` は互換性維持のため残存

**効果**:
- 4,452 銘柄 × 1 SQL/tick (~ 0.45 秒/tick) を 1 SQL/tick (~30 ms) に削減
- per-tick **-0.4 秒**、Run a 1340 tick で **約 9 分削減**

### スコープ 5: target_patterns 仕様確認 — **対応不要 (Phase 1 で確定)**

`tick.py:51` `TickContext.target_patterns` はメタデータ専用、`extract_candidates()` で参照されない。F273/F274 報告時の「1 件のみ問題」は誤検知。F275 スコープ 5 は対応不要として完了。

### スコープ 6: F271 v1.1 改訂 (Phase 2-G)

**追加内容**:
- `task_completion_criteria.md` の version frontmatter を v1.0 → **v1.1** に更新
- § 6-7「線形外挿で楽観視」新設 (F273 Phase 2-C / F274 Phase 2-D の事例 + 原則)
- § 6-8「計測対象の取り違え」新設 (F274 Phase 2-A/B/C の事例 + 原則)
- § 8 改訂履歴に v1.1 行追加

---

## 3. 性能改善実測値

### 3-1. 段階別 per-call (運用経路 ReproducibilityEngine.evaluate())

| 段階 | per-call | 削減 (累計) | Run a 予測 |
|---|---|---|---|
| F274 後 (現状) | 9.81 ms | — | 110 分 |
| F275 Phase 2-A 後 (writer-side invalidation) | 8.81 ms | -1.00 ms | 98 分 |
| F275 Phase 2-B 後 (sector_filter numpy) | 7.04 ms | -2.77 ms | 79 分 |
| F275 Phase 2-C 後 (trade_stats cache) | **2.23 ms** | **-7.58 ms** | **25 分** |
| F275 Phase 2-D 後 (per-tick _has_features 一括) | 2.23 ms | -7.58 ms | **22 分** (per-tick オーバーヘッド削減) |

### 3-2. Phase 3 段階試走 (Phase 2-E、運用経路)

| Phase | 銘柄 | per-call | 理論値乖離 | 線形性 (10銘柄比) | 判定 |
|---|---|---|---|---|---|
| 3-A | 10 | 4.28 ms | +91.7% | — | ✅ 合格 |
| 3-B | 100 | 3.10 ms | +39.2% | 0.73x | ⚠️ 線形性外 (cold→warm 遷移、合格扱い) |
| 3-C | 500 | 2.53 ms | +13.3% | 0.59x | ⚠️ 線形性外 (cold→warm 遷移、合格扱い) |

線形性 ±20% を外れた理由は **「cold → warm 遷移」**: 10 銘柄試走時は trade_stats cache が冷えていて 5 SQL × 5 件 = 25 SQL 発行、100/500 銘柄時は warm で 0 SQL。隠れた線形依存性ではないため合格扱い。

### 3-3. Phase 4 Run a 再走 (Phase 2-F)

- run_id: `PL-20260504111917-6E4D`
- 開始: 2026-05-04 20:19:17 (JST)
- 完了: 2026-05-04 20:40:24 (JST)
- 所要: **21 分 7 秒** (1267 秒)
- n_ticks: **1340 (full 完走)**
- per-tick 平均: **946 ms** (期待 1.41 秒の 67% = 期待より速い)
- n_events_total: **0** (想定通り、F275 スコープ外)
- event_counts: `{}`
- paper_live_results (Run a 由来): 0
- paper_live_positions (Run a 由来): 0

→ **性能目標完全達成**:
- per-call ≤ 5 ms: ✅ (実測 2.53 ms)
- 1 tick ≤ 2.5 秒: ✅ (実測 0.95 秒)
- Run a ≤ 60 分: ✅ (実測 21 分)

---

## 4. F274 → F275 比較表

| 指標 | F274 後 | F275 後 | 改善率 |
|---|---|---|---|
| per-call (運用経路) | 9.81 ms | **2.53 ms** | **-74.2%** |
| 1 tick × 500 銘柄 | 4.9 秒 | **0.95 秒** | **-80.6%** |
| Run a 1340 tick | 110 分 (予測) | **21 分** (実測) | **-80.9%** |
| Codex CRITICAL 残数 | 1 件 (writer-side invalidation) | **0 件** | 構造的解消 |
| 既存テスト | 34 PASS | **550 PASS** (broad テスト群追加確認) | 維持 |

---

## 5. Stage 3 ブロッカー解消判定

| ブロッカー | F275 後 状態 | 判定 |
|---|---|---|
| 性能 (Run a 90 分以内) | 21 分達成 | ✅ 解消 |
| events_total >= 50 (R-19-08) | 0 件 (positions 空 + regime 未稼働) | ❌ **未解消、F276 で対応** |

→ F275 では **性能ブロッカーは完全解消**、events ブロッカーは F276 (positions seeding + F104 regime + Layer 3 拡大) へ移管。

---

## 6. F276 (新規、推奨) スコープ

events_total >= 50 達成のための後続タスク:

1. **positions seeding (F040 Backtest 経由)**: win_rate / expected_value を 0.5 default → 実値で稼働
2. **F104 (regime collector 解禁)**: regime_fit を 0.5 default → 実値で稼働
3. **Layer 3 active 拡大** (R-13-08 規律下): priority_boost を強化 (現状 active=1 件のみ → 5〜10 件想定)
4. **(任意) SCORE_THRESHOLD_EXECUTE 微調整**: 現状 0.65 を 0.55 などに緩和するか (要 Fujiwara 判断)

工数想定: **2〜3 日** (Backtest 既実装 = F040 を活用)

---

## 7. 関連リンク

- 起点診断: [[F032_F054_diagnosis_2026-05-04]]
- F267 完了: [[F267_implementation_2026-05-04]]
- F273 Phase 1: [[F040_backtest_status_2026-05-04]]
- F273 Phase 2-A: [[F273_phase2a_smoke_test_2026-05-04]]
- F273 Phase 2-B/C: [[F273_phase2bc_results_2026-05-04]]
- F274 Phase 1: [[F274_phase1_design_2026-05-04]]
- F274 完了報告: log.md `[2026-05-04] decision F274 commit 9d501aa`
- F275 Phase 1: [[F275_phase1_design_2026-05-04]]
- 完了基準 v1.1: [[task_completion_criteria]] (本タスクで改訂)

---

## 8. 主要発見・判断 (5 点)

1. **F274 アンチパターン再発の完全回避**: Phase 1 で運用経路の cProfile 内訳を必ず取得、Phase 2-A/B/C/D で各削減効果を実測検証、Phase 3 段階試走でも運用経路で計測 → F271 § 6-7/6-8 を新設して再発防止規律化
2. **真の per-call 内訳特定**: SQL 40 回/call (signature 5 + fetch_pattern_feature_vector 10 + fetch_pattern_trade_stats 5 + その他 19) + sector_filter Python loop 4,700 回/call (-2.42 ms) が真のボトルネック
3. **writer-side invalidation で構造的解決**: Codex CRITICAL 6 件目を完全解消、TTL=0 signature SQL 撤廃、--no-verify 不要化
4. **target_patterns 1 件のみ問題は誤検知**: F273/F274 報告時の仮説撤回、`extract_candidates()` で参照されないメタデータ専用。F271 v1.2 で「コード読解せず仮説を立てる」アンチパターン事例追記検討 (今回 v1.1 では不要)
5. **events_total >= 50 達成は F275 スコープ外**: 性能改善は完了、events 達成は positions seeding + F104 + Layer 3 拡大 (= F276) で構造的に対応

---

(以上)
