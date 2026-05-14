---
id: FIRE-CODEX-R1-WAVE60.5-codex-plan
phase: 本番 v0 中核 / Wave 60.5-codex / Codex Permission Recovery
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
---

# Wave 60.5-codex Plan — Codex Permission Recovery / 8-Lane Parallel Dev Restore v1.0

## §1 目的

W60-post で発生した `codex review` permission denied / sandbox blocked を解消し、
FIRE 主要実装 wave で **Codex 8 lane 並列開発を再使用可能** な状態に戻す。
本 wave は **復旧確認のみ**、FIRE 本体の機能実装はしない。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + codex CLI 基本確認 (= which / version / help / 実行権限 / PATH) |
| 2 | Codex 設定 + repo trust 状態 read-only 確認 (= ~/.codex/config.toml) |
| 3 | permission denied 原因分類 A-H |
| 4 | codex review read-only smoke (= 最小 prompt 経由) |
| 5 | 可能なら 4-8 lane 並列 smoke |
| 6 | 失敗時の代替運用手順設計 |
| 7 | vault docs + HQ 1 ブロック + 6 KPI |

## §3 安全境界

- DB write / DB sqlite 接続 / LINE / token / .env / env 全体参照 / API call /
  launchctl / plist / cron / VACUUM / Paper Live Real / brokerage / 自動発注 /
  Computer Use / workflow / --no-verify / git push / sudo / rm -rf / TODO Excel 全 0
- ~/.codex/config.toml に secret らしきものが含まれていたら **読まずに停止 + HQ**
- Codex に上記禁止項目を触らせない
- danger-full-access 常用は避ける、workspace-write + on-request を基本
- F282 / 3 DB / W30 snapshot 完全不変維持

## §4 完了条件

- codex CLI permission denied の原因分類完了 (= A-H)
- 安全な修正可能なら実施済み (= 設定変更を伴わない場合は変更なし)
- codex --version / --help 確認可能
- FIRE repo で Codex read-only smoke 成功
- 可能なら 8 lane smoke 成功
- Codex を次 wave で使えるか / 使えないかが明確
- 使えない場合は代替運用手順完成
- F282 plist + 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

Codex に danger-full-access / network_access / token / secret / .env / env 全体 /
DB write / production-develop-staging 接続 / LINE / API / launchctl / plist / cron /
workflow / --no-verify / git push / sudo / rm -rf / TODO Excel / 楽天 / Computer Use
が必要 → 即停止 + HQ
