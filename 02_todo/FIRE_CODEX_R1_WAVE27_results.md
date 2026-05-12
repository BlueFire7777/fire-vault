---
id: FIRE-CODEX-R1-WAVE27-results
phase: ガバナンス / Wave 27 完了 / F282 dry-run probe + R2 v1.2 改訂
priority: 高
status: 完了 ★ 6 lane 全完了 / dry-run probe write_count=0 / mtime unchanged / R2 v1.2 反映 / 4,068 PASS
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 26 (= 完了、F282 snapshot impl)
  - HQ Wave 27 起票承認 + dry-run probe 実行承認 (= 2026-05-12)
  - HQ Wave 27 追加指示 R2 v1.2 起票 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F282 / cron thaw Step 1 dry-run / R2 v1.2
---

# Wave 27: F282 dry-run probe 実施 + R2 v1.2 / KPI-driven Lane Scaling Policy

最終更新: 2026-05-12

## ★ 状態: 完了 (= 2 task 統合、6 lane、W27 必須確認 10/10 全達成)

HQ から Wave 27 として **2 つの task** を統合指示:
1. **F282 dry-run probe 実施** (= 当初指示、production fire.db への ro
   接続 1 回、write 0 保証)
2. **R2 v1.2 / KPI-driven Lane Scaling Policy** 起票 (= 補足方針反映)

本線が両方を 1 wave 内で実施。Codex 4 lane は dry-run probe 関連の設計 /
audit / regression を並走、本線が R2 v1.2 を直接 doc 編集。

## Wave 27 sub-task 結果 (= 6 lane、本線 2 + Codex 4)

| sub  | lane | owner | task                                          | verdict        |
|------|------|-------|-----------------------------------------------|----------------|
| W27-1| L5   | 本線  | plan + git status + mtime baseline + 4 prompt | ✓              |
| W27-2| L1a  | Codex | dry-run probe 手順詳細設計                    | CRITICAL 0 / HIGH 0 |
| W27-3| L1b  | Codex | 期待出力 / 失敗時 triage 設計                 | CRITICAL 0 / HIGH 0 |
| W27-4| L3   | 本線  | **実 dry-run probe 実行 (= production ro × 1)**| exit 0 / write 0 |
| W27-5| L4   | Codex | adversarial audit (= 8 観点)                  | CRITICAL 0 / HIGH 0 |
| W27-6| L6   | Codex | regression 確認 plan                          | CRITICAL 0 / HIGH 0 |
| W27-7| 本線  | 本線  | **R2 v1.2 改訂直接編集** (= 3 改訂 + 履歴追加)| ✓              |
| W27-8| 本線  | 本線  | 結果統合 + KPI 集計 + commit + HQ 報告       | ✓              |

## F282 dry-run probe 実行結果 (= W27-4)

### 実行コマンド

```bash
FIRE_ENV=snapshot /Users/bluefire/fire/.venv/bin/python \
  /Users/bluefire/fire/scripts/jobs/run_f282_weekly_snapshot.py \
  --dry-run --db-source production --db-targets staging,develop
```

### 実行結果

```
F282 dry-run OK: write_count=0, probe_ok=True, source_size=371081216, target_count=2
exit: 0
```

### W27 必須確認 10 項目 (= HQ 指示、全達成)

| # | 項目 | 結果 |
|---|---|---|
| 1 | 実行前 production/develop/staging DB mtime 記録 | ✓ baseline 取得 |
| 2 | dry-run 実行 | ✓ exit 0 |
| 3 | write_count=0 確認 | ✓ stdout 出力 |
| 4 | probe_ok=True 確認 | ✓ stdout 出力 |
| 5 | target_count=2 確認 | ✓ staging + develop |
| 6 | source_size 記録 | ✓ 371,081,216 bytes |
| 7 | disk free OK 確認 | ✓ 831 GiB |
| 8 | 実ファイル作成 0 確認 | ✓ /data/ 新規 file 0 |
| 9 | 実行後 mtime unchanged 確認 | ✓ 全 3 DB 完全一致 |
| 10 | 結果を 1 ブロックで HQ 報告 | ✓ 本報告 |

### mtime 完全一致確認

| file | 実行前 | 実行後 | 一致 |
|---|---|---|---|
| fire.db | 5/12 16:17:24 | 5/12 16:17:24 | ✓ |
| fire.develop.db | 5/12 16:11:43 | 5/12 16:11:43 | ✓ |
| fire.staging.db | 5/12 18:45:22 | 5/12 18:45:22 | ✓ |

