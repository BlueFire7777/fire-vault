---
template: manual-live-pilot-review
version: 1.4.1
date: 2026-06-26
weekday: 金
owner: BlueFire7777 (Fujiwara)
pilot_day: D32
status: filled (= Fujiwara 提供範囲のみ反映、OHLCV 数値は未提供のため pending)
fujiwara_input_received: true
fujiwara_input_scope: "見送り判断 + 理由 + actual_entry status のみ。OHLCV 数値は未提供。"
linked_advisory: 04_daily/2026-06-26_morning_advisory_summary.md
linked_trade_plan: 04_daily/2026-06-26_manual_live_pilot_trade_plan.md
linked_design: 03_design/FIRE_selection_policy_v1.4.1_2026-05-15.md
linked_d31_review: 04_daily/2026-06-25_manual_live_pilot_review.md (= filled、参考)
artifact_source: f062_preview
f062_raw_kind: f062_actual_dict
f111_input_source: f111_real_batch
pilot_judgment: GO_CONDITIONAL
post_recovery: 13 連続 maintained
hold_2_avoidance: 完全継続 (= D15 以降 17 連続)
nine130_status_d32: "signal 観察 + no-new-chase (= 既定維持)"
share_unit_policy: "100 株標準、非 100 株調整不採用"
entry_status: skip
actual_entry: none
final_judgment: "初回実弾投入後の慎重判断 / 出来高薄 / スプレッド大 / 約定撤退リスク懸念で全候補 skip。Fujiwara 判断は運用上妥当。"
ohlcv_data_provision: "未提供 (= Fujiwara input required または J-Quants auto-fill 待ち、推測禁止)"
---

# Manual Live Pilot — Trade Review (2026-06-26 / D32)

> ⚠ Fujiwara 提供範囲のみ反映。OHLCV 数値 (= close/change/high/low/open/volume/turnover)
> は未提供のため "Fujiwara input required または J-Quants auto-fill 待ち" で placeholder 維持。
> Claude が actual 数値を推測しない。

## §1 [必須 1] 入った / 見送った

**skip** (= 全 4 候補で actual_entry: none、Fujiwara 提供確定)

## §2 [必須 2] 判断理由 (= Fujiwara 本人の見送り理由)

> 「初回実弾投入後で慎重に見ており、出来高が薄く、スプレッドも大きく見えたため、
> 約定リスク・撤退リスクが心配だった。」

要点:
- D31 (= 初回実弾投入日) の経験を踏まえた継続慎重判断
- 出来高 / スプレッド観点で約定リスク懸念
- 撤退時のスリッページ懸念
- 全候補 skip = 保守判断、運用上妥当

## §3 [必須 3] 実際に見た銘柄 (= ticker / name + 33 sector、100 株標準)

