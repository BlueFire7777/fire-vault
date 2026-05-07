---
title: F281-Phase2-B-mini 段階 B 設計記録 — Lane A1 本格評価走行
date: 2026-05-08
phase: F281-Phase2-B-mini
stage: B
related: F275, F281, F273, R-13-08, F271 v1.7, ミス 17
---

# F281-Phase2-B-mini 段階 B 設計記録 — Lane A1 本格評価走行

## 1. 目的

段階 A で実装した eval_mode を使い、Lane A1 (既存 patterns
status='candidate' 4,708 件) の個別期待値・勝率・DD を実測。
段階 C で Fujiwara が subset 抽出基準を確定するための判断材料を
取得する。

## 2. 重要な前提 (ミス 17 教訓適用)

本評価は F281 新方針 8 レーン構造のうち **Lane A1 単独**の評価。

- Lane A1: 既存 patterns 4,708 件 (本タスク対象)
- Lane B-H: 別 patterns / 別軸で別途評価予定 (Phase2-B / Phase3 で着手)

Lane A1 評価結果が芳しくなくても、F281 全体の評価とは独立。Fujiwara
戦略判断の選択肢は (i) Lane A1 採用 / (ii) Lane B-H 早期切替 /
(iii) 中間判断、いずれも有効。

## 3. 段階 A 結果からの参考データ

### 段階 A-fix-4 smoke test 結果 (期間 3 営業日、20 銘柄)

  events 75,612 件
  勝率 6.5% (570 / 8,793 closed)
  期待値 -11,847 円
  TP 570 件 (avg +16,734 円)
  SL 8,037 件 (avg -10,387 円)
  force_close 186 件 (avg -162,555 円)

### 本部観察 (参考、確定ではない)

F276 Phase 4-α/β (events 103,312 / 39,420、勝率 6.5-6.6%、期待値
-10K 円台) と整合。F271 v1.7 §6-7-quaternary「閾値依存性 = 構造不変」
原則の追加実証データ。ただし全体集計値であり、個別 pattern の期待値
分布は未確認 (本段階 B で取得する)。

## 4. 設計

### 4-1. 期間 + 銘柄拡大

  段階 A-fix-4: 3 営業日 × 20 銘柄 (smoke test)
  段階 B (本格): 20-60 営業日 × 全銘柄 (Core500 範囲推定)

具体パラメータ (Mac mini 実装時に CLI 仕様確認):
  --eval-mode (段階 A 機能)
  --end-date 2026-05-01 (Lane A1 patterns detected_at 最大値以後)
  --days-back 60 (3 → 60、約 20 倍に拡大)
  --no-promotion-check (smoke test と同様、F053 skip)
  --symbols-limit なし (全銘柄)
  staging 環境 (FIRE_ENV=staging DB_PATH=fire.staging.db
    LINE_DRY_RUN=true)

実行時間想定: 段階 A 走行 (3 営業日 × 20 銘柄 = 7-10 分) を基準に、
60 営業日 × 全銘柄 = 60/3 × 全銘柄/20 ≈ 数 100 倍 = 数時間規模の
可能性。Mac mini は走行前に推定時間を Fujiwara に共有、許容範囲を
確認してから走行開始。

### 4-2. 個別 pattern ランキング SQL

走行完了後、staging DB で個別 pattern の集計 SQL を実行:

  SELECT pattern_id,
         COUNT(*) as n_events,
         SUM(CASE WHEN event_type='virtual_tp' THEN 1 ELSE 0 END) as tp,
         SUM(CASE WHEN event_type='virtual_sl' THEN 1 ELSE 0 END) as sl,
         SUM(CASE WHEN event_type IN ('virtual_tp','virtual_sl','force_close')
              THEN metric_value ELSE 0 END) as total_pnl,
         AVG(CASE WHEN event_type IN ('virtual_tp','virtual_sl','force_close')
              THEN metric_value ELSE NULL END) as avg_pnl,
         (1.0 * SUM(CASE WHEN event_type='virtual_tp' THEN 1 ELSE 0 END))
           / NULLIF(SUM(CASE WHEN event_type IN ('virtual_tp','virtual_sl')
              THEN 1 ELSE 0 END), 0) as win_rate
  FROM paper_live_results
  WHERE run_mode='eval'
    AND run_id='[本走行 run_id]'
  GROUP BY pattern_id
  ORDER BY avg_pnl DESC;

(SQL 詳細は Mac mini 実装時に schema 確認後に調整可)

### 4-3. ランキング出力 + Vault 化

集計結果を 3 形式で出力:

(a) Top 100 patterns (avg_pnl 降順): 段階 C 判断材料の中核
(b) Bottom 100 patterns (avg_pnl 昇順): 凍結 / archive 候補
(c) 全体分布ヒストグラム: 期待値プラスの patterns 件数把握

出力先:
  - staging DB: 既存 paper_live_results に保存済 (再利用可能)
  - Vault: 03_design/F281_Phase2-B-mini_stageB_ranking_2026-05-XX.md
    に Top 100 + Bottom 100 + 統計サマリ記録

## 5. 受入基準

(1) 走行完了: tick error 0 件、events>>0 (段階 A の数十倍以上想定)
(2) 個別 pattern ランキング取得: 4,708 件全部の avg_pnl / win_rate
    集計成功
(3) Vault 出力: ranking 設計記録に Top 100 + Bottom 100 + 統計サマリ
    記録、commit + push
(4) staging のみ更新、production / develop DB 無触
(5) LINE 通知 dry_run で実送信ゼロ (LINE_DRY_RUN=true 強制化機能維持)
(6) Mac mini が走行前に Fujiwara に推定時間共有、許容後に走行開始
(7) Fujiwara 戦略判断材料の準備完了:
    - Lane A1 上位 N 件の期待値プラス pattern 件数
    - 期待値プラス pattern の特徴 (sector, layer, etc.)
    - Lane A1 全体としての構造的評価 (期待値プラス pattern が皆無 /
      少数 / 一定数 の判定)

## 6. 後続段階の選択肢 (Fujiwara 戦略判断)

### 選択肢 (i): Lane A1 採用継続

  Lane A1 上位 N 件 (例 100 件) で期待値プラス → approved_active 化
  (R-13-08 経路、Fujiwara 承認制) → Run a/b/c → Stage 3 Lane A1 開始

### 選択肢 (ii): Lane B-H 早期切替

  Lane A1 全件構造的にマイナス → Lane A1 は Paper Only 維持、
  F281-Phase2-B (Lane G/H + Lane B/C/D 順次追加) に早期着手
  Stage 3 開始判定は Lane A1 ではなく別 Lane で実施

### 選択肢 (iii): 中間判断 (Paper Only 期間延長等)

  Lane A1 期待値マイナスだがマイナス幅が小さい → Stage 2.5 (中間段階)
  新設、Paper Only 期間延長 + 構造改善案検討 (F119 重み再評価等)

## 7. 改訂履歴

- v1.0 (2026-05-08): 初版作成、段階 B 着手前
