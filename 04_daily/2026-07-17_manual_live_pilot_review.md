---
id: FIRE-pilot-D47-review-2026-07-17
phase: 本番 v0 / W60-pilot-D47 / top-n=100 + max=50 baseline 初回稼働 review template
priority: 高
status: pending Fujiwara input (= D47 close 後)
owner: BlueFire7777 (Fujiwara)
date: 2026-07-17 (= D47、金)
parent_trade_plan: 04_daily/2026-07-17_manual_live_pilot_trade_plan.md
---

# Manual Live Pilot — Trade Review (2026-07-17 / D47)

> 本 review は D47 close 後に Fujiwara が手動で埋める。
> OHLCV / signal_success / entry_success は推測禁止。
>
> **top-n=100 + max=50 baseline 初回稼働**. 9130 demote 12 日目、4404 entry 復帰 10 日目、
> 9247/9628 連続 16 日目 (= fixed-candidate HIGH).

## §0 actual status snapshot (= pending Fujiwara input)

- **actual_entry**: Fujiwara input required
- 9247 / 9628 / 4404 / 3089 / 9633 result: pending
- 9130 demote 効果観察: pending (= 12 日目)
- OHLCV: J-Quants auto-fill 待ち
- signal_success / entry_success: pending / NA

## §1-§10 review items (= 10 項目構造、close 後 Fujiwara 手動入力)

[省略: 過去 D44 review template と同形式、top 5 + 9130 行で構成]

## §11 D31-D46 引き継ぎ + D47 baseline 初回稼働

連続 counter:
- 9130 demote: D36(1) → D47(12) = **12 日連続**
- 4404 v1.4.2 entry: D38(1) → D47(10) = **10 日連続**
- 9247 / 9628 entry: D32 から **16 日連続**

baseline 適用状況:
- top-n=100 default = ✓ (= signal_persistence dry-run smoke confirmed)
- max_candidates=50 default = ✓ (= F111 cli=1.3.0)
- policy_version=1.4.2 = ✓

正式 recently_seen_codes (= 8 件版): `9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798`

D47 判定ルール:
- 9130 再 entry → CRITICAL
- 9130 watch 戻り → HIGH
- 4404 watch 戻り → HIGH
- 低流動性 / letter-suffix / 非 100 株 entry → HIGH
- 9247/9628 連続 17 日 → fixed-candidate HIGH 継続 (D48 海の日明け)

## §12 baseline 適用効果評価 (= D47 観測)

| 項目 | D44 (max=50 / top-n=30 staging) | D47 (max=50 / top-n=100 baseline) | 差分 |
|---|---|---|---|
| F111 raw | 35 | 35 | 0 (= staging signals 未更新による) |
| entry | 14 | 14 | 0 |
| sector (entry) | 7 | 7 | 0 |
| 安全 gate 違反 | 0 | 0 | 0 |
| top-n baseline 設定 | 30 | **100** | +70 (= W2 baseline 化) |
| persistence dry-run | (未測定) | **109 件** | +109 (= 新 baseline 確認) |

→ **設定上の baseline 化は完了**.
→ 実 universe 拡張効果は staging signals 書込み (= 別 wave or daily cron) 後の D48 以降で発現期待.

## §13 D48 handoff (= 2026-07-21 火、海の日明け)

| 観点 | 確認内容 |
|---|---|
| F111 baseline | max=50 / recently_seen 8 件版継続 |
| signal_persistence baseline | top-n=100 (= W2 適用継続) |
| 9130 demote | 13 日目観察 (= 4 営業日跨ぎ、海の日明け) |
| 4404 連続 entry | 11 日目 |
| 9247 / 9628 連続 | 17 日目 or リセット (= 4 営業日跨ぎで自然リセット可能性) |
| staging signals 書込 | D47-D48 間で発生したか確認、書込済なら raw 拡張効果発現 |
| 新顔 entry 浮上 | 商社/不動産/金融/医薬品/素材化学/小売/建設資材 等の追加候補 |
| theme overlay 起票 | F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 検討 |

## §14 safety footer

- 本 review は **手動記録用 template** (= read-only)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0 / plist 0 / cron 0 / workflow 0
- F282 plist 3 file 別個に safety final 確認 (= 1772 / 1844 / 1772)
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 0
- D31-D47 fixture / docs 分離維持
- 9247/9628 demote 本実行は別 wave (= HQ approve 後)、両方同時 demote 非推奨
- TODO Excel 更新 0
- git add / commit / push 0
