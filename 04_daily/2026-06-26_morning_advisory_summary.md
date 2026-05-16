---
template: morning-advisory-summary
version: 1.4.1
policy_applied: selection_policy_v1.4.1 (= canonical)
date: 2026-06-26
weekday: 金
owner: BlueFire7777 (Fujiwara)
linked_pilot_day: D32
linked_design: 03_design/FIRE_selection_policy_v1.4.1_2026-05-15.md
canonical_d31_advisory: 04_daily/2026-06-25_morning_advisory_summary_v1.4.1.md (= D32 判断の正本)
deprecated_d31_advisory: 04_daily/2026-06-25_morning_advisory_summary.md (= v1.0、baseline/履歴のみ、D32 判断には使わない)
linked_d31_review: 04_daily/2026-06-25_manual_live_pilot_review.md (= filled、10 項目構造)
pilot_judgment: GO_CONDITIONAL maintained
post_recovery: 13 連続 maintained
hold_2_avoidance: 完全継続 (= D15 以降 17 連続)
demoted_count: 7
nine130_chase_status: "D31 で +4.89%、追わない上限超過 → D32 で no-new-chase 維持"
purpose: "v1.4.1 framework 適用、D31 学び反映、9130 = signal 継続 + no-new-chase + demote 監視"
---

# 【FIRE 朝の銘柄通知サマリ】 — 2026-06-26 (金) / D32

## 判定

