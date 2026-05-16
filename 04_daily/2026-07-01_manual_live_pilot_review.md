---
id: FIRE-pilot-D35-review-2026-07-01
phase: 本番 v0 / W60-pilot-D35 / v1.4.1 review template (= 10 項目構造)
priority: 最高 (= 9130 5 連続 warning + D36 demote 本実行候補判定 起点)
status: pending Fujiwara input (= D35 close 後に埋める)
owner: BlueFire7777 (Fujiwara)
date: 2026-07-01 (= D35、水)
parent_trade_plan: 04_daily/2026-07-01_manual_live_pilot_trade_plan.md
demote_simulation_doc: 04_daily/2026-07-01_demote_simulation_9130.md
---

# Manual Live Pilot — Trade Review (2026-07-01 / D35)

> 本 review は D35 close 後に Fujiwara が手動で埋める。OHLCV / signal_success / entry_success は
> Fujiwara input required (= 推測禁止)。
>
> 朝 advisory 推奨 4 候補 (= 9130 / 9247 / 9628 / 4404) のみを対象。
> 9130 は no-new-chase 既定 (= **D35 連続 5 日目 = 5 連続 warning 確定**)、demote simulation 候補.

## §0 D35 actual status snapshot (= pending Fujiwara input)

本 wave 時点で Fujiwara actual input は全項目未提供:

- **actual_entry**: Fujiwara input required
- **entry/skip decision**: Fujiwara input required
- **9247 result**: pending
- **9628 result**: pending
- **9130 no-new-chase observed**: Fujiwara input required (= pre-open override で skip 確定、
  actual high / chase_limit 超過観測は Fujiwara 確認待ち)
- **4404 watch observed**: Fujiwara input required (= entry 不可既定)
- **OHLCV**: J-Quants auto-fill 待ち
- **signal_success**: pending
- **entry_success**: pending / NA
- **§1-§10 review items**: 各 [必須 N] Fujiwara input required

→ 本 wave は **pilot pre-open 準備 + demote simulation 実行 + D36 demote 本実行候補判定**.
actual fill は D35 close 後の別 wave で実施.

## §1 [必須 1] 入った / 見送った

| code | 入った / 見送った |
|---|---|
| 9130 | Fujiwara input required (= 既定 no-new-chase 5 連続、skip 想定) |
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= watch 既定、skip 想定) |

## §2 [必須 2] 判断理由

| code | 判断理由 |
|---|---|
| 9130 | Fujiwara input required (= 既定理由: no-new-chase / **D35 連続 5 日目 = 5 連続 warning 確定** / demote simulation 候補 / D36 = demote 本実行候補 HQ approve 必須) |
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= 既定理由: 100 株 risk_yen 上限超 / 非 100 株調整不採用) |

## §3 [必須 3] 実際に見た銘柄 (= D35 朝 advisory 推奨 4 件のみ)

- **9130 共栄タンカー** (= 海運業、no-new-chase、**連続 5 日目 = 5 連続 warning + demote sim 候補**)
- **9247 ＴＲＥ HD** (= サービス業、entry_candidate #1)
- **9628 燦 HD** (= サービス業、entry_candidate #2)
- **4404 ミヨシ油脂** (= 食品工業、watch 降格、entry 不可)

D31/D32/D33/D34 候補 と D35 候補を **混同しない**.

## §4 [必須 4] 板・出来高・spread + 前場/終日実績

| code | 寄値 | 高値 | 安値 | 終値 | 前日比 | 出来高 | spread 所感 | 当日方向感 |
|---|---|---|---|---|---|---|---|---|
| 9130 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= 観察のみ、5 連続 warning) |
| 9247 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 4404 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= 観察のみ、watch) | Fujiwara input required |

## §5 [必須 5] 結果メモ

Fujiwara input required (= D35 close 後)。

参考観点:
- 9130 5 連続の signal 観察 (= chase_limit 超過観測 / 過熱判定)
- 9247 / 9628 の出来高 / spread / 約定リスク
- 4404 100 株 risk 上限超で entry 不可の妥当性
- **demote simulation 結果の妥当性検証** (= sector 集中化リスクは許容か、alternate candidates 検討要か)
- **D36 demote 本実行判定** (= HQ approve するか、追加観察するか)

## §6 [必須 6] 実 entry か skip か (= D35 actual、100 株標準)

| code | 判定 |
|---|---|
| 9130 | **NO enter (= no-new-chase 5 連続 warning)** ✓ 既定 + Fujiwara 確認 input pending |
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | **NO enter (= watch 降格、100 株 risk 上限超)** ✓ 既定 |

## §7 [必須 7] 入った価格 or 見送り理由

