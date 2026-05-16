---
id: FIRE-pilot-D37-review-2026-07-03
phase: 本番 v0 / W60-pilot-D37 / v1.4.1 review template + 9130 demote 効果継続確認
priority: 高
status: pending Fujiwara input (= D37 close 後)、9130 demote 効果継続成功
owner: BlueFire7777 (Fujiwara)
date: 2026-07-03 (= D37、金)
parent_trade_plan: 04_daily/2026-07-03_manual_live_pilot_trade_plan.md
sector_diversification_sim_doc: 04_daily/2026-07-03_sector_diversification_sim.md
---

# Manual Live Pilot — Trade Review (2026-07-03 / D37)

> 本 review は D37 close 後に Fujiwara が手動で埋める。OHLCV / signal_success / entry_success は
> Fujiwara input required (= 推測禁止)。
>
> **9130 demote 効果継続成功** (= D36 demote 本実行後、D37 で excluded 維持確認).
> sector 集中化継続、多様化 sim 実施済.
> 朝 advisory 推奨 3 候補 (= 9247 / 9628 / 4404) + 9130 demote 観察.

## §0 D37 actual status snapshot (= pending Fujiwara input)

本 wave 時点で Fujiwara actual input は全項目未提供:

- **actual_entry**: Fujiwara input required
- **entry/skip decision**: Fujiwara input required
- **9247 result**: pending
- **9628 result**: pending (= 同 sector 重複注意)
- **4404 watch observed**: Fujiwara input required
- **9130 demote 効果観察**: Fujiwara input required (= 終値 / 出来高、再 entry 浮上なし継続確認)
- **OHLCV**: J-Quants auto-fill 待ち
- **signal_success**: pending
- **entry_success**: pending / NA
- **§1-§10 review items**: 各 [必須 N] Fujiwara input required

→ 本 wave は **D37 pilot pre-open 準備 + 9130 demote 効果継続確認 + sector 多様化 sim**.
actual fill は別 wave.

## §1 [必須 1] 入った / 見送った

| code | 入った / 見送った |
|---|---|
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= watch、skip 想定) |
| 9130 | **skip 確定 (= demote 効果継続、excluded 固定)** ✓ |

## §2 [必須 2] 判断理由

| code | 判断理由 |
|---|---|
| 9247 | Fujiwara input required (= sector 集中化 / 同 sector 9628 重複注意) |
| 9628 | Fujiwara input required (= 同上) |
| 4404 | Fujiwara input required (= 既定: 100 株 risk 上限超、watch 維持) |
| 9130 | **既定: D36 demote 本実行後、D37 でも excluded 継続、recently_seen_demoted、entry 不可** |

## §3 [必須 3] 実際に見た銘柄

- **9247 ＴＲＥ HD** (= サービス業、entry_candidate #1、TOPIX Small 1、100 株 entry 検討可)
- **9628 燦 HD** (= サービス業、entry_candidate #2、TOPIX Small 2、9247 と同 sector 重複注意)
- **4404 ミヨシ油脂** (= 食品工業、watch 降格、100 株 risk 上限超、entry 不可)
- **9130 共栄タンカー** (= 海運業、D36 demote 本実行で excluded 固定、D37 でも継続)

D31-D36 候補と D37 候補を **混同しない**.

## §4 [必須 4] 板・出来高・spread + 前場/終日実績

| code | 寄値 | 高値 | 安値 | 終値 | 前日比 | 出来高 | spread 所感 | 当日方向感 |
|---|---|---|---|---|---|---|---|---|
| 9247 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 4404 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= watch、entry 不可) | Fujiwara input required |
| 9130 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= excluded、観察のみ、demote 効果継続) | Fujiwara input required |

## §5 [必須 5] 結果メモ

Fujiwara input required (= D37 close 後)。

参考観点:
- 9247 / 9628 の actual entry 結果 (= sector 集中化リスク継続検証)
- **9130 demote 効果継続確認** (= 終値 / 出来高、recently_seen の妥当性)
- sector 多様化 sim 結果 (= 4914 高砂香料工業 / 8699 ＨＳ HD 等の alternate 候補検討)
- D38 以降の方針 (= 同 sector 重複許容 or max_candidates 拡張 baseline 化)

## §6 [必須 6] 実 entry か skip か (= D37 actual、100 株標準)

