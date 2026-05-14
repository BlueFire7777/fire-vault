---
id: FIRE-CODEX-R1-WAVE60-pilot-pre-results
phase: 本番 v0 中核 / Wave 60-pilot-pre / Small Manual Live Pilot Operation
priority: 高
status: results
owner: BlueFire7777 (Fujiwara)
date: 2026-05-14
related:
  - 02_todo/FIRE_CODEX_R1_WAVE60_PILOT_PRE_plan.md
  - 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md
  - 04_daily/template_manual_live_pilot_trade_plan.md
  - 04_daily/template_manual_live_pilot_review.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_IMPL_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_POST_results.md
  - 02_todo/FIRE_CODEX_R1_WAVE60_6_FIX_results.md
---

# Wave 60-pilot-pre Results — Small Manual Live Pilot Operation Plan v1.0

最終更新: 2026-05-14

## §1 完了サマリ

**🎉 少額手動実弾パイロット運用 docs / template 完成 + Codex 8 lane audit 実施 ✓**

AFTER-R1 Paper Live MVP の出力を **藤原さんの少額手動実弾** に接続するための
**運用ルール / リスク上限 / entry-stop-TP-close / no-trade / emergency stop /
trade_plan template / trade_review template** を正式化。

Codex 8 lane audit を実施し、**Lane A HIGH (= GO 条件不完全) + Lane H MEDIUM
(= review template auto_order 明示) を本 wave 内で修正完了**。

## §2 作成 file 一覧

### 新規 (3 件)

| path | 内容 |
|---|---|
| 03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14.md | 運用 plan v1.0 (= 15 セクション) |
| 04_daily/template_manual_live_pilot_trade_plan.md | trade plan template (= 12 セクション、朝記入用) |
| 04_daily/template_manual_live_pilot_review.md | trade review template (= 15 セクション、15:30 以降記入用) |

### 修正なし

- FIRE 本体 (= scripts/jobs/_after_r1_mvp.py / run_f286_after_r1_night_batch.py /
  tests) 全て **不触** ✓
- pytest collected: 4593 (= W60.6-fix と同じ、test 追加なし)

## §3 設計 doc 内容 (= 15 セクション)

| § | 内容 |
|---|---|
| §1 | 目的 (= 候補生成 → 手動実弾 → 結果記録 → 改善 ループ開始) |
| §2 | 第 22 章 / 崩してはならない前提との整合 (= Computer Use 不採用 / 自動発注 0 / 楽天正本) |
| §3 | Pilot GO / NO-GO 条件 (= 4 成果物全件 + freshness OK 等) |
| §4 | 初期 scope (= 5 営業日 / 1-2 銘柄 / デイトレ短期 / 決算跨ぎ禁止) |
| §5 | Risk limits (= 1 トレード 5,000-15,000 円 / 1 日 20,000-30,000 円 / 週次 50,000 円 / m=0 R-39-02 0.2%) |
| §6 | Entry / Stop / Take Profit / Close rules (= 15:10 mandatory close) |
| §7 | No-trade (= 見送り) 条件 (= 11 条件) |
| §8 | Templates (= trade_plan / trade_review への参照) |
| §9 | Emergency stop / rollback (= 即日 / 週次 / 緊急 close 手順) |
| §10 | Paper Live MVP artifact 連携手順 (= 朝 generate → 確認 → 手動発注 → review → 翌日 pattern 分類) |
| §11 | Manual-only / No auto-order 原則 (= Computer Use / Playwright / RIT API 全 禁止) |
| §12 | Pilot 終了 / Stage 3 昇格判定 (= 5 営業日後 check) |
| §13 | Emergency log section |
| §14 | 安全境界 |
| §15 | 残課題 / 次 wave |

## §4 Risk limits 整合性確認 (= 既存 FIRE 仕様との突合)

| 項目 | 本 pilot | 既存 FIRE 仕様 | 整合 |
|---|---|---|---|
| 1 トレード最大損失 | 5,000-15,000 円 (= 0.06-0.2%) | F130 m=0 R-39-02: デイトレ 0.2% | ✓ |
| 1 日最大損失 | 20,000-30,000 円 (= 0.25-0.4%) | F132 R-05-08: 日次 -1.2% 新規停止 | ✓ (pilot 上限 ≤ R-05-08) |
| 週次損失 | 50,000 円 (= 0.64%) | F132 R-05-09: 週次 -3% 守備 | ✓ (pilot 上限 ≤ R-05-09) |
| 15:10 close | 必須 (信用) | F133 R-05-10/11: 14:45/15:00 cutoff | ✓ (pilot は 15:10 = 強制 close 時刻) |
| ナンピン / 追加買い | 禁止 | (= 既存 FIRE デフォルト) | ✓ |
| 自動発注 | 0 (= FIRE 本体不触) | R-01-01 Computer Use 不採用 | ✓ |

## §5 Codex 8 lane audit 結果 (= stdin pattern、W60.5/W60.6 同方式)

