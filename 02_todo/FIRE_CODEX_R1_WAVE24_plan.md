---
id: FIRE-CODEX-R1-WAVE24-plan
phase: ガバナンス / Wave 24 / Phase P2 初試験 (= 8 lane) / R2 v1.1 改訂
priority: 高
status: 進行中 ☆ 8 lane (= 本線 L5 + Codex 7)、R2 v1.1 改訂、sandbox=read-only
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 23 (= 完了、6 lane 実起動初実証)
  - HQ Wave 24 起票承認 + Phase P2 条件付き承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / R2 v1.1 / Phase P2 = 8 lane
---

# Wave 24: R2 v1.1 改訂 + Phase P2 = 8 lane initial probe

Wave 23 で 6 lane 実起動初実証完了 → 3 wave 連続 PASS → Phase P2 移行
条件付き承認。本 wave で R2 v1.1 必須改訂 3 項目を反映 + 8 lane initial
probe を実行。

## Wave 24 sub-task (= 8 lane 構成、本線 L5 + Codex 7)

| sub  | lane | owner | task                                       | allowed_files |
|------|------|-------|--------------------------------------------|---------------|
| W24-1| L5   | 本線  | Wave 24 plan + 7 prompt + R2 v1.1 改訂草案 | vault, /tmp/codex_wave24/prompts/ |
| W24-2| L1a  | Codex | Phase exit 条件難度補正 設計               | /tmp 配下 only |
| W24-3| L1b  | Codex | file lock / 既存 modified 検知 設計        | /tmp 配下 only |
| W24-4| L2a  | Codex | dummy test 1 (= 純関数 3 件)               | /tmp 配下 only |
| W24-5| L2b  | Codex | dummy test 2 (= 別 file 想定の純関数)      | /tmp 配下 only |
| W24-6| L3   | Codex | R2 v1.1 doc 更新案 (= unified diff 風)     | /tmp 配下 only |
| W24-7| L4   | Codex | adversarial audit (= R2 v1.1 改訂後の整合) | /tmp 配下 only |
| W24-8| L6   | Codex | regression plan + 本線 pytest 実行         | /tmp 配下 only |
| W24-9| 本線  | 本線  | R2 v1.1 doc 確定 + commit + 報告           | vault, log.md |

8 lane 構成: L5 (= 本線) + L1a/L1b/L2a/L2b/L3/L4/L6 (= Codex 7) = 8 lane。

## file lock 表 (= R2 v1.1 改訂 1 = 既存 modified 検知の実演)

### 既存 modified (= 触らない、本 wave 範囲外)

| file | 状態 | 扱い |
|---|---|---|
| /Users/bluefire/fire/scripts/seed_pattern_layer1.py | M (= Wave 21 以前から) | **forbidden、触らない** |
| /Users/bluefire/fire/simulation/research_lane/historical_indicators.py | M (= Wave 21 以前から) | **forbidden、触らない** |
| /Users/bluefire/fire/.claude/ | ?? (= untracked) | 本 wave 範囲外、触らない |
| /Users/bluefire/fire/data/fire.staging.db.pre_restore_20260511_112053 | ?? (= staging backup) | 本 wave 範囲外、触らない |

### 本 wave で本線が書き込む file (= 1 wave 1 file 1 owner)

| file | owner_lane | merge_owner | 状態 |
|---|---|---|---|
| 02_todo/FIRE_CODEX_R1_WAVE24_plan.md | 本線 (L5) | 本線 | NEW |
| 02_todo/FIRE_CODEX_R1_WAVE24_results.md | 本線 | 本線 | NEW |
| 03_design/FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12.md | 本線 | 本線 | MOD (= R2 v1.1) |
| log.md | 本線 | 本線 | MOD |

### Codex 7 lane の output (= /tmp 配下、vault に commit せず参照のみ)

