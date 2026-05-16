---
id: FIRE-pilot-D33-trade-plan-2026-06-29
phase: 本番 v0 / W60-pilot-D33 / v1.4.1 morning advisory trade plan
priority: 高
status: pre-open
owner: BlueFire7777 (Fujiwara)
date: 2026-06-29 (= D33、月)
parent_design: 03_design/FIRE_selection_policy_v1.4.1_post_open_markdown_2026-05-15.md
codex_lanes: 4 (= pilot wave、コード変更なし)
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-06-29 / D33)

## §1 全体方針

**v1.4.1** を正本として運用。原則:

- risk_yen は順位加点に**使わない** (= 株数決定 / 損切り / pilot gate のみ)
- **100 株標準** (= 非 100 株調整絶対禁止)
- entry / watch / excluded を明確に分ける
- pre_open_score (= 寄付前 static) と post-open entry_priority (= 寄り後 dynamic) を分離
- signal_success (= 値動きベース) と entry_success (= Fujiwara 判断ベース) を分離
- 追わない上限超過銘柄 (= chase_limit_exceeded) は no-new-chase 扱い
- 9130 = D31 で signal_success 済、D32 で no-new-chase 既定、**D33 でも上位だが新規 entry 推奨にしない**
- 9130 連続 3 日目 → **D34 = 4 連続確認 / D35 = 5 連続 warning + demote simulation 候補 /
  D36 = 必要なら demote 本実行候補**

## §2 D33 F111 runner 出力 (= pre-open static snapshot)

- input file: `/tmp/fire_d33_prep/d33_f111.json`
- selection_policy_version: `1.4.1`
- share_unit_standard: `100`
- recently_seen_codes: `8747, 7991, 137A0, 3798, 3489, 340A0, 5729` (= D32 demote 7 件維持、9130 未追加)
- demoted_count: `7`

candidate 分類 (= 20 candidates):

| role | 件数 | codes |
|---|---|---|
| **entry_candidate (F111 raw)** | 3 | 9130, 9247, 9628 (= 全て 貸借 + standard_lot_ok=True + liquidity PASS) |
| **entry_candidate (運用 override 後)** | 2 | 9247, 9628 (= 9130 は §3.1 で no-new-chase 適用、§5.1 watch 移動) |
| **watch_candidate** | 1 | 4404 (= standard_lot_ok=False、100 株 risk_yen 上限超) |
| **excluded_candidate** | 16 | 8747/5729/3489/340A0/3798/137A0/7991 (= demote 7) + 331A0/4389/4317/2700/6149/288A0/2981/8152/339A0 (= liquidity FAIL 9) |

## §3 運用判定 (= F111 出力 + Fujiwara 運用 override)

### 3.1 9130 共栄タンカー の運用扱い (= 連続 3 日目)

F111 runner-level v1.4.1 では **entry_candidate** に分類されているが、運用上は以下:

- **新規 entry 推奨にしない** (= 連続 3 日目、過熱・固定化リスク)
- **no-new-chase 扱い** (= signal 観察のみ、demote 監視対象)
- **D34 = 4 連続確認 / D35 = 5 連続 warning + demote simulation 候補 /
  D36 = 必要なら demote 本実行候補** (= recently_seen_codes に追加検討)
- **本 wave で適用済**: pre-open phase で post-open update wrapper を実行し、9130 を
  fujiwara_decision="skip" + reason "no-new-chase (= 追わない上限...)" で entry → watch に
  **機械的移動済** (= override actual_flow: `/tmp/fire_d33_prep/actual_flow_d33_pre_open_override.json`
  / 適用後 payload: `/tmp/fire_d33_prep/d33_payload_after_override.json` /
  再生成 Markdown: `/tmp/fire_d33_prep/d33_morning_advisory.md`).
- 結果: entry=2 (= 9247, 9628) / watch=2 (= 9130, 4404) / excluded=16. 9130 は entry section /
  注文完成形 advisory に出ない (= watch + post_open_monitor のみ).

### 3.2 9247 ＴＲＥ HD (= D33 entry 候補 #1)

