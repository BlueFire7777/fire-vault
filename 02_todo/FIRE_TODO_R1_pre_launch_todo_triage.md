---
id: FIRE-TODO-R1
phase: P9:Stage3 移行 / Pre-Launch
priority: 最優先
status: triage 完了 (2026-05-10、Excel 未更新、提案のみ)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R3 (LINE production send path 構造的安全性 PASS)
  - F286-DATA-R2 (freshness gate 全 5 段 PASS)
chapter: 第 26 章 / 第 19 章 R-19-08 / 第 13 章
---

# FIRE-TODO-R1: Pre-Launch TODO Triage

最終更新: 2026-05-10

## 目的

本番 LINE Advisory の **初回実送信前** に残っている TODO を棚卸しし、
6 カテゴリに分類する。Excel は更新せず、提案案のみ作成する。

## 結論サマリ

- **pre_launch_required: 0 件** ★
- post_launch_high_priority: 9 件
- shadow_or_research: 6 件
- hold: 6 件
- integrated_or_done: 87 件
- delete_candidate: 1 件

つまり「**実弾投入前に追加実装すべきタスクは 0 件**」。Fujiwara が
HQ 承認の上で `--hq-approved-send` + 本物 token + recipient_id +
`--max-chunks 1 --test-message-only` を手動実行すれば、初回本番 LINE
送信は **今すぐ** 実施可能な状態。

実弾投入完了率: 約 98 %。

残り 2 % の内訳:

1. 本タスク (FIRE-TODO-R1 Pre-Launch TODO Triage) ← 本書で実施
2. First Real LINE Send Smoke (= HQ 承認 + 1 通の手動発火)
3. First Production Advisory Small Launch (= 本番運用開始)

## 調査対象

- `~/fire-vault/02_todo/*.md` (110 ファイル)
- `~/fire-vault/log.md` (5,889 行、直近 milestone 50 件)
- `~/fire/` git log develop 直近 25 commit
- `~/fire/CLAUDE.md` 完了テーブル + 「次のタスク候補」section
- 各 task md の frontmatter (status / phase / priority)

> ⚠ TODO Excel (`02_todo/FIRE_開発TODO_v9.xlsx`) は **本タスクで更新しない**。
> 提案案は本書末尾の「Excel 更新案」section にのみ記載。

## 1. pre_launch_required (= 初回本番 LINE 送信前に必須)

| task_id | reason | action |
|---|---|---|
| (該当なし) | F062-R3 で構造的安全性 PASS、DATA-R2 freshness gate PASS、F062-R1 payload + DATA-R2 pass gate + production_send_callable + HQ 承認 flag のすべてが既に実装済み。残るは Fujiwara の手動発火のみ | First Real LINE Send Smoke を実施 |

★ 「実 LINE API 1 chunk / test message」を Fujiwara が手動で 1 回送るだけで
本番運用開始可能。追加コード変更なし。

実施コマンド (HQ 承認後):

```
LINE_CHANNEL_TOKEN=<本物トークン> .venv/bin/python -m \
  scripts.jobs.run_f062_line_production_send_smoke \
  --payload-json /tmp/f062_r1_line_preview_payload.json \
  --gate-json    /tmp/f286_data_r1_3_gate_after.json \
  --send --hq-approved-send \
  --recipient-id <Fujiwara LINE userId> \
  --max-chunks 1 --test-message-only \
  --output-json       /tmp/f062_r3_first_real_send.json \
  --completion-report /tmp/f062_r3_first_real_send_report.txt
```

## 2. post_launch_high_priority (= 本番運用開始後すぐ対応)

