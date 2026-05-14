---
id: FIRE-CODEX-R1-WAVE60-pilot-D3-plan
phase: 本番 v0 中核 / Wave 60-pilot-D3 / D3 f062_preview Manual Live Pilot
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-18
pilot_day: D3
---

# Wave 60-pilot-D3 Plan — D3 f062_preview Manual Live Pilot Trade Plan v1.0

## §1 目的

2026-05-18 (月) D3 の少額手動実弾パイロット trade plan を作成。
W60-launchd-pre 確立の **4 step manual sequence** を実行し、
**artifact_source=f062_preview** の AFTER-R1 MVP 成果物を生成。
**7 invariants hard check** で Pilot GO 判定後、藤原さんが入る/見送る判断。

## §2 構成

| step | 内容 |
|---|---|
| 1 | baseline + W60-launchd-pre fixture 確認 |
| 2 | F111 advisory preview 用意 (= /tmp/fire_d3_prep/、朝 batch 未稼働なら synth) |
| 3 | DATA-R3 freshness 選択肢 A (= dry-run runner) で OK 確認 |
| 4 | F062 LINE preview runner --dry-run --require-freshness-ok |
| 5 | AFTER-R1 MVP --mode mvp --task all --output-dir reports/after_r1 |
| 6 | 7 invariants hard check |
| 7 | Pilot GO/HOLD/NO-GO 判定 |
| 8 | D3 trade plan + review template 作成 |
| 9 | vault plan + results + HQ + 6 KPI |

## §3 安全境界

- 実発注 / 楽天 / iSPEED / Computer Use / 自動発注 0
- LINE 送信 / DB write / token / API / launchctl / plist / cron 0
- 楽天 / iSPEED 言及は禁止項目として記述のみ
- ファイル write は reports/after_r1/ + fire-vault/* + /tmp/ のみ
- FIRE 本体コード変更 0
- F282 plist + 3 DB 完全不変

## §4 完了条件

- D3 4 step manual sequence 実行完了
- D3 MVP artifact 4 種類生成 (= reports/after_r1/、base_date=2026-05-18)
- artifact_source = f062_preview ✓
- f062_raw_kind = f062_actual_dict ✓
- 7 invariants hard check 全 PASS
- Pilot GO/HOLD/NO-GO 明確
- D3 trade plan 完成 (= F111 synth caveat 明記)
- D3 review template 完成
- top 1-3 整理 + D3 1 銘柄推奨 or skip 推奨
- リスク上限明記
- F282 / 3 DB 不変
- vault に plan + results + HQ + 6 KPI

## §5 停止条件

実発注 / 楽天 / iSPEED / Computer Use / DB write / LINE / token / API /
launchctl / plist / cron / workflow / --no-verify / git push / sudo / rm -rf /
TODO Excel が必要 → 即停止 + HQ
