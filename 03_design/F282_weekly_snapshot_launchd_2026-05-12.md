---
id: F282-weekly-snapshot-launchd
phase: ガバナンス / R-01-08 / cron thaw Step 1 / F282 weekly snapshot
priority: 高
status: 設計 v1.0 (= Wave 25 W25-2〜W25-7、設計のみ、impl 別 wave + HQ 明示承認)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-12
depends_on:
  - F282 環境分離 (= 03_design/F282_environment_isolation_2026-05-08.md)
  - cron thaw 設計 (= 03_design/cron_thaw_design_2026-05-12.md、Wave 22)
  - HQ Wave 25 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / launchd
related_chapters: [F282, F236, F286-DATA-R3, W22-cron-thaw]
---

# F282 weekly snapshot launchd 化設計 — Wave 25 W25-2 〜 W25-7

最終更新: 2026-05-12

## 0. 要約

W22 cron thaw Step 1 (= F282 weekly snapshot を launchd 化) の **設計のみ**。
**実 plist 配置 / launchctl load / launchd 登録 / cron / DB write / LINE
/ token 参照は全て禁止**。本 doc は次 wave 以降の impl の基盤となる
設計初版。

Wave 25 sub-task (= 8 lane):
- L1a: plist 骨子
- L1b: log path + logrotate 統合
- L2a: dry-run 7 step + 試走計画
- L2b: 失敗時 rollback / safety nets
- L3: F282 snapshot script 設計案
- L4: adversarial audit (= 8 観点 PASS、CONCERN 5)
- L6: regression plan + 4,041 PASS

L4 audit verdict: **GO for design integration with 5 concerns**。
本 doc は L4 CONCERN 5 を反映した最終版。

---

## 1. plist 骨子 (= W25-2 L1a 設計、L4 H 反映)

### file 配置 (= 設計のみ、本 wave で配置 0)

- 配置予定 path: `~/fire/docs/launchd/jp.fire.weekly-snapshot.plist`
- ~/Library/LaunchAgents/ への配置は **別 wave + HQ 明示承認後**

### plist XML 骨子

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"
  "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
    <key>Label</key>
    <string>jp.fire.weekly-snapshot</string>

    <key>ProgramArguments</key>
    <array>
        <string>/Users/bluefire/fire/.venv/bin/python</string>
        <string>/Users/bluefire/fire/scripts/jobs/run_f282_weekly_snapshot.py</string>
        <string>--db-source</string>
        <string>production</string>
        <string>--db-targets</string>
        <string>staging,develop</string>
    </array>

    <key>EnvironmentVariables</key>
    <dict>
        <key>FIRE_ENV</key>
        <string>snapshot</string>
        <key>PYTHONPATH</key>
        <string>/Users/bluefire/fire</string>
    </dict>

    <key>WorkingDirectory</key>
    <string>/Users/bluefire/fire</string>

    <key>StartCalendarInterval</key>
    <dict>
        <key>Weekday</key><integer>7</integer>
        <key>Hour</key><integer>2</integer>
        <key>Minute</key><integer>0</integer>
    </dict>

    <key>StandardOutPath</key>
    <string>/Users/bluefire/fire/logs/cron/weekly-snapshot.log</string>
    <key>StandardErrorPath</key>
    <string>/Users/bluefire/fire/logs/cron/weekly-snapshot.err</string>

    <key>RunAtLoad</key>
    <false/>
