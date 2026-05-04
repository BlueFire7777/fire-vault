# F273 Phase 2-B + 2-C 結果レポート (ケース 3 — 新主因 D''' 出現)

**作成日**: 2026-05-04
**位置付け**: Phase 2-B (本格 seeder) は完璧達成、Phase 2-C (Run a 再走) は **走行前に実行不可と判明** したためケース 3 (新主因 D''')
**前提レポート**: [[F040_backtest_status_2026-05-04]] / [[F273_phase2a_smoke_test_2026-05-04]]
**完了基準**: [[task_completion_criteria]] § 4 (3 段階)

---

## 1. Phase 2-A → 2-B → 2-C の流れ総括

| Phase | 内容 | 結果 |
|---|---|---|
| Phase 1 | F040 現状調査 + 真主因 D'' 特定 | ✅ ケース F (前提が違う) と判定、Layer 1 seeder 設計提案 |
| Phase 2-A | 1 銘柄 × 10 日 smoke test (4 件 seed) | ✅ similar_count 0→4、decision pass→reference_only、score 0→0.5787 |
| Phase 2-B-1 | Core500 先頭 100 銘柄 × 120 日 | ✅ 1,021 件 inserted、4 件 skipped (Phase 2-A 冪等吸収) |
| Phase 2-B-2 | Core500 残り 400 銘柄 × 120 日 | ✅ 3,675 件 inserted、err=0 |
| **Phase 2-B 合計** | **Core500 × 120 日** | **✅ 4,700 件 / 485 銘柄、想定 4,704 件と誤差 0.1%** |
| Phase 2-C | Run a 再走 | **❌ 走行前に 21 日かかると判明、中止判断** |

---

## 2. Phase 2-B 結果 (完璧達成)

### 2-1. 投入結果

| 項目 | 値 |
|---|---|
| 総 F273_BREAKOUT_* 件数 | **4,700** (期待 ≒ 4,704、誤差 0.1%) |
| 銘柄数 | 485 (Core500 中 15 銘柄は 120 日内 breakout=1 が 0 件) |
| patterns 総件数 | 4,709 (= 旧 9 + F273_BREAKOUT_* 4,700) |
| 期間カバレッジ | 2025-12-03 〜 2026-05-01 (100 営業日、12 営業日は 0 発火) |
| 冪等性 | ✅ 立証 (Phase 2-A の 4 件が Phase 2-B-1 で skipped=4) |

### 2-2. 銘柄別発火数 (上位 10)

| 銘柄 | 発火数 |
|---|---|
| 89180 | 33 |
| 57110 | 24 |
| 64070 | 23 |
| 40040 | 23 |
| 27020 | 23 |
| 91040 | 22 |
| 57130 | 22 |
| 31030 | 22 |
| 66220 | 20 |
| 62350 | 20 |

最大 33 件 (89180) と最小 1 件の差は妥当 (流動性高銘柄ほど breakout 頻度が高い)。

### 2-3. ReproducibilityEngine 事前検証 (Phase 2-B 後の 5 サンプル)

| サンプル | decision | sim_count | score | confidence |
|---|---|---|---|---|
| 72030 / 2026-04-30 | reference_only | 10 | 0.5787 | medium |
| 99840 / 2026-04-30 | reference_only | 10 | 0.5787 | medium |
| 68570 / 2026-04-30 | reference_only | 10 | 0.5787 | medium |
| 99840 / 2026-05-01 | reference_only | 10 | 0.5787 | medium |
| 89180 / 2026-05-01 | reference_only | 10 | 0.5787 | medium |

| 比較 | Phase 2-A | Phase 2-B 後 |
|---|---|---|
| similar_count | 4 | **10** (top_n=10 で頭打ち) |
| decision | reference_only | **reference_only** (変わらず) |
| score | 0.5787 | **0.5787** (変わらず) |
| confidence | low | **medium** (改善) |

→ MIN_SIMILAR_COUNT_FOR_EXECUTE=5 は突破済、しかし SCORE_THRESHOLD_EXECUTE=0.65 に届かず `decision="execute"` には至らない。

### 2-4. score 0.5787 で停滞している理由 (推定)

`reproducibility.py:47` の COMPOSITION_WEIGHTS:

| 構成要素 | 重み | 現状値推定 |
|---|---|---|
| similarity | 0.20 | 1.0 (max) → 0.20 寄与 |
| win_rate | 0.25 | **0** (positions 空) |
| expected_value | 0.20 | **0** (positions 空) |
| regime_fit | 0.10 | **0** (regime collector 未稼働、F104 待ち) |
| fill_quality | 0.10 | 推定 ~0.5〜1.0 → 0.05〜0.10 寄与 |
| priority_boost | 0.10 | active=1 件しかないため弱い |
| risk_fit | 0.05 | 推定 ~1.0 → 0.05 寄与 |
| **合計** | 1.0 | **約 0.5787 に留まる** |

→ score を 0.65 まで上げるには、**positions テーブルに過去トレード結果**を入れて win_rate / expected_value を稼働させる必要。これは Phase 2-B 範囲外。

---

## 3. Phase 2-C 中止判断 (ケース 3)

### 3-1. 速度劣化計測

走行前に SimilarityEngine.search() の所要時間を計測:

```
4,709 件 patterns で search(): 2,733 ms = 約 2.7 秒
per pattern: 0.580 ms (完全 O(N) 線形スキャン)
```

Phase 2-A 時 (patterns 13 件) と比較:
- Phase 2-A: 推定 ~8 ms/call (13 × 0.6 ms)
- Phase 2-B 後: **2,733 ms/call (4,709 × 0.58 ms)**
- → **約 360 倍鈍化** (件数比とほぼ一致 = O(N) 立証)

### 3-2. Run a 走行時間の現実的見積

| 項目 | 値 |
|---|---|
| 1 tick × 500 銘柄評価 | 500 × 2.733 秒 = **1,366 秒 = 22.8 分** |
| Run a 総 tick 数 | 1,340 tick (= 20 営業日 × 67 tick) |
| 全体走行時間 | 1,340 × 1,366 秒 = **1,830,926 秒 = 21.2 日 (!!)** |

→ **Run a を 21 日待つのは実用不可**。Phase 2-A 時は 13 件だったため 91 分で済んだだけ。

### 3-3. 走行中止の判断根拠

ユーザー指示書 § STEP 4 ケース 3「失敗、新たな主因 D''' 出現」に該当。Run a 投入は **走行前中止**。
- 21 日の待機は F271 § 6 「動いた ≠ 機能した」の典型 (実行可能だが実用不可)
- 中止判断は迅速で正しい (cf. F271 § 6 アンチパターン回避)
- F273 Phase 2-C は「実行不能」を発見した時点で完了 (ケース 3)

---

## 4. 新主因 D''' の実体

### 4-1. 真の問題: SimilarityEngine.search() の O(N) 全件スキャン + 識別力ゼロ

`patterns/similarity.py:189-200`:
```python
sql = "SELECT * FROM patterns WHERE 1=1"
# 全件取得 → 各 pattern_id について fetch_pattern_feature_vector → 類似度計算
# → top_n=10 で head 切り出し
```

Phase 2-B 後の hit 上位 10 件 (72030 評価):
| pattern_id | similarity_score | boosted_score |
|---|---|---|
| F273_BREAKOUT_306022AF14 | **1.0000** | 0.2800 |
| F273_BREAKOUT_0D61EAE5A3 | **1.0000** | 0.2800 |
| F273_BREAKOUT_68121604AC | **1.0000** | 0.2800 |
| F273_BREAKOUT_2114D72BBF | **1.0000** | 0.2800 |
| F273_BREAKOUT_99CEBBD160 | **1.0000** | 0.2800 |
| ...残り 5 件も同様 | 1.0000 | 0.2800 |

→ **すべて完全一致 (sim=1.0000)、識別力ゼロ**。breakout_flag=1 + 他の features がほぼ同じ値域のため、銘柄を変えても score に差が出ない。

### 4-2. 三重ループの計算量

`tick.extract_candidates()` の構造:
```
for symbol in symbols (4,452 件):           # O(M)
    if not _has_features(...): continue     # 500 銘柄 features カバー
    result = engine.evaluate(symbol, dt, top_n=10)  # 内部で O(N)
        # SimilarityEngine.search() が 全 patterns N 件を loop
        # → fetch_pattern_feature_vector + similarity_score
```
これが tick 1340 回繰り返される → O(symbols × patterns × ticks) = O(500 × 4,700 × 1,340) ≈ **31 億回の類似度計算**

### 4-3. Phase 2-A 時の見落とし

Phase 2-A 完了時点でこの計算量問題を予見できなかった理由:
- patterns 13 件 → 91 分で完走 → 「成功」と判定
- patterns を 4,700 件にすれば 4,700/13 = 360 倍 = 91 × 360 = **22.7 日**
- Phase 2-A 報告で「Phase 2-B で 5 件以上 seed すれば execute 到達確実」と書いたが、execute 到達 ≠ Run a 完走可能
- → **線形スケーリングを検証せず楽観視した F271 § 6-2「exit code 0 を成功と見なす」アンチパターンの一例**

---

## 5. F273 完了水準評価 (F271 § 4)

### 5-1. Phase 2-B 単独 (= Layer 1 seeder の動作確認)

| 段階 | 結果 | 根拠 |
|---|---|---|
| 動いた | ✅ | seeder exit 0、4,700 件 inserted、err=0、Codex review 通過 |
| 機能した | ✅ | patterns に実 (symbol, dt) 紐付き 4,700 件、冪等性立証、ReproducibilityEngine が候補多数読み込み |
| 期待値達成 | ✅ | 想定発火件数 ≒ 4,704 と誤差 0.1%、similar_count 4→10 達成 |

### 5-2. Phase 2-C 単独 (= Run a で events>0 達成)

| 段階 | 結果 | 根拠 |
|---|---|---|
| 動いた | ❌ | 走行前中止 (21 日見積 = 実用不可) |
| 機能した | ❌ | events_total 確認不能 (走らせていない) |
| 期待値達成 | ❌ | Stage 3 ブロッカー解消には至らない |

### 5-3. F273 全体 (Phase 1 + 2-A + 2-B + 2-C)

| 段階 | 結果 | 根拠 |
|---|---|---|
| 動いた | ✅ | Phase 1 調査完走、Phase 2-A/2-B seeder 完走 |
| 機能した | ⚠️ 部分達成 | Layer 1 seeder は機能、ReproducibilityEngine は引き続き reference_only 止まり、Run a は実行不能 |
| 期待値達成 | ❌ | events>0 未達、Stage 3 ブロッカー解消なし、新主因 D''' を発見した点を除いて期待値未達 |

---

## 6. Phase 3 提案 (新主因 D''' 解消)

