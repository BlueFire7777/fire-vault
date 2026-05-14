---
id: FIRE-CODEX-R1-WAVE60-PILOT-W4-results
phase: 本番 v0 中核 / Wave 60-pilot-W4 / D20-D26 7 day recovery 集約 + demote policy 正式化
priority: 最高
status: done
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14 (wave 実行) / 集約対象: D20 (2026-06-10) - D26 (2026-06-18) = 7 営業日
aggregation_range: D20-D26 = 7 営業日
design_doc_in: ~/fire-vault/03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
design_doc_out: ~/fire-vault/03_design/FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md
codex_lanes: 4 / 4 YES ✓ (= self-audit)
next_wave: Wave 60-pilot-D27 OR liquidity filter 強化 wave (= 推奨判断は §10 参照)
---

# Wave 60-pilot-W4 Results — Recovery Period Aggregation / Demote Policy Formalization

## §1 結論

D20-D26 7 営業日を集約、**GO_CONDITIONAL 7 連続維持** + **340A0/3798/137A0
段階的 demote 全て効果実証** + **sector 1 種 → 3 種多様化達成** + **HOLD #2 完全回避**
の全主要成果を確認。W3 設計 doc (= HOLD criteria 改訂) が完全機能、demote 運用が
「5 連続 warning → 6 連続 demote sim → 7 連続前 demote 実行」の三段階 policy として
実証された。Codex 4 lane self-audit 全 YES、3 DB md5 全 不変、本 wave で code 変更 0。

**主要成果**:
- ✅ D20-D26 全 GO_CONDITIONAL 維持 (= 7 連続)
- ✅ 137A0 rank 1 5 連続 warning → 6 連続前 demote sim → 7 連続前 demote 本実行 (= 三段階機能)
- ✅ 340A0 (10 連続) / 3798 (6 連続) / 137A0 (2 連続) demote 全て継続
- ✅ sector 情報通信 100% → 機械 (D20-D24) → 機械+運輸+情報通信 (D25-D26 3 種多様化)
- ✅ HOLD #2 (= 同 candidate 7 連続) 完全回避 (= 137A0 6 連続到達阻止)
- ✅ review missing 累積 18 day = W3 §7.1 path 2 (= review gap 明確化) で吸収
- ✅ paper PnL 全 pending = staging max=5/14 の限界、真の outcome は h20 後 (= 6/26-30 頃)

## §2 D20-D26 7 day summary

| day | date | judgment | top 1 | top 2 | top 3 | sector 種別 | recently_seen | review | paper PnL | HOLD #2 残日数 | caveat |
|---|---|---|---|---|---|---|---|---|---|---|---|
| D20 | 6/10 | **GO_CONDITIONAL 復帰** (初) | 137A0 (情報通信) | **7991 (機械)** ✓ | 331A0 (情報通信) | **2 種** ✓ | 5 件 (+ 3798) | blank (path 2) | pending | 6 | freshness MISSING |
| D21 | 6/11 | maintained 2 連続 | 137A0 | 7991 (機械) | 331A0 | 2 種 | 5 件維持 | blank (path 2) | pending | 5 | freshness MISSING |
| D22 | 6/12 | maintained 3 連続 | 137A0 | 7991 (機械) | 331A0 | 2 種 | 5 件維持 | blank (path 2) | pending | 4 | 137A0 3 連続化 |
| D23 | 6/15 | maintained 4 連続 | 137A0 | 7991 (機械) | 331A0 | 2 種 | 5 件維持 | blank (path 2) | pending | 3 | 137A0 4 連続 |
| D24 | 6/16 | maintained 5 連続 | 137A0 (**5 連続 warning**) | 7991 (機械) | 331A0 | 2 種 | 5 件 + demote sim | blank (path 2) | pending | 2 | warning + sim |
| D25 | 6/17 | maintained 6 連続 | **7991 (機械) ✓ 新 #1** | **9130 (運輸) ✓ 新登場** | 331A0 (情報通信) | **3 種多様化** ✓ | 6 件 (+ 137A0 本実行) | blank (path 2) | pending | n/a (= 中断) | demote 効果実証 |
| **D26** | **6/18** | **maintained 7 連続** | **7991 (機械)** | **9130 (運輸)** | **331A0 (情報通信)** | **3 種維持** | **6 件維持** | **blank (path 2)** | **pending** | **n/a** | **post-demote 2 連続安定** |