**GO_CONDITIONAL maintained 13 連続** (= D20 復帰以降、HOLD #2 完全回避 17 連続)

⚠ **9130 = signal_success 継続、ただし no-new-chase** (= D31 overshoot 反映)

## D31 学び反映

- D31: 全 entry skip、operational decision valid
- 9130: signal_success ✓、ただし +4.89% で追わない上限超過 → 新規追い NG
- 331A0: excluded_candidate_validated (= 流動性 NG 確認)
- 4389: watch_signal_success (= signal positive ながら流動性 NG で entry 外)
- Liquidity gate remains necessary
- signal_success と entry_success を分離記録

## D32 Pre-Open ranking (= staging proxy、参考)

| rank | code | name | 33 sector / 業務 | margin | scale | research | post-open 状態 |
|---|---|---|---|---|---|---|---|
| 1 (proxy) | **9130** | 共栄タンカー | 海運業 / タンカー | 貸借 | - | 0.8586 | NA (= no-new-chase) |
| 2 | 9247 | ＴＲＥ HD | サービス業 / リサイクル | 貸借 | TOPIX Small 1 | 0.8546 | **entry #1** |
| 3 | 9628 | 燦 HD | サービス業 / 葬祭 | 貸借 | TOPIX Small 2 | (中位) | **entry #2** |
| 4 | 4404 | ミヨシ油脂 | 食品工業 / 油脂 | 貸借 | - | 0.8413 | **watch (= 100 株 risk 高、降格)** |

→ 9130 は staging proxy rank 1 だが D31 overshoot で entry NA
→ 4404 は signal 高位だが 100 株 risk_yen 10,665 で entry_candidate から watch へ降格

## D32 Entry Priority (= 実弾運用順位、100 株標準) ★

⚠ **本 wave 方針修正 (= 100 株標準化)**:
- 実弾パイロットの **標準 entry 単位は 100 株**
- 非 100 株調整は **原則使わない**
- 100 株で pilot risk_yen が他 entry 候補と大幅乖離する銘柄は **entry_candidate から外す**
- 該当銘柄は **watch_candidate** へ降格

| entry_priority | code | 状態 | 理由 |
|---|---|---|---|
| **#1** | **9247** ★ | 新規 entry 検討 (100 株) | 貸借 + TOPIX Small 1、追わない上限内、risk_yen 8,060 |
| **#2** | **9628** | 新規 entry 検討 (100 株) | 貸借 + TOPIX Small 2、ディフェンシブ、risk_yen 6,950 |
| **NA (= signal 継続 + no-new-chase)** | **9130** ⚠ | 新規 entry 推奨せず | D31 overshoot 継続中の場合、追い NG |
| **watch (= 100 株 risk 高、降格)** | **4404** ⚠ | **entry 候補外、観察のみ** | 100 株 risk_yen 10,665 が #1 9247 (8,060) を大幅超過、株数調整不採用 |

### 第一確認候補
**9247 ＴＲＥ HD** (= サービス業/リサイクル、貸借+TOPIX Small 1、追わない上限内、100 株 risk_yen 8,060)

## 9130 の D32 扱い (= signal 継続 + no-new-chase + demote 準備)

### 9130 D32 判定 rule

D31 で +4.89% (= 1,479 円) overshoot。D32 で **新規追い entry は原則禁止**。
以下 6 条件 **全て** 満たした場合のみ再検討可:

| # | 条件 |
|---|---|
| 1 | 1,410 - 1,440 円付近まで自然に押す |
| 2 | 出来高と板が維持される (= 過去平均比 ≥ 80%) |
| 3 | 反発確認 (= 押し目買い、底値形成) |
| 4 | 損切り位置が近く置ける (= 1,395 円以下) |
| 5 | リスクリワード 成立 (= R/R ≥ 1.5) |
| 6 | 新しい材料 / 需給根拠 (= TDnet 開示 / 報道) |

→ 上記 1 つでも欠ければ:
- **signal_success 継続**
- **no-new-chase**
- **demote 候補 monitor** (= D34/D35 で正式 demote 検討)

### 9130 連続性 monitor

| day | 9130 rank 1 連続 | 状態 |
|---|---|---|
| D30 | 1 (初) | 9247 を抜いて新 #1 |
| D31 | 2 | +4.89%、overshoot |
| **D32** | **3 (想定)** | **連続継続だが entry 候補から外す** |
| D33 (想定) | 4 | 同上 |
| D34 (想定) | **5 = warning** | demote sim 候補化 |
| D35 (想定) | demoted | 段階 3 本実行候補 |

→ rank 1 連続性 ≠ entry 推奨、v1.4.1 で明確分離

## 9247 / 9628 / 4404 評価

### 9247 ＴＲＥホールディングス (= entry_priority #1)

- liquidity: PASS (= 貸借 + TOPIX Small 1)
- 33 sector: サービス業 / リサイクル・環境
- 基準株価: **1,612 円**
- 追わない上限: 1,644 円 (= +2.0%)
- 想定 risk_yen: 8,060 円 (= 100 株、pilot 限度内)
- ESG テーマ、ただし当日 catalyst 未確認 → theme_evidence partial
- 推奨度: **★★★** (= 最有力候補)

### 9628 燦ホールディングス (= entry_priority #2)

- liquidity: PASS (= 貸借 + TOPIX Small 2)
- 33 sector: サービス業 / 葬祭
- 基準株価: **1,390 円**
- 追わない上限: 1,418 円 (= +2.0%)
- 想定 risk_yen: 6,950 円 (= 100 株、pilot 限度内)
- ディフェンシブ、catalyst なし
- 推奨度: **★★** (= 9247 が NG 時の代替)

### 4404 ミヨシ油脂 (= watch_candidate へ降格、entry 候補外) ⚠

- liquidity: PASS (= 貸借) ✓
- 33 sector: 食品工業 / 油脂
- 基準株価: **2,133 円**
- 100 株 risk_yen: **10,665 円**
- 100 株 entry: **不採用** (= #1 9247 risk 8,060 を大幅超、entry_candidate ライン超)
- 非 100 株調整: **本方針で不採用** (= 標準は 100 株のみ)
- watch_candidate 扱い: signal は positive (research 0.8413)、ただし 100 株での pilot 適性低
- entry **不可**、観察のみ
- 推奨度: **NA** (= entry 対象外、signal は記録)

## excluded_candidate (= 8 件、entry 不可)

| code | name | 除外理由 |
|---|---|---|
| 4389 | プロパティデータバンク | 信用 + 非 TOPIX (= D31 watch_signal_success だが流動性 NG) |
| 4317 | レイ | 信用 + 非 TOPIX |
| 2700 | 木徳神糧 | 信用 + 非 TOPIX |
| 6149 | 小田原エンジニアリング | 信用 + 非 TOPIX |
| 2981 | ランディックス | 信用 + 非 TOPIX |
| 331A0 ★ | メディックス | letter-suffix 新規 + 信用 + 非 TOPIX (= D31 excluded_validated) |
| 288A0 | ラクサス・テクノロジーズ | letter-suffix 新規 + 信用 + 非 TOPIX |
| 339A0 | プログレス・テクノロジーズ | letter-suffix 新規 + 信用 + 非 TOPIX |

---

## 【注文完成形アドバイザリ】

⚠ 発注指示ではない。Fujiwara が iSPEED で手動確認するための目安。

### 候補 1: 9247 ＴＲＥ HD (サービス業 / リサイクル) ★ 第一確認

- 基準株価: **1,612 円**
- エントリー検討価格: **1,596 - 1,628 円** (= 基準 ±1%)
- 指値目安: **1,612 円**
- 追わない上限: **1,644 円** (= +2.0%)
- 利確目安: **1,660 - 1,693 円** (= +3-5%)
- 損切り目安: **1,532 円** (= -80 円)
- 想定株数: **100 株**
- 想定 risk_yen: **8,060 円**
- 見送り条件: 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,644 円超) / ESG 後退ニュース

### 候補 2: 9628 燦 HD (サービス業 / 葬祭)

- 基準株価: **1,390 円**
- エントリー検討価格: **1,376 - 1,404 円**
- 指値目安: **1,390 円**
- 追わない上限: **1,418 円** (= +2.0%)
- 利確目安: **1,432 - 1,460 円** (= +3-5%)
- 損切り目安: **1,320 円**
- 想定株数: **100 株**
- 想定 risk_yen: **6,950 円**
- 見送り条件: 板薄 / 出来高小 / spread 広 / 寄り急騰 (1,418 円超) / 悪材料

### 候補 NA: 4404 ミヨシ油脂 (食品工業 / 油脂) — watch へ降格、entry 候補外 ⚠

- 基準株価: 2,133 円
- 100 株 risk_yen 10,665 = #1 9247 (8,060) を大幅超
- **100 株標準方針**に従い entry_candidate から **watch_candidate** へ降格
- 非 100 株調整は **不採用** (= 標準は 100 株のみ)
- signal positive (research 0.8413) のため signal_success として観察記録
- entry 不可、Fujiwara 朝確認時の判断対象外

### 候補 NA: 9130 共栄タンカー (海運業) — signal 継続 + no-new-chase ⚠

- D31 で +4.89%、追わない上限 1,440 円超過
- **新規追い entry 不可**
- 例外: 上記 6 条件全充足時のみ再検討可
- 推奨: **観察のみ、demote 候補 monitor (= D34/D35 想定)**

---

## 藤原が iSPEED で見ること (= 7 軸、v1.4.1)

1. **daily volume** (= 過去 20 day 平均): ≥ 10 万株 推奨 ★ actual
2. **turnover (= 売買代金)**: ≥ 1 億円 推奨 ★ actual
3. **opening volume (= 寄り 5 分)**: ≥ 1,000 株 ★ actual
4. **spread + board depth**
5. **寄り後 5 min 出来高 + 上昇率** (= post-open 評価の核) ★
6. **ニュース / 適時開示 / catalyst** (= TDnet)
7. **9130 価格帯確認** (= 1,440 円超ならば追わない、1,410-1,440 自然押しなら 6 条件確認)

---

## review 必須 10 項目 (= v1.4.1 維持)

1. 入った / 見送った
2. 判断理由
3. 実際に見た銘柄 (= ticker / name + 33 sector)
4. 板・出来高・spread 所感
5. 結果メモ
6. 実 entry か skip か
7. 入った価格 or 見送った理由
8. 追わない上限を超えていたか
9. pre-open / post-open 判断
10. signal_success と entry_success の区別

review path: `~/fire-vault/04_daily/2026-06-26_manual_live_pilot_review.md`

---

## 安全確認

- 自動発注 0 / 楽天 0 / iSPEED 自動操作 0 / Computer Use 0
- LINE 送信 0 / token 0 / API 0 / DB write 0
- git commit 0 / git push 0 / --no-verify 0
- launchctl / plist / cron 変更 0
- F111 runner code 変更 0

### 3 DB md5 / F282 plist 明示記録 (= Codex Lane D 指摘解消)

- production md5: **b1df4673e5c3645fbe2c5f490ffac043** → 不変 ✓
- develop md5:    **0eed4ad2ec7ed2edf8f640d97341c5ad** → 不変 ✓
- staging md5:    **6cb3885cd9c90bd6fdabb127c1cd0d17** → 不変 ✓
- F282 本番 plist (= jp.fire.weekly-snapshot.plist): **size=1772 / mtime=1778593597** → 不変 ✓

## safety footer

手動運用補助 / FIRE 発注しない / auto_order_allowed=False / manual_review=True /
楽天正本 / Computer Use 不採用 / 最終判断は Fujiwara
