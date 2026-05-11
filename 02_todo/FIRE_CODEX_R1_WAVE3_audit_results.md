---
id: FIRE-CODEX-R1-WAVE3
phase: ガバナンス / Codex 並列実装 Wave 3 / R-01-08 整合
priority: 最優先
status: 完了 ★ Wave 3 完了 (2026-05-11、5 audit 全完、CRITICAL は sub-D3 着手前条件のみ)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 (= 設計)
  - Wave 1 (= audit + design)
  - Wave 2 (= impl + test + DATA-R3 skeleton + docs)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 3: Audit 5 件完了

最終更新: 2026-05-11

## ★ 状態: 完了 (= 5 audit 全完、CRITICAL は sub-D3 前提条件のみ、Wave 4 / 次フェーズ進行可)

Wave 2 で実装した F286-PNL-R2 snapshot module / F286-DATA-R3 skeleton
を develop へ split commit した後、Codex 5 lane で adversarial /
integration / smoke plan / safety / cron checklist の audit を実施。
全 audit が read-only、CRITICAL は **sub-D3 (= 将来の cron 登録)
前提条件 2 件** のみで本 Wave 内の実害なし。

## fire develop split commit 結果

| commit | 内容 | files | tests |
|---|---|---|---|
| cb17a7f | feat(F286-PNL-R2): snapshot+schema+tests | pnl/snapshot.py 新規 + pnl/schema.py 拡張 + tests/pnl/test_snapshot.py | 50 PASS |
| e9c41c3 | feat(F286-DATA-R3): daily refresh skeleton+tests | scripts/jobs/run_f286_data_r3_daily_refresh.py + tests | 16 PASS |
| b4d4022 | docs(F286-PNL-R2,F286-DATA-R3): CLAUDE.md 完了テーブル | CLAUDE.md +2 行 | N/A |

- 全 pytest スイート: **3,515 PASS** (= 3,449 + 66 新規)
- Codex pre-commit hook (= cb17a7f / e9c41c3): OK 通過
- DB mtime 全 unchanged / LINE SEND 4 のまま / token 参照 0 /
  workflow 変更 0 / --no-verify 不使用 / forbidden files 未接触

## Wave 3 audit 5 件結果

| sub | lane | 監査対象 | report | CRITICAL |
|---|---|---|---|---|
| sub-4A | L4 Audit | F286-PNL-R2 adversarial review (snapshot / schema / tests) | /tmp/f286_pnl_r2_sub4A_audit_report.md (11.9KB) | なし |
| sub-4B | L4 Audit | F062 runner integration design review | /tmp/f286_pnl_r2_sub4B_audit_report.md (4.6KB) | なし |
| sub-4C | L4 Audit | staging write smoke plan の安全確認 | /tmp/f286_pnl_r2_sub4C_smoke_plan_report.md (10.9KB) | なし |
| sub-4D | L4 Audit | DATA-R3 skeleton safety review | /tmp/f286_data_r3_sub4D_safety_audit_report.md (7.8KB) | なし |
| sub-4E | L4 Audit | Cron 登録前 checklist 整備 | /tmp/fire_cron_r1_sub4E_checklist_report.md (11.3KB) | **2 件** |

### sub-4E CRITICAL 2 件 (= sub-D3 着手前条件、Wave 3 内では実害ゼロ)

1. **F282 weekly snapshot と F286-DATA-R3 daily refresh の月曜実行順序矛盾**
   - 月曜 06:00 JST に daily refresh が staging を更新
   - 月曜 07:00 JST に F282 snapshot が **staging を production DB で
     上書き** → daily refresh の成果が消える
   - 解決案 (sub-D3 着手前に HQ 決定):
     - 案 A: daily refresh を月曜 07:30 (= F282 後) にずらす
     - 案 B: F282 weekly snapshot 自体を見直す (= FIRE-OPS-R0 案 1 と整合)
     - 案 C: 月曜は daily refresh skip、火-金のみ実行
2. **logs/cron/f286_data_r3_<date>.log は launchd StandardOutPath
   だけでは実現困難**
   - 日付付きログを runner/wrapper 側で作るか、固定ログ + rotation か、
     sub-D3 前に設計決定が必要

両 CRITICAL は **sub-D3 (= cron 登録) 着手前提条件** であり、現在
develop に commit 済の DATA-R3 skeleton には影響なし。本 Wave 3 で
cron 本番登録を実施していないため即時実害ゼロ。

### sub-4A 推奨対応 (= 非 CRITICAL、将来対応候補)

1. **symlink 対策**: SnapshotStore の write mode で
   `db_path.resolve(strict=False)` の許可 root 確認を追加、symlink は
   refuse (= 偽装ベクタの 4 段目強化候補)
