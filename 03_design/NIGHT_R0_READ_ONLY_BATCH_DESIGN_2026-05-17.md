# FIRE NIGHT-R0-READ-ONLY-BATCH-DESIGN (2026-05-17)

doc_id: FIRE-NIGHT-R0-READ-ONLY-BATCH-R1-2026-05-17
status: design-only / no impl
HQ marker: (= 設計のみ、impl marker は別 wave で要)
related:
- [[F111_DATA_FRESHNESS_GATE_R1_2026-05-17]]
- [[F282_POST_SNAPSHOT_STAGING_AUDIT_RUNNER_R1_2026-05-17]]
- [[F062_OPS_SUMMARY_FRESHNESS_ADAPTER_DESIGN_2026-05-17]]
- [[F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17]]
- [[STAGING_DB_RESTORE_FROM_BACKUP_RECENT_2026-05-17]]
- [[JQUANTS_ENV_SUPPLY_PREFLIGHT_R1_2026-05-17]]

---

## §1 目的

相場後 (= 15:00 以降) 〜 深夜 (= 02:00 F282 前) の Mac mini 稼働時間
帯に、**翌朝候補生成のための read-only 準備 batch (NIGHT-R0)** を 1
コマンドで実行する設計を確定する。

主眼:
- **完全 read-only**: DB write 0 / API call 0 / LINE 送信 0 / token 参照 0
- **9 step orchestration**: pre-flight → F282 → freshness → candidate →
  W2-B → F062 preview → Ops Summary → paper PnL → safety summary
- **6 status 判定**: READY / WAIT / RESTORE / BLOCKED / INVESTIGATE / NO_GO
- **operator next 1 action**: 6 status 各に **1 action のみ**提示
- **launchd 登録は別 wave**: 本 wave は設計のみ

---

## §2 成果物

### §2.1 設計 doc 4 file

| path | 内容 |
|---|---|
| `/tmp/fire_night_r0_design/night_r0_flow.md` | 目的 + 9 step フロー + DAG + 早期停止 + boundary |
| `/tmp/fire_night_r0_design/night_r0_status_matrix.md` | 6 status + 9 step ruleset + マトリクス + summary JSON schema |
| `/tmp/fire_night_r0_design/night_r0_implementation_plan.md` | 5 layer 構造 + 8 wave 分割 + CLI + tests + 安全 enforcement |
| `~/fire-vault/03_design/NIGHT_R0_READ_ONLY_BATCH_DESIGN_2026-05-17.md` | 本 doc (= 統合 + R1/R2 接続 + Codex 8 lane + 安全 gate) |

### §2.2 主要設計事項

1. **NIGHT-R0 9 step**:
   pre_flight → F282 audit → Freshness Gate → candidate smoke →
   W2-B classification → F062 preview → Ops Summary → paper PnL →
   safety post-flight

2. **6 status**:
   READY_FOR_MORNING_PREVIEW / WAIT_FOR_F282 / RESTORE_REQUIRED /
   DATA_FRESHNESS_BLOCKED / INVESTIGATE / NO_GO

3. **9 step ruleset**: F282 → freshness → candidate → 警告継続 →
   post-flight md5 → fallback READY

4. **line_send_allowed = false 固定** (= 全 status で false)

5. **operator_next_action**: 6 status 各に **1 action のみ**

6. **5 layer implementation**:
   planner / step adapters / orchestrator runner / aggregator / smoke

7. **subprocess 方式採用**: 既存 runner と疎結合、JSON file 経由

8. **CLI**: --output-dir / --reference-date / 3 DB / --skip-step /
   --strict / --dry-run、exit code 0-99

---

## §3 NIGHT-R0 → NIGHT-R1 → NIGHT-R2 progression

### §3.1 NIGHT-R0 (= 本設計、read-only 専用)

- **scope**: 翌朝判断材料の事前生成、異常検知、operator 1 action 提示
- **書き込み**: なし
- **昇格条件 (= R1 前提)**: R0 が 7 日以上連続 READY、md5 violation 0、
  operator が 6 status の意味を理解
- **持続時間**: 1-2 ヶ月

### §3.2 NIGHT-R1 (= 別 wave 設計、staging write 付き)

- **scope**: 夜間 staging 更新 (= 案 A.5 derived regen / 案 B signal
  regen / 案 C W2-B rerun の **自動化**)
- **書き込み**: staging DB のみ (= production-develop 不触)
- **read-only 要素**: NIGHT-R0 全 9 step は R1 でも継続
- **追加 step (= R1 新規)**:
  - financials retry (= HQ marker 必要、auto execution の場合は再 design 要)
  - derived regen
  - signal regen
  - W2-B rerun (= フル再計算)
  - 各 step は f282 audit decision に gated
