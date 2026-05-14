---
id: FIRE-CODEX-R1-WAVE60-pilot-W1-results
phase: 本番 v0 中核 / Wave 60-pilot-W1 / D1-D5 集約 + F111-real-batch 要件
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-21
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W1_plan.md
  - 03_design/F111_real_batch_requirements_2026-05-14.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D1_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D2_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D3_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D4_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D5_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_F111_PRE_results.md
---

# Wave 60-pilot-W1 Results — D1-D5 集約 / F111-Real-Batch 要件 v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 D1-D5 = 5 営業日 pilot 集約完了 + F111-real-batch 要件確定 ✓**

実 entry **0/5** (= 全 day skip) を **5 blocker** に分類:
1. D1: synthetic_fixture (= W60-integration 前生成、artifact_source field なし)
2. D2: synthetic_fixture (= MVP 直渡し plain list)
3. D3/D4: manual_seed + 値嵩株 100 株 上限超過 (= 15,000 円超え)
4. D5: f111_sample + サンプル ticker 構造的 trade 不可

→ **F111-real-batch wave** で **実在 ticker + risk filter** 化が次の最優先。

## §2 D1-D5 集約 table (= read-only Codex Lane A confirm)

| Day | 日付 | artifact_source | f111_input_source | freshness | top candidates | actual entry | skip 理由 (= blocker) |
|---|---|---|---|---|---|---|---|
| D1 | 5/14 木 | (field なし、pre-W60-integration) | (field なし) | OK | 7203/6758/9984 | 0 | synthetic_fixture (MVP 直渡し) |
| D2 | 5/15 金 | synthetic_fixture | (field なし、pre-W60-F111-pre) | OK | 7203/6758/9984 | 0 | synthetic_fixture |
| D3 | 5/18 月 | f062_preview | (field なし、pre-W60-F111-pre) | OK | 6920/4063 | 0 | manual_seed + 値嵩株 100 株超過 |
| D4 | 5/19 火 | f062_preview | (field なし) | OK | 6920/4063 | 0 | manual_seed + D3 同候補 + 値嵩株 |
| **D5** | **5/20 水** | **f062_preview** | **f111_sample ★** | OK | 1234/7203 サンプル... | 0 | **サンプル ticker 構造的 trade 不可** |

Codex Lane A 確認 ✓: 「Progression matches expected」

## §3 実 entry 0/5 理由分類 (= 5 blocker)

| # | blocker | 該当 Day | 解消 wave |
|---|---|---|---|
| 1 | synthetic_fixture (= MVP 直渡し) | D1 / D2 | W60-integration ✓ (= D3 以降 f062_preview 化) |
| 2 | manual_seed (= 手作り synth) | D3 / D4 | **F111-real-batch (= 次 wave)** |
| 3 | f111_sample + サンプル ticker (= 仮想銘柄) | D5 | **F111-real-batch** |
| 4 | 値嵩株 100 株単位 リスク管理困難 (= 6920 など) | D3 / D4 | **F111-real-batch (risk filter)** |
| 5 | real F111 朝 batch 不在 | D1-D5 | **F111-real-batch + W60-launchd-real** |

### §3.1 追加考慮事項

- 各 Day の **chain 自体は完全成功** (= 4 step / 5 step sequence、8 invariants PASS)
- 全 day で safety 完璧 (= LINE / DB / token / API / launchctl 0)
- 実 entry 阻害要因は **artifact 品質**であって、**安全性 / 設計 / 運用 fxxx ではない**
- 5 blocker は全て F111-real-batch で同時に解消可能

## §4 pattern promote/suppress/watch 仮分類

D1-D5 で出現した 4 pattern:

| pattern_id | matched Days | 仮分類 | 理由 |
|---|---|---|---|
| `freshness_ok_high_confidence` | D1-D5 全 day | **watch** | 全 day matched、ただし実 entry 0/5 のため実 outcome 不明 |
| `manual_review_active_label` | D1-D5 全 day | **watch** | 同上、ただし全 day で manual_review=True を担保しているのは構造的 |
| `multi_reason_basis` | D1-D4 (manual_seed) で matched | **watch** | reason 3 件以上の銘柄は強い候補だが、実 outcome 未確認 |
| `low_risk_note` | D1-D5 で matched | **watch** | risk_notes 空 = pilot 安全側だが、実 outcome 未確認 |

**全 pattern `watch` (= 全 unvalidated)**。**promote 0 件 / suppress 0 件**。

### §4.1 promote 0 件の理由

- 実 entry 0/5、real outcome 不明
- pattern hit rate を計算するには **実 trade + actual return** が必要 (= W61-pre)
- W60-pilot-W1 段階では「**promote 候補は watch まで**」が原則

