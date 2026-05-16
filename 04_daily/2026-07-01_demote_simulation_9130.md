---
id: FIRE-pilot-D35-demote-simulation-9130-2026-07-01
phase: 本番 v0 / W60-pilot-D35 / 9130 demote simulation (= read-only analysis)
priority: 最高 (= D36 demote 本実行候補判定の起点)
status: simulation complete、demote 本実行は HQ approve 待ち
owner: BlueFire7777 (Fujiwara)
date: 2026-07-01 (= D35、水)
parent_design: 03_design/FIRE_selection_policy_v1.4.1_post_open_markdown_2026-05-15.md
---

# FIRE W60-pilot-D35 — 9130 demote simulation (= 2026-07-01)

## §1 経緯

9130 共栄タンカーは D31 (2026-06-25) から D35 (2026-07-01) まで **連続 5 日**で
F111 entry top に出現. **5 連続 warning** 確定により demote simulation を実行.

連続日数推移:
- D31 (06-25): 1 連続目 (= reality entry +4.89%、actual 未 entry)
- D32 (06-26): 2 連続目 (= no-new-chase 既定)
- D33 (06-29): 3 連続目 (= no-new-chase 維持)
- D34 (06-30): 4 連続目 (= no-new-chase 維持)
- D35 (07-01): **5 連続目** (= **5 連続 warning 確定**) ← 本日

**simulation は本実行ではない**. recently_seen_codes への 9130 正式追加は HQ approve 後の別 wave で実施.

## §2 simulation 実行 inputs

| 項目 | baseline | simulation |
|---|---|---|
| base_date | 2026-07-01 | 2026-07-01 |
| recently_seen_codes | 8747, 5729, 3489, 340A0, 3798, 137A0, 7991 | 同 + **9130** |
| demoted_count | 7 | 8 |
| max_candidates | 20 | 20 |
| label_threshold_mode | strict | strict |
| output | /tmp/fire_d35_prep/d35_f111_baseline.json | /tmp/fire_d35_prep/d35_f111_demote_sim_9130.json |

## §3 simulation 結果比較

### 3.1 baseline (= 9130 未追加)

| role | 件数 | codes |
|---|---|---|
| entry_candidate | 3 | **9130**, 9247, 9628 |
| watch_candidate | 1 | 4404 |
| excluded_candidate | 16 | demote 7 + liquidity FAIL 9 |

### 3.2 simulation (= 9130 を recently_seen に追加)

| role | 件数 | codes |
|---|---|---|
| entry_candidate | 2 | 9247, 9628 (= 9130 消失) |
| watch_candidate | 1 | 4404 |
| excluded_candidate | 17 | demote 8 (= +9130) + liquidity FAIL 9 |

### 3.3 9130 role 変化

| 環境 | role | reason |
|---|---|---|
| baseline | **entry_candidate** | recently_seen 未登録、F111 entry top |
| simulation | **excluded_candidate** | recently_seen demote 適用、entry から外れる |

→ recently_seen demote logic 正常動作 ✓ (= 期待通り)

### 3.4 新規 entry 候補浮上の有無

**浮上 0 件**. baseline で entry に出ていた 9247 / 9628 が simulation でも entry top.
9130 が消失しても、新たな低流動性・小型・信用銘柄が entry に上がらない.

| code | baseline | simulation | 変化 |
|---|---|---|---|
| 9247 ＴＲＥ HD | entry | entry | 維持 (= TOPIX Small 1、貸借、PASS) |
| 9628 燦 HD | entry | entry | 維持 (= TOPIX Small 2、貸借、PASS) |
| 9130 共栄タンカー | entry | **excluded** | **demote 効果** |
| 4404 ミヨシ油脂 | watch | watch | 維持 |
| 331A0 / 4389 / 4317 等 9 件 | excluded | excluded | 維持 (= entry に戻らない) ✓ |
| その他 demote 7 件 | excluded | excluded | 維持 |

## §4 sector 多様化 / 流動性 / 100 株適性

