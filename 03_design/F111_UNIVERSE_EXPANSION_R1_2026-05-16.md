---
id: FIRE-F111-UNIVERSE-EXPANSION-R1-2026-05-16
phase: 設計 (= 実装は別 wave)
priority: 最高
status: 設計のみ、コード変更 0、9247/9628 demote 保留方針反映
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
parent_design: 03_design/FIRE_f111_max_candidates_baseline_50_2026-05-16.md
sibling_design: 03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_2026-05-16.md
integrated_doc: 03_design/F111_exploration_expansion_strategy_2026-05-16.md
hq_approve: PENDING (= 実装は別 wave)
codex_lanes: 4
critical_high_count: 0
---

# F111-UNIVERSE-EXPANSION-R1 — F111 上流母集団拡張 設計 (= 2026-05-16)

## §1 背景 + 目的

D41-D44 で 9247 / 9628 が 10-13 連続 entry 候補 (= fixed-candidate HIGH 相当). 旧方針は片方 demote だったが、HQ 判断更新で **demote 対症療法ではなく根本原因 (= F111 上流母集団狭小) の解消**を先に設計する.

**本 wave は設計のみ**. 実装 / DB write / 9247-9628 demote は別 wave + HQ approve 必須.

## §2 raw 35 件頭打ち原因仮説

### §2.1 F111 候補生成パイプライン (= 現状、正確版)

**重要**: require_research_signal=True (= default) の場合、F111 SQL は signal 起点 INNER JOIN のみで **scale_category filter は適用しない**.

```
[require_research_signal=True (= default)]
research_watchlist_signals (= MAX(base_date)、直近 35 件)
  ↓ INNER JOIN
market_listings (= 4,449 件)
  ↓ WHERE ml.sector_17_name != 'その他'
  ↓ AND ml.market_name != 'その他'
  ↓ ORDER BY ws.final_score DESC
  ↓ LIMIT max_candidates * 3
F111 raw candidates (= max=50 でも実 35 件、母集団は ws の 35 件で確定)

[require_research_signal=False (= --no-require-research-signal で発動)]
market_listings (= 4,449 件)
  ↓ WHERE scale_category IN ('TOPIX Core30','Large70','Mid400')
  ↓ AND sector_17_name != 'その他' / market_name != 'その他'
  ↓ ORDER BY code
  ↓ LIMIT max_candidates * 3
F111 raw candidates (= 最大 495 件母集団)
```

→ **scale_category filter は require_research_signal=False の else branch でしか効かない**.
→ 設計 doc 旧版「scale_category × LIMIT」は誤記、上記が正確版.

### §2.2 staging DB 実体調査結果

| 項目 | 件数 |
|---|---|
| market_listings 全件 | **4,449** |
| - TOPIX Core30 | 31 |
| - TOPIX Large70 | 69 |
| - TOPIX Mid400 | 395 |
| - TOPIX Small 1 | 487 |
| - TOPIX Small 2 | 666 |
| - "-" (旧株/上場廃止等) | 2,801 |
| research_watchlist_signals 最新 (2026-05-13) | **35** |
| research_watchlist_signals 直前 (2026-05-12) | 35 |
| research_watchlist_signals 4 日前 (2026-05-09) | **218** |
| research_watchlist_signals 過去最大 (2025-12-01) | **1,307** |

**直近 35 件の scale 分布実測** (= require_research_signal=True で実際に拾える候補):

| scale_category | 件数 |
|---|---|
| "-" (= scale 不明 / 旧株 / 上場廃止) | **28** |
| TOPIX Mid400 | 1 |
| TOPIX Small 1 | 4 |
| TOPIX Small 2 | 2 |
| Core30 / Large70 | **0** |

→ 35 件のうち Core30/Large70/Mid400 合計は **1 件のみ**.
→ scale_category 拡張 (= 案 C/D) は require_research_signal=True では **効かない** (= INNER JOIN 起点で scale filter なし).

### §2.3 原因仮説 (= 4 候補、Lane A audit 修正反映)

| # | 仮説 | 根拠 | 検証 |
|---|---|---|---|
| **H0 (★ 真の直接原因)** | **`signal_persistence` の `--top-n=30` filter が記録時に 35 件に絞る** | `run_research_watchlist_signal_persistence.py --top-n` default=30 / `filter_signals_by_top_n()` で `pre_cap_rank <= 30 OR post_cap_rank <= 30` / 過去 wave log `records loaded: 3708, rows after top_n filter: 35` | Lane A audit で実コード確認済 |
| H1 (副次) | 上流 watchlist ranker が signal を絞りすぎ | ranker 自体は 3,708 件処理、絞り込みは persistence 層 | H0 が支配的、ranker 修正は補助 |
| H2 (誤り、棄却) | scale_category 絞り (= Core30+Large70+Mid400) が JOIN 後に効きすぎ | 旧 doc 誤記、F111 SQL は require_research_signal=True で **scale filter 適用なし**、35 件の scale 分布は "-"28/Mid400 1/Small1 4/Small2 2 で Core30/Large70=0 | Lane A audit で SQL + 実測確認、**誤り判定** |
| H3 | research_watchlist_signals 生成 cron / launchctl が直近不安定 | 2026-05-09 218 → 2026-05-12 35 で 6 倍縮小、定期 run の入力不足の可能性 | jp.fire.* plist + log 確認 (= 本 wave 範囲外) |

