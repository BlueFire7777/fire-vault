---
id: FIRE-pilot-D33-review-2026-06-29
phase: 本番 v0 / W60-pilot-D33 / v1.4.1 review template (= 10 項目構造)
priority: 高
status: pending Fujiwara input (= D33 close 後に埋める)
owner: BlueFire7777 (Fujiwara)
date: 2026-06-29 (= D33、月)
parent_trade_plan: 04_daily/2026-06-29_manual_live_pilot_trade_plan.md
---

# Manual Live Pilot — Trade Review (2026-06-29 / D33)

> 本 review は D33 close 後に Fujiwara が手動で埋める。OHLCV / signal_success / entry_success は
> Fujiwara input required (= 推測禁止)。
>
> 朝 advisory 推奨 4 候補 (= 9130 / 9247 / 9628 / 4404) のみを対象。
> 9130 は no-new-chase 既定 (= D33 連続 3 日目)。

## §0 D33 actual status snapshot (= pending Fujiwara input)

本 wave 時点で Fujiwara actual input は全項目未提供。推測で埋めず以下のまま保持:

- **actual_entry**: Fujiwara input required
- **entry/skip decision**: Fujiwara input required
- **9247 result**: pending (= 100 株 entry したか / skip したか、Fujiwara 判断待ち)
- **9628 result**: pending (= 同上)
- **9130 no-new-chase observed**: Fujiwara input required
  (= pre-open 運用 override で skip 確定済、actual high / chase_limit 超過観測は Fujiwara 確認待ち)
- **4404 watch observed**: Fujiwara input required (= entry 不可 既定、observation のみ)
- **OHLCV**: J-Quants auto-fill 待ち (= 当 wave では取得しない、後続 wave で J-Quants API か
  Fujiwara 手動入力で補完)
- **signal_success**: pending (= OHLCV 提供後に判定可能)
- **entry_success**: pending / NA (= Fujiwara 実 entry / skip 判定後)
- **review §1-§10**: 各 [必須 N] item とも Fujiwara input required を保持 (= D33 close 後に更新)

→ 本 wave は **handoff correction** のみ (= demote 日程補正 + D33 candidate 整理)、
**actual fill は別 wave** で Fujiwara input + J-Quants auto-fill 後に実施.

## §1 [必須 1] 入った / 見送った

| code | 入った / 見送った |
|---|---|
| 9130 | Fujiwara input required (= 既定 no-new-chase、skip 想定) |
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= watch 既定、skip 想定) |

## §2 [必須 2] 判断理由 (= Fujiwara 本人の正式見送り / entry 理由)

| code | 判断理由 |
|---|---|
| 9130 | Fujiwara input required (= 既定理由: no-new-chase / D33 連続 3 日目 / D34 = 4 連続確認 / D35 = 5 連続 warning + demote simulation 候補 / D36 = 必要なら demote 本実行候補) |
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= 既定理由: 100 株 risk_yen 上限超 / 非 100 株調整不採用) |

## §3 [必須 3] 実際に見た銘柄 (= D33 朝 advisory 推奨 4 件のみ)

- **9130 共栄タンカー** (= 海運業 / タンカー、no-new-chase、連続 3 日目)
- **9247 ＴＲＥ HD** (= サービス業 / リサイクル、entry_candidate #1)
- **9628 燦 HD** (= サービス業 / 葬祭、entry_candidate #2)
- **4404 ミヨシ油脂** (= 食品工業 / 油脂、watch 降格、entry 不可)

D31 (= 9130/331A0/4389) や D32 (= 9247/9628/9130/4404) と D33 candidate を **混同しない**.
D33 review は **D33 朝 advisory 推奨 4 件のみ** が対象 (= D31 過去 actual や D32 過去 actual を
本 review に持ち込まない).

## §4 [必須 4] 板・出来高・spread + 前場/終日実績

| code | 寄値 | 高値 | 安値 | 終値 | 前日終値比 | 出来高 | spread 所感 | 当日方向感 |
|---|---|---|---|---|---|---|---|---|
| 9130 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= 観察のみ、no-new-chase 既定) |
| 9247 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required (= D32 で出来高薄 / spread 大の懸念あり) | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 4404 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= 観察のみ、watch 降格、entry 不可) | Fujiwara input required |

## §5 [必須 5] 結果メモ

Fujiwara input required (= D33 close 後)。

参考観点:
- 銘柄選定 signal の妥当性 (= F111 v1.4.1 出力が合っていたか)
- 9130 連続 3 日目の signal 観察 (= chase_limit 超過観測 / 過熱判定)
- 9247 / 9628 の出来高 / spread / 約定リスク
- 4404 100 株 risk 上限超で entry 不可の妥当性

## §6 [必須 6] 実 entry か skip か (= D33 actual、100 株標準)

| code | 判定 |
|---|---|
| 9130 | **NO enter (= no-new-chase 既定)** ✓ 既定 + Fujiwara 確認 input pending |
| 9247 | Fujiwara input required (= 100 株 entry 想定または skip) |
| 9628 | Fujiwara input required (= 100 株 entry 想定または skip) |
| 4404 | **NO enter (= watch 降格、100 株 risk_yen 上限超、entry 不可)** ✓ 既定 + Fujiwara 確認 input pending |

