# FIRE F111-W2C-RE-REVIEW-FRESH (2026-05-18)

doc_id: FIRE-F111-W2C-RE-REVIEW-FRESH-2026-05-18
status: 設計レビュー完了 / fresh data 適用結果記録 / 実装変更 0
HQ marker: HQ_APPROVE_W2C_RE_REVIEW_DOC_COMMIT_PUSH
as_of_date: 2026-05-18
related:
- [[F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17]] (= 設計 vault f45814c)
- [[F062_OPS_SUMMARY_FRESHNESS_ADAPTER_DESIGN_2026-05-17]]
- [[NIGHT_R0_READ_ONLY_BATCH_DESIGN_2026-05-17]]
- [[F111_DATA_FRESHNESS_GATE_R1_2026-05-17]]
- [[F282_POST_SNAPSHOT_STAGING_AUDIT_RUNNER_R1_2026-05-17]]

---

## §1 経緯

5/17 設計時の W2-C overlay role mapping は **旧 W2-B 旧データ示唆** (= staging
復旧前の derived/signal 古い state) に基づく **仮分類**。

5/18 staging 復旧 + 案 A.5/B/C 完了 (= derived/signal/W2-B 全 fresh 化) を受け、
fresh data で W2-C role mapping を **再評価** し、設計の妥当性を確証する。

実装変更は 0、設計レビュー結果を vault に記録するのが本 doc。

---

## §2 fresh W2-B 要約 (= as_of 2026-05-18)

| code | label              | vol_mom_20d | srs      |
|------|--------------------|-------------|----------|
| 9247 | neutral            | +22.78%     | -1.18%   |
| 9628 | neutral            |  +3.47%     | -0.88%   |
| 4404 | neutral            | -13.68%     | -0.51%   |
| 9008 | neutral            | +36.28%     | +0.08%   |
| 3134 | **leader_inertia** | -64.51%     | -12.75%  |
| 7595 | **leader_inertia** |  -8.23%     | -6.22%   |
| 9417 | **leader_inertia** | -83.48%     | -18.32%  |

label_counts: leader_inertia 3 / neutral 4

参考 (= 対象外):
- 3962 チェンジ : vol_mom +103.45% / srs -2.11% → neutral (= 出来高激増だが sector 平均)
- 6196 ストライク: vol_mom -62.42% / srs -12.22% → leader_inertia

---

## §3 W2-C 設計 9 step 適用 結果 (= 再分類)

設計 9 step (= F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17 §3.1):

1. freshness == FAIL → insufficient_data
2. W2-A == recovery_blocked → excluded_recovery
3. **W2-B == leader_inertia → inertia_warning** (= 任意 W2-A 上書き) ★ 7595 適用
4. W2-A == new_face + W2-B ∈ {leader_continue/individual_strong} → new_face_momentum_up
5. W2-A == new_face + 他 → new_face_sector_compare
6. W2-A == core + W2-B == leader_continue → continuing_core
7. W2-A == core + W2-B == individual_strong → momentum_warning
8. **W2-A == core + W2-B ∈ {neutral, sector_strong} → continuing_core_watch** ★ 4 件適用
9. fallback → continuing_core_watch

freshness state (= 本 wave 時点):
- overall: WARN
- blocks: [blocks_regime_analysis] (= index_data 別 wave)
- blocks_signal_regen: 解消済 ✓
- W2-C 体系適用: 可

---

## §4 7 銘柄 W2-C mapping (= old vs new)

| code | name        | old 仮 W2-C                | new W2-C                  | dot | 推奨    |
|------|-------------|----------------------------|---------------------------|-----|---------|
| 9247 | TRE         | momentum_warning           | **continuing_core_watch** | 🟡  | watch   |
| 9628 | 燦HD        | continuing_core_watch      | continuing_core_watch     | 🟡  | watch (流動性低) |
| 7595 | アルゴ      | **new_face_momentum_up** ★ | **inertia_warning** ★      | 🔴  | excluded |
| 9008 | 京王        | continuing_core_watch      | continuing_core_watch     | 🟡  | watch (上位) |
| 3134 | Hameel      | inertia_warning            | inertia_warning           | 🔴  | excluded |
| 9417 | スマバ      | inertia_warning            | inertia_warning           | 🔴  | excluded |
| 4404 | ミヨシ      | continuing_core_watch      | continuing_core_watch     | 🟡  | **excluded (流動性極薄)** |