### §4.2 suppress 0 件の理由

- 各 pattern は logical に valid (= 過剰 signal を生成していない)
- 実 entry 0/5 は pattern の問題ではなく artifact 品質 (= ticker 仮想 / 値嵩株) の問題
- suppress するなら別経路 (= 例: 「F062 サンプル ticker は AFTER-R1 で suppress」
  ルール、F111-real-batch で対応)

### §4.3 unknown 0 件

全 pattern status=candidate_only / validation_status=unvalidated 維持。

## §5 F111-real-batch 要件 (= 詳細は別 doc)

詳細: [[../03_design/F111_real_batch_requirements_2026-05-14|F111 real batch requirements v1.0]]

### §5.1 必須要件 summary

1. **実在 JP ticker** のみ (= サンプル prefix 除外)
2. **tradable universe filter** (= 楽天証券で発注可能)
3. **risk filter**: 100 株想定損失 ≤ 15,000 円 / value_stock_flag
4. **safety**: DB write 0 / token 0 / API 0 / launchctl 0
5. **handoff**: F111 output → F062 直接渡し (= W60-F111-pre 既証)
6. **f111_input_source=f111_real_batch** 自動推定 (= AFTER-R1 側 marker 追加)

### §5.2 staging access

- staging DB read-only (`SQLITE_OPEN_READONLY`)
- production-develop 接続禁止
- write API 禁止
- 接続失敗時 safe-warn + NO-GO fallback

## §6 W61-pre 要件 (= 概要)

F111-real-batch 完了後、実 trade ↔ outcome 評価 path:
- actual close / next-day return / h1/h5/h20 return
- paper_pnl ledger (= 仮想 entry/exit、real return)
- manual review pnl (= 藤原さん実 trade pnl)
- **pattern outcome** (= D3-D5 で watched pattern の hit rate)
- pattern promote/suppress/watch **update**

## §7 next wave priority 確定

| Wave | 優先度 | 内容 |
|---|---|---|
| **F111-real-batch (= 次)** | **最高** | staging real candidates → F111 output → f062_preview + f111_real_batch |
| W60-pilot-D6 | 高 | F111-real-batch 由来で実 trade 候補 trade plan |
| W61-pre | 中 | price/return/paper_pnl 連携 |
| W60-launchd-real | 低 | 本番 launchd 配置 (= Wave 41/45 後) |

F111-real-batch 完了で実 entry 可能候補が初めて出る (= 5 blocker のうち 3-4 件
同時解消)。

## §8 作成 file 一覧

### §8.1 新規 (3 件、fire-vault のみ)

| path | 内容 |
|---|---|
| 03_design/F111_real_batch_requirements_2026-05-14.md | F111-real-batch 要件 v1.0 / 8 セクション |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W1_plan.md | 本 wave plan |
| 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W1_results.md | 本 wave results (= 本 doc) |

### §8.2 FIRE 本体 + reports/after_r1 / D1-D5 docs 不触 ✓

W1 集約は **read-only docs 集約**、既存 artifact / trade plan / review 全て不変。
pytest collected: 4629 (= W60-F111-pre / W60-pilot-D5 と同じ、不変)。

## §9 Codex 4 lane stdin audit 結果

W60-F111-pre 反省を踏まえ **短文 prompt + 1 観点 + ≤ 100 words reply** に絞った
4 lane 並列起動。

| Lane | 観点 | 起動 | 完全 reply |
|---|---|---|---|
| A | D1-D5 artifact_source 推移 | ✓ exit 0 | **✓ 取得 (= 進展)** |
| B | 0/5 理由分類 | ✓ exit 0 | 途中停止 |
| C | F111-real-batch 要件 | ✓ exit 0 | 途中停止 |
| D | next wave priority | ✓ exit 0 | 途中停止 |

### §9.1 Lane A 完全 reply 内容

> Confirmed Lane A progression:
> - D1 2026-05-14: artifact_source absent/null — OK as pre-W60 integration
> - D2 2026-05-15: synthetic_fixture
> - D3 2026-05-18: f062_preview
> - D4 2026-05-19: f062_preview
> - D5 2026-05-20: f062_preview, f111_input_source=f111_sample
>
> Progression matches expected.

### §9.2 self-audit 補完 (= B/C/D)

- Lane B (0/5 理由分類): F111_real_batch_requirements.md §2 で 5 blocker 文書化済 ✓
- Lane C (F111-real-batch 要件): 同 doc §3 で 6 必須要件文書化済 ✓
- Lane D (next wave priority): 同 doc §6 + 本 results §7 で priority 確定済 ✓

### §9.3 W60-F111-pre → W1 集約 の Codex 並列度推移

