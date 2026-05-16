---
id: FIRE-pilot-D47-trade-plan-2026-07-17
phase: 本番 v0 / W60-pilot-D47 / top-n=100 + max=50 baseline 初回稼働
priority: 高
status: pre-open / GO_CONDITIONAL
owner: BlueFire7777 (Fujiwara)
date: 2026-07-17 (= D47、金)
parent_w2_result: 03_design/F111_UNIVERSE_EXPANSION_R1_W2_result_2026-05-16.md
codex_lanes: 4
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-07-17 / D47)

## §1 全体方針

**v1.4.2 + top-n=100 baseline 初回稼働 + max=50 baseline 5 日目** の朝 pilot.

- top-n=100 baseline 自然適用 (= signal_persistence default、W2 完了)
- max=50 baseline 自然適用 (= F111 default、W2 baseline 化完了)
- 9130 demote 効果継続 12 日目 (= excluded 維持)
- 4404 v1.4.2 entry 復帰 10 日目 (= ⚠ risk warning)
- 9247 / 9628 連続 16 日目 (= fixed-candidate HIGH 継続、demote 保留)

## §2 D47 baseline 出力

- F111: cli=1.3.0 / max=50 / policy=1.4.2 / demoted_count=8 / base_date=2026-07-17
- F111 raw: 35 (= staging signals latest=35 件、signal_persistence dry-run で 未書込)
- signal_persistence default smoke: top_n=100 / persistence 109 件 / DB write 0 / dry-run

| role | 件数 | codes |
|---|---|---|
| entry_candidate | 14 | 9247, 9628, 4404, 2146, 4828, 8699, 3089, 3712, 7803, 9633, 4914, 3479, 4540, 8057 |
| watch_candidate | 0 | |
| excluded_candidate | 21 | demote 8 (= +9130) + liquidity FAIL 13 |

## §3 優先確認 top 5

### 3.1 [top 1] 9247 ＴＲＥ HD [連続 16 日目、fixed-candidate HIGH]
価格 1,612 円 / 100 株 / 追わない 1,644 / 利確 1,773 / 損切り 1,531 / risk_yen 8,060 円

### 3.2 [top 2] 9628 燦 HD [連続 16 日目]
価格 1,390 円 / 100 株 / 追わない 1,418 / 利確 1,529 / 損切り 1,320 / risk_yen 6,950 円

### 3.3 [top 3] 4404 ミヨシ油脂 [⚠ risk warning、連続 10 日目]
> ⚠ risk warning: 100 株 risk_yen = 10,665 円 → 損切り/板/spread/RR 手動確認

価格 2,133 円 / 100 株 / 追わない 2,176 / 利確 2,346 / 損切り 2,026 / risk_yen 10,665 円 ⚠

### 3.4 [top 4] 3089 テクノアルファ [新規継続、商社 sector]
価格 1,055 円 / 100 株 / 追わない 1,076 / 利確 1,160 / 損切り 1,002 / risk_yen 5,275 円

### 3.5 [top 5] 9633 東京テアトル [新規継続、不動産 sector]
価格 1,579 円 / 100 株 / 追わない 1,610 / 利確 1,737 / 損切り 1,500 / risk_yen 7,895 円

## §4 baseline 適用確認

| 項目 | 期待 | 実測 |
|---|---|---|
| F111 cli_version | 1.3.0 | **1.3.0** ✓ |
| F111 max_candidates | 50 | **50** ✓ |
| F111 policy_version | 1.4.2 | **1.4.2** ✓ |
| signal_persistence top_n | 100 | **100** ✓ (= dry-run summary) |
| signal_persistence persistence 件数 | 109 | **109** ✓ |
| F111 raw | 35 | 35 (= staging signals 未更新、後 wave で拡張) |

## §5 9247 / 9628 固定化リスク

| code | 連続 entry 日数 | 評価 |
|---|---|---|
| 9247 | **16 日連続** (D32-D47) | fixed-candidate HIGH 継続 |
| 9628 | **16 日連続** (D32-D47) | 同上 |

- 9130 demote 閾値 (= 5) の **3.2 倍超**
- 本 wave で demote 本実行: 無
- D48 以降の HQ approve 後の片方 demote 検討 (= sim A or B、両方同時禁止維持)
- 中期的解決: F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1 (= 別 wave)

## §6 sector 多様化

| sector | 件数 (entry) | codes |
|---|---|---|
| 情報通信・サービスその他 | 6 | 9247, 9628, 2146, 4828, 3712, 7803 |
| 商社・卸売 | 2 | 3089, 8057 |
| 不動産 | 2 | 9633, 3479 |
| 食品 | 1 | 4404 |
| 金融（除く銀行） | 1 | 8699 |
| 素材・化学 | 1 | 4914 |
| 医薬品 | 1 | 4540 |
| 合計 | **14** | **7 sector** |

→ D43-D47 安定 7 sector. top-n=100 staging write 後の D48+ で sector 拡張期待 (= 12 sector へ).

## §7 安全 gate

| gate | 結果 |
|---|---|
| 9130 excluded 維持 | ✓ (recently_seen_demoted、12 日目) |
| 9130 entry / watch 復帰 | 0 ✓ |
| 9247 / 9628 demote 本実行 | 0 ✓ |
| 両方同時 demote | 禁止維持 ✓ |
| 低流動性 entry 混入 | 0 ✓ |
| letter-suffix entry 混入 | 0 ✓ |
| 非 100 株 entry 混入 | 0 ✓ |
| risk_yen 順位加点 | 0 (= v1.4.2 維持) ✓ |
| risk_yen entry 除外 gate | 0 (= v1.4.2 維持) ✓ |
| risk_yen_over_pilot_budget warning | 表示のみ ✓ (= 4 件 entry 維持) |
| 100 株標準 | 全 14 entry standard_lot_ok=True ✓ |
| LINE / API / DB write / token | 全 0 ✓ |

## §8 iSPEED 確認項目

1. 寄付前気配 / 気配指値 (= top 5)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動
4. スプレッド
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ %
7. セクター指数の相対強弱
8. **4404 限定**: risk_yen 10,665 円 / 損切り 2,026 円 / 許容損失改めて確認
9. **9247 / 9628 16 連続**: fixed-candidate HIGH、同 sector 重複片方絞り推奨
10. **新規 (3089/9633)**: D44-D47 安定、流動性確認

## §9 D47 トレード手順

### 9.1 pre-open
1. F111 D47 確認 (= max=50 baseline)、signal_persistence top-n=100 baseline 確認
2. top 5 advisory 確認
3. iSPEED で top 5 確認
4. Fujiwara 最終判断:
   - 9247 / 9628: 連続 16 日目、fixed-candidate HIGH、慎重判断
   - 4404: entry 検討可、risk 警告許容判断
   - 3089 / 9633: 新規継続、流動性確認
   - 9130: skip 確定

### 9.2 close 後 review
1. review template 埋め (= 10 項目)
2. top-n=100 + max=50 baseline 5 日目運用感想
3. staging signals 書込 timing 確認 (= 別 wave / cron timing)
4. D48 朝 pilot 準備 (= 海の日明け 2026-07-21 火)

## §10 safety footer

- 本 trade plan は **手動発注用 reference** (= read-only advisory)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0 / plist 0 / cron 0 / workflow 0
- 9247/9628 demote 本実行は別 wave (= HQ approve 後)
- F111 max=50 baseline / signal_persistence top-n=100 baseline
- TODO Excel 更新 0
- git add / commit / push 0

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
