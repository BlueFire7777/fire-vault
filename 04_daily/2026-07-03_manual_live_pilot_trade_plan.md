---
id: FIRE-pilot-D37-trade-plan-2026-07-03
phase: 本番 v0 / W60-pilot-D37 / v1.4.1 + 9130 demote 効果継続確認 + sector 多様化 sim
priority: 高
status: pre-open、9130 demote 効果継続成功、sector 集中化継続
owner: BlueFire7777 (Fujiwara)
date: 2026-07-03 (= D37、金)
parent_design: 03_design/FIRE_selection_policy_v1.4.1_post_open_markdown_2026-05-15.md
demote_execution_doc: 04_daily/2026-07-02_demote_execution_9130.md (= D36)
sector_diversification_sim_doc: 04_daily/2026-07-03_sector_diversification_sim.md
codex_lanes: 4
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-07-03 / D37)

## §1 全体方針

**v1.4.1** を正本として運用. D36 9130 demote 本実行後の **初回継続確認日**.

- risk_yen は順位加点に**使わない**
- **100 株標準** (= 非 100 株調整絶対禁止)
- entry / watch / excluded を分ける
- 9130 demote 効果継続観察 (= excluded 維持確認)
- **sector 集中化継続**、多様化 sim 実施済 (= read-only analysis)

## §2 D37 F111 runner 出力 (= 9130 demote 後 baseline)

- input file: `/tmp/fire_d37_prep/d37_f111.json`
- selection_policy_version: `1.4.1`
- share_unit_standard: `100`
- **正式 recently_seen_codes (= 8 件版)**: `9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798`
- demoted_count: **8**

candidate 分類:

| role | 件数 | codes |
|---|---|---|
| **entry_candidate** | 2 | 9247, 9628 |
| **watch_candidate** | 1 | 4404 |
| **excluded_candidate** | 17 | demote 8 (= +9130) + liquidity FAIL 9 |

## §3 9130 demote 効果継続確認

### 3.1 9130 D37 状態

| field | value |
|---|---|
| 9130 role | **excluded_candidate** ✓ |
| 9130 reason_codes | `['recently_seen_demoted']` ✓ |
| 9130 candidate_role_reason | "recently_seen demoted (= 連続 rank 1 で demote 実行済)" |
| entry に残るか | **No** ✓ |
| watch に残るか | **No** ✓ |

→ **demote 効果継続成功** (= recently_seen demoted で excluded 維持、CRITICAL/HIGH なし)

### 3.2 D36 vs D37 比較

| 観点 | D36 (= 本実行日) | D37 (= 本日) | 整合 |
|---|---|---|---|
| 9130 role | excluded | excluded | ✓ |
| 9130 reason_codes | recently_seen_demoted | recently_seen_demoted | ✓ |
| entry | 9247, 9628 | 9247, 9628 | ✓ |
| watch | 4404 | 4404 | ✓ |
| excluded | 17 (+9130) | 17 (+9130) | ✓ |

→ **D36 から D37 への引き継ぎ完全整合** ✓

## §4 D37 朝の注文完成形 advisory

### 4.1 9247 ＴＲＥ HD [運用: entry 検討可、同 sector 重複注意]

| field | value |
|---|---|
| entry 検討価格 | 1,612 円 |
| 指値目安 | 1,612 円 (= 寄付前 reference) |
| 追わない上限 | 1,644 円 (= +2.0%) |
| 利確目安 | 1,773 円 (= +2R = +161 円) |
| 損切り目安 | 1,531 円 (= -1R = -81 円) |
| 株数 | 100 株 |
| risk_yen | 8,060 円 |
| sector | 情報通信・サービスその他、TOPIX Small 1、貸借 |
| 見送り条件 | gap > 1,644 / 板薄 / 出来高低 / spread 拡大 / 9628 と同 sector のため同時 entry 慎重 |

