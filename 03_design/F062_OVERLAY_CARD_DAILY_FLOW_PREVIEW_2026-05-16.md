# FIRE F062 Overlay Card Daily Flow Preview (2026-05-16)

doc_id: FIRE-F062-OVERLAY-CARD-DAILY-FLOW-PREVIEW-2026-05-16
status: 実装完了 (= 39 new tests + 51 prior = 合計 90 tests PASS、no-send / no-token / no-LINE / no-DB-write、既存 production code 変更 0)
related:
- F062_W2A_OVERLAY_BUYABILITY_CARD_2026-05-16.md (= 前 wave overlay card 実装)
- F062_W2A_OVERLAY_LINE_PREVIEW_2026-05-16.md (= 21 chunks 問題の根拠)
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_SMOKE_2026-05-16.md (= W2-A overlay の出典)
- F286-REPORT-R1 fire/report/line_delivery.py (= chunk_length 1120 の出典)


## §1 目的

W2-A overlay Buyability Card と full morning advisory MD を**朝 daily flow 用に
セットで生成する no-send preview** を提供する。LINE 配信は overlay_card のみ
(= 1 chunk)、full MD は詳細確認用 (= Claude Code / iPhone 単一フェンス共有)。

本 wave は **runner + tests のみ追加**、既存 production code は変更しない。
本番 LINE 送信 / launchd / cron 組み込みは別 wave (= HQ approve + send_guard 経由)。


## §2 実装範囲

### §2.1 新規 file (= 2 件、既存 0 変更)

| file | 行数 | 役割 |
|---|---|---|
| scripts/jobs/run_f062_overlay_daily_flow_preview.py | ~190 | runner、4 file 出力 + path guard + --strict |
| tests/scripts/jobs/test_run_f062_overlay_daily_flow_preview.py | ~370 | 39 tests (path guard 9 + happy 9 + package 12 + error 6 + safety 2 + idempotency 1) |

### §2.2 既存 file への変更 (= 0)

`git status --short` で確認:
- 2 untracked file のみ (`??` 状態)
- 既存 production code 一切不変 (notifications/templates/overlay_buyability_card.py /
  research_advisory.py / _v1_4_1_consumer.py / _v1_4_1_morning_advisory_markdown.py /
  fire/report/line_delivery.py / 他 全保全)
- 再利用: notifications.templates.overlay_buyability_card.build_overlay_card 純関数のみ


## §3 runner 設計

### §3.1 CLI

```
.venv/bin/python -m scripts.jobs.run_f062_overlay_daily_flow_preview \
  --input-consumer-json /tmp/fire_post_merge_smoke/overlay_consumer_payload.json \
  --input-morning-md    /tmp/fire_post_merge_smoke/overlay_morning_advisory.md \
  --output-dir          /tmp/fire_f062_overlay_daily_flow \
  --strict
```

### §3.2 出力 4 file

| file | 内容 | 用途 |
|---|---|---|
| `full_morning_advisory.md` | input MD を `shutil.copyfile` で保全コピー | 詳細確認用 (= Claude Code / iPhone 単一フェンス共有) |
| `overlay_card.txt` | build_overlay_card.text (= 728 chars / 34 lines) | **LINE 1 chunk 配信本文** |
| `overlay_card_preview.json` | card metadata + safety_flags (= 既存 preview runner と同 schema) | デバッグ / 検証 |
| `daily_flow_preview_package.json` | 朝運用の入力/出力/safety/judgment を一覧 | 朝 daily flow の運用 entry point |

### §3.3 path guard

`ALLOWED_OUTPUT_PREFIXES`:
- `/tmp/`
- `/private/tmp/`
- `/private/var/folders/` (= pytest tmp_path)
- `/var/folders/`
- `~/fire-vault/`

refuse 対象 (= test で網羅):
- `/etc/`、`/Users/bluefire/fire/data/`、`/Users/bluefire/fire/.git/`、
  `/Users/bluefire/fire/.github/workflows/`、`/Users/bluefire/fire/notifications/templates/`、
  `/Users/bluefire/Library/LaunchAgents/`、`/Users/bluefire` (= home root)

### §3.4 exit code

- 0: 正常完了
- 2: input file 不在、output dir unsafe、payload に overlay_top5 不在、JSON parse error、
     `--strict` validation 違反


## §4 daily_flow_preview_package.json (= 朝運用の入力/出力一覧)

実 smoke 出力 (= 2026-07-22 base_date):

