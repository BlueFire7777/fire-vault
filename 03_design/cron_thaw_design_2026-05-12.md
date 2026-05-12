---
id: cron-thaw-design
phase: ガバナンス / R-01-08 / Wave 22 設計
priority: 高
status: 設計 v1.0 (= 設計のみ、impl 別 wave + HQ 明示承認後)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-12
depends_on:
  - F236 緊急アラート (= launchd 5 段階、実装済)
  - F282 環境分離 (= 3 環境化、Mac mini 完了)
  - F286-DATA-R3 sub-D3 (= cron 復活 plan、凍結中)
  - HQ Wave 22 起票承認 (= 2026-05-12、設計のみ)
chapter: ガバナンス / R-01-08 / cron / launchd / log rotation
related_chapters: [F013, F236, F282, F286-DATA-R3, R-01-08]
---

# cron thaw design — 凍結 cron 復活設計 (= Wave 22 W22-2〜W22-6)

最終更新: 2026-05-12

## 0. 要約

F286-DATA-R3 sub-D3 で **凍結** された cron 復活 (= thaw) を **設計のみ**
で着手。実 cron / launchd / crontab 登録は **本 wave で発生させない**。
本 doc は:

1. 既存 launchd (= F236 緊急アラート 5 plist) の整合確認
2. 復活対象 cron job 列挙
3. F282 weekly snapshot との順序整理
4. log rotation 設計
5. dry-run / no-write / no-send 確認方針
6. 6 lane Codex 実起動試験計画 (= 別 section、HQ 判断材料)

を統合した **設計の正本** とする。impl は次 wave + 別 HQ 明示承認後。

---

## 1. Background (= 凍結経緯)

### 凍結対象 (= sub-D3 で見送り)

W7-2 (= 2026-05-11) で起案された cron sub-task は以下の通り凍結:

| job 名 | 目的 | 凍結理由 |
|---|---|---|
| F100 daily refresh | 価格データ日次取得 | runner 安全 guard 未完成 |
| F101 daily fetch | TDnet/IR 取得 | API 403 (= W19-1 で別 issue 化) |
| F119 daily eval | 日次評価 | LINE 送信 path 未確定 |
| F111 daily extract | watchlist signal | 既存 lever 経路で staging-only |

W17 (= sub-D2.3.x activation) で f111 / f100 / f101 / f119 の **staging-only
guard** が完備、W18 で f111 / f100 / f101 / f119 の **staging write smoke**
完了。これにより cron 復活の **runner 側準備は完了**、残課題は cron 側
(= launchd plist) の整備と運用ガイドのみ。

### 既存 launchd 状態 (= 2026-05-12 時点)

| plist | 用途 | 状態 |
|---|---|---|
| jp.fire.emergency-1445.plist | 緊急 1 段階目 (= 14:45) | 実装済、未登録 |
| jp.fire.emergency-1455.plist | 緊急 2 段階目 (= 14:55) | 実装済、未登録 |
| jp.fire.emergency-1505.plist | 緊急 3 段階目 (= 15:05) | 実装済、未登録 |
| jp.fire.emergency-1510.plist | 緊急 4 段階目 (= 15:10) | 実装済、未登録 |
| jp.fire.emergency-1515.plist | 緊急 5 段階目 (= 15:15) | 実装済、未登録 |

5 つの plist は `~/fire/docs/launchd/` に配置されているが、
**`~/Library/LaunchAgents/` への load はまだ実行されていない** (= 本番未登録)。
これは F236 完成時の方針 = 「Stage 3 移行直前に登録」と整合。

---

## 2. cron 復活対象 (= thaw 候補)

### 2.1 daily jobs (= 平日寄付前 / 引け後)

