---
id: FIRE-CODEX-R1-WAVE28-plan
phase: ガバナンス / Wave 28 / R2 v1.2 KPI 駆動 / F282 write path impl
priority: 高
status: 進行中 ☆ 8 lane (本線 4 + Codex 4)、write path 関数定義 + tests
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 27 (= 完了、R2 v1.2 正本、F282 dry-run probe 成功)
  - HQ Wave 28 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / write path impl
---

# Wave 28: F282 write path implementation (= R2 v1.2 KPI 駆動初運用)

W26 で `NotImplementedError` のままだった write path 3 関数
(_prepare_backups / _snapshot_db / _verify_snapshot) を実装。
**production fire.db への実 VACUUM INTO 実行は未承認** (= HQ 明示)、
tests は tmp_path のみで実 VACUUM INTO 動作確認、本番 DB mtime unchanged
を保証。

## Wave 28 sub-task (= 8 lane、本線 4 + Codex 4)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W28-1| L5   | 本線  | plan + file lock + 4 Codex prompt             |
| W28-2| L1a  | Codex | write path 構造詳細設計                       |
| W28-3| L1b  | Codex | output path safety / no-overwrite guard 設計  |
| W28-4| L2a  | 本線  | TestSnapshotDb tests 実装                     |
| W28-5| L2b  | 本線  | TestPrepareBackups + TestVerifySnapshot 実装  |
| W28-6| L3   | 本線  | script write path 実装                        |
| W28-7| L4   | Codex | adversarial audit                             |
| W28-8| L6   | Codex | regression plan + 本線 pytest                 |
| W28-9| 本線  | 本線  | 結果統合 + 6 KPI + commit + HQ 報告           |

R2 v1.2 lane 選択方針: task 量「大 (= 8-10)」に対応、8 lane 構成 (= Wave
26 同様の impl wave pattern)。

## file lock 表 (= R2 v1.1 + v1.2)

### 既存 modified (= 本 wave 範囲外)

| file | 状態 | 区分 |
|---|---|---|
| /Users/bluefire/fire/scripts/seed_pattern_layer1.py | M | forbidden |
| /Users/bluefire/fire/simulation/research_lane/historical_indicators.py | M | forbidden |
| /Users/bluefire/fire/.claude/ | ?? | 範囲外 |
| /Users/bluefire/fire/data/fire.staging.db.pre_restore_* | ?? | 範囲外 |

### 本 wave で書込む file

| file | 状態 | owner_lane | merge_owner | 区分 |
|---|---|---|---|---|
| /Users/bluefire/fire/scripts/jobs/run_f282_weekly_snapshot.py | MOD | 本線 (L3) | 本線 | allowed |
| /Users/bluefire/fire/tests/scripts/jobs/test_f282_weekly_snapshot.py | MOD | 本線 (L2a+L2b) | 本線 | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE28_plan.md | NEW | 本線 (L5) | 本線 | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE28_results.md | NEW | 本線 | 本線 | allowed |
| log.md | MOD | 本線 | 本線 | allowed |

### Codex 4 lane output

- /tmp/codex_wave28/output/l1a_*.{stdout,last,stderr}
- /tmp/codex_wave28/output/l1b_*.{stdout,last,stderr}
- /tmp/codex_wave28/output/l4_*.{stdout,last,stderr}
- /tmp/codex_wave28/output/l6_*.{stdout,last,stderr}

衝突 0 (= path disjoint、sandbox=read-only)。

## 実装内容 (= W28-6 L3)

### _prepare_backups(target_paths) -> list[Path]

既存 target を `~/fire-backups/` に copy。

```python
def _prepare_backups(target_paths: tuple[Path, ...]) -> list[Path]:
    """既存 target を ~/fire-backups/ に copy.

    target が存在する場合のみ backup を作成。
    backup file 名: {basename}.bak.{YYYYMMDD_HHMMSS}
    """
    backup_dir = Path.home() / "fire-backups"
    backup_dir.mkdir(parents=True, exist_ok=True)

    backups: list[Path] = []
    timestamp = datetime.now(JST).strftime("%Y%m%d_%H%M%S")
    for target_path in target_paths:
        if not target_path.exists():
            continue
        backup_name = f"{target_path.name}.bak.{timestamp}"
        backup_path = backup_dir / backup_name
        shutil.copy2(target_path, backup_path)
        backups.append(backup_path)
    return backups
```

### _snapshot_db(source_path, target_path) -> dict

SQLite VACUUM INTO で source → target。

