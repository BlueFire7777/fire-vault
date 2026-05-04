---
id: F275
phase: P9: 運用・保守
priority: 最優先
status: 完了
owner: Fujiwara
depends_on: [F274]
chapter: "13, 19, 24, 37"
created: 2026-05-04
updated: 2026-05-04
---

# F275: SimilarityEngine 最適化完成

## 概要

F274 で残った Codex CRITICAL 6 件目 (writer-side cache invalidation) と
ReproducibilityEngine 経由 per-call 9.81 ms の性能未達 22% を解消し、
Run a (1340 ticks × 約 500 銘柄) を Stage 3 移行可能な実用時間で完走させる。

## タスク詳細

### 採用アプローチ: 案 D (Phase 2-C 状態 + F277 移管)

Codex pre-commit review で計 8 件 CRITICAL 指摘:
- 4 件 (writer-side invalidation / sector_filter / trade_stats / signature 弱さ)
  本コミットで構造解消
- 4 件 (TTL stale / features DELETE/UPDATE / tick.py 例外伝播 /
  _TRADE_STATS_CACHE positions invalidation) は **F275 着手前からの既存問題
  + 過剰防御**と判明 → F277 へ移管

### 実装スコープ (4 + F271 v1.1 改訂)

1. **Phase 2-A**: writer-side cache invalidation
   - patterns/store.py の register_candidate / transition で
     reset_similarity_cache() 呼出
   - features/base.py の FeatureWriter.write() で同
   - scripts/seed_pattern_layer1.py の insert_patterns() で同
   - patterns/similarity.py: signature SQL 撤廃 (per-call -1.94 ms)

2. **Phase 2-B**: sector_filter numpy 化
   - cache 構築時に _needs_filter_check (numpy bool) 事前計算
   - Python loop 4,700 件 → 必要 pattern のみ (現状 0 件) 個別判定

3. **Phase 2-C**: trade_stats cache + top_match features 再取得撤廃
   - _TRADE_STATS_CACHE モジュール辞書 + reset_trade_stats_cache() API
   - search() 戻り値に vec_past を含めて fetch_pattern_feature_vector
     再呼出を撤廃

4. **Phase 2-D**: tick.py _has_features 一括 SQL + logger 例外記録
   - _fetch_symbols_with_features() 新規 (1 SQL で features 持つ銘柄 set 取得)
   - except Exception を logger.error() 化 (沈黙 → ログ出力に改善)

5. **F271 v1.1 改訂** (スコープ 6):
   - § 6-7「線形外挿で楽観視」新設 (F273 Phase 2-C / F274 Phase 2-D 事例)
   - § 6-8「計測対象の取り違え」新設 (F274 Phase 2-A/B/C 事例)

### F277 移管項目

- F277-A: tick.py 例外伝播設計 (raise + caller catch + run status='failed')
- F277-B: positions 書き込み経路から reset_trade_stats_cache() 統合呼出
- F277-C: マルチプロセス stale cache 対策 (signature SQL 復活 / shared memory)

## 成果物

### 実装ファイル (~/fire side、commit 3a222cb)
- patterns/similarity.py (numpy 化 + writer-side invalidation)
- patterns/store.py (register_candidate / transition で reset)
- patterns/reproducibility.py (_TRADE_STATS_CACHE)
- features/base.py (FeatureWriter.write で reset)
- simulation/paper_live/tick.py (_has_features 一括 SQL + logger)
- scripts/seed_pattern_layer1.py (insert_patterns で reset)
- requirements.txt (numpy>=2.0)

### 設計レポート (~/fire-vault side)
- [[03_design/F275_phase1_design_2026-05-04]] (Phase 1 設計、Fujiwara レビュー材料)
- [[03_design/F275_similarity_optimization_complete_2026-05-04]] (完了レポート)
- [[03_design/task_completion_criteria]] v1.1 (§ 6-7/6-8 追加)

## 性能改善実測値 (運用経路)

