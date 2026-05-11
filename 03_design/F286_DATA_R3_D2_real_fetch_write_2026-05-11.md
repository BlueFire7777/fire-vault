# F286-DATA-R3-D2 Real Fetch / Write Integration Design
version: v1.0 (draft)
date: 2026-05-11
status: design proposal (本線 Architect 承認待ち)

## 1. 目的

F286-DATA-R3 Daily Refresh Cron Runner skeleton に、実 fetch / 実 write を統合するための設計を定義する。

本 D2-design の目的は設計 doc 作成のみであり、実装本体、cron 登録、launchd plist 作成、DB write、LINE 送信は行わない。

対象は「既存 sub-runner を daily refresh runner から順次呼び出す統合設計」である。既存 sub-runner の責務と dry-run / --write 経路を尊重し、F286 側は orchestration、plan 集約、ログ集約、exit code 集約に限定する。

D2 完成後は手動実行で 1-2 週間観察し、問題がなければ sub-D3 で launchd 登録へ進む。

## 2. HQ 暫定方針 (= 案 A + 案 B 反映)

### 案 A: F282 月曜順序矛盾への対応

F282 weekly snapshot との順序矛盾を避けるため、月曜の F286 daily refresh は 07:30 JST に後ろ倒しする。

- 月曜: 07:30 JST
- 火曜-金曜: 06:00 JST
- 土日: 原則起動なし

F282 weekly snapshot 自体の見直しは、本 D2 / sub-D3 の範囲外とし、将来 FIRE-OPS-R0 候補として扱う。

### 案 B: launchd 日付付きログ問題への対応

launchd の StandardOutPath / StandardErrorPath には日付変数展開を期待しない。

sub-D3 では固定ログ path を使い、runner 内部または OS 側 logrotate で rotation する。

- 固定ログ path: `logs/cron/f286_data_r3_daily_refresh.log`
- rotation: `logrotate` または Python `logging.handlers.TimedRotatingFileHandler`
- 日付付きログが必要な場合: launchd ではなく runner 内部で生成する

## 3. 既存 sub-runner 一覧 + 引数 mapping

F286 daily refresh は、既存 runner を subprocess で順次呼び出す。並列実行はしない。依存関係順に実行し、途中失敗したら後続を停止する。

### Job 定義の基本形

`list_daily_refresh_jobs()` は以下のような job 定義を返す設計とする。

```python
DailyRefreshJob(
    job_id="f100_historical",
    command=[
        ".venv/bin/python",
        "scripts/jobs/fetch_historical_market_data.py",
        "--db-path",
        "{db_path}",
        "--db-label",
        "{db_label}",
        "--base-date",
        "{base_date}",
        "{mode_flag}",
    ],
    timeout_seconds=3600,
    required=True,
)
```

`{mode_flag}` は dry-run 時に `--dry-run`、write 時に `--write` を入れる想定とする。ただし既存 sub-runner ごとに実引数名が異なる可能性があるため、sub-D2 Impl で確認して mapping を確定する。

### sub-runner 候補

| 順序 | job_id | 既存 runner | 目的 | dry-run mapping | write mapping | env 注入 |
|---:|---|---|---|---|---|---|
| 1 | `f100_historical` | `scripts/jobs/fetch_historical_market_data.py` | 日次価格データ取得 | `--dry-run` | `--write` | `FIRE_ENV=staging`, `DB_PATH={db_path}` |
| 2 | `f100_intraday` | `scripts/jobs/fetch_intraday_data.py` | intraday データ取得。必要時のみ | `--dry-run` | `--write` | `FIRE_ENV=staging`, `DB_PATH={db_path}` |
| 3 | `f101_announcements` | 既存 announcements 取得 runner | TDnet / announcements 取得 | 要確認 | 要確認 | `FIRE_ENV=staging`, `DB_PATH={db_path}` |
| 4 | `f111_r4_signal_persistence` | `scripts/jobs/run_research_watchlist_signal_persistence.py` | advisory signal persistence | `--dry-run` または `--check` 要確認 | `--write` | `FIRE_ENV=staging`, `DB_PATH={db_path}` |
| 5 | `f119_evaluation` | `scripts/jobs/run_f119_evaluation.py` 等 | Evaluation Agent 集計 / 提案書生成 | `--dry-run` または `--check` 要確認 | `--write` | `FIRE_ENV=staging`, `DB_PATH={db_path}` |