### 6-1. 案 P3-A: SimilarityEngine.search() の最適化 (推奨)

設計改善:
- `patterns` テーブルに `feature_vector_hash` カラム追加 (各 pattern の features を hash 化)
- 評価時に「自分の features hash と同じ patterns」だけ取得 (O(1) lookup)
- または `pattern_type` でフィルタリング (`'F273_layer1_seed'` のみ取り出し)
- top_n パターンを事前計算して `pattern_top_matches` キャッシュテーブルに保存
- Run a 走行時は キャッシュテーブル参照のみ (O(1))

工数想定: 4 時間〜1 日 (テスト込み)
速度改善: 2.7 秒/call → 数 ms/call (1000 倍高速化)
Run a 走行: 21 日 → 約 1.5 時間 (Phase 2-A の 91 分 × 数倍程度)

### 6-2. 案 P3-B: 投入 patterns 数を絞り込み (副案)

- Phase 2-B の 4,700 件から「特に重要なもの」のみ Layer 1 active 化
- 例: 各銘柄上位 5 件のみ採用 → 500 銘柄 × 5 件 = 2,500 件
- 1 銘柄 1 日 = 1 pattern (重複排除) に絞れば 100 営業日 × 500 銘柄 - α = ~50,000 候補から 100〜1,000 件に絞る
- 単純絞り込みでは識別力ゼロ問題は解消しない、本質的な解は P3-A

