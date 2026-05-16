---
id: FIRE-F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1-RESULT-2026-05-16
phase: 実装結果 doc (= W1 read-only simulation/design 完了)
priority: 最高
status: read-only simulation 完了 / 9247-9628 排除しない設計確認 / W2 実装可否判断材料整備
owner: BlueFire7777 (Fujiwara)
date: 2026-05-16
parent_design: 03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_2026-05-16.md
sibling_universe_w3: 03_design/F111_UNIVERSE_EXPANSION_R1_W3_result_2026-05-16.md
d49_morning_summary: 04_daily/2026-07-22_morning_advisory_summary.md
codex_lanes: 4
critical_high_count: 0
---

# F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W1 read-only simulation/design 結果 (= 2026-05-16)

## §1 目的再定義 (= ユーザー追加条件反映)

D49 W3 universe 拡張は成功 (= raw 50 / entry 20 / sector 9) したが、9247/9628 rank 1/2 継続.
**「固定だから外す」のではなく、継続上昇主役候補なのか、惰性固定なのか**を判別できる
theme/sector dynamic overlay の read-only simulation 設計.

重要方針 (= D49 で確定):
- 9247/9628 を消すことが目的ではない
- raw score 順位を常に保存 (= overlay 順位と併記)
- 継続上昇候補なら主役枠に残す
- 新顔/新 sector も比較枠で朝判断の選択肢拡張
- theme overlay 目的 = 「主役排除」ではなく **「継続上昇 vs 惰性固定の判別」**

## §2 入力

- F111: `/tmp/fire_d49_prep/d49_f111.json` (= D49 base_date 2026-07-22 / raw 50)
- consumer: `/tmp/fire_d49_prep/d49_consumer_payload.json` (= entry 20)

## §3 current raw score top5 (= 既存ロジック、reference)

| rank | code | 銘柄 | sector | score | risk |
|---|---|---|---|---|---|
| 1 | 9247 | ＴＲＥ HD | 情報通信 | 0.8546 | 8,060 |
| 2 | 9628 | 燦 HD | 情報通信 | 0.8479 | 6,950 |
| 3 | 4404 | ミヨシ油脂 | 食品 | 0.8413 ⚠ | 10,665 |
| 4 | 2146 | ＵＴグループ | 情報通信 | 0.8305 | 920 |
| 5 | 4828 | ビジネスエンジニアリング | 情報通信 | 0.8293 | 6,230 |

→ sector 2 種、9247-9628 同 sector 重複、新顔・新セクター 0 件.

## §4 overlay simulation top5 (= theme/sector dynamic overlay 案)

| overlay | raw rank | code | 銘柄 | sector | role | 選定理由 |
|---|---|---|---|---|---|---|
| 1 | 1 | 9247 | ＴＲＥ HD | 情報通信 | **主役継続枠 1** | score 0.8546、TOPIX Small 1、貸借、新顔から +0.04 優位 |
| 2 | 2 | 9628 | 燦 HD | 情報通信 | **主役継続枠 2** | score 0.8479、9247 同 sector だが事業 (葬祭) 別 |
| 3 | 3 | 4404 | ミヨシ油脂 | 食品 | **risk warning 枠** | score 0.8413、risk warning 付き entry 維持 |
| 4 | **16** | 9008 | 京王電鉄 | **運輸・物流 (新)** | **新顔+新sector** | W3 新顔、score 0.8119 で 9247 から -0.04、低 risk |
| 5 | **19** | 3134 | Ｈａｍｅｅ | **小売 (新)** | **新顔+新sector** | W3 新顔、score 0.8051、最低 risk 2,060 円 |

→ **sector 4 種 (= +2 拡張)** / **新顔・新セクター枠 2 件確保** ✓

## §5 9247 / 9628 継続主役 vs 惰性固定 判定

### §5.1 raw score 優位性 (= Lane A audit 修正反映)

| code | raw score | 新顔最高 (9417) | 優位 vs 9417 | 9008 比較 | 判定 |
|---|---|---|---|---|---|
| 9247 | 0.8546 | 0.8193 | **+0.0353 (+4.3%)** | +0.0427 (+5.2%) vs 0.8119 | 上位 |
| 9628 | 0.8479 | 0.8193 | **+0.0286 (+3.5%)** | +0.0360 (+4.4%) vs 0.8119 | 上位 |

注: 新顔 6 件 (= 9417/9008/6196/7595/3134/3962) の最高 score は **9417 = 0.8193** (= 情報通信、scale 不明).
9008 (運輸物流、新 sector) = 0.8119、3134 (小売、新 sector) = 0.8051.

### §5.2 sector / scale / liquidity / risk

| code | sector | scale | margin | risk_yen 100株 | risk_over |
|---|---|---|---|---|---|
| 9247 | 情報通信・サービスその他 | TOPIX Small 1 | 貸借 | 8,060 | False |
| 9628 | 情報通信・サービスその他 | TOPIX Small 2 | 貸借 | 6,950 | False |

→ 低流動性ではない、信用可、risk 上限内.

### §5.3 結論

**継続主役候補** (= 単純な惰性固定ではない):
- score 上位 0.84+ で新顔から有意な優位 (= +0.03-0.04)
- scale OK、信用可、risk 上限内
- W3 拡張後 50 件母集団でも score 上位継続

**懸念点**:
- 18 連続継続 (= 9130 閾値 5 連続の 3.6 倍超)
- 同 sector_17 重複 (= 情報通信 top 1/2)
- 新顔との score 差は overlay の影響を受けやすい (= 0.04 程度)

**判定**: 継続主役候補として残す ✓ (= 本 wave demote 0)
**残課題**: theme overlay (= volume momentum / 相対強度 / theme tag) で 継続上昇 vs 惰性失速 を実測判別

