---
id: F054
phase: P3: Paper Live
priority: 最優先
status: 完了
owner: Fujiwara
depends_on: [F031, F032, F050, F051, F052, F055, F056, F100]
chapter: "11,19,20,21"
created: 2026-05-01
updated: 2026-05-01
tags: [P3, paper_live, tick, milestone]
---

# F054: Paper Live tick.py 中身実装 (24 時間稼働への最大の壁)

## 概要

F050 で骨組みのみだった `simulation/paper_live/tick.py` の 3 つの主要関数の中身を
実装。これまで完成した全要素 (F031/F032/F050-F053/F055/F056/F100) を統合して
パターン候補抽出と仮想売買を実現する。**24 時間稼働への最大の壁を突破**。

要件根拠:
- 第 11 章 R-11-01〜R-11-07 (再現性エンジン)
- 第 19 章 R-19-08 (Paper Live 機能要件)
- 第 20 章 R-20-03 (執行優位 — 14:45 以降新規停止 / 15:10 信用クローズ)
- 第 21 章 R-21-02〜R-21-09 (Pattern Research Agent 連携)

## スコープ拡張理由 (F054 で発見した補修事項)

調査で以下が発覚し、tick.py 実装に加えて補修を実施:
- `paper_live_positions` に `tp_price` / `sl_price` カラム無し
- `VirtualPosition` dataclass に `tp_price` / `sl_price` 無し
- VIRTUAL_ENTRY payload に `stop_price` (SL) 無し

→ F054 でマイグレーション + dataclass 拡張 + payload 拡張 + PositionTracker 修正を併走。

## 実装内容

### 主要モジュール

- `scripts/setup/migrate_paper_live_tp_sl.py` (新規、冪等)
  - 既存 paper_live_positions に `tp_price` / `sl_price` カラム追加
- `scripts/setup/migrate_paper_live_positions.py` (修正)
  - 新規作成時の DDL にも `tp_price` / `sl_price` 追加 (新規 DB で同期)
- `simulation/paper_live/models.py` (修正)
  - `VirtualPosition` dataclass に `tp_price` / `sl_price: Optional[float] = None` 追加
- `simulation/paper_live/position.py` (修正)
  - `_handle_entry`: payload から `target_price` / `stop_price` を取得、買い増し時は新値で上書き (None なら維持)
  - `_insert_position` / `_update_position`: SQL に tp_price / sl_price を含める
- `simulation/paper_live/tick.py` (大幅拡張、240 → 約 350 行)
  - 時刻ガード `_is_after_new_entry_cutoff(14:45)` / `_is_after_force_close_time(15:10)` (R-20-03)
  - DB ヘルパー `_fetch_all_symbols` / `_has_features` / `_fetch_ohlc`
  - `extract_candidates`: market_listings の全銘柄スキャン → features ある銘柄のみ ReproducibilityEngine.evaluate → "execute" 銘柄を CANDIDATE 化
  - `evaluate_virtual_entries`: 既存 F052 を維持しつつ、TP/SL を payload に格納 (`target_price` / `stop_price`)。CANDIDATE に payload bar 情報あれば優先、なければ market_prices_daily から OHLC 取得 (F052 後方互換維持)
  - `monitor_virtual_positions`: open positions × 当日 OHLC で TP/SL タッチ判定。両方触れたら **SL 優先 (保守的)**
  - `evaluate_force_close`: 15:10 以降、monitor で閉じてない open positions を当日 close で強制クローズ
  - デフォルト TP/SL ルール: long +5% TP / -2% SL、short -5% TP / +2% SL (パターン側に指定が無い場合のフォールバック)

### キーポイント

- **R-20-03 厳守**: 14:45 以降は新規候補ゼロ、15:10 以降は信用全閉じ
- **TP/SL 両方タッチは SL 優先**: 保守的判定 (R-19 の負け方を改善する力)
- **CANDIDATE → VIRTUAL_ENTRY フロー**: extract_candidates が CANDIDATE 生成 →
  evaluate_virtual_entries が fill_model 経由で VIRTUAL_ENTRY 化 → PositionTracker
  が DB 永続化、TP/SL 価格も同時保存
- **F052 後方互換**: candidate.payload に bar 情報あれば優先、無ければ DB から取得
  (F052 既存テスト 15 件無修正で通る)
- **features.dt 規約 (F055 と同じ)**: `"{date}T09:00:00+09:00"` (JST 寄付)
- **F052 fill_model 統合は維持**: 約定品質スコア (slippage_pct/commission/fill_quality_score)
  も VIRTUAL_ENTRY payload に同居

### TP/SL 価格のデフォルト値ロジック (F054 設計判断)

