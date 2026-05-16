---
id: FIRE-pilot-D36-review-2026-07-02
phase: 本番 v0 / W60-pilot-D36 / v1.4.1 review template (= 10 項目構造) + 9130 demote 本実行記録
priority: 最高
status: pending Fujiwara input (= D36 close 後)、9130 demote 本実行完了
owner: BlueFire7777 (Fujiwara)
date: 2026-07-02 (= D36、木)
parent_trade_plan: 04_daily/2026-07-02_manual_live_pilot_trade_plan.md
demote_execution_doc: 04_daily/2026-07-02_demote_execution_9130.md
---

# Manual Live Pilot — Trade Review (2026-07-02 / D36)

> 本 review は D36 close 後に Fujiwara が手動で埋める。OHLCV / signal_success / entry_success は
> Fujiwara input required (= 推測禁止)。
>
> **9130 demote 本実行完了** (= HQ approve 済、recently_seen_codes 正式追加).
> 朝 advisory 推奨 3 候補 (= 9247 / 9628 / 4404) のみを対象 (= 9130 は excluded 固定).

## §0 D36 actual status snapshot (= pending Fujiwara input)

本 wave 時点で Fujiwara actual input は全項目未提供:

- **actual_entry**: Fujiwara input required
- **entry/skip decision**: Fujiwara input required
- **9247 result**: pending (= 100 株 entry / skip)
- **9628 result**: pending (= 100 株 entry / skip)
- **4404 watch observed**: Fujiwara input required (= entry 不可既定)
- **9130 demote 効果観察**: Fujiwara input required (= 終値 / 出来高観察、recently_seen 妥当性検証)
- **OHLCV**: J-Quants auto-fill 待ち
- **signal_success**: pending
- **entry_success**: pending / NA
- **§1-§10 review items**: 各 [必須 N] Fujiwara input required

→ 本 wave は **D36 pilot pre-open 準備 + 9130 demote 本実行完了記録**.
actual fill は D36 close 後の別 wave で実施.

## §1 [必須 1] 入った / 見送った

| code | 入った / 見送った |
|---|---|
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= watch 既定、skip 想定) |
| 9130 | **skip 確定 (= demote 本実行で excluded 固定)** ✓ |

## §2 [必須 2] 判断理由

| code | 判断理由 |
|---|---|
| 9247 | Fujiwara input required (= sector 集中化リスク注記要) |
| 9628 | Fujiwara input required (= 同上) |
| 4404 | Fujiwara input required (= 既定: 100 株 risk 上限超 / 非 100 株調整不採用) |
| 9130 | **既定: D36 demote 本実行で recently_seen_codes 正式追加、excluded 固定、entry 不可** |

## §3 [必須 3] 実際に見た銘柄 (= D36 朝 advisory 推奨 3 件 + 9130 demote 監視)

- **9247 ＴＲＥ HD** (= サービス業、entry_candidate #1)
- **9628 燦 HD** (= サービス業、entry_candidate #2)
- **4404 ミヨシ油脂** (= 食品工業、watch 降格、entry 不可)
- **9130 共栄タンカー** (= 海運業、demote 本実行で excluded 固定、entry 候補ではない、終値 / 出来高観察のみ)

D31-D35 候補と D36 候補を **混同しない**.

## §4 [必須 4] 板・出来高・spread + 前場/終日実績

| code | 寄値 | 高値 | 安値 | 終値 | 前日比 | 出来高 | spread 所感 | 当日方向感 |
|---|---|---|---|---|---|---|---|---|
| 9247 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 4404 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= watch、entry 不可) | Fujiwara input required |
| 9130 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= excluded、観察のみ) | Fujiwara input required |

## §5 [必須 5] 結果メモ

Fujiwara input required (= D36 close 後)。

参考観点:
- 9247 / 9628 の actual entry 結果 (= sector 集中化リスクの実地検証)
- 4404 100 株 risk 上限超で entry 不可の妥当性
- **9130 demote 効果の確認** (= 終値 / 出来高、recently_seen 妥当性)
- D37 以降の sector 多様化方針 (= 別 sector 候補追加検討要否)

## §6 [必須 6] 実 entry か skip か (= D36 actual、100 株標準)

| code | 判定 |
|---|---|
| 9247 | Fujiwara input required (= 100 株 entry 想定 or skip) |
| 9628 | Fujiwara input required (= 100 株 entry 想定 or skip、同 sector 重複注意) |
| 4404 | **NO enter (= watch、100 株 risk 上限超)** ✓ |
| 9130 | **NO enter (= demote 済、excluded 固定)** ✓ |

