---
id: F062-R5
phase: P5: 通知 / 第 14 章 LINE 通知配信 / 第 19 章 R-19-08
priority: 最優先
status: 完了 ★ (2026-05-11、Fujiwara 受信確認済み、F062-R5 シリーズ完結)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R5.8 (= action-first 行動判断カード本番送信、最終文面)
  - F062-R5.7 (= action-mode 実装)
  - F286-DATA-R1.7 (= name enrichment)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 25 章 / 第 26 章
---

# F062-R5: First Production Advisory Small Launch (受信確認)

最終更新: 2026-05-11

## ★ 状態: 完了 (Fujiwara 受信確認済み)

F062-R5 First Production Advisory Small Launch は F062-R5 → R5.1 →
R5.2 → R5.3 → R5.4 → R5.5 → R5.6 → R5.7 → R5.8 の漸進的改善を経て、
2026-05-11 18:19 JST に action-first 行動判断カード本番 Advisory を
Fujiwara 個人 LINE app へ 1 通だけ送信、**Fujiwara が LINE app で
受信内容を確認済み**。本ドキュメントは F062-R5 シリーズ全体の
完了マーカー。

本タスクは記録のみで、追加 LINE 送信は一切行わない。

## Fujiwara 受信確認内容

送信時刻: **2026-05-11T09:19:53 UTC (= 18:19 JST、F062-R5.8)**
送信件数: **1 件のみ** (= 重複なし)
chunk_length: 738

Fujiwara が LINE app で確認した文面の構成:

| 項目 | 内容 |
|---|---|
| ヘッダ           | FIRE 本番Advisory |
| Gate 表示        | Data Gate PASS |
| base_date        | 2026-05-09 |
| source / rule    | r2f4_baseline_live_v1 / r2g3_recommended_v2 |
| 今日の結論       | 待ち |
| 今すぐ買い       | 0 件 |
| 条件付き買い     | 0 件 |
| 待ち             | 5 件 |
| 見送り           | 0 件 |
| 注文価格・数量・執行指示 | **なし** ★ (構造的禁止が守られた) |
| Safety footer    | あり (production marker + 自動発注なし 等 8 行) |

## Fujiwara 評価

### Positive ✓

- **「dry-run / LINE 送信なし」表記の解消**
  F062-R5.1 で `message_mode` を `preview` / `production` に分離した
  結果、production marker (= 「本番 LINE 通知 (production send)」)
  に切り替わり、preview marker は構造的に出ない。
- **base_date freshness 問題の解消**
  F062-R5.1 の `payload_freshness_check` で
  `lag_calendar_days=0` (= base_date と gate_signal_max_base_date が
  共に 2026-05-09 で完全一致)。
- **本番 Advisory として受信確認済み**
  F062-R5.8 18:19 JST に **1 通だけ届き**、文面の構造 / 安全
  footer / 結論行と Top 5 表示の整合性をすべて確認。

### 改善候補 (= 次フェーズで対応)

- 「待ち」表現は今後「**まだ買わない / 場中監視候補**」のように、
  より直感的な表現へ改善余地あり。
- 「VWAP 上維持 + 出来高増」の方針は良いが、場中自動監視はまだ
  未実装。**F286-INTRA-R2 (Intraday Advisory Trigger Engine)** で
  対応予定。

## 必須確認項目 (= 全 ✓)

| 項目 | 結果 |
|---|---|
| sent_count                       | **1** ✓ |
| line_api_call_count              | **1** ✓ |
| partial_delivery                 | **False** ✓ |
| token leak                       | 0 ✓ |
| recipient_id leak                | 0 ✓ |
| DATA-R2 gate                     | pass ✓ |
| base_date                        | 2026-05-09 ✓ |
| source_version                   | r2f4_baseline_live_v1 ✓ |
| 注文価格 / 数量 / 執行指示       | 送信していない ✓ |
| 自動発注                          | なし ✓ |
| 楽天証券操作                      | なし ✓ |
| Computer Use                      | なし ✓ |
| TODO Excel                       | 未更新 ✓ |
| --no-verify                      | 不使用 ✓ |
| unrelated modified を stage/commit | しない ✓ |

## F062-R5 シリーズ送信履歴 (= UX 進化の全記録)

| 日時 (JST、2026-05-11) | 内容 | chunk_length |
|---|---|---|
| 22:49 (前日、F062-R4)   | test-message-only             | 234 |
| 01:50 (F062-R5)         | F119 未 wired neutral          | 1,892 |
| 13:01 (F062-R5.2)       | F119 wired compact             | 955 |
| 14:27 (F062-R5.4)       | F119 wired card                | 492 |
| 17:02 (F062-R5.6)       | buyability + names             | 1,120 |
| **18:19 (F062-R5.8)** ★ | **action-first (画面整合)**   | **738** |

F062-R5 sub-task 一覧:

- **F062-R5** : 初版 (neutral message_mode の混在、freshness 課題)
- **F062-R5.1** : message_mode 分離 + freshness guard 実装
- **F062-R5.2** : 初回 production + compact 送信成功
- **F062-R5.3** : card mode 実装 (48.6% 短縮、Codex CRITICAL 2 件対応)
- **F062-R5.4** : card mode 本番送信
- **F062-R5.5** : Practical Buyability Card UX 実装 (判定 / 理由 / 見る点)
- **F062-R5.6** : 銘柄名付き buyability 本番送信 (F286-DATA-R1.7 連携)
- **F062-R5.7** : Action-First Advisory Decision UX 実装 (画面整合)
- **F062-R5.8** : action-first 本番送信 → **Fujiwara 受信確認済み** ★