## §6 overlay 設計案 (= W2 実装候補)

### §6.1 overlay 指標

| 指標 | 内容 | 用途 |
|---|---|---|
| theme_tag | seed list + dynamic discovery (半導体/AI/電線/防衛 等) | sector_17 粒度を細分化 |
| volume_momentum | 直近 N 日平均出来高 / 過去 N 日平均出来高 | 継続上昇 vs 失速判別 |
| turnover_momentum | 売買代金 momentum | 規模考慮 |
| sector_relative_strength | sector 相対強度 | 主役 sector vs 衰退 sector 判別 |
| consecutive_signal_success | 連続日数別 +2R hit rate | 主役継続 vs 惰性失速の実績 |

### §6.2 overlay 適用 logic

```
入力: F111 entry 20 件
↓ overlay 分類 (5 枠):
  主役継続枠 (raw 1-5 + momentum > 0 + 相対強度 > 0)
  risk warning 枠 (risk_yen_warning 付き、警告表示)
  新顔枠 (recently_seen 履歴に無 or W3 後初出現)
  新 sector 枠 (前日 entry に無 sector)
  比較枠 (残り)
↓ overlay top5 (= 各枠から選定):
  主役継続 1-2 + risk warning 必要時 1 + 新顔 ≥1 + 新 sector ≥1 + 比較 残り
↓ 各候補に role 表示 + raw rank 併記
```

### §6.3 9247/9628 への 4 分類

| シナリオ | overlay 判定 |
|---|---|
| volume_momentum > 0 + 相対強度 > 0 | **継続主役** → 主役枠維持 |
| volume_momentum < 0 + 相対強度 < 0 | **惰性固定注意** → 比較枠降格 |
| volume_momentum > 0 + 相対強度 < 0 | 個別強気 → 主役枠維持、sector 警戒 |
| volume_momentum < 0 + 相対強度 > 0 | sector 強気個別弱 → 比較枠 + 同 sector 新顔検討 |

## §7 安全 gate 確認

| gate | 結果 |
|---|---|
| 9130 excluded 維持 | ✓ |
| 9130 entry/watch 復帰 | 0 ✓ |
| 低流動性 entry 混入 | 0 ✓ |
| letter-suffix entry 混入 | 0 ✓ |
| 非 100 株 entry 混入 | 0 ✓ |
| risk_yen 順位加点 / 除外 gate | 0 ✓ |
| risk warning entry 維持 | 4 件 (= 4404/3479/4540/8057) ✓ |
| consumer validation_passed | True ✓ |
| 9247/9628 demote 本実行 | 0 ✓ |
| raw score 順位保存 | ✓ (= overlay rank と併記) |
| 新顔/新 sector ≥1 件 | ✓ (= 9008/3134 で 2 件) |

## §8 W2 実装可否

| 判断材料 | 結論 |
|---|---|
| current vs overlay top5 比較可能 | ✓ |
| 9247/9628 継続主役/惰性固定 両面評価可能 | ✓ |
| 新顔/新 sector ≥1 件 | ✓ |
| top5 選定理由説明可能 | ✓ |
| safety gate all 0 | ✓ |

→ **W2 実装可** ✓

### §8.1 W2 で必要な実装要素

1. theme_tags table 追加 (= staging DB schema、別 wave で migration)
2. seed theme list (= JSON / YAML / DB)
3. assign_theme_tags runner (= 銘柄 → theme list mapping)
4. volume_momentum / turnover_momentum 計算 module (= market_prices_daily 集計)
5. sector_relative_strength 計算 module (= 指数 vs 銘柄)
6. overlay top5 builder (= consumer 拡張、role 分類)
7. morning advisory MD renderer overlay 対応

## §9 CRITICAL / HIGH

- **CRITICAL: 0**
- **HIGH: 0**

## §10 D50 handoff

- 朝 pilot top5 は overlay 案で運用検討 (= 9247/9628 主役継続 + 4404 warning + 9008/3134 新顔/新sector)
- F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2 実装 wave 起票 (= HQ approve 後)
- theme overlay クリア条件: 「主役排除」ではなく「継続上昇 vs 惰性固定の判別」
- 9247/9628 demote 本実行は引き続き 0 (= score 上位の正当な主役候補)
- 9130 demote / 4404 risk warning / 安全 gate 全 0 継続

## §11 safety footer

- 本 wave は **read-only simulation / design** (= DB write 0 / staging write 0 / production-develop 接続 0)
- API 0 / token 0 / LINE 0 / launchctl 0 / plist 0 / cron 0 / workflow 0 / VACUUM 0
- git add 0 / commit 0 / push 0 / --no-verify 0
- TODO Excel 更新 0 / sudo 0 / rm -rf 0
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use 0
- 9247 / 9628 demote 本実行 0 (= 継続主役枠維持)、9130 demote 状態維持
- fire repo コード変更 0、一時 script は /tmp 配下のみ

### F282 plist 3 file 別個確認注記
- production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
- smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
- LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)

## §12 関連 file

- current top5 JSON: `/tmp/fire_theme_sector_overlay_w1/current_score_top5.json`
- overlay top5 JSON: `/tmp/fire_theme_sector_overlay_w1/overlay_simulation_top5.json`
- comparison MD: `/tmp/fire_theme_sector_overlay_w1/overlay_comparison.md`
- 親設計 (sibling theme): `~/fire-vault/03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_2026-05-16.md`
- sibling universe W3: `~/fire-vault/03_design/F111_UNIVERSE_EXPANSION_R1_W3_result_2026-05-16.md`
- D49 入力: `~/fire-vault/04_daily/2026-07-22_morning_advisory_summary.md`
