# F281 設計記録: 複数戦略レーン管理型資産運用 OS

**作成日**: 2026-05-07
**起点**: F276 案 d 中止 (Phase 4-α/β 構造不変実証) + Fujiwara 戦略判断
         「単一聖杯探索 → 複数戦略レーン管理型資産運用 OS」根本転換
**採用プラン**: ★ プラン A (最小 MVP、Lane A1 単独 Stage 3 開始経路、5/14 頃)

---

## 1. 新方針の根本思想

### 1-1. Before (F276 までの方針)

  単一の最適な発火条件・閾値・score 設計を探索 (聖杯パターン)。
  Phase 4-α/β で「閾値依存性 = 構造不変」(§6-7-quaternary) を実証、
  単一閾値での events ≥ 50 + 期待値プラス両立は構造的に不可能と確定。

### 1-2. After (F281 新方針)

  **複数戦略レーン管理型資産運用 OS**。

  - 各レーン (Lane A-H) が独立した戦略 + 評価軸を持つ
  - レーンごとに Active Aggressive / Active Normal / Active Light /
    Paper Only / Watch Only / Hard Stop の稼働モードを切替
  - 期待値・地合い適性・劣化状況・執行品質・DD の 5 軸で各レーンを評価
  - レーン単位で独立稼働 (= OK になったレーンから本番運用、§6-22-quaternary)
  - 月次・年次運用益最大化を目指すフレームワーク (聖杯探索ではない)

---

## 2. 維持必須項目 (Fujiwara 提供文書「重要な既存前提」全数)

  - FIRE は完全自動発注ではない (兼業前提、判断負荷制限)
  - 発注は iSPEED / 楽天証券 Web で人間 (Fujiwara) が手動実行
  - LINE 通知で注文完成形 (lane_id / pattern_id / 稼働理由含む)
  - Computer Use 不採用 (発注自動化しない)
  - Playwright / Cron による強制クローズ自動化不採用
  - 強制クローズ = LINE 緊急アラート + 人間対応
  - Stage 3 Live Advisory が完成形、Full Auto は将来 (M+2 以降検討)
  - 兼業投資家通知量・判断負荷制限 (LINE 5 部屋構成 / 朝レポート)
  - 月次・年次運用益最大化 (損益直結優先、贅肉削減)

---

## 3. 中核レイヤー 3 つ

### 3-1. Strategy Portfolio Layer (25 項目、monthly_contribution オミット)

  各レーンの戦略・パラメータ・稼働状態を管理する上位レイヤー。

### 3-2. Market Regime Layer (13 種、段階別実装)

  地合い (強気 / 弱気 / レンジ / 決算相場 / テーマ集中相場 等) の判定。
  Phase 2-A は最小限 (TOPIX 単独)、Phase 3 で 13 種完全実装。

### 3-3. Do Not Trade Layer (Hard Block / Soft Block / Paper Only)

  リスク管理レイヤー。レーン単位 + 全体停止条件で発動。

  - Hard Block: 完全停止 (重大障害 / 連続損失 / 停止機能異常)
  - Soft Block: 縮小モード (DD 一定超過 / 劣化検知)
  - Paper Only: 実弾停止 + Paper Live 継続 (検証中レーン)

---

## 4. 評価軸 5 軸 (Fujiwara 核心定義文準拠)

  各レーンを以下 5 軸で評価し、稼働モードを切替:

  1. **期待値**
     - expected_value (基本期待値)
     - after_cost (手数料 + 税引後)
     - slippage_adjusted (執行 slippage 反映後)

  2. **地合い適性**
     - regime_dependency (各 regime での期待値変動)

  3. **劣化状況**
     - degradation_status (パターン劣化検知)
     - recent_X_trades_result (直近 X トレードの結果)

  4. **執行品質**
     - execution_quality (約定品質スコア)
     - slippage_adjusted_expected_value (執行込み期待値)

  5. **DD**
     - max_drawdown (歴史的最大 DD)
     - current_drawdown (現在 DD、回復モニタリング)

---

## 5. 切替モード 5 種 (Fujiwara 核心定義文準拠)

  上記 5 軸を踏まえて以下 5 種を切替:

  - 実弾通常ロット (Active Aggressive + Active Normal)
  - 実弾縮小ロット (Active Light)
  - Paper Live (Paper Only)
  - 監視のみ (Watch Only)
  - 停止 (Hard Stop)