- sector: 情報通信・サービスその他 (= サービス業 / リサイクル分野想定)
- margin: 貸借 / scale: TOPIX Small 1
- latest_close: 1,612 円
- standard_lot_ok: True / liquidity_status: PASS
- candidate_role: **entry_candidate** (= 運用上も entry 検討可)
- D32 で entry_candidate だったが Fujiwara が出来高薄 / spread 大で skip した経緯あり →
  D33 でも同じ懸念がある場合は skip 推奨

### 3.3 9628 燦 HD (= D33 entry 候補 #2)

- sector: 情報通信・サービスその他 (= サービス業 / 葬祭分野想定)
- margin: 貸借 / scale: TOPIX Small 2
- latest_close: 1,390 円
- standard_lot_ok: True / liquidity_status: PASS
- candidate_role: **entry_candidate** (= 運用上も entry 検討可)
- D32 で 9247 と同じ理由で skip された経緯あり → D33 でも同様判断の可能性

### 3.4 4404 ミヨシ油脂 (= watch 維持)

- sector: 食品 (= 食品工業 / 油脂分野想定)
- margin: 貸借 / scale: -
- latest_close: 2,133 円
- standard_lot_ok: **False** (= 100 株 risk_yen 10,665 円 > pilot 限度)
- candidate_role: **watch_candidate** (= entry 不可)
- 非 100 株調整 (= 60-share / 非標準 lot 案など) 絶対採用しない

### 3.5 excluded 16 件 の運用判断

- demote 7 件 (= 8747/5729/3489/340A0/3798/137A0/7991): recently_seen による除外、entry に戻さない
- liquidity FAIL 9 件 (= 331A0/4389/4317/2700/6149/288A0/2981/8152/339A0): 信用 / 非 TOPIX /
  letter-suffix 新規上場 + 出来高薄 → entry 完全除外、watch にも入れない

### 3.6 D31/D32/D33 比較表

| code | D31 (2026-06-25) | D32 (2026-06-26) | D33 (2026-06-29) | 運用扱い変化 |
|---|---|---|---|---|
| 9130 | reality entry (= signal_success +4.89%) | entry_candidate → no-new-chase (連続 2 日目) | entry_candidate → **no-new-chase 維持 (連続 3 日目)** | **D34 = 4 連続確認 / D35 = 5 連続 warning + demote simulation 候補 / D36 = 必要なら demote 本実行候補** |
| 9247 | (未候補) | entry_candidate (skip) | entry_candidate | 連続 2 日目、判断は Fujiwara 場中確認 |
| 9628 | (未候補) | entry_candidate (skip) | entry_candidate | 連続 2 日目、同上 |
| 4404 | (未候補) | watch_candidate | watch_candidate | **watch 維持**、entry 不可 |
| 331A0 | reality excluded (-0.21%) | excluded | excluded | excluded 維持、信用 / letter-suffix |
| 4389 | reality watch_signal (+3.13%、未 entry) | excluded | excluded | excluded 維持、信用 / 出来高薄 |
| 4317 | (未候補) | excluded | excluded | excluded 維持 |
| demote 7 件 (= 8747〜7991) | - | recently_seen | recently_seen | demote 維持 |

## §4 D33 朝の注文完成形 advisory (= Fujiwara 手動発注用 reference)

> **重要**: 9130 は **§5 watch 候補** に移動 (= pre-open 運用 override 適用済).
> 本 §4 entry advisory は **9247 / 9628 の 2 件のみ** (= 運用上 entry 検討可な候補).
> post-open update wrapper を pre-open phase で実行済 → `/tmp/fire_d33_prep/d33_payload_after_override.json` 参照.

### 4.1 9247 ＴＲＥ HD [運用: entry 検討可]

