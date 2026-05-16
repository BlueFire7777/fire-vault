---
id: FIRE-pilot-D37-sector-diversification-sim-2026-07-03
phase: 本番 v0 / W60-pilot-D37 / sector 多様化 simulation (= read-only analysis)
priority: 高
status: simulation 実行済、baseline は変更しない
owner: BlueFire7777 (Fujiwara)
date: 2026-07-03 (= D37、金)
parent_doc: 04_daily/2026-07-02_demote_execution_9130.md (= D36 demote 本実行)
---

# FIRE W60-pilot-D37 — sector 多様化 simulation (= 2026-07-03)

## §1 経緯

D36 で 9130 demote 本実行が完了し、D36/D37 共に entry 候補が **情報通信・サービスその他 2 件のみ**
(= 9247, 9628) で **sector 集中化**が継続. D37 でこのリスクを評価するため、F111 max_candidates を
baseline (= 20) → simulation (= 50) に拡張し、別 sector 候補が浮上するか確認.

**simulation は read-only analysis**、baseline は変更しない. max_candidates 拡張 logic を本実行
する場合は HQ approve 後の別 wave で実施.

## §2 simulation 実行 inputs

| 項目 | baseline (= D37 正本) | sector 多様化 simulation |
|---|---|---|
| base_date | 2026-07-03 | 2026-07-03 |
| recently_seen_codes | 8747, 5729, 3489, 340A0, 3798, 137A0, 7991, 9130 | 同 (= 8 件版) |
| demoted_count | 8 | 8 |
| **max_candidates** | **20** | **50** |
| label_threshold_mode | strict | strict |
| output | d37_f111.json | d37_f111_max_candidates_sim.json |

→ **唯一の差分は max_candidates** (= 20 vs 50)、recently_seen は同一.

## §3 simulation 結果比較

### 3.1 候補件数 + role 集計

| 項目 | baseline (max=20) | simulation (max=50) |
|---|---|---|
| 全 candidates | 20 | 35 |
| entry_candidate | 2 | **10** |
| watch_candidate | 1 | 1 |
| excluded_candidate | 17 | 24 |

### 3.2 sim entry 候補一覧 (= sector 多様化確認)

| code | name | sector_17 | margin | scale | close | risk_yen | liq | letter | 状態 |
|---|---|---|---|---|---|---|---|---|---|
| 9247 | ＴＲＥ HD | 情報通信・サービスその他 | 貸借 | TOPIX Small 1 | 1,612 | 8,060 | PASS | False | 既存 |
| 9628 | 燦 HD | 情報通信・サービスその他 | 貸借 | TOPIX Small 2 | 1,390 | 6,950 | PASS | False | 既存 |
| **2146** | ＵＴグループ | 情報通信・サービスその他 | 貸借 | TOPIX Small 1 | 184 | 920 | PASS | False | ★ 新規 |
| **4828** | ビジネスエンジニアリング | 情報通信・サービスその他 | 貸借 | TOPIX Small 2 | 1,246 | 6,230 | PASS | False | ★ 新規 |
| **8699** | ＨＳ HD | **金融（除く銀行）** | 貸借 | - | 1,140 | 5,700 | PASS | False | ★ 新規 |
| **3089** | テクノアルファ | **商社・卸売** | 貸借 | - | 1,055 | 5,275 | PASS | False | ★ 新規 |
| **3712** | 情報企画 | 情報通信・サービスその他 | 貸借 | - | 1,039 | 5,195 | PASS | False | ★ 新規 |
| **7803** | ブシロード | 情報通信・サービスその他 | 貸借 | - | 261 | 1,305 | PASS | False | ★ 新規 |
| **9633** | 東京テアトル | **不動産** | 貸借 | - | 1,579 | 7,895 | PASS | False | ★ 新規 |
| **4914** | 高砂香料工業 | **素材・化学** | 貸借 | TOPIX Small 1 | 1,180 | 5,900 | PASS | False | ★ 新規 |

→ **8 件の新規 entry 候補が浮上**、全 100 株 / 貸借 / PASS / letter=False ✓.

### 3.3 sim entry sector 分布

| sector | 件数 | codes |
|---|---|---|
| 情報通信・サービスその他 | 6 | 9247, 9628, 2146, 4828, 3712, 7803 |
| 金融（除く銀行） | 1 | 8699 |
| 商社・卸売 | 1 | 3089 |
| 不動産 | 1 | 9633 |
| 素材・化学 | 1 | 4914 |

