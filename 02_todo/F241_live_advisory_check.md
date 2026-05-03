---
id: F241
phase: P9: Stage 3 移行
priority: 最優先
status: 完了
owner: Fujiwara
depends_on: [F053, F119, F236]
chapter: "19,32"
created: 2026-05-03
updated: 2026-05-03
---

# F241: Live Advisory 昇格条件検証 (Stage 3 移行最終ゲート)

## 概要

Paper Live → Live Advisory (Stage 3) 移行前の最終検証ゲート。F053 5 項目に
加え、LINE / 緊急アラート / Fujiwara 受容 / F119 提案書の 4 項目を統合チェック。
全 9 項目 PASS で Stage 3 開始可能を判定し、Markdown レポート出力。

## 9 項目チェック構成

| カテゴリ | 項目 | 詳細 |
|---|---|---|
| F053 5 項目 | sample_size | 20 営業日 or 50 トレード |
| (流用) | expected_value | 勝率 / RR / 最大 DD |
| | halt_function | 停止機能動作確認 |
| | close_function | 強制クローズ 100% |
| | simulation_accuracy | optimistic_bias_score ≤ 0.7 |
| F241 追加 4 項目 | LINE 5 部屋疎通 | F236 send_to_room で test メッセージ |
| | 緊急ポジション整理アラート | OpenClaw cron + launchd 登録確認 |
| | Fujiwara 受容体制 | `--fujiwara-accept` 明示承認 |
| | F119 昇格提案書 | run_evaluation で Markdown 生成 |

## 重要な決定事項 (仕様書差分確認)

- **`PromotionResult.criteria_checks`** は `list[CriterionCheck]`、各要素は
  `criterion_name` / `passed` / `detail` (仕様書通り)
- **`Room` enum**: ENTRY / EXECUTION / REPORT / SYSTEM / EMERGENCY (5 部屋)
- **`run_evaluation`** は `period`, `db_path`, `save_report`, `send_line`,
  `skip_db`, `reports_dir` を受け取る (`run_id` 引数は内部で
  `DEFAULT_RUN_ID="EVAL-DEFAULT"` 使用、Stage 3 検証では default で OK)
- **緊急時刻** = `{14:45, 14:55, 15:05, 15:10, 15:15}` (F236 既定)
- **`launchctl list` 連携**: subprocess で 10 秒タイムアウト、失敗で
  `has_launchd=False` フォールバック (テスト環境で安全)
- **OpenClaw cron jobs.json 不在**でも例外を出さず empty set で続行

## 成果物

### 新規ファイル

- `simulation/paper_live/live_advisory_check.py`:
  - `CheckItem` dataclass (name / passed / detail / error)
  - `StageReadinessReport` dataclass + `to_markdown()` メソッド
  - `LiveAdvisoryChecker.check(run_id, skip_*, fujiwara_acceptance)`
  - 内部: `_check_f053` / `_check_line_rooms` / `_check_emergency_alerts`
    / `_check_fujiwara_acceptance` / `_check_and_generate_proposal`
  - `EXPECTED_EMERGENCY_TIMES = {14:45, 14:55, 15:05, 15:10, 15:15}`
- `tests/simulation/test_live_advisory_check.py`: 18 ケース

### 変更ファイル

- `simulation/paper_live/cli.py`:
  - `--check-live-advisory RUN_ID` モード + `--fujiwara-accept` /
    `--skip-line` / `--skip-emergency` / `--skip-proposal` 引数
  - Markdown レポートを stdout 出力

### テスト

- 新規 18 件
  - dataclass + Markdown: 3 件 (defaults / sections / pass mark)
  - F053 連携: 2 件 (criterion 変換 / 例外時)
  - LINE 5 部屋: 2 件 (all OK / partial fail)
  - 緊急アラート: 4 件 (no jobs / OpenClaw 全網羅 / launchd fallback / 定数)
  - Fujiwara: 2 件 (accept / reject)
  - F119 提案書: 2 件 (success / exception)
  - 統合 check: 3 件 (all pass / any fail / skip flags)
- 累計: 937 → **955 PASS** (+18)
- 既存 937 への影響: **0 件** (新規 + CLI 1 ブロック追加のみ)

### CLI Smoke 結果 (real DB、空 run_id + skip-line/skip-emergency)

