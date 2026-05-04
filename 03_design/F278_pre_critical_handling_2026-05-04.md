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

### §3.6.4 残り 10 件への適用方針

F236 で発見された新規パターン 3 件は、以下のタスクで適用検討:

| パターン | 適用検討タスク | 理由 |
|---|---|---|
| §3.6.1 Stage 別 layer | F119 / F230 / F241 | Stage 3 移行で挙動変更が必要 (Evaluation/Batch Replay/Live Advisory) |
| §3.6.2 launchd env | F057 / F058 | cron 経由実行が絡むタスク (Scheduler / E2E smoke) |
| §3.6.3 DB エラーレイヤ別 | F130-F140 / F054 | DB 操作が複雑なタスク (Risk / Paper Live tick) |

各タスク着手時に事前 grep でパターン該当を確認、本部修正方針確認に含める。

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