- **昇格条件 (= R2 前提)**: R1 が 14 日以上連続 staging write 成功、
  production-develop md5 完全不変、F119 paper PnL レポートで accuracy
  指標 ≥ baseline

### §3.3 NIGHT-R2 (= 将来設計、Paper Live / pattern 探索)

- **scope**: Paper Live tick による仮想売買、pattern 探索の自動化
- **書き込み**: paper_pnl / pattern_store のみ (= production unchanged)
- **追加 step**:
  - Paper Live tick (= F054)
  - Pattern Research Agent (= F035) のフル run
  - simulation accuracy 算出 (= F046)
  - DEATH NOTE / REHAB queue 自動判定 (= F033/F034)
- **昇格条件 (= 本番 Live Advisory)**: F053 5 項目 全 PASS +
  Fujiwara 受容 + LINE 5 部屋疎通

### §3.4 R0 → R1 promotion 条件 (= 整理)

| 項目 | 目標値 |
|---|---|
| R0 READY 連続日数 | ≥ 7 日 |
| md5 violation 検出 | 0 件 |
| operator が 6 status 理解 | confirmed by Fujiwara |
| F062/Ops Summary adapter 実装完了 | ✓ (= step 7 active 化) |
| paper PnL row 数 | ≥ 50 row (= F119 意味発生 threshold) |
| Codex 8 lane R0 implement audit | 全 APPROVE |
| HQ_APPROVE_NIGHT_R1_DESIGN_START | 別 wave |

---

## §4 Codex 8 lane 相当 self-audit

(= 本 wave は設計のみ、CLI 8 lane 並列実行は実装 wave 時に要)

| lane | 観点 | verdict | 摘要 |
|---|---|---|---|
| A | flow 設計 | APPROVE | 9 step + DAG + 早期停止 + boundary 明示、subprocess 方式採用 |
| B | status matrix | APPROVE | 6 status + 9 step ruleset + マトリクス、enum 漏れなし、operator next 1 action |
| C | safety / no-write | APPROVE | mode=ro URI + PRAGMA、subprocess env から token 除去、md5 violation 検出、launchd 別 wave |
| D | Ops Summary 連携 | APPROVE | step 7 は adapter 実装後 hook、未実装時 SKIPPED で summary 完成可、並行可 |
| E | morning preview 連携 | APPROVE | night_r0_summary を operator が朝確認 → 別承認で本番送信、判断材料の事前生成 |
| F | NIGHT-R1/R2 接続 | APPROVE | R0→R1→R2 progression 整理、各昇格条件明示、R1 は staging write、R2 は paper_pnl/pattern |
| G | tests / implementation | APPROVE | 5 layer + 8 wave、LOC ~1,730、tests/prod ≈ 0.68、6 status 全網羅、subprocess timeout |
| H | operational risk | APPROVE | line_send=false 固定、launchd 別 wave、md5 violation 検出、exit code 0-99、本番送信判断は朝 operator |

**結果**: 8 lane 全 APPROVE / CRITICAL 0 / HIGH 0

### §4.1 self-audit MEDIUM/LOW 検出 (= 本設計内 / 実装 wave で吸収)

1. **MEDIUM: subprocess 全 step での venv `sys.executable` 一致** —
   各 step が異なる venv で動くと state 矛盾。実装時 `sys.executable`
   固定 + log 化推奨。
2. **MEDIUM: step 9 md5 violation 時の partial output 取扱** —
   md5 violation 検出時に他 step output を retain vs purge を実装時決定。
   推奨: retain + INVESTIGATE 表示 (= 解析材料保持)。
3. **LOW: 8 step paper_pnl_readiness の SQLite 接続並列数** —
   step 8 と step 2-3 が同時 read で WAL competition → subprocess 順序
   制御で回避可。
4. **LOW: exit code 99 (= runner crash) の launchd 連携** —
   別 wave NIGHT-R0-LAUNCHD-1 で StandardErrorPath / log rotation 要設計。

---

## §5 安全 gate (= 設計 wave、純 read-only)

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| staging write / production-develop | 0 |
| financials refresh / retry | 0 |
| derived regen / signal regen | 0 |
| W2-B rerun (= フル再計算) | 0 |
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
| line_send_allowed=true (= 本番送信誤許可) | 0 (= 設計上 false 固定) |
| subprocess env に token 混入 | 0 (= 実装時 enforcement) |
| md5 violation 検出時の write 上書き | 0 (= 設計上 INVESTIGATE 強制) |

---

## §6 timeline (= 本設計の活用)