| job ID | 時刻 | 内容 | runner | 想定 cron 形式 |
|---|---|---|---|---|
| daily.refresh.f100 | 16:30 JST | 価格データ取得 | `scripts/jobs/fetch_historical_market_data.py --db-label staging` | launchd or cron |
| daily.refresh.f101 | 19:00 JST | TDnet 取得 | `scripts/jobs/fetch_announcements.py --db-label staging` | launchd or cron |
| daily.refresh.f111 | 09:30 JST | watchlist signal 抽出 | `scripts/jobs/run_research_watchlist_signal_persistence.py --db-label staging` | launchd |
| daily.refresh.f119 | 20:00 JST | 日次評価 | `scripts/jobs/run_f119_interpretation_evaluation.py --send-line=False` | launchd or cron |

### 2.2 emergency alerts (= F236 既存、再確認)

| alert ID | 時刻 | 内容 |
|---|---|---|
| emergency.1445 | 14:45 JST | 第一次通知 |
| emergency.1455 | 14:55 JST | 第二次通知 |
| emergency.1505 | 15:05 JST | 第三次通知 (= 最重要) |
| emergency.1510 | 15:10 JST | 期限到達 |
| emergency.1515 | 15:15 JST | 最終確認 |

### 2.3 weekly / monthly jobs

| job ID | 頻度 | 内容 |
|---|---|---|
| weekly.snapshot.f282 | 土曜 02:00 JST | F282 環境分離 weekly snapshot |
| weekly.report.f286 | 日曜 10:00 JST | F286 REPORT-R1 週次レポート |
| monthly.report.f286 | 月初 10:00 JST | F286 REPORT-R1 月次レポート |
| monthly.ci.audit | 月初 09:00 JST | CI drift 月次確認 (= CLAUDE.md 既定) |

### 2.4 maintenance jobs (= 内部)

| job ID | 頻度 | 内容 |
|---|---|---|
| daily.log.rotate | 03:00 JST | ログローテーション |
| daily.db.vacuum | 03:30 JST | SQLite VACUUM (= staging のみ、production は別承認) |
| weekly.test.smoke | 月曜 06:00 JST | pytest 簡易 smoke (= regression 早期検出) |

---

## 3. launchd vs cron 選択基準 (= W22-2 L1a)

### 推奨: **launchd 主軸 + cron は補助のみ**

理由:
- macOS 標準で sleep 復帰後の自動再実行に強い
- F236 emergency-* で既に launchd 実装済 → 整合
- plist 形式が宣言的、管理しやすい
- StandardOutPath / StandardErrorPath で log 出力先を plist に書ける

cron 補助の利点:
- 編集が `crontab -e` 1 操作で済む (= 軽微 job 用)
- @reboot などの簡易性

ただし cron は:
- システム sleep 中に miss → catchup なし
- 環境変数引継ぎ複雑
- log 出力先を別途 redirect 必要

→ **本 thaw では launchd を主軸、cron は採用しない** (= 単一 manager で
シンプル化)。

### plist 命名規則 (= 推奨)

```
jp.fire.daily-refresh-{job_id}.plist    # e.g., jp.fire.daily-refresh-f100.plist
jp.fire.weekly-{name}.plist             # e.g., jp.fire.weekly-snapshot.plist
jp.fire.monthly-{name}.plist            # e.g., jp.fire.monthly-report.plist
jp.fire.maintenance-{name}.plist        # e.g., jp.fire.maintenance-log-rotate.plist
```

既存 `jp.fire.emergency-{HHMM}.plist` と区別、命名一貫性を確保。

### plist テンプレ (= 設計骨子)

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>jp.fire.daily-refresh-f100</string>

    <key>ProgramArguments</key>
    <array>
        <string>/Users/bluefire/fire/.venv/bin/python</string>
        <string>/Users/bluefire/fire/scripts/jobs/fetch_historical_market_data.py</string>
        <string>--db-label</string>
        <string>staging</string>
    </array>

    <key>EnvironmentVariables</key>
    <dict>
        <key>FIRE_ENV</key>
        <string>staging</string>
        <key>PYTHONPATH</key>
        <string>/Users/bluefire/fire</string>
    </dict>

    <key>WorkingDirectory</key>
    <string>/Users/bluefire/fire</string>

    <key>StartCalendarInterval</key>
    <dict>
        <key>Hour</key><integer>16</integer>
        <key>Minute</key><integer>30</integer>
        <key>Weekday</key><integer>1</integer>  <!-- 1=Mon -->
    </dict>
    <!-- 平日のみ実行は plist 配列で 5 個並べる -->

    <key>StandardOutPath</key>
    <string>/Users/bluefire/fire/logs/cron/daily-refresh-f100.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/bluefire/fire/logs/cron/daily-refresh-f100.err</string>

    <key>RunAtLoad</key>
    <false/>
