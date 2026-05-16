---
id: FIRE-pilot-D48-morning-summary-2026-07-21
phase: 本番 v0 / W60-pilot-D48 / 海の日明け / top-n=100 + max=50 baseline 2 日目
priority: 高
status: pre-open ready / GO_CONDITIONAL
owner: BlueFire7777 (Fujiwara)
date: 2026-07-21 (= D48、火、海の日明け)
parent_wave: W60-pilot-D47 (= 2026-07-17 金)
---

```
[FIRE 朝 advisory v1.4.2 — D48 (2026-07-21 火、海の日明け) morning summary
  = top-n=100 + max=50 baseline 2 日目]

【判定】GO_CONDITIONAL
  baseline 確認: top_n=100 ✓ / max_candidates=50 ✓ / policy=1.4.2 ✓
  cli_version: 1.3.0 (= F111 baseline-50) + signal_persistence default=100 (= W2 baseline)
  signal_persistence dry-run: top_n=100 / persistence 109 / DB write 0 ✓
  F111 raw 35 件 (= staging signals latest 2026-05-13 / 35 件、D47 から不変)
  entry raw=14 / watch=0 / excluded=21 (= D47 完全同一)

【staging signals 状態 (= D48 重要観察)】
  latest base_date: 2026-05-13
  latest count: 35
  D47 → D48 で更新: 無 ✓
  signal_persistence dry-run 109 件母集団は F111 chain に未反映
  → universe 拡張効果 D48 でも未発現
  → 原因: signal_persistence --write 不指定 + staging への定期書込 cron 未稼働
  → staging write 実行には HQ_APPROVE_STAGING_SIGNAL_PERSISTENCE_WRITE 必要
  → 本 wave では実行せず、別 wave で承認後実装

【正式 recently_seen_codes】(= 8 件版、D36 以降継続)
  9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798
  demoted_count: 8

【★ 9130 demote 効果継続 13 日目 ★】
  role: excluded_candidate ✓
  reason_codes: ['recently_seen_demoted']
  entry/watch 復帰: No ✓
  連続 excluded: D36(1) → D48(13) = 13 日連続 (= 4 営業日跨ぎ、海の日含む)

【★ 9247 / 9628 fixed-candidate 17 連続 ★】
  9247 entry: True (= D32 以来 17 日連続、4 営業日跨ぎでも entry 継続)
  9628 entry: True (= 同上、17 日連続)
  9130 demote 閾値 (= 5) の 3.4 倍超
  両方同時 demote 禁止維持
  単独 demote 検討: D49 以降の別 wave + HQ approve 後、本 wave 範囲外
  本 wave demote 本実行: 0

【★ 4404 risk warning entry 11 日目 ★】
  role: entry_candidate ✓ (= watch 戻り なし)
  risk_yen_for_100_shares: 10,665 円
  risk_yen_over_pilot_budget: True (= ⚠ 警告フラグ継続)
  reason_codes: ['risk_yen_warning']
  連続 entry: D38(1) → D48(11) = 11 日連続
  risk 扱い矛盾: 無 (= warning 表示のみ、entry 維持で v1.4.2 整合)

【★★★ 優先確認 top 5 ★★★】
  1) 9247 ＴＲＥ HD       [連続 17 日目、fixed-candidate HIGH 継続]
     1,612 円 / 100 株 / 追わない 1,644 / 利確 1,773 / 損切り 1,531 / risk 8,060 円
  2) 9628 燦 HD           [連続 17 日目]
     1,390 円 / 100 株 / 追わない 1,418 / 利確 1,529 / 損切り 1,320 / risk 6,950 円
  3) ★ 4404 ミヨシ油脂   [⚠ risk warning、連続 11 日目]
     2,133 円 / 100 株 / 追わない 2,176 / 利確 2,346 / 損切り 2,026 / risk 10,665 円 ⚠
  4) ★ 3089 テクノアルファ [D44-D48 entry 継続、商社]
     1,055 円 / 100 株 / 追わない 1,076 / 利確 1,160 / 損切り 1,002 / risk 5,275 円
  5) ★ 9633 東京テアトル [D44-D48 entry 継続、不動産]
     1,579 円 / 100 株 / 追わない 1,610 / 利確 1,737 / 損切り 1,500 / risk 7,895 円

【参考: full entry 14 件 (= D43-D48 安定)】
  6) 2146 ＵＴグループ (情報通信、184 円、risk 920)
  7) 4828 ビジネスエンジニアリング (情報通信、1,246 円、risk 6,230)
  8) 8699 ＨＳ HD (金融、1,140 円、risk 5,700)
  9) 3712 情報企画 (情報通信、1,039 円、risk 5,195)
  10) 7803 ブシロード (情報通信、261 円、risk 1,305)
  11) 4914 高砂香料工業 (素材化学、1,180 円、risk 5,900)
  12) 3479 ティーケーピー (不動産、1,725 円、risk 8,625) ⚠
  13) 4540 ツムラ (医薬品、3,582 円、risk 17,910) ⚠
  14) 8057 内田洋行 (商社、2,053 円、risk 10,265) ⚠

【sector / 新顔】
  entry 上 7 sector (= 情報通信 6 / 商社 2 / 不動産 2 / 食品 1 / 金融 1 / 素材化学 1 / 医薬品 1)
  F111 sectors: 12
  追加新顔: 0 (= staging signals 未更新による)
  新セクター: 0 (= 同上)
  → top-n=100 baseline は dry-run 適用、実 universe 拡張は staging write 後発現期待

【universe 拡張未発現の理由】(= D48 重要記録)
  1. signal_persistence baseline 化 (= top-n 30 → 100) は W2 で完了
  2. ただし staging への定期 write は別 wave で実装
  3. 本 wave は read-only smoke + dry-run、staging signals 未更新
  4. F111 INNER JOIN with research_watchlist_signals (latest 2026-05-13 / 35 件)
  5. → F111 raw は 35 件で D47 と同等継続
  6. 実 universe 拡張効果は HQ approve 後の staging write wave で発現確認

【iSPEED 確認 10 項目】(= D47 と同様)
  寄付前気配 / 信用倍率 / 板厚出来高初動 / spread / 5MA-25MA / ギャップ% /
  セクター強弱 / 4404 risk_yen 改めて / 9247-9628 17 連続片方絞り / 新規 (3089-9633) 流動性

【判定ルール】(= 継続)
  9130 再 entry → CRITICAL / 9130 watch 戻り → HIGH / 4404 watch 戻り → HIGH
  低流動性 / letter-suffix / 非 100 株 entry → HIGH
  9247 / 9628 連続 17 日 → fixed-candidate HIGH 相当継続 (= D49+ 検討)

【安全 gate all 0】
  低流動性 entry 混入: 0 ✓
  letter-suffix entry 混入: 0 ✓
  非 100 株 entry 混入: 0 ✓
  risk_yen 順位加点: 0 ✓ (= v1.4.2 維持)
  risk_yen entry 除外 gate: 0 ✓ (= warning entry 4 件全 entry 維持)
  DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0 / plist 0 / cron 0 / workflow 0
  staging write 0 / production-develop 接続 0
  git add 0 / commit 0 / push 0 / --no-verify 0 / TODO Excel 更新 0
  VACUUM 0 / sudo 0 / rm -rf 0 / auto-order 0 / Computer Use 0

【D49 (= 2026-07-22 水) handoff】
  - F111 baseline: max=50 / recently_seen 8 件版継続
  - signal_persistence baseline: top-n=100 (= W2 適用継続)
  - 9130 demote 14 日目観察
  - 4404 連続 entry 12 日目
  - 9247 / 9628 連続 18 日目 (= 固定化継続なら HQ approve 後の単独 demote 候補)
  - **staging signals 書込 wave 起票検討** (= HQ_APPROVE_STAGING_SIGNAL_PERSISTENCE_WRITE)
    - W2 baseline (top-n=100) を実 universe に反映するための定期 write 設計
    - 別 wave 起票推奨
  - F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 起票検討 (= 9247/9628 固定化根本対策)
  - 9130 再 entry → CRITICAL / 4404 watch 戻り → HIGH

【関連 file】
  signal_persistence  : /tmp/fire_d48_prep/d48_signal_persistence.json (top_n=100, 109件 dry-run)
  F111 baseline       : /tmp/fire_d48_prep/d48_f111.json (cli=1.3.0, max=50, policy=1.4.2)
  consumer payload    : /tmp/fire_d48_prep/d48_consumer_payload.json
  morning advisory MD : /tmp/fire_d48_prep/d48_morning_advisory.md (= 14,945 bytes)
  trade plan          : ~/fire-vault/04_daily/2026-07-21_manual_live_pilot_trade_plan.md
  review template     : ~/fire-vault/04_daily/2026-07-21_manual_live_pilot_review.md
  parent D47          : ~/fire-vault/04_daily/2026-07-17_morning_advisory_summary.md
  parent W2 result    : ~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_W2_result_2026-05-16.md
```