```
[5/17 21:xx]  本設計 doc 完成 (= 本 wave 完了)
       ↓
[NIGHT-R0 implementation wave (= 別 wave、NIGHT-R0-1〜8)]
  → 1-2 週間で順次実装、各 wave Codex 8 lane CLI review
       ↓
[NIGHT-R0 main merge 後]
  → 手動 fire (= 19:00-21:00 帯) で動作確認 1-2 週間
       ↓
[NIGHT-R0-LAUNCHD-1 (= 別 wave、launchd 登録)]
  → 自動 fire 切替、READY 連続日数 計測開始
       ↓
[R0 READY ≥ 7 日連続]
  → HQ_APPROVE_NIGHT_R1_DESIGN_START (= 別 wave)
  → NIGHT-R1 設計開始
```

5/18 02:00 F282 fire + audit + retry + regen + W2-B rerun の流れと
NIGHT-R0 implement は **並行可** (= NIGHT-R0 は独立 wave):
- 5/18 朝の作業は既存 main 反映済 runner で個別実行 (= NIGHT-R0 不使用)
- NIGHT-R0 完成後は朝作業を 1 batch に集約可能

---

## §7 W2-C / F062 Ops Summary adapter 設計との関係

| 設計 wave | 完了状態 | NIGHT-R0 との関係 |
|---|---|---|
| W2-C overlay role | 設計完了、vault commit 済 (= f45814c) | step 6 F062 preview で W2-C role suffix 表示 |
| F062 / Ops Summary adapter | 設計完了、vault commit 済 (= ab86afc) | step 7 で ops_readiness 生成、各 status の operator_next_action 整合 |
| NIGHT-R0 (= 本設計) | 設計完了、vault commit 別 step | step 6/7 で W2-C + adapter を統合 |

= **3 設計の整合性**: 6 status × W2-C 8 role × ops adapter 4 readiness
が矛盾なく整列。

---

## §8 deferred (= 別 wave / 将来検討)

- **NIGHT-R0-LAUNCHD-1** (= 別 wave): launchd plist 設計 + 登録 + ログ
  rotation + StandardErrorPath
- **NIGHT-R1 設計** (= 別 wave): staging write 付き、financials retry /
  derived/signal regen / W2-B rerun の自動化
- **NIGHT-R2 設計** (= 将来): Paper Live tick / Pattern Research / F033
  DEATH NOTE / F034 REHAB
- **step 7 が adapter 実装後 active 化** (= F062 / Ops adapter wave 完了後)
- **morning preview runner 設計** (= 別 wave): NIGHT-R0 summary を入力
  に朝の本番 LINE 送信 candidate 生成

---

## §9 関連 file / commit

本 wave 出力 (= 全 untracked):
- `/tmp/fire_night_r0_design/night_r0_flow.md` (= ephemeral)
- `/tmp/fire_night_r0_design/night_r0_status_matrix.md` (= ephemeral)
- `/tmp/fire_night_r0_design/night_r0_implementation_plan.md` (= ephemeral)
- `~/fire-vault/03_design/NIGHT_R0_READ_ONLY_BATCH_DESIGN_2026-05-17.md` (= 本 doc、別 step で commit)

参照 source (= 既存、本 wave で変更なし):
- `scripts/jobs/run_f282_post_snapshot_staging_audit.py` (= step 2)
- `scripts/jobs/run_f111_data_freshness_gate.py` (= step 3)
- `scripts/jobs/run_f111_overlay_momentum_sector_sim.py` (= step 5)
- `scripts/jobs/run_f062_overlay_buyability_card_preview.py` (= step 6)
- `scripts/jobs/run_f062_overlay_daily_flow_preview.py` (= step 6)

---

## §10 success 判定 (= 本設計 wave)

| 基準 | 結果 |
|---|---|
| NIGHT-R0 の目的 / 9 step / boundary 明確化 | ✓ ([[night_r0_flow]] 全章) |
| 6 status + ruleset + マトリクス明確化 | ✓ ([[night_r0_status_matrix]]) |
| operator_next_action 1 action 集約 | ✓ (6 status 各に 1 action) |
| line_send_allowed = false 固定 | ✓ (全 status で false) |
| 5 layer + 8 wave 実装計画明確化 | ✓ ([[night_r0_implementation_plan]]) |
| NIGHT-R1 / R2 接続条件整理 | ✓ (§3、昇格条件明示) |
| W2-C + F062 / Ops adapter 設計との整合 | ✓ (§7、3 設計の関係明示) |
| DB / API / LINE / token / git fire repo 操作 | 0 (= §5 安全 gate 全 0) |
| fire-vault doc commit/push 完了 | (= 本 doc 別 step で commit、本 wave goal 含む) |

→ **全成功基準 充足、本 wave 設計部 完了 ✓**
   (= 残: fire-vault commit/push、次 step で実施)
