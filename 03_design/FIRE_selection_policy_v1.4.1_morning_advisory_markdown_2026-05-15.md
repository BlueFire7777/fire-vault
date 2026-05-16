---
id: FIRE-design-selection-policy-v1.4.1-morning-advisory-markdown-2026-05-15
phase: 本番 v0 中核 / morning advisory v1.4.1 Markdown CLI 実装
priority: 最高
status: complete
owner: BlueFire7777 (Fujiwara)
date: 2026-05-15
parent_design: 03_design/FIRE_selection_policy_v1.4.1_consumer_implementation_2026-05-15.md
codex_rounds: 4 (= 初回 + 3 巡目修正再確認)
codex_lanes: 8 / 8 YES ✓ (= Lane D 初回 NO 警告表示問題 → 3 巡目修正で完全 YES、Lane G 初回 NO output guard 不足 → 2 巡目修正で YES)
critical_high_count: 0 (= 全 HIGH 修正完了)
test_count: 60 (= 本 wave、renderer/CLI/invariants/D31-D32/safety)
test_total_pytest: 4961 PASS (= 前 wave 4901 + 本 wave 60)
---

# FIRE Selection Policy v1.4.1 — Morning Advisory Markdown CLI 実装

## §1 目的

前 wave (= consumer implementation) で構築した v1.4.1 consumer payload を、
Fujiwara が朝そのまま読める Markdown 形式に変換する read-only renderer + CLI を実装する。

`/tmp` または `~/fire-vault/` 配下のみ書き出しを許可、no_new_chase / chase_limit_exceeded
銘柄を entry 候補に **完全除外** (= 警告表示でなく出力しない) し、stdout summary でも
同じ guard を適用、判定 (GO_CONDITIONAL/HOLD/NO-GO) は safe_entry_count ベースで決定する.

## §2 実装 file

### 2.1 新規

| file | 役割 |
|---|---|
| `scripts/jobs/_v1_4_1_morning_advisory_markdown.py` | 純関数 renderer (= judgment / prices / sections / validation / stdout summary) |
| `scripts/jobs/run_v1_4_1_morning_advisory_markdown.py` | CLI runner (= consumer payload → Markdown + stdout summary、output path guard) |
| `tests/scripts/jobs/test_v1_4_1_morning_advisory_markdown.py` | 60 tests (= renderer / CLI / invariants / D31-D32 / safety) |

### 2.2 既存への影響

- consumer 側 (= `_v1_4_1_consumer.py`, `run_v1_4_1_advisory_render.py`): **手を加えていない** (= 完全 read-only consumer)
- F111 runner / F062 / F286: 手を加えていない

## §3 Markdown 構造

```
# FIRE 朝の advisory (= v1.4.1 morning advisory Markdown)
- date / base_date / consumer_version / f111_policy_version / renderer_version

## 判定
**GO_CONDITIONAL / HOLD / NO-GO** (entry raw=X / entry safe=Y / watch=Z / excluded=W)
note: 状態説明

## entry 候補 (= 第一確認候補)
### N) code name (sector / business)
  - entry 検討価格 / 指値目安 / 追わない上限 / 利確目安 / 損切り目安
  - 株数 100 株 / risk_yen_for_100_shares / liquidity_status / standard_lot_ok
  - 見送り条件
⚠ 二重 guard で除外された codes: ...  (= 存在時のみ)

## watch 候補 (= signal 観察 / entry 不可)
### N) code name
  - watch 理由 / 100 株 risk / entry 不可理由 / signal 観察 / demote 監視

## excluded 候補 (= 当日 entry 不可)
| # | code | name | sector | liquidity | exclusion_reason |

## no-new-chase / demote 監視候補
(= post-open phase 動的更新候補、pre-open 初期 snapshot は 0 件想定)

## 注文完成形 advisory (= Fujiwara 手動発注用 reference)
### N) code name
  - 銘柄コード / 株数 100 株 / 指値 / 損切り / 利確 / 追わない上限

## iSPEED 確認項目 (= 寄付前後の board / volume check)
7 項目

## review 10 項目 (v1.4.1)
1-10. v1.4.1 仕様準拠

## safety footer
- renderer_version / consumer_version / f111_policy_version / share_unit_standard
- review_required_fields_version=v1.4.1 / count=10
- validation_passed / validation_violations_count
- safety_flags 7 keys (= 全 False)
- "read-only advisory" / "auto-order なし" / "Markdown 生成のみ"
```

