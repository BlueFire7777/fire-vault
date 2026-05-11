---
id: F062-R5.7
phase: P5: 通知 / 第 14 章 LINE 通知配信 / 第 19 章 R-19-08
priority: 最優先
status: 完了 ★ (2026-05-11、行動判断カード template + 20 tests)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R5.5 (= Practical Buyability Card UX)
  - F062-R5.6 (= 銘柄名付き本番送信成功)
  - F286-DATA-R1.7 (= name enrichment)
chapter: 第 14 章 / 第 19 章 R-19-08 / 第 25 章 / 第 26 章
---

# F062-R5.7: Action-First Advisory Decision UX

最終更新: 2026-05-11

## ★ 状態: 完了

F062-R5.5/R5.6 buyability mode は「強弱混在・慎重 (boost_with_avoid)
も買い検討にカウント」のため、結論行 (= 「買い検討 30」) と表示
Top 5 (= 全 boost_with_avoid) が乖離していた。仕事中の Fujiwara が
LINE app を見て「今すぐ買い / 待ち / 見送り」を 3 秒で判断できるよう、
boost_with_avoid を「待ち」分類に整理し直す行動判断カードを実装。

## 設計判断

### label → action_bucket の再分類

| label | F062-R5.5 (buyability) | F062-R5.7 (action) |
|---|---|---|
| boost                | 買い検討 | 今すぐ買い (🟢) |
| boost_with_caution   | 買い検討 | 条件付き買い (🟡) |
| boost_with_avoid     | 買い検討 ★ | **待ち** (🟠) ★ |
| caution              | 注意 | 待ち (🟠) |
| avoid                | 見送り | 見送り (🔴) |
| suppress             | 抑制 | 見送り (🔴) |
| neutral              | 参考 | 監視のみ (⚪) |
| missing              | 未付与 | 監視のみ (⚪) |

★ R5.5 では boost_with_avoid を「買い検討候補あり (ただし慎重寄り)」
として表示していたが、R5.7 では「待ち」に分類変更。理由: 表示 Top N
と結論文言の **構造的整合性** を最優先 (= Fujiwara が画面を見て
何をすべきか迷わない)。

### 結論判定 (= Top N selected_counts ベース)

F062-R5.5 の「入力全体ベース判定 + Top N で慎重寄り修飾」というハイブ
リッドは認知負荷が高かった。R5.7 では **結論行も件数も Top N (=
selected_counts)** ベースに統一:

| Top N の bucket 分布 | 結論 |
|---|---|
| buy_now > 0           | 🟢 今日の結論: 買い候補あり |
| conditional > 0       | 🟡 今日の結論: 条件付き買い |
| wait > 0              | 🟠 今日の結論: 待ち |
| avoid > 0             | 🔴 今日の結論: 新規買い見送り |
| それ以外              | ⚪ 今日の結論: 監視のみ |

これで「画面に出ている候補 5 件」と「結論行の文言」が常に整合する。
入力全体の分布は payload metadata に保持 (= 監査用)。

### F119 内部キーの自然言語化

`_humanize_flag` で F119 flag 文字列を自然言語に翻訳:

| 内部 key (snake_case)                       | LINE 表示 |
|---|---|
| `month_of_year`                             | 月の条件 |
| `sector_17`                                 | 業種条件 |
| `interpretation_sector_17_month`            | 業種×月の条件 |
| `top_bucket_interpretation_sector_17`       | 上位rank×業種条件 |
| `material_strength`                         | 材料の強さ |
| `short_horizon_negative`                    | 短期の弱さ |
| `liquidity_low`                             | 流動性の低さ |

未登録 key は last-resort fallback で短縮 key をそのまま表示
(= test では出ない code path)。

### 「買いに変わる条件」(label → flip condition)

| label | 買いに変わる条件 |
|---|---|
| boost                | —(既に買い候補) |
| boost_with_caution   | 寄り後の値動きで強さ確認 |
| boost_with_avoid     | VWAP 上維持 + 出来高増 |
| caution              | 寄り後に崩れず高値更新方向 |
| avoid                | —(原則見送り) |
| neutral              | 新材料の出現を待つ |

## 実装結果

### Step 1: template 実装

`notifications/templates/research_advisory.py` に追加:
- 定数: `ACTION_LABEL_BADGE`, `ACTION_LABEL_VERDICT`,
  `ACTION_FLIP_CONDITION`, `_ACTION_BUCKET`, `_FLAG_HUMANIZATION`
- 関数: `count_action_buckets`, `format_action_conclusion`,
  `format_action_candidate`, `_humanize_flag`,
  `_format_action_reason`
- `build_advisory_line_preview(action_mode=False)` 追加
  (= card_mode / buyability_mode の上位 superset)

### Step 2: runner 統合

`scripts/jobs/run_f062_research_advisory_line_preview.py`:
- `--action-mode` フラグ追加
- payload.metadata.action_mode (= 監査用、独立フラグ)
- `effective_card_mode = card_mode OR buyability_mode OR action_mode`

### Step 3: dry-run smoke (F286-DATA-R1.6 advisory_preview)

```
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/f286_data_r1_6_advisory_preview.json \
  --summary-json /tmp/f286_data_r1_6_advisory_summary.json \
  --gate-json    /tmp/f062_r5_6_gate.json \
  --listings-db  data/fire.staging.db \
  --message-mode production --action-mode --card-top 5 \
  --output-json /tmp/f062_r5_7_line_payload.json \
  --output-text /tmp/f062_r5_7_line_preview.txt
```

