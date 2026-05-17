# FIRE J-Quants env supply + financials retry preflight R1 (2026-05-17)

doc_id: FIRE-JQUANTS-ENV-SUPPLY-PREFLIGHT-R1-2026-05-17
status: 完了 / 値非表示維持 / DB write 1 (= backup のみ) / API call 0 / git push 0
HQ marker: HQ_APPROVE_JQUANTS_ENV_SUPPLY_PREFLIGHT_NO_VALUE
related:
- [[JQUANTS_ENV_CONNECTIVITY_R1_2026-05-17]] — env gate 実装の起点
- [[DATA_R1_STAGING_REFRESH_FINANCIALS_2026-05-17]] — JQuantsAuthError 失敗
- [[STAGING_DB_RESTORE_FROM_BACKUP_RECENT_2026-05-17]] — 現 staging md5 起点

---

## §1 目的

financials refresh retry 前の preflight:
1. JQUANTS_API_KEY を runner process に値非表示で供給可能か boolean 確認
2. 4 供給方式 (A/B/C/D) を比較、推奨案を 1 つ出す
3. 現 staging (71a63a19...) の新 rollback backup を作成
4. F282 weekly-snapshot launchctl との衝突可能性確認

---

## §2 承認範囲 (= 全て遵守)

| 項目 | 内容 |
|---|---|
| HQ marker | HQ_APPROVE_JQUANTS_ENV_SUPPLY_PREFLIGHT_NO_VALUE |
| 許可 | env boolean 確認 / wrapper test (subshell) / rollback backup 作成 / lsof / md5 / quick_check / WAL/SHM / F282 plist / launchctl list (read-only) |
| 禁止守備 | JQUANTS_API_KEY 値表示 0 / .env 内容表示 0 / 実 API call 0 / financials refresh write 0 / DB write 0 (= backup 除く) / production-develop write 0 / LINE 0 / launchctl 実行 (load/unload/start/stop) 0 / plist 編集 0 / cron 0 / workflow 0 / git add/commit/push 0 / PR/merge 0 / sudo/rm -rf 0 / auto-order 0 / 楽天/iSPEED 0 / Computer Use 0 / TODO Excel 0 |

---

## §3 git 状態

| 項目 | 値 |
|---|---|
| branch | develop |
| HEAD | 3499929 (= 前 wave env gate commit) |
| working tree | 1 untracked (= 古い rollback artifact) |
| ahead/behind origin/develop | 0/0 |

---

## §4 env 供給確認 (= 値非表示厳守、§詳細 env_supply_audit.md)

| 項目 | 結果 |
|---|---|
| shell `$JQUANTS_API_KEY` | NOT SET |
| ~/fire/.env existence | EXISTS (1,127 bytes) |
| .env 内 JQUANTS_API_KEY key 名 | PRESENT (= 値非表示) |
| Python `os.environ.get` in main | ABSENT |
| **wrapper subshell** (set -a; source .env; set +a) | **PRESENT ✓** |
| subshell isolation | OK (main shell 不変) |
| value 表示 | 0 (全 path) |
| length / prefix / hash 表示 | 0 |

→ **wrapper 方式 (= B) で値非表示供給可能と verified**

---

## §5 推奨供給方式

### §5.1 4 方式比較

| 観点 | A. ~/.zshrc export | **B. wrapper source .env** | C. python-dotenv 組込 | D. launchd EnvironmentVariables |
|---|---|---|---|---|
| 安全性 | △ | ○ | ○ | ○ |
| 値非表示 | △ | ○ | ○ | △ |
| Claude Code/Termius 手動実行 | ○ | ○ | ○ | × |
| 将来 NIGHT-R1/launchd 互換 | × | △ | ○ | ○ |
| 実装コスト | 低 | 低 | 中 | 中 |
| 本 wave verify | - | **○ verified** | - | - |

### §5.2 推奨案 = **B (wrapper) を即時運用 + C (python-dotenv) を別 wave で長期対応**

理由:
- B は本 wave で動作 verified、retry 即実行可能
- .env 内 key 既存、再編集不要 → secret 漏洩リスク最小
- C は launchd / 全 runner 共通対応として将来実装

### §5.3 B 方式 retry コマンド (= subshell isolation)

```bash
bash -c 'set -a; source .env; set +a; FIRE_ENV=staging \
  /Users/bluefire/fire/.venv/bin/python -m \
  scripts.jobs.run_jquants_daily_refresh \
  --db-path data/fire.staging.db --db-label staging \
  --datasets financials --financials-smoke-type five_codes \
  --max-days 3 --write \
  --output-json /tmp/.../result.json'
```

precheck gate (= 本 wave 前 commit 3499929 の `assert_jquants_env_for_financials_write`) が subshell 内 env を検出して通過。

---

