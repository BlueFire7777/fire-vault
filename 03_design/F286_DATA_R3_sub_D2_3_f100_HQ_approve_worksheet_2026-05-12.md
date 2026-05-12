---
id: F286-DATA-R3-sub-D2.3.f100-HQ-approve-worksheet
phase: Wave 11 W11-3 / sub-D2.3.f100 HQ approve worksheet
priority: 高
status: 起票 ☆ 実 staging write は HQ 別 明示承認必須
owner: BlueFire7777 (Fujiwara)
depends_on:
  - W10-5 sub-D2.3.f100 smoke plan (= 元 plan)
  - HQ Wave 11 W11-3 起票承認 (= 2026-05-12、runner 別個別 approve 厳格化)
chapter: F286 DATA / R-01-08
---

# F286-DATA-R3 sub-D2.3.f100 HQ Approve Worksheet (= Wave 11 W11-3)

最終更新: 2026-05-12

## ★ 状態: 起票 (= HQ 別 明示承認必須、他 runner と必ず分離実行)

W10-5 の sub-D2.3.f100 smoke plan を踏まえ、**HQ 個別承認依頼 + 実行前
checklist + 中断条件 + 完了報告 template** を整理。

HQ Wave 11 W11-3 で明示:
- f100 / f101 / f111 / f119 を **個別分離**
- 各 staging write は **個別 HQ 承認**
- **まとめ write 禁止**

## 対象 runner

`scripts/jobs/fetch_historical_market_data.py` (= F100 historical fetch、
W8-4 で --dry-run option 実装済、W8a-fix-2 で external API probe を no-op 化)

## staging write 対象

| table | column | 操作 |
|------|--------|------|
| market_prices_daily | symbol / trade_date / open / high / low / close / volume / 関連 | INSERT |

## 完全 HQ approve template

```
=== HQ 承認依頼 / W12-X-f100 sub-D2.3.f100 staging fetch + write 実行 ===

date:                (実行予定日 YYYY-MM-DD)
parent_task:         F286-DATA-R3-sub-D2.3.f100
runner:              scripts/jobs/fetch_historical_market_data.py
plan source:         [[F286_DATA_R3_sub_D2_3_f100_smoke_plan_2026-05-12]]

予定 fetch range:
  期間:             (例: 2026-05-08 〜 2026-05-08)
  銘柄:             (例: 7203 / 9984 / 6758)
  API 呼び出し:     最大 3 回 (= J-Quants /prices/daily_quotes)

予定 staging write:
  table:            market_prices_daily
  INSERT row 数:    最大 3
  UPDATE row 数:    0
  DELETE row 数:    0

予定 production write: 0 (= mtime unchanged 確認)
予定 develop write:    0 (= mtime unchanged 確認)
予定 LINE 送信:        0
予定 secret 値露出:    0 (= JQUANTS_REFRESH_TOKEN は env、ログ出力なし)
予定 課金:             J-Quants API 課金 (= 微少額、3 銘柄 × 1 営業日)
予定 cron 登録:        0

rollback 計画:
  - smoke 後 row の活用方針 HQ 決定 (= 削除 or W9-1 paper_pnl smoke 再利用)
  - 削除する場合: DELETE FROM market_prices_daily WHERE trade_date='YYYY-MM-DD'
    AND symbol IN ('...', ...)

失敗時対応:
  - sub-step ごとに exit code 検証、失敗時即停止
  - J-Quants 課金中断
  - staging 状態を pre-smoke に戻す

排他確認:
  - F012 FIRE Runner が同時に market_prices_daily に書込んでいないこと
  - cron / OpenClaw 同時 fetch なし

承認希望: 上記 W12-X-f100 staging fetch + write smoke 実行
```

## 実行前 checklist

- [ ] HQ 明示承認受領 (= 上記 template に基づく)
- [ ] W7-2 / W10-5 plan 内容を読み返した
- [ ] pre-smoke 状態キャプチャ手順を /tmp/sub_d2_3_f100/ で実行
- [ ] production / develop / staging DB mtime 記録
- [ ] market_prices_daily 既存 schema 確認
- [ ] J-Quants 認証 env 確認 (= 値は読まず、key 存在のみ確認)
- [ ] 他プロセスが market_prices_daily に touch していないこと
- [ ] 失敗時 rollback 手順を理解

## 中断条件 (= 即停止)

- production / develop DB mtime が変わった
- staging に market_prices_daily 以外への書込
- LINE 送信が発生
- secret 値が stderr / output に露出
- rate limit / 課金超過

## 完了報告 template

```
=== W12-X-f100 完了報告 ===
date:               YYYY-MM-DD HH:MM JST
runner:             fetch_historical_market_data.py
fetch range:        YYYY-MM-DD 〜 YYYY-MM-DD, 銘柄 N 件
INSERT row count:   N
production mtime:   unchanged ✓
develop mtime:      unchanged ✓
staging mtime:      変更 ✓ (= market_prices_daily のみ)
LINE 送信:          0
secret 露出:        0
exit code:          0 (各 step)
known_issues:       (= あれば)
next:               (= rollback or 残置で paper_pnl smoke 再利用)
```

## 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE11_plan|Wave 11 plan]]
- [[F286_DATA_R3_sub_D2_3_f100_smoke_plan_2026-05-12|f100 plan]]
- [[F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|W7-2 元 plan]]
