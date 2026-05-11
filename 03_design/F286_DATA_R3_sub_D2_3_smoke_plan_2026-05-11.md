---
id: F286-DATA-R3-sub-D2-3-smoke-plan
phase: ガバナンス / Wave 7 W7-2 / sub-D2.3 fetch/write smoke 計画
priority: 最優先
status: 起票 ☆ 実 fetch / 実 write / staging write は未承認
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-DATA-R3-D2 (= W5-1+2 subprocess routing 完了)
  - F286-DATA-R3-D2.2 (= W6-5 placeholder 解消 + exit code 集約 完了)
  - HQ Wave 7 W7-2 起票 approve (= 2026-05-11)
chapter: ガバナンス / R-01-08 / Codex 運用拡張 / F286 DATA シリーズ
---

# F286-DATA-R3 sub-D2.3 Fetch/Write Smoke Plan (= Wave 7 W7-2)

最終更新: 2026-05-11

## ★ 状態: 起票 (= smoke plan のみ、実 fetch / 実 write は HQ 別 approve)

W6-5 で placeholder 2 件 (f101 / f119) を active 化、`_probe_subrunner_
supports_dry_run()` + `aggregate_dry_run_exit_code()` を導入。sub-D2.3
では **実 sub-process を起動した dry-run smoke + 各 runner の --dry-run
support 確認 + exit code 集約の動作確認** を実施。

**staging write は本 sub-task では実装しない**、必要時は別 HQ approve で
個別 sub-task (sub-D2.3.1 / sub-D2.3.2 / ...) に分割。

## HQ 制約 (= 2026-05-11 Wave 6 approve 受領時)

1. 実 fetch / 実 write / staging write は **まだ未承認**
2. まずは smoke plan / dry-run 強化 / 対象 runner 確認 / exit code 集約確認
   を進める
3. staging write が必要な場合は **別 HQ 承認** を取る
4. cron / launchd / crontab 本番登録は **凍結継続** (= sub-D3)

## 1. 対象 runner 棚卸し

W6-5 完了時点で `list_daily_refresh_jobs()` が返す 4 job:

| job_id | module | status | --dry-run support |
|--------|--------|--------|--------------------|
| f100_market_data | `scripts.jobs.fetch_historical_market_data` | active | 要確認 |
| f101_announcements | `scripts.jobs.fetch_announcements` | active (W6-5) | 要確認 |
| f111_daytrade_selection | `scripts.jobs.run_daytrade_selection` | active | 要確認 |
| f119_evaluation | `scripts.jobs.run_f119_interpretation_evaluation` | active (W6-5) | 要確認 |

### 確認手順 (= grep + --help probe)

```bash
# 各 runner の --dry-run option 有無を grep で確認
for module in fetch_historical_market_data fetch_announcements \
              run_daytrade_selection run_f119_interpretation_evaluation; do
    echo "=== $module ==="
    grep -n "add_argument.*--dry-run" scripts/jobs/${module}.py 2>/dev/null \
        || echo "  (no --dry-run option found)"
done
```

### 各 runner の現状 (= 2026-05-11 W7-2 plan 起票時点の想定)

W6-5 設計時の想定 (= 実際は smoke 実行時に再確認):

- **f100_market_data**: J-Quants V2 過去データ取得、`--dry-run` 不所持の
  可能性高 (= HistoricalDataFetcher は実 fetch 前提) → `_probe_subrunner_
  supports_dry_run()` で false → status='no_dry_run_option_skipped' で
  safe-skip
- **f101_announcements**: TDnet 適時開示取得、`--dry-run` 不所持 → 同上
  safe-skip
- **f111_daytrade_selection**: Daytrade 候補抽出、`--dry-run` 不所持の
  可能性 → safe-skip
- **f119_evaluation**: Evaluation Agent、`--dry-run` 不所持の可能性 →
  safe-skip

→ 現状では **4 runner 全て safe-skip** になる可能性が高い。

## 2. dry-run 強化 plan (= sub-D2.3 の主要成果)

### 2.1 `--dry-run` option を 4 runner 全てに追加

