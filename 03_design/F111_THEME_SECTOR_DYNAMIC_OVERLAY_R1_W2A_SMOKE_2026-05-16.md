# FIRE F111 THEME/SECTOR DYNAMIC OVERLAY R1 W2-A smoke (2026-05-16)

doc_id: F111-THEME-SECTOR-DYNAMIC-OVERLAY-R1-W2A-SMOKE-2026-05-16
status: staging/read-only smoke 完了 (= push 済 commit に対する PR 前確認)
related:
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_2026-05-16.md (W1 design)
- F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_result_2026-05-16.md (W2-A implementation 完了)
- F111_UNIVERSE_EXPANSION_R1_W3_result_2026-05-16.md (top_n=100 baseline)


## §1 目的

develop に push 済の W2-A overlay 実装が、staging/read-only で F111 → consumer →
morning advisory パイプラインに正しく表示されるかを smoke で確認する。PR 作成前。

git add / commit / push / PR 作成は行わない。staging/production/develop DB write も
行わない。LINE 送信 / API 呼出 / token 参照 / launchctl / cron 操作も 0。


## §2 前提

### §2.1 push 済 commit (= 本 smoke の対象)

- d96ca10 feat(F111): baseline research watchlist top-n to 100
- b7e43e8 feat(F111): add theme-sector overlay display layer
- 0f26f47 test(F111): cover overlay display and letter-suffix regressions

### §2.2 git 状態 (= 本 smoke 開始時)

- branch = develop
- ahead = 0, behind = 0 vs origin/develop
- working tree clean (git status --short = 空)
- remote = `git@github.com:BlueFire7777/fire.git` (SSH のみ)

### §2.3 staging DB

- path = `/Users/bluefire/fire/data/fire.staging.db`
- size = 4,804,222,976 bytes (= 約 4.8 GB)
- md5  = `f6fa86df9c22d3e5866c7d3f380286d5`
- latest signals: base_date=2026-07-22 / 109 件 (research_watchlist_signals)
- 接続 mode = sqlite URI `mode=ro` (read-only) のみ

### §2.4 production / develop DB (= 接続なし、md5 保全確認)

| DB | size | md5 |
|---|---|---|
| data/fire.db (production) | 371,081,216 | `b1df4673e5c3645fbe2c5f490ffac043` |
| data/fire.develop.db | 371,081,216 | `0eed4ad2ec7ed2edf8f640d97341c5ad` |
| data/fire.staging.db | 4,804,222,976 | `f6fa86df9c22d3e5866c7d3f380286d5` |

### §2.5 F282 plist 3 file (= safety baseline)

| file | size | mtime |
|---|---|---|
| docs/launchd/jp.fire.weekly-snapshot.plist (repo) | 1,772 | 5月 12 22:27 |
| docs/launchd/jp.fire.weekly-snapshot-smoke.plist (repo, smoke) | 1,844 | 5月 14 23:34 |
| ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist (deployed) | 1,772 | 5月 12 22:46 |

launchctl / plist 編集 0、cron 編集 0。


## §3 実行内容

### §3.1 関連 tests (= 全 PASS)

| バッチ | 件数 | 結果 |
|---|---|---|
| consumer overlay + morning advisory MD + signal_persistence (× 5 files) | 264 | PASS |
| F111 selection + research advisory + sector research (× 6 files) | 192 | PASS |
| **合計** | **456** | **PASS** |

### §3.2 F111 staging smoke (= read-only)

実行コマンド:

```
.venv/bin/python -m scripts.jobs.run_f111_real_batch_staging \
  --db-path /Users/bluefire/fire/data/fire.staging.db \
  --base-date 2026-07-22 \
  --max-candidates 50 \
  --recently-seen-codes 9130,8747,137A0,7991,340A0,5729,3489,3798 \
  --output-json /tmp/fire_overlay_smoke/overlay_staging_f111.json
```

結果:
- candidates = 50
- eligible = 48 (= 2 件 risk_above_pilot_limit で除外)
- selection_policy_version = 1.4.2
- base_date = 2026-07-22

