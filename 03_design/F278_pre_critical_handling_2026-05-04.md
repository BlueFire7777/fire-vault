---
type: design
id: F278_pre_critical_handling
task: F278-Pre
related_tasks: F100, F277, F278
status: in_progress
created: 2026-05-04
updated: 2026-05-04
tags:
  - design
  - git_governance
  - critical_handling
  - pydantic
  - exception_propagation
---

# F278-Pre CRITICAL ハンドリング設計記録

**作成日**: 2026-05-04
**起点**: [[F278_git_ガバナンス整備|F278]] (F278-Pre 部分先行) Step 7-A-2-① F100 retroactive commit
**関連設計**: [[F277_phase1_design_2026-05-04|F277 Phase 1 設計]]
**完了基準**: [[task_completion_criteria|F271 v1.1]]

---

## 1. 背景

F278-Pre (CLAUDE.md 完了表記載 untracked 群の retroactive commit) 着手中、
Step 7-A-2-① F100 J-Quants V2 で Codex pre-commit が CRITICAL 2 件を指摘。

本部判断 (案④): CRITICAL 2 件のみ最小修正 + retroactive 同時 commit 方式で対応。
本記録は F100 で確立した修正パターンをテンプレ化し、残り 13 件で同様 CRITICAL が
出た場合の判断基準を提示する。

### 発覚日時
2026-05-04 (F277 引き継ぎ後の F278-Pre Step 7-A-2-① 実施時)

### 影響範囲
- F100 J-Quants V2 (本記録での修正対象)
- 残り 13 retroactive commit で同様パターンが発生する可能性 (要計測)

---

## 2. CRITICAL 1: broad except → raise + counter

### 問題

`market_data.historical.HistoricalDataFetcher.fetch_daily` の `except Exception`
が認証失敗 / レート制限 / 構造的 DB エラーを `failed_symbols` として swallow。
ジョブ exit 0 で「成功」扱い、F230/F241 historical-data 前提を falsely 満たす。

F271 §6-2 (exit code 0 を成功と見なす) / §6-6 (偽 PASS 見逃し) のリスク。

### 修正パターン (F277 commit 1 設計準拠)

`simulation/paper_live/runner.py:138-160` の `handle_symbol_error` 設計を踏襲:

```python
threshold = Config.HISTORICAL_FETCH_FAILURE_THRESHOLD  # 50 = 1.25%
client = self._get_client()
for i, code in enumerate(symbols):
    try:
        raw_rows = client.get_daily_prices(code=code, ...)
        inserted = self._save_rows(raw_rows)
        result.n_rows_inserted += inserted
    except (JQuantsAuthError, JQuantsRateLimitError) as e:
        # 構造的エラー: 認証失敗 / レート制限枯渇 → 即 raise
        result.error_msg = f"{type(e).__name__}: {e}"
        result.completed_at = _now_utc()
        raise
    except sqlite3.Error as e:
        # 構造的エラー: DB エラー → 即 raise
        result.error_msg = f"sqlite3.{type(e).__name__}: {e}"
        result.completed_at = _now_utc()
        raise
    except Exception as e:
        # 個別銘柄エラー: counter + 閾値判定
        result.failed_symbols.append(f"{code}: {type(e).__name__}: {e}")
        logger.warning("  失敗 %s: %s", code, e)
        if len(result.failed_symbols) > threshold:
            result.error_msg = (
                f"historical fetch failure threshold exceeded: ..."
            )
            result.completed_at = _now_utc()
            raise RuntimeError(result.error_msg) from e
        continue
```

### 設計上のポイント

| 観点 | 内容 |
|---|---|
| 構造的エラーの境界 | 認証 / レート制限 / sqlite3 全般を即 raise |
| 個別銘柄エラーの扱い | failed_symbols に蓄積 + counter |
| 閾値 | `Config.HISTORICAL_FETCH_FAILURE_THRESHOLD = 50` (F277 と同基準 1.25%) |
| 閾値超で raise | `RuntimeError(...) from e` で原因例外を chain |
| 設定方式 | `utils/config.py` に環境変数経由で定義、ハードコード回避 |

### F277 設計との差分

- F277 (tick.py): `Config.TICK_FAILURE_THRESHOLD_PER_RUN = 50`
- F100 (historical.py): `Config.HISTORICAL_FETCH_FAILURE_THRESHOLD = 50`
- 両者は別運用観点だが基準値は同じ (約 4,000 銘柄の 1.25% 超で構造的問題)
- F277 は `nonlocal` counter (closure)、F100 は `result.failed_symbols` (dataclass field) で実装方法は異なるが、思想は同一

---

## 3. CRITICAL 2: dataclass → pydantic + Optional 返却 + Schema Error raise

### 問題 (CRITICAL 2-A: 空文字 corrupt データ)

外部 API rows を dataclass + `.get()` defaults でパース、pydantic バリデーション
無し。欠損 `Code` / `Date` が `""` (空文字) として保持され、`market_prices_daily`
の NOT NULL 制約 (空文字を reject しない) で corrupt データが persist。

CLAUDE.md「Codex レビュー前提情報」の方針 (pydantic で外部 API レスポンス検証必須)
と乖離。

### 問題 (CRITICAL 2-B: 全 row 失敗の偽成功)

CRITICAL 2-A の最初の修正試行で、`_parse_*` を Optional 返却 + warn + None で
skip する設計にしたところ、Codex pre-commit が追加 CRITICAL を指摘:

> API スキーマ変更や必須項目欠落時に「0件保存・失敗0件・exit 0」の偽成功になる

→ warn + None だけだと API スキーマ変更で全 row 失敗時に偽成功 (F271 §6-2 違反)。
構造的エラーとして上位に伝播させる必要がある。

### 修正パターン

#### Step 1: models.py を pydantic.BaseModel 化

```python
from pydantic import BaseModel, Field

class ListedInfo(BaseModel):
    code: str = Field(..., min_length=1)  # 空文字 reject
    company_name: Optional[str] = None
    ...
    fetched_at: Optional[str] = None

    def to_dict(self) -> dict:
        return self.model_dump()  # 既存 to_dict() 互換性維持
```

#### Step 2: 専用例外 `MarketDataSchemaError` を fetcher.py に定義

```python
class MarketDataSchemaError(Exception):
    """全 row が pydantic validation に失敗 (API スキーマ変更等の構造的問題)

    warn + None だけでは「0件保存 / exit 0」の偽成功 (F271 §6-2/§6-6)
    を引き起こすため、構造的エラーとして上位に伝播させる。
    """
```

#### Step 3: `_parse_*` を Optional 返却化 (個別 row skip)

```python
def _parse_listing(raw: dict, fetched_at: str) -> Optional[ListedInfo]:
    """空文字 code なら ValidationError → None 返却 (個別 row skip)"""
    try:
        return ListedInfo(code=raw.get("Code", ""), ...)
    except ValidationError as e:
        logger.warning("ListedInfo parse 失敗 (skip): %s (%s)", raw, e)
        return None
```

#### Step 4: 呼び出し側で None 除外 + 全 row 失敗時 raise

```python
listings = [
    li for li in (_parse_listing(r, fetched_at) for r in raw_data)
    if li is not None
]
if raw_data and not listings:
    # 全 row pydantic 失敗 → スキーマ変更等の構造的問題
    raise MarketDataSchemaError(
        f"All {len(raw_data)} listing rows failed pydantic validation; "
        f"possible API schema change."
    )
```

#### Step 5: 上位 (HistoricalDataFetcher.fetch_daily) の except に追加

```python
try:
    raw_rows = client.get_daily_prices(...)
    inserted = self._save_rows(raw_rows)  # MarketDataSchemaError 可能性あり
    result.n_rows_inserted += inserted
except (
    JQuantsAuthError,
    JQuantsRateLimitError,
    MarketDataSchemaError,  # ← 構造的エラーとして即 raise
) as e:
    result.error_msg = f"{type(e).__name__}: {e}"
    result.completed_at = _now_utc()
    raise
```

### 設計上のポイント

| 観点 | 内容 |
|---|---|
| 必須フィールド | `code` (全モデル) / `date` (DailyPrice) / `disclosure_date` (FinSummary) |
| validation | `Field(..., min_length=1)` で空文字を reject |
| 既存互換性 | `to_dict()` メソッドを残置し、内部で `model_dump()` を呼ぶ |
| 個別 row エラー | warn + None 返却、呼び出し側で除外 (個別 row 失敗は銘柄失敗扱いにしない) |
| 全 row 失敗 | `MarketDataSchemaError` raise (構造的エラー、即上位伝播) |
| エラー種別の階層 | row レベル (skip) → batch レベル (schema raise) → API/DB レベル (auth/rate/sqlite raise) |

### pydantic バージョン
- pydantic 2.13.3 採用 (Mac mini 環境で確認済)
- v2 API: `Field(..., min_length=1)` / `model_dump()` / `field_validator`

### 対応経緯

CRITICAL 2 修正は Codex pre-commit との 2 回往復で確定:

1. 初回試行: `_parse_*` を Optional + warn + None
   → Codex 追加 CRITICAL 指摘 (全 row 失敗で 0 件保存 + exit 0 偽成功)
2. 修正: `MarketDataSchemaError` 追加 + raw_rows>0 かつ parsed=0 で raise
   → F271 §6-2/§6-6 完全解消

教訓: 「skip + log」は個別エラーには有効だが、**全件失敗の検出には raise が必須**。
F271 §6-2 (exit code 0 偽成功) を回避するには、batch レベルでの構造的エラー検出が
必要。

---

## 3.5 graceful degradation 設計の取り扱い (F101 で確立)

F101 retroactive commit で発覚した追加論点。一部の関数 (XBRL パース系) は
**意図的に graceful degradation 設計** を採用しており、broad except
を限定形に絞るか broad のまま維持するかの判断軸を整理。

### graceful degradation の判定基準

以下を全て満たす場合に graceful degradation 設計と認定:

1. **コメント明記の意図的設計**
   (例: xbrl_parser.py docstring「パース失敗時は raise せずに parse_error
    を XbrlParseResult に格納 (degradation)」)
2. **試行錯誤の前提**
   (リトライ可能、失敗が予期事象、外部標準仕様の不確実性に依存)
3. **失敗を記録して処理継続が運用上妥当**
   (例: `xbrl_parse_error` カラムに記録、`success=False` で上位伝達、
    複数 announcement のうち一部失敗は許容)

### broad except を限定形に絞る vs broad 維持の判断軸

| 状況 | 判断 | 例 |
|---|---|---|
| 例外型が網羅可能 (5 種程度以下) | **限定形に絞る** | xbrl_parser.py: zipfile.BadZipFile / KeyError / OSError / ValueError / ET.ParseError |
| 例外型が多岐で網羅困難 | **broad 維持 + 構造的エラーのみ raise** | fetcher.py:423 (XBRL DL): requests / zipfile / OSError / ValueError 等多数、sqlite3.Error のみ raise に変更 |

