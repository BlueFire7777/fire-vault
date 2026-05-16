---
id: FIRE-pilot-D34-morning-summary-2026-06-30
phase: 本番 v0 / W60-pilot-D34 / v1.4.1 morning advisory summary (= iPhone コピー対応)
priority: 高
status: pre-open ready
owner: BlueFire7777 (Fujiwara)
date: 2026-06-30 (= D34、火)
---

```
[FIRE 朝 advisory v1.4.1 — D34 (2026-06-30 火) morning summary (= post-open updated by pre-open override)]

【判定】GO_CONDITIONAL
  entry raw=2 / entry safe=2 / watch=2 / excluded=16
  note: 朝の板 / 出来高 / spread 確認後に entry 検討可 (= auto-order なし、Fujiwara 手動発注)
        9130 は pre-open 運用 override (= D34 連続 4 日目) で entry → watch 機械的移動済

【post-open 更新済】(= run_v1_4_1_post_open_update wrapper を pre-open phase で実行)
  - input: /tmp/fire_d34_prep/d34_consumer_payload.json
  - actual_flow (override): /tmp/fire_d34_prep/actual_flow_d34_pre_open_override.json (= 9130 skip)
  - output: /tmp/fire_d34_prep/d34_payload_after_override.json
  - moved_entry_to_watch: ['9130']

【entry 候補】(= 第一確認候補、2 件)
  1) 9247 ＴＲＥ HD (情報通信・サービスその他 / リサイクル)
    [運用: entry 検討可]
    entry 検討価格: 1,612 円
    指値目安      : 1,612 円  (寄付前 reference)
    追わない上限  : 1,644 円  (+2.0%)
    利確目安      : 1,773 円  (+2R = +161 円)
    損切り目安    : 1,531 円  (-1R = -81 円)
    株数          : 100 株    (標準、非 100 株調整なし)
    risk_yen      : 8,060 円  (100 株あたり最大損失)
    見送り条件    : 寄付前 gap > 1,644 / 板薄 / 出来高低 / spread 拡大 / セクター逆行
    観察項目      : 寄付前気配 / 信用倍率 / 板厚 / spread / 5MA-25MA

  2) 9628 燦 HD (情報通信・サービスその他 / 葬祭)
    [運用: entry 検討可]
    entry 検討価格: 1,390 円
    指値目安      : 1,390 円  (寄付前 reference)
    追わない上限  : 1,418 円  (+2.0%)
    利確目安      : 1,529 円  (+2R = +139 円)
    損切り目安    : 1,320 円  (-1R = -70 円)
    株数          : 100 株    (標準)
    risk_yen      : 6,950 円
    見送り条件    : 寄付前 gap > 1,418 / 板薄 / 出来高低 / spread 拡大 / セクター逆行
    観察項目      : 同上

【watch 候補】(= signal 観察 / entry 不可、2 件)
  1) 9130 共栄タンカー (運輸・物流 / 海運業)  ★★★ 連続 4 日目 ★★★
     watch 理由      : post-open chase_limit 超過観測 (= 新規 entry 不可、signal 観察)
                       / pre-open 運用 override (= D34 連続 4 日目、D35 = 5 連続確認 +
                       demote simulation 候補、D36 = 必要なら demote 本実行候補)
     entry 不可理由  : no-new-chase 確定 (= fujiwara_decision='skip', reason 追わない上限)
     signal 観察     : high が chase_limit 1,438 円超過観測の有無
     demote 監視     : D34 = 4 連続確認 / D35 = 5 連続 warning + demote simulation 候補
                       (= recently_seen 追加 + F111 sim 実行) / D36 = 必要なら demote 本実行候補
     reason_codes    : fujiwara_skip, fujiwara_skip_chase, actual_flow_pending

  2) 4404 ミヨシ油脂 (食品 / 食品工業 / 油脂)
     watch 理由      : standard_lot_ok=False (= 100 株 risk_yen 10,665 円 > pilot 限度)
     entry 不可理由  : 非 100 株調整不採用 (= 非標準 lot 案禁止)、pilot 限度超
     signal 観察     : 翌日以降 risk 適正化 / liquidity 回復で再評価
     demote 監視     : recently_seen 累積で除外対象になりうる

【excluded 候補】(= 当日 entry 不可、16 件)
  demote 7 件 (= recently_seen_codes):
    8747 豊トラスティ証券 / 5729 日本精鉱 / 3489 フェイスネットワーク / 340A0 ジグザグ /
    3798 ULS グループ / 137A0 Cocolive / 7991 マミヤ・オーピー

  liquidity FAIL 9 件 (= 信用 / 非 TOPIX / letter-suffix / 出来高薄):
    331A0 メディックス (letter-suffix) / 4389 プロパティデータバンク / 4317 レイ /
    2700 木徳神糧 / 6149 小田原エンジニアリング / 288A0 ラクサス (letter-suffix) /
    2981 ランディックス / 8152 ソマール / 339A0 プログレス・テクノロジーズ (letter-suffix)

【no-new-chase / demote 監視】(= post_open_monitor、1 件)
  ★ 9130 (entry_candidate → watch_candidate) ★ 連続 4 日目 ★
    chase_limit_value      : 1438 (= 1410 × +2.0%)
    chase_limit_exceeded   : None (= 観測前、Fujiwara が actual 入力後に判定)
    no_new_chase           : True (= pre-open 運用 override で確定)
    signal_success         : pending (= OHLCV 未提供)
    entry_success          : skipped (= Fujiwara skip 確定)
    fujiwara_decision      : skip
    fujiwara_reason        : no-new-chase 確定: D34 連続 4 日目、D35 = 5 連続 warning...
    reason_codes           : actual_flow_pending, fujiwara_skip, fujiwara_skip_chase
    連続日数               : D31=1 / D32=2 / D33=3 / D34=4 (= 本日) / D35 で 5 連続なら demote sim
    継続観察               : D34 = 4 連続確認 / D35 = 5 連続 warning + demote simulation 候補 /
                             D36 = 必要なら demote 本実行候補

【9130 連続日数 + demote 日程】
  D31 (06-25): 1 連続目 (= reality entry +4.89%)
  D32 (06-26): 2 連続目 (= entry → no-new-chase)
  D33 (06-29): 3 連続目 (= 同上)
  D34 (06-30): 4 連続目 (= 本日、no-new-chase 維持) ← 今ここ
  D35 (07-01): 5 連続なら warning + demote simulation 候補 (= recently_seen 追加 + F111 sim)
  D36 (07-02): 必要なら demote 本実行候補 (= recently_seen_codes 正式追加)

【iSPEED 確認項目】(= 寄付前後 board / volume check)
  1. 寄付前気配 / 気配指値           (= 9247 / 9628 のみ)
  2. 信用倍率 / 信用残
  3. 板厚 / 出来高初動
  4. スプレッド (= ask-bid 比)
  5. 5 分足 5MA / 25MA 位置
  6. 前日終値 / 当日寄値 / ギャップ %
  7. セクター指数の相対強弱 (= 情報通信・サービスその他 / 食品 / 運輸・物流)

【review 10 項目】(= D34 close 後に記録、template = vault)
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

  signal_success 判定基準注記:
    - historical review (= ~/fire-vault/04_daily/*.md): 当日朝 advisory 利確目安に対する事後評価
    - current renderer (= compute_signal_success): post-open high/low が +2R / -1R hit 自動判定
    → 別 field / 別意味、current renderer 値で historical を上書きしない

【D31/D32/D33 からの引き継ぎ】
  D31 (2026-06-25): 9130 +4.89% (= signal_success、連続 1 日目)
  D32 (2026-06-26): 9247 / 9628 entry → skip / 9130 no-new-chase (連続 2) / 4404 watch
  D33 (2026-06-29): 9247 / 9628 entry / 9130 連続 3 / 4404 watch
  D34 (2026-06-30): 同 4 候補 + 9130 **連続 4 日目** (= 本日)

【safety footer】
  - read-only advisory / auto-order なし / Computer Use なし / LINE なし
  - 100 株標準 / 非 100 株調整なし (= 非標準 lot 案禁止)
  - DB write 0 / API 0 / token 0 / launchctl 0
  - F282 production plist (1772) / smoke plist (1844) / LaunchAgents plist (1772)
    の **3 file 別個** に safety final 確認 (= 全て別 path / 別 file)
  - D31 reality = 9130/331A0/4389、D32 候補 = 9247/9628/9130/4404、D33 候補 = 同上、D34 候補 = 同上

【関連 file】
  F111 output          : /tmp/fire_d34_prep/d34_f111.json
  consumer payload     : /tmp/fire_d34_prep/d34_consumer_payload.json
  pre-open override    : /tmp/fire_d34_prep/actual_flow_d34_pre_open_override.json (= 9130 skip)
  post-open payload    : /tmp/fire_d34_prep/d34_payload_after_override.json (= 9130 watch 移動済)
  morning advisory MD  : /tmp/fire_d34_prep/d34_morning_advisory.md (= post-open updated)
  actual_flow template : /tmp/fire_d34_prep/d34_actual_flow_template.json (= 寄り後 input 用)
  trade plan           : ~/fire-vault/04_daily/2026-06-30_manual_live_pilot_trade_plan.md
  review template      : ~/fire-vault/04_daily/2026-06-30_manual_live_pilot_review.md

【post-open 更新手順】(= 寄り後 / 前場後)
  1. /tmp/fire_d34_prep/d34_actual_flow_template.json をコピーして actual_flow_d34.json 作成
  2. OHLCV / fujiwara_decision を手動入力 (= 推測禁止、観測値のみ)
  3. .venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
       --input-json /tmp/fire_d34_prep/d34_consumer_payload.json \
       --actual-flow-json /tmp/fire_d34_prep/actual_flow_d34.json \
       --output-json /tmp/fire_d34_prep/d34_post_open_payload.json \
       --strict
  4. .venv/bin/python -m scripts.jobs.run_v1_4_1_morning_advisory_markdown \
       --input-json /tmp/fire_d34_prep/d34_post_open_payload.json \
       --output-md /tmp/fire_d34_prep/d34_morning_advisory_post_open.md \
       --today 2026-06-30 \
       --strict

【close 後手順】
  1. ~/fire-vault/04_daily/2026-06-30_manual_live_pilot_review.md を埋める
  2. OHLCV / signal_success / entry_success / 板 spread 所感 / 結果メモ を手動記録
  3. 9130 連続日数の更新: D34 = 4 連続目 → D35 で 5 連続確認 →
     demote simulation 候補 (= 9130 を recently_seen 追加した F111 sim 実行)
  4. D35 朝の F111 / advisory 準備 (= 別 wave、2026-07-01)
```
