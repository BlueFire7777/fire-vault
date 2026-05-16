---
template: manual-live-pilot-review
version: 1.0 (= D31 朝時点の advisory に対する review、post-D31 framework と分離)
date: 2026-06-25
owner: BlueFire7777 (Fujiwara)
pilot_day: D31
status: filled (= Fujiwara 提供データ + 正式見送り理由で記入済)
correction_note: |
  本 review は D31 朝時点で実際に推奨された 3 候補 (= 9130 / 331A0 / 4389)
  に限定。9247 / 9628 / 4404 は **post-D31 後続改善ロジック (= v1.1 → v1.4.1)
  で導入された改善候補** であり、D31 実績評価対象ではない。混同を防ぐ。
linked_advisory_d31_actual: 04_daily/2026-06-25_morning_advisory_summary.md (= D31 朝の v1.0 advisory、実際の推奨銘柄を含む)
linked_advisory_post_d31_improvements:
  - 04_daily/2026-06-25_morning_advisory_summary_v1.1.md (= post-D31 liquidity gate hotfix)
  - 04_daily/2026-06-25_morning_advisory_summary_v1.2.md (= post-D31 risk_yen 非加点)
  - 04_daily/2026-06-25_morning_advisory_summary_v1.3.md (= post-D31 33 sector + 6 軸加重)
  - 04_daily/2026-06-25_morning_advisory_summary_v1.4.md (= post-D31 pre/post-open 分離)
  - 04_daily/2026-06-25_morning_advisory_summary_v1.4.1.md (= post-D31 score/priority 完全分離)
entry_status: skip
actual_entry: none
final_judgment: "初回実弾投入日 / 出来高薄・スプレッド大・約定撤退リスク懸念で全候補 skip。Fujiwara 判断は運用上妥当。"
post_d31_framework_note: "v1.1 - v1.4.1 framework は D31 振り返り後の改善設計、D31 reality の評価には使わない"
pilot_judgment: GO_CONDITIONAL
post_recovery: 12 連続 maintained
related:
  - 04_daily/2026-06-25_manual_live_pilot_trade_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_results.md
---

# Manual Live Pilot — Trade Review (2026-06-25 / D31)

> ⚠ **本 review は D31 朝時点の v1.0 advisory に対する評価。**
> 推奨銘柄: **9130 / 331A0 / 4389** の 3 件のみ。
> 9247 / 9628 / 4404 は post-D31 改善 wave で導入された候補で **D31 実績対象外**。

## §0 Fujiwara verbatim confirmation (= 本人発言の verbatim 保存)

> **対象銘柄 (= D31 reality 3 候補):**
> - 9130 共栄タンカー
> - 331A0 メディックス
> - 4389 プロパティデータバンク
>
> **判断**: 全候補見送り
>
> **見送り理由 (= verbatim):**
> 「初回実弾投入日であり、出来高が薄く、スプレッドも大きく見えたため、
> 約定リスク・撤退リスクが心配だった。」
>
> **銘柄別 (= verbatim):**
> - 9130: 「見送り。シグナルは良かったが、追わない上限超過後は追わない。」
> - 331A0: 「見送り。流動性が薄く、entry 候補として不安。」
> - 4389: 「見送り。上昇はしたが、流動性・スプレッド面で不安。」
>
> **全体所感 (= verbatim):**
> 「初回は無理に入らず、運用確認とロジック修正を優先した。」

→ 本 §0 は Fujiwara 本人の verbatim 入力を保存する block (= 後続 wave / framework
改善作業の検証起点). §1-§13 は本 verbatim を解釈・整理した内容で、本 §0 と整合.
本 §0 verbatim は **上書き禁止** (= 後続 wave で paraphrase してはならない).

## §1 [必須 1] 入った / 見送った

**skip** (= 全 3 候補で actual_entry: none)

## §2 [必須 2] 判断理由 (= Fujiwara 本人の正式見送り理由、§0 verbatim 出典)

> 「初回実弾投入日であり、出来高が薄く、スプレッドも大きく見えたため、
> 約定リスク・撤退リスクが心配だった。」
>
> 全体所感: 「初回は無理に入らず、運用確認とロジック修正を優先した。」

要点 (= §0 verbatim を解釈):
- 初回実弾日であり、慎重判断
- 出来高 / スプレッド観点で約定リスク懸念
- 撤退時のスリッページ懸念
- 全候補 skip = 運用上妥当な保守判断
- 全体方針: 無理 entry より運用確認 + ロジック改善優先

## §3 [必須 3] 実際に見た銘柄 (= D31 朝 advisory 推奨 3 件のみ)

1. **9130 共栄タンカー** (= 海運業 / タンカー)
2. **331A0 メディックス** (= 情報通信、excluded)
3. **4389 プロパティデータバンク** (= 情報通信、watch_signal_success)

注: 9247 / 9628 / 4404 は post-D31 改善 wave で導入された候補。D31 朝の
   advisory には含まれず、本 review の評価対象ではない。

## §4 [必須 4] 板・出来高・spread + 前場/終日実績