| file | owner_lane |
|---|---|
| /tmp/codex_wave24/output/l1a_*.{stdout,last,stderr} | L1a |
| /tmp/codex_wave24/output/l1b_*.{stdout,last,stderr} | L1b |
| /tmp/codex_wave24/output/l2a_*.{stdout,last,stderr} | L2a |
| /tmp/codex_wave24/output/l2b_*.{stdout,last,stderr} | L2b |
| /tmp/codex_wave24/output/l3_*.{stdout,last,stderr} | L3 |
| /tmp/codex_wave24/output/l4_*.{stdout,last,stderr} | L4 |
| /tmp/codex_wave24/output/l6_*.{stdout,last,stderr} | L6 |

衝突 0 (= path disjoint)。

## R2 v1.1 必須改訂 3 項目 (= HQ Wave 24 指示)

### 改訂 1: Phase exit 条件の難度補正

R2 v1.0:
- 「2 wave 連続 PASS」のみ

R2 v1.1:
- 「2 wave 連続 PASS」
- **+ 実装あり wave を最低 1 回含める**
- **+ file ownership 衝突 0**
- **+ 本線過負荷 0**
- **+ L4 HIGH 0**

### 改訂 2: file lock / 既存 modified 検知

R2 v1.0:
- file ownership 表 (= owner_lane / merge_owner)
- 暗黙: 既存 modified は file lock 表で扱わない

R2 v1.1:
- **wave 開始時に `git status -s` を必須確認**
- **既存 modified file を file lock 表に明記**
- **触ってはいけない既存 modified を「forbidden」として明確化**
- file lock 表に「状態」列 (= NEW / MOD / forbidden / untracked) を追加

### 改訂 3: P3 初回対象から LINE/token integration 除外

R2 v1.0:
- P3 (= 10 lane) 初回候補例: REPORT-R1 LINE 実送信 token integration

R2 v1.1:
- **P3 初回は production 非接触の横断実装に限定**
- LINE / token / production DB / cron は単線維持 (= R2 v1.0 と同じ、改めて強調)
- P3 初回候補例:
  - test cleanup / 統一
  - docs 統一 / cross-link 整理
  - import / type hint 統一
  - 共通 helper の重複削減 (= 同 file 編集は L3a/b/c 担当分割)

## 成功条件 (= HQ Wave 24 指示)

| 条件 | 結果 |
|---|---|
| 8 lane 全完了 | (= 本 wave 後確認) |
| file ownership 衝突 0 | (= file lock 表で保証) |
| CRITICAL 0 | (= L4 audit verdict) |
| **L4 HIGH 0** | (= R2 v1.1 改訂による Wave 24 では確実に HIGH 0 を狙う) |
| safety violation 0 | (= 監視) |
| 全 pytest PASS or 対象 PASS + regression plan | (= 4,041 維持想定) |
| wave 実時間 < 150 分 | (= 8 lane で 40-60 分想定) |
| commit 6 件以内 | (= fire develop 0 + fire-vault 2-3 想定) |
| Codex 直接 commit 0 | (= sandbox=read-only) |
| HQ 報告 1 ブロック | (= 本 wave 完了時) |

## Hard abort 条件 (= R2 v1.1 継承)

- audit CRITICAL 1 件以上
- safety violation
- 未承認 LINE 送信
- token / secret leak
- file ownership 衝突 ≥ 2
- HQ 明示 abort

## Soft rollback (= R2 v1.1 改訂後)

- audit HIGH 1 件 → 同 wave 内 -fix sub-task で対応
- 回帰 PASS -1 以上 → 該当 test 修復
- lane 出力 90 分以上未着 → prompt 修正 or 別 lane 振替

## 禁止 (= HQ Wave 24 指示)

- DB write 全
- LINE 送信
- token/secret/channel_token 参照
- 実 API call
- cron / launchd / crontab 登録
- production/develop apply
- workflow 変更
- --no-verify
- TODO Excel 更新

## 関連リンク

- [[../03_design/FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 設計本体]]
- [[../03_design/FIRE_CODEX_R2_codex_prompt_template_v1_2026-05-12|R2 prompt template v1.0]]
- [[FIRE_CODEX_R1_WAVE23_results|Wave 23 results]]