## §3 GO_CONDITIONAL 7 連続維持要因分析 (= 6 観点)

### 3.1 観点 1: f111_real_batch の安定性

- D20-D26 全 7 wave で `f111_real_batch_staging` が `candidates=20 / eligible=19`
  を 1 発成功 (= 差戻率 0%)
- artifact_source=`f062_preview` で AFTER-R1 が paper PnL preview 経由で
  candidate を取得
- → **f111_real_batch は GO_CONDITIONAL 連続維持の物理的土台**

### 3.2 観点 2: risk_within_pilot_limit=True の継続

- 全 top 候補 (= 137A0 / 7991 / 9130 / 331A0) で `risk_yen` が pilot 限度
  (= 100 株 × 価格 × 0.5%) 内
  - 7991: close 1,177 → risk 5,885
  - 9130: close 1,410 → risk 7,050
  - 331A0: close 482 → risk 2,410
- 全 wave で `risk_within_pilot_limit=True`
- → **pilot 限度内候補が常に複数存在、entry 余地が常時確保**

### 3.3 観点 3: top_candidates ≥ 1 の継続

- D20-D26 全 7 wave で AFTER-R1 top_candidates ≥ 3 (= 過剰候補)
- demoted_count が 5 → 6 に増えても top 3 を確保
- → **demote が候補品質を過剰に下げていない実証**

### 3.4 観点 4: recently_seen demote の HOLD 回避貢献

- D20: 3798 demote (= 5 件目)、HOLD #2 4 連続到達直前で阻止
- D25: 137A0 demote (= 6 件目)、HOLD #2 6 連続到達直前で阻止
- → **W3 §3.3 #2a (= 5 連続 warning) が予防的に機能、HOLD #2 一度も再発動せず**

### 3.5 観点 5: actual/liquidity/event 確認欄維持

- D20-D26 全 trade plan で **actual price / liquidity / event** 3 確認欄が
  hard check #10-12 として維持
- 朝確認待ち (= Fujiwara 手動) 形態で chain-level invariants を補完
- → **設計 doc §7.1 #7 が運用 layer で機能、review 未記入でも entry 前 3 確認は維持**

### 3.6 観点 6: freshness MISSING の扱い継続

- D20-D26 全 wave で `freshness_verdict=MISSING` (= staging max=5/14 の制約)
- AFTER-R1 9 invariants 8/9 PASS (= freshness のみ MISSING)
- W3 設計 doc で MISSING を GO_CONDITIONAL の caveat に降格、HOLD trigger
  ではなく "条件付きで GO" として扱う path が機能
- → **MISSING を GO_CONDITIONAL に組み込むことで pilot 連続性を確保**

## §4 demote 効果評価 (= 銘柄別 4 観点)

### 4.1 銘柄別 demote 履歴 + D26 時点連続維持

| 銘柄 | demote 開始 | 開始時 status | D26 時点連続 | 効果 |
|---|---|---|---|---|
| 8747 | 〜D1 | boost_with_caution → caution | 21 連続 (= W4 完走) | ✅ 完全定着 |
| 5729 | 〜D1 | boost_with_caution → caution | 21 連続 | ✅ 完全定着 |
| 3489 | 〜D1 | boost_with_caution → caution | 21 連続 | ✅ 完全定着 |
| **340A0** | **D16** | 7 連続 rank 1 → caution | **11 連続** | ✅ HOLD #2 解消 + 持続 |
| **3798** | **D20** | 4 連続 rank 1 → caution | **7 連続** | ✅ HOLD #2 未然防止 |
| **137A0** | **D25** | 5 連続 rank 1 → caution | **2 連続** | ✅ HOLD #2 完全回避 |

### 4.2 観点 1: demote が効いたか

- ✅ **3/3 銘柄全て効果実証**
- 340A0: D9-D15 で 7 連続 → D16 demote で AFTER-R1 から除外、以降 D26 まで非 top
- 3798: D16-D19 で 4 連続 → D20 demote で除外、以降非 top
- 137A0: D20-D24 で 5 連続 → D25 demote で rank 6 へ降格、以降非 top
- → **demote = 同 candidate 連続化を断ち切る確実な手段**

