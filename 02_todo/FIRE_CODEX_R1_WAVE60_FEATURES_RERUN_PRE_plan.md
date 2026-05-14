---
id: FIRE-CODEX-R1-WAVE60-FEATURES-RERUN-PRE-plan
phase: 本番 v0 中核 / Wave 60-features-rerun-pre / 候補更新力強化 設計 wave
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
hq_marker: なし (= 本 wave は調査 + 設計のみ、staging write/API ゼロ)
---

# Wave 60-features-rerun-pre Plan — Research Features / Signal Rerun Approval Design v1.0

## §1 目的

D6/D7/D8 で top 3 = 8747/5729/3489 が 100% 重複した features cap 問題を切り
分け、`F111-real-batch` の候補を日次で変化させるための features rerun /
research signal update の **安全計画 + 承認境界 + 次 Wave 実行案** を作る。

本 wave は **調査 + 設計 + dry-run 可否確認のみ**。staging write / API call /
launchd 変更は実行しない (= 次 Wave 用 HQ marker 案を出すだけ)。

## §2 安全境界

- production / develop DB 接続: **禁止** (md5 不変確認で verify)
- staging DB: **read-only 調査のみ** (write は今回実行しない)
- API call / token / .env / env 全体参照: **禁止**
- launchctl / plist 配置 / cron 変更 / VACUUM: **禁止**
- LINE 送信 / workflow 変更 / --no-verify / git push / sudo / rm -rf: **禁止**
- 実発注 / 楽天 / iSPEED / Computer Use: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | D6/D7/D8 重複原因の features 観点切り分け |
| 2 | research_watchlist_signals upstream source map 作成 |
| 3 | staging read-only で各 table の最新 base_date / row count 確認 |
| 4 | rerun runner 探索 + safety args (--dry-run/--write/FIRE_ENV) 確認 |
| 5 | rerun 経路の A/B/C/D/E 分類 |
| 6 | 必要 HQ marker 案 + 停止条件 |
| 7 | D8/D9 候補更新方針 (= 短期 A 経路 / 中期 C 経路) |
| 8 | Codex 4 lane factual-confirm audit (= 80 words / lane) |
| 9 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- 重複原因が A〜G で分類済み
- upstream source map 完成
- features freshness staging-only 確認済み
- rerun runner 有無 + 安全引数 確認済み
- rerun 可否 A/B/C/D/E 分類済み
- 必要 HQ marker 案 完成
- D9 候補更新 next-wave 案 完成
- Codex 4 lane factual-confirm audit 実施 or 強い理由明記
- DB write 0 / production-develop 接続 0 / staging write 0 / LINE 0 /
  token 0 / API 0 / launchctl 0 / plist 0 / cron 0 / 実発注 0
- F282 plist + 3 環境 DB mtime/size 不変
- fire-vault に plan + results
- HQ 1-block + 6 KPI

## §5 停止条件

以下 16 項目のいずれかが必要になったら **即停止 + HQ 確認**:

1. staging DB write
2. production / develop DB 接続
3. token / secret / channel_token / .env / env 全体参照
4. API call
5. schema migration
6. LINE 送信
7. launchctl 実行
8. plist 配置 / 変更
9. cron / crontab 変更
10. 実発注
11. 楽天証券 / iSPEED 操作
12. Computer Use
13. workflow 変更
14. --no-verify
15. git push / sudo / rm -rf
16. TODO Excel 更新

## §6 Codex 運用

- 4 lane factual-confirm 方式
- stdin pipe pattern: `echo "<prompt>" | codex review -`
- 各 prompt = 80 words 以内、1 観点、1〜2 file 確認に限定
- DB write / LINE 送信 / token / launchctl / 楽天操作 は **Codex に絶対させない**
- reply 未取得 lane は self-audit で補完