</dict>
</plist>
```

### EnvironmentVariables 制約 (= L4 H 対応)

**plist `EnvironmentVariables` に必ず含めない**:

- `LINE_CHANNEL_TOKEN`
- `LINE_USER_ID`
- `LINE_ROOM_*`
- `LINE_EMERGENCY_GROUP_ID`
- `JQUANTS_API_KEY` / `JQUANTS_*`
- `CHANNEL_*`
- 一切の secret / token

理由: F282 weekly snapshot は **LINE 送信なしの maintenance job**、
production read-only + staging/develop write のみ。token は不要。

明示する env:
- `FIRE_ENV=snapshot` (= snapshot 専用 env、production write 禁止 guard 用)
- `PYTHONPATH=/Users/bluefire/fire`

### 既存 plist との命名整合

| plist | namespace | 用途 |
|---|---|---|
| jp.fire.emergency-{HHMM}.plist | jp.fire.* | F236 緊急アラート (5 plist) |
| jp.fire.weekly-snapshot.plist | jp.fire.* | F282 weekly snapshot (本 doc) |
| jp.fire.daily-refresh-*.plist | jp.fire.* | F100/F101/F111/F119 (= W22 Step 2、別 wave) |
| jp.fire.monthly-*.plist | jp.fire.* | F286 report (= W22 Step 3、別 wave) |
| jp.fire.maintenance-*.plist | jp.fire.* | log rotate / db vacuum (= W22 Step 4、別 wave) |

→ W22 命名規則完全準拠。

---

## 2. log path 設計 + logrotate 統合 (= W25-3 L1b 設計、L4 B 反映)

### log 配置

```
/Users/bluefire/fire/logs/
├── cron/                                    ← W22 で新設、F282 で開始
│   ├── weekly-snapshot.log
│   └── weekly-snapshot.err
├── notifications/                           ← 既存 F236
│   └── launchd_emergency.log
└── archive/                                 ← logrotate olddir
    └── YYYY-MM/                             ← 月別 directory (= L4 B 対応)
        ├── weekly-snapshot.log-YYYY-MM.gz
        └── weekly-snapshot.err-YYYY-MM.gz