## §4 Field mapping (= consumer payload → Markdown)

| Markdown 表示 | consumer payload source |
|---|---|
| date | CLI 引数 --today |
| base_date | metadata.base_date |
| consumer_version | metadata.consumer_version |
| f111_policy_version | metadata.f111_policy_version |
| 判定 | sections.entry/watch/excluded.count + safe filter |
| entry 検討価格 | latest_close |
| 追わない上限 | latest_close × 1.02 (= +2.0%) |
| 利確目安 | latest_close + 2 × risk_per_share (= +2R) |
| 損切り目安 | latest_close − risk_per_share (= -1R) |
| 株数 | EXPECTED_SHARE_UNIT=100 (= 固定) |
| risk_yen_for_100_shares | candidate.risk_yen_for_100_shares |
| liquidity_status | candidate.liquidity_status |
| standard_lot_ok | candidate.standard_lot_ok |
| no_new_chase / chase_limit | candidate.no_new_chase / .chase_limit_exceeded |
| signal_success / entry_success | candidate.signal_success / .entry_success |
| review 10 項目 | REVIEW_10_ITEMS 定数 (= v1.4.1 spec) |
| safety footer | safety_flags 7 keys |

## §5 二重 guard (= no_new_chase / chase_limit_exceeded 完全除外)

### 5.1 `_candidate_is_safe_for_entry` 適用箇所

| 関数 | 動作 |
|---|---|
| `render_entry_section` | unsafe candidate は entry 詳細出力に含まれない、skipped codes を meta note で明示 |
| `render_order_advisory` | unsafe candidate は注文 reference に含まれない、skipped codes を meta note で明示 |
| `render_stdout_summary` | safe_entry / skipped_entry に分離、unsafe は code only meta 表示 (= 価格・stop・take 非表示) |
| `compute_judgment` | safe_entry_count == 0 + watch == 0 → NO-GO、safe_entry_count == 0 + watch >= 1 → HOLD、safe_entry_count >= 1 → GO_CONDITIONAL |

### 5.2 D31 / D32 で動作確認

| ケース | raw entry | safe entry | 判定 | 動作 |
|---|---|---|---|---|
| D31 fixture (= 9130 entry) | 1 | 1 | GO_CONDITIONAL | 9130 価格付き表示 |
| D32 fixture (= 9247/9628/9130 entry) | 3 | 3 | GO_CONDITIONAL | 全 3 件価格付き表示 |
| 異常 fixture (= 9130 + chase_limit) | 1 | 0 | **NO-GO** | 9130 価格非表示、no-new-chase section に列挙 |
| 異常 fixture (= 9130 + chase_limit + 4404 watch) | 1 | 0 | **HOLD** | 9130 除外、4404 watch のみ表示 |

## §6 output path guard (= Lane G 対応)

### 6.1 ALLOWED_OUTPUT_PREFIXES

```python
ALLOWED_OUTPUT_PREFIXES: tuple[str, ...] = (
    "/tmp/",
    "/private/tmp/",
    "/private/var/folders/",  # macOS pytest tmp_path / system temp
    "/var/folders/",
    str(_HOME / "fire-vault") + "/",
)
```

### 6.2 動作

| --output-md 値 | resolve() 後 | 判定 | exit code |
|---|---|---|---|
| `/tmp/x.md` | `/tmp/x.md` | allowed | 0 |
| `~/fire-vault/x.md` | `/Users/bluefire/fire-vault/x.md` | allowed | 0 |
| pytest tmp_path | `/private/var/folders/.../x.md` | allowed | 0 |
| `/Users/bluefire/fire/data/x.md` | source tree 配下 | **rejected** | **2** |
| `/etc/x.md` | system path | **rejected** | **2** |

reject 時は `[OUTPUT GUARD]` log + exit 2、ファイルは作成しない.

## §7 smoke 結果

### 7.1 input

`/tmp/fire_after_r1_v1_4_1_consumer_smoke/advisory_payload.json` (= 前 wave 出力)

### 7.2 実行コマンド