</dict>
</plist>
```

注意点:
- `FIRE_ENV=staging` を必ず設定 (= W17 guard と整合)
- `--db-label staging` を引数で明示
- log 出力先を plist 内で固定 (= log rotation で集約)
- 平日のみは `StartCalendarInterval` を 5 個 array で記述

---

## 4. F282 weekly snapshot との順序整理 (= W22-3 L1b)

### 現状の F282 snapshot

F282 環境分離 では:
- 初期同期: production fire.db → staging.db / develop.db
- 維持同期: 週次 で production → staging / develop に snapshot

### thaw 順序 (= 推奨)

```
[Step 1] F282 weekly snapshot を launchd 化
   - jp.fire.weekly-snapshot-f282.plist
   - 土曜 02:00 JST 実行
   - production → staging / develop へ snapshot
   - dry-run で動作確認 → HQ approve → 登録

[Step 2] daily refresh jobs (= F100/F101/F111/F119) を順次登録
   - F282 で staging が常に fresh な状態を維持
   - daily refresh はその差分 (= 1 日分) を staging に書き込む
   - 順序: F100 (価格) → F101 (TDnet) → F111 (signal) → F119 (eval)

[Step 3] weekly / monthly report job 登録
   - daily refresh が安定後

[Step 4] maintenance job 登録
   - log rotate / db vacuum / smoke test
```

### 競合回避

F282 snapshot (= 土曜 02:00) と daily refresh (= 平日 16:30 / 19:00 等) は
時間帯重複なし → 競合なし。

日次 staging write と weekly snapshot 上書きの順序:
- snapshot 直後 (= 土曜 02:00 〜 月曜寄付前) は staging が production と
  同期状態
- 月曜以降の daily refresh で staging に追記
- 次の土曜 snapshot で production の最新を再同期 → 月曜以降の追記は上書き

これは F282 設計通り (= staging は production の派生、weekly で同期される)。

---

## 5. log rotation 設計 (= W22-4 L1a)

### 現状のログファイル

```
~/fire/logs/
├── cron.log (= 暫定?)
├── notifications/
│   ├── launchd_emergency.log
│   └── ...
└── (cron/ 配下を新設予定)
```

### thaw 後の log 配置 (= 設計)

```
~/fire/logs/
├── cron/                                  # ★ 新設
│   ├── daily-refresh-f100.log
│   ├── daily-refresh-f100.err
│   ├── daily-refresh-f101.log
│   ├── daily-refresh-f101.err
│   ├── daily-refresh-f111.log
│   ├── daily-refresh-f111.err
│   ├── daily-refresh-f119.log
│   ├── daily-refresh-f119.err
│   ├── weekly-snapshot.log
│   ├── weekly-snapshot.err
│   ├── weekly-report.log
│   ├── monthly-report.log
│   └── maintenance-*.log
├── notifications/                          # 既存 (F236)
│   └── launchd_emergency.log
└── archive/                               # ★ 新設 (rotation 先)
    ├── 2026-05/
    │   ├── cron/...
    │   └── notifications/...
    └── 2026-04/...
