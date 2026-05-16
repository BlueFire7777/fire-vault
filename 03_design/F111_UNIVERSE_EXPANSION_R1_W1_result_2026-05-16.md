---
id: FIRE-F111-UNIVERSE-EXPANSION-R1-W1-RESULT-2026-05-16
phase: 検証結果 doc (= W1 完了)
priority: 最高
status: read-only simulation 完了、本実行は別 wave + HQ approve
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
parent_design: 03_design/F111_UNIVERSE_EXPANSION_R1_2026-05-16.md
sibling: 03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_2026-05-16.md
audit_doc: /tmp/fire_f111_universe_expansion_r1_w1/jquants_staging_universe_audit.md
comparison_doc: /tmp/fire_f111_universe_expansion_r1_w1/universe_expansion_comparison.md
codex_lanes: 8
critical_high_count: 0
---

# F111-UNIVERSE-EXPANSION-R1-W1 検証結果 (= 2026-05-16)

## §1 検証目的 + 方法

- signal_persistence `--top-n=30` 制約を 30/100/200/ALL の 4 通りで simulation
- staging URI mode=ro / DB write 0 / API 0 / token 0
- `_fetch_records` + `build_watchlist` + `convert_watchlist_to_signal_row` + `filter_signals_by_top_n` を Python から直接呼び出し
- 完了基準: 9247/9628 順位影響 + sector 多様化 + 安全 gate 維持 確認

## §2 検証結果サマリ

### §2.1 母集団拡張効果

| top-n | persistence 全 | sector 数 | letter-suffix |
|---|---|---|---|
| 30 (= 現 baseline) | **35** | 12 | 6 |
| 100 | **109** | 15 | 13 |
| 200 | **233** | 16 | 20 |
| ALL | **3,708** | **17** | 128 |

→ 母集団は **最大 106 倍** (= 35 → 3,708) 拡張可能、sector_17 全種網羅可能.

### §2.2 9247 / 9628 / 9130 順位

| code | rank | sector | scale | final_score |
|---|---|---|---|---|
| **9130** 共栄タンカー | **8** | 運輸・物流 | - | 0.8586 |
| **9247** ＴＲＥ HD | **11** | 情報通信・サービスその他 | TOPIX Small 1 | 0.8546 |
| **9628** 燦 HD | **13** | 情報通信・サービスその他 | TOPIX Small 2 | 0.8479 |

→ 9247/9628 は **score 順位が高位継続**. universe 拡張 (= top-n 増加) **だけでは順位低下なし**.
→ score 上位 8 件全て (= 8/8) が recently_seen_demoted (= 8747/5729/3489/340A0/3798/137A0/7991/9130 = v1.4.2 recently_seen_codes 8 件版そのもの). consumer 側で excluded 化される.

### §2.3 4404 順位

- top-n=ALL でも score top 15 内に出現なし
- 4404 = ミヨシ油脂 (食品 sector) → 食品 sector の score 順位は score 0.85 以下
- F111 entry 14 件に出ている理由は consumer filter 後の事情 (= research_advisory_label / liquidity 等)

## §3 真の根本原因の再確認

**signal_persistence `--top-n=30`** が universe 35 件頭打ちの直接原因 (= 設計 wave 仮説 H0) → **本 W1 で実証**:
- records loaded: 3,708 件
- top-n=30 → 35 件 (= 既存と一致)
- top-n=ALL → 3,708 件 (= 拡張可能)
- → 拡張は技術的に可能、安全 gate 維持

ただし、9247/9628 順位は score 自体が高位なので、**universe 拡張だけでは固定化解消しない**.

## §4 安全 gate 維持確認 (= 全 top-n)

| gate | 結果 |
|---|---|
| 低流動性 (liquidity_fail) entry 混入 | **0** ✓ (= consumer 側 filter) |
| letter-suffix entry 混入 | **0** ✓ (= excluded) |
| 非 100 株 entry 混入 | **0** ✓ |
| 9130 recently_seen demote | 維持 ✓ |
| 9130 excluded (= consumer 側) | 維持 ✓ |
| risk_yen 非加点 / 非 gate | 維持 ✓ |
| 100 株標準 | 維持 ✓ |
| 9247/9628 demote 本実行 | 保留 ✓ (= 本 wave 範囲外) |