### §3.3 consumer payload + overlay (= 表示レイヤー、build_consumer_payload(overlay_input=...))

入力:
- prior_day_entry_codes = W1 design D48 推定 14 件 (= D49 entry \ 新顔 6 件)
- prior_day_entry_sectors = W1 design D48 推定 7 種 (= D49 sector \ 運輸・物流 \ 小売)
- excluded_recovery_codes = {"9130"}

結果:
- entry = 20 / watch = 0 / excluded = 30
- validation_passed = True / validation_violations = 0
- safety_flags 全 False (db_write / api_call / line_send / token_access / launchctl_call / auto_order / computer_use)
- overlay_policy_version = `1.0.0-w2a-display-only`
- overlay_top5 codes = `[9247, 9628, 4404, 9008, 3134]` ← W1 design 完全一致 ✓

### §3.4 current raw score top5 (= 既存ロジック reference)

| raw | code | 銘柄 | sector | score | risk_yen | warning |
|---|---|---|---|---|---|---|
| 1 | 9247 | ＴＲＥホールディングス | 情報通信・サービスその他 | 0.8546 | 8,060 | |
| 2 | 9628 | 燦ホールディングス | 情報通信・サービスその他 | 0.8479 | 6,950 | |
| 3 | 4404 | ミヨシ油脂 | 食品 | 0.8413 | 10,665 | ⚠ |
| 4 | 2146 | ＵＴグループ | 情報通信・サービスその他 | 0.8305 | 920 | |
| 5 | 4828 | ビジネスエンジニアリング | 情報通信・サービスその他 | 0.8293 | 6,230 | |

### §3.5 overlay top5 (= role 分類後の朝判断候補、raw rank 併記)

| overlay | raw | code | 銘柄 | sector | role | 新顔 | 新sec | warning |
|---|---|---|---|---|---|---|---|---|
| 1 | 1 | 9247 | ＴＲＥホールディングス | 情報通信・サービスその他 | 主役継続枠 | | | |
| 2 | 2 | 9628 | 燦ホールディングス | 情報通信・サービスその他 | 主役継続枠 | | | |
| 3 | 3 | 4404 | ミヨシ油脂 | 食品 | risk warning 枠 | | | ⚠ |
| 4 | 16 | 9008 | 京王電鉄 | 運輸・物流 | 新顔+新sector 比較枠 | ✓ | ✓ | |
| 5 | 19 | 3134 | Ｈａｍｅｅ | 小売 | 新顔+新sector 比較枠 | ✓ | ✓ | |

W1 design 完全一致 ✓ (= F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_result_2026-05-16.md §3.2 と同値)

### §3.6 overlay_summary

| 項目 | 値 |
|---|---|
| total_entry | 20 |
| eligible_count | 20 |
| continuing_core_count | 2 |
| risk_warning_count | 1 |
| new_face_sector_compare_count | 2 |
| comparison_count | 0 |
| overlay_total | 5 |
| new_face_codes_in_entry | 3134, 3962, 6196, 7595, 9008, 9417 |
| new_sectors_in_entry | 小売, 運輸・物流 |
| excluded_recovery_codes_active | 9130 |
| prior_day_entry_codes_count | 14 |
| prior_day_entry_sectors_count | 7 |

### §3.7 morning advisory markdown

- file: `/tmp/fire_overlay_smoke/overlay_morning_advisory.md`
- 行数 = 655
- bytes = 31,958
- validate_markdown_output → 0 violations
- FORBIDDEN_PHRASES (17 種) → hit 0
- 手動判断前提文言:
  - "Fujiwara が手動確認" ✓
  - "iSPEED で板 / 出来高 / spread を見てから entry" ✓
  - "auto-order なし" ✓
  - "Computer Use なし" ✓
  - "手動発注" ✓


## §4 12 項目検証 (= W2-A smoke 要件)

