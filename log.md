# FIRE 開発ログ

時系列の追記専用ファイル。設計判断・章追加・障害・セットアップ変更などを記録する。

## フォーマット

## [YYYY-MM-DD] type | summary
- 詳細1
- 詳細2

## type 一覧
- setup: 環境構築・ツール導入
- decision: 設計判断・方針確定
- ingest: 章追加・要件書更新
- incident: 障害・想定外事象
- lint: 健康チェック実行
- milestone: フェーズ移行・タスク完了

## 検索コマンド
- 直近5件: grep "^## \[" log.md | tail -5
- 設計判断のみ: grep "decision" log.md
- 月別: grep "^## \[2026-04" log.md

---

## [2026-04-30] setup | obsidian-skills 導入
- kepano/obsidian-skills を .claude/ に git clone
- 5 skills(markdown / bases / json-canvas / cli / defuddle)を有効化
- CLAUDE.md 末尾に Obsidian Skills セクション追記
- /init で CLAUDE.md ベース作成

## [2026-04-30] setup | log.md 運用開始
- 時系列の追記運用を導入
- CLAUDE.md に log.md ルールを追記

## [2026-05-03] decision | v3.4 空売り対応パッチ適用
- 12 章追記 + 第42章新設 (空売り対応総論)
- 第02/03/05/06/08/10/11/17/21/26/37/40 章 version v3.4 / updated 2026-05-03 に更新
- 番号衝突回避: R-06-21〜25 (元 14-18) / R-21-21〜22 (元 08-09) / R-26-11〜12 (元 04-05) / R-37-13〜15 (元 07-09) にリナンバリング
- 設計の根本原則: 空売りは補助レーン、direction 軸独立、半自動主軸維持、デイトレ専用、一般信用のみ、段階導入 (M=0〜M+1 Paper Live → M+2 100株 → M+3 本格)
- MVP: S1 悪材料初動 + S4 VWAP 割れ。S2/S3/S5 は F104 完成後
- _index.md を v3.4 に更新、第42章リンク追加

## [2026-05-03] decision | 空売り対応 段階導入ロードマップ追加
- 03_design/空売り対応_段階導入ロードマップ.md 新規作成 (Phase 1〜4 + リスク表 + 撤退基準)
- 第42章 ナビゲーションに本ロードマップへのリンク追加
- タイムライン: 2026-05-10 買い側 Stage 3 → 2026-06 中旬 空売り Phase 1 完了 → 2026-07 中旬 Phase 2 → 2026-07 下旬 Stage 3 (M+2) → 2026-08 中旬 本格運用 (M+3)
- 着手判定: Stage 3 安定運用 (M=0 で問題なし) を確認してから Phase 1
- 撤退基準: Paper Live 20 営業日で期待値マイナス / 踏み上げ 5 回 / 楽天在庫データ取得不可 / 買い側 ROI 優先判断

## [2026-05-03] decision | F260 Block 1 完了 (OpenClaw 仕様調査)

- ~/fire/docs/openclaw/agent_registration_guide.md を新規作成 (7 節構成)
- openclaw agents サブコマンド全 7 件 (add/bind/bindings/delete/list/set-identity/unbind) 仕様確定
- main エージェントの 3 設定 JSON (models.json/auth-profiles.json/auth-state.json) 構造把握、Anthropic key はマスク
- cron add/edit の delivery オプション (--channel/--to/--account/--announce 等) 仕様確定
- routing/workspaces サブコマンドは存在せず → bind/bindings/unbind と --workspace で代替
- F242 scope upgrade ループは再発なし (doctor 確認)、orphan transcripts 124 件は別判断
- 既存 3 cron は agent dir 不在で delivery 失敗中 → Block 2 で agents add → Block 3 で cron edit
- 次は Block 2 (既存3エージェント中身実装) へ

## [2026-05-03] decision | F260 Block 2 完了 (3エージェント初期化)

- 3エージェント (market_intelligence / monitoring_alert / review_journal) を `openclaw agents add` で初期化
- agent/ 配下 (models.json/auth-profiles.json/auth-state.json) は **初回 invoke 時に lazy 生成** (確定)
- auth は main から **自動継承** (sha256 完全一致): c23c3c70...
- test ping 3 件すべて 200 OK (claude-opus-4-7 / 247-463 token)
- 「pong」は返らず BOOTSTRAP.md ルールで identity 確立要求 → 設計通り
- 既存 sessions/ は破壊されず、openclaw.json は自動 backup
- バックアップ (.{name}.bak) は Block 4 まで保持
- cron 3 件は依然 `delivery: announce -> last (no route, fail-closed)` 状態 → Block 3 で `--to <userId|groupId|roomId>` 修正
- 次は Block 3 (cron delivery 設定 = LINE 部屋 binding)

## [2026-05-03] decision | F260 Block 4 完了 (OpenClaw 統合フェーズ完了)

- Phase 4-A: Skills 仕様調査 (skills CLI 6 サブコマンド / SKILL.md フォーマット / agent-skill 紐付けは LLM 自動判定)
- Phase 4-B: 残り 7 エージェント (daytrade_selection / swing_selection / longterm_selection / portfolio_balance / trade_decision / goal_tracking / evaluation) を agents add で登録
- Phase 4-C: 全 10 サブエージェントに name + emoji 設定 (set-identity 4 項目のみ、long-form role は F261 へ)
- Phase 4-D: F012 _oc_agent() 経由で全 10 エージェント `status: ok` 確認
- バックアップ 3 件 (.bak) 削除完了
- ガイド (~/fire/docs/openclaw/agent_registration_guide.md) 11 節構成で最終化
- Skills 詳細実装 (13 Skill の SKILL.md + scripts/wrapper) は **F261 として切り出し**
- 次は Block 3 (Stage 3 環境設定: cron delivery 修正) または F261 (Skills 詳細)

## F260 全体まとめ

Block 1: OpenClaw 公式仕様調査 → ガイド初版作成
Block 2: 既存 3 サブエージェント初期化 + lazy 生成挙動確定 + ガイド更新
Block 4: 残り 7 エージェント登録 + identity + Skills 仕様調査 + F012 統合 + ガイド最終化
(Block 3: Stage 3 環境設定の一部として切り出し)

成果物:
- ~/fire/docs/openclaw/agent_registration_guide.md (11 節構成、トークンマスク済)
- ~/.openclaw/agents/ 配下に 11 エージェント (main + 10 サブ)、auth main 共有
- F012 FIRERunner から全 10 エージェント呼び出し可能

## Stage 3 Block 3 進捗 (2026-05-03 案Y 採用)

- LINE 5 部屋作成 + Bot 招待 完了
- ただし OpenClaw が groupId 取得には Webhook 公開が必要と判明 (LINE Messaging API 仕様)
  - 「Bot 参加グループ一覧 API」は存在せず、Webhook の join イベント経由のみ
  - 私の手順書のバグ (Webhook OFF を推奨していた)
- 案Y を採用: 緊急アラートは LINE_USER_ID 直送で運用、5 部屋運用は Stage 3 後に別途構築
- cron 3 ジョブ (fire_morning_scan / fire_intraday_monitor / fire_eod_review) は disable 済み
  → 案 A のまま、agent 系は将来 F261 / F104 完了後に有効化
- 5 部屋自体は LINE 上に既に作成済み、将来 Webhook 公開設計時に再利用可能
- 次は emergency_alert.py のフォールバック実装確認 → Phase 4 (launchctl 5plist load)

## Stage 3 Block 3 Phase 4-1 + 4-2 案C 完了 (2026-05-03)

- launchctl 5plist load 完了 (~/Library/LaunchAgents/ に配置、launchctl list で5件確認)
- 発火時刻: 14:45 / 14:55 / 15:05 / 15:10 / 15:15 (月-金)
- emergency_alert.py DRY mode 実行成功
  - router 経路: LINE_ROOM_EMERGENCY (空) → LINE_EMERGENCY_GROUP_ID (空) → LINE_USER_ID にフォールバック
  - 解決された宛先: Ud6bfa86... (Fujiwara 個人 1:1 トーク)
- 実 LINE 通知は送信せず、コードパス検証のみ
- 次: 案 A (実機 LINE 通知テスト) 実行可否を Anthropic Claude (このチャット) で判断

## Stage 3 Block 3 完全完了 (2026-05-03)

### Phase 1-4 完了サマリ

- Phase 1: LINE Bot 5部屋作成 + Bot 招待完了 (将来 Webhook 公開設計後に活用予定)
- Phase 2: groupId 取得不可と判明 (Webhook OFF のため、これは LINE Messaging API 仕様)
  → 案Y 採用: LINE_USER_ID 直送方式に切替
- Phase 3:
  - .env に LINE_CHANNEL_TOKEN + LINE_USER_ID 投入
  - OpenClaw cron 3ジョブ (fire_morning_scan/fire_intraday_monitor/fire_eod_review) を disable
    → agent ノイズ通知の流入回避、F261/F104 完了後に有効化予定
- Phase 4:
  - launchctl 5plist load (jp.fire.emergency-1445/1455/1505/1510/1515)
  - 案A3 実機テスト: paper_live_positions クリーン → skipped 確認 →
    TEST 建玉挿入 → 実機発火 → LINE 個人到達確認 → クリーンアップ
  - end-to-end 動作確認完了 (emergency_alert.py → router → line_bot.py → LINE Push API)

### 設計判断記録

- 案Y (LINE_USER_ID 直送) を採用、5部屋運用は将来 Webhook 公開設計後に追加
- 案A (cron 3ジョブ disable) を採用、agent 自動レポートは F261/F104 完了後
- paper_live_positions を Stage 3 開始のため完全クリーン (バックアップは logs/ に保存)

### Stage 3 移行残作業

- F100 historical 実 API 実行 (過去6ヶ月分、1-2時間放置)
- F230 batch_replay (20営業日 Paper Live リプレイ、数時間放置)
- F241 --check-live-advisory --fujiwara-accept で全9項目 PASS
- Stage 3 開始最終承認 (Fujiwara さん意思決定)

### Block 3 で発見・解決した非自明事項

