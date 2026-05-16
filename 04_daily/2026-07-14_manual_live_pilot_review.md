---
id: FIRE-pilot-D44-review-2026-07-14
phase: 本番 v0 / W60-pilot-D44 / v1.4.2 朝 pilot 7 日目 / max=50 baseline 2 日目 review template
priority: 高
status: pending Fujiwara input (= D44 close 後)
owner: BlueFire7777 (Fujiwara)
date: 2026-07-14 (= D44、火)
parent_trade_plan: 04_daily/2026-07-14_manual_live_pilot_trade_plan.md
parent_baseline_doc: 03_design/FIRE_f111_max_candidates_baseline_50_2026-05-16.md
---

# Manual Live Pilot — Trade Review (2026-07-14 / D44)

> 本 review は D44 close 後に Fujiwara が手動で埋める。OHLCV / signal_success / entry_success は
> Fujiwara input required (= 推測禁止)。
>
> **v1.4.2 朝 pilot 7 日目 / max=50 baseline 2 日目**.
> 9130 demote 9 日目、4404 entry 復帰 7 日目、**9247/9628 連続 13 日目 (= fixed-candidate HIGH 相当)**.

## §0 D44 actual status snapshot (= pending Fujiwara input)

- **actual_entry**: Fujiwara input required
- **9247 result**: pending (= 連続 13 日目、fixed-candidate HIGH)
- **9628 result**: pending (= 連続 13 日目、fixed-candidate HIGH)
- **4404 result**: pending (= v1.4.2 7 日目、⚠ risk warning)
- **3089 result**: pending (= 新規継続、商社 sector)
- **9633 result**: pending (= 新規継続、不動産 sector)
- **9130 demote 効果観察**: Fujiwara input required (= 9 日目)
- **OHLCV**: J-Quants auto-fill 待ち
- **signal_success / entry_success**: pending / NA
- **§1-§10 review items**: 各 [必須 N] Fujiwara input required

## §1 [必須 1] 入った / 見送った

| code | 入った / 見送った |
|---|---|
| 9247 | Fujiwara input required (= 連続 13 日目、fixed-candidate HIGH) |
| 9628 | Fujiwara input required (= 同上、9247 同 sector) |
| 4404 | Fujiwara input required (= v1.4.2 復帰 7 日目、risk warning 付き) |
| 3089 | Fujiwara input required (= 新規継続、商社) |
| 9633 | Fujiwara input required (= 新規継続、不動産) |
| 9130 | **skip 確定 (= demote 効果継続 9 日目)** ✓ |

## §2 [必須 2] 判断理由

| code | 判断理由 |
|---|---|
| 9247 | Fujiwara input required (= 連続 13 日目、fixed-candidate HIGH、片方絞り検討) |
| 9628 | Fujiwara input required (= 同上、9247 同 sector 重複) |
| 4404 | Fujiwara input required (= v1.4.2 復帰 7 日目、risk 警告 10,665 円許容判断) |
| 3089 | Fujiwara input required (= 新規継続、商社 sector 流動性確認) |
| 9633 | Fujiwara input required (= 新規継続、不動産 sector 流動性確認) |
| 9130 | 既定: demote 効果継続 9 日目、recently_seen_demoted、excluded 固定 |

## §3 [必須 3] 実際に見た銘柄

