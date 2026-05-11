---
id: F286-DATA-R3-D2
phase: P9 / 第 25 章 F286 DATA pipeline / 第 22 章 R-22-08
priority: 中
status: 起票 (= 2026-05-11、Wave 4 W4-4、設計のみ、Impl は別 Wave)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-DATA-R3 skeleton (= commit e9c41c3)
  - F286-DATA-R1.5 / R1.6 (= 既存 staging write 実績)
  - FIRE-CODEX-R1 v1.1 Wave 3 sub-4D / sub-4E
chapter: 第 25 章 / R-22-08
---

# F286-DATA-R3-D2: 実 fetch / 実 write 統合 設計 + dry-run plan

最終更新: 2026-05-11

## ★ 状態: 起票 (= Wave 4 W4-4、設計 doc のみ、Impl は Wave 5+)

F286-DATA-R3 skeleton (= commit e9c41c3) に **実 fetch / 実 write**
を統合する設計案を作成。本タスクは **設計 doc 出力のみ**、実装本体は
HQ approve 後の Wave 5+ で別途実施。

cron 本番登録 (= sub-D3) は **凍結継続**。

## sub-D3 cron 登録への HQ 暫定方針 (= 受領済)

Wave 3 sub-4E CRITICAL 2 件への HQ 判定:

1. **F282 weekly snapshot との月曜順序矛盾**:
   - **案 A 暫定採用**: 月曜 daily refresh は **07:30 JST** に後ろ倒し
     (= 7:00 JST の F282 snapshot 後に走らせる)
   - 火-金は 06:00 JST のまま
   - F282 weekly snapshot 自体の見直しは **将来 FIRE-OPS-R0 候補**

2. **launchd 日付付きログ問題**:
   - **固定ログ + rotation 設計優先**
   - launchd `StandardOutPath` には日付変数を期待しない
   - runner 内部で `logs/cron/daily_refresh.log` 等の固定 path にログ
     出力、別途 logrotate で日次 rotate

これらの暫定方針は sub-D3 (= 将来の cron 登録) で適用、本 D2 では
適用しない (= 設計 doc 内で参照のみ)。

## 設計対象スコープ

### 1. 実 fetch の sub-runner 一覧 (= 既存 runner を呼ぶか、新規実装か)

F286-DATA-R3 skeleton の `list_daily_refresh_jobs()` が返す job 定義
list が、実際にどの既存 runner を呼ぶかの設計:

| job_id                                     | 既存 runner                                          | 必要な引数 |
|---|---|---|
| f100_historical                            | scripts/jobs/fetch_historical_market_data.py         | --from / --to / --db-path |
| f101_announcements                         | scripts/jobs/fetch_announcements.py (= 既存確認要)  | --from / --to |
| f119_evaluation_seed                       | (= 別 runner、確認要)                                | --base-date |
| f111_research_watchlist_signal_persistence | scripts/jobs/run_research_watchlist_signal_persistence.py | --write |
| f142_replay (option)                        | (= 後続候補)                                         | (TBD) |

各 job は subprocess で起動するか、Python から import して呼ぶか:
- subprocess 案: 既存 runner をそのまま呼べる、ログが分離
- import 案: より control 細かい、ただし依存関係が複雑

**推奨**: subprocess 案 (= 既存 runner pattern 維持、shell からも同じ
コマンドで再実行可能、log も分離)

### 2. 実 write 統合 (= 既存 runner の --write を D2 から渡す)

各 sub-runner は既に default dry-run / --write 経路を持つ (= F100
historical / F101 / F111-R4 等は establish 済)。D2 は以下を行う:

1. `plan_daily_refresh(read_only=False)` で「実 write を伴う計画」を返す
2. 計画通り subprocess で各 sub-runner を順次呼び出し
3. 各 sub-runner の --db-path に **fire.staging.db** を強制
4. 各 sub-runner の exit code を集計、いずれか失敗なら D2 全体 fail

ガード:
- F286-DATA-R3 自体の三段ガード (= db_label='staging' + basename)
- 各 sub-runner にも個別ガードがある (= 多層防御)
- subprocess 呼び出し時の env として FIRE_ENV=staging を明示

### 3. dry-run plan の詳細化

skeleton の `plan_daily_refresh(read_only=True)` は現在「current_rows
SELECT + jobs list」のみ。D2 で以下を追加:

- 各 sub-runner の --dry-run / --check 経路を順次呼び出し、想定 row 数
  を集計
- 「もし --write したらどうなるか」のプレビュー出力

