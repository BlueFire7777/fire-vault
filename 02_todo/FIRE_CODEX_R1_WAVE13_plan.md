---
id: FIRE-CODEX-R1-WAVE13-plan
phase: ガバナンス / Wave 13 起票 (= SCHEMA-R1 impl + tests + audit + apply plans) / R-01-08
priority: 最優先
status: 起票 ☆ Wave 13 4 sub-task + 本線 並列実行中 (2026-05-12)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 12 (= 完了)
  - HQ Wave 13 approve (= SCHEMA-R1 implementation + tests + audit + apply plans、
    schema 統合派採用、2026-05-12)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 13: SCHEMA-R1 impl + audit + develop/production apply plans

最終更新: 2026-05-12

## ★ 状態: 起票 (= 4 sub-task、impl + audit + 2 plan、fire code change W13-1 のみ)

HQ Wave 13 approve 受領後、**schema 統合派 (= 案 a)** で進行:
- paper_reason を含む 22 列の advisory_decisions table 完全定義
- F286-PNL-R3-MIG-R1 (paper_reason 単体 ALTER) は **不要化** (= staging のみ
  既存 paper_reason 維持、production/develop には schema 統合で同時作成)

## Wave 13 構成 (= 4 sub-task)

| sub | lane | task | 成果物 |
|-----|------|------|--------|
| W13-1 | L3+L2 (Codex) | SCHEMA-R1 implementation + tests | fire code |
| W13-1b | L4 (Codex) | SCHEMA-R1 audit | audit report |
| W13-2 | 本線 | develop apply plan | vault doc |
| W13-3 | 本線 | production apply plan + backup plan | vault doc |

## W13-1 SCHEMA-R1 implementation 設計

### 新規 script: `scripts/setup/migrate_advisory_decisions_full.py`

仕様:
- `CREATE TABLE IF NOT EXISTS advisory_decisions` + 22 列 + PK + 2 indexes + CHECK
- staging schema を **完全に再現** (= W12-4 vault doc に template SQL)
- idempotent (= 既存 table があれば skip)
- schema 不一致時は **warning + skip** (= 上書きしない)
- HQ approve marker (= F286_SCHEMA_R1_HQ_APPROVE) 必須 (production/develop)
- --db-label / --db-path / --dry-run / --write

action 戻り値:
| 状況 | action |
|------|--------|
| table 不在 + dry-run | "dry_run_would_create" |
| table 不在 + write | "created" |
| table 既存 + schema 一致 | "skip_already_exists" |
| table 既存 + schema 不一致 | "schema_mismatch_warning_skipped" |

### tests

- TestSchemaR1CreateInDevelop / Production / Staging (= tmp DB)
- TestSchemaR1Idempotent (= 2 回連続 CREATE で skip)
- TestSchemaR1SchemaMismatchWarning
- TestSchemaR1RequiresHQMarker (= production / develop)
- TestSchemaR1StagingNoMarker
- TestSchemaR1SQLLimited (= CREATE TABLE / CREATE INDEX 以外発行しない)
- TestSchemaR1Indexes (= 2 indexes 作成確認)
- TestSchemaR1CheckConstraints (= fujiwara_decision / actual_trade)
- TestSchemaR1PrimaryKey (= (advisory_id, code) 複合 PK)

合計 15-25 件追加見込。

## W13-2 develop apply plan

### 目的

develop DB (= ~/fire/data/fire.develop.db) に advisory_decisions table を
CREATE する plan。staging からの完全 schema 移植。

### 実行手順 (= 9 step)

1. pre-mtime 記録 (= 全 3 DB)
2. develop schema 記録 (= advisory_decisions 不在確認)
3. develop dry-run 再確認 ("dry_run_would_create" 期待)
4. develop apply 実行 (= CREATE TABLE + 2 INDEXES)
5. develop post-schema 確認 (= 22 列 + PK + 2 indexes 完成)
6. develop row count 0 確認 (= 新規 table、空のはず)
7. production / staging mtime unchanged 確認
8. idempotent 再実行 ("skip_already_exists" 期待)
9. HQ 報告

