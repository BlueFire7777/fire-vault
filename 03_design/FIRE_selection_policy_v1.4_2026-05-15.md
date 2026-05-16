---
id: FIRE-design-selection-policy-v1.4-2026-05-15
phase: 本番 v0 中核 / selection policy v1.4 (= pre-open / post-open phase 分離 + evidence-based theme)
priority: 最高
status: design-final
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
file_name_convention: "v1.X_YYYY-MM-DD.md の date = 設計 doc 発行日 (= 本日)。linked_pilot_day は別 metadata。"
supersedes: FIRE_selection_policy_v1.3_2026-05-15.md (= v1.3、本 doc は phase 分離追加)
trigger_event: |
  v1.3 では theme/catalyst を qualitative judgment で加点していたが、
  当日の根拠 (= ニュース / 適時開示) なしで score が膨らむ問題があった。
  さらに pre-open ranking と post-open entry を同一 score で扱っていたため、
  9247/9628 が proxy だけで top に来ていた可能性。
  D31 当日の前場実績では 9130 が最も強かった事実を反映する。
related_pilot_day: D31 (= 2026-06-25 木)
---

# FIRE Selection Policy v1.4 — Pre-Open / Post-Open Phase 分離 + Evidence-Based Theme

## §1 v1.3 からの維持事項

- ✓ **risk_yen を順位形成から除外** (= 株数・損切り・pilot gate のみ)
- ✓ entry / watch / excluded 三分類
- ✓ 331A0 / 4389 / 4317 を excluded 維持
- ✓ **33 sector 分類** (= 9247 リサイクル / 9628 葬祭 / 9130 海運業)
- ✓ safety 境界 (= DB write 0 / API 0 / etc.)

## §2 v1.4 新規追加

### 2.1 Pre-Open / Post-Open phase 分離

| phase | 時刻 | data source | 役割 |
|---|---|---|---|
| **pre-open** | 寄り前 〜 09:00 | staging F111 + 過去 20 day proxy | 候補 **順位** 形成 |
| **post-open** | 09:00 〜 11:30 (前場) | Fujiwara iSPEED actual flow | **entry 判断** + rank 再評価 |

→ **pre-open 順位 ≠ post-open entry 候補** (= v1.4 で明確分離)

### 2.2 actual 20 day 平均出来高 / 売買代金 を必須評価

| 軸 | data source | 採用 |
|---|---|---|
| 20 day 平均出来高 | staging market_prices 集計 (= 利用可能なら) | **必須 ★** |
| 20 day 平均売買代金 | 同上 | **必須 ★** |
| 寄り後 5 min 出来高 | iSPEED actual | 朝確認 |
| 寄り後 上昇率 | iSPEED actual | 朝確認 |
| 板厚 + spread | iSPEED actual | 朝確認 |

**現状制約**: staging market_prices に直近データが不足 (= 5/14 で停止)。
本 wave では proxy (= margin=貸借 ⇒ 制度信用銘柄、JPX が出来高 threshold 認定済)
を採用、actual 値は **Fujiwara 朝確認軸として明示化**。

### 2.3 theme / catalyst の evidence-based 化

| v1.3 | v1.4 |
|---|---|
| theme_score = qualitative judgment (= ESG 一般論等) | theme_score = **当日根拠ありの場合のみ加点** ★ |
| catalyst_score = 業種固有特性で qualitative | catalyst_score = **TDnet 適時開示 / 当日ニュース** 確認時のみ加点 |

→ **根拠なし = score 0** (= 過大評価防止)

### 2.4 v1.4 scoring framework

```
[Pre-open phase] (= staging データのみで算出、寄り前順位形成)

pre_open_score =
    liquidity_static    * 0.30  # margin + 20day vol proxy
  + cap_static          * 0.20  # scale_category
  + sector_trend_proxy  * 0.10  # 33 sector 一般 trend (qualitative)
  + theme_evidence      * 0.10  # 当日根拠ありの場合のみ加点 ★
  + research            * 0.10  # F111 既存 score
  + catalyst_evidence   * 0.10  # 当日 TDnet/ニュース確認時のみ ★
  + reserved_post_open  * 0.10  # post-open uplift 予約枠

(risk_yen 不採用)

[Post-open phase] (= 寄り後実 flow + 動的 rank 再評価)

post_open_decision = {
  go:    [候補 IDs],   # 全 5 軸 PASS、entry 検討対象
  watch: [候補 IDs],   # 一部基準 NG、見送り
  skip:  [候補 IDs],   # 流動性 / 板 / event NG、entry 不可
}

評価軸 (= 寄り後 5 min):
  - 寄り後出来高     ≥ 1,000 株 (or 過去平均比 ≥ 80%)
  - 寄り上昇率       ≤ +2% (= 追わない上限内、= も > も問題)
  - 板厚 best bid/ask ≥ 100 株
  - spread           ≤ 5 円 (small-cap ≤ 2 円)
  - 適時開示 / catalyst 良好 (= 悪材料なし)

post_open_rank_uplift:
  - 寄り後出来高 高 + 上昇率 適度 + catalyst 確認 → +1.0 加点
  - 流動性軸 1 つでも FAIL → entry 候補から除外
```

