---
id: FIRE-manual-live-pilot-operation-plan
phase: 本番 v0 中核 / Manual Live Pilot
priority: 高
status: design v1.0
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 03_design/F286_AFTER_R1_paper_live_mvp_2026-05-14.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_IMPL_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_POST_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_6_FIX_results.md
  - 04_daily/template_manual_live_pilot_trade_plan.md
  - 04_daily/template_manual_live_pilot_review.md
---

# FIRE Small Manual Live Pilot Operation Plan v1.0

## §1 目的

FIRE 本番 v0 中核として、AFTER-R1 Paper Live MVP が生成する
`reports/after_r1/` 配下 4 種類成果物 (= paper_live_ledger /
good_candidate_ranking / pattern_candidate_report / morning_line_material) を
**藤原さんの少額手動実弾** に接続し、

「良銘柄候補生成 → 手動少額実弾 → 結果記録 → 翌日改善」

という**再現性優位の学習ループ** を開始する。

本 doc は **運用ルール / リスク上限 / 記録テンプレ / 緊急停止条件** を正式化する。
**FIRE は候補と判断材料を出すだけ。実発注は藤原さんが iSPEED / 楽天 Web で手動。
自動発注 / Computer Use / Playwright は採用しない。**

## §2 第 22 章 / 崩してはならない前提との整合

CLAUDE.md「崩してはならない前提」より:
1. 発注は人間 (Fujiwara) が手動実行 — Computer Use 不採用
2. 強制クローズは LINE 緊急アラート → 人間対応 — Playwright 不採用
3. 建玉の正本は楽天証券 — FIRE は整合チェックのみ
4. Evaluation Agent の提案は承認制 — 自動反映禁止
5. Stage 飛ばし禁止

本 pilot は **Stage 2 Paper Live と Stage 3 Live Advisory の中間** として位置付け:
- FIRE の出力 (= 候補) を **藤原さん本人が judgement** する
- F062 朝 LINE 通知の前段階 (= LINE 接続前は手動 read で運用)
- Stage 3 昇格条件 (R-19-08 / F053 / F241) を満たすまでは少額・手動限定

## §3 Pilot GO / NO-GO 条件

### §3.1 GO 条件 (= 全 件 満たす)

(= W60-post Codex Lane A HIGH 指摘の修正: 4 成果物全件 + DATA-R3 OK 必須を明確化)

- Wave 60-impl / W60-post / W60.6-fix が **完了済** (= 本 doc 作成時点で OK)
- **`paper_live_ledger` が生成できる**
- **`good_candidate_ranking` が生成できる**
- **`pattern_candidate_report` が生成できる**
- **`morning_line_material` が生成できる**
- `forbidden_phrases_check.passed = True`
- 全 ledger entries で `auto_order_allowed = False` / `manual_review_required = True`
- **DATA-R3 freshness verdict = `OK` (= 必須)。`MISSING` / `FAILED` / 不明 は §3.2 NO-GO へ。
  「取引サイズ最小化」運用は §3.1 GO 後の §5.2 内で別途扱う**
- 当日朝に藤原さんがレポートを確認できる時間がある
- 15:10 までに手動クローズ可能な日である
- 体調 / 本業都合で寄付き〜場中の監視が可能
- 大型イベント (= FOMC / 雇用統計 / 重要 IR) / 決算跨ぎ ではない

### §3.2 NO-GO 条件 (= 1 件 でも 該当 → 当日 trade 禁止)

- `forbidden_phrases_check.passed = False`
- **DATA-R3 freshness verdict が `OK` 以外** (= `FAILED` / `MISSING` / 不明 / 取得失敗、全て NO-GO)
- 候補根拠 (`reason_tags`) が空、または `risk_notes` が重大
- 出力が `reports/after_r1/` 以外
- 4 成果物 (paper_live_ledger / good_candidate_ranking / pattern_candidate_report /
  morning_line_material) のいずれかが **生成失敗 / 不在**