| field | value |
|---|---|
| entry 検討価格 | 1,612 円 |
| 指値目安 | 1,612 円 (= 寄付前 reference、actual 寄値で調整) |
| 追わない上限 | 1,644 円 (= +2.0%) |
| 利確目安 | 1,773 円 (= +2R = +161 円) |
| 損切り目安 | 1,531 円 (= -1R = -81 円) |
| 株数 | 100 株 (= 標準、非 100 株調整なし) |
| risk_yen | 8,060 円 (= 100 株あたり最大損失 budget) |
| 見送り条件 | 寄付前 gap > 1,644 / 板薄 / 出来高低 / spread 拡大 / セクター逆行 |
| 観察項目 | 寄付前気配 / 信用倍率 / 板厚 / spread / 5MA-25MA / ギャップ % |

### 4.2 9628 燦 HD [運用: entry 検討可]

| field | value |
|---|---|
| entry 検討価格 | 1,390 円 |
| 指値目安 | 1,390 円 (= 寄付前 reference) |
| 追わない上限 | 1,418 円 (= +2.0%) |
| 利確目安 | 1,529 円 (= +2R = +139 円) |
| 損切り目安 | 1,320 円 (= -1R = -70 円) |
| 株数 | 100 株 (= 標準) |
| risk_yen | 6,950 円 |
| 見送り条件 | 寄付前 gap > 1,418 / 板薄 / 出来高低 / spread 拡大 / セクター逆行 |
| 観察項目 | 同上 |

## §5 watch / no-new-chase / demote 監視

### 5.1 9130 共栄タンカー [運用: no-new-chase / **新規 entry 不可**]

post-open update wrapper で entry → watch に **機械的移動済** (= pre-open 運用 override 適用):
- input override: `/tmp/fire_d33_prep/actual_flow_d33_pre_open_override.json` (= fujiwara_decision='skip', reason '追わない上限...')
- 移動先 payload: `/tmp/fire_d33_prep/d33_payload_after_override.json`

| field | value |
|---|---|
| chase_limit_value | 1,438 円 (= 1,410 × +2.0%) |
| chase_limit_exceeded | None (= 観測前、Fujiwara が actual 入力後に判定) |
| no_new_chase | **True** (= 運用 override で確定) |
| signal_success | pending (= OHLCV 未提供、推測禁止) |
| entry_success | skipped (= Fujiwara skip 確定、実 entry なし) |
| fujiwara_decision | skip |
| reason_codes | actual_flow_pending, fujiwara_skip, fujiwara_skip_chase |
| 見送り条件 | **D33 連続 3 日目、D34 = 4 連続確認 / D35 = 5 連続 warning + demote simulation 候補 / D36 = 必要なら demote 本実行候補、新規 entry 不可** |
| 継続観察 | high が 1,438 円超過 / signal_success の事後評価 / 出来高 / spread |
| demote 監視 | D34 = 4 連続確認 / D35 = 5 連続 warning + demote simulation 候補 (= recently_seen 追加 + F111 role 変化観察) / D36 = 必要なら demote 本実行候補 |

### 5.2 4404 ミヨシ油脂 [運用: watch 維持、entry 不可]

| field | value |
|---|---|
| watch 理由 | standard_lot_ok=False (= 100 株 risk_yen 10,665 円 > pilot 限度) |
| 100 株 risk | 10,665 円 |
| entry 不可理由 | 非 100 株調整不採用、pilot 限度超 |
| signal 観察 | 翌日以降 risk 適正化 / liquidity 回復で再評価 |
| demote 監視 | recently_seen 累積で除外対象になりうる |

## §6 D33 iSPEED 確認項目 (= 寄付前後 board / volume check)

1. 寄付前気配 / 気配指値 (= 9247 / 9628 のみ、9130 は no-new-chase で観察のみ)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動
4. スプレッド (= ask-bid 比)
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ %
7. セクター指数の相対強弱 (= 情報通信・サービスその他 / 食品 / 運輸・物流)

## §7 D33 トレード手順 (= pre-open / post-open / close 後)

### 7.1 pre-open (= D33 朝、寄付前)

