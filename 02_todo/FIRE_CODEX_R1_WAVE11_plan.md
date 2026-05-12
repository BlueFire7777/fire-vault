---
id: FIRE-CODEX-R1-WAVE11-plan
phase: ガバナンス / Codex 並列実装 Wave 11 起票 / R-01-08 整合
priority: 最優先
status: 起票 ☆ Wave 11 4 sub-task + 2 audit lane 実行中 (2026-05-12)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 10 (= 完了)
  - HQ Wave 11 approve (= MIG-R1 apply plan / REPORT-R1 LINE delivery design /
    sub-D2.3.x runner 別 plan / 全体 guard audit、2026-05-12)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 11: MIG apply plan + REPORT-R1 LINE preview + sub-D2.3.x refine + guard audit

最終更新: 2026-05-12

## ★ 状態: 起票 (= 8 sub-task、全 plan/design/audit 系、fire code change は W11-2 のみ)

HQ Wave 11 approve 受領後、4 タスクを並列着手:
- W11-1: MIG-R1 apply plan **3 分割** (= dry-run / develop / production)
- W11-2: REPORT-R1 **LINE delivery preview** + send_guard helper (= **実 LINE
  送信なし**、token 不参照、preview / dry-run / template 中心)
- W11-3: sub-D2.3.x **runner 別** plan refine (= f100/f101/f111/f119 個別)
- W11-4: 全体 read-only / secret / LINE guard audit (= Wave 1-10 横断)

## Wave 11 構成 (= 8 sub-task)

| sub | lane | task | 成果物 |
|-----|------|------|--------|
| W11-1a | 本線 | dry-run 確認 plan | vault doc |
| W11-1b | 本線 | develop apply plan | vault doc |
| W11-1c | 本線 | production apply plan | vault doc |
| W11-2 | L3+L2 (Codex) | LINE delivery preview impl + tests | fire code + tests |
| W11-2b | L4 (Codex) | LINE delivery preview audit | audit report |
| W11-3 | 本線 | sub-D2.3.x 4 runner plan refine | vault docs (= W10-5 拡張) |
| W11-4 | L4 (Codex) | 全体 guard audit | audit report |

## W11-1 MIG-R1 apply plan 3 分割

### W11-1a: dry-run 確認 plan

3 環境 (= production / develop / staging) で **--dry-run 実行**:
- column 存在状況を確認
- production: 不在 → 「will alter」
- develop: 不在 → 「will alter」
- staging: 既存 (= W9-1c で適用済) → 「skip_already_exists」

dry-run は **HQ approve marker (= env F286_MIG_R1_HQ_APPROVE) 必須** (= W10-1a-fix
で常時 enforce)。

期待:
- production / develop は dry-run でも **status="dry_run_would_alter"**
- staging は **status="skip_already_exists"**
- DB write 0 / mtime unchanged 全 環境

### W11-1b: develop DB apply plan

develop に migration 適用:
- env `F286_MIG_R1_HQ_APPROVE=develop` 設定
- `--db-label develop --db-path ~/fire/data/fire.develop.db --write` 実行
- 期待: ALTER 1 回、column 追加
- production DB mtime unchanged 確認
- staging DB mtime unchanged 確認

**実行は別 HQ 明示承認必須**。

### W11-1c: production DB apply plan

production に migration 適用:
- env `F286_MIG_R1_HQ_APPROVE=production` 設定
- `--db-label production --db-path ~/fire/data/fire.db --write` 実行
- 期待: ALTER 1 回
- develop / staging DB mtime unchanged 確認
- backup 推奨 (= production はメインデータ)

**実行は別 HQ 明示承認必須**。

## W11-2 REPORT-R1 LINE delivery preview impl

### scope

LINE delivery 機能の **dry-run / preview / template / send_guard** を実装。
**実 LINE 送信なし、token / channel_token 不参照、推奨送信内容を出力するのみ**。

### 新規 module: `fire/report/line_delivery.py`

