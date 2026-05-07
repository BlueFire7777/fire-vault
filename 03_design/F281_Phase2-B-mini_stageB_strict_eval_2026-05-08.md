---
title: F281-Phase2-B-mini eval_mode_strict 設計記録 (暫定仕様)
date: 2026-05-08
phase: F281-Phase2-B-mini
stage: B-extended
status: 暫定仕様 (シミュレーション後に確定判定)
related: F281 v1.2 §5-bis (Active Light 仕様), ミス 18, 段階 A-fix-4
trigger: 段階 B-3 走行中 grep でミス 18 (Active Light 仕様乖離) 確定検出 + Fujiwara 指摘 (確率論的検証 + シミュレーション後確定原則)
---

# F281-Phase2-B-mini eval_mode_strict 設計記録 (暫定仕様)

## 1. 目的

段階 A eval_mode は STATUS_ADJUSTMENT bypass のみ実装、Active Light
制約は未実装。eval_mode_strict は Active Light 制約を実装した
評価モード、Stage 3 想定の運用条件下で Lane A1 patterns 4,708 件の
真の期待値ランキングを取得する。

★ 本仕様は **暫定**。シミュレーション結果分析後、Fujiwara 戦略判断で
確定 (memory 改訂) または再設計 (v0.3 修正) のいずれかに進む。

## 2. 暫定 Active Light 制約 (5 項目)

(a) 1 日最大 5 trade
(b) 1 trade 100 株固定
(c) 1 日累積損失 stop -0.5% (= -50,000 円 / 1000 万円)
(d) DD -2% で paper_only モード切替、DD 回復閾値で解除
    暫定: DD 改善で -1% まで回復 → 実弾再開
(e) pattern × symbol × day で max 1 entry (累積エントリー禁止)
(f) Fujiwara 手動承認制 (Stage 3 開始時)

★ 当初候補「連続 3 件損失 pause」は確率論的検証 (勝率 70% で 100 trade
   中 93% で発生) で削除確定、本仕様には含めない。

## 3. シミュレーション後の判定基準

シミュレーション結果 (段階 B-strict-3 完了後) で以下を分析:

(1) 各制約の発動頻度
    - (a) 1 日 5 trade 上限到達日数 / 全営業日
    - (c) -0.5% halt 発動日数 / 全営業日
    - (d) DD -2% paper_only 移行回数、回復閾値 -1% 到達までの期間
    - (e) max 1 entry 制約による entry skip 件数

(2) events / win_rate / 期待値の変化
    - 段階 B-3 (制約なし) との比較で乖離度を定量化
    - 期待値プラス pattern 件数の変化

(3) 暫定仕様の妥当性判定
    Pattern A 妥当: 各制約が想定通り発動、events 統計取得可能、
                    Stage 3 想定との乖離小
    Pattern B 過剰: 制約が頻繁に発動しすぎて events 不足 = 評価精度低下
    Pattern C 不足: 制約発動頻度が低く、結果が制約なしと変わらず =
                    Active Light の本来目的未達
    Pattern D 想定外: Fujiwara memory に未記載の構造問題発覚 = 仕様
                      根本見直し

(4) 結果次第の進行:
    Pattern A → memory 改訂 + 段階 C 進行
    Pattern B → 制約緩和 (例: max 2 entry, 累積損失閾値 -1%) で再走行
    Pattern C → 制約強化 (例: max 1 trade/day) で再走行
    Pattern D → 設計根本見直し、本部品質改善ブロック第 5 弾起票

## 4. 実装スコープ (段階 B-strict-1 〜 B-strict-4)

- B-strict-1: 設計記録 Vault 化 (本ドキュメント) + ミス 18/19/20 Vault 化
              (本部品質改善ブロック第 4 弾)
- B-strict-2: account.py / position.py / tick.py 修正 + run_mode enum
              拡張 (eval_strict 値追加) + tests/
- B-strict-3: staging で smoke test (3 営業日 × 全銘柄、約 7-15 分)
              + 本格走行 (案 X 同等の 60 営業日 or Fujiwara 戦略判断
              で短縮)
- B-strict-4: シミュレーション結果分析 + 暫定仕様妥当性判定 +
              段階 C 進行 / 再設計判断

## 5. フラグ経路

CLI: paper_live batch-replay --eval-mode-strict
  ↓
BatchReplayRunner(strict_active_light=True) (eval_mode=True 内包)
  ↓
PaperLiveRunner.start_session(strict_active_light=True)
  ↓
extract_candidates / 既存経路 (eval_mode=True 動作継続)
  ↓
account.py 内で daily_realized_pnl + daily_trade_count track 新規
  ↓
position.py 内で entry 直前に check_active_light_constraints():
  - if daily_trade_count >= 5: skip entry
  - if pattern × symbol × day で既に entry 済: skip
  - if account.is_halted: skip
  - if account.in_paper_only: skip (paper trade は記録のみ)
  ↓
tick.py 内で日付変わり検出 + reset_daily_counters()

## 6. 実装上の重要事項

(1) 累積エントリーロジック (existing.quantity + qty) を strict モード時
    のみ disable
(2) DD 計算ロジック新規追加 (account.py): daily_realized_pnl /
    initial_capital ベース
(3) paper_only モード時の VirtualPosition は status='paper_only' で
    記録、realized_pnl は計算するが current_capital には反映しない
(4) 日付変わり検出: tick の as_of_dt の date 部分が前回と異なる時に
    reset 実行

## 7. 受入基準

(1) eval_strict=True 走行で daily_trade_count > 5 の事象ゼロ
(2) eval_strict=True 走行で同 pattern × symbol × day で 2 回以上
    entry 事象ゼロ
(3) is_halted 発動時、当日中の新規 entry 件数ゼロ
(4) DD -2% 到達時、paper_only モード移行確認
(5) staging のみ更新、production / develop DB 無触
(6) tick error 0 件、LINE dry_run 実送信ゼロ (二重防御維持)
(7) Codex pre-commit 一発通過、--no-verify 不使用

## 8. 段階 C 判断材料 (本タスク完了後)

eval_mode (制約なし、段階 B-3 結果) と eval_mode_strict (制約あり、
本タスク結果) の両走行データを Fujiwara が比較:

  Pattern ランキング比較:
    Top 100 (eval_mode): 制約なし理論最大期待値順
    Top 100 (eval_strict): Stage 3 実取引相当の期待値順
    一致度 / 乖離度 = pattern の構造的優劣指標

  全体集計比較:
    eval_mode 期待値 X / eval_strict 期待値 Y
    乖離小 → Lane A1 採用検討
    乖離大 (制約下大幅悪化) → Lane B-H 切替検討

  分岐 i/ii/iii (再掲):
    (i) Lane A1 採用継続: eval_strict で期待値プラス pattern 一定数あり
    (ii) Lane B-H 早期切替: eval_strict で期待値プラスほぼなし
    (iii) 中間判断: eval_mode 良いが eval_strict で大幅悪化 = 構造改善案

## 9. 改訂履歴

- v0.1 (2026-05-08、本部初稿): 内部レビュー前
- v0.2 (2026-05-08、Fujiwara レビュー反映、暫定仕様確定):
    - 論点 1: 連続 3 件損失 pause 削除 (確率論的検証)
    - 論点 2: pattern × symbol × day max 1 entry 採用
    - 論点 3: DD 回復閾値方式採用 (-1% 復帰)
    - 「シミュレーション後確定」原則明記 (v1.0 への移行はシミュ後)
- v1.0 (シミュレーション結果分析後、Fujiwara 確定後): 反映予定