---

## 6. 稼働モード 6 種 (Strategy Portfolio Layer 内)

  - **Active Aggressive**: 強気時 + 高品質パターン、ロット +25%
  - **Active Normal**: 通常運用、本部 default ロット
  - **Active Light**: 弱気時 + 劣化兆候、ロット -50%
  - **Paper Only**: 実弾停止、Paper Live で検証継続
  - **Watch Only**: 監視のみ (シグナル記録、エントリーなし)
  - **Hard Stop**: 完全停止

---

## 7. Lane A-H 役割定義 + 段階別実装計画

| Lane | 名称 | 性格 | 段階 |
|---|---|---|---|
| **A** | 高品質材料初動 (買い) | 主軸戦略、Phase 2-A 開始 | Lane A1 = 既存 4,700 件再分類 |
| **B** | 主役テーマ前場初動 (買い) | テーマ集中相場 | Phase 2-B (新規 patterns 抽出) |
| **C** | 前日強銘柄初押し再加速 (買い) | 順張り | Phase 2-B |
| **D** | スイング順張り (買い) | 5-10 日保有 | Phase 2-B |
| **E** | 長期高配当・成長株 (買い) | F101 TDnet IR 統合 | Phase 3 |
| **F** | 空売り Paper Live 専用 (売り) | M+2 以降、慎重設計 | Phase 3 |
| **G** | 防御・現金化 | リスク管理 | Phase 2-B 簡易 + Phase 3 本格 |
| **H** | 保有株管理 | ポジション管理 | Phase 2-B 簡易 + Phase 3 本格 |

---

## 8. Pattern Store 再構造化

### 8-1. 既存 patterns 4,700 件の扱い

  廃棄せず、**Lane A1 として再分類** (F273_BRE prefix → Lane A1 紐付け)。

### 8-2. 新構造 (3 階層)

  Strategy Portfolio → Lane → Pattern

  - Strategy Portfolio: 全 Lane の上位管理 (資金配分 / 稼働状態)
  - Lane: 戦略単位 (A-H、各々独立した期待値/地合い適性等)
  - Pattern: 発火条件 + features 組み合わせ (4,700 件 → 拡充)

---

## 9. 劣化検知ルール

### 9-1. 判定基準 11 項目

  1. 直近 N トレード勝率
  2. 直近 N トレード期待値
  3. 直近 X 営業日 DD
  4. score 安定性 (Phase 3 で監視)
  5. similarity 分散変化
  6. regime 別期待値偏差
  7. execution slippage 増加
  8. 連続損切回数
  9. force_close 増加
  10. 想定外 sl 到達率
  11. Pattern Store 内位置 (Layer 推移)

### 9-2. 状態遷移

  approved_active → watchlist → active_light → paper_only → rehab →
  frozen → death_note

  各遷移は Fujiwara 承認制 (R-13-08「エッジ定義変更」相当)。

---

## 10. 資金配分・ロット配分

  レーン単位 + 既存総資産ベース管理:

  - デイトレ通常 0.4%
  - 強気日 0.5%
  - 弱気日 0.25-0.3%
  - 実弾初月 0.2% (M=0、半減)
  - 日次 -1.2% で新規停止
  - 日次 -1.8% で当日終了
  - 週次 -3.0% で守備モード
  - 信用建玉は 15:10 までクローズ (R-05-10 強制クローズ規律)

---

## 11. KPI 体系 (損益直結優先、Fujiwara オミット項目除外、22 項目)

  ※ オミット: 目標 3000 万トラッキング / monthly_contribution /
              Dashboard 総資産進捗率 / KPI 月次目標進捗率 /
              Dashboard 見送り失敗候補分析 / 攻守判定高度ロジック

  維持 22 項目 (損益直結):
  - レーン別 win_rate / expected_value / sharpe_ratio
  - レーン別 DD (max / current)
  - レーン別 execution_quality
  - レーン別 regime_dependency
  - 全体 月次/週次/日次損益
  - Active Aggressive ロット適正度
  - Pattern 劣化件数
  - Force Close 発動回数
  - 強制クローズ成功率
  - 停止機能発動回数
  - LINE 通知到達率
  - Evaluation Agent 提案数
  - Stage 3 開始からの日数
  - Lane 切替回数 (Watch Only ↔ Active Normal 等)

---