- `auto_order_allowed = True` が 1 件でもある
- `manual_review_required = False` が 1 件でもある
- 藤原さんが相場を見られない (= 出張 / 会議 / 体調不良)
- 15:10 までに手動クローズできない
- 大型イベント / 決算跨ぎ
- レポート間で不整合 (= morning_line_material の top 候補が ranking 上位と乖離)

## §4 初期 scope

| 項目 | 値 |
|---|---|
| 期間 | **初期 5 営業日** (= 2026-05-15 〜 2026-05-21 のうち営業日 5 日、または HQ 判断で調整) |
| 1 日銘柄数 | **最大 1〜2 銘柄** |
| 選択元 | `morning_line_material` の top 1〜3 候補から **藤原さんが手動選択** |
| 取引種別 | **デイトレ / 短期のみ** (= 信用は当日 15:10 までに必ず手動クローズ) |
| 決算跨ぎ | **原則禁止** |
| 自動発注 | **なし** (= 楽天 / iSPEED は藤原さん手動操作のみ) |
| ナンピン / 追加買い | **禁止** |
| 初週ロット拡大 | **禁止** (= 連勝してもロット固定) |

## §5 Risk limits (= 第 5 章 R-05-02/04/06 + 第 39 章 R-39-02 整合)

### §5.1 資産前提

- 投資元本 (= pilot 用): **780 万円** 想定
- 第 5 章 R-05-02: デイトレ通常 0.4% / スイング 0.6%
- 第 39 章 R-39-02 (= m=0 初月半減): デイトレ 0.2% / スイング 0.3%

### §5.2 初期 pilot 上限 (= R-39-02 m=0 相当を採用)

| 項目 | 上限 | 資産比 |
|---|---|---|
| 1 トレード最大損失 | **5,000 〜 15,000 円** | 0.06% 〜 0.2% |
| 1 日最大損失 | **20,000 〜 30,000 円** | 0.25% 〜 0.4% |
| 連敗停止 | **2 連敗 または 1 日最大損失到達** で 当日停止 |
| 週次損失停止 | **週次 50,000 円超 で 翌週まで停止 または ロット半減** |
| 最大建玉 | **1〜2 銘柄** 同時保有 |
| ナンピン | **禁止** |
| 追加買い | **禁止** |
| 初週ロット拡大 | **禁止** |

### §5.3 ロットサイズ算出 (= 参考)

```
1 トレード最大損失 ÷ 1 株あたり想定リスク (= entry - stop_loss 価格差) = 株数
```

例:
- 1 トレード最大損失 10,000 円
- entry 1,000 円、stop_loss 950 円 → 1 株 50 円リスク
- 株数 = 10,000 ÷ 50 = 200 株 → 100 株単位なので 100 株 (= 1 単元)、または 200 株

100 株未満の minilot は対象外 (= 楽天証券で minilot 非対応の場合)。

## §6 Entry / Stop / Take Profit / Close rules

### §6.1 Entry

- **FIRE 候補だけで即買いしない** — 藤原さんの目視確認必須
- 寄付き直後の **乱高下は避ける** (= 寄付き 5 分以内は様子見推奨)
- **必ず手動確認**:
  - 出来高 (= 当日 5 分足 出来高 vs 過去平均)
  - 気配 (= 寄付き気配 / 板の厚さ)
  - 指数 (= 日経 / TOPIX / セクター指数 の朝の地合い)
  - セクター (= 同セクター上位銘柄の動き)
  - 当日材料 (= TDnet 確認、F062 が読んだ材料と整合)
- `manual_buy_checklist` を全件満たす場合のみ entry

### §6.2 Stop loss

- **事前に損切り価格を決める** (= entry 前に値段を書く、後付け禁止)
- 価格が損切りに到達 → **手動撤退** (= 楽天逆指値推奨、ただし発注は手動)
- **損切りを後ろにずらさない** (= 「もう少し待てば戻る」厳禁)

### §6.3 Take profit

