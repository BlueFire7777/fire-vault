---
id: F286-PNL-R2-runner
phase: P5 / 第 13 章 / 第 19 章 R-19-08 Phase 2
priority: 高
status: 起票 (= 2026-05-11、Wave 4 W4-1、Codex 投入 HQ approve 待ち)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R2 (= snapshot module、commit cb17a7f)
  - F062-R5.8 (= production send 動作確認済)
  - FIRE-CODEX-R1 v1.1 Wave 3 sub-4B (= integration design audit)
chapter: 第 13 章 / 第 19 章 R-19-08 / 第 14 章 LINE 通知配信
---

# F286-PNL-R2-runner: F062 --record-decisions 統合

最終更新: 2026-05-11

## ★ 状態: 起票 (= Wave 4 W4-1)

F062 本番 LINE 送信成功 **直後** に F286-PNL-R2 SnapshotStore を呼び、
advisory_decisions seed + snapshot 保存を行う統合 runner 改修。

Wave 4 内では **dry-run / safety path 中心**、staging DB write は
**実施しない** (= HQ 別 approve 要、Wave 4.1 候補)。

## 設計参照

- [[FIRE_CODEX_R1_WAVE3_audit_results|Wave 3 sub-4B audit]] の integration
  design proposal
- HQ 承認 #3 (= LINE 成功 / snapshot 失敗 → non-zero + 後追い helper)
- HQ 承認 #4 (= LINE 失敗時 → snapshot 残さない)
- HQ 承認 #7 (= --record-decisions は production-only)

## 実装内容 (= Codex 投入予定)

### scripts/jobs/run_f062_line_production_send_smoke.py 修正

CLI 追加:
```
--record-decisions                   # off default、明示時のみ snapshot 保存
--record-db-path data/fire.staging.db
--record-db-label staging
```

ガード:
- `--record-decisions` は `--message-mode production` + `--send` +
  `--hq-approved-send` の **すべて** が成立した場合のみ有効
- preview / dry-run / --send なし では --record-decisions を **無視**
  (= safety 違反でなく、単純に snapshot 保存しない)
- `--record-db-label` が staging 以外、または basename が
  fire.staging.db 以外なら parse_args で refuse

フロー:
1. payload 生成 (= 既存)
2. safety validation (= 既存 + token preflight)
3. canonical payload hash 計算 + send_id 追記 (= W4-1 で新規)
4. LINE 送信
5. sent_count >= 1 確認 (= 既存)
6. **--record-decisions 有効時のみ**: SnapshotStore.save_snapshot 呼出
7. snapshot 保存失敗時: exit code = 1 (= 部分失敗、LINE は OK)、
   stderr に後追い ingest command を表示 (= HQ #3)

exit code:
- 0: LINE 成功 + snapshot 保存成功 (or --record-decisions なし)
- 1: LINE 成功 + snapshot 保存失敗 (= 後追い helper 必要)
- 2: parse_args refuse (= safety violation)
- 3: LINE 送信失敗 (= snapshot 保存しない、HQ #4)

### scripts/jobs/run_f062_research_advisory_line_preview.py 修正 (可能性)

payload 生成時に `advisory_id` / `send_id` / `payload_hash` を payload
top-level + metadata に埋め込む。これにより send_smoke は payload を
読むだけで send_id を取得できる。

ただし、send_smoke が独立に `build_advisory_id(payload)` を呼べば同じ
ID が出るため、preview runner 改修は **必須ではない** (= 設計判断)。

### tests/scripts/jobs/test_run_f062_line_production_send_smoke.py 拡張

- `--record-decisions` + 各種 production-only ガード refuse
- mocked LineBotClient で送信成功 → snapshot 保存呼出される
- mocked LineBotClient で送信失敗 → snapshot 保存呼ばれない (HQ #4)
- snapshot 保存失敗 → exit 1 + stderr に retry command (HQ #3)
- DB write は tmp_path のみ (= production / develop / staging 不可)

## allowed_files

- scripts/jobs/run_f062_line_production_send_smoke.py (= 既存修正)
- scripts/jobs/run_f062_research_advisory_line_preview.py (= 必要なら修正)
- tests/scripts/jobs/test_run_f062_line_production_send_smoke.py (= 既存修正)

## forbidden_files

- pnl/* (= R2 module は既に commit 済、touched なし、import のみ可)
- notifications/* (= W4-3 FIRE-LABEL-R1 担当、衝突回避)
- scripts/seed_pattern_layer1.py
- simulation/research_lane/historical_indicators.py
- TODO Excel
- .github/workflows/*
- ~/.fire_secrets/*

## expected_outputs

1. F062 send_smoke runner の --record-decisions 経路実装
2. (必要なら) preview runner の send_id 埋め込み
3. send_smoke tests 拡張 (= 既存 PASS 維持 + 新規 5-10 件追加)

## test_command

```
.venv/bin/pytest tests/scripts/jobs/test_run_f062_line_production_send_smoke.py -v
```

全 PASS。staging へ実 write しない (= tmp_path のみ)。

## 安全要件

- LINE 本番送信なし (= 本タスク内、tests は mocked)
- production / develop / staging DB write なし
- token / channel_token / secret 参照なし
- 注文価格 / 数量 / 執行指示の生成 helper を追加しない (= 既存運用維持)
- cron / launchd / Computer Use なし
- workflow 変更なし / --no-verify 不使用
- scripts/seed_pattern_layer1.py / historical_indicators.py 未接触
- TODO Excel 未更新

## Codex 投入予定 (= HQ approve 後)

| 段階 | 内容 | lane | 想定時間 |
|---|---|---|---|
| Codex L3 Impl | runner 修正本体 + 必要なら preview 修正 | L3 | 10-15 分 |
| 本線 Integrator review | smoke + 安全項目 + ガード検証 | (本線) | 5-10 分 |
| commit | feat(F286-PNL-R2): integrate --record-decisions in F062 send smoke | (本線) | 5 分 |

## Wave 4.1 staging smoke 候補 (= 本タスク完了後、HQ 別 approve 要)

本 runner 統合完了 + W4-2 ingest helper 完了後、**Wave 4.1** として
staging smoke を実施する候補:

- 経路 1: F062 send_smoke + --record-decisions (= LINE 送信あり、token
  実読み込み、HQ approve 必須 + 1 通だけ送信)
- 経路 2: ingest helper のみ (= LINE 送信なし、payload JSON から再 ingest)
- 推奨: 経路 2 を先に smoke (= LINE 送信 0、staging 書き込みのみで安全
  確認)、その後 経路 1 を smoke

ただし **本タスク (W4-1) では実施しない**。

## 関連リンク

- [[FIRE_CODEX_R1_WAVE4_plan|Wave 4 plan]]
- [[F286_PNL_R2_advisory_snapshot_auto_ingest|F286-PNL-R2 (= snapshot module 完成版)]]
- [[F062_R5_receipt_confirmation|F062-R5 受信確認 (= LINE 送信実績)]]
- [[FIRE_CODEX_R1_WAVE3_audit_results|Wave 3 sub-4B integration audit]]
