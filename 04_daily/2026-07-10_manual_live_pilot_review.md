---
id: FIRE-pilot-D42-review-2026-07-10
phase: 本番 v0 / W60-pilot-D42 / v1.4.2 朝 pilot 5 日目 review template
priority: 高
status: pending Fujiwara input (= D42 close 後)
owner: BlueFire7777 (Fujiwara)
date: 2026-07-10 (= D42、金)
parent_trade_plan: 04_daily/2026-07-10_manual_live_pilot_trade_plan.md
demote_decision_doc: 04_daily/2026-07-10_demote_decision_9247_9628.md
---

# Manual Live Pilot — Trade Review (2026-07-10 / D42)

> 本 review は D42 close 後に Fujiwara が手動で埋める。OHLCV / signal_success / entry_success は
> Fujiwara input required (= 推測禁止)。
>
> **v1.4.2 朝 pilot 5 日目**. 9130 demote 7 日目、4404 entry 復帰 5 日目、
> 9247/9628 連続 11 日目固定化リスク継続. D41 sim A/B/C + D42 max=50 sim 完了 (= read-only analysis).

## §0 D42 actual status snapshot (= pending Fujiwara input)

- **actual_entry**: Fujiwara input required
- **9247 result**: pending (= 連続 11 日目、固定化リスク継続)
- **9628 result**: pending (= 連続 11 日目、固定化リスク継続)
- **4404 result**: pending (= v1.4.2 復帰 5 日目)
- **9130 demote 効果観察**: Fujiwara input required (= 7 日目)
- **OHLCV**: J-Quants auto-fill 待ち
- **signal_success / entry_success**: pending / NA
- **§1-§10 review items**: 各 [必須 N] Fujiwara input required

## §1 [必須 1] 入った / 見送った

| code | 入った / 見送った |
|---|---|
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= v1.4.2 復帰 5 日目、risk warning 付き) |
| 9130 | **skip 確定 (= demote 効果継続 7 日目)** ✓ |

## §2 [必須 2] 判断理由

| code | 判断理由 |
|---|---|
| 9247 | Fujiwara input required (= 連続 11 日目、固定化リスク継続) |
| 9628 | Fujiwara input required (= 同上、9247 同 sector 重複) |
| 4404 | Fujiwara input required (= v1.4.2 復帰 5 日目、risk 警告 10,665 円許容判断) |
| 9130 | 既定: demote 効果継続 7 日目、recently_seen_demoted、excluded 固定 |

## §3 [必須 3] 実際に見た銘柄

