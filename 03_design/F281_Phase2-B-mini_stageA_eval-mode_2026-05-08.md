---
title: F281-Phase2-B-mini 段階 A 設計記録 — 評価モード実装
date: 2026-05-08
phase: F281-Phase2-B-mini
stage: A
related: F275, F230, F273, R-13-08, F271 v1.7
---

# F281-Phase2-B-mini 段階 A 設計記録 — 評価モード実装

## 1. 目的

paper_live --batch-replay の評価走行時のみ STATUS_ADJUSTMENT 重み付けを
bypass する評価モードを実装し、F273_BRE 4,708 件 (status='candidate')
の期待値・勝率・DD を実評価可能にする。R-13-08 (approved_active 昇格は
Fujiwara 承認必須) 規律は維持。

## 2. 経緯

### 2-1. 経緯概要

(1) 引き継ぎプロンプト前提「方針 R = target_patterns 拡張」が F275 結論
    (target_patterns はメタデータ専用) と矛盾、本部側ミス 12 として記録
(2) 段階 A 真の実装位置の追加 grep で STATUS_ADJUSTMENT 重み付け機構
    (similarity.py:87、CANDIDATE=0.4 倍減衰) が実質除外として機能と判明、
    本部側ミス 13 として記録
(3) 案 A/B/C/D 比較で C 案 (R-13-08 candidate → approved_active 昇格運用)
    採用、ただし段階 B 評価走行に暫定措置必要のため段階 A 再定義

### 2-2. 関連 commit / 文書

- log.md 1728-1744: F275 結論追記 (commit c0b9515)
- F271 v1.7 §6-22-novenary ミス 12/13: 構造的再発防止策 (commit 05846fa)
- F275 完了レポート: 03_design/F275_similarity_optimization_complete_2026-05-04.md

## 3. 設計

### 3-1. フラグ経路

  CLI: paper_live batch-replay --eval-mode [--target-patterns ...]
    ↓
  BatchReplayRunner(eval_mode: bool = False)
    ↓
  PaperLiveRunner.start_session(eval_mode=eval_mode)
    ↓
  extract_candidates(ctx: TickContext)  ← ctx.eval_mode を新規追加
    ↓
  ReproducibilityEngine(db_path, eval_mode=eval_mode)  ← 引数追加
    ↓
  SimilarityEngine.search(..., eval_mode=False)  ← デフォルト False で
                                                  後方互換、評価モード時のみ True

### 3-2. SimilarityEngine.search() 内部の bypass 動作

patterns/similarity.py 修正案 (line 626 周辺、Python 擬似コード):

  if eval_mode:
      評価モード: status_mults を 1.0
      (death_note / frozen / rehab は 0.0 維持)
      safe_status_mults = np.where(
          np.isin(self._status_array,
                  [Status.DEATH_NOTE, Status.FROZEN, Status.REHAB]),
          0.0, 1.0
      )
      boosted = final_score * self._layer_mults * safe_status_mults
  else:
      通常モード: 現状維持
      boosted = final_score * self._layer_mults * self._status_mults

### 3-3. 評価結果の識別

paper_live_results テーブルに run_mode TEXT default 'normal' を追加
(migration スクリプト)。評価モード走行時は run_mode='eval' で記録、
production 実取引と混同しない設計。

### 3-4. 環境分離との整合 (F282)

評価モード走行は staging 環境のみ で実施。production / develop には
影響しない。F282 ルール 16 (production DB 直接操作禁止、bin/fire-* wrapper
経由) を遵守: bin/fire-staging python -m simulation.paper_live batch-replay
--eval-mode の経路で実行。

## 4. 実装スコープ (段階 A-2 〜 A-5)

- 段階 A-2: similarity.py に eval_mode 引数追加 + bypass 実装
- 段階 A-3: extract_candidates / ReproducibilityEngine / PaperLiveRunner
           / BatchReplayRunner の引数連鎖追加
- 段階 A-4: paper_live_results テーブル run_mode カラム追加 (migration)
- 段階 A-5: tests/ 追加 + staging smoke test

## 5. 受入基準

(1) eval_mode=True 走行で全 status (death_note/frozen/rehab 除く) が
    1.0 重み付け扱いされること (test_eval_mode_status_bypass)
(2) eval_mode=False 走行で従来通り STATUS_ADJUSTMENT が適用されること
    (test_normal_mode_status_adjustment、後方互換)
(3) paper_live_results.run_mode カラムが追加され、eval/normal で識別可能
    なこと (test_run_mode_column_persistence)
(4) staging smoke test (10-20 銘柄、3 営業日範囲) で eval_mode 走行が
    events>0 で完了すること
(5) production / develop DB 無触確認 (running on staging only)
(6) Codex pre-commit 一発通過、CRITICAL ゼロ、--no-verify 不使用

## 6. 後送り項目 (段階 A-2 〜 A-5 の対象外)

- 段階 B (4,708 件本格評価走行) は別タスクで実施
- subset 抽出基準 (X 保守 / Y 中庸 / Z 積極) は段階 C で Fujiwara 戦略判断
- approved_active 化 (R-13-08 経路) は段階 D で実施

## 7. 改訂履歴

- v1.0 (2026-05-08): 初版作成、段階 A 着手前