---

## §5 重要変更 (= 7 銘柄 detail)

### §5.1 7595 アルゴグラフィックス (= 大降格 ★)

- old 仮: W2-A new_face_sector_compare + W2-B individual_strong → **new_face_momentum_up**
- fresh: W2-B **leader_inertia** (= vol_mom -8.23% / srs -6.22% で両劣位)
- step 3 適用 (= W2-B leader_inertia 優先、W2-A 上書き)
- new W2-C: **inertia_warning**
- → 仮昇格仮説を **完全棄却** ★

判定根拠:
- vol_mom 5d/20d 共に負、出来高 5d 平均 225K と OK だが trend 減速
- sector_relative_strength -6.22% で sector 平均から劣位
- 情報通信・サービスその他 sector の peer median は -0.81%

### §5.2 9247 ＴＲＥホールディングス (= momentum_warning → continuing_core_watch)

- old 仮: W2-A continuing_core + W2-B neutral 出来高弱め示唆 → **momentum_warning**
- fresh: W2-B **neutral** (= vol_mom +22.78% / srs -1.18%)
- step 8 適用 (= core + neutral → continuing_core_watch)
- new W2-C: **continuing_core_watch** (🟡 要確認)

判定根拠:
- vol_mom +22.78% で出来高 **強気側**、旧示唆「弱め」は否定
- srs -1.18% で sector 平均並、突出感なし → neutral
- **過熱気配 possible** (= 過去連続上位 + 出来高強気)
- **no-new-chase 監視対象** ★、追わない上限 1,644 円 厳守

### §5.3 9008 京王電鉄 (= watch 上位継続)

- old 仮: continuing_core_watch
- fresh: W2-B neutral (= vol_mom +36.28% / srs +0.08%)
- step 8 適用
- new W2-C: **continuing_core_watch** (= 上位 watch)

判定根拠:
- vol_mom +36.28% で出来高強気継続、旧示唆「強め」を fresh で confirm ✓
- srs +0.08% で運輸 sector 平均並 (= sector 全体 -3.56% 中、自己 -3.48%)
- 5d 出来高 2.16M = **流動性最高**、大型株 spread 狭
- watch **最上位**

### §5.4 3134 Ｈａｍｅｅ (= leader_inertia 継続 ✓)

- old 仮: inertia_warning
- fresh: W2-B leader_inertia (= vol_mom -64.51% / srs -12.75%)
- step 3 適用
- new W2-C: **inertia_warning** (= 設計通り、変更なし)

判定根拠:
- vol_mom 5d -71.5% / 20d -64.5% で出来高完全枯渇
- srs -12.75% で sector 平均比劣位明確 (= 小売 -5.72% 中、自己 -18.47%)
- entry 完全 skip 推奨、demote 完了銘柄

### §5.5 9417 スマートバリュー (= leader_inertia 最強度)

- old 仮: inertia_warning (= 当初予測)
- fresh: W2-B leader_inertia (= vol_mom -83.48% / srs -18.32%)
- step 3 適用
- new W2-C: **inertia_warning** (= 7 銘柄中 最強度)
- 加えて流動性最低 (= 5d 14K)、観察すら不要

### §5.6 9628 燦ホールディングス (= 真 neutral + 流動性低)

- old 仮: continuing_core_watch
- fresh: W2-B neutral (= vol_mom +3.47% / srs -0.88%、両 band 内)
- step 8 適用
- new W2-C: **continuing_core_watch** (= 設計通り)

判定根拠:
- 両 band 内 = 完全 neutral、判別保留
- 5d 出来高 40K = **流動性低** (= 100 株比率 0.248%、板薄リスク)
- watch 限定、entry 条件付き (= 板 5 ティック合計 ≥ 10K 株 必須)

### §5.7 4404 ミヨシ油脂 (= neutral だが運用 excluded)

