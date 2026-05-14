---
id: FIRE-CODEX-R1-WAVE60.6-fix-results
phase: 本番 v0 中核 / Wave 60.6-fix / output path guard hardening
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_6_FIX_plan.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_5_CODEX_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_POST_results.md
---

# Wave 60.6-fix Results — Output Path Guard Hardening + Codex 8-Lane Recovery Smoke v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 W60.5 Codex Lane F MEDIUM 解消 + 8 lane parallel smoke 全成功 ✓**

W60.5 で発見した `is_safe_output_path()` の terminal forbidden path edge case を
hardening し、少額手動実弾パイロット前の safety を強化。
同時に W60.5 で復旧した **stdin 経由 codex review** pattern で **Codex 8 lane
parallel smoke を初の本格運用** し、全 8 lane 成功 + 各 lane LOW 確認 = 問題なし。

W60 wave 系列の path guard / Codex parallel 両方を v0 移行前に整える wave 完了。

## §2 W60.5 Lane F MEDIUM 内容 (= 解消対象)

```
MEDIUM scripts/jobs/run_f286_after_r1_night_batch.py:390

is_safe_output_path() only checks `if seg in p_str`, and every forbidden
segment has trailing slashes. This refuses children like
/Users/bluefire/fire/data/foo.json, but does NOT refuse the exact terminal
path /Users/bluefire/fire/data, /patterns, .git, LaunchAgents, or
reports/dashboard. Use Path.resolve().parts or append a trailing slash
before matching.
```

## §3 修正実施

### §3.1 新規 constant

```python
OUTPUT_FORBIDDEN_TERMINALS: tuple[str, ...] = (
    "/data", "/.git", "/.github",
    "/LaunchAgents", "/.fire_secrets",
    "/reports/dashboard", "/patterns",
)
OUTPUT_TMPDIR_PREFIXES: tuple[str, ...] = (
    "/tmp/", "/var/folders/", "/private/tmp/", "/private/var/folders/",
)
OUTPUT_REPORTS_REQUIRED_SUBPATH: str = "/reports/after_r1"
```

### §3.2 is_safe_output_path() hardening

```python
def is_safe_output_path(path: Path) -> bool:
    if not path.is_absolute():
        return False
    try:
        p_resolved = path.resolve()  # symlink / .. 正規化
    except (OSError, RuntimeError):
        return False
    p_str = str(p_resolved)

    # forbidden child segment (= /data/foo.json)
    for seg in OUTPUT_FORBIDDEN_SEGMENTS:
        if seg in p_str:
            return False

    # W60.6-fix: forbidden terminal path (= /Users/.../fire/data slash なし)
    for term in OUTPUT_FORBIDDEN_TERMINALS:
        if p_str.endswith(term):
            return False

    # safe-prefix 限定
    if not any(p_str.startswith(prefix) for prefix in OUTPUT_SAFE_PREFIXES):
        return False

    # W60.6-fix: reports/ 配下なら after_r1/ 必須
    if "/reports/" in p_str or p_str.endswith("/reports"):
        if (
            OUTPUT_REPORTS_REQUIRED_SUBPATH + "/" not in p_str
            and not p_str.endswith(OUTPUT_REPORTS_REQUIRED_SUBPATH)
        ):
            return False

    # W60.6-fix: /Users/ 配下では reports/after_r1/ 必須 (tmpdir 例外)
    is_tmpdir = any(
        p_str.startswith(prefix) for prefix in OUTPUT_TMPDIR_PREFIXES
    )
    if not is_tmpdir:
        if (
            OUTPUT_REPORTS_REQUIRED_SUBPATH + "/" not in p_str
            and not p_str.endswith(OUTPUT_REPORTS_REQUIRED_SUBPATH)
        ):
            return False

    return True
```

### §3.3 動作確認 matrix

