---
title: F281 Lane C Phase C2 実装計画 v1.0 (HQ 補正 3 点反映)
date: 2026-05-09
phase: F281-Phase-C2 (Lane C 真実装、c6 backfill 完了 + Phase C2 PASS 受領後)
status: 実装計画 v1.0、HQ 条件付き承認 (補正 3 点反映)、C2-0 (本ドキュメント) Vault 化で着手準備完了
related: F281_Lane_C_design_2026-05-08, F281_Lane_C_universe_precheck_2026-05-08, F284_F105_c6_final_result_2026-05-09, F105_J-Quants_intraday_validation_2026-05-08
trigger: HQ Phase C2 GO 判断 (2026-05-09、c6 完了 + Phase C2 PASS 5 条件達成 + 補正 3 点指示)
---

# F281 Lane C Phase C2 実装計画 v1.0

★ HQ 条件付き承認 (補正 3 点) を反映した Phase C2 実装計画 v1.0。
   c6 full backfill 完了 (commit 439e2cf) + Phase C2 PASS 5 条件達成
   後の F281 Lane C 真実装フェーズ。

★ HQ 補正 3 点 (v1.0 反映済):
   1. **高値再ブレイク判定**: current bar を含む intraday_high_t では
      なく **prior_peak_high (= [09:00, t-1] の max(high))** を使う。
      Lane C setup を **状態遷移** で定義:
      peak 形成 → pullback 成立 → VWAP 回復 or prior high 再ブレイク → entry
   2. **smoke test 範囲**: Tier2 内 **3 銘柄 × 5 営業日 × preset B**
      (1 銘柄 × 3 営業日から拡張)、利益ではなく candidate 抽出 / setup
      判定 / entry/exit / skipped_reason / force_close / 72030 除外を確認
   3. **TP/SL preset**: A 0.8%/0.6% / B 1.2%/0.8% / C 1.5%/1.0% /
      D 2.0%/1.0% / E ATR 連動 (実装重ければ後回し可、A-D 優先)

## 1. 概観 / 戦略仮説

Lane C は **前日強銘柄の当日初押し再加速** を狙う intraday day-trade
strategy。F281 設計 (2026-05-08) に基づく真実装。持ち越し禁止、当日中 exit。

```
[前日 t-1 close]
  universe pass (Tier2、prev_change/volume/turnover、信用可)
        ↓
[当日 t 09:00 〜 t]
  intraday data (1min OHLC + vwap_cumulative、c6 backfill 済)
        ↓
  状態遷移 (HQ 補正 1):
    State 1: peak 形成 (intraday high 到達)
    State 2: pullback 成立 (peak から PULLBACK_THRESHOLD 以上の押し目)
    State 3-A: VWAP 回復 (close >= vwap × 1.001) → entry 候補
    State 3-B: prior high 再ブレイク (close > prior_peak_high × 1.001)
               → entry 候補
        ↓
[entry 条件 4 つ AND]
  universe + setup + 過熱でない + 14:45 cutoff
        ↓
[exit 条件、当日中決済必須]
  TP / SL / 15:10 force_close
        ↓
[strict 評価]
  preset A-D × 478 codes × 60 営業日 = 114,720 評価
```

## 2. Lane C intraday setup 定義 (状態遷移、HQ 補正 1)

### 2.1 状態遷移図

```
[State 0: 監視開始]
   t=09:00、universe pass の銘柄を監視開始
   intraday_high_t = high_t (初期値)
        ↓
[State 1: peak 形成]
   intraday_high_t が更新される (新高値到達)
   peak_high_t を記録
        ↓
[State 2: pullback 成立]
   pullback_pct_t = (peak_high_t - close_t) / peak_high_t
   pullback_pct_t >= PULLBACK_THRESHOLD (0.5%) で State 2 入り
   peak_high_t を peak として fix (= 今後の prior_peak_high 候補)
        ↓
[State 3-A: VWAP 回復]
   close_t >= vwap_cumulative_t × (1 + EPS_VWAP=0.001) で成立
        OR
[State 3-B: prior high 再ブレイク]
   prior_peak_high = max(high) over [09:00, t-1] (t-1 までの max、
                     current bar 除く)
   close_t > prior_peak_high × (1 + EPS_BREAK=0.001) で成立
        ↓
[entry 候補]
   過熱でない + 時刻 < 14:45 + 出来高継続 OK で entry 確定
```