- old 仮: continuing_core_watch (= 示唆なし初算出)
- fresh: W2-B neutral (= vol_mom -13.68% / srs -0.51%、片側 only)
- step 8 適用 → 設計 role は continuing_core_watch
- 運用判断: **excluded** (= 5d 出来高 17K で流動性極薄、entry 不可)

注: W2-C role と運用判断が乖離するケース。設計上 continuing_core_watch だが、
    流動性 check で運用 excluded。

---

## §6 overlay top5 案 (= 設計 ruleset 適用)

| rank | code | name | W2-C role             | dot | 流動性 (5d) | note |
|------|------|------|-----------------------|-----|------------|------|
| 1    | 9008 | 京王 | continuing_core_watch | 🟡  | 2.16M (最高) | vol_mom +36.28% 強気継続、entry 適性高 |
| 2    | 9247 | TRE  | continuing_core_watch | 🟡  | 370K        | vol_mom +22.78%、過熱注意 |
| 3    | 9628 | 燦HD | continuing_core_watch | 🟡  | 40K (低)    | neutral 真、流動性低 watch 限定 |
| 4    | 3134 | Hameel | inertia_warning     | 🔴  | 327K        | 見送り、demote 確定 |
| 5    | 7595 | アルゴ | inertia_warning     | 🔴  | 225K        | 仮昇格棄却、降格 |

設計 ruleset 検証:
- continuing_core ≤ 3: ✓ (= 3 件)
- new_face ≥ 1: 該当なし (= fresh で new_face_momentum_up 0、設計上は許容、警告 log のみ)
- inertia_warning は 4-5 位枠: ✓
- risk_yen 流入: 0 ✓

excluded (= top5 外):
- 4404 (= 5d 17K)
- 9417 (= 5d 14K + 最強度 inertia)

---

## §7 entry / watch / excluded 案 (= 朝サマリ入力)

### §7.1 判定: **HOLD** (= 本日 entry 0)

理由:
- continuing_core (= leader_continue 由来) 該当 0
- watch 候補は continuing_core_watch (= neutral) のみ、即 entry 根拠不足
- 7595 が leader_inertia へ降格 (= 仮 new_face_momentum_up 棄却)
- 9247 過熱気配 possible (= no-new-chase)
- blocks_regime_analysis 残 (= index_data 別 wave)

### §7.2 entry 候補: **0 件**

GO_CONDITIONAL 条件 (= 将来昇格):
- W2-B fresh で leader_continue or individual_strong に昇格する銘柄出現
- 9247/9008 の vol_mom 強気 + srs 改善 (= sector 突出) で個別強気確証
- index_data refresh (= blocks_regime_analysis 解消)

### §7.3 watch 候補 (= iSPEED 観察のみ、執行 0)

| 順位 | code | name | dot | 観察ポイント |
|------|------|------|-----|------------|
| 1    | 9008 | 京王 | 🟡  | 寄付き出来高 vs 5d 2.16M、運輸 sector 動向、大型 spread |
| 2    | 9247 | TRE  | 🟡  | 寄付き出来高 vs 5d 370K、過熱兆候 visual confirm |
| 3    | 9628 | 燦HD | 🟡  | 板 5 ティック厚さ (= < 10K で entry 不可) |

### §7.4 excluded 候補

| code | 排除理由 |
|------|--------|
| 3134 | leader_inertia 継続、vol_mom -64.51% で出来高完全枯渇 |
| 7595 | **降格** ★ 仮 new_face_momentum_up 棄却 → inertia_warning |
| 9417 | leader_inertia 最強度 + 流動性最低 (5d 14K) |
| 4404 | vol_mom 弱 + 流動性極薄 (5d 17K)、設計 watch だが運用 excluded |

### §7.5 no-new-chase / demote 監視

**no-new-chase** (= 過熱気配):
- **9247 (TRE)**: 過去連続上位 + vol_mom +22.78% 強気側
  寄付き陽線 = 過熱兆候、追わない上限 1,644 円 ★ 厳守

**demote 完了** (= 記録、entry 完全外):
- 3134 Hameel : leader_inertia 継続 (= 2 週連続確証)
- 7595 アルゴ : new_face_momentum_up 仮説 棄却 → leader_inertia 降格 ★
- 9417 スマバ : leader_inertia 最強度 + 流動性最低

