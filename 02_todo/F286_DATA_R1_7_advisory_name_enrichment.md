---
id: F286-DATA-R1.7
phase: P5: 通知 / 第 14 章 LINE 通知配信 / 第 25 章 F286 DATA pipeline
priority: 最優先
status: 完了 ★ (2026-05-11、code → 銘柄名 enrichment 実装 + 23 tests)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F062-R5.5 (= Practical Buyability Card UX、銘柄名表示 template 実装)
  - F286-DATA-R1.6 (= F119 historical artifacts wiring、advisory_preview 生成元)
  - F100 (= market_listings table 維持元 / J-Quants V2 lazy fetch)
chapter: 第 14 章 / 第 25 章 / 第 26 章
---

# F286-DATA-R1.7: Advisory Candidate Name Enrichment

最終更新: 2026-05-11

## ★ 状態: 完了

F062-R5.5 buyability mode は「銘柄コード + 銘柄名」を表示する設計
だったが、F286-DATA-R1.6 で wiring された advisory_preview JSON で
`name` 列が空のため、LINE 文面に銘柄コードのみが表示されていた:

```
🟠 強弱混在・慎重 1. 57290           ← 銘柄名なし
判定: 強弱混在・慎重
理由: F119 boost month_of_year / avoid sector_17 一部該当
見る点: 寄り後の値動き優先 / avoid 影響度
```

本タスクで read-only `market_listings` lookup helper を追加し、F062
runner で optional に code → company_name enrichment を行えるように
した。**実 LINE 送信なし / DB write なし / 既存 name の上書きなし**。

## 実施結果

### Step 1: market_listings 確認

| 項目 | 結果 |
|---|---|
| table 名                | market_listings |
| 列                      | code (PK) / company_name / company_name_en / sector_17_* / sector_33_* / scale_category / market_* / margin_* / fetched_at |
| 件数                    | 4,449 |
| ソース                  | J-Quants V2 (F100、fetched_at 2026-05-01) |
| advisory_preview の code 全件が含まれるか | ✅ 30 件中 30 件 hit (= missing 0) |

### Step 2: 新規 module `notifications/listing_name_lookup.py`

```python
class ListingNameLookup:
    def __init__(self, db_path, *, table='market_listings',
                 code_col='code', name_col='company_name'): ...
    def lookup(self, code) -> Optional[str]: ...
    def enrich_rows(self, rows: Iterable[Mapping]) -> dict: ...
```

仕様:
- DB は **read-only** で開く (`mode=ro` URI + `immutable=1` URI +
  `PRAGMA query_only=ON`、3 段防御)
- 初回 lookup 時に全件 cache load (= 4,449 行は単発 load の方が単純)
- DB 無し / 破損 / table 無し / connect 失敗 → no-op (= load_error 記録、
  rows は変更せず、runner 落ちず)
- enrich_rows: dict 型かつ `row['name']` が空 / None / 空白のみの場合
  のみ in-place で `row['name']` を埋める (**既存 name は上書きしない**)
- stats: `attempted` / `enriched` / `already_has_name` / `missing` /
  `cache_size` / `db_path` / `load_error`

LineBotClient / linebot SDK / channel token / send_text /
push_message を **import / 参照しない** (= ast-based test で検証)。
SQL DDL/DML literal (INSERT/UPDATE/DELETE 等) を **コード中に
持たない** (= source 検査 test で検証)。

### Step 3: F062 runner 統合 (`--listings-db`)

```bash
.venv/bin/python -m scripts.jobs.run_f062_research_advisory_line_preview \
  --preview-json /tmp/f286_data_r1_6_advisory_preview.json \
  --summary-json /tmp/f286_data_r1_6_advisory_summary.json \
  --gate-json    /tmp/f062_r5_4_gate.json \
  --listings-db  data/fire.staging.db \
  --message-mode production --buyability-mode --card-top 5 \
  --output-json    /tmp/f286_data_r1_7_line_payload.json \
  --output-text    /tmp/f286_data_r1_7_line_preview.txt \
  --output-summary-json /tmp/f286_data_r1_7_line_summary.json
```

| field | value |
|---|---|
| --listings-db                                    | data/fire.staging.db (read-only) |
| name_enrichment.attempted                        | 30 |
| name_enrichment.enriched                         | **30** ★ (= 全件成功) |
| name_enrichment.already_has_name                 | 0 |
| name_enrichment.missing                          | 0 |
| name_enrichment.cache_size                       | 4,449 |
| name_enrichment.load_error                       | None |
| chunks                                           | 1 |
| chunk[0] length                                  | **1,120** (= F062-R5.5 1,086 から +34、銘柄名追加分) |
| forbidden_phrase_count                           | 0 |
| safety_footer_present                            | True |
| selected_count                                   | 5 (= buyability Top 5) |
| metadata.buyability_mode                         | True |
| metadata.card_mode (effective)                   | True |

### Step 4: before / after 文面比較

**Before** (F062-R5.5 smoke、name 空):
```
🟠 強弱混在・慎重 1. 57290
🟠 強弱混在・慎重 2. 340A0
🟠 強弱混在・慎重 3. 37980
🟠 強弱混在・慎重 4. 137A0
🟠 強弱混在・慎重 5. 331A0
```