```json
{
  "schema_version": "1.0",
  "cli_version": "1.0.0",
  "card_version": "1.0.0-w2a-overlay-card",
  "generated_at": "2026-05-16T13:41:XXZ",
  "base_date": "2026-07-22",
  "judgment": "GO_CONDITIONAL",
  "entry_count": 20,
  "watch_count": 0,
  "excluded_count": 30,
  "overlay_top5_codes": ["9247", "9628", "4404", "9008", "3134"],
  "excluded_recovery_codes": ["9130"],
  "card_char_count": 728,
  "card_line_count": 34,
  "fits_one_chunk": true,
  "line_one_chunk_body_max": 1112,
  "no_send": true,
  "line_send": false,
  "token_access": false,
  "db_write": false,
  "card_forbidden_hits": [],
  "card_reminder_present": true,
  "card_validation_passed": true,
  "safety_flags": {
    "db_write": false, "api_call": false, "line_send": false,
    "token_access": false, "launchctl_call": false,
    "auto_order": false, "computer_use": false
  },
  "input_consumer_json_path": ".../overlay_consumer_payload.json",
  "input_morning_md_path": ".../overlay_morning_advisory.md",
  "output_overlay_card_path": ".../overlay_card.txt",
  "output_overlay_card_json_path": ".../overlay_card_preview.json",
  "output_full_morning_advisory_path": ".../full_morning_advisory.md",
  "line_delivery_target": "overlay_card_only",
  "full_markdown_role": "detail_reference",
  "notes": "LINE 配信は overlay_card のみ (= 1 chunk)。 ..."
}
```


## §5 overlay card 確認 (= goal §6 要件)

| 要件 | 結果 |
|---|---|
| 9247/9628 = 👑 主役｜🟢 有力 | ✓ (= continuing_core mapping) |
| 4404 = ⚠️ 注意｜🟡 比較・注意 | ✓ (= risk_warning_track mapping) |
| 9008/3134 = 🆕 新顔｜🟡 比較 | ✓ (= new_face_sector_compare mapping) |
| 9130 = ⛔ 復帰なし｜🔴 見送り | ✓ (= excluded_recovery_codes_active note) |
| 日本語ラベル必須 | ✓ (= 主役/注意/新顔/復帰なし/有力/比較・注意/比較/見送り) |
| 価格 5 値保持 (= 参考/追上限/利確/損切/risk) | ✓ (全 5 銘柄) |
| Fujiwara 手動確認 reminder | ✓ (= "iSPEED で 板 / 出来高 / spread を確認してから entry") |
| auto-order なし note | ✓ (= "Fujiwara が手動発注 (auto-order なし、Computer Use なし、楽天/iSPEED 自動操作なし)") |


## §6 full Markdown 連携

| 観点 | 状態 |
|---|---|
| input MD | `/tmp/fire_post_merge_smoke/overlay_morning_advisory.md` (= 31,958 bytes / 656 行) |
| 出力 | `/tmp/fire_f062_overlay_daily_flow/full_morning_advisory.md` (= 同 bytes / 内容完全一致) |
| copy 方式 | `shutil.copyfile` (= binary level、修正なし) |
| package 内 path | `output_full_morning_advisory_path` field に明示 |
| role | `full_markdown_role = "detail_reference"` (= 詳細確認用、LINE 配信対象外) |
| LINE 配信 | overlay_card のみ (= `line_delivery_target = "overlay_card_only"`) |


## §7 文字数 / chunk

| 項目 | 値 |
|---|---|
| input MD bytes | 31,958 (= 656 行) |
| overlay_card char_count | 728 (= 34 行) |
| LINE_ONE_CHUNK_BODY_MAX | 1,112 (= F286-REPORT-R1 chunk_length 1,120 - fence overhead 8) |
| fits_one_chunk | True (= 728 < 1,112、margin 384 chars) |
| 圧縮率 | 31,958 → 728 chars ≈ **97.7% 圧縮** |


## §8 tests (= 39 件新規、全 PASS、regression なし)

| クラス | 件数 | カバレッジ |
|---|---|---|
| TestOutputDirGuard | 9 | tmp / etc / data_db / repo_source / git_dir / github_workflows / launchagents / home_root / fire_vault |
| TestMainRunnerHappyPath | 9 | exit_code_zero / 4 file 生成 / 1 chunk / overlay_top5 / 役割アイコン+色丸+ラベル / 9130 / 5 価格 / full MD intact / reminder |
| TestPackageJson | 12 | schema / cli / generated_at / base_date / judgment / counts / overlay codes / card meta / paths / no_send_flags / safety_flags 全 False / line_delivery_target / forbidden_hits 空 |
| TestRunnerErrorPaths | 6 | consumer_not_found / morning_md_not_found / unsafe_output_dir / missing_overlay_top5 / invalid_json / strict_violation |
| TestSafetyImports | 2 | no_dangerous_imports (linebot / sqlite3 / requests / aiohttp / subprocess / os.environ 不在) / does_not_send_line (= safety_flags 全 7 key False 検査) |
| TestIdempotency | 1 | 2 回実行で安定 |
| **合計** | **39** | **全 PASS** |

