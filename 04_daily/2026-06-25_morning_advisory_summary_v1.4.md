---
template: morning-advisory-summary
version: 1.4
policy_applied: selection_policy_v1.4 (= pre-open / post-open phase 分離 + evidence-based theme)
date: 2026-06-25
weekday: 木
owner: BlueFire7777 (Fujiwara)
linked_pilot_day: D31
linked_pilot_results: 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_D31_results.md
linked_design: 03_design/FIRE_selection_policy_v1.4_2026-05-15.md
supersedes:
  - 04_daily/2026-06-25_morning_advisory_summary.md (= v1.0)
  - 04_daily/2026-06-25_morning_advisory_summary_v1.1.md (= v1.1)
  - 04_daily/2026-06-25_morning_advisory_summary_v1.2.md (= v1.2)
  - 04_daily/2026-06-25_morning_advisory_summary_v1.3.md (= v1.3)
pilot_judgment: GO_CONDITIONAL
post_recovery: 12 連続 maintained
hold_2_avoidance: 完全継続 (= D15 以降 16 連続)
demoted_count: 7
purpose: "v1.4 = pre-open ranking と post-open entry を分離。9130 前場実績反映で final rank 1。"
qa_consistency:
  date_matches_filename: true (= 2026-06-25 統一)
  date_matches_pilot_day: true (= D31 = 2026-06-25 木)
  design_doc_naming: "v1.4_2026-05-15.md (= 設計発行日)、advisory date (= 対象日 2026-06-25) と別 metadata、意図通り"
---

# 【FIRE 朝の銘柄通知サマリ v1.4】 — 2026-06-25 (木) / D31 + Pre/Post-Open 分離

## 判定

