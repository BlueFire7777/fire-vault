---
id: FIRE-pilot-D35-trade-plan-2026-07-01
phase: 本番 v0 / W60-pilot-D35 / v1.4.1 morning advisory trade plan + 9130 5 連続 warning + demote simulation 候補
priority: 最高
status: pre-open、demote simulation 実行済、D36 demote 本実行候補確定
owner: BlueFire7777 (Fujiwara)
date: 2026-07-01 (= D35、水)
parent_design: 03_design/FIRE_selection_policy_v1.4.1_post_open_markdown_2026-05-15.md
demote_simulation_doc: 04_daily/2026-07-01_demote_simulation_9130.md
codex_lanes: 4 (= pilot wave、コード変更なし)
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-07-01 / D35)

## §1 全体方針

**v1.4.1** を正本として運用. 原則:

- risk_yen は順位加点に**使わない**
- **100 株標準** (= 非 100 株調整絶対禁止)
- entry / watch / excluded を明確に分ける
- pre_open_score / post-open entry_priority 分離
- signal_success / entry_success 分離
- 追わない上限超過銘柄は no-new-chase
- **9130 は D35 = 連続 5 日目 = 5 連続 warning + demote simulation 候補確定**
- **D36 = demote 本実行候補確定** (= HQ approve 必須)

## §2 D35 F111 runner 出力 (= pre-open static snapshot、baseline)

- input file: `/tmp/fire_d35_prep/d35_f111_baseline.json`
- selection_policy_version: `1.4.1`
- share_unit_standard: `100`
- recently_seen_codes: `8747, 5729, 3489, 340A0, 3798, 137A0, 7991` (= D34 維持、**9130 未追加**)
- demoted_count: `7`

candidate 分類 (= 20 candidates):

| role | 件数 | codes |
|---|---|---|
| **entry_candidate (F111 raw)** | 3 | 9130, 9247, 9628 |
| **entry_candidate (運用 override 後)** | 2 | 9247, 9628 (= 9130 は §3.1 で no-new-chase 適用) |
| **watch_candidate** | 1 | 4404 (= standard_lot_ok=False) |
| **excluded_candidate** | 16 | 8747/5729/3489/340A0/3798/137A0/7991 (demote 7) + 331A0/4389/4317/2700/6149/288A0/2981/8152/339A0 (liquidity FAIL 9) |

## §3 運用判定 + 9130 demote simulation

### 3.1 9130 共栄タンカー の運用扱い (= **連続 5 日目 = 5 連続 warning**)

**5 連続 warning 確定**:
- D31 (06-25): 1 連続目
- D32 (06-26): 2 連続目
- D33 (06-29): 3 連続目
- D34 (06-30): 4 連続目
- D35 (07-01): **5 連続目 = warning 確定** ← 本日

**本 wave で適用済**:
- pre-open phase で post-open update wrapper を実行し、9130 を fujiwara_decision="skip" で
  entry → watch に **機械的移動済**
- override actual_flow: `/tmp/fire_d35_prep/actual_flow_d35_pre_open_override.json`
- 適用後 payload: `/tmp/fire_d35_prep/d35_payload_after_override.json`
- 再生成 Markdown: `/tmp/fire_d35_prep/d35_morning_advisory.md`

**結果**: entry=2 (= 9247, 9628) / watch=2 (= 9130, 4404) / excluded=16.

### 3.2 demote simulation 結果 (= 別 doc 詳細: 2026-07-01_demote_simulation_9130.md)

| 観点 | baseline | simulation (= 9130 追加) |
|---|---|---|
| entry 件数 | 3 (9130, 9247, 9628) | 2 (9247, 9628) |
| 9130 role | entry_candidate | **excluded_candidate** |
| 新規 entry 浮上 | - | **0 件** (= 9247/9628 維持) |
| watch | 4404 (1 件) | 4404 (1 件、維持) |
| excluded | 16 件 | 17 件 (= +9130) |
| sector 多様化 | 運輸 1 + サービス 2 | サービス 2 のみ (= **集中化**) |
| 低流動性 entry 復帰 | なし | なし ✓ |
| 100 株適性 | 全 entry True | 全 entry True ✓ |
| letter-suffix entry 浮上 | なし | なし ✓ |

→ **demote 効果確認**: 9130 が entry → excluded、新候補劣化なし、安全側
→ **唯一の懸念**: sector 集中化 (= サービス業 2 件のみ)

### 3.3 D36 demote 本実行候補判定

