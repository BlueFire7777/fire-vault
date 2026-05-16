---
id: FIRE-pilot-D38-trade-plan-2026-07-06
phase: 本番 v0 / W60-pilot-D38 / v1.4.2 初の朝 pilot + 4404 risk warning 復帰
priority: 最高
status: pre-open、v1.4.2 baseline 初運用
owner: BlueFire7777 (Fujiwara)
date: 2026-07-06 (= D38、月)
parent_design: 03_design/FIRE_selection_policy_v1.4.2_risk_warning_display_2026-05-16.md
demote_execution_doc: 04_daily/2026-07-02_demote_execution_9130.md (= D36)
codex_lanes: 4
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-07-06 / D38)

## §1 全体方針

**v1.4.2** を正本として初の朝 pilot. risk_yen は **表示・警告・損切り設計用**.
risk_yen を順位加点 / entry 除外 gate に使わない.

- 100 株標準、非 100 株調整禁止
- entry / watch / excluded 分離
- pre_open_score / post-open entry_priority 分離
- signal_success / entry_success 分離
- 9130 demote 効果継続 (= excluded 維持)
- **4404 が v1.4.2 で entry 復帰 (= ⚠ risk_yen_warning 付き)**

## §2 D38 F111 出力 (= v1.4.2 baseline)

- input: `/tmp/fire_d38_prep/d38_f111.json`
- selection_policy_version: **1.4.2** ✓
- recently_seen_codes (= 8 件版): `9130, 8747, 5729, 137A0, 3798, 340A0, 7991, 3489`
- demoted_count: 8

| role | 件数 | codes |
|---|---|---|
| **entry_candidate** | 3 | 9247, 9628, **4404** (= v1.4.2 で復帰) |
| watch_candidate | 0 | (= 4404 は entry へ、9130 は excluded) |
| excluded_candidate | 17 | demote 8 (= +9130) + liquidity FAIL 9 |

## §3 9130 demote 効果継続 + 4404 v1.4.2 復帰

### 3.1 9130 (= demote 効果継続)

| field | value |
|---|---|
| candidate_role | **excluded_candidate** ✓ (= D36/D37 から維持) |
| reason_codes | `['recently_seen_demoted']` |
| entry に戻るか | No ✓ |
| watch に戻るか | No ✓ |

### 3.2 4404 (= v1.4.2 risk warning)

| field | value |
|---|---|
| candidate_role | **entry_candidate** ✓ (= v1.4.2 で復帰) |
| standard_lot_ok | True |
| risk_yen_for_100_shares | 10,665 円 |
| **risk_yen_over_pilot_budget** | **True** ✓ (= ⚠ 警告フラグ) |
| reason_codes | `['risk_yen_warning']` |
| Markdown 表示 | ⚠ risk warning header + 全価格 field |

## §4 D38 朝の注文完成形 advisory

### 4.1 9247 ＴＲＥ HD [entry 検討可]

| field | value |
|---|---|
| entry 検討価格 | 1,612 円 |
| 指値目安 | 1,612 円 |
| 追わない上限 | 1,644 円 (= +2.0%) |
| 利確目安 | 1,773 円 (= +2R = +161 円) |
| 損切り目安 | 1,531 円 (= -1R = -81 円) |
| 株数 | 100 株 |
| risk_yen | 8,060 円 |
| 見送り条件 | gap > 1,644 / 板薄 / 出来高低 / spread 拡大 / 9628 と同 sector のため同時 entry 慎重 |

### 4.2 9628 燦 HD [entry 検討可]

| field | value |
|---|---|
| entry 検討価格 | 1,390 円 |
| 指値目安 | 1,390 円 |
| 追わない上限 | 1,418 円 (= +2.0%) |
| 利確目安 | 1,529 円 (= +2R = +139 円) |
| 損切り目安 | 1,320 円 (= -1R = -70 円) |
| 株数 | 100 株 |
| risk_yen | 6,950 円 |
| 見送り条件 | gap > 1,418 / 板薄 / 出来高低 / spread 拡大 / 9247 と同 sector のため同時 entry 慎重 |

### 4.3 4404 ミヨシ油脂 [⚠ risk warning 付き entry 検討可]

> ⚠ **risk warning** (= 旧 pilot 上限超過): 100 株 risk_yen = 10,665 円
>   → 損切り価格 / 許容損失 / 板 / 出来高 / spread / RR を **手動確認** してから entry 判断

