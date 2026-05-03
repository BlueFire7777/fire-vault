---
id: F210-PhaseB
phase: P4: LINE/通知基盤
priority: 凍結
status: 凍結 (2026-05-04、Dashboard 着手 / F118 設計時に再評価)
owner: Fujiwara
depends_on: [F210-PhaseA, F100-historical, F230]
chapter: "01, 13, 27"
created: 2026-05-04
updated: 2026-05-04
---

# F210 Phase 1B: 資産履歴 DB 永続化 + 集計層 (凍結)

## ステータス: 凍結 (2026-05-04)

F210 は **Phase 1A 完了で「F210 完了」扱い**。Phase 1B は凍結し、解凍トリガーが
発生するまで着手しない。

### 凍結理由

1. **F211 が F210 出力を判定材料に使わない**
   - F211 は `severity` 不問で `reason="deviation"` 系を REJECT
     (`goal/discipline.py:343` `if reason in DEVIATION_BASED_REASONS`)
   - severity は rationale 文字列 (`line 350`) と
     `EvaluationProposal.deviation_severity` 代入 (`line 565`) のみで参照、
     `if`/`match` 分岐に未使用
   - → **履歴トラッキング (Phase 1B の主スコープ) は F211 規律ガードに不要**

2. **毎日の LINE/YAML 入力は Fujiwara の運用負荷が重い**
   - 楽天証券アプリで残高は既に確認可能 → Fujiwara の本来運用と二重管理
   - 半自動主軸 (R-01-08) 思想とは別軸の手作業負担 (毎朝の固定タスク化)
   - 第 33 章 (人間の運用負荷の上限設計、穴 11 対策) と整合: 増やさない判断

3. **Phase 1A の `calculate_deviation` でアドホック計算は可能**
   - 「現在の残高 → 乖離率 / 5 段階判定」は純粋関数で 1 回計算で完結
   - 履歴トラッキングの不在は致命的でない (毎日でなく必要時に計算で代替)
   - F211 統合経路は Phase 1A だけで成立

4. **Dashboard 自体が未着手のため F210 の出力先がない**
   - R-13 line 67 / R-27-07 の「日/週/月の必要進捗」表示先 = Dashboard
   - Dashboard (F310 系想定) 着手前に集計層を作っても表示先なし
   - 表示要件確定前の集計仕様策定は手戻りリスク

### 解凍トリガー (Phase 1B 再開条件)

以下のいずれかが発生したら凍結解除し、本メモを再評価:

- **トリガー 1**: Dashboard (F310 系想定) 着手時に「履歴トラッキング必要」と
  判断された場合
  - 進捗チャート / PnL 推移 / 乖離率時系列を表示する仕様確定時
  - `goal_asset_history` の DDL と表示要件を整合させて再設計

- **トリガー 2**: F118 Goal Tracking Agent オーケストレーション設計時に
  「自動記録が必要」と判断された場合
  - F118 が日次バッチで残高スナップショット取得・保存する設計の場合
  - `paper_live_account` 自動連携が現実解になった時点

- **トリガー 3**: 楽天証券側の API 自動取得経路が確立した場合
  - R-01-08 半自動主軸方針の例外として Fujiwara が認めた場合
  - 入力負荷ゼロでの履歴蓄積が可能になった時点

### 凍結時の Phase 1A 機能保証

凍結中も以下は Phase 1A で利用可能:
- `calculate_deviation(current, elapsed, config)` → アドホック乖離計算
- `classify_severity(deviation_pct)` → 5 段階判定
- `calculate_required_weekly/monthly_progress` → 必要進捗 (円・率)
- F211 `check_lot_increase_request` 等への入力供給

→ F211 規律ガード経路 + 単発の進捗チェックは Phase 1A で完結する。

---

## (以下、凍結解除時の参考資料として保持) Phase 1B スコープ

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
