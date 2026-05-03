---
id: F230
phase: P9: Stage 3 移行
priority: 最優先
status: 完了
owner: Fujiwara
depends_on: [F050, F053, F057]
chapter: "19,32"
created: 2026-05-03
updated: 2026-05-03
---

# F230: Paper Live 過去日付バッチ実行モード (最新→過去 遡及)

## 概要

Mac mini 24h 稼働 + OpenClaw のエッジを構造的に活かすため、過去データを
Paper Live モードでバッチ実行し `paper_live_results` /
`paper_live_positions` に蓄積する。リアルタイム待ち 4 週間 → 過去データ
バッチで数時間に短縮。

## 設計仕様 (Fujiwara 2026-05-03 議論)

- **最新営業日 → 過去** の順で遡及 (古いデータでの過学習回避)
- **1 run_id で N 営業日を貫通** (F053 が DISTINCT date でカウントできる構造)
- look-ahead bias 排除は既存 `process_tick(as_of_dt)` で保証
- **シリアル実行 (Phase 1)**、並列化は Phase 2

戦略的意義:
- 直近の相場特性に最適化 (regime 変化への適応性確認)
- 早期 Stage 3 移行判定 (直近 20 営業日で PASS したら即昇格)
- パターン劣化の早期検知 (直近で動かなくなったパターンを最初に発見)
- Mac mini 24h 夜間バッチで「直近検証が必ず完了する」サイクル

## 重要な決定事項 (仕様書差分)

- **`market_prices_daily` の列名は `date`** (仕様書 `dt` は誤り)
- **`PromotionResult.criteria_checks` はリスト** (仕様書の dataclass 個別属性 `result.sample_size.passed` は誤り)。各要素に `criterion_name` / `passed` / `detail`
- **`target_patterns` は `list[str] | None`** (仕様書 `"all"` 文字列は誤り)
- **`PaperLiveRunner.start_session` 戻り値は `str` (run_id)**、`end_session(run_id=None)` で current_run_id 使用可

## 成果物

### 新規ファイル

- `simulation/paper_live/batch_replay.py`:
  - `list_business_days_descending(end_date, days_back, db_path)` - 営業日
    リスト生成 (最新→過去)
  - `generate_tick_times(business_day, interval_min, skip_lunch)` - 1 日内
    の tick 時刻列挙 (JST aware、昼休み除外)
  - `BatchReplayRunner.run(end_date, days_back, interval_min, patterns,
    fill_model, skip_lunch, check_promotion)` - メインエントリ
  - `BatchReplayResult` dataclass - 結果 (run_id / business_days / n_ticks /
    n_events / n_positions_opened/closed / promotion_check / tick_errors /
    error_msg)
- `tests/simulation/test_paper_live_batch_replay.py`: 17 ケース

### 変更ファイル

- `simulation/paper_live/cli.py`:
  - `--batch-replay` モード + `--end-date` / `--days-back` / `--interval-min`
    / `--no-skip-lunch` / `--no-promotion-check` 引数追加
  - 完了後に F053 Promotion Check を Markdown 風サマリで表示

### テスト

- 新規 17 件
  - list_business_days_descending: 5 件 (desc 順 / end_date filter / 空 →
    ValueError / days_back=0 / partial 警告)
  - generate_tick_times: 5 件 (5 分 default / lunch skip / no-skip-lunch /
    JST aware / interval 0 で ValueError)
  - BatchReplayRunner: 7 件 (1 day / multi day single run_id / no data
    ValueError / days_back=0 / promotion included / promotion skipped /
    tick error continues)
- 累計: 906 → **923 PASS** (+17)
- 既存 906 への影響: **0 件** (新規 + CLI 1 ブロック追加のみ)

### Smoke 結果 (real DB、3 営業日)

```
$ python -m simulation.paper_live --batch-replay --end-date 2026-05-02 --days-back 3 --interval-min 60

# F230 Batch Replay 完了
- run_id: PL-20260503053954-F3EC
- 営業日数: 3
- 期間: 2026-04-27 〜 2026-04-30
- tick 数: 18 / イベント総数: 0 / tick エラー: 0

## F053 Promotion Check
  全体判定: FAIL
  - [FAIL] sample_size: n_days=0 (>=20? False), n_trades=0 (>=50? False)
  - [FAIL] expected_value: no closed trades to evaluate
  - [FAIL] halt_function: halt_reason recorded? False
  - [FAIL] close_function: no force_close events (untested)
  - [PASS] simulation_accuracy: optimistic_bias_score=0.000 (<= 0.7? True)
```

エラーハンドリング:
```
$ python -m simulation.paper_live --batch-replay --end-date 2025-01-01 --days-back 5
[F230] エラー: market_prices_daily に 2025-01-01 以前のデータがありません。
F100 過去データ取得バッチを先に実行してください。
```

## CLI 拡張オプション

| オプション | デフォルト | 説明 |
|---|---|---|
| `--batch-replay` | (off) | F230 バッチ実行モード起動 |
| `--end-date YYYY-MM-DD` | 本日 | 最新営業日 |
| `--days-back N` | 20 | 遡及営業日数 |
| `--interval-min N` | 5 | tick 間隔 (分) |
| `--patterns "p1,p2"` | None | 対象パターン (カンマ区切り) |
| `--fill-model strict` | strict | 約定モデル (strict/realistic/ideal) |
| `--no-skip-lunch` | (off) | 昼休み 11:30-12:30 もスキップしない |
| `--no-promotion-check` | (off) | 完了後の F053 自動実行を抑止 |

## 設計上の意図

- **n_business_days 確認**: paper_live_runs に 1 行のみ作成 (1 run_id で
  全営業日を貫通) を smoke で確認。`paper_live_results` のイベント数は
  パターン候補数依存なので空 DB では 0、N=1 の検証は `paper_live_runs`
  テーブルで実施。
- **tick エラー耐性**: 1 tick 失敗で全体停止しない、`tick_errors` 配列に
  ISO 時刻 + 例外メッセージを蓄積、他の tick は続行
- **KeyboardInterrupt 対応**: Ctrl+C で中断しても `end_session()` を
  finally で呼んで paper_live_runs を completed にする
- **F053 自動実行**: `check_promotion=True` (デフォルト) で完了後
  `evaluate_promotion(run_id)` を呼び、5 項目判定をサマリ表示

## スコープ外 (将来 / 別タスク)

- F100 過去データ取得 (B、別タスク並行)
- F035 Pattern Research の修正 (動作確認のみ)
- 並列化 (Phase 2、F045 と同等の ProcessPoolExecutor)
- リアルタイム F057 scheduler との統合 (既存と分離)
- F119 Evaluation の自動起動 (CLI で個別呼び出し)
- F241 Live Advisory 昇格条件検証 (C、別タスク)

## 関連リンク

- 要件書: 第 19 章 R-19-08 / 第 32 章 (本番移行基準)
- 関連: [[F050_Paper_Live_Mode_Stage_2_本体]] /
  [[F053_Paper_Live_昇格基準実装]] /
  [[F045_過去データ高速リプレイ並列実行]] (エッジ思想の元) /
  [[F119_Evaluation_Agent]] (バッチ後の改善提案)
- コード: `simulation/paper_live/batch_replay.py`,
  `simulation/paper_live/cli.py` (CLI 拡張)
