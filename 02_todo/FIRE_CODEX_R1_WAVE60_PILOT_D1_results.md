---
id: FIRE-CODEX-R1-WAVE60-pilot-D1-results
phase: 本番 v0 中核 / Wave 60-pilot-D1 / Small Manual Live Pilot D1
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
pilot_day: D1
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_plan.md
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/2026-05-14_manual_live_pilot_trade_plan.md
  - 04_daily/2026-05-14_manual_live_pilot_review.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_PRE_results.md
---

# Wave 60-pilot-D1 Results — Small Manual Live Pilot D1 Trade Plan v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 少額手動実弾パイロット D1 (= 初日) trade plan + review template 完成 ✓**

W60-pilot-pre 設計の運用 plan に基づき、**5/14 (= 木) D1 の trade plan +
review template + reports/after_r1/ 4 成果物** を生成。

藤原さんが朝に確認し、**入る / 見送る を手動判断できる状態** に到達。

**初日特記**: 入力は **synthetic fixture** (= W60-impl smoke 用 hardcoded data)、
実 F062 朝 batch / 実 DATA-R3 batch ではない。trade plan / review に明示済。

## §2 Pilot GO/NO-GO 判定

### §2.1 GO check (= 構造的に確認可能な項目、全 PASS ✓)

| 項目 | 結果 |
|---|---|
| 4 成果物 reports/after_r1/ に揃っている | ✓ |
| DATA-R3 freshness verdict | OK ✓ |
| forbidden_phrases_check.passed | True ✓ |
| material auto_order_allowed | False ✓ |
| material manual_review_required | True ✓ |
| ledger all_auto_order_disallowed | True ✓ |
| ledger all_manual_review_required | True ✓ |
| ranking と material の top ticker 整合 | ✓ (7203/6758/9984) |
| 出力 path | `reports/after_r1/` ✓ |

### §2.2 藤原さん 判断項目 (= trade_plan §1, §2, §6-§13 で記入)

- 当日朝レポート確認時間: 藤原さん判定
- 15:10 までに手動 close 可能: 藤原さん判定
- 監視可能時間: 藤原さん判定
- 大型イベント / 決算跨ぎチェック: 藤原さん判定 (= TDnet / 日経)
- top 1-3 から selection: 藤原さん判定 (= 初日 7203 推奨)
- final decision (= enter / watch / skip): 藤原さん判定

### §2.3 判定結果

- **Claude Code 側 GO check**: **全 PASS** ✓
- **藤原さん側判断**: trade_plan に記入欄を整備済
- **初日特記**: 入力 synthetic のため「skip」推奨 (= 実 batch 稼働後に再開) /
  ただし運用フロー検証目的で「enter」も自己責任で可

## §3 5/14 MVP artifact 生成結果

```bash
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all \
  --base-date 2026-05-14 \
  --f062-preview-json /tmp/fire_w60_smoke/f062_preview.json \
  --f062-preview-summary-json /tmp/fire_w60_smoke/f062_summary.json \
  --data-r3-freshness-json /tmp/fire_w60_smoke/data_r3_freshness.json \
  --ops-summary-json /tmp/fire_w60_smoke/ops_summary.json \
  --output-dir /Users/bluefire/fire/reports/after_r1
```

生成 file (= 4 種類 × (JSON + Markdown) = 8 file):
- `reports/after_r1/paper_live_ledger_2026-05-14.{json,md}`
- `reports/after_r1/good_candidate_ranking_2026-05-14.{json,md}`
- `reports/after_r1/pattern_candidate_report_2026-05-14.{json,md}`
- `reports/after_r1/morning_line_material_2026-05-14.{json,md}`

### §3.1 Top candidates (= morning_line_material 抽出)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | **7203** | トヨタ自動車 | 積極的買い推奨 | 🟢 | 0.85 | 194.0 |
| 2 | **6758** | ソニーG | 積極的買い推奨 | 🟢 | 0.72 | 179.0 |
| 3 | **9984** | ソフトバンクG | 条件付き買い推奨 | 🟡 | 0.65 | 138.0 |

### §3.2 Pattern matched (= pattern_candidate_report)

| pattern_id | matched | tickers |
|---|---|---|
| freshness_ok_high_confidence | 2 | 7203, 6758 |
| manual_review_active_label | 3 | 7203, 6758, 9984 |
| multi_reason_basis | 2 | 7203, 6758 |
| low_risk_note | 3 | 7203, 6758, 9984 |

