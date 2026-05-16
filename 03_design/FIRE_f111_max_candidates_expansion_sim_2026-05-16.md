---
id: FIRE-F111-MAX-CANDIDATES-EXPANSION-SIM-2026-05-16
phase: 設計 / D43 準備用 read-only simulation
priority: 最高
status: simulation 完了、baseline 変更なし、本実行 (= Phase 1 max=50 baseline 化) は別 wave + HQ 承認後
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16 (= D42 直近、base_date 2026-07-10)
parent_design: 03_design/FIRE_selection_policy_v1.4.2_risk_warning_display_2026-05-16.md
d42_max50_sim_doc: 04_daily/2026-07-10_demote_decision_9247_9628.md
hq_approve: PENDING (= Phase 1 本実行は HQ approve 後の別 wave)
codex_lanes: 4
---

# FIRE F111 max_candidates expansion simulation (= 2026-05-16)

## §1 目的

F111 候補探索範囲 (= max_candidates) を 20/50/100/200/300 で比較し、
9247/9628 固定化 (= 連続 11 日目) を自然に薄める baseline 候補数 + theme overlay 必要性を判断する.

**simulation は本実行ではない**. baseline 変更 / recently_seen 変更 / 本番反映は HQ approve 後の別 wave.

## §2 結論 (= 推奨案 D)

**max_candidates 拡張だけでは不十分**.

理由:
1. F111 出力 母集団は **35 件で頭打ち** (= staging research_watchlist_signals 制約)
2. max=50/100/200/300 で **完全同一結果**
3. F111 出力に **theme tag / business_label field が不在** (= 半導体/電線/AI 等)

ただし max=20 → 50 は明確改善 (= sectors 8 → 12、entry 3 → 14).

### §2.1 段階的推奨

| Phase | 内容 | 推奨度 | 別 wave |
|---|---|---|---|
| **1 (即時)** | max_candidates 20 → 50 baseline 化 | ★★★★★ | HQ approve 後 |
| 2 (中期) | F111-THEME-SECTOR-OVERLAY-R1 設計 | ★★★★ | 設計 wave 起票 |
| 3 (長期) | research_watchlist_signals 母集団拡張 | ★★★ | 研究 wave 起票 |

## §3 simulation 結果

### §3.1 比較表 (= 18 項目)

| 項目 | max=20 | max=50 | max=100 | max=200 | max=300 |
|---|---|---|---|---|---|
| raw candidates | 20 | **35** | 35 | 35 | 35 |
| entry count | 3 | 14 | 14 | 14 | 14 |
| watch count | 0 | 0 | 0 | 0 | 0 |
| excluded count | 17 | 21 | 21 | 21 | 21 |
| sector_17 count | 8 | **12** | 12 | 12 | 12 |
| business_label count | **0 (field 不在)** | 0 | 0 | 0 | 0 |
| 低流動性 entry 混入 | 0 ✓ | 0 ✓ | 0 ✓ | 0 ✓ | 0 ✓ |
| letter-suffix entry 混入 | 0 ✓ | 0 ✓ | 0 ✓ | 0 ✓ | 0 ✓ |
| 非 100 株調整 | 無 ✓ | 無 ✓ | 無 ✓ | 無 ✓ | 無 ✓ |
| risk_yen_warning (entry) | 1 (4404) | 4 (+3479,4540,8057) | 4 | 4 | 4 |
| 9130 demote 維持 | ✓ | ✓ | ✓ | ✓ | ✓ |
| 9247 entry 維持 | ✓ | ✓ | ✓ | ✓ | ✓ |
| 9628 entry 維持 | ✓ | ✓ | ✓ | ✓ | ✓ |
| 4404 entry 維持 | ✓ | ✓ | ✓ | ✓ | ✓ |
| 処理時間 | ~40ms | ~40ms | ~40ms | ~40ms | ~40ms |

### §3.2 max=20 → max=50 で新規浮上した entry 11 件

| code | 銘柄 | sector_17 | risk_yen |
|---|---|---|---|
| 2146 | ＵＴグループ | 情報通信・サービスその他 | 920 |
| 4828 | ビジネスエンジニアリング | 情報通信・サービスその他 | 6,230 |
| 8699 | ＨＳホールディングス | 金融（除く銀行） | 5,700 |
| 3089 | テクノアルファ | 商社・卸売 | 5,275 |
| 3712 | 情報企画 | 情報通信・サービスその他 | 5,195 |
| 7803 | ブシロード | 情報通信・サービスその他 | 1,305 |
| 9633 | 東京テアトル | 不動産 | 7,895 |
| 4914 | 高砂香料工業 | 素材・化学 | 5,900 |
| 3479 | ティーケーピー | 不動産 | 8,625 ⚠ |
| 4540 | ツムラ | 医薬品 | 17,910 ⚠ |
| 8057 | 内田洋行 | 商社・卸売 | 10,265 ⚠ |