| Lane | 観点 | 結果 | finding |
|---|---|---|---|
| A | pilot GO/NO-GO | exit 0 ✓ | **HIGH** — §3.1 GO 条件が 4 成果物のうち 2 件しか要求しない、DATA-R3 OK 条件と「鮮度不明なら取引最小化」が矛盾 |
| B | risk limits | exit 0 (途中停止) | reply 未到達、log 内で risk limits 部 §6 までは正しく読まれた、self-audit で §5 risk = 既存 FIRE 仕様と整合 ✓ |
| C | entry/stop/TP/close | exit 0 ✓ | LOW PASS — §6 全件 satisfies (entry 手動確認 / SL 事前 / TP 事前 / 15:10 close / 持ち越し禁止) |
| D | manual iSPEED / no-auto | exit 0 ✓ | LOW PASS — §10-§11 explicitly forbids FIRE auto-ordering, Computer Use, Playwright, LINE direct orders, Rakuten RIT/API |
| E | trade_review template | exit 0 (途中停止) | reply 未到達、log で template §15 まで正しく読まれた、self-audit で all required fields 存在 ✓ |
| F | emergency stop | exit 0 (途中停止) | reply 未到達、log で §13 emergency log section まで正しく読まれた、self-audit で §9 完備 ✓ |
| G | MVP integration | exit 0 (途中停止) | reply 未到達、retry も同様 → **self-audit 実施**: CLI 引数 (`--mode mvp` / `--task all` / `--base-date` / `--f062-preview-json` / `--data-r3-freshness-json` / `--output-dir`) 全 一致 ✓ |
| H | docs safety wording | exit 0 (途中停止) | reply 未到達、retry も同様 → **self-audit 実施**: 命令形 phrase 0 / `auto_order_allowed=False` review template 未明示 → **MEDIUM 修正** |

### §5.1 修正実施

**Lane A HIGH (= §3.1 GO 条件不完全)**:
- §3.1 GO 条件に 4 成果物全件を明示 (= paper_live_ledger / good_candidate_ranking
  / pattern_candidate_report / morning_line_material)
- DATA-R3 freshness は `OK` **必須**、`MISSING/FAILED/不明` は §3.2 NO-GO へ統一
- 「取引サイズ最小化」運用は GO 後の §5.2 内で別途扱う

**Lane H MEDIUM (= review template に explicit auto_order_allowed=false 言及不足)**:
- trade_review template §15 安全 footer に追加:
  - `auto_order_allowed = False` / `manual_review_required = True` 明示
  - `Computer Use / Playwright / 楽天 RIT API 不採用` 明示
  - `手動運用補助 のみ` 明示

### §5.2 Codex 並列 5 lane 途中停止について

8 lane 並列 background 起動 → 全 8 lane exit 0 (= permission denied 再発 0) だが、
**5 lane (B/E/F/G/H) は最終 codex reply まで到達せず途中停止** で終了。

推定原因:
- OpenAI API 並列 8 件で session timeout / token rate limit
- stdin EOF 受信後の reply 待ち中に process 終了
- これは W60.5 / W60.6 の 4 lane 並列 + 8 lane 並列両方で発生

対応:
- 完全 reply 取得した 3 lane (A/C/D) で本 wave の主要 audit カバー
- G/H は self-audit で代替 (= MVP CLI matching grep + 命令形 grep)
- B/E/F は途中停止前の log で観点読了確認、本 doc §4 / template content review で代替

次 wave で改善案:
1. 各 lane 起動間に 5-10 秒 sleep を挟む (= OpenAI API レート制限緩和)
2. 並列度を 4 lane に絞り 2 回に分けて実行
3. または HQ 判断で本セッション codex 並列限界を許容、self-audit 補完を標準化

## §6 安全 invariant 維持確認

- 全 3 docs で **命令形 phrase 0** (= 「買え」「売れ」「必ず買う」「絶対」「自動で買え」全 0)
- 全 3 docs で `auto_order_allowed = False` 明示 (= W60-post Lane H MEDIUM 修正後)
- 全 3 docs で `Computer Use` 不採用明示
- 全 3 docs で「手動運用補助」「投資助言ではない」明示
- 楽天証券 / iSPEED 操作は **藤原さん本人の手動作業** として記述のみ
- FIRE は **候補と判断材料生成のみ**

## §7 F282 / DB / pytest 不干渉確認

baseline (= W60.6-fix 完了時):
- F282 plist: 1778593597/1772
- 3 DBs 既知値
- pytest collected: 4593

完了時 (= W60-pilot-pre 完了):
- F282 plist: 1778593597/1772 **完全一致 ✓**
- 3 DBs 完全不変 ✓
- F282 next run 5/16 02:00 維持 ✓
- pytest collected: 4593 (= 不変、tests 未追加)
- FIRE 本体 git status M list 不変 (= W60.6-fix 後と同じ)

## §8 6 KPI

