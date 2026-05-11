---
id: FIRE-CODEX-R1-WAVE7-results
phase: ガバナンス / Codex 並列実装 Wave 7 完了 / R-01-08 整合
priority: 最優先
status: 完了 ★ Wave 7 4 sub-task 完了 (2026-05-11、Codex audit HIGH 1 件検出、3,637 PASS 維持)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 6 (= 完了)
  - HQ Wave 7 推奨 (= 2026-05-11)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 7: plan / design / audit phase (= 全 非 write)

最終更新: 2026-05-11

## ★ 状態: 完了 (= 4 sub-task 完了、全 vault docs、fire code change なし)

HQ Wave 7 推奨後、Architect 判断で 4 sub-task を起票。**全 plan/design/
audit 系**、fire repo に code change なし、DB write 0、LINE 送信 0。

Wave 7 で重要な発見:
- **W7-3 Codex audit が HIGH #1 (TOCTOU race) を検出** → W8-1 staging UPDATE
  smoke は **Wave 8 W8-0-fix 後** に着手する prerequisite が追加された
- Codex W7-3 は CRITICAL 0 / HIGH 1 / MEDIUM 2 / LOW 3、51 PASS

## Wave 7 投入結果 (= 4 sub-task)

| sub  | lane | task | 成果物 | 状態 |
|------|------|------|--------|------|
| W7-1 | L1 Architect | PNL-R3 staging UPDATE smoke plan | vault design doc | ✓ 起票 |
| W7-2 | L1 Architect | DATA-R3 sub-D2.3 fetch/write smoke plan | vault design doc | ✓ 起票 |
| W7-3 | L4 Audit (Codex) | PNL-R3 paper_pnl + runner audit | audit report | ✓ CRITICAL 0 / HIGH 1 |
| W7-4 | L1 Architect | REPORT-R1 設計起票 (= Daily PnL Report) | vault design doc | ✓ 設計起票 |

## fire-vault main split commit 予定

| commit | 内容 |
|---|---|
| (TBD) | docs(FIRE-CODEX-R1): record Wave 7 4 sub-task (plan/design/audit) + log milestone |

## W7-1: PNL-R3 staging UPDATE smoke plan 主要内容

[[../03_design/F286_PNL_R3_staging_update_smoke_plan_2026-05-11|vault doc]]

### 対象ラベル行用意 (= 3 案比較)

| 案 | 内容 | 判定 |
|---|---|---|
| 案 a | 既存 5 row の decision_label を書換 | ❌ 却下 (= 既存 row 不触ルール違反) |
| 案 b | 新規 5 row を INSERT (= 全 label 網羅) | ✅ 推奨 (= 主案) |
| 案 c | production / develop から複製 | ⚠️ 後続検討 |

### 案 b seed runner 仕様

新規 `scripts/jobs/run_f286_pnl_r3_seed_staging_smoke.py`
(= Wave 8 W8-1 着手時に実装、W7-1 plan では未実装):

- INSERT 5 row 固定 (積極/条件/注意/場中/見送り 各 1 + 計算対象 3 / 除外 2)
- `--rollback` flag で seed row 全 DELETE
- 六段ガード再利用 (= staging-only / basename / symlink / hardlink)

### smoke 実行手順 (= 7 step、Wave 8 W8-1 で実行)

1. pre-smoke 状態キャプチャ (= row 数 + hash)
2. seed 実行 (= INSERT 5 row)
3. paper_pnl compute 実行 (= W6-1+2 runner)
4. post-smoke 検証 (= 既存 5 row hash 一致)
5. 新規 5 row の paper_pnl / updated_at 確認
6. rollback (= seed 5 row DELETE)
7. final 検証 (= pre と完全一致)

## W7-2: DATA-R3 sub-D2.3 fetch/write smoke plan 主要内容

[[../03_design/F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|vault doc]]

### 対象 4 runner の --dry-run 強化 plan

| runner | 現状 --dry-run | W7-2-impl-X 提案 |
|--------|--------------|------------------|
| f100_market_data | 不所持 | impl-a で追加 |
| f101_announcements | 不所持 | impl-b で追加 |
| f111_daytrade_selection | 不所持 | impl-c で追加 |
| f119_evaluation | 不所持 | impl-d で追加 |

各 impl 個別 HQ approve 要。staging write は **絶対に発生しない** ため
Wave 8+ で並列実行可。

### --dry-run 統一仕様

```python
parser.add_argument("--dry-run", action="store_true",
    help="Connection probe only (no fetch / no write)")
# runner 本体: if args.dry_run: return DRY_RUN_OK_EXIT
```

### exit code 集約の実 subprocess 動作確認

