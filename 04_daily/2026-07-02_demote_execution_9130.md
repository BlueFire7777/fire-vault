---
id: FIRE-pilot-D36-demote-execution-9130-2026-07-02
phase: 本番 v0 / W60-pilot-D36 / 9130 demote 本実行記録
priority: 最高
status: demote 本実行完了 (= HQ approve 済、recently_seen_codes 正式追加)
owner: BlueFire7777 (Fujiwara)
date: 2026-07-02 (= D36、木)
parent_doc: 04_daily/2026-07-01_demote_simulation_9130.md (= D35 simulation)
parent_design: 03_design/FIRE_selection_policy_v1.4.1_post_open_markdown_2026-05-15.md
hq_approve: YES (= D36 wave 指示文で HQ/Fujiwara が明示承認)
---

# FIRE W60-pilot-D36 — 9130 demote 本実行記録 (= 2026-07-02)

## §1 経緯 + HQ approve

9130 共栄タンカーは D31-D35 まで連続 5 日 F111 entry top に出現. D35 で 5 連続 warning 確定、
demote simulation 実行済 (= 安全側確認).

**D36 wave 指示文で HQ/Fujiwara が以下を明示承認**:
- 9130 を D36 baseline の recently_seen_codes へ正式追加
- D36 の F111 baseline を 9130 追加済み recently_seen_codes で実行
- ただし売買指示ではない
- auto-order・楽天/iSPEED 操作・Computer Use は引き続き禁止

→ **D36 demote 本実行 prerequisites 4 件のうち #1 (HQ approve) 達成** ✓

## §2 D36 demote 本実行 inputs

| 項目 | D35 baseline | D35 simulation | **D36 本実行** |
|---|---|---|---|
| base_date | 2026-07-01 | 2026-07-01 | **2026-07-02** |
| recently_seen_codes | 8747, 5729, 3489, 340A0, 3798, 137A0, 7991 | 同 + 9130 | 同 + **9130** (= 正式追加) |
| demoted_count | 7 | 8 | **8** |
| max_candidates | 20 | 20 | 20 |
| label_threshold_mode | strict | strict | strict |
| output | d35_f111_baseline.json | d35_f111_demote_sim_9130.json | **d36_f111.json** |

## §3 D36 demote 本実行結果

### 3.1 candidate role 集計

| role | 件数 | codes |
|---|---|---|
| entry_candidate | 2 | 9247, 9628 |
| watch_candidate | 1 | 4404 |
| excluded_candidate | 17 | 8747, 5729, 3489, 340A0, 3798, 137A0, 7991, **9130**, 331A0, 4389, 4317, 2700, 6149, 288A0, 2981, 8152, 339A0 |

### 3.2 9130 demote 本実行判定

| field | value |
|---|---|
| 9130 role | **excluded_candidate** ✓ |
| 9130 reason_codes | `['recently_seen_demoted']` ✓ |
| 9130 candidate_role_reason | "recently_seen demoted (= 連続 rank 1 で demote 実行済)" |
| entry に残るか | **No** ✓ |
| watch に残るか | **No** ✓ |

→ **★★★ 9130 demote 本実行成功 ★★★** (= recently_seen_codes 正式追加で excluded 確定)

## §4 D35 simulation vs D36 本実行 比較

| 観点 | D35 simulation | D36 本実行 | 一致 |
|---|---|---|---|
| 9130 role | excluded_candidate | excluded_candidate | ✓ |
| 9130 reason_codes | recently_seen_demoted | recently_seen_demoted | ✓ |
| entry 件数 | 2 (9247, 9628) | 2 (9247, 9628) | ✓ |
| watch 件数 | 1 (4404) | 1 (4404) | ✓ |
| excluded 件数 | 17 (+9130) | 17 (+9130) | ✓ |
| 新規 entry 浮上 | 0 件 | 0 件 | ✓ |
| 低流動性 entry 浮上 | なし | なし | ✓ |
| letter-suffix entry 浮上 | なし | なし | ✓ |
| 4404 watch 維持 | YES | YES | ✓ |
| 331A0/4389/4317 excluded 維持 | YES | YES | ✓ |
| sector 多様化 | サービス 2 のみ | サービス 2 のみ | ✓ |

→ **simulation と D36 本実行が完全一致** ✓ (= demote logic 再現性確認)

## §5 entry 候補特性 (= 低流動性 / 非 TOPIX / 100 株不適 銘柄混入 0 確認)

| code | name | sector_17 | margin | scale | liquidity | standard_lot_ok | letter |
|---|---|---|---|---|---|---|---|
| 9247 | ＴＲＥ HD | 情報通信・サービスその他 | 貸借 | **TOPIX Small 1** | PASS | True | False |
| 9628 | 燦 HD | 情報通信・サービスその他 | 貸借 | **TOPIX Small 2** | PASS | True | False |

→ 全 entry は 貸借 + TOPIX Small + liquidity PASS + 100 株 lot ok + letter-suffix False ✓
→ 低流動性 / 非 TOPIX / 100 株不適 銘柄混入 **0 件** ✓

