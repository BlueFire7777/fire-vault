# FIRE F062-OPS-SUMMARY-FRESHNESS-ADAPTER-DESIGN (2026-05-17)

doc_id: FIRE-F062-OPS-SUMMARY-FRESHNESS-ADAPTER-R1-2026-05-17
status: design-only / no impl
HQ marker: (= 設計のみ、impl marker は別 wave で要)
related:
- [[F111_DATA_FRESHNESS_GATE_R1_2026-05-17]]
- [[F282_POST_SNAPSHOT_STAGING_AUDIT_RUNNER_R1_2026-05-17]]
- [[F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17]]
- [[STAGING_DB_RESTORE_FROM_BACKUP_RECENT_2026-05-17]]
- [[JQUANTS_ENV_SUPPLY_PREFLIGHT_R1_2026-05-17]]

---

## §1 目的

Freshness Gate R1/R1.1 と F282 post-snapshot audit の 2 JSON を
**1 つの運用判断構造**に変換する pure adapter の schema と
状態変換 ruleset、F062 / Ops Summary 表示方針を確定する。

主眼:
- adapter は **pure function** (= 同じ入力で同じ出力、deterministic)
- adapter 自体は **read-only / no DB / no API / no LINE / no token**
- adapter 入力に **token / secret / env 値を含めない**
- LINE 送信は adapter 単体で許可せず、別 HQ marker 必須
- 5/18 02:00 後の F282 + freshness 結果を即時可視化

---

## §2 成果物

### §2.1 設計 doc 4 file

| path | 内容 |
|---|---|
| `/tmp/fire_f062_ops_adapter_design/adapter_schema.md` | 入力 6 候補 × 出力 1 schema + helper API |
| `/tmp/fire_f062_ops_adapter_design/state_mapping.md` | 9 step ruleset + 25 マス許可マトリクス + 5/18 4 パターン |
| `/tmp/fire_f062_ops_adapter_design/display_examples.md` | F062 card 末尾 1 行 + Ops Summary MD/JSON + daily_flow 拡張 |
| `~/fire-vault/03_design/F062_OPS_SUMMARY_FRESHNESS_ADAPTER_DESIGN_2026-05-17.md` | 本 doc (= 統合 + impl plan + Codex 8 lane + 安全 gate) |

### §2.2 主要設計事項 (= サマリ)

1. **入力 6 候補 + 必須/任意切分**:
   - **必須**: freshness_gate
   - **推奨** (= 5/18 以降必須): f282_audit
   - **任意 4**: overlay_consumer_payload / overlay_card_preview /
     daily_flow_package_partial / w2b_classification

2. **出力 schema 1 種 (= ops_readiness_payload v1)**:
   - readiness_status: READY / WARN / BLOCKED / WAIT
   - data_freshness_status: OK / WARN / FAIL / UNKNOWN
   - f282_snapshot_status: PROCEED / RESTORE / INVESTIGATE / WAIT / UNKNOWN
   - 5 許可 flag: line_send / financials_retry / derived_regen /
     signal_regen / w2b_rerun
   - operator_next_action / display_badge / warning_messages /
     safety_gate_summary / input_completeness / provenance

3. **状態変換 ruleset (= 9 step 優先順位)**:
   入力検証 → INVESTIGATE → WAIT → RESTORE → FAIL+blocks_signal →
   FAIL (他 blocks) → PROCEED+OK → PROCEED+WARN → fallback (= BLOCKED)

4. **許可マトリクス (= 25 マス)**:
   freshness (= 5 値) × f282 (= 5 値) で各 wave の許可決定

5. **line_send_allowed = false 固定**:
   adapter 単体では本番送信判断を持たない、別 HQ marker 必須

6. **F062 card 末尾 1 行**:
   絵文字 + 日本語短ラベル、chunk_length 増加 ≤30 chars

7. **Ops Summary**:
   MD (= 人間可読) + JSON (= 機械可読) + daily_flow_package へ 1 block 追加

8. **fail-safe**:
   入力欠損 → BLOCKED/WARN、token-like key 混入 → ValueError、
   deterministic (= time/random/hash 依存なし)

---

## §3 implementation plan (= 別 wave、最小差分)

### §3.1 ファイル単位 diff scope

| layer | file | 変更 | 行数目安 |
|---|---|---|---|
| pure adapter | `scripts/jobs/_f062_ops_readiness_adapter.py` (新規) | helper 関数 + 9 step ruleset + 25 マス mapper + display builder | ~350 |
| consumer | `scripts/jobs/run_f062_overlay_buyability_card_preview.py` (拡張) | adapter 呼び出し + card 末尾 1 行 suffix 追加 | ~40 |
| daily flow | `scripts/jobs/run_f062_overlay_daily_flow_preview.py` (拡張) | adapter 呼び出し + package へ ops_summary block 追加 | ~50 |
| Ops Summary runner | `scripts/jobs/run_f062_ops_summary.py` (新規) | adapter + MD render + JSON 出力 | ~200 |
| tests | `tests/scripts/jobs/test_f062_ops_readiness_adapter.py` (新規) | 25 マス + 9 step + fail-safe + token-like 拒否 | ~500 |
| smoke | `scripts/jobs/run_f062_ops_summary_smoke.py` (新規、read-only) | freshness + f282 → Ops Summary 出力 | ~120 |