### sqlite3.Error 等の構造的エラーの原則

graceful degradation 内でも以下は raise:

- `sqlite3.Error` 全般 (DB 構造的エラー)
- `JQuantsAuthError` / `JQuantsRateLimitError` (認証/レート制限)
- 各 schema error (`MarketDataSchemaError` / `MaterialsSchemaError` 等)

graceful 内 raise の実装パターン:

```python
try:
    zip_bytes = client.fetch_xbrl_zip(xbrl_url)
except sqlite3.Error:
    raise  # 構造的エラーは graceful 内でも raise
except Exception as e:
    # graceful degradation: DB に xbrl_parse_error 記録
    self._record_xbrl_error(announcement_id, str(e))
```

### F278-Pre 残り件で graceful degradation 採用可能性

- **F260 OpenClaw agents**: ドキュメント主、graceful 採用低
- **F230 Batch Replay**: 過去日付バッチ、リトライ可能性あり、graceful 検討
- **F236 LINE 5 段階**: 通知失敗の扱い、graceful 採用可能性中

各タスク着手時に上記判断軸を適用する。

---

## 3.6 F236 で発見された新規パターン (3 件)

F236 retroactive commit で Codex pre-commit が連続発覚した新規 CRITICAL を
分類。F100/F101 で確立した「broad except / pydantic / silent success」とは
異なる、運用ステージや実行環境に絡む構造的問題。

### §3.6.1 Stage 別 layer ハンドリング

**問題**: 仮想ポジション数 = 0 を「建玉なし」と即断する設計は、Stage 3
(Live Advisory) 移行後に実建玉が楽天証券側に存在する場合 silent suppress
の致命リスク。FIRE 内部 0 件 ≠ 実建玉 0 件。

**設計原則**:

- `OPERATIONAL_STAGE = m0/m1/m2_plus` では count = 0 でも通知
- メッセージに「FIRE 認識 0 件、楽天確認」を併記
- paper_live モード (Stage 0〜2) では従来通り skip

**判定基準**:

| 運用ステージ | 内部状態の信頼性 | count=0 時の挙動 |
|---|---|---|
| Stage 0〜2 (paper_live) | 内部状態を信用 | skip OK |
| Stage 3 以降 (m0/m1/m2_plus) | 内部状態を疑う | 通知必須 (人間確認を促す) |

**該当箇所**: F236 `scripts/emergency_alert.py` (Stage 別分岐)

**実装パターン**:

```python
STAGE3_OPERATIONAL_MODES = frozenset({"m0", "m1", "m2_plus"})

def _is_stage3() -> bool:
    return os.getenv("OPERATIONAL_STAGE", "paper_live") in (
        STAGE3_OPERATIONAL_MODES
    )

# send_emergency_alert 内
if n_open == 0 and not _is_stage3():
    return {"status": "skipped", ...}
if n_open == 0 and _is_stage3():
    message += "\n※ FIRE 認識 0 件、実建玉は楽天証券で確認してください"
```

### §3.6.2 launchd / cron 経由実行時の env ロード

**問題**: launchd plist の `EnvironmentVariables` に `PYTHONPATH` のみ設定し、
`LINE_*` / `DB_PATH` / `FIRE_HOME` 等が読まれない。秘密情報を plist に書くのは
禁忌 (logs に出る、git tracked 化リスク等)。

**設計原則**:

- plist 側は `PYTHONPATH` / `WorkingDirectory` のみ
- Python 側で `from utils.config import Config` import 時に
  `load_dotenv()` で `.env` 自動ロード
- 秘密情報は `.env` のみで管理、plist には書かない

**該当箇所**: F236 `docs/launchd/*.plist` + `scripts/emergency_alert.py` +
`utils/config.py:14-20`

**実装パターン**:

```python
# scripts/emergency_alert.py 冒頭
from utils.config import Config  # noqa: F401  -- dotenv 自動ロード起動
```

```xml
<!-- plist 側 -->
<key>EnvironmentVariables</key>
<dict>
    <key>PYTHONPATH</key>
    <string>/Users/bluefire/fire</string>
</dict>
```

### §3.6.3 DB エラー時のレイヤ別ハンドリング (3 層分離)

**問題**: DB 不調時の例外処理を単一レイヤで扱うと、Stage 3 で「建玉数不明 +
楽天確認」緊急通知が silent fail する。逆に Stage 0〜2 で raise しすぎると
launchd の再試行機構が機能しない。

**設計原則 (3 層分離)**:

| レイヤ | 関数 | 挙動 |
|---|---|---|
| Layer 1 | `count_open_positions` | `sqlite3.Error` は raise (構造的エラーは隠さない) |
| Layer 2 | `send_emergency_alert` | catch、Stage 別に分岐 |
| Layer 3 | `main` (launchd 起動点) | 致命エラー catch、stack trace ログ記録 |

**Layer 2 の Stage 別分岐**:

- Stage 3 (m0/m1/m2_plus): 「建玉数不明 + 楽天確認」緊急通知
- Stage 0〜2 (paper_live): raise (launchd 次回 cron で再試行、R-22-09)

**該当箇所**: F236 `scripts/emergency_alert.py` (3 層分離)

**実装パターン**:

```python
# Layer 2: send_emergency_alert
try:
    n_open = count_open_positions(db_path)
except sqlite3.Error as e:
    if _is_stage3():
        # 建玉数不明として緊急通知
        message = template.format(n="?") + (
            f"\n※ DB 取得失敗 ({type(e).__name__}): {e}"
            "\n※ 実建玉は楽天証券で確認してください"
        )
        return send_to_room(...)
    raise  # Stage 0-2: launchd に委ねる
```

### §3.6.4 残り適用方針 (パターン → 検討タスクの対応表)

F236 で発見された新規パターン 3 件 + F116 で発見された §3.6.5 を含めた
適用検討マッピング:

| パターン | 適用検討タスク | 理由 |
|---|---|---|
| §3.6.1 Stage 別 layer | F119 / F230 / F241 | Stage 3 移行で挙動変更が必要 (Evaluation/Batch Replay/Live Advisory) |
| §3.6.2 launchd env | F057 / F058 | cron 経由実行が絡むタスク (Scheduler / E2E smoke) |
| §3.6.3 DB エラーレイヤ別 | F130-F140 / F054 | DB 操作が複雑なタスク (Risk / Paper Live tick) |
| §3.6.5 戻り値 status 検証 | F119 / F230 / F241 / F054 / F277 残作業 | graceful 戻り値関数を呼ぶ側の検証要確認 |
| §3.6.6 後知恵バイアス防止 | F230 / F241 / F054 / F277 残作業 | as_of_dt / 基準時刻フィルタの SQL 適用要確認 |
| §3.6.7-9 daemon パターン | F054 / F230 / F241 / F277 残作業 | SIGTERM ハンドラ / stale tick 防止 / 分割 sleep 要確認 |
| §3.6.10 判定ロジックの判定基準完全性 | F230 / F241 / F054 | is_go / is_success 等の部分判定リスク確認 |
| §3.6.11 state-ful 時系列順実行原則 | F241 / F054 | state-ful システムの時系列逆順実行による look-ahead bias 確認 |

各タスク着手時に事前 grep でパターン該当を確認、本部修正方針確認に含める。

### §3.6.5 戻り値 status 検証 (silent fail 防止) ★F116 で発見

**問題**: graceful degradation 設計で `status=ok / status=error` を戻り値辞書で
返却するパターンを採用した時、呼び出し側が status を検証しないと silent fail
発生。例外なしで処理継続 → 「送信済」と誤認 → 利確/損切指示等の永久欠落リスク。

**設計原則**:

- graceful 戻り値関数 (status を辞書で返却) を呼ぶ側は必ず status 検証
- status が `("ok", "dry_run")` 等の成功扱い値以外なら失敗処理
  (リトライ / アラート / 記録)
- duplicate 記録や「送信済」フラグ更新は status 成功時のみ実施

**判定基準**:

- 呼び出し関数が graceful 戻り値関数を使用 → status 検証必須
- status を見ずに副作用 (duplicate 記録 / フラグ更新) を実行 → CRITICAL

**該当箇所**: F116 `_send_to_execution_room` (commit `e1b360a`)

**実装パターン**:

```python
# Before (silent fail リスク)
self._send_to_execution_room(message)  # status を見ない
self._record_notified(...)  # 失敗でも duplicate 記録される
result.n_sent += 1

# After (F116 commit e1b360a)
send_result = self._send_to_execution_room(message)
status = (
    send_result.get("status")
    if isinstance(send_result, dict) else None
)
if status not in ("ok", "dry_run"):
    result.n_failed += 1
    result.failed_events.append({...})
    continue  # duplicate 記録なし、再試行可能性を保つ
self._record_notified(...)  # 成功時のみ
result.n_sent += 1
```

**残り 8 件への適用方針**:

- F119 Evaluation 3 Phase: 評価結果を辞書で返却するパターン要確認
- F230 Batch Replay / F241 Live Advisory: graceful 戻り値関数呼び出し側の
  検証要確認
- F054 Paper Live tick: position 更新の戻り値検証要確認
- F277 残作業 (commit 2/3/4): cache invalidation 後の戻り値検証要確認

### §3.6.6 後知恵バイアス (look-ahead bias) 防止 ★F119 で発見

**問題**: 評価ロジックで「この時点までの情報で評価する」基準時刻 (例:
`as_of_dt`) を引数で受け取るが、各 SQL の WHERE 条件で適用していないと、
評価対象期間内で `as_of_dt` より後のデータが混入する。Replay / Paper Live
の評価・提案が構造的に歪み、F271 §6-2 (動いた / 機能した混同) の重大違反。

**設計原則**:

- `as_of_dt` 等の基準時刻引数を受け取る関数は、**すべての SQL の WHERE 条件**
  に適用必須
- 動的データ (positions / results / features / observed_at) は基準時刻で
  フィルタ
- 静的データ (patterns 等の確定済みマスタ) はフィルタ不要 (ただし設計判断を
  コメントで明示)

**判定基準**:

- 関数が「特定時刻までの情報で評価する」契約を持つ → 全 SQL に時刻フィルタ必須
- フィルタ不要のデータは設計コメントで明示 (静的マスタ等)

**検証方法**:

- `as_of_dt` 境界値テスト必須 (`as_of_dt` より後のデータが除外されることを実証)
- mock ではなく **実 DB に未来データを INSERT** して検証 (F271 §6-6 偽 PASS
  見逃し回避)

**該当箇所**: F119 `aggregators.py` (commit 予定、retroactive 同時)

**実装パターン**:

```python
# Before (silent look-ahead bias)
cursor = conn.execute(
    """
    SELECT COUNT(*) FROM paper_live_positions
    WHERE run_id = ? AND DATE(entry_dt) BETWEEN ? AND ?
    """,
    (run_id, period_start.isoformat(), period_end.isoformat()),
)

# After (F119 commit)
cursor = conn.execute(
    """
    SELECT COUNT(*) FROM paper_live_positions
    WHERE run_id = ? AND DATE(entry_dt) BETWEEN ? AND ?
      AND entry_dt <= ?
    """,
    (run_id, period_start.isoformat(), period_end.isoformat(),
     as_of_dt.isoformat()),
)
```

**TZ 考慮**:

`as_of_dt` (UTC) と DB の dt (ISO 文字列、TZ あり) を比較する場合、
SQL は文字列比較で動作する。同じ TZ 表記 (`+00:00` 等) なら辞書順で
正確に比較できる。

集計関数で `_resolve_period` が JST 換算で period を決める場合、
`as_of_dt` (UTC) を JST date に変換してから period 算出するため、
テストの `as_of_dt` は UTC 14:59:59 (= JST 23:59:59) のように設定する
必要がある (F119 retroactive で発見)。

**残り 7 件への適用方針**:

- F230 Batch Replay: バックテストの基準時刻フィルタ必須、要確認
- F241 Live Advisory: 推奨時刻基準の評価フィルタ必須、要確認
- F054 Paper Live tick: tick 時刻基準のデータ参照、F277-B との関連
- F277 残作業: cache 経由のデータ参照に基準時刻があるか要確認

### §3.6.7 daemon プロセスの SIGTERM ハンドラ必須 ★F057 で発見

**問題**: 長時間動作する daemon プロセス (scheduler / tick loop / cron 系)
で SIGTERM ハンドラを実装していない場合、launchd / systemd / Docker からの
停止要求で強制 kill となり、中途半端な DB 状態 (トランザクション未 commit、
position 状態不整合) が残存するリスク。

**設計原則**:

- daemon プロセスは SIGTERM / SIGINT を `install_signal_handlers` で捕捉
- 受信時は `_stop_requested` フラグを True に設定、現在の処理を完了後に終了
- `end_session` 等の cleanup を保証

**判定基準**:

- `while True` ループ等で長時間動作するプロセス → SIGTERM ハンドラ必須
- one-shot 実行 (CLI コマンド一発) → 不要

**該当箇所**: F057 `scheduler.py` `install_signal_handlers` (commit `a4fc339`)

**実装パターン**:

```python
def install_signal_handlers(self) -> None:
    """SIGINT/SIGTERM を request_stop に接続"""
    signal.signal(signal.SIGINT, lambda s, f: self.request_stop())
    signal.signal(signal.SIGTERM, lambda s, f: self.request_stop())

# main() で起動時に呼ぶ (テスト時は呼ばない、pytest 環境に signal handler
# を残さない)
```

### §3.6.8 daemon 再起動時の stale tick 防止 ★F057 で発見

**問題**: daemon プロセスが起動時に「前回の最終 tick 時刻」を引き継ぐと、
再起動時に過去時刻の tick が誤実行される (例: 14:50 で停止 → 15:00 に再起動
→ 14:55 / 15:00 の tick が連続実行)。実取引で誤発注リスク。

**設計原則**:

- daemon 起動時に start_dt を「現時刻」または「設定値の整列値」に設定
- 過去の未実行 tick はスキップ (catch-up しない)
- データ整合性を時刻の精度より優先

**判定基準**:

- 起動時刻基準で tick を実行する設計 → start_dt を現時刻整列必須
- バックテストで過去時刻を再生する設計 → 例外 (意図的な過去時刻実行、
  本番運用と区別するフラグで切替)

**該当箇所**: F057 `scheduler.py` start_dt 整列 (commit `a4fc339`)

**実装パターン**:

```python
if self.sleep_until_next_tick:  # 本番運用フラグ
    now = datetime.now()
    if now > start_dt:
        elapsed = (now - start_dt).total_seconds()
        n_intervals = int(elapsed // self.interval_sec) + 1
        start_dt = start_dt + timedelta(
            seconds=n_intervals * self.interval_sec
        )
```

### §3.6.9 daemon 分割 sleep による graceful shutdown ★F057 で発見

**問題**: `time.sleep(interval_sec)` で長時間 sleep すると、SIGTERM 受信時に
即応できず launchd / systemd のタイムアウト (通常 5-10 秒) で強制 kill される。
graceful shutdown 不可。

`time.sleep` は Python 3.5+ PEP 475 でも OS / signal 種別によって signal 受信
後に再開され得る (実装依存)。確実な signal 即応のためには分割 sleep が必須。

**設計原則**:

- 長時間 sleep は **1 秒チャンクに分割**
- 各チャンク後に `_stop_requested` を確認
- 停止要求受信時は即座に break して cleanup へ

**判定基準**:

- daemon プロセスで `interval_sec >= 5 秒` の sleep → 分割必須
- one-shot 短時間 sleep → 不要

**該当箇所**: F057 `scheduler.py` 分割 sleep 実装 (commit `a4fc339`)

**実装パターン**:

```python
if self.sleep_until_next_tick:
    delta = (current - datetime.now()).total_seconds()
    while delta > 0 and not self._stop_requested:
        chunk = min(delta, 1.0)  # 最大 1 秒ずつ
        time.sleep(chunk)
        delta -= chunk
    if self._stop_requested:
        break
```

