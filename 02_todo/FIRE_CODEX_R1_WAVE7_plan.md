---
id: FIRE-CODEX-R1-WAVE7-plan
phase: ガバナンス / Codex 並列実装 Wave 7 起票 / R-01-08 整合
priority: 最優先
status: 起票 ☆ Wave 7 4 sub-task 並列実行中 (2026-05-11)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 6 (= 2026-05-11 PASS / 完了)
  - HQ Wave 7 推奨 (= 2026-05-11、W7-1/W7-2 起票明示 approve、
    W7-3/W7-4 は「Wave 7 推奨」枠)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 7: plan / design / audit phase (= 全 非 write)

最終更新: 2026-05-11

## ★ 状態: 起票 (= 4 sub-task 並列実行中、全 plan/design/audit)

HQ Wave 6 PASS 受領後、Wave 7 推奨 4 sub-task を Architect 判断で
起票。**全 sub-task が plan/design/audit 系 (= fire コード変更なし、
DB write なし、LINE 送信なし)**。

Wave 6 までと違い Wave 7 は実装フェーズではなく **計画/設計/audit
フェーズ**。staging UPDATE smoke / 実 fetch / 実 write は本 Wave で
全 「plan 起票のみ」で停止、実行は HQ 別 approve 後に Wave 8+ で別起票。

## Wave 7 4 sub-task 構成

| sub  | lane | task | 成果物 | HQ 承認状態 |
|------|------|------|--------|------------|
| W7-1 | L1 Architect | PNL-R3 staging UPDATE smoke plan | vault doc | 起票 approve、実行 unapproved |
| W7-2 | L1 Architect | DATA-R3 sub-D2.3 fetch/write smoke plan | vault doc | 起票 approve、実行 unapproved |
| W7-3 | L4 Audit (Codex) | PNL-R3 paper_pnl module + runner audit | audit report | 推奨枠、実行 OK (audit のみ) |
| W7-4 | L1 Architect | REPORT-R1 設計起票 (= REPORT-R1 主案) | vault design doc | 推奨枠、設計起票 OK |

## REPORT-R1 vs SIM-R1 選定理由 (= W7-4)

HQ "REPORT-R1 or SIM-R1 設計起票" の OR は Architect 判断対象。
本 Wave では **REPORT-R1 を主案** として起票、SIM-R1 は次フェーズ
検討枠とする。

### REPORT-R1 推奨理由

1. **データ基盤揃った**: F286-PNL-R1 (decisions) / R-2 (snapshots) /
   R-3 (paper_pnl) が完成、advisory→snapshot→paper_pnl の集約レポート
   出力に必要なデータが揃った
2. **Fujiwara への可視性**: 日次/週次/月次 PnL レポートは兼業前提の
   利益最大化に直結 (= 第 1 章 R-01-04/05 = 3000 万円 2 年目標)
3. **F119 と相補的**: F119 Evaluation Agent は「提案」、REPORT-R1 は
   「定常報告」、機能が重ならない
4. **LINE 配信パイプ整備済み**: F062 / F236 で 5 部屋構成済み、
   REPORT 部屋がレポート送信先として既存
5. **Stage 3 移行ゲート補強**: F241 に Markdown レポートが既に組込
   まれているが、F286 PnL data 統合は未対応 → REPORT-R1 で補完

### SIM-R1 後回し理由

1. F040-F047 Backtest / Replay 既存、F050-F053 Paper Live 既存、
   F241 Stage 3 ゲート既存 → 既存 sim 基盤で十分
2. F286 PnL data を逆に活用した「過去 advisory pattern simulation」は
   面白いが、Stage 2 (Paper Live) で稼働しながら考えるべき段階
3. データ蓄積期間が短い (= R3 完了直後)、まず REPORT-R1 で日次集約 →
   1-2 ヶ月運用 → SIM-R1 検討の流れが自然

## 安全制約 (= Wave 7 全 ✓ 維持)

- **実 LINE 送信なし** (= W4.1-B / REPORT 部屋送信は全 plan 段階のみ)
- **production / develop / staging DB write なし**
  (= W7-1 staging UPDATE smoke は plan のみ、実行は Wave 8+ で別 approve)