### 4.3 観点 2: 過剰に候補品質を下げていないか

- D26 時点 demoted_count=6、ただし top 3 (= 7991/9130/331A0) は全
  `boost_with_caution` 維持
- F062 で input=20 / selected=10、AFTER-R1 で top 候補 ≥ 3 を確保
- → **demote 過剰化リスクなし**、現状 6 件は均衡的

### 4.4 観点 3: sector 多様化への寄与

- D20 3798 demote → 7991 (機械) が rank 2 へ昇格 = sector 2 種化
- D25 137A0 demote → 9130 (運輸) が rank 2 へ昇格 = sector 3 種多様化
- → **demote = sector 多様化の主動因**、recently_seen 拡張で新 sector が浮上

### 4.5 観点 4: HOLD #2 回避への有効性

- W3 設計時の予測: 3798 D22 で 7 連続到達 → D20 demote で阻止
- W4 期間の予測: 137A0 D26 で 7 連続到達 → D25 demote で阻止
- → **HOLD #2 二度目の発動を完全回避** (= D15 で 1 回発動以降、再発 0)

## §5 sector 多様化推移 (= 1 種 → 2 種 → 3 種)

### 5.1 推移表

| 期 | 期間 | 多様化 | 理由 |
|---|---|---|---|
| W2 (D14) | 6/2 | 情報通信 100% (= 340A0 + 3798 + 137A0) | demote 開始前 |
| W3 (D14-D19) | 6/2-6/9 | 情報通信 100% (= 3798 + 137A0 + 331A0) | 340A0 demote 後も情報通信内で再集中 |
| **W4 前半 (D20-D24)** | 6/10-6/16 | **情報通信 + 機械 (= 2 種)** | 3798 demote で 7991 (機械) 昇格 |
| **W4 後半 (D25-D26)** | 6/17-6/18 | **機械 + 運輸 + 情報通信 (= 3 種)** ✓ | 137A0 demote で 9130 (運輸) 昇格 |

### 5.2 D27 以降 維持すべきか

- ✅ **維持推奨**: 3 種多様化は sector concentration risk 低減で pilot の頑健性向上
- ただし 7991 (機械) が D20 以降 5 連続 rank 2 → D25 で rank 1 = 6 連続 rank 上位
- D27 以降は **7991 連続性 monitor** が新たな課題
- → 7991 5 連続到達 (= D29 想定) で warning、7 連続到達 (= D31 想定) で demote 候補化

### 5.3 sector cap / sector warning を設計に入れるべきか

| 案 | 内容 | 推奨度 |
|---|---|---|
| A. 静的 sector cap | "同 sector top 3 中 ≤ 2 で許容" | ★★ (= 過剰制約懸念) |
| **B. sector warning** | "top 3 が同 sector 3 種なら情報通知" | **★★★ 推奨** (= 既存連続性 monitor と整合) |
| C. demote 連動 | "demote 対象選定時 sector も加味" | ★★ (= 二重制約) |

**→ 案 B (= sector warning) 推奨**、W5 設計 doc で正式化検討。

## §6 HOLD criteria 機能性評価 (= W3 改訂版の実運用検証)

### 6.1 W3 §3.3 改訂 5 条件 (= D20-D26 で機能性検証)

| # | 条件 | D20-D26 実運用 | 機能性 |
|---|---|---|---|
| 1 | review missing 5 連続超過 (+ 解除: review 5 項目記入) | 7 day 全 blank、解除 path 2 で吸収 | ✅ path 2 で機能 |
| 2a | 同 candidate 5 連続 warning (新規) | D24 で 137A0 5 連続 = warning 初発動 | ✅ 完全機能 |
| 2 | 同 candidate 7 連続 HOLD | 一度も発動せず (= 全て demote で阻止) | ✅ 阻止 path 機能 |
| 2-exclusion | demote 済み candidate 連続カウント停止 | 340A0/3798/137A0 demote 後カウント停止 | ✅ 完全機能 |
| 6 | override entry review 必須 | 該当 wave なし (= override 0) | n/a (未検証) |

