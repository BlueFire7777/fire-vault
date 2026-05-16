---
id: FIRE-pilot-D35-morning-summary-2026-07-01
phase: 本番 v0 / W60-pilot-D35 / v1.4.1 morning advisory + 9130 5 連続 warning + demote sim 候補
priority: 最高
status: pre-open ready、demote simulation 実行済、D36 demote 本実行候補確定
owner: BlueFire7777 (Fujiwara)
date: 2026-07-01 (= D35、水)
---

```
[FIRE 朝 advisory v1.4.1 — D35 (2026-07-01 水) morning summary + 9130 demote simulation]

【判定】GO_CONDITIONAL (= post-open updated by pre-open override)
  entry raw=2 / entry safe=2 / watch=2 / excluded=16
  note: 朝の板 / 出来高 / spread 確認後に entry 検討可 (= Fujiwara 手動発注)
        9130 は pre-open 運用 override (= D35 連続 5 日目 = 5 連続 warning 確定) で
        entry → watch 機械的移動済

【post-open 更新済】(= run_v1_4_1_post_open_update wrapper を pre-open phase で実行)
  - input: /tmp/fire_d35_prep/d35_consumer_payload_baseline.json
  - actual_flow (override): /tmp/fire_d35_prep/actual_flow_d35_pre_open_override.json
  - output: /tmp/fire_d35_prep/d35_payload_after_override.json
  - moved_entry_to_watch: ['9130']

【★★★ 9130 5 連続 warning 確定 + demote simulation 実行済 ★★★】
  連続日数: D31=1 / D32=2 / D33=3 / D34=4 / D35=**5** (= 本日 warning 確定)
  D36 (07-02 予定): demote 本実行候補確定 (= HQ approve 必須)

  demote simulation 結果 (= 詳細: 2026-07-01_demote_simulation_9130.md):
    baseline (9130 未追加 recently_seen):
      entry: 9130, 9247, 9628 / watch: 4404 / excluded: 16
    simulation (9130 追加 recently_seen):
      entry: 9247, 9628 / watch: 4404 / excluded: 17 (= +9130)

    9130 role 変化      : entry → excluded (= demote 効果 ✓)
    新規 entry 浮上      : 0 件 (= 9247/9628 維持、安全側)
    100 株適性          : 全 entry 維持 ✓
    流動性              : 全 entry PASS 維持 ✓
    sector 多様化       : 運輸 1 + サービス 2 → サービス 2 のみ (= 集中化、唯一の懸念)
    低流動性 entry 復帰  : なし ✓
    letter-suffix 浮上  : なし ✓
    D36 demote 本実行   : 候補確定 (= HQ approve 必須、慎重判断付き)

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
  1) 9130 共栄タンカー (運輸・物流 / 海運業)  ★★★ 連続 5 日目 = 5 連続 warning ★★★
     watch 理由      : post-open chase_limit 超過観測 + pre-open 運用 override
                       (= D35 連続 5 日目 = 5 連続 warning + demote simulation 候補確定 /
                       D36 = demote 本実行候補 HQ approve 必須)
     entry 不可理由  : no-new-chase 確定 (= fujiwara_decision='skip')
     signal 観察     : high が chase_limit 1,438 円超過観測の有無
     demote 監視     : 本 wave で simulation 実行済、D36 で本実行候補
     reason_codes    : fujiwara_skip, fujiwara_skip_chase, actual_flow_pending

  2) 4404 ミヨシ油脂 (食品 / 食品工業 / 油脂)
     watch 理由      : standard_lot_ok=False (= 100 株 risk_yen 10,665 円 > pilot 限度)
     entry 不可理由  : 非 100 株調整不採用、pilot 限度超
     signal 観察     : 翌日以降 risk 適正化 / liquidity 回復で再評価

【excluded 候補】(= 当日 entry 不可、16 件 / simulation 後 17 件)
  demote 7 件 (= recently_seen_codes、baseline):
    8747 豊トラスティ証券 / 5729 日本精鉱 / 3489 フェイスネットワーク / 340A0 ジグザグ /
    3798 ULS グループ / 137A0 Cocolive / 7991 マミヤ・オーピー
  + 9130 追加 → simulation 8 件

  liquidity FAIL 9 件:
    331A0 メディックス (letter-suffix) / 4389 プロパティデータバンク / 4317 レイ /
    2700 木徳神糧 / 6149 小田原エンジニアリング / 288A0 ラクサス (letter-suffix) /
    2981 ランディックス / 8152 ソマール / 339A0 プログレス・テクノロジーズ (letter-suffix)

【iSPEED 確認項目】
  1. 寄付前気配 / 気配指値           (= 9247 / 9628 のみ)
  2. 信用倍率 / 信用残
  3. 板厚 / 出来高初動
  4. スプレッド (= ask-bid 比)
  5. 5 分足 5MA / 25MA 位置
  6. 前日終値 / 当日寄値 / ギャップ %
  7. セクター指数の相対強弱 (= 情報通信・サービスその他 / 食品 / 運輸・物流)

【review 10 項目】(= D35 close 後に記録)
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
    - historical review: 当日朝 advisory 利確目安への事後評価 (= Fujiwara 手動)
    - current renderer: post-open high/low の +2R / -1R hit 自動判定 (= 推測禁止)
    → 別 field / 別意味、current renderer 値で historical を上書きしない

【D31/D32/D33/D34 からの引き継ぎ】
  D31 (06-25): 9130 +4.89% (= signal_success、連続 1)
  D32 (06-26): 9247/9628 entry → skip / 9130 no-new-chase (連続 2) / 4404 watch
  D33 (06-29): 同 + 9130 連続 3
  D34 (06-30): 同 + 9130 連続 4
  D35 (07-01): 同 + 9130 **連続 5 = warning 確定 + demote sim 候補確定** (= 本日)
  D36 (07-02 予定): demote 本実行候補 (= HQ approve 必須)

【D36 demote 本実行候補判定】= 本実行候補確定 (= 慎重判断付き)
  Prerequisites:
    1. HQ approve (= Fujiwara 明示承認 "9130 recently_seen 正式追加")
    2. D35 actual review (= D35 close 後 Fujiwara verbatim review)
    3. sector 集中化方針確定 (= 許容 OR alternate candidates 検討)
    4. 本番 F111 再実行 (= HQ approve 後)

【safety footer】
  - read-only advisory / auto-order なし / Computer Use なし / LINE なし
  - 100 株標準 / 非 100 株調整なし (= 非標準 lot 案禁止)
  - DB write 0 / API 0 / token 0 / launchctl 0
  - F282 production plist (1772) / smoke plist (1844) / LaunchAgents plist (1772)
    の **3 file 別個** に safety final 確認
  - demote simulation は **本実行ではない** (= recently_seen_codes 正式追加は HQ approve 後)
  - D31 reality = 9130/331A0/4389、D32-D35 候補 = 9247/9628/9130/4404 (= 完全分離)

【関連 file】
  F111 baseline           : /tmp/fire_d35_prep/d35_f111_baseline.json
  F111 demote simulation  : /tmp/fire_d35_prep/d35_f111_demote_sim_9130.json
  consumer baseline       : /tmp/fire_d35_prep/d35_consumer_payload_baseline.json
  consumer simulation     : /tmp/fire_d35_prep/d35_consumer_payload_demote_sim_9130.json
  pre-open override       : /tmp/fire_d35_prep/actual_flow_d35_pre_open_override.json
  post-open payload       : /tmp/fire_d35_prep/d35_payload_after_override.json
  morning advisory MD     : /tmp/fire_d35_prep/d35_morning_advisory.md
  comparison doc (tmp)    : /tmp/fire_d35_prep/d35_demote_sim_comparison.md
  actual_flow template    : /tmp/fire_d35_prep/d35_actual_flow_template.json
  vault demote sim doc    : ~/fire-vault/04_daily/2026-07-01_demote_simulation_9130.md
  trade plan              : ~/fire-vault/04_daily/2026-07-01_manual_live_pilot_trade_plan.md
  review template         : ~/fire-vault/04_daily/2026-07-01_manual_live_pilot_review.md

【post-open 更新手順】(= 寄り後 / 前場後)
  1. /tmp/fire_d35_prep/d35_actual_flow_template.json をコピーして actual_flow_d35.json 作成
  2. OHLCV / fujiwara_decision を手動入力 (= 推測禁止)
  3. .venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
       --input-json /tmp/fire_d35_prep/d35_consumer_payload_baseline.json \
       --actual-flow-json /tmp/fire_d35_prep/actual_flow_d35.json \
       --output-json /tmp/fire_d35_prep/d35_post_open_payload.json \
       --strict
  4. .venv/bin/python -m scripts.jobs.run_v1_4_1_morning_advisory_markdown \
       --input-json /tmp/fire_d35_prep/d35_post_open_payload.json \
       --output-md /tmp/fire_d35_prep/d35_morning_advisory_post_open.md \
       --today 2026-07-01 \
       --strict

【close 後手順】
  1. ~/fire-vault/04_daily/2026-07-01_manual_live_pilot_review.md を埋める
  2. OHLCV / signal_success / entry_success / 板 spread 所感 / 結果メモ を手動記録
  3. 9130 5 連続実績を historical review で確定
  4. **D36 朝の HQ approve 待ち**
     - HQ approve **あり**: D36 で recently_seen に 9130 追加、demote 本実行
     - HQ approve **なし**: D36 baseline 通常実行、追加観察継続
```