1. F111 D33 出力 確認 (= `/tmp/fire_d33_prep/d33_f111.json`)
2. consumer payload 生成済 確認 (= `/tmp/fire_d33_prep/d33_consumer_payload.json`)
3. **pre-open Markdown 確認** (= `/tmp/fire_d33_prep/d33_morning_advisory.md`)
4. 本 trade plan を再読 (= 特に 9130 = no-new-chase、4404 = watch 維持)
5. iSPEED で 9247 / 9628 の寄付前気配 / 信用倍率 / 板厚を確認
6. Fujiwara 最終判断:
   - 9130: skip 確定 (= no-new-chase)
   - 9247: entry 検討、ただし出来高 / spread / gap 確認後
   - 9628: entry 検討、同上
   - 4404: skip 確定 (= watch)

### 7.2 寄り後 / 前場後

1. `/tmp/fire_d33_prep/d33_actual_flow_template.json` をコピーして実 actual_flow_d33.json を作成
2. 各 candidate の observed_price / high / low / open / close / volume / board_spread_status /
   fujiwara_decision / fujiwara_reason / observed_at を **手動入力**
   - **推測禁止**: 観測できなかった field は null のまま (= Fujiwara が iSPEED で確認した値のみ入力)
3. post-open update wrapper を実行:

```bash
.venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
  --input-json /tmp/fire_d33_prep/d33_consumer_payload.json \
  --actual-flow-json /tmp/fire_d33_prep/actual_flow_d33.json \
  --output-json /tmp/fire_d33_prep/d33_post_open_payload.json \
  --strict
```

4. post-open Markdown 生成:

```bash
.venv/bin/python -m scripts.jobs.run_v1_4_1_morning_advisory_markdown \
  --input-json /tmp/fire_d33_prep/d33_post_open_payload.json \
  --output-md /tmp/fire_d33_prep/d33_morning_advisory_post_open.md \
  --today 2026-06-29 \
  --strict
```

5. 9130 が actual_flow で skip + chase 理由を入れていれば、entry → watch に機械的に移動済を確認

### 7.3 close 後 (= D33 終了後)

1. review template (= `~/fire-vault/04_daily/2026-06-29_manual_live_pilot_review.md`) を埋める
2. **OHLCV** / **signal_success** / **entry_success** / **板スプレッド所感** / **結果メモ** を
   Fujiwara が手動記録
3. **historical review の signal_success** (= 当日 chase_limit 観測 + 利確目安到達判断) は
   Fujiwara が決定、current renderer の computed signal_success と **別 field** で保持
4. D34 朝の F111 / advisory 準備の参考とする
5. 9130 連続日数の更新: D33 = **3 連続目** (= 当日)。D34 close 後に再確認:
   - D34 で 4 連続 → 翌 D35 で recently_seen_codes 追加した F111 sim 実行
   - D35 で 5 連続 → demote simulation 候補 (= warning)、Fujiwara 承認確認
   - D36 で 必要なら demote 本実行候補 (= recently_seen_codes 正式追加)

## §8 safety footer (= 本 trade plan の運用前提)

- 本 trade plan は **手動発注用 reference** (= read-only advisory)
- **auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信は禁止**
- **DB write 0 / API 0 / token 0 / launchctl 0**
- F111 runner / consumer / morning advisory MD / post-open update wrapper は全て read-only
- 生成 file は `/tmp/fire_d33_prep/` 配下 (= 一時 artifact) + vault
  (`~/fire-vault/04_daily/2026-06-29_*.md`)
- production / develop DB 接続なし、staging read-only URI mode=ro のみ

### F282 plist 分離注記
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist` (= 本番 launchctl 対象)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (= smoke test 用、launchctl 未投入)
- 両者は **別ファイル**、safety final 確認では別々に size + mtime を記録

## §9 D31/D32 からの引き継ぎ

- D31 (2026-06-25): 9130 +4.89% / 331A0 -0.21% / 4389 +3.13% → 9130 = signal_success / 331A0 =
  excluded_candidate_validated / 4389 = watch_signal_success
- D32 (2026-06-26): 9247 / 9628 entry → skip (= 出来高薄 / spread 大) / 9130 no-new-chase / 4404 watch
- D33 (2026-06-29): F111 出力同じ 4 候補 + 16 excluded、運用判断は本 trade plan で確定