### §3.6.10 判定ロジックの判定基準完全性 ★F058 で発見

**問題**: 「Go/No-Go」「成功/失敗」等の判定関数で、判定に必要な要素を全て参照
していない場合、特定条件下で常時誤判定 (例: 常時 NO-GO 化、常時 OK 化) する。
F271 §6-2 (動いた / 機能した混同) の典型違反。

F058 で発覚したパターン:
- `is_go(session_result)` が session_result のみ参照 → readiness / events /
  promotion が 0 件でも GO 出力リスク
- 加えて `aggregate_events` に `events_total` 不在で `is_go(events=events)`
  時に常時 NO-GO 化 (集計関数と判定関数の戻り値構造の不整合)

**設計原則**:

- 判定関数は判定に関わる **全要素** を引数として受け取る
- 部分情報のみで判定する場合は「未確定」「保留」等の中間状態を返す
- 後方互換のために Optional 引数とする場合、**全要素 None 時の挙動を明示**
- 集計関数 → 判定関数のデータフローで、戻り値構造の不整合がないか確認

**判定基準**:

- `is_go` / `is_success` / `is_valid` 等の判定関数 → 判定要素の網羅性確認必須
- 判定要素を集計関数等に依存する場合、集計関数の戻り値構造を確認
- 判定基準を「session_result のみ」のような部分情報で行うのは CRITICAL

**該当箇所**: F058 `is_go` 判定 + `aggregate_events` events_total
(commit `f4754c4`)

**実装パターン**:

```python
def is_go(
    session_result: dict,
    readiness: Optional[dict] = None,
    events: Optional[dict] = None,
    promotion: Optional[dict] = None,
) -> bool:
    """全要素 AND で判定 (Optional 引数は後方互換)"""
    session_ok = (
        len(session_result.get("errors", [])) == 0
        and session_result.get("tick_count", 0) > 0
        and not session_result.get("stopped_by_request", False)
    )
    if not session_ok:
        return False

    if readiness is not None:
        if (readiness.get("market_listings", 0) <= 0
                or readiness.get("features_for_date", 0) <= 0
                or ...):
            return False

    if events is not None and not events.get("events_total"):
        return False

    if promotion is not None and "error" in promotion:
        return False

    return True
```

集計関数側の補完:

```python
def aggregate_events(...) -> dict:
    return {
        "events_by_type": events_by_type,
        # 判定関数で参照される集計値を露出
        "events_total": sum(events_by_type.values()),
        ...
    }
```

**残り 3 件への適用方針**:

- F230 Batch Replay: バックテストの成功判定が is_go 同型なら適用
- F241 Live Advisory: Advisory の出力可否判定が is_go 同型なら適用
- F054 Paper Live tick: tick 結果の判定 (TickResult.status) が部分判定なら適用

### §3.6.10 (補足) 設計意図と要件根拠の混同警戒 ★F241 で発見

**問題**: 「環境固有プレフィックス」「ローカル設定前提」等の設計意図を理由に
部分判定 (substring 一致 / 時刻のみ判定) を許容すると、要件根拠 (具体的な
登録名 / 期待件数) を満たさない偽 PASS を生む。F271 §6-2 違反 + F058 is_go
常時 NO-GO 化と対称的な「常時 OK 化」リスク。

**設計原則**:
- 設計意図 (= なぜそう実装するか) と要件根拠 (= 何を検証すべきか) を分離
- 部分判定は「要件根拠の証明として不十分」が前提、デフォルト避ける
- 環境固有性が必要な場合は「具体的な期待値セット」を定数化

**判定基準**:
- substring 一致 / 部分一致のみで判定 → 要件根拠の証明として不十分、CRITICAL 候補
- 期待値セット (具体名 / 件数) を定数化していれば適切
- 設計意図のコメントがあっても、要件根拠の証明性を別途確認

**該当箇所**: F241 _has_bluefire_launchd 厳密化 (commit 後追補)
- 旧: `"bluefire.fire" in proc.stdout` (部分一致、関連 plist だけで PASS)
- 新: `EXPECTED_LAUNCHD_PLISTS = {jp.fire.emergency-1445/1455/1505/1510/1515}`
  との厳密 set 比較
- 旧: OpenClaw cron は schedule.expr の時刻のみ判定 (無関係ジョブで偽 PASS)
- 新: command/script に "emergency_alert" / "scripts.emergency_alert" 含む
  ジョブのみカウント

**事前 grep の限界 6 度目実証**:
- 本部が「設計意図 (環境固有プレフィックス) 維持」と判定したが誤り
- F230 §3.6.11 で学んだ「コメントへの過信警戒」と同型の認知バイアス
- F271 v1.2 アンチパターンとして追加検討候補

**残り 1 件への適用方針**:
- F054 Paper Live tick: 設計意図と要件根拠の分離を事前 grep に追加、
  部分判定 / substring 一致 / 時刻のみ判定がないか厳密確認

### §3.6.10 (補足 v2) 配信成功 / 副作用検証への拡張 ★F241 で発見

**問題**: 配信 / 永続化 / 副作用を伴う処理で `status="ok"` を成功の証明として
扱うと、以下の偽 PASS が発生:
- dry_run mode で実送信なしでも `"ok"` 扱い (例: `LINE_DRY_RUN=true`)
- fallback 先 (個人 ID 等) に統合されて全部成功扱い (例: 5 部屋が同一宛先)
- 戻り値の構造的検証なし (例: 重複宛先の検出なし)