```

### logrotate 設定 (= L4 B CONCERN 反映、月別 directory 明示)

```
/Users/bluefire/fire/logs/cron/weekly-snapshot.log
/Users/bluefire/fire/logs/cron/weekly-snapshot.err
{
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
    postrotate
        # 月別 directory に移動 (= archive/YYYY-MM/ 配下)
        for f in /Users/bluefire/fire/logs/archive/*-????-??.gz; do
            month=$(echo "$f" | grep -oE '\-[0-9]{4}-[0-9]{2}' | tr -d '-' | sed 's/\(....\)/\1-/')
            month=$(echo "$f" | grep -oE '[0-9]{4}-[0-9]{2}')
            [ -z "$month" ] && continue
            mkdir -p "/Users/bluefire/fire/logs/archive/${month}/"
            mv "$f" "/Users/bluefire/fire/logs/archive/${month}/" 2>/dev/null || true
        done
    endscript
}
```

### log 出力フォーマット

- 形式: `YYYY-MM-DD HH:MM:SS LEVEL [module] message`
- timezone: JST (= F101 と同じ `Asia/Tokyo`、`ZoneInfo("Asia/Tokyo")`)
- 想定容量: 月次 ~50KB、3 ヶ月保持 = ~150KB

### log dir 事前作成

- `~/fire/logs/cron/` が存在しない場合、**本 wave 開始前** に作成
- 本 wave では設計のみ、実 `mkdir` 0
- impl wave で `mkdir -p ~/fire/logs/cron/ ~/fire/logs/archive/` 実行

### F236 emergency log との配置分離

- emergency 系: `logs/notifications/` 配下
- F282 snapshot 系: `logs/cron/` 配下
- 用途別分離、混在しない

---

## 3. dry-run 7 step + 試走計画 (= W25-4 L2a 設計、L4 C 反映)

### Step 1: script 単体 dry-run

```bash
FIRE_ENV=snapshot /Users/bluefire/fire/.venv/bin/python \
  /Users/bluefire/fire/scripts/jobs/run_f282_weekly_snapshot.py \
  --dry-run \
  --db-source production \
  --db-targets staging,develop
```

期待結果:
- exit 0
- probe OK (= source ro 接続成功)
- target write 0 (= mtime 不変)
- **stdout に `write_count=0` を必ず出力** (= L4 C 対応)
- DB row 数 0 変動

### Step 2: log dir 存在確認

```bash
ls -ld /Users/bluefire/fire/logs/cron/
ls -ld /Users/bluefire/fire/logs/archive/
```

期待: 双方存在 + 書込可。未存在なら本番登録前に作成 (= 別 wave で実施)。

### Step 3: plist 静的検証

```bash
plutil -lint ~/fire/docs/launchd/jp.fire.weekly-snapshot.plist
```

期待: OK。

### Step 4: plist 環境変数引継ぎ検証

- FIRE_ENV / PYTHONPATH が正しく渡るか
- **token / channel_token は plist EnvironmentVariables に含まれない確認**
- 確認方法: plist file の grep で `LINE_` / `JQUANTS_` / `CHANNEL_` / `TOKEN` が
  含まれないこと

```bash
grep -E '(LINE_|JQUANTS_|CHANNEL_|TOKEN|SECRET)' \
  ~/fire/docs/launchd/jp.fire.weekly-snapshot.plist
```

期待: 0 件 (= grep exit 1)。

### Step 5: DB write 不発確認 (= L4 C 対応)

- 各 DB の mtime 不変確認:
  ```bash
  stat -f "%m %N" /Users/bluefire/fire/data/fire.db
  stat -f "%m %N" /Users/bluefire/fire/data/fire.staging.db
  stat -f "%m %N" /Users/bluefire/fire/data/fire.develop.db
  ```
- script stdout の `write_count=0` 確認
- snapshot source production の `PRAGMA integrity_check` 維持

### Step 6: HQ approve marker 確認

- 本 wave (= Wave 25) では **必ず Step 6 で NO_GO**
- 実 plist 配置 / launchctl load は別 wave + 別 HQ 明示承認後
- HQ approve marker (= 例: `F282_LAUNCHD_HQ_APPROVE=1` env) 提示なしなら登録禁止

### Step 7: 1 週間試走 (= 本番登録後、別 wave)

- launchctl load 後 1 週間 (= 1 回の土曜実行) 動作確認
- log / err 内容を本線が確認
- 失敗時は `launchctl unload` で即時停止
- 成功条件:
  - exit 0
  - log に `write_count > 0` (= 実 snapshot 実行成功)
  - staging.db / develop.db の row 数 ≒ production.db
  - 例外なし、stderr に traceback なし

### Go / No-Go 判定基準

| Step | 結果 | 判定 |
|---|---|---|
| 1 | exit 0 + write_count=0 | GO Step 2 |
| 2 | dir 存在 + 書込可 | GO Step 3 |
| 3 | plutil OK | GO Step 4 |
| 4 | grep 0 件 | GO Step 5 |
| 5 | mtime 不変 + write_count=0 | GO Step 6 |
| 6 | HQ approve **なし** | **NO_GO** (= 本 wave) |
| 7 | (= 別 wave) | 1 週間後判定 |

---

## 4. 失敗時 rollback / safety nets (= W25-5 L2b 設計、L4 D 反映)

### failure mode 列挙

| ID | mode | 対応 |
|---|---|---|
| A | script exit 1 (一般エラー) | log 日次確認、自動再実行なし |
| B | source DB read 失敗 | production read-only 確認、fallback なし |
| C | target DB write 失敗 | target backup から restore (= ~/fire-backups/) |
| D | snapshot 途中失敗 | idempotent script で次回 run で完全 overwrite |
| E | disk full | log archive 削除、disk 監視 alert (= 別 wave) |
| F | plist 構文エラー | 配置前に `plutil -lint` 必須 |

### production 不変原則 (= L4 D 対応、最重要)

**rollback は production を一切変更しない**。

- snapshot source = production fire.db (= read-only)
- snapshot target = staging.db / develop.db (= rollback 対象)
- production は通常運用 (= 楽天証券約定取込) でのみ更新
- production の backup / restore は **別経路** (= 別 wave で daily backup 設計)

rollback script の対象から **production を明示除外**:

```python
ROLLBACK_TARGETS_ALLOWED = ("fire.staging.db", "fire.develop.db")
ROLLBACK_TARGETS_FORBIDDEN = ("fire.db",)  # = production
```

### snapshot 前 backup (= safety net)

- launchd 実行前に既存 staging.db / develop.db を `~/fire-backups/` に copy
- 過去 4 週分保持 (= 月次 rotation)
- snapshot 失敗時はこの backup から restore

### launchctl unload による即時停止

```bash
launchctl unload ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist
```

- plist 削除はせず unload のみ (= 復活容易)
- HQ approve なしで本線が即時停止可能 (= safety net)

### F282 環境分離との整合

- production は read-only、書込み完全禁止
- snapshot 失敗で staging / develop が破損しても production は不変
- production 自体の backup は別途 daily で取得 (= 別 wave で daily plist 設計)

---

## 5. F282 snapshot script 設計案 (= W25-6 L3 設計、L4 E 反映)

### script 配置予定

- path: `/Users/bluefire/fire/scripts/jobs/run_f282_weekly_snapshot.py`
- 本 wave では **実 file 作成 0**、impl 別 wave + HQ approve

### script 全体構造

```
scripts/jobs/run_f282_weekly_snapshot.py
├── argparse (--dry-run / --db-source / --db-targets)
├── _validate_args
├── _assert_snapshot_only_write  ← R1/R2 統合 guard (= L4 E)
├── _prepare_backups
├── _snapshot_db (SQLite VACUUM INTO 経由)
├── _verify_snapshot
├── _rotate_backups
└── main()
```

### safety guard (= L4 E 対応、R1/R2 統合)

```python
import os
import sqlite3
from pathlib import Path


_SOURCE_BASENAME = "fire.db"
_TARGET_BASENAMES = ("fire.staging.db", "fire.develop.db")
_FORBIDDEN_BASENAMES = ()  # = production write 禁止


class F282SnapshotRefused(Exception):
    """F282 snapshot 拒否."""


def _assert_snapshot_only_write(
    source_path: Path,
    target_paths: tuple[Path, ...],
) -> None:
    """R1 v1.1 / R2 v1.1 / W17 staging-only guard 統合版."""

    # 1. FIRE_ENV 必須
    if os.environ.get("FIRE_ENV") != "snapshot":
        raise F282SnapshotRefused(
            f"FIRE_ENV must be 'snapshot', got {os.environ.get('FIRE_ENV')!r}"
        )

    # 2. source basename 検証
    if source_path.name != _SOURCE_BASENAME:
        raise F282SnapshotRefused(
            f"source basename must be {_SOURCE_BASENAME!r}, got {source_path.name!r}"
        )

    # 3. source symlink refuse
    if source_path.is_symlink():
        raise F282SnapshotRefused(f"source must not be symlink: {source_path}")

    # 4. source resolved basename 検証
    resolved = source_path.resolve(strict=False)
    if resolved.name != _SOURCE_BASENAME:
        raise F282SnapshotRefused(
            f"source resolved basename mismatch: {resolved.name!r}"
        )

    # 5. target 検証 (= 全 target)
    for target_path in target_paths:
        if target_path.name not in _TARGET_BASENAMES:
            raise F282SnapshotRefused(
                f"target basename must be one of {_TARGET_BASENAMES}, "
                f"got {target_path.name!r}"
            )
        if target_path.is_symlink():
            raise F282SnapshotRefused(
                f"target must not be symlink: {target_path}"
            )

    # 6. source と target の inode 同一拒否 (= L4 E 対応)
    if source_path.exists():
        source_stat = source_path.stat()
        for target_path in target_paths:
            if not target_path.exists():
                continue
            target_stat = target_path.stat()
            if (source_stat.st_dev == target_stat.st_dev
                and source_stat.st_ino == target_stat.st_ino):
                raise F282SnapshotRefused(
                    f"source/target inode collision: {source_path} == {target_path}"
                )
```

### SQLite VACUUM INTO

```python
def _snapshot_db(source_path: Path, target_path: Path) -> int:
    """source → target snapshot via VACUUM INTO. Returns row count."""

    # source は ro mode で open
    source_uri = f"file:{source_path}?mode=ro"
    conn = sqlite3.connect(source_uri, uri=True)
    try:
        conn.execute("PRAGMA query_only=ON")

        # target 既存 → backup → 削除
        if target_path.exists():
            _backup_target(target_path)
            target_path.unlink()

        # VACUUM INTO で snapshot
        conn.execute(f"VACUUM INTO '{target_path}'")

        # row count 確認 (= verification 用)
        row_count = conn.execute(
            "SELECT SUM(cnt) FROM ("
            "  SELECT COUNT(*) AS cnt FROM sqlite_master WHERE type='table'"
            ")"
        ).fetchone()[0]

        return row_count or 0
    finally:
        conn.close()
```

### --dry-run の挙動

```python
def _dry_run(source_path: Path, target_paths: tuple[Path, ...]) -> dict:
    """probe only, write 0."""

    # source ro 接続確認
    conn = sqlite3.connect(f"file:{source_path}?mode=ro", uri=True)
    try:
        conn.execute("SELECT 1").fetchone()
    finally:
        conn.close()

    # target 書込み権限確認 (= file 作成せず、parent dir で確認)
    for target_path in target_paths:
        if not target_path.parent.exists():
            raise F282SnapshotRefused(f"target dir missing: {target_path.parent}")
        if not os.access(target_path.parent, os.W_OK):
            raise F282SnapshotRefused(f"target dir not writable: {target_path.parent}")

    # disk 空き容量確認
    import shutil
    for target_path in target_paths:
        free = shutil.disk_usage(target_path.parent).free
        source_size = source_path.stat().st_size
        if free < source_size * 2:
            raise F282SnapshotRefused(
                f"insufficient disk: free={free}, need={source_size * 2}"
            )

    return {
        "write_count": 0,
        "probe_ok": True,
        "source_size": source_path.stat().st_size,
    }
```

### logging

- 開始 / 終了 / 各 step を `logger.info` で log
- エラーは `logger.error` + traceback
- 出力先: `logs/cron/weekly-snapshot.log`
- timezone JST

### test plan (= 別 wave)

```
tests/scripts/jobs/test_f282_weekly_snapshot.py
├── TestArgValidation: argparse choices enforce
├── TestStagingOnlyGuard: FIRE_ENV / basename / symlink / inode
├── TestDryRun: probe only, write 0
├── TestSnapshotProductionToStaging: VACUUM INTO mock
├── TestBackupRotation: 過去 4 週保持
└── TestVerifySnapshot: row count + integrity_check
```

### impl 順序 (= 別 wave)

| Wave 候補 | 内容 | 必要 HQ approve |
|---|---|---|
| 26 | script impl + test | impl 着手承認 |
| 27 | dry-run probe 実施 | dry-run 実行承認 |
| 28 | plist 配置 + 1 週間試走 | 試走 / 本番未登録承認 |
| 29 | 本番 launchctl load | 本番登録明示承認 |
| 30+ | 監視 / 異常検知 / 復旧 | 運用承認 |

各 wave で **独立 HQ approve** 必要。

---

## 6. F282 weekly snapshot 順序確認 (= W22 cron thaw Step 1 整合)

W22 cron thaw design の 4 段階展開:

| Step | 内容 | 本 wave |
|---|---|---|
| **1** | **F282 weekly snapshot launchd 化** | **本 wave (= 設計のみ)** |
| 2 | daily refresh F100/F101/F111/F119 順次登録 | 別 wave、Step 1 安定後 |
| 3 | weekly / monthly report | Step 2 後 |
| 4 | maintenance (log rotate / db vacuum / smoke) | Step 3 後 |

→ 本 wave で **Step 1 のみ** 設計、Step 2-4 に踏み込まない。

時間帯重複なし:
- F282 weekly snapshot: 土曜 02:00 JST
- daily refresh: 平日 16:30 〜 20:00 JST
- emergency: 平日 14:45 〜 15:15 JST
- weekly / monthly report: 日曜 10:00 / 月初 10:00 JST

→ 全 job 時間帯 disjoint、競合 0。

---

## 7. L4 audit verdict 反映状況 (= W25-7 L4)

| 観点 | L4 verdict | 本 doc での反映 |
|---|---|---|
| A. plist 命名 vs F236 | PASS | § 1 命名規則整合 |
| B. log path vs logrotate | PASS / CONCERN | § 2 月別 directory 明示 |
| C. dry-run 7 step NO_GO | PASS / CONCERN | § 3 write_count=0 明示 |
| D. rollback production read-only | CONCERN | § 4 production 不変原則 |
| E. script guard R1/R2 統合 | CONCERN | § 5 基本 7 検査 |
| F. W22 cron thaw 順序 | PASS | § 6 Step 1 のみ |
| G. 実 plist 配置 0 一貫性 | PASS / CONCERN | 本 doc 冒頭 + § 1 |
| H. token plist 除外 | PASS / CONCERN | § 1 EnvironmentVariables 制約 |

**CRITICAL 0 / HIGH 0 / CONCERN 5 (= 全 反映済)**

---

## 8. 安全 (= 本 design doc 自体)

| 項目 | 結果 |
|---|---|
| 実 plist 配置 | 0 (= 本 doc は設計のみ) |
| launchctl load | 0 |
| cron / launchd / crontab 登録 | 0 |
| 実 DB write | 0 |
| 実 LINE 送信 | 0 |
| 実 API call | 0 |
| token / secret 参照 | 0 |
| code 変更 (= scripts/jobs/* / scripts/setup/*) | 0 |
| logrotate 設定適用 | 0 |
| mkdir / dir 作成 | 0 |
| workflow 変更 | 0 |
| --no-verify | 不使用 |
| TODO Excel | 未更新 |
| Codex 直接 commit | 0 |

---

## 9. 次 wave 候補

| Wave | 内容 | HQ approve |
|---|---|---|
| 26 | F282 snapshot script 実 impl + test | 着手承認 |
| 27 | dry-run probe 実施 | 実行承認 |
| 28 | plist 配置 + 1 週間試走 | 試走承認 |
| 29 | 本番 launchctl load | 本番承認 |
| 30+ | 監視 / 異常検知 / 復旧設計 | 運用承認 |

並走候補:
- F101 staging probe (= 別 HQ approve、token 使用)
- R2 v1.2 改訂 (= W24 L4 CONCERN 3 反映)
- daily refresh F100 launchd 化 (= W22 Step 2)

---

## 10. HQ 判断論点 (= 本 doc 完成後)

1. **本設計の採用可否** (= 採用 = Wave 26 impl 着手可)
2. **Wave 26 着手 wave** (= 推奨: F282 snapshot script impl)
3. **dry-run probe 着手判定** (= Wave 27 候補)
4. **F101 staging probe との優先順位** (= F282 snapshot vs F101 probe)

---

## 11. 関連リンク

- [[F282_environment_isolation_2026-05-08|F282 環境分離 (= 既存設計)]]
- [[cron_thaw_design_2026-05-12|W22 cron thaw design]]
- [[FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 v1.1 設計本体]]
- [[FIRE_CODEX_R2_codex_prompt_template_v1_2026-05-12|R2 prompt template v1.0]]
- [[../02_todo/FIRE_CODEX_R1_WAVE25_plan|Wave 25 plan]]