| code | close | change | high | volume | 流動性所感 | result |
|---|---|---|---|---|---|---|
| 9130 | 1,479 | **+4.89%** | 1,497 | 55,200 株 | 取引可能水準だが、初回 entry には慎重判断 | signal_success |
| 331A0 | 481 | -0.21% | (flat) | 18,600 株 | 低流動性、small-cap、初回 entry には不適 | excluded_candidate_validated |
| 4389 | 890 | +3.13% | (未記録) | 18,700 株 | 出来高 / spread 懸念、watch 扱いが妥当 | watch_signal_success |

### 流動性所感 (= Fujiwara 観測)
- 9130 (= 55,200 株、貸借) は取引可能水準ながら、初回実弾日として慎重判断
- 331A0 (= 18,600 株) と 4389 (= 18,700 株) は出来高薄、small-cap で約定 / 撤退リスク高い
- 共通: スプレッド大きく見えた、entry/exit でスリッページ懸念

## §5 [必須 5] 結果メモ

- **Fujiwara skipped all 3 candidates** (= D31 操作 = 全 skip)
- 銘柄選定の signal 自体は positive (= 9130 +4.89%、4389 +3.13%)
- ただし「signal が正しい」と「初回実弾日に実 entry できる」は別事象
- 出来高 / spread / 約定リスク懸念で見送りは保守的判断、運用上妥当
- 9130 は朝 1,395-1,425 円で entry していれば利確目安 1,450-1,470 円到達 (= シグナル success)、ただし当日未 entry
- D31 経験を踏まえ、post-D31 改善 wave で liquidity gate / risk_yen 順位非加点 等の policy 整備へ

## §6 [必須 6] 実 entry か skip か (= D31 actual)

| code | 判定 |
|---|---|
| 9130 | **skip** (= 朝 entry なし、後場 +4.89% で追わない上限超過、新規追い不可) |
| 331A0 | **skip** (= 流動性懸念、entry 候補としては不適) |
| 4389 | **skip** (= 出来高 / spread 懸念、entry 不可) |

### 計画 vs 実際

