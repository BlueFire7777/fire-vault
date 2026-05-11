---
id: FIRE-LABEL-R1
phase: P5: 通知 / 第 14 章 LINE 通知配信 / Advisory UX
priority: 高
status: Wave 4 W4-3 着手準備 (= 2026-05-11、HQ approve 受領、Codex 投入待ち)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F286-PNL-R2 (= snapshot decision_label 列を共有)
  - F062-R5.7 (= action mode 旧ラベル実装)
  - F062-R5.8 (= 既送信 LINE 文面と整合)
chapter: 第 14 章 / 第 19 章 R-19-08 / Advisory UX
---

# FIRE-LABEL-R1: Advisory / LINE UX Label Refresh

最終更新: 2026-05-11

## 起票背景

FIRE-CODEX-R1 v1.1 Wave 2 終盤 (= 2026-05-11) に HQ から **FIRE
Advisory / LINE UX 表示ラベル方針変更** を受領。Wave 2 / Wave 3 の
scope を拡張せず、新ラベル方針対応を分離タスクとして起票。

## HQ 受領: 新ラベル方針 (= 5 段階)

| 新ラベル | 意味 |
|---|---|
| 🟢 積極的買い推奨   | 最上位、条件が崩れていなければ手動買い優先可 |
| 🟡 条件付き買い推奨 | VWAP 上 / 出来高増 / 地合い悪化なし 等の条件成立で買い候補 |
| 🟠 場中監視         | まだ買わない、初押し・再加速・出来高確認待ち |
| ⚠️ 注意つき買い候補 | 上昇余地はあるが、ボラ・決算・需給に注意 (新規 bucket) |
| 🔴 見送り推奨       | 今日は触らない方がよい |

## 旧 → 新 mapping (= 確定)

| 旧 (F062-R5.7 action mode) | 新 (FIRE-LABEL-R1) |
|---|---|
| 🟢 買い候補                | 🟢 積極的買い推奨 |
| 🟡 条件付き買い            | 🟡 条件付き買い推奨 |
| 🟠 待ち                    | 🟠 場中監視 |
| (新規)                     | ⚠️ 注意つき買い候補 |
| 🔴 見送り                  | 🔴 見送り推奨 |
| ⚪ 監視のみ                | (継続、ラベル名のみ調整余地) |

## 言葉遣い禁止 (= HQ 受領)

- 「買え」「今すぐ買え」のような **命令口調禁止**
- 既存 FORBIDDEN_PHRASES (notifications/templates/research_advisory.py)
  には「買え」が既に登録済 ✓
- 新ラベル実装時に「今すぐ買い」「今すぐ買え」等が混入しないか
  test で追加検査

## 影響範囲

### notifications/templates/research_advisory.py
- `ACTION_LABEL_BADGE` (dict): 5 ラベル絵文字 + テキスト更新
- `ACTION_LABEL_VERDICT` (dict): 判定行テキスト更新
- `ACTION_FLIP_CONDITION` (dict): 「買いに変わる条件」を新ラベル軸で再構成
- `_ACTION_BUCKET` (dict): label → bucket mapping は維持 (= bwa → wait
  等の構造、ただし新たに「注意つき買い候補」bucket を追加)
- 結論行: `format_action_conclusion()` の出力文言を新ラベルに更新
- 既存 `_BUY_LABELS` / `_AVOID_LABELS` 集合: 新ラベル名で再評価
- 新規 bucket: `⚠️ 注意つき買い候補` = ボラ / 決算 / 需給注意 の条件付き
  推奨、boost_with_caution の上位 case を分離する候補

### pnl/snapshot.py
- `_ACTION_VERDICT` (dict): 上記と完全同期 (= snapshot decision_label が
  LINE 表示と同じ文字列を保持)

### tests
- tests/notifications/templates/test_research_advisory_line_template.py:
  TestActionFirstAdvisoryUX 全 16 件の assertion を新ラベルで更新
- tests/scripts/jobs/test_run_f062_research_advisory_line_preview.py:
  TestF062R57RunnerActionMode 全 3 件の assertion 更新
- tests/pnl/test_snapshot.py: action_mode 変換 test の expected 更新

### 既送信 LINE との整合性
- F062-R5.8 (= 2026-05-11 18:19 JST 送信、`待ち 5 件 / 見送り 0 件`)
  と新文面を Fujiwara が見比べた時の混乱を防ぐため、log.md に
  「label refresh 切替日」を decision entry で記録

## 設計判断ポイント (= 未確定)

1. **新規 bucket 「⚠️ 注意つき買い候補」の F119 ラベル mapping**:
   - 案 a: 既存 `boost_with_caution` を「注意つき買い候補」に再分類
   - 案 b: 新ラベル用に F119 側の `caution_flags` を読んで動的に分類
   - 案 c: 暫定で `boost_with_caution` をそのまま「注意つき買い候補」
     扱い、F119 拡張は別タスク
   - 推奨: 案 c (= 最小変更で開始)