- LINE Messaging API には「Bot 参加グループ一覧取得 API」がない (Webhook 経由のみ)
- agent 系 cron が動くと意味のない自然言語ノイズが 5分毎流れる (現状 prompt 整備不足)
- paper_live_positions は paper_live_runs への FK あり (TEST 挿入時に parent run も必要)
- emergency_alert.py の宛先決定は notifications/router.py:45-59 で 3段階フォールバック実装済み

## F261 Step 3 完了 (2026-05-03 20:20 頃)

### 成果物 (line-notify Skill パターン確立)

- `~/fire/scripts/wrapper/__init__.py` (Python module 化)
- `~/fire/scripts/wrapper/line_notify.py` (CLI ラッパー、router.send_to_room を引数 5 部屋で呼ぶ)
- `~/.openclaw/workspace/skills/line-notify/SKILL.md` (OpenClaw 認識確認済 ✓ Ready)
- `~/fire/docs/openclaw/agents/monitoring_alert_identity.md` (案 B に切替、後述)

### 動作確認

- `openclaw skills info line-notify` で `Source: openclaw-workspace, Path: ~/.openclaw/workspace/skills/line-notify/SKILL.md, Requirements: python3 ✓` 認識
- DRY モード送信成功: `{"status": "dry_run", "to": "Ud6bfa86...", "room": "emergency"}` (.env export 後)

### IDENTITY.md ファイル方式 → 案 B にフォールバック切替

- `openclaw agents set-identity --agent monitoring_alert --identity-file <path>` 実行で
  `No identity data found in <path>` 警告 → FIRE のロール定義 markdown は OpenClaw が
  期待する Kit スタイル (Name/Creature/Vibe/Emoji/Avatar) と互換性なし
- フォールバック発動: 配置先を `~/fire/docs/openclaw/agents/<agent>_identity.md` (案 B) に切替
- F261 完了基準は「IDENTITY.md ファイル作成完了」のみ、OpenClaw 認識は将来仕様変更時に再考
- 6 本すべて案 B で作成する方針

### 留意点 (次の Step / F265 関連)

- `python-dotenv` が `.venv` 未インストール、wrapper 単体実行時は `set -a; source .env; set +a` 必須
- F265 完全消化のため `~/fire/.venv/bin/pip install python-dotenv` を別途検討
- 現状の wrapper は try/except で dotenv import を回避、未インストール時は env export を要求

## F261 Step 4 完了 (2026-05-03 20:30 頃)

### 成果物 (monitoring_alert 系 Skill 2 本完成、3 Skill 体制で line-notify/price-monitor/position-check 揃う)

- `~/fire/scripts/wrapper/price_monitor.py` (paper_live_results → MonitoringAlertAgent.process_events)
- `~/fire/scripts/wrapper/position_check.py` (paper_live_positions read-only query)
- `~/.openclaw/workspace/skills/price-monitor/SKILL.md` (✓ Ready)
- `~/.openclaw/workspace/skills/position-check/SKILL.md` (✓ Ready)
- `~/fire/requirements.txt` 新規作成 (pip freeze ベース 32 行、python-dotenv==1.2.2 含む)
- `~/fire/scripts/wrapper/line_notify.py` コメント追記 (requirements.txt 追加済明示)
- `~/.openclaw/workspace/skills/line-notify/SKILL.md` Commands から `set -a; source .env; set +a` 削除 (dotenv 自動読込前提に簡素化)
- `~/fire/docs/openclaw/agent_registration_guide.md` に F261 検証結果追記 (IDENTITY.md Kit スタイル限定、Custom Skills 配置先確定、Wrapper 規約)

### F265 完全消化

- `~/fire/.venv` に python-dotenv 1.2.2 インストール
- `requirements.txt` に追記 (Mac mini 再構築時の再発防止)
- 全 wrapper で `.env` 自動読込 (`set -a; source` 不要)
- 動作確認: `set -a; source` なしで `dotenv 自動 .env 読込み確認` DRY 送信成功

### IDENTITY.md 完了基準の修正 (Step 3 で発覚した制約)

- 旧仕様: `~/.openclaw/agents/<agent>/agent/IDENTITY.md` (公式仕様未対応)
- 新仕様: `~/fire/docs/openclaw/agents/<agent>_identity.md` (案 B、git 管理下、機密情報なし)
- 理由: OpenClaw `set-identity --identity-file` は **Kit スタイル (Name/Creature/Vibe/Emoji/Avatar) のみ** パース、FIRE のロール定義 markdown は受け付けず
- F261 完了基準は「IDENTITY.md ファイル作成完了」のみとする
- 6 本すべて案 B で作成する方針は継続

### Custom Skills 認識状態

```
✓ ready  📣 line-notify       openclaw-workspace
✓ ready  👀 position-check    openclaw-workspace
✓ ready  📡 price-monitor     openclaw-workspace
```

3 件すべて `Source: openclaw-workspace`、`Requirements: ✓ python3`。

### 残作業 (F261 Phase 1A の続き)

- Step 5: 残り 6 Skill (lot-calculator / order-generator / risk-validator / daytrade-score / candidate-filter / sector-flow-analysis)
- Step 5: 残り 5 IDENTITY.md (daytrade_selection / trade_decision / pattern_research / sector_research / evaluation)
- Step 6: 統合テスト (Chain 完了後)
- Step 7: agent_registration_guide.md 最終化

## F261 Step 5-A 完了 (2026-05-03 20:35 頃)

### trade_decision 系 3 Skill 完成 + 実機動作確認

- `~/fire/scripts/wrapper/lot_calculator.py` (F130 許容損失計算 + 株数算出)
- `~/fire/scripts/wrapper/risk_validator.py` (F140 R-17-07 Execution Quality Gate)
- `~/fire/scripts/wrapper/order_generator.py` (F115 decide_orders ラップ、TradeOrder 11 項目構築)
- `~/.openclaw/workspace/skills/lot-calculator/SKILL.md` (✓ Ready)
- `~/.openclaw/workspace/skills/risk-validator/SKILL.md` (✓ Ready)
- `~/.openclaw/workspace/skills/order-generator/SKILL.md` (✓ Ready)

### 実機動作確認 (read-only、Chain 並行 OK)

- **lot-calculator**: 余力 100 万円 / daytrade × 通常 → allowable_loss = 4000 円。
  entry/SL = 2500/2450 (リスク 50 円/株) → raw_qty=80 株 → 100 株未満 → status: error (insufficient_funds、期待通り)。
  entry/SL = 100/99 (リスク 1 円/株) → raw_qty=4000 株 → 余力内、status: ok、required_capital=400,000。
- **risk-validator**: 7203 long 100 株 → passed=true, reason="features 不在のため通過 (warning)"、
  追加 attribute `data_available=false`, `metrics={}`, `warnings=["execution_features_unavailable"]`。
  F140 fail-open 仕様通り。
- **order-generator**: 7203 (skip-time/loss-control) → 1 order 生成完了。
  entry=1520 (market_prices_daily から自動取得)、quantity=100、sl=1489.6、tp=1596、
  margin_type=信用買建、valid_until=12:50 まで、invalid_condition=1512 を割れたら見送り、
  allowable_loss=4000、estimated_pnl=7600、RR=2.5、gate_passed=true。
  summary: orders=1, rejected=0, remaining_equity=848,000。

### InsufficientFundsError 扱い (補足2 準拠)

exit code 0 + JSON `{"status": "error", "error_type": "insufficient_funds", "message": ...}` 形式に統一。
LLM (OpenClaw agent) が解釈しやすい設計。exit code 非ゼロは pip install 失敗等の異常系のみに限定。

### Custom Skills 認識状態 (6/9 Phase 1A 進捗)

```
✓ ready  📣 line-notify       (Step 3)
✓ ready  💴 lot-calculator    (Step 5-A)
✓ ready  📝 order-generator   (Step 5-A)
✓ ready  👀 position-check    (Step 4)
✓ ready  📡 price-monitor     (Step 4)
✓ ready  🛡️ risk-validator    (Step 5-A)
```

### 残作業 (F261 Phase 1A の続き)

- Step 5-B: daytrade_selection 系 2 Skill (daytrade-score + candidate-filter)
- Step 5-C: sector_research 系 1 Skill (sector-flow-analysis、責務確認も兼ねて)
- Step 5-D: IDENTITY.md 6 本量産 (案 B 統一: ~/fire/docs/openclaw/agents/<agent>_identity.md)
- Step 6: 統合テスト (Chain 完了後)
- Step 7: agent_registration_guide.md 最終化

## F261 Step 5-B 完了 (2026-05-03 20:42 頃)

### daytrade_selection 系 2 Skill 完成

- `~/fire/scripts/wrapper/daytrade_score.py` (生スコア確認、フィルタ無し、デバッグ寄り)
- `~/fire/scripts/wrapper/candidate_filter.py` (F111 select_candidates ラップ、1-3 銘柄厳選 + 戦略コメント生成)
- `~/.openclaw/workspace/skills/daytrade-score/SKILL.md` (✓ Ready)
- `~/.openclaw/workspace/skills/candidate-filter/SKILL.md` (✓ Ready)

### 責務分担 (補足3 準拠、Option 3: scoring と selecting を分離)

- **daytrade-score**: paper_live_results CANDIDATE → 生スコア top-N (read-only、フィルタなし)
- **candidate-filter**: 同 CANDIDATE → 1-3 銘柄厳選 + 戦略コメント (R-07-01 + R-07-04)

統合せず分離した理由: デバッグ用途 (スコア確認) と本番用途 (厳選) で利用シーンが
明確に異なる。daytrade-score は LLM が「なぜこの symbol が選ばれた/外れた」を理解
する際の前段 inspection、candidate-filter は実運用 cron `fire_morning_scan` で呼ぶ。

### Custom Skills 認識状態 (8/9 Phase 1A 進捗、残り 1 = sector-flow-analysis)

```
✓ ready  📣 line-notify       (Step 3)
✓ ready  💴 lot-calculator    (Step 5-A)
✓ ready  📝 order-generator   (Step 5-A)
✓ ready  👀 position-check    (Step 4)
✓ ready  📡 price-monitor     (Step 4)
✓ ready  🛡️ risk-validator    (Step 5-A)
✓ ready  📊 daytrade-score    (Step 5-B)
✓ ready  🎯 candidate-filter  (Step 5-B)
```

