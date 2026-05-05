# F300 Phase 1: scripts/wrapper/ safety inventory (2026-05-05)

## §1 概要

F300「wrapper 群 production CLI safety 全体再設計」Phase 1 成果物。
F261-fix を HQ-Lean ルール 5 自己検証 2 回目で停止 → F300 Pivot に統合した
経緯を踏まえ、全 9 wrapper + 関連 internal (agents/trade_decision.py 等)
の safety hole を全数列挙する。

Phase 2 (修正方針確定) で本部レビュー → Phase 3 (1.5 日) で 1-2 commit に
集約した一括実装、Codex 連鎖検出ゼロを目指す。

調査対象:
- scripts/wrapper/ 配下 9 ファイル + __init__.py
- 関連 internal: agents/trade_decision.py / agents/monitoring_alert.py
- Codex 既検出 + 補強観点 (a-h カテゴリ)

## §2 検出 safety hole サマリ表

| # | ファイル | 種別 | line | 内容 | 重大度 | 修正方針案 |
|---|---|---|---|---|---|---|
| 1 | price_monitor.py | a/g | (working tree 削除済) | --skip-send / --dry-run / LINE_DRY_RUN env | 高 | CLI 削除 (working tree で実施済、F300 で commit) |
| 2 | price_monitor.py | f | line 56-66 SQL | LIMIT + duplicate 判定で古い未通知永久未処理 | ★致命的 | LEFT JOIN paper_live_notifications で未通知のみ取得 |
| 3 | line_notify.py | a/g | line 36, 45, 57 | --dry-run + status="dry_run" 成功扱い | 中 | dry_run は明示 FAIL or 別 exit code (例: 2) |
| 4 | order_generator.py | a | (working tree 削除済) | --skip-time-guard / --skip-loss-control | 高 | CLI 削除 (working tree 実施済、F300 で commit) |
| 5 | order_generator.py | b | line 97-112 (修正済) | except → sys.exit(1) | 高 | 適用済 (working tree、F300 で commit) |
| 6 | order_generator.py | d | --run-id 任意 | sentinel run_id=None で F132 暗黙スキップ | ★致命的 | --run-id required=True + agents/trade_decision.py:280 注釈 |
| 7 | order_generator.py | c/e | --entry-price なし | hardcoded 1000.0 fallback + multi-symbol 矛盾 | ★致命的 | --symbols 単数化 (--symbol) + --entry-price required、または --prices ペア対応 |
| 8 | candidate_filter.py | b | line 96-103 | except → return 0 (silent fail) | 高 | sys.exit(1) + sys.stderr |
| 9 | lot_calculator.py | b | line 66-74 | except → return 0 (silent fail) | 高 | sys.exit(1) + sys.stderr |
| 10 | risk_validator.py | b | line 87-95 | except → return 0 (silent fail) | 高 | sys.exit(1) + sys.stderr |
| 11 | risk_validator.py | b | line 119 | 内部 except: pass (graceful) | 低 | 業務 graceful 妥当性は Phase 2 確認 |
| 12 | sector_flow_analysis.py | b | line 83-90 / 102-106 / 116-123 (3 箇所) | except → return 0 | 高 | sys.exit(1) + sys.stderr (3 箇所) |
| 13 | agents/trade_decision.py | c | line 44 / 593 | FALLBACK_EQUITY = 1_000_000.0 (account 未初期化) | 中 | wrapper で --account-equity required で遮断、または raise InsufficientFundsError |
| 14 | agents/trade_decision.py | c | line 621 | return 1000.0 (entry_price hardcoded fallback) | ★致命的 | wrapper で --entry-price required で遮断 + 内部 raise に変更検討 |
| 15 | agents/trade_decision.py | d | line 139 / 280 | run_id Optional[str]=None + sentinel bypass | ★致命的 | wrapper で --run-id required (#6) + 内部注釈追加 |
| 16 | daytrade_score.py | - | - | 検出なし (try-finally のみ、silent fail なし) | - | 修正不要 |
| 17 | position_check.py | - | - | 検出なし (try-finally のみ、silent fail なし) | - | 修正不要 |
| 18 | __init__.py | - | - | docstring のみ | - | 修正不要 |

合計 15 件 修正対象、4 件 ★致命的 (Stage 3 直撃事故リスク)。
非修正 3 ファイル (daytrade_score / position_check / __init__.py)。

## §3 種別別詳細

### §3.a --skip-* / bypass CLI 引数

| ファイル | 引数 | working tree 状態 | 内部実装側 (維持) |
|---|---|---|---|
| price_monitor.py | --skip-send | 削除済 | agents/monitoring_alert.py:64 skip_send (test 用) |
| price_monitor.py | --dry-run | 削除済 | LINE_DRY_RUN env 経由 (test 用) |
| order_generator.py | --skip-time-guard | 削除済 | agents/trade_decision.py:141 skip_time_guard |
| order_generator.py | --skip-loss-control | 削除済 | agents/trade_decision.py:140 skip_loss_control |
| line_notify.py | --dry-run | 残存 | LineBotClient(dry_run=...) |

修正方針:
- 全 wrapper で `--skip-*` 系を CLI 公開しない (production CLI safety boundary)
- 内部実装側は test 用に skip_*=True 直接呼出維持
- line_notify --dry-run は別問題 (g で扱う)

### §3.b silent return 0 / fail-open

合計 6 ファイル、6 箇所:
- candidate_filter.py:96-103 (1)
- lot_calculator.py:66-74 (1)
- risk_validator.py:87-95 (1) + 119 (内部 except、graceful 妥当性要確認)
- sector_flow_analysis.py:83-90 / 102-106 / 116-123 (3)
- order_generator.py: working tree 修正済 (sys.exit(1) 適用済)

修正方針:
全 except Exception → sys.exit(1) + sys.stderr 適用 (F271 v1.2 §3.6.12
production CLI exit code 一貫性)。

実装パターン (B-3b 同型):
```
except Exception as e:
    logger.exception(...)
    print(f"ERROR: ...", file=sys.stderr)
    sys.exit(1)
```

### §3.c hardcoded fallback 値

- agents/trade_decision.py:44 FALLBACK_EQUITY = 1_000_000.0
- agents/trade_decision.py:593 return FALLBACK_EQUITY (account 未初期化時)
- agents/trade_decision.py:621 return 1000.0 ★(entry_price hardcoded fallback)

修正方針:
- wrapper 側で --entry-price / --account-equity required にして hardcoded
  fallback 経路に到達させない (production CLI safety boundary)
- 内部実装側:
  - 案 A: コメント追加のみ (test 経路は fallback 維持、production CLI は
          wrapper で required 強制)
  - 案 B: hardcoded → raise に変更 (test 経路含めて変更、影響範囲大)
- ★Mac mini 推奨: 案 A (Phase 2 で本部判断)

### §3.d sentinel value bypass

- agents/trade_decision.py:139 run_id: Optional[str] = None
- agents/trade_decision.py:280 if run_id is not None and not skip_loss_control:
  → run_id=None で F132 損失制御暗黙スキップ

修正方針:
- order_generator.py で --run-id required=True
- agents/trade_decision.py:280 直前に注釈コメント追加 (test 経路専用と明記)

### §3.e multi-symbol + 単一 entry_price 矛盾

- order_generator.py --symbols 複数受付 + 内部 _get_entry_price() による DB 引き
- DB 引き失敗時 hardcoded 1000.0 fallback (#3.c と複合)

修正方針:
- 案 A: --symbols → --symbol 単数化、--entry-price 単一値 (Phase 1 簡素化)
- 案 B: --symbols 複数 + --prices カンマ区切り (symbols とペア対応)
- 案 C: --entry-prices JSON ファイル指定 (柔軟だが複雑)
- ★Mac mini 推奨: 案 A (Phase 1 起動には単一銘柄ずつで十分、複数対応は
  Phase 2 拡張、Codex 提案と整合)

### §3.f LIMIT + duplicate 構造問題

- price_monitor.py:56-66 SQL
  `SELECT ... FROM paper_live_results WHERE event_type IN ('virtual_tp',
  'virtual_sl') ORDER BY id DESC LIMIT ?`
- duplicate 判定: MonitoringAlertAgent 側で paper_live_notifications 照合
- 古い未通知 TP/SL イベントが LIMIT 件数より古くなると永久未処理

修正方針:
- 案 A: SQL に LEFT JOIN paper_live_notifications + WHERE notification_id
  IS NULL で未通知のみ取得 (Codex 提案、構造的解消)
- 案 B: NOT EXISTS subquery で同等
- 案 C: --limit デフォルト値増加 (対症療法、§6-7 違反、棄却)
- ★Mac mini 推奨: 案 A (LEFT JOIN、SQL 単純、構造的解消、
  paper_live_notifications テーブル既存)

確認要 (Phase 2):
- paper_live_notifications テーブルの schema
- duplicate 判定の現行ロジック (MonitoringAlertAgent 側)

### §3.g dry_run / status 成功扱い

- line_notify.py:57 `return 0 if result.get("status") in ("ok", "dry_run") else 1`
  → dry_run 経路で実送信なしでも exit 0、production OpenClaw が success
    扱いするリスク

修正方針:
- 案 A: dry_run は明示 FAIL (return 1)、production CLI で実送信のみ成功
- 案 B: --dry-run 引数自体を削除、test 経路は LineBotClient 直接呼出
- 案 C: dry_run 専用 exit code (例: 2)、OpenClaw 側で dry_run 識別
- ★Mac mini 推奨: 案 B (CLI から dry_run 経路完全削除、test 経路は内部
  直接呼出維持、F261-fix-line で当初検討された方針)

各 wrapper の `"status": "ok"` JSON 出力は通常の業務成功表現で問題なし
(エラー時は "status": "error" / "error_type" / "message" で表現、現状維持)。

### §3.h その他構造的 safety hole

- order_generator.py の --account-equity Optional (account 未初期化時の
  FALLBACK_EQUITY=1_000_000.0 fallback)
  → §3.c と複合、wrapper 側で --account-equity required にすべきか要検討
- risk_validator.py:119 の内部 except: pass
  → graceful な業務判定 (features 取得失敗時の fail-open) で妥当性確認要、
    F140 仕様 (現状 fail-open) と整合確認要

## §4 修正方針サマリ

★致命的 (Stage 3 直撃事故リスク) 4 件:
1. price_monitor.py LIMIT + duplicate (#2、§3.f)
2. order_generator.py --run-id sentinel bypass (#6、§3.d)
3. order_generator.py multi-symbol + entry_price (#7、§3.e)
4. agents/trade_decision.py:621 hardcoded 1000.0 fallback (#14、§3.c)

高 (production CLI safety) 6 件:
5. price_monitor.py --skip-send / --dry-run (#1、§3.a)
6. order_generator.py --skip-* (#4、§3.a)
7. order_generator.py except → sys.exit (#5、§3.b、working tree 修正済)
8-10. candidate_filter / lot_calculator / risk_validator silent fail (#8/9/10、§3.b)
11. sector_flow_analysis silent fail × 3 (#12、§3.b)

中 (運用上の改善) 3 件:
12. line_notify --dry-run + status=dry_run (#3、§3.g)
13. agents/trade_decision.py FALLBACK_EQUITY (#13、§3.c)
14. risk_validator 内部 except: pass (#11、§3.b)、業務妥当性要確認

## §5 Phase 3 commit 分割案 (Mac mini 推奨)

Phase 1 で全数列挙完了 → Phase 3 で 1-2 commit 化。

### 案 A (推奨): 2 commit 分割

commit-1 「F300: production CLI safety boundary 確立」(scope: a/c/d/e)
- price_monitor.py: --skip-send / --dry-run 削除 (a)
- order_generator.py: --skip-* 削除 + except → sys.exit(1) +
  --symbols → --symbol 単数化 + --entry-price required + --run-id required (a/b/d/e)
- agents/trade_decision.py: 注釈コメント追加 (c/d 関連、ロジック変更なし)
- docs/openclaw/agent_registration_guide.md: 関連修正

commit-2 「F300: silent fail 解消 + dry_run 仕様変更」(scope: b/f/g)
- candidate_filter.py / lot_calculator.py / risk_validator.py /
  sector_flow_analysis.py: except → sys.exit(1) (b)
- price_monitor.py: SQL LEFT JOIN paper_live_notifications (f)
- line_notify.py: --dry-run 削除 (g、内部経路は LineBotClient 直接維持)

### 案 B: 1 commit 化 (scope 最大)

全修正を 1 commit に集約、F300 完結。Codex review でファイル横断的に
全数チェック → 連鎖検出ゼロを確認できれば最もシンプル。

★Mac mini 推奨: 案 A (commit 単位の責務明確、Codex review 単位調整しやすい)

## §6 Phase 2 で本部判断要請 (Q1-Q5)

- Q1: §3.c agents/trade_decision.py:621 hardcoded 1000.0 → 案 A (注釈のみ) /
      案 B (raise に変更)
- Q2: §3.e --symbols → --symbol 単数化 (案 A) / --prices ペア対応 (案 B)
- Q3: §3.f price_monitor.py LEFT JOIN paper_live_notifications (案 A) /
      NOT EXISTS (案 B) → schema 確認要
- Q4: §3.g line_notify --dry-run 削除 (案 B) / 明示 FAIL (案 A)
- Q5: Phase 3 commit 分割 案 A (2 commit) / 案 B (1 commit)

## §7 関連

- F261-fix 停止経緯: HQ-Lean ルール 5 自己検証 2 回目、Codex 連鎖検出
  4 件で症状 scope の修正限界
- F261-fix-line / F261-fix-tp-sl / F261-fix-order の起票準備キャンセル、
  F300 一本化
- 関連 commit: なし (F300 Phase 1 は調査のみ、Phase 3 で実装)
- 参考: F271 v1.2 §3.6.12 production CLI exit code 一貫性 / §6-13 既存
  内部機構の API 表面公開時の到達可能性過信
- working tree 修正の活用: price_monitor.py / order_generator.py の
  F261-fix 用修正分は Phase 3 で再 stage して活用