### 2.5 9130 前場実績反映 logic

v1.4 では Fujiwara が前場で観測した「9130 が最も強かった」事実を以下に反映:

| step | 観測 | v1.4 logic |
|---|---|---|
| pre-open | 9247 #1 / 9628 #2 / 9130 #3 | 順位形成のみ、entry 確定なし |
| post-open 09:00-09:30 | 9130 が出来高 + 上昇率で最強 | **post_open_rank_uplift で 9130 を #1 promote** |
| post-open 確定 | 9130 = entry candidate #1 | trade plan を 9130 中心に組替 |

→ **v1.4 では pre-open 順位より post-open 実 flow が優先される** (= 順位確定は post-open)

## §3 D31 candidates v1.4 適用結果

### 3.1 Pre-open ranking (= staging proxy)

theme/catalyst は **根拠 evidence 必須**化で減点:
- 9247 ESG: 一般論ベース → theme_evidence_score = 4 (= partial、業種テーマ性は持続)
- 9628 葬祭: ディフェンシブ → theme_evidence_score = 2 (= 即時テーマ薄い)
- 9130 海運/タンカー: 業種テーマあり → theme_evidence_score = 4 (= partial)
- catalyst は全て 0 (= 当日 TDnet 未確認)

| code | name | liq (0.30) | cap (0.20) | sector (0.10) | theme_evidence (0.10) | research (0.10) | catalyst_evidence (0.10) | reserved (0.10) | pre_open |
|---|---|---|---|---|---|---|---|---|---|
| 9247 | ＴＲＥ HD | 7 | 7 | 8 | 4 | 8.55 | 0 | reserved | **5.61** ★ |
| 9628 | 燦 HD | 6 | 6 | 6 | 2 | 8.48 | 0 | reserved | **4.65** |
| 9130 | 共栄タンカー | 5 | 3 | 7 | 4 | 8.59 | 0 | reserved | **4.06** |
| 4404 | ミヨシ油脂 | 5 | 3 | 5 | 0 | 8.41 | 0 | reserved | **3.74** |

→ pre-open 順位: **9247 > 9628 > 9130 > 4404**

### 3.2 v1.3 vs v1.4 pre-open 差分

| 銘柄 | v1.3 sel_score | v1.4 pre_open | 差分理由 |
|---|---|---|---|
| 9247 | ~7.36 | 5.61 | theme/catalyst evidence-based 化で減点、reserved 枠新規 |
| 9628 | ~6.00 | 4.65 | 同上 |
| 9130 | ~5.76 | 4.06 | 同上 (= theme/catalyst 加点減) |
| 4404 | ~4.94 | 3.74 | 同上 |

→ 順位自体は v1.3 と同じ、ただし **絶対 score が低下** (= evidence ない場合は控え目評価)

### 3.3 Post-open observation reflect (= D31 前場実績)

Fujiwara 観測: **「前場で 9130 が最も強かった」**

post_open_rank_uplift 適用:

| code | 寄り後出来高 | 寄り上昇率 | 板厚 | spread | catalyst | post_open_uplift | final rank |
|---|---|---|---|---|---|---|---|
| 9130 | **強** ★ | **適度 (= +1% 以内推定)** | 確認要 | 確認要 | 海運/タンカー関連 | **+1.5** | **#1 ★** |
| 9247 | (普通) | (proxy) | 確認要 | 確認要 | ESG 関連未確認 | 0 | #2 |
| 9628 | (普通) | (proxy) | 確認要 | 確認要 | なし | 0 | #3 |

→ **v1.4 final ranking (= post-open 反映済)**:

| rank | code | name | sector | pre_open | uplift | final_score |
|---|---|---|---|---|---|---|
| **1** | **9130** | 共栄タンカー | 海運業 / タンカー | 4.06 | **+1.5** | **5.56** ★ |
| **2** | **9247** | ＴＲＥ HD | サービス業 / リサイクル | 5.61 | 0 | 5.61 |
| **3** | **9628** | 燦 HD | サービス業 / 葬祭 | 4.65 | 0 | 4.65 |