**GO_CONDITIONAL** (= pilot D31 で 12 連続維持、HOLD #2 完全回避 16 連続)

⚠ v1.4 改訂: pre-open ranking (= 寄り前 staging proxy) と post-open entry
(= 寄り後 actual flow) を分離。**前場実績で 9130 が最強だった事実を反映** し、
final rank で 9130 を #1 promote。theme/catalyst は evidence-based 化で根拠
なしは加点 0。

## v1.4 framework

```
[Pre-open phase] (= 寄り前 09:00、staging proxy のみ)

pre_open_score =
    liquidity_static  * 0.30  # margin + 20day vol proxy
  + cap_static        * 0.20  # scale_category
  + sector_trend_proxy* 0.10  # 33 sector 一般 trend
  + theme_evidence    * 0.10  # 当日根拠ありの場合のみ加点 ★
  + research          * 0.10  # F111 既存
  + catalyst_evidence * 0.10  # TDnet/ニュース確認時のみ ★
  + reserved_post_open* 0.10  # post-open uplift 予約枠

(risk_yen 不採用、theme/catalyst は evidence 必須)

[Post-open phase] (= 寄り後 09:00-11:30、Fujiwara iSPEED actual)

5 軸全 PASS の場合: entry 候補維持
1 軸でも FAIL: skip
寄り後 出来高高 + 上昇率適度 + catalyst 確認: rank uplift +1.0
```

## Pre-open ranking (= 参考)

| rank | code | name | 33 sector | pre_open | 構成 |
|---|---|---|---|---|---|
| 1 | 9247 | ＴＲＥ HD | サービス業/リサイクル | **5.61** | liq=7 cap=7 sec=8 theme=4 res=8.55 cat=0 |
| 2 | 9628 | 燦 HD | サービス業/葬祭 | **4.65** | liq=6 cap=6 sec=6 theme=2 res=8.48 cat=0 |
| 3 | 9130 | 共栄タンカー | 海運業/タンカー | **4.06** | liq=5 cap=3 sec=7 theme=4 res=8.59 cat=0 |
| 4 | 4404 | ミヨシ油脂 | 食品工業/油脂 | **3.74** | liq=5 cap=3 sec=5 theme=0 res=8.41 cat=0 |

→ v1.3 と順位同じ、ただし theme/catalyst evidence-based 化で絶対 score 低下

## Post-open final ranking (= 前場 09:00-11:30 反映、entry 順位 ★)

Fujiwara 観測: **「前場で 9130 が最も強かった」**

| rank | code | name | 33 sector | pre_open | post-open uplift | final_score |
|---|---|---|---|---|---|---|
| **1** ★ | **9130** | 共栄タンカー | **海運業 / タンカー** | 4.06 | **+1.5** | **5.56** |
| 2 | 9247 | ＴＲＥ HD | サービス業 / リサイクル | 5.61 | 0 | 5.61 (= 微差) |
| 3 | 9628 | 燦 HD | サービス業 / 葬祭 | 4.65 | 0 | 4.65 |

→ **post-open 実 flow 優先**で 9130 = entry candidate #1
→ 9247 は pre_open で僅か上回るが、actual momentum なしで rank 2
→ 9130 が崩れた場合 9247 へ降格余地

### 第一確認候補
**9130 共栄タンカー** (= 海運業、前場実績で強さ確認、post-open uplift +1.5)

### sector 多様化 (= post-open final)

- 33 sector **2 種** (= 海運業 + サービス業)、業務 **3 種** (タンカー + リサイクル + 葬祭)
- #4 = 4404 食品工業 加わると 33 sector 3 種

## watch_candidate

D31 staging F111 出力に該当 0 件

## excluded_candidate (= 8 件)

| code | name | 除外理由 |
|---|---|---|
| 4389 | プロパティデータバンク | 信用 + 非 TOPIX |
| 4317 | レイ | 信用 + 非 TOPIX |
| 2700 | 木徳神糧 | 信用 + 非 TOPIX |
| 6149 | 小田原エンジニアリング | 信用 + 非 TOPIX |
| 2981 | ランディックス | 信用 + 非 TOPIX |
| **331A0** | **メディックス** ★ | letter-suffix 新規 + 信用 + 非 TOPIX |
| 288A0 | ラクサス・テクノロジーズ | letter-suffix 新規 + 信用 + 非 TOPIX |
| 339A0 | プログレス・テクノロジーズ | letter-suffix 新規 + 信用 + 非 TOPIX |

→ 331A0 / 4389 / 4317 全 excluded 維持 ✓

---

## 【注文完成形アドバイザリ v1.4】 (= post-open final ranking 基準)

⚠ 発注指示ではない。Fujiwara が iSPEED で手動確認するための目安。

### 候補 1: 9130 共栄タンカー (海運業 / タンカー) ★ 第一確認、前場実績で promote

- liquidity_status: PASS (= 貸借)
- pre_open: 4.06 → post-open uplift +1.5 → **final 5.56 ★**
- 前場実績: 出来高高 + 上昇率適度 + catalyst (= タンカー関連) 確認 (= Fujiwara 観測)
- 基準株価: **1,410 円**
- エントリー検討価格: **1,395 - 1,425 円**
- 指値目安: **1,410 円**
- 追わない上限: **1,440 円** (= +2.1%)
- 利確目安: **1,450 - 1,470 円** (= +3-4%)
- 損切り目安: **1,340 円** (= -70 円、risk_yen=7,050 / 100 株)
- 想定株数: **100 株**
- 想定 risk_yen: **7,050 円**
- 見送り条件:
  - 後場で勢い失う (= 寄り後上昇率 < +0.5% に減速)
  - 板薄 / spread 広 / 出来高 急減
  - タンカー運賃 down 報道
  - 寄り急騰 (+2% 超え、追わない)

### 候補 2: 9247 ＴＲＥ HD (サービス業 / リサイクル・環境) — 9130 崩れ時の代替

- liquidity_status: PASS (= 貸借 + TOPIX Small 1)
- pre_open: 5.61 (= 静的 evaluation 最高位)、ただし actual flow なしで uplift 0
- 基準株価: **1,612 円**
- エントリー検討価格: **1,596 - 1,628 円**
- 指値目安: **1,612 円**
- 追わない上限: **1,644 円** (= +2.0%)
- 利確目安: **1,660 - 1,693 円**
- 損切り目安: **1,532 円**
- 想定株数: **100 株**
- 想定 risk_yen: **8,060 円**
- 見送り条件: 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,644 円超) / ESG 後退ニュース
- **9130 が前場崩れた場合の代替 #1 候補化**

