---
id: FIRE-pilot-D37-morning-summary-2026-07-03
phase: 本番 v0 / W60-pilot-D37 / v1.4.1 + 9130 demote 効果継続確認 + sector 多様化 sim
priority: 高
status: pre-open ready、9130 demote 効果継続成功、sector 集中化継続
owner: BlueFire7777 (Fujiwara)
date: 2026-07-03 (= D37、金)
---

```
[FIRE 朝 advisory v1.4.1 — D37 (2026-07-03 金) morning summary + 9130 demote 効果継続]

【判定】GO_CONDITIONAL
  entry raw=2 / entry safe=2 / watch=1 / excluded=17
  note: 朝の板 / 出来高 / spread 確認後に entry 検討可 (= Fujiwara 手動発注)
        9130 demote 効果継続成功 (= excluded 維持、CRITICAL/HIGH なし)
        sector 集中化継続 (= サービス業 2 件のみ、多様化 sim 実施済)

【★ 9130 demote 効果継続確認 ★】(= D36 demote 本実行後、D37 で継続確認)
  9130 role: excluded_candidate ✓ (= D36 から維持)
  reason_codes: ['recently_seen_demoted'] ✓
  entry に残るか: No ✓
  watch に残るか: No ✓
  → demote 効果継続成功、recently_seen logic 正常動作

  正式 recently_seen_codes (= 8 件、D36 から継続):
    9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798
  demoted_count: 8

【entry 候補】(= 第一確認候補、2 件)
  1) 9247 ＴＲＥ HD (情報通信・サービスその他 / リサイクル)
    [運用: entry 検討可、ただし 9628 と同 sector 重複注意]
    entry 検討価格: 1,612 円
    指値目安      : 1,612 円  (寄付前 reference)
    追わない上限  : 1,644 円  (+2.0%)
    利確目安      : 1,773 円  (+2R = +161 円)
    損切り目安    : 1,531 円  (-1R = -81 円)
    株数          : 100 株    (標準、非 100 株調整なし)
    risk_yen      : 8,060 円  (100 株あたり最大損失)
    sector        : 情報通信・サービスその他、TOPIX Small 1、貸借
    見送り条件    : gap > 1,644 / 板薄 / 出来高低 / spread 拡大 / 9628 と同 sector のため同時 entry 慎重
    観察項目      : 寄付前気配 / 信用倍率 / 板厚 / spread / 5MA-25MA

  2) 9628 燦 HD (情報通信・サービスその他 / 葬祭)
    [運用: entry 検討可、ただし 9247 と同 sector 重複注意]
    entry 検討価格: 1,390 円
    指値目安      : 1,390 円
    追わない上限  : 1,418 円  (+2.0%)
    利確目安      : 1,529 円  (+2R = +139 円)
    損切り目安    : 1,320 円  (-1R = -70 円)
    株数          : 100 株
    risk_yen      : 6,950 円
    sector        : 情報通信・サービスその他、TOPIX Small 2、貸借
    見送り条件    : gap > 1,418 / 板薄 / 出来高低 / spread 拡大 / 9247 と同 sector のため同時 entry 慎重
    観察項目      : 同上

【watch 候補】(= 1 件)
  1) 4404 ミヨシ油脂 (食品 / 食品工業 / 油脂)
     watch 理由      : standard_lot_ok=False (= 100 株 risk_yen 10,665 円 > pilot 限度)
     entry 不可理由  : 非 100 株調整不採用、pilot 限度超

【excluded 候補】(= 17 件、9130 demote 後継続)
  demote 8 件 (= recently_seen_codes):
    9130 共栄タンカー (= D36 demote 本実行、D37 でも excluded 継続) /
    8747 / 5729 / 3489 / 340A0 / 3798 / 137A0 / 7991
  liquidity FAIL 9 件:
    331A0 (letter-suffix) / 4389 / 4317 / 2700 / 6149 /
    288A0 (letter-suffix) / 2981 / 8152 / 339A0 (letter-suffix)

【★ sector 集中化リスク継続 + 多様化 simulation 結果 ★】
  baseline (max=20): entry 2 件、sector 1 (= 情報通信・サービスその他のみ)
  simulation (max=50): entry 10 件、sector **5** (= 情報通信 6 + 金融 1 + 商社 1 + 不動産 1 + 素材 1)

  新規 entry 候補 8 件 (= simulation のみ、全 100 株 / 貸借 / PASS / letter=False):
    ★ 2146 ＵＴグループ (情報通信、TOPIX Small 1、risk 920 円)
    ★ 4828 ビジネスエンジニアリング (情報通信、TOPIX Small 2、risk 6,230 円)
    ★ 8699 ＨＳ HD (金融、risk 5,700 円)
    ★ 3089 テクノアルファ (商社・卸売、risk 5,275 円)
    ★ 3712 情報企画 (情報通信、risk 5,195 円)
    ★ 7803 ブシロード (情報通信、risk 1,305 円)
    ★ 9633 東京テアトル (不動産、risk 7,895 円)
    ★ 4914 高砂香料工業 (素材・化学、TOPIX Small 1、risk 5,900 円)

  Fujiwara 判断要:
    - 短期 (D37): baseline (max=20) を正本、同 sector 重複許容 or 1 件絞り判断
    - 中期 (D38+): max_candidates 拡張 baseline 化検討 (= 別 wave、HQ approve 必須)

  詳細: 04_daily/2026-07-03_sector_diversification_sim.md

【iSPEED 確認項目】
  1. 寄付前気配 / 気配指値           (= 9247 / 9628 のみ)
  2. 信用倍率 / 信用残
  3. 板厚 / 出来高初動
  4. スプレッド (= ask-bid 比)
  5. 5 分足 5MA / 25MA 位置
  6. 前日終値 / 当日寄値 / ギャップ %
  7. セクター指数の相対強弱 (= 情報通信・サービスその他 / 食品 / 運輸・物流)

【review 10 項目】(= D37 close 後に記録)
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

【D31-D36 からの引き継ぎ + D38 以降】
  D31 (06-25): 9130 +4.89%、連続 1
  D32 (06-26): 9247/9628 entry skip / 9130 連続 2 / 4404 watch
  D33-D34: 同 + 9130 連続 3-4
  D35 (07-01): 9130 連続 5 = warning + demote sim
  D36 (07-02): **9130 demote 本実行完了** (= recently_seen 正式追加)
  D37 (07-03): **9130 demote 効果継続成功** (= excluded 維持、再 entry なし) ← 本日

  D38 (07-06 月 予定):
  - F111 baseline: recently_seen 8 件版 (= 9130 含む) 継続
  - 9130 demote 効果継続観察
  - sector 集中化 follow-up (= 同 sector 重複 or 多様化拡張判断)
  - 9130 が D38+ で再 entry 浮上したら CRITICAL (= recently_seen logic 不具合)

【safety footer】
  - read-only advisory / auto-order なし / Computer Use なし / LINE なし
  - 100 株標準 / 非 100 株調整なし (= 非標準 lot 案禁止)
  - DB write 0 / API 0 / token 0 / launchctl 0
  - F282 production plist (1772) / smoke plist (1844) / LaunchAgents plist (1772)
    の **3 file 別個** に safety final 確認
  - 9130 demote 効果は F111 実行時 recently_seen_codes 渡しで反映 (= 永続 DB write なし)
  - sector 多様化 sim は **read-only analysis** (= baseline 不変、本番反映なし)
  - D31 reality = 9130/331A0/4389、D32-D36 候補 = 9247/9628/9130/4404、D37 候補 = 9247/9628/4404
    (= 9130 demote で entry/watch 除外維持)

【関連 file】
  F111 baseline          : /tmp/fire_d37_prep/d37_f111.json (= recently_seen 8 件版)
  F111 sector sim (max=50): /tmp/fire_d37_prep/d37_f111_max_candidates_sim.json
  consumer payload       : /tmp/fire_d37_prep/d37_consumer_payload.json
  morning advisory MD    : /tmp/fire_d37_prep/d37_morning_advisory.md
  actual_flow template   : /tmp/fire_d37_prep/d37_actual_flow_template.json
  sector diversification : ~/fire-vault/04_daily/2026-07-03_sector_diversification_sim.md ★ NEW
  trade plan             : ~/fire-vault/04_daily/2026-07-03_manual_live_pilot_trade_plan.md
  review template        : ~/fire-vault/04_daily/2026-07-03_manual_live_pilot_review.md
  D36 demote execution   : ~/fire-vault/04_daily/2026-07-02_demote_execution_9130.md
```
