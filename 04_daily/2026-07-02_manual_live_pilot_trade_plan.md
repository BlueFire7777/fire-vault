---
id: FIRE-pilot-D36-trade-plan-2026-07-02
phase: 本番 v0 / W60-pilot-D36 / v1.4.1 morning advisory trade plan + 9130 demote 本実行記録
priority: 最高
status: pre-open、9130 demote 本実行完了 (= HQ approve 済)
owner: BlueFire7777 (Fujiwara)
date: 2026-07-02 (= D36、木)
parent_design: 03_design/FIRE_selection_policy_v1.4.1_post_open_markdown_2026-05-15.md
demote_execution_doc: 04_daily/2026-07-02_demote_execution_9130.md
demote_simulation_doc: 04_daily/2026-07-01_demote_simulation_9130.md
codex_lanes: 4
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-07-02 / D36)

## §1 全体方針

**v1.4.1** を正本として運用. 9130 demote 本実行完了 (= HQ approve 済、recently_seen_codes 正式追加).

- risk_yen は順位加点に**使わない**
- **100 株標準** (= 非 100 株調整絶対禁止)
- entry / watch / excluded を明確に分ける
- pre_open_score / post-open entry_priority 分離
- signal_success / entry_success 分離
- **9130 は demote 本実行完了**、excluded 固定 (= recently_seen で除外維持)
- **sector 集中化リスクあり** (= 情報通信・サービスその他 2 候補のみ)

## §2 D36 F111 runner 出力 (= 9130 正式追加後の baseline)

- input file: `/tmp/fire_d36_prep/d36_f111.json`
- selection_policy_version: `1.4.1`
- share_unit_standard: `100`
- **正式 recently_seen_codes (= 8 件版)**: `9130, 8747, 5729, 137A0, 3798, 340A0, 7991, 3489`
- demoted_count: **8** (= D35 の 7 件 + 9130)

candidate 分類 (= 20 candidates):

| role | 件数 | codes |
|---|---|---|
| **entry_candidate** | 2 | 9247, 9628 (= 全て 貸借 + TOPIX Small + PASS + lot_ok) |
| **watch_candidate** | 1 | 4404 (= standard_lot_ok=False) |
| **excluded_candidate** | 17 | demote 8 (= +9130) + liquidity FAIL 9 |

## §3 9130 demote 本実行結果

### 3.1 9130 role / reason

| field | value |
|---|---|
| 9130 role (D36) | **excluded_candidate** ✓ |
| 9130 reason_codes | `['recently_seen_demoted']` ✓ |
| 9130 candidate_role_reason | "recently_seen demoted (= 連続 rank 1 で demote 実行済)" |
| entry に残るか | **No** ✓ |
| watch に残るか | **No** ✓ (= 直接 excluded、post-open update wrapper 不要) |

→ **9130 demote 本実行成功** (= D35 simulation と完全一致)

### 3.2 D35 simulation vs D36 本実行 比較

| 観点 | D35 sim | D36 本実行 |
|---|---|---|
| 9130 role | excluded | excluded ✓ |
| entry | 9247, 9628 | 9247, 9628 ✓ |
| watch | 4404 | 4404 ✓ |
| excluded | 17 (+9130) | 17 (+9130) ✓ |
| 新規 entry 浮上 | 0 件 | 0 件 ✓ |

→ **完全一致** ✓ (= demote logic 再現性確認)

### 3.3 entry 特性 (= 低流動性銘柄混入 0 確認)

| code | name | sector | margin | scale | liquidity | lot_ok | letter |
|---|---|---|---|---|---|---|---|
| 9247 | ＴＲＥ HD | 情報通信・サービスその他 | 貸借 | TOPIX Small 1 | PASS | True | False |
| 9628 | 燦 HD | 情報通信・サービスその他 | 貸借 | TOPIX Small 2 | PASS | True | False |

→ 全 entry 適正 (= 100 株 / 貸借 / TOPIX Small / PASS) ✓

### 3.4 sector 集中化リスク

| sector | 件数 | codes |
|---|---|---|
| 情報通信・サービスその他 | **2** | 9247, 9628 |
| その他 sector | 0 | - |