検証ツールが `status="ok"` を信じて配信成功と判定 → 実態は配信されていない
or 意図しない宛先に配信されたまま「OK」表示 = F271 §6-2 + §6-6 違反。

**設計原則**:
- 配信検証では `status="ok"` だけでなく配信実態の付随検証必須:
  - dry_run mode の判別 (`status="dry_run"` は配信成功ではない)
  - fallback の検出 (個人 ID への統合等)
  - 宛先一意性の検証 (重複宛先での全成功は偽 PASS)
  - 配信件数の妥当性確認 (期待数と一致するか)
- 副作用検証 (DB 保存、ファイル書き込み等) でも同型のチェックが必要

**判定基準**:
- `send_*`, `write_*`, `save_*` 等の副作用関数の戻り値を `status="ok"` のみ
  で判定 → CRITICAL 候補
- 戻り値に dry_run / fallback 状態を表すフィールドがあれば、それも検証
- 複数宛先 / 複数操作の場合、一意性 / 妥当性の前提検証を追加

**該当箇所**: F241 `_check_line_rooms` 厳密化 (commit 後追補)
- success 判定: `status="ok"` のみ通過、`"dry_run"` は明示的に FAIL
- `_check_line_room_id_uniqueness` 新メソッド: 5 部屋一意性前提検証
  (未設定 / fallback 統合 / 重複の 3 観点)

**連鎖検出パターン**:
- 登録検証 (補足 v1) の偽 PASS を直したら配信検証 (補足 v2) の偽 PASS が
  連鎖検出。検証ツールの偽 PASS は層が深く、Codex の補完価値継続証明。
- 事前 grep の限界 7 度目実証。

**残り 1 件への適用方針**:
- F054 Paper Live tick: 評価結果保存 / 通知送信の status 検証に適用
  - DB 保存の戻り値検証 (rowcount / lastrowid)
  - 通知送信の dry_run / fallback 検出

### §3.6.11 state-ful システムの時系列順実行原則 ★F230 で発見

**問題**: state-ful システム (仮想建玉・資金・履歴を保持する Paper Live 等) で
時系列逆順実行 + 同一 run_id 維持を行うと、未来日の state が過去日に引き継がれる。
SQL レベルの look-ahead bias を `as_of_dt` フィルタ (§3.6.6) で解消しても、
state レベルで再導入される構造的問題。F271 §6-2 重大違反。

F119 §3.6.6 で SQL レベル look-ahead bias を解消したが、F230 の「最新→過去
順遡及 + 同一 run_id 貫通」設計で state レベル look-ahead bias が再導入されて
いた事例。

**設計原則**:

- state-ful システムは時系列順 (過去→最新) で実行する
- 順序を逆にする必要がある場合は、各時点で state リセット (新 run_id 等)
- コメントで「look-ahead bias なし」と主張していても、**state 引き継ぎ構造を
  実装で確認** する (Codex が設計意図と実装の矛盾を検出)

**判定基準**:

- `runner.tick(as_of_dt=...)` のような関数を時系列逆順で呼ぶ → 設計違反
- バッチ実行で複数日を貫通する場合、各日の state 整合性を確認
- 「古いデータ過学習回避」目的で逆順を選ぶ場合、state 影響を別途解消

**該当箇所**: F230 `batch_replay.py` 順序変更 (commit `bb4d00d`)

**実装パターン**:

```python
# Before (state レベル look-ahead bias)
def list_business_days_descending(...) -> list[date]:
    """end_date から遡って最新→過去の順で返却"""
    # SQL: ORDER BY date DESC で最新 N 日取得
    return [date.fromisoformat(d) for d in rows]  # descending

# After (F230 commit bb4d00d)
def list_business_days_chronological(...) -> list[date]:
    """最新 N 日を時系列順 (過去→最新) で返却"""
    # SQL は DESC (最新 N 日取得) のまま、Python 側で sorted() で ASC
    return sorted(date.fromisoformat(d) for d in rows)  # chronological
```

**残り 2 件への適用方針**:

- F241 Live Advisory: 状態保持の有無確認、time-series 順実行原則適用候補
- F054 Paper Live tick: tick loop が state-ful そのもの、★最重要適用候補
  (F230 と同じく仮想建玉・資金保持、tick の as_of_dt 順序を確認)

**教訓 (F271 v1.2 提案候補)**:

「コメント主張への過信警戒」: 「look-ahead bias なし」とコメントに書いて
あっても、構造で実証されているか確認する。SQL レベルだけでなく state レベル
も含めた多層的検証が必要。

---

## 4. 残り 13 件への適用判断基準

各 retroactive commit で Codex pre-commit hook が CRITICAL を出した場合の判断:

### 案A: 既知パターン → 案④継続 (本部協議不要)

以下のパターンに該当する CRITICAL は本テンプレを参照して修正実施:

- broad `except Exception` (構造的エラーの swallow リスク)
- dataclass + `.get()` defaults (pydantic 不在)
- NOT NULL 制約緩和 (空文字 persist リスク)
- 個別エラー閾値の不在 (累積偽 PASS リスク)

修正後は本記録を更新し、新たな修正例として追記する。

### 案B: 新規パターン → 本部協議

以下に該当する CRITICAL は 1 件ごとに本部協議:

- 非同期処理 (aiohttp ClientSession 管理 / Semaphore)
- launchd 連携 (緊急アラート plist の整合)
- LINE 配信 (5 部屋ルーティング / 失敗時リトライ)
- SQLite トランザクション境界 / WAL / マイグレーション冪等性
- 外部 API リトライ・フォールバックの構造的不備

これらは fire 固有のレビュー観点 (CLAUDE.md 「fire 固有のレビュー観点」セクション)
で個別対応が必要。

### 案C: 累積で 1.5 日超過見込み → 本部再協議

CRITICAL が 3 件以上累積し F278-Pre 完了が 3 日超見込みになった時点で本部再協議。
場合により案② (F277 完了後の F278 本体に持ち越し) に切替判断。

---

## 5. 適用予測 (残り 13 件)

| 番号 | タスク | 想定 CRITICAL | 案別判定 |
|---|---|---|---|
| ② | F101 TDnet/IR | dataclass + 外部 API 多用 | 案A (CRITICAL 2 同様) |
| ③ | F062 LINE 注文テンプレ | LINE 配信周辺 | 案B 可能性 |
| ④ | F236 LINE 5段階 | launchd 整合 / LINE 配信 | 案B 可能性 |
| ⑤ | F111 Daytrade Selection | 軽量 | 案A 不要 |
| ⑥ | F116 Monitoring Alert | LINE 配信周辺 | 案B 可能性 |
| ⑦ | F119 Evaluation 3 Phase | 大量実装 | 案A/B 混在可能性 |
| ⑧ | F130-F140 Risk 5件 | 軽量ロジック | 案A 不要 |
| ⑨ | F260 OpenClaw agents | docs のみ | CRITICAL 出にくい |
| ⑩ | F057 Scheduler | 単純スケジューラ | 案A 不要 |
| ⑪ | F058 E2E smoke | smoke test | 案A 不要 |
| ⑫ | F230 Batch Replay | broad except 既知 | 案A |
| ⑬ | F241 Live Advisory | 検証スクリプト | 案A 不要 |
| ⑭ | F054 Paper Live tick | F277 が既に対応 | 案A 不要 |

楽観: 1〜2 件の CRITICAL のみ → 1.5〜2 日
標準: 3〜4 件 → 3〜4 日
悲観: 5 件超 → 案C (本部再協議)

---

## 6. F278 本体への申し送り

### 本記録で **意図的に未対応** な事項

F278-Pre スコープ外として F278 本体で対応する事項:

- **F101 materials / F119 evaluation 等の pydantic 化**
  : F100 で pydantic 採用第 1 号となったが、他モジュールも順次 pydantic 化が必要。
    F278-Pre は CRITICAL 指摘箇所のみ修正、未指摘箇所は F278 本体で計画的に統一。
- **pre-commit hook の untracked 検知ルール追加**
  : 「コードファイルが untracked のまま 1 週間以上経過したら警告」等の自動検出。
- **CLAUDE.md 完了表更新ワークフロー再整備**
  : タスク完了時に「実装ファイル commit 確認」を組み込む。
- **dev → main merge 戦略整備 + tag**
- **TODO Excel 乖離問題の解消** (F111 着手時に発見、2026-05-05 追加)
  : Mac mini Vault 内 `02_todo/FIRE_開発TODO_v9.xlsx` (2026-04-26 更新、F243 まで)
    と Fujiwara 手元最新版 `FIRE_開発TODO_v9_20_F274-F279追加.xlsx` の乖離。
    F111 retroactive commit 時に AI ファンダ統合構想の起票 ID 確認で発覚した
    構造的問題。
    - Vault 内 v9.xlsx を最新版に同期する手順整備
    - Vault 内 Excel と Fujiwara 手元 Excel の同期ルール (どちらをマスター)
    - Mac mini Claude Code から最新版 ID を参照する手段 (運用アクセス確保)
    - ファイル名規約統一 (`v9_20` → `v9.20` 等の SemVer 系命名)

### F271 v1.2 改訂候補 (F278 本体で検討)

本事故を踏まえた追加アンチパターン:

- §6-9 候補: 「コード読解せず仮説」 (F278 git_governance 議論記録より)
- §6-10 候補: 「untracked 放置」 (F278-Pre の起点となった構造的問題)

詳細: [[git_governance_2026-05-04]]

---

## 7. 検証

### 既存テスト回帰
- `tests/market_data/test_historical.py` (14 件): 全 PASS
- `tests/market_data/test_jquants_client.py` (20 件): 全 PASS

### 新規テスト (CRITICAL 解消の意味検証)
- `tests/market_data/test_critical_handling.py` (16 件): 全 PASS
  - CRITICAL 1: 構造的エラー raise (4 件: auth / rate / threshold / under-threshold)
  - CRITICAL 2-A: pydantic validation (8 件: empty code/date reject, parse skip)
  - CRITICAL 2-B: MarketDataSchemaError (4 件: all-invalid raise / partial continue /
    fetch_daily propagation / fetch_and_save_listings raise)

合計 50 件全 PASS、F271 §6-2/§6-6 リスク解消を実証。

---

## 8. 関連リンク

- 起点 (F278): [[F278_git_ガバナンス整備]]
- F277 設計参考: [[F277_phase1_design_2026-05-04]]
- 完了基準: [[task_completion_criteria]]
- F275 関連: [[F275_similarity_optimization_complete_2026-05-04]]
- 議論記録 (F278): [[git_governance_2026-05-04]]
- code commit (予定): F100 retroactive (~/fire dev branch)
- vault commit (予定): 本記録 (~/fire-vault main branch)
