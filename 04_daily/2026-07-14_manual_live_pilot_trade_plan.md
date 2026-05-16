---
id: FIRE-pilot-D44-trade-plan-2026-07-14
phase: 本番 v0 / W60-pilot-D44 / v1.4.2 朝 pilot 7 日目 / max=50 baseline 2 日目
priority: 高
status: pre-open、v1.4.2 継続運用 + max=50 baseline 2 日目 + 9247/9628 13 連続到達 (fixed-candidate HIGH)
owner: BlueFire7777 (Fujiwara)
date: 2026-07-14 (= D44、火)
parent_baseline_doc: 03_design/FIRE_f111_max_candidates_baseline_50_2026-05-16.md
codex_lanes: 4
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-07-14 / D44)

## §1 全体方針

**v1.4.2** を正本として 7 日目の朝 pilot. F111 max=50 baseline 2 日目.

- 100 株標準、非 100 株調整禁止
- entry / watch / excluded 分離
- 9130 demote 効果継続観察 (= excluded 維持、9 日目)
- 4404 v1.4.2 entry 復帰継続 (= ⚠ risk_yen_warning、7 日目)
- **9247 / 9628 連続 13 日目 (= fixed-candidate HIGH 相当、D45+ 片方 demote 候補)**
- max=50 baseline で entry 14 件、朝判断は **top 5 優先** + full list 参考

## §2 D44 F111 出力 (= v1.4.2 + max=50 baseline)

- input: `/tmp/fire_d44_prep/d44_f111.json`
- cli_version: **1.3.0** ✓
- max_candidates: **50** ✓ (= baseline 自然適用 2 日目)
- selection_policy_version: **1.4.2** ✓
- recently_seen_codes (= 8 件版): `9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798`
- demoted_count: 8
- base_date: 2026-07-14

| role | 件数 | codes |
|---|---|---|
| **entry_candidate** | 14 | top 5: 9247, 9628, 4404, 3089, 9633 / 残り 9 件 |
| watch_candidate | 0 | |
| excluded_candidate | 21 | demote 8 (= +9130) + liquidity FAIL 13 |

## §3 9130 demote 効果継続 (9 日目) + 4404 v1.4.2 継続 (7 日目)

### 3.1 9130 (= demote 効果継続 9 日目: D36-D44)

| field | value |
|---|---|
| candidate_role | **excluded_candidate** ✓ |
| reason_codes | `['recently_seen_demoted']` |
| entry/watch 復帰 | No / No ✓ |
| 連続 excluded | **9 日連続** (= D36 を 1 とする counter) |

### 3.2 4404 (= v1.4.2 entry 復帰 7 日目)

| field | value |
|---|---|
| candidate_role | **entry_candidate** ✓ |
| standard_lot_ok | True / share_unit: 100 |
| risk_yen_for_100_shares | 10,665 円 |
| **risk_yen_over_pilot_budget** | **True** ✓ (= ⚠ 警告フラグ継続) |
| reason_codes | `['risk_yen_warning']` |
| 連続 entry | **7 日連続** (= D38-D44) |

### 3.3 9247 / 9628 fixed-candidate HIGH (= 連続 13 日目)

| code | 連続 entry 日数 | 固定化リスク |
|---|---|---|
| 9247 | **13 日連続** (D32-D44) | 9130 閾値 (5 連続) の 2.6 倍超 |
| 9628 | **13 日連続** (D32-D44) | 同上 |

→ **fixed-candidate HIGH 相当** (= 本 wave で記録、本実行はしない)
→ D45 以降の HQ approve 後の単独 demote 候補 (= sim A or B、D41 sim 結果)
→ **両方同時 demote は原則非推奨**

## §4 D44 朝の優先確認 top 5

### 4.1 [top 1] 9247 ＴＲＥ HD [連続 13 日目、fixed-candidate HIGH]

