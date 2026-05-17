# FIRE JPX-MARKET-EXPLORER-OPENCLAW-ACQUISITION-DESIGN (2026-05-18)

doc_id: FIRE-JPX-MARKET-EXPLORER-OPENCLAW-ACQUISITION-DESIGN-2026-05-18
status: 設計 / impl は別 wave (= R3-2 / R3-3)、本 wave は doc 作成のみ
HQ marker: (= 本 doc commit/push は別 wave、impl は別 HQ marker)
related:
- [[F111_JPX_MARKET_EXPLORER_EXPORT_API_AUDIT_R1_5_2026-05-18]] (= 本 wave で §0 追記)
- [[F111_JPX_MARKET_EXPLORER_SEED_R1_2026-05-17]] (= R1 受け皿、main merge 済)
- [[JPX_MARKET_EXPLORER_AGENT_CHOICE_2026-05-18]] (= OpenClaw 一択)

---

## §1 経緯と前提

R1.5 operator 手動確認 (= 2026-05-18) の結果:

| 観点 | 結果 |
|---|---|
| 公式 API docs / link | なさそう |
| CSV / Excel export | **なし** |
| 表コピー (= 画面選択 → クリップボード) | できなさそう |
| URL share | できそう |
| 利用規約の自動取得制限 | 明示なし、ただし scraping 風機能は画面側で抑止 |

→ URL share だけでは表データ取得不能 → **OpenClaw 取得補助 R3** を採用。

---

## §2 OpenClaw が「やること / やらないこと」

### §2.1 やること (= R3 scope 内)

- Fujiwara 指定の JPX screen URL を **dedicated Chromium で開く**
- 表 DOM を snapshot 取得し、**code 列 + 銘柄名列** を読み取る
- `/tmp/fire_jpx_openclaw_capture/<date>/captured.txt` に保存
  (= R1 manual text 互換 format)
- **Fujiwara 人間 review 後**、既 R1 seed runner に渡す
- gateway log に URL / 時刻 / 件数 / 失敗有無を保存

### §2.2 やらないこと (= R3 scope 外、完全禁止)

| 項目 | 理由 |
|---|---|
| DB write (= production / develop / staging) | R3 は read-only |
| LINE 送信 | 本番送信は別 HQ marker |
| 自動発注 | FIRE「崩してはならない前提 1」 |
| 楽天 / iSPEED 操作 | 完全別 scope |
| production DB 操作 | 同上 |
| token / env / secret 値表示 | log / capture file に出さない |
| 個人 account login 自動化 | login 不要 page のみ操作 |
| cookies 書込 (= session 永続化) | profile 毎回新規、session-less |
| burst 取得 / 連続多 URL | 低頻度 (= 1 取得 / 数分)、1 fire = 1 URL |
| 他サイトへの自動遷移 | jpx-explorer.com 配下のみ allow |
| 取得データ再配布 | FIRE 内部のみ |
| schema migration | 別 HQ marker |
| cron 自動 trigger | R3 では Fujiwara 手動のみ (= R4 で別 HQ 強検討) |

---

## §3 取得 flow (= 6 step、1 取得サイクル)

```
step 0: Fujiwara が JPX 画面で screen 条件を作成
        → URL bar から URL コピー
        → openclaw に 手動 trigger (= 1 取得 1 URL)

step 1: openclaw browser open <URL>
        → dedicated Chromium 起動 (= profile fire-jpx-r3 隔離)
        → URL load、login prompt 出ない確認
        → load timeout 30 sec、超過で 即停止

step 2: openclaw browser snapshot
        → DOM snapshot 取得 (= 表領域のみ、ref 一覧)
        → 表セル の code / name 列を抽出

step 3: local file 保存
        → /tmp/fire_jpx_openclaw_capture/<date>/captured.txt
        → format: "<code> <name>" 1 行 1 銘柄 (= R1 manual text 互換)
        → URL / 取得時刻 / 件数 を header コメント (= "# url=...") で保存

step 4: Fujiwara が local file を確認 (= 人間 review)
        → ファイル内容 OK なら次 step、NG なら 削除 + 再 step 1

step 5: FIRE R1 seed runner call
        → .venv/bin/python -m scripts.jobs.run_f111_jpx_market_explorer_seed_preview \
            --input-text /tmp/fire_jpx_openclaw_capture/<date>/captured.txt \
            --screen-name "<URL の screen 条件>" \
            --screen-conditions "<URL>" \
            --base-date YYYY-MM-DD \
            --output-dir /tmp/fire_jpx_seed/<date>

step 6: seed JSON 取得 → 後段 wave (= F111 hook / W2-B enrich / Paper Live)
```

