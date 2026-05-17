# FIRE F111-W2C-OVERLAY-ROLE-RECLASSIFICATION-DESIGN (2026-05-17)

doc_id: FIRE-F111-W2C-OVERLAY-ROLE-RECLASSIFICATION-DESIGN-R1-2026-05-17
status: design-only / no impl
HQ marker: (= 設計のみ、specific impl marker は別 wave で要)
related:
- [[F111_DATA_FRESHNESS_GATE_R1_2026-05-17]] — freshness OK/WARN/FAIL
- [[F282_POST_SNAPSHOT_STAGING_AUDIT_RUNNER_R1_2026-05-17]] — fresh 取得経路
- [[STAGING_DB_RESTORE_FROM_BACKUP_RECENT_2026-05-17]] — staging 基準
- [[JQUANTS_ENV_SUPPLY_PREFLIGHT_R1_2026-05-17]] — financials retry 前提

---

## §1 目的

W2-B が産出する **momentum / sector 5 label** と F111 **Freshness Gate**
の結果を、W2-A の overlay role 体系に **W2-C として統合**し、F062
Buyability Card と morning advisory に表示する設計を確定する。

主眼:
- 9247 / 9628 等を **機械的に消さない**「警告付き表示」体系
- leader_inertia を上位でも 🔴 注意枠化
- individual_strong 新顔 を 🟢 昇格候補化
- Freshness FAIL 時は W2-C 体系自体を freeze (= W2-A 旧表示 fallback)
- risk_yen は順位 / 排除 / 加点に**流入させない**

---

## §2 成果物

### §2.1 設計 doc 4 file

| path | 内容 |
|---|---|
| `/tmp/fire_w2c_overlay_role_design/w2c_role_mapping.md` | 8 role 定義 + W2-A × W2-B × Freshness mapping table + icon/dot |
| `/tmp/fire_w2c_overlay_role_design/w2c_selection_rules.md` | overlay top5 選抜 5 step + 機械排除禁止 + leader_inertia 細則 |
| `/tmp/fire_w2c_overlay_role_design/w2c_display_examples.md` | 7 銘柄暫定マッピング (= 参考例) + F062 card + advisory 表示例 |
| `~/fire-vault/03_design/F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17.md` | 本 doc (= 統合 + impl plan + Codex 8 lane + 安全 gate) |

### §2.2 主要設計事項 (= サマリ)

1. **W2-C 8 role**: continuing_core / continuing_core_watch /
   momentum_warning / new_face_sector_compare / new_face_momentum_up /
   inertia_warning / excluded_recovery / insufficient_data
2. **role 決定優先順位** (= 9 step ruleset): freshness FAIL → recovery
   → leader_inertia → new_face × momentum → continuing 系 → fallback
3. **top5 選抜**: continuing_core ≤3 / new_face ≥1 保証 / inertia_warning
   は raw 上位でも 4-5 位 / overlay_rank は raw_rank と別保存
4. **freshness FAIL fallback**: W2-A 旧 4 role 維持 + 体系 freeze 明示
5. **risk_yen 隔離**: 順位 / 排除 / 加点に一切流入させない
6. **9247 等の機械排除 0 件保証**: 警告付き表示で残置

---

## §3 implementation plan (= 別 wave、最小差分)

### §3.1 ファイル単位 diff scope

| layer | file | 変更内容 | 行数目安 |
|---|---|---|---|
| pure mapper | `scripts/jobs/_v1_4_1_w2c_role_mapper.py` (新規) | 8 role 決定関数 + icon/dot mapper + reason_text builder | ~250 |
| consumer | `scripts/jobs/_v1_4_1_consumer.py` (拡張) | overlay_top5 entry に w2b_label/vol_mom/srs/freshness_status/w2c_role/dot_color 追加 | ~80 |
| Buyability Card | `notifications/templates/overlay_buyability_card.py` (拡張) | ROLE_BADGES に W2-C 4 新 role 追加 (= continuing_core_watch / momentum_warning / new_face_momentum_up / inertia_warning / insufficient_data) | ~40 |
| advisory renderer | `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` (拡張) | overlay top5 table に W2-C role / W2-B label / freshness columns 追加 + role 変更理由 section | ~120 |
| freshness 連動 | `scripts/jobs/_v1_4_1_consumer.py` (= 上の拡張内) | freshness gate 結果を input 引数で受領、FAIL → 全 entry insufficient_data 強制 | (上記内含) |
| tests | `tests/scripts/jobs/test_v1_4_1_w2c_role_mapper.py` (新規) + 既 test 拡張 | 8 role 決定 + 60 マス mapping + edge case | ~400 |
| smoke | `scripts/jobs/run_v1_4_1_w2c_smoke.py` (新規、read-only) | fresh staging + W2-B + freshness を join → top5 出力 | ~150 |

