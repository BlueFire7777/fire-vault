---
id: FIRE-pilot-D49-review-2026-07-22
phase: 本番 v0 / W60-pilot-D49 / W3 universe 拡張後の初回稼働 review template
priority: 高
status: pending Fujiwara input (= D49 close 後)
owner: BlueFire7777 (Fujiwara)
date: 2026-07-22 (= D49、水)
parent_trade_plan: 04_daily/2026-07-22_manual_live_pilot_trade_plan.md
---

# Manual Live Pilot — Trade Review (2026-07-22 / D49)

> 本 review は D49 close 後に Fujiwara が手動で埋める。OHLCV / signal_success / entry_success は推測禁止。
>
> **W3 universe 拡張後の初回稼働**. raw 35→50 / entry 14→20 / sector entry 7→9 確認後の運用観察.

## §0 actual status snapshot (= pending Fujiwara input)
- actual_entry: pending
- top 5 (9247/9628/4404/9008/3134) result: pending
- 9130 demote 14 日目: pending
- W3 新顔 6 件 (9417/9008/6196/7595/3134/3962) result: pending
- OHLCV: J-Quants auto-fill 待ち
- signal_success / entry_success: pending / NA

## §1-§10 review items (= 10 項目構造、Fujiwara 手動入力)

[省略: D47/D48 review template と同形式、top 5 + 9130 行]

## §11 D31-D48 引き継ぎ + D49 W3 拡張効果

連続 counter:
- 9130 demote: D36(1) → D49(14) = **14 日連続**
- 4404 v1.4.2 entry: D38(1) → D49(12) = **12 日連続**
- 9247 / 9628 entry: D32 から **18 日連続**

W3 拡張効果 (= W3 vs D49):
| 項目 | W3 post-write | D49 | 評価 |
|---|---|---|---|
| F111 raw | 50 | 50 | 維持 ✓ |
| entry | 20 | 20 | 維持 ✓ |
| sector entry | 9 | 9 | 維持 ✓ |
| 新顔 6 件 | 全 entry | 全 entry | 維持 ✓ |
| 新セクター 2 件 | 運輸物流+小売 | 運輸物流+小売 | 維持 ✓ |

→ **W3 拡張効果は D49 で完全継続** ✓

baseline 適用状況:
- top-n=100 staging 反映: ✓ (= 2026-07-22 / 109 件)
- max_candidates=50: ✓
- policy_version=1.4.2: ✓

正式 recently_seen_codes: `9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798`

D49 判定ルール:
- 9130 再 entry → CRITICAL
- 9130 watch 戻り → HIGH
- 4404 watch 戻り → HIGH
- 低流動性 / letter-suffix / 非 100 株 entry → HIGH
- 9247/9628 連続 19 日 (D50) → fixed-candidate HIGH 継続

## §12 朝判断 top 5 評価

| code | rank | 観点 | 評価 |
|---|---|---|---|
| 9247 | 1 | 主役、連続 18 日 | rank 1 維持、固定化継続 |
| 9628 | 2 | 主役、9247 同 sector | rank 2 維持、片方絞り推奨 |
| 4404 | 3 | 主役、risk warning | rank 3 維持、warning 表示のみ entry |
| 9008 | 16 | W3 新顔、運輸物流 | 新セクター、低 risk |
| 3134 | 19 | W3 新顔、小売 | 新セクター、最低 risk |

## §13 D50 (= 2026-07-23 木) handoff

| 観点 | 内容 |
|---|---|
| F111 baseline | max=50 / recently_seen 8 件版継続 |
| signal_persistence baseline | top-n=100 (= W3 staging signals 109 件で継続) |
| staging signals | latest 2026-07-22 / 109 件 (= 別 write なしなら継続) |
| 9130 demote | 15 日目観察 |
| 4404 risk warning | 13 日目 |
| 9247 / 9628 連続 | 19 日目、固定化継続なら theme overlay / 単独 demote 別 wave |
| W3 新顔継続 | 9417/9008/6196/7595/3134/3962 |
| theme overlay 起票 | F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 検討 |
| top-n=200 検証 | F111-UNIVERSE-EXPANSION-R1-W4 検討 |

## §14 safety footer

- 本 review は **手動記録用 template** (= read-only)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / staging write 0 / API 0 / token 0 / LINE 0 / launchctl 0 / plist 0 / cron 0 / workflow 0
- git add / commit / push / --no-verify 0 / TODO Excel 更新 0 / VACUUM 0 / sudo 0 / rm -rf 0
- F282 plist 3 file 別個に safety final 確認 (= 1772 / 1844 / 1772)
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 0
- 9247/9628 demote 本実行: 別 wave (= HQ approve 後)、両方同時 demote 禁止