(= step 1-3 は OpenClaw、step 0/4 は人間、step 5/6 は既 R1 runner)

---

## §4 seed schema 拡張案 (= 1.0.0 → 1.1.0)

### §4.1 追加 field (= all optional、後方互換)

| field | 型 | 例 | 説明 |
|---|---|---|---|
| `screen_url` | str | "https://jpx-explorer.com/ja-JP/screener?..." | 取得元 URL (= 再現性 audit) |
| `screen_conditions` | str | "出来高 上位 100" | (= R1 既存、人間語 説明) |
| `acquired_by` | enum | "openclaw" / "manual" | 取得主体 |
| `acquisition_method` | enum | "url_share_browser_assist" / "manual" / "csv" / "local_text" | 取得方式 |
| `acquisition_notes` | str / null | "出来高 順 + 株価 < 1000" | 補足 (= 任意) |
| `captured_at_jst` | ISO8601 | "2026-05-18T08:30:00+09:00" | 表データ取得時刻 (= generated_at_jst とは別) |

### §4.2 互換性

- 既 R1 JSON (= 1.0.0) は 6 field 欠落でも valid
- 6 field は **all optional**
- schema_version を 1.1.0 へ bump
- runner CLI に `--screen-url` / `--acquired-by` / `--acquisition-method` /
  `--acquisition-notes` / `--captured-at-jst` を任意引数で追加

### §4.3 既 field との関係

- `source` (= "jpx_market_explorer_screener"): 不変、固定
- `import_method` (= R1 既) → `acquisition_method` が **より詳細**
- `created_by` (= doc 作成者) と `acquired_by` (= 取得主体) は別概念
- `generated_at_jst` (= runner 実行時刻) と `captured_at_jst` (= 表データ取得時刻) は別

---

## §5 安全境界 (= 多重 gate)

### §5.1 HQ marker 多重 gate

| gate | 内容 |
|---|---|
| HQ_APPROVE_JPX_SEED_R3_OPENCLAW_ACQUISITION_DESIGN | 本 doc 設計承認 |
| HQ_APPROVE_JPX_SEED_R3_OPENCLAW_LOCAL_CAPTURE | 実装承認 (= 1 取得 試行) |
| HQ_APPROVE_JPX_SEED_R3_FIRE_SEED_PIPELINE_SMOKE | 取得結果 → R1 runner smoke |
| HQ_APPROVE_JPX_BROWSER_AUTOMATION (= R4) | 将来 cron 自動化 (= 本 R3 では除外) |

### §5.2 R3 全段階共通制約

| 制約 | 内容 |
|---|---|
| URL 指定 | **Fujiwara 手動のみ**、cron / 自動指定 0 |
| 取得頻度 | 低頻度 (= 1 取得 / 数分以上、burst 禁止) |
| 失敗停止 | 3 連続 fail → 自動停止、HQ marker 失効 |
| 取得対象 | jpx-explorer.com 配下のみ、他 site 遷移 0 |
| 取得列 | code + name のみ |
| 取得結果 | local file のみ、DB write 0 |
| R1 runner 連携 | **人間 review 後**、自動 chain なし |
| profile | `--profile fire-jpx-r3` (= 既 fire profile と分離) |
| cookies | 0 (= session-less、毎回新規 profile) |
| login 自動化 | 禁止 |
| browser log | gateway log 隔離、外部送信 0 |
| レート違反 (= 429) | 即停止 + 24h cooldown |
| DOM 変化 | 即停止 + HTML snapshot 保存 + operator 通知 |
| 取得データ再配布 | 禁止 (= FIRE 内部のみ) |
| line_send_allowed | false 固定 |
| auto_order | false 固定 |

