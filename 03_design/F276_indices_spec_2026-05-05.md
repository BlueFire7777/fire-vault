# F276 Step 1c-1: J-Quants V2 indices spec 確認 + 案 X-1 採用記録

**作成日**: 2026-05-05
**起点**: F276 Phase 2-B (commit 8e097b9) 完了 → Step 1 試行 fetch で
endpoint not exist 検出 → 本部 Q-step-1c=1c-1 確定 (公式 spec 確認 +
正しい path 再試行)
**本 step commit**: ~/fire dev = 7da77bb (Step 1c-1-fix、案 X-1 採用)
**前 commit**: ~/fire dev = 8e097b9 (Phase 2-B、誤 path)

---

## 1. 概要

F276 Phase 2-B 試行運用 (Step 1) で Mac mini 推測 path 3 種すべて 403
"endpoint does not exist" 確認 → WebFetch で J-Quants V2 公式 spec 取得 →
正しい endpoint path + N225 取得不可確定 → 本部 Q-step-1c-1-fix=案 X-1
(TOPIX のみ MVP) 確定 → 実装修正 + 1 day 試行 fetch 成功 (TOPIX 取得確認)。

---

## 2. J-Quants V2 公式 spec (確定、原文ママ引用)

### 2-1. /v2/indices/bars/daily endpoint 仕様

WebFetch source:
- https://jpx-jquants.com/spec/ (2 ページ目で /indices/bars/daily/topix
  記述、最終的に /indices/bars/daily に統合と確認)
- https://jpx-jquants.com/ja/spec/idx-bars-daily/indexcodes (利用可能
  index codes 一覧)
- https://jpx-jquants.com/ja/spec/idx-bars-daily (詳細仕様)

```
URL:        GET https://api.jquants.com/v2/indices/bars/daily
HTTP Method: GET
Authentication: x-api-key header (~/fire/.env JQUANTS_API_KEY)

Request parameters:
  code         string  (one of code/date 必須): 例 "0000" = TOPIX
  date         string  (one of code/date 必須): "20260502" or "2026-05-02"
  from         string  (optional): 範囲開始
  to           string  (optional): 範囲終了
  pagination_key string (optional): pagination

Response JSON:
  top-level keys: data (array) + pagination_key
  data row fields:
    Date  (string YYYY-MM-DD)
    Code  (string)
    O / H / L / C  (number、F100 /equities/bars/daily 同型短縮形)
  ⚠ 一部指数は O/H/L が null、C のみ (close-only 指数)
  ⚠ Volume field は /indices/bars/daily に存在しない (個別株 endpoint と相違)

Example:
  GET /v2/indices/bars/daily?code=0028&date=20231201
  Header: x-api-key: {API_KEY}

Response:
  {
    "data": [{
      "Date": "2023-12-01", "Code": "0028",
      "O": 1199.18, "H": 1202.58, "L": 1195.01, "C": 1200.17
    }],
    "pagination_key": "value1.value2."
  }
```

### 2-2. 利用可能 index codes (Standard プラン取得可)

| Code | 指数名 | データ範囲 | プラン |
|---|---|---|---|
| 0000 | **TOPIX** | 2008/5/7 〜 | Standard |
| 0028 | TOPIX Core30 | (本 MVP 未活用) | Standard |
| 002A | TOPIX 100 | 同上 | Standard |
| 002B | TOPIX Mid400 | 同上 | Standard |
| 0070 | 東証グロース市場 250 指数 (旧マザーズ) | 2008/5/7 〜 | Standard |
| 0500 | 東証プライム市場指数 | 2022/6/27 〜 | Standard |
| 0501 | 東証スタンダード市場指数 | 同上 | Standard |
| 0502 | 東証グロース市場指数 | 同上 | Standard |
| 0503 | JPX プライム 150 指数 | 2023/5/29 〜 | Standard |
| 0080-0090 | TOPIX セクター指数 17 種 | (本 MVP 未活用) | Standard |
| B507 | JPX 日経 400 (配当込) | 2013/11/18 〜 | **Premium** |

★ プレフィックス "6" / "B" は Premium 専用。

---

## 3. ★ 日経 225 (N225) 取得不可確定

J-Quants V2 indexcodes に **日経 225 (N225 / 日経平均)** は存在しない。