2. **INSERT ON CONFLICT DO NOTHING**: advisory_decisions seed を
   atomic INSERT IGNORE 系に変更し、並行同一 ingest を正常 idempotent
   に保証 (= 現状は SELECT → INSERT/UPDATE で race condition 微小)

### sub-4B 主要内容 (CRITICAL なし、推奨設計案)

- F062 runner 統合フロー: payload 生成 → LINE 送信 → snapshot save の
  順序、HQ #3/#4/#7 が明示的に組まれている
- send_id 採番タイミング: preview runner が payload に send_id を
  埋め込み、send_smoke が読み出す案を推奨
- 後追い ingest helper: `scripts/jobs/run_f286_pnl_r2_ingest_snapshot.py`
  または `pnl/snapshot_ingest.py` (= 設計判断)
- exit code: 1=部分失敗 (LINE OK / save NG) / 2=完全失敗 / 3=safety
  violation

### sub-4C smoke plan 注記

- 初回 staging smoke は **DDL を含む** (= ensure_schema で
  advisory_snapshots / advisory_snapshot_rows table を作る)
- HQ approve 前にこの「初回 DDL」も明示承認が必要
- smoke 推奨日: F282 weekly snapshot 直前の 5/17 (土) 夕方 (= 消える
  前提で残量小、または 5/12 〜 5/17 内の任意の日)

### sub-4D DATA-R3 safety (= CRITICAL なし)

- skeleton 明示性: CLI help / docstring で「実 fetch / cron 登録は
  別 sub-task」明示済 ✓
- 三段ガード再利用: pnl.storage 経由で適用済 ✓
- cron / launchd 呼出: 含まれない ✓
- forbidden import: なし ✓
- subprocess / 既存 runner chain: skeleton には含まれない ✓ (= sub-D2
  で別実装)

## 安全要件 (= Wave 3 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信 (Wave 3 中)                       | 0 通 ✓ (= SEND 件数 4 のまま) |
| DB write                                       | 0 ✓ (= production / develop / staging 全 mtime unchanged) |
| token / channel_token / secret 参照            | 0 ✓ |
| .github/workflows/ 変更                       | 0 ✓ |
| --no-verify                                   | 不使用 ✓ |
| scripts/seed_pattern_layer1.py                | 未接触 ✓ |
| simulation/research_lane/historical_indicators.py | 未接触 ✓ |
| TODO Excel                                    | 未更新 ✓ |
| cron / launchd / crontab 本番登録              | 0 ✓ (= 設計のみ) |
| staging smoke 実施                             | 0 ✓ (= HQ approve 後の Wave 4 候補) |
| Codex 直接 commit                              | なし ✓ |
| Audit 中の fire リポ source 変更                | 0 行 ✓ (= 全 read-only) |

## 並列効果計測 (= Wave 3)

| 項目 | 値 |
|---|---|
| Wave 3 実時間              | 約 30-40 分 (= 5 lane 順次 audit) |
| 本線単独推定 (= 5 観点 audit) | 150-180 分 |
| 速度向上                   | 約 70-80% 短縮 |

Wave 1 / Wave 2 / Wave 3 通して **本線単独 比 70-80% の安定短縮** を
3 wave 連続で達成、Codex 実装部隊化の効果が確実に確認できた。

## HQ 判断が必要な論点 (= Wave 4 / 次フェーズ)

1. **Wave 3 audit クリア → 次フェーズ進行可否**:
   - sub-4A/B/C/D は CRITICAL なし
   - sub-4E の 2 CRITICAL は sub-D3 着手前条件、Wave 3 内の実害なし
   - 推奨: 次フェーズ進行 approve
2. **F286-PNL-R2 staging smoke の実施判断** (= sub-4C plan ベース):
   - 推奨日: 5/12〜5/17 内の任意 (= F282 weekly snapshot 5/18 前)
   - 初回 DDL 込みの ensure_schema を含む明示承認が必要
3. **F286-PNL-R2 sub-runner** (= F062 --record-decisions 統合) を Wave 4
   として起票するか
4. **後追い ingest helper** (= sub-4B 推奨) の sub-task 分離
5. **F286-DATA-R3 sub-D2** (= 実 fetch / 実 write 統合) の優先度
6. **F286-DATA-R3 sub-D3** (= cron 本番登録) は **2 CRITICAL 解決後**
   のみ進める方針確認
7. **FIRE-LABEL-R1 起票** (= 新ラベル方針対応、本 Wave 3 と並行候補)

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1 設計]]
- [[FIRE_CODEX_R1_WAVE1_results|Wave 1 結果]]
- [[F286_PNL_R2_advisory_snapshot_auto_ingest|Wave 2 結果 + F286-PNL-R2 まとめ]]
- [[FIRE_LABEL_R1_advisory_label_refresh|FIRE-LABEL-R1 (= 新ラベル方針対応、別途起票)]]
- [[../log]]
