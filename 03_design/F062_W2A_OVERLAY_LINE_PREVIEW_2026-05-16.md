# FIRE F062 W2-A overlay LINE preview (2026-05-16)

doc_id: FIRE-F062-W2A-OVERLAY-LINE-PREVIEW-2026-05-16
status: read-only preview 完了 (= production code 変更 0、LINE 送信 0)
related:
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_SMOKE_2026-05-16.md
- FIRE_PR2_scope_audit_merge_strategy_2026-05-16.md
- F286-REPORT-R1 fire/report/line_delivery.py (chunker source)


## §1 目的

W2-A overlay morning advisory MD を F062 LINE preview 相当で no-send 検証し、
iPhone/LINE 幅での表示崩れ・overlay 5 要素 (raw score top5 / overlay top5 /
主役継続枠 / risk warning 枠 / 新顔+新sector 比較枠) の保持状況を評価する。

実 LINE 送信 / token 参照 / DB write / API 呼出 / production code 変更は 0。
本 wave は preview とレビューのみ。実装変更は別 wave へ。


## §2 入力 / 設定

| 項目 | 値 |
|---|---|
| 入力 MD | `/tmp/fire_post_merge_smoke/overlay_morning_advisory.md` |
| MD bytes | 31,958 |
| MD lines | 656 |
| chunker | `fire.report.line_delivery._generate_preview` |
| chunk_length | 1,120 (= F286-REPORT-R1 `DEFAULT_CHUNK_LENGTH`) |
| fence policy | `_enforce_per_chunk_fence_format` (= 各 chunk を ``` で wrap、内部 ``` は ZWSP escape) |
| 出力 dir | `/tmp/fire_f062_overlay_line_preview/` |
| LINE 送信 | **0** (= linebot SDK / channel token / push_message いずれも未呼) |
| production code 変更 | **0** (= git diff origin/develop = 空) |


## §3 chunk 結果

| 項目 | 値 |
|---|---|
| total_chunks | **21** |
| partial_safe | False (= 21 > 1) |
| chunk size min/max/avg | 376 / 1,119 / 1,050 |
| fence integrity | 21/21 正常 (= 外側 wrap、内部衝突 0) |
| forbidden token-like hit | 0 (LINE_CHANNEL / CHANNEL_TOKEN / CHANNEL_SECRET / .env / AKIA / gho_ / glpat- / sk_live) |

### §3.1 key 要素の chunks 分布

| 要素 | chunks |
|---|---|
| raw score top5 section (= 既存ロジック reference) | [1] |
| overlay top5 section (= W2-A 表示レイヤー) | **[2]** ← 5 件全て同一 chunk 内 ✓ |
| 主役継続枠 (continuing_core) | [2, 3] |
| risk warning 枠 | [2, 3] |
| 新顔+新sector 比較枠 | [2, 3] |
| 9247 ＴＲＥ HD | [1, 2, 3, 15] |
| 9628 燦 HD | [1, 2, 3, 4, 15] |
| 4404 ミヨシ油脂 (⚠) | [1, 2, 4, 15] |
| 9008 京王電鉄 | [2, 9, 18] |
| 3134 Hamee | [2, 11, 18] |
| 9130 (excluded 警告) | [1, 12, 19] |
| overlay_policy_version | [1] |
| 手動確認文言 | [3] |
| auto-order なし | [1, 15] |

朝判断の根幹 (= 判定 + overlay top5 + 4 種 role 説明) は **chunks 1-3 (約 3,189 chars)** に集約。
残り chunks 4-21 は entry 20 件詳細 / post-open / appendix。


## §4 LINE/iPhone 表示崩れ評価

### §4.1 強み