検出経路:
- WebFetch /ja/spec/idx-bars-daily/indexcodes 結果: N225 系コード一切なし
- 関連 endpoint: `/derivatives/bars/daily/options/225` (オプション専用、
  原資産価格ではない)

理由 (推定):
- J-Quants V2 (旧 JPX、東証傘下) は東証指数 (TOPIX 系) のみを公式提供
- 日経 225 (日経平均) は日本経済新聞社の指数で、別ライセンス
- J-Quants が「東証指数のみ提供」のライセンス境界を堅持

→ 本部 G-4 承認 (TOPIX + N225 MVP) 前提崩れ → Q-step-1c-1-fix 発動。

---

## 4. 本部 Q-step-1c-1-fix 確定 = 案 X-1 採用

### 4-1. 採用案

★ 案 X-1: TOPIX のみ MVP、TOPIX で market 方向性代替 (classify_market_regime
の topix_change_pct fallback)

採用根拠 (Mac mini 推奨 + 本部承認、3 行):
- R1. J-Quants V2 で N225 取得不可は仕様確定事実 (本ファイル §3 確認)
- R2. classify_market_regime() を TOPIX 代替判定に修正で機能損失軽微
      (TOPIX = 東証全銘柄時価総額加重、N225 と相関高で実運用上代替可)
- R3. 案 X-2 (別データソース) は F103 候補として将来分離が筋

### 4-2. 棄却案

| 案 | 内容 | 棄却根拠 |
|---|---|---|
| X-2 | 別データソースで N225 (TradingView MCP / Yahoo Finance / 別 API) | scope 大幅拡大 (1.0-2.0 日)、F103 候補として分離 |
| X-3 | options /derivatives/bars/daily/options/225 で N225 spot 逆算 | 精度劣る、実用性低 |

---

## 5. 実装修正 (commit 7da77bb)

### 5-1. 修正ファイル一覧

| ファイル | 修正内容 |
|---|---|
| `market_data/client.py` | get_indices_topix + get_indices 統合 → get_indices_daily(code, ...)、確定 path /v2/indices/bars/daily |
| `market_data/index_fetcher.py` | DEFAULT_INDEX_SYMBOLS = ("TOPIX",)、SYMBOL_TO_JQUANTS_CODE = {"TOPIX": "0000"}、_normalize_indices_row O/H/L/C 短縮形対応、close-only 指数対応、★Codex 補強で空レスポンス / schema 不一致を error 扱い (no_rows_fetched) |
| `features/regime.py` | classify_market_regime() に TOPIX fallback (nikkei_change_pct=None なら topix_change_pct)、build_snapshot_from_db で code '0000' を nikkei 扱いから除外 |
| tests 3 ファイル | 40 PASS (Phase 2-B 31 → 40、+9) |

### 5-2. 主要設計変更点

```
旧 (commit 8e097b9):
  market_data/client.py
    get_indices_topix → /indices/topix (★endpoint 不在)
    get_indices → /indices (★endpoint 不在)
  market_data/index_fetcher.py
    DEFAULT_INDEX_SYMBOLS = ("TOPIX", "N225")
    Open/High/Low/Close フル形想定 (★schema 相違)
  features/regime.py
    build_snapshot_from_db: symbol IN ('N225', '0000') (★code '0000' は TOPIX)

新 (commit 7da77bb):
  market_data/client.py
    get_indices_daily(code, date, from, to) → /v2/indices/bars/daily
  market_data/index_fetcher.py
    DEFAULT_INDEX_SYMBOLS = ("TOPIX",)
    SYMBOL_TO_JQUANTS_CODE = {"TOPIX": "0000"} (人間可読 → J-Quants code)
    O/H/L/C 短縮形 (公式 spec 準拠)
    Codex 補強: 空レスポンス / 全行 None で status="error" + no_rows_fetched
  features/regime.py
    classify_market_regime: TOPIX fallback (topix_direction で代替判定)
    build_snapshot_from_db: symbol = 'N225' のみ (code '0000' は TOPIX 扱い)
```

### 5-3. Codex pre-commit 履歴

連鎖 1 件目 (本 step):
- Codex 検出: fetch_indices() で API レスポンス空 / 全行 None でも status="ok"
- 修正: errors 空 + all_rows 空 → status="error" + no_rows_fetched 記録
- 修正後: Codex pre-commit 通過 (commit 7da77bb)