### 気づき: F140 Execution Quality Gate の fail-open 挙動 (補足4)

Step 5-A の risk-validator 実機テストで確認:

```
{"status": "ok", "passed": true, "reason": "features 不在のため通過 (warning)",
 "data_available": false, "warnings": ["execution_features_unavailable"]}
```

F140 (`risk/execution_gate.check_execution_gate`) は **features テーブルにデータが
無い銘柄に対して fail-open** で通過させる現行設計。FIRE Phase 1 では features の
拡充が間に合っていないため fail-open は妥当だが、**Stage 3 移行 (F266 / 実弾運用)
時には fail-closed (features 不在で reject) への切り替えを検討**すべき。

F261 のスコープ外。F141/F144 (執行品質関連の追加実装) と一緒に判断する別タスクと
してログに残す (このまま見過ごすとリスク管理上の盲点になる)。

### 残作業 (F261 Phase 1A の続き)

- Step 5-C: sector-flow-analysis 1 Skill (sector_research.py 574 行を Read 後に設計)
- Step 5-D: IDENTITY.md 6 本量産
  - 第 1 陣 (5-B 直後): daytrade_selection / trade_decision / monitoring_alert / pattern_research
  - 第 2 陣 (5-C 後): sector_research / evaluation
- Step 6: 統合テスト (Chain 完了後)
- Step 7: agent_registration_guide.md 最終化

## F261 Step 5-D 第 1 陣 完了 (2026-05-03 20:48 頃)

### IDENTITY.md 3 本量産 (案 B 統一: ~/fire/docs/openclaw/agents/<agent>_identity.md)

- `daytrade_selection_identity.md` (4,460 byte 想定、F111 役割定義 + R-07-01 / R-07-04)
- `trade_decision_identity.md` (5,200 byte 想定、F115 役割定義 + R-06-03 11 項目 + 第 5 章 R-05-XX)
- `pattern_research_identity.md` (4,800 byte 想定、F035 + F243 統合役割、Phase 1A 未 Skill 化を明記)

monitoring_alert_identity.md (Step 3 既存、3,150 byte) と統一テンプレで Role / Responsibility / Skills / Strict Constraints / Trigger / Output Format / Out of Scope / Related Files。

Phase 1A 4/6 IDENTITY.md 完成。残り 2 (sector_research / evaluation) は Step 5-C / 5-D 第 2 陣。

### F268 候補 (技術的負債): pattern_research.py 重複メソッド

`~/fire/agents/pattern_research.py` 行 333 と 415 で `research_sector_individuality`
が**重複定義** (415 のみ有効、333 は Python の class 再定義で上書きされ dead code)。

- F261 のスコープ外
- 別タスク (F268 候補) として整理: 333 行目を削除 + テスト追加 (どちらの版が想定動作か検証)
- 影響範囲: F243 sector 特化研究 (F035 から委譲)、Sector / Symbol whitelist 提案 (F039)
- 優先度: 中 (動作上は 415 の定義が使われており実害無し、ただしコード理解の障害)

### F140 fail-open の確認 (Step 5-A 既記録、Step 5-D で再確認)

risk-validator 実機テスト + order-generator 実機テストの両方で確認:

```
"reason": "features 不在のため通過 (warning)"
"data_available": false
"warnings": ["execution_features_unavailable"]
```

→ Stage 3 移行時 (F266 / 実弾運用) に fail-closed 切替を検討。F141/F144 と一緒に判断。

### Chain 進捗 (5-D 第 1 陣完了時点)

- **Run 1 (20d) 完了** 20:44:28 (所要 32 分、Run 1 開始 20:12:12)
  - exit=0, summary: closed trades 0、events 0 (テスト DB が空のため期待通り)
- **Run 2 (60d) 開始** 20:44:28 (PID 58654、CPU 95%+ で計算中)
- Run 3 (120d) は Run 2 完了後 (Run 2 ETA 約 1.5-3 時間 → Run 3 開始 22:30〜23:30、ALL DONE 翌朝 03:00 ± 90 分)
- ALL DONE 未検出 → F261 継続 OK

### 残作業

- Step 5-C: sector-flow-analysis Skill (sector_research.py 574 行を Read 後に設計)
- Step 5-D 第 2 陣: sector_research_identity.md / evaluation_identity.md
- Step 6: 統合テスト (Chain 完了後)
- Step 7: agent_registration_guide.md 最終化 (Section 9 拡充)

## F261 Step 5-C 完了 (2026-05-03 20:55 頃) — Phase 1A Skill 9/9 完成 🎉

### sector-flow-analysis Skill 完成

- `~/fire/scripts/wrapper/sector_flow_analysis.py` (F243 sector_research の関数を委譲、ロジック書き直しなし、補足3 準拠)
- `~/.openclaw/workspace/skills/sector-flow-analysis/SKILL.md` (✓ Ready)

### 設計修正適用 (Fujiwara 指示)

- **`--with-proposals` を default on に変更** → `--no-proposals` で軽量版に切替
  (理由: IndividualityFinding 取得後ほぼ必ず proposals が欲しくなる、副作用ゼロ)
- `--mode scan` 用の `--max-patterns` / `--only-with-individuality` 将来用オプション追加
  (現状 active 1 件で恩恵小だが、Pattern Store 拡充時に肥大化対策)

### 実機動作確認 (read-only、Chain Run 2 並行 OK)

- **--mode pattern + 既存 active**: pattern_id `material_initial-strong_market-breakout-AM-highliq@v1.0`
  → finding (overall 0 trades) + proposals (個別性なし) 正常返却
- **--mode pattern --no-proposals**: finding のみで proposals キー無し ✅
- **--mode scan**: target_statuses 3 件で active 全スキャン → n_findings=0 (個別性検出なし、現状妥当)
- **invalid input**: `--mode pattern` で `--pattern-id` 欠落 → status=error, error_type=invalid_input ✅

### Custom Skills 認識状態 (Phase 1A 9/9 完成)

```
✓ ready  📣 line-notify           (Step 3)
✓ ready  💴 lot-calculator        (Step 5-A)
✓ ready  📝 order-generator       (Step 5-A)
✓ ready  👀 position-check        (Step 4)
✓ ready  📡 price-monitor         (Step 4)
✓ ready  🛡️ risk-validator        (Step 5-A)
✓ ready  📊 daytrade-score        (Step 5-B)
✓ ready  🎯 candidate-filter      (Step 5-B)
✓ ready  🏢 sector-flow-analysis  (Step 5-C)
```

### F053 Run 1 結果 (2026-05-03 Run 1 完了時記録、Step 5-C で確定)

batch_replay 20 営業日 (run_id: PL-20260503111212-D43E、tick 1340) の F053 Promotion Check:

- **sample_size**: FAIL (n_days=0, n_trades=0、>= 20/50 必要)
- **expected_value**: FAIL (no closed trades to evaluate)
- **halt_function**: FAIL (halt_reason 未記録、暫定判定の仕様)
- **close_function**: FAIL (no force_close events、untested)
- **simulation_accuracy**: PASS (optimistic_bias_score=0.000)

推測:
- F032 ReproducibilityEngine が動いていない
- または現在の active パターン 1 件が入力データ (market_prices_daily 4400+ 銘柄) にヒットしていない

→ F266 Stage 3 ゲート判定で再評価必要。F261 完了後に Fujiwara 判断 (Pattern Store
拡充 / extract_candidates のロジック確認 / F032 動作検証など)。F261 のスコープ外。

### 残作業 (F261 Phase 1A 完了に向けて)

- **Step 5-D 第 2 陣**: sector_research_identity / evaluation_identity (~/fire/evaluation/ 構造確認後)
- **Step 6**: 統合テスト (Chain ALL DONE 後、9 Skill chain 動作確認)
- **Step 7**: agent_registration_guide.md 最終化 (Section 9 拡充: 9 Skill レシピ集)

## F261 Step 5-D 第 2 陣 完了 (2026-05-03 21:05 頃)

### IDENTITY.md 残り 2 本完成

- `~/fire/docs/openclaw/agents/sector_research_identity.md` (7,685 byte、Skills あり版: sector-flow-analysis Phase 1A 実装済を明記)
- `~/fire/docs/openclaw/agents/evaluation_identity.md` (11,482 byte、Implementation Status 詳細記載: 9 ファイル / 1,933 行 / Phase 1-3 全実装済)

### evaluation/ 構造判断結果 (パターン 1 確定)

調査結果から**パターン 1: F119 Evaluation Agent 本実装** で確定:

- `__init__.py` で「F119: Evaluation Agent (第 13 章) Phase 1/2/3 完了」と明記
- 9 ファイル / 1,933 行
- `optimistic_bias` / `simulation_accuracy` / `promotion` / `昇格` キーワードは含まれない
  → F046/F047 (Simulation Accuracy) や F053 (Promotion) とは別物
- 全 9 ファイルに `EvaluationAgent` / `R-13` キーワード含む → F119 確定

**F266 Stage 3 ゲート判断材料**: `evaluation.orchestrator.run_evaluation()` で
日次 evaluation レポート + 改善提案 + Markdown 提案書 + LINE 通知 + DB 永続化が
即生成可能。Skill 化 (Phase 1B `evaluation-run`) は未だが、本体は Stage 3 移行時
に呼び出し可能。

---

# 🎉 F261 Phase 1A 完了マーク (2026-05-03 21:05) 🎉

## 成果物サマリ

### Skills 9 個 (✓ Ready 全件)

