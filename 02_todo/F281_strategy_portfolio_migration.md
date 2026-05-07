---
id: F281
phase: P10: 戦略レイヤー再設計
priority: 最優先
status: 着手中 (Phase 1 = 本日 Vault 化、Phase 2-A 着手準備)
owner: Fujiwara
depends_on: [F276 中止、F273/F275 Phase 4-α/β 結果]
chapter: "32, 13, 19, 14, 21"
created: 2026-05-07
updated: 2026-05-07
---

# F281: 複数戦略レーン管理型資産運用 OS 移行

## 1. 概要

F276 (events ≥ 50 達成) 案 d 中止後の新方針切替。Phase 4-α/β 実測で
「閾値依存性 = 構造不変」(§6-7-quaternary) + Vault §6-3 line 223
「識別力向上は限定的」警告 + Fujiwara 戦略観 (a 懐疑/b 確信、機械学習
未来予知不採用、FIRE は日本株で稼げるシステム) + 戦略方針 (損益直結優先、
贅肉削減) を踏まえ、単一聖杯パターン探索から **複数戦略レーン管理型
資産運用 OS** への根本転換。

採用プラン: ★ プラン A (最小 MVP、Lane A1 単独 Stage 3 開始経路、5/14 頃)

## 2. 移行根拠

1. Phase 4-α (env=0.55): events 103,312、勝率 6.6%、期待値 -10,367 円
2. Phase 4-β (env=0.60): events 39,420、勝率 6.5%、期待値 -10,108 円
3. 結論: 閾値微調整で events 数は 1000 倍変動、勝率/期待値は構造不変
4. Vault F273_phase2bc §6-3 line 223 が「識別力向上は限定的」警告
5. Fujiwara 戦略観: 機械学習未来予知不採用、複数戦略レーンで月次・年次
   利益最大化を目指すフレームワーク優先

## 3. Phase 構成

### Phase 1: 即時 Vault 化 (本日、2026-05-07)

  - F281 todo 起票 (本ファイル)
  - F281 設計記録: ~/fire-vault/03_design/F281_strategy_portfolio_design_2026-05-07.md
  - F271 v1.3 一括化: ~/fire-vault/03_design/F271_v1.3_2026-05-07.md
  - log.md milestone 追記
  - F276 status 「中止 (F281 統合)」変更

  工数: 1 日 (Vault commit + push)

### Phase 2-A: 最小 MVP 4 項目 (3.5 日、Stage 3 開始経路)

  基盤項目 (Stage 3 開始に必須):

  (1) lane_id 設計
      - patterns / paper_live_results / paper_live_positions に lane_id
        カラム追加 (schema migration)
      - 既存 patterns 4,700 件を Lane A1 として再分類 (廃棄せず)

  (2) レーン別成績集計
      - evaluation/aggregators.py 拡張、lane_id 別 win_rate / expected_value /
        sharpe_ratio 集計
      - F119 Evaluation Agent と integrate

  (4) Do Not Trade 3 分類
      - Hard Block (停止) / Soft Block (Paper Only) / Paper Only (実弾なし)
      - patterns.status 拡張 (frozen / paper_only / death_note を統合)

  (8) Pattern Store の lane_id 対応
      - 既存 patterns 4,700 件 (F273_BRE prefix) を Lane A1 として再分類
      - 新規追加 patterns は lane_id 必須

  完了 trigger: Run a/b/c 順次実施 → R-32-01 7 項目達成 → F235 +
                F266 並走 → Stage 3 開始 5/14 頃