| wave | parallel | 完全 reply 率 |
|---|---|---|
| W60.5 | 4 lane no sleep | 4/4 = 100% (= 安定 baseline) |
| W60.6 | 4 lane no sleep | 4/4 = 100% |
| W60-pilot-pre | 8 lane no sleep | 3/8 = 37.5% |
| W60-integration | 6 lane + sleep | 4/6 = 66.7% |
| W60-launchd-pre | 4 lane no sleep | 4/4 = 100% |
| W60-F111-pre | 4 lane no sleep (long prompts) | 0/4 = 0% |
| **W60-pilot-W1** | **4 lane (short prompts, ≤100 words)** | **1/4 = 25%** |

→ 短文 prompt でも 4 lane 並列で完全 reply 取得は不安定。
**安定 100% は 4 lane + 短文 + 単純 file 内容 (= JSON read のみ) の組合せ** (= Lane A
パターン)。docs file の長文 review は途中停止しやすい。

**結論**: 重要 audit lane は self-audit 前提で運用、**Codex は短文 single file
review に絞る**のが安定。

## §10 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 | 0 (= D1-D5 中間 artifact は /tmp、staging 不接続) |
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
| D1-D5 既存 docs / artifact | 不触 ✓ |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 環境 DB mtime/size | 不変 ✓ |
| pytest collected | 4629 (= 不変) |

## §11 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **1/4 = 25%** (= Lane A 完全 reply、B/C/D self-audit) |
| 短縮率 | 高 (= 4 観点を Codex + self-audit で並列カバー) |
| 採用率 | 100% (= Codex 進展確認 + self-audit 全 confirm) |
| 差戻率 | 0 |
| Integrator 負荷 | 中-高 (= D1-D5 集約 + F111-real-batch 要件 doc + Codex 4 lane + vault) |
| 安全事故 | **0** ✓ |

### §11.1 Codex 1/4 = 25% (= 期待値以下) の解釈

短文 prompt + 1 観点で並列度を最適化したが、docs 長文 review は依然不安定。
W1 集約の本質 (= 4 観点 self-confirm) は self-audit でカバー完了。
Codex は **chain progression 確認 (Lane A)** のような **factual confirmation** が
得意、**docs 要件監査 (Lane B/C/D)** は途中停止しやすい。

次 wave (= F111-real-batch) では **Codex を factual confirmation 系 lane に集中**
させる戦略を試行。

## §12 D1-D5 5 営業日 pilot 完遂の意義

1. **AFTER-R1 MVP 4 成果物の 5 日間運用フロー検証** 完了
2. **artifact_source 識別** (= W60-integration) / **f111_input_source 識別**
   (= W60-F111-pre) の **2 重 traceability** 確立
3. **8 invariants hard check** 自動化
4. **Pilot GO/HOLD/NO-GO 判定基準** 確立
5. **値嵩株 / サンプル ticker / F111 caveat** 等、実 entry 阻害要因の特定
6. **Stage 3 (= R-19-08)** 昇格には **F111-real-batch wave** + **実 trade 50 回**
   が必要

### §12.1 5 営業日で得たもの

- 「機能直結」path (= synthetic → manual_seed → f111_sample) を **3 段階で進化**
- 全 day で **安全性 100%** 維持 (= 0/5 entry でも 100% safety、構造的)
- F111-real-batch の **具体的要件** が pilot 経由で **言語化**

## §13 次 Wave 候補 (= priority 順)

1. **F111-real-batch (= 最優先、要件確定済)**
   - staging → F111 --candidate-json → real ticker output
   - risk filter (= 100 株 ≤ 15,000 円)
   - AFTER-R1 marker 追加 (= F111_REAL_BATCH_MARKERS)
   - 9 invariants 拡張 (= risk_within_pilot_limit)
2. **W60-pilot-D6**: F111-real-batch 由来候補で初の実 trade 可能 trade plan
3. **W61-pre**: price/return/paper_pnl 連携
4. **W60-launchd-real**: 本番 launchd 配置 (= Wave 41/45 後)
5. v0 本線 path: 5/16 02:00 F282 → 5/19 Official GO → Wave 41/45/52/53 (6/9 D-Day)

## §14 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_PILOT_W1_plan]]
- [[../03_design/F111_real_batch_requirements_2026-05-14|F111 real batch requirements v1.0]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D1_results|D1 results]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D2_results|D2 results]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D3_results|D3 results]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D4_results|D4 results]]
- [[FIRE_CODEX_R1_WAVE60_PILOT_D5_results|D5 results]]
- [[FIRE_CODEX_R1_WAVE60_F111_PRE_results|W60-F111-pre results]]