## 12. Dashboard 設計 (損益直結優先、オミット項目除外)

  画面構成:
  - レーン別成績一覧 (Active Aggressive / Normal / Light)
  - 全体 月次/週次/日次損益
  - DD 状態 (現在 + 履歴)
  - 劣化検知アラート
  - Evaluation Agent 提案 (承認待ち / 採用済み / 却下済み)
  - LINE 通知履歴 (lane_id 別)
  - Stage 3 開始 / 凍結条件モニタリング

---

## 13. 朝レポート 13 順序 (Fujiwara 維持判断、本業中の情報収集軽減)

  Fujiwara 戦略判断で維持 (損益直結ではないが、本業中の情報収集軽減目的)。
  既存 13 順序を継続:

  1. 前日米株指数
  2. 為替
  3. 日経 225 / TOPIX 寄り前
  4. 日本株 sector 強弱
  5. 朝の 5 銘柄候補
  6. 各候補の発火 lane / pattern
  7. 各候補の期待値
  8. 各候補の DD 状態
  9. 各候補の執行品質
  10. 全体ロット配分
  11. 本日の Watch Only レーン
  12. 本日の Hard Stop レーン
  13. Evaluation Agent 改善提案 (要承認)

---

## 14. LINE 通知変更 (損益直結 + 判断材料、lane_id/pattern_id/稼働理由等)

  通知に含める情報:
  - lane_id (Lane A1 等)
  - pattern_id (発火 pattern)
  - 稼働理由 (Active Normal で採用、強気日でロット +0.1% 等)
  - 期待値 (lane 平均)
  - DD 状態
  - 注文完成形 (R-06-03 11 項目)
  - 強制クローズ条件

---

## 15. 段階別実装ロードマップ (プラン A)

### Phase 1 (本日、2026-05-07): 即時 Vault 化

  - F281 todo 起票 + 設計記録 + F271 v1.3 + log.md
  - 工数: 1 日

### Phase 2-A (3.5 日、Stage 3 開始経路): 最小 MVP 4 項目

  (1) lane_id 設計 + schema migration
  (2) レーン別成績集計 (evaluation/aggregators 拡張)
  (4) Do Not Trade 3 分類 (Hard Block / Soft Block / Paper Only)
  (8) Pattern Store の lane_id 対応 (既存 4,700 件 → Lane A1)

  完了 → Run a/b/c → R-32-01 7 項目 + F053 + F241 達成判定 →
  F235 + F266 並走 → ★Stage 3 開始 5/14 頃

### Phase 2-B (14 日、Stage 3 並走): 拡張実装

  基盤拡張 5:
  (3) 稼働モード 6 種完全
  (5) Dashboard 簡易版
  (6) 朝レポート 13 順序
  (7) LINE 通知 lane_id 追加
  (9) 劣化検知最小実装

  Lane G/H/B/C/D 実装 (各 1-3 日)

### Phase 3 (35 日、Stage 3 運用 1-3 ヶ月並走): 本質課題対応 + Lane E/F + 完全版

  本質課題 4:
  - score 感度分析
  - features 寄与度分析
  - patterns 抽出見直し (working tree の find_firing_pairs_multi 再活用)
  - 中間検証経路

  Phase 2-A 後送り 3:
  - Market Regime 判定 13 種完全実装
  - regime_dependency 評価軸
  - 執行品質評価

  Lane E (F101 TDnet IR 統合) + Lane F (空売り Paper Live)

  Lane G/H 本格版 + Lane B/C/D patterns 拡充

  Strategy Portfolio Layer 25 項目完全版

  Market Regime Layer 13 種完全判定

  レーン別 KPI 完全体系 (22 項目)

  高度な資金配分最適化 + 自動レーン昇格・降格

---

## 16. 既存資産の再分類方針 (廃棄せず)

  - F273_BRE prefix 4,700 件 → Lane A1 として再分類
  - その他 9 件 (template / theme_flow / policy_benefit) → Lane E 候補
  - F273 phase2bc の Layer 1 seeder 実装は Phase 3 patterns 抽出見直しで
    再活用 (working tree の find_firing_pairs_multi 含む)

---

## 17. Stage 3 開始判定構造

### 17-1. 過去データ判定経路

  R-32-01 ※注記により、Run a/b/c (過去データ Backtest) で 20 営業日 +
  50 トレード達成可。Lane A1 単独で:

  - Run a (1 day): events ベンチマーク
  - Run b (60 営業日): 期待値 / DD 確認
  - Run c (120 営業日): 統計的有意性確認