- **9247 ＴＲＥ HD** (= サービス業 / リサイクル、entry_candidate #1、100 株想定)
- **9628 燦 HD** (= サービス業 / 葬祭、entry_candidate #2、100 株想定)
- **9130 共栄タンカー** (= 海運業、no-new-chase、観察のみ)
- **4404 ミヨシ油脂** (= 食品工業 / 油脂、watch 降格、entry 不可)

## §4 [必須 4] 板・出来高・spread + 前場/終日実績

| code | close | change | high | low | open | volume | turnover | 板/spread 所感 | event |
|---|---|---|---|---|---|---|---|---|---|
| 9247 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | 出来高薄 / spread 大 (= Fujiwara 観測) | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | 出来高薄 / spread 大 (= Fujiwara 観測) | Fujiwara input required |
| 9130 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= 観察のみ、Fujiwara input required) | Fujiwara input required |
| 4404 | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | Fujiwara input required | (= watch 降格、Fujiwara input required) | Fujiwara input required |

注: 数値項目は **Fujiwara 未提供** = 推測禁止で placeholder 維持。
J-Quants auto-fill または Fujiwara 観測 input で後埋め予定。

### 流動性所感 (= Fujiwara 提供範囲)

- 全候補で「出来高薄」「スプレッド大」を観測 (= 共通)
- 9247 / 9628 は entry_candidate だが約定 / 撤退リスク懸念で skip
- 9130 は no-new-chase 既定で観察のみ、actual 値は input pending
- 4404 は watch 降格既定で entry 不可、actual 値は input pending

## §5 [必須 5] 結果メモ

- **Fujiwara skipped all 4 candidates** (= D32 操作 = 全 skip)
- D31 (= 初回実弾投入日) で全 skip 後、D32 でも継続慎重判断
- 出来高薄 / スプレッド大 / 約定撤退リスク懸念は D31 と同じ pattern
- 数値での裏付け確認は OHLCV input 待ち (= 推測しない)

## §6 [必須 6] 実 entry か skip か (= 100 株標準)

- 9247: **skip** (= 100 株 entry 想定だったが見送り) ✓ Fujiwara 確定
- 9628: **skip** (= 100 株 entry 想定だったが見送り) ✓ Fujiwara 確定
- 4404: **NO enter (= watch 降格、100 株 risk_yen 上限超、entry 不可)** ✓ 既定 + Fujiwara 確認
- 9130: **NO enter (= no-new-chase 既定)** ✓ 既定 + Fujiwara 確認、6 条件充足判定は input pending

## §7 [必須 7] 入った価格 or 見送った理由

| code | 価格 / 見送り理由 |
|---|---|
| 9247 | 見送り (= 初回実弾投入後の慎重判断 / 出来高薄 / spread 大 / 約定撤退リスク懸念) |
| 9628 | 見送り (= 同上、Fujiwara 共通理由) |
| 4404 | NO enter (= watch 降格、100 株 risk_yen 上限超、entry 不可) — 確定 |
| 9130 | NO enter (= no-new-chase 既定、6 条件充足判定は input pending) |

## §8 [必須 8] 追わない上限を超えていたか

| code | 追わない上限 | 高値 | 超過 | 超過率 |
|---|---|---|---|---|
| 9247 | 1,644 | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 9628 | 1,418 | Fujiwara input required | Fujiwara input required | Fujiwara input required |
| 4404 | 2,176 | Fujiwara input required | (= watch、entry 不可) | (= 判定不要) |
| 9130 | 1,440 | Fujiwara input required | Fujiwara input required (= D31 から継続超過なら Yes) | Fujiwara input required |

→ 高値情報なしで超過判定不可、Fujiwara input または J-Quants auto-fill 待ち

## §9 [必須 9] pre-open 判断か post-open 判断か

| code | 判断 phase |
|---|---|
| 9247 | **pre-open** (= 朝の流動性懸念で見送り、Fujiwara 提供) |
| 9628 | **pre-open** (= 同上) |
| 4404 | **pre-open** (= watch 降格既定) — 確定 |
| 9130 | **pre-open** (= no-new-chase 既定) — 確定、6 条件 post-open 確認は input pending |

## §10 [必須 10] signal_success と entry_success の区別

| code | signal_success | entry_success | comment |
|---|---|---|---|
| 9247 | Fujiwara input required (= 数値未確認のため推測しない) | **NO** (= skip) | Fujiwara 慎重判断で skip、signal 判定は OHLCV input 後 |
| 9628 | Fujiwara input required (= 数値未確認のため推測しない) | **NO** (= skip) | 同上 |
| 4404 | Fujiwara input required (= 数値未確認のため推測しない) | **NA** (= watch 降格、entry 不可) | watch 妥当性 input pending |
| 9130 | Fujiwara input required (= D31 から継続観察、数値未確認のため推測しない) | **NA** (= no-new-chase) | D31 から signal 継続観察、D32 数値 input pending |

→ **signal_success / entry_success は別軸で記録**
→ entry_success は Fujiwara 確定 (= NO / NA)、signal_success は OHLCV 提供後に確定

---

## §11 9130 D32 状態 monitor

- D32 9130 rank 1 連続: **3 連続 (想定)**、actual 確認: Fujiwara input required
- D32 9130 actual close: Fujiwara input required
- D32 9130 高値: Fujiwara input required
- 1,440 円超過状態継続? Fujiwara input required (= D31 +4.89% からの継続性確認)
- 6 条件充足? Fujiwara input required (= 1,410-1,440 押し戻し / 出来高板 / 反発 / 損切り近置き / R/R / 新材料)
- demote 候補 status: monitor 継続 (= 既定、D34/D35 sim 着手準備)

## §12 4404 D32 状態 monitor (= watch 候補)

- D32 4404 actual close: Fujiwara input required
- 100 株 risk_yen が他候補と乖離継続? Fujiwara input required (= entry 候補外維持判定)
- watch 降格妥当性: Fujiwara input required (= signal observation 評価後)
- D33 以降の扱い: Fujiwara input required (= watch 継続 / excluded / entry 候補復帰)

## §13 D33 / D34 / D35 引き継ぎ memo

- D33 (= 2026-06-29 月) 観察項目: 9247 / 9628 actual flow 再観察、9130 連続性
- D34 (= 2026-06-30 火) sim 着手要否: 9130 5 連続到達なら sim 着手
- D35 (= 2026-07-01 水) demote 本実行可否: D34 sim 結果次第

## §14 W5 集約 (= D20-D33 14 day) memo

- D31 reality: 9130 signal Y / entry N、331A0 excluded validated、4389 watch_signal_success
- D32 reality: 全 skip (= 初回実弾後の継続慎重判断)、OHLCV input 待ち
- selection policy v1.4.1 framework 継続検証: actual data 提供後
- 100 株標準方針実弾検証: 9247 / 9628 entry 機会あったが skip = 方針自体の妥当性は維持

## §15 v1.4.1 framework validation (= D32 結果反映後に判定)

D31 で validated 済の 4 軸:
- ✓ pre_open / entry_priority 分離
- ✓ signal_success vs entry_success 分離
- ✓ 追わない上限超過時の no-new-chase logic
- ✓ liquidity gate (= 331A0 excluded 妥当)

D32 で追加検証する観点:
- 100 株標準方針 (= 4404 watch 降格妥当か): Fujiwara observation 後判定
- 9130 D32 no-new-chase 継続 (= 価格帯と 6 条件): Fujiwara observation 後判定
- 9247 / 9628 entry 妥当性: Fujiwara observation 後判定 (= 出来高薄 / spread 大の状況下)

## §16 Next action

- ✓ **Fujiwara 提供範囲 (= skip 判定 + 見送り理由 + entry_success NO/NA) で D32 review fill 完了**
- ☐ OHLCV 数値 input (= Fujiwara 観測 or J-Quants auto-fill) → §4 / §8 / §10-12 の Fujiwara input required を上書き
- ☐ signal_success 判定 (= OHLCV 数値後、各銘柄 +/- change で判定)
- ☐ paper PnL handoff (= 場後、`--review-md` 付きで再 run、OHLCV 提供後)
- ☐ D33 pilot (= 2026-06-29 月)
- ☐ D34 demote sim 候補 (= 9130 5 連続 warning)
- ☐ D35 demote 本実行候補
- ☐ W5 集約 wave (= D20-D33 後)
- ☐ F111 runner-level 正式実装 wave

## §17 safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara

## §18 review 状態

- status: **filled (= Fujiwara 提供範囲、OHLCV pending)**
- Fujiwara 提供完了範囲: skip 判定 / 見送り理由 / entry_success / no-new-chase 既定 / watch 降格既定
- pending 範囲: OHLCV 数値 / signal_success / 追わない上限超過判定 / 6 条件充足判定 / monitor 数値
- pending 上書き path: Fujiwara 観測 input または J-Quants auto-fill

### 3 DB md5 / F282 plist 明示記録 (= 本 wave 不変確認)

- production md5: b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓
- develop md5:    0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓
- staging md5:    6cb3885cd9c90bd6fdabb127c1cd0d17 → 不変 ✓
- F282 plist (weekly-snapshot): size=1772 / mtime=1778593597 → 不変 ✓

### 本 wave write 範囲 + 禁止操作 0 一覧

本 wave write:
- vault doc 1 file (= 本 D32 review.md、Fujiwara 提供範囲で fill 化、OHLCV は推測せず placeholder 維持) のみ

禁止操作 0 (= 全 0):
- DB write 0 / production / develop DB 接続 0 / staging 未使用
- API 0 / token 0 / env secret 参照 0
- LINE 送信 0
- launchctl 実行 0 / plist 配置 0 / cron 変更 0
- workflow 変更 0
- git commit 0 / git push 0 / --no-verify 0
- sudo 0 / rm -rf 0
- 自動発注 0 / 楽天 0 / iSPEED 自動操作 0 / Computer Use 0
- 推測禁止 (= OHLCV 等数値は Fujiwara input required または J-Quants auto-fill 待ち)