### 2.2 状態遷移の Python 擬似コード

```python
state = 0
peak_high = None
prior_peak_high = None  # State 1 で fix した peak の高値

for t, bar in intraday_minute_bars(code, date):
    # t-1 までの max(high) を計算 (current bar 除く)
    bars_until_prev = bars_history[: index_of(t)]
    prior_peak_high = max(b.high for b in bars_until_prev) if bars_until_prev else None

    if state == 0:
        # 監視開始、最初の bar で peak_high 初期化
        peak_high = bar.high
        state = 1
    elif state == 1:
        # peak 更新
        if bar.high > peak_high:
            peak_high = bar.high
        # pullback 成立チェック
        pullback_pct = (peak_high - bar.close) / peak_high
        if pullback_pct >= PULLBACK_THRESHOLD:
            state = 2
    elif state == 2:
        # VWAP 回復 OR prior high 再ブレイク
        vwap_recovered = bar.close >= bar.vwap_cumulative * (1 + EPS_VWAP)
        prior_break = (
            prior_peak_high is not None
            and bar.close > prior_peak_high * (1 + EPS_BREAK)
        )
        if vwap_recovered or prior_break:
            # entry 候補、過熱 / 時刻 / 出来高ガードを別途確認
            return EntryCandidate(t, bar, recovery="vwap" if vwap_recovered else "break")
        # peak 更新で State 1 に戻る (新たな peak 形成)
        if bar.high > peak_high:
            peak_high = bar.high
            state = 1
```

★ State 0 → 1 → 2 → entry の遷移、または State 2 で新高値で State 1
  に戻る (pullback 失敗時)。

## 3. 初押し判定 (State 1 → State 2 移行)

```
peak_high_t = max(high) over [09:00, t]
pullback_pct_t = (peak_high_t - close_t) / peak_high_t
初押し成立: pullback_pct_t >= PULLBACK_THRESHOLD
```

`PULLBACK_THRESHOLD` 初期値: **0.5%** (0.005)
- preset で 0.5% / 1.0% / 1.5% を比較可能 (Phase C2 strict 評価で
  calibration、初期は 0.5% 固定)

## 4. VWAP 回復判定 (State 3-A)

```
vwap_t = market_prices_intraday.vwap_cumulative
          (1 min interval、当日 09:00 から累積)
回復成立: close_t >= vwap_t × (1 + EPS_VWAP)
```

`EPS_VWAP` = **0.001** (= 0.1% 上回り、ノイズ除去)

★ HQ 必須: TPM を VWAP 代替にしない。`vwap_cumulative` (c6 で
  全 universe 完備、4 条件 PASS 確認済) を直接使用。

## 5. 高値再ブレイク判定 (State 3-B、HQ 補正 1)

```
prior_peak_high = max(high) over [09:00, t-1]
                  (= current bar を含まない、過去の最高値)
再ブレイク成立: close_t > prior_peak_high × (1 + EPS_BREAK)
```

`EPS_BREAK` = **0.001** (= 0.1% 抜き)

★ HQ 補正 1: current bar を含む `intraday_high_t` ではなく、
  `prior_peak_high` を使う。これにより「現在足の自己 peak」と「過去の
  peak からのブレイク」を区別。

実装注意:
- t=09:00 (1 件目の bar) では `prior_peak_high` 不在 → State 3-B 不成立
- prior_peak_high は state machine の State 2 で fix した peak の値
  (= 直近の peak 形成タイミングの high)、または bar history 全体の
  max(high) のいずれを採用するかは設計判断:
  - **採用案 A (推奨)**: bar history 全体 [09:00, t-1] の max(high)
    (= rolling max、現在足を除く)
  - 採用案 B: 直近 peak 形成タイミングの high (= state machine 内 fix)
