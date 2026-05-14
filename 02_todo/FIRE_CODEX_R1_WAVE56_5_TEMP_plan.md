---
id: FIRE-CODEX-R1-WAVE56.5-temp-plan
phase: 本番 v0 Launch / Wave 56.5-temp / F119 dry-run triage
priority: 高
status: plan
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
---

# Wave 56.5-temp Plan — F119 Dry-Run Compatibility Triage / DATA-R3 Recovery

## §1 目的
W56-temp の f119_evaluation dry-run failure を切り分け、安全に解消可能なら最小修正し、
DATA-R3 temp smoke を Engineering GO に近づける。本物 launchctl 再実行は原則不実施。
DB write / API / token / 本番 plist には触らない。

## §2 構成 (= Codex lane 0/8 で本線完結)
本 wave は CRITICAL 解析 + 局所修正 + tests のため、W51 / W52-pre / W54-pre / W55-temp / W56-temp
と同じ「prescriptive 本線完結」パターンで Codex 0/8。

## §3 実施項目
1. baseline + W56-temp artifacts 再確認
2. F119 dry-run failure 原因特定 (= reproduce + log 解析)
3. A/B/C/D 分類
4. 最小修正適用 (= 該当時のみ)
5. tests 追加
6. **launchctl を使わない直接 dry-run subprocess で動作再確認**
7. Engineering GO/HOLD/NO-GO 判定
8. vault に W56.5-temp plan / results

## §4 完了条件
- F119 dry-run failure 原因分類完了
- 安全に修正可能なら最小修正済 + tests PASS
- pytest collected 数報告
- DATA-R3 expected_sequence 維持
- DB write / DB sqlite 接続 / LINE / token / API / launchctl / plist / cron 0
- F282 本番 plist mtime/size/next run 不変
- DATA-R3 / F062 本番 plist 未配置/未load 維持
- 3 DB mtime/size 不変
- vault に W56.5-temp 報告 + HQ 1 ブロック + 6 KPI
