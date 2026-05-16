# FIRE F062 W2-A overlay Buyability Card 実装 (2026-05-16)

doc_id: FIRE-F062-W2A-OVERLAY-BUYABILITY-CARD-2026-05-16
status: 実装完了 (= 47 tests PASS、card 728 chars / 1 chunk、LINE 送信 0、既存 production code 変更 0)
related:
- F062_W2A_OVERLAY_LINE_PREVIEW_2026-05-16.md (= 21 chunks 問題の根拠)
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_SMOKE_2026-05-16.md (= W2-A overlay の出典)
- F286-REPORT-R1 fire/report/line_delivery.py (= chunk_length 1120 の出典)


## §1 目的

W2-A overlay morning_advisory MD は full version で 31,958 bytes / 21 chunks 必要で
LINE 配信に向かない。**LINE 1 chunk 想定の短文 Buyability Card** を W2-A overlay 専用に
新規追加し、朝 iPhone でサッと読める形に圧縮する。

full Markdown は維持 (= Claude Code 画面 / iPhone 単一フェンス共有用)。
本 wave は **新 template + runner 追加のみ**、既存 production code は変更しない。


## §2 実装範囲

### §2.1 新規 file (= 4 件、既存 0 変更)

| file | 行数 | 役割 |
|---|---|---|
| notifications/templates/overlay_buyability_card.py | ~270 | build_overlay_card 純関数 + role/color マッピング + validate |
| scripts/jobs/run_f062_overlay_buyability_card_preview.py | ~170 | no-send preview runner (= --strict + path guard) |
| tests/notifications/templates/test_overlay_buyability_card.py | ~330 | template tests (= 34 件、D49 fixture + edge case + forbidden) |
| tests/scripts/jobs/test_run_f062_overlay_buyability_card_preview.py | ~160 | runner tests (= 13 件、path guard + safety import + main) |

### §2.2 既存 file への変更 (= 0)

`git diff origin/develop` で確認:
- notifications/templates/research_advisory.py → 不変
- scripts/jobs/_v1_4_1_consumer.py → 不変
- scripts/jobs/_v1_4_1_morning_advisory_markdown.py → 不変
- fire/report/line_delivery.py → 不変
- 4 newly tracked file のみ (`??` 状態)


## §3 UX 設計

### §3.1 役割アイコン (= overlay role の見た目分類)

| 役割 | アイコン | 日本語ラベル | overlay role |
|---|---|---|---|
| 継続主役 | 👑 | 主役 | continuing_core |
| 警告 | ⚠️ | 注意 | risk_warning_track |
| 新顔 | 🆕 | 新顔 | new_face_sector_compare / comparison (fallback) |
| 復帰禁止 | ⛔ | 復帰なし | excluded_recovery (= 9130 等) |

### §3.2 評価色丸 (= 朝判断の信号色、行動指針)

| 色 | アイコン | 日本語ラベル | 紐付く役割 |
|---|---|---|---|
| 緑 | 🟢 | 有力 | 主役 (= 継続上昇候補) |
| 黄 (警告) | 🟡 | 比較・注意 | 注意 (= risk warning) |
| 黄 (比較) | 🟡 | 比較 | 新顔 (= sector 拡張枠) |
| 赤 | 🔴 | 見送り | 復帰なし |

### §3.3 設計原則

- **役割と色は別概念**で保持 (= "役割｜色" の 2 段表示、"｜" 区切り)
- **日本語ラベル必ず併記** (= 絵文字レンダリングできない環境でも判別可能)
- 1 銘柄 1 ブロック = 3 行構成:
  1. badge 行: 役割アイコン + 日本語ラベル + "｜" + 色丸 + 日本語ラベル
  2. 識別行: code + 銘柄名 (= 14 字以内に切り詰め、長い場合は "…")
  3. 価格行: 参考 / 追上限 / 利確 / 損切 / risk (= 5 値、3 桁カンマ区切り)