**判定**: **本実行候補確定** (= 慎重判断付き、HQ approve 必須)

**Prerequisites**:
1. HQ approve (= Fujiwara が "9130 recently_seen 正式追加" を明示承認)
2. D35 actual review (= D35 close 後の Fujiwara verbatim review、demote 妥当性最終検証)
3. sector 集中化方針確定 (= 許容 OR alternate candidates 検討)
4. 本番 F111 再実行 (= HQ approve 後、recently_seen に 9130 を追加した F111 を本番 baseline 化)

### 3.4 D31/D32/D33/D34/D35 比較表

| code | D31 | D32 | D33 | D34 | D35 (= 本日) | 9130 連続 |
|---|---|---|---|---|---|---|
| 9130 | reality entry (+4.89%) | entry → no-new-chase | 同 (連続 3) | 同 (連続 4) | entry → **no-new-chase 5 連続 + demote sim 候補** | **5 連続 warning** |
| 9247 | (未候補) | entry (skip) | entry | entry | entry | - |
| 9628 | (未候補) | entry (skip) | entry | entry | entry | - |
| 4404 | (未候補) | watch | watch | watch | watch | - |
| 331A0 | reality excluded | excluded | excluded | excluded | excluded | - |
| 4389 | reality watch_signal | excluded | excluded | excluded | excluded | - |
| 4317 | (未候補) | excluded | excluded | excluded | excluded | - |

## §4 D35 朝の注文完成形 advisory (= Fujiwara 手動発注用 reference)

> **重要**: 9130 は **§5 watch / demote 監視** に移動 (= pre-open 運用 override 適用済).
> §4 entry advisory は **9247 / 9628 の 2 件のみ**.

### 4.1 9247 ＴＲＥ HD [運用: entry 検討可]

| field | value |
|---|---|
| entry 検討価格 | 1,612 円 |
| 指値目安 | 1,612 円 (= 寄付前 reference) |
| 追わない上限 | 1,644 円 (= +2.0%) |
| 利確目安 | 1,773 円 (= +2R = +161 円) |
| 損切り目安 | 1,531 円 (= -1R = -81 円) |
| 株数 | 100 株 (= 標準) |
| risk_yen | 8,060 円 |
| 見送り条件 | 寄付前 gap > 1,644 / 板薄 / 出来高低 / spread 拡大 / セクター逆行 |
| 観察項目 | 寄付前気配 / 信用倍率 / 板厚 / spread / 5MA-25MA / ギャップ % |

### 4.2 9628 燦 HD [運用: entry 検討可]

| field | value |
|---|---|
| entry 検討価格 | 1,390 円 |
| 指値目安 | 1,390 円 |
| 追わない上限 | 1,418 円 (= +2.0%) |
| 利確目安 | 1,529 円 (= +2R = +139 円) |
| 損切り目安 | 1,320 円 (= -1R = -70 円) |
| 株数 | 100 株 |
| risk_yen | 6,950 円 |
| 見送り条件 | 寄付前 gap > 1,418 / 板薄 / 出来高低 / spread 拡大 / セクター逆行 |
| 観察項目 | 同上 |

## §5 watch / no-new-chase / demote 監視

### 5.1 9130 共栄タンカー [運用: no-new-chase / **連続 5 日目 = 5 連続 warning + demote simulation 候補**]

post-open update wrapper で entry → watch 機械的移動済:
- input override: `/tmp/fire_d35_prep/actual_flow_d35_pre_open_override.json`
- 移動先 payload: `/tmp/fire_d35_prep/d35_payload_after_override.json`

| field | value |
|---|---|
| chase_limit_value | 1,438 円 |
| chase_limit_exceeded | None (= 観測前) |
| no_new_chase | **True** (= 運用 override で確定) |
| signal_success | pending (= OHLCV 未提供) |
| entry_success | skipped (= Fujiwara skip 確定) |
| fujiwara_decision | skip |
| 見送り条件 | **D35 連続 5 日目 = 5 連続 warning + demote simulation 候補 / D36 = demote 本実行候補 (HQ approve 必須)** |
| demote simulation | 実行済 (= 04_daily/2026-07-01_demote_simulation_9130.md 参照) |
| 継続観察 | high が 1,438 円超過 / signal_success の事後評価 / 出来高 / spread |

### 5.2 4404 ミヨシ油脂 [運用: watch 維持、entry 不可]