→ **sector 集中化リスクあり** (= 1 sector 2 候補)

**Fujiwara 判断要**:
- 短期対応案: 9247 か 9628 のいずれか 1 件に 100 株 entry (= 同 sector 重複避け)
- 中期対応案: F111 max_candidates 拡張 sim で別 sector 候補浮上を確認 (= 別 wave)

## §4 D36 朝の注文完成形 advisory (= Fujiwara 手動発注用 reference)

### 4.1 9247 ＴＲＥ HD [運用: entry 検討可]

| field | value |
|---|---|
| entry 検討価格 | 1,612 円 |
| 指値目安 | 1,612 円 (= 寄付前 reference) |
| 追わない上限 | 1,644 円 (= +2.0%) |
| 利確目安 | 1,773 円 (= +2R = +161 円) |
| 損切り目安 | 1,531 円 (= -1R = -81 円) |
| 株数 | 100 株 (= 標準、非 100 株調整なし) |
| risk_yen | 8,060 円 (= 100 株あたり最大損失) |
| 見送り条件 | 寄付前 gap > 1,644 / 板薄 / 出来高低 / spread 拡大 / セクター逆行 / **9628 と同 sector のため同時 entry 慎重** |
| 観察項目 | 寄付前気配 / 信用倍率 / 板厚 / spread / 5MA-25MA |

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
| 見送り条件 | 寄付前 gap > 1,418 / 板薄 / 出来高低 / spread 拡大 / セクター逆行 / **9247 と同 sector のため同時 entry 慎重** |
| 観察項目 | 同上 |

## §5 watch + excluded

### 5.1 4404 ミヨシ油脂 [運用: watch 維持]

| field | value |
|---|---|
| watch 理由 | standard_lot_ok=False (= 100 株 risk_yen 10,665 円 > pilot 限度) |
| entry 不可理由 | 非 100 株調整不採用、pilot 限度超 |

### 5.2 9130 共栄タンカー [運用: **demote 本実行完了、excluded 固定**]

| field | value |
|---|---|
| role | excluded_candidate |
| reason_codes | recently_seen_demoted |
| 連続日数 counter | リセット (= 0) |
| 9130 entry 上位再浮上 | **想定外** (= D37 以降に再 entry なら CRITICAL = recently_seen logic 不具合) |

### 5.3 excluded 全 17 件

- demote 8 (= recently_seen_codes、本 D36 追加分の 9130 含む):
  8747 / 5729 / 3489 / 340A0 / 3798 / 137A0 / 7991 / **9130** (= 本 D36 新規追加)
- liquidity FAIL 9 (= 信用 / 非 TOPIX / letter-suffix / 出来高薄):
  331A0 (letter-suffix) / 4389 / 4317 / 2700 / 6149 / 288A0 (letter-suffix) /
  2981 / 8152 / 339A0 (letter-suffix)

## §6 D36 iSPEED 確認項目

1. 寄付前気配 / 気配指値 (= 9247 / 9628 のみ)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動
4. スプレッド (= ask-bid 比)
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ %
7. セクター指数の相対強弱 (= 情報通信・サービスその他 / 食品 / 運輸・物流)

## §7 D36 トレード手順

### 7.1 pre-open

1. F111 D36 確認 (= `/tmp/fire_d36_prep/d36_f111.json`)
2. consumer payload 確認 (= `/tmp/fire_d36_prep/d36_consumer_payload.json`)
3. **Markdown 確認** (= `/tmp/fire_d36_prep/d36_morning_advisory.md`)
4. demote execution doc 確認 (= `04_daily/2026-07-02_demote_execution_9130.md`)
5. iSPEED で 9247 / 9628 の寄付前気配 / 板厚 / spread / 信用倍率を確認
6. Fujiwara 最終判断:
   - 9247 / 9628: entry 検討、ただし同 sector 重複に注意
   - 4404: skip 確定 (= watch)
   - 9130: skip 確定 (= demote 済、excluded 固定)

### 7.2 寄り後 / 前場後