| path | 結果 |
|---|---|
| `/Users/bluefire/fire/data` (terminal) | **refused ✓** (W60.6-fix で塞いだ) |
| `/Users/bluefire/fire/.git` (terminal) | refused ✓ |
| `/Users/bluefire/fire/LaunchAgents` (terminal) | refused ✓ |
| `/Users/bluefire/fire/reports/dashboard` (terminal) | refused ✓ |
| `/Users/bluefire/fire/patterns` (terminal) | refused ✓ |
| `/Users/bluefire/fire/data/foo.json` (child) | refused ✓ |
| `/Users/bluefire/fire/.github/workflows/x.yml` (child) | refused ✓ |
| `/Users/bluefire/fire/reports/after_r1/x.json` | **allow ✓** |
| `/Users/bluefire/fire/reports/after_r1/sub/out.md` | allow ✓ |
| `/Users/bluefire/fire/reports/leak.md` (after_r1 配下でない) | **refused ✓** (新規制約) |
| `/Users/bluefire/Library/LaunchAgents/jp.fire.weekly-snapshot.plist` | refused ✓ |
| `/tmp/random/out/x.json` (tmpdir 任意) | allow ✓ |
| `/tmp/random/reports/dashboard/leak.md` | refused ✓ |
| `/tmp/random/reports/foo/x.md` (after_r1 配下でない) | refused ✓ |
| `/tmp/random/reports/after_r1/x.md` | allow ✓ |
| `reports/after_r1/x.md` (relative) | refused ✓ |
| `/data/../reports/after_r1/foo.json` (traversal → 安全 path) | allow ✓ |
| `/Users/.../reports/after_r1/../data/leak.json` (traversal → 危険) | refused ✓ |

### §3.4 dead import 削除

W60.6-fix Codex Lane G が `_after_r1_mvp.py:19 import os` (= 未使用) を指摘 →
削除。`os.` 使用箇所が 0 と確認済。

### §3.5 CLI_VERSION

1.1.1 (W60-post) → **1.1.2** (W60.6-fix)

## §4 tests 結果

### test_run_f286_after_r1_night_batch.py

- W60-post 完了時: 133 tests
- W60.6-fix 完了時: **160 tests** (= +27 件)

新規 W60.6-fix tests (合計 27):
- `TestW606FixOutputPathHardening` 25 件:
  - terminal forbidden 7 件 (data / .git / .github / LaunchAgents /
    reports/dashboard / patterns / .fire_secrets)
  - child forbidden 5 件 (data/foo / .github/workflows / reports/dashboard /
    patterns / LaunchAgents)
  - after_r1 allow 3 件 (root / subdir / terminal)
  - tmpdir 5 件 (任意 / var_folders / dashboard refuse / arbitrary reports refuse /
    tmpdir reports/after_r1 allow)
  - traversal 2 件 (危険 / 安全)
  - relative path 1 件
  - F282 plist path refuse 1 件
  - その他 1 件
- `TestW606FixForbiddenTerminalsConst` 2 件 (terminals_match_segments /
  reports_required_subpath)

既存 `test_users_allowed` は **削除して 2 件に分割**:
- `test_users_after_r1_allowed` (= /reports/after_r1/x.md は allow)
- `test_users_arbitrary_reports_refused` (= /reports/x.md は refuse)

### 全体 pytest

| wave | collected |
|---|---|
| W60-post | 4566 |
| W60.6-fix | **4593** (= +27) |

全 **4593 件 PASS** ✓

## §5 Codex 8 lane parallel smoke (= W60.5 stdin pattern 復帰確認)

### §5.1 起動 pattern (= W60.5 results §11 標準 template)

```bash
mkdir -p /tmp/fire_codex_w60_6
for lane in A B C D E F G H; do
  echo "Lane $lane: <focus>. Reply CRITICAL/HIGH/MEDIUM/LOW under 150 words." \
    | codex review - > /tmp/fire_codex_w60_6/lane_$lane.log 2>&1 &
done
wait
```

### §5.2 結果

**8/8 lane 起動成功 + exit 0、permission denied 再発 0 ✓**