- 本実装は採用案 A (rolling max) を初期実装、Phase C2 strict 評価で
  比較検討。

## 6. entry 条件 (4 つの AND)

```
(i)   universe pass:        code IN Tier2 478 codes (smoke 72030 除外)
(ii)  setup 充足:           State 3-A (VWAP 回復) OR State 3-B (prior 再ブレイク)
(iii) 過熱でない:            (open_t - open_0900) / open_0900 < HEAT_THRESHOLD
(iv)  時刻ガード:            t < 14:45 (R-05-10/11、F133 既存と整合)
```

加えて出来高継続 (補助):
```
cum_volume_t / SMA20(daily_volume) >= VOLUME_THRESHOLD (0.7)
```

`HEAT_THRESHOLD` = **0.10** (= +10% 以上の過熱は乗り遅れリスクで除外)
`VOLUME_THRESHOLD` = **0.7** (= 当日累積出来高が SMA20 の 70% 以上)

## 7. exit 条件 (持ち越し禁止)

以下のいずれかで **当日中** に決済:

```
TP 到達:        close_t >= entry_price × (1 + TP_PCT)
SL 到達:        close_t <= entry_price × (1 - SL_PCT)
時刻 force_close: t >= 15:10 (F133 既存 force_close と整合)
```

★ **翌営業日への持ち越し禁止** (F281 §1 / R-19-XX 系の前提を厳守)
★ VIRTUAL_TP / SL は F054 既存ロジックを継承
★ TP/SL preset は §9 で定義、preset 別に評価

## 8. skipped_reason 一覧

candidate を skip した理由を strict 評価で集計:

| reason | 内容 |
|---|---|
| `universe_out` | Tier2 478 外 (smoke 72030 等) |
| `liquidity_low` | 出来高 / 売買代金不足 (universe precheck 既) |
| `overheated` | open 比 +10% 以上の過熱 |
| `time_cutoff` | 14:45 以降 entry 試行 |
| `no_pullback` | 初押し未成立 (State 1 で終わる) |
| `vwap_below` | VWAP 回復未達 (State 2 で State 3-A 不成立) |
| `no_breakout` | prior high 再ブレイク未達 (State 2 で State 3-B 不成立) |
| `volume_low` | 出来高継続 0.7 未達 |
| `no_data` | market_prices_intraday に data なし (= no_data 銘柄) |
| `guard_loss` | F132 日次 / 週次累計損失到達 |
| `guard_force_close` | F133 時刻 force_close で entry 不可 |
| `execution_gate` | F140 板厚 / スプレッド / 値飛び等 |
| `peak_not_formed` | t=09:00 直後で peak 未形成 (State 0) |

各理由の比率を strict 評価レポートに記録。

## 9. TP/SL preset 比較 (HQ 補正 3)

| preset | TP | SL | reward/risk | 想定特性 |
|---|---|---|---|---|
| **A** | +0.8% | -0.6% | 1.33 | 小利確、高頻度 |
| **B** | +1.2% | -0.8% | 1.50 | 標準、バランス重視 |
| **C** | +1.5% | -1.0% | 1.50 | 中庸利伸ばし |
| **D** | +2.0% | -1.0% | 2.00 | trend follow |
| **E** | 1.5×ATR | 1.0×ATR | 1.50 | volatility 適応 (実装重ければ後回し可) |

★ HQ 補正 3: A〜D を優先評価、E は実装重ければ後回し可。本計画では
  **A〜D を優先** で実装、E は C2-5 strict evaluator で interface のみ
  整え、実装は別タスクで切り出し可能とする。

## 10. strict 評価接続方針

既存 B-strict 評価基盤 (`simulation/paper_live/`、F230 batch_replay
等) に **Lane C 専用 evaluator** を追加。