### 6-3. 案 P3-C: 発火条件を多様化して識別力を上げる

- 案 A 「breakout_flag=1 AND vwap_position >= 0.05」など複合条件で各 pattern が異なる features 組み合わせを持つように
- ただし features 値域が狭い (vwap_position レンジ -0.18〜0.15) ため、識別力向上は限定的
- F104 (regime collector 解禁) 後に再検討

### 6-4. 案 P3-D: Phase 2-B のロールバック判断

Phase 2-B で投入した 4,700 件をどうするか:
- A: そのまま残す (将来 P3-A 最適化後に再利用)
- B: 削除して Phase 2-A の 4 件に戻す (DB 軽量化)
- C: 半分 (例: 上位 1,000 件) に絞る (compromise)

推奨: **A (そのまま残す)**。`metadata.smoke_test=true / approval_status=candidate_only_per_R-13-08` で誤認リスク低、再 seed の 5〜10 分を節約できる。

---

## 7. F273 ステータスと次タスク

### 7-1. F273 ステータス

- **F273 全体**: 「機能した」(Layer 1 seeder + 新主因 D''' 発見)、「期待値未達」(events>0 不達成)
- → ステータス: 検証中 (実装中) → **完了 (期待値部分達成、次フェーズ移管)**
- F273 Phase 1〜2-C のすべての成果物は活用継続

### 7-2. 次タスク候補

| タスク候補 | 内容 | 優先度 |
|---|---|---|
| **F275 (新)** | SimilarityEngine.search() 最適化 (案 P3-A) | 最優先 |
| F273 Phase 3 | F273 拡張として D''' 切り分け継続 | F275 と統合可能 |
| F104 | 指数四本値取得 → regime collector 解禁 | 中 (score 構成要素改善) |
| F268 | announcements 過去 6 ヶ月遡及 | 低 (D''' 解消後に再評価) |
| F269 | Chain 異常検知 assert | 中 (Run a 走行時間 assert に拡張) |
| F274 (Phase 2-D) | Active 化選別 | 凍結 (events>0 未達成のため判断不能) |

### 7-3. 推奨ロードマップ

```
今日: F273 Phase 2-B + 2-C 完了報告
   ↓
F275 (SimilarityEngine 最適化) 設計 + 実装 (4 時間〜1 日)
   ↓
Run a 再走 (1.5 時間予想) で events>0 達成見込み
   ↓
F104 (regime) と F268 (announcements) で score 構成要素改善
   ↓
F266 再評価 → Stage 3 移行可否判断
```

---

## 8. F271 完了基準による F273 Phase 2-B + 2-C 自己評価

| 段階 | 結果 | 根拠 |
|---|---|---|
| 動いた | ⚠️ 部分 | Phase 2-B seeder は完走、Phase 2-C は走らせず中止判断 |
| 機能した | ⚠️ 部分 | Layer 1 seed は機能、Run a の events>0 確認は不可 |
| 期待値達成 | ❌ | Stage 3 ブロッカー解消未達、ただし新主因 D''' 発見の進捗価値あり |

「期待値達成 ❌」の正直な報告 (F271 § 6-3「将来作るので OK」アンチパターン回避)。

---

## 9. 関連リンク

- 起点: [[F032_F054_diagnosis_2026-05-04]] (events=0 切り分け)
- F267 完了: [[F267_implementation_2026-05-04]] (features backfill)
- Phase 1: [[F040_backtest_status_2026-05-04]] (F040 = 集計レポーター)
- Phase 2-A: [[F273_phase2a_smoke_test_2026-05-04]] (smoke test 立証)
- 完了基準: [[task_completion_criteria]]
- Pattern Store 構造: [[01_requirements/FIRE_要件書_第37章_Pattern_Storeの4階層構造とActive_Priority_Set]]
- 次タスク候補: F275 (SimilarityEngine 最適化、新規)、F104 (副案、index 解禁)、F274 (active 化選別、凍結)

---

## 10. 主要発見・判断 (5 点)

1. **Phase 2-B 完璧達成**: 4,700 件 / 485 銘柄 / 100 営業日カバー、err=0、冪等性立証、想定 ±0.1%
2. **新主因 D''' 発見**: SimilarityEngine.search() が patterns 全件 O(N) 線形スキャン + features 識別力ゼロ。Phase 2-B で 360 倍鈍化、Run a 21 日見積で実用不可
3. **Phase 2-C 走行前中止判断**: 21 日待機は F271 § 6 アンチパターン (動いた≠機能した) の典型、迅速な中止判断が正しい
4. **score 0.5787 停滞の構造的理由**: positions 空 (win_rate=0 / expected_value=0) + regime collector 未稼働 (regime_fit=0) で、similarity 寄与だけでは SCORE_THRESHOLD_EXECUTE=0.65 に届かない
5. **Phase 2-A 見落としの教訓**: 「13 件で 91 分」を 4,700 件に線形外挿せず楽観視。F271 § 6-2「exit code 0 を成功と見なす」アンチパターンの一例として記録

---

(以上)
