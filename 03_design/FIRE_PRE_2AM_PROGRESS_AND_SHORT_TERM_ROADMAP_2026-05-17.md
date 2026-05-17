# FIRE 2 時前タスク進捗 + 短期ロードマップ (2026-05-17)

doc_id: FIRE-PRE-2AM-PROGRESS-AND-SHORT-TERM-ROADMAP-2026-05-17
status: progress-log + planning
HQ marker: HQ_APPROVE_PRE_2AM_ROADMAP_TODO_UPDATE
時刻: 2026-05-17 21:2x JST (= 5/18 02:00 F282 fire まで 約 4 時間 30 分)
related:
- [[JQUANTS_ENV_SUPPLY_PREFLIGHT_R1_2026-05-17]]
- [[STAGING_DB_RESTORE_FROM_BACKUP_RECENT_2026-05-17]]
- [[F111_DATA_FRESHNESS_GATE_R1_2026-05-17]]
- [[F282_POST_SNAPSHOT_STAGING_AUDIT_RUNNER_R1_2026-05-17]]
- [[F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17]]
- [[F062_OPS_SUMMARY_FRESHNESS_ADAPTER_DESIGN_2026-05-17]]
- [[NIGHT_R0_READ_ONLY_BATCH_DESIGN_2026-05-17]]

---

## §1 2 時前タスク 8 件 進捗表