### §3.3 sector 12 種 (= max=50 以降)

不動産 / 医薬品 / 商社・卸売 / 情報通信・サービスその他 / 機械 /
素材・化学 / 運輸・物流 / 金融（除く銀行）/ 鉄鋼・非鉄 / 電機・精密 /
電気・ガス / 食品

## §4 theme overlay 不在の明示

ユーザー要件 §8 で確認すべき theme 11 種:

| theme | F111 field 存在 | 備考 |
|---|---|---|
| semiconductor | **不在** | sample_row keys 0 件 |
| electric_wire | **不在** | 同 |
| ai_datacenter | **不在** | 同 |
| power_grid | **不在** | 同 |
| optical_fiber | **不在** | 同 |
| defense | **不在** | 同 |
| bank | **不在** | sector_17_name=`金融（除く銀行）` で「銀行」自体 0 件 |
| trading_company | **不在** | 同 |
| machinery | **不在** | sector_17=`機械` のみ、theme tag 別途必要 |
| electric_equipment | **不在** | 同 |
| nonferrous_metals | **不在** | sector_17=`鉄鋼・非鉄` のみ |

→ **theme overlay 未実装** (= 推測禁止、明示する).

F111 sample_row field 一覧 (= max=20 確認):
- auto_order_allowed / code / estimated_100_share_risk / expected_h20
- f111_input_source / freshness_context / latest_close / manual_review_required
- margin_name / name / post_cap_rank / realtime_market_cap
- reason_tags / research_advisory_label / research_base_date / research_final_score
- risk_notes / risk_within_pilot_limit / scale_category / sector_17_name
- source_modules / tradable_universe / v1_4_1{policy_version, share_unit, standard_lot_ok, risk_yen_for_100_shares, risk_yen_over_pilot_budget}

→ `theme_*` / `*_trend` / `business_label*` 全て **不在**.

## §5 D43 以降の推奨アクション

### §5.1 Phase 1: max_candidates 20 → 50 baseline 化 (= 即時推奨)

- 影響範囲: F111 / consumer / morning advisory MD 出力件数増
- リスク: ★★★★★ (= 最も安全、母集団 35 件で頭打ち、ノイズなし)
- 9247/9628 固定化緩和: 余地 (= 12 件追加候補で sector 多様化)
- HQ approve 後の別 wave で本実行

### §5.2 Phase 2: F111-THEME-SECTOR-OVERLAY-R1 設計 (= 中期推奨)

- 内容: 半導体 / 電線 / AI データセンター / 電力インフラ / 防衛 / 銀行 / 商社 / 機械 / 電機 / 非鉄金属 等の theme tag を F111 出力に追加
- 前提: theme 定義 + 銘柄 mapping データソース (= 内製で十分)
- 影響: F111 field 拡張、consumer / MD レンダラ拡張
- 別 wave 起票推奨

### §5.3 Phase 3: research_watchlist_signals 母集団拡張 (= 長期推奨)

- 現状: 35 件 → 目標: 100+ 件
- F035 Pattern Research Agent と連携
- 別 wave 起票推奨

## §6 CRITICAL/HIGH 判定

- **CRITICAL: 0** (= 9130 entry 戻り無、低流動性 entry 浮上無)
- **HIGH: 0** (= letter-suffix 浮上無、非 100 株調整無、liquidity_fail 浮上無)

設計指摘 (= CRITICAL/HIGH ではない、要対応):
1. F111 母集団 35 件頭打ち
2. theme/business_label field 不在
3. max=20 baseline が 9247/9628 固定化原因

## §7 関連 file

- 比較 MD: `/tmp/fire_max_candidates_expansion_sim/max_candidates_comparison.md`
- 比較 JSON: `/tmp/fire_max_candidates_expansion_sim/max_candidates_comparison.json`
- F111 max=20: `/tmp/fire_max_candidates_expansion_sim/max20_f111.json`
- F111 max=50: `/tmp/fire_max_candidates_expansion_sim/max50_f111.json`
- F111 max=100: `/tmp/fire_max_candidates_expansion_sim/max100_f111.json`
- F111 max=200: `/tmp/fire_max_candidates_expansion_sim/max200_f111.json`
- F111 max=300: `/tmp/fire_max_candidates_expansion_sim/max300_f111.json`
- D42 max=50 sim doc: `~/fire-vault/04_daily/2026-07-10_demote_decision_9247_9628.md`

## §8 safety footer

- 本 simulation は **read-only analysis** (= DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0)
- 本 simulation は本実行ではない、HQ approve 後の別 wave で実行
- F111 staging read-only URI mode=ro のみ使用
- production / develop DB 接続なし
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- 100 株標準、要件表明: 100 株単位を標準、それ以外の株数調整は行わない
- forbidden phrase 全 0 件 (= grep 検証済)
- git add 0 / git commit 0 / git push 0 / --no-verify 0

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
