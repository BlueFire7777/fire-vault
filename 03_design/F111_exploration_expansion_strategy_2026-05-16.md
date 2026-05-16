---
id: FIRE-F111-EXPLORATION-EXPANSION-STRATEGY-2026-05-16
phase: 統合設計 doc (= F111 探索拡張戦略全体像)
priority: 最高
status: 設計統合、9247/9628 demote 保留
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
sibling_universe: 03_design/F111_UNIVERSE_EXPANSION_R1_2026-05-16.md
sibling_theme: 03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_2026-05-16.md
codex_lanes: 4
critical_high_count: 0
---

# FIRE F111 exploration expansion strategy (= 2026-05-16)

## §1 統合戦略 (= sibling 2 doc の上位)

D41-D44 で 9247/9628 fixed-candidate HIGH 相当. HQ 判断更新で demote 対症療法ではなく、
**F111 探索拡張 (= universe + theme overlay)** で根本解消を狙う.

本 doc は 2 つの sibling doc を統合:
- `F111_UNIVERSE_EXPANSION_R1`: 上流母集団拡張
- `F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1`: theme overlay (= seed + dynamic discovery)

## §2 raw 35 件頭打ち根本原因サマリ (= Lane A audit 修正反映)

| 層 | 件数 | 原因 |
|---|---|---|
| market_listings 全件 | 4,449 | 全上場 |
| TOPIX (Core30 + Large70 + Mid400) | 495 | scale category 分布 (= 旧設計 doc の誤記、現 F111 SQL は require=True で scale filter 適用しない) |
| research_lane records loaded | 3,708 | ranker 自体は 3,708 件処理 |
| **signal_persistence `--top-n=30` filter 適用後** | **35** | **真の直接原因: persistence の top-n filter (= pre_cap_rank or post_cap_rank ≤ 30 + sector_cap_deferred 5 件で 35)** |
| F111 raw candidates | 35 | INNER JOIN with research_watchlist_signals (= 直近 35 件で確定) |

→ **真の根本原因**: signal_persistence の `--top-n=30` filter (= H0). ranker (= H1) は副次.
→ F111 SQL は require_research_signal=True で scale filter 適用なし (= 旧仮説 H2 は誤り、Lane A audit で訂正).

## §3 段階的実装ロードマップ (= 統合、Lane A audit 反映)

| Phase | wave | 内容 | 優先 | 期待効果 |
|---|---|---|---|---|
| **P1 (★)** | UNIVERSE-EXPANSION-R1-W1 | **signal_persistence `--top-n` 拡大** (= 30 → 100/200/ALL) | **最優先** | raw 35 → 100-3,708 |
| P2 | THEME-OVERLAY-R1-W1+W2+W3 | seed theme list + table + F111 出力拡張 | 高 | sector_17 → theme tag 粒度 |
| P3 | UNIVERSE-EXPANSION-R1-W3 | scale_category 拡張 (Mid400+Small1/2) | 中 | raw 200-500 → 500-1,000+ |
| P4 | THEME-OVERLAY-R1-W5-W7 | dynamic discovery (= A/B/C) | 中 | 新 theme 自動拡張 |
| P5 | scoring 拡張 wave | theme_score / sector_strength 加算 | 中 | 9247/9628 順位自然低下 |
| P6 | paper live validation wave | theme 別勝率 / signal_success | 中 | Stage 3 移行前 validation |
| P7 | dashboard / heartbeat 接続 | F282 plist 機構連携 | 低 | 運用監視 |

## §4 9247 / 9628 固定化解消の論理パス

```
[現状] research_watchlist_signals 35 件
  9247 / 9628 が情報通信 14 件中の top 2 (= 集中)
  ↓
[P1 後] research_watchlist_signals 200-500+ 件
  情報通信 sector 候補 14 → 50+ 件
  9247 / 9628 が相対順位低下、固定化自然解消の可能性
  ↓
[P2 後] theme overlay 適用
  情報通信 sector → 「サービス業」「葬祭」「IT インフラ」等 細分 theme に分離
  9247 / 9628 が「ニッチ theme」で top に残るが、別 theme から候補浮上
  ↓
[P3-P5 後] scale + scoring 拡張
  全 entry 候補数 14 → 30-50+
  Fujiwara 朝判断は top 5 圧縮継続
  9247 / 9628 demote 不要可能性
  ↓
[P5 後も固定化残るなら] demote / rotation penalty 再検討
  HQ approve 後の別 wave
```

