# FIRE F111-JPX-MARKET-EXPLORER-EXPORT-API-AUDIT-R1.5 (2026-05-18)

doc_id: FIRE-F111-JPX-EXPORT-API-AUDIT-R1.5-2026-05-18
status: 調査設計 / **operator 手動確認待ち** / WebFetch / scraping 不使用
HQ marker: (= 調査 wave、本 doc commit/push は別 wave)
related:
- [[F111_JPX_MARKET_EXPLORER_SEED_R1_2026-05-17]]
- [[JPX_MARKET_EXPLORER_AGENT_CHOICE_2026-05-18]]
- [[F062_OPS_SUMMARY_FRESHNESS_ADAPTER_DESIGN_2026-05-17]]
- [[NIGHT_R0_READ_ONLY_BATCH_DESIGN_2026-05-17]]

---

## §1 目的

JPX Market Explorer (= https://jpx-explorer.com/ja-JP/screener) の
**公式 API / CSV export / URL share / filter parameter** の可否を read-only
で確認するためのテンプレートを設計し、FIRE JPX Seed R2 以降の優先順位
方針を確定する。

本 wave は **設計 + operator 手動確認テンプレート整理のみ**:
- WebFetch / 外部 HTTP / スクレイピング / ブラウザ自動操作 = **0 件**
- 実 JPX サイト確認は **operator (Fujiwara) が手動 browser で実施**
- FIRE 側コード変更 = 0 件、DB write = 0 件

---

## §2 成果物

### §2.1 設計 doc 3 file

| path | 内容 |
|---|---|
| `/tmp/fire_jpx_export_api_audit/jpx_export_api_audit.md` | API/export/URL share 38 確認項目テンプレート + 確認手順 7 step |
| `/tmp/fire_jpx_export_api_audit/recommendation.md` | 6 シナリオ A-F + 決定木 + 共通安全境界 + 推奨実行順序 |
| `~/fire-vault/03_design/F111_JPX_MARKET_EXPLORER_EXPORT_API_AUDIT_R1_5_2026-05-18.md` | 本 doc (= 統合 + Codex 8 lane + 安全 gate + 推奨予想) |

### §2.2 確認項目総数 (= operator 手動 約 30 分想定)

| カテゴリ | 項目数 | 用途 |
|---|---|---|
| API 可否 (= §2 jpx_export_api_audit.md) | 10 | API 採用可否判定 |
| API FIRE 内部利用懸念 | 4 | TOS 違反リスク確認 |
| CSV / Excel export | 10 | export 採用可否 + R1 互換性 |
| URL share / filter parameter | 8 | URL 安定性 + 条件保存 |
| TOS / 安全 | 8 | 自動取得 / scraping 許否 |
| robots.txt | 1 | crawl 制限 |
| **合計** | **41** | |

---

## §3 推奨方針 (= operator 確認後の決定木、最有力 想定)

### §3.1 決定木 サマリ

```
Q1: 公式 API が個人 / FIRE 内部利用で TOS OK か?
  YES → シナリオ A: API クライアント実装
  NO →
    Q2: 画面で CSV / Excel export 可能か?
      YES + 件数十分 → シナリオ B: 手動 DL → R1 runner (★ 最有力予想)
      YES + 件数不足 → シナリオ B + C
      NO →
        Q3: URL share / filter parameter 安定か?
          YES → シナリオ C + D
          NO → シナリオ D: local parser のみ
```

### §3.2 推奨優先順位 (= TOS OK 前提)

| 優先 | シナリオ | 採用条件 | 別 wave 要否 |
|---|---|---|---|
| 1 | A. 公式 API | API docs + TOS OK + rate 緩 | HQ_APPROVE_JPX_SEED_R2_API_CLIENT |
| 2 | **B. CSV export (画面 + 手動 DL)** ★ | export ボタンあり + TOS OK | **本 R1 で即対応、別 wave 不要** |
| 3 | C. URL share + 画面コピー | URL 再現性あり | 本 R1 で対応 |
| 4 | D. local CSV/HTML/Excel parser | A-C 困難時 | HQ_APPROVE_JPX_SEED_R2_LOCAL_PARSER |
| 5 | E. OpenClaw cron 取得補助 | B/D 稼働後 | HQ_APPROVE_JPX_SEED_R3_OPENCLAW_CRON |
| 6 | F. browser automation | 最終手段 | HQ_APPROVE_JPX_BROWSER_AUTOMATION (= 別 HQ 強) |

→ **最有力予想**: シナリオ B (= 既 R1 runner で即運用可、別 wave 不要)

---

## §4 R2 / R3 / R4 案

### §4.1 R2 (= local 強化、6 月目処)

並行可能 2 wave:
- **JPX-SEED-R2-API-CLIENT** (= シナリオ A 採用時のみ)
  - aiohttp + retry + rate limit
  - 認証 key を ~/.env.jpx 別管理
  - tests + smoke
- **JPX-SEED-R2-LOCAL-PARSER** (= シナリオ D 採用時)
  - lxml で HTML table parse
  - openpyxl で Excel 直読
  - HTTP 0、入力は local file のみ
  - tests + smoke

### §4.2 R3 (= OpenClaw cron 化、R2 稼働後)

- **JPX-SEED-R3-OPENCLAW-CRON**
  - HQ marker: HQ_APPROVE_JPX_SEED_R3_OPENCLAW_CRON
  - OpenClaw cron で R1 runner を定時 fire (= JPX 直接アクセス 0)
  - profile `fire-jpx-r3` で 既 fire profile と分離
  - exec-policy で R1 runner 以外 deny
  - file watcher: `/tmp/fire_jpx_inputs/` に operator が file drop
  - browser / cookies / HTTP 0

### §4.3 R4 (= browser automation、最終手段)

- **JPX-SEED-R4-BROWSER-AUTOMATION**
  - HQ marker: **HQ_APPROVE_JPX_BROWSER_AUTOMATION** (= 別 HQ 強)
  - **必須事前条件**:
    - TOS で自動取得 / scraping 明示許可
    - robots.txt で /screener disallow なし
    - rate 遵守 (= 1 取得 / 数分、burst 禁止)
    - login 不要 page のみ
    - 取得 データ FIRE 内部のみ、再配布禁止
  - OpenClaw browser subcommand 使用 (= dedicated Chromium)
  - 3 fail で自動停止、429 で 24h cooldown

---

## §5 利用規約 / 安全境界 (= 共通、R1-R4 通底)

| 項目 | 状態 |
|---|---|
| `line_send_allowed` | false 固定 |
| `auto_order` | false 固定 (= FIRE「崩してはならない前提 1」) |
| DB write | staging のみ、production / develop 不触 |
| financials refresh / retry | 0 (= 別 wave) |
| derived regen / signal regen | 0 (= 別 wave) |
| W2-B rerun (= フル再計算) | 0 (= 別 wave) |
| 実 JPX site HTTP call | 0 (= 本 R1.5 範囲外、R2-R4 で段階的) |
| スクレイピング / Browser automation | 0 (= R4 のみ、別 HQ 強) |
| Computer Use / Playwright / Selenium | 0 (= R4 でも OpenClaw browser 経由) |
| token / .env / secret 値表示 | 0 |
| LINE 送信 | 0 (= 別 HQ marker) |
| launchctl / plist / cron 編集 | 0 (= 本 R1.5 範囲外) |
| 楽天 / iSPEED 操作 | 0 (= 完全別 scope) |
| 取得データ再配布 | 禁止 (= 全 R1-R4) |
| 個人 account login 自動化 | 禁止 (= R4 でも) |
| robots.txt 違反 / レート違反 | 禁止 (= R4 でも 429 即停止) |

---

## §6 期待される最有力結論 (= 一般 screener 傾向、参考予想)

(= 実 operator 確認で覆る可能性あり)

| 観点 | 期待値 | 確率予想 |
|---|---|---|
| 公式 API 公開 | 不在 or 有料 | 60-70% NG |
| CSV / Excel export | あり | 60-70% OK ★ |
| URL share 安定性 | SPA で session-based、安定性低 | 40-50% NG |
| TOS 個人利用 | OK | 80-90% OK |
| TOS 自動取得 | 禁止 or 明示なし | 50-60% NG |
| TOS 再配布 | 禁止 | 80-90% NG (= FIRE 内部利用なので問題なし) |
| robots.txt /screener disallow | 不在 (= SPA 一般) | 70-80% OK |

→ **最有力予想**: シナリオ B (= 手動 DL → R1 runner) で即運用、
   R3 (= OpenClaw cron) で operator 工数削減、R4 (= browser automation)
   は不要 / 別 HQ 強で慎重検討

---

## §7 Codex 8 lane 相当 self-audit

(= 本 wave は CLI 並列実行ではなく self-audit 表で代替、merge / impl 別 wave 時に CLI 要)

| lane | 観点 | verdict | 摘要 |
|---|---|---|---|
| A | 公式 API 可否 | APPROVE | API 10 + FIRE 懸念 4 = 14 項目テンプレ、API 採用判断軸明確 |
| B | CSV / export 可否 | APPROVE | export 10 項目テンプレ + R1 互換性 5 項目、シナリオ B が最有力 |
| C | URL share / filter | APPROVE | URL 8 項目テンプレ、SPA session-based の場合 安定性低想定 |
| D | 利用規約 / 安全 | APPROVE | TOS 8 + robots.txt、自動取得 / 再配布 / login 自動化 禁止境界明示 |
| E | R2 local parser 案 | APPROVE | lxml / openpyxl 限定、HTTP 0、tests + smoke 明示 |
| F | OpenClaw / R3 案 | APPROVE | profile 分離 + exec-policy + file watcher、JPX 直接アクセス 0 |
| G | browser automation / R4 境界 | APPROVE | TOS + robots.txt + rate + login 不要 + 3 fail 停止、別 HQ 強必須 |
| H | next-wave sequencing | APPROVE | R1.5 → 判定 → B 即運用 / A or D 別 wave → R3 → R4 段階 |

**結果**: 8 lane 全 APPROVE / CRITICAL 0 / HIGH 0
MEDIUM 1: シナリオ A の認証 key を secrets として ~/.env.jpx で分離管理する設計
は本 doc で言及済だが、実装時の secrets reload / log 漏洩 enforcement は別 wave 詳細

---

## §8 安全 gate (= 本 wave、純調査 + 設計)

| 項目 | 結果 |
|---|---|
| WebFetch / 外部 HTTP 自動取得 | 0 (= 本 wave 不使用) |
| 実 JPX サイト アクセス | 0 (= operator 手動確認の指示のみ) |
| スクレイピング実装 | 0 |
| ブラウザ自動操作 | 0 |
| Computer Use | 0 |
| Playwright / Selenium / webdriver | 0 |
| DB write / staging write / production-develop write | 0 |
| financials refresh / derived regen / signal regen / W2-B rerun | 0 |
| 実 J-Quants API call | 0 |
| TDnet / XBRL 取得 | 0 |
| token / env / secret 値表示 | 0 |
| LINE 送信 | 0 |
| launchctl / plist / cron 編集 | 0 |
| workflow 変更 | 0 |
| fire repo code 変更 | 0 |
| git add / commit / push (= fire repo) | 0 |
| git add / commit / push (= fire-vault) | 0 (= 本 wave 範囲外、別 wave) |
| PR 作成 / merge | 0 |
| schema migration | 0 |
| auto-order / 楽天 / iSPEED | 0 |
| --no-verify / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |

3 DB md5 (= wave 期間中、完全不変):
- `data/fire.db`: b1df4673... ✓
- `data/fire.develop.db`: 085799da... ✓
- `data/fire.staging.db`: 71a63a19... ✓

---

## §9 next wave 候補

### §9.1 直近 (= 本 wave 後、5/18-5/19)

| 優先 | wave | HQ marker |
|---|---|---|
| 1 | 本 audit doc commit/push | HQ_APPROVE_JPX_EXPORT_API_AUDIT_DOC_COMMIT_PUSH |
| 2 | **operator 手動確認** (= §6 step 1-7 実施) | (= HQ 不要、operator 30 分手作業) |
| 3 | 確認結果を本 doc に追記 + 推奨確定 | (= HQ 不要、本 doc 更新 + 別 commit 可) |

### §9.2 短期 (= 確認結果次第、5/末-6/初)

| 優先 | シナリオ 確定 | 着手 wave | HQ marker |
|---|---|---|---|
| B (CSV 手動 DL) | (★ 最有力) | 既 R1 runner で即運用 | (= 不要、operator 手動運用開始) |
| A (公式 API) | 別 wave | JPX-SEED-R2-API-CLIENT | HQ_APPROVE_JPX_SEED_R2_API_CLIENT |
| D (local parser) | 別 wave | JPX-SEED-R2-LOCAL-PARSER | HQ_APPROVE_JPX_SEED_R2_LOCAL_PARSER |

### §9.3 中期 (= 6 月)

| 優先 | wave | HQ marker |
|---|---|---|
| - | JPX-SEED-R3-OPENCLAW-CRON | HQ_APPROVE_JPX_SEED_R3_OPENCLAW_CRON |

### §9.4 将来 (= 7 月以降、別 HQ 強)

| 優先 | wave | HQ marker |
|---|---|---|
| - | JPX-SEED-R4-BROWSER-AUTOMATION | **HQ_APPROVE_JPX_BROWSER_AUTOMATION** |

---

## §10 success 判定 (= 本 wave)

| 基準 | 結果 |
|---|---|
| API / export / URL share の可否 整理 | ✓ (= 41 項目テンプレート + 6 シナリオ 決定木) |
| R2 / R3 / R4 の優先順位 明確化 | ✓ (= §4、各 wave 別 HQ marker 明示) |
| スクレイピング実装 | 0 (= §8 安全 gate) |
| DB / API / LINE / token / ブラウザ自動操作 | 0 (= §8) |
| fire repo 変更 | 0 (= §8、本 wave 一切なし) |
| vault doc 作成 | ✓ (= 本 doc + /tmp 2 file、計 3 file) |
| commit / push | 別 wave (= goal 指示通り) |

→ **全成功基準 充足、本 wave 完了 ✓**
   (= 残: vault doc commit/push を別 HQ marker で実施推奨)