## §6 retry precheck (= 詳細 retry_precheck.json)

| 項目 | 値 |
|---|---|
| production md5 | b1df4673... (size 371,081,216) |
| develop md5 | 085799da... (size 353,128,448) |
| staging md5 | 71a63a19... (size 4,804,788,224) |
| staging quick_check | ok |
| mf_v2 rows | 164,678 |
| mf_v2 latest disclosure_date | 2026-05-08 |
| WAL | 0 bytes |
| SHM | 32 KB (= 害なし) |
| lsof | empty (= no holder) |
| .github/workflows | not present (= OK) |

---

## §7 rollback backup

新 backup 作成:
- path: `~/fire-backups/fire.staging.db.bak.20260517_post_restore_71a63a19`
- size: 4,804,788,224 bytes
- md5: 71a63a19694385db4344246be60a2f91 (= current staging と完全一致)
- quick_check: ok
- cp 所要時間: 0.92 sec
- staging md5 cp 前後: 不変 ✓
- production / develop md5: 不変 ✓

→ retry 失敗時の rollback artifact ready。古い 353 MB rollback artifact
   は別 wave で削除推奨。

---

## §8 F282/launchctl 確認 (= read-only、★ HIGH 確認実害)

### §8.1 launchctl list (read-only) 結果

| Label | Status | LastExit |
|---|---|---|
| jp.fire.weekly-snapshot | **loaded but not running** | 0 |
| jp.fire.emergency-1445 | loaded but not running | 0 |
| jp.fire.emergency-1455 | loaded but not running | 0 |
| jp.fire.emergency-1505 | loaded but not running | 0 |
| jp.fire.emergency-1510 | loaded but not running | 0 |
| jp.fire.emergency-1515 | loaded but not running | 0 |

### §8.2 weekly-snapshot plist 詳細 (= read-only)

```
Label: jp.fire.weekly-snapshot
ProgramArguments: [.venv/bin/python, scripts/jobs/run_f282_weekly_snapshot.py,
                   --db-source, production,
                   --db-targets, staging,develop]
StartCalendarInterval: Weekday=7 (Sunday), Hour=2, Minute=0
RunAtLoad: False
EnvironmentVariables: FIRE_ENV=snapshot
```

★ **次回実行**: **2026-05-18 02:00 JST** (= 明日朝)
★ **動作**: production DB を staging + develop に上書き
   → retry 結果は 5/18 02:00 で消失する可能性

### §8.3 emergency-* について
LINE 緊急アラート用 (= 14:45/14:55/15:05/15:10/15:15)。
staging DB には影響しない (= LINE 送信のみ)。

---

## §9 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| DB write (= backup のみ許可) | 1 (= ~/fire-backups/...post_restore_71a63a19 cp) |
| production / develop DB write | 0 (md5 不変) |
| staging DB write | 0 (= cp src のみ、md5 不変) |
| 実 J-Quants API call | 0 |
| token / .env / secret 値表示 | 0 (= 全 path boolean のみ) |
| .env 内容表示 | 0 (= key 名のみ抽出) |
| printenv / echo $JQUANTS_API_KEY | 実行せず |
| cat .env | 実行せず |
| LINE 送信 | 0 |
| launchctl 実行 (load/unload/start/stop) | 0 (= list のみ) |
| plist 編集 / cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push / PR | 0 |
| schema migration | 0 |
| sudo / rm -rf / --no-verify | 0 |
| auto-order / 楽天 / iSPEED | 0 |
| Computer Use / Playwright / Selenium | 0 |
| TODO Excel 更新 | 0 |

---

## §10 Codex 8 lane 監査結果

| lane | verdict | CRITICAL | HIGH | 摘要 |
|---|---|---|---|---|
| A (env supply no-value) | APPROVE | 0 | 0 | wrapper subshell PASSED / main shell REFUSED 確認、定数 drift なし |
| B (4 方式比較) | APPROVE | 0 | 0 | B+C 推奨判定に矛盾なし、tests が dotenv 禁止のため C は別 wave 妥当 |
| C (retry precheck) | APPROVE | 0 | 0 | 全 main 数値 (md5/rows/quick_check/WAL/SHM/lsof) 一致 |
| D (rollback backup) | APPROVE | 0 | 0 | backup と staging md5 完全一致、size 4,804,788,224、quick_check ok |
| E (F282 衝突) | CONCERN | 0 | **1** | F282 weekly-snapshot loaded、5/18 02:00 staging,develop 上書き確認 |
| F (token/secret 漏洩) | CONCERN | 0 | **1** | **新指摘**: .env permission **0644** (= group/other readable)、0600 推奨 |
| G (retry readiness) | CONCERN | 0 | **1** | F282 衝突リスクで retry 結果消失可能性、option_y (5/18 02:00 後) で回避 |
| H (next-wave) | APPROVE | 0 | 0 | **RECOMMENDED_NEXT: option_y_wait_5_18** |

