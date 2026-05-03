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
