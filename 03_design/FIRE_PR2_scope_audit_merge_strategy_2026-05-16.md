# FIRE PR #2 scope audit / merge strategy review (2026-05-16)

doc_id: FIRE-PR2-SCOPE-AUDIT-MERGE-STRATEGY-2026-05-16
status: read-only audit 完了 (= merge は未実施)
related:
- PR: https://github.com/BlueFire7777/fire/pull/2
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_SMOKE_2026-05-16.md (W2-A smoke)
- F286-PNL-R3 / F286-REPORT-R1 / F062-R5 系の累積 wave doc 群


## §1 目的

PR #2 (develop → main、226 commit / 311 files / +133,235 / -70) を merge する前に、
scope と安全性を read-only で監査し、merge 戦略を提案する。
本 wave は audit のみ。merge / PR update / PR close は実施しない。


## §2 PR 概要

| 項目 | 値 |
|---|---|
| number | #2 |
| title | feat(F111): W2-A theme/sector overlay + Universe Expansion baseline |
| state | OPEN |
| base | main |
| head | develop |
| commits | 226 |
| changedFiles | 311 |
| additions | +133,235 |
| deletions | -70 |
| mergeable | MERGEABLE |
| mergeStateStatus | **BLOCKED** (= review 必須、branch protection rule on main) |
| reviewDecision | REVIEW_REQUIRED |
| isDraft | false |
| labels | (none) |
| url | https://github.com/BlueFire7777/fire/pull/2 |


## §3 changed files 311 件 カテゴリ分類