| task_id | current_status | reason | action |
|---|---|---|---|
| F242 | OpenClaw セットアップ 未着手 | 24h 稼働 / 自動再起動 / 並列実行の基盤、本番開始後 1-2 週内に立てたい | OpenClaw 初期セットアップ |
| F022 | FIRE Runner 骨組み 未着手 | 朝寄り前 fetch → DATA-R2 → F062-R1 → F062-R3 send の自動連鎖 runner、現状は手動 | FIRE_Runner エントリーポイント実装 |
| F013 | launchd 常駐 未着手 | Mac mini 起動時に FIRE Runner を自動起動、再起動耐性確保 | jp.fire.runner.plist 作成 |
| F267 | features_pipeline 実装中 | features 集計の自動化、F021 完了後の運用フェーズで必須 | Phase 1 完了 |
| F235 | 楽天証券約定通知メール 未着手 | M+2 以降の自動取込、初月は LINE 手動報告で代替可能 | Gmail filter + IMAP fetch |
| F276 | events 達成 positions seeding 未着手 | events テーブルに対する positions の seeding ジョブ、本番運用安定後に追加 | 設計再評価 |
| F277 | paper_live 例外伝播 着手中 (Phase 0) | paper live runner の例外伝播 / マルチプロセス cache の安定化、本番後にテストを増やしたい | Phase 1 着手 |
| F278 | git ガバナンス整備 未着手 | pre-commit / branch protection / Codex 連携の継続改善、本番運用と並走 | hook 棚卸し |
| F286-DATA-R3 | (新規提案) cron / launchd で daily refresh 自動化 | 現状は手動で `fetch_historical_market_data.py` を回す。本番運用後に cron 化 | DATA-R1 + DATA-R2 を 18:00 JST に自動実行 |

## 3. shadow_or_research (= 本番に入れず裏で検証)

| task_id | current_status | reason | action |
|---|---|---|---|
| F285 | Research Lane 中長期銘柄発掘 仕様 v1.1 完了、PhaseR0 待機 | 中長期 lane は本番フローに直接乗らない。shadow で検証して有効性確認後に統合 | PhaseR0 feasibility |
| F286_R2_G | regime/sector integration | R2-F4 で baseline 最強と判明、追加 rule は shadow 検証 | shadow signal 蓄積 |
| F286_R2_G2 | rule_refinement | R2-G の発展、shadow 専用 | shadow 比較 |
| F286_R2_G3 | rule_finalization | shadow 比較完了後の rule 確定 | hold until R2-G2 |
| F286_R1_Sector_Flow_and_financials_backfill | HQ Q1-Q5 判断待ち | Sector Flow Agent MVP、Lane 拡張の前提 | HQ 判断 |
| F105 | J-Quants 個別銘柄分足 PhaseC1 完了 / C2 着手可能 | Lane C 真実装の前提、shadow で 1m bar の有効性検証 | PhaseC2 着手判断 |

## 4. hold (= 今やらない、将来候補)

| task_id | current_status | reason | action |
|---|---|---|---|
| F210_phase_1b | 凍結 (2026-05-04) | Dashboard 着手 / F118 設計時に再評価 | 再評価 |
| F279 | 休日専業モード 設計 草案あり (低 priority) | M+1 以降の運用形態、当面は不要 | 草案保持 |
| F287 | 決算カレンダー / AI 分析 / スライド / LINE 通知ダッシュボード 仕様 v1.1 完了 | F285 下流、本番安定後に着手 | hold until 本番安定 |
| F268 | announcements_backfill 未着手 | TDnet 過去 announcement の backfill、F101 既存運用で十分 | 必要性再評価 |
| F269 | chain_assertions 未着手 | パイプライン chain の整合性検査、本番後の保守タスク | hold |
| F281 | strategy_portfolio_migration Phase2-B-mini 完了 (LaneA1 broad 不採用) | Stage3 候補から除外、HQ 判断待ち | HQ 判断 |

## 5. integrated_or_done (= 統合済み or 完了済み)

直近 50 件の milestone と各 task md frontmatter から確定した完了済み
タスク 87 件。一部のみ列挙、詳細は CLAUDE.md 完了テーブル + log.md
milestone を参照。

### P0: 基盤
F001 / F002 / F003 / F004 / F005 / F008 / F009 / F010 / F012

### P1: Pattern / Feature Store
F020 / F021 / F023 / F024 / F025 / F026 / F027 / F028 / F029 / F030 /
F031 / F032 / F033 / F034 / F035 / F036 / F037 / F038 / F039 / F056 /
F180 / F181 / F182 / F183 / F200 / F201 / F202 / F243