| 項目 | planned (= D31 advisory) | actual |
|---|---|---|
| entry 銘柄 | 9130 (= 推奨候補 #1) | none |
| entry 価格 | 1,395-1,425 円付近 | (= 未約定) |
| entry 時刻 | 09:00 想定 | (= 未約定) |
| entry 株数 | 100 株 想定 | (= 未約定) |
| 総 PnL | (= 想定なし) | 0 円 (= 未約定) |

## §7 [必須 7] 入った価格 or 見送った理由

| code | 価格 / 見送り理由 |
|---|---|
| 9130 | 見送り (= 初回実弾日 / 出来高・spread 懸念 / 約定撤退リスク) |
| 331A0 | 見送り (= 低流動性 18,600 株、small-cap、entry 不適) |
| 4389 | 見送り (= 出来高 18,700 株、spread 懸念、watch 扱い妥当) |

## §8 [必須 8] 追わない上限を超えていたか

| code | 推奨買い上限 (= D31 advisory) | 高値 | 超過 | 超過率 |
|---|---|---|---|---|
| 9130 | 1,440 円 | **1,497 円** | **Yes** ★ | **+4.03% 超過** |
| 331A0 | (= D31 advisory 想定上限) | flat | No | n/a |
| 4389 | (= D31 advisory 想定上限) | (未記録) | (= データ不足) | (= データ不足) |

→ 9130 は明確に追わない上限超過、9130 を **未 entry なら新規追い skip 確定** logic 妥当

## §9 [必須 9] pre-open 判断か post-open 判断か

| code | 判断 phase |
|---|---|
| 9130 | **pre-open + post-open** (= 朝で慎重 + 後場 +4.89% で追わない確認) |
| 331A0 | **pre-open** (= 朝の流動性確認で見送り判断) |
| 4389 | **pre-open** (= 朝の流動性確認で見送り判断) |

## §10 [必須 10] signal_success と entry_success の区別

| code | signal_success | entry_success | comment |
|---|---|---|---|
| **9130** | **YES** ✓ | **NO** (= skip) | 銘柄選定 logic 妥当 (= +4.89%)、ただし初回実弾日の慎重判断 + 追わない上限超過で新規追い不可 |
| **331A0** | **NA / excluded_validated** | **NA** (= skip) | 低流動性で entry 候補として不適、excluded gate 妥当性検証された |
| **4389** | **YES** (= watch_signal_success) | **NA** (= skip) | signal positive (+3.13%) だが出来高 / spread 懸念で watch 扱い妥当 |

→ 「銘柄選定が正しかった (= signal)」と「実際に入れた (= entry)」は別事象、両軸で記録

---

## §11 D31 実績の学び (= 後続改善 wave の起点)

### 11.1 9130 で validated

- ✓ 選定 signal は妥当 (= +4.89% で利確目安到達するレベル)
- ✓ ただし初回実弾日の保守判断 + 追わない上限超過で実 entry 不可
- → 「signal positive と実 entry success は別」の認識を強化

### 11.2 331A0 で validated

- ✓ 低流動性 (= 18,600 株) で result flat
- ✓ entry 不適性が実績で確認された
- → post-D31 で **liquidity gate** を強化する根拠

### 11.3 4389 で validated (= watch trade-off)

- ✓ signal positive (+3.13%) だが流動性 (= 18,700 株) で entry 不適
- ⚠ liquidity gate のトレードオフ: signal 取り逃がしの reverse-cost 存在
- → Fujiwara 判断: "Liquidity gate remains necessary" = trade-off 容認

### 11.4 Final judgment (= Fujiwara 提供)

- Fujiwara skipped all entries (= 全 候補 skip)
- 見送り理由: 初回実弾投入日 / 出来高薄 / スプレッド大 / 約定撤退リスク懸念
- Operational decision was valid (= 保守判断、運用上妥当)
- Selection signal was positive for 9130 and 4389
- Liquidity gate remains necessary
- Need to distinguish signal_success from entry_success

---

## §12 post-D31 後続改善ロジック (= D31 実績対象ではない、参考)

D31 振り返り後、以下の改善 wave で 9247 / 9628 / 4404 等の候補が導入された:

| version | 主要改善 | 新候補 |
|---|---|---|
| v1.1 | liquidity gate hotfix (= 信用 + 非 TOPIX 除外) | 9247 / 9628 / 9130 (= 貸借 のみ) |
| v1.2 | risk_yen 非加点 + entry/watch/excluded 三分類 | 9247 が #1 promote |
| v1.3 | 33 sector + 6 軸加重 | 4404 食品 sector 加える |
| v1.4 | pre-open / post-open phase 分離 | 9130 post-open uplift |
| v1.4.1 | pre_open_score / entry_priority 完全分離 + 100 株標準 | 4404 watch 降格 |

→ これらは **D31 実績の評価には使わない**。D31 reality は 9130 / 331A0 / 4389 のみ。
→ D32 以降の advisory に v1.4.1 framework を適用、D32 review で validation。

---

## §13 7991 demote 効果評価 (= 別軸、D31 時点)

- D31 でも 7991 caution 維持 (= rank 7、top 除外): **Yes** ✓
- W3 §3.3 #2-exclusion (= demote 済み連続カウント停止) 機能維持: **Yes** ✓

## §14 9130 rank 1 連続性 monitor (= 次の demote 候補、D31 時点)

- 9130 rank 1 連続 = **2** (D30-D31)
- 段階 1 warning 予想: D34 (= 月、5 連続到達想定)
- 段階 3 demote 本実行予想: D35 (= 火)

## §15 W5 集約 (= D20-D33 14 day) 引き継ぎ memo

- D31 reality: 9130 signal Y / entry N、331A0 excluded validated、4389 watch_signal_success
- Fujiwara 全 skip = 運用上妥当
- post-D31 改善 wave で liquidity gate / risk_yen 非加点 / 100 株標準 等を整備
- D34 / D35 で 9130 demote sim → 本実行想定

## §16 Next action

- ✓ **D31 review filled** (= 本 file、Fujiwara 見送り理由反映 + 9247/9628/4404 混同解消)
- ☐ paper PnL handoff (= 場後、`--review-md` 付きで再 run、本 review filled 参照)
- ☐ D32 pilot wave (= 2026-06-26 金、v1.4.1 framework 適用、entry 9247/9628、watch 4404、no-new-chase 9130)
- ☐ D33 / D34 (= 9130 demote sim 候補) / D35 (= demote 本実行)
- ☐ W5 集約 wave

## §17 safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara

## §18 review 完了印 (= 補正後)

filled by Fujiwara via Claude Code review 受領、10 項目構造で記入。
本 wave (= D31 review correction) で:
- 9247 / 9628 / 4404 混入を解消 (= D31 実績対象外として明示)
- Fujiwara 見送り正式理由 を明記
- post-D31 改善 wave references を §12 で分離整理
- 推測禁止遵守 (= actual data は Fujiwara 提供分のみ)

### 3 DB md5 / F282 plist 明示記録 (= 本 wave 不変確認)

- production md5: b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓
- develop md5:    0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓
- staging md5:    6cb3885cd9c90bd6fdabb127c1cd0d17 → 不変 ✓
- F282 plist (weekly-snapshot): size=1772 / mtime=1778593597 → 不変 ✓

### 本 wave write 範囲 + 禁止操作 0 一覧

本 wave write:
- vault doc 1 file (= 本 D31 review.md 補正書き換え) のみ

禁止操作 0 (= 全 0):
- DB write 0 / production / develop DB 接続 0 / staging read-only のみ (= 未使用)
- API 0 / token 0 / env secret 参照 0
- LINE 送信 0
- launchctl 実行 0 / plist 配置 0 / cron 変更 0
- workflow 変更 0
- git commit 0 / git push 0 / --no-verify 0
- sudo 0 / rm -rf 0
- auto-order 0 / 楽天 0 / iSPEED 自動操作 0 / Computer Use 0
- 推測禁止 (= actual data は Fujiwara 提供分のみ反映)
