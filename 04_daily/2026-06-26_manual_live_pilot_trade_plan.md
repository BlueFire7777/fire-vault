---
template: manual-live-pilot-trade-plan
version: 1.4.1
date: 2026-06-26
weekday: 金
owner: BlueFire7777 (Fujiwara)
pilot_day: D32
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: GO_CONDITIONAL maintained
post_recovery: 13 連続 maintained
hold_2_avoidance: 完全継続 (= D15 以降 17 連続)
demoted_count: 7
nine130_status: "signal_success 継続 + no-new-chase (= D31 overshoot 反映)"
entry_priority_top: 9247 (= ＴＲＥ HD、サービス業 / リサイクル)
related:
  - 04_daily/2026-06-26_morning_advisory_summary.md
  - 04_daily/2026-06-26_manual_live_pilot_review.md
  - 04_daily/2026-06-25_manual_live_pilot_review.md (= D31 filled、学び反映 source)
  - 03_design/FIRE_selection_policy_v1.4.1_2026-05-15.md
---

# Manual Live Pilot — Trade Plan (2026-06-26 / D32)

## §1 D31 recap

- D31 = GO_CONDITIONAL maintained 12 連続、HOLD #2 完全回避 16 連続
- 9130 D31: signal_success ✓、entry_success NO (= 追わない上限超過)
- 331A0 / 4389: excluded_validated
- Fujiwara 全 entry skip、operational decision valid
- v1.4.1 framework validated

## §2 D32 entry_priority (= post-open override 反映、100 株標準) ★

⚠ **100 株標準方針** (= 本 wave 修正):
- 標準 entry 単位 = 100 株、非 100 株調整は不採用
- 100 株 risk_yen が他 entry 候補と大幅乖離する銘柄は entry から除外
- 該当銘柄 (= 4404) は watch_candidate へ降格

| entry_priority | code | name | 33 sector | 基準株価 | 100 株 risk_yen |
|---|---|---|---|---|---|
| **#1** ★ | 9247 | ＴＲＥ HD | サービス業/リサイクル | 1,612 | 8,060 |
| **#2** | 9628 | 燦 HD | サービス業/葬祭 | 1,390 | 6,950 |
| **NA (no-new-chase)** | **9130** ⚠ | 共栄タンカー | 海運業/タンカー | 1,410-1,479 | (entry 推奨せず) |
| **watch (= 降格)** | **4404** ⚠ | ミヨシ油脂 | 食品工業/油脂 | 2,133 | 10,665 (= 100 株で entry 候補外) |

### 第一確認候補: 9247 ＴＲＥ HD (= 100 株 entry)

## §3 9130 no-new-chase + 6 条件再 entry rule

D31 で +4.89% (= 1,479 円)、追わない上限 1,440 円超過。

新規 entry 原則禁止。以下 **全 6 条件** 充足時のみ再検討可:

1. 1,410-1,440 円付近まで自然に押す
2. 出来高と板が維持される
3. 反発確認 (= 押し目買い、底値形成)
4. 損切り位置が近く置ける (= 1,395 円以下)
5. リスクリワード 成立 (= R/R ≥ 1.5)
6. 新しい材料 / 需給根拠

→ 1 つでも欠ければ:
- signal_success 継続記録
- no-new-chase
- demote 候補 monitor (= D34/D35 で正式 demote 検討)

## §4 9130 demote 想定 timeline

| day | 9130 rank 1 連続 | status |
|---|---|---|
| D30 | 1 (初昇格) | 7991 demote 後の新 #1 |
| D31 | 2 | +4.89% overshoot |
| **D32 (本日)** | **3 (想定)** | **entry 候補外、signal 継続観察** |
| D33 | 4 | 同上 |
| D34 | **5 = warning** | demote sim 候補化 |
| D35 | demoted | 段階 3 本実行候補 |

## §5 entry 候補詳細

### 9247 ＴＲＥ HD (= 第一確認) ★

- liquidity_status: PASS (= 貸借 + TOPIX Small 1)
- 33 sector: サービス業 / リサイクル・環境
- 基準株価: 1,612 円
- エントリー検討: 1,596 - 1,628 円
- 指値目安: 1,612 円
- 追わない上限: 1,644 円 (+2.0%)
- 利確目安: 1,660 - 1,693 円 (+3-5%)
- 損切り: 1,532 円 (-80 円)
- 想定株数: 100 株
- 想定 risk_yen: 8,060 円

### 9628 燦 HD

- liquidity_status: PASS (= 貸借 + TOPIX Small 2)
- 33 sector: サービス業 / 葬祭
- 基準株価: 1,390 円
- エントリー検討: 1,376 - 1,404 円
- 指値目安: 1,390 円
- 追わない上限: 1,418 円 (+2.0%)
- 利確目安: 1,432 - 1,460 円
- 損切り: 1,320 円
- 想定株数: 100 株
- 想定 risk_yen: 6,950 円

### 4404 ミヨシ油脂 — watch_candidate へ降格、entry 候補外 ⚠

- liquidity_status: PASS (= 貸借)
- 33 sector: 食品工業 / 油脂
- 基準株価: 2,133 円
- 100 株 risk_yen = 10,665 円 (= #1 9247 risk 8,060 を大幅超)
- **100 株標準方針** で entry_candidate から watch へ降格
- 非 100 株調整: **不採用** (= 標準は 100 株のみ)
- signal は positive (research 0.8413)、ただし 100 株 entry の risk は entry_priority から除外
- entry 不可、観察のみ

## §6 朝 iSPEED 確認軸 (= 7 軸)

1. daily volume ≥ 10 万株
2. turnover ≥ 1 億円
3. opening volume (寄り 5 分) ≥ 1,000 株
4. spread + board depth
5. 寄り後 5 min 出来高 + 上昇率
6. ニュース / 適時開示 / catalyst
7. **9130 価格帯確認** (= 6 条件チェック)

## §7 actual price + liquidity + event (= 朝記入)

| code | actual 6/26 | 乖離率 | liquidity | event | OK/NG |
|---|---|---|---|---|---|
| 9247 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG (= entry #1) |
| 9628 | ____ | __.__% | ____ | ____ | ☐ OK ☐ NG (= entry #2) |
| **4404 (= watch のみ)** | ____ | __.__% | (= entry 不要) | (= entry 不要) | (= 観察、降格) |
| **9130 (= 監視のみ)** | ____ | __.__% | (= entry 不要) | (= entry 不要) | (= signal 観察、no-new-chase) |

## §8 final decision (= 100 株標準)

- ☐ **enter 9247 (= サービス業/リサイクル、第一確認、100 株)** ★
- ☐ enter 9628 (= サービス業/葬祭、代替、100 株)
- ☐ **NO enter 4404** (= 100 株 risk_yen 上限超、watch 降格、entry 候補外)
- ☐ **NO enter 9130** (= signal 観察 + no-new-chase、6 条件充足時のみ例外検討)
- ☐ watch / skip

## §9 safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