### 安全制約

- CREATE TABLE IF NOT EXISTS のみ
- INSERT / UPDATE / DELETE 不発行
- staging / production mtime unchanged enforce
- LINE 送信 0 / token 0 / cron 0

## W13-3 production apply plan + backup plan

### backup plan

```bash
# 必須、最大級の慎重さ
mkdir -p /tmp/w14_X/
cp ~/fire/data/fire.db \
   /tmp/w14_X/fire.db.pre_schema_r1_$(date +%Y%m%d_%H%M%S)
sha256sum ~/fire/data/fire.db /tmp/w14_X/fire.db.pre_schema_r1_*
```

backup ファイル整合性確認、保存場所 / 期間 HQ 指示待ち。

### apply 手順 (= 10 step、最も慎重)

1. backup 作成 (= 必須、上記)
2. pre-mtime + schema 記録 (= 全 3 DB)
3. production schema 確認 (= advisory_decisions 不在)
4. production dry-run 再確認
5. production apply 実行 (= CREATE TABLE + INDEXES)
6. production post-schema 確認
7. production row count 0 確認
8. develop / staging mtime unchanged 確認
9. idempotent 再実行確認
10. HQ 報告

### 安全制約

- W13-2 develop apply 完了後のみ実行
- backup 必須
- production DB 使用他プロセスとの排他確認
- CREATE TABLE IF NOT EXISTS のみ
- staging / develop touch 禁止

## 安全制約 (= Wave 13 全 ✓ 維持)

- 実 LINE 送信 0
- W4.1-B F062 経由 smoke 保留継続
- production DB write 0 (= W13-3 plan のみ、実行は別 HQ approve)
- develop DB write 0 (= W13-2 plan のみ、実行は別 HQ approve)
- staging DB write 0
- token / channel_token / secret 参照 0
- 楽天 / 自動発注 / Computer Use / Playwright なし
- cron / launchd / crontab 本番登録 0
- .github/workflows/ 変更 0
- --no-verify 不使用
- scripts/seed_pattern_layer1.py 未接触
- simulation/research_lane/historical_indicators.py 未接触
- TODO Excel 未更新
- Codex 直接 commit 0

## 並列実行方針

- Phase 1: W13-1 Codex impl + 本線 W13-2/W13-3 plan 並走
- Phase 2: W13-1b audit (= Codex L4、W13-1 完了後)
- Phase 3: 本線 review + commits + Wave 13 results + log + HQ 報告

## 受入基準

- [ ] W13-1 SCHEMA-R1 impl + tests 全 PASS
- [ ] W13-1b audit CRITICAL 0
- [ ] W13-2 develop apply plan vault doc 完成
- [ ] W13-3 production apply plan + backup plan vault doc 完成
- [ ] fire develop split commit
- [ ] fire-vault main commit
- [ ] CLAUDE.md 完了 table 追加
- [ ] 3,952 baseline 維持 + 新規分追加
- [ ] HQ 1 ブロック報告

## Wave 14 候補

- W14-1: SCHEMA-R1 dry-run 3 環境実行 (= 別 HQ approve)
- W14-2: develop apply 実行 (= 別 HQ approve)
- W14-3: production apply 実行 (= 別 HQ approve、backup 必須)
- W14-4: F286-DATA-R3 sub-D2.3.x 個別 staging write (= 個別 HQ approve)
- 並走候補: REPORT-R1 LINE 実送信 token integration / sub-D3 cron

## 関連リンク

- [[FIRE_CODEX_R1_WAVE12_results|Wave 12 results]]
- [[../03_design/F286_PNL_SCHEMA_R1_advisory_decisions_full_migration_2026-05-12|SCHEMA-R1 起票]]
- [[../log]]
