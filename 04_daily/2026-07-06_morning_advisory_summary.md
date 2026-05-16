---
id: FIRE-pilot-D38-morning-summary-2026-07-06
phase: 本番 v0 / W60-pilot-D38 / v1.4.2 初の朝 pilot + 4404 risk warning 復帰
priority: 最高
status: pre-open ready
owner: BlueFire7777 (Fujiwara)
date: 2026-07-06 (= D38、月)
---

```
[FIRE 朝 advisory v1.4.2 — D38 (2026-07-06 月) morning summary]

【判定】GO_CONDITIONAL
  entry raw=3 / entry safe=3 / watch=0 / excluded=17
  policy_version: 1.4.2 ✓ (= initial pilot run)
  note: 朝の板 / 出来高 / spread 確認後に entry 検討可 (= Fujiwara 手動発注)
        9130 demote 効果継続成功 (= excluded 維持)
        4404 v1.4.2 で entry 復帰 (= ⚠ risk_yen_warning 付き、損切り設計手動確認要)
        sector 多様化進展 (= 情報通信 2 + 食品 1 = 2 sector)

【正式 recently_seen_codes】(= 8 件版、D36 から継続)
  9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798
  demoted_count: 8

【★ 9130 demote 効果継続 ★】
  role: excluded_candidate ✓
  reason_codes: ['recently_seen_demoted']
  entry に戻るか: No ✓
  watch に戻るか: No ✓
  → demote 効果継続成功

【★ 4404 v1.4.2 entry 復帰 + risk warning ★】
  role: entry_candidate ✓ (= 旧 v1.4.1 watch から復帰)
  standard_lot_ok: True
  risk_yen_for_100_shares: 10,665 円
  risk_yen_over_pilot_budget: **True** (= ⚠ 警告フラグ)
  reason_codes: ['risk_yen_warning']
  → v1.4.2 risk cap 撤廃で entry 候補、⚠ risk warning 表示

【entry 候補】(= 第一確認候補、3 件)
  1) 9247 ＴＲＥ HD (情報通信・サービスその他 / リサイクル)
    entry 検討価格: 1,612 円
    指値目安      : 1,612 円  (寄付前 reference)
    追わない上限  : 1,644 円  (+2.0%)
    利確目安      : 1,773 円  (+2R = +161 円)
    損切り目安    : 1,531 円  (-1R = -81 円)
    株数          : 100 株    (標準、非標準調整なし)
    risk_yen      : 8,060 円
    sector        : 情報通信・サービスその他、TOPIX Small 1、貸借
    見送り条件    : gap > 1,644 / 板薄 / 出来高低 / spread 拡大 / 9628 と同 sector 重複慎重

  2) 9628 燦 HD (情報通信・サービスその他 / 葬祭)
    entry 検討価格: 1,390 円
    指値目安      : 1,390 円
    追わない上限  : 1,418 円  (+2.0%)
    利確目安      : 1,529 円  (+2R = +139 円)
    損切り目安    : 1,320 円  (-1R = -70 円)
    株数          : 100 株
    risk_yen      : 6,950 円
    sector        : 情報通信・サービスその他、TOPIX Small 2、貸借
    見送り条件    : gap > 1,418 / 板薄 / 出来高低 / spread 拡大 / 9247 と同 sector 重複慎重

  3) ★ 4404 ミヨシ油脂 (食品 / 食品工業 / 油脂) ⚠ risk warning
    ⚠ risk warning (= 旧 pilot 上限超過): 100 株 risk_yen = 10,665 円
      → 損切り価格 / 許容損失 / 板 / 出来高 / spread / RR を **手動確認** してから entry
    entry 検討価格: 2,133 円
    指値目安      : 2,133 円  (寄付前 reference)
    追わない上限  : 2,176 円  (+2.0%)
    利確目安      : 2,346 円  (+2R = +213 円)
    損切り目安    : 2,026 円  (-1R = -107 円)
    株数          : 100 株    (標準、非標準調整なし)
    risk_yen      : 10,665 円 ⚠ 旧 pilot 上限超過警告
    sector        : 食品、貸借、scale=-
    見送り条件    : gap > 2,176 / 板薄 / 出来高低 / spread 拡大 / 損切り価格に納得できない場合

【watch 候補】(= 0 件)
  (なし、4404 が entry 復帰、9130 は excluded)

【excluded 候補】(= 17 件)
  demote 8 件 (= recently_seen_codes、9130 含む):
    9130 共栄タンカー (= D36 demote 本実行で正式追加) /
    8747 / 5729 / 3489 / 340A0 / 3798 / 137A0 / 7991

  liquidity FAIL 9 件:
    331A0 (letter-suffix) / 4389 / 4317 / 2700 / 6149 /
    288A0 (letter-suffix) / 2981 / 8152 / 339A0 (letter-suffix)

【sector 多様化】
  D37 baseline (max=20): 情報通信 2 のみ (= 1 sector)
  D38 baseline (max=20): 情報通信 2 + **食品** 1 (= 2 sector、4404 復帰効果)
  → 多様化進展確認

【iSPEED 確認項目】
  1. 寄付前気配 / 気配指値           (= 9247 / 9628 / 4404)
  2. 信用倍率 / 信用残
  3. 板厚 / 出来高初動
  4. スプレッド (= ask-bid 比)
  5. 5 分足 5MA / 25MA 位置
  6. 前日終値 / 当日寄値 / ギャップ %
  7. セクター指数の相対強弱 (= 情報通信・サービスその他 / 食品 / 運輸・物流)
  8. ★ 4404 限定: risk_yen 10,665 円 / 損切り 2,026 円 / 許容損失を改めて確認

【review 10 項目】(= D38 close 後に記録)
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

【D31-D37 引き継ぎ + D38 v1.4.2 初運用】
  D31 (06-25): 9130 +4.89% / 連続 1
  D32-D34: 9130 連続 2-4 (= no-new-chase) / 4404 watch (= 旧 v1.4.1 risk cap)
  D35 (07-01): 9130 連続 5 = warning + demote sim
  D36 (07-02): **9130 demote 本実行**
  D37 (07-03): 9130 demote 継続 / sector 集中化 / max_candidates sim
  D38 (07-06): **v1.4.2 初 pilot / 4404 entry 復帰 + risk warning / sector 2 多様化** ← 本日

【D39 (= 2026-07-07 火 予定)】
  - F111 baseline: recently_seen 8 件版継続
  - 9130 demote 長期観察
  - 4404 v1.4.2 復帰の継続観察 (= 連続 entry 日数、warning フラグ動作)
  - sector 多様化評価 (= 食品 sector 継続するか)
  - 9130 が D39+ で再 entry 浮上 → CRITICAL
  - 4404 が D39+ で watch に戻る → HIGH (= v1.4.2 logic 不具合)

【safety footer】
  - read-only advisory / auto-order なし / Computer Use なし / LINE なし
  - 100 株標準 / 非 100 株調整なし (= 非標準 lot 案禁止)
  - DB write 0 / API 0 / token 0 / launchctl 0
  - F282 production plist (1772) / smoke plist (1844) / LaunchAgents plist (1772)
    の **3 file 別個** に safety final 確認
  - D31 reality = 9130/331A0/4389、D32-D35 候補 = 9247/9628/9130/4404、D36-D37 = 9130 demote 後、
    D38 = v1.4.2 初 pilot (= 9247/9628/4404 entry / 9130 excluded)

【関連 file】
  F111 baseline         : /tmp/fire_d38_prep/d38_f111.json (= policy_version 1.4.2)
  consumer payload      : /tmp/fire_d38_prep/d38_consumer_payload.json
  morning advisory MD   : /tmp/fire_d38_prep/d38_morning_advisory.md (= 7,594 bytes)
  actual_flow template  : /tmp/fire_d38_prep/d38_actual_flow_template.json
  trade plan            : ~/fire-vault/04_daily/2026-07-06_manual_live_pilot_trade_plan.md
  review template       : ~/fire-vault/04_daily/2026-07-06_manual_live_pilot_review.md
  D36 demote execution  : ~/fire-vault/04_daily/2026-07-02_demote_execution_9130.md
  v1.4.2 design         : ~/fire-vault/03_design/FIRE_selection_policy_v1.4.2_risk_warning_display_2026-05-16.md
```