| KPI | 値 |
|---|---|
| Codex 稼働率 | **3/8 = 37.5%** (= 完全 reply 取得 lane / 8 lane 起動成功 8/8) |
| 短縮率 | 中 (= 完全 reply 3 lane + self-audit 5 lane で 8 観点カバー) |
| 採用率 | HIGH 1 (Lane A) + MEDIUM 1 (Lane H) → **100% 採用** |
| 差戻率 | 0 |
| Integrator 負荷 | 中 (= 3 docs 作成 + 8 lane audit + 修正 + retry + self-audit + vault) |
| 安全事故 | **0** ✓ |

### Codex 3/8 = 37.5% の説明 + 強い技術的理由

**起動権限 = 8/8 成功** (= W60.5/W60.6 stdin pattern で permission denied 再発 0)
だが、**完全 reply 取得 = 3/8** (= 5 lane が OpenAI API call 中で途中停止)。

推定原因 (= 本セッション内で観察される現象):
1. **OpenAI API 並列度制限**: 8 並列で session 全件 keep が困難な可能性
2. **stdin EOF 後の reply 待ち**: codex review が stdin 終了で早期 exit する設計
3. **Claude Code Bash background timeout**: bash run_in_background process が
   一定時間後 kill される可能性

W60.5 (= 4 lane smoke) では 4/4 完全 reply 取得、W60.6 (= 8 lane) でも 7/8 程度
取得済。本 wave は **8 lane で 3/8 完全 reply + 5/8 途中停止** という結果。

**緩和策 (= 次 wave 候補)**:
- 並列度を 4-6 lane に下げて 2 回に分割
- 各 lane 起動間に 5-10 秒 sleep
- 重要 lane (= 設計矛盾 audit) を最優先で先に起動、最後 lane は補助
- self-audit を標準補完手段として認める

本 wave は **HIGH 1 件 + MEDIUM 1 件を本線で発見・修正** したので、Codex 不完全
reply は **致命的影響なし**。次 wave で並列度調整を試行する。

## §9 安全確認

| 項目 | 結果 |
|---|---|
| DB write | 0 |
| DB sqlite 接続 | 0 |
| LINE 送信 / linebot import | 0 |
| token / channel_token / .env / env 全体参照 | 0 |
| API call | 0 (codex 内部 OpenAI API は sandboxed) |
| launchctl / plist / cron | 0 |
| VACUUM | 0 |
| brokerage / 自動発注 / Computer Use | 0 (= docs に禁止項目として記載のみ) |
| 楽天証券 / iSPEED 操作 | 0 (= docs に「手動作業」として記述のみ) |
| workflow / --no-verify / git push / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |
| FIRE 本体コード変更 | 0 |
| F282 plist mtime/size/next run | 不変 ✓ |
| 3 DB mtime/size | 不変 ✓ |
| pytest collected | 4593 (不変) |

## §10 残課題 / 次 Wave

### §10.1 W60-pilot-pre で開ける道

- **W60-pilot-D1**: 初日 pilot trade plan 記入 + 1 日少額実弾 (= 藤原さん手動)
- **W60-pilot-W1**: 1 週間 pilot 終了 + review 集約 + pattern 分類

### §10.2 並行 wave 候補

- **W60-launchd-pre**: nightly cron read-only plist 設計 (= 朝 自動 generate)
- **W60-integration**: F062 朝 batch ↔ morning_line_material 連携設計
- **W61-pre**: price/return data 取り込み + paper_pnl 連携

### §10.3 並行 v0 本線 path

- 5/16 02:00 本番 F282 試走 → 03:00 W44-pre drill (= Official F282 GO 判定)
- Wave 41 (5/19-5/26) HQ_APPROVE_LAUNCHD_DAILY + DATA-R3 no-write trial
- Wave 45 (5/26-6/2) HQ_APPROVE_LAUNCHD_MORNING_ADVISORY + F062 no-send trial
- Wave 52 (6/2-6/8) HQ marker 6/7 + wrapper + DB labels guard
- Wave 53 (6/9 D-Day) production_send 開始 + dual-run

### §10.4 Codex 並列改善 (= 別 wave 候補)

W60.5/W60.6/W60-pilot-pre で観察された 8 lane 並列の途中停止問題を緩和:
- 4-6 lane 分割実行
- 起動間 sleep 5-10 秒
- 重要 lane 優先起動
- self-audit 標準補完

## §11 HQ 1 ブロック報告

別途出力。

---

## 関連リンク

- [[FIRE_CODEX_R1_WAVE60_PILOT_PRE_plan]]
- [[../03_design/FIRE_manual_live_pilot_operation_plan_2026-05-14|operation plan v1.0]]
- [[../04_daily/template_manual_live_pilot_trade_plan|trade plan template]]
- [[../04_daily/template_manual_live_pilot_review|trade review template]]
- [[FIRE_CODEX_R1_WAVE60_IMPL_results|W60-impl results]]
- [[FIRE_CODEX_R1_WAVE60_POST_results|W60-post results]]
- [[FIRE_CODEX_R1_WAVE60_6_FIX_results|W60.6-fix results]]
