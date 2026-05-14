---
id: FIRE-CODEX-R1-WAVE60-PILOT-D14-plan
phase: 本番 v0 中核 / Wave 60-pilot-D14 / 初回少額実弾開始判定 day
priority: 最高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-06-02
pilot_day: D14
hq_markers:
  - HQ_APPROVE_STAGING_JQUANTS_DAILY_REFRESH=1
  - HQ_APPROVE_JQUANTS_TOKEN_READ=1
design_doc: ~/fire-vault/03_design/FIRE_manual_live_pilot_D14_entry_criteria_2026-06-01.md
---

# Wave 60-pilot-D14 Plan — D14 First Small Manual Live Entry Decision v1.0

## §1 目的

W2 で確定した 4 段階 criteria に基づき、**初回少額実弾開始可否を判定**。D14 は
「開発」ではなく **実弾開始判定 wave**。340A0 / 3798 / 137A0 / 7991 の 4 候補から、
朝寄付き actual + liquidity + event の 3 確認後、Fujiwara が手動 entry 判断する。

## §2 安全境界

- production / develop DB 接続: **禁止**
- staging DB: **read-only 基本**、focused refresh 時のみ staging write
- token 値表示 / env 全体表示: **禁止**
- API call: J-Quants daily_quotes の D14 候補 focused refresh に限定
- LINE / launchctl / plist / cron / VACUUM / workflow / --no-verify / git push /
  sudo / rm -rf / 実発注 / 楽天 / iSPEED / Computer Use / TODO Excel: **禁止**

## §3 構成

| step | 内容 |
|---|---|
| 1 | baseline (F282 plist + 3 DB md5) + D13 review 確認 |
| 2 | J-Quants focused refresh 試行 (= dry-run → 実 write or skip) |
| 3 | F111-real-batch + F062 + AFTER-R1 + DATA-R3 + paper PnL chain |
| 4 | hard check 12 項目 |
| 5 | 4 段階 criteria 判定 (= GO/GO_CONDITIONAL/HOLD/NO-GO) |
| 6 | D14 trade plan + review template (= 最低 5 項目必須記入) |
| 7 | paper PnL handoff (= 場後 + 翌朝 + h20 後) |
| 8 | vault plan + results + HQ 1-block + 6 KPI |

## §4 完了条件

- D13 review 確認完了 (= status blank 確認)
- J-Quants focused refresh 試行完了 (= 実行 or skip 理由明記)
- chain 完走 (= F111 + F062 + AFTER-R1 + DATA-R3 + paper PnL)
- hard check 12 項目 全 ✓
- 4 段階 criteria 判定完了
- D14 trade plan + review template 作成完了
- D14 review 最低 5 項目必須記入 明記
- paper PnL handoff 手順明記
- production / develop DB 接続 0
- staging 変更があれば md5 + row 数報告
- LINE 0 / token 値表示 0 / env 全体表示 0
- launchctl 0 / plist 0 / cron 0
- 実発注 / 楽天 / iSPEED / Computer Use 0
- F282 plist mtime/size/next run 不変
- HQ 1-block + 6 KPI

## §5 停止条件

15 項目のいずれかが必要になったら **即停止 + HQ 確認**:
production-develop 接続 / token 値表示 / env 全体表示 /
J-Quants 以外 API / LINE / launchctl / plist / cron / 実発注 / 楽天 /
iSPEED / Computer Use / workflow / --no-verify / git push / sudo /
rm -rf / TODO Excel

## §6 Codex 運用

実弾開始判定 wave = 本線主導。code 変更 0 → Codex skip 妥当。W2 集約や
ロジック変更時に 4 lane 以上を使う。