```
$ python -m simulation.paper_live --check-live-advisory PL-NONEXISTENT \
    --skip-line --skip-emergency

# F241 Live Advisory 昇格条件検証レポート
- run_id: PL-NONEXISTENT
- 検証日時: 2026-05-03T05:59:51+00:00
- **全体判定: ❌ FAIL**

## F053 5 項目 (Paper Live → Semi Auto 昇格基準)
- ❌ F053 sample_size: n_days=0, n_trades=0
- ❌ F053 expected_value: no closed trades
- ❌ F053 halt_function: no account record
- ❌ F053 close_function: no force_close events
- ✅ F053 simulation_accuracy: optimistic_bias_score=0.000

## F241 追加検証項目
- ❌ Fujiwara 受容体制: 未承認
- ✅ F119 昇格提案書生成: 日次 提案書 (0 件、承認待ち 0 件)

## 関連ファイル
- 昇格提案書: `/Users/bluefire/fire/reports/eod_2026-05-03.md`
```

## CLI 拡張オプション

| オプション | デフォルト | 説明 |
|---|---|---|
| `--check-live-advisory RUN_ID` | (off) | F241 検証モード |
| `--fujiwara-accept` | (off) | Fujiwara 明示承認 |
| `--skip-line` | (off) | LINE 5 部屋疎通スキップ |
| `--skip-emergency` | (off) | 緊急アラート確認スキップ |
| `--skip-proposal` | (off) | F119 提案書生成スキップ |

## Stage 3 開始までのチェックリスト (Fujiwara さん向け)

### コード側 (完了)

- [x] F050 Paper Live 本体
- [x] F053 evaluate_promotion 5 項目判定
- [x] F100 historical 過去データ取得
- [x] F230 Paper Live バッチ実行
- [x] F119 Evaluation Agent
- [x] F236 LINE 5 部屋ルーティング
- [x] F241 Live Advisory 昇格条件検証 (本タスク)

### Fujiwara 手動作業 (Stage 3 直前)

- [ ] `JQUANTS_API_KEY` を `.env` に設定
- [ ] F100 historical で過去 6 ヶ月分の market_prices_daily 取得実行
- [ ] F230 batch_replay で直近 20 営業日 Paper Live バッチ実行
- [ ] `LINE_CHANNEL_TOKEN` / `LINE_USER_ID` / `LINE_ROOM_*` を `.env` に設定
- [ ] LINE Bot を 5 部屋に招待
- [ ] launchctl で 5 つの緊急アラート plist を load
- [ ] (任意) OpenClaw cron jobs.json に 14:45/14:55/15:05/15:10/15:15 登録
- [ ] F241 `--check-live-advisory <run_id> --fujiwara-accept` で全 PASS 確認
- [ ] 楽天証券 iSPEED でモバイル発注フロー検証
- [ ] F241 Markdown レポート読み込み + Stage 3 開始最終承認

## あと何が必要か (Stage 3 開始までの差分)

1. **データ流入**: F100 historical で過去 6 ヶ月分一括取得 (実 API 実行)
2. **バッチ実行**: F230 で 20 営業日分 Paper Live バッチ実行
3. **環境設定**: `.env` の LINE 系 + `LINE_USER_ID` 設定
4. **launchd 登録**: `~/Library/LaunchAgents/com.bluefire.fire.alert*.plist` を
   load (5 つ)
5. **Fujiwara 受容**: F241 で `--fujiwara-accept` を付けて全項目 PASS 確認

→ コード実装は完了、残りは Fujiwara さん手動作業のみ。

## スコープ外 (将来 / 別タスク)

- F238 (LINE 約定パーサー) 連携 (Stage 3 開始時に追加)
- 自動 Stage 3 移行 (必ず Fujiwara 明示承認)
- Dashboard 表示 (F090 別タスク)
- Stage 3 後の運用ロジック (F221 別タスク)
- LINE 経由の Fujiwara 受容承認 (F064 別タスク)

## 関連リンク

- 要件書: 第 19 章 R-19-08 / 第 32 章 (本番移行基準)
- 関連: [[F053_Paper_Live_昇格基準実装]] (5 項目流用) /
  [[F119_Evaluation_Agent]] (提案書生成) /
  [[F236_LINE_5段階アラート]] (5 部屋疎通) /
  [[F230_Paper_Live_batch_replay]] (前提となる蓄積) /
  [[F100_historical_fetch]] (前提となる市場データ)
- コード: `simulation/paper_live/live_advisory_check.py`,
  `simulation/paper_live/cli.py` (CLI 拡張)