### 4.2 9628 燦 HD [運用: entry 検討可、同 sector 重複注意]

| field | value |
|---|---|
| entry 検討価格 | 1,390 円 |
| 指値目安 | 1,390 円 |
| 追わない上限 | 1,418 円 (= +2.0%) |
| 利確目安 | 1,529 円 (= +2R = +139 円) |
| 損切り目安 | 1,320 円 (= -1R = -70 円) |
| 株数 | 100 株 |
| risk_yen | 6,950 円 |
| sector | 情報通信・サービスその他、TOPIX Small 2、貸借 |
| 見送り条件 | gap > 1,418 / 板薄 / 出来高低 / spread 拡大 / 9247 と同 sector のため同時 entry 慎重 |

## §5 watch + excluded

### 5.1 4404 ミヨシ油脂 [watch 維持]

| field | value |
|---|---|
| watch 理由 | standard_lot_ok=False (= 100 株 risk_yen 10,665 円 > pilot 限度) |
| entry 不可理由 | 非 100 株調整不採用、pilot 限度超 |

### 5.2 9130 共栄タンカー [demote 効果継続、excluded 固定]

| field | value |
|---|---|
| role | excluded_candidate |
| reason_codes | recently_seen_demoted |
| D37 状態 | demote 効果継続成功 (= D36 から維持) |

### 5.3 excluded 全 17 件

- demote 8 件: 8747 / 5729 / 3489 / 340A0 / 3798 / 137A0 / 7991 / **9130** (= D36 追加)
- liquidity FAIL 9 件: 331A0 (letter-suffix) / 4389 / 4317 / 2700 / 6149 / 288A0 (letter-suffix) /
  2981 / 8152 / 339A0 (letter-suffix)

## §6 sector 集中化リスク + 多様化 simulation 結果

### 6.1 D37 baseline sector 分布

| sector | 件数 | codes |
|---|---|---|
| 情報通信・サービスその他 | **2** | 9247, 9628 |

→ **sector 集中化継続** (= D36 と同じ状態)

### 6.2 sector 多様化 simulation 結果 (= 詳細: 04_daily/2026-07-03_sector_diversification_sim.md)

max_candidates=50 で 10 件 entry / **5 sector** に多様化:

| sector | 件数 | codes (★ 新規) |
|---|---|---|
| 情報通信・サービスその他 | 6 | 9247, 9628, ★ 2146, ★ 4828, ★ 3712, ★ 7803 |
| 金融（除く銀行） | 1 | ★ 8699 ＨＳ HD |
| 商社・卸売 | 1 | ★ 3089 テクノアルファ |
| 不動産 | 1 | ★ 9633 東京テアトル |
| 素材・化学 | 1 | ★ 4914 高砂香料工業 |

全 新規 entry が 100 株 / 貸借 / PASS / letter=False ✓

### 6.3 Fujiwara 判断要

**短期 (= D37 当日)**:
- baseline (= max=20、9247/9628 のみ) を D37 朝の正本として使用
- 同 sector 重複許容 → 9247 か 9628 のいずれか 1 件に絞って entry 検討
- sector 多様化希望 → simulation 結果から 1-2 件を手動選定

**中期 (= D38 以降の別 wave)**:
- max_candidates=50 baseline 化検討 (= HQ approve 後)
- F111 実行時間 / メモリ使用量確認

## §7 D37 iSPEED 確認項目

1. 寄付前気配 / 気配指値 (= 9247 / 9628 のみ)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動
4. スプレッド (= ask-bid 比)
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ %
7. セクター指数の相対強弱 (= 情報通信・サービスその他 / 食品 / 運輸・物流)

## §8 D37 トレード手順

### 8.1 pre-open