| field | value |
|---|---|
| entry 検討価格 | 1,612 円 |
| 指値目安 | 1,612 円 |
| 追わない上限 | 1,644 円 (+2.0%) |
| 利確目安 | 1,773 円 (+2R = +161 円) |
| 損切り目安 | 1,531 円 (-1R = -81 円) |
| 株数 | 100 株 |
| risk_yen | 8,060 円 |
| sector | 情報通信・サービスその他、TOPIX Small 1、貸借 |
| 見送り条件 | gap > 1,644 / 板薄 / spread 拡大 / 9628 同 sector 重複 / 連続 13 日固定化注意 |

### 4.2 [top 2] 9628 燦 HD [連続 13 日目、fixed-candidate HIGH]

| field | value |
|---|---|
| entry 検討価格 | 1,390 円 |
| 指値目安 | 1,390 円 |
| 追わない上限 | 1,418 円 (+2.0%) |
| 利確目安 | 1,529 円 (+2R = +139 円) |
| 損切り目安 | 1,320 円 (-1R = -70 円) |
| 株数 | 100 株 |
| risk_yen | 6,950 円 |
| sector | 情報通信・サービスその他、TOPIX Small 2、貸借 |
| 見送り条件 | gap > 1,418 / 板薄 / spread 拡大 / 9247 同 sector 重複 / 連続 13 日固定化注意 |

### 4.3 [top 3] 4404 ミヨシ油脂 [⚠ risk warning、連続 7 日目]

> ⚠ **risk warning** (= 旧 pilot 上限超過): 100 株 risk_yen = 10,665 円
>   → 損切り価格 / 許容損失 / 板 / 出来高 / spread / RR を **手動確認** してから entry

| field | value |
|---|---|
| entry 検討価格 | 2,133 円 |
| 指値目安 | 2,133 円 |
| 追わない上限 | 2,176 円 (+2.0%) |
| 利確目安 | 2,346 円 (+2R = +213 円) |
| 損切り目安 | 2,026 円 (-1R = -107 円) |
| 株数 | 100 株 |
| risk_yen | **10,665 円** ⚠ 旧 pilot 上限超過警告 |
| sector | 食品、貸借、scale=- |
| 見送り条件 | gap > 2,176 / 板薄 / spread 拡大 / 損切り価格不納得 |

### 4.4 [top 4] 3089 テクノアルファ [新規継続、商社 sector 多様化]

| field | value |
|---|---|
| entry 検討価格 | 1,055 円 |
| 指値目安 | 1,055 円 |
| 追わない上限 | 1,076 円 (+2.0%) |
| 利確目安 | 1,160 円 (+2R = +105 円) |
| 損切り目安 | 1,002 円 (-1R = -53 円) |
| 株数 | 100 株 |
| risk_yen | 5,275 円 |
| sector | 商社・卸売 (= sector 多様化、D43 から継続) |
| 見送り条件 | gap > 1,076 / 板薄 / spread 拡大 / 出来高薄注意 |

### 4.5 [top 5] 9633 東京テアトル [新規継続、不動産 sector 多様化]

| field | value |
|---|---|
| entry 検討価格 | 1,579 円 |
| 指値目安 | 1,579 円 |
| 追わない上限 | 1,610 円 (+2.0%) |
| 利確目安 | 1,737 円 (+2R = +158 円) |
| 損切り目安 | 1,500 円 (-1R = -79 円) |
| 株数 | 100 株 |
| risk_yen | 7,895 円 |
| sector | 不動産 (= sector 多様化、D43 から継続) |
| 見送り条件 | gap > 1,610 / 板薄 / spread 拡大 / 出来高薄注意 |

## §5 参考: top 5 以外の entry 9 件