**After** (F286-DATA-R1.7 smoke、name enriched):
```
🟠 強弱混在・慎重 1. 57290 日本精鉱       ★
🟠 強弱混在・慎重 2. 340A0 ジグザグ        ★
🟠 強弱混在・慎重 3. 37980 ＵＬＳグループ  ★
🟠 強弱混在・慎重 4. 137A0 Ｃｏｃｏｌｉｖｅ ★
🟠 強弱混在・慎重 5. 331A0 メディックス    ★
```

Fujiwara が LINE app で銘柄を即座に識別できるようになった。

### Step 5: tests (= 23 件)

unit tests (`tests/notifications/test_listing_name_lookup.py`、18 件):
- `TestListingNameLookupBasic` (6): existing code → name / missing
  code → None / None code / int code coerce / cache_size /
  null or empty name skipped
- `TestEnrichRows` (5): empty fill / existing not overwritten /
  missing counted / non-dict skipped / whitespace treated as empty
- `TestFailureModes` (4): no db_path / missing file / missing
  table / corrupt file (全 no-op で落ちない)
- `TestReadOnlyEnforcement` (1): DB mtime unchanged
- `TestModuleSourceSafety` (2): ast-based linebot SDK import 検査 /
  ast-based SQL DDL/DML literal 検査

integration tests (`TestF286DataR17RunnerListingsDb`、5 件):
- enriches names → chunks 内に code + name 表示
- --listings-db 未指定 → metadata.name_enrichment is None
- missing DB file → runner 落ちず load_error 記録
- existing name not overwritten → already_has_name=1
- read-only mtime/size 不変

### Step 6: 回帰

| 範囲 | 結果 |
|---|---|
| 新規 R1.7 tests              | 23 PASS |
| 全 pytest スイート           | **3,311 PASS** (= 3,288 baseline + 23 新規) |

### Step 7: 安全要件 (= 全 ✅)

| 項目 | 結果 |
|---|---|
| 実 LINE 送信                              | 0 ✅ |
| DB write                                  | 0 ✅ (= read-only URI + immutable + PRAGMA) |
| production / develop / staging DB mtime   | 全 unchanged ✅ |
| token / recipient 平文出力                | 0 ✅ |
| forbidden phrase                          | 0 ✅ |
| 自動発注 / 楽天操作 / Computer Use        | なし ✅ |
| 既存 name の上書き                        | しない (= already_has_name で計上) ✅ |
| TODO Excel                                | 未更新 ✅ |
| --no-verify                              | 不使用 ✅ |
| scripts/seed_pattern_layer1.py            | 未接触 ✅ |
| simulation/.../historical_indicators.py   | 未接触 ✅ |
| unrelated modified を stage / commit      | しない ✅ |

## commits

```
fire (develop):
  f30833f  fix(F286-DATA-R1.7): enrich advisory candidates with listing names
  662f798  test(F286-DATA-R1.7): add advisory name enrichment tests

fire-vault (main):
  (本 docs commit)
```

Codex pre-commit: fix / test ともに OK。

## 注意 / 観察

1. **listings-db は staging を使った**: F100 が J-Quants V2 から
   `market_listings` を fetch して書き込むのは staging も production
   も同じスキーマ。本 enrichment は read-only なので staging を
   読んでも production を読んでも結果は同じ。F282 weekly snapshot
   で 5/18 月曜 07:00 JST に staging が production をミラーで上書き
   されるが、`market_listings` の中身は production 側でも保持。
2. **scale_category / market_name も拾える**: 現状は company_name のみ
   引いているが、`market_listings` には scale_category (= TOPIX Small
   1 等) や market_name (= プライム/グロース) も入っており、将来の
   Advisory UX 拡張 (例: 「プライム」「Topix Small」のラベル付加) に
   利用可能。
3. **F111-R4 wiring で name を埋める方が根本的**: 本 R1.7 は
   "downstream patch" だが、上流 (F286-DATA-R1.6 の F111-R4 persistence
   段階) で name を埋めるのが理想。F286-DATA-R1.8 として F111-R4 側に
   `--listings-db` を追加する別タスクが候補。本 R1.7 で十分実用的
   ではあるので、優先度は低い。

## 次タスク

1. ★ F286-DATA-R1.7 完了 (本書、helper module + runner + 23 tests)
2. HQ (Fujiwara) が次の方向を判断:
   - 案 Y1: F062-R5.6 として `--buyability-mode --listings-db` で
     本番 LINE 送信 (= 銘柄名付き判断カードを Fujiwara LINE app へ)
   - 案 Y2: F286-PNL-R1 Advisory Decision / Actual PnL Tracking 設計
3. 並走候補:
   - F286-DATA-R1.8 として F111-R4 persistence runner に
     `--listings-db` を追加 (= 上流で name を埋める根本対応)
   - FIRE-OPS-R0 再発防止策案 1 設計 (= production write 統一)
   - F286-DATA-R3 daily refresh cron 化
   - F282 weekly snapshot 次回 (2026-05-18 07:00 JST) 前に再送
