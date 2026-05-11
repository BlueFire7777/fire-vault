# F286-PNL-R3 Paper PnL Simulator Hook Design (Draft)
version: v1.0 (draft)
date: 2026-05-11
status: design proposal (本線 Architect 承認待ち)

## 1. 目的

F286-PNL-R1 で追加済みの `advisory_decisions.paper_pnl` を、手入力ではなく後追いの自動計算で埋める設計を定義する。

`paper_pnl` は「Fujiwara が実際に採用したか」ではなく、「FIRE の助言をもし採用していたら、一定の仮想ルールでどれだけの損益になったか」を測る評価用 PnL である。したがって、本設計は注文生成、実発注、LINE 通知、実建玉更新には一切関与しない。

本タスクは L1 Design レーンの設計 draft のみを成果物とし、実装本体は Wave 6+ の別タスクで行う。

## 2. 現状把握 (= F286-PNL-R1 schema / paper_live module / FIRE-LABEL-R1 / R2)

### F286-PNL-R1 `advisory_decisions`

`advisory_decisions` は、F062 の `send_id` を `advisory_id` として、助言単位の Fujiwara 判断、実取引、実損益、仮想損益を保持する。

主要列:

| column | role |
|---|---|
| `advisory_id` | F062 `send_id`。助言通知単位の ID |
| `code` | 銘柄コード |
| `decision_label` | FIRE-LABEL-R1 の判断ラベル。「積極的買い推奨」「場中監視」等 |
| `fujiwara_decision` | `adopted` / `watched` / `skipped` / `rejected` / `unknown` |
| `actual_trade` | `none` / `buy` / `sell` / `partial` |
| `entry_price`, `entry_qty`, `exit_price`, `exit_qty` | 実取引記録 |
| `realized_pnl`, `unrealized_pnl` | 実損益 |
| `paper_pnl` | 本設計対象。「もし採用していたら」の仮想 PnL |

F286-PNL-R1 時点では `paper_pnl` 列は存在するが、入力前提は手動であり、算出 hook は未実装。

### `simulation/paper_live`

既存の Stage 2 Paper Live 基盤には、仮想建玉、厳しめ約定モデル、VIRTUAL_TP / VIRTUAL_SL 判定、Paper Live 昇格判定が存在する。

関連パス例:

| path | assumed role |
|---|---|
| `simulation/paper_live/tick.py` | Paper Live tick 処理、候補抽出、監視、TP/SL 判定 |
| `simulation/paper_live/models.py` | Paper Live 用モデル |
| `simulation/paper_live/position.py` | 仮想建玉 / 余力 / PnL 管理 |

本 R3 では `simulation/paper_live` の VIRTUAL_TP / SL と深く結合せず、まずは deterministic な h20 close ベースの paper PnL を計算する。TP/SL hit 型は将来拡張候補に留める。

### FIRE-LABEL-R1

`decision_label` は paper trade の対象判定に使う。

初期推奨:

| category | labels | `paper_pnl` |
|---|---|---|
| paper trade 対象 | `積極的買い推奨`, `条件付き買い推奨`, `注意つき買い候補`, `⚠️ 注意つき買い候補` | 計算対象 |
| 対象外 | `場中監視`, `見送り推奨`, `監視のみ` | `NULL` |
| unknown / 新規ラベル | 未定義ラベル | `NULL` + warning log |

新規ラベルが追加された場合に誤計算しないよう、allowlist 方式で対象ラベルを固定する。

### F286-PNL-R2 `advisory_snapshot_rows`

R2 では、各 `advisory_id` に紐づく最大 5 銘柄分の snapshot 情報が作られている。

想定利用列:

| column | use |
|---|---|
| `advisory_id` | `advisory_decisions` との join key |
| `code` | 銘柄単位の join key |
| `base_date` | 仮想 entry / exit の起点日 |
| `advisory_label`, `decision_label` | ラベル判定 |
| `f119_expected_h20_return` | h20 horizon との整合確認用 |
| `f119_expected_h20_win_rate` | 評価分析用。PnL 計算式には直接使わない |

R3 は `advisory_snapshot_rows` から `base_date` と対象銘柄を引き、価格データから仮想 entry / exit を決定して `advisory_decisions.paper_pnl` を更新する。

## 3. 仮想エントリ / エグジット定義 (= 案 a/b/c, x/y/z, 1/2/3 比較 + 推奨)

### 仮想エントリ価格