---

## §8 朝サマリ再生成 結果 (= /tmp/.../fresh_morning_summary.md)

判定: **HOLD**
entry: 0 件
watch: 9008 (1 位) / 9247 (2 位) / 9628 (3 位)
excluded: 3134 / 7595 / 9417 / 4404
no-new-chase: 9247 (= 過熱気配)
demote 完了: 3134 / 7595 / 9417

注文完成形 アドバイザリ (= 参考、本日 HOLD 中 執行 0):

### §8.1 9008 京王 (= 1 位 watch)
- 基準価格: 775.5 円 (5/15 close)
- entry 検討: 769 〜 782 円
- 追わない上限: 788 円 (= +1 ATR 12.7)
- 利確: 801 円 / 損切: 763 円
- 株数: 100 / notional: 77,550 / risk_yen: 約 1,268 円
- 流動性: 0.005% (= 最高)

### §8.2 9247 TRE (= 2 位 watch、過熱注意 ★)
- 基準価格: 1,606 円
- entry 検討: 1,587 〜 1,625 円
- **追わない上限: 1,644 円 ★ 厳守** (= +1 ATR 38.2)
- 利確: 1,682 円 / 損切: 1,568 円
- 株数: 100 / notional: 160,600 / risk_yen: 約 3,825 円
- 流動性: 0.027%

### §8.3 9628 燦HD (= 3 位 watch、流動性低)
- 基準価格: 1,363 円
- entry 検討: 1,352 〜 1,374 円
- 追わない上限: 1,384 円 (= +1 ATR 21.2)
- 株数: 100 / notional: 136,300 / risk_yen: 約 2,120 円
- 流動性: 0.248% (= 板薄 ⚠)
- 見送り条件: 板 5 ティック合計 < 10K / spread 30 円超 / 寄付き出来高 < 1K

---

## §9 W2-C 設計の検証 (= fresh data での妥当性)

| 設計項目 | 検証結果 |
|---|---|
| step 3 (= W2-B leader_inertia 優先) | ✓ 7595 降格に有効、誤 entry 防止 |
| step 8 (= core + neutral → watch) | ✓ 9247/9008 強気 vol_mom も真 entry にせず |
| 9247/9628 機械的排除 0 件 | ✓ continuing_core_watch で保持、警告付き |
| leader_inertia 上位でも 🔴 注意 | ✓ 3134/7595/9417 全件 🔴、明確 |
| new_face_momentum_up の発火条件 | ✓ 厳密、仮昇格を fresh で棄却可能 |
| risk_yen 流入 0 | ✓ rank 算出に流動性 + label のみ使用 |
| freshness FAIL 時の体系 freeze | 本 wave は WARN のため適用、freeze なし |

→ W2-C 設計は fresh data で **妥当性確認** ✓

---

## §10 安全 gate (= 本 wave、純設計レビュー + doc 作成)

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| financials refresh / prices refresh / index refresh | 0 |
| derived regen / signal regen / W2-B rerun | 0 |
| schema migration | 0 |
| 実 J-Quants API call | 0 |
| TDnet / XBRL 取得 | 0 |
| token / env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| fire repo code 変更 | 0 |
| git add / commit / push (fire repo) | 0 |
| PR 作成 / merge | 0 |
| auto-order / 楽天 / iSPEED | 0 |
| Computer Use | 0 |
| --no-verify / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |
| F111 実装変更 | 0 |
| morning advisory 実装変更 | 0 |

3 DB md5 (= 本 wave 期間中、完全一致):
- data/fire.db         : b1df4673... ✓
- data/fire.develop.db : 085799da... ✓
- data/fire.staging.db : a9050502... ✓ (= W2-B rerun 後 read-only 維持)

legacy cron active line : 0 ✓ (= decommission 維持)
chmod -x bin/fire-*.sh   : 維持 ✓
backup 健在               : 4.8 GB ✓
F282 plist 3 file md5    : 完全不変 ✓

---

## §11 次アクション

### §11.1 直近 (= 今週)

