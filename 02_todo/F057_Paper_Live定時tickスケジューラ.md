---
id: F057
phase: P3: Paper Live
priority: 最優先
status: 完了
owner: Fujiwara
depends_on: [F050, F054, F055]
chapter: "08,19,23"
created: 2026-05-01
updated: 2026-05-01
tags: [P3, paper_live, scheduler]
---

# F057: Paper Live 定時 tick スケジューラ (寄付き〜引け)

## 概要

`PaperLiveRunner` を寄付き (09:00) 〜 引け (15:30) まで定時 tick で回す
スケジューラ。F050 設計の「単発 CLI tick で runner state 永続化されない」
制約を解消し、長時間プロセスとして稼働する。

要件根拠:
- 第 8 章 (運用スケジュール - 時間帯別ルール)
- 第 19 章 R-19-08 (Paper Live)
- 第 23 章 (FIRE Runtime - 常駐基盤)

## スコープ変更経緯

当初 F057 は「候補抽出パイプライン」だったが、F054 で実質完成 (extract_candidates
が features → similarity → reproducibility → 候補のフルチェーンを実装) のため
スコープ重複を回避してスケジューラに変更。

旧 stub `F057_候補抽出パイプライン.md` は削除し、本ファイルで完了版を保持。

## 実装内容

### 主要モジュール

- `simulation/paper_live/scheduler.py` (新規)
  - `PaperLiveScheduler` クラス: 寄付き〜引けの定時 tick ループ
  - `_is_market_hours(dt)`: 市場時間判定 (寄付き〜引け、ランチ除外)
  - `_next_tick_time(current)`: 次 tick 時刻計算 (ランチ入り → 12:30 ジャンプ)
  - `request_stop()`: graceful shutdown 要求 (SIGINT 連動)
  - `run()` → `{run_id, tick_count, events_total, errors, ...}` 集計返却
  - CLI: `--date / --interval-seconds / --start / --end / --no-skip-lunch / --no-sleep / --db-path`
  - SIGINT (Ctrl+C) で `request_stop` を呼ぶ signal handler 登録

### 定数

| 定数 | 値 | 出典 |
|---|---|---|
| `DEFAULT_OPEN_TIME` | 09:00 | 第 8 章 |
| `DEFAULT_CLOSE_TIME` | 15:30 | 第 8 章 |
| `DEFAULT_INTERVAL_SEC` | 300 (5 分) | バランス重視 |
| `LUNCH_BREAK_START` | 11:30 | 前場引け |
| `LUNCH_BREAK_END` | 12:30 | 後場寄り |
| `MIN_INTERVAL_SEC` | 1 | 無限ループ防止 (0 を許容しない) |

### キーポイント

- **テスト用 sleep 抑制**: `sleep_until_next_tick=False` でテストや過去日バックテスト
  は即時連続実行 (sleep スキップ)
- **エラー時は続行**: 1 tick の例外は `errors` に記録するだけ、ループ継続 (R-20-04
  停止優位とは別レイヤ。ハード停止は per-tick 例外ではなく停止判定で実施)
- **graceful shutdown**: try-finally で必ず `end_session()` 呼び出し
- **ランチタイム自動スキップ**: `_next_tick_time` で 11:30 跨ぎを 12:30 にジャンプ
- **時刻はナイーブ datetime**: tick.py 内部で `as_of_dt.replace(hour=...)` を使うため
  tzinfo 依存なし (`datetime.combine(date, time)` で OK)

### F022 / F242 との棲み分け

- **F057**: Paper Live 専用の日中スケジューラ (P3 完成のため先行)
- **F022 FIRE Runner**: 全エージェント統合オーケストレーション (P0 全体基盤)
- **F242 OpenClaw**: Mac mini 上の常駐実行基盤 (cron 起動側)

F057 は F022 / F242 の下で動く子プロセス相当。朝 cron で起動 → 寄付き直前まで待機 →
09:00-15:30 で定時 tick → 15:30 終了。

## テスト

- `tests/simulation/test_scheduler.py`: **14 / 14 PASS**
  - TestMarketHours (4): 寄付き / ランチ / 後場寄り / 引け後
  - TestNextTickTime (2): 通常 increment / ランチ跨ぎジャンプ
  - TestRun (4): 短時間 / ランチ除外カウント / stop 早期終了 / 例外時続行
  - TestEndToEnd (2): 全日 66 tick / DB 状態 completed
  - TestValidation (2): interval=0 拒否 / request_stop フラグ
- 累計 **549 PASS** (535 → 549、F040-F056 + F100 既存全 PASS 維持)

## CLI smoke 結果 (実 J-Quants データ)

```
$ python -m simulation.paper_live.scheduler --date 2026-04-28 \
    --start 09:00 --end 09:30 --no-sleep
Session started: PL-20260501134944-355F for 2026-04-28 (09:00〜09:30, interval=300s)
Session ended: PL-20260501134944-355F

=== Session Result ===
Run ID:            PL-20260501134944-355F
Tick count:        6        ← 09:00, 09:05, 09:10, 09:15, 09:20, 09:25
Events total:      0        ← 過去パターン未蓄積で評価結果は全て pass
Errors:            0
Stopped by signal: False
```

→ 6 tick / 約 9 秒 (各 tick で 4,449 銘柄スキャン → features ある銘柄のみ
ReproducibilityEngine.evaluate)。Pattern Store に過去パターンが蓄積されれば
events_total が増えていく。

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| scheduler.py 新規 | ✅ |
| PaperLiveScheduler クラス + 4 メソッド | ✅ |
| CLI 起動可能 (`python -m simulation.paper_live.scheduler`) | ✅ |
| SIGINT graceful shutdown | ✅ |
| ランチタイム自動スキップ | ✅ |
| 14 ケース全 PASS | ✅ |
| 既存 535 PASS 非破壊 | ✅ |
| 累計 549 PASS | ✅ |
| CLI smoke (実データ) | ✅ |

## 関連リンク

- 要件書: 第 8 章 (運用スケジュール) / 第 19 章 (Paper Live) / 第 23 章 (Runtime)
- 関連: [[F050_Paper_Live_Mode_Stage_2_本体]] / [[F054_tick_py中身実装]] /
  [[F055_特徴量抽出ジョブ起動スクリプト]] / [[F022_FIRE_Runner骨組み]]
- 次タスク: **F058** (E2E smoke test、1 セッション通しで 24 時間稼働確証)
- コード: `~/fire/simulation/paper_live/scheduler.py`

## スコープ外メモ

- 4,449 銘柄並列スキャン (現状逐次、別タスクで最適化)
- 永続化された systemd / launchd デーモン化 (F022 / F242 で対応)
- 複数日連続実行 (F058 で対応)
- リアルタイムストリーミング (将来検討)