- **9247 ＴＲＥ HD** (= サービス業、entry_candidate #1、TOPIX Small 1、連続 11 日目)
- **9628 燦 HD** (= サービス業、entry_candidate #2、TOPIX Small 2、連続 11 日目)
- **4404 ミヨシ油脂** (= 食品工業、v1.4.2 entry 復帰 5 日目、⚠ risk warning)
- **9130 共栄タンカー** (= 海運業、D36 demote、D42 で excluded 7 日目)

D31-D41 と D42 を **混同しない**.

## §4 [必須 4] 板・出来高・spread + 前場/終日実績

| code | 寄値 | 高値 | 安値 | 終値 | 前日比 | 出来高 | spread 所感 | 当日方向感 |
|---|---|---|---|---|---|---|---|---|
| 9247 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 4404 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required (= risk warning 付き) | Fujiwara input required |
| 9130 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= excluded、demote 7 日目観察) | Fujiwara input required |

## §5 [必須 5] 結果メモ

Fujiwara input required (= D42 close 後)。

参考観点:
- 9247 / 9628 連続 11 日目固定化評価 (= demote 単独 候補化検討)
- 4404 v1.4.2 連続 5 日目運用検証
- 9130 demote 7 日目効果継続
- sector 多様化 (= 2 sector) 継続評価
- D43 以降の本実行 (= max_candidates 拡張 or 片方 demote) 候補化判定

## §6 [必須 6] 実 entry か skip か (= D42 actual、100 株標準)

| code | 判定 |
|---|---|
| 9247 | Fujiwara input required (= 連続 11 日目で entry 慎重) |
| 9628 | Fujiwara input required (= 同 sector 重複、9247 と片方絞り推奨) |
| 4404 | Fujiwara input required (= risk 警告許容判断後) |
| 9130 | **NO enter (= demote 済、excluded 固定)** ✓ |

## §7 [必須 7] 入った価格 or 見送り理由

| code | 価格 or 理由 |
|---|---|
| 9247 | Fujiwara input required (= 連続 11 日目固定化リスク継続) |
| 9628 | Fujiwara input required (= 同上) |
| 4404 | Fujiwara input required (= v1.4.2 連続 5 日目、risk warning 判断) |
| 9130 | 見送り (= D36 demote、D42 で excluded 7 日目) |

## §8 [必須 8] 追わない上限を超えていたか

| code | chase_limit (+2.0%) | actual high | 超過 |
|---|---|---|---|
| 9247 | 1,644 円 | Fujiwara input required | Fujiwara input required |
| 9628 | 1,418 円 | Fujiwara input required | Fujiwara input required |
| 4404 | 2,176 円 | Fujiwara input required | Fujiwara input required |
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
| 4404 | Fujiwara input required (= v1.4.2 復帰 5 日目) | Fujiwara input required |
| 9130 | Fujiwara input required (= 観察用) | **NA / demoted** ✓ |

**signal_success 判定基準注記**:
- historical review: 当日朝 advisory 利確目安への事後評価、Fujiwara 手動
- current renderer: post-open high/low の +2R / -1R hit 自動判定、推測禁止
- → 別 field / 別意味、current renderer 値で historical を **上書きしない**

## §11 D31-D41 引き継ぎ + 9130 demote / 4404 / 9247-9628 観察

- D31 (06-25): 9130 +4.89% / 連続 1
- D32-D34: 9130 連続 2-4 / 4404 watch (= 旧 v1.4.1)
- D35: 9130 連続 5 + demote sim
- D36: 9130 demote 本実行
- D37: 9130 demote 継続 / sector 集中化 / max_candidates sim
- D38: v1.4.2 初 pilot / 4404 復帰 + warning / sector 2 多様化
- D39: v1.4.2 2 日目
- D40: v1.4.2 3 日目 / 9247-9628 連続 9 日目
- D41: v1.4.2 4 日目 / 9247-9628 連続 10 日目 / demote sim 完了
- **D42 (07-10): v1.4.2 5 日目 / 9130 demote 7 日目 / 4404 連続 5 日目 / 9247-9628 連続 11 日目 / max=50 sim 完了** ← 本日

連続 counter (= demote 実行日 D36 を 1 とする):
- 9130 demote: D36 (1) / D37 (2) / D38 (3) / D39 (4) / D40 (5) / D41 (6) / D42 (7) = **7 日連続**
- 4404 v1.4.2 entry: D38 (1) / D39 (2) / D40 (3) / D41 (4) / D42 (5) = **5 日連続**
- 9247 / 9628 entry: D32 から 11 日連続 (= 9130 閾値の 2.2 倍超)

正式 recently_seen_codes (= 8 件版、D36 以降継続):
`9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798` (= demoted_count: 8)

D42 判定ルール 明記:
- 9130 再 entry → **CRITICAL** (= D42 完了扱い禁止 / recently_seen logic / consumer mapping 調査)
- 9130 watch 戻り → **HIGH** (= D42 完了扱い禁止または HOLD / watch 戻り理由調査)
- 4404 watch 戻り → **HIGH** (= v1.4.2 risk cap 撤廃 consumer/MD 側崩れ可能性)
- 低流動性銘柄 entry 浮上 → **HIGH** (= HOLD または NO-GO)

## §12 9247 / 9628 demote decision matrix サマリ

**D41 sim (max=20、recently_seen 拡張)**:
- sim A (9247 demote) → entry 2 件 / sector 2 / 慎重 OK
- sim B (9628 demote) → entry 2 件 / sector 2 / 慎重 OK
- sim C (両方 demote) → entry 1 件 / sector 1 / **非推奨**

**D42 sim (max=50、recently_seen 同 8 件)**:
- entry 14 件 / sector 7 / letter-suffix 0 / 非 100 株 0 / liquidity_fail 0 ✓
- risk_yen_warning 浮上 4 件 (4404, 3479, 4540, 8057)

**D43 以降の本実行候補 (= HQ approve 後の別 wave)**:
- 最推奨 A: max_candidates 20→50 baseline 化 (= demote なし) ★★★★★
- 次点 B/C: A + 9247 or 9628 単独 demote ★★★★
- 慎重 D/E: max=20 維持 + 単独 demote ★★★
- **非推奨 F**: 両方同時 demote (= max=20) ★

→ 詳細: `~/fire-vault/04_daily/2026-07-10_demote_decision_9247_9628.md`

## §13 v1.4.2 継続運用検証項目

D42 close 後に Fujiwara が確認:
1. 4404 risk warning が判断材料として機能継続したか (= 5 日目)
2. 9247/9628 連続 11 日目で entry 判断が安定しているか
3. demote 本実行 (= max_candidates 拡張 or sim A/B) 候補化判断
4. max_candidates 拡張 baseline 化の HQ approve タイミング

## §14 safety footer

- 本 review は **手動記録用 template** (= read-only)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0
- F282 plist 3 file 別個に safety final 確認:
  - production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
  - smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
  - LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 0
- D31-D42 fixture / docs 分離維持
- 9247/9628 demote sim (D41 + D42) は read-only analysis、本実行ではない
- 両方同時 demote は原則非推奨