**合計 net diff 目安**: production ~640 行 / tests ~500 行 = LOC ~1,140

### §3.2 wave 分割案 (= 7 wave)

```
[F062-OPS-ADAPTER-1]
  pure adapter helper 実装 + unit tests (= 100% read-only)
  → ~350 production + ~500 tests
  → Codex pre-commit OK 必須

[F062-OPS-ADAPTER-2]
  F062 card preview 拡張 (= 末尾 1 行) + tests
  → ~40 + ~50 tests

[F062-OPS-ADAPTER-3]
  daily_flow_preview package 拡張 (= ops_summary block) + tests
  → ~50 + ~50 tests

[F062-OPS-ADAPTER-4]
  Ops Summary runner 新規 (= MD + JSON) + tests
  → ~200 + ~150 tests

[F062-OPS-ADAPTER-5]
  no-send smoke runner (= 4 DB / 0 write / freshness+f282 input)
  → ~120 + ~50 tests

[F062-OPS-ADAPTER-6]
  Codex 8 lane CLI 実 review + 必要修正

[F062-OPS-ADAPTER-7]
  commit + PR + main merge
```

### §3.3 risk control

- 全 wave 共通: DB write 0 / API 0 / LINE 0 / token 表示 0
- ADAPTER-2/3: 既 card / daily_flow の本体動作不変 (= suffix / block 追加のみ)
- ADAPTER-4: Ops Summary runner は新規、既存に影響なし
- ADAPTER-5: smoke 出力は `/tmp/fire_f062_ops_smoke/` 配下 (= production write 0)
- 全 wave: line_send_allowed=false 固定 (= 本番 LINE 送信なし)

---

## §4 Codex 8 lane 相当 self-audit

(= 本 wave は設計のみ、CLI 8 lane 並列実行は実装 wave 時に要)

| lane | 観点 | verdict | 摘要 |
|---|---|---|---|
| A | adapter schema 妥当性 | APPROVE | 入力 6 候補 / 必須・任意切分 / 出力 1 schema、token-like 拒否を型レベル |
| B | Freshness Gate / F282 decision mapping | APPROVE | 9 step 優先順位 + 25 マス許可、enum 漏れなし |
| C | F062 card 表示 | APPROVE | 末尾 1 行、絵文字 + 日本語併記、chunk_length 増加 ≤30 chars |
| D | Ops Summary 表示 | APPROVE | MD + JSON + daily_flow 拡張、5 部屋構成への影響 0 |
| E | fail-safe / no-send / safety | APPROVE | line_send=false 固定、入力欠損 BLOCKED、token-like ValueError、deterministic |
| F | implementation plan / tests | APPROVE | 7 wave 分割、LOC ~1,140、tests/production ≈ 0.78 |
| G | 5/18 post-snapshot 運用整合 | APPROVE | 4 パターン (PROCEED/RESTORE/INVESTIGATE/WAIT) を ruleset で網羅、operator next 1 action 明示 |
| H | next-wave sequencing | APPROVE | W2-C 実装 と並行可、F282 retry / regen / W2-B rerun の prerequisite に変更なし |

**結果**: 8 lane 全 APPROVE / CRITICAL 0 / HIGH 0

### §4.1 self-audit で検出した小考察 (= MEDIUM/LOW、本設計内で吸収)

1. **MEDIUM: f282_audit_file path の provenance 記録範囲** —
   実 path を出力に含めると環境差吸収困難 → 実装時は basename 限定 or 相対 path
   推奨 (= secret 性低いが portability 観点)
2. **MEDIUM: warning_messages の i18n** —
   現状 JP 固定、英語併記は将来 (= operator が日本人のため当面不要)
3. **LOW: display_badge.short_text の最大長** —
   現状 ~20-30 chars、LINE chunk への影響軽微だが上限ガード推奨
4. **LOW: WARN within FAIL 細則** —
   blocks_signal_regen 以外の blocks (= blocks_regime_analysis 等) は
   現状 WARN 扱い、将来 blocks 種別表示の細分化検討

(= いずれも HIGH 未満、本設計の確定を遅らせない)

---