- **9247 ＴＲＥ HD** (= サービス業、entry_candidate #1、TOPIX Small 1、連続 13 日目)
- **9628 燦 HD** (= サービス業、entry_candidate #2、TOPIX Small 2、連続 13 日目)
- **4404 ミヨシ油脂** (= 食品工業、v1.4.2 entry 復帰 7 日目、⚠ risk warning)
- **3089 テクノアルファ** (= 商社・卸売、新規 entry top 4、D43 から継続)
- **9633 東京テアトル** (= 不動産、新規 entry top 5、D43 から継続)
- **9130 共栄タンカー** (= 海運業、D36 demote、D44 で excluded 9 日目)

(参考: top 5 以外の entry 9 件 = 2146/4828/8699/3712/7803/4914/3479/4540/8057、D43 と同)

D31-D43 と D44 を **混同しない**.

## §4 [必須 4] 板・出来高・spread + 前場/終日実績

| code | 寄値 | 高値 | 安値 | 終値 | 前日比 | 出来高 | spread 所感 | 当日方向感 |
|---|---|---|---|---|---|---|---|---|
| 9247 | Fujiwara input required | ... | ... | ... | ... | ... | ... | ... |
| 9628 | Fujiwara input required | ... | ... | ... | ... | ... | ... | ... |
| 4404 | Fujiwara input required | ... | ... | ... | ... | ... | Fujiwara input required (= risk warning 付き) | ... |
| 3089 | Fujiwara input required | ... | ... | ... | ... | ... | Fujiwara input required (= D43 から継続、流動性) | ... |
| 9633 | Fujiwara input required | ... | ... | ... | ... | ... | Fujiwara input required (= D43 から継続、流動性) | ... |
| 9130 | Fujiwara input required | ... | ... | ... | ... | ... | (= excluded、demote 9 日目観察) | ... |

## §5 [必須 5] 結果メモ

Fujiwara input required (= D44 close 後)。

参考観点:
- max=50 baseline 2 日目運用評価 (= D43 と比較、entry 14 件継続)
- top 5 圧縮の継続有効性 (= 主役 3 + 新規 2)
- 9247 / 9628 連続 13 日目 fixed-candidate HIGH 評価 (= D45+ 片方 demote 候補化)
- 4404 v1.4.2 連続 7 日目運用検証
- 9130 demote 9 日目効果継続
- sector 多様化 (= entry 上 7 sector) 継続評価
- 新規 entry (3089/9633) の D43 → D44 連続性
- F111-THEME-SECTOR-OVERLAY-R1 起票判断材料 (= seed + dynamic discovery、固定 10 テーマ限定無)

## §6 [必須 6] 実 entry か skip か (= D44 actual、100 株標準)

| code | 判定 |
|---|---|
| 9247 | Fujiwara input required (= 連続 13 日目 fixed-candidate HIGH 慎重) |
| 9628 | Fujiwara input required (= 同 sector 重複、9247 と片方絞り推奨) |
| 4404 | Fujiwara input required (= risk 警告許容判断後) |
| 3089 | Fujiwara input required (= 新規継続、流動性確認後) |
| 9633 | Fujiwara input required (= 新規継続、流動性確認後) |
| 9130 | **NO enter (= demote 済、excluded 固定)** ✓ |

## §7 [必須 7] 入った価格 or 見送り理由

| code | 価格 or 理由 |
|---|---|
| 9247 | Fujiwara input required (= 連続 13 日目 fixed-candidate HIGH) |
| 9628 | Fujiwara input required (= 同上) |
| 4404 | Fujiwara input required (= v1.4.2 連続 7 日目、risk warning) |
| 3089 | Fujiwara input required (= 新規継続) |
| 9633 | Fujiwara input required (= 新規継続) |
| 9130 | 見送り (= D36 demote、D44 で excluded 9 日目) |

## §8 [必須 8] 追わない上限を超えていたか

| code | chase_limit (+2.0%) | actual high | 超過 |
|---|---|---|---|
| 9247 | 1,644 円 | Fujiwara input required | Fujiwara input required |
| 9628 | 1,418 円 | Fujiwara input required | Fujiwara input required |
| 4404 | 2,176 円 | Fujiwara input required | Fujiwara input required |
| 3089 | 1,076 円 | Fujiwara input required | Fujiwara input required |
| 9633 | 1,610 円 | Fujiwara input required | Fujiwara input required |
| 9130 | 1,438 円 | Fujiwara input required | (= excluded、参考値) |

## §9 [必須 9] pre-open 判断か post-open 判断か

| code | 判断 phase |
|---|---|
| 9247 | Fujiwara input required |
| 9628 | Fujiwara input required |
| 4404 | Fujiwara input required (= risk warning は pre-open 表示済) |
| 3089 | Fujiwara input required |
| 9633 | Fujiwara input required |
| 9130 | **pre-open 判断** (= demote 効果継続、excluded 固定) |

## §10 [必須 10] signal_success と entry_success の区別

| code | signal_success | entry_success |
|---|---|---|
| 9247 | Fujiwara input required | Fujiwara input required |
| 9628 | Fujiwara input required | Fujiwara input required |
| 4404 | Fujiwara input required (= v1.4.2 復帰 7 日目) | Fujiwara input required |
| 3089 | Fujiwara input required (= 新規継続) | Fujiwara input required |
| 9633 | Fujiwara input required (= 新規継続) | Fujiwara input required |
| 9130 | Fujiwara input required (= 観察用) | **NA / demoted** ✓ |

**signal_success 判定基準注記**:
- historical review: 当日朝 advisory 利確目安への事後評価、Fujiwara 手動
- current renderer: post-open high/low の +2R / -1R hit 自動判定、推測禁止
- → 別 field / 別意味、current renderer 値で historical を **上書きしない**

## §11 D31-D43 引き継ぎ + 9130 demote / 4404 / 9247-9628 / max=50 観察

- D31 (06-25): 9130 +4.89% / 連続 1
- D32-D34: 9130 連続 2-4 / 4404 watch (= 旧 v1.4.1)
- D35: 9130 連続 5 + demote sim
- D36: 9130 demote 本実行
- D37: 9130 demote 継続 / sector 集中化 / max_candidates sim
- D38: v1.4.2 初 pilot / 4404 復帰 + warning / sector 2 多様化
- D39-D40: v1.4.2 2-3 日目
- D41: v1.4.2 4 日目 / 9247-9628 連続 10 日目 / demote sim 完了
- D42: v1.4.2 5 日目 / 9247-9628 連続 11 日目 / max=50 sim 完了
- D43: v1.4.2 6 日目 / 9247-9628 連続 12 日目 / max=50 baseline 初回稼働
- **D44 (07-14): v1.4.2 7 日目 / 9130 demote 9 日目 / 4404 連続 7 日目 / 9247-9628 連続 13 日目 (= fixed-candidate HIGH) / max=50 baseline 2 日目** ← 本日

連続 counter (= demote 実行日 D36 を 1 とする):
- 9130 demote: D36 (1) → D44 (9) = **9 日連続**
- 4404 v1.4.2 entry: D38 (1) → D44 (7) = **7 日連続**
- 9247 / 9628 entry: D32 から **13 日連続** (= 9130 閾値の 2.6 倍超、fixed-candidate HIGH)

正式 recently_seen_codes (= 8 件版、D36 以降継続):
`9130, 8747, 137A0, 7991, 340A0, 5729, 3489, 3798` (= demoted_count: 8)

D44 判定ルール 明記:
- 9130 再 entry → **CRITICAL** (= D44 完了扱い禁止 / recently_seen logic 調査)
- 9130 watch 戻り → **HIGH** (= D44 完了扱い禁止または HOLD)
- 4404 watch 戻り → **HIGH** (= v1.4.2 risk cap 撤廃 consumer/MD 崩れ可能性)
- 低流動性 / letter-suffix / 非 100 株 entry 浮上 → **HIGH** (= HOLD/NO-GO)
- 9247 / 9628 連続 13 日 → **fixed-candidate HIGH 相当** (= 本 wave 記録、D45+ 検討)

## §12 max=50 baseline 2 日目評価

| 観点 | D43 結果 | D44 結果 | 比較 |
|---|---|---|---|
| cli_version | 1.3.0 | 1.3.0 | 維持 |
| max_candidates | 50 | 50 | 維持 |
| policy_version | 1.4.2 | 1.4.2 | 維持 |
| entry 件数 | 14 | 14 | 維持 |
| sector (entry 上) | 7 | 7 | 維持 |
| 低流動性混入 | 0 | 0 | 維持 |
| letter-suffix 混入 | 0 | 0 | 維持 |
| 非 100 株 | 0 | 0 | 維持 |
| risk_warning entry | 4 | 4 | 維持 |
| 朝判断負荷 | top 5 圧縮 | top 5 圧縮 | 同方針 |

→ D44 = D43 と同 entry / 同 sector / 同方針 (= 母集団 35 件で安定).

## §13 9247 / 9628 fixed-candidate HIGH 整理

| 観点 | 状態 |
|---|---|
| 9247 連続 entry | 13 日 (D32-D44) |
| 9628 連続 entry | 13 日 (D32-D44) |
| 9130 demote 閾値 (参考) | 5 連続で demote 実行 (D35→D36) |
| 倍率 | 9130 閾値の 2.6 倍超 |
| D41 sim A (9247 demote) | 慎重 OK (entry 2 件残、sector 2 維持) |
| D41 sim B (9628 demote) | 慎重 OK (同上) |
| D41 sim C (両方 demote) | **非推奨** (entry 1 件のみ) |
| D42 max=50 sim | entry 14 件 / sector 7 浮上、demote 後の安全余地確認 |

→ **D45 以降の本実行候補** (= HQ approve 後の別 wave):
- 第 1 推奨: A (9247 単独 demote)
- 第 2 推奨: B (9628 単独 demote)
- 非推奨: C (両方同時 demote)

## §14 F111-THEME-SECTOR-OVERLAY-R1 起票判断材料

D44 close 後に Fujiwara が確認:
1. seed list (= 半導体 / 電線 / AI / 防衛 / 銀行 / 商社 / 機械 / 電機 / 非鉄 等) 確定
2. dynamic discovery 方式設計 (= 銘柄ニュース / 業種コード / 業績タグ等)
3. 固定 10 テーマ限定しない (= 拡張性確保)
4. F111 出力 field 拡張要否
5. consumer / MD レンダラ拡張要否

## §15 safety footer

- 本 review は **手動記録用 template** (= read-only)
- auto-order / 楽天証券・iSPEED 自動操作 / Computer Use / LINE 自動送信 **禁止**
- DB write 0 / API 0 / token 0 / LINE 0 / launchctl 0
- F282 plist 3 file 別個に safety final 確認:
  - production plist = `docs/launchd/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778592445)
  - smoke plist = `docs/launchd/jp.fire.weekly-snapshot-smoke.plist` (size=1844 / mtime=1778769277)
  - LaunchAgents loaded plist = `~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` (size=1772 / mtime=1778593597)
- 100 株標準 / 非 100 株調整 0 / forbidden phrase 0
- D31-D44 fixture / docs 分離維持
- 9247/9628 demote 本実行は別 wave (= HQ approve 後)
- F111 max=50 baseline / max=100/200/300 採用せず
- 両方同時 demote 原則非推奨
- theme overlay は seed + dynamic discovery 方式、固定 10 テーマ限定無