| 観点 | baseline | simulation | 評価 |
|---|---|---|---|
| sector 多様化 | 運輸 1 + サービス 2 | サービス 2 のみ | **集中化** (= 海運業除外、情報通信・サービス 2 件のみ) |
| 流動性 (liquidity_status) | 全 entry = PASS | 全 entry = PASS | 維持 |
| 100 株適性 (standard_lot_ok) | 全 entry = True | 全 entry = True | 維持 |
| 信用区分 | 全 entry = 貸借 | 全 entry = 貸借 | 維持 |
| letter-suffix 新規上場 | entry に 0 件 | entry に 0 件 | 維持 |

→ **sector 集中化は唯一の懸念点** (= 1 セクター 2 候補)、ただし 100 株 / liquidity / 貸借 適性は維持.

## §5 D36 demote 本実行候補判定

### 判定: **本実行候補確定** (= 慎重判断付き)

**根拠 (= 安全側)**:
1. baseline と simulation の entry 候補 **劣化なし** (= 9247/9628 維持)
2. 低流動性 / 非 TOPIX / 100 株不適候補が entry に浮上しない
3. 9130 demote 効果は予測通り (= recently_seen 追加で excluded 移動)
4. 4404 watch + 100 株 risk 上限超で entry に戻らない
5. 331A0/4389/4317 は依然 excluded (= entry 復帰なし)

**慎重判断要件**:
- D36 demote 本実行は **HQ approve 必須** (= recently_seen_codes 正式追加 = 本番 advisory pipeline 反映)
- sector 集中化 (= サービス業 2 件のみ) のリスクを Fujiwara が許容するか確認
- D36 で 9130 が再度 entry に上がらない事 (= demote 効果) を本番 F111 で再確認
- D35 actual review (= D35 close 後の OHLCV / signal_success) を待って最終判断推奨

### D36 demote 本実行 prerequisites

1. **HQ approve** (= Fujiwara 明示承認、"9130 を recently_seen_codes に正式追加" 宣言)
2. **D35 actual review** (= D35 close 後の Fujiwara verbatim review、demote 妥当性最終検証)
3. **新候補多様化方針確定** (= sector 集中化許容 OR alternate candidates 検討)
4. **本番 F111 sim 再実行** (= HQ approve 後、recently_seen に 9130 追加した F111 を本番 baseline 化)

## §6 safety footer

- **本 simulation は read-only analysis** (= DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0)
- **本 simulation は本実行ではない** (= recently_seen_codes の正式追加は HQ approve 後の別 wave)
- F111 staging read-only URI mode=ro のみ使用
- 出力先 = /tmp/fire_d35_prep/ (= 一時 artifact) + 本 vault doc
- production / develop DB 接続なし
- git commit 0 / git push 0 / git add 0 / --no-verify 0
- launchctl / plist / cron / workflow 変更 0
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 全 0 件

### F282 plist 分離注記 (= 3 file 別個確認)
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- **LaunchAgents loaded plist** = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
- 3 file は **全て別 path / 別 file**、safety final で 3 file 別個に size + mtime 確認

## §7 関連 file

- baseline F111: /tmp/fire_d35_prep/d35_f111_baseline.json
- baseline consumer: /tmp/fire_d35_prep/d35_consumer_payload_baseline.json
- baseline post-open: /tmp/fire_d35_prep/d35_payload_after_override.json
- baseline Markdown: /tmp/fire_d35_prep/d35_morning_advisory.md
- simulation F111: /tmp/fire_d35_prep/d35_f111_demote_sim_9130.json
- simulation consumer: /tmp/fire_d35_prep/d35_consumer_payload_demote_sim_9130.json
- comparison doc (= tmp 版): /tmp/fire_d35_prep/d35_demote_sim_comparison.md
- 本 vault doc: ~/fire-vault/04_daily/2026-07-01_demote_simulation_9130.md
- D35 trade plan: ~/fire-vault/04_daily/2026-07-01_manual_live_pilot_trade_plan.md
- D35 review: ~/fire-vault/04_daily/2026-07-01_manual_live_pilot_review.md
- D35 morning summary: ~/fire-vault/04_daily/2026-07-01_morning_advisory_summary.md