2. **mode 切替 / migration**:
   - 案 X: `--label-version v1 | v2` option を追加、旧 / 新を選べる
   - 案 Y: 即時全面切替 (= mode option なし)
   - 推奨: 案 Y (= シンプル、旧ラベルは vault 記録のみ保持)

3. **F286-PNL-R2 advisory_decisions / advisory_snapshot_rows の
   既存 row 扱い**:
   - 案 P: 既存 row (= F286-PNL-R1 smoke で書いた 5 件、旧ラベル
     `watched` 等) はそのまま保持
   - 案 Q: data migration で旧→新 ラベル一斉変換
   - 推奨: 案 P (= 既存値保持、新規 ingest からは新ラベル文字列)
   - 注: `fujiwara_decision` (= adopted/watched/skipped/rejected/unknown)
     は別概念 (= Fujiwara 判断 status)、本タスク変更対象外

## 実装計画 (= Codex 分割案)

| sub | lane | task |
|---|---|---|
| sub-1 | L1 Design       | 新ラベル mapping + ⚠️ bucket 設計確定 + mode 切替判断 |
| sub-2 | L3 Implementation | notifications/templates/research_advisory.py 全 ACTION_* dict 更新 + 結論行 |
| sub-3 | L3 Implementation | pnl/snapshot.py _ACTION_VERDICT 更新 |
| sub-4 | L2 Test         | 全関連 test (= template + runner + snapshot) を新ラベルで更新 |
| sub-5 | L4 Audit        | 新ラベルが「買え」「今すぐ買え」命令口調を含まないか / 旧
                          ラベル文字列が code から完全除去されたか ast/grep audit |
| sub-6 | L5 Docs         | vault docs 更新 (= F062-R5.7 / R5.8 / Wave 1-3 に新ラベルへの
                          切替日 note 追加、本書を完了マーカー化) |

## 安全要件 (= 起票時点で 0 違反)

- LINE 本番送信なし (= 本タスクは template / module 変更のみ、smoke は
  別途 HQ approve 後)
- DB write なし
- token / channel_token / secret 参照なし
- workflow 変更なし / --no-verify 不使用
- scripts/seed_pattern_layer1.py / historical_indicators.py 未接触
- TODO Excel 未更新
- 注文価格 / 数量 / 執行指示の生成 helper を含めない (= 既存禁止維持)

## 次タスク状態

**Wave 4 W4-3 着手準備済** (= 2026-05-11 HQ approve)、Codex 投入 HQ
approve 待ち。

### Wave 4 W4-3 で Codex に投入する想定

| 段階 | 内容 | lane | 想定時間 |
|---|---|---|---|
| Codex L3 Impl | notifications/templates/research_advisory.py 更新 (= ACTION_LABEL_BADGE / ACTION_LABEL_VERDICT / ACTION_FLIP_CONDITION / 結論行) + pnl/snapshot.py の `_ACTION_VERDICT` 同期 | L3 | 10-15 分 |
| Codex L2 Test | tests/notifications/templates/test_research_advisory_line_template.py + tests/scripts/jobs/test_run_f062_research_advisory_line_preview.py + tests/pnl/test_snapshot.py の expected 文字列を新ラベルに更新 | L2 | 10-15 分 |
| 本線 Integrator review | 全 pytest 回帰 + 新ラベル / 旧ラベル混在検査 (= grep) + 命令口調検査 | (本線) | 10 分 |
| commit | feat(FIRE-LABEL-R1): refresh advisory label vocabulary (= 5 新ラベル) | (本線) | 5 分 |

W4-1 (F062 send_smoke) / W4-2 (ingest helper) との衝突回避: W4-3 が
触る `notifications/templates/research_advisory.py` は W4-1 / W4-2 の
allowed_files に **入っていない** (= 衝突なし)。pnl/snapshot.py の
`_ACTION_VERDICT` 更新も W4-1 / W4-2 が触らないため衝突なし。

### 設計判断 (= 本線仮承認、HQ approve 待ち)

1. **新規 ⚠️ bucket の mapping**: 案 c 推奨 (= 暫定で
   boost_with_caution を「注意つき買い候補」扱い、F119 拡張は別タスク)
2. **mode 切替**: 案 Y 推奨 (= 即時全面切替、mode option なし、
   旧ラベルは vault 記録のみ保持)
3. **既存 advisory_decisions row 扱い**: 案 P 推奨 (= 既存値保持、
   新規 ingest からは新ラベル)

これら 3 件は Codex 投入時のプロンプトに HQ approve 結果を反映する。

## 関連リンク

- [[../03_design/FIRE_CODEX_R1_multi_lane_parallel_orchestration_2026-05-11|FIRE-CODEX-R1 v1.1]]
- [[F286_PNL_R2_advisory_snapshot_auto_ingest|F286-PNL-R2 (= 旧ラベルで保存)]]
- [[F062_R5_8_action_first_production_send|F062-R5.8 (= 旧ラベル送信実績)]]
- [[FIRE_CODEX_R1_WAVE3_audit_results|Wave 3 audit 結果]]
- [[../log]]
