---
id: FIRE-pilot-D36-morning-summary-2026-07-02
phase: 本番 v0 / W60-pilot-D36 / v1.4.1 morning advisory + 9130 demote 本実行完了
priority: 最高
status: pre-open ready、9130 demote 本実行完了 (= HQ approve 済)
owner: BlueFire7777 (Fujiwara)
date: 2026-07-02 (= D36、木)
---

```
[FIRE 朝 advisory v1.4.1 — D36 (2026-07-02 木) morning summary + 9130 demote 本実行完了]

【判定】GO_CONDITIONAL
  entry raw=2 / entry safe=2 / watch=1 / excluded=17
  note: 朝の板 / 出来高 / spread 確認後に entry 検討可 (= Fujiwara 手動発注)
        9130 demote 本実行完了 (= recently_seen_codes 正式追加、excluded 固定)
        sector 集中化リスクあり (= サービス業 2 件のみ、Fujiwara 判断要)

【★★★ 9130 demote 本実行完了 ★★★】(= HQ approve 済)
  HQ approve: YES (= D36 wave 指示文で明示)
  9130 role: entry (D35 baseline) → **excluded** (D36 本実行) ✓
  reason_codes: ['recently_seen_demoted'] ✓
  D35 simulation との一致: **完全一致** ✓
  entry に残るか: No ✓
  watch に残るか: No (= 直接 excluded、post-open update wrapper 不要)

  正式 recently_seen_codes (= 8 件):
    9130, 8747, 5729, 137A0, 3798, 340A0, 7991, 3489
  demoted_count: 8

【D35 simulation vs D36 本実行 比較】
  | 観点 | D35 sim | D36 本実行 |
  | 9130 role | excluded | excluded ✓ |
  | entry | 9247, 9628 | 9247, 9628 ✓ |
  | watch | 4404 | 4404 ✓ |
  | excluded | 17 (+9130) | 17 (+9130) ✓ |
  | 新規 entry 浮上 | 0 件 | 0 件 ✓ |
  → demote logic 再現性確認 ✓

【entry 候補】(= 第一確認候補、2 件)
  1) 9247 ＴＲＥ HD (情報通信・サービスその他 / リサイクル)
    [運用: entry 検討可、ただし同 sector 重複注意]
    entry 検討価格: 1,612 円
    指値目安      : 1,612 円  (寄付前 reference)
    追わない上限  : 1,644 円  (+2.0%)
    利確目安      : 1,773 円  (+2R = +161 円)
    損切り目安    : 1,531 円  (-1R = -81 円)
    株数          : 100 株    (標準、非 100 株調整なし)
    risk_yen      : 8,060 円  (100 株あたり最大損失)
    sector        : 情報通信・サービスその他、TOPIX Small 1、貸借
    見送り条件    : 寄付前 gap > 1,644 / 板薄 / 出来高低 / spread 拡大 / セクター逆行 /
                    9628 と同 sector のため同時 entry 慎重
    観察項目      : 寄付前気配 / 信用倍率 / 板厚 / spread / 5MA-25MA

  2) 9628 燦 HD (情報通信・サービスその他 / 葬祭)
    [運用: entry 検討可、ただし同 sector 重複注意]
    entry 検討価格: 1,390 円
    指値目安      : 1,390 円
    追わない上限  : 1,418 円  (+2.0%)
    利確目安      : 1,529 円  (+2R = +139 円)
    損切り目安    : 1,320 円  (-1R = -70 円)
    株数          : 100 株
    risk_yen      : 6,950 円
    sector        : 情報通信・サービスその他、TOPIX Small 2、貸借
    見送り条件    : 寄付前 gap > 1,418 / 板薄 / 出来高低 / spread 拡大 / セクター逆行 /
                    9247 と同 sector のため同時 entry 慎重
    観察項目      : 同上

【watch 候補】(= 1 件、9130 は demote で excluded のため watch にいない)
  1) 4404 ミヨシ油脂 (食品 / 食品工業 / 油脂)
     watch 理由      : standard_lot_ok=False (= 100 株 risk_yen 10,665 円 > pilot 限度)
     entry 不可理由  : 非 100 株調整不採用、pilot 限度超
     signal 観察     : 翌日以降 risk 適正化 / liquidity 回復で再評価

【★ sector 集中化リスク ★】
  D36 entry 候補は **情報通信・サービスその他 2 件のみ** (= 9247 / 9628).
  Fujiwara 判断要:
    - 同 sector 重複を許容して両方 entry 検討するか
    - いずれか 1 件に限定して entry するか
    - alternate candidates (= 別 sector) を別 wave で検討するか
  短期対応案: 9247 か 9628 のいずれか 1 件に 100 株 entry
  中期対応案: F111 max_candidates 拡張 sim で別 sector 候補浮上を確認 (= 別 wave)

【excluded 候補】(= 当日 entry 不可、17 件)
  demote 8 件 (= recently_seen_codes、本 D36 新規追加 9130 含む):
    9130 共栄タンカー (★ 本 D36 新規 demote、recently_seen_demoted) /
    8747 豊トラスティ証券 / 5729 日本精鉱 / 3489 フェイスネットワーク / 340A0 ジグザグ /
    3798 ULS グループ / 137A0 Cocolive / 7991 マミヤ・オーピー

  liquidity FAIL 9 件:
    331A0 メディックス (letter-suffix) / 4389 プロパティデータバンク / 4317 レイ /
    2700 木徳神糧 / 6149 小田原エンジニアリング / 288A0 ラクサス (letter-suffix) /
    2981 ランディックス / 8152 ソマール / 339A0 プログレス・テクノロジーズ (letter-suffix)

【9130 連続日数 + demote 状態】
  D31 (06-25): 1 連続目 (= reality entry +4.89%)
  D32 (06-26): 2 連続目 (= no-new-chase)
  D33 (06-29): 3 連続目
  D34 (06-30): 4 連続目
  D35 (07-01): 5 連続目 = warning 確定 + demote sim 候補確定
  D36 (07-02): **demote 本実行完了** (= recently_seen 正式追加、excluded 固定) ← 本日
  D37 (07-03 予定): demote 効果継続観察、9130 再 entry 浮上は CRITICAL (= recently_seen logic 不具合)

【iSPEED 確認項目】
  1. 寄付前気配 / 気配指値           (= 9247 / 9628 のみ)
  2. 信用倍率 / 信用残
  3. 板厚 / 出来高初動
  4. スプレッド (= ask-bid 比)
  5. 5 分足 5MA / 25MA 位置
  6. 前日終値 / 当日寄値 / ギャップ %
  7. セクター指数の相対強弱 (= 情報通信・サービスその他 / 食品 / 運輸・物流)

【review 10 項目】(= D36 close 後に記録)
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

【D31/D32/D33/D34/D35 からの引き継ぎ + D37 以降】
  D31 (06-25): 9130 +4.89% (= signal_success、actual 未 entry、連続 1)
  D32 (06-26): 9247/9628 entry → skip / 9130 no-new-chase (連続 2) / 4404 watch
  D33 (06-29): 同 + 9130 連続 3
  D34 (06-30): 同 + 9130 連続 4
  D35 (07-01): 同 + 9130 連続 5 = warning + demote sim 候補
  D36 (07-02): **9130 demote 本実行完了** (= recently_seen 正式追加) ← 本日
  D37 (07-03 予定): F111 baseline = 9130 含む recently_seen 8 件版、demote 効果継続観察

【D37 以降の方針】
  - F111 baseline: 9130 含む recently_seen_codes 8 件版 (= 8747, 5729, 3489, 340A0,
    3798, 137A0, 7991, 9130) で継続実行
  - 9130 連続 counter リセット
  - 9130 が D37+ で再 entry に浮上したら CRITICAL (= recently_seen logic 不具合)
  - sector 集中化 follow-up:
    * Fujiwara が sector 集中許容 → D37 通常 pilot 継続
    * 別 sector 候補が欲しい → max_candidates 拡張 sim を別 wave で実行
  - 永続化 (= 環境変数 / config / runner default) は別 wave で実施 (= DB write 禁止維持)

【safety footer】
  - read-only advisory / auto-order なし / Computer Use なし / LINE なし
  - 100 株標準 / 非 100 株調整なし (= 非標準 lot 案禁止)
  - DB write 0 / API 0 / token 0 / launchctl 0
  - F282 production plist (1772) / smoke plist (1844) / LaunchAgents plist (1772)
    の **3 file 別個** に safety final 確認
  - HQ approve 済 (= D36 wave 指示文で明示)
  - 9130 demote 本実行は F111 実行時の recently_seen_codes 渡しのみ (= 永続 DB / config write なし)
  - D31 reality = 9130/331A0/4389、D32-D35 候補 = 9247/9628/9130/4404、D36 候補 = 9247/9628/4404
    (= 9130 demote で entry/watch から除外)

【関連 file】
  F111 output           : /tmp/fire_d36_prep/d36_f111.json (= 正式 recently_seen 8 件版)
  consumer payload      : /tmp/fire_d36_prep/d36_consumer_payload.json
  morning advisory MD   : /tmp/fire_d36_prep/d36_morning_advisory.md
  actual_flow template  : /tmp/fire_d36_prep/d36_actual_flow_template.json
  demote execution doc  : ~/fire-vault/04_daily/2026-07-02_demote_execution_9130.md ★ NEW
  trade plan            : ~/fire-vault/04_daily/2026-07-02_manual_live_pilot_trade_plan.md
  review template       : ~/fire-vault/04_daily/2026-07-02_manual_live_pilot_review.md

【post-open 更新手順】(= optional、9247/9628 で no-new-chase 発生時のみ)
  1. /tmp/fire_d36_prep/d36_actual_flow_template.json をコピーして actual_flow_d36.json 作成
  2. OHLCV / fujiwara_decision を手動入力 (= 推測禁止)
  3. .venv/bin/python -m scripts.jobs.run_v1_4_1_post_open_update \
       --input-json /tmp/fire_d36_prep/d36_consumer_payload.json \
       --actual-flow-json /tmp/fire_d36_prep/actual_flow_d36.json \
       --output-json /tmp/fire_d36_prep/d36_post_open_payload.json \
       --strict
  4. .venv/bin/python -m scripts.jobs.run_v1_4_1_morning_advisory_markdown \
       --input-json /tmp/fire_d36_prep/d36_post_open_payload.json \
       --output-md /tmp/fire_d36_prep/d36_morning_advisory_post_open.md \
       --today 2026-07-02 \
       --strict

【close 後手順】
  1. ~/fire-vault/04_daily/2026-07-02_manual_live_pilot_review.md を埋める
  2. OHLCV / signal_success / entry_success / 板 spread 所感 / 結果メモ を手動記録
  3. 9247 / 9628 の actual entry 結果 (= sector 集中化リスクの実地検証)
  4. **9130 demote 効果の確認** (= 終値 / 出来高、recently_seen 妥当性)
  5. D37 朝の F111 / advisory 準備 (= recently_seen 8 件版継続使用)
```