実装:
```python
def dry_run_each_job(jobs: list[dict], *, db_path: Path) -> list[dict]:
    """各 job の dry-run を実行、想定影響を返す."""
    out = []
    for job in jobs:
        # subprocess で job の --dry-run 経路を呼ぶ
        # estimated_writes を集計
        ...
    return out
```

### 4. エラーハンドリング戦略

- 1 sub-runner 失敗 → 後続 jobs を実行するか即停止か
- 推奨: 即停止 (= 依存関係がある、F100 失敗 → F119 失敗連鎖)
- 失敗時の rollback: 個別 sub-runner は idempotent 想定、再実行で復旧

### 5. cron 移行への準備 (= sub-D3 前提条件への申送り)

D2 が完成 → 手動実行で安定性確認 → sub-D3 で cron 登録 という流れ:

- D2 完成後、Fujiwara が **手動で** 毎日 06:00 JST に実行して
  1-2 週間運用観察
- 1-2 週間問題なければ sub-D3 で launchd 登録 (= HQ approve 必須)
- sub-D3 では:
  - 月曜 07:30 JST + 火-金 06:00 JST スケジュール (= 案 A)
  - 固定ログ + logrotate 設計
  - failure alert 経路 (= LINE 通知部屋連携、要設計)

### 6. cron 暫定方針への対応

D2 では cron 起動を実装しないが、**runner の挙動として**:
- `--target-date YYYY-MM-DD` で任意日 fetch 可能 (= 月曜 07:30 から
  起動しても、--target-date で前営業日を指定できる)
- 営業日判定: 当日 JST が土日なら前営業日 fallback (= 既存)
- F282 snapshot 直後の月曜 07:30 起動時の動作確認 test を計画

## 期待される設計 doc (= Wave 4 W4-4 成果物)

新規 vault doc: `03_design/F286_DATA_R3_D2_real_fetch_write_2026-05-11.md`

構造:
```
# F286-DATA-R3-D2 Real Fetch / Write Integration Design
version: v1.0 (draft)
date: 2026-05-11

## 1. 目的
## 2. 既存 sub-runner 一覧 + 引数 mapping
## 3. subprocess vs import 比較 + 推奨
## 4. dry-run / --write 経路の統合
## 5. エラーハンドリング戦略
## 6. cron 暫定方針 (案 A + 固定ログ) への対応
## 7. 実装計画 (= sub-D2 Impl の sub-task 分割)
## 8. 既知のリスク
## 9. Wave 5+ への申送り (= Impl 着手前条件)
```

## 実装計画 (= sub-D2-Impl 着手時の sub-task 分割案)

Wave 5 候補 (= D2 設計 approve 後):
- W5-1 (L3 Impl): F286-DATA-R3 runner に実 fetch 統合
- W5-2 (L2 Test): 各 sub-runner mock test + 失敗 propagation test
- W5-3 (L4 Audit): subprocess 呼出 / env injection の安全 audit
- W5-4 (L5 Docs): D2 完成 vault doc

## allowed_files (= Wave 4 W4-4 の Codex 投入用)

- /tmp/f286_data_r3_d2_design_draft.md (= Codex 出力先)
- (本線で 03_design/ に migrate する)

## forbidden_files

- scripts/jobs/run_f286_data_r3_daily_refresh.py (= 既存 skeleton、
  touched なし、修正は Wave 5 W5-1)
- 全 既存 sub-runner (= scripts/jobs/fetch_*, run_*)
- ~/Library/LaunchAgents/*
- docs/launchd/* (= 既存緊急アラート plist)
- crontab
- 全 既存禁止 files (= seed_pattern_layer1.py / historical_indicators.py /
  TODO Excel / workflows)

## 安全要件

- 本 W4-4 は **設計 doc 出力のみ**、実装変更なし
- DB write 0 / LINE 送信 0 / cron 登録 0
- subprocess 呼出も実施しない (= 設計のみ)
- token / secret / channel_token 参照なし

## Codex 投入予定 (= HQ approve 後)

| 段階 | 内容 | lane | 想定時間 |
|---|---|---|---|
| Codex L1 Design | 設計 doc draft を /tmp/ に出力 | L1 | 10-15 分 |
| 本線 Architect review | 設計判断 + 03_design/ へ migrate | (本線) | 10 分 |
| commit | docs(F286-DATA-R3): D2 real fetch/write integration design | (本線) | 5 分 |

## 関連リンク

- [[FIRE_CODEX_R1_WAVE4_plan|Wave 4 plan]]
- [[FIRE_CODEX_R1_WAVE3_audit_results|Wave 3 sub-4D / sub-4E audit]]
- F286-DATA-R3 skeleton commit: `e9c41c3`
- [[../log]]
