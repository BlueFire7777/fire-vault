---
id: F266
phase: P9: Stage 3 移行
priority: 最優先
status: ブロック中
owner: Fujiwara
depends_on: [F053, F241, F267, F230]
chapter: "32,40"
created: 2026-05-03
updated: 2026-05-04
---

# [F266] Stage 3 最終ゲート (合議体判定)

## 基本情報

- フェーズ: P9: Stage 3 移行
- カテゴリ: Stage gate
- 優先度: 最優先
- 依存: F053, F241, F267, F230 (Run a/b/c)
- 担当: Fujiwara (最終承認者)
- 想定工数: 0.5 日 (判定 + Markdown レポート発行のみ)
- ステータス: ブロック中 (F267 完了待ち / events=0 課題で凍結)
- 着手日: -
- 完了日: -

## タスク詳細

Paper Live (Stage 2) → Live Advisory (Stage 3) 移行の **最終合議体ゲート**。
F053 5 項目 / F241 9 項目 / events>0 / Fujiwara 明示承認 のすべてが揃って
初めて Stage 3 開始を許可する。

機能的には F241 (`live_advisory_check.py`) の延長だが、F266 は判定の
**統合と最終承認** を担う。具体的には:

- F267 の Run a/b/c が events>0 で完走していること
- F053 promotion_check が 4/5 以上 PASS (sample_size / expected_value /
  halt / close / accuracy)
- F241 9 項目が 9/9 PASS (F053 5 項目 + LINE 5 部屋 + 緊急アラート +
  Fujiwara 受容 + F119 提案書)
- Fujiwara が `--fujiwara-accept` で明示承認
- 合議体判定 Markdown レポート発行 (`reports/stage3_gate_YYYY-MM-DD.md`)

ブロック理由 (2026-05-04 時点):
- 2026-05-03 夜の Chain で 3 run すべて events=0 → F053 4/5 FAIL
- F267 で features 整備中、Run a 再走で events>0 確認待ち

## 成果物

- `~/fire/scripts/stage3_gate.py` 新規 (F241 ラッパー + 統合判定)
- `~/fire/reports/stage3_gate_YYYY-MM-DD.md` (判定 Markdown レポート)
- (任意) `~/fire/.stage3_started` フラグファイル (Stage 3 開始を運用側が
  検知できるマーカー)

## 関連ドキュメント

- 要件書: 第 32 章 (本番移行基準) / 第 40 章 (実装ロードマップ Stage 0〜3)
- 関連タスク: [[F053_Paper_Live_昇格基準実装|F053]] (5 項目判定、流用)
- 関連タスク: [[F241_live_advisory_check|F241]] (9 項目判定、本タスクの
  実装上のベース)
- 関連タスク: [[F267_features_pipeline|F267]] (前提、events>0 確認の
  ブロック解消)
- 関連タスク: [[F230_Paper_Live_batch_replay|F230]] (前提、Run a/b/c 走行)
- 関連タスク: [[F119_Evaluation_Agent]] (提案書生成、F241 経由で連携)
- 関連タスク: [[F236_LINE_5段階アラート]] (5 部屋疎通 + 緊急アラート)
- 学び: [[log.md]] (2026-05-04 朝セクション、ブロック確定経緯)

## 進捗チェックリスト

- [ ] F267 完了 (Run a で events>0 確認)
- [ ] Run b (60 営業日) 完走、events>0
- [ ] Run c (120 営業日) 完走、events>0
- [ ] F053 promotion_check で 4/5 以上 PASS (3 run のうち最良)
- [ ] F241 9 項目で 9/9 PASS
- [ ] LINE 5 部屋疎通テスト
- [ ] 緊急アラート 5 つ launchd 登録確認 (14:45/14:55/15:05/15:10/15:15)
- [ ] Fujiwara `--fujiwara-accept` 明示承認
- [ ] 合議体判定 Markdown レポート発行
- [ ] Stage 3 開始通知 (LINE 経由)

## 作業ログ

- **2026-05-03 夜**: F260 Block 4 完了に伴い Stage 3 移行 P0 として起票
- **2026-05-04 朝**: Chain 完走 events=0 でブロック確定。F267 (features
  パイプライン整備) 完了待ちの状態に
- **2026-05-04 (現在)**: F267 backfill 完了、Run a 再走中

## 完了条件

3 run (Run a/b/c) すべてで events>0、F053 + F241 を統合した判定で **9/9 +
Fujiwara 明示承認** が揃い、合議体判定 Markdown レポートが発行された
状態。

Stage 3 (Live Advisory) が開始され、楽天証券 iSPEED で人間 (Fujiwara)
による発注フェーズに移行可能になる。
