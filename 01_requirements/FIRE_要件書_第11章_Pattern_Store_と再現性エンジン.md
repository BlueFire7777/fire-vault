---
type: requirement_chapter
chapter: "11"
title: "Pattern Store と再現性エンジン"
version: v3.4
updated: 2026-05-03
---

# 第11章: Pattern Store と再現性エンジン

「大量学習によるブラックボックスAI」ではなく、**「過去局面DB+類似局面検索+統計優位判定」**の構造とする。

## Pattern Store の階層管理

1. **LIBRARY**: 勝ちパターンの保管庫(Active Priority Set を最優先とする)
2. **CANDIDATE**: 検証中パターン
3. **DEATH NOTE**: 廃止・凍結パターン(N回失敗等で無条件即死させず、地合いや執行品質を考慮して「凍結」とする)
4. **REHAB**: いったん死んだが再検証候補の保管庫

## direction 軸 (v3.4 追加)

Pattern Store のパターンには、**direction = "long" (買い) / "short" (空売り)** を必須軸として持たせる。買いと空売りは類似度計算・スコア合成・劣化判定すべて分離する。

- 類似度検索は同じ direction 内でのみ行う (買いの過去局面と空売りの過去局面を混ぜない)
- Active Priority Set / Candidate / Frozen / DEATH NOTE / REHAB の階層は direction 別に独立管理
- レジーム適合度は direction で意味が反転する (買いは強気レジーム適合、空売りは弱気レジーム適合)

### 本章追加要件ID (v3.4)
- **R-11-08** Pattern Store に direction 軸 (long/short) を必須化
- **R-11-09** 類似度検索は同 direction 内でのみ実施、混合禁止
- **R-11-10** 階層管理は direction 別に独立 (Active Priority Set / Candidate / Frozen / DEATH NOTE / REHAB それぞれ direction 別)

## 最終執行判断(再現性スコア)

現在の特徴量、類似度、類似件数、パターン勝率、期待値、レジーム適合度、執行品質、Active Priority重み、リスク管理適合度の**合成**で判断する。
**類似件数が少なすぎる場合は参考止まり**とする。

## 本章の要件ID

- **R-11-01** 再現性エンジン = 過去DB+類似検索+統計優位判定(ブラックボックスAIにしない)
- **R-11-02** Pattern Store階層: LIBRARY(Active Priority Set)
- **R-11-03** Pattern Store階層: CANDIDATE
- **R-11-04** Pattern Store階層: DEATH NOTE(N回失敗即死禁止・凍結判定)
- **R-11-05** Pattern Store階層: REHAB
- **R-11-06** 最終執行判断(再現性スコア合成)
- **R-11-07** 類似件数少なすぎは参考止まり


---

## ナビゲーション

[[FIRE_要件書_第10章_スコアリングシステムと特徴量|← 第10章: スコアリングシステムと特徴量]]　|　[[FIRE_要件書_第12章_全自動運用モードと移行ステップ|第12章: 全自動運用モードと移行ステップ →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
- タスク: [[../02_todo/_index|TODO索引]]
- 運用ルール: [[../タスク運用ルール|タスク運用ルール]]