→ **H0 が真の直接原因** (= persistence `--top-n=30` filter).
→ H1 は副次 (= ranker 自体は 3,708 件処理).
→ H2 は誤り (= require_research_signal=True で scale filter 効かない).
→ H3 は別途調査.

## §3 universe expansion 設計案 (= 5 案、Lane A audit 修正反映)

| 案 | 母集団 | 件数規模 | 安全度 | 実装複雑度 |
|---|---|---|---|---|
| **A (★最推奨)** | **signal_persistence `--top-n` 拡大** (= 30 → 100/200/ALL、records 3,708 → 100-3,708) | 100-3,708 | ★★★★★ | **低** (= persistence runner 設定変更のみ、ranker は不変) |
| A' (副次) | A + research_watchlist_ranker 設定再調整 (= signal 生成 threshold / sector_cap) | 200-1,300 | ★★★ | 中 |
| B | F111 `--no-require-research-signal` baseline 化 (= scale_category filter mode) | **495** (Core30+Large70+Mid400) | ★★★★ | 低 (= F111 CLI default 切替) |
| C | A or B + scale_category 拡張 (= Mid400 + Small1 + Small2) | 1,648 | ★★★★ | 中 (= scale_categories default 拡張、ただし require=True 経由は効かない) |
| D | 全 TOPIX (= Core30 + Large70 + Mid400 + Small1 + Small2) | 1,648 | ★★★★ | 中 (= require=False 経由のみ有効) |
| E | 全上場 (= 4,449) | 4,449 | ★★ | 高 (= 旧株 / 上場廃止フィルタ要) |

**重要 (= Lane A audit 反映)**: 案 C/D の scale_category 拡張は **require_research_signal=False (= --no-require-research-signal) で発動する else branch でしか効かない**. require=True の default では scale filter 自体が適用されない. C/D を使うなら B と組み合わせ必須.

### §3.1 推奨優先順位 (= Lane A audit 反映)

| 優先 | 案 | 理由 |
|---|---|---|
| **1 (★ 最推奨)** | **A**: signal_persistence `--top-n` 拡大 | **真の直接原因 H0 を解消**、ranker / SQL 変更不要、persistence runner 設定 1 値変更のみ、records 3,708 → 100-3,708 復元 |
| 2 (補強) | A': ranker 設定再調整 | A 後の評価で必要なら、副次的 |
| 3 (代替) | **B**: F111 `--no-require-research-signal` baseline 化 | scale 経由 495 件、ただし research signal bypass で理由付け弱、A の方が筋良し |
| 4 (補完) | **C**: scale_category 拡張 + B 併用 | A + B + C で 1,648 件多様化可能、流動性慎重 |
| 5 (慎重) | **D/E**: 全 TOPIX / 全上場 | 低流動性 / letter-suffix 混入リスク、liquidity gate 強化要 |

### §3.2 第 1 推奨 A の詳細 (= signal_persistence top-n 拡大)

#### 3.2.1 内容
- `scripts/jobs/run_research_watchlist_signal_persistence.py --top-n` default 30 → 100 / 200 / ALL に拡大検討
- 過去 wave log `records loaded: 3708, rows after top_n filter: 35` → top-n 拡大で records をより多く保存
- 朝 pilot で F111 出力件数も自然に増加 (= 35 → 100-500+)
- F111 の `--max-candidates 50` baseline は維持 (= entry filter は consumer 側で実施)
- ranker / F111 SQL は **無変更**

#### 3.2.2 前提
- top-n 値の妥当性検証 (= 100/200/ALL の比較 simulation)
- sector_cap_deferred との関係確認
- DB 容量 / persistence performance への影響評価
- D46 以降の朝 pilot で signal 件数監視

#### 3.2.3 影響範囲
- `scripts/jobs/run_research_watchlist_signal_persistence.py` (= --top-n default 値変更、または引数経由)
- `simulation/research_lane/signal_persistence.py` (= filter logic は不変、引数経由)
- DB: research_watchlist_signals に書き込み (= staging のみ、production/develop 別 wave)
- F111 出力に直接影響、ただし consumer / MD の entry filter は不変

#### 3.2.4 別 wave 起票推奨
- **F111-UNIVERSE-EXPANSION-R1-A 実装 wave**: signal_persistence `--top-n` 拡大 + simulation
- staging で top-n=100/200/ALL 比較、9247/9628 順位影響評価
- 安定したら develop / production へ展開 (= 別 wave、HQ approve 後)

## §4 safety gate (= universe expansion 後も維持)