| option | definition | pros | cons | judgment |
|---|---|---|---|---|
| 案 a | `base_date` の翌営業日の寄付価格 | deterministic。実際に翌営業日に採用する前提と近い。daily OHLC から取得しやすい | 寄付ギャップの影響を受ける | 推奨 |
| 案 b | 翌営業日 9:00-9:30 の VWAP | 板寄せ直後のノイズをならせる | intraday 出来高データが必要。欠損時の扱いが複雑 | 将来候補 |
| 案 c | F119 expected h20 return から逆算 | F119 評価値と機械的に整合する | 実価格に基づかないため PnL 評価として循環参照に近い | 不採用 |

推奨は案 a。`market_prices_daily` の `open` を使い、翌営業日の寄付で仮想エントリしたとみなす。

### 仮想エグジット価格

| option | definition | pros | cons | judgment |
|---|---|---|---|---|
| 案 x | `base_date + 20 営業日` の終値 | F119 h20 horizon と一致。deterministic。検証しやすい | TP/SL による途中退出を反映しない | 推奨 |
| 案 y | VIRTUAL_TP / VIRTUAL_SL hit 時 | 既存 Paper Live の退出思想に近い | intraday または tick 経路との結合が必要。R3 初期実装として重い | Wave 6 初期では不採用 |
| 案 z | action / decision label 別 hold 期間 | ラベルの意味に応じた評価が可能 | ラベルごとの期間定義が恣意的になりやすい | 将来候補 |

推奨は案 x。F119 の h20 horizon と比較可能な gross PnL として、20 営業日後の close を使う。

### 仮想数量

| option | definition | pros | cons | judgment |
|---|---|---|---|---|
| 案 1 | F130/F131 の stage 別許容損失から逆算 | 実運用の risk sizing と整合 | stop distance 定義が必要。初期実装で依存が増える | 推奨。ただし fallback を定義 |
| 案 2 | 固定 100 株 | 単純。日本株の 1 単元評価として分かりやすい | Fujiwara の実運用サイズと乖離 | fallback |
| 案 3 | code ごとに固定 | 銘柄特性を反映可能 | 別途設定管理が必要 | 不採用 |

推奨は案 1。ただし R3 初期実装で stop distance が得られない場合は、明示的に案 2 の 100 株 fallback を使う。

数量計算の初期方針:

1. `allowance_loss` と `stop_distance_per_share` が取得できる場合:
   `qty = floor(allowance_loss / stop_distance_per_share / 100) * 100`
2. `qty < 100` の場合:
   `qty = 100` ではなく `paper_pnl = NULL` とし、最小単元を満たせない記録として warning
3. `stop_distance_per_share` が未定義の場合:
   `qty = 100` fallback を使う

この設計は paper PnL 評価のみであり、注文数量や執行指示の helper は作らない。

## 4. paper_pnl 計算式 (= 推奨案ベース、Python skeleton)

推奨案ベースの定義:

| item | value |
|---|---|
| entry date | `base_date` の翌営業日 |
| entry price | entry date の `open` |
| exit date | `base_date` から 20 営業日後 |
| exit price | exit date の `close` |
| side | 初期は long のみ |
| qty | F130/F131 sizing。取得不能時は 100 株 fallback |
| fees / tax | 計上しない。gross PnL |
| corporate actions | 分割 / 配当補正なし |

計算式:

```python
paper_pnl = (exit_price - entry_price) * qty
```

対象外または価格欠落時:

```python
paper_pnl = None
```

Python skeleton:

```python
from __future__ import annotations

from dataclasses import dataclass
from datetime import date
from math import floor
from typing import Optional


PAPER_TRADE_LABELS = {
    "積極的買い推奨",
    "条件付き買い推奨",
    "注意つき買い候補",
    "⚠️ 注意つき買い候補",
}


@dataclass(frozen=True)
class DailyPrice:
    date: date
    open: float | None
    close: float | None


def should_compute_paper_pnl(decision_label: str | None) -> bool:
    return decision_label in PAPER_TRADE_LABELS


def compute_virtual_qty(
    *,
    allowance_loss: float | None,
    stop_distance_per_share: float | None,
    fallback_qty: int = 100,
) -> int | None:
    if allowance_loss is None or stop_distance_per_share is None:
        return fallback_qty
    if allowance_loss <= 0 or stop_distance_per_share <= 0:
        return None

    qty = floor(allowance_loss / stop_distance_per_share / 100) * 100
    if qty < 100:
        return None
    return qty


def compute_paper_pnl(
    *,
    advisory_id: str,
    code: str,
    base_date: date,
    decision_label: str | None,
    entry_price: float | None,
    exit_price: float | None,
    allowance_loss: float | None = None,
    stop_distance_per_share: float | None = None,
) -> Optional[float]:
    """Compute gross paper PnL for one advisory_id/code pair.

    This function only computes evaluation PnL. It must not generate orders,
    execution instructions, LINE messages, or actual position updates.
    """
    if not should_compute_paper_pnl(decision_label):
        return None
    if entry_price is None or exit_price is None:
        return None
    if entry_price <= 0 or exit_price <= 0:
        return None

    qty = compute_virtual_qty(
        allowance_loss=allowance_loss,
        stop_distance_per_share=stop_distance_per_share,
    )
    if qty is None:
        return None

    return (exit_price - entry_price) * qty
```