### P2: Simulation
F040 / F041 / F042 / F043 / F044 / F045 / F046 / F047

### P3: Paper Live
F050 (本体) / F051 / F052 / F053 / F054 / F055 / F057 / F058 / F116 /
F119-Phase1 / F119-Phase2 / F119-Phase3

### P4: 通知 / Trade Decision
F111 / F115 / F130 / F131 / F132 / F133 / F140 / F210 (Phase1A 完了 +
Phase1B 凍結で F210 自体は完了) / F062 / F236

### P6: データソース
F100 / F100-historical / F101 (Phase1+2+3)

### P9: Stage 3 移行 / 運用
F230 / F233 / F241 / F260 / F271 / F275

### F286 系 Research Lane R1/R2 (完了済 phase)
F286_R1_B2_5 / F286_R1_B4 / F286_R2_A1 / F286_R2_A2 / F286_R2_A3 /
F286_R2_B / F286_R2_C / F286_R2_D / F286_R2_D2 / F286_R2_E /
F286_R2_F / F286_R2_F2 / F286_R2_F3 / F286_R2_F4 / F286_R2_H /
F286_Research_Lane_R0_feasibility (R0 precheck 完了)

### 新規 R-prefix 系 (frontmatter 無、最近完了)
F062-R1 / F062-R2 / F062-R3 / F111-R1 / F111-R2 / F111-R3 / F111-R4 /
F119-interpretation / F286-DATA-R0 / F286-DATA-R1 / F286-DATA-R1.1 /
F286-DATA-R1.2 / F286-DATA-R1.3 / F286-DATA-R2

### 統合済み (= 別タスクに吸収)
| task_id | 吸収先 |
|---|---|
| F011 (ローカル DB 選定) | SQLite / staging.db を既に使用、F008 + F020 で統合済み |
| F062 (元のテンプレート) | F062-R1/R2/R3 で拡張完了、原稿は維持 |
| F111 (元の Daytrade Selection) | F111-R1〜R4 で R+S 統合完了、原稿は維持 |
| F119 (Evaluation) | Phase1/2/3 完了 |

## 6. delete_candidate (= 不要化したもの)

| task_id | 理由 | action 提案 |
|---|---|---|
| F050_Paper_Live_Stage_2.md | F050_Paper_Live_Mode_Stage_2_本体.md と重複 (空 placeholder)、本体側が status:完了 | 削除 (vault 上の重複ファイル) |

> 注: F001 (Windows 開発環境) は「役目終了」だが歴史記録としての価値が
> あるため delete_candidate ではなく integrated_or_done で扱う。

## 重要観点別の要約

### LINE 本番送信前に本当に残っている必須項目

**0 件**。F062-R3 で:
- LineBotClient 隔離 module
- HQ 承認 flag 必須 (`--hq-approved-send`)
- recipient broadcast 防御 (先頭 U/C/R)
- max_chunks 1 default
- preflight + partial_delivery handling
- DATA-R2 freshness gate 連携

すべて構造的に整っている。**追加コード変更なし**。

### DATA freshness / daily refresh 系

| 完了 | 内容 |
|---|---|
| DATA-R0 | freshness audit 設計 |
| DATA-R1 | daily refresh pipeline |
| DATA-R1.1 | limited write smoke (rate limit safe) |
| DATA-R1.2 | refresh coverage expansion + derived/signals refresh |
| DATA-R1.3 | remaining symbols price refresh + gate pass smoke |
| DATA-R2 | freshness gate / stale data warning (5 段判定 + line_send_allowed bool) |

残: **DATA-R3 (cron 自動化)** = post_launch_high_priority。手動運用で
本番開始は問題なし。

### F111 / F062 / F119 で統合済みの重複タスク

