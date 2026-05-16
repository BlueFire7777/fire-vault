---
id: FIRE-pilot-D40-trade-plan-2026-07-08
phase: 本番 v0 / W60-pilot-D40 / v1.4.2 朝 pilot 3 日目
priority: 高
status: pre-open、v1.4.2 継続運用 + 9247/9628 固定化リスク監視
owner: BlueFire7777 (Fujiwara)
date: 2026-07-08 (= D40、水)
parent_design: 03_design/FIRE_selection_policy_v1.4.2_risk_warning_display_2026-05-16.md
codex_lanes: 4
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-07-08 / D40)

## §1 全体方針

**v1.4.2** を正本として 3 日目の朝 pilot. risk_yen は **表示・警告・損切り設計用**.
risk_yen を順位加点 / entry 除外 gate に使わない.

- 100 株標準、非 100 株調整禁止
- entry / watch / excluded 分離
- 9130 demote 効果継続観察 (= excluded 維持、5 日目)
- 4404 v1.4.2 entry 復帰継続 (= ⚠ risk_yen_warning 付き、3 日目)
- **9247 / 9628 連続上位固定化リスク監視** (= D32 以来 9 日連続)

## §2 D40 F111 出力 (= v1.4.2 baseline)

- input: `/tmp/fire_d40_prep/d40_f111.json`
- selection_policy_version: **1.4.2** ✓
- recently_seen_codes (= 8 件版): `9130, 8747, 5729, 137A0, 3798, 340A0, 7991, 3489`
- demoted_count: 8

| role | 件数 | codes |
|---|---|---|
| **entry_candidate** | 3 | 9247, 9628, **4404** (= v1.4.2 復帰継続) |
| watch_candidate | 0 | |
| excluded_candidate | 17 | demote 8 (= +9130) + liquidity FAIL 9 |

## §3 9130 demote 効果継続 (5 日目) + 4404 v1.4.2 継続 (3 日目)

### 3.1 9130 (= demote 効果継続 5 日目: D36/D37/D38/D39/D40)

| field | value |
|---|---|
| candidate_role | **excluded_candidate** ✓ |
| reason_codes | `['recently_seen_demoted']` |
| entry/watch 復帰 | No / No ✓ |
| 連続 excluded | **5 日連続** |

### 3.2 4404 (= v1.4.2 entry 復帰 3 日目)

| field | value |
|---|---|
| candidate_role | **entry_candidate** ✓ |
| standard_lot_ok | True |
| risk_yen_for_100_shares | 10,665 円 |
| **risk_yen_over_pilot_budget** | **True** ✓ (= ⚠ 警告フラグ継続) |
| reason_codes | `['risk_yen_warning']` |
| 連続 entry | **3 日連続** (= D38, D39, D40) |

### 3.3 9247 / 9628 固定化リスク (= 連続 9 日目)

| code | 連続 entry 日数 | 固定化リスク |
|---|---|---|
| 9247 | 9 日連続 (D32-D40) | **中** (= 多角化候補は要観察、9130 demote sim 同様の pattern) |
| 9628 | 9 日連続 (D32-D40) | **中** (= 同上) |

**Fujiwara 判断要**:
- 短期: D40 当日は entry 検討継続可、ただし同 sector 重複に注意
- 中期: D41 で 10 連続到達 → demote sim 候補検討 (= 9130 と同じ閾値ルール)
- 長期: max_candidates 拡張 baseline 化で sector 多様化検討

## §4 D40 朝の注文完成形 advisory

### 4.1 9247 ＴＲＥ HD [連続 9 日目]

| field | value |
|---|---|
| entry 検討価格 | 1,612 円 |
| 指値目安 | 1,612 円 |
| 追わない上限 | 1,644 円 (+2.0%) |
| 利確目安 | 1,773 円 (+2R = +161 円) |
| 損切り目安 | 1,531 円 (-1R = -81 円) |
| 株数 | 100 株 |
| risk_yen | 8,060 円 |
| sector | 情報通信・サービスその他、TOPIX Small 1、貸借 |
| 見送り条件 | gap > 1,644 / 板薄 / spread 拡大 / 9628 同 sector 重複 / **連続上位固定化注意** |

### 4.2 9628 燦 HD [連続 9 日目]

| field | value |
|---|---|
| entry 検討価格 | 1,390 円 |
| 指値目安 | 1,390 円 |
| 追わない上限 | 1,418 円 (+2.0%) |
| 利確目安 | 1,529 円 (+2R = +139 円) |
| 損切り目安 | 1,320 円 (-1R = -70 円) |
| 株数 | 100 株 |
| risk_yen | 6,950 円 |
| sector | 情報通信・サービスその他、TOPIX Small 2、貸借 |
| 見送り条件 | gap > 1,418 / 板薄 / spread 拡大 / 9247 同 sector 重複 / **連続上位固定化注意** |

### 4.3 4404 ミヨシ油脂 [⚠ risk warning、連続 3 日目]

> ⚠ **risk warning** (= 旧 pilot 上限超過): 100 株 risk_yen = 10,665 円
>   → 損切り価格 / 許容損失 / 板 / 出来高 / spread / RR を **手動確認** してから entry