---

## 6. 1 day 試行 fetch 結果 (Step b、案 X-1 検証)

### 6-1. 実行コマンド

```bash
cd ~/fire && .venv/bin/python -m market_data.index_fetcher \
    --date-from 2026-05-01 --date-to 2026-05-02 --json
```

### 6-2. 結果 (★ ケース b-A、status="ok"、INSERT 成功)

```json
{
  "status": "ok",
  "date_from": "2026-05-01",
  "date_to": "2026-05-02",
  "symbols": ["TOPIX"],
  "fetched_per_symbol": {"TOPIX": 1},
  "inserted_total": 1,
  "errors": []
}
```

### 6-3. index_data DB 確認 (production fire.db)

```
date        | symbol | open    | high    | low     | close   | change_pct
2026-05-01  | TOPIX  | 3718.74 | 3741.19 | 3691.63 | 3728.73 | None (1 行のみ)
```

→ J-Quants V2 /v2/indices/bars/daily?code=0000 が Standard プランで完全動作確認。

### 6-4. Step 2 (60 営業日本格 fetch) 着手準備

本部承認待機 → 着手:
```bash
cd ~/fire && .venv/bin/python -m market_data.index_fetcher \
    --date-from 2026-02-05 --date-to 2026-05-02 --json
```

期待: TOPIX × 60 営業日 = 60 行 INSERT、所要 5-15 分

---

## 7. F271 v1.3 候補追加事例 (Mac mini 側ミス記録)

### 7-1. §6-7 適用違反 (Vault「未確定マーカー」放置)

F276_phase2_design line 161:
> 日経 225: `/indices/quotes` (code=N225 等、J-Quants 仕様確認後確定)

「J-Quants 仕様確認後確定」と Mac mini が Vault に書いたにもかかわらず、
Phase 2-B 実装時に確認をスキップ → 試行運用で endpoint 不在発覚。

### 7-2. §6-22 候補事例追加 (Vault 参照漏れ)

F100 todo line 127「公式 spec: https://jpx-jquants.com/spec/」を Phase 1
inventory 観点 v3 §2-1 で参照しなかった。spec の確認義務が Vault にあった
にもかかわらず実装時にスキップ。

### 7-3. §6-23 候補事例 (本部承認時の前提検証)

本部 G-4 承認 (TOPIX + N225 MVP) は Mac mini Phase 1 handover の「N225
取得可能」前提を引用して承認、Mac mini が前提検証していなかった = 本部承認時
に Mac mini が「Vault 未確定マーカー解消済」と暗黙的に主張した形 (実態は未解消)。

### 7-4. §6-24 候補事例 (実装内容 grep 照合)

F276 Phase 1 handover line 110「market_data/client.py | J-Quants V2 client
(aiohttp)」と Mac mini が記載 (実態 requests 同期実装、誤)。実装内容
(同期 / 非同期 / 例外パターン) を Vault 一次資料に記載する際は実コード
grep で照合する規律を §6-24 で確立。

→ F271 v1.3 改訂で §6-7-bis / §6-22 / §6-23 / §6-24 を一括採用候補。

---

## 8. 関連リンク

- F276 Phase 2 design (§B / §E、line 161 更新予定): [[F276_phase2_design_2026-05-05]]
- F276 Phase 2-A: [[F276_phase2a_2026-05-05]] (commit 08e42f4)
- F276 Phase 2-B: [[F276_phase2b_2026-05-05]] (commit 525046a、誤 path 含む)
- F040 design (D''' 候補 3 = F104 待ち): [[F040_backtest_status_2026-05-04]]
- F100 todo: [[../02_todo/F100_市場データAPI]]
- F104 todo: [[../02_todo/F104_指数四本値取得]]
- 要件書 R-14-02: [[../01_requirements/FIRE_要件書_第14章_データソース優先順位と縮退運転]]
- F271 v1.2: [[F271_v1.2]]

---

## 9. 次フェーズ

本部承認待機 → Step 2 (60 営業日本格 fetch、TOPIX × 60 営業日) 着手 →
Run a 1 day 実測 → events_total ベンチマーク取得 → ケース 4-A/4-B 判定 →
Phase 2-C 着手判断。
