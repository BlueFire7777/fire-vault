---
id: FIRE-design-selection-policy-v1.2-2026-05-15
phase: 本番 v0 中核 / selection policy v1.2 (= risk_yen 非加点 + liquidity/cap 加重)
priority: 最高
status: design-final
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
supersedes: FIRE_liquidity_gate_hotfix_2026-05-15.md (= v1.1、本 doc は upstream policy 修正)
trigger_event: "v1.1 hotfix だけでは risk_yen が暗黙の順位加点として残っていた = 小型・低流動性銘柄が依然 top に上がる"
related_pilot_day: D31 (= 2026-06-25 木)
---

# FIRE Selection Policy v1.2 — risk_yen 非加点 + liquidity/cap/momentum 加重

## §1 問題意識 (= v1.1 の限界)

v1.1 hotfix で「貸借 + TOPIX」を PASS、「信用 + 非 TOPIX」を FAIL に分類した。
ただし v1.1 の advisory top 3 は **risk_yen が低い順** で並んでいた可能性があり、
- 9130 (= 1,410 円、貸借、scale=-) が #1
- 9247 (= 1,612 円、貸借、Small 1) が #2
- 9628 (= 1,390 円、貸借、Small 2) が #3
の順だった。

これは依然「risk_yen 低 = 安全 = 優先」という暗黙加点を残しており、
小型銘柄 (= 9130, scale=-) を高位に出してしまう構造。

## §2 新方針 v1.2

### 2.1 risk_yen の役割明確化

| 用途 | 採否 |
|---|---|
| **候補順位の加点要素** | **不採用** ★ (= 旧方針からの最大の変更) |
| ポジションサイズ計算 (= F130 許容損失) | 採用 (= 既存) |
| 損切り幅設計 (= 100 株 × risk_yen / 100) | 採用 (= 既存) |
| risk_within_pilot_limit gate | 採用 (= 既存) |

### 2.2 銘柄選定の優先軸 (= 順位形成に使う要素)

| # | 軸 | proxy data source | weight |
|---|---|---|---|
| 1 | **liquidity / turnover** | margin_name (= 貸借/信用) | **0.40** |
| 2 | **market cap / float** | scale_category (TOPIX 構成) | **0.25** |
| 3 | **sector trend / momentum / catalyst** | research_final_score | **0.25** |
| 4 | **entry feasibility** | scale_category (= 二次利用) | **0.10** |

→ selection_score = liquidity * 0.40 + cap * 0.25 + (research * 10) * 0.25 + feasibility * 0.10

### 2.3 三分類 (= entry / watch / excluded)

| 分類 | 条件 | 取扱 |
|---|---|---|
| **entry_candidate** | margin_name=貸借 (= liquidity_tier ≥ 5) | top 3 へ promote、Fujiwara entry 検討対象 |
| **watch_candidate** | margin_name=信用 + TOPIX Small (= 4 or 3) | top 3 外、monitor only |
| **excluded_candidate** | 信用 + scale=- OR letter-suffix 新規上場 + 非 TOPIX | entry 不可、watch も非推奨 |

### 2.4 「小型・低流動性銘柄」の定義明確化

| 条件 | 判定 |
|---|---|
| 貸借 OK + scale=- | 流動性 OK + 小型 → **entry 可** (= 流動性確認済) |
| 信用 + TOPIX Small | 流動性中 + 中小型 → **watch only** |
| 信用 + scale=- | 流動性不足 + 小型 → **excluded** ★ |
| letter-suffix + 非 TOPIX | 新規上場 + 薄商い → **excluded** ★ |

→ **excluded** = 「小型 AND 低流動性」、entry 不可

## §3 D31 candidates 再判定 (= v1.2 適用)

### 3.1 scoring (= staging F111 出力に v1.2 logic 適用)

| rank | code | name | sector | margin | scale | liq | cap | research | sel_score |
|---|---|---|---|---|---|---|---|---|---|
| 1 | **9247** | ＴＲＥホールディングス | 情報通信 | 貸借 | TOPIX Small 1 | 7 | 7 | 0.8546 | **7.39** ★ |
| 2 | **9628** | 燦ホールディングス | 情報通信 | 貸借 | TOPIX Small 2 | 6 | 6 | 0.8479 | **6.62** |
| 3 | **9130** | 共栄タンカー | 運輸・物流 | 貸借 | - | 5 | 3 | 0.8586 | **5.20** |
| 4 | 4404 | ミヨシ油脂 | 食品 | 貸借 | - | 5 | 3 | 0.8413 | 5.15 |