| カテゴリ | count | 主な内容 |
|---|---|---|
| tests | **145** | 多数の新規テスト (research_lane / scripts/jobs / agents 系) |
| scripts/jobs | 62 | F062 / F111 / F119 / F282 / F286 / production_v0 系 runner |
| simulation code | 34 | lane_c / research_lane / paper_live 拡張 |
| scripts/setup | 12 | migration 系 (F281 strict / F286 PNL / SCHEMA-R1 等) |
| agents code | 9 | research_advisory / line_*_send / daytrade_selection / sector_research 等 |
| pnl code | 8 | F286 PNL モジュール (paper_pnl / ingestion / snapshot 等) |
| fire/report/* (other) | 7 | F286-REPORT-R1 reporting module (新規) |
| bin (shell) | 7 | bin/fire-{dev,prod,staging,*-env.sh,*-init-develop.sh, ...} F282 環境分離 wrapper |
| notifications code | 4 | line_bot / listing_name_lookup / send_guard 等 |
| evaluation code | 4 | aggregators / interpretation_evaluator 等 |
| market_data code | 3 | client / index_fetcher 等 |
| patterns code | 2 | reproducibility / similarity |
| docs/launchd plist | 2 | jp.fire.weekly-snapshot{,-smoke}.plist (source 配置のみ) |
| execution code | 1 | slippage_monitor |
| features code | 1 | regime |
| materials code | 1 | client |
| docs (markdown) | 1 | codex-review-setup.md |
| docs/cron | 1 | f282_cron.txt (source のみ、crontab install ではない) |
| docs/logrotate.d | 1 | fire (= log rotation config source) |
| scripts/hooks | 1 | pre-commit (Codex review enforcement) |
| scripts/other | 2 | emergency_alert / seed_pattern_layer1 |
| data (text/csv) | 1 | data/lane_c_tier2_universe.txt (= text、DB ではない) |
| CLAUDE.md | 1 | プロジェクト記憶更新 |
| .gitignore | 1 | DB 除外などの ignore 更新 |
| **危険カテゴリ** | **0** | env / secret / token / credential / workflows / DB / LaunchAgents 直接 deploy |


## §4 危険ファイル混入確認

| 危険カテゴリ | 検出件数 | 備考 |
|---|---|---|
| `.env` / `.env.*` | **0** | ✓ |
| `*token*` / `*secret*` / `*credential*` | **0** | ✓ |
| `.github/workflows/` | **0** | ✓ (R-01-08 不採用方針継続) |
| `data/*.db` / `*.db-*` / `.db.bak*` | **0** | ✓ (.gitignore で除外済) |
| `~/Library/LaunchAgents/` への直接配置 | **0** | ✓ (plist source は docs/launchd/ のみ) |
| 大型生成物 / バイナリ | **0** | ✓ (最大 file は tests/scripts/jobs/test_run_f286_after_r1_night_batch.py = 3018 行) |
| `--no-verify` bypass 実行 | **0** | ✓ (記載は全て「禁止」ポリシー + test assertion) |

`--no-verify` 記載 8 箇所すべて確認:
- pre-commit hook: 「**`--no-verify` での回避は禁止**」
- CLAUDE.md: 「**`--no-verify` を含むあらゆる skip を行わない**」
- task doc: 「Codex pre-commit / `--no-verify` 禁止」
- test assertion: `assert "--no-verify" not in arg.value`

→ bypass を **強化する** 文脈のみ。bypass 実行コードなし。


## §5 主な変更領域 (= main に入ると何が変わるか)

### §5.1 commit type 内訳 (226 commit)

| type | count | 比率 |
|---|---|---|
| feat | 105 | 46.5% |
| test | 44 | 19.5% |
| chore | 37 | 16.4% |
| docs | 21 | 9.3% |
| fix | 17 | 7.5% |
| その他 | 2 | 0.9% |

### §5.2 wave/feature 別 (top タグ)

| タグ | commits | 内容 |
|---|---|---|
| F286-R2 | **45** | Advisory Decision PnL tracking (advisory_decisions / snapshot / staging-only / 三段ガード) の R2 evolution 全体 |
| F062-R5 | 13 | Buyability Card UX / Action UX / Production Send / Label Refresh |
| F286-DATA-R1 | 8 | Daily Refresh + R3 plan |
| F286-R1 / F281-C2 | 7+7 | initial schema / strict eval mode + lane_code |
| F111 + R1-R4 | 6+12 | Daytrade Selection + Universe Expansion W1-W3 |
| F119 | 6 | Evaluation Agent Phase 1-3 完成 + 承認フロー + LINE 通知 |
| F286-PNL-R3 / R2 / R1 | 4+4+3 | Paper PnL Computation + Snapshot ingest + Decision tracking |
| F286-REPORT-R1 | 3 | Daily/Weekly/Monthly PnL Report Generator |
| F062-R1〜R4 | 4+3+4+4 | Advisory Render + LINE preview + send_guard |
| F282 | 5 | Environment Isolation + Weekly Snapshot launchd plist |
| F284 / F276 | 4+4 | J-Quants V2 + F104 regime collector + Phase 4 SCORE_THRESHOLD_EXECUTE |
| F286-DATA-R3 / R2 | 5+3 | Daily Refresh subprocess dry-run plan + write 設計 |
| F105 | 6 | (詳細別 doc) |

### §5.3 production v0 系

- F286 系全体が production v0 readiness の核 (= Advisory Decision tracking、Paper PnL、Daily Report)
- F282 環境分離 (staging / develop / production DB 切替)
- production v0 readiness check / ops summary CLI

### §5.4 safety / ops 系

- bin/fire-{dev,prod,staging,*-init-develop.sh} 環境切替 wrapper (FIRE_ENV → DB_PATH)
- docs/launchd/jp.fire.weekly-snapshot{,-smoke}.plist source
- docs/cron/f282_cron.txt cron template
- scripts/hooks/pre-commit Codex review enforcement
- CLAUDE.md「崩してはならない前提」9 点 維持

### §5.5 tests

- 145 files の新規 / 拡張テスト
- 全 wave で test PASS 累計 **4,000+ PASS** (CLAUDE.md log 参照)
- 直近 W2-A smoke で 456 PASS 確認済


## §6 merge 戦略比較 6 案

| 戦略 | 概要 | メリット | デメリット | FIRE 整合性 |
|---|---|---|---|---|
| **A. 一括 merge commit** | "Create a merge commit" で 226 commit を保全して main に入れる | history 全保全 / bisect 可能 / revert はその merge commit 1 個でロールバック可 | history 量が増える / main 上で複雑な merge graph | ◎ (CI/CD なし環境では問題小) |
| **B. squash merge** | 226 commit を main 上で 1 commit に圧縮 | history 簡潔 / 1 commit で表現 / revert 容易 | individual commit の bisect/blame が消失 / 学習材料が失われる | △ (個別 commit 詳細は develop に残るが main からは消える) |
| **C. 通常 merge commit** | A と実質同等 (3-way merge commit) | A と同じ | A と同じ | A と同じ |
| **D. rebase merge** | 226 commit を main 上に rebase で線形化 | history 線形 / bisect 可 | 226 commit に rebase は高 risk (conflict / SHA 変化) / main の history が大幅変化 | △ (main 上の 1 commit が 226 にスプレッド、後続の参照 SHA が変わる) |
| **E. 分割 PR 化** | PR #2 を Close し、W2-A 専用 feature branch を d96ca10 起点で切り出して別 PR、他 wave は段階的 | review しやすい / scope 単位で承認可 | branch 分割 + 226 commit を依存順に再構築する手間 / 既に develop で動いている状態を別 branch に切る複雑度 | × (個人開発 + Codex pre-commit 既通過済なので過剰) |
| **F. main 段階追従** | main を一旦 develop の途中 commit に fast-forward、その後段階的 merge | scope 縮小 / 段階確認可 | main 直接更新は branch protection 抵触 / PR 経由しない更新は方針外 | × (main 直 push 禁止) |


## §7 推奨戦略

**推奨: 戦略 A (一括 merge commit)** — GitHub UI の "Create a merge commit" ボタン。

### §7.1 理由

1. **危険ファイル混入 0** (= env / secret / workflows / DB 全て不在)
2. **CI/CD 不採用** (R-01-08) のため、merge 規模が CI 時間を圧迫しない
3. **個人開発** + Mac mini 単独 deploy のため、history の複雑さによる協調コストなし
4. **bisect / blame の保全** が将来の incident 調査で有用
5. **revert 容易** (= 失敗時はその merge commit 1 個を revert)
6. **squash すると 226 commit の各 wave doc / 設計判断と main の commit が乖離** (= ~/fire-vault/03_design/ の各 doc が「commit XXX で実装」と書いているのに main 上には無い、という不整合が発生)

### §7.2 主リスク

| risk | 緩和策 |
|---|---|
| 226 commit 規模ゆえ smoke 漏れ | merge 後 §9 smoke を read-only で実施 |
| main pull 後の production deploy 時に dependency 不整合 | merge 前に `.venv/bin/pip freeze` 比較 (= dependency 変化を事前確認) |
| F286 family 45 commit の scope drift | Codex Lane B verdict 待ち / 必要なら 追加 wave で深掘り |
| F282 環境分離後、production 系 DB 接続経路が変わる影響 | bin/fire-prod 経路を本 audit 後に体感確認 |
| 過去の incident commit 単独 revert が squash で不可能 | A 戦略を選んで個別 commit 残存 |


## §8 merge 前必須確認 (= reviewer = Fujiwara)

1. PR #2 の "Files changed" タブで 311 files の sampling 確認 (= bin/ / docs/launchd/ /
   scripts/hooks/pre-commit / CLAUDE.md は必読)
2. PR #2 の "Commits" タブで F286-R2 45 件の最低 5 件を中身 sampling
3. CLAUDE.md「崩してはならない前提」9 点との抵触なきこと確認
4. `.venv/bin/pip freeze > /tmp/dep_pre_merge.txt` で merge 前 dependency snapshot 取得
5. 3 DB md5 取得 (= production / develop / staging) で merge 前ベースライン保存
6. F282 plist 2 file が `~/Library/LaunchAgents/` に **deploy されないこと** を再確認
   (= source 配置のみ、launchctl load は人間 manual)
7. 緊急時 revert 手順を頭に入れる (= GitHub UI から "Revert" → 新 PR 自動生成 → review → merge)


## §9 merge 後 smoke 案 (= read-only、6 step)

```
Step 1: branch 同期
  git checkout main
  git pull origin main
  git log --oneline -1     (= merge commit を確認)

Step 2: tests 再実行 (= 456 PASS 維持確認、まず W2-A 関連のみ)
  .venv/bin/pytest tests/scripts/jobs/test_v1_4_1_consumer.py \
                   tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py \
                   tests/agents/test_daytrade_selection.py -q

Step 3: F286 family 主要 test を読み取り側のみ実施
  .venv/bin/pytest tests/pnl/ tests/evaluation/ -q

Step 4: staging read-only smoke
  .venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
    --db-path /Users/bluefire/fire/data/fire.staging.db \
    --base-date 2026-07-22 --max-candidates 50 \
    --recently-seen-codes 9130,8747,137A0,7991,340A0,5729,3489,3798 \
    --output-json /tmp/fire_post_merge_smoke/overlay_staging_f111.json
  (= W2-A overlay top5 が前 wave と一致するか確認)

Step 5: 3 DB md5 比較 (= merge 後不変確認)
  md5 -q data/fire.db data/fire.develop.db data/fire.staging.db
  (= merge 前のベースラインと一致確認)

Step 6: F282 plist 不 deploy 確認
  ls -la ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist
  (= size/mtime が merge 前と同一)
```

実行制約:
- DB write 0 / LINE 送信 0 / API call 0 / token 参照 0 / launchctl 0 / cron 0
- staging DB は URI mode=ro のみ
- 出力は /tmp/fire_post_merge_smoke/ 限定


## §10 Codex 4 lane 監査結果

| Lane | scope | verdict | severity |
|---|---|---|---|
| A | changed files / dangerous files | **APPROVE** | CRITICAL/HIGH 0 |
| B | commit scope / cumulative changes | **APPROVE** | CRITICAL/HIGH 0、LOW のみ (= EOF blank line 2 件) |
| C | merge strategy | **APPROVE_MERGE_WITH_CONDITIONS** / RECOMMENDED_STRATEGY: **A** | CRITICAL/HIGH 0 |
| D | safety / post-merge smoke | **APPROVE** | CRITICAL/HIGH 0 |

集約: APPROVE 4 (= 1 件は WITH_CONDITIONS)、CRITICAL/HIGH 0。

### §10.1 Lane A 詳細
- changed files 311 一致 / env / token / secret / credential / `.db` / `.github/workflows/` 混入 0
- `--no-verify` は禁止ポリシー文書 + test assertion のみ (= bypass 実行 0)
- data/lane_c_tier2_universe.txt は 477 行 ticker list (= DB / 機密ではない)
- plist 2 file は docs/launchd/ source 配置のみ (LaunchAgents deploy なし)
- 大型 file 上位は test / runner / template (= 生成物なし)

### §10.2 Lane B 詳細
- 226 commit 一致、author 全 226 件 `BlueFire7777`、merge commit 0、earliest = b778eb2 (F276-A)
- main 最新も 156b879 (Merge F300) と一致
- commit type 数の小差: feat 105 + test 44 + chore 37 + docs 21 + fix 17 + `docs+test` 等 compound 2 = 226
- F286-R2 45 件は research lane / derived indicators / watchlist / regime / return evaluation の累積で
  「scope drift ではなく研究基盤の正常な大塊」
- F281 strict migration は `FIRE_ENV != "staging"` reject + basename `fire.staging.db` 以外 reject +
  symlink / traversal reject で staging 限定確認
- suspect commit (binary / DB 追加 / 異常 author / merge 事故) なし
- LOW only: `git diff --check` で EOF blank line 2 件 (= merge 阻止級ではない、別 wave で清掃可)

### §10.3 Lane C 詳細
- 6 戦略 (A 一括 merge / B squash / C 通常 merge / D rebase / E 分割 / F 段階追従) を評価
- Mac mini 個人開発 + CI/CD 不採用 (R-01-08) + 自動 deploy なし + main pull は人間操作 という
  FIRE 制約下では「履歴の監査性」「戻しやすさ」が最重要
- **推奨 A**: 226 commit のタスク履歴を壊さず、PR 単位の revert (`git revert -m 1`) も可能、
  rebase/split の追加 risk 回避
- 主リスク: 巨大差分の post-merge 不具合検出遅れ → 緩和策: §9 smoke を merge 直後に実施
- 「今すぐ merge してよいか」: **条件付き可** (= reviewDecision=REVIEW_REQUIRED / mergeStateStatus=BLOCKED
  のため、必須 review 承認待ち、その後 A 戦略で merge 進行可)

### §10.4 Lane D 詳細
- merge だけで自動起動するものは確認できず (`.github/workflows` 変更 0、GitHub Actions 不採用)
- `docs/cron/f282_cron.txt` と `docs/launchd/*.plist` は source 配置のみ
  (= crontab / launchctl install は手動、plist も `RunAtLoad=false`)
- production 反映は Mac mini 上で `git pull origin main` 手動実行が必要
- **重要運用 note**: 「手動 pull 後に既存スケジューラが更新コードを読む」運用リスク
  (= 既存 install 済 cron/launchd は次回実行で更新後 code を読む、merge ≠ 即 deploy だが pull 後は次回 cron 起動が deploy 同等)
- post-merge smoke 案 8 step (Lane D 提案): fetch/stat → 危険ファイル grep → F111 staging read-only →
  v1.4.1 consumer render → morning MD render → F062 preview dry-run → F119 `--dry-run` → F286 report/LINE preview
- 全 step `/tmp` 出力、`--send` / `--write` 不使用、DB read-only、LINE 送信 0、API 呼出 0
- merge 前必須: PR head/stat 一致、`.github/workflows` 不在、env/secret/DB 実体なし、cron/plist 未 install 確認、
  F062/F286 の危険フラグ未使用、DB md5 不変、`core.hooksPath=scripts/hooks`
- revert: 226 commit 個別 revert は非実用、`git revert -m 1` (merge commit revert) で 1 操作で戻せる
  (Lane D は squash merge の revert 容易性を nuance として挙げるが、A 採用なら `-m 1` で同等)


## §11 結論

- 危険ファイル混入 **0**
- Codex 4 lane CRITICAL/HIGH **0** (= APPROVE 4 件、Lane C は WITH_CONDITIONS / RECOMMENDED A)
- mergeable=MERGEABLE / mergeStateStatus=BLOCKED (= review 必須、branch protection 機能中)
- 推奨戦略: **A 一括 merge commit** (GitHub UI "Create a merge commit")

### §11.1 merge 可否

**条件付き merge 可能** — 以下を満たした上で別途 HQ 承認下に実施:
1. branch protection の review 必須を承認 (= Fujiwara が GitHub UI で Review → Approve)
2. §8 merge 前必須確認 7 項目クリア
3. merge 直後に §9 post-merge smoke 6 step 実施 (= read-only / staging のみ / `/tmp` 出力限定)
4. 失敗時は `git revert -m 1 <merge_commit_sha>` で速やかに rollback (= 別 PR 自動生成)

### §11.2 本 wave 範囲

本 wave は **scope audit + merge 戦略提案のみ**。merge / PR update / PR close は実施せず。
次 wave で「PR #2 merge を実施」を別途 /goal で HQ_APPROVE_PR2_MERGE_DEVELOP_TO_MAIN マーカー
付きで起票し、戦略 A (= "Create a merge commit") で実行する。