| 絵文字 | Skill | wrapper.py | 役割 |
|---|---|---|---|
| 📣 | `line-notify` | `line_notify.py` | 5 部屋ルーティング + LINE_USER_ID フォールバック |
| 💴 | `lot-calculator` | `lot_calculator.py` | F130 許容損失 + 株数算出 (R-05-02) |
| 📝 | `order-generator` | `order_generator.py` | F115 decide_orders ラップ (R-06-03 11 項目) |
| 👀 | `position-check` | `position_check.py` | paper_live_positions read-only 照会 |
| 📡 | `price-monitor` | `price_monitor.py` | F054 TP/SL → EXECUTION 通知 (F116) |
| 🛡️ | `risk-validator` | `risk_validator.py` | F140 R-17-07 Execution Quality Gate |
| 📊 | `daytrade-score` | `daytrade_score.py` | 生スコア確認 (フィルタなし、デバッグ寄り) |
| 🎯 | `candidate-filter` | `candidate_filter.py` | F111 select_candidates ラップ (R-07-01/04) |
| 🏢 | `sector-flow-analysis` | `sector_flow_analysis.py` | F243 個別性検出 + フィルタ提案 |

### Wrappers 9 個

`~/fire/scripts/wrapper/` 配下、全 wrapper で `try: load_dotenv() except ImportError: pass` パターン統一。
status code 0 + JSON `{"status": "ok"|"error", ...}` 形式統一。

### IDENTITY.md 6 本

`~/fire/docs/openclaw/agents/*_identity.md` 配下 (案 B、git 管理下、機密情報なし):

| ファイル | 行数 (バイト) | Skills セクション |
|---|---|---|
| `monitoring_alert_identity.md` | 3,150 byte | Phase 1A 実装済: price-monitor, position-check, line-notify |
| `daytrade_selection_identity.md` | 6,290 byte | Phase 1A 実装済: daytrade-score, candidate-filter, line-notify |
| `trade_decision_identity.md` | 8,459 byte | Phase 1A 実装済: lot-calculator, risk-validator, order-generator, line-notify |
| `pattern_research_identity.md` | 7,375 byte | Phase 1A 未作成 (Phase 1B 繰延、F267 候補) |
| `sector_research_identity.md` | 7,685 byte | Phase 1A 実装済: sector-flow-analysis |
| `evaluation_identity.md` | 11,482 byte | Phase 1A 未作成 (Phase 1B 繰延、F267: evaluation-run / evaluation-approve) |

統一テンプレ: Role / Responsibility / Skills / Strict Constraints / Trigger / Output Format / Out of Scope / Related Files (+ evaluation のみ Implementation Status 追加)。

### F265 完全消化

- `python-dotenv 1.2.2` インストール (`~/fire/.venv`)
- `~/fire/requirements.txt` 新規作成 (32 行、pip freeze ベース)
- 全 9 wrapper で `.env` 自動読込 (set -a; source 不要)
- 動作確認済 (Step 4 で `.env export なし`の DRY 送信成功)

### F261 検証結果 (agent_registration_guide.md Section 9 で記録)

- IDENTITY.md ファイル方式は Kit スタイル限定 → 案 B (`~/fire/docs/openclaw/agents/<agent>_identity.md`) 採用
- Custom Skills 配置先: `~/.openclaw/workspace/skills/<name>/SKILL.md` (workspace 共有、`Source: openclaw-workspace`)
- Wrapper 規約: `~/fire/scripts/wrapper/<skill_name>.py` (Python module、underscore 命名)
- 呼び出し: `cd ~/fire && .venv/bin/python -m scripts.wrapper.<skill_name>` (`.env` 自動読込)

## Chain 並行運用結果

- **Chain 投入** 2026-05-03 19:34:10 (PID 56992)
- **Phase 1**: F100 完了待ち → 20:12:11 完了 (38 分、122 営業日 / 全 4449 銘柄取得)
- **Phase 2**: 失敗銘柄再取得 → 20:12:11〜20:12:12 (誤抽出 `0`/`6` 含む 8 件、正規 6 件は 720 行挿入で復旧)
- **Phase 4-A (Run 1)** 開始 20:12:12 → 完了 20:44:28 (所要 32 分、20 営業日 / 1,340 ticks / 0 events / F053 FAIL)
- **Phase 4-B (Run 2)** 開始 20:44:28、ELAPSED 18:42、CPU 96% 継続 (ETA 22:00-23:30)
- **Phase 4-C (Run 3)** 未開始 (Run 2 完了後、ETA 翌朝 04:00 ± 90 分)
- ALL DONE 未検出、F261 Phase 1A 完了時点で **Chain 並行成立** ✅

並行実行リスク (SQLite WAL 競合等) は問題なし、本 Phase 1A 中に Run 1 + Run 2 が
順次完了 / 開始する状態を観察できた。

## 残作業 (Phase 1A 完了後、Chain ALL DONE 後に実施)

### Step 6: 統合テスト (約 1-2 時間)

- 9 Skill chain の動作確認 (line-notify ↔ price-monitor ↔ position-check 等)
- 想定 cron シナリオ rehearsal (`fire_morning_scan` → daytrade-score → candidate-filter → order-generator → line-notify)
- F012 FIRE Runner からの Skill 呼び出しテスト
- price-monitor / order-generator の実機実行 (Chain 完了後、DB 競合無し状態で)

### Step 7: agent_registration_guide.md 最終化 (約 30 分)

- Section 9 拡充: 9 Skill レシピ集 (各 Skill の責務 + 呼び出し例)
- IDENTITY.md 案 B 経緯のまとめ
- F265 dotenv 経緯のまとめ
- Phase 1B 繰延項目の明示

### Step 8: TODO 更新 / F261 完了確定 (約 15 分)

- ~/fire-vault/02_todo/F261_*.md (詳細メモ) 作成
- CLAUDE.md の完了テーブルに F261 1 行追加
- Google Sheets TODO_Master 更新 (Fujiwara 手動)

## Phase 1B 繰延項目 (F267 候補で正式起票予定)

### 未実装 5 エージェント (本体 + Skill + IDENTITY 全部繰延)

- F112 Swing Selection Agent (`agents/swing/` 空)
- F113 Long-term Selection Agent (本体無し)
- F114 Portfolio Balance Agent (本体無し)
- F118 Goal Tracking Agent (本体無し)
- F072 Review & Journal Agent (`agents/review/` 不在、新規作成必要)

### 既存実装の Skill 化

- `pattern-research-daily-report` (F035 `daily_research_report()` ラップ)
- `pattern-decay-scan` (F035 `detect_pattern_decay()` ラップ、週次)
- `pattern-rehab-scan` (F035 `propose_rehab_revival()` ラップ)
- `pattern-adopt-request` (F035 `submit_adopt_approval_request()` ラップ)
- `evaluation-run` (F119 `orchestrator.run_evaluation()` ラップ、cron `fire_eod_review` 連携)
- `evaluation-approve` (F119 `EvaluationApprovalManager` 経由の承認操作)

### 共通基盤改善

- F063 LINE コマンド受信 (Webhook 公開設計と連動、約定報告の手動入力経路)
- F072 引け後レビュー (Review & Journal Agent 新規実装)

## 別タスク候補 (F268 候補)

### F268-A: pattern_research.py 重複メソッド整理

`~/fire/agents/pattern_research.py` 行 333 と 415 で `research_sector_individuality`
が重複定義 (415 のみ有効、333 は dead code、Step 1 で発見)。

- 影響: コード理解の障害、実害なし (415 が動いている)
- 優先度: 中
- 工数: 30 分 (333 行目削除 + 該当テスト追加 + 既存テスト確認)

### F268-B: F140 fail-open → fail-closed 切替検討

Step 5-A の risk-validator 実機テストで判明:
- F140 `risk/execution_gate.check_execution_gate` は features 不在で fail-open (warning + 通過)
- Phase 1 では妥当だが、**Stage 3 移行 (F266 / 実弾運用) で fail-closed への切替検討必要**
- F141/F144 (執行品質追加実装) と一緒に設計すべき
- 別タスク化推奨

## F053 Run 1 結果 (F266 判断材料、既記録、Step 5-D 第 2 陣で再掲)

batch_replay 20 営業日 (run_id: PL-20260503111212-D43E) F053 Promotion Check:

- **sample_size**: FAIL (n_days=0, n_trades=0)
- **expected_value**: FAIL (no closed trades)
- **halt_function**: FAIL (halt_reason 未記録、暫定判定の仕様)
- **close_function**: FAIL (no force_close events)
- **simulation_accuracy**: PASS (optimistic_bias_score=0.000)

推測:
- F032 ReproducibilityEngine が動いていない or
- 現在の active パターン 1 件 (`material_initial-strong_market-breakout-AM-highliq@v1.0`) が
  入力データ (market_prices_daily 4400+ 銘柄 × 122 営業日) にヒットしていない

→ F266 Stage 3 ゲートで再評価必要。Pattern Store 拡充 / extract_candidates のロジック確認 /
F032 動作検証など、Fujiwara 判断。F261 のスコープ外。

---

**F261 Phase 1A の中断時点での状態**: Phase 1A 完了確定。
Step 6-8 は Chain ALL DONE (翌朝予測) 後に Fujiwara が再開判断可能。
F266 / F267 / F268 候補が log.md 上で TODO Excel 更新可能な形に整理済み。


## F261 Step 6 read-only 部分 + Step 7 完了 (2026-05-03 21:25 頃)

### Step 6 read-only 動作再確認 (Chain Run 2 並行 OK)

全 9 Skill `openclaw skills list` 認識確認 ✅ (Source: openclaw-workspace × 9 件)

実機 read-only 5 件 PASS:
- `position-check --json`: `{"total":0, "n_open":0, "n_closed":0, "positions":[]}` (案 A3 クリーン状態維持)
- `daytrade-score --top 5 --json`: `{"n_raw_input":0, "n_top":0, "candidates":[]}` (Run 1/2 でも CANDIDATE 未生成)
- `candidate-filter --json`: `{"status":"no_candidates", "message":"入力候補なし"}` (空完走)
- `sector-flow-analysis --mode pattern --pattern-id <active>`: `{"status":"ok", "finding":{"has_individuality":false}, "proposals":{"rationale":"個別性なし、フィルタ提案不要"}}`
- `sector-flow-analysis --mode scan`: `{"status":"ok", "n_findings":0, "findings":[]}`

`line-notify --dry-run`: `{"status":"dry_run", "to":"Ud6bfa86...", "preview":"F261 Step 6 read-only test (DRY)"}` ✅