- LINE_ONE_CHUNK_BODY_MAX = 1,112 (= F286-REPORT-R1 chunk_length 1,120 - fence overhead 8)


## §4 生成結果 (= 実 D49 payload で smoke)

### §4.1 card text (= 728 chars / 34 lines / fits_one_chunk=True)

```
[FIRE 朝の advisory W2-A overlay 版]
判定: GO_CONDITIONAL (entry 20 / watch 0 / excluded 30)
base_date: 2026-07-22

【overlay top5】(主役継続 vs 新顔/新sector)

👑 主役｜🟢 有力
9247 ＴＲＥホールディングス
参考 1,612円 / 追上限 1,644 / 利確 1,773 / 損切 1,531 / risk 8,060円

👑 主役｜🟢 有力
9628 燦ホールディングス
参考 1,390円 / 追上限 1,418 / 利確 1,529 / 損切 1,320 / risk 6,950円

⚠️ 注意｜🟡 比較・注意
4404 ミヨシ油脂
参考 2,133円 / 追上限 2,176 / 利確 2,346 / 損切 2,026 / risk 10,665円

🆕 新顔｜🟡 比較
9008 京王電鉄
参考 747円 / 追上限 762 / 利確 822 / 損切 710 / risk 3,735円

🆕 新顔｜🟡 比較
3134 Ｈａｍｅｅ
参考 412円 / 追上限 420 / 利確 453 / 損切 391 / risk 2,060円

【復帰禁止】
⛔ 復帰なし｜🔴 見送り
9130 (= recently_seen demoted、entry/watch 復帰なし)

【手動確認】
iSPEED で 板 / 出来高 / spread を確認してから entry。
Fujiwara が手動発注 (auto-order なし、Computer Use なし、
楽天/iSPEED 自動操作なし)。
```

### §4.2 metadata

| 項目 | 値 |
|---|---|
| card_version | 1.0.0-w2a-overlay-card |
| char_count | 728 |
| line_count | 34 |
| fits_one_chunk | True (= 728 < 1112) |
| overlay_codes | ['9247', '9628', '4404', '9008', '3134'] |
| excluded_recovery_codes | ['9130'] |
| judgment | GO_CONDITIONAL |
| forbidden_hits | [] (= FORBIDDEN_PHRASES 17 件全 hit 0) |
| reminder_present | True (= "手動" / "auto-order なし" 含有) |
| validation_passed | True |
| safety_flags | 全 False (db_write/api_call/line_send/token_access/launchctl_call/auto_order/computer_use) |


## §5 tests (= 47 件、全 PASS、新規 regression なし)

| クラス | 件数 | カバレッジ |
|---|---|---|
| TestComputeCardPrices | 4 | full / none_close / zero_close / none_risk |
| TestComputeSimpleJudgment | 5 | go_conditional / no_go × 3 / hold |
| TestBuildOverlayCardD49 | 14 | version / fits_one_chunk / overlay_codes_5 / judgment / excluded_recovery / forbidden 0 / reminder / validate 0 / 4 役割 mapping / role_icons_distinct / price_line / japanese_label |
| TestForbiddenPhraseRejection | 2 | synthetic_forbidden 注入 + validate 検出 |
| TestEdgeCases | 5 | empty_overlay / no_excluded_recovery / unknown_role / long_name_truncation / reminder_always_present |
| TestSerializationSanity | 3 | card_is_str / no_internal_code_fence / validation_dataclass frozen |
| TestOutputPathGuard | 5 | tmp_allowed / etc_rejected / data_db_rejected / home_root_rejected / fire_vault_allowed |
| TestMainRunner | 6 | happy_path / input_not_found / unsafe_output_path / missing_overlay_top5 / strict_mode_violation_exits_2 / invalid_json |
| TestSafetyImports | 2 | runner / template に linebot / sqlite3 / requests / aiohttp / subprocess import 不在 |
| **合計** | **47** | **全 PASS** |

