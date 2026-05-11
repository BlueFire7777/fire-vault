---
id: FIRE-CODEX-R1-WAVE4-results
phase: ガバナンス / Codex 並列実装 Wave 4 完了 / R-01-08 整合
priority: 最優先
status: 完了 ★ Wave 4 4 lane 完了 (2026-05-11、6 split commit、Codex CRITICAL 5 件即修正、3,568 PASS)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - FIRE-CODEX-R1 v1.1 / Wave 4 plan
  - HQ Wave 4 approve (= 2026-05-11)
chapter: ガバナンス / R-01-08 / Codex 運用拡張
---

# FIRE-CODEX-R1 v1.1 Wave 4: 4 sub-task 並列投入完了

最終更新: 2026-05-11

## ★ 状態: 完了 (= 4 lane Codex 完了、6 split commit、全 pytest 3,568 PASS)

HQ Wave 4 approve 受領 (= 2026-05-11) を踏まえ、Codex 4 lane を順次
投入。Wave 1/2/3 と同じく本線 Integrator が成果物 review、CRITICAL を
即修正して develop へ split commit。

Wave 4.1 staging smoke は **未承認**、cron 本番登録は **凍結継続**。

## Wave 4 投入結果

| sub  | lane                 | 主要内容                                                   | 状態        | Codex 内 CRITICAL |
|------|----------------------|---------------------------------------------------------|--------------|---------------------|
| W4-1 | L3 Implementation    | F062 send_smoke --record-decisions 統合                  | ✓ 完了 / 33 PASS | 1 件即修正 (exit code 0 維持) |
| W4-2 | L3 Implementation    | 後追い ingest helper 独立 runner                         | ✓ 完了 / 40 PASS | 4 件即修正 (output path / symlink / hardlink 攻撃) |
| W4-3 | L3 Impl + L2 Test    | FIRE-LABEL-R1 新 5 ラベル即時切替                        | ✓ 完了 / 29 件 modified | 0 |
| W4-4 | L1 Design            | F286-DATA-R3-D2 実 fetch / 実 write 設計 doc             | ✓ 完了 / 18KB design draft | 0 |

## fire develop split commit (= HQ 指示通り別 commit)

| commit | 内容 |
|---|---|
| 860d867 | feat(F286-PNL-R2): integrate --record-decisions in F062 send smoke (W4-1) |
| 5e06423 | feat(F286-PNL-R2): add ingest helper runner (W4-2) |
| 66e5c19 | feat(FIRE-LABEL-R1): refresh advisory label vocabulary (W4-3) |
| 1a4d8cd | docs(F286-DATA-R3-D2): real fetch / write integration design (W4-4) (= vault) |
| 4853986 | docs(FIRE-CODEX-R1): add Wave 4 sub-task completion table entries |
| e4f1aea | test(F286-PNL-R2-ingest-helper): whitelist W4-2 runner in db_path consistency test |

合計: fire 5 commit + fire-vault 1 commit = 6 commit。Codex pre-commit
hook 全 OK 通過 (= CRITICAL 即修正後)。

## Codex CRITICAL 即修正履歴 (= 5 件)

### W4-1 CRITICAL #1 (exit code 設計)

- **指摘**: LINE 送信成功 / snapshot 保存失敗時に exit code 1 を返すと
  caller (cron / launchd / 手動再実行) が「全失敗 → 再実行」と判断し、
  LINE が **二重送信** されるリスク。
- **修正**: snapshot 保存失敗時も exit code **0** を維持 (= LINE 送信
  は成功扱い)、output JSON.snapshot_save_result.status='failed' +
  stderr の retry command で別経路通知に変更。

### W4-2 CRITICAL #1 (output path 上書きリスク)

- **指摘**: --output-json / --completion-report が任意 path を取れて
  Path.write_text() で上書きするため、ユーザが `data/fire.db` /
  `data/fire.staging.db` / WAL path を指定すると SQLite を壊す。
- **修正**: `DANGEROUS_OUTPUT_BASENAMES` / `DANGEROUS_OUTPUT_SUFFIXES`
  を追加、output path が DB / WAL / SHM / sqlite 系の path を指定
  できないよう parse_args で refuse。

### W4-2 CRITICAL #2 (symlink 攻撃)

- **指摘**: `fire.staging.db` という symlink を `data/fire.db` に
  向けると basename ガードを迂回。
- **修正**: --db-path 自体の symlink refuse + Path.resolve(strict=False)
  後の basename も fire.staging.db で一致確認。

### W4-2 CRITICAL #3 (hardlink 攻撃)

- **指摘**: symlink でなく hardlink で fire.staging.db と production
  DB を同一 inode にすれば basename ガード + resolve も迂回可能。
- **修正**: os.stat の (st_dev, st_ino) で forbidden DB との同一 inode
  を refuse する第六段ガードを追加。`_DEFAULT_FORBIDDEN_DB_PATHS =
  (Path("data/fire.db"), Path("data/fire.develop.db"))` を定義、cwd
  依存、test では monkeypatch で書き換え。

### W4-2 CRITICAL #4 (output path にも symlink/hardlink 攻撃)

- **指摘**: --output-json も symlink/hardlink で DB を指せる。
- **修正**: `_assert_safe_output_path` に symlink refuse + inode 比較を
  追加。