regression sanity: 既存 overlay_buyability_card tests (51) + 新 daily flow (39) = **合計 90 PASS** (= 既存 51 不変、新 39 追加)


## §9 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| DB write (production / develop / staging) | 0 |
| LINE 送信 | 0 (= linebot SDK / push_message / reply_message 不使用) |
| API call (J-Quants / TDnet / 外部 HTTP) | 0 (= requests / aiohttp import 0) |
| token / channel_token / secret / .env 値参照 | 0 (= os.environ 不参照) |
| launchctl / plist 配置 / cron 編集 | 0 (= 本 wave で組み込みなし) |
| workflow 変更 | 0 |
| git add / commit / push | 0 |
| PR 作成 / merge | 0 |
| VACUUM | 0 |
| sudo / rm -rf / --no-verify | 0 |
| auto-order / 楽天 / iSPEED / Computer Use | 0 |
| TODO Excel 更新 | 0 |
| 既存 production code 変更 | 0 (= git diff origin/develop で 2 untracked file のみ) |
| TestSafetyImports で independent 検査済 | linebot / sqlite3 / requests / aiohttp / subprocess / os.environ 全 import 0 |
| 出力先 | /tmp/fire_f062_overlay_daily_flow/ + ~/fire-vault/03_design/ (= 安全 prefix 限定) |


## §10 Codex 4 lane 監査結果

| Lane | scope | verdict | severity |
|---|---|---|---|
| A | runner 設計 / no-send daily flow | **CONCERN** | CRITICAL/HIGH 0、LOW 2 件 (文言精度) |
| B | LINE / iPhone UX / overlay card 保持 | **APPROVE** | CRITICAL/HIGH 0 |
| C | tests / fixture / output path guard | **APPROVE** | CRITICAL/HIGH 0 |
| D | safety / no-token / no-DB-write / next integration | **CONCERN** / RECOMMENDED_NEXT: 1 | CRITICAL/HIGH 0、LOW 2 件 (文言精度) |

集約: APPROVE 2 + CONCERN 2 (LOW)、CRITICAL/HIGH 0。

### §10.1 Lane A 詳細 (CONCERN、軽微)
- 危険 import / `os.environ` / 指定 production code 変更 0
- path guard semantics 正常
- `--strict` exit code 分岐正常 (= 違反 2 / pass 0)
- **LOW**: `overlay_input` キー自体は検証されず、`overlay_top5` 不在時のみ exit 2
  → 実装上 `build_consumer_payload(payload, overlay_input=ov)` を経由した payload には
     必ず overlay_top5 が付与されるため、検査ポイントとしては適切。本 doc で明示。
- **LOW**: write 経路は `shutil.copyfile` だけではなく `mkdir` + `write_text` 3 箇所
  → 全て output_dir path guard 通過後、production source / DB / network への write は一切なし
- ## §10.1-fix: 本 doc §3.2 で「shutil.copyfile が唯一の binary copy」と表記したが、
  正確には「**output dir 内の write は shutil.copyfile (full MD コピー) + write_text 3 箇所
  (card_txt / card_json / package_json) の合計 4 箇所、全て path guard 通過後**」

### §10.2 Lane B 詳細
- overlay card 1,142 bytes UTF-8 / 728 chars / 34 lines、1 chunk 内
- 5 銘柄は badge / identifier / price 3 段構成、「役割｜色」区切り全件保持
- iPhone 縦持ち: 価格行のみ 2 行折返し想定、銘柄ごとの情報単位崩れにくい
- markdown 表 / fence の card 内不在 (= LINE plain text 最適)
- 絵文字フォールバック OK (= 「主役｜有力」「注意｜比較・注意」「復帰なし｜見送り」言語ラベル単独成立)
- 手動確認 + auto-order なし note 明記
- 朝判断 critical info は overlay_card 単独で完結 (= full MD は補助)