## §7 [必須 7] 入った価格 or 見送り理由

| code | 価格 or 理由 |
|---|---|
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | 見送り (= watch、100 株 risk 上限超) |
| 9130 | 見送り (= D36 demote 本実行、recently_seen 正式追加、excluded 固定) |

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
| 9247 | Fujiwara input required (= pre-open or post-open) |
| 9628 | Fujiwara input required |
| 4404 | **pre-open 判断** (= watch、entry 不可確定) |
| 9130 | **pre-open 判断** (= demote 本実行、excluded 固定確定) |

## §10 [必須 10] signal_success と entry_success の区別

| code | signal_success | entry_success |
|---|---|---|
| 9247 | Fujiwara input required (= +2R 1,773 到達か) | Fujiwara input required |
| 9628 | Fujiwara input required (= +2R 1,529 到達か) | Fujiwara input required |
| 4404 | Fujiwara input required (= 観察用、entry 不可) | **NA / skipped** (= watch) ✓ |
| 9130 | Fujiwara input required (= 観察用、excluded 固定) | **NA / demoted** (= demote 済) ✓ |

**signal_success 判定基準注記** (= historical vs current renderer):
- historical review: 当日朝 advisory 利確目安への事後評価、Fujiwara 手動
- current renderer: post-open high/low の +2R / -1R hit 自動判定、推測禁止
- → 別 field / 別意味、current renderer 値で historical を **上書きしない**

## §11 D31/D32/D33/D34/D35 からの引き継ぎ + 9130 demote 本実行完了

- D31 (06-25): 9130 reality entry +4.89%、連続 1 日目
- D32 (06-26): 9130 entry → no-new-chase、連続 2 日目
- D33 (06-29): 同、連続 3 日目
- D34 (06-30): 同、連続 4 日目
- D35 (07-01): 同、**連続 5 日目 = warning 確定 + demote sim 候補確定**
- D36 (07-02): **9130 demote 本実行完了** (= recently_seen 正式追加、excluded 固定) ← 本日

連続 counter: D36 で **リセット** (= 0). D37 以降は demote 効果を継続観察.

## §12 sector 集中化リスク

D36 entry 候補は **情報通信・サービスその他 2 件のみ** (= 9247 / 9628).

**Fujiwara 判断要**:
- 同 sector 重複を許容して両方 entry 検討するか
- いずれか 1 件に限定して entry するか
- alternate candidates (= 別 sector) を別 wave で検討するか

## §13 9130 demote 本実行記録 (= 本 wave 完了)

詳細: `04_daily/2026-07-02_demote_execution_9130.md`

| 観点 | 結果 |
|---|---|
| HQ approve | YES (= D36 wave 指示文で明示) |
| 9130 role 変化 | entry (D35 baseline) → **excluded** (D36 本実行) ✓ |
| reason_codes | recently_seen_demoted ✓ |
| D35 sim との一致 | **完全一致** ✓ |
| 低流動性 entry 浮上 | 0 件 ✓ |
| entry 件数変化 | 3 → 2 (= 9130 のみ消失、9247/9628 維持) |
| sector 集中化 | あり (= 情報通信・サービスその他 2 件のみ) |

## §14 D36 actual_flow input (= post-open wrapper 用、optional)

template: `/tmp/fire_d36_prep/d36_actual_flow_template.json`

post-open update wrapper 実行コマンド (= 9247 / 9628 で no-new-chase 発生時のみ):

```bash
.venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
  --input-json /tmp/fire_d36_prep/d36_consumer_payload.json \
  --actual-flow-json /tmp/fire_d36_prep/actual_flow_d36.json \
  --output-json /tmp/fire_d36_prep/d36_post_open_payload.json \
  --strict
```

## §15 safety footer

- 本 review は **手動記録用 template** (= read-only)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / launchctl 0
- F282 production plist (1772) / smoke plist (1844) / LaunchAgents plist (1772)
  の **3 file 別個** に safety final 確認
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 0
- D31/D32/D33/D34/D35/D36 を fixture / docs で分離維持
- 9130 demote 本実行は **D36 F111 実行時の recently_seen_codes 渡し**のみ
  (= 永続 DB / config write なし、別 wave で永続化検討)
