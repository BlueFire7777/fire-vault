---
id: F210-PhaseB
phase: P4: LINE/通知基盤
priority: 中
status: 未着手
owner: Fujiwara
depends_on: [F210-PhaseA, F100-historical, F230]
chapter: "01, 13, 27"
created: 2026-05-04
updated: 2026-05-04
---

# F210 Phase 1B: 資産履歴 DB 永続化 + 集計層

## 概要

F210 Phase 1A (純粋関数群) の DB 連携 + 集計拡張。Stage 3 移行直後に必要となる
資産履歴の永続化と日/週/月集計を提供する。Phase 1A は F211 統合に必要な API
だけ完結させ、本タスクで「日・週・月次で乖離率管理」(R-01-05 + R-13 line 67 +
R-27-07) を完成させる。

## スコープ

### 1. 新規テーブル `goal_asset_history`

```sql
CREATE TABLE IF NOT EXISTS goal_asset_history (
    record_date         TEXT PRIMARY KEY,    -- "YYYY-MM-DD" 日次スナップショット
    current_assets_yen  REAL NOT NULL,
    source              TEXT NOT NULL,        -- "manual_yaml"/"paper_live_account"/"rakuten_api"
    note                TEXT,
    created_at          TEXT NOT NULL,
    updated_at          TEXT NOT NULL
);
CREATE INDEX IF NOT EXISTS idx_goal_asset_history_source
    ON goal_asset_history(source);
```

- マイグレーション: `scripts/setup/migrate_goal_asset_history.py` 新規
- Chain 並行性 OK (CREATE IF NOT EXISTS、既存テーブル無干渉)

### 2. リポジトリ層 (`goal/tracking.py` に追加)

```python
@dataclass(frozen=True)
class AssetRecord:
    """日次資産スナップショット (1 行 = 1 日)"""
    record_date: date
    current_assets_yen: float
    source: str                    # "manual_yaml"/"paper_live_account"/"rakuten_api"
    note: Optional[str] = None


def upsert_asset_record(
    record_date: date,
    current_assets_yen: float,
    source: str,
    note: Optional[str] = None,
    db_path: Path = DB_PATH,
) -> None:
    """日次資産を永続化 (INSERT OR REPLACE で冪等)"""


def get_asset_history(
    start: date,
    end: date,
    db_path: Path = DB_PATH,
) -> list[AssetRecord]:
    """期間内の資産履歴を昇順で取得"""
```

### 3. 集計層 (`goal/tracking.py` に追加)

```python
@dataclass
class AggregatedDeviation:
    """期間集計結果 (日/週/月)"""
    period: str                   # "daily"/"weekly"/"monthly"
    period_start: date
    period_end: date
    avg_current_assets: float
    avg_deviation_pct: float
    pnl_in_period: float          # 期間内損益 (period_end - period_start の現在資産差)
    severity_at_end: DeviationSeverity


def aggregate_by_period(
    records: list[AssetRecord],
    period: Literal["daily", "weekly", "monthly"],
    config: GoalConfig,
) -> list[AggregatedDeviation]:
    """日/週/月集計

    daily : records をそのまま GoalDeviationInput 化したラップ
    weekly: 月曜起点で 5 営業日まとめ (週次 PnL = 週末資産 - 週初資産)
    monthly: 月初起点 (週跨ぎ・月跨ぎ・休日穴・空配列対応)

    Args:
        records: get_asset_history() の出力 (時系列昇順前提)
        period: 集計粒度
        config: GoalConfig (target_assets_yen / duration_business_days)

    Returns:
        list[AggregatedDeviation] (period 単位、時系列昇順)
    """
```

### 4. 営業日カウントヘルパー

```python
def count_business_days_between(
    start: date,
    end: date,
    db_path: Path = DB_PATH,
) -> int:
    """start <= date <= end の営業日数を market_prices_daily.date から DISTINCT count

    既存 list_business_days_descending (simulation/paper_live/batch_replay.py:60)
    と同じ「日経 225 取引日 = 営業日」前提。
    """
```

### 5. データソース両対応

- `manual_yaml` (Stage 3 移行直後): 朝一で Fujiwara が現在資産を YAML 更新
  - 配置案: `~/fire/data/goal_input.yaml` または `~/fire-vault/03_runtime/`
  - 形式: `{ "date": "YYYY-MM-DD", "current_assets_yen": 7800000, "note": "..." }`
- `paper_live_account` (Paper Live 中): `AccountTracker.refresh()` 経由で
  current_capital + unrealized_pnl_total を取得 (Stage 2 中の自動 source)
- 切替ロジック: `FIRE_MODE` (paper / live) または明示的な引数

## テスト計画 (10 ケース以上)

- `upsert_asset_record` / `get_asset_history` 冪等性 (3 ケース)
- `count_business_days_between` (3 ケース: 通常 / 連続 / 月跨ぎ)
- `aggregate_by_period` (週跨ぎ・月跨ぎ・休日穴・空配列・1 件のみ、6 ケース)

## 重要な決定事項 (Phase 2 / 着手時に再確認)

- `manual_yaml` の配置先: `~/fire/data/` か `~/fire-vault/` か
- `aggregate_by_period.period="daily"` は records 1 件 → 1 AggregatedDeviation か?
  (Dashboard 単位整合)
- 期間境界 (週初 = 月曜? 日曜?、月初 = 1 日?) を実運用で再確認

## Out of Scope (本 Phase でも対応しない)

- LINE 通知 (Q4): F118 Goal Tracking Agent で実装
- Dashboard 可視化 (R-27-07): F088 GoalProgress.tsx 別タスク
- 楽天 API 自動取得 (R-01-08 半自動主軸方針外): 想定外

## 着手前提

- F210 Phase 1A 完了 (本タスクが拡張する基盤、~/fire/goal/tracking.py 既存)
- Phase 1A の Q2 境界値・Q5 補間方式について Phase 2 確認結果を反映
- Chain 並行性: 新規テーブル追加 (CREATE IF NOT EXISTS) のみで既存無干渉 OK

## 関連リンク

- 親タスク: `F210_3000万円2年目標トラッキング.md` (Phase 1A 完了メモ)
- 既存実装: `~/fire/goal/tracking.py` (Phase 1A)
- DB 既存パターン: `scripts/setup/migrate_paper_live_positions.py`
- 営業日リスト前例: `~/fire/simulation/paper_live/batch_replay.py:60` (`list_business_days_descending`)
