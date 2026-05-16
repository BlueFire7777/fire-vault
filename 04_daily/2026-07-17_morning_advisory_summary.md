---
id: FIRE-pilot-D47-morning-summary-2026-07-17
phase: 本番 v0 / W60-pilot-D47 / top-n=100 + max=50 baseline 初回稼働
priority: 高
status: pre-open ready / GO_CONDITIONAL
owner: BlueFire7777 (Fujiwara)
date: 2026-07-17 (= D47、金)
parent_wave: F111-UNIVERSE-EXPANSION-R1-W2
---

```
[FIRE 朝 advisory v1.4.2 — D47 (2026-07-17 金) morning summary
  = top-n=100 baseline 初回稼働 + max=50 baseline 4 日目]

【判定】GO_CONDITIONAL
  baseline 確認: top_n=100 ✓ / max_candidates=50 ✓ / policy=1.4.2 ✓
  cli_version: 1.3.0 (= F111 baseline-50) + signal_persistence default=100 (= W2 baseline)
  entry raw=14 / watch=0 / excluded=21
  F111 raw 35 件 (= staging signals latest=35 件、signal_persistence --write 不指定で
    signals 未更新、新 universe 拡張効果は staging write 別 wave 後に発現)

【正式 recently_seen_codes】(= 8 件版、D36 以降継続)
  9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798
  demoted_count: 8

【★ 9130 demote 効果継続 12 日目 ★】
  role: excluded_candidate ✓
  reason_codes: ['recently_seen_demoted']
  entry/watch 復帰: No ✓
  連続 excluded: D36(1) → D47(12) = 12 日連続

【★ 9247 / 9628 状態 (= 連続 entry 継続) ★】
  9247 entry: True (= D32 以来 16 日連続、fixed-candidate HIGH 相当継続)
  9628 entry: True (= 同上、16 日連続)
  両方同時 demote 禁止維持
  片方 demote 本実行: 無 (= 本 wave 範囲外、theme overlay or D48+ 検討)

【★ 4404 risk warning entry 継続 10 日目 ★】
  role: entry_candidate ✓
  risk_yen_for_100_shares: 10,665 円
  risk_yen_over_pilot_budget: True (= ⚠ 警告フラグ継続)
  reason_codes: ['risk_yen_warning']
  連続 entry: D38(1) → D47(10) = 10 日連続

【★★★ 優先確認 top 5 ★★★】
  1) 9247 ＴＲＥ HD       [連続 16 日目、fixed-candidate HIGH 継続]
     1,612 円 / 100 株 / 追わない 1,644 / 利確 1,773 / 損切り 1,531 / risk 8,060 円
  2) 9628 燦 HD           [連続 16 日目]
     1,390 円 / 100 株 / 追わない 1,418 / 利確 1,529 / 損切り 1,320 / risk 6,950 円
  3) ★ 4404 ミヨシ油脂   [⚠ risk warning、連続 10 日目]
     2,133 円 / 100 株 / 追わない 2,176 / 利確 2,346 / 損切り 2,026 / risk 10,665 円 ⚠
  4) ★ 3089 テクノアルファ [新規継続、商社 sector]
     1,055 円 / 100 株 / 追わない 1,076 / 利確 1,160 / 損切り 1,002 / risk 5,275 円
  5) ★ 9633 東京テアトル [新規継続、不動産 sector]
     1,579 円 / 100 株 / 追わない 1,610 / 利確 1,737 / 損切り 1,500 / risk 7,895 円

【参考: full entry 14 件 (= D43-D47 安定)】
  6) 2146 ＵＴグループ (情報通信、184 円、risk 920)
  7) 4828 ビジネスエンジニアリング (情報通信、1,246 円、risk 6,230)
  8) 8699 ＨＳ HD (金融、1,140 円、risk 5,700)
  9) 3712 情報企画 (情報通信、1,039 円、risk 5,195)
  10) 7803 ブシロード (情報通信、261 円、risk 1,305)
  11) 4914 高砂香料工業 (素材化学、1,180 円、risk 5,900)
  12) 3479 ティーケーピー (不動産、1,725 円、risk 8,625) ⚠
  13) 4540 ツムラ (医薬品、3,582 円、risk 17,910) ⚠
  14) 8057 内田洋行 (商社、2,053 円、risk 10,265) ⚠

【sector 多様化】
  entry 上 7 sector: 情報通信 6 / 商社 2 / 不動産 2 / 食品 1 / 金融 1 / 素材化学 1 / 医薬品 1
  D44 と完全一致 (= staging signals 未更新のため新 universe 拡張未発現)

【★★ top-n=100 baseline 実発現確認 (= 本 wave での観察) ★★】
  signal_persistence default smoke:
    top_n: 100 ✓ (= W2 baseline 自然適用)
    persistence: 109 件 ✓
    mode: dry-run / write_enabled: False ✓
    row_count: 13,730 → 13,730 (= DB write 0)

  実 F111 chain で 109 件母集団が反映されるには、staging への signals 書込が必要:
  - 本 wave: dry-run のみ、staging signals 未更新
  - 別 wave (= F111-UNIVERSE-EXPANSION-R1-W3 or daily cron job 反映後) で実 wider universe pilot

【新顔候補確認】
  D44 で出た新顔 (= 3089/9633/8699/4914) D47 でも entry 継続 ✓
  追加新顔: 0 (= staging signals 未更新による)
  → top-n=100 baseline 実 write 後の D48 以降で新顔追加期待

【iSPEED 確認 10 項目】(= D44 と同様)
  寄付前気配 / 信用倍率 / 板厚出来高初動 / spread / 5MA-25MA / ギャップ% /
  セクター強弱 / 4404 risk_yen 改めて / 9247-9628 16 連続片方絞り / 新規 (3089-9633) 流動性

【判定ルール】(= 継続)
  9130 再 entry → CRITICAL / 9130 watch 戻り → HIGH / 4404 watch 戻り → HIGH
  低流動性 / letter-suffix / 非 100 株 entry → HIGH
  9247 / 9628 連続 16 日 → fixed-candidate HIGH 相当継続 (= 本 wave 記録、D48+ 検討)

【safety footer】
  read-only advisory / auto-order なし / 楽天証券・iSPEED 自動操作なし / Computer Use なし / LINE なし
  100 株標準 / 非 100 株調整なし
  DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0 / plist 0 / cron 0 / workflow 0
  git add 0 / git commit 0 / git push 0 / --no-verify 0
  TODO Excel 更新 0
  F282 production plist (1772) / smoke plist (1844) / LaunchAgents plist (1772) 3 file 別個確認
  9247/9628 demote 本実行は別 wave (= HQ approve 後)、両方同時 demote 非推奨
  F111 max=50 baseline / signal_persistence top-n=100 baseline 維持
  theme overlay は seed + dynamic discovery、別 wave (F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1)
  staging への signals 書込は別 wave (= W3 or daily refresh)

【D48 (= 2026-07-21 火、海の日明け) handoff】
  - F111 baseline: max=50 / recently_seen 8 件版継続
  - signal_persistence baseline: top-n=100 (= W2 適用継続)
  - 9130 demote 13 日目観察
  - 4404 連続 entry 11 日目
  - 9247 / 9628 連続 17 日目 (= 海の日跨ぎ) or リセット
  - **staging signals 書込が D47-D48 間で発生していれば universe 拡張効果発現確認**
  - 新顔 entry 浮上確認 (= 商社/不動産/金融/医薬品/素材化学/小売/建設資材 等)
  - F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 起票検討
  - 9130 再 entry → CRITICAL / 4404 watch 戻り → HIGH

【関連 file】
  signal_persistence: /tmp/fire_d47_prep/d47_signal_persistence.json (top_n=100, 109件 dry-run)
  F111 baseline       : /tmp/fire_d47_prep/d47_f111.json (cli=1.3.0, max=50, policy=1.4.2)
  consumer payload    : /tmp/fire_d47_prep/d47_consumer_payload.json
  morning advisory MD : /tmp/fire_d47_prep/d47_morning_advisory.md (= 14,945 bytes)
  trade plan          : ~/fire-vault/04_daily/2026-07-17_manual_live_pilot_trade_plan.md
  review template     : ~/fire-vault/04_daily/2026-07-17_manual_live_pilot_review.md
  parent W2 result    : ~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_W2_result_2026-05-16.md
```