## §6 sector 集中化リスク (= 唯一の残懸念)

| sector | 件数 | codes |
|---|---|---|
| 情報通信・サービスその他 | 2 | 9247, 9628 |
| **その他 sector** | 0 | - |

→ **sector 集中化リスクあり** (= 1 sector 2 候補、運輸/食品/機械等への分散なし)

**評価**:
- D35 simulation 段階で予測済の懸念 (= 9130 demote で運輸セクター除外)
- 100 株 lot / liquidity / 貸借 適性は維持されているため、entry の "質" は劣化していない
- **Fujiwara 判断要**: サービス sector 集中を許容するか、alternate candidates (= 別 sector) を
  検討するか
- 短期対応案: D36/D37 で 9247 か 9628 のみに 100 株 entry (= 同 sector 重複避け)
- 中期対応案: F111 max_candidates を 30-50 に拡張し、別 sector の TOPIX Small/Mid 候補を浮上させる

## §7 D37 以降の引き継ぎ

### 7.1 recently_seen_codes 正式状態

- **D36 以降の正式 recently_seen_codes**: `8747, 5729, 3489, 340A0, 3798, 137A0, 7991, 9130`
- demoted_count: **8** (= 7 → 8)

### 7.2 9130 連続日数 counter

- D31-D35 で 5 連続 → D36 demote 本実行で counter リセット (= 0)
- 9130 が D37 以降に再び F111 entry top に出ることは **想定しない** (= recently_seen で除外維持)
- もし D37+ で 9130 が再 entry に浮上した場合は CRITICAL (= recently_seen logic 不具合)

### 7.3 D37 朝 pilot wave

- F111 baseline: D36 と同じ recently_seen_codes (= 9130 含む) で実行
- entry 候補: 9247 / 9628 維持想定
- sector 集中化 follow-up:
  - Fujiwara が sector 集中許容 → D37 通常 pilot 継続
  - 別 sector 候補が欲しい → max_candidates 拡張 sim を別 wave で実行

### 7.4 demote 永続化方針

本 wave での 9130 demote は **D36 F111 実行時点での反映**.
永続 DB / 設定 file への write は **本 wave では行わない** (= R-01-08 + DB write 禁止方針).

**永続化の選択肢** (= 後続 wave):
- A. 環境変数 / config file に recently_seen_codes 8 件版を記録
- B. F111 runner default に 9130 含む 8 件を embed (= code 変更要、別 wave)
- C. cron / launchd で D37 以降の F111 起動時に 9130 含む recently_seen_codes を渡す (= ops 整備要)

## §8 D36 advisory への反映

- D36 pre-open Markdown: `/tmp/fire_d36_prep/d36_morning_advisory.md`
- D36 trade plan: `~/fire-vault/04_daily/2026-07-02_manual_live_pilot_trade_plan.md`
- D36 review template: `~/fire-vault/04_daily/2026-07-02_manual_live_pilot_review.md`
- D36 morning summary: `~/fire-vault/04_daily/2026-07-02_morning_advisory_summary.md`

全 D36 docs で:
- 9130 = excluded として記録 (= recently_seen demote 反映済)
- entry = 9247, 9628 (= sector 集中化注記付き)
- watch = 4404
- recently_seen_codes 8 件版 (= 9130 含む) を正本として表示

## §9 safety footer

- 本 wave は **read-only F111 実行 + vault doc 作成** のみ
- auto-order 0 / 楽天 0 / iSPEED 自動操作 0 / Computer Use 0
- LINE 送信 0 / token 0 / API 0 / DB write 0
- production / develop DB 接続 0 / staging read-only URI mode=ro のみ
- git commit 0 / git push 0 / git add 0 / --no-verify 0
- launchctl / plist / cron / workflow 変更 0
- sudo / rm -rf 0
- forbidden phrase 全 0 件
- HQ approve: D36 wave 指示文で明示承認済

### F282 plist 分離注記 (= 3 file 別個確認)
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- **LaunchAgents loaded plist** = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
- 3 file は **全て別 path / 別 file**、safety final で 3 file 別個に size + mtime 確認

## §10 関連 file

- D35 simulation doc (= 起点): ~/fire-vault/04_daily/2026-07-01_demote_simulation_9130.md
- D36 F111 output (= 本実行): /tmp/fire_d36_prep/d36_f111.json
- D36 consumer payload: /tmp/fire_d36_prep/d36_consumer_payload.json
- D36 morning advisory MD: /tmp/fire_d36_prep/d36_morning_advisory.md
- D36 actual_flow template: /tmp/fire_d36_prep/d36_actual_flow_template.json
- D36 trade plan: ~/fire-vault/04_daily/2026-07-02_manual_live_pilot_trade_plan.md
- D36 review template: ~/fire-vault/04_daily/2026-07-02_manual_live_pilot_review.md
- D36 morning summary: ~/fire-vault/04_daily/2026-07-02_morning_advisory_summary.md
- 本 demote execution doc: ~/fire-vault/04_daily/2026-07-02_demote_execution_9130.md