## §7 [必須 7] 入った価格 or 見送り理由

| code | 価格 or 理由 |
|---|---|
| 9130 | 見送り (= no-new-chase、D33 連続 3 日目、D34 = 4 連続確認 / D35 = 5 連続 warning + demote simulation 候補 / D36 = 必要なら demote 本実行候補) |
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | 見送り (= watch 降格、100 株 risk_yen 上限超) |

## §8 [必須 8] 追わない上限を超えていたか

| code | chase_limit (= +2.0%) | actual high | 超過 |
|---|---|---|---|
| 9130 | 1,438 円 | Fujiwara input required | Fujiwara input required (= 観察、超過なら post-open chase_limit_exceeded=True) |
| 9247 | 1,644 円 | Fujiwara input required | Fujiwara input required |
| 9628 | 1,418 円 | Fujiwara input required | Fujiwara input required |
| 4404 | 2,176 円 | Fujiwara input required | (= entry 不可、参考値) |

## §9 [必須 9] pre-open 判断か post-open 判断か

| code | 判断 phase |
|---|---|
| 9130 | **pre-open 判断** (= 朝 advisory 段階で no-new-chase 確定、D33 連続 3 日目で過熱判定) |
| 9247 | Fujiwara input required (= pre-open or post-open) |
| 9628 | Fujiwara input required |
| 4404 | **pre-open 判断** (= watch 降格、entry 不可、朝 advisory 段階で確定) |

## §10 [必須 10] signal_success と entry_success の区別

| code | signal_success (= 値動きベース) | entry_success (= Fujiwara 判断ベース) |
|---|---|---|
| 9130 | Fujiwara input required (= historical review、当日 chase_limit 1,438 超過 + 利確目安到達判定) | **skipped** (= no-new-chase 既定で 実 entry なし) ✓ |
| 9247 | Fujiwara input required (= 当日 +2R take_profit 1,773 到達したか) | Fujiwara input required (= 100 株 entry したか / skip したか) |
| 9628 | Fujiwara input required (= +2R 1,529 到達か) | Fujiwara input required |
| 4404 | Fujiwara input required (= 観察用、entry 不可) | **NA / skipped** (= entry 不可、watch 既定) ✓ |

**signal_success 判定基準注記** (= historical vs current renderer):

- **historical review** (= 本 file): 当日朝 advisory で出した利確目安 / 追わない上限 / 損切り目安に
  対する事後評価。Fujiwara が D33 close 後に手動記録 (= OHLCV / 板 / iSPEED 確認)。
  例: D31 9130 = chase_limit 1,440 観測上限超過 → signal_success=YES。
- **current renderer** (= compute_signal_success in `_v1_4_1_post_open_update.py`): post-open
  observation の high / low が candidate の +2R take_profit / -1R stop_loss を hit したかで
  自動判定。obs 未提供 / partial / 両方 hit → pending。推測禁止。

→ 両者は **別 field / 別意味** として記録。current renderer 値で historical review を
**上書きしない** (= post-open update wrapper は historical を変更しない)。

## §11 D31/D32 からの引き継ぎ / 9130 連続観察 (= demote 日程補正版)

- D31 (= 2026-06-25): 9130 reality entry (= signal_success +4.89%、actual 未 entry)、**連続 1 日目**
- D32 (= 2026-06-26): 9130 entry_candidate → no-new-chase、**連続 2 日目**
- D33 (= 2026-06-29): 9130 entry_candidate → **no-new-chase 維持**、**連続 3 日目** (= 本日)
- **D34 (= 2026-06-30 予定): 4 連続確認** (= 9130 が再び F111 entry 上位に出るかを確認、
  pre-open phase で運用 override 継続)
- **D35 (= 2026-07-01 予定): 5 連続 warning + demote simulation 候補** (= recently_seen_codes に
  追加した F111 を別途実行し、role 変化 / 候補 churn を確認)
- **D36 (= 2026-07-02 予定): 必要なら demote 本実行候補** (= D35 sim 結果 + Fujiwara 承認後に
  recently_seen_codes に 9130 を正式追加、本番 advisory pipeline へ反映)

注: 連続日数は 9130 が F111 entry 上位 (= 必須 21 fields で candidate_role=entry_candidate) に
**出る限り**カウントを継続。1 日でも entry 上位から外れたら counter リセット (= 通常パターン復帰).

## §12 D33 actual_flow input (= post-open wrapper 用)

template: `/tmp/fire_d33_prep/d33_actual_flow_template.json`

Fujiwara が D33 場中・前場後に手動入力する用の placeholder。推測禁止、未観測は null のまま。

post-open update wrapper 実行コマンド (= trade plan §6.2 参照):

```bash
.venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
  --input-json /tmp/fire_d33_prep/d33_consumer_payload.json \
  --actual-flow-json /tmp/fire_d33_prep/actual_flow_d33.json \
  --output-json /tmp/fire_d33_prep/d33_post_open_payload.json \
  --strict
```

## §13 safety footer (= 本 review の運用前提)

- 本 review は **手動記録用 template** (= read-only)
- **auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信は禁止**
- **DB write 0 / API 0 / token 0 / launchctl 0**
- F282 production plist と smoke plist を別 file として safety final 確認
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 0
- D31/D32/D33 を fixture / docs で分離維持