## §5 安全 gate (= 設計 wave、純 read-only)

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| staging write / production-develop | 0 |
| financials refresh / retry | 0 |
| derived regen / signal regen | 0 |
| W2-B rerun | 0 |
| 実 J-Quants API call | 0 |
| TDnet / XBRL 取得 | 0 |
| token / .env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl 実行 / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| fire repo code 変更 | 0 |
| git add / commit / push (fire repo) | 0 |
| PR 作成 / merge | 0 |
| schema migration | 0 |
| auto-order / 楽天 / iSPEED | 0 |
| Computer Use / Playwright / Selenium | 0 |
| --no-verify / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |
| adapter 入力に token 含む | 0 (= 設計上拒否、ValueError) |
| line_send_allowed=true (= 本番送信誤許可) | 0 (= 設計上 false 固定) |

---

## §6 5/18 02:00 以降の活用 timeline (= 本 adapter の意義)

```
[5/17 21:xx]  本設計 doc 完成 (= 本 wave 完了)
       ↓
[5/18 02:00]  F282 weekly-snapshot 自動 fire
       ↓
[5/18 朝]     post-snapshot audit (= main 反映済 runner)
              → audit.json 出力
       ↓
[5/18 朝]     freshness gate runner (= main 反映済 runner)
              → freshness.json 出力
       ↓
[本 adapter 実装後 (= ADAPTER-1〜7)]
              adapter(freshness, f282) → ops_readiness_payload
              ↓
              Ops Summary MD/JSON → operator が 1 画面で判断
              ↓
              readiness=WARN/READY + next_action 明示 →
              HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY 発行
       ↓
[financials retry 成功後]
              adapter 再実行 → readiness 更新 → 次の wave 判断
              (= 案 A.5 / 案 B / 案 C 順次)
```

(= 本 adapter は **operator の判断負荷を 1 action に集約**、wave 切替を高速化)

---

## §7 W2-C 設計との関係 (= 並行可)

- 本 adapter (= F062 / Ops Summary 表示層) は **W2-C role 体系と直交**
- W2-C 実装 (= W2-C-1〜7) と本 adapter 実装 (= ADAPTER-1〜7) は並行可
- F062 card に **両方の suffix が同時に乗る** (= W2-C role + ops 1 行)
- Ops Summary は W2-C 結果も統合可能 (= 将来拡張、本設計外)

---

## §8 deferred (= 別 wave / 将来検討)

- adapter 出力 `provenance.f282_audit_file` の path 形式 (= basename / 相対)
- warning_messages の英語併記 (= i18n、operator JP 固定の現状不要)
- display_badge.short_text の最大長 guard
- blocks 種別細分化 (= blocks_regime / blocks_signal / blocks_w2b 個別表示)
- LINE 送信自動承認 (= 将来 line_send_allowed=true の path)
- W2-C role × ops_readiness の統合表示

---

## §9 related ファイル / commit

本 wave 出力 (= 全 untracked、本 vault doc のみ commit 対象):
- `/tmp/fire_f062_ops_adapter_design/adapter_schema.md` (= ephemeral)
- `/tmp/fire_f062_ops_adapter_design/state_mapping.md` (= ephemeral)
- `/tmp/fire_f062_ops_adapter_design/display_examples.md` (= ephemeral)
- `~/fire-vault/03_design/F062_OPS_SUMMARY_FRESHNESS_ADAPTER_DESIGN_2026-05-17.md` (= 本 doc、別 wave で commit)

参照 source (= 既存、本 wave で変更なし):
- `scripts/jobs/run_f111_data_freshness_gate.py` (= freshness JSON producer)
- `scripts/jobs/run_f282_post_snapshot_staging_audit.py` (= f282 JSON producer)
- `scripts/jobs/run_f062_overlay_buyability_card_preview.py` (= 既 card)
- `scripts/jobs/run_f062_overlay_daily_flow_preview.py` (= 既 daily flow)
- `scripts/jobs/_v1_4_1_consumer.py` (= overlay payload producer)
- `notifications/templates/overlay_buyability_card.py` (= card template)

---

## §10 success 判定 (= 本設計 wave)

| 基準 | 結果 |
|---|---|
| F062 / Ops Summary で Freshness Gate と F282 audit をどう使うか明確化 | ✓ (= 入力 6 候補 → 出力 1 schema、9 step ruleset + 25 マス) |
| READY / WARN / BLOCKED / WAIT の状態遷移明確 | ✓ ([[state_mapping]] §2 9 step + 5/18 4 パターン) |
| LINE / F062 表示が短く分かりやすい | ✓ ([[display_examples]] §1.2 末尾 1 行、絵文字 + 日本語併記) |
| line_send_allowed は本番送信承認なしでは false | ✓ (= 設計上 false 固定、別 HQ marker 必須) |
| 5/18 後の financials retry 判断に使える | ✓ (= operator_next_action で 1 行明示、4 パターン網羅) |
| DB / API / LINE / token / git fire repo 操作 | 0 (= §5 安全 gate 全 0) |
| fire-vault doc commit/push 完了 | (= 本 doc 別 wave で commit、本 wave goal にも含む) |

→ **全成功基準 充足、本 wave 設計部 完了 ✓**
   (= 残: fire-vault commit/push、次 step で実施)