| # | 項目 | 結果 |
|---|---|---|
| 1 | raw score top5 が出る | ✓ (5 件) |
| 2 | overlay top5 が出る | ✓ (5 件 = 9247/9628/4404/9008/3134) |
| 3 | overlay_rank と raw_rank が併記される | ✓ (overlay rank 1-5, raw rank 1,2,3,16,19) |
| 4 | role / role_label_jp / reason_text が出る | ✓ (全 5 件で完備) |
| 5 | 9247/9628 は主役継続枠 | ✓ (continuing_core / 主役継続枠) |
| 6 | 4404 は risk warning 枠 | ✓ (risk_warning_track / risk warning 枠 / risk_warning=True / ⚠) |
| 7 | 9008/3134 等の新顔+新sector 比較枠が ≥ 1 件 | ✓ (2 件: 9008=運輸・物流 / 3134=小売) |
| 8 | 9130 は entry/watch に復帰しない | ✓ (excluded_recovery_codes_active=['9130']、excluded のみ) |
| 9 | 低流動性/letter-suffix/非100株が混入しない | ✓ (entry 20 件全件 PASS / share_unit=100 / letter_suffix=False) |
| 10 | risk_yen が順位加点/entry 除外 gate に使われない | ✓ (4404 risk_yen=10665 が entry 残留、warning 表示のみ) |
| 11 | consumer validation_passed=True | ✓ (violations=0) |
| 12 | forbidden phrase 0 / 手動判断前提の文言 | ✓ (FORBIDDEN_PHRASES hit 0、手動判断文言完備) |


## §5 安全 gate 結果

| 項目 | 結果 |
|---|---|
| DB write (production / develop / staging) | 0 |
| schema migration | 0 |
| VACUUM | 0 |
| LINE 送信 | 0 |
| API 呼出 (J-Quants / TDnet 等) | 0 |
| token / secret / env 参照 | 0 |
| launchctl 実行 | 0 |
| plist 編集 | 0 |
| cron 編集 | 0 |
| workflow 変更 | 0 |
| git add / commit / push | 0 |
| PR 作成 | 0 |
| --no-verify | 0 |
| sudo / rm -rf | 0 |
| auto-order / 楽天証券・iSPEED 操作 | 0 |
| Computer Use | 0 |
| TODO Excel 更新 | 0 |

3 DB md5 不変 (= smoke 開始時と同一)。F282 plist 3 file size/mtime 不変。


## §6 出力ファイル

| file | size | 備考 |
|---|---|---|
| /tmp/fire_overlay_smoke/overlay_staging_f111.json | 約 200KB | F111 raw output (read-only 生成) |
| /tmp/fire_overlay_smoke/overlay_consumer_payload.json | 約 50KB | consumer payload + overlay block |
| /tmp/fire_overlay_smoke/overlay_morning_advisory.md | 31,958 bytes | morning advisory MD (655 行) |
| ~/fire-vault/03_design/F111_THEME_SECTOR_DYNAMIC_OVERLAY_R1_W2A_SMOKE_2026-05-16.md | 本 doc | smoke 結果記録 |


## §7 Codex 4 lane 監査結果

| Lane | scope | verdict | severity |
|---|---|---|---|
| A | git/tests | **APPROVE** | CRITICAL/HIGH なし |
| B | F111/consumer overlay 出力 | **APPROVE** | CRITICAL/HIGH なし |
| C | Markdown 表示/手動判断文言 | **CONCERN** | LOW (overlay 表が LINE/iPhone で列揃え崩れる可能性。内容欠落や自動発注示唆ではない) |
| D | safety gate / DB / LINE / git 禁止 | **APPROVE** | CRITICAL/HIGH なし。軽微: smoke script は /tmp 配下へ write、ただし安全 path 限定 |

集約: **APPROVE 3 + CONCERN 1 (LOW)**、CRITICAL/HIGH なし。

### §7.1 Lane A (git/tests)
- 主張通り develop / clean / ahead=0 / behind=0 / SSH remote / 直近 3 commit 一致を確認。
- pytest 再実行は省略 (= 時間節約、Claude 報告 456 PASS について追加矛盾なし)。

