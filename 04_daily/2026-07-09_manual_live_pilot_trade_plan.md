---
id: FIRE-pilot-D41-trade-plan-2026-07-09
phase: 本番 v0 / W60-pilot-D41 / v1.4.2 朝 pilot 4 日目
priority: 高
status: pre-open、v1.4.2 継続運用 + 9247/9628 demote sim 完了
owner: BlueFire7777 (Fujiwara)
date: 2026-07-09 (= D41、木)
parent_design: 03_design/FIRE_selection_policy_v1.4.2_risk_warning_display_2026-05-16.md
demote_sim_doc: 04_daily/2026-07-09_demote_simulation_9247_9628.md
codex_lanes: 4
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-07-09 / D41)

## §1 全体方針

**v1.4.2** を正本として 4 日目の朝 pilot. risk_yen は **表示・警告・損切り設計用**.
risk_yen を順位加点 / entry 除外 gate に使わない.

- 100 株標準、非 100 株調整禁止
- entry / watch / excluded 分離
- 9130 demote 効果継続観察 (= excluded 維持、6 日目)
- 4404 v1.4.2 entry 復帰継続 (= ⚠ risk_yen_warning 付き、4 日目)
- **9247 / 9628 連続 10 日目到達 → demote sim 3 通り完了** (= 別 vault doc 参照)

## §2 D41 F111 出力 (= v1.4.2 baseline)

- input: `/tmp/fire_d41_prep/d41_f111.json`
- selection_policy_version: **1.4.2** ✓
- recently_seen_codes (= 8 件版): `9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798`
- demoted_count: 8

| role | 件数 | codes |
|---|---|---|
| **entry_candidate** | 3 | 9247, 9628, **4404** (= v1.4.2 復帰継続) |
| watch_candidate | 0 | |
| excluded_candidate | 17 | demote 8 (= +9130) + liquidity FAIL 9 |

## §3 9130 demote 効果継続 (6 日目) + 4404 v1.4.2 継続 (4 日目)

### 3.1 9130 (= demote 効果継続 6 日目: D36/D37/D38/D39/D40/D41)

| field | value |
|---|---|
| candidate_role | **excluded_candidate** ✓ |
| reason_codes | `['recently_seen_demoted']` |
| entry/watch 復帰 | No / No ✓ |
| 連続 excluded | **6 日連続** |

### 3.2 4404 (= v1.4.2 entry 復帰 4 日目)

| field | value |
|---|---|
| candidate_role | **entry_candidate** ✓ |
| standard_lot_ok | True |
| risk_yen_for_100_shares | 10,665 円 |
| **risk_yen_over_pilot_budget** | **True** ✓ (= ⚠ 警告フラグ継続) |
| reason_codes | `['risk_yen_warning']` |
| 連続 entry | **4 日連続** (= D38, D39, D40, D41) |

### 3.3 9247 / 9628 連続 10 日目到達 → demote sim 完了

| code | 連続 entry 日数 | 固定化リスク |
|---|---|---|
| 9247 | **10 日連続** (D32-D41) | **demote sim 候補化到達** (= 9130 閾値の 2 倍超) |
| 9628 | **10 日連続** (D32-D41) | 同上 |

**demote sim 3 通り完了** (= read-only analysis):

| sim | recently_seen 追加 | entry 件数 | sector | 評価 |
|---|---|---|---|---|
| A | +9247 | 2 (9628+4404) | 情報通信+食品 | **慎重 OK** |
| B | +9628 | 2 (9247+4404) | 情報通信+食品 | **慎重 OK** |
| C | +両方 | **1 (4404 only)** | 食品のみ | **推奨しない** |

→ **本実行は本 wave で行わない**. HQ approve 後の別 wave で sim A or B 検討.

詳細: `~/fire-vault/04_daily/2026-07-09_demote_simulation_9247_9628.md`

## §4 D41 朝の注文完成形 advisory

### 4.1 9247 ＴＲＥ HD [連続 10 日目、demote sim 候補化到達]

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
| 見送り条件 | gap > 1,644 / 板薄 / spread 拡大 / 9628 同 sector 重複 / **連続 10 日固定化注意** |

### 4.2 9628 燦 HD [連続 10 日目、demote sim 候補化到達]

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
| 見送り条件 | gap > 1,418 / 板薄 / spread 拡大 / 9247 同 sector 重複 / **連続 10 日固定化注意** |

### 4.3 4404 ミヨシ油脂 [⚠ risk warning、連続 4 日目]

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

