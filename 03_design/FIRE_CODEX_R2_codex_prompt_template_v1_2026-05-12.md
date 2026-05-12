---
id: FIRE-CODEX-R2-codex-prompt-template-v1
phase: ガバナンス / R-01-08 / FIRE-CODEX-R2 / Codex prompt template
priority: 高
status: v1.0 確定 (= Wave 21 W21-1)、Phase P1 着手 wave で適用開始
owner: BlueFire7777 (Fujiwara)
date: 2026-05-12
depends_on:
  - FIRE-CODEX-R2 10-Lane Scaling Design (= Wave 20)
  - HQ Wave 21 prompt template 更新指示 (= 2026-05-12)
chapter: ガバナンス / R-01-08
---

# FIRE-CODEX-R2 / Codex prompt template v1.0

Phase P1 (= 6 lane) 以降で **必須適用** する Codex prompt 標準形式。
R1 v1.1 の暗黙運用を明示化し、owner_lane_id を **必須項目化**。

## 必須項目 (= HQ Wave 21 明示)

| 項目 | 内容 |
|---|---|
| `owner_lane_id` | 本 sub-task の owner lane (= L1a / L1b / L2 / L3 / L4 / L5 / L6) |
| `allowed_files` | 修正・新規可な file の絶対 / 相対 path リスト |
| `forbidden_files` | 触れてはいけない file の path リスト |
| `expected_outputs` | sub-task 完了時の成果物 (= file 名 / test 数 / vault doc 等) |
| `test_command` | pytest 等の実行 command (= 実行 lane が L2 / L6 のとき必須) |
| `merge_owner` | 本線 (= PM / Integrator) が確定的に merge する旨明示 |
| `abort_conditions` | hard abort / soft rollback 条件 (= 本 prompt 固有部分) |

## 標準フォーマット

```
あなたは FIRE Codex-{lane_role} レーン担当 (= {owner_lane_id})。

task_id:         {task_id}
owner_lane_id:   {owner_lane_id}
parent_wave:     {wave_id}
merge_owner:     Claude Code 本線 (= PM/Integrator)、Codex 直接 commit 禁止

================================================================
目的
================================================================
{purpose 1-3 行}

================================================================
HQ 必須項目
================================================================
{HQ 明示制約 列挙}

================================================================
具体仕様
================================================================
{設計 / impl 仕様}

================================================================
allowed_files
================================================================
{path list}

================================================================
forbidden_files
================================================================
{path list、共通 7 件 + sub-task 固有}

共通 forbidden 7 件 (R1 v1.1 継承):
- scripts/seed_pattern_layer1.py
- simulation/research_lane/historical_indicators.py
- notifications/* (= LINE 送信経路、別承認時のみ)
- scripts/setup/migrate_*.py (= production schema、別承認時のみ)
- .github/workflows/*
- data/fire.db / data/fire.develop.db (= production / develop)
- TODO Excel

================================================================
expected_outputs
================================================================
{成果物列挙、test 数、行数目安、doc path}

================================================================
test_command
================================================================
{pytest コマンド}

例: `.venv/bin/pytest tests/{module}/ -v --tb=short`

================================================================
abort_conditions
================================================================
- audit CRITICAL 1 件以上 → wave 中断
- safety violation (= production DB write / LINE 実送信 / token leak)
  → 即時 wave 中断 + incident doc
- file ownership 衝突 → 本線が再分配
- HQ 明示 abort

================================================================
== 必ず守ること ===
================================================================
- 実 DB write なし (本 sub-task でなければ)
- 実 LINE 送信 / subprocess / token 参照 0
- 既存 contract 維持
- allowed_files 以外触らない
- git add / git commit を Codex 自身で実行しない (= merge_owner = 本線)

完了したら stdout に「=== Codex 完了報告 (= {owner_lane_id}) ===」、
CRITICAL/HIGH 0 明示。
```

## owner_lane_id 表

