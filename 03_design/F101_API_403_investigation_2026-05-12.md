---
id: F101-API-403-investigation
phase: Wave 19 W19-1 / F101 API behavior investigation
priority: 高
status: 調査完了 ☆ 修正実装は別 HQ approve
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W18-2 f101 staging write smoke (= HTTP 403 で fail)
  - HQ Wave 19 W19-1 起票承認 (= 2026-05-12)
chapter: F101 / API integration
---

# F101 J-Quants API HTTP 403 Investigation (= W19-1)

最終更新: 2026-05-12

## ★ 状態: 調査完了 (= read-only 静的調査、実 API call なし、修正実装は別 HQ approve)

W18-2 で /fins/announcement HTTP 403 "endpoint does not exist" 発生。
本 Wave で **read-only 静的調査**、実装修正は別 task。

## 1. 観察された事象

W18-2 smoke 実行で:
- runner: `scripts/jobs/fetch_announcements.py`
- endpoint: `https://api.jquants.com/v2/fins/announcement`
- response: HTTP 403
- error message: "The requested endpoint does not exist. Please check
  the URL, HTTP method, and API version: https://jpx-jquants.com/spec/"

## 2. コードベース調査

### base URL

```python
# /Users/bluefire/fire/market_data/client.py:19
JQUANTS_BASE_URL = "https://api.jquants.com/v2"
```

F100 / F101 共通 base URL。

### F100 endpoints (= 動作確認済)

| endpoint | 用途 | F100 動作 |
|----------|------|----------|
| /prices/daily_quotes | 日足株価 | ✓ (W18-1 で動作) |
| /fins/details | 財務 | ✓ (= 推察) |
| /fins/summary | 決算サマリ | ✓ |
| /fins/dividend | 配当 | ✓ |

→ F100 の `/fins/*` 系は V2 で動作。

### F101 endpoint (= 動作不可)

| endpoint | 用途 | F101 動作 |
|----------|------|----------|
| /fins/announcement | 決算発表予定日 | ❌ 403 (W18-2) |

`materials/client.py:138`:
```python
return self._request("GET", "/fins/announcement", params=params)
```

★ **単数形** `announcement` で実装。

### F100 系 `/fins/*` との対比

| endpoint | 単数/複数 | F100/F101 | 動作 |
|----------|-----------|-----------|------|
| /fins/details | 単数 | F100 系 | OK |
| /fins/summary | 単数 | F100 系 | OK |
| /fins/dividend | 単数 | F100 系 | OK |
| /fins/announcement | 単数 | F101 | NG |

→ 単数形なのは共通、F101 だけ 403。

## 3. 仮説

### 仮説 A: endpoint 名違い (= 改名 / 複数形 / 別名)

J-Quants V2 で `/fins/announcement` が削除され、別名 (= `/fins/announcements` 複数形、`/fins/disclosure`、`/news/...` 等) に変更されている可能性。

検証手段 (= 別 task):
- J-Quants V2 spec (= https://jpx-jquants.com/spec/) を確認
- 代替 endpoint name 候補を試行

### 仮説 B: plan 制限

Free / Light plan で `/fins/announcement` 不可、Standard / Premium のみ
可能。

検証手段:
- 現在の J-Quants plan 確認
- plan 別 endpoint 一覧確認

### 仮説 C: deprecated

旧 V1 endpoint で V2 では deprecated、別経路で取得 (= TDnet HTML 直接、
EDINET API、別 API provider 等)。

## 4. 影響範囲

### 直接影響

- F101 fetch_announcements 動作不可
- 既存 staging `announcements` table 1,098 row は **過去取得済 data** (=
  W18 以前に動作していた時期のデータ、本 audit は変更しない)
- F119 / Pattern Research / REPORT-R1 等の announcements 依存処理は
  既存 data のみで動作 (= 新規取得不可)

### 関連 F286 series

- W17-2-fix guard は機能 (= staging-only enforce 通過)
- W17-2-fix が API fail を block しない (= guard は OK、API は別 issue)
- staging / production / develop 全 mtime unchanged (= 安全)

## 5. 推奨対応 (= 別 Wave / 別 HQ approve)

### 短期 (= 次 wave 候補)

A1. J-Quants V2 spec を確認、正しい endpoint 名を確定
   (= 外部 spec page fetch、本 Wave 範囲外)

A2. 候補 endpoint で実 dry-run 試行:
   - `/fins/announcements` (= 複数形)
   - `/fins/disclosure`
   - `/news/announcement`
   - `/fins/earnings_announcement`
   各候補で 1 request、HTTP 200 確認後 1 件取得確認。

A3. J-Quants plan 確認、必要なら upgrade 検討

### 中期 (= 別 wave)

B1. announcements 取得経路の代替実装 (= TDnet HTML 直接 fetch、EDINET API
    連携等)
B2. F101 client 修正 + tests + smoke
B3. cron 連携 (= sub-D3 凍結中)

### 長期

C1. announcements / disclosure API 取得の冗長化 (= 複数 source 統合)

## 6. 本 Wave で実施したこと

- 静的調査のみ (= source code read、API 仕様文書 fetch なし)
- 実 API call 0
- staging / production / develop DB write 0
- LINE 0 / token 0 / cron 0
- 実装修正なし (= 別 HQ approve 後別 task で)

## 7. HQ 判断仰ぐ事項

1. J-Quants V2 spec 確認の許可 (= 外部 URL fetch、別 Wave で実施)
2. plan 確認のタイミング (= 現契約状況 / upgrade 要否)
3. 代替 endpoint 試行の HQ approve (= 実 API call、別 wave)

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE19_results|Wave 19 results]] (= 起票時点)
- [[../07_incidents/F286_DATA_R3_W18_final_audit_2026-05-12|W18-4 audit]]
- W18-2 smoke 結果: /tmp/w18_2/step2_stdout.txt
