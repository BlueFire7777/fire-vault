---
id: FIRE-production-v0-launch-plan
phase: 本番 v0 Launch / 最優先
priority: 最高
status: 設計 v1.0 (= Wave 38 確定、本番 v0 開始までの正本)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - W37 (= AFTER-R1 staging coverage smoke 完了)
  - HQ 本番 v0 最優先方針 (= 2026-05-13)
---

# FIRE 本番 v0 Launch Plan — 設計 v1.0

## 0. 本番 v0 定義 (= HQ 明示)

毎朝 FIRE が自動で:
1. Advisory を生成
2. LINE で銘柄候補を送信
3. 送信内容を DB 記録 (= advisory_decisions)
4. 失敗時は送信しない

**自動発注 / 楽天証券操作は含めない** (= v0+α)。

## 1. v0 必須要素 (= Wave 38 L1a 反映)

| 要素 | 状態 | 関連 wave / module |
|---|---|---|
| DATA 更新 (F100/F101/F111/F119 daily) | 部分完了 | DATA-R3 sub-D2.3.x staging-only guard 完備、launchd 登録未 |
| freshness gate | 既存有 | F286-DATA-R2 / run_data_freshness_gate.py |
| 朝 Advisory 生成 | 既存実装 | F062-R5.7 / F111 |
| LINE 自動送信 | 既存実装 + 三段ガード | F062-R5.8 (dry-run smoke 済) |
| record-decisions 自動記録 | 既存実装 | F286-PNL-R2 |
| F282 snapshot 安全運用 | 試走中 (5/16-5/19) | W25-W34、W33 plist 配置済 |
| ログ + log rotation | 既存 + 設計済 | W32 logrotate install 済 |
| 失敗時停止 | 三段ガード既存 | F062 staging-only guard |
| no-send 試走 | 未実施 | 本番化前必須 |

## 2. 依存グラフ (= Wave 38 L1b 反映)

```
F100 (price) ──┐
F101 (TDnet) ──┤
F111 (signal) ─┤── freshness gate ── F062 (advisory) ── LINE send ── record-decisions
F119 (eval) ───┘                                                          │
                                                                          ↓
                                                              advisory_decisions DB
F282 weekly snapshot (= 別系統、production→staging/develop、土曜 02:00)
```

### break 検出 + abort

- F100 fetch 失敗 → freshness gate trigger → F062 abort
- LINE token 不在 → F062 advisory 生成成功でも LINE send block
- F282 試走 NO-GO → 別系統で修正、v0 影響なし

### cron / launchd schedule (= 想定)

| 時間帯 (JST) | 用途 |
|---|---|
| 朝 06:00-08:00 | F100/F101 daily refresh |
| 朝 08:00 | F111 signal extraction |
| 朝 08:30 | F119 evaluation |
| **朝 08:45** | **freshness gate → F062 → LINE send → record-decisions** |
| 夕 16:30-19:00 | daily refresh 遅れ補正 |
| 夜 23:00 | AFTER-R1 (= v0 後) |
| 土曜 02:00 | F282 weekly snapshot |

## 3. no-send 試走計画 (= Wave 38 L2a 反映)

### 目的

本番 token 未投入で advisory 生成 + record-decisions 動作を 1 週間検証。
LINE 実送信は **行わない**。

### 期間

2026-05-16 〜 2026-05-22 (= 1 週間、5 営業日)。
F282 試走 (= 5/16-5/19) と並走。

### daily 実行内容

- 朝 08:45 cron / launchd 起動 (= no-send mode)
- F062 advisory 生成 → record-decisions snapshot
- LINE template render (= dry-run、実送信 0)
- log 確認 (= morning_advisory.log)

### 確認項目

- advisory 生成 exit 0
- record-decisions 件数 increment
- LINE template 1 chunk 制限 OK
- failure mode (= F100 fetch 失敗) で advisory 停止
- log 出力 OK

### abort 条件

- 1 回でも LINE 実送信痕跡 → 即時 abort + token rotate
- record-decisions 重複 / 矛盾 → abort + 調査
- production DB 意図しない write → abort
- F282 試走干渉 → abort

### GO/NO-GO 判定 (= 5/22 後)

- GO: 5 営業日 advisory 生成成功 + LINE 0 + 異常 0 → 本番 v0 D-Day 設定
- NO-GO: 任意失敗 → 修正 + 再試走

## 4. v0 safety check (= Wave 38 L2b 反映)

### v0 開始前 必須 check (= 全 PASS 必須)

[DB safety]
- production fire.db mtime 過去 24h 以内
- advisory_decisions schema 完全 (= W14 適用済)
- record-decisions 重複防止 (= UNIQUE)
- 三段ガード動作

[LINE safety]
- token は production 専用 (= staging と分離)
- 1 chunk 制限 (= F062-R5.8)
- dry-run option 動作確認
- send_guard helper 経由
- 失敗時 retry policy

[freshness gate]
- F286-DATA-R2 動作
- threshold 明示 (= 24h?)
- gate 通過/阻止 log

[F282 干渉防止]
- F282 試走完了 (= 5/19 GO)
- 朝 Advisory (= 08:45) と F282 (= 土 02:00) 時間重複 0
- 同 source read のみで干渉 0

[failure mode]
- 各 step 失敗で Advisory abort
- 全 abort で log 残し

### test 項目

- test_morning_advisory_with_stale_data
- test_morning_advisory_with_no_line_token
- test_record_decisions_duplicate_prevention
- test_f282_concurrent_safety

### D-Day -1 check

launchctl list / .env / token / schema / log dir / disk free