```

### rotation 方針 (= 推奨)

| 観点 | 設計 |
|---|---|
| ローテーション周期 | 月次 (= 1 ヶ月境界で archive へ移動) |
| 保持期間 | 過去 3 ヶ月分 (= archive/YYYY-MM/、それ以前は削除) |
| 圧縮 | gzip (= 月次 archive 化時) |
| 実行ツール | logrotate or 自作 Python script |
| 実行時刻 | 毎月 1 日 03:00 JST |
| 実行 job | jp.fire.monthly-log-rotate.plist |

**recommended**: `logrotate` (= macOS Homebrew で提供) を採用。
コード変更不要 + 設定 file (`/usr/local/etc/logrotate.d/fire`) で完結。

logrotate 設定案 (= 別 wave で実装):

```
/Users/bluefire/fire/logs/cron/*.log {
    monthly
    rotate 3
    compress
    delaycompress
    missingok
    notifempty
    create 0644 bluefire staff
    olddir /Users/bluefire/fire/logs/archive/
    dateext
    dateformat -%Y-%m
}
```

### 容量試算

cron job 1 つあたり 1 日平均 ~10KB → 月次 ~300KB。
12 job × 3 ヶ月保持 = 約 10MB。問題なし。

---

## 6. dry-run / no-write / no-send 確認方針 (= W22-5 L1b)

### cron 登録前の必須確認 step (= 7 step)

各 plist 登録前に **全て PASS** 必須:

```
[Step 1] 手動実行で動作確認 (= cwd / Python path / FIRE_ENV / 引数)
  $ FIRE_ENV=staging /Users/bluefire/fire/.venv/bin/python \
    /Users/bluefire/fire/scripts/jobs/{runner}.py --db-label staging --dry-run
  期待: exit 0 / probe OK / fetch 0 / write 0

[Step 2] log 出力先 dir 存在確認
  $ ls -ld /Users/bluefire/fire/logs/cron/
  期待: 存在 + 書込可

[Step 3] FIRE_ENV / --db-label を plist から確実に渡る確認
  $ launchctl print user/$(id -u)/jp.fire.daily-refresh-f100
  (= 実 load 前は plist parse 検証のみ: plutil -lint plist)

[Step 4] LINE 送信 path 不発確認 (= dry-run で send_line=False or
  F286_LINE_DISABLE=1)
  期待: notifications/ に新 log 追加なし

[Step 5] DB write 不発確認 (= dry-run で write 0)
  期待: fire.staging.db の mtime 不変

[Step 6] HQ approve marker 確認 (= 別 HQ approve なき限り cron 登録禁止)
  期待: 本 wave では Step 6 で必ず NO_GO (= 別 wave 待ち)

[Step 7] dry-run で 5 連続 PASS (= 5 日分の launchctl start 模擬)
  期待: 5 回中 0 failure / 0 write / 0 send
```

### 安全制約

- 本 wave (= Wave 22) では **Step 6 で必ず NO_GO** (= 設計のみ、登録なし)
- 実 plist load は Wave 23 以降 + 別 HQ 明示承認後
- 各 job の dry-run は次 wave で minimal probe (= 1 回ずつ) で確認

### dry-run 実行 cron job 化 (= 補助案)

- 「実 cron 登録前に dry-run を 1 週間 launchctl で 1 回試走」する **試走
  step** を本番登録手順に組み込む
- 1 週間 / 5 営業日でログを観察 → 異常なし → 本番登録

---

## 7. 6 lane Codex 実起動試験計画 (= W22-6 L1a)

### 目的

R2 prompt template v1.0 (= Wave 21 確立) を実際に Codex 6 lane 並列起動で
試験運用し、以下を確認:

1. lane 別 prompt 投入 → Codex CLI 並列起動 → 個別出力受領
2. file ownership 衝突 0 を実証
3. 本線 Integrator が 6 lane の出力を merge できることを実証
4. wave 実時間が 150 分上限内に収まることを実証

### 本 wave (= Wave 22) で実起動するか

**推奨: 本 wave では実起動しない**。

理由:
- 本 Wave 22 の主 deliverable は cron thaw design (= 設計のみ、HQ 承認範囲)
- 6 lane Codex 実起動は **別 wave** で minimal probe として実施するのが安全
- 本 wave 内で実起動すると、設計 + Codex 並列起動 + audit + 報告 の同時進行で
  本線 Integrator が過負荷 (= 上限 150 分超過 risk)

### 別 wave 候補 (= Wave 23 提案、HQ 別 approve)

**Wave 23 候補**: 6 lane Codex 実起動 minimal probe

| sub | lane | task |
|---|---|---|
| W23-1 | L5 | Wave 23 plan |
| W23-2 | L1a | minimal probe 用 design |
| W23-3 | L1b | 代替設計 (= 比較案) |
| W23-4 | L2 | dummy test (= L2 lane の動作確認、修正対象 0) |
| W23-5 | L3 | dummy impl (= no-op patch、merge 動作確認のみ) |
| W23-6 | L4 | audit (= 6 lane 並列出力の検証) |

各 Codex prompt:
- allowed_files: 各 lane で disjoint (= 重複なし)
- 修正対象: minimal (= read-only もしくは comment 1 行追加程度)
- 実 API call / DB write / LINE 0
- 実行 token: 6 lane × ~10K = ~60K (= 通常 wave の 1-2 倍)

### 試験成功基準 (= Wave 23 で適用)

- 6 lane 全 完了 (= timeout なし)
- 各 lane 出力に CRITICAL 0
- file 衝突 0 (= 本線 merge で conflict なし)
- wave 実時間 < 150 分
- 安全 violation 0

### 試験失敗 → 後退

- 1 lane 失敗 → Step 1 後退 (= 5 lane) で再試験
- 2 lane 以上失敗 → R1 5 lane に後退、根本原因分析

---

## 8. 安全 (= 本設計 doc 自体)

| 項目 | 結果 |
|---|---|
| 実 cron / launchd / crontab 登録 | 0 (= 本 doc は設計のみ) |
| 実 LINE 送信 | 0 |
| 実 API call | 0 |
| 全 DB write | 0 |
| token / secret 参照 | 0 |
| code 変更 | 0 |
| workflow 変更 | 0 |
| --no-verify | 不使用 |
| TODO Excel | 未更新 |
| Codex 直接 commit | 0 (= 本 wave Codex 起動なし) |
| 楽天 / 自動発注 / Computer Use | なし |

---

## 9. 次 wave 候補

### Wave 23 (= 推奨、HQ 別 approve)

- 6 lane Codex 実起動 minimal probe (= R2 prompt template v1.0 実証)
- 設計 / dummy patch 中心、実 system 変更 0

### Wave 24

- F101 staging probe (= HQ 別 approve、token 使用、staging write 1 日分)
- 候補 endpoint `/fins/announcements` 検証

### Wave 25 以降

- F282 weekly snapshot launchd 化 (= impl + 1 週間 dry-run 試走 + 本番登録)
- F100 daily refresh launchd 化 (= 同上)
- F101 / F111 / F119 daily refresh 順次

---

## 10. HQ 判断論点 (= 本 wave 報告で提示予定)

1. **Wave 22 完了 → cron thaw 設計初版採用可否**
2. **Wave 23 候補選定**:
   - 推奨: 6 lane Codex 実起動 minimal probe (= R2 template 実証)
   - 別案: F101 staging probe (= API 確定)
   - 別案: F282 weekly snapshot launchd 化 (= 順序整理 Step 1)
3. **launchd 主軸採用可否** (= cron 非採用、launchd 単一 manager)
4. **logrotate 採用可否** (= 自作 vs 既存ツール)

---

## 11. 関連リンク

- [[F282_environment_isolation_2026-05-08|F282 環境分離]]
- [[FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 10-lane 設計]]
- [[FIRE_CODEX_R2_codex_prompt_template_v1_2026-05-12|R2 prompt template v1.0]]
- [[F286_DATA_R3_D2_real_fetch_write_2026-05-11|sub-D2/D3 real fetch design]]
- [[F286_DATA_R3_sub_D2_3_smoke_plan_2026-05-11|sub-D2.3 smoke plan]]
- [[../02_todo/F013_Mac_mini_launchd常駐|F013 launchd 常駐]]
- [[../02_todo/FIRE_CODEX_R1_WAVE22_plan|Wave 22 plan]]