```python
@dataclass(frozen=True)
class LineDeliveryPreview:
    """LINE 送信 preview (= 実送信なし、内容のみ).

    LINE REPORT 部屋への送信を想定した chunk 化された message preview.
    """
    target_room: str            # masked (= REPORT、生 ID は出さない)
    chunks: list[str]           # message chunk (= 1 message = max chunk_length)
    chunk_length: int           # 各 chunk の最大文字数 (e.g., 1120)
    total_chunks: int           # len(chunks)
    estimated_send_count: int   # 実送信時の API 呼び出し回数
    partial_safe: bool          # partial 配信耐性

@dataclass(frozen=True)
class LineSendGuard:
    """送信前の三段ガード.

    HQ approve marker, recipient masking enforce, send rate limit 等.
    """
    hq_approve_marker_present: bool  # F286_LINE_HQ_APPROVE env
    recipient_id_masked: bool        # 生 ID 露出禁止
    rate_limit_safe: bool            # 1 分以内に同じ chunk 連投なし
    production_only: bool            # production label 経由かどうか
    refuse_reasons: list[str]


def generate_daily_preview(
    daily_report,  # = DailyReport
    chunk_length: int = 1120,
) -> LineDeliveryPreview:
    """daily report の LINE preview 生成 (= 実送信なし)."""

def generate_weekly_preview(
    weekly_report,  # = WeeklyReport
    chunk_length: int = 1120,
) -> LineDeliveryPreview:
    """weekly report の LINE preview 生成."""

def generate_monthly_preview(
    monthly_report,  # = MonthlyReport
    chunk_length: int = 1120,
) -> LineDeliveryPreview:
    """monthly report の LINE preview 生成."""

def _mask_recipient_id(raw_room_id: str) -> str:
    """REPORT 部屋 ID を mask. 生 ID は返さない."""
    # 例: "Ucabc..." → "REPORT (***)"

def evaluate_send_guard(
    preview: LineDeliveryPreview,
    env: dict | None = None,  # default os.environ
) -> LineSendGuard:
    """送信前ガード判定. HQ marker 不在で refuse."""
    # F286_LINE_HQ_APPROVE env, production_only, masked, rate_limit
```

### 新規 runner: `scripts/jobs/run_f286_report_r1_line_preview.py`

```
required:
  --db-label / --db-path / --base-date / --period (daily|weekly|monthly)

optional:
  --output-preview-json <path>     (= preview JSON 出力)
  --chunk-length <int>             (default 1120)
  --dry-run                        (= default、preview のみ stdout 出力)
  --evaluate-send-guard            (= send guard 判定 only、実送信なし)
```

**`--send` flag は実装しない**。実 LINE 送信は本起票 scope 外、別 HQ approve
後に別 sub-task。

### 安全制約

- linebot SDK / requests / aiohttp 不 import (= LINE 送信 API call なし)
- LINE token / channel_token 不参照 (= env 読出 0)
- 生 recipient ID 出力禁止 (= mask 必須)
- preview / template 生成は read-only DB (= W8-3 / W10-2/3 runner 経由)
- subprocess なし / 楽天 / Computer Use なし
- cron 登録なし

### tests

- TestGenerateDailyPreview / Weekly / Monthly
- TestPreviewChunkLength (= 1120 char 上限、boundary)
- TestPreviewIphoneCopyFormat (= 単一コードフェンス + 内部 fence なし)
- TestMaskRecipientId (= 生 ID 露出なし)
- TestEvaluateSendGuardRefuses (= HQ marker 不在で refuse)
- TestEvaluateSendGuardPasses (= marker 設定で pass)
- TestForbiddenImports (= linebot / requests / aiohttp 不 import)
- TestRunnerDryRunOnly (= --send flag 不在、preview のみ生成)

合計 20-40 件追加見込。

## W11-2b LINE delivery preview audit

W11-2 の adversarial 観点 audit:
- 実 LINE 送信不発生の確認 (= mock で send API not called)
- token 不参照確認 (= AST レベル)
- recipient masking enforce
- send_guard refuse 動作
- preview と send_guard の責任分離

## W11-3 sub-D2.3.x runner 別 plan refine

W10-5 で書いた 4 plan を **runner 別に詳細化**:
- HQ approve template の厳密化
- 「runner 別個別 approve / まとめ write 禁止」を明文化
- 各 runner の rollback 手順詳細