| field | value |
|---|---|
| entry 検討価格 | 2,133 円 |
| 指値目安 | 2,133 円 |
| 追わない上限 | 2,176 円 (+2.0%) |
| 利確目安 | 2,346 円 (+2R = +213 円) |
| 損切り目安 | 2,026 円 (-1R = -107 円) |
| 株数 | 100 株 |
| risk_yen | **10,665 円** ⚠ 旧 pilot 上限超過警告 |
| sector | 食品、貸借、scale=- |
| 見送り条件 | gap > 2,176 / 板薄 / spread 拡大 / 損切り価格不納得 |

## §5 excluded 17 件

- demote 8 件 (= 9130 含む): 9130 (D36 demote、5 日目) / 8747 / 5729 / 3489 / 340A0 / 3798 / 137A0 / 7991
- liquidity FAIL 9 件: 331A0 (letter-suffix) / 4389 / 4317 / 2700 / 6149 / 288A0 (letter-suffix) / 2981 / 8152 / 339A0 (letter-suffix)

## §6 sector 多様化状況

| sector | 件数 | codes |
|---|---|---|
| 情報通信・サービスその他 | 2 | 9247, 9628 |
| 食品 | 1 | 4404 |

→ **D38/D39 と同じ 2 sector 多様化維持** ✓

## §7 9247 / 9628 連続上位固定化リスク

| 観点 | 状態 | 評価 |
|---|---|---|
| 9247 連続 entry | 9 日 (D32-D40) | 9130 同様 pattern、demote 候補化リスク |
| 9628 連続 entry | 9 日 (D32-D40) | 同上 |
| 9130 連続 entry | 5 日 (D31-D35) → demote 実行 (D36) | 参考: 5 日で demote 閾値到達 |

→ **D41 で 9247/9628 が 10 日連続なら demote sim 候補化判断**.
ただし v1.4.2 では risk_yen 高い候補 (= 4404) でも entry 維持できるため、
9247/9628 を demote しても代替候補が確保できる構造になっている.

## §8 D40 iSPEED 確認項目

1. 寄付前気配 / 気配指値 (= 9247 / 9628 / 4404)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動
4. スプレッド
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ %
7. セクター指数の相対強弱
8. **4404 限定**: risk_yen 10,665 円 / 損切り 2,026 円 / 許容損失改めて確認

## §9 D40 トレード手順

### 9.1 pre-open
1. F111 D40 確認、consumer payload 確認、Markdown 確認
2. iSPEED で 9247 / 9628 / 4404 確認
3. Fujiwara 最終判断:
   - 9247 / 9628: entry 検討、ただし連続 9 日目 + 同 sector 重複慎重
   - 4404: entry 検討可、risk 警告許容判断
   - 9130: skip 確定

### 9.2 寄り後 / 前場後
1. actual_flow_d40.json 作成 (= template コピー)
2. OHLCV / fujiwara_decision を手動入力
3. post-open update wrapper 実行 (= optional)

### 9.3 close 後
1. review template 埋め (= 10 項目)
2. v1.4.2 3 日目運用検証
3. 9130 demote 5 日目確認
4. 9247/9628 連続日数更新 (= D41 で 10 日到達なら demote sim 検討)

### 9.4 review 10 項目

1. 入った / 見送った
2. 判断理由
3. 実際に見た銘柄
4. 板 / 出来高 / spread の所感
5. 結果メモ
6. 実 entry か skip か
7. 入った価格 or 見送り理由
8. 追わない上限を超えていたか
9. pre-open 判断か post-open 判断か
10. signal_success と entry_success の区別

### signal_success 判定基準注記
- historical review: 当日朝 advisory 利確目安への事後評価、Fujiwara 手動
- current renderer: post-open high/low の +2R / -1R hit 自動判定、推測禁止
- → 別 field / 別意味、current renderer 値で historical を **上書きしない**

## §10 safety footer

- 本 trade plan は **手動発注用 reference** (= read-only advisory)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / launchctl 0
- 生成 file: `/tmp/fire_d40_prep/` + vault `~/fire-vault/04_daily/2026-07-08_*.md`

### F282 plist 分離注記 (= 3 file 別個確認)
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- **LaunchAgents loaded plist** = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)

## §11 D31-D39 引き継ぎ + D41 以降

- D31 (06-25): 9130 +4.89% / 連続 1
- D32-D34: 9130 連続 2-4 / 4404 watch (= 旧 v1.4.1)
- D35: 9130 連続 5 + demote sim
- D36: **9130 demote 本実行**
- D37: 9130 demote 継続 / sector 集中化 / max_candidates sim
- D38: v1.4.2 初 pilot / 4404 復帰 + warning / sector 2 多様化
- D39: v1.4.2 2 日目
- **D40 (07-08): v1.4.2 3 日目 / 9130 demote 5 日目 / 4404 連続 3 日目 / 9247-9628 連続 9 日目** ← 本日

### D41 (= 2026-07-09 木 予定)
- F111 baseline: recently_seen 8 件版継続
- 9130 demote 6 日目観察
- 4404 連続 entry 日数 4 日目
- **9247 / 9628 連続 10 日目** → demote sim 候補検討
- sector 多様化評価
- 9130 再 entry → CRITICAL、4404 watch 戻り → HIGH