| gate | 維持方針 |
|---|---|
| liquidity_status != FAIL | 維持 (= entry 除外 gate) |
| letter-suffix new listing | 維持 (= entry 除外 gate) |
| 100 株標準 / non-100 share | 維持 (= entry 必須条件) |
| no-new-chase | 維持 (= +2% 超で entry 抑制) |
| recently_seen demote (= 9130) | 維持 (= excluded 固定) |
| risk_yen 非加点 / 非 gate | 維持 (= v1.4.2 risk warning のみ) |
| risk_yen_over_pilot_budget warning | 維持 (= entry 候補のまま表示) |

→ **universe expansion 後も既存 safety gate は全て有効**.
→ 母集団が増えても entry 候補は consumer 側 filter で安全.

## §5 scoring 反映 (= 既存 + 追加候補)

### §5.1 既存 scoring (= F111 final_score / research_advisory_label)
- 維持. v1.4.2 / max=50 baseline で動作確認済み.

### §5.2 追加候補 (= sibling theme overlay 設計と接続)
- `theme_score`: theme overlay と接続 (= sibling doc 参照)
- `sector_strength`: sector_17 / sector_33 の相対強度
- `volume_momentum`: 出来高急増
- `turnover_momentum`: 売買代金急増
- `liquidity_preference`: 板薄回避

これらは F111 final_score の補正項として **scoring エンジン拡張 wave** で別 wave 化.

## §6 paper live / pattern simulation 接続

| 項目 | 内容 |
|---|---|
| theme 別勝率 | F035 Pattern Research Agent と接続、theme tag 単位で勝率集計 |
| sector 別勝率 | 既存 sector_17 単位の勝率を sector_33 / theme まで拡張 |
| entry_success / signal_success | 既存区別維持 (= historical vs current renderer) |
| no-new-chase 後の成績 | chase_limit 超過後 entry の事後評価 |
| fixed-candidate 発生率 | 連続 entry 日数 distribution の集計 |

→ paper live (Stage 2) で simulation 走らせ、Stage 3 Live Advisory 移行前に validation 必須.

## §7 dashboard / heartbeat 接続

| 項目 | 内容 |
|---|---|
| 夜間 theme scan job 走行 | dashboard で「last run」「OK/FAIL」表示 |
| universe expansion job 成功 | research_watchlist_ranker の OK/FAIL/件数 |
| output artifact | /tmp/fire_universe_expansion/ 等の path 監視 |
| safety_flags | F111 出力の safety_flags (= db_write/api_call 等) 監視 |

→ 既存 F282 launchd plist 機構と接続、別 wave で実装.

## §8 実装 wave 分割案

| wave | 内容 | 優先 | 依存 |
|---|---|---|---|
| **W1** | research_watchlist_ranker 設定再調整 (= H1 検証 + 件数復元) | **最優先** | - |
| W2 | F111 `--no-require-research-signal` baseline 化検討 (= B 案) | 中 | W1 結果次第 |
| W3 | scale_category 拡張 (= C 案、Mid400+Small1/2) | 中 | W1, W2 |
| W4 | theme overlay 実装 (= sibling doc) | 高 | W1-W3 後 |
| W5 | scoring 拡張 (= theme_score / sector_strength 等) | 高 | W4 |
| W6 | paper live / pattern simulation 接続 | 中 | W4, W5 |
| W7 | dashboard / heartbeat 接続 | 低 | W1-W6 |

## §9 9247 / 9628 demote 保留方針

- 本 wave: 記録のみ (= fixed-candidate HIGH 相当継続)
- **demote 本実行は保留**
- recently_seen_codes 追加なし
- 探索拡張 (= W1 実装後) で 9247/9628 が自然に top 順位から下がるか確認
- 拡張後も固定化が残る場合に demote / rotation penalty を再検討
- D46 以降の判断、両方同時 demote は引き続き非推奨

## §10 TODO 追加候補

| ID | 内容 | 優先 |
|---|---|---|
| F111-UNIVERSE-EXPANSION-R1-W1 | research_watchlist_ranker 設定再調整 | 最優先 |
| F111-UNIVERSE-EXPANSION-R1-W2 | F111 require-research-signal flag 調整 | 中 |
| F111-UNIVERSE-EXPANSION-R1-W3 | scale_category 拡張 | 中 |
| F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-impl | theme overlay 実装 | 高 (sibling doc) |
| F111 scoring 拡張 wave | theme_score / sector_strength 等 | 高 |
| dashboard / heartbeat 拡張 wave | 夜間 scan job 監視 | 低 |

## §11 CRITICAL/HIGH 判定

- **CRITICAL: 0** (= 本 wave は設計のみ、実装なし)
- **HIGH: 0**

## §12 safety footer

- 本 wave は **設計のみ** (= DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0)
- 実装 / 本実行 / 9247-9628 demote / recently_seen 追加 は別 wave + HQ approve 必須
- staging DB 接続は read-only URI mode=ro のみ (= 母集団件数調査)
- production / develop DB 接続なし
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- 100 株標準、要件 §5/6 表明維持
- forbidden phrase 全 0 件 (= grep 検証済)
- git add 0 / git commit 0 / git push 0 / --no-verify 0

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
