---
id: FIRE-CODEX-R1-WAVE21-plan
phase: ガバナンス / Wave 21 / Phase P1 試験投入 / F101 API behavior investigation 実装
priority: 高
status: 進行中 ☆ Phase P1 (= 6 lane) 初試験投入
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 20 (= 完了、R2 採用)
  - HQ Wave 21 起票承認 + Phase P1 採用 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / F101 修正
---

# Wave 21: F101 API behavior investigation implementation (= Phase P1 初試験投入)

W19-1 で特定した F101 `/fins/announcement` 403 問題に対し、
**endpoint resolution の切替可能化** を実装する。

★ **実 API call / token 参照 / DB write / LINE 送信 / cron は全て禁止**。
本 wave は **既存 hardcode endpoint を config 化** + **mock test** + **設計
doc** までを範囲とする。実 probe は別 wave + HQ 明示承認後。

## Wave 21 sub-task (= 8 件、Phase P1 = 6 lane 構成)

| sub  | lane | task                                       |
|------|------|--------------------------------------------|
| W21-1| L5   | Wave 21 plan + R2 prompt template v1.0     |
| W21-2| L1a  | F101 V2 spec + endpoint 候補列挙           |
| W21-3| L1b  | endpoint resolution 設計                   |
| W21-4| L3   | materials/client.py 修正実装               |
| W21-5| L2   | tests (mock response、複数 endpoint pattern)|
| W21-6| L4   | adversarial audit                          |
| W21-7| L6   | regression PASS 確認                       |
| W21-8| 本線 | commit + log.md + HQ 報告                  |

## Phase P1 lane 構成 (= 6 lane、HQ 承認下)

- L1a Design (主)
- L1b Design (副)
- L2 Test
- L3 Implementation
- L4 Audit
- L5 Docs (= 本線 or Codex)
- L6 Regression (= 本線 or Codex)

本 Wave 21 では、まず **lane の役割分担を本線が演じる形** で実行し、
R2 prompt template v1.0 を確立。次 wave (= Wave 22 以降) で実 Codex 6
lane 並列起動を試験。

## スコープ (= HQ 承認範囲)

✓ F101 official V2 spec 確認 (= 静的調査、Web 検索可否は本線判断)
✓ candidate endpoint 整理 (= /fins/announcement / /fins/announcements 等)
✓ endpoint resolution 設計 (= env / arg / config 切替)
✓ materials/client.py 修正実装 (= endpoint hardcode → 切替可能化)
✓ tests (= unittest.mock、実 HTTP なし)
✓ audit (= 本線 or Codex L4)
✓ vault / docs

## 禁止 (= HQ Wave 21 指示)

✗ 実 API call (= 全ての endpoint 候補に対して、staging も production も)
✗ token / secret / channel_token 参照 (= JQUANTS_API_KEY 値読出禁止)
✗ staging DB write
✗ production / develop DB write
✗ LINE 送信 (実)
✗ cron / launchd / crontab 登録
✗ workflow 変更
✗ --no-verify
✗ TODO Excel 更新

## Hard abort 条件 (= HQ 提示)

- audit CRITICAL 1 件以上
- safety violation (= 上記禁止項目に該当)
- 未承認 LINE 送信
- token / secret leak
- file ownership 衝突 ≥ 2
- HQ 明示 abort 指示

## 関連リンク

- [[../03_design/F101_API_403_investigation_2026-05-12|W19-1 F101 調査]]
- [[../03_design/FIRE_CODEX_R2_10_lane_scaling_design_2026-05-12|R2 設計]]
- [[../03_design/FIRE_CODEX_R2_codex_prompt_template_v1_2026-05-12|R2 prompt template v1.0]]
- [[../03_design/F101_endpoint_resolution_2026-05-12|endpoint resolution 設計]]
- [[FIRE_CODEX_R1_WAVE20_results|Wave 20 results]]
