---
id: F286-PNL-R3
phase: P5 / 第 13 章 / 第 19 章 R-19-08 Phase 2 / Stage 3 移行準備
priority: 最優先
status: 完了 ★ (= Wave 6 で実装完了、HQ 統合 approve + 本線 merge 待ち)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R1 (= advisory_decisions.paper_pnl 列、UPDATE 対象)
  - F286-PNL-R2 (= advisory_snapshot_rows、計算 source data)
  - FIRE-LABEL-R1 (= 新ラベル文字列、include/exclude 判定)
  - F286-DATA-R3 (= market_prices_daily を staging に入れる経路)
  - FIRE-CODEX-R1 v1.1 Wave 5 / W5-4 design (= 仮承認設計)
chapter: 第 13 章 / 第 19 章 R-19-08 / 第 25 章 / 第 26 章
---

# F286-PNL-R3: Paper PnL Simulator Hook

最終更新: 2026-05-11

## ★ 状態: 完了

(= Wave 6 で実装 + tests + audit + docs、本線 Integrator review 済、
HQ 統合 approve + 本線 merge 待ち)

## 設計サマリ (= HQ 仮承認の確定 3 件)

- 仮想エントリ価格: 翌営業日寄付 (= market_prices_daily.open)
- 仮想エグジット価格: h20 後の終値 (= market_prices_daily.close)
- 仮想数量: F130 許容損失で逆算 (= allowable_loss ÷ entry × stop_pct)

## paper_pnl 計算対象 (= FIRE-LABEL-R1 新ラベル基準)

| decision_label | 計算 |
|---|---|
| 🟢 積極的買い推奨   | ✓ |
| 🟡 条件付き買い推奨 | ✓ |
| ⚠️ 注意つき買い候補 | ✓ |
| 🟠 場中監視         | × (= None) |
| 🔴 見送り推奨       | × |
| ⚪ 監視のみ         | × |

## 実装ファイル (= Codex W6-1+2 生成、本線 review 済)

| ファイル | 内容 | tests |
|---|---|---|
| pnl/paper_pnl.py | compute_paper_pnl / compute_paper_entry_qty / market data helper | (純関数 test、tests/pnl/test_paper_pnl.py) |
| scripts/jobs/run_f286_pnl_r3_compute_paper_pnl.py | 後追い runner、default dry-run / --write で staging UPDATE | (runner test、tests/scripts/jobs/test_run_f286_pnl_r3_*.py) |
| tests/pnl/test_paper_pnl.py | 純関数 test 多数 | included PASS |
| tests/scripts/jobs/test_run_f286_pnl_r3_compute_paper_pnl.py | runner + 六段ガード test | included PASS |

合計 48 tests PASS (= W6-1+2 で確認)。

## 六段ガード適用 (= W4-2 と同じ pattern)

1. SnapshotStore コンストラクタ read_only=False 強制 (or runner 専用)
2. db_label='staging' 必須
3. db_path basename='fire.staging.db' 必須
4. output path basename / suffix / WAL/SHM 系 path 不可
5. db_path 自体 symlink refuse + Path.resolve() 後の basename 一致
6. db_path が forbidden DB と同一 inode (= hardlink 攻撃) refuse

## UPDATE の限定性 (= 既存 row 保護)

- WHERE advisory_id=? AND code=?
- SET paper_pnl=?, updated_at=?
- 既存 fujiwara_decision / actual_trade / notes / created_at /
  entry_price / 等は touched しない
- paper_pnl IS NULL の行のみ対象 (= idempotent rerun)

## W6-3 audit 結果

- CRITICAL: **0 件** ★
- 観察: 全観点 OK / 推奨対応は Wave 7+ への申送り
- report: /tmp/f286_pnl_r3_audit_report.md

## 安全要件 (= Wave 6 全 ✓)

- 未承認 LINE 送信 0
- production / develop DB write 0
- 未承認 staging DB write 0 (= 本タスク内、tests は tmp_path)
- token / channel_token / secret 参照 0
- 楽天 / 自動発注 / Computer Use / Playwright 不使用
- 注文価格 / 数量 / 執行指示の生成 helper 0 (= paper_pnl は
  シミュレーションのみ、実発注に転用されない設計)
- workflow 変更 0 / --no-verify 不使用
- scripts/seed_pattern_layer1.py / historical_indicators.py 未接触
- TODO Excel 未更新 / cron / launchd 未登録

## Known Limitations / 既知の制約

1. side='buy' (= long) のみ計算、空売り未対応
2. 手数料 / 税金 / 分割 / 配当 補正なし (= gross PnL)
3. F130 許容損失額の default は仮値 (= 10000 円)、実運用は
   --allowable-loss-amount で外部から渡す
4. paper_pnl 計算には market_prices_daily の future close が必要、
   h20 経過後でないと計算不可
5. 市場データ欠落 row は paper_pnl=None で skip (= UPDATE しない)
6. F286-PNL-R3 runner の staging write は **Wave 7 で別 HQ approve**

## 次タスク

1. ★ F286-PNL-R3 W6 完了 (= 本書、実装 + tests + audit + docs)
2. HQ 統合 approve → 本線 Integrator が develop へ feat / test / docs
   commit
3. Wave 7 候補:
   - W7-1: F286-PNL-R3 staging write smoke (= 単独 helper で paper_pnl
     計算、HQ 別 approve 要、W4.1-A と同じ pattern)
   - W7-2: cron 連携 (= sub-D3 凍結解除後、毎営業日 18:00 JST に過去
     20 営業日分の paper_pnl 計算)
   - W7-3: F286-DATA-R3 sub-D2.2 staging write smoke (= 実 fetch /
     実 write 単独 smoke、別 HQ approve 要)

## 関連リンク

- [[../03_design/F286_PNL_R3_paper_pnl_simulator_hook_2026-05-11|F286-PNL-R3 design draft]]
- [[F286_PNL_R1_advisory_decision_pnl_tracking|F286-PNL-R1]]
- [[F286_PNL_R2_advisory_snapshot_auto_ingest|F286-PNL-R2]]
- [[FIRE_LABEL_R1_advisory_label_refresh|FIRE-LABEL-R1 新ラベル]]
- [[FIRE_CODEX_R1_WAVE6_plan|Wave 6 plan]]
- [[../log]]