- **事前に利確目安を決める**
- 急騰時は 100 株単位なので **利確 / 継続を明確に判断** (= 分割利確は minilot 不可)
- **迷ったら利益確保優先** (= 含み益→含み損は最大級失敗パターン)

### §6.4 Close (= デイトレ / 信用)

- 信用デイトレ は **15:10 までに必ず手動クローズ** (= F133 時刻ガード相当)
- **持ち越し禁止** (= 信用は強制決済リスク、現物でも翌日 GD 下落リスク)
- 15:00 時点 で未決 → **原則クローズ準備** (= 15:00-15:10 で手動成行)
- システムではなく **藤原さんが手動で実行**

## §7 No-trade (= 見送り) 条件

以下 1 件でも該当 → 当該銘柄 skip:

- 候補の `reason_tags` が **薄い** (= 0〜1 件)
- `risk_notes` が **重大** (= 「DATA-R3 FAILED」「材料疑義」等)
- 出来高が **弱い** (= 平均比 50% 未満)
- 寄付きが **荒すぎる** (= 気配が ±5% 以上で乱高下)
- 指数が **逆風** (= 日経 -1% 超で寄付き)
- セクターが **弱い** (= 同セクター上位が下落)
- 板が **薄い** (= 気配本数 5 本未満 / 5,000 株未満)
- 決算 / 重要 IR / FOMC 直前
- **FIRE 候補と藤原さんの裁量が一致しない** (= 違和感)
- 監視時間が取れない (= 寄付き〜15:10 通して画面前にいられない)
- レポートに不整合 (= ranking と morning_line_material で違う ticker)

## §8 Templates

### §8.1 Trade plan template

→ [[../04_daily/template_manual_live_pilot_trade_plan|template_manual_live_pilot_trade_plan.md]]

毎朝 trade 前 に記入:
- date / market condition / FIRE report paths / top candidates / selected ticker /
  why / entry / SL / TP / no-trade conditions / manual checklist / final decision
- `auto_order_allowed = false` / `manual_review_required = true` を明示
- final decision = `enter` / `watch` / `skip`

### §8.2 Trade review template

→ [[../04_daily/template_manual_live_pilot_review|template_manual_live_pilot_review.md]]

15:30 以降に記入:
- planned vs actual / PnL / pattern matched? / what worked / what failed /
  improvement / promote / suppress / watch 提案 / next action

## §9 Emergency stop / rollback 条件

### §9.1 即日停止 (= 当日 trade 中止 / 残ポジは 15:10 までに手動 close)

- `forbidden_phrases_check.passed = False`
- 任意 ledger entry で `auto_order_allowed = True`
- 任意 ledger entry で `manual_review_required = False`
- 2 連敗
- 1 日最大損失到達
- システム出力に矛盾 (= ranking と material で違う ticker)
- 想定外の銘柄が出る (= 知らない code / 上場廃止間近)
- 発注ミス (= 数量間違い / 売買逆 / 銘柄違い)
- 15:10 クローズ困難 (= 値段付かない / 板薄)
- 藤原さんが監視不能になった (= 急用 / 体調)

### §9.2 週次停止 (= 翌週月曜まで pilot 一時休止)

- 週次損失 50,000 円超 (= 案 A: 翌週停止 / 案 B: ロット半減で再開)
- 週内で review 未記入が 2 日以上
- 週内でルール違反が 2 回以上 (= 損切ずらし / 持ち越し / ロット拡大)
- system artifact 欠損が連続 (= morning_line_material 生成失敗 が 2 日以上)

### §9.3 緊急 close 手順

1. 楽天 Web / iSPEED で **手動で全銘柄成行売り** (= 信用は反対売買)
2. クローズ確認後、本 doc 「Emergency log」に記入 (= §13 参照)
3. HQ 報告 (= fire-vault / 04_daily に記録)
4. 翌日 pilot 再開は HQ 判断

**LINE 緊急 アラート部屋 (= F236) は別 wave 統合、現状は手動 close 前提。**