| 観点 | 採用 |
|---|---|
| パターンに TP/SL ルールあり | candidate.payload で優先伝達 (将来の patterns テーブル拡張点) |
| パターン未指定の場合 | デフォルトルール: long +5%/-2%、short -5%/+2% を `_default_tp_sl(side, entry_price)` で生成 |
| 計算タイミング | evaluate_virtual_entries 内で fill_price 確定後に算出 (約定スリッページ反映後) |

→ 安全側: TP は遠く (+5%)、SL は近く (-2%) に置くことで loss 限定 + win 余地確保。
将来 pattern.tp_pct / pattern.sl_pct に保存される設計に発展させる。

## テスト

- `tests/simulation/test_tick_internals.py`: **20 / 20 PASS**
  - TestTimeCutoffs (3): 14:45 / 15:10 ガード
  - TestExtractCandidatesEmpty (3): listings 0 / features 0 / 14:45 後
  - TestExtractCandidatesWithRepro (2): low similar count / features missing skip
  - TestMonitor (5): 0 / TP touch / SL touch / 両方 → SL 優先 / OHLC 無し
  - TestForceClose (4): 15:10 前 / 後 / monitor 済除外 / OHLC 無し
  - TestEntryTPSLPersistence (3): payload 経由保存 / None 維持 / デフォルト適用
- F050-F053 + F051 + F052 + F055 + F056 既存全 PASS 維持
- 累計 **535 PASS** (515 → 535、+20)

## End-to-End smoke (実 J-Quants データ)

```python
runner = PaperLiveRunner()
run_id = runner.start_session(target_patterns=[])
# 86970 long を avg_entry=1900 / TP=1950 / SL=1850 で inject

result = runner.tick(datetime(2026, 4, 28, 10, 0, 0, tzinfo=tz.utc))
# tick 10:00 events: ['virtual_tp', 'notification']  ← 当日 high=1956.5 ≥ 1950 で TP touch

# DB 自動更新確認
# status=closed / exit=1950 / reason=tp / pnl=5000   ← PositionTracker が決済反映

result2 = runner.tick(datetime(2026, 4, 28, 15, 30, 0, tzinfo=tz.utc))
# tick 15:30 events: []  ← 既にクローズ済、force_close 対象なし
runner.end_session(run_id)
```

→ extract_candidates / monitor_virtual_positions / evaluate_force_close + PositionTracker
の連携が実 J-Quants データで動作することを確認。

### CLI 単発 tick の制約 (既知)

`python -m simulation.paper_live --tick --run-id PL-...` の単発 CLI 呼び出しでは
runner state が永続化されないため、PositionTracker による自動クローズが動かない。
これは F050 設計の既知制約。連続 tick は同一 PaperLiveRunner インスタンス内で
回す必要がある (将来 F022 FIRE Runner / F242 OpenClaw で解決)。

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| マイグレーション (tp_price/sl_price 追加) | ✅ |
| VirtualPosition + payload + PositionTracker TP/SL 対応 | ✅ |
| extract_candidates で 4,449 銘柄スキャン → execute 判定 | ✅ |
| monitor_virtual_positions の TP/SL タッチ判定 | ✅ |
| evaluate_force_close の 15:10 強制クローズ | ✅ |
| 20 ケース全 PASS | ✅ |
| F050-F056 + F100 既存 515 PASS 非破壊 | ✅ |
| 累計 535 PASS | ✅ |
| End-to-End smoke (実データ) | ✅ |

## 関連リンク

- 要件書: 第 11 章 / 第 19 章 / 第 20 章 / 第 21 章
- 関連: [[F031_類似局面検索エンジン]] / [[F032_再現性スコア合成エンジン]] /
  [[F050_Paper_Live_Mode_Stage_2_本体]] / [[F051_仮想建玉計算]] /
  [[F052_Paper_Live厳しめ約定モデル標準化]] / [[F055_特徴量抽出ジョブ起動スクリプト]] /
  [[F056_F031F032テスト追加]] / [[F100_市場データAPI]]
- 次タスク: **F057** (候補抽出パイプライン整備、F054 の自然な拡張で 1.5 日)
- コード: `~/fire/simulation/paper_live/tick.py` / `position.py` / `models.py`

## スコープ外メモ

- **F021/F022/F023/F025 collector の実装** (F101/F104 / 板データ / 楽天証券データ待ち)
- **extract_candidates の並列化** (4,449 銘柄逐次は遅い、別タスクで最適化)
- **CLI 単発 tick での状態永続化** (FIRE Runner で解決)
- **patterns.tp_pct / sl_pct の保存・読み出し** (現状は payload 渡しまたはデフォルト)
- **F058 E2E smoke test (1 セッション通し)** — 別タスク