- **token / channel_token / secret 参照なし**
- **楽天 / 自動発注 / Computer Use / Playwright なし**
- **cron / launchd / crontab 本番登録なし** (= sub-D3 凍結継続)
- **W4.1-B F062 経由 smoke 保留継続**
- **--no-verify 不使用 / workflow 変更なし**
- **scripts/seed_pattern_layer1.py 未接触**
- **simulation/research_lane/historical_indicators.py 未接触**
- **TODO Excel 未更新**
- **Codex 直接 commit 0** (= 本線 Integrator が全 commit)

## 並列実行方針

| lane | 担当 | 並列性 |
|---|---|---|
| W7-1 | 本線 Architect | 順次 (= 本線で書く) |
| W7-2 | 本線 Architect | 順次 (= 本線で書く) |
| W7-3 | Codex L4 | 並列 (= background 起動) |
| W7-4 | 本線 Architect | 順次 (= 本線で書く、最大の doc) |

Wave 7 は Codex を 1 lane のみ起動 (= W7-3 audit)、残り 3 は本線 Architect
で順次作成。理由: 計画/設計 doc は Architect 文脈 (= プロジェクト全体把握)
が必要、Codex に投げると context shortage で品質低下リスク。

## 各 sub-task 詳細

### W7-1 PNL-R3 staging UPDATE smoke plan

**HQ 制約**:
- W4.1-A の staging rows (5 件) は decision_label='場中監視' で paper_pnl
  対象外 → paper_pnl 対象ラベル行をどう用意するか **smoke plan に明記必須**
- UPDATE 対象は **paper_pnl + updated_at のみ**
- fujiwara_decision / actual_trade / notes / created_at / 既存 row は触らない
- 実行前 HQ 明示承認必須

**plan に含めるべき項目**:
1. 対象ラベル行の用意方法 (3 案比較)
   - 案 a: 既存 W4.1-A 5 件の decision_label を一時的に「積極的買い推奨」に
     書換 → smoke 後 rollback (= 既存 row 不触ルールに **反する**、却下)
   - 案 b: 新規 advisory_decisions row を seed (= 既存 5 件は触らず、追加
     のみ、INSERT) → idempotent な seed runner 別途用意
   - 案 c: production / develop の advisory_decisions から 「積極的買い推奨」
     row を選び staging に複製 (= 真の本番データに近い smoke)
2. 推奨案: **案 b** (= INSERT only、既存 row 不触、明確な lineage)
3. seed runner 仕様
4. smoke 実行手順
5. before/after DB diff 検証手順 (= UPDATE 範囲が paper_pnl + updated_at
   のみであることを確認)
6. rollback 手順 (= INSERT した seed row を DELETE)
7. 実行前 HQ approve template
8. 受容判定 (= paper_pnl 計算成功 N 件 / skip 適切 / 既存 row mtime
   unchanged)

### W7-2 DATA-R3 sub-D2.3 fetch/write smoke plan

**HQ 制約**:
- 実 fetch / 実 write / staging write はまだ未承認
- まず smoke plan / dry-run 強化 / 対象 runner 確認 / exit code 集約確認 を
  進める
- staging write が必要なら別途 HQ 承認

**plan に含めるべき項目**:
1. 対象 runner 一覧 (= W5-1+2 + W6-5 で active 化済み 4 runner + placeholder 解消後の
   f101 / f119 の現状確認)
2. 各 runner の --dry-run option support 状況 grep 確認
3. dry-run 強化案: --dry-run 不所持 runner に --dry-run option を追加する
   案 (= 既存 runner 本体への code change が必要、別 sub-task 切出推奨)
4. exit code 集約の動作確認手順 (= aggregate_dry_run_exit_code() を実 sub
   process で検証)
5. staging write が必要になる条件 (= F100 historical fetch / F101 announcements
   fetch / F111 daytrade selection / F119 evaluation のいずれが staging
   write を要するか棚卸)
6. staging write 案 (= 個別 sub-D2.3.1, sub-D2.3.2, ... に分割、HQ 別 approve)
7. cron 登録は引き続き凍結 (= sub-D3、HQ 別 approve 後)

### W7-3 PNL-R3 audit (Codex L4)