- 入力: `market_prices_intraday` (Tier2 478 codes filter、72030 除外)
- 評価期間: 60 営業日 (= c6 backfill 済 data、2026-02-03 〜 2026-05-01、
  休場日 4 日除く)
- 比較: **TP/SL preset A〜D** × universe 478 codes × 60 営業日
  = 478 × 60 × 4 = **114,720 評価** (E は後回しで除外)
- 出力: candidate count / entry count / hit count / TP/SL/force /
        実現損益 / max drawdown / 勝率 / 期待値 / skipped_reason 集計

既存 strict 評価との分離:
- Lane C 結果は別 namespace (`strict_lane_c/`) で保存
- 既存 Lane A/B 結果に影響しない

## 11. 追加 / 変更ファイル

### 追加 (~/fire 側)

```
agents/lane_c_extraction.py            (Lane C 候補抽出 + universe filter)
simulation/lane_c/__init__.py
simulation/lane_c/setup_judge.py       (state machine、初押し/VWAP/prior 再)
simulation/lane_c/runner.py            (preset 切替 + entry/exit simulator)
simulation/lane_c/strict_eval.py       (60 営業日 + preset A-D evaluator)
simulation/lane_c/universe_loader.py   (Tier2 478 + 72030 除外)
tests/agents/test_lane_c_extraction.py
tests/simulation/test_lane_c_setup_judge.py     (state machine test)
tests/simulation/test_lane_c_runner.py
tests/simulation/test_lane_c_strict_eval.py
tests/simulation/test_lane_c_universe_loader.py (72030 除外 test 必須)
scripts/jobs/run_lane_c_strict_eval.py (CLI、staging 専用 4 段 guard)
scripts/jobs/run_lane_c_smoke.py       (smoke CLI、3 codes × 5 days × B)
```

### 変更 (オプション、C2-8 で後回し)

```
simulation/paper_live/tick.py          (Lane C extraction 呼出追加)
notifications/templates/order.py       (Lane C 注文テンプレ追加)
```

★ HQ 重要: tick.py / order template 統合は **strict 評価が基準を満た
した後** で OK。まずはオフライン strict 評価で期待値を確認 (= C2-7 まで
完了後に C2-8 着手判断)。

## 12. migration 要否

★ **既存 schema で完結、新 migration 不要** ★

- `market_prices_intraday` (1/5/15 分 + vwap) は c6 で完備
- Lane C 結果は既存 `simulation_runs` / `orders` / `fills` テーブルを使用
- skipped_reason は既存 reason field で表現可

## 13. smoke test 計画 (HQ 補正 2)

### 13.1 範囲 (補正 2 反映)

| 項目 | 値 |
|---|---|
| 銘柄数 | **Tier2 内 3 銘柄** (例: 67580 Sony / 80350 東京エレクトロン / 84110 三菱 UFJ FG、流動性高) |
| 営業日 | **5 営業日** (例: 2026-04-21 〜 2026-04-25) |
| preset | **B 標準** (TP +1.2% / SL -0.8%) |
| 環境 | FIRE_ENV=staging、staging DB 専用 4 段 guard |

### 13.2 確認項目 (補正 2、利益ではなく動作確認)

| # | 検証項目 |
|---|---|
| 1 | candidate 抽出: extract_lane_c_candidates が universe 内の銘柄を返す |
| 2 | setup 判定: state machine が State 0→1→2→3-A/B を正しく遷移 |
| 3 | entry/exit: runner が entry → exit を生成、TP/SL/force_close で正確に決済 |
| 4 | skipped_reason: 各 reason が記録される (no_pullback / vwap_below / overheated 等) |
| 5 | force_close: 15:10 到達で当日 exit、持ち越し禁止が動作 |
| 6 | 72030 除外: smoke 残 72030 が universe filter で除外される |

★ HQ 補正 2: smoke では「利益が出るか」ではなく「動作が正しいか」を
  確認。期待値や勝率の評価は strict 60 営業日評価 (C2-7) で実施。

### 13.3 受入