| Lane | 観点 | 結果 | finding |
|---|---|---|---|
| A | path guard spec | exit 0 ✓ | LOW No issue — absolute path / resolve() / forbidden segments+terminals 全 OK |
| B | terminal forbidden edge | exit 0 ✓ | LOW No gap — OUTPUT_FORBIDDEN_TERMINALS 全 cover、`endswith` で 全 path refuse 確認 |
| C | reports/after_r1 allowlist | exit 0 ✓ | LOW OK — codex が exec で実 Python で動作確認 (= /reports/foo False / after_r1/sub OK / tmp OK / tmp/reports/foo False) |
| D | tmpdir test path | exit 0 ✓ | LOW OK — tmpdir prefix 4 種 / after_r1 exemption / forbidden 順序確認 |
| E | unsafe namespace rejection | exit 0 ✓ | LOW OK — codex が exec で実動作確認 (= /data /github /LaunchAgents /patterns /reports/dashboard 全 False) |
| F | tests coverage | exit 0 ✓ | LOW OK — test 行番号引用で coverage 確認 |
| G | AST safety | exit 0 ✓ | LOW — `import os` in `_after_r1_mvp.py:19` (未使用) を指摘 → **本 wave 内で削除** |
| H | docs alignment | exit 0 ✓ | LOW OK — Wave 60.5 Lane F MEDIUM 解消を W60.6 で確認 |

**CRITICAL 0 / HIGH 0 / MEDIUM 0 / LOW 1 (= dead import)** → 即削除済み。

### §5.3 W60.5 で復旧した stdin 経由 pattern の本格運用確認

- 8 lane 並列 background 起動成功 (= run_in_background=true × 8)
- 各 lane が独立 codex プロセス、相互干渉なし
- 各 lane が OpenAI API 経由で独立 reasoning
- 各 lane が `/tmp/fire_codex_w60_6/lane_<X>.log` に独立 log
- workspace-write sandbox で fire repo の read 可、write は許可 prefix のみ
- F282 / 3 DB 完全不変

**今後の実装 wave で 8 lane 並列が標準運用可能** と判明。次 wave 以降は
W60.5 results §11 の template 通り運用する。

## §6 safety 確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 (production/develop/staging) | 0 |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体参照 | 0 |
| API call | 0 (= codex は OpenAI API 経由のみ、Codex 内部 sandboxed) |
| launchctl / plist / cron 変更 | 0 |
| VACUUM | 0 |
| brokerage / 自動発注 / Computer Use | 0 |
| workflow / --no-verify / git push / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |
| 既存 v0 path 変更 | 0 (= F282/DATA-R3/F062/readiness/Ops/wrapper 不触) |

## §7 F282 / DB 不干渉確認

baseline (= W60.5-codex 完了時):
- F282 plist: 1778593597/1772
- 3 DBs 既知値、W30 snapshot 既知値
- pytest collected: 4566

完了時 (= W60.6-fix 完了):
- F282 plist: 1778593597/1772 **完全一致 ✓**
- 3 DBs 完全不変 ✓
- F282 next run 5/16 02:00 維持 ✓
- DATA-R3 / morning-advisory plist 未配置維持 ✓
- pytest collected: 4593 (= +27 regression、全 PASS)

## §8 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **8/8 = 100%** (= 全 lane 成功 + finding 取得) |
| 短縮率 | 高 (= 8 lane 並列、本線 audit を 8 分の 1 の時間で) |
| 採用率 | LOW 1 件 (`import os` 削除) → **100% 採用** |
| 差戻率 | 0 |
| Integrator 負荷 | 中 (= hardening 実装 + 27 regression tests + 8 lane smoke + docs) |
| 安全事故 | **0** ✓ |

### Codex 8/8 = 100% の意義

- W60-impl (Codex 0/8) → W60-post (Codex 0/8、permission denied) → W60.5-codex
  (Codex 4/8 復旧確認) → **W60.6-fix (Codex 8/8 復帰運用)**
- stdin 経由 pattern の本格運用が実証されたため、今後の実装 wave で 8 lane
  並列 audit が標準化

## §9 残課題 / 次 Wave

| Wave | 内容 |
|---|---|
| W60-launchd-pre | nightly cron read-only plist 設計 + Codex 8 lane audit |
| W60-integration | F062 朝 batch ↔ morning_line_material 連携設計 + Codex 8 lane |
| W60-pilot-pre | 少額手動実弾パイロット運用 doc 正式化 + Codex 8 lane |
| W61-pre | price/return data 取り込み + paper_pnl 連携設計 |

並行 v0 本線:
- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

## §10 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_6_FIX_plan]]
- [[FIRE_CODEX_R1_WAVE60_5_CODEX_results|W60.5-codex results (= Codex 復旧)]]
- [[FIRE_CODEX_R1_WAVE60_POST_results|W60-post results]]