## §5 9247 / 9628 demote 保留方針 (= 明示)

| 項目 | 方針 |
|---|---|
| 本 wave での demote 本実行 | **しない** |
| recently_seen_codes 追加 | **しない** |
| fixed-candidate HIGH 記録 | する (= D41-D44 で既記録) |
| P1 (= ranker 再調整) 後の再判定 | **必須** |
| P2-P5 後の再判定 | 必須 |
| demote 本実行 (= P5 後も固定化残る場合) | HQ / Fujiwara 承認必須、別 wave |
| 両方同時 demote | **原則非推奨** (継続) |
| 単独 demote (= sim A or B) | P1-P5 後の再判断材料が出てから検討 |

## §6 safety gate (= 全 phase で維持)

| gate | 方針 |
|---|---|
| liquidity_status != FAIL | 維持 |
| letter-suffix new listing | 維持 |
| 100 株標準 / non-100 share | 維持 |
| no-new-chase | 維持 |
| recently_seen demote (= 9130) | 維持 |
| risk_yen 非加点 / 非 gate | 維持 |
| risk_yen_over_pilot_budget warning | 維持 |
| theme tag は entry gate にしない | sibling doc §4 と整合 |

## §7 TODO 追加候補 (= sibling 2 doc 集約)

### §7.1 UNIVERSE EXPANSION R1
- F111-UNIVERSE-EXPANSION-R1-W1 (research_watchlist_ranker 再調整) **最優先**
- F111-UNIVERSE-EXPANSION-R1-W2 (F111 require-research-signal flag 調整) 中
- F111-UNIVERSE-EXPANSION-R1-W3 (scale_category 拡張) 中

### §7.2 THEME SECTOR DYNAMIC OVERLAY R1
- F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 (seed theme list + runner) **最優先**
- F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2 (theme_tags table) 中
- F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W3 (F111 output 拡張) 中
- F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W4 (consumer / MD 拡張) 中
- F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W5 (dynamic discovery A 業種) 中
- F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W6 (dynamic discovery C 出来高) 低

### §7.3 scoring / validation / monitoring
- F111 scoring 拡張 wave (= theme_score / sector_strength 等) 中
- paper live validation wave (= theme 別勝率) 中
- dashboard / heartbeat 接続 wave 低

## §8 D46 以降の推奨着手順序

| 日 | 内容 |
|---|---|
| D46 (= 2026-07-16 木) | 朝 pilot 継続 + **F111-UNIVERSE-EXPANSION-R1-W1 着手判断** (= HQ approve 後) |
| D47 | UNIVERSE-EXPANSION-R1-W1 実装 / simulation (= ranker 設定再調整、staging のみ) |
| D48 | W1 結果評価 (= raw 件数復元か、9247/9628 順位変化) |
| D49 | THEME-OVERLAY-R1-W1 着手 (= seed theme list + runner simulation) |
| D50+ | 残り phase の順次着手 |

## §9 CRITICAL/HIGH

- **CRITICAL: 0**
- **HIGH: 0**

## §10 safety footer

- 本 wave は **統合設計 doc のみ** (= 実装 / DB write / 本実行は別 wave + HQ approve)
- DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0
- production / develop DB 接続なし、staging read-only URI mode=ro のみ
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- 100 株標準維持、theme tag は entry gate に使わない
- forbidden phrase 全 0 件
- git add 0 / git commit 0 / git push 0 / --no-verify 0
- 9247 / 9628 demote 本実行は保留 (= P1-P5 後の再判断後、HQ approve 必須)
- 両方同時 demote は原則非推奨継続
- 固定 10 テーマ限定無、seed + dynamic discovery

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
