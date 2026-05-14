---
id: FIRE-CODEX-R1-WAVE60-pilot-D7-results
phase: 本番 v0 中核 / Wave 60-pilot-D7 / 再現性確認 day
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-22
pilot_day: D7
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D7_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D6_results.md
  - 04_daily/2026-05-22_manual_live_pilot_trade_plan.md
  - 04_daily/2026-05-22_manual_live_pilot_review.md
---

# Wave 60-pilot-D7 Results — Day 7 Real Batch Research-Enriched Pilot v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**D7 (= 2026-05-22 金) chain 再現性確認完了 ✓**:
- 9 hard invariants 全 PASS ✓
- **D6/D7 candidate overlap 100%** (= 8747/5729/3489 完全一致)
- pilot_judgment = **HOLD** (= 構造的 GO だが運用 caveat 2 件)

主な発見:
1. chain 出力 deterministic ✓ (= staging signal が同じなら同候補出力)
2. staging signal 日次更新なし (= base_date 2026-05-12 で固定)
3. D6 review.md は blank (= 藤原さん未記入想定)
4. → 真の本番化には **F111 朝 batch wave (= staging signal 日次更新)** が必要

## §2 D6 review status 確認

`~/fire-vault/04_daily/2026-05-21_manual_live_pilot_review.md`:
- status: **blank (未記入)**
- 全 §1-§16 が template 通り (= claude code 事前生成 後 藤原さん未記入)

→ D7 trade plan に "D6 review missing" caveat を記載済。

## §3 D7 chain 結果

```
Step 1: F111-real-batch-staging --base-date 2026-05-22
  → candidates=15 / eligible=14 / exclusions risk_above=1, risk_estimate_missing=1
Step 2: DATA-R3 dry-run runner --base-date 2026-05-22
  → verdict=OK / db_row_writes=0
Step 3: F062 (= F111 dict 直渡し相当、AFTER-R1 が actual_dict 認識)
Step 4: AFTER-R1 MVP --base-date 2026-05-22 --output-dir reports/after_r1
  → artifact_source=f062_preview / f111_input_source=f111_real_batch /
    ranking_size=15
Step 5: 9 hard invariants check → 全 PASS ✓
```

### §3.1 9 hard invariants 結果

| # | invariant | 結果 |
|---|---|---|
| 1 | artifact_source = `f062_preview` | ✓ |
| 2 | f062_raw_kind = `f062_actual_dict` | ✓ |
| 3 | f111_input_source = `f111_real_batch` | ✓ |
| 4 | DATA-R3 freshness verdict = `OK` | ✓ |
| 5 | forbidden_check.passed = True | ✓ |
| 6 | material.auto_order_allowed = False | ✓ |
| 7 | material.manual_review_required = True | ✓ |
| 8 | safety_flags 13 keys 全 False | ✓ |
| 9 | top_candidates count = 3 ≥ 1 | ✓ |

✓ Pilot D7 check PASSED

### §3.2 D6 → D7 candidate overlap = 100%

| Day | top tickers |
|---|---|
| D6 (5/21 木) | 8747, 5729, 3489 |
| D7 (5/22 金) | 8747, 5729, 3489 |
| overlap | **100%** |
| 原因 | staging signal latest base_date = 2026-05-12 で D6/D7 共通 |

これは:
- ✓ chain 出力 deterministic (= 再現性確認)
- ⚠ staging signal 日次更新がない (= F111 朝 batch 未稼働)

### §3.3 exclusions_summary

D7: sample 0 / tradable_false 0 / risk_above_pilot_limit 1 (= Cocolive 137A0)
= D6 と同じ pattern。

## §4 D7 Pilot judgment: HOLD

### §4.1 GO 条件 (= 構造的 invariants)

9 hard invariants 全 PASS ✓

### §4.2 caveat (= 運用上 HOLD trigger)

**Caveat 1: D6 review missing**:
- D6 review.md は blank
- 藤原さん未記入 → D6 学習を D7 で反映できない
- liquidity actual / spread actual / 実 entry/skip 理由 不明

**Caveat 2: D6/D7 同候補 100% 再現**:
- D6 で entry した場合 → D7 同候補は **ナンピン抵触** → skip 必須
- D6 で skip した場合 → D7 同候補で再評価可
- ただし「同候補で新規 entry の追加学習価値」は **低い**
- staging signal 日次更新なしのため、真の翌日 trade simulation でない

→ 推奨判定 = **HOLD** (= 月曜 D8 で staging signal 更新を期待)

## §5 作成 file 一覧

### §5.1 新規 (4 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 04_daily/2026-05-22_manual_live_pilot_trade_plan.md | D7 trade plan / 16 セクション / HOLD 推奨 + D6 review missing + D6/D7 overlap 100% caveat |
| 04_daily/2026-05-22_manual_live_pilot_review.md | D7 review blank / 16 セクション / D6 → D7 連続性評価欄 |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D7_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D7_results.md | 本 wave results (= 本 doc) |

### §5.2 reports/after_r1/ (= 8 file 追加、D7 用 base_date=2026-05-22)

### §5.3 /tmp/fire_d7_prep/ (= 2 中間 artifact)

### §5.4 FIRE 本体不触 ✓

scripts/jobs/ + tests/ 不触。pytest collected: 4705 (= 不変)

## §6 D1-D7 path artifact 推移