```
.venv/bin/python -m scripts.jobs.run_v1_4_1_morning_advisory_markdown \
  --input-json /tmp/fire_after_r1_v1_4_1_consumer_smoke/advisory_payload.json \
  --output-md  /tmp/fire_morning_advisory_v1_4_1_markdown_smoke/morning_advisory.md \
  --today 2026-05-15 \
  --strict
```

### 7.3 結果

- exit 0
- output file: `/tmp/fire_morning_advisory_v1_4_1_markdown_smoke/morning_advisory.md`
- md size: 6,188 chars
- md_violations: 0
- payload_violations: 0
- payload_passed: True
- 判定: **GO_CONDITIONAL** (entry raw=3 / entry safe=3 / watch=1 / excluded=16)
- entry candidates: 9130 / 9247 / 9628
- watch: 4404 (= standard_lot_ok=False)
- excluded: 16 (= demote 7 + liquidity FAIL 9、331A0/4389/4317 含む)
- 二重 guard skipped: 0 件 (= 全 safe)
- "60 株" / "60株" / "30 株" / "50 株" / "買え" / "売れ" / "全力" / "必ず" / "100%" / "保証" / "自動発注" / "自動売買" 等 forbidden phrase 全 0 件 ✓

### 7.4 output guard reject 動作

```
$ .venv/bin/python -m scripts.jobs.run_v1_4_1_morning_advisory_markdown \
    --input-json ... \
    --output-md /Users/bluefire/fire/data/forbidden.md
[morning-advisory-md] OUTPUT GUARD: --output-md must be under ['/tmp/', '/private/tmp/',
'/private/var/folders/', '/var/folders/', '/Users/bluefire/fire-vault/']
(got: /Users/bluefire/fire/data/forbidden.md)
exit=2
```

→ ファイル作成されず、reject 動作確認 ✓

## §8 tests 結果

### 8.1 morning_advisory_markdown tests

| 系統 | 件数 |
|---|---|
| TestComputeJudgment | 7 (= D31/D32/empty/watch-only/validation NG + safe-count 分離 + unsafe-all → NO-GO/HOLD) |
| TestComputeReferencePrices | 3 |
| TestRenderEntrySection | 3 |
| TestRenderWatchSection | 2 |
| TestRenderExcludedSection | 2 |
| TestRenderNoNewChase | 2 |
| TestRenderOrderAdvisory | 2 |
| TestRenderReview10 | 1 |
| TestRenderSafetyFooter | 1 |
| TestRenderFullMarkdown | 3 |
| TestValidateMarkdownOutput | 4 (= clean + 60 株 + 買え + missing review 10) |
| TestInvariants | 8 (= 100 株標準 + forbidden phrase + no-new-chase 除外 + review 10 順序 + validation_violations) |
| TestRenderStdoutSummary | 5 (= 単一フェンス + key 確認 + violations 表示 + unsafe 除外 + safe 価格表示) |
| TestCli | 5 (= 正常 + missing input + no-stdout-summary + output guard reject + tmp allowed) |
| TestSafety | 3 (= import / token / DB write 系) |
| TestD31D32Separation | 5 (= 9130/331A0/4389 vs 9247/9628/4404 分離) |
| TestSignalEntrySuccessSeparation | 1 |
| TestMarkdownEncoding | 2 |
| **合計** | **60 PASS** |

### 8.2 全 pytest regression

- 前 wave smoke 後: 4901 PASS
- 本 wave 新規追加: 60 PASS
- 全 pytest 最終: **4961 PASS** (= 4901 + 60、regression 0)

## §9 Codex 8 lane 監査 (= 4 巡)

### 9.1 巡目別結果

| 巡 | Lane A | B | C | D | E | F | G | H | 追加 HIGH |
|---|---|---|---|---|---|---|---|---|---|
| 1 巡目 | YES | YES | YES | **NO** | YES | YES | **NO** | YES | Lane D, Lane G |
| 2 巡目 (= Lane D + Lane G 修正後) | - | - | - | **NO** (= stdout bypass) | - | - | YES | - | Lane D 2 巡目 |
| 3 巡目 (= stdout guard 追加) | - | - | - | YES | - | - | - | - | Lane D 残 HIGH (= judgment) |
| 4 巡目 (= compute_judgment safe-count) | - | - | - | **YES** | - | - | - | - | **なし** |

### 9.2 採用 / 棄却理由

