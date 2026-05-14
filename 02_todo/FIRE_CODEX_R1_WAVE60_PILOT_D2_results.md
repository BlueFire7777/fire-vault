---
id: FIRE-CODEX-R1-WAVE60-pilot-D2-results
phase: 本番 v0 中核 / Wave 60-pilot-D2 / D2 Manual Live Pilot
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
pilot_day: D2
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_INTEGRATION_results.md
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/2026-05-15_manual_live_pilot_trade_plan.md
  - 04_daily/2026-05-15_manual_live_pilot_review.md
---

# Wave 60-pilot-D2 Results — D2 Manual Live Pilot v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 D2 (= 2026-05-15 金) trade plan + review template 完成、artifact_source 判定で
HOLD/skip_recommended 結論 ✓**

W60-integration の artifact_source 判定 (= synthetic_fixture / f062_preview /
unknown) を **実運用初日として使用**。実 F062 朝 batch は未稼働のため D2 は
**synthetic_fixture** で生成 → **pilot_use = skip_recommended** → **判定 HOLD**。

参考用に F062 actual format fixture (= /tmp/fire_w60_int_smoke/) でも 5/15 base_date
で別 path に生成し、**artifact_source = f062_preview** 動作を実証 (= 内容は手作りなので
正本ではない)。

## §2 D2 MVP artifact 生成

### §2.1 Primary (= synthetic_fixture、reports/after_r1/ に正規生成)

```bash
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all --base-date 2026-05-15 \
  --f062-preview-json /tmp/fire_w60_smoke/f062_preview.json \
  --f062-preview-summary-json /tmp/fire_w60_smoke/f062_summary.json \
  --data-r3-freshness-json /tmp/fire_w60_smoke/data_r3_freshness.json \
  --ops-summary-json /tmp/fire_w60_smoke/ops_summary.json \
  --output-dir /Users/bluefire/fire/reports/after_r1
```

結果:
- artifact_source = **synthetic_fixture** ✓
- f062_raw_kind = plain_list
- ranking_size = 6
- 4 file × (JSON+Markdown) = 8 file 生成 in `reports/after_r1/`

### §2.2 Reference (= F062 actual format、/tmp/fire_w60_pilot_d2_ref/)

```bash
.venv/bin/python -m scripts.jobs.run_f286_after_r1_night_batch \
  --mode mvp --task all --base-date 2026-05-15 \
  --f062-preview-json /tmp/fire_w60_int_smoke/f062_actual.json \
  --data-r3-freshness-json /tmp/fire_w60_int_smoke/data_r3_ok.json \
  --output-dir /tmp/fire_w60_pilot_d2_ref/output
```

結果:
- artifact_source = **f062_preview** ✓ (= 自動推定)
- f062_raw_kind = f062_actual_dict
- ranking_size = 2 (= レーザーテック 6920 / 信越化学 4063)
- ※ 内容は手作り、F062 朝 batch 実稼働ではない (= 動作実証用)

両 fixture で **artifact_source 判定が正常動作** することを実証。

## §3 D2 判定: HOLD (= SKIP_RECOMMENDED)

### §3.1 GO check (= 構造的に確認可能、全 PASS ✓)

| 項目 | 結果 |
|---|---|
| 4 成果物 reports/after_r1/ に揃っている | ✓ |
| DATA-R3 freshness verdict | OK (= synthetic 由来) |
| forbidden_phrases_check.passed | True ✓ |
| material auto_order_allowed | False ✓ |
| material manual_review_required | True ✓ |
| ledger all_auto_order_disallowed | True ✓ |
| ledger all_manual_review_required | True ✓ |
| ranking と material の top ticker 整合 | ✓ (7203/6758/9984) |
| 出力 path | `reports/after_r1/` ✓ |
| **artifact_source** | **synthetic_fixture** ⚠ |

### §3.2 Pilot judgment

artifact_source = `synthetic_fixture`:
- **pilot_use = skip_recommended**
- **判定 = HOLD**
- 実 trade は **非推奨** (= 運用フロー検証のみ可)
- 藤原さんが「運用検証用に self-責任で 1 銘柄少額試行」する場合は自己責任
- 原則は **skip**

GO 判定にするための条件:
- 実 F062 朝 batch 稼働 (= W60-launchd-pre + Wave 41 / Wave 45 進捗待ち)
- artifact_source = `f062_preview` で生成

