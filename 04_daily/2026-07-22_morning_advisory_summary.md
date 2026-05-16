---
id: FIRE-pilot-D49-morning-summary-2026-07-22
phase: 本番 v0 / W60-pilot-D49 / W3 universe 拡張後の初回稼働
priority: 高
status: pre-open ready / GO_CONDITIONAL
owner: BlueFire7777 (Fujiwara)
date: 2026-07-22 (= D49、水)
parent_wave: F111-UNIVERSE-EXPANSION-R1-W3
---

```
[FIRE 朝 advisory v1.4.2 — D49 (2026-07-22 水) morning summary
  = W3 universe 拡張後の初回稼働、top-n=100 + max=50 baseline 連動運用 1 日目]

【判定】GO_CONDITIONAL
  W3 拡張効果継続: raw 50 / entry 20 / sector entry 9 ✓
  staging signals latest: 2026-07-22 / 109 件 (= W3 write 反映済) ✓
  baseline 確認: top_n=100 ✓ / max_candidates=50 ✓ / policy=1.4.2 ✓
  9130 excluded 維持 / 9247-9628 rank 1/2 継続 / 4404 risk warning entry
  W3 新顔 6 件全 entry 継続 ✓
  安全 gate 全 0 ✓

【W3 拡張反映確認】
  staging signals latest base_date: 2026-07-22
  latest count: 109 件
  source_version: w60_universe_expansion_r1_w3_top100
  F111 raw: 35 (D48) → 50 (D49) (= W3 拡張効果継続) ✓
  entry: 14 (D48) → 20 (D49) (= +6 件新顔継続) ✓
  sector entry: 7 (D48) → 9 (D49) (= 運輸・物流 + 小売 追加) ✓

【正式 recently_seen_codes】(= 8 件版、D36 以降継続)
  9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798
  demoted_count: 8

【★ 9130 demote 効果継続 14 日目 ★】
  role: excluded_candidate ✓
  reason_codes: ['recently_seen_demoted']
  entry/watch 復帰: No ✓
  連続 excluded: D36 → D49 = 14 日連続

【★ 9247 / 9628 = 「継続主役候補」 vs 「惰性固定」両面評価 ★】
  9247 entry rank 1: True (= D32 以来 18 日連続、score 0.8546)
  9628 entry rank 2: True (= 同上、18 日連続、score 0.8479)
  universe 拡張後 (= 50 件母集団) でも rank 1/2 継続

  ★ 継続主役候補としての観点 (= 正当な score 優位):
    - score: 9247=0.8546 / 9628=0.8479 (= entry top 20 内で最上位、新顔 9008=0.8119 / 3134=0.8051 から +0.04-0.05)
    - scale: 9247=TOPIX Small 1 / 9628=TOPIX Small 2 (= 中小型、低流動性ではない)
    - margin: 両者「貸借」(= 信用可、流動性 OK)
    - sector: 情報通信・サービスその他 (= 9247 リサイクル、9628 葬祭、ニッチだが事業内容は別)
    - risk/reward: risk_yen 8,060 / 6,950 円 (= pilot 上限内、warning 無)
    - W3 後 50 件母集団でも依然 score 上位 = 単なる狭い母集団による惰性ではなく、score 自体が高い

  ★ 惰性固定の懸念 (= 18 連続継続):
    - 9130 demote 閾値 (5 連続) の 3.6 倍超
    - 同 sector 重複 (= 情報通信 2 件 top 1/2)
    - 4 営業日跨ぎ (= 海の日) でも rank 維持 → score 算式が time-invariant
    - 新顔候補との score 差は小さい (= 0.04-0.05) ので、theme/scoring 補正の影響を受けやすい

  ★ 判別: 単純な惰性固定ではない (= score 0.84+ で正当な上位)、
    ただし theme/sector 多様化 / 出来高 momentum / news 等の overlay があれば、
    9247/9628 の継続上昇期待度 vs 新顔の伸び代をより正確に比較可能.

  → demote 本実行は 0 (= rank 1/2 は正当な score 上位、無理に外さない)
  → theme overlay は別 wave、目的は「主役排除」ではなく「継続上昇 vs 惰性固定の判別」

【★ 4404 risk warning entry 12 日目 ★】
  role: entry_candidate ✓ / rank 3
  risk_yen_for_100_shares: 10,665 円
  risk_yen_over_pilot_budget: True (= ⚠ 警告フラグ継続)
  reason_codes: ['risk_yen_warning']
  watch 戻り: 0 ✓ / risk 扱い矛盾: 無 (= warning 表示のみ entry 維持)

【★★★ 朝判断 top 5 (= 主役 3 + W3 新顔 + 新セクター) ★★★】
  1) 9247 ＴＲＥ HD       [主役、rank 1、情報通信]
     参考価格 1,612 円 / 100 株 / 追わない 1,644 / 利確 1,773 / 損切り 1,531 / risk_yen 8,060 円
     (= 連続 18 日目、fixed-candidate HIGH 継続。同 sector 重複に 9628 あり。手動判断推奨)

  2) 9628 燦 HD           [主役、rank 2、情報通信 (9247 同 sector)]
     参考価格 1,390 円 / 100 株 / 追わない 1,418 / 利確 1,529 / 損切り 1,320 / risk_yen 6,950 円
     (= 連続 18 日目。9247 と sector 重複、片方絞り推奨)

  3) ★ 4404 ミヨシ油脂   [⚠ risk warning、rank 3、食品、連続 12 日目]
     参考価格 2,133 円 / 100 株 / 追わない 2,176 / 利確 2,346 / 損切り 2,026
     risk_yen 10,665 円 ⚠ (= 旧 pilot 上限超過警告、損切り/板/spread/RR 手動確認後 entry)

  4) ★ 9008 京王電鉄     [W3 新顔、rank 16、運輸・物流 (新セクター)]
     参考価格 747 円 / 100 株 / 追わない 762 / 利確 822 / 損切り 710 / risk_yen 3,735 円
     (= W3 universe 拡張で浮上、低 risk + 新セクター多様化候補)

  5) ★ 3134 Ｈａｍｅｅ   [W3 新顔、rank 19、小売 (新セクター)]
     参考価格 412 円 / 100 株 / 追わない 420 / 利確 453 / 損切り 391 / risk_yen 2,060 円
     (= W3 universe 拡張で浮上、最低 risk + 新セクター多様化候補)

【参考: full entry 20 件 (= W3 拡張後)】
  rank 1-3: 9247 / 9628 / 4404 (= 主役 3 件)
  rank 4-12: 2146 / 4828 / 8699 / 3089 / 3712 / 7803 / 9633 / 4914 / 3479 (= D44-D48 既存)
  rank 13: ★ 9417 (情報通信)         310 円 / risk 1,550
  rank 14: 4540 ツムラ ⚠           3,582 円 / risk 17,910
  rank 15: 8057 内田洋行 ⚠         2,053 円 / risk 10,265
  rank 16: ★ 9008 京王電鉄         747 円 / risk 3,735 (= 運輸・物流 新)
  rank 17: ★ 6196 (情報通信)       1,233 円 / risk 6,165
  rank 18: ★ 7595 (情報通信)       1,356 円 / risk 6,780
  rank 19: ★ 3134 Ｈａｍｅｅ       412 円 / risk 2,060 (= 小売 新)
  rank 20: ★ 3962 (情報通信)       921 円 / risk 4,605

【W3 新顔 6 件 全 entry 継続】
  9417 rank 13 / 情報通信
  9008 rank 16 / 運輸・物流 (新セクター)
  6196 rank 17 / 情報通信
  7595 rank 18 / 情報通信
  3134 rank 19 / 小売 (新セクター)
  3962 rank 20 / 情報通信

【sector 多様化 9 種 (= D44 の 7 種から +2)】
  情報通信・サービスその他 (= 6 件 → 10 件 + W3 新顔 4 件)
  商社・卸売 (= 2 件) / 不動産 (= 2 件) / 食品 (= 1 件) / 金融 (= 1 件) /
  素材化学 (= 1 件) / 医薬品 (= 1 件) / **運輸・物流 (= 1 件、新)** / **小売 (= 1 件、新)**

【watch / excluded】
  watch: 0 件
  excluded: 30 件 (= recently_seen_demoted 8 件 + liquidity_fail 22 件)

【判定ルール】(= 継続)
  9130 再 entry → CRITICAL / 9130 watch 戻り → HIGH / 4404 watch 戻り → HIGH
  低流動性 / letter-suffix / 非 100 株 entry → HIGH

【iSPEED 確認 10 項目】
  寄付前気配 / 信用倍率 / 板厚出来高初動 / spread / 5MA-25MA / ギャップ% /
  セクター強弱 / 4404 risk_yen 改めて / 新顔 (9008/3134) 流動性 / W3 拡張 entry 全 100 株 確認

【安全 gate all 0】
  低流動性 entry 混入: 0 / letter-suffix entry 混入: 0 / 非 100 株 entry 混入: 0
  risk_yen 順位加点: 0 / risk_yen entry 除外 gate: 0
  risk_yen_over_pilot_budget warning entry 維持: 4 件 (= 4404/3479/4540/8057)
  consumer validation_passed: True / violations: 0
  DB write: 0 / staging write: 0 / production-develop 接続: 0
  API: 0 / token: 0 / LINE: 0 / launchctl: 0 / plist: 0 / cron: 0 / workflow: 0
  git add: 0 / commit: 0 / push: 0 / --no-verify: 0
  TODO Excel 更新: 0 / VACUUM: 0 / sudo: 0 / rm -rf: 0
  auto-order: 0 / 楽天証券・iSPEED 自動操作: 0 / Computer Use: 0

【D50 (= 2026-07-23 木) handoff】
  - F111 baseline: max=50 / recently_seen 8 件版継続
  - signal_persistence baseline: top-n=100 (= W3 staging signals 109 件で継続)
  - staging signals latest: 2026-07-22 / 109 件 (= 別 write なしなら継続)
  - 9130 demote: 15 日目観察
  - 4404 risk warning entry: 13 日目
  - 9247 / 9628: 連続 19 日目、score 0.84+ で継続上昇候補として観察 (= 主役継続枠維持)
  - W3 新顔 6 件継続観察 (= 9417/9008/6196/7595/3134/3962)
  - 朝判断 top 5 = 主役継続枠 3 + 新顔/新セクター比較枠 2 (= 9008/3134)
  - F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 起票検討:
      クリア条件 = 「9247/9628 等の継続上昇候補」と「惰性固定」を判別できる overlay
      目的 = 主役候補排除ではなく、主役の継続上昇判定 + 新顔/新セクターの比較表示
      指標例:
        - 出来高 momentum / 売買代金 momentum
        - sector 相対強度
        - theme 単位 (= 半導体/AI/電線/防衛 等) の seed + dynamic discovery
        - 連続日数別の signal_success rate (= 主役継続 vs 惰性失速)
  - F111-UNIVERSE-EXPANSION-R1-W4 (= top-n=200 検証) 検討
  - 判定ルール継続: 9130 entry → CRITICAL / 4404 watch → HIGH 等

【関連 file】
  F111 baseline       : /tmp/fire_d49_prep/d49_f111.json (cli=1.3.0, max=50, raw=50)
  consumer payload    : /tmp/fire_d49_prep/d49_consumer_payload.json
  morning advisory MD : /tmp/fire_d49_prep/d49_morning_advisory.md (= 19,160 bytes)
  trade plan          : ~/fire-vault/04_daily/2026-07-22_manual_live_pilot_trade_plan.md
  review template     : ~/fire-vault/04_daily/2026-07-22_manual_live_pilot_review.md
  parent W3 result    : ~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_W3_result_2026-05-16.md
```