### 引数 mapping 方針

F286 側の public CLI は以下を標準化する。

```bash
.venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh \
  --db-path data/fire.staging.db \
  --db-label staging \
  --base-date 2026-05-11 \
  --write
```

F286 は受け取った `--db-path`, `--db-label`, `--base-date`, `--dry-run`, `--write` を各 sub-runner の既存引数へ変換する。

原則:

- sub-runner に `--db-path` があれば CLI 引数として渡す
- sub-runner に `--db-label` があれば CLI 引数として渡す
- sub-runner が `DB_PATH` を読む設計なら env にも注入する
- sub-runner が `FIRE_ENV` を読むかは未確定のため、env 注入はしてよいが、それだけに依存しない
- `--base-date` が存在しない runner では、その runner の既存 date 引数へ mapping する
- date range が必要な runner では、daily refresh の対象日から from/to を組み立てる

## 4. subprocess vs import 比較 + 推奨

### subprocess 呼び出し

利点:

- 既存 runner pattern を維持できる
- shell から同じコマンドで単体再実行できる
- stdout / stderr / exit code を job 単位で分離できる
- sub-runner の import side effect や依存関係を F286 側へ持ち込まない
- timeout、ログ保存、失敗時停止を orchestration として扱いやすい

欠点:

- Python 関数呼び出しより overhead がある
- dry-run preview JSON の schema が runner ごとに揃っていない可能性がある
- 引数 mapping の確認が必要

### Python import 呼び出し

利点:

- 型付き API として統合できる場合は結果集約しやすい
- process 起動 overhead がない
- テストで関数単位 mock がしやすい

欠点:

- 既存 runner の CLI 実行経路と乖離する
- runner 内部の global config / logging / env 前提を F286 に持ち込む可能性がある
- 失敗分離、timeout、stdout / stderr 分離が複雑になる
- import した時点で副作用が出る runner があると危険

### 推奨

推奨は subprocess 呼び出し。

理由は、F286-DATA-R3 は cron runner / daily refresh orchestration であり、各 data fetch / persistence の業務ロジックを再実装しないためである。既存 runner の CLI を正本として扱えば、手動再実行、ログ調査、障害切り分けが単純になる。

## 5. dry-run / --write 経路の統合

### dry-run plan

`plan_daily_refresh()` は skeleton の静的 plan から、実 sub-runner の dry-run 結果を集約する plan へ強化する。

dry-run 時の流れ:

1. `list_daily_refresh_jobs()` で対象 job を取得
2. 各 job の dry-run command を生成
3. subprocess で順次実行
4. stdout / stderr / exit code / elapsed time を収集
5. stdout が JSON の場合は parse し、想定 row 数や差分 summary を抽出
6. stdout が非 JSON の場合は raw summary として保存し、`preview_parse_status="raw"` とする
7. 全 job の preview をまとめて F286 の output JSON として出力

preview JSON 例:

```json
{
  "task_id": "F286-DATA-R3",
  "mode": "dry_run",
  "db_label": "staging",
  "db_path": "data/fire.staging.db",
  "base_date": "2026-05-11",
  "jobs": [
    {
      "job_id": "f100_historical",
      "command": [".venv/bin/python", "scripts/jobs/fetch_historical_market_data.py", "..."],
      "exit_code": 0,
      "elapsed_seconds": 12.4,
      "preview_parse_status": "json",
      "estimated_rows": {
        "market_prices": 3200
      },
      "would_write": true,
      "log_stdout_path": "logs/f286/f100_historical.stdout.log",
      "log_stderr_path": "logs/f286/f100_historical.stderr.log"
    }
  ],
  "summary": {
    "total_jobs": 5,
    "success_jobs": 5,
    "failed_jobs": 0,
    "estimated_total_rows": 3200
  }
}
```