## §10 Paper Live MVP artifact との連携手順

### §10.1 朝 (= 寄付き前 8:30 〜 9:00 想定)

**D3 (= 2026-05-18 月) 以降の標準手順**:

詳細は [[../04_daily/template_d3_real_artifact_prep|template_d3_real_artifact_prep.md]] 参照。
要約 (= 4 step manual command):

1. **F111 advisory preview** を用意 (= 朝 batch 未稼働の場合は藤原さん手作り、
   雛形 `template_f111_synthetic_preview.json`)
2. **DATA-R3 freshness** を用意 (= dry-run runner または synthetic OK)
3. **F062 LINE preview runner** を `--dry-run --require-freshness-ok` で実行 →
   `/tmp/fire_d3_prep/f062_actual_output_<date>.json` 生成 (= dict with chunks/
   selected_count/...、artifact_source=f062_preview 判定の根拠)
4. **AFTER-R1 MVP** を実行 → `reports/after_r1/` に 4 種類成果物生成

```bash
cd ~/fire && export DATE=$(date +%Y-%m-%d)

# Step 3 - F062 no-send preview
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/fire_d3_prep/f111_preview_$DATE.json \
  --output-json /tmp/fire_d3_prep/f062_actual_output_$DATE.json \
  --output-summary-json /tmp/fire_d3_prep/f062_summary_$DATE.json \
  --freshness-report-json /tmp/fire_d3_prep/data_r3_freshness_$DATE.json \
  --require-freshness-ok --dry-run

# Step 4 - AFTER-R1 MVP
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all --base-date $DATE \
  --f062-preview-json /tmp/fire_d3_prep/f062_actual_output_$DATE.json \
  --f062-preview-summary-json /tmp/fire_d3_prep/f062_summary_$DATE.json \
  --data-r3-freshness-json /tmp/fire_d3_prep/data_r3_freshness_$DATE.json \
  --output-dir reports/after_r1
```

5. `reports/after_r1/morning_line_material_<base_date>.md` を iPhone / Mac で確認
6. `reports/after_r1/good_candidate_ranking_<base_date>.md` の score / why_selected 確認
7. `reports/after_r1/pattern_candidate_report_<base_date>.md` の pattern tag 確認
8. `reports/after_r1/paper_live_ledger_<base_date>.md` の entry/SL/TP 目安確認
9. **artifact_source 確認**: f062_preview → GO 候補 / synthetic_fixture → HOLD/skip /
   unknown → NO-GO

**D1 (= 2026-05-14) / D2 (= 2026-05-15) は synthetic_fixture 直接 input のため
HOLD 判定**だった。D3 以降は F062 runner 経由で f062_preview 判定可能。

### §10.2 trade plan 記入

template_manual_live_pilot_trade_plan.md を以下に複製:
```
~/fire-vault/04_daily/<YYYY-MM-DD>_manual_live_pilot_trade_plan.md
```

### §10.3 寄付き〜場中 (= 9:00 〜 15:10)

- 藤原さんが **iSPEED / 楽天 Web で手動発注**
- FIRE は **何もしない** (= read-only material 生成済、追加 action なし)
- entry / 損切到達 / 利確到達 で手動操作

### §10.4 15:10 強制 close

- 信用 → 必ず手動 close
- 現物 → 持ち越し可だが、pilot scope では **持ち越し禁止** (= デイトレ限定)

### §10.5 終了後 review (= 15:30 〜 当日中)

template_manual_live_pilot_review.md を以下に複製:
```
~/fire-vault/04_daily/<YYYY-MM-DD>_manual_live_pilot_review.md
```

### §10.6 翌日 pattern 分類

- review の `pattern matched?` 欄を見て、pattern_candidate_report の各 pattern を
  **promote / suppress / watch** に手動分類
- これは **memory 蓄積** (= 再現性パターンの第一歩)
- pattern_candidate_report への自動反映は **次 wave** (= W61-pre 想定)

## §11 Manual-only / No auto-order 原則