### §5.3 失敗時 停止条件 詳細

| 状況 | 動作 |
|---|---|
| URL load timeout (= 30 sec) | 即停止、log 残置 |
| 想定外 login prompt | 即停止 |
| DOM 変化 | 即停止 + HTML snapshot + operator 通知 |
| レート違反 (= 429) | 即停止 + 24h cooldown |
| 3 連続 fail | **自動停止**、HQ marker 失効 |
| 取得件数 0 | warn log + operator 通知 |
| 取得 row > 1000 | warn log + truncate (= 異常防御) |
| 想定外 dialog | dialog arm せず即停止 |

---

## §6 implementation plan (= 別 wave、3 + 将来 1)

### §6.1 R3-1: 設計 doc 作成 (= 本 wave)

- HQ marker: (= 本 wave で doc 作成、commit/push は HQ_APPROVE_JPX_SEED_R3_OPENCLAW_ACQUISITION_DESIGN)
- 実装: 0 (= doc のみ)
- 安全: 全 read-only

### §6.2 R3-2: OpenClaw local capture 実装

- HQ marker: HQ_APPROVE_JPX_SEED_R3_OPENCLAW_LOCAL_CAPTURE
- 内容:
  - `~/fire/scripts/openclaw/jpx_capture.sh` (= openclaw browser wrapper)
  - profile `fire-jpx-r3` 作成 (= 手動初回のみ)
  - exec-policy で jpx-explorer.com 配下のみ allow
  - 1 取得 試行 (= Fujiwara が URL 指定、smoke)
- 安全: §5 全制約遵守、cron 0、自動 trigger 0
- LOC 目安: shell script ~80 行 + docs

### §6.3 R3-3: FIRE seed pipeline smoke

- HQ marker: HQ_APPROVE_JPX_SEED_R3_FIRE_SEED_PIPELINE_SMOKE
- 内容:
  - R3-2 で取得した local text を 既 R1 runner に渡す smoke
  - schema 1.1.0 拡張 (= §4、新 6 field optional)
  - tests +10 程度 (= 新 field / acquisition_method / schema version)
  - smoke: candidate_count / safety_flags / 新 field 充填 確認
- 安全: R3-2 と同等 + R1 既制約遵守

### §6.4 R3-4 (= 将来、別 HQ 強)

- HQ marker: **HQ_APPROVE_JPX_BROWSER_AUTOMATION**
- 内容: OpenClaw cron で定時取得 (= URL は事前設定)
- 必須: R3-2/3 が 1 ヶ月以上安定 + TOS + robots.txt 再確認 + 倫理 OK
- 本 R3 設計では **scope 外**

---

## §7 R3 → R4 昇格条件

1. R3 (= 手動 trigger) が 1 ヶ月以上、3 fail 0 件で運用
2. operator が seed JSON 内容を全 review 済 (= 件数 / 鮮度 / 妥当性)
3. JPX TOS + robots.txt 再確認、変更なし
4. 失敗時 LINE 通知 path 設計済 (= 別 wave、本 R3 では LINE 送信 0)
5. **HQ_APPROVE_JPX_BROWSER_AUTOMATION** 発行 (= 別 HQ 強)

---

## §8 Codex 6 lane 相当 self-audit