1. **HQ_APPROVE_DATA_R1_STAGING_REFRESH_INDEX_TARGETED**
   - blocks_regime_analysis 解消 (= index_data 5/11 → 5/15)
   - regime 機能解禁
   - Freshness Gate overall WARN → OK 期待

### §11.2 中期 (= 5/末 - 6 月)

2. **HQ_APPROVE_F282_LEGACY_BIN_DECOMMISSION**
   - bin/fire-snapshot-staging.sh / bin/fire-init-develop.sh の rename or delete
   - 三重防衛 (= crontab disable + chmod -x + file 自体削除)

3. **HQ_APPROVE_STAGING_DB_MD5_WATCHER**
   - 5 分監視 + 変化検出時 LINE 通知
   - 別 launcher 経由の意図せぬ staging 上書きを早期検出

4. **HQ_APPROVE_JPX_SEED_R3_OPENCLAW_LOCAL_CAPTURE** (= JPX rank schema)
   - JPX Market Explorer OpenClaw 取得補助 R3-2 実装
   - schema 1.1.0 拡張 (= screen_url / acquired_by / captured_at_jst 等)
   - 既設計 vault: JPX_MARKET_EXPLORER_OPENCLAW_ACQUISITION_DESIGN_2026-05-18

### §11.3 中長期 (= 6-7 月)

5. **HQ_APPROVE_NIGHT_R0_IMPLEMENTATION**
   - 9 step read-only batch 実装 (= 8 wave)
   - 既設計 vault: NIGHT_R0_READ_ONLY_BATCH_DESIGN_2026-05-17
   - 5/18 朝の手作業を自動化 (= F282 audit / Freshness Gate / W2-B / 朝サマリ)

### §11.4 長期 (= 7 月以降)

6. NIGHT-R1 (= staging write 付き) → NIGHT-R2 (= Paper Live)
7. Live Advisory 昇格 (= F053 全 PASS + Fujiwara 受容 + LINE 5 部屋)

---

## §12 関連 file / commit

本 wave 出力 (= /tmp、ephemeral):
- /tmp/fire_w2c_re_review_fresh/w2c_re_review.md (= 設計詳細)
- /tmp/fire_w2c_re_review_fresh/w2c_mapping.json (= 機械可読)
- /tmp/fire_w2c_re_review_fresh/morning_summary_inputs.md
- /tmp/fire_w2c_re_review_fresh/fresh_morning_summary.md

本 doc (= vault 永続化):
- ~/fire-vault/03_design/F111_W2C_RE_REVIEW_FRESH_2026-05-18.md (= 本 doc)

参照 (= read-only):
- /tmp/fire_w2b_rerun_fresh_data/overlay_classification.json (= 直近 W2-B 結果)
- fire-vault/03_design/F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17.md (= 既設計、f45814c)

不変 (= 触れず):
- data/fire.db / data/fire.develop.db / data/fire.staging.db
- ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist
- docs/launchd/jp.fire.weekly-snapshot*.plist
- /var/at/tabs/bluefire (= crontab、disabled 維持)

---

## §13 success 判定 (= 本 wave)

| 基準 | 結果 |
|---|---|
| fresh W2-B 7 銘柄 W2-C 再分類 完了 | ✓ (= §4) |
| 設計 ruleset 適用 妥当性確証 | ✓ (= §9、全項目 OK) |
| 7595 仮昇格 棄却 + inertia_warning 降格 | ✓ ★ (= §5.1) |
| 9247 過熱 monitor (= no-new-chase) | ✓ (= §5.2, §7.5) |
| 9008 watch 上位 | ✓ (= §5.3, §6) |
| 3134 / 9417 inertia 継続 | ✓ (= §5.4, §5.5) |
| 4404 neutral だが運用 excluded | ✓ (= §5.7) |
| 朝サマリ再生成 入力完備 | ✓ (= §8) |
| 次 wave 順序明示 | ✓ (= §11) |
| DB / API / LINE / token / fire repo 操作 | 0 (= §10) |
| vault commit/push | (= 本 doc 別 step で実施、本 wave goal 含む) |

→ **全成功基準 充足、本 wave 完了 ✓**