### §7.2 Lane B (F111/consumer overlay)
- F111 raw: selected_count=50 / eligible_count=48 / risk_above_pilot_limit=2 確認。
- consumer: entry=20 / watch=0 / excluded=30, validation PASS, safety_flags 全 False。
- overlay_rank monotonic [1,2,3,4,5]、top5 = [9247, 9628, 4404, 9008, 3134] (D49 W1 design 完全一致)。
- 9130 は excluded のみ、entry/watch には出ていない。
- letter-suffix / 非100株 / 低流動性 混入なし。
- 4404 は risk_yen=10665 / risk_warning=True のまま entry 残留、score sort や entry gate には未使用 (risk warning bucket への role 分類のみ、設計通り)。

### §7.3 Lane C (Markdown/手動判断文言)
- md 655 行 / 31,958 bytes / validator 0 violations / FORBIDDEN_PHRASES hit 0。
- raw + overlay top5 両方表示、9247/9628 主役継続、4404 risk warning ⚠、9008/3134 新顔+新sector の表示確認。
- 各 overlay 行に role + 選定理由 (reason_text) あり。
- Fujiwara 手動確認 / iSPEED 確認 / auto-order なし / Computer Use なし 全明示。
- **LOW concern**: overlay 表が最大 200 字超、説明文も最大 284 字あり、LINE/iPhone では表の列揃え崩れの可能性。内容欠落 / 自動発注示唆ではないため smoke の本旨は満たすが、F062 LINE preview wave で chunk 化 / 簡略レイアウトを別途検討。

### §7.4 Lane D (safety gate)
- DB md5 3 件主張値と一致 (= staging smoke で書き込み 0 を独立確認)。
- F282 plist 3 file size/mtime 一致。
- git status 空 / ahead=0 / behind=0 確認。
- consumer payload safety_flags 全 False / validation_passed=True / violations 0 確認。
- **軽微疑義**: `/tmp/fire_overlay_smoke/run_overlay_smoke.py` が `/tmp/fire_overlay_smoke/` 配下に `write_text` する箇所が「read-only file I/O only」の字義通りではないが、安全 path 限定 (production/develop/config/source tree への書き込みなし、DB 不変)。本 smoke 用 artifact 書き出しは仕様。

### §7.5 集約結論

CRITICAL/HIGH なし → smoke の **GO 判定維持**。Lane C CONCERN は LOW (LINE 整形は別 wave 担当)。
PR 作成可否は §8.1 へ。


## §8 結論 / 次アクション

12 項目全 PASS、456 tests PASS、staging/read-only smoke 結果が W1 design と完全一致。

### §8.1 PR 作成可否

**PR 作成可能** (develop → main、要 HQ 承認)。Codex 4 lane 集約 = APPROVE 3 + CONCERN 1 (LOW)、
CRITICAL/HIGH なし。Lane C LOW concern (LINE 整形時の表崩れ) は F062 W2-A LINE preview 別 wave で対応。
本 W2-A は morning advisory MD (= Claude Code 画面 + Fujiwara 手動コピー想定) の表示レイヤーとして十分。

### §8.2 次 wave 候補

1. PR 作成 (develop → main、要 HQ 承認、要 PR body ドラフト指示)
2. F062 W2-A overlay LINE preview (= Markdown を LINE delivery preview に流し込む dry-run、send_guard 経由)
3. W2-B (= momentum / sector_relative_strength の数値判別、theme_tags table 設計)
4. Universe Expansion R1 W4+ (= top_n=100 baseline 上での更なる expansion 検証)


## §9 注意

- W2-A は **表示レイヤーのみ**。momentum / sector_relative_strength の数値判別は W2-B+ で実装予定。
- overlay 順位は raw rank → role 分類のみで決定 (risk_yen は加点・gate に効かない)。
- 9130 は recently_seen demoted で entry/watch 復帰禁止。
- F111 staging runner は production/develop DB 接続を構造的に refuse (= is_safe_db_path)。
- consumer payload + MD は inline smoke script (= /tmp/fire_overlay_smoke/run_overlay_smoke.py) で生成。コードは /tmp に置き repo を汚さない。