- demote 8 件 (= 9130 含む): 9130 (D36 demote、6 日目) / 8747 / 5729 / 3489 / 340A0 / 3798 / 137A0 / 7991
- liquidity FAIL 9 件: 331A0 (letter-suffix) / 4389 / 4317 / 2700 / 6149 / 288A0 (letter-suffix) / 2981 / 8152 / 339A0 (letter-suffix)

## §6 sector 多様化状況

| sector | 件数 | codes |
|---|---|---|
| 情報通信・サービスその他 | 2 | 9247, 9628 |
| 食品 | 1 | 4404 |

→ **D38/D39/D40 と同じ 2 sector 多様化維持** ✓

## §7 9247 / 9628 demote sim 結果反映方針

| 観点 | 状態 | 評価 |
|---|---|---|
| 9247 連続 entry | 10 日 (D32-D41) | 9130 閾値の 2 倍超 |
| 9628 連続 entry | 10 日 (D32-D41) | 同上 |
| 9130 連続 entry | 5 日 (D31-D35) → demote 実行 (D36) | 参考: 5 日で demote 閾値到達 |
| sim A (9247 demote) | 慎重 OK | entry 2 件残、sector 2 維持 |
| sim B (9628 demote) | 慎重 OK | 同上 |
| sim C (両方 demote) | 推奨しない | entry 1 件、sector 1 のみ |

→ **本 wave では本実行しない**. D42 以降で HQ approve 後の別 wave で検討.
- sim A or B 単独 demote が安全
- sim C 両方同時は推奨しない (= entry 不足リスク)
- max_candidates 拡張 baseline 化併用検討 (= D37 で 5 sector 浮上確認済)

## §8 D41 iSPEED 確認項目

1. 寄付前気配 / 気配指値 (= 9247 / 9628 / 4404)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動
4. スプレッド
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ %
7. セクター指数の相対強弱
8. **4404 限定**: risk_yen 10,665 円 / 損切り 2,026 円 / 許容損失改めて確認
9. **9247 / 9628 10 連続**: 固定化リスク再評価、同 sector 重複は片方絞り推奨

## §9 D41 トレード手順

### 9.1 pre-open
1. F111 D41 確認、consumer payload 確認、Markdown 確認
2. demote sim 結果確認 (= 別 vault doc)
3. iSPEED で 9247 / 9628 / 4404 確認
4. Fujiwara 最終判断:
   - 9247 / 9628: entry 検討、ただし連続 10 日目 + 同 sector 重複慎重
   - 4404: entry 検討可、risk 警告許容判断
   - 9130: skip 確定

### 9.2 寄り後 / 前場後
1. actual_flow_d41.json 作成 (= template コピー)
2. OHLCV / fujiwara_decision を手動入力
3. post-open update wrapper 実行 (= optional)

### 9.3 close 後
1. review template 埋め (= 10 項目)
2. v1.4.2 4 日目運用検証
3. 9130 demote 6 日目確認
4. 9247/9628 連続日数更新 (= D42 で 11 日到達 or リセット)
5. demote 本実行候補化判定 (= D42 以降の別 wave)

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
- DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0
- 生成 file: `/tmp/fire_d41_prep/` + vault `~/fire-vault/04_daily/2026-07-09_*.md`
- 9247/9628 demote sim 3 通りは read-only analysis、本実行ではない

### F282 plist 分離注記 (= 3 file 別個確認)
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- **LaunchAgents loaded plist** = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)

## §11 D31-D40 引き継ぎ + D42 以降

- D31 (06-25): 9130 +4.89% / 連続 1
- D32-D34: 9130 連続 2-4 / 4404 watch (= 旧 v1.4.1)
- D35: 9130 連続 5 + demote sim
- D36: **9130 demote 本実行**
- D37: 9130 demote 継続 / sector 集中化 / max_candidates sim
- D38: v1.4.2 初 pilot / 4404 復帰 + warning / sector 2 多様化
- D39: v1.4.2 2 日目
- D40: v1.4.2 3 日目 / 9247-9628 連続 9 日目
- **D41 (07-09): v1.4.2 4 日目 / 9130 demote 6 日目 / 4404 連続 4 日目 / 9247-9628 連続 10 日目 / demote sim 完了** ← 本日

### D42 (= 2026-07-10 金 予定)
- F111 baseline: recently_seen 8 件版継続 (= demote sim 結果反映なし、本実行は別 wave)
- 9130 demote 7 日目観察
- 4404 連続 entry 5 日目
- **9247 / 9628 連続 11 日目** (= 固定化継続 or リセット)
- HQ approve 後の demote 本実行検討 (= sim A or B、HQ 承認後)
- 9130 再 entry → CRITICAL、4404 watch 戻り → HIGH
