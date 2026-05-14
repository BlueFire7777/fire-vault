---
id: FIRE-CODEX-R1-WAVE60-pilot-D4-plan
phase: 本番 v0 中核 / Wave 60-pilot-D4 / D4 f062_preview Manual Live Pilot
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-19
pilot_day: D4
---

# Wave 60-pilot-D4 Plan — D4 f062_preview Manual Live Pilot Trade Plan v1.0

## §1 目的

2026-05-19 (火) D4 の少額手動実弾パイロット trade plan を作成。
D3 で確立した 4 step manual sequence を再実行し、**artifact_source=f062_preview**
の AFTER-R1 MVP 成果物を生成。**7 invariants hard check** で Pilot GO 判定。

D4 では特に **F111 朝 batch 稼働状況** と **f062_preview 候補の安定性** を確認。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + F111 朝 batch 稼働確認 |
| 2 | F111 advisory preview 用意 (= 朝 batch 未稼働なら D3 同 synth 流用) |
| 3 | DATA-R3 freshness 選択肢 A (= dry-run runner) |
| 4 | F062 LINE preview runner --dry-run --require-freshness-ok |
| 5 | AFTER-R1 MVP --mode mvp --task all |
| 6 | 7 invariants hard check |
| 7 | F111 input source 判定 (= real_batch / synthetic_sample / unknown) |
| 8 | Pilot GO/HOLD/NO-GO 判定 |
| 9 | D4 trade plan + review template 作成 |
| 10 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- 実発注 / 楽天 / iSPEED / Computer Use / 自動発注 0
- LINE 送信 / DB write / token / API / launchctl / plist / cron 0
- ファイル write は reports/after_r1/ + fire-vault/* + /tmp/ のみ
- FIRE 本体コード変更 0
- F282 plist + 3 DB 完全不変

## §4 完了条件

- D4 4 step manual sequence 実行完了
- D4 MVP artifact 4 種類生成 (= base_date=2026-05-19)
- artifact_source = f062_preview ✓
- f062_raw_kind = f062_actual_dict ✓
- F111 input source 判定 (= real_batch / synthetic_sample / unknown)
- 7 invariants hard check 全 PASS
- Pilot GO/HOLD/NO-GO 明確
- D4 trade plan 完成 (= F111 caveat + 値嵩株リスク明記)
- D4 review template 完成
- リスク上限明記
- F282 / 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token / API /
launchctl / plist / cron / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