### 最終的に追加した六段ガード (= W4-2 runner)

1. SnapshotStore コンストラクタ read_only=False 強制
2. --db-label staging のみ許可
3. --db-path basename='fire.staging.db' のみ許可
4. --output-json / --completion-report が DB/WAL/SHM/sqlite path 不可
5. --db-path symlink refuse + Path.resolve() 後の basename 一致
6. --db-path 既存ファイルが forbidden DB と同一 inode (st_dev + st_ino)
   不可、output path も同様

なお、親ディレクトリ symlink 単体は refuse 対象から除外 (= macOS の
/tmp が /private/tmp への symlink である現実に対応、攻撃ベクタは
resolve 後の inode 比較で完全に閉じる)。

## 安全要件 (= Wave 4 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信 (Wave 4 中)                       | 0 通 ✓ (= SEND 件数 4 のまま、R5.6/R5.8) |
| DB write                                       | 0 ✓ |
| production / develop / staging DB mtime         | 全 unchanged ✓ |
| 未承認 staging DB write                        | 0 ✓ |
| token / channel_token / secret 参照            | 0 ✓ |
| .github/workflows/ 変更                       | 0 ✓ |
| --no-verify                                   | 不使用 ✓ |
| scripts/seed_pattern_layer1.py                | 未接触 (mtime May 7 17:39) ✓ |
| simulation/research_lane/historical_indicators.py | 未接触 (mtime May 9 21:10) ✓ |
| TODO Excel                                    | 未更新 ✓ |
| cron / launchd / crontab 本番登録              | 0 ✓ |
| Codex 直接 commit                              | 0 ✓ (= 本線 Integrator が全 commit) |

## 並列効果計測 (= Wave 1/2/3/4 通算)

| Wave | 内容                                          | 実時間  | 本線単独推定 | 短縮率 |
|------|----------------------------------------------|---------|---------------|--------|
| 1    | Audit×2 + Design                              | 25-30 分 | 90-120 分     | 70-75% |
| 2    | Impl + Test + DATA-R3 skeleton + Docs         | 25-30 分 | 120-150 分    | 75-80% |
| 3    | Audit×5                                       | 30-40 分 | 150-180 分    | 70-80% |
| 4    | Impl + Impl + Impl+Test + Design (= 4 lane)   | 50-60 分 (CRITICAL 5 件即修正含) | 180-240 分 | 65-75% |

Wave 4 は CRITICAL 即修正コスト (= 5 件) を含めて 50-60 分、本線単独
180-240 分の **65-75% 短縮**。Codex pre-commit hook が確実に CRITICAL
を捕捉して即修正サイクルが回る (= "2 つの目" 運用が正規化) ことを 4
wave 連続で実証。

## HQ 判断が必要な論点 (= 4 件)

1. **Wave 4 完了 → 次フェーズ進行可否** (推奨: approve)

2. **Wave 4.1 staging smoke 着手判断** (= HQ 別 approve 要):
   - 推奨経路: W4-2 ingest helper 単独 smoke (= LINE 送信 0)
     → W4-1 F062 経由 smoke (= LINE 1 通)
   - 推奨日: 5/12〜5/17 内 (= F282 weekly snapshot 5/18 前)
   - 初回 DDL 込みの ensure_schema 明示承認要

3. **F286-DATA-R3 sub-D2 Impl 起票** (= W4-4 設計 doc を Architect
   approve 後):
   - subprocess 案で実 fetch / 実 write 統合
   - 既存 sub-runner (F100 / F101 / F119 / F111-R4) を順次呼出
   - Wave 5 候補

4. **F286-DATA-R3 sub-D3 (= cron 本番登録) 凍結継続確認**:
   - sub-D2 完成 + 手動運用 1-2 週間観察後、HQ approve 必須
   - Wave 3 sub-4E の 2 CRITICAL (= 案 A 月曜 07:30 + 案 B 固定ログ
     rotation) を反映済の plist 設計

## 次タスク候補

### Wave 4.1 (= staging smoke、HQ 別 approve 後)

- W4.1-A: W4-2 ingest helper 単独 staging smoke
- W4.1-B: W4-1 F062 経由 staging smoke (LINE 1 通)

### Wave 5 (= W4-4 設計 approve 後)

- W5-1: F286-DATA-R3 sub-D2 Impl (= 実 fetch / 実 write 統合)
- W5-2: 関連 tests (= mocked subprocess + 失敗 propagation)
- W5-3: subprocess 呼出 / env 注入 audit

### 並走候補

- FIRE-AUDIT-R1 v1.2 (= AST + docstring context 除外、Wave 1 で
  500+ false positive 発生したため)
- FIRE-OPS-R0 案 1 (= production write 統一、staging-only 設計の根本対応)

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[FIRE_CODEX_R1_WAVE4_plan|Wave 4 plan]]
- [[F286_PNL_R2_runner_record_decisions|W4-1 runner doc]]
- [[F286_PNL_R2_ingest_helper|W4-2 ingest helper doc]]
- [[FIRE_LABEL_R1_advisory_label_refresh|W4-3 FIRE-LABEL-R1 doc]]
- [[F286_DATA_R3_D2_real_fetch_write_plan|W4-4 plan doc]]
- [[../03_design/F286_DATA_R3_D2_real_fetch_write_2026-05-11|W4-4 design draft]]
- [[../log]]