| field | value |
|---|---|
| watch 理由 | standard_lot_ok=False (= 100 株 risk_yen 10,665 円 > pilot 限度) |
| entry 不可理由 | 非 100 株調整不採用、pilot 限度超 |
| signal 観察 | 翌日以降 risk 適正化 / liquidity 回復で再評価 |

## §6 D35 iSPEED 確認項目

1. 寄付前気配 / 気配指値 (= 9247 / 9628 のみ、9130 は no-new-chase で観察のみ)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動
4. スプレッド (= ask-bid 比)
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ %
7. セクター指数の相対強弱 (= 情報通信・サービスその他 / 食品 / 運輸・物流)

## §7 D35 トレード手順

### 7.1 pre-open

1. F111 baseline 確認 (= `/tmp/fire_d35_prep/d35_f111_baseline.json`)
2. demote simulation 確認 (= `/tmp/fire_d35_prep/d35_f111_demote_sim_9130.json` + comparison doc)
3. post-open updated Markdown 確認 (= `/tmp/fire_d35_prep/d35_morning_advisory.md`)
4. Fujiwara 最終判断:
   - 9130: skip 確定 (= no-new-chase / 5 連続 warning)
   - 9247: entry 検討、ただし出来高 / spread / gap 確認後
   - 9628: entry 検討、同上
   - 4404: skip 確定 (= watch)

### 7.2 寄り後 / 前場後

1. `/tmp/fire_d35_prep/d35_actual_flow_template.json` をコピーして実 actual_flow_d35.json 作成
2. OHLCV / fujiwara_decision を手動入力 (= 推測禁止)
3. post-open update wrapper 実行:

```bash
.venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
  --input-json /tmp/fire_d35_prep/d35_consumer_payload_baseline.json \
  --actual-flow-json /tmp/fire_d35_prep/actual_flow_d35.json \
  --output-json /tmp/fire_d35_prep/d35_post_open_payload.json \
  --strict
```

4. post-open Markdown 生成 (= 同様)

### 7.3 close 後 (= D35 終了後)

1. review template (= `~/fire-vault/04_daily/2026-07-01_manual_live_pilot_review.md`) を埋める
2. OHLCV / signal_success / entry_success / 板スプレッド所感 / 結果メモ を手動記録
3. 9130 5 連続実績を historical review で記録
4. **D36 朝の F111 / advisory 準備時に HQ approve 確認**
   - HQ approve **あり**: D36 で recently_seen に 9130 追加、demote 本実行
   - HQ approve **なし or 保留**: D36 baseline 通常実行、追加観察継続

### 7.4 review 10 項目 (= D35 close 後に記録)

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

→ 9 番 / 10 番をそれぞれ分離して記録、signal_success と entry_success は **同一行に書かない**.

### signal_success 判定基準注記 (= historical vs current renderer)
- **historical review**: 当日朝 advisory の利確目安 / 追わない上限 / 損切り目安に対する事後評価
- **current renderer**: post-open observation の high / low が +2R / -1R hit したか自動判定
- → 別 field / 別意味、current renderer 値で historical review を **上書きしない**

## §8 safety footer

- 本 trade plan は **手動発注用 reference** (= read-only advisory)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / launchctl 0
- 全 generators (= F111 / consumer / MD / post-open update) は read-only
- 生成 file: `/tmp/fire_d35_prep/` + vault `~/fire-vault/04_daily/2026-07-01_*.md`
- production / develop DB 接続なし、staging read-only URI mode=ro のみ

### F282 plist 分離注記 (= 3 file 別個確認)
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist`
  (size=1772 / mtime=1778592445、launchctl bootstrap 対象 source)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist`
  (size=1844 / mtime=1778769277、smoke test 用、launchctl 未投入)
- **LaunchAgents loaded plist** = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist`
  (size=1772 / mtime=1778593597、launchctl 読み込み実 file、production の copy)
- 3 file は **全て別 path / 別 file**、safety final で **3 file 全て** size + mtime 記録

## §9 D31/D32/D33/D34 からの引き継ぎ

- D31 (06-25): 9130 +4.89% (= signal_success、actual 未 entry、連続 1)
- D32 (06-26): 9247/9628 entry → skip / 9130 no-new-chase (連続 2) / 4404 watch
- D33 (06-29): 同 4 候補 + 9130 連続 3
- D34 (06-30): 同 4 候補 + 9130 連続 4
- D35 (07-01): 同 4 候補 + 9130 **連続 5 = warning 確定 + demote sim 候補確定**
- D36 (07-02 予定): **demote 本実行候補確定** (= HQ approve 必須、recently_seen 正式追加)