- Lane A (1 巡目 YES): sections.entry/watch/excluded mapping 完備、judgment logic 妥当
- Lane B (1 巡目 YES): entry 価格 / 指値 / 追わない上限 / 利確 / 損切り / 100 株 / risk_yen 全出力、命令表現なし
- Lane C (1 巡目 YES): 100 株固定、60/30/50 株表現 0 件、4404 watch 降格
- Lane D (4 巡目 YES): render_entry / render_order / render_stdout 全部で _candidate_is_safe_for_entry 適用、unsafe を価格付き表示しない、compute_judgment が safe_entry_count で判定
- Lane E (1 巡目 YES): review 10 項目全出力、signal_success / entry_success 分離記録
- Lane F (1 巡目 YES): renderer/CLI/D31-D32 fixture tests 揃う
- Lane G (2 巡目 YES): _is_output_path_allowed で /tmp / fire-vault / pytest tmp_path のみ allowed、それ以外 exit 2
- Lane H (1 巡目 YES): D31 (= 9130/331A0/4389) と D32 (= 9247/9628/9130/4404) を fixture + stdout summary で明示分離

### 9.3 CRITICAL / HIGH 0

最終 (= 4 巡目時点): 残 CRITICAL/HIGH **なし**.

## §10 safety final verification

| 観点 | 結果 |
|---|---|
| production md5 (= fire.db) | b1df4673e5c3645fbe2c5f490ffac043 → 不変 ✓ |
| develop md5 | 0eed4ad2ec7ed2edf8f640d97341c5ad → 不変 ✓ |
| staging md5 | 6cb3885cd9c90bd6fdabb127c1cd0d17 → 不変 ✓ |
| F282 plist | mtime/size 不変 ✓ |
| DB write / API / token | 全 0 ✓ |
| LINE / launchctl / plist / cron | 全 0 ✓ |
| git commit / git push / --no-verify | 全 0 ✓ |
| forbidden import (sqlite3/subprocess/aiohttp/requests/linebot) | 全 0 ✓ |
| token literal (LINE_*/JQUANTS_*) | 全 0 ✓ |
| 出力先 | /tmp/fire_morning_advisory_v1_4_1_markdown_smoke/ + pytest tmp_path のみ |
| Markdown forbidden phrase | "60 株" / "買え" / "必ず" / "自動発注" 等 全 0 件 ✓ |

## §11 次 wave 提案

### Priority A
1. **post-open phase 動的更新 wrapper** (= chase_limit_exceeded / entry_priority を actual flow ベースで F111 runner output に back-fill する pipeline、本 wave の二重 guard 完全活用)
2. **D33 pilot で morning advisory MD 実 advisory に反映** (= Fujiwara が iPhone で実 stdout summary をコピーし、iSPEED 確認項目 + 注文 reference を実利用)

### Priority B
3. F062 advisory / paper PnL preview runner で morning advisory MD を統合 (= 各 runner で render_full_markdown を呼ぶ optional flag)
4. weekly summary (= 5 営業日分の morning advisory MD を集約する weekly digest renderer)
5. iSPEED checklist customization (= Fujiwara からの fb で項目調整)

### Priority C
6. W5 集約 (= D20-D33 14 day、v1.4.1 framework 演化総括)
7. F111 / consumer / morning advisory コード git commit + push (= HQ approve 後)

## §12 まとめ

v1.4.1 morning advisory Markdown renderer + CLI **完全実装**.
F111 runner pre-open snapshot → v1.4.1 consumer payload → morning advisory Markdown +
stdout summary の **3 段 pipeline 完成**.

二重 guard (= _candidate_is_safe_for_entry) を render_entry / render_order /
render_stdout / compute_judgment の **4 箇所すべて** で適用、unsafe candidate (=
no_new_chase / chase_limit_exceeded / liquidity FAIL 等) が朝の Markdown でも
stdout summary でも entry 欄に **絶対に価格付きで表示されない** invariant 確立.

output path guard (= /tmp / fire-vault / pytest tmp_path 以外 exit 2) により
production DB / source tree への誤書き込みを防止.

Codex 8 lane 4 巡監査で全 YES、CRITICAL/HIGH 0、tests 60 PASS、全 pytest 4961 PASS、
safety 0 違反、3 DB md5 / F282 plist 不変、"60 株" / "買え" / forbidden phrase 0 件.
