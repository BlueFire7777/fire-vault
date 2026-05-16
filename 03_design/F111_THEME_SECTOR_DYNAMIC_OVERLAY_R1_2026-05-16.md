---
id: FIRE-F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-2026-05-16
phase: 設計 (= 実装は別 wave)
priority: 最高
status: 設計のみ、コード変更 0、固定 10 テーマ限定無 + seed + dynamic discovery
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
parent_design: 03_design/FIRE_f111_max_candidates_baseline_50_2026-05-16.md
sibling_design: 03_design/F111_UNIVERSE_EXPANSION_R1_2026-05-16.md
integrated_doc: 03_design/F111_exploration_expansion_strategy_2026-05-16.md
hq_approve: PENDING
codex_lanes: 4
critical_high_count: 0
---

# F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1 — theme overlay 設計 (= 2026-05-16)

## §1 背景 + 目的

D43-D44 max=50 baseline で sector entry 上 7 種に多様化したが、**sector_17 (= 17 業種) は粗い**.
Fujiwara が朝みる時、市場テーマ単位 (= 半導体 / 電線 / AI データセンター / 防衛 / 銀行 等) の整理が必要.

**本 wave は設計のみ**. 実装 / theme tag 付与 / DB write は別 wave + HQ approve.

## §2 既存 staging DB 調査結果

| field | 存在 | 内容 |
|---|---|---|
| `market_listings.sector_17_name` | ✓ | 17 業種 (= 情報通信、機械、食品 等) |
| `market_listings.sector_33_name` | ✓ | 33 業種 (= 化学、機械、食料品、銀行業 等) |
| `market_listings.market_code` / `market_name` | ✓ | TOPIX / プライム / スタンダード等 |
| `market_listings.margin_code` / `margin_name` | ✓ | 信用区分 |
| `market_listings.scale_category` | ✓ | Core30 / Large70 / Mid400 / Small1 / Small2 |
| **theme** field | **不在** | - |
| **business_label** field | **不在** | - |
| **theme_tag** field | **不在** | - |

→ **theme overlay は完全未実装**. sector_33 で「化学」「機械」等は拾えるが、「半導体」「AI データセンター」「防衛」「電線」等の市場テーマ単位は粗い (= 化学に半導体材料も化粧品も混ざる).

## §3 設計方針: seed + dynamic discovery (= 固定 10 テーマ限定無)

### §3.1 seed theme 例 (= 起点、20+ 種、拡張可能)

| # | theme | 主要 sector_33 mapping | 銘柄例 |
|---|---|---|---|
| 1 | 半導体 | 電気機器 / 化学 (= 半導体材料) | 東京エレクトロン、レーザーテック、信越化学 |
| 2 | 電線 | 非鉄金属 / 電気機器 | 古河電工、フジクラ、住友電工 |
| 3 | AI データセンター | 情報通信 / 電気機器 / 不動産 (= DC REIT) | さくらインターネット |
| 4 | 電力インフラ | 電気・ガス / 機械 / 電気機器 | 関西電力、日立、東芝 |
| 5 | 防衛 | 輸送用機器 / 機械 / 電気機器 | 三菱重工、IHI、川崎重工 |
| 6 | 銀行 | 銀行業 | 三菱 UFJ、三井住友 FG、みずほ FG |
| 7 | 商社 | 卸売業 | 三菱商事、三井物産、伊藤忠 |
| 8 | 機械 | 機械 / 輸送用機器 | コマツ、ファナック |
| 9 | 電機 | 電気機器 / 精密機器 | キーエンス、ソニー G |
| 10 | 非鉄金属 | 非鉄金属 (= sector_17 鉄鋼・非鉄) | 住友金属鉱山、三井金属 |
| 11 | 海運 | 海運業 | 日本郵船、商船三井、川崎汽船 |
| 12 | 不動産 | 不動産業 | 三井不動産、住友不動産、三菱地所 |
| 13 | 食品 | 食料品 | 味の素、キッコーマン |
| 14 | 医薬 | 医薬品 | 武田薬品、エーザイ |
| 15 | 素材化学 | 化学 (= 半導体以外) | 旭化成、信越化学、住友化学 |
| 16 | サイバーセキュリティ | 情報通信・サービスその他 | トレンドマイクロ、ラック |
| 17 | 空調・冷却 | 機械 / 電気機器 | ダイキン、富士電機 |
| 18 | 光ファイバー | 非鉄金属 / 電気機器 | フジクラ、古河電工 |
| 19 | インバウンド | 小売 / サービス / 運輸 | JR 東海、三越伊勢丹 HD |
| 20 | ゲーム / IP | 情報通信 / その他製品 | カプコン、任天堂、サンリオ |

(seed は逐次追加可能、固定 10 テーマ限定しない)

### §3.2 dynamic discovery (= 動的補完)

| 方法 | 内容 | データソース |
|---|---|---|
| **A. 業種コード mapping** | sector_33 + 銘柄名キーワードで自動 tag | market_listings.sector_33_name + company_name |
| **B. ニュース解析** | 銘柄ニュースから theme tag 抽出 (= LLM or rule-based) | news_articles table (= 別 wave で取り込み) |
| **C. 出来高急増 + sector_33 相対強度** | 急騰銘柄の sector を tag、新 theme として登録 | research_watchlist_signals.final_score / volume_momentum |
| **D. 業績タグ** | 「半導体製造装置」「AI」等の業績 description から抽出 | jquants_quarterly_results 等 (= 既存 import 利用) |

→ **seed + A (= 最小実装) で初期版を作成、後段で B-D を順次追加**.

