---
id: FIRE-CODEX-R1-WAVE27-plan
phase: ガバナンス / Wave 27 / Phase P2 + 6 KPI 初試行 / F282 dry-run probe 実施
priority: 高
status: 進行中 ☆ 6 lane (本線 2 + Codex 4)、実 dry-run probe 1 回、ro 接続のみ
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 26 (= 完了、F282 snapshot impl)
  - HQ Wave 27 起票承認 + dry-run probe 実行承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / cron thaw Step 1 dry-run
---

# Wave 27: F282 dry-run probe 実施 + 6 KPI 初試行

W26 で実装した F282 snapshot script の `--dry-run` path を **実行**。
production fire.db への read-only 接続 1 回、実 write 0 を保証。
HQ 補足方針 (= 2026-05-12) に従い **6 KPI 駆動運用** を本 wave で初試行、
lane 数は 6 (= task に必要十分) に絞る。

## Wave 27 sub-task (= 6 lane、本線 2 + Codex 4)

| sub  | lane | owner | task                                          |
|------|------|-------|-----------------------------------------------|
| W27-1| L5   | 本線  | plan + git status + mtime baseline + 4 prompt |
| W27-2| L1a  | Codex | dry-run probe 手順詳細設計                    |
| W27-3| L1b  | Codex | 期待出力 / 失敗時 triage 設計                 |
| W27-4| L3   | 本線  | **実 dry-run probe 実行** (= production ro × 1)|
| W27-5| L4   | Codex | adversarial audit                             |
| W27-6| L6   | Codex | regression 確認 plan                          |
| W27-7| 本線  | 本線  | 結果統合 + 6 KPI 集計 + commit + 報告         |

6 lane 構成: 本線 2 (L5+L3) + Codex 4 (L1a+L1b+L4+L6)。補足方針に従い
「数」より「task 量」で適応選択。

## 実行前 mtime baseline (= W27 必須確認 #1)

| file | size | mtime |
|---|---|---|
| /Users/bluefire/fire/data/fire.db | 371,081,216 | 2026-05-12 16:17:24 |
| /Users/bluefire/fire/data/fire.develop.db | 371,081,216 | 2026-05-12 16:11:43 |
| /Users/bluefire/fire/data/fire.staging.db | 4,804,063,232 | 2026-05-12 18:45:22 |

実行後にこれら全てが **unchanged** であることを確認する。

## disk free baseline (= W27 必須確認 #7)

- `/Users/bluefire/fire/data/` 配下 disk free: 831 GiB
- source_size × 2 必要量: 742 MB (= 371 MB × 2)
- 余裕: 831 GiB - 742 MB ≒ 830 GiB (= 桁違いに余裕)

## file lock 表 (= R2 v1.1)

### 既存 modified (= 本 wave 範囲外)

| file | 状態 | 区分 |
|---|---|---|
| scripts/seed_pattern_layer1.py | M | forbidden |
| simulation/research_lane/historical_indicators.py | M | forbidden |
| .claude/ | ?? | 範囲外 |
| data/fire.staging.db.pre_restore_* | ?? | 範囲外 |

### 本 wave で書込む file

| file | 状態 | owner_lane | 区分 |
|---|---|---|---|
| 02_todo/FIRE_CODEX_R1_WAVE27_plan.md | NEW | 本線 (L5) | allowed |
| 02_todo/FIRE_CODEX_R1_WAVE27_results.md | NEW | 本線 | allowed |
| log.md | MOD | 本線 | allowed |

fire develop には **commit なし** (= 本 wave は実行 wave、code 変更 0)。

### Codex 4 lane output

- /tmp/codex_wave27/output/l1a_*.{stdout,last,stderr}
- /tmp/codex_wave27/output/l1b_*.{stdout,last,stderr}
- /tmp/codex_wave27/output/l4_*.{stdout,last,stderr}
- /tmp/codex_wave27/output/l6_*.{stdout,last,stderr}

### 本線 L3 が触る file (= read-only 接続のみ)

- /Users/bluefire/fire/data/fire.db (= production、read-only)
- /Users/bluefire/fire/data/ (= target_dir、書込み権限 probe のみ)
- /Users/bluefire/fire/data/fire.staging.db (= target path resolve、書込み 0)
- /Users/bluefire/fire/data/fire.develop.db (= target path resolve、書込み 0)

## 実行コマンド (= HQ 承認下)

```bash
FIRE_ENV=snapshot /Users/bluefire/fire/.venv/bin/python \
  /Users/bluefire/fire/scripts/jobs/run_f282_weekly_snapshot.py \
  --dry-run \
  --db-source production \
  --db-targets staging,develop
```

期待結果:
- exit 0
- stdout に `write_count=0, probe_ok=True, source_size=371081216, target_count=2`
- 全 DB mtime 不変

## W27 必須確認 (= HQ 指示 10 項目、全 達成必須)

1. ✓ 実行前 production/develop/staging DB mtime 記録 (= 上記 baseline)
2. dry-run 実行
3. write_count=0 確認
4. probe_ok=True 確認
5. target_count=2 確認
6. source_size 記録
7. ✓ disk free OK 確認 (= 上記 baseline)
8. 実ファイル作成 0 確認
9. 実行後 production/develop/staging DB mtime unchanged 確認
10. 結果を 1 ブロックで HQ 報告 (= 本線最終 task)

## 6 KPI 集計 (= HQ 補足方針 2026-05-12 初試行)

| KPI | 目標 | 計測方法 |
|---|---|---|
| Codex 稼働率 | 50% 以上 | 並列起動 lane 数 / 想定上限 (= 12+) |
| 本線短縮率 | 50% 以上 | (本線単独推定 - wave 実時間) / 本線単独推定 |
| 成果物採用率 | 100% | merge された Codex 出力数 / 全 |
| 差し戻し率 | 0% | 破棄された Codex 出力数 / 全 |
| Integrator 負荷 | < 60 分 | 本線実時間 / 150 分上限 |
| 安全事故 0 | 0 | 未承認 LINE / DB write / token 等 |

## 成功条件 (= P2 運用条件 + W27 固有)

| 条件 | 期待 |
|---|---|
| 6 lane 全完了 | ✓ |
| file ownership 衝突 0 | ✓ |
| CRITICAL 0 | ✓ |
| L4 HIGH 0 | ✓ |
| safety violation 0 | ✓ |
| 全 pytest PASS | ✓ (= 4,068 維持) |
| wave 実時間 < 150 分 | ✓ |
| Codex 直接 commit 0 | ✓ |
| 本線過負荷 0 | ✓ |
| 全 DB mtime unchanged | ✓ (= 必須確認 #9) |
| dry-run probe exit 0 | ✓ |
| write_count=0 | ✓ |
| HQ 報告 1 ブロック | ✓ |

## 禁止 (= HQ Wave 27 指示)

- VACUUM INTO 実行
- snapshot file 生成
- plist 配置
- launchctl load
- cron / launchd / crontab 登録
- DB write
- LINE 送信
- token / secret / channel_token 参照
- F101 staging probe
- workflow 変更
- --no-verify
- TODO Excel 更新

## 関連リンク

- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計 (W25)]]
- [[FIRE_CODEX_R1_WAVE26_results|Wave 26 results (= F282 impl)]]
- /tmp/codex_wave27/prompts/* (= session-local)