### Step 6-4 (lot-calculator / risk-validator / order-generator) はスキップ

Step 5-A で実機 PASS 済 (4,000 円許容損失 / 7203 fail-open / TradeOrder 11 項目構築)。
Run 2 並行で書き込み発生のリスク回避のため再実行はせず、Step 6 統合テストの範囲は
read-only に限定。書き込みを伴うフルテストは Chain ALL DONE 後 (Step 8 で実施)。

### Step 6-5 Skill chain 論理接続レビュー

agent_registration_guide.md Section 12.6 で図示。3 cron シナリオ (朝 / 場中 / 夕)
の Skill 接続が JSON 出力統一で実現可能と確認。要注意点:
- candidate-filter の `candidates[].symbol` → order-generator の `--symbols` 変換は
  agent (LLM) の責務 (wrapper では一括 chain しない設計)
- TP/SL 価格の上書き経路は order-generator 引数拡張が必要 (Phase 1B 候補)
- evaluation-run Phase 1B 繰延 → fire_eod_review cron は当面 position-check +
  sector-flow-analysis のみで運用

### Step 7 agent_registration_guide.md Section 12 本格拡充

旧 Step 4 で line 711 に追加した `## 9. F261 検証結果` (3 サブセクション、56 行) は
**Section 番号衝突** (line 559 既存 `## 9. identity 設定方式` と重複) があったため、
Edit で全置換 → `## 12. F261 OpenClaw Skill 統合 (Phase 1A 実装記録)` にリネーム。

12.1 概要 / 12.2 9 Skill レシピ集 (絵文字+引数+output 一覧表) / 12.3 Wrapper 規約
(共通テンプレ提示) / 12.4 IDENTITY.md 案 B 採用経緯 / 12.5 SKILL.md フォーマット
(bundled weather 参照) / 12.6 cron シナリオ設計 (9 Skill chain 接続図) /
12.7 既知の制約と Phase 1B 繰延項目 / 12.8 トラブルシューティング (5 ケース)。

ガイド総行数: 767 → 990 行 (+223 行、F261 統合記録として整理)。

### 残作業 (Step 8、Chain ALL DONE 後の明朝)

- Step 8-A: 統合テスト残り (price-monitor / order-generator フル実行、DB 競合無し状態で)
- Step 8-B: ~/fire-vault/02_todo/F261_*.md 詳細メモ作成
- Step 8-C: CLAUDE.md の完了テーブルに F261 1 行追加
- Step 8-D: Google Sheets TODO_Master 更新 (Fujiwara 手動)

## F141 ハイブリッド執行方針 完了 (2026-05-03 21:42)

優先度・最優先 (TODO Excel)。F140 ✅ 完了に依存して着手、Phase 1A の終了後に Chain Run 2 並行で実装。

### 成果物 (~/fire commit 932adb6)

- 新規: `~/fire/execution/__init__.py` (F141 package、20 行)
- 新規: `~/fire/execution/hybrid.py` (本体、302 行、`HybridExecutionDecision` + `decide_hybrid_execution`)
- 新規: `~/fire/tests/execution/__init__.py` (空)
- 新規: `~/fire/tests/execution/test_hybrid.py` (13 テスト、278 行、全 PASS)
- 初 commit: `~/fire/agents/trade_decision.py` (528 行、F115 全体 + F141 組み込み)
  - `TradeDecisionResult` に `hybrid_decisions: list` フィールド追加
  - `decide_orders` で `_build_order` 完了後に F141 後処理を呼び出し
  - **注: trade_decision.py は元々 untracked、F141 commit で初めて git 追跡 (528 行が追加扱い)**

### 実装した要件

- 第 1 章 line 84-85「ハイブリッド執行方針 (Sランクはブレイク狙い、通常は指値主体)」
- 第 17 章 R-17-01 (成行乱用禁止・指値標準)
- 第 17 章 R-17-08 (執行ルール: ENTRY 指値/利確指値/損切逆指値必須)

### 判定ロジック

| 条件 | entry_method | rationale |
|---|---|---|
| `Rank.S` (通常) | `stop_market` + `max_slippage_pct=0.3%` | 第 1 章ハイブリッド例外、ブレイク狙い |
| `Rank.S` + 値嵩株 (>=3,000円) + 高スリッページ (>30bps) | `limit` (`fallback_to_limit=True`) | R-17-04 整合、安全側 |
| `Rank.A/B/C/D/X` | `limit` | R-17-01 通常運用 |
| `pattern_rank=None` (DB 引きも失敗) | `limit` (`warnings=["pattern_rank_unknown"]`) | 安全側、R-17-01 標準 |
| `side=="short"` | `limit` (`warnings=["short_side_not_supported_by_F141"]`) | v3.4 F320/F352 系で別途整備、F141 非対応 |

### ブレイク基準価格 (`breakout_trigger_price`) 段階的フォールバック

1. 引数 `breakout_trigger_price` (最優先、`source="argument"`)
2. `order.candidate.breakout_trigger_price` (将来 F111 拡張時用、`source="candidate"`)
3. `market_prices_daily.high` 当日最新値 (AM 高値の代用、`source="am_high"`、F100 minute Phase 2 で精緻化予定)
4. `entry_price * (1 + 0.005)` (`source="default_pct"`)

### 設計判断 (Fujiwara 2026-05-03 確認、不明点 5 への回答)

- **不明点 1**: 案 A (stop_market) 採用、`max_slippage_pct=0.3%` (R-17-04 整合) でスリッページ防御
- **不明点 2**: 段階的フォールバック (引数 → candidate → AM 高値 → entry_price*1.005)
- **不明点 3**: 案 Y 採用、`HybridExecutionDecision` を `TradeDecisionResult.hybrid_decisions` に並行保持。`TradeOrder` の R-06-03 11 項目は変更せず
- **不明点 4**: 案 Q 採用、`_build_order` 完了後に `decide_hybrid_execution(order)` を後処理として呼ぶ
- **不明点 5**: R-17-08 通常運用 + 第 1 章ハイブリッド例外として記述、要件書 17 章本体は未改訂 → F269 候補に記録

### テスト結果

- F141 単体テスト: **13/13 PASS** (Rank.S/A/B/C/D/unknown + breakout fallback 3 種 + short + 値嵩株 fallback 境界 + DB 引き)
- F115 trade_decision 既存テスト: **22/22 PASS** (回帰なし、`TradeDecisionResult` への新フィールド追加で破綻せず、`default_factory=list` で空リスト初期化)
- 統合テスト: `TradeDecisionAgent.decide_orders(candidates=[7203 SimpleNamespace])` → `hybrid_decisions=[HybridExecutionDecision(entry_method="limit", rank="unknown", warnings=["pattern_rank_unknown"])]` 期待通り

### F269 候補 (技術的負債、要件書改訂候補)

**R-17-08「ENTRY 指値必須」と第 1 章 line 85「Sランクはブレイク狙い」の整合性整理**

- 現状: F141 docstring に「R-17-08 通常運用 + 第 1 章ハイブリッド例外」と明記
- 要件書 17 章本体には Sランク例外の明文化なし
- 17 章改訂時に R-17-01 / R-17-08 の例外条項としてハイブリッド執行を追記すべき
- 優先度: 中 (実害なし、コード上は F141 docstring + 第 1 章で根拠示される、ただし要件書整合性の観点で改善余地)

### F141 と F140 の連携 (現状と将来)

現状: `decide_orders` 内で F140 (`_check_execution_gate`) → F141 (`decide_hybrid_execution`) の順で呼ぶ。F141 は F140 の features (`entry_slippage_estimate`) を読んで値嵩株+高スリッページ時のフォールバック判定。

将来 (F141.B): F141 の `entry_method=stop_market` 結果を F140 が読んで「stop_market 時は別の閾値」(例: `EXPENSIVE_SLIP_BPS_CRITICAL` を厳しめに) で判定する経路。今回は F140 改修せず、F141 単体で fallback 判定を完結。

### 残作業 (本タスクで完了、次タスクへ)

なし (F141 完了)。F142 (寄り直後制限 / 特買売禁止 / 価格乖離上限) が次の R-17 系候補。

## F144 執行品質 3 段階監視 + 自己キャリブレーション 完了 (2026-05-03 21:55)

優先度・最優先 (TODO Excel)、F140 ✅ + F141 ✅ + F144 で執行品質 3 点セット完結。F261 Phase 1A の Skill 9 個 / F141 / F144 が今夜揃って Stage 3 移行の執行レイヤが大幅前進。

### 成果物 (~/fire commit 7091c27)

- 新規: `~/fire/execution/slippage_monitor.py` (405 行、`SlippageAlertResult` + `check_slippage_alert` + `record_slippage` + `initialize_slippage_table` + 内部ヘルパー 6 個)
- 新規: `~/fire/tests/execution/test_slippage_monitor.py` (15 テスト、全 PASS)
- 編集: `~/fire/agents/trade_decision.py` (`decide_orders` に F144 case P 組み込み、TradeDecisionResult.summary に slippage_alert dict)
  - シグネチャ追加: `skip_slippage_check: bool = False`, `risk_multiplier: float = 1.0`
  - F133 時刻ガード後 / F132 損失制御前に `check_slippage_alert` 呼び出し
  - `action=halt` → 全候補 reject (rejected[].reason に「F144 slippage halt: ...」)
  - `action=reduce_risk` → equity に risk_multiplier=0.5 を乗算 → F130 許容損失も半減
  - `action=warn` → summary に記録のみ、orders は通常通り構築
  - F144 内部例外は halt せず warning に降格 (F140 fail-open と同方針)
- 編集: `~/fire/.env.example` に `STAGE_3_START_DATE` テンプレ追加

### 実装した要件

- **R-17-09**: 3 段階監視 (注意 / 警戒 / 停止)
- **R-17-10**: 30 日後実測スリッページ中央値ベース判定 (calibration_mode 切替)
- **R-17-11**: Dashboard 可視化のための JSON 出力 (本タスクは未実装、output dataclass 提供のみ)

### 3 段階閾値 + アクション