| 元タスク | 統合先 | 状態 |
|---|---|---|
| F062 (LINE 注文完成形テンプレート) | F062-R1 (preview) → F062-R2 (separation) → F062-R3 (production path) | 完全統合 |
| F111 (Daytrade Selection Agent) | F111-R1 (research advisory signal) → R2 (preview) → R3 (real wiring) → R4 (multi-date + F119 insights) | 完全統合 |
| F119 (Evaluation Agent) | Phase1 (集計) + Phase2 (判定 + Markdown) + Phase3 (承認 DB + LINE 通知) + interpretation_evaluation | 完全閉ループ |
| F011 (ローカル DB 選定) | F008 + F020 で SQLite/WAL 採用済み | 統合済 |

### Lane C / R2-G4 / F111-R5 など本番後でよい研究タスク

- **Lane C** (= 1m bar / J-Quants Add-ons): F105 PhaseC2 着手可能、shadow
- **R2-G/G2/G3** (= regime/sector integration → rule refinement → finalization): R2-F4 で baseline 最強判明、shadow 検証
- **F111-R5 候補**: 該当なし (= R-1〜R-4 で完結)
- **F285** (Research Lane 中長期): shadow_or_research

### 自動発注・楽天操作・Computer Use につながりそうな危険タスク

**該当なし**。R-01-08 / 第 04 章で Computer Use / Playwright / Selenium /
自動発注の不採用が明文化されており、02_todo 配下にこれらに繋がる task は
存在しない。

唯一注意すべきは:
- F235 楽天証券約定通知メール: メール **受信** 専用、自動発注ではない (= 安全)
- 将来の Stage 4 Full Auto: 02_todo に該当なし、要件書側で「将来オプション」扱い

### 本番後の Lane Expansion Agent 候補

- F285 Research Lane 中長期銘柄発掘 (shadow_or_research)
- F287 決算カレンダー / AI 分析 / LINE 通知ダッシュボード (hold)
- F286_R2_G/G2/G3 regime/sector integration (shadow_or_research)
- F286_R1_Sector_Flow Agent MVP (shadow_or_research、HQ 判断待ち)

### 削除でなく「統合済み」扱いにすべきもの

| task_id | 一見不要 | 実は |
|---|---|---|
| F011 ローカル DB 選定 | 「未着手」 | SQLite/staging.db で既に統合済 |
| F062 LINE 注文完成形 | 「完了」だが原稿あり | F062-R1/R2/R3 で拡張、原稿は履歴として保持 |
| F111 Daytrade Selection | 同上 | F111-R1〜R4 で拡張、原稿は履歴 |
| F119 Evaluation | 同上 | Phase1/2/3 + interpretation で完全閉ループ、原稿保持 |
| F210_phase_1b | 「凍結」 | F210 本体 (Phase1A) は完了、Phase1B は将来候補 = hold |

## Excel 更新案 (本タスクで適用しない)

下記は提案案であり、Fujiwara が判断のうえで TODO Excel に手動反映する
こと。

