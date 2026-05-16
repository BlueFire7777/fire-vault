---
id: FIRE-pilot-D48-trade-plan-2026-07-21
phase: 本番 v0 / W60-pilot-D48 / 海の日明け / top-n=100 + max=50 baseline 2 日目
priority: 高
status: pre-open / GO_CONDITIONAL
owner: BlueFire7777 (Fujiwara)
date: 2026-07-21 (= D48、火、海の日明け)
parent_d47: 04_daily/2026-07-17_morning_advisory_summary.md
codex_lanes: 4
critical_high_count: 0
---

# Manual Live Pilot — Trade Plan (2026-07-21 / D48)

## §1 全体方針

**v1.4.2 + top-n=100 baseline 2 日目 + max=50 baseline 6 日目** の朝 pilot.
海の日明け (= 2026-07-20 月祝、4 営業日跨ぎ).

- top-n=100 baseline 自然適用 (= signal_persistence default、W2 完了)
- max=50 baseline 自然適用 (= F111 default)
- 9130 demote 効果継続 13 日目 (= excluded 維持)
- 4404 v1.4.2 entry 復帰 11 日目 (= ⚠ risk warning)
- 9247 / 9628 連続 17 日目 (= fixed-candidate HIGH 継続、4 営業日跨ぎでも entry 維持)
- staging signals **未更新** (= D47 から不変、universe 拡張未発現)

## §2 D48 baseline 出力

- signal_persistence dry-run: top_n=100 / persistence 109 件 / DB write 0
- F111: cli=1.3.0 / max=50 / policy=1.4.2 / base_date=2026-07-21
- F111 raw: **35** (= staging signals latest 2026-05-13 / 35 件、D47 と同等)

| role | 件数 | codes |
|---|---|---|
| entry_candidate | **14** | 9247, 9628, 4404, 2146, 4828, 8699, 3089, 3712, 7803, 9633, 4914, 3479, 4540, 8057 |
| watch_candidate | **0** | |
| excluded_candidate | **21** | demote 8 (= +9130) + liquidity FAIL 13 |

## §3 優先確認 top 5

### 3.1 [top 1] 9247 ＴＲＥ HD [連続 17 日目]
1,612 円 / 100 株 / 追わない 1,644 / 利確 1,773 / 損切り 1,531 / risk 8,060 円

### 3.2 [top 2] 9628 燦 HD [連続 17 日目]
1,390 円 / 100 株 / 追わない 1,418 / 利確 1,529 / 損切り 1,320 / risk 6,950 円

### 3.3 [top 3] 4404 ミヨシ油脂 [⚠ risk warning、連続 11 日目]
2,133 円 / 100 株 / 追わない 2,176 / 利確 2,346 / 損切り 2,026 / risk 10,665 円 ⚠

### 3.4 [top 4] 3089 テクノアルファ [D44-D48 entry 継続]
1,055 円 / 100 株 / 追わない 1,076 / 利確 1,160 / 損切り 1,002 / risk 5,275 円

### 3.5 [top 5] 9633 東京テアトル [D44-D48 entry 継続]
1,579 円 / 100 株 / 追わない 1,610 / 利確 1,737 / 損切り 1,500 / risk 7,895 円

## §4 baseline 適用確認

| 項目 | 期待 | 実測 |
|---|---|---|
| F111 cli_version | 1.3.0 | **1.3.0** ✓ |
| F111 max_candidates | 50 | **50** ✓ |
| F111 policy_version | 1.4.2 | **1.4.2** ✓ |
| signal_persistence top_n | 100 | **100** ✓ |
| signal_persistence persistence 件数 | 109 | **109** ✓ |
| F111 raw | 35 (= staging 未更新) | 35 |

## §5 staging signals 状態 (= 重要観察)

- latest base_date: **2026-05-13**
- latest count: **35 件**
- D47 → D48 で更新: **無**
- W2 baseline (top-n=100) は dry-run のみで反映、staging への定期 write 未稼働
- → F111 raw 35 件継続、universe 拡張未発現
- staging write 実行には **HQ_APPROVE_STAGING_SIGNAL_PERSISTENCE_WRITE** 必要 (= 別 wave)

## §6 sector 多様化 + 新顔

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

D44-D48 安定 7 sector. 追加新顔 / 新セクター: **0** (= staging signals 未更新による).

## §7 9247 / 9628 fixed-candidate 17 連続

| code | 連続 entry 日数 |
|---|---|
| 9247 | **17 日連続** (D32-D48、9130 閾値の 3.4 倍超) |
| 9628 | **17 日連続** (D32-D48) |

- 4 営業日跨ぎ (= 海の日含む) でも entry 維持 = 強い固定化
- 本 wave demote 本実行: **0** (= 範囲外)
- 両方同時 demote 提案: **禁止**
- 単独 demote 検討: D49 以降 + HQ approve 必須
- 中期解決: F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1 別 wave

## §8 安全 gate

| gate | 結果 |
|---|---|
| 9130 excluded 維持 | ✓ (= recently_seen_demoted、13 日目) |
| 9130 entry / watch 復帰 | 0 ✓ |
| 9247 / 9628 demote 本実行 | 0 ✓ |
| 両方同時 demote | 禁止維持 ✓ |
| 低流動性 entry 混入 | 0 ✓ |
| letter-suffix entry 混入 | 0 ✓ |
| 非 100 株 entry 混入 | 0 ✓ |
| risk_yen 順位加点 | 0 (= v1.4.2 維持) ✓ |
| risk_yen entry 除外 gate | 0 (= 4404/3479/4540/8057 entry 維持) ✓ |
| risk_yen_over_pilot_budget warning | 表示のみ ✓ |
| 100 株標準 | 全 14 entry standard_lot_ok=True ✓ |
| DB write / API / token / LINE / launchctl / plist / cron / workflow | 全 0 ✓ |
| staging write | 0 ✓ (= HQ 未承認、本 wave 範囲外) |
| git add / commit / push / --no-verify | 全 0 ✓ |
| TODO Excel 更新 / VACUUM / sudo / rm -rf | 全 0 ✓ |

## §9 iSPEED 確認 10 項目
1. 寄付前気配 / 気配指値 (= top 5)
2. 信用倍率 / 信用残
3. 板厚 / 出来高初動
4. スプレッド
5. 5 分足 5MA / 25MA 位置
6. 前日終値 / 当日寄値 / ギャップ % (= 海の日明け 4 営業日空白注意)
7. セクター指数の相対強弱
8. 4404 限定: risk_yen 10,665 円 / 損切り 2,026 円 / 許容損失改めて確認
9. 9247 / 9628 17 連続: fixed-candidate HIGH、同 sector 重複片方絞り推奨
10. 新規 (3089/9633): D44-D48 安定、流動性確認 + 海の日明けギャップ

## §10 safety footer

- 本 trade plan は **手動発注用 reference** (= read-only advisory)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0 / plist 0 / cron 0 / workflow 0
- staging write 0 (= HQ_APPROVE_STAGING_SIGNAL_PERSISTENCE_WRITE 未取得)
- 9247/9628 demote 本実行は別 wave (= HQ approve 後)
- F111 max=50 baseline / signal_persistence top-n=100 baseline 維持
- TODO Excel 更新 0
- git add / commit / push / --no-verify 0

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