### 3.2 旧 v1.0 / v1.1 → v1.2 順位変化

| 銘柄 | v1.0 rank | v1.1 status | v1.2 rank | 変化理由 |
|---|---|---|---|---|
| 9130 (運輸、貸借、scale=-) | #1 (risk_yen 順) | PASS | **#3** | risk_yen 非加点で cap が低い分降格 |
| 331A0 (情通、信用、scale=-、letter-suffix) | #2 | FAIL | excluded | v1.1 で除外維持 |
| 4389 (情通、信用、scale=-) | #3 | FAIL | excluded | v1.1 で除外維持 |
| 9247 (情通、貸借、Small 1) | #4 | PASS | **#1 ★** | cap + liquidity 高で promote |
| 4317 (情通、信用、scale=-) | #5 | FAIL | excluded | v1.1 で除外維持 |
| 9628 (情通、貸借、Small 2) | n/a | PASS | **#2** | v1.1 で promote、v1.2 で #2 |

### 3.3 excluded_candidate (= 信用 + 非 TOPIX、6 銘柄)

| code | name | sector | sel_score |
|---|---|---|---|
| 4389 | プロパティデータバンク | 情報通信 | 3.59 |
| 4317 | レイ | 情報通信 | 3.58 |
| 2700 | 木徳神糧 | 商社 | 3.57 |
| 6149 | 小田原エンジニアリング | 機械 | 3.56 |
| 2981 | ランディックス | 不動産 | 3.55 |
| 331A0 | メディックス | 情報通信 | 3.19 (= letter-suffix) |
| 288A0 | ラクサス・テクノロジーズ | 情報通信 | 3.15 (= letter-suffix) |
| 339A0 | プログレス・テクノロジーズ・グループ | 情報通信 | 3.13 (= letter-suffix) |

### 3.4 watch_candidate (= 信用 + TOPIX Small、本 D31 では該当 0)

D31 staging F111 出力には margin=信用 + TOPIX Small 構成銘柄なし。
→ watch_candidate **0 件**。全 候補は entry (PASS 貸借) または excluded (FAIL 非 TOPIX 信用)。

## §4 sector 多様化への影響

| 観点 | v1.0 | v1.1 | v1.2 |
|---|---|---|---|
| #1 sector | 運輸 (9130) | 運輸 (9130) | **情報通信 (9247)** |
| #2 sector | 情報通信 (331A0) | 情報通信 (9247) | **情報通信 (9628)** |
| #3 sector | 情報通信 (4389) | 情報通信 (9628) | **運輸 (9130)** |
| 多様化 | 2 種 | 2 種 | **2 種** (= 運輸 + 情報通信) |

→ entry top 3 では依然 2 種、ただし #4 候補 4404 (食品) を含めれば 3 種多様化可能。
   sector 多様化 priority を別途 explicit constraint に入れる場合は別 wave で検討。

## §5 v1.2 morning advisory 出力規約

### 5.1 entry_candidate top 3 ordering

selection_score 降順で top 3 を提示。risk_yen は **表示のみ** (= 損切り設計の参考)、
順位形成には使わない。

### 5.2 watch_candidate / excluded_candidate

- watch: 別 section に list 化、Fujiwara が monitor only
- excluded: 簡潔に list 化 + 除外理由明示

### 5.3 朝 iSPEED 確認軸 (= v1.1 から継承)

5 軸 + 追加 3 軸:
1. daily volume / 2. turnover / 3. opening volume / 4. spread / 5. board depth
6. 寄り上昇率 / 7. ニュース・適時開示 / 8. 約定しやすさ

## §6 v1.2 advisory 適用 timing

- **即時適用** (= 本日 2026-05-15 / 連動 pilot D31 = 2026-06-25 の advisory v1.2 から)
- v1.0 / v1.1 baseline 保持 (= 設計演化の歴史的 record)
- 以降の morning advisory は v1.2 を default 形式とする

## §7 将来の F111 内部 logic 正式 commit

- 本 doc は **policy 設計**、F111 source 改修は別 wave
- F111 runner に `liquidity_tier` / `cap_tier` / `selection_score` fields 追加
- AFTER-R1 で `entry_candidate` / `watch_candidate` / `excluded_candidate` の三分類 output
- 既存 risk_within_pilot_limit は gate として維持 (= 加点ではない)

## §8 安全境界

- F111 source code 変更 0 (= 本 wave は design doc + advisory v1.2 のみ)
- DB write 0 / API 0 / token 0 / LINE 0 / commit 0 / push 0
- 既存 v1.0 / v1.1 advisory は保持 (= 削除しない)
