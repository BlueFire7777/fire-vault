---
id: FIRE-pilot-D38-review-2026-07-06
phase: 本番 v0 / W60-pilot-D38 / v1.4.2 初の朝 pilot review
priority: 最高
status: pending Fujiwara input (= D38 close 後)、v1.4.2 初運用検証
owner: BlueFire7777 (Fujiwara)
date: 2026-07-06 (= D38、月)
parent_trade_plan: 04_daily/2026-07-06_manual_live_pilot_trade_plan.md
---

# Manual Live Pilot — Trade Review (2026-07-06 / D38)

> 本 review は D38 close 後に Fujiwara が手動で埋める。OHLCV / signal_success / entry_success は
> Fujiwara input required (= 推測禁止)。
>
> **v1.4.2 初の朝 pilot**. 4404 が entry 候補に復帰 (= ⚠ risk warning 付き).
> 朝 advisory 推奨 4 候補 (= 9247 / 9628 / **4404** / 9130 demote 観察).

## §0 D38 actual status snapshot (= pending Fujiwara input)

- **actual_entry**: Fujiwara input required
- **9247 result**: pending
- **9628 result**: pending
- **4404 result**: pending (= v1.4.2 復帰初運用、risk warning 確認要)
- **9130 demote 効果観察**: Fujiwara input required (= D38 でも excluded 継続確認)
- **OHLCV**: J-Quants auto-fill 待ち
- **signal_success / entry_success**: pending / NA
- **§1-§10 review items**: 各 [必須 N] Fujiwara input required

## §1 [必須 1] 入った / 見送った

| code | 入った / 見送った |
|---|---|
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= v1.4.2 復帰、risk warning 付き) |
| 9130 | **skip 確定 (= demote 効果継続、excluded 固定)** ✓ |

## §2 [必須 2] 判断理由

| code | 判断理由 |
|---|---|
| 9247 | Fujiwara input required (= 同 sector 重複 9628 注意) |
| 9628 | Fujiwara input required (= 同上) |
| 4404 | Fujiwara input required (= v1.4.2 復帰、risk 警告 10,665 円許容判断、損切り設計、板確認) |
| 9130 | 既定: demote 効果継続、recently_seen_demoted、excluded 固定 |

## §3 [必須 3] 実際に見た銘柄

- **9247 ＴＲＥ HD** (= サービス業、entry_candidate #1、TOPIX Small 1)
- **9628 燦 HD** (= サービス業、entry_candidate #2、TOPIX Small 2)
- **4404 ミヨシ油脂** (= 食品工業、v1.4.2 entry 復帰、⚠ risk warning 付き、scale=-)
- **9130 共栄タンカー** (= 海運業、D36 demote 本実行で excluded 固定、D38 継続)

D31-D37 と D38 を **混同しない**.

## §4 [必須 4] 板・出来高・spread + 前場/終日実績

| code | 寄値 | 高値 | 安値 | 終値 | 前日比 | 出来高 | spread 所感 | 当日方向感 |
|---|---|---|---|---|---|---|---|---|
| 9247 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 4404 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required (= risk warning 付き、特に板・spread 注意) | Fujiwara input required |
| 9130 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= excluded、demote 効果観察) | Fujiwara input required |

## §5 [必須 5] 結果メモ

Fujiwara input required (= D38 close 後)。

参考観点:
- 9247 / 9628 の actual entry 結果 (= 同 sector 重複リスク実地検証)
- **4404 v1.4.2 復帰初運用検証** (= risk warning が判断材料として機能したか)
- 9130 demote 効果継続 (= 終値 / 出来高、recently_seen 妥当性)
- sector 多様化評価 (= 食品 sector 4404 追加効果)
- D39 以降の方針

## §6 [必須 6] 実 entry か skip か (= D38 actual、100 株標準)

| code | 判定 |
|---|---|
| 9247 | Fujiwara input required (= 100 株 entry 想定 or skip) |
| 9628 | Fujiwara input required (= 同上、同 sector 重複注意) |
| 4404 | Fujiwara input required (= v1.4.2 復帰、risk 10,665 円許容判断後 entry or skip) |
| 9130 | **NO enter (= demote 済、excluded 固定)** ✓ |

## §7 [必須 7] 入った価格 or 見送り理由

| code | 価格 or 理由 |
|---|---|
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= v1.4.2 復帰、risk 警告許容したか) |
| 9130 | 見送り (= D36 demote 本実行、D38 でも excluded 継続) |

## §8 [必須 8] 追わない上限を超えていたか

| code | chase_limit (+2.0%) | actual high | 超過 |
|---|---|---|---|
| 9247 | 1,644 円 | Fujiwara input required | Fujiwara input required |
| 9628 | 1,418 円 | Fujiwara input required | Fujiwara input required |
| 4404 | 2,176 円 | Fujiwara input required | Fujiwara input required (= v1.4.2 復帰特に注意) |
| 9130 | 1,438 円 | Fujiwara input required | (= excluded、参考値) |

## §9 [必須 9] pre-open 判断か post-open 判断か

| code | 判断 phase |
|---|---|
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= risk warning は pre-open 表示済) |
| 9130 | **pre-open 判断** (= demote 効果継続、excluded 固定) |

## §10 [必須 10] signal_success と entry_success の区別

| code | signal_success | entry_success |
|---|---|---|
| 9247 | Fujiwara input required | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required |
| 4404 | Fujiwara input required (= v1.4.2 復帰初運用) | Fujiwara input required |
| 9130 | Fujiwara input required (= 観察用) | **NA / demoted** (= demote 済) ✓ |

**signal_success 判定基準注記**:
- historical review: 当日朝 advisory 利確目安への事後評価、Fujiwara 手動
- current renderer: post-open high/low の +2R / -1R hit 自動判定、推測禁止
- → 別 field / 別意味、current renderer 値で historical を **上書きしない**

## §11 D31-D37 引き継ぎ + 9130 demote / 4404 v1.4.2 復帰観察

- D31 (06-25): 9130 reality +4.89% / 連続 1 日目
- D32-D34: 9130 連続 2-4 (= no-new-chase)
- D35 (07-01): 9130 連続 5 = warning + demote sim
- D36 (07-02): **9130 demote 本実行**
- D37 (07-03): 9130 demote 効果継続、sector 集中化、max_candidates sim
- **D38 (07-06): v1.4.2 初 pilot、4404 entry 復帰 (⚠ risk warning)** ← 本日

連続 counter:
- 9130: D36 で reset 後、D37/D38 excluded 維持
- 4404: D32 から watch (D32-D37) → **D38 で v1.4.2 entry 復帰**

## §12 v1.4.2 initial pilot 検証項目

D38 close 後に Fujiwara が以下を確認:
1. 4404 risk warning 表示が判断材料として機能したか
2. risk_yen 10,665 円の許容損失を実地で確認したか
3. 損切り 2,026 円が納得できる水準か
4. v1.4.2 で 4404 を entry 候補にした判断は妥当か
5. D39 以降も同じ baseline (= v1.4.2 + risk warning) で運用継続するか

## §13 safety footer

- 本 review は **手動記録用 template** (= read-only)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / launchctl 0
- F282 production plist (1772) / smoke plist (1844) / LaunchAgents plist (1772)
  の **3 file 別個** に safety final 確認
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 0
- D31/D32/D33/D34/D35/D36/D37/D38 を fixture / docs で分離維持