### D-Day check (= 本番初日 08:46 以降)

- LINE 着信 (= Fujiwara)
- record-decisions 1 行追加
- log OK
- F282 干渉なし

### NO-GO trigger (= D-Day)

- LINE 不達
- record-decisions 不在
- production DB 不整合
- 多 channel 重複送信

## 5. 最短 Wave 順 (= Wave 38 L3 反映)

### Phase A: Foundation 完了 (= 5/19 まで)

- **Wave 38 (= 本 wave)**: v0 Launch Plan 起票
- F282 試走 5/16-5/19 (= 別 wave 不要、Fujiwara 観察)
- 5/19: F282 GO/NO-GO 判定

### Phase B: launchd 登録 (= 5/19-5/26)

- Wave 39 候補: F282 試走 GO + 本番化承認
- Wave 40 候補: F100 daily refresh launchd 設計
- Wave 41 候補: F100 launchd 実配置 (= HQ_APPROVE_LAUNCHD_DAILY)
- Wave 42 候補: F101 / F111 / F119 launchd 順次登録

### Phase C: Morning Advisory launchd (= 5/26-6/2)

- Wave 43 候補: F062 morning advisory launchd 設計
- Wave 44 候補: freshness gate 統合確認
- Wave 45 候補: F062 launchd 実配置 + no-send mode (=
  HQ_APPROVE_LAUNCHD_MORNING_ADVISORY)

### Phase D: no-send 試走 (= 6/2-6/9、1 週間)

- Wave 46 候補: no-send 試走開始 (= HQ_APPROVE_NO_SEND_TRIAL)
- Wave 47-50 候補: daily check / 異常検知 / log 確認
- Wave 51 候補: 試走 GO/NO-GO 判定

### Phase E: LINE token 投入 + 本番開始 (= 6/9 D-Day)

- Wave 52 候補: production LINE token 投入 (= HQ_APPROVE_LINE_TOKEN_PRODUCTION)
- **Wave 53 候補: D-Day 朝 advisory 実送信 1 回目**
- Wave 54+ 候補: daily monitoring + 異常時対応

### D-Day: **2026-06-09 (火曜) 確定** ※ W44.5-pre で曜日確定 / W44.5-post で再確認

- 5/13 W38 (= 本日)
- 5/16 F282 試走実行
- 5/19 F282 GO 判定
- 5/19-5/26 launchd 登録
- 5/26-6/2 morning advisory launchd
- 6/2-6/9 no-send 試走 (= 1 週間)
- **6/9 D-Day 本番開始**

### 並走可能 task

- F101 API 修正 (= 並走 OK、v0 必須ではない)
- R2 v1.3 改訂
- AFTER-R1 本格実装 (= v0 後)

### HQ approve marker 順序 ※ W40.5 / W40.6 / W44.5-pre で 7 段固定、W44.5-post で確認

1. F282 GO (= W39、自然遷移)
2. HQ_APPROVE_LAUNCHD_DAILY (= W41 開始、DATA-R3 plist 配置)
3. DATA-R3 no-write 試走 GO (= W41 完了、自然遷移)
4. HQ_APPROVE_LAUNCHD_MORNING_ADVISORY (= W45 開始、F062 plist 配置)
5. F062 no-send trial GO (= W45 完了、自然遷移)
6. HQ_APPROVE_LINE_TOKEN_PRODUCTION (= W52、token 投入許可)
7. HQ_APPROVE_PRODUCTION_V0_LAUNCH (= W53、production send 開始許可)
補. HQ_APPROVE_PRODUCTION_V0_RECOVERY (= rollback 後再起動)

> ※ 旧版 (Wave 38) では `HQ_APPROVE_NO_SEND_TRIAL` を独立 marker としていたが、
> W40.5 §10 で自然遷移 (= marker 5) に統合する方針が確定。本表は最新版。

### リスク

- F282 試走 NO-GO → +2-3 週間
- F100 launchd 設計問題 → +1 週間
- no-send 試走で異常 → +1 週間
- LINE token 問題 → +2 日

### ベスト / ワースト

- ベスト: 5/13 → 6/9 D-Day (= 27 日)
- ワースト: 5/13 → 7/初旬 D-Day (= 50 日)

## 6. v0 後の拡張 (= 後回し、HQ 明示)

- AFTER-R1 本格実装 (= 夜間 Paper Live)
- ML / feature engineering
- Dashboard (= DASH-R1)
- ORDER-R1 (= 自動発注)
- RISK-R1 (= drawdown 監視)
- INTRA-R1 (= 場中監視)
- Lane 自動昇格 (= LANE-R2)
- 新規銘柄探索

## 7. 安全要件 (= 本 doc 自体)

| 項目 | 結果 |
|---|---|
| 実 DB write | 0 (= 本 doc 設計のみ) |
| 実 LINE 送信 | 0 |
| token / channel_token / secret 参照 | 0 |
| cron / launchd / crontab 登録変更 | 0 |
| F282 試走干渉 | 0 |
| F101 staging probe | 未実行 |
| 楽天 / 自動発注 / Computer Use | なし |
| workflow / --no-verify / TODO Excel | 0 |
| Codex 直接 commit | 0 |

## 8. 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE38_plan|Wave 38 plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE37_results|Wave 37 results (= AFTER-R1 staging smoke)]]
- [[F282_weekly_snapshot_trial_monitoring_2026-05-12|F282 試走監視]]
- [[F286_AFTER_R1_night_paper_live_batch_2026-05-12|F286-AFTER-R1 設計 v1.0 (= v0 後拡張)]]
- /tmp/codex_wave38/output/* (= 7 lane stdout)
