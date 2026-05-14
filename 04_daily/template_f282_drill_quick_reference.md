---
id: template-f282-drill-quick-reference
type: template
status: 1-page quick reference (= Fujiwara 5/16 03:00 当日参照用)
chapter: production-v0 / 5/16 F282 drill 1-page reference
created_by: Wave 52.5-pre
---

# 5/16 F282 Drill — 1-Page Quick Reference

> **2026-05-16 03:00 JST 当日に参照する 1 ページ。**
> 詳細仕様は [[../03_design/F282_baseline_capture_and_post_run_drill_2026-05-14|W44-pre drill doc]] 参照。

---

## §0 前提 (= 2026-05-15 までに済んでいること)

- [ ] F282 plist 配置済 (= mtime=1778593597, size=1772)
- [ ] baseline JSON 存在: `/tmp/f282_baseline_2026-05-16.json`
- [ ] schema_version = "1.0"
- [ ] safety_flags 全 false

確認:
```
ls -la /tmp/f282_baseline_2026-05-16.json
python -m json.tool /tmp/f282_baseline_2026-05-16.json | head -30
```

---

## §1 02:00 JST: F282 自動試走 (= launchd、人間操作なし)

- launchd schedule (Weekday=7, Hour=2, Minute=0) で自動起動
- snapshot 出力: `~/fire/data/snapshot/fire.staging.db` / `fire.develop.db`
- log: `~/fire/logs/cron/weekly-snapshot.log` / `.err`
- 想定実行時間: 数分〜十数分

**禁止事項** (= 当日も):
- `launchctl unload` / `load` / `kickstart` 0
- F282 plist 編集 0
- F282 手動実行 0 (= `python ... f282 ...`)
- DB write / VACUUM 0
- LINE 送信 / token 参照 0

---

## §2 03:00 JST: 6 step drill

### Step 1: launchctl print 保存 (= 人間操作)

```bash
launchctl print gui/$(id -u)/jp.fire.weekly-snapshot \
  > /tmp/f282_print_2026-05-16.txt
wc -l /tmp/f282_print_2026-05-16.txt   # 数十行期待
```

確認 (= 3 点同時、W39-temp 教訓):
1. `runs` フィールド ≥ 1
2. `last exit code` = 0
3. log + snapshot output file 存在

### Step 2: baseline JSON 確認

```bash
ls -la /tmp/f282_baseline_2026-05-16.json
python -m json.tool /tmp/f282_baseline_2026-05-16.json | head -30
```

確認:
- schema_version = "1.0"
- f282_plist.mtime = 1778593597 / size = 1772
- db keys 3 個存在
- safety_flags 全 false

### Step 3: W40.8 post-run inspection CLI 実行

```bash
.venv/bin/python -m scripts.jobs.run_f282_post_run_inspection_report \
  --launchctl-print-file /tmp/f282_print_2026-05-16.txt \
  --baseline-json /tmp/f282_baseline_2026-05-16.json \
  --markdown reports/f282/post_run_2026-05-16.md \
  --expected-run-date 2026-05-16
```

期待:
- exit 0 (= GO) または 1 (= NO-GO)
- markdown report 生成 (= `reports/f282/post_run_2026-05-16.md`)
- markdown 内 `- verdict: **GO**` または `**NO-GO**`

### Step 4: W43-pre / W51 readiness CLI v1.2 実行

```bash
.venv/bin/python -m scripts.jobs.run_production_v0_readiness_check \
  --phase post-f282 --json
```

期待:
- cli_version "1.2" (= W51 fix 後)
- F282_POST_RUN_REPORT_EXISTS = PASS
- D_DAY_WEEKDAY_HQ_DECISION_PENDING = PASS (= W51 Tuesday lock 反映)

### Step 4.5: W44.6-pre / W44.6-post Ops Summary CLI smoke

```bash
.venv/bin/python -m scripts.jobs.run_production_v0_ops_summary \
  --phase post-f282 --date 2026-05-16 \
  --markdown reports/ops/v0_summary_2026-05-16_post_f282.md
```

期待:
- overall_verdict GO (= F282 report PASS) または HOLD (= checklist 未記入で WARN)
- cli_version "1.0.1" (= W44.6-post fix 後)
- safety_flags 8 keys 全 false (= production_send_executed=false 含む)
- Markdown 生成

### Step 5: 成果物確認

```bash
ls -la reports/f282/post_run_2026-05-16.md
ls -la reports/ops/v0_summary_2026-05-16_post_f282.md
cat reports/f282/post_run_2026-05-16.md | head -30
```

確認:
- 両 file 生成済
- size > 0 (= 0 byte は W44.6-post fix で WARN/FAIL)
- 内容: F282 verdict / Ops Summary overall_verdict

### Step 6: GO/NO-GO 報告記入

template `~/fire-vault/04_daily/template_f282_post_run.md` を copy:

```bash
cp ~/fire-vault/04_daily/template_f282_post_run.md \
   ~/fire-vault/04_daily/2026-05-16_f282_post_run.md
```

12 section 記入:
- §3 3 点同時確認 (runs / log content / output file size)
- §4 plist mtime unchanged (= 1778593597 / 1772)
- §5 DB unchanged except intended outputs
- §6 snapshot integrity (PRAGMA integrity_check via W40.8 CLI)
- §7 errors / warnings
- §8 W40.8 CLI verdict
- §9 W43-pre / W51 readiness CLI v1.2 verdict
- §10 final recommendation (= GO / NO-GO / HOLD)
- §11 next action
- §12 添付 path 一覧