| # | code | 銘柄 | sector | 価格 | risk_yen | warning |
|---|---|---|---|---|---|---|
| 6 | 2146 | ＵＴグループ | 情報通信 | 184 円 | 920 | - |
| 7 | 4828 | ビジネスエンジニアリング | 情報通信 | 1,246 円 | 6,230 | - |
| 8 | 8699 | ＨＳ HD | 金融（除く銀行） | 1,140 円 | 5,700 | - |
| 9 | 3712 | 情報企画 | 情報通信 | 1,039 円 | 5,195 | - |
| 10 | 7803 | ブシロード | 情報通信 | 261 円 | 1,305 | - |
| 11 | 4914 | 高砂香料工業 | 素材・化学 | 1,180 円 | 5,900 | - |
| 12 | 3479 | ティーケーピー | 不動産 | 1,725 円 | 8,625 | ⚠ |
| 13 | 4540 | ツムラ | 医薬品 | 3,582 円 | 17,910 | ⚠ |
| 14 | 8057 | 内田洋行 | 商社・卸売 | 2,053 円 | 10,265 | ⚠ |

## §6 excluded 21 件

- demote 8 件: 9130 (D36 demote、9 日目) / 8747 / 5729 / 3489 / 340A0 / 3798 / 137A0 / 7991
- liquidity FAIL 13 件: 331A0 / 4389 / 4317 / 2700 / 6149 / 288A0 / 2981 / 8152 / 339A0 等

## §7 sector 多様化状況

| sector | 件数 (entry) | 代表 code |
|---|---|---|
| 情報通信・サービスその他 | 6 | 9247, 9628, 2146, 4828, 3712, 7803 |
| 商社・卸売 | 2 | 3089, 8057 |
| 不動産 | 2 | 9633, 3479 |
| 食品 | 1 | 4404 |
| 金融（除く銀行） | 1 | 8699 |
| 素材・化学 | 1 | 4914 |
| 医薬品 | 1 | 4540 |
| 合計 | **14** | **7 sector** |

→ D38-D42 baseline (max=20) の 2 sector → max=50 baseline で **7 sector**、D43-D44 で継続 ✓

## §8 F111-THEME-SECTOR-OVERLAY-R1 必要性整理 (= seed + dynamic discovery)

### 8.1 必要性

- sector_17 は 17 分類で粗い (= 例: 情報通信 6 件集中)
- Fujiwara が朝みる時、市場テーマ単位 (= 半導体/電線/AI/防衛/銀行 等) の整理がほしい
- 9247/9628 fixed-candidate HIGH 状態解消のためにも theme tag による視覚化が有効

### 8.2 設計方針 (= seed + dynamic discovery、固定 10 テーマ限定しない)

- **seed (= 起点 theme 例)**:
  半導体 / 電線 / AI データセンター / 電力インフラ / 防衛 / 銀行 / 商社 /
  機械 / 電機 / 非鉄金属 / 海運 / 不動産 / 食品 / 医薬品 / ...
- **dynamic discovery**:
  - 銘柄ニュース / 業種コード / 業績タグ等から自動拡張
  - 新興テーマ (= AI / EV / 量子等) を逐次追加
- **固定 10 テーマ限定しない** (= 拡張性確保)
- 本 wave は設計引き継ぎメモのみ、実装は別 wave (= F111-THEME-SECTOR-OVERLAY-R1)

### 8.3 D45 以降の起票検討

- D45-D46 で本 wave 結果を踏まえた設計 wave 起票候補
- 設計 wave で seed list 確定 + dynamic discovery 設計
- 実装 wave で F111 出力 field 拡張 + consumer / MD 拡張

## §9 D44 iSPEED 確認項目

1. 寄付前気配 / 気配指値 (= top 5 全件)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動
4. スプレッド
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ %
7. セクター指数の相対強弱
8. **4404 限定**: risk_yen 10,665 円 / 損切り 2,026 円 / 許容損失改めて確認
9. **9247 / 9628 13 連続**: fixed-candidate HIGH、同 sector 重複は片方絞り推奨
10. **新規 (3089/9633)**: D43 と同候補、流動性・出来高 D43 results 参考

## §10 D44 トレード手順

