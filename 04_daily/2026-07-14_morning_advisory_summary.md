---
id: FIRE-pilot-D44-morning-summary-2026-07-14
phase: 本番 v0 / W60-pilot-D44 / v1.4.2 朝 pilot 7 日目 / max=50 baseline 2 日目
priority: 高
status: pre-open ready
owner: BlueFire7777 (Fujiwara)
date: 2026-07-14 (= D44、火)
---

```
[FIRE 朝 advisory v1.4.2 — D44 (2026-07-14 火) morning summary (= 7 日目 / max=50 baseline 2 日目)]

【判定】GO_CONDITIONAL
  entry raw=14 / entry safe=14 / watch=0 / excluded=21
  policy_version: 1.4.2 ✓ (= 7 日目)
  cli_version: 1.3.0 (= W60-F111-baseline-50)
  max_candidates: 50 ✓ (= baseline 自然適用 2 日目)
  note: 朝の板 / 出来高 / spread 確認後に entry 検討可 (= Fujiwara 手動発注)
        9130 demote 効果継続 9 日目 (= excluded 維持)
        4404 v1.4.2 entry 復帰 7 日目 (= ⚠ risk_yen_warning 付き)
        9247 / 9628 連続 **13 日目** (= fixed-candidate HIGH 相当、D45+ 片方 demote 候補)
        sector entry 上 7 種多様化継続 (= D43 と同)

【正式 recently_seen_codes】(= 8 件版、D36 以降継続)
  9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798
  demoted_count: 8

【★ 9130 demote 効果継続 9 日目 ★】
  role: excluded_candidate ✓
  reason_codes: ['recently_seen_demoted']
  entry に戻るか: No ✓
  watch に戻るか: No ✓
  連続 excluded: D36(1)/D37(2)/D38(3)/D39(4)/D40(5)/D41(6)/D42(7)/D43(8)/D44(9) = 9 日連続

【★ 4404 v1.4.2 entry 復帰 7 日目 + risk warning ★】
  role: entry_candidate ✓
  standard_lot_ok: True / share_unit: 100
  risk_yen_for_100_shares: 10,665 円
  risk_yen_over_pilot_budget: True (= ⚠ 警告フラグ継続)
  reason_codes: ['risk_yen_warning']
  連続 entry: D38(1)/D39(2)/D40(3)/D41(4)/D42(5)/D43(6)/D44(7) = 7 日連続

【★★ 9247 / 9628 連続 13 日目 = fixed-candidate HIGH 相当 ★★】
  9247: D32 以来 **13 日連続** entry (top 1)
  9628: D32 以来 **13 日連続** entry (top 2、同 sector)
  9130 demote 閾値参考: 5 連続 → 2.6 倍超

  D44 fixed-candidate HIGH 判定:
  - 9247 / 9628 が依然 top 固定 (= 連続 13 日)
  - D43 D41 sim A/B/C 既完了 + D42 max=50 sim 既完了
  - max=50 baseline 化後も 9247/9628 score 上位継続 (= 緩和限定)
  - **D45 以降の片方 demote 本実行検討対象**

  【D45 以降の判断】(= HQ approve 後の別 wave)
    A (9247 単独 demote) → entry 残 12+1 (= 9628+4404 を含む)、慎重 OK ★★★★
    B (9628 単独 demote) → entry 残 12+1 (= 9247+4404 を含む)、慎重 OK ★★★★
    C 両方同時 demote → 推奨しない (= 9130 と同じ pattern を 2 件同時は段階的でない)
    本 wave では実行しない、HQ approve 必須

【★★★ 優先確認 top 5 (= 朝判断高速化用) ★★★】
  1) 9247 ＴＲＥ HD [連続 13 日目、fixed-candidate HIGH]
     1,612 円 / 100 株 / 追わない 1,644 円 / 利確 1,773 円 / 損切り 1,531 円
     risk_yen 8,060 円 / sector 情報通信・サービスその他

  2) 9628 燦 HD [連続 13 日目、fixed-candidate HIGH、9247 同 sector]
     1,390 円 / 100 株 / 追わない 1,418 円 / 利確 1,529 円 / 損切り 1,320 円
     risk_yen 6,950 円 / sector 情報通信・サービスその他

  3) ★ 4404 ミヨシ油脂 [⚠ risk warning、連続 7 日目]
     2,133 円 / 100 株 / 追わない 2,176 円 / 利確 2,346 円 / 損切り 2,026 円
     risk_yen 10,665 円 ⚠ 旧 pilot 上限超過警告
     → 損切り / 板 / 出来高 / spread / RR を **手動確認** してから entry

  4) ★ 3089 テクノアルファ [新規継続、商社 sector 多様化]
     1,055 円 / 100 株 / 追わない 1,076 円 / 利確 1,160 円 / 損切り 1,002 円
     risk_yen 5,275 円 / sector 商社・卸売

  5) ★ 9633 東京テアトル [新規継続、不動産 sector 多様化]
     1,579 円 / 100 株 / 追わない 1,610 円 / 利確 1,737 円 / 損切り 1,500 円
     risk_yen 7,895 円 / sector 不動産

【参考: full entry 候補 14 件 (= max=50 baseline 効果継続)】
  top 5 以外の新規 entry 9 件 (= D43 と同):
   6) 2146 ＵＴグループ (情報通信、184 円、risk 920) — 最低 risk
   7) 4828 ビジネスエンジニアリング (情報通信、1,246 円、risk 6,230)
   8) 8699 ＨＳ HD (金融、1,140 円、risk 5,700) — 金融 sector
   9) 3712 情報企画 (情報通信、1,039 円、risk 5,195)
  10) 7803 ブシロード (情報通信、261 円、risk 1,305) — 低 risk
  11) 4914 高砂香料工業 (素材化学、1,180 円、risk 5,900) — 素材化学 sector
  12) 3479 ティーケーピー (不動産、1,725 円、risk 8,625) ⚠ warning
  13) 4540 ツムラ (医薬品、3,582 円、risk 17,910) ⚠ warning — 最大 risk
  14) 8057 内田洋行 (商社、2,053 円、risk 10,265) ⚠ warning

【watch 候補】(= 0 件、4404 が entry 復帰継続、9130 は excluded)

【excluded 候補】(= 21 件)
  demote 8 件 (= recently_seen_codes、9130 含む):
    9130 共栄タンカー (= D36 demote、D44 で 9 日連続 excluded) /
    8747 / 5729 / 3489 / 340A0 / 3798 / 137A0 / 7991
  liquidity FAIL 13 件:
    331A0 (letter-suffix) / 4389 / 4317 / 2700 / 6149 /
    288A0 (letter-suffix) / 2981 / 8152 / 339A0 (letter-suffix) / + 4 件

【sector 多様化状況】
  D38-D42 baseline (max=20): 情報通信 2 + 食品 1 = 2 sector
  D43-D44 baseline (max=50): **entry 上 7 sector** (= 情報通信/食品/金融/商社/不動産/素材化学/医薬品)
  business_label: field 不在 (= F111-THEME-SECTOR-OVERLAY-R1 必要、別 wave)

【F111-THEME-SECTOR-OVERLAY-R1 必要性整理】
  目的:
    sector_17 は 17 分類で粗い (= 例: 情報通信に 6 件集中)
    Fujiwara が朝みる時、市場テーマ単位 (= 半導体/電線/AI 等) の整理がほしい

  方針 (= seed + dynamic discovery):
    - 固定 10 テーマ限定しない
    - seed 例: 半導体 / 電線 / AI データセンター / 電力インフラ / 防衛 / 銀行 / 商社 /
              機械 / 電機 / 非鉄金属 / 海運 / 不動産 / 食品 ...
    - dynamic discovery: 銘柄ニュース / 業種コード / 業績タグ等から自動拡張
    - 本 wave では設計引き継ぎメモのみ
    - 実装は別 wave (= F111-THEME-SECTOR-OVERLAY-R1 設計 wave)

  D44 結果からの必要性:
    9247/9628 同 sector (情報通信) 集中 = theme overlay があれば自然に「サービス業」
    と「葬祭」など別 theme に分離可能、固定化緩和材料

【iSPEED 確認項目】
  1. 寄付前気配 / 気配指値 (= top 5 全件)
  2. 信用倍率 / 信用残
  3. 板厚 / 出来高初動
  4. スプレッド (= ask-bid 比)
  5. 5 分足 5MA / 25MA 位置
  6. 前日終値 / 当日寄値 / ギャップ %
  7. セクター指数の相対強弱
  8. ★ 4404 限定: risk_yen 10,665 円 / 損切り 2,026 円 / 許容損失改めて確認
  9. ★ 9247/9628 13 連続: fixed-candidate HIGH、同 sector 重複は片方絞り推奨
  10. ★ 新規 (3089/9633): D43 と同候補、流動性・出来高 D43 results 参考

【review 10 項目】(= D44 close 後に記録)
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

【D31-D43 引き継ぎ + D45 以降】
  D31 (06-25): 9130 +4.89% / 連続 1
  D32-D34: 9130 連続 2-4 / 4404 watch (= 旧 v1.4.1)
  D35: 9130 連続 5 + demote sim
  D36: 9130 demote 本実行
  D37: 9130 demote 継続 / sector 集中化 / max_candidates sim
  D38: v1.4.2 初 pilot / 4404 復帰 + warning / sector 2 多様化
  D39-D40: v1.4.2 2-3 日目
  D41: v1.4.2 4 日目 / 9247-9628 連続 10 日目 / demote sim (A/B/C) 完了
  D42: v1.4.2 5 日目 / 9247-9628 連続 11 日目 / max=50 sim 完了
  D43: v1.4.2 6 日目 / 9247-9628 連続 12 日目 / max=50 baseline 初回稼働 / top 5 圧縮初回
  本日 D44 (07-14): **v1.4.2 7 日目 / 9130 demote 9 日目 / 4404 連続 7 日目 / 9247-9628 連続 13 日目 (= fixed-candidate HIGH 到達) / max=50 baseline 2 日目**

  D45 (= 2026-07-15 水 予定):
  - F111 baseline: max=50 / recently_seen 8 件版継続
  - 9130 demote 10 日目観察
  - 4404 連続 entry 8 日目
  - 9247 / 9628 連続 14 日目 or リセット
  - **HQ approve 後の 9247 or 9628 単独 demote 本実行検討** (= sim A or B、D41 結果反映)
  - F111-THEME-SECTOR-OVERLAY-R1 設計 wave 起票検討 (= seed + dynamic discovery)
  - 9130 再 entry → CRITICAL / 9130 watch 戻り → HIGH / 4404 watch 戻り → HIGH

【判定ルール】
  9130 再 entry → **CRITICAL** / 9130 watch 戻り → **HIGH** / 4404 watch 戻り → **HIGH**
  低流動性銘柄 entry 浮上 → **HIGH** / letter-suffix entry → **HIGH** / 非 100 株 entry → **HIGH**
  9247 / 9628 連続 13 日 → **fixed-candidate HIGH 相当** (= 本 wave で記録、D45+ 検討)

【safety footer】
  - read-only advisory / auto-order なし / 楽天証券・iSPEED 自動操作なし / Computer Use なし / LINE なし
  - 100 株標準 / 非 100 株調整なし
  - DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0
  - F282 production plist (1772) / smoke plist (1844) / LaunchAgents plist (1772)
    の **3 file 別個** に safety final 確認
  - 9247/9628 demote 本実行は別 wave (= HQ approve 後)
  - F111 max=50 baseline / max=100/200/300 採用せず
  - 両方同時 demote 原則非推奨
  - theme overlay は seed + dynamic discovery 方式、固定 10 テーマ限定しない

【関連 file】
  F111 baseline         : /tmp/fire_d44_prep/d44_f111.json (= cli=1.3.0, max=50, policy=1.4.2)
  consumer payload      : /tmp/fire_d44_prep/d44_consumer_payload.json
  morning advisory MD   : /tmp/fire_d44_prep/d44_morning_advisory.md (= 14,945 bytes)
  actual_flow template  : /tmp/fire_d44_prep/d44_actual_flow_template.json
  trade plan            : ~/fire-vault/04_daily/2026-07-14_manual_live_pilot_trade_plan.md
  review template       : ~/fire-vault/04_daily/2026-07-14_manual_live_pilot_review.md
  parent baseline doc   : ~/fire-vault/03_design/FIRE_f111_max_candidates_baseline_50_2026-05-16.md
```
