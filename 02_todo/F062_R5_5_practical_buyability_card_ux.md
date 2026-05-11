---
id: F062-R5.5
phase: P5: 通知 / 第 14 章 LINE 通知配信 / 第 19 章 R-19-08
priority: 最優先
status: 完了 ★ (2026-05-11、実用判断カード UX 実装 + 18 tests)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R5.3 (= card mode 実装 + Codex CRITICAL 2 件対応)
  - F062-R5.4 (= card mode 本番送信 chunk_length=492 達成)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 26 章
---

# F062-R5.5: Practical Buyability Card UX (= 実用判断カード)

最終更新: 2026-05-11

## ★ 状態: 完了

F062-R5.3 で実装した card mode (chunk_length 491、48.6% 短縮) は
情報量が少なすぎ、本業中の Fujiwara が「次に何を確認すべきか」を
3 秒で判断するには不足だった。本タスクで card mode の superset として
**buyability_mode** を追加:

- 冒頭結論 3 行 (= 入力全体ベースで買い検討/見送り判定 + 表示
  Top N ベースで「慎重寄り」判定 → 画面と結論文言を整合)
- 各候補 4 行 (= 銘柄コード+名 / 判定 / 理由 / 見る点)
- 注文価格 / 数量 / 執行指示は構造的に出さず、Fujiwara が手動で
  確認すべき観察対象 (= 寄り後 VWAP / 出来高 等) のみ示す
- 自動発注なし / 手動レビュー必須 footer 維持

実 LINE 送信なし / DB write なし / token-recipient 平文出力なし。
回帰 3,288 PASS。

## 実施結果

### Step 1: template 実装

`notifications/templates/research_advisory.py` に以下を追加:

| 関数 / 定数 | 役割 |
|---|---|
| `BUYABILITY_LABEL_VERDICT` | label → 短縮判定テキスト (= 「判定: ...」行) |
| `BUYABILITY_LABEL_FOCUS` | label → 手動確認ポイント (= 「見る点: ...」行) |
| `_short_flag(flag)` | F119 flag 詳細統計 (`n=`, `h20=`, `win=`) を payload JSON 側に残し文面には key のみ |
| `_format_buyability_reason(row)` | 理由行を F119 boost/avoid/caution flags + sector × month で生成 |
| `format_buyability_conclusion(input_counts, selected_counts)` | 冒頭結論 3 行。入力全体で「買い検討/見送り/該当なし」、Top N で「慎重寄り」判定 |
| `format_buyability_overview(input_counts)` | 1 行 summary 「買い検討 X / 注意 Y / 見送り Z」(入力全体ベース) |
| `format_buyability_candidate(row, index=...)` | 4 行 candidate (= 番号付き title / 判定 / 理由 / 見る点) |
| `build_advisory_line_preview(buyability_mode=...)` | buyability_mode=True で card_mode=True 扱い + 実用判断カード文面を生成 |

### Step 2: runner 実装

`scripts/jobs/run_f062_research_advisory_line_preview.py` に追加:

- `--buyability-mode` フラグ (= 実用判断カード mode を有効化)
- payload metadata に `buyability_mode` を別途保持
- `effective_card_mode = args.card_mode or args.buyability_mode`

### Step 3: 結論判定ロジック (Codex CRITICAL F062-R5.3 #2 互換)

| 入力分布 | 表示 Top N (counts) | 結論 |
|---|---|---|
| boost 系 > 0 | sel_pure_boost ≥ sel_bwa | 🟢 結論: 買い検討候補あり |
| boost 系 > 0 | sel_bwa ≥ sel_pure_boost (sel_bwa > 0) | 🟠 結論: 買い検討候補あり。ただし慎重寄り |
| avoid 系のみ | (n/a) | 🔴 結論: 新規買い検討候補なし |
| neutral/missing のみ | (n/a) | ⚪ 結論: 該当候補なし |