| code | 価格 or 理由 |
|---|---|
| 9130 | 見送り (= no-new-chase / **D35 連続 5 日目 = 5 連続 warning + demote sim 候補** / D36 = demote 本実行候補 HQ approve 必須) |
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | 見送り (= watch 降格、100 株 risk 上限超) |

## §8 [必須 8] 追わない上限を超えていたか

| code | chase_limit (+2.0%) | actual high | 超過 |
|---|---|---|---|
| 9130 | 1,438 円 | Fujiwara input required | Fujiwara input required |
| 9247 | 1,644 円 | Fujiwara input required | Fujiwara input required |
| 9628 | 1,418 円 | Fujiwara input required | Fujiwara input required |
| 4404 | 2,176 円 | Fujiwara input required | (= entry 不可、参考値) |

## §9 [必須 9] pre-open 判断か post-open 判断か

| code | 判断 phase |
|---|---|
| 9130 | **pre-open 判断** (= 朝 advisory で no-new-chase 5 連続 warning 確定、demote sim 候補確定) |
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | **pre-open 判断** (= watch 降格、entry 不可) |

## §10 [必須 10] signal_success と entry_success の区別

| code | signal_success (= 値動きベース) | entry_success (= Fujiwara 判断ベース) |
|---|---|---|
| 9130 | Fujiwara input required (= historical review) | **skipped** (= no-new-chase 5 連続) ✓ |
| 9247 | Fujiwara input required | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required |
| 4404 | Fujiwara input required (= 観察用) | **NA / skipped** (= watch 既定) ✓ |

**signal_success 判定基準注記** (= historical vs current renderer):
- historical review: 当日朝 advisory 利確目安に対する事後評価、Fujiwara 手動記録
- current renderer: post-open high/low が +2R / -1R hit で自動判定、推測禁止
- → 別 field / 別意味、current renderer 値で historical を **上書きしない**

## §11 D31/D32/D33/D34 からの引き継ぎ / 9130 連続観察 (= 5 連続 warning 確定)

- D31 (06-25): 9130 reality entry +4.89% (= signal_success、actual 未 entry)、**連続 1 日目**
- D32 (06-26): 9130 entry_candidate → no-new-chase、**連続 2 日目**
- D33 (06-29): 同、**連続 3 日目**
- D34 (06-30): 同、**連続 4 日目**
- D35 (07-01): 同、**連続 5 日目 = 5 連続 warning 確定** (= 本日)
- **D36 (07-02 予定): demote 本実行候補** (= HQ approve 必須、recently_seen_codes 正式追加)

注: 連続日数は 9130 が F111 entry 上位に出る限りカウント. 1 日でも entry から外れたらリセット.

## §12 9130 demote simulation 結果 (= 本 wave 実行済)

詳細: `04_daily/2026-07-01_demote_simulation_9130.md`

**判定: D36 demote 本実行候補確定 (= 慎重判断付き)**

| 観点 | 結果 |
|---|---|
| 9130 role 変化 | entry → excluded (= demote 効果確認) ✓ |
| 新規 entry 浮上 | **0 件** (= 9247/9628 維持、安全側) ✓ |
| 100 株適性 | 全 entry 維持 ✓ |
| 流動性 | 全 entry PASS 維持 ✓ |
| sector 多様化 | 運輸 1 + サービス 2 → サービス 2 のみ (= 集中化、唯一の懸念) |
| 低流動性 entry 復帰 | なし ✓ |
| letter-suffix entry 浮上 | なし ✓ |

**D36 demote 本実行 prerequisites**:
1. HQ approve (= Fujiwara 明示承認)
2. D35 actual review (= 本 file 完成)
3. sector 集中化方針確定
4. 本番 F111 再実行 (= HQ approve 後)

## §13 D35 actual_flow input

template: `/tmp/fire_d35_prep/d35_actual_flow_template.json`

post-open update wrapper 実行コマンド (= trade plan §7.2 参照):

```bash
.venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
  --input-json /tmp/fire_d35_prep/d35_consumer_payload_baseline.json \
  --actual-flow-json /tmp/fire_d35_prep/actual_flow_d35.json \
  --output-json /tmp/fire_d35_prep/d35_post_open_payload.json \
  --strict
```

## §14 safety footer

- 本 review は **手動記録用 template** (= read-only)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / launchctl 0
- F282 production plist / smoke plist / LaunchAgents plist を **3 file 別個**に safety final 確認
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 0
- D31/D32/D33/D34/D35 を fixture / docs で分離維持
- demote simulation は **本実行ではない** (= recently_seen_codes 正式追加は HQ approve 後)