vault commit:
```bash
cd ~/fire-vault && git add 04_daily/2026-05-16_f282_post_run.md && \
git commit -m "F282 post-run report 2026-05-16"
```

---

## §3 GO/NO-GO 判定

### GO 条件 (= 全件 PASS)
- F282 plist mtime/size = baseline と一致
- runs ≥ 1 + last exit 0 + log "F282 snapshot OK" 存在
- snapshot output file 2 個 (= staging / develop) size > 0
- PRAGMA integrity_check = ok
- W40.8 CLI verdict = GO
- W51 readiness CLI verdict = GO
- W44.6-post Ops Summary verdict = GO or HOLD

### NO-GO 条件 (= いずれか触発)
- F282 plist mtime/size 変化
- runs increment 0 (= 起動失敗)
- last exit non-zero
- log 不在 / mtime stale
- snapshot output 不在 / 0 byte
- W40.8 verdict = NO-GO
- W51 strict FAIL

### HOLD 条件
- W44.6-post Ops Summary が WARN only (= checklist 未記入)
- 24h 観察、5/19 判定 wave で再評価

---

## §4 NO-GO 時の対応

1. 7-incidents 起票:
   ```
   echo "## [2026-05-16] F282 trial NO-GO" \
     >> ~/fire-vault/07_incidents/f282_trial_no_go_2026-05-16.md
   ```

2. 詳細を `04_daily/2026-05-16_f282_post_run.md` §10 NO-GO に記入
3. HQ 即時相談 (= LINE 緊急部屋 or 別 channel)
4. **rollback は実行しない** (= 当日中の判断材料収集まで)
5. 5/19 判定 wave で再評価

---

## §5 timeline 確定 (= v0 全体)

| 日 | アクション |
|---|---|
| **2026-05-16 02:00 火** | F282 自動試走 (launchd) |
| **2026-05-16 03:00 火** | **本 drill 6-step + Step 4.5** |
| 2026-05-19 月 | F282 GO/NO-GO 判定 → Wave 41 着手前提 |
| Wave 41 (= 5/19-5/26) | DATA-R3 daily-refresh launchd + 1 週間 no-write 試走 |
| Wave 45 (= 5/26-6/2) | F062 morning advisory launchd + 1 週間 no-send 試走 |
| Wave 52 (= 6/2-6/8) | HQ_APPROVE_LINE_TOKEN_PRODUCTION + wrapper script 配置 |
| **2026-06-08 月** | **final strict check** (= readiness CLI v1.2 `--phase pre-v0-launch --strict`) |
| **2026-06-09 火** | **D-Day Production v0 launch** (= 08:45 LINE 初回送信) |
| 2026-06-09〜15 | dual-run monitoring 5 営業日 + 毎朝 Ops Summary daily |

---

## §6 HQ marker 7 段固定 (= 表示用)

| # | marker | 種別 | wave |
|---|---|---|---|
| 1 | F282 GO 判定 | 自然遷移 | 5/19 (= 本 drill 後) |
| 2 | `HQ_APPROVE_LAUNCHD_DAILY` | env | Wave 41 |
| 3 | DATA-R3 no-write 試走 GO | 自然遷移 | W41 完了 |
| 4 | `HQ_APPROVE_LAUNCHD_MORNING_ADVISORY` | env | Wave 45 |
| 5 | F062 no-send trial GO | 自然遷移 | W45 完了 |
| 6 | `HQ_APPROVE_LINE_TOKEN_PRODUCTION` | env | Wave 52 |
| 7 | `HQ_APPROVE_PRODUCTION_V0_LAUNCH` | env | Wave 53 (= 6/8 月夕方) |
| 補 | `HQ_APPROVE_PRODUCTION_V0_RECOVERY` | env | rollback 後 |

**禁止 alias** (= W52-pre/post で確定):
- `HQ_APPROVE_SEND_MARKER`
- `HQ_SEND_MARKER`
- `SEND_MARKER`

---

## §7 当日禁止事項 (= 5/16 03:00 drill 中)

- launchctl unload / load / kickstart
- F282 plist 編集
- F282 手動実行
- DB write / VACUUM
- LINE 送信
- token / channel_token / secret 値参照
- cron / crontab 変更
- workflow / --no-verify / sudo / rm -rf / git push (= drill 結果 commit 以外)
- 楽天証券操作 / 自動発注 / Computer Use

---

## §8 当日 reference doc (= 詳細を見る時)

- [[../03_design/F282_baseline_capture_and_post_run_drill_2026-05-14|W44-pre drill doc (= 詳細)]]
- [[../03_design/F282_post_run_inspection_report_cli_2026-05-14|W40.8 post-run CLI 仕様]]
- [[../03_design/Production_v0_readiness_check_cli_v1_2_2026-05-14|W51 readiness CLI v1.2]]
- [[../03_design/Production_v0_ops_summary_cli_v1_0_2026-05-14|W44.6 Ops Summary CLI]]
- [[../03_design/Production_v0_cutover_rollback_token_runbook_2026-05-14|cutover runbook (= v0 全体)]]
- [[template_f282_post_run|template (= 12 section GO/NO-GO 記入用)]]