| Day | 日付 | artifact_source | f111_input_source | top | 実 trade 候補 | judgment |
|---|---|---|---|---|---|---|
| D1 | 5/14 木 | (none) | (none) | 0 | No | HOLD |
| D2 | 5/15 金 | synthetic | (none) | 0 | No | HOLD |
| D3 | 5/18 月 | f062_preview | manual_seed | 0 | No (値嵩) | GO+caveat |
| D4 | 5/19 火 | f062_preview | manual_seed | 0 | No | GO+caveat |
| D5 | 5/20 水 | f062_preview | f111_sample | 0 | No (サンプル) | GO+caveat |
| D6 | 5/21 木 | f062_preview | f111_real_batch | 3 | **Yes** | GO+liquidity |
| **D7** | **5/22 金** | **f062_preview** | **f111_real_batch** | **3 (= D6 同)** | **Yes (= 同候補)** | **HOLD** |

進化:
- artifact_source: 1/7 (= D1) → 6/7 f062_preview
- f111_input_source 進化: 0→manual_seed→f111_sample→f111_real_batch
- top_candidates: D1-D5 全 0 件 → D6/D7 各 3 件
- 実 trade 候補 構造的 Yes: D1-D5 No → D6/D7 Yes (= 同候補)
- 再現性: D6/D7 100% overlap

## §7 W2 集約 (= D8-D10 後) 引き継ぎ材料

### §7.1 D6-D7 で確認できたこと

- chain 出力 deterministic (= 同 staging signal → 同候補)
- f111_real_batch 路線の安定性
- 中小型成長株 universe (= 8747 金融 / 5729 鉄鋼 / 3489 不動産)
- exclusion 1 件継続 (= Cocolive 値嵩)

### §7.2 D6-D7 で確認できなかったこと (= W2 で要確認)

- D6 review missing → 実 entry / 実 outcome / liquidity actual 不明
- staging signal 日次更新 (= 月曜 D8 で 2026-05-12 → 2026-05-21 等への更新期待)
- pattern hit 率 (= 実 outcome 未蓄積)

### §7.3 D8 (= 2026-05-26 月) で期待

- staging signal 更新 (= 新 base_date) → top candidates 変化
- D6 review 記入 (= 藤原さん action 待ち)
- liquidity filter 強化 wave 着手判断
- W61-pre (price/return/paper_pnl) 着手判断

## §8 Codex 0/8 判断 (= 本線主導)

ユーザー spec で「日次 trade plan は本線主導でよい」明示。
W60-research-advisory-staging で chain logic Codex audit 済。
W60-pilot-W2 (= D6-D10 後集約) で Codex 4 lane 実施予定。

本線 self-check:
- ✓ 9 hard invariants 自動 PASS
- ✓ D6/D7 overlap 機械的 100% 確認
- ✓ D6 review missing 機械的確認
- ✓ HOLD 推奨判定 (= 運用 caveat 2 件)

## §9 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 (production/develop) | 0 |
| DB sqlite 接続 (staging) | read-only 1 (URI mode=ro、F111-real-batch-staging 経由) |
| 3 DB row_writes | 0 (= DATA-R3 dry-run 確認) |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体 | 0 |
| API call | 0 |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| 実発注 / 楽天 / iSPEED / Computer Use | 0 |
| 自動発注 | 0 |
| sudo / git push / rm -rf / --no-verify | 0 |
| workflow / TODO Excel | 0 |
| FIRE 本体コード変更 | 0 |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4705 (= 不変) |

## §10 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **0/8 = 0%** (= 本線主導、強い技術的理由) |
| 短縮率 | N/A |
| 採用率 | N/A |
| 差戻率 | 0 |
| Integrator 負荷 | 中 (= chain re-run + invariants + overlap 確認 + trade plan + vault) |
| 安全事故 | **0** ✓ |

## §11 今日 (5/22 金) の人間アクション

### §11.1 藤原さん が朝行うこと

1. `~/fire-vault/04_daily/2026-05-22_manual_live_pilot_trade_plan.md` 確認
2. §2 D6 review status (= missing) 認識
3. §5 D6/D7 同候補 100% overlap 認識
4. §7.1 D6 残ポジ 確認 (= iSPEED)
5. §15 final decision (= 推奨: **skip**)
6. **理想は D6 review 記入** (= §14 next action 等)

### §11.2 場後 行うこと

1. `~/fire-vault/04_daily/2026-05-22_manual_live_pilot_review.md` 記入
2. §8 D6 → D7 同候補 再現性 評価
3. §10 D8 (= 月曜 5/26) 準備事項
4. **D6 review.md も合わせて記入** (= W2 集約に必要)

### §11.3 Claude Code 行わないこと (= 絶対禁止)

実発注 / 楽天 / iSPEED / Computer Use / 自動発注 / LINE / DB write /
token / API / launchctl / plist / cron / workflow / git push / sudo / rm

## §12 残課題 / 次 Wave

### §12.1 D7 → D8 path

- D8 (= 2026-05-26 月) で staging signal 更新を確認
- 新 base_date が出れば新候補
- staging signal 同じなら同候補 → HOLD 継続

### §12.2 次 wave 候補 (= 優先度順)

1. **W60-pilot-D8** (= 月曜): staging signal 更新確認 + 新候補 trade plan
2. **W60-pilot-W2** (= D8-D10 後集約): pattern 結論 + Codex 4 lane
3. **F111 朝 batch wave** (= 優先度高): staging signal 日次更新
4. **流動性 filter 強化**: 出来高 / 板厚 を risk_notes に追記
5. **W61-pre**: price/return/paper_pnl 連携

### §12.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2)
- Wave 52 (6/2-6/8)
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

## §13 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_PILOT_D7_plan]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D6_results|D6 results]]
- [[../04_daily/2026-05-22_manual_live_pilot_trade_plan|D7 trade plan]]
- [[../04_daily/2026-05-22_manual_live_pilot_review|D7 review (blank)]]