全 pattern `status = candidate_only` / `validation_status = unvalidated`。

### §3.3 初日 推奨選択

**7203 トヨタ自動車** (= 1 銘柄):
- 最高 score (= 194.0) / 4 pattern 全 matched
- conf = 0.85 (= 高)
- 根拠: 決算ポジティブ / 出来高伴う上昇 / 上方修正期待
- 大型株 (= 板厚い、初日に適する)

ただし **初日 synthetic fixture** なので、藤原さんが「実 batch 稼働後まで skip」も可。

## §4 作成 file 一覧

### 新規 (4 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 04_daily/2026-05-14_manual_live_pilot_trade_plan.md | D1 trade plan (= 14 セクション、初日特記あり) |
| 04_daily/2026-05-14_manual_live_pilot_review.md | D1 review blank (= 15 セクション、初日特記あり) |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_results.md | 本 wave results (= 本 doc) |

### reports/after_r1/ (= 8 file 新規)

- 4 種類 × (JSON + Markdown) (= D1 入力で生成)

### FIRE 本体 (= 不触 ✓)

- scripts/jobs/_after_r1_mvp.py / run_f286_after_r1_night_batch.py / tests
  全て **不触**
- pytest collected: 4593 (= 不変)

## §5 D1 リスク上限 (= operation_plan §5 / template §12)

| 項目 | 上限 | 整合 |
|---|---|---|
| 1 日銘柄数 | 最大 1-2 銘柄、**D1 = 1 銘柄推奨** | ✓ |
| 1 トレード最大損失 | 5,000-15,000 円 (= 0.06-0.2%) | ✓ F130 m=0 R-39-02 |
| 1 日最大損失 | 20,000-30,000 円 (= 0.25-0.4%) | ✓ |
| ナンピン | 禁止 | ✓ |
| 追加買い | 禁止 | ✓ |
| 持ち越し | 禁止 (= デイトレ限定) | ✓ |
| 信用 15:10 close | 必須 | ✓ F133 R-05-10/11 |
| 自動発注 | 0 (= FIRE 構造的) | ✓ |
| 楽天 / iSPEED 操作 | 藤原さん手動のみ | ✓ |
| Computer Use | 不採用 | ✓ |

## §6 Codex 8 lane 判断

本 wave は **本線主導** (= 実発注直前の運用計画) であり、user spec で:
> Codex運用:
> - 今回は実発注直前の運用計画なので、本線主導。
> - Codexを使う場合はread-only auditのみ。
> - 4〜6 lane分割、5〜10秒sleep推奨。

W60-pilot-pre / W60.6-fix で Codex 8 lane 検証済、本 wave は **Codex 不使用 を選択**:
- trade plan は構造的に MVP artifact から自動転記 + 藤原さん記入欄
- review template は blank 雛形 (= 内容判断は当日場後)
- W60-pilot-pre で operation_plan / templates 既に Codex audit 済
- 本 wave で新規追加した 5/14 _trade_plan.md / _review.md は **template 由来**、
  template が W60-pilot-pre で audit 済なので **構造的に safe**
- 追加 audit 価値が低い (= 同 content review)

これは「Codex lanes 未使用の強い技術的理由」として明示。

代わりに **本線 self-check** 実施:
- ✓ template から複製 (= W60-pilot-pre Codex audit 済 base)
- ✓ MVP artifact からの値転記 (= 構造的に正しい)
- ✓ 初日特記 (= synthetic fixture) 明示
- ✓ 命令形 phrase 0 (= 手動 grep 確認)
- ✓ auto_order_allowed=False / manual_review_required=True 全箇所明示

## §7 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 | 0 |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体参照 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| **実発注** | 0 (= claude code は何も発注しない) |
| **楽天証券 / iSPEED 操作** | 0 (= claude code は触らない、藤原さんが手動) |
| **Computer Use** | 0 |
| 自動発注 | 0 |
| workflow / --no-verify / git push / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |
| FIRE 本体コード変更 | 0 |
| file write 範囲 | reports/after_r1/ (= 8 file) + fire-vault/04_daily,02_todo/ (= 4 file) |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 DB mtime/size | 不変 ✓ |
| pytest collected | 4593 (= 不変、tests 未追加) |