| lane | 観点 | verdict | 摘要 |
|---|---|---|---|
| A | operator 結果反映 | APPROVE | R1.5 §0 追記、5 観点 + 結論 + 決定木 到達点 + 期待値 vs 実 + 次 wave |
| B | OpenClaw 設計 | APPROVE | 6 step flow + やる/やらない 明示 + profile 隔離 + 多重 gate |
| C | seed schema 拡張 | APPROVE | 6 field 全 optional、後方互換、schema 1.1.0 bump、CLI 任意引数 |
| D | safety 境界 | APPROVE | 多重 HQ marker / cron 0 / 低頻度 / 3 fail 停止 / 再配布禁止 / login 禁止 |
| E | next-wave 順序 | APPROVE | R3-1 (本 wave) → R3-2 capture → R3-3 smoke → R3-4 cron (別 HQ 強) |
| F | 運用リスク | APPROVE | DOM 変化 / TOS 改訂 / IP block / Fujiwara 不在 / 投資判断誤用 各対策明示 |

**結果**: 6 lane 全 APPROVE / CRITICAL 0 / HIGH 0
MEDIUM 1: openclaw browser snapshot の DOM ref 一覧抽出は OpenClaw 側 API
依存、実装 wave (= R3-2) で対応詳細

---

## §9 安全 gate (= 本 wave、純設計 + doc 作成)

| 項目 | 結果 |
|---|---|
| JPX サイト 自動操作 | 0 |
| ブラウザ自動操作 | 0 (= 設計のみ) |
| スクレイピング実装 | 0 (= 設計のみ) |
| 外部 HTTP 自動取得 | 0 |
| DB write / staging write | 0 |
| financials refresh | 0 |
| 実 J-Quants API call | 0 |
| TDnet / XBRL 取得 | 0 |
| token / env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| 楽天 / iSPEED 操作 | 0 |
| 自動発注 | 0 |
| Computer Use | 0 (= 本 R3 でも OpenClaw browser 経由のみ、Anthropic Computer Use API 不使用) |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| fire repo code 変更 | 0 (= 本 wave、impl は別 wave) |
| git add / commit / push | 0 (= 本 wave、commit は別 HQ marker) |
| PR 作成 / merge | 0 |
| schema migration | 0 |
| --no-verify / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |

3 DB md5 (= wave 期間中、完全不変):
- `data/fire.db`: b1df4673... ✓
- `data/fire.develop.db`: 085799da... ✓
- `data/fire.staging.db`: 71a63a19... ✓

---

## §10 次 wave 候補

| 優先 | wave | HQ marker |
|---|---|---|
| 1 | 本 doc + R1.5 doc 更新 commit/push | HQ_APPROVE_JPX_SEED_R3_OPENCLAW_ACQUISITION_DESIGN |
| 2 | OpenClaw local capture 実装 (= R3-2) | HQ_APPROVE_JPX_SEED_R3_OPENCLAW_LOCAL_CAPTURE |
| 3 | FIRE seed pipeline smoke (= R3-3) | HQ_APPROVE_JPX_SEED_R3_FIRE_SEED_PIPELINE_SMOKE |
| 4 | (= 将来、別 HQ 強) cron 自動化 (= R3-4) | HQ_APPROVE_JPX_BROWSER_AUTOMATION |

---

## §11 success 判定 (= 本 wave)

| 基準 | 結果 |
|---|---|
| R1.5 doc に operator 結果反映 | ✓ (= §0 追記、5 観点 + 結論 + 期待値 vs 実) |
| OpenClaw 取得補助 R3 設計 | ✓ (= 本 doc、§2-§6) |
| seed schema 1.1.0 拡張案 | ✓ (= §4、6 field optional、後方互換) |
| 安全境界 (= 多重 HQ + cron 0 + 3 fail 停止) | ✓ (= §5) |
| 次 wave 4 段 (= R3-1〜R3-4) 明示 | ✓ (= §6, §10) |
| DB / API / LINE / token / ブラウザ操作 | 0 (= §9) |
| fire repo 変更 | 0 (= §9) |
| commit/push | 別 wave (= goal 指示通り) |

→ **全成功基準 充足、本 wave 設計部 完了 ✓**
   (= 残: 本 doc + R1.5 doc 更新の commit/push を別 wave HQ marker で実施)