**合計 net diff 目安**: production ~490 行 / tests ~400 行 = LOC ~890

### §3.2 sequencing (= depends on)

```
[pre-condition (= 既完了)]
- W2-A overlay role 4 種 (= _v1_4_1_consumer.py)
- W2-B 5 label (= _f111_overlay_momentum_sector.py)
- F111 Freshness Gate R1/R1.1 (= 本 PR で main 反映済)
- F282 post-snapshot audit runner (= 本 PR で main 反映済)

[必要 wave (= W2-C 実装前 prerequisite)]
1. 5/18 02:00 F282 自動 fire
2. post-snapshot audit decision = PROCEED 確定
3. HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY (= wrapper 方式)
4. HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH (= 案 A.5)
5. HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH (= 案 B)
6. HQ_APPROVE_W2B_RE_RUN_FRESH_DATA (= 案 C)
7. **本 W2-C 設計 review** (= 本 doc + Fujiwara confirm)

[W2-C 実装 wave (= 順序)]
W2-C-1: pure mapper helper 実装 + unit tests (= 100% read-only)
W2-C-2: consumer 拡張 + freshness 連動 + tests
W2-C-3: F062 Buyability Card 拡張 + tests
W2-C-4: advisory renderer 拡張 + tests
W2-C-5: smoke runner 実装 + read-only smoke (= 4 DB / 0 write)
W2-C-6: Codex 8 lane review + 必要修正
W2-C-7: commit + PR + main merge
```

### §3.3 risk control (= 各実装 wave 内)

- 全 wave 共通: DB write 0 / API 0 / LINE 0 / token 表示 0
- W2-C-3: 既 4 role の表示は**変更しない** (= 後方互換)
- W2-C-4: 既 raw_score top5 table は**そのまま維持** (= overlay 側拡張のみ)
- W2-C-5: smoke output は `/tmp/fire_w2c_*/` 配下 (= production write 0)

---

## §4 Codex 8 lane 相当 self-audit

(= 本 wave は設計のみのため Codex CLI 8 lane 並列実行ではなく self-audit 表で代替。
 実装 wave 時に CLI 8 lane を実 review として要)

| lane | 観点 | verdict | 摘要 |
|---|---|---|---|
| A | role mapping 妥当性 | APPROVE | 8 role 漏れなく重複なく、優先順位 9 step 明示。W2-A 旧 4 role 後方互換 |
| B | overlay selection rule | APPROVE | 5 step + 機械排除禁止 + new_face ≥1 保証。raw_rank / overlay_rank 分離保存 |
| C | 9247/9628 排除しない安全性 | APPROVE | momentum_warning / continuing_core_watch 枠で残置、機械排除 0 件保証明示 |
| D | new face / new sector 扱い | APPROVE | individual_strong 新顔 → new_face_momentum_up 🟢 昇格、sector_strong 新顔 → 比較枠維持 |
| E | Freshness Gate 連携 | APPROVE | FAIL → W2-C 体系 freeze + W2-A 旧 fallback 明示、WARN → 適用 + 注意付記、OK → 通常 |
| F | F062 card / advisory 表示 | APPROVE | F062 ROLE_BADGES 拡張のみ (= 既 4 既存削除なし)、advisory に W2-C role/W2-B/freshness 3 column 追加 |
| G | implementation plan / tests | APPROVE | 7 wave 順序明示、LOC ~890、production ~490 / tests ~400、smoke read-only |
| H | operational risk / next-wave sequencing | APPROVE | F282/financials retry/derived regen/signal regen/W2-B rerun の 7 prerequisite 明示、5/18 fire 待ち、本日は設計のみ |

**結果**: 8 lane 全 APPROVE / CRITICAL 0 / HIGH 0

### §4.1 self-audit で検出した小考察 (= MEDIUM/LOW、本設計内で吸収済)

1. **MEDIUM: `momentum_warning` 発火条件の precision** —
   "neutral かつ vol_mom < -0.02" の閾値 -0.02 は magic number、
   将来 config 化推奨 (= [[w2c_role_mapping]] §3.4 で明示済)
2. **MEDIUM: top5 整列の sort key 衝突** —
   同 raw_rank の同 role 多発時の tie-breaker 未定義 → 実装時は
   `(raw_rank, code)` で deterministic 明示推奨 (= 実装 wave で対応)
3. **LOW: insufficient_data の reason_text の追加詳細** —
   freshness FAIL vs W2-B 算出不能 を区別表示推奨 → 実装時 reason_text
   builder で分岐 (= [[w2c_role_mapping]] §4 で雛形のみ提示済)
4. **LOW: `excluded_recovery` の dot 厳密化** —
   現状全 🔴 だが「復帰検討中」と「完全除外」の区別が将来必要 →
   別 wave で recovery_state field 追加検討

(= いずれも HIGH 未満、本設計の確定を遅らせない)

---