| ID | 役割 | 主担当 | 代表 task |
|---|---|---|---|
| L1a | Design (主) | 設計初稿 | 設計 doc / API 仕様 |
| L1b | Design (副) | 代替設計 / 補助 | 比較案 / fallback 設計 |
| L2 | Test | test 実装 | unit / integration / mock |
| L3 | Implementation | 主実装 | source 修正 |
| L4 | Audit | adversarial レビュー | CRITICAL / HIGH 検出 |
| L5 | Docs | vault doc / log.md draft | plan / results / design |
| L6 | Regression | 既存 PASS 監視 | pytest 全 / contract 確認 |

L3 並列展開時 (= P2 以降):
- L3a / L3b / L3c で domain 別に file 分配
- 各 lane の allowed_files は **disjoint** (= 重複なし)

## allowed_files / forbidden_files 設計原則

### allowed_files

- **絶対 path** または **repo root 相対 path** で記述
- ワイルドカードは慎重 (= 過剰許可 risk、できれば exact path 列挙)
- 新規 file の場合は **作成予定 path を明示**
- 1 sub-task で 5 file 以上 → wave 分割検討

### forbidden_files

- 共通 7 件 (= R1 継承) は必ず含める
- sub-task 固有の forbidden を追加 (= 他 lane の所有 file)
- pattern: `path/to/forbidden/*` は許容 (= 配下全)

## merge_owner ルール

- 本線 (= Claude Code PM/Integrator) が **唯一の commit owner**
- Codex は patch / diff を stdout / file に出力
- Codex 自身が `git add` / `git commit` / `git push` を実行することは禁止
- merge 時のレビュー責任は本線
- audit (= L4) が CRITICAL を検出した場合、本線が merge せず差し戻し

## abort_conditions テンプレ (= sub-task 固有部分用)

```
# Hard abort (= 即時 wave 中断)
- audit CRITICAL 1 件以上
- safety violation
- 未承認 LINE 実送信
- token / secret leak
- file ownership 衝突 ≥ 2
- HQ 明示 abort

# Soft rollback (= wave 継続可、-fix sub-task で対応)
- audit HIGH 1 件
- 回帰 PASS -1 以上
- lane 出力 90 分以上未着
```

## R2 prompt template 利用例 (= Wave 21 W21-4 L3)

```
あなたは FIRE Codex-Implementation レーン担当 (= L3)。

task_id:         F101-endpoint-resolution-impl (= W21-4)
owner_lane_id:   L3
parent_wave:     Wave 21 (= Phase P1 6 lane 試験投入)
merge_owner:     Claude Code 本線 (= PM/Integrator)

目的:
materials/client.py の `/fins/announcement` hardcode を切替可能化。
endpoint constant をモジュール定数化、env / arg で上書き可。実 API
call は本 sub-task では発生させない。

HQ 必須項目:
- 実 API call 0
- token 値読出禁止
- 既存 API シグネチャ維持 (= fetch_announcements / fetch_all_announcements)
- mock test で endpoint 切替検証

allowed_files:
- materials/client.py

forbidden_files (= 共通 7 件 + sub-task 固有):
- (共通 7 件 ...)
- tests/materials/test_client*.py (= L2 担当)
- materials/fetcher.py (= 別 sub-task 担当時のみ)

expected_outputs:
- materials/client.py 修正 (= 行数目安 +30〜+60)
- 新規 ANNOUNCEMENT_ENDPOINT_DEFAULT / _CANDIDATES 定数
- _resolve_announcement_endpoint() helper

test_command:
- `.venv/bin/pytest tests/materials/test_client.py -v --tb=short`

abort_conditions:
- audit CRITICAL 1 件以上 → wave 中断
- 既存 API breaking change → 修正後再 commit
- 実 API call 痕跡 (= requests.get 直接呼出 等) → 即修正
```

## 関連リンク

- [[FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 設計本体]]
- [[FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|R1 v1.1 既存運用]]
- [[../02_todo/FIRE_CODEX_R1_WAVE21_plan|Wave 21 plan]]