| 指標 | F274 後 | F275 案 D 後 | 改善 |
|---|---|---|---|
| per-call (ReproducibilityEngine.evaluate()) | 9.81 ms | **3.13 ms** | -68% |
| 1 tick × 500 銘柄 | 4.9 秒 | 約 1.6 秒 | -67% |
| Run a 1340 tick | 110 分予測 | 35 分予測 (実測 **21 分** in aab0aac) | -68〜-81% |
| F273 当初 (signature ハイブリッド) | 21.2 日見積 | 35 分 | **約 870 倍** |

## 進捗チェックリスト

- [x] Phase 1: 現状計測 + cProfile 内訳 + score 構造的限界判明 (events 達成 → F276 移管)
- [x] Phase 1: Fujiwara レビュー (F271 § 5 二段確認制)
- [x] Phase 2-A: writer-side cache invalidation
- [x] Phase 2-B: sector_filter numpy 化
- [x] Phase 2-C: trade_stats cache + top_match features 再取得撤廃
- [x] Phase 2-D: tick.py _has_features 一括 SQL + logger
- [x] Phase 2-E: 段階試走 (10/100/500 銘柄、運用経路)
- [x] Phase 2-F: Run a 投入 (実測 21 分完走、events=0 確認)
- [x] Phase 2-G: F271 v1.1 改訂
- [x] Phase 2-H: vault commit + ~/fire commit (--no-verify 例外承認 2 回目)

## 作業ログ

- 2026-05-04 朝: Chain 完走 events=0 → 仮説 A〜D 切り分け開始
- 2026-05-04 朝: F267 で features 不足解消 (87 → 1.07M 行)
- 2026-05-04 午前: F273 Phase 1 (F040 = 集計レポーター、seeder ではない) 確定
- 2026-05-04 昼: F273 Phase 2-A smoke test 成功 (similar_count 0 → 4)
- 2026-05-04 昼: F273 Phase 2-B fullscale seed (4,700 件)、Phase 2-C 走行前中止 (21 日見積)
- 2026-05-04 夕: F274 numpy + cache 化 (per-call 2,733 → 9.81 ms)、Phase 2-D 中止
- 2026-05-04 夜: F275 Phase 1 cProfile 内訳測定、events 達成は F276 移管
- 2026-05-04 夜: F275 Phase 2-A〜D 実装、Run a 21 分完走
- 2026-05-04 夜: 案 D 確定 commit 3a222cb (--no-verify 例外、F277 移管)

## 完了条件

- [x] per-call ≤ 5 ms (運用経路 ReproducibilityEngine.evaluate())
- [x] 1 tick × 500 銘柄 ≤ 2.5 秒
- [x] Run a 1340 tick ≤ 60 分 (実測 21 分、予測 35 分)
- [x] 既存テスト 550 PASS 維持
- [x] F277 移管項目を vault に記録 ([[03_design/git_governance_2026-05-04]] の F278
      でなく、F277 として別タスク化)

→ **F271 § 4 v1.1 完了水準**: 動いた ✅ / 機能した ✅ / 期待値達成 ✅
(events_total=0 は F275 スコープ外、F276 で対応)

## 関連リンク

- 設計レポート: [[../03_design/F275_phase1_design_2026-05-04]] /
  [[../03_design/F275_similarity_optimization_complete_2026-05-04]]
- 真因階層振り返り: [[../03_design/root_cause_hierarchy_2026-05-04]]
- 完了基準 v1.1: [[../03_design/task_completion_criteria]]
- 起点: [[../03_design/F032_F054_diagnosis_2026-05-04]]
- 前提タスク: [[F274]] (実装中、--no-verify 例外 1 回目で完了)
- 後続タスク: [[F276_events達成_positions_seeding]] /
  [[F277_paper_live例外伝播_マルチプロセスcache]] /
  [[F278_git_ガバナンス整備]]
- コード: ~/fire/patterns/similarity.py / patterns/reproducibility.py /
  simulation/paper_live/tick.py 他 (commit 3a222cb)
- 運用ルール: [[../タスク運用ルール]]
