---
id: FIRE-pilot-D49-trade-plan-2026-07-22
phase: 本番 v0 / W60-pilot-D49 / W3 universe 拡張後の初回稼働
priority: 高
status: pre-open / GO_CONDITIONAL
owner: BlueFire7777 (Fujiwara)
date: 2026-07-22 (= D49、水)
parent_w3: 03_design/F111_UNIVERSE_EXPANSION_R1_W3_result_2026-05-16.md
codex_lanes: 4
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-07-22 / D49)

## §1 全体方針

**v1.4.2 + W3 universe 拡張 (staging 109 件反映) 初回稼働**:
- top-n=100 baseline 実反映 (= staging signals 2026-07-22 / 109 件)
- max=50 baseline 自然適用
- F111 raw: 35 → **50** (= +43% 拡張継続)
- entry: 14 → **20** (= W3 新顔 6 件継続)
- entry sector: 7 → **9** (= 運輸・物流 + 小売 新セクター)

注意:
- 9247 / 9628 rank 1/2 継続 (= universe 拡張だけでは固定化緩和不十分)
- theme overlay / scoring 補正 / 単独 demote は別 wave で根本対策

## §2 D49 baseline 出力

- F111: cli=1.3.0 / max=50 / policy=1.4.2 / base_date=2026-07-22 / raw 50
- staging signals: 2026-07-22 / 109 件 (= W3 反映済)
- consumer: entry 20 / watch 0 / excluded 30 / validation_passed=True
- MD: 19,160 bytes

| role | 件数 |
|---|---|
| entry_candidate | **20** (= W3 拡張継続) |
| watch_candidate | 0 |
| excluded_candidate | 30 |

## §3 朝判断 top 5 (= 主役 3 + 新顔 + 新セクター)

### 3.1 [top 1] 9247 ＴＲＥ HD [主役、rank 1、情報通信]
- 参考価格: 1,612 円
- 100 株 / 追わない上限 1,644 円 (+2.0%) / 利確目安 1,773 円 / 損切り目安 1,531 円
- risk_yen 8,060 円
- 注: 連続 18 日目、fixed-candidate HIGH 継続. 9628 同 sector 重複、片方絞り推奨.

### 3.2 [top 2] 9628 燦 HD [主役、rank 2、情報通信]
- 参考価格: 1,390 円
- 100 株 / 追わない上限 1,418 円 / 利確目安 1,529 円 / 損切り目安 1,320 円
- risk_yen 6,950 円
- 注: 連続 18 日目、9247 同 sector.

### 3.3 [top 3] ★ 4404 ミヨシ油脂 [⚠ risk warning、rank 3、食品]
- 参考価格: 2,133 円
- 100 株 / 追わない上限 2,176 円 / 利確目安 2,346 円 / 損切り目安 2,026 円
- risk_yen **10,665 円 ⚠** (= 旧 pilot 上限超過警告)
- 注: 損切り価格 / 許容損失 / 板 / 出来高 / spread / RR を手動確認してから entry. 連続 12 日目.

### 3.4 [top 4] ★ 9008 京王電鉄 [W3 新顔、rank 16、運輸・物流 (新セクター)]
- 参考価格: 747 円
- 100 株 / 追わない上限 762 円 / 利確目安 822 円 / 損切り目安 710 円
- risk_yen 3,735 円
- 注: W3 universe 拡張で浮上、低 risk + 新セクター多様化候補. iSPEED で流動性確認推奨.

### 3.5 [top 5] ★ 3134 Ｈａｍｅｅ [W3 新顔、rank 19、小売 (新セクター)]
- 参考価格: 412 円
- 100 株 / 追わない上限 420 円 / 利確目安 453 円 / 損切り目安 391 円
- risk_yen 2,060 円
- 注: W3 universe 拡張で浮上、最低 risk + 新セクター多様化候補.

## §4 W3 新顔 6 件 D49 継続確認

| code | rank | sector | risk_yen |
|---|---|---|---|
| 9417 | 13 | 情報通信 | 1,550 |
| 9008 | 16 | **運輸・物流 (新)** | 3,735 |
| 6196 | 17 | 情報通信 | 6,165 |
| 7595 | 18 | 情報通信 | 6,780 |
| 3134 | 19 | **小売 (新)** | 2,060 |
| 3962 | 20 | 情報通信 | 4,605 |