### 10.1 pre-open
1. F111 D44 確認 (= max=50 baseline 2 日目)、consumer payload 確認、Markdown 確認
2. top 5 advisory 確認 (= 主役 3 + sector 多様化新規 2)
3. iSPEED で top 5 確認
4. Fujiwara 最終判断:
   - 9247 / 9628: 連続 13 日目 fixed-candidate HIGH、同 sector 重複片方絞り推奨
   - 4404: entry 検討可、risk 警告許容判断
   - 3089 / 9633: D43 と同候補、流動性慎重確認
   - 9130: skip 確定

### 10.2 寄り後 / 前場後
1. actual_flow_d44.json 作成 (= template コピー)
2. OHLCV / fujiwara_decision を手動入力 (= top 5 + 9130)
3. post-open update wrapper 実行 (= optional)

### 10.3 close 後
1. review template 埋め (= 10 項目)
2. v1.4.2 7 日目 / max=50 baseline 2 日目運用検証
3. 9130 demote 9 日目確認
4. 9247/9628 連続日数更新 (= D45 で 14 日 or リセット)
5. 新規 (3089/9633) の D43 → D44 連続性評価
6. F111-THEME-SECTOR-OVERLAY-R1 起票判断材料整理

### 10.4 review 10 項目

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

### signal_success 判定基準注記
- historical review: 当日朝 advisory 利確目安への事後評価、Fujiwara 手動
- current renderer: post-open high/low の +2R / -1R hit 自動判定、推測禁止
- → 別 field / 別意味、current renderer 値で historical を **上書きしない**

## §11 safety footer

- 本 trade plan は **手動発注用 reference** (= read-only advisory)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0
- 生成 file: `/tmp/fire_d44_prep/` + vault `~/fire-vault/04_daily/2026-07-14_*.md`
- 9247/9628 demote 本実行は別 wave (= HQ approve 後)
- F111 max=50 baseline / max=100/200/300 採用せず
- 両方同時 demote 原則非推奨
- theme overlay は seed + dynamic discovery 方式、固定 10 テーマ限定しない

### F282 plist 分離注記 (= 3 file 別個確認)
- **production plist** = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- **smoke plist** = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- **LaunchAgents loaded plist** = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)

## §12 D31-D43 引き継ぎ + D45 以降

- D31 (06-25): 9130 +4.89% / 連続 1
- D32-D34: 9130 連続 2-4 / 4404 watch (= 旧 v1.4.1)
- D35: 9130 連続 5 + demote sim
- D36: **9130 demote 本実行**
- D37: 9130 demote 継続 / sector 集中化 / max_candidates sim
- D38: v1.4.2 初 pilot / 4404 復帰 + warning / sector 2 多様化
- D39-D40: v1.4.2 2-3 日目
- D41: v1.4.2 4 日目 / 9247-9628 連続 10 日目 / demote sim 完了
- D42: v1.4.2 5 日目 / 9247-9628 連続 11 日目 / max=50 sim 完了
- D43: v1.4.2 6 日目 / 9247-9628 連続 12 日目 / max=50 baseline 初回稼働
- **D44 (07-14): v1.4.2 7 日目 / 9130 demote 9 日目 / 4404 連続 7 日目 / 9247-9628 連続 13 日目 (fixed-candidate HIGH) / max=50 baseline 2 日目** ← 本日

### D45 (= 2026-07-15 水 予定)
- F111 baseline: max=50 / recently_seen 8 件版継続
- 9130 demote 10 日目観察
- 4404 連続 entry 8 日目
- **9247 / 9628 連続 14 日目** or リセット
- **HQ approve 後の 9247 or 9628 単独 demote 本実行検討** (= sim A or B)
- F111-THEME-SECTOR-OVERLAY-R1 設計 wave 起票検討 (= seed + dynamic discovery)
- 9130 再 entry → CRITICAL / 9130 watch 戻り → HIGH / 4404 watch 戻り → HIGH