```python
def _snapshot_db(source_path: Path, target_path: Path) -> dict[str, Any]:
    """source → target VACUUM INTO."""
    # target 既存 → 削除 (= backup は _prepare_backups で済)
    if target_path.exists():
        target_path.unlink()

    # source ro 接続
    source_uri = f"file:{source_path}?mode=ro"
    conn = sqlite3.connect(source_uri, uri=True)
    try:
        conn.execute("PRAGMA query_only=ON")
        target_path_escaped = str(target_path).replace("'", "''")
        conn.execute(f"VACUUM INTO '{target_path_escaped}'")
    finally:
        conn.close()

    return {
        "target_path": str(target_path),
        "target_size": target_path.stat().st_size,
    }
```

### _verify_snapshot(target_path, source_size) -> dict

PRAGMA integrity_check + size 検証。

```python
def _verify_snapshot(target_path: Path, source_size: int) -> dict[str, Any]:
    """target snapshot の integrity 確認."""
    conn = sqlite3.connect(str(target_path))
    try:
        result = conn.execute("PRAGMA integrity_check").fetchone()
        integrity_ok = result[0] == "ok"
    finally:
        conn.close()

    target_size = target_path.stat().st_size
    # VACUUM INTO は元 DB 圧縮するので size 完全一致は期待しない
    # (= 通常 90% 〜 110% 範囲)
    size_ok = (
        target_size > 0 and target_size <= source_size * 1.5
    )

    return {
        "integrity_ok": integrity_ok,
        "target_size": target_size,
        "source_size": source_size,
        "size_ok": size_ok,
    }
```

### main() の write path 開放

W26 までの `print + return 1` を、実装された関数の呼び出しに置き換え:

```python
if args.dry_run:
    return _connection_probe(args)  # existing dry-run

# write path
backups = _prepare_backups(target_paths)
for target_path in target_paths:
    snapshot_result = _snapshot_db(source_path, target_path)
    verify_result = _verify_snapshot(
        target_path, source_path.stat().st_size,
    )
    if not verify_result["integrity_ok"]:
        print(f"F282 verify failed: {target_path}", file=sys.stderr)
        return 2
print(...)  # summary
return 0
```

ただし **本 wave では main() の write path を直接 production fire.db で
実行しない**。tests で tmp_path 経由のみ動作確認。

### 追加 safety guard (= HQ 指示の no-overwrite guard)

- target が production fire.db (= source 自身) でないこと再確認
  (= _assert_snapshot_only_write が既に block)
- target 既存時は backup 必須 (= 削除前に copy)
- backup 失敗時は VACUUM INTO 中止

## tests 計画

新規 test class:

| class | 件数 | 検証点 |
|---|---|---|
| TestPrepareBackups | 4-5 | 既存 target backup / 不在時 skip / timestamp 形式 / backup dir 自動作成 |
| TestSnapshotDb | 5-6 | source → target VACUUM INTO / target 既存削除 / source ro 維持 / integrity 維持 / target_size > 0 |
| TestVerifySnapshot | 4-5 | integrity_ok=True / size 範囲 / 破損 DB 検出 / NotImplementedError 削除 |
| TestMainWritePath | 3-4 | main() で write path が呼ばれること (= tmp_path) |

合計 16-20 件追加見込。既存 27 + 新規 18 ≒ 45 件。

## 安全 (= HQ Wave 28 指示通り)

✓ VACUUM INTO 関数定義 + tests
✗ 実 VACUUM INTO 実行 (= production fire.db 対象)
✗ 実 DB snapshot 作成 (= production / develop / staging)
✗ plist 配置 / launchctl load / cron 登録
✗ LINE / token / secret / channel_token / F101 probe
✗ workflow / --no-verify / TODO Excel

## 成功条件 (= P2 + Wave 28 固有 + 6 KPI)

| 条件 | 期待 |
|---|---|
| 8 lane 全完了 | ✓ |
| file ownership 衝突 0 | ✓ |
| CRITICAL 0 | ✓ |
| L4 HIGH 0 | ✓ |
| safety violation 0 | ✓ (= 絶対条件) |
| 全 pytest PASS | ✓ (= 4,068 + 新規 16-20) |
| wave 実時間 < 150 分 | ✓ |
| commit 6 件以内 | ✓ (= fire develop 1 + fire-vault 2) |
| Codex 直接 commit 0 | ✓ |
| 本線過負荷 0 | ✓ |
| 全 production DB mtime unchanged | ✓ |
| 新 VACUUM INTO output file 0 (= /data/) | ✓ |
| HQ 報告 1 ブロック + 6 KPI table | ✓ |

## 関連リンク

- [[../03_design/FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 v1.2 設計]]
- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計 (W25)]]
- [[FIRE_CODEX_R1_WAVE27_results|Wave 27 results (= dry-run probe)]]
- [[FIRE_CODEX_R1_WAVE26_results|Wave 26 results (= dry-run path impl)]]