## §4 Top candidates (= D1 と同じ synthetic fixture のため同じ)

| rank | ticker | name | label | label_emoji | confidence | score |
|---|---|---|---|---|---|---|
| 1 | 7203 | トヨタ自動車 | 積極的買い推奨 | 🟢 | 0.85 | 194.0 |
| 2 | 6758 | ソニーG | 積極的買い推奨 | 🟢 | 0.72 | 179.0 |
| 3 | 9984 | ソフトバンクG | 条件付き買い推奨 | 🟡 | 0.65 | 138.0 |

(= 参考: F062 actual format fixture では別 candidates 6920 レーザーテック / 4063 信越化学)

Pattern matched (= D1 と同じ):
| pattern_id | matched | tickers |
|---|---|---|
| freshness_ok_high_confidence | 2 | 7203, 6758 |
| manual_review_active_label | 3 | 7203, 6758, 9984 |
| multi_reason_basis | 2 | 7203, 6758 |
| low_risk_note | 3 | 7203, 6758, 9984 |

全 pattern `status=candidate_only` / `validation_status=unvalidated`。

## §5 作成 file 一覧

### §5.1 新規 (4 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 04_daily/2026-05-15_manual_live_pilot_trade_plan.md | D2 trade plan / 14 セクション / HOLD 明示 |
| 04_daily/2026-05-15_manual_live_pilot_review.md | D2 review blank / 15 セクション / synthetic 特記 |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_results.md | 本 wave results (= 本 doc) |

### §5.2 reports/after_r1/ 追加 (= 8 file)

D2 用 base_date=2026-05-15 で 4 種類 × (JSON+Markdown) = 8 file 追加。
D1 (5/14) artifact と共存。

### §5.3 /tmp/fire_w60_pilot_d2_ref/ (= 参考用)

F062 actual format で生成した artifact が 8 file 保存。
正本ではない (= reports/after_r1/ に置かない)、artifact_source=f062_preview の
動作実証用。

### §5.4 FIRE 本体不触 ✓

- scripts/jobs/_after_r1_mvp.py / run_f286_after_r1_night_batch.py / tests 全て **不触**
- pytest collected: 4610 (= W60-integration と同じ、不変)

## §6 D2 リスク上限 (= operation_plan §5 / m=0 R-39-02 整合)

| 項目 | 上限 | D2 想定 |
|---|---|---|
| 1 日銘柄数 | 最大 1-2 / **D2 = skip 推奨** | skip |
| 1 トレード最大損失 | 5,000-15,000 円 | skip |
| 1 日最大損失 | 20,000-30,000 円 | skip |
| ナンピン | 禁止 | — |
| 追加買い | 禁止 | — |
| 持ち越し | 禁止 | — |
| 信用 15:10 close | 必須 | — |
| 自動発注 | 0 (= 構造的) | ✓ |
| 楽天 / iSPEED 操作 | 藤原さん手動のみ | ✓ |
| Computer Use | 不採用 | ✓ |

## §7 Codex 8 lane 判断: 不使用

本 wave は **日次 trade plan 作成 + 本線主導**、ユーザー spec で:
> Codex運用:
> - 今回は日次trade plan作成なので本線主導でよい。
> - 重要日次では4 lane priority batchまで推奨。

判断:
- D1 (= W60-pilot-D1) で同 pattern を確立済 (= Codex 0/8 + 本線 self-check)
- 本 D2 は D1 template から複製 + artifact_source 判定追記のみ
- artifact_source 判定 logic は W60-integration で Codex 6 lane audit 済 (LOW No finding)
- 追加 Codex audit 価値が低い (= 同 template / 同 logic)
- 毎日 Codex 8 lane は過剰、W1 集約 (W60-pilot-W1) で実施が効率的

代わりに本線 self-check:
- ✓ D1 template から複製
- ✓ artifact_source 判定 自動推定動作確認
- ✓ HOLD/skip 推奨明示
- ✓ 命令形 phrase 0
- ✓ auto_order_allowed=False / manual_review_required=True 全箇所明示

## §8 安全確認

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
| **楽天証券 / iSPEED 操作** | 0 (= claude code は触らない) |
| **Computer Use** | 0 |
| 自動発注 | 0 |
| workflow / --no-verify / git push / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |
| FIRE 本体コード変更 | 0 |
| file write 範囲 | reports/after_r1/ (= D2 8 file) + fire-vault/04_daily,02_todo/ (= 4 file) + /tmp/fire_w60_pilot_d2_ref/ (= reference) |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 DB mtime/size | 不変 ✓ |
| pytest collected | 4610 (= 不変、tests 未追加) |

