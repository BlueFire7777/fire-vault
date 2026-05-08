---
id: F285
phase: P4: 中長期 Lane (Research Lane)
priority: 高 (Phase R0 仕様設計完了、Phase R0 feasibility precheck 着手前)
status: 仕様設計完了 (HQ 5 点修正反映 v1.0)、Phase R0 precheck 待機
owner: Fujiwara
depends_on: [F100 J-Quants V2 daily, F101 TDnet, F104 indices daily, F119 Evaluation Agent, F200 3 レーン戦略構造, F276 indices spec]
chapter: "27"
created: 2026-05-08
updated: 2026-05-08
related: F285_Research_Lane_requirements_and_spec_2026-05-08, F281_Lane_C_design_2026-05-08, F210_phase_1b, F200_3レーン戦略構造, F119_Evaluation
---

★ **F285 Research Lane 中長期銘柄発掘エンジン**: HQ 並行作業指示
   (2026-05-08、F284/F105 c6 backfill PID 92822 走行中の vault 作業)
   で仕様 + 要件定義を vault 化。Phase R0 仕様設計完了、HQ 5 点修正
   反映済 (v1.0)。

# F285 中長期 Research Lane 仕様 + 実装計画

## サマリ

Sector Flow / Cyclical Value / Earnings Preview / Dividend Growth の
4 Agent + Watchlist Ranker で中長期銘柄を発掘、毎朝 Research 専用
LINE 部屋 (or 朝レポート枠) に Top10 を配信。短期 Lane C との接続は
5 段階 (A1/A2 boost / B neutral / C risk flag / D HQ review)。
Phase R0-R6 で段階着手、Phase R5 受入合格まで本番運用しない。

## 詳細仕様

[[F285_Research_Lane_requirements_and_spec_2026-05-08|仕様書 v1.0]]
を参照 (15 章 + メタ情報、要件 ID R-285-01〜14)。

## Phase 着手順

| Phase | 内容 | 目安期間 | 状態 |
|---|---|---|---|
| **R0** | 仕様設計 + data feasibility precheck | 1 週間 | **仕様完成 ✅、precheck 残** |
| R1 | Sector Flow + Cyclical Value 実装 (Standard daily) | 2-3 週間 | 未着手 |
| R2 | Earnings Preview + Dividend Growth 実装 (Phase R0 precheck 結果次第) | 2-3 週間 | 未着手 |
| R3 | Watchlist Ranker 統合 (weighted_score + A1/A2/B/C/D) | 1 週間 | 未着手 |
| R4 | 朝レポート + Portfolio + Lane C 接続 | 1-2 週間 | 未着手 |
| R5 | shadow mode 受入評価 (12 指標) | 3-6 ヶ月 | 未着手 |
| R6 | Live 接続 (R5 合格後) | (継続) | 未着手 |

## HQ 確認済み修正点 (v1.0 反映済)

1. **A1/A2 定義** = weighted score + 主要 Agent 強度の **AND 条件**
   - A1: weighted_score >= 80 AND 主要 Agent (score >= 0.7) 2 以上
   - A2: weighted_score >= 65 AND 主要 Agent 1 以上
   - hit 数のみで判定しない

2. **Lane C 接続 5 段階**
   - A1/A2: confidence boost
   - B: neutral
   - C: risk flag (優先度低下 / 追加条件要求)
   - D: 原則除外、ただし当日主役テーマ / 出来高急増 / 明確材料あれば
     HQ レビュー対象

3. **Phase R2 Premium 前提を断定しない**
   - Phase R0 feasibility precheck で Standard 取得可否確定
   - 追加課金が必要な場合は Fujiwara 判断必須
   - 不足分は Standard 範囲内で代替手段検討 (TDnet 既 + 公開有報パース等)

4. **LINE 通知先**
   - Research Lane 専用 LINE 部屋 (新設) または 朝レポート枠
   - 短期 ENTRY 部屋と **混同しない** 設計
   - 部屋追加 vs REPORT 枠は Phase R4 で HQ 承認

5. **受入評価指標 7 種追加** (合計 12 指標)
   - Top10 1m/3m/6m リターン
   - TOPIX 相対リターン
   - 業種指数相対リターン
   - 最大下落率
   - 決算後ギャップ勝率
   - 増配 / 上方修正的中率
   - 買わなかった候補の機会損益 (selection bias 検証)
   - + Sharpe / winrate × 期待値 / Fujiwara 受容

## 次の着手 (Phase R0 precheck)

c6 backfill 完了後 / Phase C2 完了後 / その他 HQ 判断で着手:

1. J-Quants /fins/details が Standard で動くか実検証
2. /fins/statements が Standard で動くか実検証
3. /fins/dividend が Standard で動くか実検証
4. 業種コードマスタ (TOPIX 33 / J-Quants 17) の取得経路確定
5. 不足分は代替手段検討 (TDnet 既 / 公開有報 PDF パース等)
6. 追加課金が必要な API を特定 → Fujiwara 判断要請

precheck 結果をもとに Phase R1 / R2 の実装範囲を確定。

## 制約 (R-03-01 / R-13-08 / FIRE 全体ルール準拠)

- 自動発注禁止、提案のみ (Fujiwara 手動発注)
- Stage 飛ばし禁止 (Phase R5 受入合格まで本番運用しない)
- Evaluation Agent 提案制 (rank 閾値 / weight 等は F119 提案 + Fujiwara 承認)
- LINE 通知混同禁止 (短期 ENTRY 部屋に Research を入れない)
- 追加課金は Fujiwara 個別判断必須

## 関連リンク

- [[F285_Research_Lane_requirements_and_spec_2026-05-08|F285 仕様書 v1.0]] (本タスクの仕様詳細)
- [[F281_Lane_C_design_2026-05-08|F281 Lane C]] (短期 Lane C 接続先、5 段階)
- [[F210_phase_1b|F210 Phase 1B]] (3000 万円 2 年目標)
- [[F200_3レーン戦略構造|F200]] (Lane 全体設計)
- [[F119_Evaluation|F119 Evaluation Agent]] (受入評価実施者)
- [[F101|F101 TDnet]] (Earnings Preview / Dividend Growth ソース)
- [[F100|F100 J-Quants V2]] (daily / fins データソース)