### --write mode

write 時の流れ:

1. `list_daily_refresh_jobs()` で job 定義取得
2. 各 sub-runner の write command を生成
3. subprocess で依存関係順に順次実行
4. exit code、stdout、stderr、elapsed time、ログ path を記録
5. いずれか失敗したら後続停止
6. 失敗時は全体 exit 1
7. 全成功時は全体 exit 0
8. output JSON に各 sub-runner 結果を集約

write 結果 JSON 例:

```json
{
  "task_id": "F286-DATA-R3",
  "mode": "write",
  "db_label": "staging",
  "db_path": "data/fire.staging.db",
  "base_date": "2026-05-11",
  "overall_status": "success",
  "exit_code": 0,
  "jobs": [
    {
      "job_id": "f100_historical",
      "status": "success",
      "exit_code": 0,
      "elapsed_seconds": 58.2,
      "log_stdout_path": "logs/f286/f100_historical.stdout.log",
      "log_stderr_path": "logs/f286/f100_historical.stderr.log",
      "parsed_result": {
        "written_rows": {
          "market_prices": 3200
        }
      }
    }
  ]
}
```

### three-stage guard

F286 側は以下の 3 段階 guard を明示する。

1. `dry-run`: sub-runner の dry-run / check 経路のみを実行し、DB write しない
2. `write`: 明示 `--write` がある場合のみ sub-runner の write 経路を実行する
3. `cron`: sub-D3 まで凍結。D2 / sub-D2 では launchd 登録しない

`--write` と `--dry-run` が同時指定された場合は CLI validation error とする。

## 6. エラーハンドリング戦略

### timeout

各 job に timeout を持たせる。

- default: 3600 秒
- F100 historical など長時間化しうる job は個別 override 可能
- timeout 時は対象 job を failed とし、後続 job を停止する
- 全体 exit code は 1

### stdout / stderr capture

subprocess 実行時は stdout / stderr を capture し、job 単位で logs 配下へ保存する。

推奨 path:

```text
logs/f286/{base_date}/{run_id}/{job_id}.stdout.log
logs/f286/{base_date}/{run_id}/{job_id}.stderr.log
logs/f286/{base_date}/{run_id}/summary.json
```

launchd 用の固定ログとは役割を分ける。

- `logs/cron/f286_data_r3_daily_refresh.log`: cron runner 全体の固定ログ
- `logs/f286/...`: job 単位の詳細ログ

### 失敗時の停止

いずれかの required job が失敗した場合:

- 後続 job は実行しない
- summary JSON に `skipped_due_to_previous_failure` を記録する
- 全体 exit code は 1
- LINE 送信は F286-DATA-R3-D2 / sub-D2 では行わない

optional job を設ける場合:

- optional job 失敗時も summary には failed と記録する
- 後続停止するかは job 定義の `continue_on_failure` で明示する
- 初期設計では全 job required を推奨する

### rollback

F286 runner 自体で横断 rollback は行わない。

理由:

- 各 sub-runner は既存の idempotent 設計を前提とする
- SQLite 複数処理を process 横断 transaction にするのは複雑で危険
- 途中成功した fetch 結果は再実行で整合回復する前提が実務上扱いやすい

失敗時復旧方針:

- summary JSON と job log を確認
- 失敗 job から再実行するか、F286 全体を再実行する
- sub-runner は upsert / unique key / date scope により重複 write を避ける

## 7. cron 移行準備 (= 手動運用 1-2 週間 → sub-D3)