- overlay top5 表は **chunk 2 内に完整** (= 5 件全て同一 chunk、分断なし)
- 4 種 role (主役継続 / risk warning / 新顔+新sector / 比較) の説明は chunk 3 で完結
- 朝判断 critical info は **chunks 1-3 で完結** (= 必要なら冒頭 3 通だけ見ればよい)
- fence wrap 全 21 chunks 完全、内部 fence 衝突 0
- forbidden token-like 文字列 0
- 9130 excluded 警告 chunk 12 で保持 (= 復帰禁止伝達 OK)
- 手動判断前提 (chunk 3) + auto-order なし (chunks 1, 15) で安全文化維持

### §4.2 懸念 (= Lane B CONCERN)

| # | 懸念 | severity | 影響 |
|---|---|---|---|
| 1 | 21 chunks 過多 | MEDIUM | LINE 通知過剰、scroll 負荷大 |
| 2 | markdown 表が LINE plain text 化 | MEDIUM | `|` `-` リテラル表示、iPhone 縦持ち (~30-40 字) で列対応崩れ |
| 3 | raw score 表 fragment (chunks 1-2) | LOW | 「途中で表が切れた」視覚混乱 |
| 4 | `**太字**` / `### 見出し` が literal 表示 | LOW | markdown 装飾無効 |
| 5 | code fence ``` literal 表示 | LOW | LINE は monospace 強調なし、` ``` ` がそのまま見える |
| 6 | risk_yen 表記の不統一 (`10,665` vs `10665`) | LOW | raw 表 vs overlay 表で format 不一致 |
| 7 | entry 20 件詳細 (chunks 3-14) は LINE には冗長 | MEDIUM | Markdown 共有のほうが体感良 |

### §4.3 安全観点 (= リスクなし)

- LINE 送信 0、token 参照 0、DB write 0、API call 0
- launchctl / cron 0、production code 変更 0
- artifact は /tmp/fire_f062_overlay_line_preview/ 限定 (= safe path)


## §5 改善案 (= 3 案、本 wave では実装変更しない)

### 案 A: W2-A overlay 専用 Buyability Card 1 chunk (推奨)

morning_advisory MD は Claude Code 画面 / iPhone 単一フェンス共有用に維持し、
LINE 配信用には W2-A overlay 専用の **1 chunk Buyability Card** を新規 template で生成。

含めるべき要素 (= 約 1,000-1,200 chars に圧縮):

```
[朝の advisory W2-A overlay 版]

判定: GO_CONDITIONAL (entry 20 / watch 0 / excluded 30)

【overlay top5】(raw rank と併記)
1 (raw 1) 9247 TRE HD       継続主役  情報通信   0.8546 risk 8,060
2 (raw 2) 9628 燦 HD         継続主役  情報通信   0.8479 risk 6,950
3 (raw 3) 4404 ミヨシ油脂   ⚠ risk warn 食品     0.8413 risk 10,665
4 (raw 16) 9008 京王電鉄    新顔+新sector 運輸    0.8119 risk 3,735
5 (raw 19) 3134 Hamee       新顔+新sector 小売    0.8051 risk 2,060

【excluded 警告】9130 復帰禁止 (recently_seen demoted)

【手動確認 reminder】
- iSPEED で板 / 出来高 / spread を見てから entry
- auto-order なし / Computer Use なし / 楽天/iSPEED 自動操作なし
```

実装場所案:
- `notifications/templates/research_advisory.py` に `build_overlay_card()` 純関数追加
- 新規 runner `scripts/jobs/run_f062_overlay_buyability_card_preview.py`
- 既存 F062-R5 Buyability Card と並列、選択式
- 送信は preview のみ、本番送信は別 HQ 承認

### 案 B: chunk_length を 2000-2500 に拡大

`fire.report.line_delivery.DEFAULT_CHUNK_LENGTH = 1120` を 2000-2500 に拡大すれば
21 chunks → 12-15 chunks 圧縮可能。

欠点:
- iPhone 縦持ちで 1 chunk 内 scroll 負荷増す
- LINE 1 メッセージ 5000 字制限内には依然余裕、しかし可読性改善せず
- 表 fragment 問題は本質的に解消しない
- 既存 F286-REPORT-R1 / 他 LINE preview にも影響、慎重な検証必要