## 周辺サブタスクとの連携

### DATA 系

- F286-DATA-R1.4: staging restore from F282 bak
- F286-DATA-R1.5: r2f4_baseline_live_v1 / 2026-05-09 / 109 rows を
  staging に書き込み
- F286-DATA-R1.6: F119 historical artifacts wiring
- F286-DATA-R1.7: code → company_name read-only enrichment
  (= F062-R5.6 以降の銘柄名表示の前提)
- DATA-R2 gate: prices / signals / index / derived / other の
  5 階層 freshness gate

### Safety 系

- F062-R3: production send 経路 + max_chunks + partial_delivery
  preservation + token preflight
- F062-R4.1: token ASCII preflight (= U+2028 等のすり抜け防止)
- F062-R4.2: recipient_id masking (= 全 artifact)
- F236-R1: LineBotClient log masking

### Audit

- FIRE-OPS-R0: staging rollback root cause audit (= F282 weekly
  snapshot は by-design)

## token / recipient leak 検査 ✓

検査対象 (= F062-R5 シリーズ全 artifact):
- `/tmp/f062_r5*_{json,txt}`
- `logs/notifications/notifications_line.log`

結果:
- **TOKEN_LEAK: 0** ✓
- **FULL_RECIPIENT: 0** ✓

LINE log の SEND 行は全て masked (= `prefix=U:len=33:hash8=...`)。
Channel token は payload / log / report どこにも平文出力なし。

## DB 不変 ✓

| DB | mtime / size | 不変 |
|---|---|---|
| data/fire.db          | May  7 16:12 / 371 MB | ✓ |
| data/fire.develop.db  | May  7 18:14 / 371 MB | ✓ |
| data/fire.staging.db  | May 11 11:48 / 4.8 GB | ✓ (= F286-DATA-R1.5 末尾維持) |

本タスク (= 受信確認記録) は実 LINE 送信なし / DB write なしのため、
3 DB 全 mtime / size 変化なし。

## 安全要件遵守 (= 全 ✓)

本タスクは記録のみで、新たな実行アクションはなし:

| 項目 | 結果 |
|---|---|
| 追加 LINE 送信                                | 0 通 ✓ |
| DB write                                     | 0 ✓ |
| 自動発注 / 楽天操作 / Computer Use            | なし ✓ |
| 注文価格 / 数量 / 執行指示                    | 送信していない ✓ |
| token / recipient 平文出力                    | 0 ✓ |
| TODO Excel                                   | 未更新 ✓ |
| --no-verify                                  | 不使用 ✓ |
| scripts/seed_pattern_layer1.py               | 未接触 ✓ |
| simulation/research_lane/historical_indicators.py | 未接触 ✓ |
| unrelated modified を stage / commit          | しない ✓ |

## 次タスク提案 (= Fujiwara からの提示順)

1. **F286-PNL-R1 Advisory Decision / Actual PnL Tracking** ★
   Advisory を Fujiwara が見てから手動発注 → 結果反映までの
   「Advisory → 判断 → 実約定 → PnL」をトラッキングする基盤。
   本番 LINE が回り始めた今、PnL 計測が次の主要 KPI。

2. **F286-DATA-R3 daily refresh cron 化**
   F286-DATA-R1.5/R1.6 を毎営業日 06:00 JST 頃に自動再生成。
   F282 weekly snapshot (5/18 07:00 JST) でも生き残る運用へ。

3. **F286-INTRA-R2 Intraday Advisory Trigger Engine**
   「VWAP 上維持 + 出来高増」を場中に自動監視し、待ち候補が条件を
   満たした瞬間に 2 通目の LINE をトリガする。Fujiwara 評価で
   「場中自動監視は未実装」と明示された改善項目。

4. **F286-ORDER-R1 Manual Order Draft Generator**
   Fujiwara が手動発注する際の注文価格 / 数量 / OCO の draft を
   LINE 通知に追加 (= 自動発注ではなく「下書き」として表示)。
   Stage 3 移行 (R-19-08 全項目 PASS 後) の中核機能。

### 並走候補

- F286-DATA-R1.8: F111-R4 persistence に `--listings-db` 追加
  (= 上流で name を埋める根本対応、F286-DATA-R1.7 の downstream
  patch を依存ゼロにする)
- FIRE-OPS-R0 再発防止策案 1: 本番運用データを production
  fire.db に書く運用統一 (= F282 weekly snapshot の影響なくす)
- `03_design/F282_environment_isolation_*.md` の運用ルール明文化

## 完了宣言

F062-R5 First Production Advisory Small Launch は本ドキュメントを
もって **完結**。FIRE は v3.3 要件書 R-19-08 の「本番 LINE 通知 →
人間発注」の Phase 1 (Advisory 配信、注文 draft 未) を実現した。

Stage 3 (Live Advisory) 本格運用に向けた次ステップは F286-PNL-R1
(PnL 計測) と F286-ORDER-R1 (注文 draft 生成) の組合せで、
Fujiwara の判断 → 実約定 → 振り返りループが完成する。