各 sub-runner に共通の `--dry-run` option を追加:

```python
parser.add_argument(
    "--dry-run",
    action="store_true",
    help="Connection probe only (no fetch / no write). "
         "Validates config, DB path, env vars, and API auth without "
         "executing actual fetch or write. Returns exit 0 on probe success, "
         "1 on config error, 2 on connectivity failure.",
)
```

### 2.2 各 runner の --dry-run 挙動 (= 統一仕様)

| runner | --dry-run で確認する項目 | exit 0 条件 |
|--------|--------------------------|-------------|
| f100_market_data | env / DB path / J-Quants V2 auth ping | auth ping 成功 |
| f101_announcements | env / DB path / TDnet/J-Quants V2 ping | ping 成功 |
| f111_daytrade_selection | env / DB path / Feature Store 接続 | DB read 成功 |
| f119_evaluation | env / DB path / 必要 table 存在確認 | DB read 成功 |

**実 fetch / 実 write は --dry-run で発生しない**。各 runner の本体は
`if args.dry_run: return DRY_RUN_OK_EXIT; else: ...` の分岐。

### 2.3 各 runner の本体修正方針 (= 別 sub-task 提案)

dry-run 追加は 4 runner の本体に code change を必要とするため、本 W7-2
plan では **plan のみ**、実装は以下 4 sub-task で分割:

- **W7-2-impl-a**: f100_market_data --dry-run 追加 (= L3+L2)
- **W7-2-impl-b**: f101_announcements --dry-run 追加 (= L3+L2)
- **W7-2-impl-c**: f111_daytrade_selection --dry-run 追加 (= L3+L2)
- **W7-2-impl-d**: f119_evaluation --dry-run 追加 (= L3+L2)

各 sub-task は **HQ 別 approve** 要。staging write は **絶対に発生しない**
ため、本 sub-task 群は Wave 8+ の早期で並列実行可。

## 3. exit code 集約の動作確認

W6-5 で導入した `aggregate_dry_run_exit_code()` は unit test で 0/1/2/3 を
検証済 (= 36 PASS)。sub-D2.3 では **実 subprocess を起動して動作確認**:

### 3.1 mocked subprocess での再現 (= 既存 unit test)

```python
# W6-5 既存 TestAggregateDryRunExitCode (= unit)
- 全 ok → 0
- 1 件 failed → 1
- 1 件 timeout → 2
- 想定外 status → 3
```

### 3.2 実 subprocess での動作確認 (= sub-D2.3 新規)

```bash
# 全 runner --dry-run で起動 (= dry-run 強化後)
.venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh \
    --execute-dry-run-subprocesses \
    --db-label staging \
    --db-path ~/fire/data/fire.staging.db \
    --base-date 2026-05-08 \
    --output-json /tmp/sub_d2_3_dry_run.json

# 期待 exit:
#   全 runner --dry-run 成功 → exit 0
#   1 件 auth fail → exit 1
#   1 件 timeout (= TDnet が遅い) → exit 2
```

### 3.3 staging write を含まないことの確認

```bash
# pre-smoke 状態
ls -la ~/fire/data/fire.staging.db | awk '{print $6, $7, $8}'
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT name FROM sqlite_master WHERE type='table' ORDER BY name;" \
    > /tmp/sub_d2_3_pre_tables.txt
sha256sum /tmp/sub_d2_3_pre_tables.txt

# subprocess --dry-run 全 4 件実行

# post-smoke 状態
ls -la ~/fire/data/fire.staging.db | awk '{print $6, $7, $8}'
# → mtime は **変化なし** (= --dry-run なので DB write 0)
sqlite3 ~/fire/data/fire.staging.db \
    "SELECT name FROM sqlite_master WHERE type='table' ORDER BY name;" \
    > /tmp/sub_d2_3_post_tables.txt
diff /tmp/sub_d2_3_pre_tables.txt /tmp/sub_d2_3_post_tables.txt
# → diff 0 行 (= schema 変化なし)
```

## 4. staging write が必要になる条件 (= 棚卸し)