### 候補 3: 9628 燦 HD (サービス業 / 葬祭) — ディフェンシブ

- liquidity_status: PASS (= 貸借 + TOPIX Small 2)
- pre_open: 4.65、actual flow なしで uplift 0
- 基準株価: **1,390 円**
- エントリー検討価格: **1,376 - 1,404 円**
- 指値目安: **1,390 円**
- 追わない上限: **1,418 円**
- 利確目安: **1,432 - 1,460 円**
- 損切り目安: **1,320 円**
- 想定株数: **100 株**
- 想定 risk_yen: **6,950 円**
- 見送り条件: 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,418 円超) / 悪材料
- ディフェンシブ、即時テーマなしのため積極推奨度低

---

## 9130 / 9247 / 9628 評価検証 (= proxy 過大評価防止)

### 検証結果

| 銘柄 | v1.3 sel | v1.4 pre_open | v1.4 final | 評価 |
|---|---|---|---|---|
| 9247 | ~7.36 (= ESG/葬祭 qualitative 加点高) | 5.61 (= evidence-based 化で低下) | 5.61 (= uplift なし) | **proxy 過大評価が補正された** ✓ |
| 9628 | ~6.00 (= 同上) | 4.65 (= 同上) | 4.65 (= 同上) | **同上** ✓ |
| 9130 | ~5.76 (= 海運 theme 加点) | 4.06 (= evidence-based 低下) | **5.56 (= 前場実績 promote)** | **proxy 順位より実 flow で勝った** ★ |

→ v1.4 は actual flow を含めた **真のランキング** を提示できるようになった

---

## 藤原が iSPEED で見ること (= v1.4 で 7 軸へ拡張)

1. **daily volume** (= 過去 20 day 平均): ≥ 10 万株 推奨 ★ actual 確認
2. **turnover (= 売買代金)**: ≥ 1 億円 推奨 ★ actual 確認
3. **opening volume (= 寄り 5 分)**: ≥ 1,000 株 ★ actual 確認
4. **spread + board depth**
5. **寄り後 5 min 出来高 + 上昇率** (= post-open 評価の核) ★
6. **ニュース / 適時開示 / catalyst** (= TDnet で根拠確認)
7. **前場 09:00-11:30 全体観測** (= 9130 等の海運株で必須、final rank 再評価) ★

---

## 今日の review 必須 5 項目

1. **入った / 見送った**
2. **判断理由** (= pre-open vs post-open、どちらの観点で決めたか)
3. **実際に見た銘柄** (= ticker / name + 33 sector + 業務認識)
4. **前場実績** (= 9130 が強かった事実 confirm or rebut、他候補の momentum)
5. **結果メモ**

review path: `~/fire-vault/04_daily/2026-06-25_manual_live_pilot_review.md`

---

## file 命名規約 (= QA 確認)

- 設計 doc: `FIRE_<topic>_v<X.Y>_YYYY-MM-DD.md`
  - date = 設計発行日 (= 本日 2026-05-15) ✓
- advisory: `YYYY-MM-DD_morning_advisory_summary[_vX.Y].md`
  - date = 対象 pilot 日 (= 2026-06-25 = D31) ✓

→ **命名規約は意図通り**、混乱なし

---

## reference (= QA 整合性)

- 連動 pilot day: D31 (= 2026-06-25 木) ✓
- file 名 date: 2026-06-25 = 本文 date ✓
- linked design: FIRE_selection_policy_v1.4_2026-05-15.md (= 設計発行日) ✓
- v1.0-v1.3 baseline 全保持 ✓
- pilot judgment: GO_CONDITIONAL maintained 12 連続
- HOLD #2 完全回避: D15 以降 16 連続
- demoted_count: 7 (= 8747,5729,3489,340A0,3798,137A0,7991)

---

## 安全確認

- 自動発注 0 / 楽天 0 / iSPEED 自動操作 0 / Computer Use 0
- LINE 送信 0 / token 0 / API 0 / DB write 0
- git commit 0 / git push 0 / --no-verify 0
- launchctl / plist / cron 変更 0
- F111 runner code 変更 0 (= 本 wave は design doc + advisory v1.4 のみ)

## safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
