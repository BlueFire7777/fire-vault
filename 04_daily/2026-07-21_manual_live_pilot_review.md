---
id: FIRE-pilot-D48-review-2026-07-21
phase: 本番 v0 / W60-pilot-D48 / 海の日明け / top-n=100 + max=50 baseline 2 日目 review template
priority: 高
status: pending Fujiwara input (= D48 close 後)
owner: BlueFire7777 (Fujiwara)
date: 2026-07-21 (= D48、火、海の日明け)
parent_trade_plan: 04_daily/2026-07-21_manual_live_pilot_trade_plan.md
---

# Manual Live Pilot — Trade Review (2026-07-21 / D48)

> 本 review は D48 close 後に Fujiwara が手動で埋める。
> OHLCV / signal_success / entry_success は推測禁止。
>
> **海の日明け、top-n=100 + max=50 baseline 2 日目**.
> 9130 demote 13 日目、4404 entry 復帰 11 日目、9247/9628 連続 17 日目 (= fixed-candidate HIGH).

## §0 actual status snapshot (= pending Fujiwara input)
- actual_entry: pending
- 9247 / 9628 / 4404 / 3089 / 9633 result: pending
- 9130 demote 効果観察: pending (= 13 日目、4 営業日跨ぎ)
- OHLCV: J-Quants auto-fill 待ち
- signal_success / entry_success: pending / NA

## §1-§10 review items (= 10 項目構造、Fujiwara 手動入力)

[省略: D47 review template と同形式、top 5 + 9130 行]

## §11 D31-D47 引き継ぎ + D48 海の日明け baseline 継続

連続 counter:
- 9130 demote: D36(1) → D48(13) = **13 日連続** (= 海の日跨ぎ)
- 4404 v1.4.2 entry: D38(1) → D48(11) = **11 日連続**
- 9247 / 9628 entry: D32 から **17 日連続** (= 4 営業日跨ぎでも継続、強い固定化)

baseline 適用状況:
- top-n=100 default = ✓ (= signal_persistence dry-run smoke confirmed)
- max_candidates=50 default = ✓ (= F111 cli=1.3.0)
- policy_version=1.4.2 = ✓
- staging signals 更新: 無 (= D47 から不変、universe 拡張未発現)

正式 recently_seen_codes (= 8 件版): `9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798`

D48 判定ルール:
- 9130 再 entry → CRITICAL
- 9130 watch 戻り → HIGH
- 4404 watch 戻り → HIGH (= 本 D48 では entry 維持確認済)
- 低流動性 / letter-suffix / 非 100 株 entry → HIGH
- 9247/9628 連続 18 日 (= D49) → fixed-candidate HIGH 継続

## §12 staging signals 状態 + universe 拡張未発現の記録

| 項目 | 値 |
|---|---|
| staging signals latest base_date | 2026-05-13 |
| latest count | 35 件 |
| D47 → D48 更新 | 無 |
| signal_persistence dry-run | top_n=100, 109 件 |
| F111 raw | 35 件 (= D47 と同等) |
| universe 拡張効果 | **未発現** (= staging signals 未更新による) |
| 必要承認 | HQ_APPROVE_STAGING_SIGNAL_PERSISTENCE_WRITE |
| 本 wave 範囲 | read-only smoke + dry-run のみ |

## §13 D49 handoff

| 観点 | 内容 |
|---|---|
| F111 baseline | max=50 / recently_seen 8 件版継続 |
| signal_persistence baseline | top-n=100 |
| 9130 demote | 14 日目観察 |
| 4404 連続 entry | 12 日目 |
| 9247 / 9628 連続 | 18 日目 (= 固定化継続なら HQ approve 後の単独 demote 候補) |
| staging signals 書込 | HQ_APPROVE_STAGING_SIGNAL_PERSISTENCE_WRITE 起票検討 |
| theme overlay 起票 | F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 検討 |

## §14 safety footer

- 本 review は **手動記録用 template** (= read-only)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0 / plist 0 / cron 0 / workflow 0
- staging write 0 / production-develop 接続 0
- git add / commit / push / --no-verify 0
- TODO Excel 更新 / VACUUM / sudo / rm -rf 0
- F282 plist 3 file 別個に safety final 確認 (= 1772 / 1844 / 1772)
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 0
- D31-D48 fixture / docs 分離維持
- 9247/9628 demote 本実行は別 wave (= HQ approve 後)、両方同時 demote 禁止