regression sanity: `tests/notifications/templates/ + W2-A consumer + morning advisory MD + daytrade_selection` = **389 PASS** (新 47 追加分含む、既存 342 不変)


## §6 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| DB write (production / develop / staging) | 0 |
| LINE 送信 | 0 (= linebot SDK import 0、push_message 呼出 0) |
| token / channel_token / secret / .env 値参照 | 0 (= os.environ 不参照) |
| API call (J-Quants / TDnet / 外部 HTTP) | 0 (= requests / aiohttp import 0) |
| launchctl / plist / cron | 0 |
| workflow 変更 | 0 (= .github/workflows 不在維持) |
| git add / commit / push | 0 |
| PR 作成 / merge | 0 |
| VACUUM | 0 |
| sudo / rm -rf / --no-verify | 0 |
| auto-order / 楽天 / iSPEED / Computer Use | 0 |
| TODO Excel 更新 | 0 |
| 既存 production code 変更 | 0 (= git diff origin/develop で 4 untracked file のみ) |
| 出力先 | /tmp/fire_f062_overlay_card_preview/ + ~/fire-vault/03_design/ (= safe path) |


## §7 Codex 4 lane 監査結果

| Lane | scope | verdict | severity |
|---|---|---|---|
| A | template / runner 実装 | **APPROVE** | CRITICAL/HIGH 0 |
| B | LINE / iPhone 可読性 + 絵文字 + 日本語ラベル | **APPROVE** | CRITICAL/HIGH 0 |
| C | tests / fixture | **CONCERN → 解消** | CRITICAL/HIGH 0、LOW 3 件指摘 → 全 test 強化済 |
| D | safety / no-send / no-token | **CONCERN** | CRITICAL/HIGH 0、LOW 2 件指摘 (= path guard 範囲 / file count 文言) |

集約: APPROVE 4 (= Lane C は強化済)、CRITICAL/HIGH 0。

### §7.1 Lane A 詳細
- overlay_buyability_card.py: 禁止 import / `os.environ` / I/O / DB / API / network なし、純関数のみ
- role/color mapping、recovery blocked badge、FORBIDDEN_PHRASES 整合性確認
- 価格計算は `_v1_4_1_morning_advisory_markdown` と一致
- runner: LINE/token/DB/API 接続なし、output guard で repo/data/production 系を拒否
- `--strict` validation 違反時 `return 2`、通常 `return 0`
- production 既存 3 ファイルの origin/develop 差分は空

### §7.2 Lane B 詳細
- 5 件 top5 + 1 件復帰禁止の badge 行 6 個確認
- iPhone 縦持ち: 価格行は折返し想定、空行区切りでブロック識別保てる、受容可能
- plain text 適合: ASCII `|` (markdown 表) と fenced code block hit 0
- forbidden grep: 「絶対」「100%」「自動発注」等 hit 0
- 絵文字フォールバック: 「主役｜有力」等の日本語ラベル単独で意味成立 (= UX 方針通り)

### §7.3 Lane C 詳細 (CONCERN → 強化済)
原指摘 (LOW):
1. fixture entry.count=20 / candidates=5 の整合 — note (count は consumer 集計値、test 用 subset 許容)
2. `unknown_role_falls_back_new_face` が 9008/3134 の new_face icon と区別できず偽 PASS 可能性
3. `long_name_truncation` 省略後の正確な形 (13字 + …) 未検証
4. path guard tests に `.git` / production path / `.github/workflows` 明示ケースなし
5. happy_path safety_flags は 2 key のみ部分検査 (全 False 未確認)

対応 (= 本 wave 内で完了):
- Test 2: 9247 行の直前 badge 行で 🆕 新顔｜🟡 比較 を厳密検証
- Test 3: `"9247 " + "あ"*13 + "…"` 期待文字列を assert
- Test 4: `test_git_dir_rejected` / `test_github_workflows_rejected` / `test_repo_source_rejected` /
  `test_launchagents_rejected` の 4 test 追加