| code | 判定 |
|---|---|
| 9247 | Fujiwara input required (= 100 株 entry 想定 or skip) |
| 9628 | Fujiwara input required (= 同上、同 sector 重複注意) |
| 4404 | **NO enter (= watch、100 株 risk 上限超)** ✓ |
| 9130 | **NO enter (= demote 済、excluded 固定)** ✓ |

## §7 [必須 7] 入った価格 or 見送り理由

| code | 価格 or 理由 |
|---|---|
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | 見送り (= watch、100 株 risk 上限超) |
| 9130 | 見送り (= D36 demote 本実行、D37 でも excluded 継続、demote 効果継続) |

## §8 [必須 8] 追わない上限を超えていたか

| code | chase_limit (+2.0%) | actual high | 超過 |
|---|---|---|---|
| 9247 | 1,644 円 | Fujiwara input required | Fujiwara input required |
| 9628 | 1,418 円 | Fujiwara input required | Fujiwara input required |
| 4404 | 2,176 円 | Fujiwara input required | (= entry 不可、参考値) |
| 9130 | 1,438 円 | Fujiwara input required | (= excluded、参考値、demote 効果観察) |

## §9 [必須 9] pre-open 判断か post-open 判断か

| code | 判断 phase |
|---|---|
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | **pre-open 判断** (= watch、entry 不可) |
| 9130 | **pre-open 判断** (= demote 効果継続、excluded 固定) |

## §10 [必須 10] signal_success と entry_success の区別

| code | signal_success | entry_success |
|---|---|---|
| 9247 | Fujiwara input required | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required |
| 4404 | Fujiwara input required | **NA / skipped** (= watch) ✓ |
| 9130 | Fujiwara input required (= 観察用) | **NA / demoted** (= demote 済) ✓ |

**signal_success 判定基準注記**:
- historical review: 当日朝 advisory 利確目安への事後評価、Fujiwara 手動
- current renderer: post-open high/low の +2R / -1R hit 自動判定、推測禁止
- → 別 field / 別意味、current renderer 値で historical を **上書きしない**

## §11 D31/D32/D33/D34/D35/D36 からの引き継ぎ + 9130 demote 効果継続

- D31 (06-25): 9130 reality entry +4.89%、連続 1 日目
- D32 (06-26): 9130 no-new-chase、連続 2 日目
- D33 (06-29): 連続 3 日目
- D34 (06-30): 連続 4 日目
- D35 (07-01): 連続 5 = warning + demote sim 候補確定
- D36 (07-02): 9130 demote 本実行完了 (= recently_seen 正式追加、excluded 固定)
- D37 (07-03): **9130 demote 効果継続成功** (= excluded 維持、再 entry 浮上なし) ← 本日

連続 counter: 0 (= リセット維持). D38 以降も継続観察.

## §12 sector 集中化リスク + 多様化 sim

D37 baseline entry 候補は **情報通信・サービスその他 2 件のみ** (= 9247 / 9628).

**sector 多様化 simulation 実施済** (= 詳細: `04_daily/2026-07-03_sector_diversification_sim.md`):

| 観点 | baseline (max=20) | sim (max=50) |
|---|---|---|
| entry 件数 | 2 | 10 |
| sector 数 | 1 | **5** (= 情報通信 + 金融 + 商社 + 不動産 + 素材) |
| 新規 entry 浮上 | - | 8 件 (= 全 100 株 / 貸借 / PASS) |
| 低流動性 entry 浮上 | - | なし ✓ |

**Fujiwara 判断要**:
- 短期: 同 sector 重複許容 or 1 件絞り
- 中期: max_candidates 拡張 baseline 化検討 (= 別 wave)

## §13 D37 actual_flow input

template: `/tmp/fire_d37_prep/d37_actual_flow_template.json`

post-open update wrapper 実行 (= optional):

```bash
.venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
  --input-json /tmp/fire_d37_prep/d37_consumer_payload.json \
  --actual-flow-json /tmp/fire_d37_prep/actual_flow_d37.json \
  --output-json /tmp/fire_d37_prep/d37_post_open_payload.json \
  --strict
```

## §14 safety footer

- 本 review は **手動記録用 template** (= read-only)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / launchctl 0
- F282 production plist (1772) / smoke plist (1844) / LaunchAgents plist (1772)
  の **3 file 別個** に safety final 確認
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 0
- D31/D32/D33/D34/D35/D36/D37 を fixture / docs で分離維持
- 9130 demote 効果は **F111 実行時 recently_seen_codes 渡し**で継続反映 (= 永続 DB write なし)
- sector 多様化 sim は **read-only analysis** (= baseline 不変、本番反映なし)
