---
id: FIRE-CODEX-R1-WAVE22-plan
phase: ガバナンス / Wave 22 / Phase P1 試験継続 / cron thaw design only
priority: 高
status: 進行中 ☆ 設計のみ、code change 0、cron 登録 0
owner: BlueFire7777 (Fujiwara)
depends_on:
  - Wave 21 (= 完了、Phase P1 役割分担確立)
  - HQ Wave 22 起票承認 (= 2026-05-12)
chapter: ガバナンス / R-01-08 / cron / launchd
---

# Wave 22: cron thaw design only (= Phase P1 試験継続)

過去に凍結された cron 復活 (= thaw) の **設計のみ**。実 cron / launchd /
crontab 登録、実 LINE 送信、実 API call、token / secret 参照、実 DB write
は **全て禁止**。本 wave で実装まで進めず、HQ 明示承認後の別 wave で impl。

## Wave 22 sub-task (= 8 件、Phase P1 = 6 lane 役割表)

| sub  | lane | task                                       |
|------|------|--------------------------------------------|
| W22-1| L5   | Wave 22 plan                               |
| W22-2| L1a  | cron / launchd 構造設計 (= 既存 plist 整理) |
| W22-3| L1b  | F282 weekly snapshot 順序整理              |
| W22-4| L1a  | log rotation 設計                          |
| W22-5| L1b  | dry-run / no-write / no-send 確認方針      |
| W22-6| L1a  | 6 lane Codex 実起動試験計画                |
| W22-7| L4   | adversarial audit                          |
| W22-8| 本線 | commit + log.md + HQ 報告                  |

## Phase P1 試験継続の趣旨

Wave 21 で R2 prompt template v1.0 / lane 役割表を **本線が演じる形** で
確立。本 Wave 22 では:

- **設計 doc を集約形で 1 件** 完成 (= cron thaw 本体設計)
- **6 lane Codex 実起動試験計画** を別 section で起案
- 実 Codex 6 lane 起動可否は本 wave 内で HQ 判断材料として提示
  (= 本 wave では実起動はせず、別 wave で minimal probe 提案)

## スコープ (= HQ Wave 22 承認範囲)

✓ cron thaw design only (= 設計のみ、実 cron 登録なし)
✓ 6 lane Codex 実起動の安全試験 (= 計画立案、実起動は本 wave 内では行わず別 wave 候補)
✓ launchd / cron 設計 (= 既存 jp.fire.emergency-*.plist の整合 + 拡張案)
✓ F282 weekly snapshot との順序整理
✓ log rotation 設計
✓ dry-run / no-write / no-send 確認方針
✓ audit
✓ vault / docs

## 禁止 (= HQ Wave 22 指示)

✗ cron / launchd / crontab 登録 (= 実)
✗ 実 LINE 送信
✗ token / secret / channel_token 参照
✗ production / develop DB write
✗ staging DB write
✗ 実 API call
✗ F101 staging probe (= 別 wave + 別 HQ approve)
✗ workflow 変更
✗ --no-verify
✗ TODO Excel 更新

## Hard abort 条件 (= R2 prompt template v1.0 継承)

- audit CRITICAL 1 件以上
- safety violation (= 上記禁止項目該当)
- 未承認 LINE 実送信
- token / secret leak
- file ownership 衝突 ≥ 2
- HQ 明示 abort

## 関連リンク

- [[../03_design/cron_thaw_design_2026-05-12|cron thaw 設計本体]]
- [[../03_design/FIRE_CODEX_R2_codex_prompt_template_v1_2026-05-12|R2 prompt template v1.0]]
- [[../03_design/F282_environment_isolation_2026-05-08|F282 環境分離]]
- [[F013_Mac_mini_launchd常駐|F013 launchd 常駐]]
- [[FIRE_CODEX_R1_WAVE21_results|Wave 21 results]]