| task_id | current_status | proposed_status | proposed_category | reason | action |
|---|---|---|---|---|---|
| F062 | 完了 | 完了 (F062-R1/R2/R3 で拡張) | integrated_or_done | R-prefix 系で延伸 | コメント欄に統合履歴追記 |
| F111 | 完了 | 完了 (F111-R1〜R4 で拡張) | integrated_or_done | 同上 | 同上 |
| F119 | 完了 (Phase1/2/3) | 完了 (interpretation 含む) | integrated_or_done | 第 13 章 R-13-02〜10 完全実装 | 同上 |
| F062-R1 | (Excel 未登録) | 完了 | integrated_or_done | 2026-05-10 完了 | 行追加 |
| F062-R2 | (Excel 未登録) | 完了 | integrated_or_done | 2026-05-10 完了 | 行追加 |
| F062-R3 | (Excel 未登録) | 完了 | integrated_or_done | 2026-05-10 完了 | 行追加 |
| F111-R1〜R4 | (Excel 未登録) | 完了 | integrated_or_done | 2026-05-10 完了 | 行追加 (4 行) |
| F119-interpretation | (Excel 未登録) | 完了 | integrated_or_done | 2026-05-10 完了 | 行追加 |
| F286-DATA-R0〜R2 | (Excel 未登録) | 完了 | integrated_or_done | 2026-05-10 完了 | 行追加 (6 行) |
| F286-DATA-R3 | (Excel 未登録、新規提案) | 未着手 | post_launch_high_priority | cron 自動化、本番後 | 行追加 |
| F011 | 未着手 | 完了 (統合済) | integrated_or_done | F008+F020 に吸収 | status 更新 |
| F022 | 未着手 | 未着手 | post_launch_high_priority | OpenClaw + FIRE_Runner 自動連鎖 | priority 維持 |
| F242 | 未着手 | 未着手 | post_launch_high_priority | 24h 稼働基盤 | priority 維持 |
| F013 | 未着手 | 未着手 | post_launch_high_priority | launchd 常駐 | priority 維持 |
| F267 | 実装中 | 実装中 | post_launch_high_priority | features pipeline | Phase 進捗共有 |
| F235 | 未着手 | 未着手 | post_launch_high_priority | 楽天約定メール、M+2 以降 | hold until M+2 |
| F276 | 未着手 | 未着手 | post_launch_high_priority | events seeding | 本番後設計 |
| F277 | 着手中 (Phase0) | 着手中 | post_launch_high_priority | 例外伝播 | Phase1 着手 |
| F278 | 未着手 | 未着手 | post_launch_high_priority | git ガバナンス | 並行作業 |
| F285 | PhaseR0 待機 | 着手判断待ち | shadow_or_research | 中長期 lane shadow | R0 feasibility |
| F286_R1_Sector_Flow | HQ 判断待ち | hold | shadow_or_research | Sector Flow MVP | HQ 判断 |
| F286_R2_G/G2/G3 | (一部完了) | shadow_or_research | shadow_or_research | regime/sector | shadow 検証 |
| F104 | 未着手 | shadow_or_research | shadow_or_research | 指数四本値、regime 補強 | F110 と並行 |
| F105 | PhaseC1 完了 | shadow_or_research | shadow_or_research | 1m bar Lane C | PhaseC2 判断 |
| F243 | 完了 | 統合済 | integrated_or_done | Sector/Symbol 特化、shadow 運用 | コメント欄追記 |
| F210_phase_1b | 凍結 | hold | hold | Dashboard 着手時に再評価 | 再評価日設定 |
| F279 | 未着手 (草案あり) | hold | hold | 休日専業、M+1 以降 | 草案保持 |
| F287 | 仕様 v1.1 完了 | hold | hold | F285 下流、本番安定後 | hold until 安定 |
| F268 | 未着手 | hold | hold | announcements backfill | 必要性再評価 |
| F269 | 未着手 | hold | hold | chain assertions | hold |
| F281 | Phase2-B-mini 完了 | HQ 判断待ち | hold | LaneA1 broad 不採用 | HQ 判断 |
| F050_Paper_Live_Stage_2.md (重複) | (vault 上の重複 md) | 削除 | delete_candidate | 本体側が完了済 | vault doc 削除 |
| F006 | 未着手 | 未着手 | post_launch_high_priority | README 初稿 | 本番安定後 |

## 安全要件遵守

- TODO Excel **未更新** (= 提案案のみ)
- DB write 0 (= 本タスクは vault doc 作成のみ)
- LINE 送信 0
- 自動発注 / 楽天操作 / Computer Use 0
- production / develop / staging.db 全 unchanged
- scripts/seed_pattern_layer1.py 未接触
- simulation/research_lane/historical_indicators.py 未接触
- unrelated modified file を stage / commit しない
- --no-verify 不使用
- Codex pre-commit (docs commit のため Codex review 対象外)

## 成果物

1. 本書 (`02_todo/FIRE_TODO_R1_pre_launch_todo_triage.md`)
2. log.md milestone エントリ (`docs(FIRE-TODO-R1): log milestone`)
3. 完了報告 (`/tmp/fire_todo_r1_completion_report.txt`)

## 次タスク

1. **First Real LINE Send Smoke** (HQ 承認 + 1 通の手動発火)
2. **First Production Advisory Small Launch** (= 本番運用開始)