### §10.3 Lane C 詳細
- 39 tests 収集、関連 regression 34 + 17 + 39 = 90 tests 収集確認 (= 既存 51 + 新 39)
- skip / skipif / xfail 該当なし (= 真の 90 PASS)
- path guard は data / .git / .github/workflows / LaunchAgents / repo source / home root 網羅
- safety_flags 検査は 7 key 完全一致 (= 部分検査ではない)
- idempotency は同一 dir 2 回実行で `exist_ok=True` + 固定 file 上書き設計、テスト OK

### §10.4 Lane D 詳細 (CONCERN、軽微)
- runner は危険 import / `os.environ` / SQL / LINE send / HTTP / subprocess 不在
- `shutil.copyfile` は input MD を許可 output dir へコピーするのみ
- 実プレビュー 4 file に forbidden token hit なし
- git status 未追跡 2 file のみ、send 系 runner の diff なし
- **LOW**: `/tmp/fire_f062_overlay_daily_flow/` は (runner 完了直後の) 4 file ではなく
  9 file 観測 (= 4 runner output + 4 codex prompts + 1 incomplete result が時点で残存)
  → ## §10.4-fix: 本 doc §3.2 で「出力 4 file」と書いたが、正確には
    「**runner が生成する 4 file**」+ 「**本 audit wave の codex 4 lane prompt/result 計 8 file**」
    = peak 12 file。runner 出力に限れば 4 file。
- **LOW**: 許可 prefix は `/tmp` 限定ではなく `~/fire-vault` (+ pytest tmp_path) も含む
  → ## §10.4-fix: 本 doc では「`/tmp/` + `~/fire-vault/`」と既に併記済、Lane D 指摘で再強調
- RECOMMENDED_NEXT: **1 (= send_guard 経路追加)** — F286-REPORT-R1 LineSendGuard 再利用、
  HQ_APPROVE marker 必須、本番送信は更に別 HQ approve

### §10.5 集約結論
- CRITICAL/HIGH 0
- Lane A/D CONCERN は **文言精度の問題のみ**、機能 / 安全 / test には影響なし
- 本 doc §3.2 / §10.1-fix / §10.4-fix で正確な記述に揃え済
- merge / send / DB / token 経路への影響なし
- 次 integration の優先は **案 1 = send_guard 経路追加** (Lane D 推奨)


## §11 結論 / 次アクション

### §11.1 結論

- runner + tests を追加 (= 2 file)
- 既存 production code 変更 0
- 4 file 出力で daily flow の入力/出力/safety を可視化
- overlay card 728 chars / 1 chunk fits / overlay top5 5 件 + 復帰禁止 1 件 + reminder 完備
- 39 tests PASS / 90 PASS regression なし
- forbidden phrase 0 / safety_flags 全 False / validation_passed=True

### §11.2 次アクション候補

1. /goal clear して本 wave 終了
2. 別 /goal で commit + push (= HQ_APPROVE_F062_DAILY_FLOW_COMMIT 推奨)
3. send_guard 経路追加 (= F286-REPORT-R1 LineSendGuard 再利用、HQ_APPROVE marker 必須、別 wave)
4. F286-PNL-R2 `--record-decisions` 統合 (= overlay card 配信時に pending decision 記録、別 wave)
5. cron / launchd 組み込み (= 本番送信時、別 wave + HQ approve)
6. W2-B 設計着手 (= momentum / sector_relative_strength 数値判別、theme_tags table 設計)

### §11.3 関連 file

```
新規 (= 本 wave 追加):
  scripts/jobs/run_f062_overlay_daily_flow_preview.py
  tests/scripts/jobs/test_run_f062_overlay_daily_flow_preview.py

出力 (= /tmp 限定):
  /tmp/fire_f062_overlay_daily_flow/full_morning_advisory.md         (31,958 bytes)
  /tmp/fire_f062_overlay_daily_flow/overlay_card.txt                 (1,142 bytes / 728 chars)
  /tmp/fire_f062_overlay_daily_flow/overlay_card_preview.json        (~2 KB)
  /tmp/fire_f062_overlay_daily_flow/daily_flow_preview_package.json  (~2 KB)
  /tmp/fire_f062_overlay_daily_flow/codex_lane_{A,B,C,D}_prompt.txt
  /tmp/fire_f062_overlay_daily_flow/codex_lane_{A,B,C,D}_result.txt

入力 (= 既存 W2-A post-merge smoke 生成物、原本不変):
  /tmp/fire_post_merge_smoke/overlay_consumer_payload.json
  /tmp/fire_post_merge_smoke/overlay_morning_advisory.md

本 doc:
  ~/fire-vault/03_design/F062_OVERLAY_CARD_DAILY_FLOW_PREVIEW_2026-05-16.md
```