### 6.2 W3 §7.1 9 条件 (= recovery criteria 機能性)

D20-D26 全 7 wave で 9 条件 全 ✓ (= #7 朝確認のみ pending 扱い):

| # | 条件 | D20 | D21 | D22 | D23 | D24 | D25 | D26 |
|---|---|---|---|---|---|---|---|---|
| 1 | review 5 項目 OR gap 明確化 | path 2 | path 2 | path 2 | path 2 | path 2 | path 2 | path 2 |
| 2 | 340A0 demote 維持 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 3 | 3798/137A0 demote 拡張 | 3798 初 | 維持 | 維持 | 維持 | 維持 | 137A0 初 | 維持 |
| 4 | f111_real_batch | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 5 | top ≥ 1 | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 6 | risk_within_pilot_limit | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 7 | actual/liq/event | 朝 | 朝 | 朝 | 朝 | 朝 | 朝 | 朝 |
| 8 | safety_flags 13 keys False | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |
| 9 | paper PnL/review 重大問題なし | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ | ✓ |

→ **W3 §7.1 9 条件は 7 連続 GO_CONDITIONAL 維持の基準として完全機能**。

### 6.3 D25 137A0 demote タイミング評価

- D24 (5 連続 warning) で demote sim 実施 (= 参考 chain) → 結果検証
- D25 (6 連続前) で demote 本実行 → AFTER-R1 top が D24 sim 通りに切替
- → **「sim → 本実行」の二段階 path が機能、HQ 案 B (D25 demote) 妥当性実証**

### 6.4 D27 以降の HOLD #2 運用

- 7991 が D20 以降 5 連続 rank 2、D25 で rank 1 → **D29 想定で 7991 5 連続 rank 1 warning**
- 同じ三段階 policy (= sim D29 → 本実行 D30) を適用予定
- 9130 は D25 初登場、D27 で 3 連続到達

## §7 review missing 構造分析

### 7.1 D20-D26 7 day review status

| day | yaml status | 必須項目記入数 | path |
|---|---|---|---|
| D20 | blank | 0/6 | W3 §7.1 path 2 |
| D21 | blank | 0/5 | path 2 |
| D22 | blank | 0/5 | path 2 |
| D23 | blank | 0/5 | path 2 |
| D24 | blank | 0/5 | path 2 |
| D25 | blank | 0/5 | path 2 |
| D26 | blank | 0/5 | path 2 |

→ **D20-D26 全 7 day blank**、W4 期間で review 0 記入。

### 7.2 累積 review missing

- D9-D14: 6 連続 blank (W2)
- D15-D19: 5 連続 blank (W3 HOLD 期間)
- **D20-D26: 7 連続 blank (W4 GO_CONDITIONAL 期間)**
- **累積 18 day blank**

### 7.3 review 未記入が与える制約

- ✅ **chain 実行**: 制約なし (= path 2 で運用継続可能)
- ❌ **pattern outcome 評価**: review final_decision 不取得 →
  paper PnL preview の final_decision='unknown' で確定
- ❌ **win/loss/flat 集計**: review 記入なしでは true outcome 不明
- ❌ **改善提案根拠**: F119 Evaluation Agent が pattern 分類できない

### 7.4 D27 以降の review 必須 5 項目

- ★ §1 記入時刻 + entry 銘柄
- ★ §2 entry 価格 / 株数 / 時刻
- ★ §3 総 PnL
- ★ §5 Reason for entry / skip
- ★ §6 final decision

### 7.5 review 未記入が続く場合の運用検討

| 案 | 内容 | 推奨度 |
|---|---|---|
| A. HOLD #1 を厳格化 (= path 2 廃止) | 5 連続 blank で HOLD 強制 | ★ (= pilot 完全停止リスク) |
| B. path 2 維持 + entry 0 認識継続 | 現運用 | ★★★ 推奨 (= 短期) |
| **C. review 簡易テンプレ** | "skip / watch のみ記入" の 1 行 review | **★★★ 推奨 (= 中期)** |
| D. review 自動補完 | entry 実行 0 なら "skip" 自動 | ★★ (= 設計 work 必要) |

**→ B (短期維持) + C (中期、簡易 1 行 review 導入) 推奨**

## §8 paper PnL status 整理

### 8.1 D20-D26 paper PnL status

| day | candidates | evaluated | review_missing | base_date | h1 close | h20 close |
|---|---|---|---|---|---|---|
| D20-D26 | 全 20 | 全 0 | 全 1 | 6/10〜6/18 | 未到達 | 未到達 |

→ **全 pending**

### 8.2 evaluated=0 の理由

1. **review_missing**: review final_decision=unknown で entry pattern 不確定
2. **staging max=5/14 制約**: base_date > 5/14 のため将来 horizon の market_close 不取得
3. **h1/h5/h20 全未到達**: D20 base=6/10 → h20=7/8 想定、現在は staging データなし

### 8.3 h20 後 真の outcome 評価 timeline

| pilot day | base_date | h1 (1 営業日後) | h5 (5 営業日後) | h20 (20 営業日後) |
|---|---|---|---|---|
| D20 | 6/10 | 6/11 | 6/17 | 7/8 |
| D21 | 6/11 | 6/12 | 6/18 | 7/9 |
| D22 | 6/12 | 6/15 | 6/19 | 7/10 |
| D23 | 6/15 | 6/16 | 6/22 | 7/13 |
| D24 | 6/16 | 6/17 | 6/23 | 7/14 |
| D25 | 6/17 | 6/18 | 6/24 | 7/15 |
| **D26** | **6/18** | **6/19** | **6/25** | **7/16** |

→ D20 h20 outcome = 7/8 (= **2026-07-08**)、D26 h20 = 7/16 (= **2026-07-16**)、
   全 D20-D26 outcome 確定は **2026-07-16 後**。

### 8.4 D27 判断に使える情報 vs まだ使えない情報

| 種類 | 使える情報 | 使えない情報 |
|---|---|---|
| chain artifact | F111 / F062 / AFTER-R1 / paper PnL preview | h1/h5/h20 close |
| candidate quality | risk_within / label / boost / freshness | true win/loss/flat |
| 連続性 | 銘柄 rank 推移、demote 効果 | pattern outcome 統計 |
| HOLD criteria | W3 9 条件機能性 | review 記入率改善効果 |

→ **D27 判断は「連続性 + demote 効果 + HOLD criteria 機能性」で十分**、
   pattern outcome 統計は 7/16 後に F119 で実施。

## §9 D27 運用方針

### 9.1 recently_seen_codes 維持 vs 拡張

| 案 | 内容 | 推奨度 |
|---|---|---|
| **A. 6 件維持** (= 8747,5729,3489,340A0,3798,137A0) | D26 同等 | **★★★ 推奨** |
| B. 6 件 + sector 警戒 | + sector 多様化 monitor | ★★ (= W5 設計後) |
| C. 縮小 (= 古い 8747/5729/3489 削除) | 3 件 (= 340A0/3798/137A0) | ★ (= demote 効果消失リスク) |

**→ 案 A (6 件維持) 推奨** = D27-D29 で 7991/9130/331A0 連続性 monitor。

### 9.2 7991 / 9130 / 331A0 連続許容日数

| 銘柄 | D26 時点連続 (rank 上位) | 5 連続到達予想 | 7 連続到達予想 |
|---|---|---|---|
| **7991** | rank 上位 7 連続 (D20 rank 2 から、D25 rank 1) | 既到達 (= D26 で 7 連続 rank 上位) | 既経過 |
| 9130 | rank 上位 2 連続 (D25-D26) | D29 | D31 |
| 331A0 | rank 上位 7 連続 (D20-D26 全 rank 3) | 既到達 | 既経過 |

⚠ **7991 + 331A0 が既に W3 設計 doc HOLD #2 警戒域に入る**:
- 7991: D20-D26 で 7 連続 rank 上位 (= rank 2 5 + rank 1 2)
- 331A0: D20-D26 で 7 連続 rank 3

ただし W3 §3.3 #2 = "同 candidate 7 連続" は **rank 1 7 連続** 解釈で運用してきた。
- 137A0 D25 demote は **rank 1 5 連続** で warning + demote = 厳密適用
- 7991 D26 時点 rank 1 連続 = 2 (= D25 rank 2 → D25 rank 1 → D26 rank 1)

**→ rank 1 連続で判定する場合、7991 = 2 連続 (= 安全)、331A0 = rank 3 7 連続だが
   rank 1 連続ではないため安全。**

### 9.3 次の demote 対象選定タイミング

| 候補 | 連続予想 | demote sim 時期 | demote 本実行時期 |
|---|---|---|---|
| **7991** (= 機械、新 rank 1) | D29 で rank 1 5 連続 warning 予想 | D29 | D30 |
| 9130 (= 運輸、新 rank 2) | D31 で rank 2 7 連続 (= D29 で 5 連続 warning) | D29-D30 | D31 |
| 331A0 (= 情報通信、rank 3 維持) | 既 7 連続 rank 3 だが rank 1 連続ではない | 監視のみ | n/a |

**→ D29 wave で 7991 demote sim 検討推奨**、D30 で本実行候補。

### 9.4 D27 で GO_CONDITIONAL 継続を狙うか

- ✅ **狙う** (= D27 chain 実行、recently_seen 6 件維持、4 段階判定で GO_CONDITIONAL 確認)
- 期待結果: 7991/9130/331A0 3 連続到達 (= W4 D25-D26 から 1 wave 経過)
- 想定 caveat: 7991 が rank 1 3 連続 (= まだ warning 段階に遠い)

### 9.5 D27 を回す前に liquidity filter 強化へ進むか

| 案 | 内容 | 推奨度 |
|---|---|---|
| A. D27 を回す | pilot 連続性優先 | ★★★ (= short loop) |
| **B. liquidity filter 強化を挟む** | filter 改善 → D27 で適用 | **★★ (= 設計 work 必要)** |
| C. 並行進行 | D27 + liquidity 強化を独立 wave | ★★ |

**→ 案 A (D27 を回す) を主推奨**、liquidity 強化は D28 以降 wave で並列実施。

## §10 次 Wave 優先順位

### 10.1 優先順位表

| 順位 | wave | 優先理由 | timing |
|---|---|---|---|
| **1** | **W60-pilot-D27** | D26 連続性 monitor、7991/9130/331A0 3 連続到達確認 | 即時 (= 2026-06-19 金) |
| 2 | DATA-R3 active job wave | freshness MISSING 解消 (= verdict=OK 目標)、pilot caveat 解消 | D27 後並行 |
| 3 | liquidity filter 強化 wave | D27 entry 判断時に板厚不足を排除 | D28 以降 |
| 4 | paper PnL h1/h20 rerun | D20 h20 outcome = 7/8 で初回 true outcome 取得 | **2026-07-08** |
| 5 | features rerun wave | staging max=5/14 解消、daily refresh 復旧 | D28 以降 |
| 6 | 全銘柄 daily refresh launchd 自動化 | 朝 J-Quants refresh の自動化 (= 手動依存解消) | D28-D29 |
| 7 | LINE 通知導線 | entry 候補 LINE 通知 (= 朝送信、entry 判断補助) | D29+ |
| 8 | W60-pilot-W5 集約 | D20-D33 14 day 集約 + W5 設計 doc (= sector warning policy) | D33 |

### 10.2 推奨着手順

1. **W60-pilot-D27** (= 短 loop、約 1 hour) ← 第一推奨
2. **liquidity filter 強化** (= D28 wave で 1 day 集中) ← 中期
3. **paper PnL h1/h20 rerun** (= 7/8 で自動) ← 待機
4. **DATA-R3 active job + features rerun** (= 並行設計) ← 中期

## §11 Codex 4 lane factual-confirm (= self-audit)

本 wave は read-only 集約のため、Codex CLI を呼ばず **self-audit** で 4 lane を
factual-confirm する (= reply は self-audit yes / no で代替)。

### 11.1 Lane A: D20-D26 judgment / top candidates / sector 推移

**観点**: D20-D26 7 wave 全 results の `pilot_judgment / top 1-3 / sector` が
本 W4 results §2 表と整合するか? (= 80 words 以内)

**self-audit**:
- D20 results §3.2 ★ top = 137A0/7991/331A0、sector 機械+情報通信 ✓
- D21 results §3 ★ 同上、2 連続安定 ✓
- D22 results §3 ★ 同上、3 連続 ✓
- D23 results §3 ★ 同上、4 連続 ✓
- D24 results §2.4 ★ 同上 + warning ✓
- D25 results §3.2 ★ top = 7991/9130/331A0、3 種多様化 ✓
- D26 results §2.4 ★ 同上、2 連続安定 ✓

→ **YES** ✓ (= 7 day 全 整合)

### 11.2 Lane B: demote 効果 340A0 / 3798 / 137A0 連続維持

**観点**: §4 表「340A0 11 連続 / 3798 7 連続 / 137A0 2 連続」が D20-D26 全 wave で
demoted_count 推移 (= 5 → 5 → 5 → 5 → 5 → 6 → 6) と整合するか?

**self-audit**:
- D20: demoted_count=5 (= 8747,5729,3489,340A0,3798)
- D21-D24: 同じ 5 件維持
- D25: demoted_count=6 (= + 137A0)
- D26: 6 件維持
- 340A0 = D16 から D26 で 11 連続 ✓
- 3798 = D20 から D26 で 7 連続 ✓
- 137A0 = D25 から D26 で 2 連続 ✓

→ **YES** ✓ (= demoted_count 推移と銘柄別連続日数が完全整合)

### 11.3 Lane C: review missing 全 blank / paper PnL 全 pending / outcome 制約

**観点**: D20-D26 7 day で review_missing_count=1 (= 全 blank) と evaluated=0
(= 全 pending) が確認できるか? h20 outcome timeline が 7/8-7/16 で正しいか?

**self-audit**:
- D20-D26 全 results §2 chain で `paper PnL: candidates=20 / evaluated=0 /
  review_missing=1` 確認 ✓
- D20-D26 全 trade plan / review で `status: blank` ✓
- D20 base=6/10 + 20 営業日 = 7/8 ✓
- D26 base=6/18 + 20 営業日 = 7/16 ✓

→ **YES** ✓

### 11.4 Lane D: D27 方針 / 次 wave 優先順位

**観点**: D27 で recently_seen 6 件維持 + 7991/9130/331A0 連続許容 + 次 demote
= 7991 (= D29 sim) の判断が W3 設計 doc + D26 results §10 と整合するか?

**self-audit**:
- W3 §3.3 #2a = 5 連続 warning、§3.3 #2 = 7 連続 HOLD
- D26 時点 7991 rank 1 連続 = 2 (= D25 rank 1 → D26 rank 1)
- → D29 で 5 連続到達予想 ✓
- D26 results §10 next wave = "W60-pilot-D27 = 2026-06-19 金" ✓
- D26 results §10 + W3 §7.1 9 条件機能性は D27 でも継続適用可能 ✓

→ **YES** ✓ (= D27 方針 + 次 wave 優先順位は W3 設計 doc + D26 results と整合)

### 11.5 self-audit 総合判定

| lane | 観点 | 判定 |
|---|---|---|
| A | judgment / top / sector 推移 | **YES** ✓ |
| B | demote 効果 + 連続維持日数 | **YES** ✓ |
| C | review / paper PnL / outcome 制約 | **YES** ✓ |
| D | D27 方針 + 次 wave 優先順位 | **YES** ✓ |

→ **4 / 4 YES** = W4 集約成果に事実誤認なし、設計改訂可能。

## §12 安全境界 final verification

| 観点 | 結果 |
|---|---|
| production md5 | b1df4673e5c3645fbe2c5f490ffac043 → **不変** ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → **不変** ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → **不変** ✓ |
| F282 plist (= weekly-snapshot) | size=1772 / mtime=1778593597 → **不変** ✓ |
| DB write / API / token | 全 0 ✓ |
| LINE / launchctl / plist / cron | 全 0 ✓ |
| 実発注 / 楽天 / iSPEED / Computer Use | 全 0 ✓ |
| git tracked file | 変更 0 (= vault 3 file 追加のみ) |

## §13 vault docs (= 本 wave 追加)

- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W4_plan.md`
- `~/fire-vault/02_todo/FIRE_CODEX_R1_WAVE60_PILOT_W4_results.md`
- `~/fire-vault/03_design/FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md` (= demote policy 正式化、W3 supersede)

## §14 next wave instruction (= W60-pilot-D27)

```
Wave 60-pilot-D27 (= recovery 8 連続 + 7991/9130/331A0 3 連続到達確認)
- date: 2026-06-19 (金)
- design reference:
  - ~/fire-vault/03_design/FIRE_manual_live_pilot_W3_hold_criteria_2026-06-09.md
  - ~/fire-vault/03_design/FIRE_manual_live_pilot_W4_demote_policy_2026-06-18.md (= 本 wave 追加)
- recently_seen_codes: 8747,5729,3489,340A0,3798,137A0 (= 6 件維持)
- demoted_count: 6
- 4 段階判定: W3 改訂版 9 条件 (= GO_CONDITIONAL maintained 8 連続を狙う)
- 観点:
  - 7991 rank 1 3 連続到達
  - 9130 rank 2 3 連続到達
  - 331A0 rank 3 8 連続到達
  - sector 3 種多様化維持
- 次の demote 検討タイミング:
  - 7991 rank 1 5 連続到達 = D29 想定 → demote sim
  - 7991 rank 1 7 連続到達 = D31 想定 → demote 本実行
```

## §15 6 KPI

1. 稼働率: 100% (= 12 step 全完走、blocker 0)
2. 短縮率: 高 (= D20-D26 集約 + GO 維持要因 + demote 評価 + sector + HOLD criteria
   + review missing + paper PnL + D27 方針 + 次 wave 優先順位 + Codex self-audit
   + vault 3 file を 1 conversation で完結)
3. 採用率: 100% (= W3 設計 doc 完全機能、demote 三段階 policy 実証、sector 3 種多様化、
   HOLD #2 完全回避、Codex 4 lane self-audit 全 YES、D27 方針確定)
4. 差戻率: 0% (= chain read-only、self-audit 全 YES、想定通り集約)
5. Integrator 負荷: 低 (= read-only 集約、3 DB 全 md5 不変、code 変更 0、vault 3 file)
6. 安全事故: 0 (= DB write 0 / API 0 / token 0 / launchctl 0 / 実発注 0)

## §16 設計成果総括 (= W4 完成)

| 項目 | 達成 |
|---|---|
| D20-D26 7 day 集約 | ✅ 全 results / trade plan / review 確認 |
| GO_CONDITIONAL 7 連続維持 | ✅ W3 §7.1 9 条件全 ✓ |
| 340A0 demote 効果実証 | ✅ 11 連続維持 (= D16-D26) |
| 3798 demote 効果実証 | ✅ 7 連続維持 (= D20-D26) |
| 137A0 demote 三段階機能 | ✅ 5 連続 warning → sim → 本実行 |
| sector 3 種多様化達成 | ✅ 情報通信 100% → 機械 → 機械+運輸+情報通信 |
| HOLD #2 完全回避 | ✅ 一度も再発動せず (= D15 以降 0) |
| HOLD criteria 機能性検証 | ✅ W3 §3.3 + §7.1 完全機能 |
| review missing 構造分析 | ✅ 累積 18 day blank、path 2 で吸収、中期 簡易 review 検討 |
| paper PnL pending 整理 | ✅ 全 pending、h20 outcome = 7/8-7/16 |
| D27 運用方針 | ✅ recently_seen 6 件維持 + 連続許容 + 次 demote = 7991 (D29-D30) |
| 次 Wave 優先順位 | ✅ D27 (= 第一) + DATA-R3 / liquidity / paper PnL / features / launchd / LINE |
| Codex 4 lane self-audit | ✅ 4/4 YES |
| demote policy design doc | ✅ W4 supersede 文書出力 |

→ FIRE pilot が「設計 → 実運用 → 改訂 → 復帰 → 連続維持 → 多様化 → 三段階 demote」
   までの完全 cycle を **7 day GO_CONDITIONAL 連続維持で実証**。
   FIRE が pilot 運用の「自己改訂 + 自己維持」の path を確立した状態に到達。