W7-2 では staging write を実装しないが、将来 sub-D2.3.x として必要に
なるケースを以下に棚卸し:

| runner | staging write が必要なケース | 別 HQ approve のスコープ |
|--------|------------------------------|-----------------------|
| f100_market_data | 過去 6 ヶ月 historical を staging に蓄積 | sub-D2.3.f100 (= 単独 approve) |
| f101_announcements | TDnet 適時開示を staging announcements table に書込 | sub-D2.3.f101 |
| f111_daytrade_selection | 候補抽出結果を staging に書込 | sub-D2.3.f111 |
| f119_evaluation | Evaluation 結果を staging に書込 | sub-D2.3.f119 |

各 sub-D2.3.* は別 HQ approve で承認、smoke plan を個別に提示。

**現在の W7-2 では plan のみ**、staging write は実施しない。

## 5. cron 連携 (= sub-D3、凍結継続)

HQ 制約: cron / launchd / crontab 本番登録は引き続き **凍結**。

sub-D3 着手は以下の前提が揃った後:

1. W7-2-impl-a〜d (= 4 runner --dry-run 追加) 完了
2. sub-D2.3.* (= 各 runner の staging write smoke) 個別完了
3. sub-D3-plan (= cron 登録時刻 / launchd plist 内容 / 失敗時通知ルート)
   提示 → HQ approve
4. crontab / launchd 実 登録 → 24h 観察 → 正常稼働確認

最短でも Wave 9+。

## 6. 受容判定 (= sub-D2.3 完了条件)

W7-2 W7-2-impl-a〜d を全完了した時点で:

- [ ] 4 runner に `--dry-run` option 追加 (= 統一仕様、connection probe only)
- [ ] `_probe_subrunner_supports_dry_run()` で 4 runner 全て True 返却
- [ ] `aggregate_dry_run_exit_code()` が実 subprocess で 0/1/2/3 を正しく集約
- [ ] subprocess --dry-run 全 4 件実行で staging DB mtime / schema **変化なし**
- [ ] production / develop DB は touch されない (= 三段+六段ガードで refuse)
- [ ] token / channel_token / secret は env に注入されない
- [ ] tests 全 PASS (= 3,637 baseline + 各 impl で 5-15 PASS 追加見込)

## 7. 実行前 HQ approve template

W7-2-impl-a〜d 個別着手前、および staging write smoke 着手前に HQ 承認:

```
=== HQ 承認依頼 / Wave 8+ W7-2-impl-<X> dry-run 追加実装 ===

date:                (実行予定日)
target_runner:       <module name>
change_scope:        --dry-run option 追加、本体は dry-run 分岐のみ
production_impact:   なし (= 既存挙動は --dry-run 不渡しで完全保持)
staging_write:       0 (= --dry-run smoke のみ、本体 write 不発生)
LINE 送信:           なし
secrets touched:     なし

承認希望: <X> runner への --dry-run option 追加実装
```

## 8. リスク評価

| risk | 影響 | 対策 |
|---|---|---|
| f100 J-Quants auth ping が課金対象 | API call 発生 | --dry-run で 1 回のみ、課金許容 |
| f101 TDnet ping が rate limit 抵触 | 後続失敗 | timeout + retry policy 設定 |
| dry-run probe が誤って fetch を実行 | 想定外 write | runner 本体で if args.dry_run: return EARLY を厳守、test で probe-only 担保 |
| F282 weekly snapshot (Monday 07:00) が smoke と衝突 | staging 上書き | smoke は他時間帯に実施 |

## 9. 関連リンク

- [[../02_todo/FIRE_CODEX_R1_WAVE7_plan|Wave 7 plan]]
- [[F286_DATA_R3_D2_2_design_2026-05-11|F286-DATA-R3 sub-D2.2 design]] (= 仮 link、別途確認)
- W6-5 実装: scripts/jobs/run_f286_data_r3_daily_refresh.py
- HQ 制約 source: FIRE-CODEX-R1 v1.1 Wave 6 approve (= 2026-05-11)