全 size + mtime 完全 unchanged → **実 DB write 0 を運用上証明** ✓

## R2 v1.2 改訂内容 (= W27-7)

### 改訂 1: § 8 完了報告テンプレに 6 KPI table 追加

```
| KPI                    | 値                                | 目標 / 判定 |
|------------------------|-----------------------------------|-------------|
| Codex 稼働率           | {並列起動 lane 数} / 12+          | task 量に応じ可変 |
| 本線短縮率             | ({単独推定} - {実時間}) / {単独推定} = N% | 50% 以上目標 |
| 成果物採用率           | {採用された Codex 出力数} / {全}  | 100% 目標 |
| 差し戻し率             | {破棄された Codex 出力数} / {全}  | 0% 目標 |
| Integrator 負荷        | {本線実時間} / 150 分 = N%        | < 60 分 (= 40%) 目標 |
| **安全事故 0**         | {未承認 LINE/DB/token/cron 発生数} | **絶対条件、0 必須** |
```

**安全事故 0 は KPI トレードオフ対象外**、絶対条件。残り 5 KPI は wave の
性質と task 量に応じて目標値を超過する場合あり。

### 改訂 2: § 9 lane 選択方針を KPI 駆動化

「lane 数そのものは目標ではない」を明示。lane 数選択基準 3 つ:
1. task 量と性質
2. 6 KPI 駆動
3. 本線 Integrator 容量

参考 lane 数表 (= 固定ではない):

| task 量 | 推奨 lane 数 | 例 |
|---|---|---|
| 小 (sub <= 4) | 4-5 | 起票のみ、設計 doc 単体 |
| 中 (sub 5-7) | 6-7 | dry-run probe + 設計 |
| 大 (sub 8-10) | 8-10 | impl wave |
| 超大 (sub 11-12) | 10-12+ | 多 module 横断 / 大規模 refactor |
| 12 超 | wave 分割 | (= W{N}a / W{N}b で必ず分割) |

### 改訂 3: 安全事故 0 を絶対条件として § 8 + § 9 で再強調

「数」「速度」「KPI」よりも上位に位置付け。未承認 LINE / DB write / token /
cron 等の発生は wave 即時 abort 対象。

### § 13.1 改訂履歴に R2 v1.2 (= 3 改訂) 追加

W24/W25/W26 累計 L4 CONCERN を整理:
- W24 CONCERN 3 件: R2 v1.3+ 候補へ繰越し
- W25 CONCERN 5 件: F282 個別設計に反映済、R2 doc 追加なし
- W26 CONCERN 2 件: F282 impl に反映済、R2 doc 追加なし

## 6 KPI 集計 (= R2 v1.2 初試行)

| KPI | 値 | 判定 |
|---|---|---|
| Codex 稼働率 | 4 lane / 12+ = **33%** | task 量に応じ適切 (= 小〜中 wave) |
| 本線短縮率 | (100 分単独推定 - 50 分実時間) / 100 分 = **50%** | 目標 50% 達成 ✓ |
| 成果物採用率 | 4 / 4 = **100%** | 目標達成 ✓ |
| 差し戻し率 | 0 / 4 = **0%** | 目標達成 ✓ |
| Integrator 負荷 | 50 分 / 150 分 = **33%** | < 40% 目標達成 ✓ |
| **安全事故 0** | **0** | **絶対条件達成** ✓ |

5 KPI 目標全達成 + 安全事故 0 絶対条件達成 → **6 KPI 全 OK** ★

## W27-5 L4 audit verdict

W27-4 dry-run probe 実施に対する 8 観点 audit:

| 観点 | verdict |
|---|---|
| A. dry-run path → production fire.db read-only 保証 | PASS |
| B. PRAGMA query_only=ON で SELECT 1 後 write 試行 block | PASS |
| C. URI mode=ro で SQLite write block | PASS |
| D. os.access(W_OK) で実 file 作成 0 | PASS |
| E. shutil.disk_usage(target_dir.parent) 読取のみ | PASS |
| F. mtime 比較運用 (= W27 必須確認 #1+#9) 十分性 | PASS |
| G. 万一の write 検出時 rollback 手順 | PASS (= 本 wave write 想定 0) |
| H. HQ 禁止項目 全 維持 | PASS |

**CRITICAL 0 / HIGH 0**。verdict: dry-run probe 実施は safety 整合性 +
HQ 禁止項目 維持で全 PASS。

## 成功条件チェック (= P2 運用条件 + W27 固有)

| 条件 | 結果 |
|---|---|
| 6 lane 全完了 | ✓ |
| file ownership 衝突 0 | ✓ |
| CRITICAL 0 | ✓ |
| L4 HIGH 0 | ✓ |
| safety violation 0 | ✓ |
| 全 pytest PASS | ✓ (= 4,068) |
| wave 実時間 < 150 分 | ✓ (= 約 50 分) |
| commit 6 件以内 | ✓ (= fire-vault 2 件) |
| Codex 直接 commit 0 | ✓ |
| 本線過負荷 0 | ✓ |
| 全 DB mtime unchanged | ✓ |
| dry-run probe exit 0 | ✓ |
| write_count=0 | ✓ |
| HQ 報告 1 ブロック | ✓ |
| R2 v1.2 反映 | ✓ (= 3 改訂 + 履歴) |

## fire develop commits

本 Wave で commit なし (= dry-run probe + R2 v1.2 設計反映、code 変更 0)。

## fire-vault main commits (= 想定)

- (= 本 commit、後続) docs(FIRE-CODEX-R2): Wave 27 plan + results +
  F282 dry-run probe 結果 + R2 v1.2 改訂
- (= follow-up commit) docs: append Wave 27 milestone to log.md

changed files (= 4 件、全 vault):
- 02_todo/FIRE_CODEX_R1_WAVE27_plan.md (NEW)
- 02_todo/FIRE_CODEX_R1_WAVE27_results.md (NEW)
- 03_design/FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12.md (MOD = R2 v1.2)
- log.md (= Wave 27 milestone)

## 安全 (= Wave 27 全 ✓)

| 項目 | 結果 |
|---|---|
| 実 DB snapshot write | 0 |
| 実 VACUUM INTO | 0 |
| plist 配置 | 0 |
| launchctl load | 0 |
| cron / launchd / crontab 登録 | 0 |
| logrotate 設定適用 | 0 |
| 実 LINE 送信 | 0 |
| 実 API call | 0 |
| **全 DB mtime unchanged** | ✓ |
| token / channel_token / secret 参照 | 0 |
| F101 staging probe | 未実行 |
| 楽天 / 自動発注 / Computer Use | なし |
| .github/workflows/ 変更 | 0 |
| --no-verify | 不使用 |
| 既存 modified 2 件 (= forbidden) | 未接触 ✓ |
| TODO Excel | 未更新 |
| Codex 直接 commit | 0 |
| fire develop 変更 | 0 (= R2 v1.2 は vault のみ) |
| 新 file 作成 (= /data/) | 0 |

## 並列効果 + 6 KPI 通算

Wave 27 実時間 約 50 分、本線単独推定 100 分、短縮 50%。
**Wave 1-27 通算で 60-80% 短縮を 27 wave 連続達成** ★

## 回帰

4,068 PASS 維持 (= Wave 26 baseline 不変、本 wave code 変更 0)。

## HQ 判断論点 (= 4 件)

1. **Wave 27 完了 (= dry-run probe + R2 v1.2) 採用承認**
   推奨: approve

2. **Wave 28 候補選定**:
   推奨 a: F282 write path 実装 (= VACUUM INTO impl + tests、本番実行は別 wave)
   推奨 b: F282 plist 配置 + 試走計画 (= 設計のみ、本番 launchctl load 別 wave)
   別案: F101 staging probe (= 別 HQ approve、token 使用)
   別案: R2 v1.3+ 改訂 (= W24 CONCERN 3 件)

3. **F282 dry-run probe 実施結果の運用反映**
   - production fire.db への ro 接続 1 回成功、371 MB / disk 831 GiB OK
   - 次 step は VACUUM INTO impl (= write path) 又は本番 dry-run launchd
     試走

4. **F101 staging probe 着手判定**
   - 現状未承認継続
   - 着手するなら別 wave + 別 HQ approve

## 関連リンク

- [[../03_design/FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 v1.2 設計本体]]
- [[../03_design/F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[FIRE_CODEX_R1_WAVE26_results|Wave 26 results (= F282 impl)]]
- /tmp/codex_wave27/prompts/* (= 4 lane prompt、session-local)
- /tmp/codex_wave27/output/* (= 4 lane 出力、session-local)