### 案 C: 現状放置 + 二系列化

morning_advisory MD は Claude Code / Markdown 共有用 (= iPhone 単一フェンス) と割り切り、
LINE 配信は現状の Buyability Card (F062-R5) のみとする。W2-A overlay 専用 LINE template は
作らず、overlay 情報を見たい場合は Markdown を見る (= 二系列化)。

### 案 比較

| 観点 | 案 A | 案 B | 案 C |
|---|---|---|---|
| 実装工数 | 中 (= 新 template + runner) | 小 (= constant 変更) | 0 (= 現状維持) |
| LINE UX | ◎ (= 1 chunk 完結) | △ (= chunks 半減のみ) | ○ (= 既存 Buyability Card) |
| overlay 情報伝達 | ◎ (overlay 専用) | △ (依然分散) | × (LINE では非表示) |
| 朝判断速度 | ◎ | ○ | ○ |
| 既存 morning_advisory MD への影響 | 0 | 0 | 0 |

**推奨: 案 A** (Lane D RECOMMENDED_NEXT = A 一致)
理由:
- 朝 iPhone での手動判断には「結論 + overlay Top5 + 主要警告」に絞るのが
  FIRE の再現性優位 (= 確認パターン固定) と運用負荷の両方に合う
- 案 B は LINE 表示崩れを圧縮するだけで根本解決にならない
- 案 C は overlay の判断価値を LINE 側に載せきれない


## §6 Codex 4 lane 監査結果

| Lane | scope | verdict | severity |
|---|---|---|---|
| A | F062 / no-send / token 安全 | **APPROVE** | CRITICAL/HIGH 0 |
| B | LINE / iPhone 表示 | **CONCERN** | CRITICAL/HIGH 0、LOW: 21 chunks 過多、表 plain 化、raw 表 fragment |
| C | overlay 内容保持 | **APPROVE** | CRITICAL/HIGH 0。chunk 2 に 5 件保持確認、`10,665` vs `10665` 表記差は LOW |
| D | safety / 次実装判断 | **APPROVE** / RECOMMENDED_NEXT: **A** | CRITICAL/HIGH 0、production send 誤起動 / token 露出 / LINE 大量送信 risk すべて見えず |

集約: APPROVE 3 + CONCERN 1 (LOW、LINE 表示崩れ)、CRITICAL/HIGH 0。

### §6.1 Lane A 詳細
- run_preview.py は _generate_preview を使う preview 生成のみ
- LINE SDK / import、token / env 参照、DB 接続、外部 API、launchctl / cron / git 操作なし
- 出力先 /tmp/fire_f062_overlay_line_preview/ 固定
- fire/report/line_delivery.py は dataclasses / re / typing のみ、linebot SDK / HTTP / sqlite3 / env / file I/O なし
- preview_chunks.json forbidden substring 0、全 chunk が正常 fence wrap
- send 系 runner は本監査では起動していない