→ **5 sector へ多様化** (= baseline 1 sector のみ → simulation 5 sector).

## §4 sim 結果評価

### 4.1 sector 多様化効果

| 観点 | baseline (max=20) | simulation (max=50) | 評価 |
|---|---|---|---|
| sector 数 | 1 (情報通信・サービスその他) | 5 (情報通信 + 金融 + 商社 + 不動産 + 素材) | **大幅多様化** ✓ |
| entry 件数 | 2 | 10 | 5 倍増 |
| 新規 entry 浮上 | - | 8 件 | 多様化候補豊富 |
| 100 株適性 | 全 True | 全 True | 維持 ✓ |
| 流動性 (PASS) | 全 PASS | 全 PASS | 維持 ✓ |
| 信用区分 (貸借) | 全 貸借 | 全 貸借 | 維持 ✓ |
| letter-suffix entry | 0 | 0 | 維持 ✓ |
| 低流動性 entry 浮上 | - | **なし** | 維持 ✓ |

### 4.2 効果 vs リスク

**効果 (= 安全側)**:
- sector 多様化が実現 (= 1 → 5 sector)
- 全 新規 entry が 100 株 / 貸借 / PASS で適性維持
- 低流動性 / letter-suffix / 100 株不適候補は entry に浮上していない

**リスク (= 慎重判断要)**:
- entry 候補が 2 → 10 件に増加 (= Fujiwara の Top N 選択判断が必要)
- Top N 選択 logic (= 既存) で 10 件から実際に entry 検討する N 件を絞る必要
- max_candidates=50 を baseline 化する場合、F111 実行時間 / メモリ使用量増加 (= 確認要)

## §5 D38 以降の提案

### 5.1 短期 (= D37 当日)

- baseline (= max=20) を **D37 朝の正本** として使用 (= 既存 invariants 維持)
- 本 simulation 結果は **read-only 参考情報** (= Fujiwara が sector 多様化方針を判断する材料)
- Fujiwara が同 sector 重複を許容する場合: 9247 / 9628 のいずれか 1 件に絞って entry 検討
- Fujiwara が sector 多様化を希望する場合: simulation 結果から 1-2 件を **手動選定**

### 5.2 中期 (= D38 以降の別 wave)

- max_candidates=50 を **baseline 化検討** (= HQ approve 後、別 wave で実施)
  - 効果: sector 多様化、entry 選択肢の質的向上
  - 確認: F111 実行時間 / メモリ使用量
  - tests: max_candidates パラメータの境界値テスト

### 5.3 長期 (= 別 wave、要 HQ approve)

- max_candidates パラメータを env / config / runner default で永続化
- F111 runner core で sector 多様化 logic を組み込み検討 (= 既存 sector cap 30% logic と整合)
- Top N 選択 logic で多様化を考慮した自動絞り込み

## §6 safety footer

- 本 simulation は **read-only analysis** (= DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0)
- baseline (= D37 正本) は変更しない (= F111 max_candidates=20 で実行済)
- 本 simulation 結果を本番反映する場合は HQ approve 後の別 wave で実施
- F111 staging read-only URI mode=ro のみ使用
- 出力先 = /tmp/fire_d37_prep/ (= 一時 artifact) + 本 vault doc
- production / develop DB 接続なし
- git commit 0 / git push 0 / git add 0 / --no-verify 0
- launchctl / plist / cron / workflow 変更 0
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 全 0 件

### F282 plist 分離注記 (= 3 file 別個確認)
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- **LaunchAgents loaded plist** = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
- 3 file は **全て別 path / 別 file**

## §7 関連 file

- baseline F111: /tmp/fire_d37_prep/d37_f111.json
- baseline consumer: /tmp/fire_d37_prep/d37_consumer_payload.json
- baseline Markdown: /tmp/fire_d37_prep/d37_morning_advisory.md
- simulation F111 (max=50): /tmp/fire_d37_prep/d37_f111_max_candidates_sim.json
- 本 sector diversification sim doc: ~/fire-vault/04_daily/2026-07-03_sector_diversification_sim.md
- D37 trade plan: ~/fire-vault/04_daily/2026-07-03_manual_live_pilot_trade_plan.md
- D37 review: ~/fire-vault/04_daily/2026-07-03_manual_live_pilot_review.md
- D37 morning summary: ~/fire-vault/04_daily/2026-07-03_morning_advisory_summary.md
- D36 demote execution: ~/fire-vault/04_daily/2026-07-02_demote_execution_9130.md