## §5 安全 gate (= 設計 wave、純 read-only + write zero)

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| staging write / production-develop | 0 |
| financials refresh / retry | 0 |
| derived regen / signal regen | 0 |
| W2-B rerun | 0 |
| 実 J-Quants API call | 0 |
| TDnet HTML / XBRL 取得 | 0 |
| token / .env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl 実行 / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push / PR / merge | 0 (= 本 wave 設計 doc も別 wave で commit 要) |
| schema migration | 0 |
| auto-order / 楽天 / iSPEED | 0 |
| Computer Use / Playwright / Selenium | 0 |
| --no-verify / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |
| 既 W2-A role の削除 / rename | 0 (= W2-C は追加のみ) |
| 9247 / 9628 等の機械的排除 | 0 (= 体系設計上保証) |
| risk_yen の順位 / 排除 / 加点流入 | 0 (= 体系設計上隔離) |

---

## §6 5/18 02:00 以降の流れ (= 本設計の活用 timeline)

```
[5/17 20:45]  PR #4 main merge ✓ (= 本日完了)
[5/17 21:xx]  本設計 doc 完成 ✓ (= 本 wave 完了)
       ↓
[5/18 02:00]  F282 weekly-snapshot 自動 fire (= operator 操作不要)
       ↓
[5/18 朝]     post-snapshot audit (= 本 PR の runner)
              decision = PROCEED → 次へ
              decision = RESTORE → 別 wave (= atomic restore)
              decision = INVESTIGATE → 停止 + log 確認
       ↓
[5/18 朝]     HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY
              (= wrapper 方式 env 供給で financials retry)
       ↓
[5/18 夜?]    HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH (= 案 A.5)
[5/18 夜?]    HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH (= 案 B)
[5/19 ?]      HQ_APPROVE_W2B_RE_RUN_FRESH_DATA (= 案 C)
       ↓
[5/19+ ?]     fresh data 上で本 W2-C 設計 re-review
              ↓
              本設計の §3 impl plan 着手 (= W2-C-1 〜 W2-C-7、7 wave)
       ↓
[5/2x+ ?]     W2-C 実装完了 + PR + main merge
              → F062 / morning advisory に W2-C 表示活性化
              → 9247/9628 機械排除なし / leader_inertia 🔴 警告化
              → fresh data 上で初の本格運用
```

---

## §7 deferred (= 別 wave / 将来検討)

- `momentum_warning` 閾値 -0.02 の config 化
- `excluded_recovery` の recovery_state field (= "検討中" vs "完全除外")
- W2-B label vs W2-C role の **時系列 history 保存** (= 翌日の role 変化 audit 用)
- W2-C role の **KPI tracking** (= 8 role 各カテゴリの hit rate / 損益)
- F062 card に **W2-B 数値 (vol_mom/srs)** を直接表示 vs reason_text のみ
- Freshness WARN 時の **table 別注意付記** (= どの table が WARN か)
- W2-C role が変わった銘柄の **LINE 差分通知** (= 翌日 advisory diff)

---

## §8 related ファイル / commit

本 wave 出力 (= 全 untracked、commit は別 wave で):
- `/tmp/fire_w2c_overlay_role_design/w2c_role_mapping.md`
- `/tmp/fire_w2c_overlay_role_design/w2c_selection_rules.md`
- `/tmp/fire_w2c_overlay_role_design/w2c_display_examples.md`
- `~/fire-vault/03_design/F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17.md` (= 本 doc)

参照 source (= 既存、本 wave で変更なし):
- `scripts/jobs/_v1_4_1_consumer.py` (= W2-A overlay role 4 種)
- `scripts/jobs/_f111_overlay_momentum_sector.py` (= W2-B 5 label)
- `scripts/jobs/_f111_data_freshness_gate.py` (= freshness OK/WARN/FAIL)
- `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` (= advisory renderer)
- `notifications/templates/overlay_buyability_card.py` (= F062 card)

---

## §9 success 判定 (= 本設計 wave)

| 基準 | 結果 |
|---|---|
| W2-C 役割再分類ルール明確化 | ✓ (= 8 role + 9 step ruleset + 60 マス mapping) |
| fresh W2-B rerun 後にすぐ実装可能 | ✓ (= impl plan §3 で 7 wave 順序明示、LOC ~890) |
| 9247 / 9628 を機械的に消さない | ✓ (= 体系設計上保証、機械排除 0 件) |
| 3134 / 7595 の入替判断例明確 | ✓ ([[w2c_display_examples]] §2.3 で 3 銘柄入替表) |
| Freshness FAIL 時の扱い明確 | ✓ (= W2-C freeze + W2-A 旧 fallback + 体系 freeze 表示) |
| DB / API / LINE / token / git 操作 | 0 (= §5 安全 gate 全 0) |

→ **全成功基準 充足、本 wave 完了 ✓**
