---
id: FIRE-design-liquidity-gate-hotfix-2026-05-15
phase: 本番 v0 中核 / liquidity gate hotfix v1.0
priority: 最高
status: design-final
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
trigger_event: "331A0 (= 信用 + scale=- + letter-suffix 新規上場) が risk_yen 低だけで entry top3 入り = 流動性リスク隠蔽"
related_pilot_day: D31 (= 2026-06-25 木)
supersedes: なし (= 新規設計)
---

# FIRE Liquidity Gate Hotfix v1.0

## §1 問題

D31 advisory v1.0 では:
- 331A0 メディックス (= 信用 + scale=- + letter-suffix 新規上場、薄商い疑い) が entry top 3 (= #2) に入っていた
- 「risk_yen 低 = 低リスク」と扱われていたが、実態は **流動性リスク高**
- 4389 / 4317 / 2700 / 6149 等も同類

→ pilot entry の前提が崩れる (= 寄り付きで指値が約定しない / 滑る / スプレッド広い)

## §2 hotfix design (= F111 出力後の追加 gate)

### 2.1 評価軸 (= staging 利用可能な proxy)

| 軸 | データ source | role |
|---|---|---|
| margin_name | F111 candidate field | 貸借 / 信用 / なし |
| scale_category | F111 candidate field | TOPIX 構成 (Core30/Large70/Mid400/Small 1/Small 2) or "-" |
| code format | F111 code (= ticker) | letter-suffix (= XXXAY) = 新規上場 (2025+ IPO) |

### 2.2 朝 manual confirm 軸 (= staging 不取得、iSPEED で Fujiwara 確認)

| 軸 | 確認方法 | 基準 |
|---|---|---|
| daily volume | iSPEED 銘柄詳細 | 過去 20 営業日平均 ≥ 10 万株 (= 推奨) |
| turnover (= 売買代金) | iSPEED | ≥ 1 億円 (= 推奨) |
| opening volume | iSPEED 寄り後 5 分 | ≥ 1,000 株 |
| spread (= 板気配差) | iSPEED 板 | ≤ 5 円 (小型は ≤ 2 円) |
| board depth | iSPEED 5 本値 | best bid/ask ≥ 100 株 |

### 2.3 liquidity_status 判定 logic

```
if margin_name == '貸借':
    # 制度信用銘柄 = 貸借取引可能 = 流動性十分の判定基準
    return PASS
elif margin_name == '信用':
    if scale_category in ('TOPIX Core30','Large70','Mid400','Small 1','Small 2'):
        return WARN   # TOPIX 構成、ただし制度信用未満
    elif is_letter_suffix_code(code):
        return FAIL   # 新規上場 + 信用 + 非 TOPIX = 薄商い疑い
    else:
        return FAIL   # 信用 + 非 TOPIX = 制度信用未到達、低流動性疑い
else:
    return WARN
```

### 2.4 候補分離

| 分類 | 条件 | 取扱 |
|---|---|---|
| **entry_candidate** | liquidity_status = PASS | top 3 に入れる、Fujiwara が iSPEED で板確認後 entry 可 |
| **watch_candidate** | liquidity_status in (WARN, FAIL) | top 3 から除外、ただし monitor 対象として記録 |
| (WARN は条件付き) | 朝 iSPEED で板 + 出来高 + spread 確認、PASS 基準満たせば entry 可 | (= 原則見送り、Fujiwara が明示判断時のみ) |
| (FAIL は原則 entry 不可) | 朝確認で例外的に基準満たした場合のみ Fujiwara が明示判断 | (= 推奨せず) |

## §3 D31 適用結果 (= F111 staging output 由来、検証済)

### 3.1 D31 旧 top 5 → liquidity_status

| rank | code | name | margin | scale | letter-suffix? | status | 判定理由 |
|---|---|---|---|---|---|---|---|
| 1 | 9130 共栄タンカー | 運輸 | 貸借 | - | no | **PASS** | 貸借 = 制度信用 |
| 2 | 331A0 メディックス | 情報通信 | 信用 | - | YES | **FAIL** ★ | 信用 + scale=- + letter-suffix |
| 3 | 4389 プロパティデータバンク | 情報通信 | 信用 | - | no | **FAIL** | 信用 + scale=- |
| 4 | 9247 ＴＲＥホールディングス | 情報通信 | 貸借 | Small 1 | no | **PASS** | 貸借 + TOPIX |
| 5 | 4317 レイ | 情報通信 | 信用 | - | no | **FAIL** | 信用 + scale=- |

→ 331A0 (= user 明示の問題銘柄) は FAIL 判定で entry 除外 ✓

### 3.2 D31 補充候補 (= rank 6-10 から PASS 抽出)

| rank | code | name | sector | margin | scale | status |
|---|---|---|---|---|---|---|
| 6 | 9628 燦ホールディングス | 情報通信 | 貸借 | Small 2 | **PASS** |
| 7 | 2700 木徳神糧 | 商社 | 信用 | - | FAIL |
| 8 | 6149 小田原エンジニアリング | 機械 | 信用 | - | FAIL |
| 9 | 288A0 ラクサス・テクノロジーズ | 情報通信 | 信用 | - | FAIL (letter-suffix) |
| 10 | 4404 ミヨシ油脂 | 食品 | 貸借 | - | **PASS** |

### 3.3 hotfix 適用後の D31 entry_candidate top 3 (= PASS only)

| # | code | name | sector | margin | scale | close | risk_yen |
|---|---|---|---|---|---|---|---|
| **1** | **9130** | 共栄タンカー | **運輸・物流** | 貸借 | - | 1,410 | 7,050 |
| **2** | **9247** | ＴＲＥホールディングス | **情報通信** | 貸借 | TOPIX Small 1 | 1,612 | 8,060 |
| **3** | **9628** | 燦ホールディングス | **情報通信** | 貸借 | TOPIX Small 2 | (要確認) | (要確認) |

(参考) #4: 4404 ミヨシ油脂 (食品、貸借) — 食品 sector で多様化候補

### 3.4 watch_candidate (= 原則 entry 不可、monitor only)

- 331A0 メディックス (FAIL: letter-suffix 新規上場 + 信用 + 非 TOPIX)
- 4389 プロパティデータバンク (FAIL: 信用 + 非 TOPIX)
- 4317 レイ (FAIL)
- 2700 木徳神糧 (FAIL)
- 6149 小田原エンジニアリング (FAIL)
- 288A0 ラクサス (FAIL: letter-suffix)

## §4 sector 多様化への影響

| 観点 | hotfix 適用前 | hotfix 適用後 |
|---|---|---|
| #1 sector | 運輸 (9130) | 運輸 (9130、変わらず) |
| #2 sector | 情報通信 (331A0) | **情報通信 (9247)** = sector 同じ、銘柄交代 |
| #3 sector | 情報通信 (4389) | **情報通信 (9628)** = sector 同じ、銘柄交代 |
| 多様化 | 運輸+情報通信 (2 種) | 運輸+情報通信 (2 種、変わらず) |

→ sector 多様化は 2 種維持、ただし全 entry 候補が制度信用 (= 流動性確認済) になる

### 4.1 #4 (= 4404 ミヨシ油脂、食品) を含めた場合

- 食品 sector 加わると 3 種多様化 (= 運輸 + 情報通信 + 食品)
- D32 以降の運用で食品 sector を entry 候補に含める検討余地

## §5 朝 iSPEED で必ず確認する流動性基準 (= 各候補について)

### 5.1 推奨基準

| 軸 | PASS 基準 | WARN 基準 | FAIL 基準 |
|---|---|---|---|
| daily volume (= 20 day 平均) | ≥ 10 万株 | 5-10 万株 | < 5 万株 |
| turnover (= 売買代金) | ≥ 1 億円 | 5,000 万-1 億円 | < 5,000 万円 |
| opening volume (= 寄り 5 分) | ≥ 1,000 株 | 300-1,000 株 | < 300 株 |
| spread (= 板気配差) | ≤ 5 円 (小型 ≤ 2 円) | 5-10 円 | > 10 円 |
| board depth (= best bid/ask) | ≥ 100 株 | 30-100 株 | < 30 株 |

### 5.2 entry 判断 logic (= Fujiwara 朝寄り前後)

1. iSPEED で daily volume + turnover を確認 → 1 軸でも FAIL なら **見送り**
2. 寄り 5 分後の opening volume を確認 → FAIL なら **見送り**
3. 板を見て spread + depth を確認 → 1 軸でも FAIL なら **見送り**
4. 全 PASS なら entry 検討、指値で発注 (= 滑り防止)
5. WARN は条件付き (= 該当軸を他の軸でカバーできるか吟味)

## §6 hotfix 適用 timing

- **即時適用** (= 2026-05-15 / D31 advisory v1.1 から)
- 既存 D31 advisory v1.0 (= 2026-06-25_morning_advisory_summary.md) は **歴史的 record 保持**
- 新 v1.1 (= 2026-06-25_morning_advisory_summary_v1.1.md) を併存
- 以降の morning advisory は全て liquidity gate 適用必須

## §7 将来の正式実装

### 7.1 F111-real-batch-staging 改修案

```python
def evaluate_liquidity_status(candidate):
    margin = candidate.get('margin_name')
    scale = candidate.get('scale_category')
    code = candidate.get('code')
    is_letter_suffix = re.match(r'^[0-9]{3}[A-Z][0-9]$', code) is not None
    has_topix = scale and scale != '-'
    if margin == '貸借':
        return 'PASS'
    elif margin == '信用':
        if has_topix:
            return 'WARN'
        return 'FAIL'
    return 'WARN'
```

### 7.2 F111 candidate に liquidity_status field を追加

- AFTER-R1 で `entry_candidate` / `watch_candidate` 分離
- morning advisory CLI で entry_candidate のみ top 3 として promote
- watch_candidate は monitor section に分離

### 7.3 W5+ で正式 commit 化検討

- 本 hotfix は緊急対応 (= design doc + advisory v1.1)
- 設計 review 後、F111 内部 logic への正式 commit を別 wave で実施

## §8 安全境界

- F111 runner code 変更 0 (= 本 wave は design doc + v1.1 advisory のみ)
- DB write 0 / API 0 / token 0 / LINE 0 / commit 0 / push 0
- 既存 advisory v1.0 (= D31 baseline record) 保持