**重要**: 「買い検討/見送り/該当なし」の根本判定は **入力全体**
(= Codex CRITICAL F062-R5.3 #2 で確定した原則) を維持。一方
「慎重寄り」修飾は **表示 Top N** (= sel_bwa, sel_pure_boost)
で判定し、Fujiwara が実際に LINE 画面で見る候補と結論文言を
整合させる。

### Step 4: dry-run smoke (F286-DATA-R1.6 の 30 件 advisory_preview)

```
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/f286_data_r1_6_advisory_preview.json \
  --summary-json /tmp/f286_data_r1_6_advisory_summary.json \
  --gate-json /tmp/f062_r5_4_gate.json \
  --message-mode production --buyability-mode --card-top 5 \
  --output-json /tmp/f062_r5_5_line_payload.json \
  --output-text /tmp/f062_r5_5_line_preview.txt
```

| field | value |
|---|---|
| input rows                     | 30 |
| selected                       | 5 |
| chunks                         | 1 |
| chunk[0] length                | 1,086 |
| buyability_mode                | True ✓ |
| card_mode (metadata)           | True ✓ |
| forbidden_phrase_count         | 0 |
| safety_footer_present          | True |
| selected_label_counts          | boost_with_avoid: 5 |

文面冒頭 (= 実出力):
```
FIRE 本番Advisory
Data Gate PASS
base_date: 2026-05-09
source: r2f4_baseline_live_v1 / r2g3_recommended_v2

🟠 結論: 買い検討候補あり。ただし慎重寄り
boost はあるが avoid 条件も混在。
寄り後の値動きで判断。

買い検討 30 / 注意 0 / 見送り 0

🟠 強弱混在・慎重 1. 57290
判定: 強弱混在・慎重
理由: F119 boost month_of_year / avoid sector_17 一部該当
見る点: 寄り後の値動き優先 / avoid 影響度

...(Top 5、各 4 行)

Safety
- 本番 LINE 通知 (production send)
- 自動発注なし
- 楽天操作なし
- Computer Use なし
- 注文価格・数量・執行指示は生成しない
- F119 の優位は 20d 寄り (daytrade 即時判断には使わない)
- Fujiwara manual review required
```

★ 結論行: 入力 30 件中 6 件が boost_with_avoid で表示 Top 5 が
全て boost_with_avoid のため、「ただし慎重寄り」が出る。F062-R5.4
では結論が「🟢 買い検討候補あり」のままで Top 5 表示と乖離していた
が、F062-R5.5 で整合した。

### Step 5: tests (18 件)

template (`TestBuyabilityCardLineUX`、15 件):
- `test_buyability_boost_only_says_buy`
- `test_buyability_boost_with_avoid_majority_says_cautious`
- `test_buyability_pure_boost_with_avoid_only_says_cautious`
- `test_buyability_avoid_only_says_no_buy`
- `test_buyability_neutral_only_says_no_candidate`
- `test_buyability_candidate_shows_name` (= 銘柄名表示)
- `test_buyability_candidate_has_verdict_reason_focus` (= 4 行)
- `test_buyability_reason_uses_sector_month_for_neutral`
- `test_buyability_shorter_than_compact_realistic` (= 30 件入力)
- `test_buyability_within_card_mode_overhead`
- `test_buyability_safety_footer_full_production`
- `test_buyability_forbidden_phrase_zero`
- `test_buyability_no_order_price_or_quantity_in_chunks` (= body から
  「発注/注文/成行/指値/OCO/100株/stop_loss/TP=/SL=」等を排除、
  footer 行は除外して検査)
- `test_buyability_boost_total_summary`
- `test_buyability_card_top_zero_refused`

runner (`TestF062R55RunnerBuyabilityMode`、3 件):
- `test_buyability_runner_payload_metadata`
- `test_buyability_runner_no_gate_omits_gate_line`
- `test_buyability_runner_top_3`

### Step 6: 回帰

| 範囲 | 結果 |
|---|---|
| 新規 buyability tests          | 18 PASS |
| F062 関連 (template + runner + sender + smoke) | 237 PASS |
| **全 pytest スイート**         | **3,288 PASS** (= 3,270 baseline + 18 新規) |

### Step 7: 安全要件 (= 全 ✅)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信                              | 0 ✅ |
| DB write                                  | 0 ✅ |
| token / recipient 平文出力                | 0 ✅ |
| 注文価格 / 数量 / 執行指示の生成          | しない (構造的) ✅ |
| 自動発注 / 楽天操作 / Computer Use        | なし ✅ |
| forbidden phrase                          | 0 ✅ |
| safety_footer_present                     | True ✅ |
| manual_review_required (= input 全件)     | 30 / 30 ✅ |
| auto_order_allowed=True                   | 0 / 30 ✅ |
| 3 DB mtime (production/develop/staging)   | 全 unchanged ✅ |
| TODO Excel                                | 未更新 ✅ |
| --no-verify                              | 不使用 ✅ |
| scripts/seed_pattern_layer1.py            | 未接触 ✅ |
| simulation/.../historical_indicators.py   | 未接触 ✅ |
| unrelated modified を stage / commit      | しない ✅ |

## 文面 UX の進化 (= F062-R5 シリーズ)

| 版 | 日時 (JST) | 内容 | chunk_length |
|---|---|---|---|
| F062-R4    | 2026-05-10 22:49 | test-message-only          | 234 |
| F062-R5    | 2026-05-11 01:50 | F119 未 wired neutral      | 1,892 |
| F062-R5.2  | 2026-05-11 13:01 | F119 wired compact         | 955 |
| F062-R5.4  | 2026-05-11 14:27 | F119 wired card            | 492 |
| **F062-R5.5** | (dry-run smoke) | **buyability (= 実用判断カード)** | **1,086** |

F062-R5.5 は card mode の 491 chars より長いが、これは判断に必要な
情報密度を上げた結果。F062-R5.2 compact (955) と同等の長さで:
- 銘柄名表示
- 「慎重寄り」判定が画面と整合
- 「見る点」(= 寄り後 VWAP / 出来高) で次のアクションが明確

## 次タスク

1. ★ F062-R5.5 完了 (本書、template / runner / tests / docs)
2. HQ (Fujiwara) が card mode と buyability mode の使い分けを判断:
   - 案 Y1: F062-R5.6 として --buyability-mode で 1 chunk 本番送信
   - 案 Y2: F286-PNL-R1 Advisory Decision / Actual PnL Tracking 設計
3. 並走候補:
   - 銘柄名 (name 列) を F286-DATA-R1.6 の F111-R4 wiring 段階で
     埋める (= 現在 advisory_preview JSON で name="" のため smoke
     文面に銘柄名が出ていない。template は実装済だが入力データが
     name 欠落)
   - FIRE-OPS-R0 再発防止策案 1 設計 (= production write 統一)
   - F282 weekly snapshot 次回 (2026-05-18 07:00 JST) 前に再送 or
     再発防止策案 1 実装