### §6.2 Lane B 詳細
- 21 chunks、本文長 376-1119、外側 ``` 1 組 wrap、内部 fence 衝突なし
- iPhone 縦持ちで raw/overlay の Markdown 表は横幅過大で列対応崩れ
- raw score top5 が chunk 1 rows 1-3 / chunk 2 rows 4-5 に分断、「途中で表が切れた」感
- overlay top5 は chunk 2 内で完整
- 21 メッセージ連続は通知過多 / scroll 負荷大、朝判断用 LINE としては UX 懸念
- 最低限の朝判断「判定 + overlay top5」は chunk 1-2 の 2 メッセージで完結

### §6.3 Lane C 詳細
- chunk 2 は overlay top5 5 件 (9247 / 9628 / 4404 / 9008 / 3134) 全保持
- 各行で role 判別可能 (= 主役継続枠 / risk warning 枠 / 新顔+新sector 比較枠)
- 4404 の risk_yen 表記は chunk 2 で `risk_yen=10665` (= overlay 表内 reason_text の key=value 形式)、
  chunk 1 の raw 表では `10,665` (= 通常 format)。LOW 表記不統一だが意味は伝わる
- 4404 ⚠ 保持確認
- 9130 は entry/watch に出ず、chunk 12 excluded 行 + chunk 19 説明例のみ。watch=0
- 手動判断文言は chunk 3 が主、chunk 1/4/15 にも関連文言
- 1 chunk 圧縮なら「判定 + overlay top5 + 4404 risk warning + 手動確認 + auto-order なし」を残し、role 詳説 / entry 20 件詳細は削るのが妥当

### §6.4 Lane D 詳細
- git diff --exit-code origin/develop = 0、production code 変更 0
- 指定 3 ファイル (line_delivery / no_send_wrapper_preview / research_advisory) も差分なし
- /tmp/fire_f062_overlay_line_preview/ は /private/tmp/... 配下の repo 外通常ファイルのみ
- run_preview.py は read/write とも /tmp 限定
- 推奨は A。B は LINE 表示崩れ圧縮だけで根本解決にならず、C は overlay 判断価値を LINE 側に載せきれない


## §7 安全 gate (= 本 wave 行為)

| 項目 | 結果 |
|---|---|
| LINE 送信 | 0 |
| token / channel_token / secret / .env 値参照 | 0 |
| DB write (production / develop / staging) | 0 |
| API 呼出 (J-Quants / TDnet / 外部 HTTP) | 0 |
| launchctl / plist / cron | 0 |
| workflow 変更 | 0 |
| git add / commit / push | 0 |
| PR 作成 / merge | 0 |
| sudo / rm -rf / --no-verify | 0 |
| auto-order / 楽天 / iSPEED / Computer Use | 0 |
| production code 変更 | 0 (= git diff origin/develop = 空) |
| TODO Excel 更新 | 0 |
| 出力先 | /tmp/fire_f062_overlay_line_preview/ + 本 doc (= safe path) |


## §8 結論 / 次アクション

W2-A overlay morning_advisory MD は **Claude Code / Markdown / iPhone 単一フェンス共有用途**で
完成度が高い (= 12 項目 PASS、validation_passed=True、forbidden phrase 0)。

LINE 直送には 21 chunks 必要で UX 過多、markdown 表 / 太字 / 見出し / fence の LINE 非対応により
**LINE 直送には不向き**。

### §8.1 改善要否

- 本 wave: **実装変更 0** (= production code 不変、新 template 追加なし)
- 次 wave: **案 A (W2-A overlay 専用 Buyability Card 1 chunk)** を別 /goal で起票推奨

### §8.2 次 wave 候補

1. F062 W2-A overlay Buyability Card 実装 (案 A、推奨、~ 1 wave)
2. W2-B 設計着手 (= momentum / sector_relative_strength の数値判別、theme_tags table)
3. F286 production v0 readiness 最終チェック
4. Universe Expansion R1 W4+ (= top_n=100 baseline 上の更なる expansion)
5. /goal clear して本 wave 終了

### §8.3 関連 file

- /tmp/fire_f062_overlay_line_preview/run_preview.py (preview driver)
- /tmp/fire_f062_overlay_line_preview/preview_chunks.json (21 chunks 詳細)
- /tmp/fire_f062_overlay_line_preview/preview_text.md (人間可読 concat)
- /tmp/fire_f062_overlay_line_preview/line_readability_report.md (本 doc の元 report)
- /tmp/fire_f062_overlay_line_preview/codex_lane_{A,B,C,D}_prompt.txt
- /tmp/fire_f062_overlay_line_preview/codex_lane_{A,B,C,D}_result.txt
- /tmp/fire_post_merge_smoke/overlay_morning_advisory.md (入力 MD)
- 本 doc: ~/fire-vault/03_design/F062_W2A_OVERLAY_LINE_PREVIEW_2026-05-16.md