1. `/tmp/fire_d36_prep/d36_actual_flow_template.json` をコピーして実 actual_flow_d36.json 作成
2. OHLCV / fujiwara_decision を手動入力 (= 推測禁止)
3. post-open update wrapper 実行 (= optional、entry 候補に no-new-chase が発生した場合のみ):

```bash
.venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
  --input-json /tmp/fire_d36_prep/d36_consumer_payload.json \
  --actual-flow-json /tmp/fire_d36_prep/actual_flow_d36.json \
  --output-json /tmp/fire_d36_prep/d36_post_open_payload.json \
  --strict
```

4. post-open Markdown 生成 (= 同様)

### 7.3 close 後 (= D36 終了後)

1. review template (= `~/fire-vault/04_daily/2026-07-02_manual_live_pilot_review.md`) を埋める
2. OHLCV / signal_success / entry_success / 板 spread 所感 / 結果メモ を手動記録
3. 9247 / 9628 の actual entry 結果を historical review で記録
4. **9130 の demote 効果を historical review で確認** (= 終値 / 出来高、recently_seen の妥当性)
5. D37 朝の F111 / advisory 準備:
   - 9130 が再 entry に出ない事を確認 (= demote logic 動作確認)
   - sector 集中化 follow-up 判断 (= 別 sector 候補追加検討)

### 7.4 review 10 項目 (= D36 close 後)

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

→ 9 番 / 10 番をそれぞれ分離して記録. signal_success と entry_success は **同一行に書かない**.

### signal_success 判定基準注記 (= historical vs current renderer)
- **historical review**: 当日朝 advisory の利確目安に対する事後評価、Fujiwara 手動
- **current renderer**: post-open observation の +2R / -1R hit 自動判定、推測禁止
- → 別 field / 別意味、current renderer 値で historical を **上書きしない**

## §8 safety footer

- 本 trade plan は **手動発注用 reference** (= read-only advisory)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / launchctl 0
- 全 generators (= F111 / consumer / MD / post-open update) は read-only
- 生成 file: `/tmp/fire_d36_prep/` + vault `~/fire-vault/04_daily/2026-07-02_*.md`
- production / develop DB 接続なし、staging read-only URI mode=ro のみ
- 9130 demote 永続化は **F111 実行時の recently_seen_codes 渡し**のみ (= 永続 DB / config write なし)
- HQ approve 済 (= D36 wave 指示文で明示)

### F282 plist 分離注記 (= 3 file 別個確認)
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist`
  (size=1772 / mtime=1778592445、launchctl bootstrap 対象 source)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist`
  (size=1844 / mtime=1778769277、smoke test 用、launchctl 未投入)
- **LaunchAgents loaded plist** = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist`
  (size=1772 / mtime=1778593597、launchctl 読み込み実 file、production の copy)
- 3 file は **全て別 path / 別 file**、safety final で 3 file 別個に size + mtime 記録

## §9 D31/D32/D33/D34/D35 からの引き継ぎ + D37 以降

### 9.1 連続観察 サマリ

- D31 (06-25): 9130 reality entry +4.89% (= signal_success、連続 1)
- D32 (06-26): 9247/9628 entry → skip / 9130 no-new-chase (連続 2) / 4404 watch
- D33 (06-29): 同 + 9130 連続 3
- D34 (06-30): 同 + 9130 連続 4
- D35 (07-01): 同 + 9130 **連続 5 = warning 確定 + demote sim 候補確定**
- D36 (07-02): **9130 demote 本実行完了** (= recently_seen 正式追加、excluded 固定) ← **本日**

### 9.2 D37 以降の方針

- recently_seen_codes 8 件版 (= 9130 含む) を D37 以降の F111 baseline として継続使用
- 9130 連続 counter リセット (= 0)
- 9130 が D37+ で再 entry に浮上したら CRITICAL (= recently_seen logic 不具合)
- sector 集中化 follow-up (= Fujiwara 判断、別 sector 候補追加検討)

### 9.3 永続化 (= 別 wave)

本 wave での 9130 demote は **D36 F111 実行時の recently_seen_codes 渡し**で反映.
永続化 (= 環境変数 / config file / runner default) は別 wave で実施 (= R-01-08 + DB write 禁止維持).
