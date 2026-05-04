---
id: F267
phase: P9: 運用・保守
priority: 最優先
status: 実装中
owner: Claude (Mac mini)
depends_on: [F100, F266]
chapter: "20,32,40"
created: 2026-05-04
updated: 2026-05-04
---

# [F267] features 生成パイプライン整備

## 基本情報

- フェーズ: P9: 運用・保守
- カテゴリ: Runtime
- 優先度: 最優先
- 依存: F100, F266
- 担当: Claude (Mac mini)
- 想定工数: 1日
- ステータス: 実装中
- 着手日: 2026-05-04
- 完了日: -

## タスク詳細

夜間 Chain (full_chain.sh) が exit 0 で完走したのに 3 run すべて
events=0 / trades=0 / positions=0。原因は features テーブルが
**87 行 / 5 銘柄 / 1 週間** しかなく、`tick.py:164` の `_has_features`
で全 4,452 銘柄が即 continue されていたため (仮説 A 確定、market_prices_daily
の 0.0009%)。

修正案 1 (2026-05-04 朝 Fujiwara 承認): `extract_features.py` を batch 化、
TOPIX Core500 代替を universe として導入、F101 完了反映で material collector
を ready=True 化。

主要な変更:
- `extract_features.py` の `--code` 必須を解除 (`--codes` / `--scope` /
  `--all-symbols` 受付)
- `--parallel N` で `ThreadPoolExecutor` 並列実行 (default 1)
- `--business-days-only` で `market_prices_daily.DISTINCT date` のみ走査
- material collector の `ready=True` 化 + `run_material_collector` 新規実装
  (announcements 検索 → MaterialEvent 化 → events 0 件でもデフォルト 6
  features を書き込み)
- TOPIX Core500 代替 universe (流動性 60 日窓 `AVG(turnover_value) DESC
  LIMIT 500`、`HAVING n_days >= 30`) を `data/universe/core500.txt` に固定

## 成果物

- `scripts/jobs/extract_features.py` 拡張 (commit `c73da95`、batch モード +
  universe + material wiring)
- `scripts/jobs/backfill_features.py` 新規 (薄いラッパー、`--scope core500
  --from --to --parallel 12` の頻出パターン)
- `data/universe/core500.txt` 新規 (500 銘柄 + ヘッダコメント、計 508 行)
- `data/universe/README.md` 新規 (抽出根拠 + 差し替え可能性)
- `tests/scripts/jobs/test_extract_features.py` 拡充 (12 → 20 PASS)
- `03_design/F267_implementation_2026-05-04.md` (実装レポート)

## 関連ドキュメント

- 設計書: [[03_design/F032_F054_diagnosis_2026-05-04|F032/F054 診断レポート]]
- 設計書: [[03_design/F267_implementation_2026-05-04|F267 実装レポート]]
- 関連タスク: [[F266_Stage3_最終ゲート|F266]] (本タスクのブロック対象)
- 関連タスク: [[F268_announcements_backfill|F268]] (副因 B 対応)
- 関連タスク: [[F269_chain_assertions|F269]] (再発防止 = Chain 異常検知)
- 関連タスク: [[F230_Paper_Live_batch_replay|F230]] (Run a/b/c 走行先)

## 進捗チェックリスト

- [x] 4 仮説切り分け (A 確定 = features 未生成)
- [x] `extract_features.py` に batch モード追加 (`--codes` / `--scope` /
      `--all-symbols` / `--parallel N` / `--business-days-only`)
- [x] `COLLECTOR_STATUS["material"]["ready"] = True` 修正 (F101 完了反映)
- [x] `run_material_collector` 新規実装 (announcements 検索 + MaterialEvent
      化 + events=0 でも default features 書き込み)
- [x] `scripts/jobs/backfill_features.py` 新規作成
- [x] Core500 抽出 (流動性 60 日窓ベース、500 銘柄、508 行)
- [x] `data/universe/README.md` 新規 (抽出根拠 + 差し替え可能性)
- [x] backfill 実行 (12 並列、2 分 44 秒で 1,074,280 行追加、err=0)
- [x] features 行数: 旧 87 行 → 新 1,074,331 行 (504 銘柄 / 129 dates、
      Run a/b/c 範囲完全カバー)
- [x] テスト追加 (12 → 20 PASS)
- [ ] Run a (20 営業日) 単独再走で events>0 確認 ← **現在ここ**
- [ ] events>0 なら Run b/c 連続 → F053 / F241 再評価 → F266 通過判定
- [ ] events=0 なら 仮説 C/D 追加診断 (active パターン起動条件 / F032 score
      閾値)

## 作業ログ

- **2026-05-04 朝**: 4 仮説切り分け、仮説 A 確定。Fujiwara 承認で案 1 GO
- **2026-05-04 13:39**: backfill 開始 (Core500 × 過去 120 営業日)
- **2026-05-04 13:42**: backfill 完了 (2 分 44 秒、1,074,280 行追加)。事前
  見積 1.5 時間 → 実測で約 32x 速い (SQLite WAL 一括 INSERT が並列効率
  高)
- **2026-05-04 (現在)**: Run a (20 営業日) 単独再走中 (PID 70984、
  約 4.1 秒/tick)
- 詳細: [[log.md]] 2026-05-04 朝セクション + [[03_design/F267_implementation_2026-05-04]]

## 完了条件

Run a (20 営業日) 単独再実行で **events>0 を先行確認** すること。

events>0 なら Run b/c 連続 or 段階実行 (合計約 5 時間) で全 Chain 再走、
F053 5 項目 + F241 9 項目を再評価し、F266 (Stage 3 最終ゲート) 通過判定に
進む。

events=0 なら仮説 C (パターン起動条件) / 仮説 D (F032 score 閾値) の追加
診断に切替。