- 全 6 検証項目が確認できる
- candidate 抽出 0 件は許容 (= setup 不成立で skip 多発でも OK、機構が動作)
- 致命的 error 0 件
- staging DB のみ、production / develop DB 無触

## 14. strict 60 営業日評価計画

### 14.1 範囲

| 項目 | 値 |
|---|---|
| 銘柄 | Tier2 478 codes (smoke 72030 除外) |
| 営業日 | 60 (= 2026-02-03 〜 2026-05-01、休場日 4 日除く) |
| preset | A / B / C / D (E は後回し可、HQ 補正 3) |
| 評価数 | 478 × 60 × 4 = **114,720 (code, date, preset)** |

### 14.2 出力指標

per (preset):
- candidate count (= setup 充足候補数)
- entry count (= 4 条件 AND 通過 entry 数)
- exit 内訳: TP 到達 / SL 到達 / force_close 内訳
- 実現損益 (合計、平均、中央値)
- 勝率 (winrate)
- 期待値 (= 平均損益)
- max drawdown
- reward/risk 実績
- skipped_reason 件数 (各 reason の比率)

per (code):
- code 別 candidate / entry / 勝率 / 損益

per (date):
- 日次 candidate / entry / 損益 / drawdown

### 14.3 受入基準

- 全 4 preset で評価完了、出力レポート Vault 化
- candidate 抽出が non-zero (= 機構動作)
- skipped_reason が網羅的に集計される
- F281 R-19-08 (Live Advisory 移行前必須 20 営業日条件) を満たす data 量
- 評価期間に致命的 error 0 件
- production / develop DB 無触
- 最良 preset 候補 1 つを Fujiwara レビューに提出

評価所要時間: 推定 1-2h (preset 4 種 × 478 codes × 60 営業日)

## 15. commit 分割案 (HQ 補正 3 反映)

全 Phase C2 実装は ~/fire 側、各 commit に **Codex pre-commit 必須**、
**--no-verify 禁止**、**個別 commit 厳守**。

| # | commit | 内容 |
|---|---|---|
| **C2-0** | `docs(F281-C2): Phase C2 implementation plan Vault化` | 本ドキュメント (fire-vault) |
| **C2-1** | `feat(F281-C2): Tier2 universe loader + 72030 exclusion test` | universe_loader.py + test (72030 除外 test 必須) |
| **C2-2** | `feat(F281-C2): Lane C setup_judge state machine` | setup_judge.py + state machine test |
| **C2-3** | `feat(F281-C2): Lane C candidate extraction + skipped_reason` | extraction.py + skipped_reason 集計 test |
| **C2-4** | `feat(F281-C2): Lane C intraday entry/exit simulator` | runner.py + entry/exit + TP/SL/force test |
| **C2-5** | `feat(F281-C2): strict 60営業日 evaluator + preset comparison` | strict_eval.py + preset A-D 比較 test |
| **C2-6** | `chore(F281-C2): smoke test + result Vault化` | smoke 実行 + smoke 結果 Vault |
| **C2-7** | `chore(F281-C2): strict 60営業日評価 + result Vault化` | strict 60 実行 + 評価結果 Vault |
| **C2-8** | `optional feat(F281-C2): paper_live/tick/order template統合` | tick.py + order.py、strict 評価 PASS 後の判断 |

★ HQ 重要: **C2-8 は strict 評価 PASS 後の判断で OK**。まず C2-0 〜 C2-7
  でオフライン strict 評価を完了し、期待値が確認できてから C2-8 着手。

### Codex review 重点領域 (CLAUDE.md 既)

- 非同期処理 (なし、Lane C はオフライン評価が主体)
- SQLite (read-only、market_prices_intraday filter で問題なし)
- pydantic (skipped_reason 等の構造化出力で活用)
- 例外処理 (state machine の各遷移で適切)
- 新規 logic は pytest 必須