D2 / sub-D2 完成後は cron 登録せず、手動実行で 1-2 週間観察する。

手動実行コマンド:

```bash
.venv/bin/python -m scripts.jobs.run_f286_data_r3_daily_refresh \
  --db-path data/fire.staging.db \
  --db-label staging \
  --base-date $(date +%Y-%m-%d) \
  --write
```

観察項目:

- 全 job が想定順序で実行されるか
- dry-run preview と write 結果の row 数に大きな乖離がないか
- F100 / F101 API 失敗時に exit code と logs が追えるか
- 再実行時に duplicate や不整合が発生しないか
- 1 回あたりの実行時間が運用 window に収まるか
- 月曜 07:30 起動想定で既存 F100 取得 timing と衝突しないか

sub-D3 への昇格条件:

- 1-2 週間の staging 手動運用で致命的失敗なし
- 失敗時の再実行手順が確認済み
- logs / summary JSON から障害切り分け可能
- Architect が launchd 登録を承認

## 8. ログ設計 (= 固定 path + logrotate)

### ログ種別

F286 ではログを 3 種に分ける。

| 種別 | path | 用途 |
|---|---|---|
| cron 固定ログ | `logs/cron/f286_data_r3_daily_refresh.log` | launchd StandardOutPath / StandardErrorPath から見る全体ログ |
| job 詳細ログ | `logs/f286/{base_date}/{run_id}/{job_id}.stdout.log` / `.stderr.log` | sub-runner 単位の調査 |
| summary JSON | `logs/f286/{base_date}/{run_id}/summary.json` | 機械可読な実行結果 |

### rotation 方針

sub-D3 で以下のいずれかを採用する。

推奨 1: OS 側 logrotate

- launchd 固定ログを OS 側で rotate
- runner 実装が単純
- Mac mini 運用で logrotate 設定の管理が別途必要

推奨 2: Python `TimedRotatingFileHandler`

- runner 内部で日次 rotate
- Python のみで完結
- launchd stdout/stderr とは別に application log を持つ設計にする必要がある

D3 初期実装では、固定 path + Python `TimedRotatingFileHandler` を推奨する。理由は、F286 runner の内部状態と summary JSON 生成を同じ run_id で紐づけやすいためである。

ただし launchd の StandardOutPath / StandardErrorPath は固定 path のままとし、日付変数は使わない。

## 9. 既知のリスク (= 月曜 07:30 起動と F100 取得衝突等)

### 月曜 07:30 起動と F100 J-Quants 取得衝突

月曜 daily refresh を 07:30 JST に後ろ倒しすることで F282 weekly snapshot との順序矛盾は避けられる。一方で、当日 06:00-07:30 に別系統の F100 J-Quants 取得が走る場合、DB write や API rate limit が衝突する可能性がある。

sub-D2 Impl で確認すること:

- 既存 F100 定時取得の曜日 / 時刻
- F100 historical と intraday の対象 date range
- SQLite writer が同時に走らないこと
- J-Quants API rate limit に抵触しないこと

### Python 環境の明示

subprocess 呼び出しでは `.venv/bin/python` を明示する。

`python` や system Python に依存しない。

### env / CLI 引数の優先順位

既存 sub-runner が `FIRE_ENV` を見るかは未確定である。多くの runner は `--db-path` / `--db-label` で十分な可能性がある。

設計上は CLI 引数を正とし、env は補助として注入する。

優先順位:

1. F286 CLI の `--db-path`
2. F286 CLI の `--db-label`
3. subprocess env `DB_PATH`
4. subprocess env `FIRE_ENV`

### dry-run 出力 schema の不統一

既存 sub-runner の dry-run / check 出力が JSON に統一されていない可能性がある。

sub-D2 では以下の対応を行う。

- JSON parse できる場合は structured result として集約
- parse できない場合は raw log として保存
- 将来、各 sub-runner の dry-run output schema を揃える改善 task を切り出す