### 17-2. ケース別経路

  - ケース 1 (R-32-01 7 項目達成): Lane A1 Stage 3 開始
  - ケース 2 (期待値プラス未達): 本質課題対応追加実施 (5-7 日遅延)
  - ケース 3 (再実測でも未達): 戦略再設計検討
  - ケース 4 (events 不達): 段階 2-B 着手で他 Lane 並走

---

## 18. Stage 3 開始凍結条件 (4 段階)

  1. Run a/b/c 期待値プラス未達 → 本質課題対応 5-7 日 → 5/22 頃まで遅延
  2. 再実測でも未達 → 戦略再設計
  3. Stage 3 開始後 1 ヶ月で実取引マイナス → 全レーン Paper Only 降格
  4. Stage 3 開始後 3 ヶ月で月次累計マイナス → FIRE 一時休止

---

## 19. 採用根拠

### 19-1. Phase 4-α/β 実測 (Vault 一次資料)

  - Phase 4-α (env=0.55): events 103,312、勝率 6.6%、期待値 -10,367 円
  - Phase 4-β (env=0.60): events 39,420、勝率 6.5%、期待値 -10,108 円
  - 結論: 閾値微調整で events は 1000 倍変動、勝率/期待値は構造不変
    (§6-7-quaternary)

### 19-2. Vault §6-3 警告

  F273_phase2bc_results_2026-05-04.md line 223:
  > 「ただし features 値域が狭い (vwap_position レンジ -0.18〜0.15) ため、
  > 識別力向上は限定的」

### 19-3. Fujiwara 戦略観

  - a 懐疑: 機械学習で未来予知は不採用 (個人投資家のリソースで誤った方向)
  - b 確信: FIRE は日本株で稼げるシステム (複数戦略レーン管理が正攻法)

### 19-4. Fujiwara 戦略方針

  - 損益直結優先
  - 贅肉削減 (オミット 6 項目)
  - 朝レポート維持 (本業中の情報収集軽減目的)
  - 新メモリ #24 (コスト許容、ただし三重確認)

---

## 20. 重要制約 (Fujiwara 提供文書 15 項目)

  1. FIRE は完全自動発注ではない
  2. 発注は iSPEED/楽天証券 Web 手動
  3. LINE 通知で注文完成形
  4. Computer Use 不採用
  5. Playwright/Cron 強制クローズ自動化不採用
  6. 強制クローズ = LINE 緊急アラート + 人間対応
  7. Stage 3 Live Advisory が完成形、Full Auto は将来
  8. 兼業投資家通知量・判断負荷制限
  9. 月次・年次運用益最大化
  10. Mac mini 24 時間稼働 (シミュレーション主導戦略)
  11. Stage 飛ばし禁止 (R-32-03)
  12. Evaluation Agent 自動反映禁止 (R-13-08 承認制)
  13. Pattern Store 4 階層構造維持
  14. 既存 features 廃棄禁止 (再分類で活用)
  15. Run a/b/c の過去データ判定経路活用 (R-32-01 ※注記)

---

## 21. オミット項目 6 件 (Fujiwara 戦略判断、損益直結せず)

  以下は損益直結せず、本指示書範囲外:

  1. 目標 3000 万トラッキング
  2. monthly_contribution
  3. Dashboard 総資産進捗率
  4. KPI 月次目標進捗率
  5. Dashboard 見送り失敗候補分析
  6. 攻守判定高度ロジック

  朝レポート維持 (本業中の情報収集軽減目的、Fujiwara 判断)

---

## 関連リンク

- F281 todo: [[../02_todo/F281_strategy_portfolio_migration]]
- F271 v1.3: [[F271_v1.3_2026-05-07]]
- F276 Phase 4-α/β: [[F276_phase4_2026-05-05]]
- F273 phase2bc §6-3: [[F273_phase2bc_results_2026-05-04]]
- F275 完了: [[F275_similarity_optimization_complete_2026-05-04]]
- 要件書 R-32-01: [[../01_requirements/FIRE_要件書_第32章_本番移行基準の厳密化_穴10対策_]]
- 要件書 R-13-08: [[../01_requirements/FIRE_要件書_第13章_Evaluation_Agent_と_Dashboard]]