### §3.3 theme tag 付与方法

| step | 内容 |
|---|---|
| 1 | `theme_tags` table 新規追加 (= staging DB) |
| 2 | seed list を JSON / YAML / DB に保存 (= 拡張容易) |
| 3 | `assign_theme_tags` runner (= 銘柄 → theme list mapping) |
| 4 | F111 出力 field に `theme_tags` 配列を追加 |
| 5 | consumer / MD レンダラに theme tag 表示拡張 |

### §3.4 theme strength 計算

| 指標 | 内容 |
|---|---|
| `theme_relative_strength` | 当該 theme 銘柄群の平均 final_score / 全市場平均 |
| `theme_volume_momentum` | theme 銘柄群の出来高急増率 |
| `theme_consecutive_days` | theme としての連続 strong 日数 |

→ theme tag は **entry 確定条件ではなく、scoring / explanation / watchlist expansion に使う**.

## §4 safety gate (= theme overlay 後も維持)

| gate | 維持方針 |
|---|---|
| liquidity_status != FAIL | 維持 (= theme tag があっても entry 除外) |
| letter-suffix new listing | 維持 |
| 100 株標準 / non-100 share | 維持 |
| no-new-chase | 維持 |
| recently_seen demote (= 9130) | 維持 |
| risk_yen 非加点 / 非 gate | 維持 (= theme tag も risk gate に使わない) |
| risk_yen_over_pilot_budget warning | 維持 |

→ **theme overlay は補助情報**、entry filter には使わない.

## §5 scoring 反映

### §5.1 theme tag は **entry 除外 gate にしない**
- v1.4.2 方針継続: theme tag が無くても entry 可能
- theme tag は scoring 補正 + Fujiwara 朝判断補助のみ

### §5.2 theme_score 加算案
- 既存 final_score に `theme_score` を補正項として加算
- 重み調整は paper live (Stage 2) で validation
- 加算前後で 9247/9628 順位がどう動くか simulation 必須 (= 別 wave)

## §6 paper live / pattern simulation 接続

| 項目 | 内容 |
|---|---|
| theme 別勝率 | theme tag 単位で entry_success / signal_success 集計 |
| sector_33 別勝率 | sector_33 単位の勝率を併存集計 |
| theme dynamic discovery 効果 | 新 theme tag が paper live で勝率向上に寄与するか |
| no-new-chase 後の成績 | theme 別の chase_limit 後 entry 評価 |

→ Stage 3 Live Advisory 移行前に validation 必須.

## §7 dashboard / heartbeat 接続

| 項目 | 内容 |
|---|---|
| theme scan job 走行 | 夜間 / 朝定期で theme tag 更新 job |
| dynamic discovery 新 theme | 新規 theme tag の自動登録通知 |
| theme strength 急変 | sector_17 等の急変通知 |

→ F282 plist 機構と接続、別 wave で実装.

## §8 9247 / 9628 demote 保留方針 (= sibling doc と同)

- 本 wave: 記録のみ
- demote 本実行は保留
- theme overlay 実装後に 9247 / 9628 が「サービス業」「葬祭」等の細分 theme に分離可能か確認
- 分離後も固定化が残る場合に demote / rotation penalty を再検討

## §9 実装 wave 分割案

| wave | 内容 | 優先 | 依存 |
|---|---|---|---|
| **W1** | seed theme JSON / YAML 作成 + assign_theme_tags runner (= simulation のみ) | **最優先** | - |
| W2 | theme_tags table 追加 (= staging DB schema) | 中 | W1 |
| W3 | F111 出力 field 拡張 (= theme_tags array) | 中 | W2 |
| W4 | consumer / MD レンダラ theme tag 表示 | 中 | W3 |
| W5 | dynamic discovery (= A 業種 mapping) 実装 | 中 | W1-W4 |
| W6 | dynamic discovery (= C 出来高 + 相対強度) 実装 | 低 | W5 |
| W7 | dynamic discovery (= B ニュース解析) 実装 | 低 | W6 |
| W8 | theme_score 加算 (= scoring 拡張) | 中 | W3-W6 |
| W9 | paper live / pattern simulation 接続 | 中 | W8 |
| W10 | dashboard / heartbeat 接続 | 低 | W1-W9 |

## §10 TODO 追加候補

| ID | 内容 | 優先 |
|---|---|---|
| F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 | seed theme list + assign runner (simulation) | 最優先 |
| F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2 | theme_tags table schema | 中 |
| F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W3 | F111 output 拡張 | 中 |
| F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W4 | consumer / MD 拡張 | 中 |
| F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W5 | dynamic discovery A 業種 | 中 |
| F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W6 | dynamic discovery C 出来高 | 低 |
| F111 scoring 拡張 wave (= theme_score) | scoring | 中 |
| paper live validation wave (= theme 別勝率) | validation | 中 |

## §11 CRITICAL/HIGH 判定

- **CRITICAL: 0** (= 本 wave は設計のみ、実装なし)
- **HIGH: 0**

## §12 safety footer

- 本 wave は **設計のみ** (= DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0)
- 実装 / theme tag 付与 / DB write は別 wave + HQ approve 必須
- staging DB 接続は read-only URI mode=ro のみ (= field 調査)
- production / develop DB 接続なし
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- 100 株標準、要件 §5/6 表明維持
- forbidden phrase 全 0 件 (= grep 検証済)
- git add 0 / git commit 0 / git push 0 / --no-verify 0
- 固定 10 テーマ限定無、seed + dynamic discovery 方式
- theme tag は entry 除外 gate に使わない (= v1.4.2 risk_yen 方針と整合)

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