具体: W10-5 の 4 plan doc に **追記/置換** ではなく、別 doc として
**HQ approve worksheet** を作成:
- `03_design/F286_DATA_R3_sub_D2_3_f100_HQ_approve_worksheet_2026-05-12.md`
- 同様に f101 / f111 / f119

各 worksheet に:
- HQ approve template の完全版
- 実行前 checklist
- 失敗時の中断条件
- 完了報告 template

## W11-4 全体 guard audit

Codex L4 で **Wave 1-10 成果物全体を read-only / secret / LINE guard 観点で
横断 audit**:
- 全 production code で linebot / requests / aiohttp / 楽天 / Playwright /
  subprocess の import / call を AST レベルで grep
- 全 runner の HQ approve marker 経路を整合性確認
- secret 露出 (= token / channel_token / API key) 経路の最終確認
- staging-only enforce の整合性
- iPhone コピー format の全 markdown output で遵守確認

成果物: `/tmp/codex_wave11/w11-4_global_audit_report.md`

## 安全要件 (= Wave 11 全 ✓ 維持)

- 実 LINE 送信 0 (= W11-2 全 preview 段階)
- W4.1-B F062 経由 smoke 保留継続
- production DB write 0 (= W11-1c plan のみ、実行は別 HQ approve)
- develop DB write 0 (= W11-1b plan のみ、実行は別 HQ approve)
- staging DB write 0
- token / channel_token / secret 参照 0
- 楽天 / 自動発注 / Computer Use / Playwright なし
- cron / launchd / crontab 本番登録 0 (= sub-D3 凍結継続)
- .github/workflows/ 変更 0
- --no-verify 不使用
- scripts/seed_pattern_layer1.py 未接触
- simulation/research_lane/historical_indicators.py 未接触
- TODO Excel 未更新
- Codex 直接 commit 0

## 並列実行方針

- **Phase 1 (= 並列起動)**:
  - 本線: W11-1 (3 docs) + W11-3 (4 worksheet docs)
  - Codex: W11-2 (= L3+L2、LINE preview impl)
- **Phase 2**:
  - Codex: W11-2b (= L4 audit、W11-2 完了後)
  - Codex: W11-4 (= L4 全体 audit、W11-2 と並走可)
- **Phase 3**: 本線 review + split commits + Wave 11 results + log + HQ 報告

最大同時 Codex lane: 2-3 並列。

## 受入基準

- [ ] W11-1 3 plan docs 完成
- [ ] W11-2 LINE preview impl + tests 全 PASS
- [ ] W11-2b audit CRITICAL 0
- [ ] W11-3 4 worksheet docs 完成
- [ ] W11-4 全体 audit 完了 (= CRITICAL 検出があれば即対応)
- [ ] fire develop split commit
- [ ] fire-vault main split commit
- [ ] CLAUDE.md 完了 table 追加
- [ ] 3,895 baseline 維持 + W11-2 新規分追加
- [ ] HQ 1 ブロック報告

## Wave 12 候補

- W12-1: MIG-R1 develop apply 実行 (= 別 HQ approve)
- W12-2: MIG-R1 production apply 実行 (= 別 HQ approve)
- W12-3: REPORT-R1 LINE 実送信 token integration (= 別 HQ approve)
- W12-4: sub-D2.3.x f100/f101/f111/f119 個別 staging write 実行 (= 別 HQ approve × 4)
- 並走候補: FIRE-AUDIT-R1 v1.2 / F271 v1.7 / sub-D3 cron 解凍判定

## 関連リンク

- [[FIRE_CODEX_R1_WAVE10_results|Wave 10 results]]
- [[../03_design/F286_REPORT_R1_daily_pnl_report_generator_2026-05-11|REPORT-R1 design]]
- [[../03_design/F286_DATA_R3_sub_D2_3_f100_smoke_plan_2026-05-12|f100 plan]]
- [[../03_design/F286_DATA_R3_sub_D2_3_f101_smoke_plan_2026-05-12|f101 plan]]
- [[../03_design/F286_DATA_R3_sub_D2_3_f111_smoke_plan_2026-05-12|f111 plan]]
- [[../03_design/F286_DATA_R3_sub_D2_3_f119_smoke_plan_2026-05-12|f119 plan]]
- [[../log]]