**audit 対象**:
- pnl/paper_pnl.py (= W6-1+2、772 行)
- scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py (= W6-1+2 runner)
- W6-1+2 で適用した CRITICAL fix (= per-row try/except、PaperPnlError /
  ValueError / sqlite3.Error catch)

**audit 観点**:
- W6-3 で既に CRITICAL 0 だが、独立視点で再 audit
- CRITICAL fix の安全性 (= per-row skip で何か穴がないか)
- edge case (= market_prices_daily 欠落 / 営業日跨ぎ / decision_label
  境界値 / F130 ゼロ除算 等)
- race condition (= 複数 runner 同時実行時、idempotent UPSERT 担保)
- forbidden import 不在 (= line-bot-sdk / requests / aiohttp / 楽天 SDK)
- staging-only refuse 経路 (= 六段ガード 各段の bypass 不可)
- read-only enforcement (= UPDATE 以外の SQL 不発行)
- F286 R1/R2 の既存 row 不触担保

### W7-4 REPORT-R1 設計起票

**doc 構成**:
1. REPORT-R1 概要 (= Daily PnL Report Generator)
2. SIM-R1 比較表 + REPORT-R1 選定理由 (= 本 plan §「REPORT-R1 vs SIM-R1 選定
   理由」を継承)
3. 機能仕様
   - 日次 PnL レポート (= advisory_decisions + paper_pnl + actual_pnl)
   - 週次 PnL レポート
   - 月次 PnL レポート
   - 出力 format (= Markdown、F119 Evaluation Agent と同じ system)
   - 配信先 (= LINE REPORT 部屋、別 HQ approve 後)
4. データソース
   - advisory_decisions (= F286-PNL-R1)
   - advisory_snapshots / advisory_snapshot_rows (= F286-PNL-R2)
   - paper_pnl (= F286-PNL-R3、W6 で実装)
   - 実 PnL (= 楽天約定メール経由、M+2 以降)
   - 進捗 (= F210 3000 万円 2 年目標)
5. 集約ロジック (= 純関数群、F119 と同じ approach)
6. 三段+六段ガード (= staging-only / db_label / basename / output path /
   symlink / hardlink)
7. テスト方針 (= 純関数 unit + runner integration、3,637 baseline 維持)
8. cron 連携 (= sub-D3 凍結中、Wave 8+ 別 approve 後に登録)
9. 受入基準 (= 3 段階: 動いた / 機能した / 期待値達成)
10. 関連リンク (= F119 / F210 / F286 全シリーズ / F236 LINE)

**doc 配置**: `03_design/F286_REPORT_R1_daily_pnl_report_generator_2026-05-11.md`

(F286 シリーズに含めるか別シリーズにするかは設計 doc 内で議論。
仮置きで F286-REPORT-R1 prefix で配置、最終命名は Architect 判断)

## 受入基準 (= Wave 7 完了条件)

- [ ] 4 sub-task の vault doc / audit report が全 揃う
- [ ] fire repo に code change なし (= 全 vault docs only)
- [ ] DB writes 0 / LINE sends 0 / secrets 0 / forbidden files 0
- [ ] Codex W7-3 audit で CRITICAL 検出があれば即修正、なければ note のみ
- [ ] fire-vault に split commit (= 4 doc + log + plan + results)
- [ ] CLAUDE.md 完了 table に Wave 7 entry 4 件追加
- [ ] HQ 1 block report 提示

## Wave 8 起票候補 (= HQ approve 後)

- W8-1: PNL-R3 staging UPDATE smoke 実行 (= W7-1 plan を実行、HQ 別 approve 要)
- W8-2: DATA-R3 sub-D2.3.x staging write 実行 (= W7-2 plan を細分化、HQ 別 approve 要)
- W8-3: REPORT-R1 implementation (= W7-4 設計起票を実装、Codex L3+L2 並列)
- 並走候補: FIRE-AUDIT-R1 v1.2 (= AST + docstring context 除外強化)

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE6_plan|Wave 6 plan]]
- [[FIRE_CODEX_R1_WAVE6_results|Wave 6 results]]
- [[../03_design/F286_PNL_R3_paper_pnl_simulator_hook_2026-05-11|F286-PNL-R3 design]]
- [[F286_PNL_R3_paper_pnl_simulator_hook|F286-PNL-R3 完了マーカー]]
- [[../log]]