実装時は、営業日計算、価格取得、DB 更新はこの pure function の外側に置く。pure function は L2 Test で価格・ラベル・数量の境界値を重点的に検証する。

## 5. 自動更新 hook 経路 (= 後追い runner、staging-only)

`paper_pnl` は `base_date + 20 営業日` の close が確定して初めて計算できる。そのため、`SnapshotStore.save_snapshot` の同期処理では計算せず、後追い runner で埋める。

新規 runner 案:

```text
scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py
```

役割:

1. `advisory_snapshot_rows` から対象 `advisory_id × code` を取得
2. `decision_label` が paper trade 対象か判定
3. `base_date` の翌営業日 open と h20 close を価格テーブルから取得
4. `compute_paper_pnl()` で gross PnL を計算
5. `--write` 指定時のみ `advisory_decisions.paper_pnl` を UPDATE
6. Fujiwara 判断列、実取引列、実損益列は更新しない

CLI 案:

```bash
.venv/bin/python scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py \
  --from-base-date 2026-04-01 \
  --to-base-date 2026-04-30
```

```bash
.venv/bin/python scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py \
  --advisory-id <send_id> \
  --write
```

default は dry-run:

| mode | behavior |
|---|---|
| default | DB write なし。対象件数、計算可能件数、NULL 理由別件数、更新予定差分を stdout に表示 |
| `--write` | staging guard 通過時のみ `paper_pnl` を UPDATE |

staging-only 三段ガード:

1. environment guard: production / live advisory DB では write 禁止
2. explicit flag guard: `--write` がない限り write しない
3. scope guard: `--advisory-id` または `--from-base-date` / `--to-base-date` の明示を必須にする

UPDATE 対象列:

```sql
UPDATE advisory_decisions
SET paper_pnl = ?
WHERE advisory_id = ?
  AND code = ?;
```

禁止:

| forbidden behavior | reason |
|---|---|
| `fujiwara_decision` の更新 | 人間判断の正本を汚さない |
| `actual_trade` / `entry_price` / `exit_price` / qty 系の更新 | 実取引記録と paper trade を混同しない |
| LINE 送信 | R3 は評価値の後追い計算のみ |
| 注文価格 / 数量 / 執行指示 helper 生成 | F286-PNL-R3 の責務外 |

## 6. cron 連携方針 (= Wave 7+、sub-D3 凍結中は手動運用)

cron / launchd 連携は Wave 7+ の別 sub-task とし、R3 初期実装には含めない。

将来方針:

| item | proposed value |
|---|---|
| schedule | 毎営業日 18:00 JST |
| target | h20 close が確定した `base_date` の advisory |
| range | 過去 20 営業日分を再計算して欠損や遅延データを吸収 |
| mode | default dry-run ではなく、staging で検証後に controlled write |

sub-D3 cron が凍結中の間は、Architect / operator が手動で runner を実行する。

手動運用例:

```bash
.venv/bin/python scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py \
  --from-base-date 2026-04-01 \
  --to-base-date 2026-04-30
```

差分確認後:

```bash
.venv/bin/python scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py \
  --from-base-date 2026-04-01 \
  --to-base-date 2026-04-30 \
  --write
```

## 7. F286-PNL-R1 seed との整合性 (= Fujiwara 判断列を touched なし)

F286-PNL-R1 seed 直後:

| column group | expected state |
|---|---|
| advisory identity | `advisory_id`, `code`, `decision_label` が seed 済み |
| Fujiwara 判断 | 初期値または手動入力値 |
| 実取引 | 実記録がなければ `none` / `NULL` |
| `paper_pnl` | `NULL` |

R3 runner 実行後:

| column group | expected state |
|---|---|
| `paper_pnl` | h20 確定済みかつ対象ラベルなら数値。対象外 / 未確定 / 欠損なら `NULL` |
| Fujiwara 判断 | touched なし |
| 実取引 | touched なし |
| 実損益 | touched なし |

`paper_pnl` は実取引の正誤判定ではなく、FIRE 助言の機会損益を測る評価列として扱う。

想定分析:

| case | interpretation |
|---|---|
| `fujiwara_decision = skipped`, `paper_pnl > 0` | 見送り機会損失の候補 |
| `fujiwara_decision = adopted`, `realized_pnl > paper_pnl` | 実裁量が paper rule を上回った可能性 |
| `fujiwara_decision = adopted`, `realized_pnl < paper_pnl` | 退出 / サイズ / 約定の改善余地 |
| `paper_pnl IS NULL` | 対象外、未確定、または価格欠損 |

## 8. 既知のリスク (= 仮想価格欠落、h20 未来、分割 / 配当、手数料)

| risk | handling in R3 |
|---|---|
| 翌営業日 open が存在しない | `paper_pnl = NULL`。理由を dry-run summary に出す |
| h20 close が未来 | `paper_pnl = NULL`。未確定として再実行対象に残す |
| 休場 / 取引停止 | 営業日 calendar と価格存在チェックで吸収。価格欠損なら `NULL` |
| 銘柄分割 / 併合 | 初期は補正なし。gross unadjusted PnL と明記 |
| 配当 | 初期は計上なし |
| 手数料 / 税金 | 初期は計上なし。F286-PNL-R1 と同様 gross PnL |
| ラベル追加 | allowlist 外は `NULL`。warning で検知 |
| F130/F131 sizing 依存 | 取得不能時は 100 株 fallback。ただし注文 helper は作らない |
| 実取引列との混同 | UPDATE 対象を `paper_pnl` のみに限定 |
| production 誤 write | staging-only 三段ガードで抑止 |

## 9. 実装計画 (= W6-1〜W7-1 の sub-task 分割)

| sub | lane | task | output |
|---|---|---|---|
| W6-1 | L1 Design 確定 | 本 W5-4 draft を Architect approve し、`03_design/` へ migrate | approved design doc |
| W6-2 | L3 Impl | `scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py` 実装 | dry-run / write runner |
| W6-3 | L2 Test | `tests/scripts/jobs/test_run_f286_pnl_r3_compute_paper_pnl.py` 追加 | label / price missing / h20 future / write guard tests |
| W6-4 | L4 Audit | paper_pnl 計算ロジック + staging 三段ガード audit | audit note / findings |
| W6-5 | L5 Docs | vault doc + 完了マーカー | `~/fire-vault/02_todo/F286_*.md` update |
| W7-1 | cron | sub-D3 cron 凍結解除後、毎営業日 18:00 JST の再計算 job 化 | launchd / cron design and implementation |

Wave 6 実装時のテスト観点:

| area | required tests |
|---|---|
| label gating | 対象ラベルのみ計算。監視 / 見送りは `NULL` |
| price availability | entry open 欠損、exit close 欠損、未来日は `NULL` |
| qty | F130/F131 sizing、100 株 fallback、最小単元未満 |
| dry-run | DB write なし |
| write guard | `--write` + staging + explicit scope のときだけ UPDATE |
| column safety | `paper_pnl` 以外を更新しない |

## 10. 本線 Architect への確認事項

1. entry price は「`base_date` 翌営業日の open」で確定してよいか。
2. exit price は初期実装では「`base_date + 20 営業日` の close」で確定してよいか。
3. 対象ラベル allowlist は以下でよいか。

```python
{
    "積極的買い推奨",
    "条件付き買い推奨",
    "注意つき買い候補",
    "⚠️ 注意つき買い候補",
}
```

4. sizing は F130/F131 を優先し、stop distance が取れない場合のみ 100 株 fallback でよいか。
5. `paper_pnl = NULL` の理由を DB に保存する列は R3 では追加しない方針でよいか。初期は dry-run summary / log のみとする。
6. market price source は `market_prices_daily` 前提でよいか。実テーブル名や OHLC 列名は W6-2 実装時に既存 schema に合わせる。
7. runner は staging-only write で開始し、production / live advisory への自動 write は Wave 7+ の cron 設計時に再承認でよいか。
8. VIRTUAL_TP / VIRTUAL_SL hit 型の paper_pnl は、R3 初期ではなく将来 R3.1 または別 task に分離してよいか。

