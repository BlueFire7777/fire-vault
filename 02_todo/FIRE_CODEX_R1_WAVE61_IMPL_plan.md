---
id: FIRE-CODEX-R1-WAVE61-IMPL-plan
phase: 本番 v0 中核 / Wave 61-impl / paper PnL preview runner 実装
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
implementation_reference: ~/fire-vault/03_design/FIRE_price_return_paper_pnl_linkage_2026-05-14.md
---

# Wave 61-impl Plan — After-R1 Paper PnL Preview Runner Implementation v1.0

## §1 目的

Wave 61-pre で完成した Price / Return / Paper PnL Linkage 設計を実装する。
D10 候補について、価格推移・paper PnL・manual review・pattern outcome を
**read-only** で集約する runner を新規実装。

実装 file:
- `scripts/jobs/run_after_r1_paper_pnl_preview.py` (= 新規)
- `tests/scripts/jobs/test_run_after_r1_paper_pnl_preview.py` (= 新規、20+ tests)
- `tests/scripts/test_db_path_consistency.py` (= 既存 file の allow list 追加)

## §2 安全境界

- DB write: **禁止**
- production / develop DB 接続: **禁止**
- staging DB: **read-only のみ** (= URI mode=ro + PRAGMA query_only=ON)
- LINE / token / API / launchctl / plist / cron / VACUUM / workflow /
  --no-verify / git push / sudo / rm -rf / 実発注 / 楽天 / iSPEED /
  Computer Use / TODO Excel: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist + 3 DB md5 + pnl.paper_pnl helpers signature 確認) |
| 2 | runner skeleton 設計 + 既存 pattern (= run_f111_real_batch_staging) 流用 |
| 3 | runner 実装: guards + input resolver + price/return + paper PnL |
| 4 | runner 実装: review parsing + pattern outcome + JSON/Markdown 出力 |
| 5 | tests 実装 (= 15+ tests、最終 50+) |
| 6 | D10 smoke (= F111 + AFTER-R1 chain + review blank で完整) |
| 7 | Codex 4 lane factual-confirm |
| 8 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- run_after_r1_paper_pnl_preview.py 実装完了
- D10 候補 paper PnL preview 生成可能
- JSON/Markdown 出力可能
- review_missing warning 動作
- skip でも paper 評価可能
- pattern outcome summary 生成可能
- tests PASS (= 50+ tests)
- Codex 4 lane factual-confirm 実施
- DB write 0 / production-develop 接続 0
- staging read-only のみ
- LINE 0 / token 0 / API 0
- launchctl 0 / plist 0 / cron 0
- 実発注 0 / 楽天 0 / iSPEED 0 / Computer Use 0
- F282 plist + 3 DB mtime/size 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
DB write / production-develop 接続 / staging write / token / .env /
API / LINE / launchctl / plist / cron / 実発注 / 楽天 / iSPEED /
Computer Use / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel

## §6 Codex 4 lane (factual-confirm、80 words/lane)

- A: runner schema / CLI
- B: price read-only source (= URI mode=ro + PRAGMA query_only)
- C: review parsing (= is_blank → final_decision='unknown')
- D: safety_flags 10 keys + pattern watch default