W6-5 unit test (36 PASS) + sub-D2.3 で実 subprocess 起動して動作確認:
- 全 ok → 0
- 1 件 fail → 1
- 1 件 timeout → 2
- 想定外 → 3

### staging write 必要時の HQ approve (= sub-D2.3.f100/f101/f111/f119)

各 runner の staging write を必要に応じて個別 HQ approve、smoke plan
個別提示。

## W7-3: PNL-R3 audit (= Codex L4) 結果

[[../07_incidents/F286_PNL_R3_audit_2026-05-11|vault audit report]]

### 概要 verdict

| カテゴリ | 件数 | 判定 |
|----------|------|------|
| CRITICAL | 0 ★ | - |
| HIGH | 1 | TOCTOU race |
| MEDIUM | 2 | output path TOCTOU / label silent skip |
| LOW/note | 3 | per-row catch OK / payload safe / 51 PASS |
| tests | 51 PASS | (= 49 と報告したが実 51) |

merge_recommendation: **「Wave 8 W8-1 (staging UPDATE smoke) 着手 NG」**

### HIGH #1: TOCTOU race on `--db-path`

- 該当: `run_f286_pnl_r3_compute_paper_pnl.py:187/227/262/468`
- 内容: `parse_args()` で symlink / hardlink / basename 検査するが、
  `_connect_write()` での write open までに symlink/hardlink 差し替え
  可能。検査時点と open 時点で対象一致の保証が弱い。
- 推奨修正: write 直前 / `_connect_write()` 内で再 stat、可能なら
  `os.open(O_NOFOLLOW)` 相当で fd 取得後 st_dev/st_ino 検証。

### MEDIUM #1: output path TOCTOU

- 該当: `run_f286_pnl_r3_compute_paper_pnl.py:136/348/356/528`
- 内容: `--output-json` / `--completion-report` も parse_args() で
  検査するが、Path.write_text() 直前に再検査しない
- 推奨修正: `_write_outputs()` 直前で再検査、または atomic create
  (`'x'` mode) + temp + os.replace

### MEDIUM #2: decision_label exact match で whitespace/NFKC silent skip

- 該当: `pnl/paper_pnl.py:59`, `run_f286_pnl_r3_compute_paper_pnl.py:329`
- 内容: include label と完全一致のみ、末尾空白 / NFKC 差分 / 不可視文字
  混入で silent skip (= paper_reason='not_target' すら残らない)
- 推奨修正: `strip()` + `unicodedata.normalize('NFKC', ...)`、もしくは
  W8 smoke で audit query (= include/exclude 以外 残件 count)

### Wave 8 W8-0-fix (= W8-1 prerequisite) で対応

- HIGH #1: write 直前再 stat / `_assert_db_path_not_symlink_attack` 再実行
- MEDIUM #1: output path も同様再検査 / atomic create
- MEDIUM #2: NFKC normalize 導入 or smoke audit query

## W7-4: REPORT-R1 設計起票 (= 主案、SIM-R1 後回し)

[[../03_design/F286_REPORT_R1_daily_pnl_report_generator_2026-05-11|vault doc]]

### 選定理由 (= REPORT-R1 主案)

1. F286 PNL R1/R2/R3 でデータ蓄積基盤完成、次は「読み出して集約」が自然
2. Fujiwara への日次/週次/月次 PnL レポートは兼業前提利益最大化に直結
3. F119 Evaluation Agent (= 提案) と相補的、機能棲み分け明確
4. LINE 配信パイプ (F062/F236) 整備済
5. Stage 3 移行ゲート (F241) との補完

### SIM-R1 後回し理由

- 既存 sim 基盤で Stage 2 稼働中、新規 SIM 緊急性低
- データ蓄積期間短く SIM 結果蓄積前提が成立しにくい
- REPORT-R1 で「何を見たいか」が固まってから SIM-R1 設計が手戻り少

### 機能仕様

| 機能 | format | 配信 | 頻度 |
|------|--------|------|------|
| 日次 PnL | Markdown | LINE REPORT + ファイル | 毎営業日 18:00 JST |
| 週次 PnL | Markdown | 同上 | 毎週金曜 18:00 JST |
| 月次 PnL | Markdown | 同上 | 毎月最終営業日 18:00 |

(cron 登録は sub-D3 凍結中、Wave 9+ で別 approve)

### データソース

- advisory_decisions (= F286-PNL-R1)
- advisory_snapshots / rows (= F286-PNL-R2)
- paper_pnl (= F286-PNL-R3、W6 で実装)
- 実 PnL (= F235 楽天約定経由、M+2 以降)
- F210 GoalProgress

### 集約ロジック (= 純関数群、read-only)

- aggregate_daily_advisory
- aggregate_daily_paper_pnl
- aggregate_daily_actual_pnl
- fetch_goal_progress
- render_daily_markdown