### Phase 2-B: 拡張実装 (14 日、Stage 3 並走)

  基盤拡張 5 項目:

  (3) 稼働モード 6 種完全
      - Active Aggressive / Active Normal / Active Light / Paper Only /
        Watch Only / Hard Stop
      - patterns.operational_mode カラム追加

  (5) Dashboard 簡易版
      - レーン別成績可視化 (損益直結優先)
      - オミット項目除外 (3000 万トラッキング / monthly_contribution 等)

  (6) 朝レポート 13 順序 (Fujiwara 維持判断)
      - 本業中の情報収集軽減目的、現状運用継続

  (7) LINE 通知 lane_id 追加
      - 注文通知に lane_id / pattern_id / 稼働理由を含める

  (9) 劣化検知最小実装
      - 判定基準 11 項目 + 状態遷移 (approved_active → watchlist →
        active_light → paper_only → rehab → frozen → death_note)

  Lane G/H/B/C/D:

  - Lane G (防御・現金化、簡易版): 弱気日のロット縮小 + ヘッジ判断
  - Lane H (保有株管理、簡易版): 既存ポジション売り判断
  - Lane B (主役テーマ前場初動、新規 patterns): F101 TDnet 連携
  - Lane C (前日強銘柄初押し再加速): 過去 features 抽出
  - Lane D (スイング順張り): 5-10 日保有想定

### Phase 3: Stage 3 運用後本格実装 (35 日、運用 1-3 ヶ月並走)

  本質課題対応 4 項目:
  - score 感度分析 (各 weight の寄与度)
  - features 寄与度分析 (どの feature が勝率/期待値に効くか)
  - patterns 抽出ロジック見直し (working tree の find_firing_pairs_multi
    再活用)
  - 中間検証経路 (Phase 2-A/B の score 安定性監視)

  Phase 2-A 後送り 3 項目:
  - Market Regime 判定 (13 種完全実装)
  - regime_dependency (各 lane の regime 別期待値)
  - 執行品質評価 (slippage_adjusted_expected_value)

  Lane E/F:
  - Lane E (長期高配当・成長株、F101 TDnet IR 統合)
  - Lane F (空売り Paper Live 専用、M+2 以降、慎重設計)

  Lane G/H 本格版 + Lane B/C/D patterns 拡充

  Strategy Portfolio Layer 25 項目完全版 (monthly_contribution オミット)

  Market Regime Layer 13 種完全判定

  レーン別 KPI 完全体系 (22 項目)

  高度な資金配分最適化 + 自動レーン昇格・降格

## 4. Stage 3 開始時期目標 + 凍結条件

### 開始時期目標
  - **2026-05-14 頃** (Phase 2-A 完了 + Run a/b/c + F235 + F266)

### 開始凍結条件
  1. Run a/b/c 期待値プラス未達 → 本質課題対応追加実施 (5-7 日、5/22 頃まで遅延)
  2. 再実測でも未達 → 戦略再設計検討
  3. Stage 3 開始後 1 ヶ月で実取引マイナス → 全レーン Paper Only 降格
  4. Stage 3 開始後 3 ヶ月で月次累計マイナス → FIRE 一時休止

## 5. オミット 6 項目 (Fujiwara 戦略判断、損益直結せず)

  本指示書範囲外:
  - 目標 3000 万トラッキング
  - monthly_contribution
  - Dashboard 総資産進捗率
  - KPI 月次目標進捗率
  - Dashboard 見送り失敗候補分析
  - 攻守判定高度ロジック

  朝レポート維持 (Fujiwara 判断、本業中の情報収集軽減目的)

## 6. F276 中止経緯 (本ファイル起票根拠)

  - F276 案 d Run a 実測未着手 (本日中止)
  - working tree (M scripts/seed_pattern_layer1.py、find_firing_pairs_multi
    追加) は維持、Phase 3 patterns 抽出ロジック見直しで再活用予定
  - 既存 patterns 4,700 件は廃棄せず Phase 2-A で Lane A1 として再分類

## 7. 関連リンク

- F281 設計記録: [[../03_design/F281_strategy_portfolio_design_2026-05-07]]
- F271 v1.3: [[../03_design/F271_v1.3_2026-05-07]]
- F276 Phase 4-α/β 実測: [[../03_design/F276_phase4_2026-05-05]]
- F273 phase2bc §6-3: [[../03_design/F273_phase2bc_results_2026-05-04]]
- F275 完了 line 161: [[../03_design/F275_similarity_optimization_complete_2026-05-04]]
- 要件書 R-32-01: [[../01_requirements/FIRE_要件書_第32章_本番移行基準の厳密化_穴10対策_]]
- 要件書 R-13-08: [[../01_requirements/FIRE_要件書_第13章_Evaluation_Agent_と_Dashboard]]