1. F111 D37 確認 (= `/tmp/fire_d37_prep/d37_f111.json`、recently_seen 8 件版)
2. consumer payload 確認 (= `/tmp/fire_d37_prep/d37_consumer_payload.json`)
3. Markdown 確認 (= `/tmp/fire_d37_prep/d37_morning_advisory.md`)
4. sector 多様化 sim doc 確認 (= `04_daily/2026-07-03_sector_diversification_sim.md`)
5. iSPEED で 9247 / 9628 の寄付前気配 / 板厚 / spread / 信用倍率を確認
6. Fujiwara 最終判断:
   - 9247 / 9628: entry 検討、ただし同 sector 重複に注意 (= 1 件絞り推奨)
   - 4404: skip 確定 (= watch)
   - 9130: skip 確定 (= demote 済、excluded 固定)

### 8.2 寄り後 / 前場後

1. `/tmp/fire_d37_prep/d37_actual_flow_template.json` をコピーして実 actual_flow_d37.json 作成
2. OHLCV / fujiwara_decision を手動入力 (= 推測禁止)
3. post-open update wrapper 実行 (= optional)

### 8.3 close 後

1. review template (= `~/fire-vault/04_daily/2026-07-03_manual_live_pilot_review.md`) を埋める
2. OHLCV / signal_success / entry_success / 板 spread 所感 / 結果メモ
3. 9130 demote 効果継続確認 (= 終値 / 出来高観察)
4. sector 集中化 follow-up 判断

### 8.4 review 10 項目 (= D37 close 後)

1. 入った / 見送った
2. 判断理由
3. 実際に見た銘柄
4. 板 / 出来高 / spread の所感
5. 結果メモ
6. 実 entry か skip か
7. 入った価格 or 見送り理由
8. 追わない上限を超えていたか
9. pre-open 判断か post-open 判断か
10. signal_success と entry_success の区別

### signal_success 判定基準注記 (= historical vs current renderer)
- historical review: 当日朝 advisory 利確目安への事後評価、Fujiwara 手動
- current renderer: post-open high/low の +2R / -1R hit 自動判定、推測禁止
- → 別 field / 別意味、current renderer 値で historical を **上書きしない**

## §9 safety footer

- 本 trade plan は **手動発注用 reference** (= read-only advisory)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / launchctl 0
- 全 generators (= F111 / consumer / MD / post-open update) は read-only
- 生成 file: `/tmp/fire_d37_prep/` + vault `~/fire-vault/04_daily/2026-07-03_*.md`
- production / develop DB 接続なし、staging read-only URI mode=ro のみ
- sector 多様化 simulation は read-only analysis (= 本番反映なし、別 wave で検討)

### F282 plist 分離注記 (= 3 file 別個確認)
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- **LaunchAgents loaded plist** = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
- 3 file は **全て別 path / 別 file**

## §10 D31-D36 からの引き継ぎ + D38 以降

### 10.1 D31-D37 連続観察 サマリ

- D31 (06-25): 9130 reality entry +4.89%、連続 1 日目
- D32 (06-26): 9247/9628 entry → skip / 9130 no-new-chase (連続 2) / 4404 watch
- D33 (06-29): 同 + 9130 連続 3
- D34 (06-30): 同 + 9130 連続 4
- D35 (07-01): 同 + 9130 連続 5 = warning + demote sim
- D36 (07-02): **9130 demote 本実行完了** (= recently_seen 正式追加)
- D37 (07-03): **9130 demote 効果継続成功** (= excluded 維持) ← **本日**

### 10.2 D38 (= 2026-07-06 月) 以降

- F111 baseline: recently_seen 8 件版 (= 9130 含む) 継続使用
- 9130 demote 効果の長期観察 (= 1 週間後 / 1 ヶ月後の再 entry 浮上有無)
- sector 集中化 follow-up:
  - 短期: 同 sector 重複許容 or 1 件絞り判断
  - 中期: max_candidates 拡張 baseline 化検討 (= 別 wave、HQ approve 必須)
- D32 / D33 / D34 / D35 / D36 / D37 actual fill (= Fujiwara verbatim review、J-Quants 待ち)