| level | サンプル | 閾値 | action | risk_multiplier |
|---|---|---|---|---|
| `normal` | - | - | `pass` | 1.0 |
| `caution` | 直近 5 中央値 | > 想定 × 2.0 | `warn` | 1.0 (LINE 警告のみ) |
| `warning` | 直近 10 中央値 | > 想定 × 2.5 | `reduce_risk` | 0.5 (リスク半減) |
| `stop` | 直近 10 中央値 | > 想定 × 3.0 | `halt` | 0.0 (即時停止) |

### baseline (`expected_slippage_bps`) 段階フォールバック

1. 引数 `expected_slippage_bps` (テスト用、最優先)
2. `calibration_mode==post_30d` の場合: 直近 30 日実測スリッページ中央値
3. `features.entry_slippage_estimate` の全 symbol 平均 (F140 と同源)
4. F141 `DEFAULT_MAX_SLIPPAGE_PCT * 10000 = 30 bps` (R-17-04 整合)
5. ハードコード fallback (10 bps、最終手段、未使用)

各段階で fallback 発動時に `warnings` に追加 (運用初期に「features 機能してない?」を早期発見)。

### calibration_mode 切替 (R-17-10)

```python
if business_days_since_start < 30:
    calibration_mode = "pre_30d"   # 想定値ベース判定
else:
    calibration_mode = "post_30d"  # 直近 30 日実測中央値ベース
```

`STAGE_3_START_DATE` は `.env` で指定 (例: `STAGE_3_START_DATE=2026-06-01`)、未設定なら pre_30d 安全側フォールバック + `stage_3_start_date_unset_pre_30d_fallback` warning。

### 当日リセット (9:00 JST 起点、F133 と整合)

- 当日 9:00 JST 以降の記録のみで判定
- 9:00 JST 前は前日 9:00 JST 起点
- 「警戒/停止レベルは当該日の新規エントリー」要件 (R-17-09) を実装

### 設計判断 (Fujiwara 2026-05-03 確認、5 不明点 + 追加提案 2 件への対応)

| # | 採用案 | 内容 |
|---|---|---|
| 1 | 案 A | `~/fire/fire.db` に `slippage_records` テーブル新設 (CREATE IF NOT EXISTS、idempotent) |
| 2 | 段階フォールバック | 引数 > features > F141 default > hardcode、各段階で warning 出力 (F140 fail-open の教訓) |
| 3 | 9:00 JST | 当日リセット起点、F133 時刻ガード (14:45/15:00/15:15) と整合 |
| 4 | .env STAGE_3_START_DATE | 未設定 → pre_30d 安全側 + warning |
| 5 | 案 P | F133 時刻ガード後 / F132 損失制御前、halt 条件として並列配置 |
| 追加 1 | SlippageAlertResult dataclass | F141 HybridExecutionDecision と統一感、`risk_multiplier` フィールド明示 |
| 追加 2 | テスト容易性モック | `current_datetime` + `stage_3_start_date` 引数注入で「Stage 3 開始 35 日後」テスト可能 |

### 連携 (F140 / F141 / F236)

- **F140**: 同じ features ソース (`entry_slippage_estimate`) を使う、F144 で値嵩株フォールバックの bps 閾値も EXPENSIVE_SLIP_BPS_WARNING (30 bps) と整合
- **F141**: `HybridExecutionDecision.max_slippage_pct=0.3%` を F144 の baseline 候補として参照可能 (FALLBACK_F141_DEFAULT_BPS=30)
- **F236 緊急ポジション整理**: 独立経路 (F236=launchd 直撃 LINE、F144=F115 経路の発注品質監視)

### テスト結果 (3 段階全 PASS)

- **F144 単体**: 15/15 PASS (NORMAL / CAUTION / WARNING / STOP + calibration 切替 + フォールバック + 引数注入 + 当日リセット + record_slippage + idempotent)
- **F141 単体**: 13/13 PASS (回帰なし)
- **F115 trade_decision 既存**: 22/22 PASS (回帰なし、`risk_multiplier=1.0` default で破壊なし)
- **F115 統合 3 パターン**:
  - HALT (ratio=3.5): orders=0、rejected reason に「F144 slippage halt: 停止レベル...」 ✅
  - REDUCE_RISK (ratio=2.7): equity 100万 → 50万、許容損失も半減 → 株数不足で reject ✅ (要件通り、安全側)
  - WARN (ratio=2.5): equity 維持、`risk_multiplier=1.0`、`summary["slippage_alert"]` に記録 ✅

### F144 と Stage 3 移行への影響

`evaluation.orchestrator.run_evaluation()` (F119、Phase 1-3 完了済) と組合せると Stage 3 移行直後から:

1. F141 が事前に Sランク → stop_market を判定
2. F144 が実測スリッページを監視、3 段階で発注フローに介入
3. F140 が個別注文の執行品質をゲートチェック
4. F119 が evaluation で日次/週次に滑り傾向を提案

→ Stage 3 における執行品質ループが論理的に閉じる (R-19-08 本番移行条件の「執行品質」項目への大きな進捗)。

### 残作業 / 別タスク候補 (本タスクで完了、log.md に記録)

なし (F144 完了)。次の R-17 系候補:
- **F142**: R-17-02/03/04 (寄り直後制限 / 特買売禁止 / 価格乖離上限)
- **F143**: R-17-05 (部分約定処理)
- **F063**: LINE コマンド受信 (F144 警告通知の自動送信先、Webhook 設計含む)

### 今夜の達成サマリ (2026-05-03 全体)

- ✅ Stage 3 Block 3 完全完了 (Phase 1-4、案 Y / 案 A3 採用、Fujiwara 個人 LINE 到達)
- ✅ F100 historical 過去 6 ヶ月分取得 + Run 1 (20d) batch_replay 完了 / Run 2 (60d) 進行中
- ✅ F261 Phase 1A 完了 (Skill 9 個 + IDENTITY 6 本 + F265 dotenv 統一 + Section 12 ガイド拡充)
- ✅ F141 ハイブリッド執行方針 (R-17-01/08 + 第 1 章) 完了
- ✅ F144 執行品質 3 段階監視 (R-17-09/10) 完了

第 17 章 R-17-01/07/08/09/10 が網羅実装済み (R-17-02/03/04/05 は別タスク残)。

## F142 発注前フィルタ 完了 (2026-05-03 22:15)

優先度・高 (TODO Excel)、F140/F141/F144 の判定実装パターンを継承。第 17 章 R-17-02/03/04 を網羅実装。

### 成果物 (~/fire commit 33bf767、3 files / +813 行)

- 新規: `~/fire/execution/pre_trade_filters.py` (340 行)
  - `PreTradeFilterResult` dataclass + `check_pre_trade_filters` 統合関数
  - `check_opening_restriction` (R-17-02) / `check_special_quote` (R-17-03 hook) / `check_price_deviation` (R-17-04)
  - 内部ヘルパー `_fetch_current_price` (market_prices_daily.close 最新値)
- 新規: `~/fire/tests/execution/test_pre_trade_filters.py` (20 テスト全 PASS)
- 編集: `~/fire/agents/trade_decision.py` (案 P 混合配置で F142 統合)
  - シグネチャに `skip_pre_trade_filters: bool = False` 追加
  - F133 時刻ガード後 / F144 スリッページ前: `check_opening_restriction` 1 回判定 (全候補一括 reject)
  - for ループ内 F140 ゲート後: `check_pre_trade_filters` per candidate (`blocked_by` で reject、F141 hybrid 前)
  - `TradeDecisionResult.summary` に `opening_restriction` (R-17-02 結果) を追加

### 実装した要件

| 要件 ID | 内容 | F142 実装 |
|---|---|---|
| **R-17-02** 寄り直後制限 | 9:00-9:04:59 JST reject、9:05 以降通過 | `check_opening_restriction` |
| **R-17-03** 特買売禁止 | hook only (Phase 1A は常に通過 + warning) | `check_special_quote` |
| **R-17-04** 価格乖離上限 | 通常 0.3%、値嵩 (>=3000円) ±20円 | `check_price_deviation` |

### 設計判断 (Fujiwara 2026-05-03 確認、5 不明点 + 5 追加指針すべて推奨案採用)

| # | 採用 | 内容 |
|---|---|---|
| 1 | 案 (a) | R-17-03 hook only、引数 `(symbol, db_path, current_datetime)` を将来拡張可能に |
| 2 | 案 X | 9:05 JST 固定 (「初動安定後」は Phase 1B / F110 で動的化) |
| 3 | 案 P | 混合配置 (寄り直後 = 冒頭 / 特買売+価格乖離 = per candidate) |
| 4 | 案 M | `current_price` は `market_prices_daily.close` 最新値、`None` → fail-open fallback (F140 教訓) |
| 5 | 3 dict | `opening_check` / `special_quote_check` / `price_deviation_check` 並行保持 (F141/F144 統一) |
| + | プロパティ | `is_passed` / `blocked_reasons` で Dashboard 連携容易性確保 |

### 連携 (F140 / F141 / F144 / F236)

- **F140**: `EXPENSIVE_PRICE_THRESHOLD=3000` を F142 と整合 (値嵩株判定)
- **F141**: `_get_entry_price` と同源 (`market_prices_daily.close`)、F142 で `_fetch_current_price` として再利用
- **F144**: 同じ案 P 階層 (F133 → F142 → F144 → F132 → for ループ)
- **F236**: 独立経路 (F236=launchd 緊急、F142=F115 経路の発注品質)

### テスト結果 (3 段階全 PASS、+ 統合)

- **F142 単体**: 20/20 PASS (寄り直後 6 + 特買売 hook 2 + 価格乖離 8 + 統合 4)
- **F141 + F144 単体**: 48/48 PASS (回帰なし、F142 含む `tests/execution/`)
- **F115 trade_decision 既存**: 22/22 PASS (回帰なし、`skip_pre_trade_filters=False` default で破壊しない)
- **F115 統合**: 寄り直後 (9:03) → 全候補 reject ✅ / 価格乖離 (1.0%) → per candidate reject ✅

### 第 17 章 R-17 進捗 (11 項目中 9 項目完了)