### §10.1-4 APPROVE 詳細
Lane A-D は本 wave の主要主張 (= wrapper verified / 4 方式比較 / precheck 値 / backup md5) を全て独立検証で一致確認。

### §10.5 Lane E HIGH (= 採用、§8 で既反映)
F282 weekly-snapshot plist 確認:
- ProgramArguments: `--db-source production --db-targets staging,develop`
- StartCalendarInterval: Sunday Hour=2 Minute=0
- 次回 fire: **2026-05-18 02:00 JST** (= 明日朝)
- retry 結果消失リスク確認 → 本 doc §12 で対策 option Y/X/Z 整理

### §10.6 Lane F HIGH (= 採用、新規対応必要)
**.env permission が 0644** (= group/other readable):
- secret 含むファイルとしては **0600 推奨**
- 同一マシン上他 user から読める設定
- Mac mini 個人マシンなら影響軽微だが、defensive practice として 0600 が業界標準

**推奨対応 (= 別 wave、本 wave 外)**:
```bash
chmod 600 .env
ls -la .env  # verify 0600
```
- 値表示なし、permission 変更のみ
- 別 HQ 承認 (HQ_APPROVE_DOTENV_PERMISSION_HARDENING 等) で実施推奨

### §10.7 Lane G HIGH (= 採用、E と同根)
F282 衝突リスクの再確認 + retry readiness 評価:
- staging/backup md5 一致、wrapper env supply 妥当
- ただし retry 実行は **weekly snapshot 通過後** または **launchctl pause wave 後** が条件
- 採用判定: option_y_wait_5_18 が最も安全

### §10.8 Lane H RECOMMENDED_NEXT = option_y_wait_5_18
- 監査ファイル主張と現物確認 整合
- F282 5/18 02:00 上書きリスクで今日中 4-wave は時間 risk 高
- launchctl 操作は別承認 + 禁止のため option_z 不採用
- derived 先行は stale financials 2026-05-08 前提になるため option_b' (skip) より option_y が retry 目的に合致

---

## §11 financials retry 可否

| 条件 | 状態 |
|---|---|
| precheck env gate 実装 | ✅ (= 3499929) |
| env 供給方式 verified | ✅ (= wrapper 方式 B) |
| 新 rollback backup 作成 | ✅ |
| DB md5 完全不変 (= 本 wave) | ✅ |
| F282 衝突対策 | ⚠ 必要 (= 5/18 02:00 後または launchctl unload wave) |

→ **conditional ready**: F282 衝突対策 + Fujiwara 設定不要 (= wrapper 経由) で即 retry 可能。
   ただし衝突回避タイミング選択要。

---

## §12 次アクション

### Option Y (★ 推奨、最も安全): 5/18 02:00 通過後に retry
- 5/18 02:00 → F282 が weekly-snapshot 実行 (= staging が production 同期される)
- production restore 完了後の staging に対して retry 実行
- 5/25 02:00 まで 1 週間の安全窓

### Option X: 5/17 中に 4 wave 連続実行
- HQ_APPROVE_DATA_R1_STAGING_REFRESH_FINANCIALS_RETRY (= wrapper 経由)
- → HQ_APPROVE_DERIVED_INDICATORS_REGEN_FRESH
- → HQ_APPROVE_SIGNAL_PERSISTENCE_FRESH
- → HQ_APPROVE_W2B_RE_RUN_FRESH_DATA
- 全 ~12 時間以内に完了必要 (= 5/18 02:00 前)

### Option Z: launchctl unload + retry + reload (= 別 wave 必要)
- 任意のタイミング可
- launchctl 操作 = 別 HQ 承認 + plist 操作禁止のため別 wave

### Option B' (= Lane H 別案): retry skip → derived regen 即進行
- 現 mf_v2 (5-08、9 days stale) で derived regen 実行
- financials retry は別 wave で後追い

---

## §13 関連 file

```
本 wave 出力:
  /tmp/fire_jquants_env_supply_preflight/
    env_supply_audit.md
    retry_precheck.json
    rollback_backup_result.json
    next_retry_plan.md
    codex_lane_{A,B,C,D,E,F,G,H}_prompt.txt + result.txt

設計 doc (= untracked):
  ~/fire-vault/03_design/JQUANTS_ENV_SUPPLY_PREFLIGHT_R1_2026-05-17.md

backup (= 本 wave 作成):
  ~/fire-backups/fire.staging.db.bak.20260517_post_restore_71a63a19 (4.8 GB)

old rollback artifact (= 別 wave 削除推奨):
  data/fire.staging.db.before_restore_20260517_134908 (353 MB)
```