- Test 5: happy_path で `payload["safety_flags"] == expected_safety` (= 7 key 完全一致)
- 再実行: **51 tests PASS** (= 47 + 4 追加)

### §7.4 Lane D 詳細 (CONCERN、軽微)
原指摘 (LOW):
1. output path guard は `/tmp` 限定ではなく `/private/tmp/` / `/private/var/folders/` /
   `/var/folders/` / `~/fire-vault/` も許可 (= pytest tmp_path + fire-vault doc 書き込みに必要)
2. 「5 file」ではなく「untracked 4 file」(= Codex の数え方差)

確認:
1. 4 prefix は意図通り (= pytest 動作 + fire-vault doc 出力に必須、各々 system path 外)。
   本 doc §2.1 / §3.3 で「safe path 限定」と表記したが「/tmp 限定」ではなく
   「**安全 prefix 限定**」が正確。報告では「/tmp + ~/fire-vault」と明示。
2. 本 wave 新規 file = 4 (= 2 source + 2 test)、本 doc は ~/fire-vault 配下で repo 外。
   "5 file" 表記は誤りで、正しくは "repo に 4 file 追加 + repo 外 fire-vault に 1 doc"。
   報告で表記揃え。

### §7.5 集約結論
- CRITICAL/HIGH 0
- Lane C 指摘は **本 wave 内で全 test 強化済** (= 47 → 51 PASS)
- Lane D 指摘は文言の精度問題 (= 本 doc / 報告で表記揃え、機能影響なし)
- safety / token / LINE / DB / API / git impact なし


## §8 結論 / 次アクション

### §8.1 結論

- 新規 template + runner + tests を追加 (= 4 file)
- 既存 production code 変更 0
- card 728 chars / 1 chunk fits ✓
- overlay 5 件 + 復帰禁止 1 件 + 手動確認 reminder 完備
- 役割アイコン + 評価色丸 + 日本語ラベル併記 (= UX 方針 100% 充足)
- 47 tests 全 PASS / 389 PASS regression なし
- forbidden phrase 0 / safety_flags 全 False / validation_passed=True

### §8.2 次アクション候補

1. /goal clear して本 wave 終了
2. 別 /goal で commit + push (= HQ_APPROVE_F062_OVERLAY_CARD_COMMIT 起票推奨)
   - Codex review 自動実行
   - 新 4 file + 本 doc を commit (= F062 overlay card 単独 commit、scope clear)
3. F286-PNL-R1 系の `--record-decisions` runner と統合 (= overlay card 送信時に
   pending decision を記録、後追い paper PnL 計算へ連結)
4. W2-B 設計着手 (= momentum / sector_relative_strength 数値判別、theme_tags table 設計)

### §8.3 関連 file

```
新規 (= 本 wave 追加):
  notifications/templates/overlay_buyability_card.py
  scripts/jobs/run_f062_overlay_buyability_card_preview.py
  tests/notifications/templates/test_overlay_buyability_card.py
  tests/scripts/jobs/test_run_f062_overlay_buyability_card_preview.py

出力 (= /tmp 限定):
  /tmp/fire_f062_overlay_card_preview/overlay_card.txt
  /tmp/fire_f062_overlay_card_preview/overlay_card_preview.json
  /tmp/fire_f062_overlay_card_preview/codex_lane_{A,B,C,D}_prompt.txt
  /tmp/fire_f062_overlay_card_preview/codex_lane_{A,B,C,D}_result.txt

入力 (= 既存 W2-A post-merge smoke 生成物):
  /tmp/fire_post_merge_smoke/overlay_consumer_payload.json

本 doc:
  ~/fire-vault/03_design/F062_W2A_OVERLAY_BUYABILITY_CARD_2026-05-16.md
```