| # | wave / task | 状態 | 反映先 | 関連 SHA / 成果物 |
|---|---|---|---|---|
| 1 | PR #4 create + scope audit | ✓ 完了 | fire repo develop / PR #4 | 5 commit (= DATA-R1 / Freshness Gate / F282 audit / HIGH fix 含む) |
| 2 | PR #4 merge + post-merge smoke | ✓ 完了 | fire repo **main** | merge commit `b324f7e` / 155 PASS / smoke (= FAIL + PROCEED_FINANCIALS_RETRY) |
| 3 | F282 post-snapshot audit runner | ✓ 完了 | fire repo main (= PR #4) | `scripts/jobs/run_f282_post_snapshot_staging_audit.py` (= 4 decision、28 tests) |
| 4 | W2-C overlay role 再分類 設計 | ✓ 完了 | fire-vault main | commit `f45814c` / 8 role + 9 step ruleset + 60 マス mapping |
| 5 | F062 / Ops Summary adapter 設計 | ✓ 完了 | fire-vault main | commit `ab86afc` / 入力 6 + 出力 1 schema + 9 step ruleset + 25 マス |
| 6 | J-Quants wrapper 運用 設計 | ✓ 完了 | fire-vault main (= 前 wave) | [[JQUANTS_ENV_SUPPLY_PREFLIGHT_R1_2026-05-17]] / wrapper 方式 `set -a; source .env; set +a` |
| 7 | NIGHT-R0 read-only batch 設計 | ✓ 完了 | fire-vault main | commit `86fb710` / 9 step + 6 status + 5 layer + 8 wave |
| 8 | TODO / roadmap 更新 (= 本 wave) | ✓ 完了 | fire-vault main | 本 doc + 別 commit |

(= 8 件全完了、5/18 02:00 自動 fire を operator 操作 0 で迎えられる状態)

---

## §2 main / develop 反映状況

### §2.1 fire repo (= ~/fire、BlueFire7777/fire)

main HEAD: `b324f7e` Merge pull request #4 from BlueFire7777/develop
develop HEAD: `2ae2317` fix(F111): harden freshness gate path guard
ahead/behind (= origin/main vs main): 0/0 ✓
3 DB md5 (= 直近 wave 期間中 完全不変):
- `data/fire.db`: b1df4673e5c3645fbe2c5f490ffac043
- `data/fire.develop.db`: 085799daf117e59b05d4b9d9aeb7662d
- `data/fire.staging.db`: 71a63a19694385db4344246be60a2f91

PR #4 で main 反映済 (= 9 file、production code + tests + docs):
- `scripts/jobs/run_f111_data_freshness_gate.py` (= R1 / R1.1 / HIGH fix)
- `scripts/jobs/_f111_data_freshness_gate.py` (= 純 helper)
- `scripts/jobs/run_f282_post_snapshot_staging_audit.py`
- `agents/jquants_daily_refresh.py` (= DATA-R1 enablement)
- `tests/scripts/jobs/test_run_f111_data_freshness_gate.py` (= 56 PASS)
- `tests/scripts/jobs/test_run_f282_post_snapshot_staging_audit.py` (= 28 PASS)
- `tests/agents/test_jquants_daily_refresh.py` (= 71 PASS、拡張)
- `.gitignore` (= rollback artifact ignore 強化)
- (= 計 9 file changed)

### §2.2 fire-vault (= ~/fire-vault、BlueFire7777/fire-vault)

最新 5 commits (= 直近 design wave):
- `86fb710` docs(NIGHT): design R0 read-only batch
- `ab86afc` docs(F062): design freshness adapter for ops summary
- `f45814c` docs(F111): design W2-C overlay role reclassification
- `7b0f9d2` docs(F282): document post-snapshot staging audit runner
- `460bcb5` docs(F111): update data freshness gate hardening

W2-C / F062 adapter / NIGHT-R0 = **設計 doc のみ fire-vault 反映済**
(= 実装 wave は別、本 doc §4 参照)

---

## §3 5/18 02:00 後 最短ロードマップ

```
[5/17 21:xx 〜 5/18 01:59]  待機 (= operator idle、Mac mini 通常)
       ↓
[5/18 02:00]  F282 weekly-snapshot 自動 fire
              (= launchd plist 不変、operator 操作 0)
       ↓
[5/18 朝 (= operator 起動時刻)]
   step 1: F282 post-snapshot audit 実行 (= main 反映済 runner)
       ↓
   step 2: decision 別分岐
       │
       ├─ PROCEED_FINANCIALS_RETRY (= 期待 happy path)
       │   ↓
       │   HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY
       │   (= wrapper 方式で env 供給後 financials retry)
       │   ↓
       │   HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH (= 案 A.5)
       │   ↓
       │   HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH (= 案 B)
       │   ↓
       │   HQ_APPROVE_W2B_RE_RUN_FRESH_DATA (= 案 C)
       │   ↓
       │   W2-C 設計 re-review (= fresh vol_mom/srs で 7 銘柄再評価)
       │
       ├─ RESTORE_FROM_BACKUP_REQUIRED (= staging 上書き検出)
       │   ↓
       │   HQ_APPROVE_STAGING_DB_RESTORE_FROM_BACKUP_RECENT (= atomic restore)
       │   ↓ (restore 後 audit 再実行)
       │   PROCEED 分岐へ
       │
       ├─ INVESTIGATE (= 想定外 / error patterns / WAL violation)
       │   ↓
       │   即停止 → logs/cron/weekly-snapshot.* + audit reasons 確認
       │   → HQ 判断 → 必要なら手動 audit / restore / retry 別 wave
       │
       └─ WAIT_FOR_SNAPSHOT (= 02:00 過ぎていれば異常)
           ↓
           launchd 状態確認、F282 plist / launchctl logs 確認
```

### §3.1 各 HQ marker のスコープ (= 1 wave 1 marker)

| HQ marker | scope |
|---|---|
| HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY | wrapper 経由 env 供給 → financials retry (= staging のみ) |
| HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH | fresh financials を入力に derived 再生成 (= staging) |
| HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH | fresh derived を入力に signal 永続化 (= staging) |
| HQ_APPROVE_W2B_RE_RUN_FRESH_DATA | fresh signal で W2-B classification rerun (= staging read-only smoke) |
| (任意) HQ_APPROVE_W2C_DESIGN_RE_REVIEW | fresh W2-B 結果で W2-C 設計 doc 更新 (= vault commit) |

---

## §4 並行開発候補 (= 3 wave 設計済み、独立着手可)

| wave | 設計 doc | 実装 LOC 目安 | 実装 wave 数 | DB write 依存 |
|---|---|---|---|---|
| W2-C 実装 | [[F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17]] | production ~490 + tests ~400 = ~890 | 7 (= W2-C-1〜7) | **要 fresh W2-B** (= §3 案 C 完了後) |
| F062 / Ops Summary adapter 実装 | [[F062_OPS_SUMMARY_FRESHNESS_ADAPTER_DESIGN_2026-05-17]] | production ~640 + tests ~500 = ~1,140 | 7 (= ADAPTER-1〜7) | 不要 (= pure adapter、即着手可) |
| NIGHT-R0 read-only batch 実装 | [[NIGHT_R0_READ_ONLY_BATCH_DESIGN_2026-05-17]] | production ~1,030 + tests ~700 = ~1,730 | 8 (= NIGHT-R0-1〜8) | 不要 (= read-only、即着手可) |

### §4.1 着手順序の推奨

```
[5/18 02:00 後 (= fresh data 取得経路 完了前)]
   1. F062 / Ops Summary adapter ADAPTER-1〜5 (= pure / read-only)
   2. NIGHT-R0 NIGHT-R0-1〜5 (= pure / read-only)
   (= 2 wave を並行可、互いに独立)

[5/18 朝 (= retry / regen / W2-B rerun 完了後)]
   3. W2-C-1〜5 (= 設計に fresh data 反映後の調整含む)
   4. NIGHT-R0-6 (= ADAPTER-1〜5 完了後 hook 接続)
   5. W2-C / ADAPTER / NIGHT-R0 の Codex 8 lane CLI review (= 各 -6 / -7)
   6. 各 wave PR + main merge (= 各 -7 / -8)

[全実装後 (= 1-3 週間 想定)]
   7. NIGHT-R0-LAUNCHD-1 (= launchd 登録、自動 fire 切替)
   8. R0 READY ≥7 日連続 → NIGHT-R1 設計開始
```

### §4.2 DB write 系の gating

| wave | 5/18 02:00 audit 結果待ち |
|---|---|
| ADAPTER-1〜7 | 不要 (= 全 read-only) |
| NIGHT-R0-1〜8 | 不要 (= 全 read-only) |
| W2-C-1〜7 | 不要 (= 全 read-only)、ただし設計の re-review は fresh W2-B 推奨 |
| financials retry | **PROCEED 待ち必須** |
| derived regen | retry 完了待ち |
| signal regen | derived regen 完了待ち |
| W2-B rerun | signal regen 完了待ち |

---

## §5 Codex 活用方針 (= 本 wave 以降の運用)

### §5.1 通常実装 (= Codex 6 lane 以上)

新規 production code / 既存 production 改修 / tests 拡張 等:
- Lane A: 機能正しさ / 設計準拠
- Lane B: tests / regression / edge case
- Lane C: 安全 (= no-write 観点 / API / LINE / token)
- Lane D: code review (= 既存パターンとの整合)
- Lane E: 失敗系 / rollback / error handling
- Lane F: next-wave 分解 / 残課題

### §5.2 DB / restore / refresh / merge 系 (= Codex 8 lane 相当)

DB 書込 / staging restore / financials refresh / production-develop merge 等:
- 上記 6 lane + 追加 2 lane:
- Lane G: operational risk / rollback readiness / blast radius
- Lane H: 2 時前 / 02:00 fire / weekly snapshot 等の時刻整合

### §5.3 Codex 活用範囲 (= 設計から rollback まで)

| 用途 | Codex 活用 |
|---|---|
| 設計 doc レビュー | self-audit 表で代替可、CLI 並列実行は CRITICAL 候補時のみ |
| 実装案 提案 | 複数案を Codex に並列生成させ、トレードオフ比較 |
| tests 設計 | edge case / failure mode / coverage gap を Codex に網羅 |
| 安全監査 | DB / API / LINE / token / launchd の no-write enforcement 確認 |
| rollback 手順 | git revert / DB restore / launchd 停止の手順をレビュー |
| next-wave 分解 | 大 wave を 3-8 sub-wave に分解、各 wave に独立 HQ marker 案 |

### §5.4 Codex pre-commit hook の遵守

- 全 commit で `scripts/hooks/pre-commit` 通過必須
- `--no-verify` 禁止 (= 唯一の例外: `.github/workflows/*.yml` 事前承認時のみ `FIRE_ALLOW_WORKFLOW_CHANGE=1`)
- CRITICAL 検出 → 修正してから再 commit (= 別 commit 禁止)
- HIGH 検出 → 修正推奨、別 wave 切出し可

---

## §6 安全状態 (= 本 wave 終了時点)

| 項目 | 累計 |
|---|---|
| DB write | 0 |
| staging write | 0 |
| production-develop write | 0 |
| financials refresh / retry | 0 |
| derived regen / signal regen | 0 |
| W2-B rerun | 0 |
| 実 J-Quants API call | 0 |
| TDnet / XBRL 取得 | 0 |
| token / .env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl 実行 | 0 |
| plist / cron 編集 | 0 |
| workflow 変更 (= .github/workflows) | 0 |
| schema migration | 0 |
| force push | 0 |
| main 直 push (= fire repo) | 0 |
| --no-verify | 0 |
| sudo / rm -rf | 0 |
| auto-order / 楽天 / iSPEED | 0 |
| Computer Use / Playwright / Selenium | 0 |
| TODO Excel 本体更新 | 0 (= goal 指示通り vault markdown のみ) |

### §6.1 rollback artifact 状態

`data/fire.staging.db.before_restore_20260517_134908` (= 353 MB):
- `.gitignore:37` で正しく ignored ✓
- 物理削除は **後日別 wave** (= 削除 wave で安全確認後)
- 現状: untracked にも表示されない (= git 管理外完全達成)

### §6.2 3 DB md5 不変期間

5/17 14:00 (= staging restore 完了) 〜 5/17 21:xx (= 本 wave 終了):
- `data/fire.db`: b1df4673... (= 不変)
- `data/fire.develop.db`: 085799da... (= 不変)
- `data/fire.staging.db`: 71a63a19... (= 不変、restore_backup と一致)

→ 5/18 02:00 F282 fire 直前まで read-only 状態維持

---

## §7 5/18 朝以降の operator チェックリスト

```
[5/18 朝 (= operator 起動時)]

□ 1. F282 launchctl 状態確認
     $ launchctl list | grep jp.fire.weekly-snapshot
     (= LastExit=0 / runs ≥1 を期待)

□ 2. F282 log 確認
     $ tail -30 logs/cron/weekly-snapshot.log
     (= "snapshot completed" 等を期待)

□ 3. F282 post-snapshot audit 実行 (= main 反映済 runner)
     $ .venv/bin/python -m scripts.jobs.run_f282_post_snapshot_staging_audit \
         --prod-db data/fire.db --develop-db data/fire.develop.db \
         --staging-db data/fire.staging.db \
         --restore-backup ~/fire-backups/fire.staging.db.bak.20260517_post_restore_71a63a19 \
         --reference-date 2026-05-18 \
         --f282-log logs/cron/weekly-snapshot.log \
         --f282-err logs/cron/weekly-snapshot.err \
         --output-json /tmp/fire_post_snapshot_audit/audit.json \
         --output-md   /tmp/fire_post_snapshot_audit/audit.md

□ 4. audit decision を確認:
     PROCEED   → §3 happy path
     RESTORE   → §3 restore path
     INVESTIGATE → §3 stop + log 確認
     WAIT      → 02:00 過ぎなら異常、launchd 確認

□ 5. PROCEED 時:
     wrapper 方式で env 供給 (= [[JQUANTS_ENV_SUPPLY_PREFLIGHT_R1_2026-05-17]] §X 参照)
     HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY 発行
     → financials retry → derived regen → signal regen → W2-B rerun → W2-C re-review
```

---

## §8 long-term roadmap (= 1-3 ヶ月)

```
[Phase 1: fresh data 確立]  (= 5/18-5/19 想定)
   PROCEED → retry → regen → W2-B rerun → W2-C re-review

[Phase 2: 3 設計の実装]  (= 1-2 週間、並行可)
   - ADAPTER-1〜7 (= F062 / Ops Summary)
   - NIGHT-R0-1〜8
   - W2-C-1〜7
   各 wave Codex 6-8 lane review、PR、main merge

[Phase 3: NIGHT-R0 launchd 化]  (= 1-2 週間)
   - NIGHT-R0-LAUNCHD-1 (= plist 設計 + 登録)
   - 1-2 週間自動 fire 観測
   - READY 連続日数 計測

[Phase 4: NIGHT-R1 設計 + 実装]  (= R0 7 日連続 READY 後)
   - HQ_APPROVE_NIGHT_R1_DESIGN_START
   - 設計 → 実装 (= staging write 含む)
   - 14 日以上 staging 書込 安定運用

[Phase 5: NIGHT-R2 (= Paper Live / Pattern 探索)]  (= 将来)
   - F054 Paper Live tick
   - F035 Pattern Research Agent フル run
   - F033 / F034 DEATH NOTE / REHAB

[Phase 6: Live Advisory 昇格]  (= F053 全 PASS + Fujiwara 受容 + LINE 5 部屋疎通)
   - F241 9 項目検証 → Stage 3 昇格
   - 本番 LINE 送信 (= HQ_APPROVE_LINE_PROD_SEND 各 scope)
```

---

## §9 deferred (= 別 wave / 将来検討)

- `data/fire.staging.db.before_restore_20260517_134908` 物理削除
  (= 現 ignored 状態、後日削除 wave)
- F062 chunk_length 調整 (= W2-C suffix + ops 1 行で 760-800 chars 想定)
- W2-C `momentum_warning` 閾値 -0.02 の config 化
- Ops adapter の `warning_messages` i18n / blocks 細分化
- NIGHT-R0 `step 7` adapter 実装後 active 化 (= NIGHT-R0-6)
- LINE 自動承認 path (= 将来 NIGHT-R1+ で line_send_allowed=true 検討)
- TODO Excel (= Google Sheets) との同期 (= 本 wave スコープ外、運用見直し時)

---

## §10 success 判定 (= 本 wave)

| 基準 | 結果 |
|---|---|
| 2 時前 8 タスク進捗表 完成 | ✓ (§1) |
| main / develop 反映状況 明確化 | ✓ (§2、fire repo + fire-vault) |
| 5/18 02:00 後 最短ロードマップ 明確化 | ✓ (§3、4 decision 別分岐 + HQ marker scope) |
| 並行開発候補 整理 | ✓ (§4、3 wave 独立着手可) |
| Codex 活用方針 明確化 | ✓ (§5、6 lane / 8 lane / 全範囲活用) |
| 安全状態 0 件確認 | ✓ (§6、全項目 0) |
| operator チェックリスト 提供 | ✓ (§7、5/18 朝 5 step) |
| long-term roadmap 整理 | ✓ (§8、Phase 1-6) |
| DB / API / LINE / token / fire repo 操作 | 0 (= §6 全 0) |
| fire-vault commit/push | (= 本 doc 別 step で実施、本 wave goal 含む) |

→ **全成功基準 充足、本 wave 設計部 完了 ✓**
   (= 残: fire-vault commit/push、次 step で実施)
