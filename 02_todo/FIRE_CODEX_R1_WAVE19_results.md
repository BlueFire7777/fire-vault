---
id: FIRE-CODEX-R1-WAVE19-results
phase: ガバナンス / Wave 19 完了 / F101 + staging cleanup
priority: 高
status: 完了 ★ 3 sub-task / API call 0 / staging write 0 / 4,024 PASS 維持
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 18 (= 完了)
  - HQ Wave 19 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F101 + staging cleanup policy
---

# Wave 19: F101 API 403 調査 + staging smoke row cleanup policy

最終更新: 2026-05-12

## ★ 状態: 完了 (= 3 sub-task、全 read-only / vault doc 中心)

W18-2 で発見した F101 API 403 別 issue を静的調査 + W17-3/W18-1 残置 row
の inventory + filter policy 確立。実 API call 0、staging write 0。

## Wave 19 sub-task 結果

| sub | task | 結果 |
|---|---|---|
| W19-1 | F101 API behavior investigation | ✓ vault doc、推奨対応 3 段階提示 |
| W19-2 | staging smoke row inventory + cleanup policy | ✓ vault doc、filter 方針確定 |

## W19-1 F101 API 調査結果

[[../03_design/F101_API_403_investigation_2026-05-12]]

### key 発見

- base URL: `https://api.jquants.com/v2` (= F100 / F101 共通)
- F100 系 `/fins/details/summary/dividend` 等は **動作確認済**
- F101 `/fins/announcement` のみ HTTP 403 "endpoint does not exist"

### 仮説 (= 3 候補)

A) endpoint 名違い (= /fins/announcements 複数形 / /fins/disclosure 等)
B) plan 制限 (= Free / Light で利用不可)
C) deprecated (= V1 のみ、V2 で代替経路)

### 推奨対応 (= 別 wave / HQ approve 必要)

短期:
- A1. J-Quants V2 spec (= https://jpx-jquants.com/spec/) 確認
- A2. 候補 endpoint 名で実 dry-run 試行 (= HQ approve 後)
- A3. plan 確認

中期:
- B1. announcements 取得経路の代替実装
- B2. F101 client 修正 + tests + smoke
- B3. cron 連携 (= sub-D3 凍結中)

長期:
- C1. announcements / disclosure API 取得の冗長化

### 影響範囲

- F101 fetch_announcements 動作不可 (= 新規取得不可)
- 既存 staging announcements 1,098 row は W18 以前の取得 data
- W17-2-fix guard は機能 (= staging-only enforce、API fail で safe stop)

## W19-2 staging smoke row inventory + cleanup policy

[[../03_design/F286_staging_smoke_row_inventory_cleanup_policy_2026-05-12]]

### staging 残置 smoke row (= 2026-05-12)

| sub-task | table | 識別 marker | row count |
|----------|-------|------------|-----------|
| W17-3 | research_watchlist_signals | source_version='w17-3-smoke' | 35 |
| W18-1 | market_prices_daily | date='2026-05-08' AND code IN (72030,99840,67580) | 3 (= REPLACE) |
| W18-2 | (= 書込未発生) | - | 0 |
| W18-3 | (= read-only) | - | 0 |

### filter 方針 (= 本番経路で smoke row 除外)

#### research_watchlist_signals

`SMOKE_SOURCE_VERSIONS_BLACKLIST = ("w17-3-smoke",)` を application で
適用、`source_version NOT IN` で filter。

#### market_prices_daily

INSERT OR REPLACE のため smoke row と本番 row が区別不可。3 row は本番
fetch 経由と同等値、運用影響なし。

#### advisory_decisions

staging の 10 row は **本番経路で参照しない** (= production / develop の
advisory_decisions 使用)。

### 削除条件 (= 全 必須)

1. HQ 明示承認
2. 後続 wave で再利用不要
3. 削除前 inventory 再確認
4. 削除後 production / develop unchanged 確認

### 即時必須 filter 適用

- F119 評価 Agent
- REPORT-R1 daily / weekly / monthly
- Pattern Research / 候補抽出

## fire develop commits

本 Wave で commit なし (= 全 vault doc、code change 0)。

## fire-vault main commits

- (本起票) docs(FIRE-CODEX-R1): Wave 19 plan + W19-1 F101 調査 + W19-2 cleanup policy + log

## 安全要件 (= Wave 19 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信 | 0 |
| 実 API call | 0 (= 静的調査のみ) |
| production / develop / staging DB write | 0 |
| 全 DB mtime | unchanged |
| token / channel_token / secret 参照 | 0 |
| 楽天 / 自動発注 / Computer Use / Playwright | なし |
| .github/workflows/ 変更 | 0 |
| --no-verify | 不使用 |
| cron / launchd / crontab 本番登録 | 0 |
| scripts/seed_pattern_layer1.py | 未接触 |
| simulation/research_lane/historical_indicators.py | 未接触 |
| TODO Excel | 未更新 |
| Codex 直接 commit | 0 |
| subprocess 起動 | 0 |

## tests

4,024 PASS 維持 (= 本 Wave で code change 0)。

## HQ 判断が必要な論点 (= 3 件)

1. **Wave 19 完了 → 次フェーズ進行可否** (推奨: approve)

2. **F101 API 修正実装着手判定** (= 別 wave、別 HQ approve):
   - V2 spec 確認 → 代替 endpoint 試行 → client 修正
   - Codex L3+L2 で実装、L4 audit
   - 段階的 HQ approve (= spec 確認 / endpoint 試行 / client 修正 / smoke 各別)

3. **staging smoke row cleanup 着手判定** (= 別 wave、別 HQ approve):
   - 削除は当面不要 (= 後続 paper_pnl smoke 等で再利用可)
   - filter policy は本番 application 実装時に適用

## Wave 20 候補

- W20-X: F101 API 修正 phase 1 (= V2 spec 確認 + 候補 endpoint 試行)
- W20-X: filter helper module 実装 (= application 経路で smoke row 除外)
- W20-X: DATA-R3 sub-D2.3.x final audit (= W18 で実施済、再度なら別 task)
- W20-X: cron thaw design (= sub-D3 凍結解除設計、HQ 明示承認待ち)

## 関連リンク

- [[FIRE_CODEX_R1_WAVE18_results|Wave 18 results]]
- [[../03_design/F101_API_403_investigation_2026-05-12|W19-1 F101 調査]]
- [[../03_design/F286_staging_smoke_row_inventory_cleanup_policy_2026-05-12|W19-2 cleanup policy]]
- [[../log]]