すべて DB write しない。

### 受入基準 (= 3 段階)

- 動いた: exit 0 + Markdown 生成
- 機能した: 集計値が DB の SELECT 結果と一致 + iPhone コピー対応 format
- 期待値達成: Fujiwara が「翌日の判断材料になる」と認定 + LINE 配信 +
  F210 進捗トラッキング機能化

## 安全要件 (= Wave 7 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信 (Wave 7 中) | 0 通 ✓ (= SEND 件数 4 のまま) |
| DB writes (production/develop/staging) | 0 ✓ |
| DB mtime production | 5/7 16:12 (= unchanged) |
| DB mtime develop | 5/7 18:14 (= unchanged) |
| DB mtime staging | 5/11 21:25 (= W4.1-A 以降変化なし) |
| token / channel_token / secret 参照 | 0 ✓ |
| 楽天 / 自動発注 / Computer Use / Playwright | なし ✓ |
| 注文価格 / 数量 / 執行指示 helper | 含めない ✓ |
| .github/workflows/ 変更 | 0 ✓ |
| --no-verify | 不使用 ✓ |
| cron / launchd / crontab 本番登録 | 0 ✓ (= sub-D3 凍結継続) |
| W4.1-B F062 経由 smoke | 保留継続 ✓ |
| scripts/seed_pattern_layer1.py | 未接触 ✓ |
| simulation/research_lane/historical_indicators.py | 未接触 ✓ |
| TODO Excel | 未更新 ✓ |
| Codex 直接 commit | 0 ✓ |
| fire repo code change | 0 ✓ (= 全 vault docs only) |

## 並列効果計測 (= Wave 1-7 通算)

| Wave | 実時間 | 本線単独推定 | 短縮率 |
|------|---------|---------------|--------|
| 1 | 25-30 分 | 90-120 分 | 70-75% |
| 2 | 25-30 分 | 120-150 分 | 75-80% |
| 3 | 30-40 分 | 150-180 分 | 70-80% |
| 4 | 50-60 分 | 180-240 分 | 65-75% |
| 5 | 30-35 分 | 150-180 分 | 80% |
| 6 | 40-50 分 | 180-220 分 | 75-80% |
| 7 | 35-45 分 | 150-200 分 | 75-80% |

**7 wave 連続で 65-80% 短縮を達成** ★

## HQ 判断が必要な論点 (= 4 件)

1. **Wave 7 完了 → 次フェーズ進行可否** (推奨: approve)

2. **Wave 8 W8-0-fix 着手判断** (= W7-3 audit 指摘解消):
   - HIGH #1: `_connect_write()` 内 再検査
   - MEDIUM #1: output path 再検査 / atomic create
   - MEDIUM #2: decision_label NFKC normalize or smoke audit query
   - Codex L3+L2 並列で 1 wave、本線統合
   - **W8-1 staging UPDATE smoke の prerequisite**

3. **Wave 8 W8-1 staging UPDATE smoke 実行判断** (= W8-0-fix 後):
   - W7-1 plan を実行、HQ 明示承認必須
   - 案 b (= INSERT 5 row + rollback) 採用

4. **Wave 8 REPORT-R1 implementation 着手判断** (= W7-4 設計を実装):
   - aggregators / daily_report / runner の Codex L3+L2 並列
   - 単独で進行可、W8-0-fix と並走可

## 次タスク候補 (= Wave 8+)

- **W8-0-fix**: PNL-R3 runner TOCTOU prevention + output path hardening +
  NFKC normalize (= W7-3 audit 解消、Codex L3+L2)
- **W8-1**: PNL-R3 staging UPDATE smoke 実行 (= W7-1 plan、HQ 別 approve)
- **W8-impl-aggregators**: REPORT-R1 aggregators.py + tests
- **W8-impl-daily**: REPORT-R1 daily_report + Markdown renderer + runner
- **W8-audit-daily**: REPORT-R1 daily の adversarial audit
- 並走候補: W7-2-impl-a〜d (= 4 runner --dry-run 追加)
- 凍結継続: cron 登録 (= sub-D3) / W4.1-B F062 経由 smoke

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE7_plan|Wave 7 plan]]
- [[FIRE_CODEX_R1_WAVE6_results|Wave 6 results]]
- [[../03_design/F286_PNL_R3_staging_update_smoke_plan_2026-05-11|W7-1 staging UPDATE smoke plan]]
- [[../03_design/F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|W7-2 sub-D2.3 smoke plan]]
- [[../07_incidents/F286_PNL_R3_audit_2026-05-11|W7-3 audit report]]
- [[../03_design/F286_REPORT_R1_daily_pnl_report_generator_2026-05-11|W7-4 REPORT-R1 design]]
- [[../log]]