## §8 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **0/8 = 0%** (= 本線主導 wave、強い技術的理由は §6) |
| 短縮率 | N/A |
| 採用率 | N/A |
| 差戻率 | 0 |
| Integrator 負荷 | 中 (= MVP runner 実行 + trade plan 作成 + review 作成 + vault docs) |
| 安全事故 | **0** ✓ |

### Codex 0/8 = 0% の強い技術的理由

1. **本 wave は実発注直前の運用計画** (= user spec 明示「本線主導」)
2. **追加 audit 価値が低い**: trade_plan / review は W60-pilot-pre で audit 済
   template から複製 + MVP artifact から値転記、構造的に safe
3. **本線 self-check で十分** (= §6)
4. **次 wave (D2 以降) でも同じ pattern が続く** ため、毎日 Codex 8 lane を回すのは
   過剰 (= W1 集約 wave で Codex audit する方が効率的)

W60-pilot-pre で Codex 3/8 + self-audit 5、W60-pilot-D1 で Codex 0/8 + self-check
合計、というのが意図的な使い分け。

## §9 今日 (5/14) の人間アクション

### §9.1 藤原さん が朝行うこと

1. `~/fire-vault/04_daily/2026-05-14_manual_live_pilot_trade_plan.md` を確認
2. §1 §2 を寄付き前に記入 (= 監視可能時間 / 大型イベント有無 / 市場コンディション)
3. §6 で ticker 選択 (= 推奨は 7203、ただし synthetic fixture 段階のため skip 推奨)
4. §7 entry plan / §8 SL / §9 TP 確定 (= enter 選択時)
5. §10 no-trade 条件 check
6. §11 manual buy checklist 全 ✓
7. §13 final decision (= enter / watch / skip)
8. enter の場合 → iSPEED で **手動発注**
9. 15:10 まで監視、15:10 までに手動 close (= 信用)

### §9.2 藤原さん が場後 (15:30 以降) 行うこと

1. `~/fire-vault/04_daily/2026-05-14_manual_live_pilot_review.md` を開く
2. §2-§13 を記入 (= skip の場合も §13 next action は記入推奨)
3. §10 pattern promote/suppress/watch (= D1 = 1 サンプル、結論は W1 で出す)
4. §13 next action 選択 (= 翌日継続 / ロット半減 / stop / 設計 doc 修正提案 等)

### §9.3 Claude Code が 行わないこと (= 絶対禁止)

- 実発注
- 楽天証券 / iSPEED 操作
- Computer Use
- 自動発注
- LINE 送信
- DB write
- token / API 参照
- launchctl / plist / cron 変更

## §10 残課題 / 次 Wave

### §10.1 W60-pilot-D1 で開ける道

- **D1 当日**: 藤原さんが trade plan を埋めて enter/skip 判断
- **D1 場後**: 藤原さんが review 記入
- **D2 (= 5/15 金、または営業日次回)**: 同じ pattern で次の trade plan

### §10.2 D1 → W1 への path (= W60-integration で修正、5/17 日も skip)

- **D1**: 2026-05-14 (木)
- **D2**: 2026-05-15 (金)
- skip: 2026-05-16 (土) / 2026-05-17 (日)
- **D3**: 2026-05-18 (月)
- **D4**: 2026-05-19 (火)
- **D5**: 2026-05-20 (水)
- **W1 集約 (= W60-pilot-W1)**: D5 完了後

### §10.3 並行 wave

- **W60-launchd-pre**: nightly cron read-only plist 設計 (= 朝 自動 generate、
  synthetic fixture からの脱却)
- **W60-integration**: F062 朝 batch ↔ morning_line_material 連携 (= 実 batch 化)
- **W61-pre**: price/return data 取り込み + paper_pnl 連携

### §10.4 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §10.5 初日 synthetic fixture からの脱却

D1 は synthetic fixture で運用フロー検証。実 F062 朝 batch / 実 DATA-R3 batch
連携には:
- W60-integration (= 設計 + 実装)
- W60-launchd-pre (= cron 起動)
- W44-pre / Wave 41 (= DATA-R3 launchd 配置)
が必要。これらは v0 本線 path と並行進行。

## §11 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_PILOT_D1_plan]]
- [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- [[../04_daily/2026-05-14_manual_live_pilot_trade_plan|5/14 trade plan]]
- [[../04_daily/2026-05-14_manual_live_pilot_review|5/14 review (blank)]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_PRE_results|W60-pilot-pre results]]