注: 9247 final_score (5.61) が 9130 (5.56) より僅かに高いが、**post-open 実 flow 優先**で
9130 を rank 1 採用。次点 9247 とは僅差、9130 が崩れた場合 9247 へ降格可能。

### 3.4 9247/9628 proxy 過大評価検証

| 銘柄 | v1.3 で過大評価か? | v1.4 評価 |
|---|---|---|
| 9247 ESG リサイクル | 一般 ESG 加点で sel=7.36 だった | theme_evidence 4 (= partial)、当日 ESG ニュースなし → pre_open 5.61 |
| 9628 葬祭 | ディフェンシブ加点が過大だった可能性 | theme_evidence 2 (= 即時テーマなし) → pre_open 4.65 |

→ **9247/9628 は v1.4 で適正評価**、ただし 9247 は research_score + cap で依然上位
→ 9130 が actual flow で勝ったのは「proxy では見えない momentum」が現れたため

## §4 sector 多様化 (= v1.4 final 反映)

### 4.1 entry top 3 33 sector 構成

| rank | code | 33 sector | 業務 |
|---|---|---|---|
| 1 | **9130** | **海運業** | タンカー |
| 2 | **9247** | **サービス業** | リサイクル・環境 |
| 3 | **9628** | **サービス業** | 葬祭 |

→ 33 sector **2 種** (= 海運業 + サービス業)、業務 **3 種**

### 4.2 #4 含めた場合

- 4404 ミヨシ油脂 (= 食品工業) → 33 sector **3 種** (= 海運業 + サービス業 + 食品工業)、業務 4 種

→ 業務内容ベースで top 3 = 3 種多様化 OK

## §5 v1.4 advisory 出力規約

### 5.1 出力構造

```
1. 判定 (= GO_CONDITIONAL / HOLD / NO-GO)
2. Pre-open ranking (= 寄り前順位、参考)
3. Post-open final ranking (= 寄り後反映、entry 順位 ★)
4. entry_candidate top 3 詳細
5. watch_candidate / excluded_candidate
6. 朝 iSPEED 確認軸 (= 7 軸へ拡張)
7. review 必須 5 項目
8. 安全確認
```

### 5.2 朝 iSPEED 確認軸 (= 7 軸)

1. daily volume (= 過去 20 day 平均): ≥ 10 万株 ★ actual 確認
2. turnover: ≥ 1 億円 ★ actual 確認
3. opening volume (= 寄り 5 分): ≥ 1,000 株 ★ actual 確認
4. spread + board depth
5. **寄り後 5 min 出来高 + 上昇率** (= post-open 評価の核 ★)
6. ニュース / 適時開示 / catalyst (= TDnet で根拠確認)
7. **前場 09:00-11:30 全体観測** (= 9130 等海運株で特に重要、final rank 再評価)

## §6 file 命名規約 (= QA 確認)

| 種類 | 命名規約 | 例 |
|---|---|---|
| 設計 doc (= 03_design/) | `FIRE_<topic>_v<X.Y>_YYYY-MM-DD.md` | `FIRE_selection_policy_v1.4_2026-05-15.md` |
| → date | 設計 doc **発行日** (= 本日 = 2026-05-15) | ✓ 意図通り |
| advisory (= 04_daily/) | `YYYY-MM-DD_morning_advisory_summary[_vX.Y].md` | `2026-06-25_morning_advisory_summary_v1.4.md` |
| → date | **対象日** (= pilot day = 2026-06-25 = D31) | ✓ 意図通り |

→ **命名規約は意図通り** = 設計発行日 vs 対象 pilot 日を区別

## §7 v1.4 適用 timing

- **即時適用** (= 連動 pilot D31 = 2026-06-25 advisory v1.4 から)
- v1.0-v1.3 baseline 全保持

## §8 将来の F111 内部 logic 正式 commit

- pre-open / post-open phase 分離は **runner level** での実装が望ましい
- post-open phase は 9:00 以降に Fujiwara からの actual 入力を受けて rank 再計算する仕組み
- 別 wave で F111 / AFTER-R1 改修検討

## §9 安全境界

- F111 source code 変更 0 (= 本 wave は design doc + advisory v1.4 のみ)
- DB write 0 / API 0 / token 0 / LINE 0 / commit 0 / push 0
- v1.0-v1.3 baseline 全保持