| field | value |
|---|---|
| entry 検討価格 | 2,133 円 |
| 指値目安 | 2,133 円 |
| 追わない上限 | 2,176 円 (= +2.0%) |
| 利確目安 | 2,346 円 (= +2R = +213 円) |
| 損切り目安 | 2,026 円 (= -1R = -107 円) |
| 株数 | 100 株 (= 標準、非標準調整なし) |
| risk_yen | **10,665 円** ⚠ 旧 pilot 上限超過警告 |
| 見送り条件 | gap > 2,176 / 板薄 / 出来高低 / spread 拡大 / 損切り価格に納得できない場合 |

## §5 excluded 17 件

- demote 8 件 (= recently_seen_codes、9130 含む):
  9130, 8747, 5729, 3489, 340A0, 3798, 137A0, 7991
- liquidity FAIL 9 件:
  331A0 (letter-suffix) / 4389 / 4317 / 2700 / 6149 /
  288A0 (letter-suffix) / 2981 / 8152 / 339A0 (letter-suffix)

## §6 sector 集中化 / 多様化状況

| sector | 件数 | codes |
|---|---|---|
| 情報通信・サービスその他 | 2 | 9247, 9628 |
| **食品** | 1 | **4404** (= v1.4.2 復帰で新 sector 追加) |

→ **D37 比で sector 多様化進展** (= 1 sector → 2 sector). 4404 risk warning 復帰の副次効果.

## §7 D38 iSPEED 確認項目

1. 寄付前気配 / 気配指値 (= 9247 / 9628 / 4404、4404 は特に注意)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動
4. スプレッド (= ask-bid 比)
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ %
7. セクター指数の相対強弱 (= 情報通信・サービスその他 / 食品 / 運輸・物流)
8. **4404 限定**: risk_yen 10,665 円 / 損切り 2,026 円の許容損失を Fujiwara が改めて確認

## §8 D38 トレード手順

### 8.1 pre-open

1. F111 D38 確認 (= `/tmp/fire_d38_prep/d38_f111.json`、v1.4.2)
2. consumer payload 確認 (= `/tmp/fire_d38_prep/d38_consumer_payload.json`)
3. Markdown 確認 (= `/tmp/fire_d38_prep/d38_morning_advisory.md`、4404 warning 含む)
4. iSPEED で 9247 / 9628 / 4404 の寄付前気配 / 板厚 / spread / 信用倍率を確認
5. Fujiwara 最終判断:
   - 9247 / 9628: entry 検討、同 sector 重複に注意
   - **4404: entry 検討可、ただし risk_yen 10,665 円許容できるか改めて判断**
   - 9130: skip 確定 (= demote 済 excluded 固定)

### 8.2 寄り後 / 前場後

1. `/tmp/fire_d38_prep/d38_actual_flow_template.json` をコピーして actual_flow_d38.json 作成
2. OHLCV / fujiwara_decision を手動入力 (= 推測禁止)
3. post-open update wrapper 実行 (= optional)

### 8.3 close 後

1. review template (= `~/fire-vault/04_daily/2026-07-06_manual_live_pilot_review.md`) を埋める
2. **4404 v1.4.2 復帰初運用の検証** (= risk warning が判断材料として機能したか)
3. 9130 demote 効果継続確認 (= 終値 / 出来高)
4. sector 多様化効果評価

### 8.4 review 10 項目 (= D38 close 後)

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

## §9 safety footer

- 本 trade plan は **手動発注用 reference** (= read-only advisory)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / launchctl 0
- 生成 file: `/tmp/fire_d38_prep/` + vault `~/fire-vault/04_daily/2026-07-06_*.md`
- production / develop DB 接続なし、staging read-only URI mode=ro のみ

### F282 plist 分離注記 (= 3 file 別個確認)
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- **LaunchAgents loaded plist** = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
- 3 file は **全て別 path / 別 file**

## §10 D31-D37 引き継ぎ + D39 以降

### 10.1 D31-D38 履歴

- D31 (06-25): 9130 reality entry +4.89%、連続 1 日目
- D32-D34: 9130 連続 2-4 日目 (= no-new-chase)
- D35 (07-01): 9130 連続 5 = warning + demote sim
- D36 (07-02): **9130 demote 本実行完了**
- D37 (07-03): 9130 demote 効果継続成功、sector 集中化、max_candidates 多様化 sim 実施
- D38 (07-06): **v1.4.2 初の朝 pilot、4404 entry 復帰 + risk warning** ← 本日

### 10.2 D39 (= 2026-07-07 火 予定)

- F111 baseline: recently_seen 8 件版継続
- 9130 demote 長期観察
- 4404 v1.4.2 復帰の継続観察 (= 連続 entry 日数、warning フラグ動作)
- sector 多様化評価 (= 食品 sector が継続するか)