## §9 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **0/8 = 0%** (= 本線主導、強い技術的理由 §7) |
| 短縮率 | N/A |
| 採用率 | N/A |
| 差戻率 | 0 |
| Integrator 負荷 | 中 (= D2 artifact 生成 × 2 + trade plan + review + reference + vault) |
| 安全事故 | **0** ✓ |

### Codex 0/8 の理由

- 日次 trade plan 作成 wave (= user spec で「本線主導でよい」)
- D1 template + W60-integration logic を継承するだけで新規 logic なし
- artifact_source 判定は W60-integration で Codex 6 lane audit 済 (LOW No finding)
- W1 集約 wave (= W60-pilot-W1) で Codex 8 lane audit する方が効率的

## §10 D1 → W1 path (= W60-integration 修正済)

| Day | 日付 | 曜日 | 状態 |
|---|---|---|---|
| D1 | 2026-05-14 | 木 | trade plan 作成済、artifact_source=synthetic_fixture |
| **D2** | **2026-05-15** | **金** | **本 wave、artifact_source=synthetic_fixture / HOLD** |
| - | 2026-05-16 | 土 | skip (= 休場) |
| - | 2026-05-17 | 日 | skip (= 休場) |
| D3 | 2026-05-18 | 月 | 予定 |
| D4 | 2026-05-19 | 火 | 予定 |
| D5 | 2026-05-20 | 水 | 予定、W60-pilot-W1 集約 |

## §11 今日 (5/15) の人間アクション

### §11.1 藤原さん が朝行うこと

1. `~/fire-vault/04_daily/2026-05-15_manual_live_pilot_trade_plan.md` を確認
2. §3.0 artifact_source = synthetic_fixture / pilot_use=skip_recommended 確認
3. §13 final decision で **skip** を選択 (= 推奨)
4. (自己責任の場合) §6-§11 を埋めて 1 銘柄 少額 entry も可
5. enter 選択時 → iSPEED で 手動発注 / 15:10 までに手動 close

### §11.2 藤原さん が場後 行うこと

1. `~/fire-vault/04_daily/2026-05-15_manual_live_pilot_review.md` 開く
2. §1-§13 記入 (= skip でも §8.4 D2 運用フロー評価 / §13 next action 重要)
3. 翌週 D3 (5/18 月) の準備:
   - F062 朝 batch 稼働状況確認
   - W60-launchd-pre / Wave 41 進捗確認
   - artifact_source 推移期待 (= synthetic → f062_preview)

### §11.3 Claude Code 行わないこと (= 絶対禁止)

- 実発注 / 楽天 / iSPEED / Computer Use / 自動発注 / LINE / DB write /
  token / API / launchctl / plist / cron / workflow / git push / sudo / rm

## §12 残課題 / 次 Wave

### §12.1 D2 → D3 移行

- 5/16 (土) / 5/17 (日) 休場
- D3 = 5/18 (月) → 同じ pattern で trade plan 作成
- F062 朝 batch 稼働状況に応じて artifact_source 変化期待

### §12.2 並行 wave

- **W60-launchd-pre**: nightly cron read-only plist 設計 (= 朝 自動 generate、
  synthetic → f062_preview path)
- F062 本体側 artifact_source 明示 (= 別 wave 検討)
- **W61-pre**: price/return data 取り込み + paper_pnl 連携

### §12.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §12.4 W60-pilot-W1 集約 wave (= D5 完了後)

D1-D5 集約レビューで:
- ルール遵守率 集計
- pattern promote/suppress/watch 結論
- artifact_source 推移 (= synthetic → f062_preview の到達度)
- Stage 3 昇格条件 (R-19-08 / F053 / F241) との突合

## §13 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_PILOT_D2_plan]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D1_results|W60-pilot-D1 results]]
- [[FIRE_CODEX_R1_WAVE60_INTEGRATION_results|W60-integration results]]
- [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- [[../04_daily/2026-05-15_manual_live_pilot_trade_plan|5/15 trade plan]]
- [[../04_daily/2026-05-15_manual_live_pilot_review|5/15 review (blank)]]