## 16. Phase C2 で守ること (HQ 指示厳守、最終確認)

  ✅ daily 近似禁止 (intraday data 前提)
  ✅ TPM を VWAP 代替にしない (vwap_cumulative 使用)
  ✅ 持ち越し禁止 (当日 exit 強制、F133 force_close 整合)
  ✅ Phase C2 universe = Tier2 478 codes 固定
  ✅ market_prices_intraday 全体を無条件参照しない (universe filter 必須)
  ✅ smoke 残 72030 を評価対象から除外 (universe_loader で除外 test 必須)
  ✅ production / develop DB 無触 (FIRE_ENV=staging guard、4 段)
  ✅ 自動発注禁止 / 楽天証券操作自動化禁止 / Computer Use 禁止
  ✅ LINE 通知は注文完成形提示まで (Fujiwara 手動発注、C2-8 後回し)
  ✅ Codex pre-commit 必須 / --no-verify 禁止 / 個別 commit 厳守

## 17. リスクと制約

### リスク

1. **state machine の境界条件**: t=09:00 の peak 未形成、t-1 不在等の
   edge case で setup が誤判定する可能性 → 単体 test で網羅
2. **prior_peak_high の rolling 計算コスト**: 1 銘柄あたり 327 bars/day、
   478 codes × 60 days × 327 bars = 約 938 万 bars の計算負荷 → 各
   bar の rolling max は O(N) で十分高速
3. **setup 不成立による entry 0 件**: PULLBACK_THRESHOLD / EPS_VWAP /
   EPS_BREAK の閾値次第で entry が出ない可能性 → smoke で動作確認、
   strict で閾値感度分析
4. **TP/SL preset の最適 preset 選択**: 4 preset で勝率 / 期待値が
   割れる可能性 → strict 結果から Fujiwara レビューで最良選定
5. **smoke 走行で 72030 が混入**: universe_loader の test 漏れで混入
   する可能性 → C2-1 で 72030 除外 test を必須化

### 制約

1. **オフライン strict 評価が先**: tick.py / order template 統合は
   C2-8 で strict PASS 後 (HQ 重要指示)
2. **paper_live / Live Advisory への接続は別タスク**: Phase C2 では
   strict 評価まで、Live 接続は Phase C3 以降
3. **R-03-01 / R-13-08 厳守**: 自動発注禁止、Fujiwara 最終判断
4. **Stage 飛ばし禁止**: Phase C2 strict PASS まで Live 接続しない

## 18. 関連 Vault ファイル

- [[F281_Lane_C_design_2026-05-08|F281 Lane C 設計]] (本実装の上位仕様)
- [[F281_Lane_C_universe_precheck_2026-05-08|F281 universe precheck]]
  (Tier2 478 codes リスト)
- [[F284_F105_c6_final_result_2026-05-09|c6 final result]] (本実装の前提)
- [[F105_J-Quants_intraday_validation_2026-05-08|F105 J-Quants 検証]]
- [[F105_Phase_C1_smoke_2026-05-08|F105 Phase C1 smoke]]
- [[F285_Research_Lane_requirements_and_spec_2026-05-08|F285 Research Lane]]
  (中長期 Lane、Phase C2 と並行検討候補)

## 19. 改訂履歴

- v1.0 (2026-05-09): 初版、c6 完了 + Phase C2 PASS 受領後の Vault 化、
  HQ 補正 3 点反映:
  - 補正 1: 高値再ブレイク判定を `prior_peak_high (= t-1 までの max)`
    に修正、Lane C setup を **状態遷移** で定義
    (peak 形成 → pullback → VWAP 回復 or prior 再ブレイク → entry)
  - 補正 2: smoke を **Tier2 内 3 銘柄 × 5 営業日 × preset B** に変更、
    candidate / setup / entry/exit / skipped_reason / force_close /
    72030 除外を確認
  - 補正 3: TP/SL preset を A 0.8/0.6 / B 1.2/0.8 / C 1.5/1.0 /
    D 2.0/1.0 / E ATR 1.5/1.0 (E 後回し可) に変更、commit 分割を
    C2-0 〜 C2-8 に変更 (C2-8 = tick/order 統合は strict PASS 後)