| 項目 | 明示禁止 |
|---|---|
| FIRE による自動発注 | **しない** |
| Computer Use による画面操作 | **しない** |
| Playwright / Selenium による発注 | **しない** |
| LINE 経由の直接発注 | **しない** (= LINE は通知のみ) |
| 楽天 RIT / API による自動発注 | **しない** |

FIRE は:
- 候補生成 (= `reports/after_r1/*.json/md`)
- 判断材料の提示 (= why_selected / risk_notes / pattern_tags)
のみを行う。

**最終判断 / 発注 / 損切実行 は全て藤原さん本人** が iSPEED / 楽天 Web から手動で行う。

これは投資助言ではなく、**手動運用補助** (= 藤原さんが material を参考に自己責任で判断) として運用する。

## §12 Pilot 終了 / Stage 3 昇格判定

5 営業日後 (= 約 1 週間後) に以下を check:

| 項目 | 判定基準 |
|---|---|
| ルール遵守率 | ≥ 90% (= 損切ずらし 0 / 持ち越し 0 / 自動発注 0) |
| 週次損失 | ≤ 50,000 円 |
| ルール違反 | 0 件 (1 件あれば原因分析必須) |
| pattern hit 率 | review で計測、Stage 3 昇格条件 R-19-08 (= 20 営業日 / 50 トレード) との突合 |
| 緊急停止発動 | 0 件理想 |
| 楽天証券との照合 | 全 trade で建玉一致 (= FIRE は正本でない) |

pilot 評価:
- 全項目 PASS → 期間延長 + ロット段階拡大 (= R-39-03 m+1 0.3% へ)
- 1 項目 FAIL → 原因分析 + 翌週同条件で再 pilot
- 2 項目以上 FAIL → pilot 一時停止 + HQ 判断

## §13 Emergency log section (= 緊急停止時 記録)

```
[Emergency stop log]
- date:
- time:
- trigger: (forbidden_check_failed / 2連敗 / 1日上限 / artifact_mismatch / 発注ミス / 監視不能 / その他)
- 状況詳細:
- 当日損益:
- 残ポジ処理:
- 翌日対応: (停止 / ロット半減 / 通常再開)
- HQ 報告先:
```

## §14 安全境界 (= 本 doc 適用範囲)

- 本 doc は **docs / template のみ** (= FIRE 本体コード変更 0)
- 実発注 / 自動発注 / LINE 送信 / DB write / token 参照 / API call は **0**
- 楽天証券 / iSPEED 操作は **藤原さん本人の手動作業** として記述するのみ
- FIRE は **候補と判断材料を出すだけ**、最終判断は藤原さん
- 投資助言ではなく、**手動運用補助記録** として位置付け

## §15 残課題 / 次 wave

| Wave | 内容 |
|---|---|
| W60-pilot-pre (= 本 wave) | docs / template 整備 |
| W60-pilot-D1 (= 想定) | 初回 pilot trade plan 記入 + 1 日実弾 (= 藤原さん手動) |
| W60-pilot-W1 (= 想定) | 1 週間 pilot 終了 + review 集約 + pattern 分類 |
| W60-launchd-pre | nightly cron read-only plist 設計 (= 朝 自動生成) |
| W60-integration | F062 朝 batch ↔ morning_line_material 連携 |
| W61-pre | price/return data 取り込み + paper_pnl |

---

## 関連リンク

- [[../04_daily/template_manual_live_pilot_trade_plan|trade plan template]]
- [[../04_daily/template_manual_live_pilot_review|trade review template]]
- [[F286_AFTER_R1_paper_live_mvp_2026-05-14|MVP 設計 doc]]
- [[../02_todo/FIRE_CODEX_R1_WAVE60_IMPL_results|W60-impl results]]
- [[../02_todo/FIRE_CODEX_R1_WAVE60_POST_results|W60-post adversarial audit]]
- [[../02_todo/FIRE_CODEX_R1_WAVE60_6_FIX_results|W60.6-fix output guard hardening]]