### idempotency 前提

途中失敗時の rollback は F286 では行わないため、各 sub-runner の idempotency が重要である。

sub-D2 では少なくとも以下を確認する。

- date scope が明示されること
- unique key / upsert / replace 方針があること
- 再実行で row 重複しないこと

## 10. 実装計画 (= sub-D2 Impl の sub-task 分割)

### sub-D2-1: job 定義と引数 mapping

- `DailyRefreshJob` dataclass を定義
- `list_daily_refresh_jobs()` を実 sub-runner 一覧に更新
- F100 historical / intraday / F101 announcements / F111-R4 / F119 の command mapping を確定
- 既存 runner の dry-run / write 引数名を確認

### sub-D2-2: subprocess helper

- `.venv/bin/python` を使う command builder を実装
- env 注入 helper を実装
- timeout を実装
- stdout / stderr capture を実装
- elapsed time を記録
- exit code を `JobRunResult` に格納

### sub-D2-3: dry-run plan 詳細化

- `plan_daily_refresh()` から各 sub-runner の dry-run / check を呼ぶ
- stdout JSON parse を実装
- parse 不能時の raw fallback を実装
- estimated row count / would_write summary を集約

### sub-D2-4: --write 統合

- sequential execution を実装
- required job failure で後続停止
- 全体 exit code 0 / 1 を制御
- summary JSON を出力
- job 詳細 logs を保存

### sub-D2-5: tests

- mocked subprocess で dry-run success を検証
- mocked subprocess で write success を検証
- timeout 時に後続停止することを検証
- non-zero exit code 時に後続停止することを検証
- stdout JSON parse / raw fallback を検証
- `--dry-run` と `--write` 同時指定が validation error になることを検証

### sub-D2-6: manual operation doc

- staging 手動実行手順を README または design doc に追記
- 失敗時の再実行手順を明記
- sub-D3 へ進む前の観察 checklist を明記

## 11. sub-D3 への申送り (= cron 登録時の 2 暫定方針反映)

sub-D3 は D2 / sub-D2 の手動運用観察後に実施する。

必ず反映する HQ 暫定方針:

- 案 A: 月曜 07:30 JST、火-金 06:00 JST
- 案 B: 固定ログ path + rotation

sub-D3 実装対象:

- `docs/launchd/jp.fire.data-r3-daily-refresh.plist` 作成
- launchd 登録手順の doc 化
- `logs/cron/f286_data_r3_daily_refresh.log` 固定 path の採用
- logrotate または Python `TimedRotatingFileHandler` の確定
- staging で 1 週間運用観察
- 安定性 review

sub-D3 で行わないこと:

- F282 weekly snapshot 自体の再設計
- Computer Use / Playwright による証券操作
- LINE 本番送信の追加
- DB schema migration
- GitHub workflow 変更

## 本線 Architect への確認事項

1. F101 announcements 取得 runner の正式ファイル名は何か。
2. F119 runner は `scripts/jobs/run_f119_evaluation.py` が正か、別名 runner が正か。
3. 各 sub-runner の dry-run / check / write 引数名は統一されているか。
4. F100 intraday は daily refresh で毎回必須か、条件付き optional か。
5. 月曜 07:30 JST 起動時に、既存 F100 J-Quants 取得と衝突する定時 job はあるか。
6. `FIRE_ENV=staging` を参照する runner はどれか。CLI `--db-path` 優先で問題ないか。
7. summary JSON の保存先は `logs/f286/{base_date}/{run_id}/summary.json` でよいか。
8. sub-D3 の rotation は OS logrotate と Python `TimedRotatingFileHandler` のどちらを採用するか。初期推奨は Python `TimedRotatingFileHandler`。
9. staging 手動運用の観察期間は 1 週間で足りるか、2 週間を必須とするか。
10. optional job を許容するか。初期推奨は全 job required。