| 要件 | タスク | 状態 |
|---|---|---|
| R-17-01 成行乱用禁止 | F141 | ✅ |
| R-17-02 寄り直後制限 | F142 | ✅ (本日) |
| R-17-03 特買売禁止 | F142 hook | ✅ (本日、Phase 1A hook、本格化は F100/F235) |
| R-17-04 価格乖離上限 | F142 | ✅ (本日) |
| R-17-05 部分約定処理 | F143 | ❌ 未着手 (R-17 章で唯一の実装系残課題) |
| R-17-06 強制クローズ | F236 | ✅ (Stage 3 Block 3 で完了) |
| R-17-07 Execution Quality Gate | F140 | ✅ |
| R-17-08 ENTRY 指値必須 | F141 | ✅ |
| R-17-09 3 段階監視 | F144 | ✅ |
| R-17-10 30 日後実測ベース | F144 | ✅ |
| R-17-11 Dashboard 可視化 | (将来) | ❌ Dashboard 整備で別領域 |

→ **執行品質設計の実装系は F143 (部分約定処理) のみが残課題**。

### 残課題 / 将来タスク

- **F143** R-17-05 部分約定処理: F105 (発注管理) 連携前提のため先行価値小、優先度・高だが順序として後回し可
- **F100 拡張 / F235**: R-17-03 本格化 (special_quote データソース)
- **F110**: R-17-02 動的化 (「初動安定後」の柔軟判定)
- **Stage 3 楽天 API 連携**: `current_price` をリアルタイム値に置換 (現状は `market_prices_daily.close` = 前日終値で代用)
- **Dashboard 整備**: R-17-11、F141 HybridExecutionDecision / F144 SlippageAlertResult / F142 PreTradeFilterResult を可視化

### 今夜の達成サマリ更新 (2026-05-03 全体、F142 追加)

- ✅ Stage 3 Block 3 完全完了
- ✅ F100 historical 6 ヶ月分 + Run 1 (20d) 完了 / Run 2 (60d) 進行中
- ✅ F261 Phase 1A 完了 (Skill 9 個 + IDENTITY 6 本 + F265 + Section 12 ガイド拡充)
- ✅ F141 ハイブリッド執行方針 (R-17-01/08)
- ✅ F144 執行品質 3 段階監視 (R-17-09/10)
- ✅ **F142 発注前フィルタ (R-17-02/03/04)** ← 本タスク

第 17 章実装系: 9/10 完了 (残 F143 のみ、F142 で R-17 設計の主要部分はほぼ完成)。

## F211 目標圧力と運用規律の分離 完了 (2026-05-03 22:45) — 第27章実装系完了

優先度・最優先 (TODO Excel)、第 27 章 (穴 5 対策) を網羅実装。FIRE 全体の最重要原則
「目標未達を理由にした規律崩壊防止」をコード化。

### 成果物 (~/fire commit、4 files / 新規ディレクトリ goal/)

- 新規: `~/fire/goal/__init__.py` (package init、主要 API export)
- 新規: `~/fire/goal/discipline.py` (~530 行、F211 規律ガード本体)
- 新規: `~/fire/tests/goal/__init__.py` (空)
- 新規: `~/fire/tests/goal/test_discipline.py` (22 テスト全 PASS)

### 実装した要件 (R-27-01〜07)

| 要件 | 実装 |
|---|---|
| **R-27-01** 規律崩壊防止 | 全関数の根幹理念、docstring に明記 |
| **R-27-02** 固定原則 4 点 | 定数 + 各関数のロジック |
| **R-27-03** 禁止事項 4 関数 | `check_lot_increase_request` / `check_criteria_loosening_request` / `check_strategy_promotion_request` / `check_stop_condition_relaxation_request` |
| **R-27-04** リスク量変更可否 | `evaluate_market_conditions` (5 項目厳格、データ欠損 fail-safe) |
| **R-27-05** Eval 提案順序 | `propose_evaluation_order` (固定 ["頻度","精度","候補","約定","ロット"]) |
| **R-27-06** 攻める日/守る日 | `decide_attack_or_defend_day` (4+ で発動) |
| **R-27-07** Dashboard | スコープ外 (to_dict() 連携準備) |

### 哲学の一貫性 (全関数で貫く、FIRE 哲学根幹)

- **「足りないから上げる」ではなく「勝ちやすいから上げる」**
- 乖離理由のリスク変更要求は **severity 不問で必ず REJECT** (R-27-03、案 a)
- リスク量変更の正当性は **市場条件 5 項目のみ** (R-27-04、案 X 厳格)
- 攻める日/守る日 は **市場条件のみ** で決定 (R-27-06、目標乖離は不使用)
- データ欠損は **fail-safe** (False 側、F140 fail-open とは反対方針)

### 設計判断 (Fujiwara 2026-05-03 確認)

| # | 採用案 | 内容 |
|---|---|---|
| 1 | 案 X 厳格 | 5 項目すべて favorable で `can_increase=True`、データ欠損は False に倒す |
| 2 | 案 P 全 severity 同一 | 固定 5 項目順序、severity に関わらず同じ |
| 3 | 案 D 攻撃/守備 4+ | `ATTACK_THRESHOLD=DEFEND_THRESHOLD=4`、両方同時 → neutral + warning |
| 4 | 新規 `goal/` ディレクトリ | F210 `goal/tracking.py` 想定で整合 |
| 5 | 案 a severity 不問 | deviation 系 reason は severity に関わらず REJECT (抜け道封じ) |
| + | RejectionReason Enum | deviation/goal_pressure/time_pressure/market_condition/rule の 5 値 |

### F210 mock 経路 (将来統合)

`F211` の入力 `GoalDeviationInput` は Caller が構築。将来 F210 `goal/tracking.py` 完成時:

```python
# 将来の統合例 (F118 Goal Tracking Agent から)
from goal.tracking import calculate_deviation
from goal.discipline import check_lot_increase_request, RejectionReason

deviation = calculate_deviation(...)  # F210 が生成
result = check_lot_increase_request(
    deviation=deviation,
    reason=RejectionReason.MARKET_CONDITION.value,
    market_conditions=...,
)
```

差し替え不要 (`GoalDeviationInput` dataclass の構造を保ったまま F210 が実データから生成)。

### MarketConditions 連携 (5 項目 + 補助 1 項目)

| field | 入力ソース | 状態 |
|---|---|---|
| `pattern_score` | F032 `reproducibility.py` | ✅ 既存 |
| `regime_fit` | F110 (未実装) | ❌ mock |
| `theme_clarity` | F110 (未実装、補助) | ❌ mock |
| `execution_quality` | F140/F144 | ✅ 既存 (Caller 責務で変換) |
| `recent_stability` | F032/F119 | ✅ 既存 (Caller 責務で変換) |
| `system_health_score` | F119 SystemMetrics | ✅ 既存 |

### テスト結果

- **F211 単体**: 22/22 PASS (R-27-03 6 + R-27-04 4 + R-27-06 4 + R-27-05 1 + 統合 7 + rule)
- **F141 + F142 + F144 + F211 + F115 既存**: 92/92 PASS (回帰なし)
- **F115 統合**: 不要 (F211 は別経路の規律ガード、F115 から呼ばれない純粋関数群)

### 第 27 章 R-27 進捗 (実装系 7/7 完了)

R-27-01 ✅ / R-27-02 ✅ / R-27-03 ✅ / R-27-04 ✅ / R-27-05 ✅ / R-27-06 ✅ / R-27-07 (Dashboard) ⭐

**第 27 章は実装系すべて完了**、Dashboard 整備で R-27-07 を将来カバー。

### 残課題 / 将来タスク

- **F210** `goal/tracking.py`: 3000 万円 2 年目標トラッキング (本タスクの mock 入力源)
- **F032/F110/F118 連携**: MarketConditions の各 field を実データ化
- **F244 System Health 拡充**: 現状は F119 SystemMetrics で代用、専用タスクで精緻化
- **F211 Stage 3 運用後の閾値再評価**:
  - `PATTERN_SCORE_FAVORABLE_THRESHOLD=0.7` / `SYSTEM_HEALTH_FAVORABLE_THRESHOLD=0.8`
  - `ATTACK_THRESHOLD=DEFEND_THRESHOLD=4`
  - 要件書未明示、運用知見で調整
- **R-27-07 Dashboard**: F141/F142/F144/F211 の `to_dict()` を可視化する整備タスク

### 今夜の達成サマリ更新 (2026-05-03 全体、F211 追加)

- ✅ Stage 3 Block 3 完全完了 + LINE 個人到達確認
- ✅ F100 historical 6 ヶ月分 + Run 1/Run 2 完了 / Run 3 進行中
- ✅ F261 Phase 1A 完了 (Skill 9 + IDENTITY 6 + F265 + Section 12)
- ✅ F141 ハイブリッド執行 (R-17-01/08)
- ✅ F144 執行品質 3 段階監視 (R-17-09/10)
- ✅ F142 発注前フィルタ (R-17-02/03/04)
- ✅ **F211 規律ガード (R-27-01〜07、第 27 章実装系完了)** ← 本タスク

実装系完了状況:
- 第 17 章: 9/10 (R-17-05 のみ残、F143)
- 第 27 章: 7/7 (Dashboard R-27-07 別領域)
- → **章単位での実装網羅性が大きく前進**

---

## 2026-05-04: F210 Phase 1A 完了 (3000 万円 2 年目標トラッキング、R-01-04/05)

### 完了内容

第 1 章 R-01-04 + R-01-05 の算出層を Phase 1A 範囲で実装。F211 `GoalDeviationInput`
の mock 入力を実関数 `calculate_deviation` に置き換える基盤が完成。Phase 1B (DB
永続化 + 集計) は別タスク `~/fire-vault/02_todo/F210_phase_1b.md` に分割。

### 実装した要素

- **`goal/tracking.py`** (新規): 純粋関数群 + dataclass
  - `DeviationSeverity` Enum (5 段階、R-01-05 完全準拠)
  - `GoalConfig` dataclass (frozen + `from_env()`)
  - `RequiredProgress` NamedTuple (週/月 必要進捗、円・率両方)
  - `classify_severity` (>= 階段判定、Q2 案 a)
  - `severity_to_legacy_label` (F211 互換マッピング、Q1 案 a 修正版)
  - `calculate_deviation` (主役 API、F211 統合エントリ)
  - `calculate_required_weekly_progress` / `calculate_required_monthly_progress`