## §5 推奨実装 (= 別 wave、HQ approve 後)

### §5.1 W2: top-n=100 baseline 化 (= 最推奨)

- `run_research_watchlist_signal_persistence.py --top-n` default 30 → 100
- 母集団 35 → 109 (= 3 倍)
- sector 12 → 15 (= +3)
- 安全 gate 完全維持
- runtime 影響なし (= ~200ms 一定)
- 朝判断負荷 → consumer 側 top 5 圧縮継続で対応

### §5.2 W3: top-n=200 検討 (= 次点)

- 母集団 109 → 233 (= 6.7x)
- sector 15 → 16 (= +1)
- consumer / MD レンダラの大量 candidate 処理確認要

### §5.3 W4: F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1 (= sibling)

- 9247/9628 score 順位対応の根本解
- seed 20 種 + dynamic discovery

## §6 J-Quants/staging universe audit 結果 (= 別 doc 参照)

- staging DB に **大量 J-Quants 由来 data 既存** (market_listings 4,449 + market_prices_daily 2.08M + financials_v2 164K + announcements 1,098 等)
- **API 0 で universe 拡張可能** (= signal_persistence top-n 拡大で完結)
- theme overlay は seed list + sector mapping で staging 内実装可能
- 業績 description / news theme discovery は別 wave (= API 承認後)

## §7 次 wave 候補

| ID | 内容 | 優先 | 依存 |
|---|---|---|---|
| F111-UNIVERSE-EXPANSION-R1-W2 | top-n=100 baseline 化実装 | **最優先** | HQ approve |
| F111-UNIVERSE-EXPANSION-R1-W3 | top-n=200 検証 + 必要なら baseline 化 | 中 | W2 結果 |
| F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 | seed theme list + assign runner | 高 | W2 並行 |
| F111 scoring 拡張 wave (= theme_score) | scoring 補正 | 中 | OVERLAY-W1 |
| F111-JQUANTS-DESCRIPTION-FETCH-R1 | 業績 description 取得 | 低 | API approve |
| F111-NEWS-THEME-DISCOVERY-R1 | ニュース theme 動的発見 | 低 | API approve |

## §8 CRITICAL/HIGH 判定

- **CRITICAL: 0**
- **HIGH: 0**

## §9 safety footer

- 本 wave は **read-only simulation + audit** (= DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0 / **plist 0 / cron 0 / workflow 0**)
- v1.4.2 policy 維持確認: risk cap gate 撤廃 / risk_yen 非加点 / risk_yen_over_pilot_budget warning 表示のみ / risk_yen_warning entry 維持 / 100 株標準 / no-new-chase
- 半導体 / 電線 / AI データセンター / 電力インフラ / 防衛 等の市場テーマは sector_17 17 業種では拾えず、theme overlay 別 wave (= F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1) 必須
- staging URI mode=ro のみ使用 (= production/develop 接続なし)
- top-n 拡張本番 baseline 化は別 wave + HQ approve 必須
- F111 default 変更なし (= cli=1.3.0 / max=50 維持)
- 9247/9628 demote 本実行は保留継続
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- 100 株標準維持、要件 §5/6 表明
- forbidden phrase 全 0 件
- git add 0 / git commit 0 / git push 0 / --no-verify 0

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)

## §10 関連 file

- 親設計: `~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_2026-05-16.md`
- sibling (theme): `~/fire-vault/03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_2026-05-16.md`
- 統合 strategy: `~/fire-vault/03_design/F111_exploration_expansion_strategy_2026-05-16.md`
- 比較 MD: `/tmp/fire_f111_universe_expansion_r1_w1/universe_expansion_comparison.md`
- 比較 JSON: `/tmp/fire_f111_universe_expansion_r1_w1/universe_expansion_comparison.json`
- J-Quants audit: `/tmp/fire_f111_universe_expansion_r1_w1/jquants_staging_universe_audit.md`
- top-n=30/100/200/ALL signal_persistence JSON 4 file