→ W3 → D49 で **全 6 件 entry 継続** ✓ (= universe 拡張効果安定)

## §5 sector 多様化 9 種

| sector | 件数 | 代表 |
|---|---|---|
| 情報通信・サービスその他 | 10 | 9247, 9628, 2146, 4828, 3712, 7803, 9417, 6196, 7595, 3962 |
| 商社・卸売 | 2 | 3089, 8057 |
| 不動産 | 2 | 9633, 3479 |
| 食品 | 1 | 4404 |
| 金融（除く銀行） | 1 | 8699 |
| 素材・化学 | 1 | 4914 |
| 医薬品 | 1 | 4540 |
| **運輸・物流** (W3 新) | 1 | 9008 |
| **小売** (W3 新) | 1 | 3134 |

## §6 重要銘柄状態

| code | 状態 | 評価 |
|---|---|---|
| 9130 | excluded (= recently_seen_demoted) | demote 14 日目維持 ✓ |
| 9130 entry/watch 復帰 | 0 | ✓ |
| 9247 | entry rank 1 | 18 連続、固定化継続 |
| 9628 | entry rank 2 | 18 連続、同 sector |
| 4404 | entry rank 3 / risk warning | 12 日目、warning 表示のみ entry 維持 |
| 4404 watch 戻り | 0 | risk 扱い矛盾無 ✓ |
| W3 新顔 6 件 | 全 entry 継続 | ✓ |

## §7 安全 gate

| gate | 結果 |
|---|---|
| 低流動性 entry 混入 | 0 ✓ |
| letter-suffix entry 混入 | 0 ✓ |
| 非 100 株 entry 混入 | 0 ✓ |
| risk_yen 順位加点 | 0 ✓ |
| risk_yen entry 除外 gate | 0 ✓ |
| risk warning entry 維持 | 4 件 (= 4404/3479/4540/8057) ✓ |
| consumer validation_passed | True ✓ |
| violations | 0 ✓ |
| 9130 demote 状態 | 維持 ✓ |
| DB write / staging write / production-develop 接続 | 全 0 ✓ |
| API / token / LINE / launchctl / plist / cron / workflow / VACUUM | 全 0 ✓ |
| git add / commit / push / --no-verify | 全 0 ✓ |
| TODO Excel 更新 / sudo / rm -rf | 全 0 ✓ |
| auto-order / Computer Use / 楽天証券・iSPEED 自動操作 | 全 0 ✓ |

## §8 iSPEED 確認 10 項目

1. 寄付前気配 / 気配指値 (= top 5 全件)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動 (= 特に新顔 9008 / 3134 確認)
4. スプレッド
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ %
7. セクター指数の相対強弱 (= 運輸・物流 / 小売 新セクター注意)
8. **4404 限定**: risk_yen 10,665 円 / 損切り 2,026 円 / 許容損失改めて確認
9. **9247 / 9628 18 連続**: fixed-candidate HIGH、同 sector 重複片方絞り推奨
10. **W3 新顔 (9008/3134)**: 流動性確認、ギャップ確認

## §9 D49 トレード手順

### 9.1 pre-open
1. F111 D49 確認 (= raw 50、entry 20、新顔 6 件継続)
2. top 5 advisory 確認 (= 主役 3 + 新顔 2)
3. iSPEED で top 5 確認
4. Fujiwara 最終判断:
   - 9247 / 9628: 連続 18 日、fixed-candidate HIGH、片方絞り推奨
   - 4404: entry 検討可、risk 警告許容判断
   - 9008 (新顔): 運輸・物流 sector 多様化、低 risk
   - 3134 (新顔): 小売 sector 多様化、最低 risk
   - 9130: skip 確定

### 9.2 close 後 review
1. review template 埋め (= 10 項目)
2. W3 universe 拡張効果実 entry 結果評価
3. 新顔 (9008/3134) の実取引観察
4. D50 朝 pilot 準備

## §10 safety footer

- 本 trade plan は **手動発注用 reference** (= read-only advisory)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / staging write 0 / API 0 / token 0 / LINE 0 / launchctl 0 / plist 0 / cron 0 / workflow 0 / VACUUM 0
- 9247/9628 demote 本実行: 別 wave (= HQ approve 後)
- F111 max=50 baseline / signal_persistence top-n=100 baseline 維持
- TODO Excel 更新 0 / git add / commit / push / --no-verify 0

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