- **`goal/__init__.py`**: 8 シンボル export 追加 (Phase 1A 限定、`aggregate_by_period`
  は実体なしのため未含)
- **`utils/config.py`**: `GOAL_TARGET_YEN` / `GOAL_START_DATE` /
  `GOAL_DURATION_BUSINESS_DAYS` の 3 キー追加

### 設計判断 (Fujiwara 2026-05-04 確認、Q1-Q7)

- **Q1 案 a 修正版**: `DeviationSeverity` Enum 新規。`AHEAD → "ahead"` で
  R-01-04 思想 (先行も「無理に攻めない」) を反映、legacy `"ok"` 誤読を回避
- **Q2 案 a**: 境界値は `>=` で上側 (Phase 2 確認待ち、`>` で下側案 b への切替余地)
- **Q3 修正版**: `GoalConfig` frozen + デフォルト値なし、`from_env()` 経由
  `ValueError` fail-fast (`RuntimeError` ではなく `ValueError`)
- **Q4 案 a**: F210 は判定のみ、LINE/Eval Agent 連動は F118 で実装
- **Q5 案 a**: 線形補間 (`expected = elapsed / total`)
- **Q6 Phase 1B 回し**: `paper_live_account` は run_id PRIMARY KEY + FK 設計で
  Stage 3 不適合 (構造的根拠 5 件)、Phase 1A では一切触らない
- **Q7 両方出力**: `RequiredProgress` NamedTuple で円・率両方を返却

### deviation_pct セマンティクス整合

F211 既存インラインコメント `(current - target) / target` は misleading で、
実際は F211 fixture (`tests/goal/test_discipline.py:39-49`) と整合する
**進捗ベース** `actual_progress_pct - expected_progress_pct` が正しい。F210 は
fixture 整合解釈を採用 (F211 は severity を分岐に使わないため不整合は無害)。

### テスト結果

- F210 新規 22 PASS (`tests/goal/test_tracking.py`)
- F211 既存 22 PASS + `test_23` (F210 新ラベル直接渡し動作確認) PASS
- `tests/goal/` 合計 **45 PASS**
- 全プロジェクト **1047 PASS** (累計大幅増)

回帰: 1 件失敗 (`tests/risk/test_execution_gate.py::test_f115_decide_orders_rejected_via_gate`)
は F142 (33bf767、寄り直後制限) commit 由来の既存事象。stash 検証で F210 と
無関係であることを確認済。

### 残課題 / Phase 1B

`~/fire-vault/02_todo/F210_phase_1b.md` に分割明記:

- `goal_asset_history` テーブル新規 (CREATE IF NOT EXISTS、Chain 並行性 OK)
- `upsert_asset_record` / `get_asset_history` リポジトリ層
- `aggregate_by_period` (日/週/月集計、`AggregatedDeviation` dataclass)
- `count_business_days_between` (`market_prices_daily` ベース営業日カウント)
- データソース両対応 (`manual_yaml` / `paper_live_account`)

### 次のタスク候補

- F118 Goal Tracking Agent (`agents/goal_tracking.py`、F210 のラッパー)
- F088 Dashboard GoalProgress.tsx (進捗系 5 項目可視化、R-27-07)
- F143 部分約定処理 (R-17-05、第 17 章 10/10 化)

### Phase 1B 凍結判断 (2026-05-04 追記、F210 正式クロージング)

Phase 1A 完了後の P1 議論で、Phase 1B (DB 永続化 + 集計層) は **凍結 = F210
完了扱い** とする決定。判断軸:

- F211 が `severity` 不問で REJECT する設計のため、履歴トラッキングは規律
  ガード経路に不要 (`discipline.py:350/565` の severity 参照は debug/storage)
- 毎朝の手動 LINE/YAML 入力は楽天アプリ残高確認との二重管理になり、
  R-01-08 半自動主軸 + 第 33 章 (運用負荷上限) と整合せず
- Phase 1A の `calculate_deviation` 純粋関数でアドホック計算が可能、
  履歴不在は致命的でない
- Dashboard 自体が未着手で F210 の出力先がない (表示要件確定前の集計仕様
  策定は手戻りリスク)

解凍トリガー (Dashboard 着手時の履歴必要判断 / F118 自動記録要件 / 楽天 API
経路確立) は `02_todo/F210_phase_1b.md` に明記。要件書 R-01-04 の動的目標化
改訂は今回見送り (Phase 1B 凍結により実装も保留、Dashboard 着手時に再評価)。

## 2026-05-04 朝: Chain 完走 → events=0 確定 → 仮説 A (features 未生成) 確定

夜間 Chain (full_chain.sh) が exit 0 で完走したのに、3 run すべて
events=0 / trades=0 / positions=0。F266 (Stage 3 最終ゲート) は昇格基準
未達でブロック。Fujiwara から「案 1 GO + 案 3 並走」指示。

朝のセッションで原因切り分け 4 仮説:

- A 特徴量未生成: **確定 (主因)** — features テーブル 87 行 / 5 銘柄 /
  1 週間のみ。market_prices_daily 526K 行 / 4452 銘柄 / 6 ヶ月の **0.0009%**。
- B announcements 不足: **副因** — 7 件 / 2026-04-30〜05-01 の 2 日分のみ。
- C パターン起動条件: 保留 (主因 A の壁で reject されているため切り分け不能)
- D F032 score 閾値: 不発 (`tick.py:164` の `_has_features` でその前に弾かれる)

ソース確認: `simulation/paper_live/tick.py:139-193 extract_candidates()`
は features 不在の銘柄を即 continue する設計。4452 銘柄全部スキップ →
ReproducibilityEngine.evaluate() に到達せず → events=0。

修正案 (案 1 推奨): `scripts/jobs/extract_features.py` に batch モード
追加 (`--all-symbols` / `--symbols` / `--scope core500` / `--parallel N`)
+ `COLLECTOR_STATUS["material"]["ready"]` を True に修正 (F101 完了反映)。
新規ラッパー `scripts/jobs/backfill_features.py` で TOPIX Core500 ×
過去 120 日を 1 回 backfill。M4 Pro 12 並列で **約 1.5 時間**、その後
Chain 再走 5 時間で events>0 を確認。

並走 STEP 3 (F100 失敗 6 銘柄補完) は全成功:
150A0/15140/15150/15180/151A0/152A0 各 120 行 = 720 行追加挿入、
market_prices_daily は 4452 銘柄キープ。

詳細レポート: `03_design/F032_F054_diagnosis_2026-05-04.md`

学び:
- Chain が exit 0 でも success とは限らない (n_events=0 を異常検知できる
  仕組みが Chain にない → assert を追加すべき)。
- `extract_features.py` が `--code` 単一銘柄前提なのは F055 タスク完了時の
  ミニマル実装のため。Phase 1A 実機運用に向けて batch 化が必要だったが、
  F100 historical 完成と extract_features の batch 化が**並行タスクとして
  リンクされていなかった**のが事故の主因。
- F101 完了 → COLLECTOR_STATUS の更新が漏れた (1 行修正で済む副因)。

次アクション (Fujiwara 承認待ち): 案 1 実装 4 時間 → backfill 1.5 時間 →
Chain 再走 5 時間 = 約 11 時間で events>0 達成見込み。Run a (20 営業日)
だけで先行確認すれば 1.5 + 0.5 時間で events>0 / events=0 の結論が出る。

## [2026-05-04] milestone | F233 週次レビューテンプレ整備 完了

Run a 再走中の並行作業として F233 (M-3〜M+3 週次レビューテンプレ整備、
0.5 日タスク) を実施・完了。Fujiwara さんが毎週日曜夜 (or 月曜朝) に
Obsidian で記入する週次レビュー用テンプレ一式を Vault 内に整備。

### 経緯

- F267 実装 (extract_features batch 化) + backfill 完了後、Run a (20 営業日)
  再走中の空き時間を活用
- 本来の成果物パスは `~/fire/docs/` だが、Obsidian で記入する運用上
  Vault 内が適切と判断し `_templates/weekly_review/` 配下に作成
- ~/fire 側は Run a 走行中のため一切触らず

### 成果物

- `_templates/weekly_review/weekly_review_template.md` — 本体テンプレ
  (8 セクション: ステージ位置 / 開発進捗 / 運用成績 / データ・パイプライン
  健全性 / リスク・ブロッカー / 学び・改善 / 来週計画 / メタ・ジャーナル)
- `_templates/weekly_review/README.md` — 使い方・命名規則・Templater 対応・
  TODO Excel 連携・データ取得 SQL
- `_templates/weekly_review/examples/weekly_review_2026-05-04_W18.md` —
  W18 (2026-04-27〜05-03) の記入例。F267 実装途中・Run a 再走中の
  現状をそのまま反映

### 設計判断 (主要 3 点)

- **Stage 別の記入セクション目安をテンプレ冒頭にマトリクスで明示**:
  Stage 0/1 で運用成績欄 (セクション 3) が空白になる挫折を防ぐ
- **Obsidian core Templates plugin の `{{date:...}}` 構文採用**: Templater
  未導入で動作。Templater 導入時の置換ルールも README 完備
- **TODO Excel との結合は ID + Wikilink (`[[F267|F267]]`)**: 02_todo/ に
  ノート未作成でもリンク切れで OK 運用

### 次回実施

- 初回運用日: 2026-05-10 (日) 夜 21:00 推奨 (Stage 3 開始前の現状から運用
  開始するか、Stage 3 開始月 (M=0) からにするかは Fujiwara 判断)
- 配置場所の確定: `_templates/weekly_review/` で問題なければ確定、別案
  (`04_review/` のようなトップレベルカテゴリ作成) 検討余地あり

### 関連

- 元タスク: [[02_todo/F233_週次レビューテンプレ整備]]
- 要件書: 第 32 章 / 第 38 章 / 第 40 章