| field | value |
|---|---|
| input rows                  | 30 |
| selected                    | 5 (= Top N) |
| chunks                      | 1 |
| chunk[0] length             | **738** ★ (= F062-R5.6 1,120 から 34% 短縮) |
| name_enrichment.enriched    | 30 / 30 |
| forbidden_phrase_count      | 0 |
| safety_footer_present       | True |
| metadata.action_mode        | True |
| metadata.buyability_mode    | False |
| metadata.card_mode (effective) | True |

### Step 4: before / after 文面

**Before (F062-R5.6 buyability、結論と Top 表示が乖離)**:
```
🟠 結論: 買い検討候補あり。ただし慎重寄り
買い検討 30 / 注意 0 / 見送り 0      ← 入力全体 (= 認知負荷)

🟠 強弱混在・慎重 1. 57290 日本精鉱   ← Top 表示は「慎重」
判定: 強弱混在・慎重
理由: F119 boost month_of_year / avoid sector_17 一部該当 ★ 内部 key
見る点: 寄り後の値動き優先 / avoid 影響度
```

**After (F062-R5.7 action、画面整合 + 内部キー除去)**:
```
🟠 今日の結論: 待ち
今すぐ買い: 0件                       ← 画面の Top N と完全一致
条件付き買い: 0件
待ち: 5件
見送り: 0件

🟠 待ち 57290 日本精鉱                ← 結論と表示が一致
判断: 待ち
理由: 月の条件は良いが、業種条件に弱さあり ★ 自然言語
買いに変わる条件: VWAP 上維持 + 出来高増   ★ 次のアクション明示
```

### Step 5: tests (= 20 件追加)

template `TestActionFirstAdvisoryUX` (16 件):
- 結論ルール 4 段階 + ⚪
- 優先順位 (= Top N 整合 vs 入力全体)
- boost_with_avoid が「今すぐ買い」非カウント
- bwa 24 + avoid 6 → Top 5 が avoid 5 で 🔴
- 候補 4 行 (= title / 判断 / 理由 / 買いに変わる条件)
- _humanize_flag テスト (month_of_year / top_bucket_...)
- F119 内部 key 漏洩なし
- production footer 8 行維持
- forbidden phrase 0
- 注文価格 / 数量 / 執行指示が body に出ない
- R5.6 シナリオで「今すぐ買い: 0件 / 待ち: 5件」
- count_action_buckets 単体検証

runner `TestF062R57RunnerActionMode` (3 件):
- --action-mode + --gate-json + --listings-db で結論 + 件数 + 銘柄名
- --gate-json 未指定で Gate 行省略
- --action-mode + --buyability-mode 両指定で action 優先

### Step 6: 回帰

| 範囲 | 結果 |
|---|---|
| 新規 R5.7 tests              | 20 PASS |
| 全 pytest スイート           | **3,331 PASS** (= 3,311 baseline + 20 新規) |

### Step 7: 安全要件 (= 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信                              | 0 ✓ |
| DB write                                  | 0 ✓ |
| token / recipient 平文出力                | 0 ✓ |
| forbidden phrase                          | 0 ✓ |
| 注文価格 / 数量 / 執行指示                | 出さない (構造的) ✓ |
| F119 内部キー (snake_case) の LINE 漏洩    | 0 ✓ |
| 統計詳細 (n=, h20=, win=) の漏洩          | 0 ✓ |
| 自動発注 / 楽天操作 / Computer Use        | なし ✓ |
| 3 DB mtime (production/develop/staging)   | 全 unchanged ✓ |
| TODO Excel                                | 未更新 ✓ |
| --no-verify                              | 不使用 ✓ |
| scripts/seed_pattern_layer1.py            | 未接触 ✓ |
| simulation/.../historical_indicators.py   | 未接触 ✓ |
| unrelated modified を stage / commit      | しない ✓ |

## F062-R5 シリーズ文面 UX 進化

| 版 | 文面 chunk_length | 結論文言 |
|---|---|---|
| F062-R4    | 234   | (test) |
| F062-R5    | 1,892 | neutral 説明 |
| F062-R5.2  | 955   | 「買い検討 30」(分析説明) |
| F062-R5.4  | 492   | 「買い検討 30 / 注意 0 / 見送り 0」(分析説明) |
| F062-R5.6  | 1,120 | 「ただし慎重寄り」(画面と乖離) |
| **F062-R5.7** | **738** ★ | **「今日の結論: 待ち / 今すぐ買い: 0件」(画面整合 + 行動判断)** |

## 次タスク

1. ★ F062-R5.7 完了 (本書、template + runner + 20 tests)
2. HQ (Fujiwara) が次の方向を判断:
   - 案 Y1: F062-R5.8 として `--action-mode --listings-db` で銘柄名
     付き行動判断カードを本番 LINE 送信
   - 案 Y2: F286-PNL-R1 Advisory Decision / Actual PnL Tracking 設計
3. 並走候補:
   - F286-DATA-R1.8: F111-R4 persistence に --listings-db 追加 (=
     上流で name を埋める根本対応)
   - FIRE-OPS-R0 再発防止策案 1 設計 (= production write 統一)
   - F282 weekly snapshot 次回 (2026-05-18 07:00 JST) 前に再送
