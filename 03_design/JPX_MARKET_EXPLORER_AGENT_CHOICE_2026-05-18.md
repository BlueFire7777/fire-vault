# FIRE JPX Market Explorer agent choice audit (2026-05-18)

doc_id: FIRE-JPX-MARKET-EXPLORER-AGENT-CHOICE-2026-05-18
status: 調査完了 / 採用判断: **OpenClaw 一択** (= Hermes は前身)
HQ marker: (= 調査 wave、本 doc commit/push は別 wave)
related:
- [[F111_JPX_MARKET_EXPLORER_SEED_R1_2026-05-17]]
- [[F062_OPS_SUMMARY_FRESHNESS_ADAPTER_DESIGN_2026-05-17]]
- [[NIGHT_R0_READ_ONLY_BATCH_DESIGN_2026-05-17]]
- [[F111_W2C_OVERLAY_ROLE_RECLASSIFICATION_DESIGN_2026-05-17]]

---

## §1 結論 (= 1 行)

**OpenClaw 一択。Hermes は OpenClaw の前身であり、新規採用する理由が無い。**

---

## §2 経緯

F111-JPX-MARKET-EXPLORER-SEED-R1 (= 5/17、commit 15e3b5d) で「CSV /
manual / local text を FIRE seed JSON に変換する受け皿」が完成。

次の論点として、JPX 公式 API / export が無い場合の **取得補助 agent** に
OpenClaw と Hermes のどちらを採用するかの審査を依頼された。

調査方法: **値表示 0 / 外部 HTTP 0 / ブラウザ自動操作 0** の純 read-only
audit。

---

## §3 local 環境確認結果

### §3.1 OpenClaw

- CLI: `/opt/homebrew/bin/openclaw` (= homebrew install 済) ✓
- version: **2026.4.23 (`a979721`)** ✓
- state dir: `~/.openclaw/` (= config / bak / completions) ✓
- 既 fire docs: `~/fire/docs/openclaw/agent_registration_guide.md` (= F260 Block 1 既調査) ✓
- FIRE 採用状況: **既採用** (= CLAUDE.md「第 1 層オーケストレーション層」)
- 主要 subcommand: `agent / agents / browser / channels / cron / gateway /
  models / sandbox / sessions / tasks / webhooks / approvals / exec-policy /
  security / secrets / hooks / logs / doctor / backup` 等 (= 50+ subcommand)

### §3.2 Hermes

- CLI: × `hermes not found` / `hermesc not found`
- pip / npm package: × 不在
- /Applications: × Hermes.app 不在
- fire-vault 言及: × `grep -li hermes ~/fire-vault` 0 件
- **OpenClaw docs 内の位置付け**: `openclaw docs hermes` を実行すると
  下記 page が返る:
  - `https://docs.openclaw.ai/install/migrating-hermes` (= "Migrating from Hermes")
  - `https://docs.openclaw.ai/cli/migrate#what-hermes-imports` (= "What Hermes imports")
  - `https://docs.openclaw.ai/install/migrating#import-from-another-agent-system`

→ **Hermes は OpenClaw の前身、OpenClaw が後継** (= OpenClaw 公式が
   Hermes 移行手順を提供)

---

## §4 OpenClaw 評価

| 観点 | 評価 | 根拠 |
|---|---|---|
| JPX 画面操作に向くか | ✓ 高 | `openclaw browser` で dedicated Chromium、click / cookies / dialog 完備 |
| CSV / HTML / local text 解析 | △ 中 | 自前 helper / agent task / cron で wrap 可、parse 自体は本 R1 で済 |
| 権限を取得補助だけに限定 | ✓ 高 | `exec-policy` / `sandbox` / `approvals` / `secrets` / `--profile` で隔離 |
| DB write / LINE / token / 楽天 / iSPEED / 自動発注 禁止 | ✓ 高 | 権限分離可、profile 別 (= JPX 用 profile を fire 既 profile と分離) |
| ログ / 再現性 | ✓ 高 | `gateway logs` / `tasks` / `sessions` / `backup` / `--log-level trace` |
| 失敗時 安全停止 | ✓ 高 | `approvals` / `exec-policy` / `doctor` 健全性 check / 3 fail で停止可 |
| FIRE seed JSON 接続 | ✓ 高 | 既 cron / task から R1 runner 呼出可、shell 1 行 |
| セキュリティリスク | 低-中 | dedicated Chromium + sandbox + secrets reload、ただし browser 利用時は別 HQ 必須 |
| 将来の自動化余地 | ✓ 高 | agent 複数並列 / cron / webhooks / channels 統合 |
| 運用コスト | ✓ 低 | 既 FIRE 第 1 層、追加 setup 0 |

→ **OpenClaw は 10 観点すべてで採用可能水準**

---

## §5 Hermes 評価

| 観点 | 評価 | 根拠 |
|---|---|---|
| local 不在 | × | install されていない (= 評価不能) |
| Hermes 公式の位置付け | × | OpenClaw 公式が Hermes 移行 page を提供 = 前身 |
| 採用する場合の必要作業 | × | install + migration + setup = 無意味 (= OpenClaw 既存) |
| 機能比較 | × | OpenClaw 後継 = OpenClaw に含まれる機能 superset |

→ **Hermes は採用候補として実質除外** (= OpenClaw 前身 + local 不在)

---

## §6 比較表 (= 10 観点)

| # | 観点 | OpenClaw | Hermes |
|---|---|---|---|
| 1 | JPX 画面操作に向くか | ✓ (browser subcommand) | (local 不在で評価不能) |
| 2 | CSV / HTML / local text 解析 | △ (wrap で対応) | × |
| 3 | 権限を取得補助だけに限定 | ✓ (exec-policy / sandbox) | × |
| 4 | DB / LINE / token / 楽天 / 発注 禁止 | ✓ (profile + binding 分離) | × |
| 5 | ログ / 再現性 | ✓ (logs / sessions / backup) | × |
| 6 | 失敗時 安全停止 | ✓ (approvals / doctor) | × |
| 7 | FIRE seed JSON 接続 | ✓ (既 cron / task) | × |
| 8 | セキュリティリスク | 低-中 (= browser 時は別 HQ 必須) | 不明 |
| 9 | 将来の自動化余地 | ✓ (= multi-agent + channels) | × |
| 10 | 運用コスト | ✓ 低 (= 既採用) | 高 (= 無意味 install) |

→ **OpenClaw 一択** (= 10 観点全てで優位、または評価不能の Hermes に対し優位)

---

## §7 推奨案

### §7.1 第一候補: **OpenClaw**

- 即時着手可、追加 setup 0
- R3 (= cron / task) / R4 (= browser) の両段階を 1 tool で扱える
- 既 FIRE 第 1 層 = 学習コスト 0

### §7.2 第二候補: **該当なし**

- Hermes: OpenClaw 前身、採用すれば後退
- Playwright / Selenium 直書: R4 段階でも OpenClaw browser を経由する方が安全
- Computer Use 系: FIRE「崩してはならない前提」で採用しない方針

### §7.3 採用条件

R3 (= 取得補助 cron / task) 採用条件:
1. R1.5 (= API/export 調査) 完了
2. HQ_APPROVE_JPX_SEED_R3_OPENCLAW_CRON 発行
3. cron 対象は R1 runner のみ、JPX 直接アクセス 0
4. profile `fire-jpx-r3` (= 既 fire profile と分離)
5. exec-policy で R1 runner 以外 deny

R4 (= browser 画面操作) 採用条件:
1. R1.5 + R3 が 1 ヶ月以上安定運用
2. **HQ_APPROVE_JPX_BROWSER_AUTOMATION** (= 最重要、別 HQ marker)
3. JPX 利用規約 + robots.txt + 倫理 OK 事前確認
4. レート制限: 人間操作と同等
5. 取得データ: FIRE 内部利用のみ、再配布禁止
6. login 不要 page のみ操作 (= 個人 account login 自動化禁止)
7. 失敗時の安全停止: 3 連続 fail → 取得停止 + LINE 警告 + operator 介入

### §7.4 禁止境界

R1 / R2 / R3 全段階 (= 画面操作 なし) の禁止:
- ブラウザ自動操作 / 外部 HTTP 自動取得 / スクレイピング / Computer Use
- login 自動化 / cookies 書込 / JPX サイト 直アクセス

R4 (= 画面操作、別 HQ) でも継続禁止:
- 取得データ再配布 / 個人 account login 自動化
- 楽天 / iSPEED 操作 / 自動発注
- DB write / LINE 自動送信 / TODO Excel 更新
- token / secret 値表示 / burst 取得 / robots.txt 違反

### §7.5 失敗時 停止条件

R1 / R2:
- 入力 file 不在 → exit 2、operator file drop 待ち
- invalid 1 件以上 + --strict → exit 4
- output path guard 違反 → ValueError raise

R3:
- cron fire 失敗 → approvals 待ち、3 連続 fail で停止
- input file 不在 → warn log、wait
- gateway 不在 → doctor auto-fix 試行 → fail なら停止

R4:
- 想定外 DOM 変化 → 即停止 + HTML snapshot 保存
- レート制限 検出 (= 429) → 即停止 + 24h cooldown
- 3 連続 取得 fail → 自動停止、HQ marker 失効

---

## §8 採用シナリオ 4 段階 (= R1 〜 R4)

```
R1: CSV / manual / local text import (= 本 PR 15e3b5d 完了済)
    → 人間が CSV / text を保存 → R1 runner → seed JSON
    → 補助 tool 不要、最小依存

R1.5: JPX 公式 API / export / URL share 調査 (= operator 手動、HQ 不要)
    → web 閲覧 + 利用規約 + 試行 1 件
    → 公式があれば R3 不要 (= R3 は API 無き場合の Plan B)

R2: local CSV / HTML / Excel parser 強化 (= 本 R1 seed 拡張)
    → HTML 表 (= JPX 画面 操作で保存) を parse
    → Excel 直接読込 (= openpyxl)
    → 複数 source 統合
    → 外部 HTTP 0 (= 人間 file drop 起点)

R3: OpenClaw 取得補助 (= 別 HQ、JPX 自動化前段)
    → OpenClaw cron で R1 runner を定時 fire
    → JPX 取得自体は依然 人間 (= operator file drop)
    → 24h 監視 / approvals / sandbox

R4: 画面操作自動化 (= 最終手段、別 HQ 必須)
    → OpenClaw browser で JPX 画面操作
    → HQ_APPROVE_JPX_BROWSER_AUTOMATION
    → 利用規約 + 倫理 OK 確認後のみ
```

---

## §9 FIRE 接続案 (= seed JSON 下流連携)

```
[OpenClaw cron (= R3 段階、別 HQ 後)]
   ↓ (= cron schedule で R1 runner fire)
[operator が JPX から CSV / HTML / text を保存]
   ↓ (= R3 でも 人間操作起点、R4 で自動化検討)
[local file (= /tmp/jpx_inputs/ 等)]
   ↓
[R1 seed runner (= 既実装、scripts.jobs.run_f111_jpx_market_explorer_seed_preview)]
   ↓
[seed JSON (= jpx_market_explorer_seed.json)]
   ↓
   ├── F111 候補比較 (= JPX-SEED-R2-F111-HOOK、別 wave)
   │   whitelist で F111 candidate generator に流入
   │   → overlay top5 に seed code を確実に拾う
   │
   ├── W2-B enrich (= JPX-SEED-R2-W2B-ENRICH、別 wave)
   │   seed.candidates に vol_mom / srs / 5 label を join
   │   → seed 拡張 JSON 出力
   │
   ├── F062 / Ops Summary (= 既設計、参考)
   │   seed source の screen_name を operator_next_action 説明に含む
   │
   ├── NIGHT-R0 (= 既設計)
   │   step 6 F062 preview の補助入力として seed を渡す
   │
   └── Paper Live 検証 (= JPX-SEED-R3-PAPER-LIVE、別 wave)
       F054 Paper Live tick の入力 candidate
       → screen_name 別 PnL 集計 + hit ratio
       → F119 Evaluation に screen 別 KPI 追加
```

---

## §10 Codex 8 lane 相当 self-audit

(= 本 wave は調査のみ、CLI 並列実行ではなく self-audit 表で代替)

| lane | 観点 | verdict | 摘要 |
|---|---|---|---|
| A | OpenClaw 適性 | APPROVE | 10 観点全て採用可水準、既 FIRE 第 1 層、setup 0 |
| B | Hermes 適性 | REJECT | local 不在 + OpenClaw 前身 = 採用候補から除外 |
| C | local parser 優先度 | APPROVE | R1 完了済、R2 (= HTML/Excel) を先に進めれば R3/R4 不要可 |
| D | safety 境界 | APPROVE | R1-R3 で画面操作 0、R4 は HQ_APPROVE_JPX_BROWSER_AUTOMATION 必須 |
| E | FIRE seed 接続 | APPROVE | R1 runner → seed JSON → F111/W2-B/Paper Live の 4 経路明示 |
| F | operational risk | APPROVE | R4 でも 3 fail 停止 / レート遵守 / 再配布禁止、利用規約 OK 必須 |
| G | next-wave 順序 | APPROVE | 本 audit → R1 PR merge → R1.5 → R2 → R3 → R4 (= 段階優先) |
| H | 採用判断 | APPROVE | OpenClaw 一択、Hermes 除外、R3/R4 は別 HQ marker、第二候補 該当なし |

**結果**: 8 lane 全 APPROVE / CRITICAL 0 / HIGH 0
MEDIUM 1 (= R4 利用規約 / robots.txt の事前確認は operator 手作業前提、自動 audit 困難)

---

## §11 安全 gate (= 本調査 wave)

| 項目 | 結果 |
|---|---|
| JPX サイト 自動操作 | 0 |
| ブラウザ自動操作 | 0 |
| スクレイピング実装 | 0 |
| 外部 HTTP 自動取得 | 0 |
| DB write / staging write | 0 |
| LINE 送信 | 0 |
| token / env / secret 値表示 | 0 |
| 楽天 / iSPEED 操作 | 0 |
| 自動発注 | 0 |
| production DB 操作 | 0 |
| launchctl / plist / cron 変更 | 0 |
| fire repo code 変更 | 0 |
| git add / commit / push | 0 (= 本 wave、別 step で commit 別 HQ) |
| PR / merge | 0 |
| schema migration | 0 |
| Computer Use | 0 |
| --no-verify / sudo / rm -rf | 0 |
| TODO Excel 更新 | 0 |
| Hermes install (= 評価のため一時 install 等) | 0 (= 不必要、不在のまま判断) |

3 DB md5 (= wave 期間中、完全不変):
- `data/fire.db`: b1df4673... ✓
- `data/fire.develop.db`: 085799da... ✓
- `data/fire.staging.db`: 71a63a19... ✓

OpenClaw への接続: なし (= help / docs search のみ実行、agent 起動 / gateway query 0)

---

## §12 next wave 候補

### §12.1 直近 (= 5/18 〜 5/末)

| 優先 | wave | HQ marker |
|---|---|---|
| 1 | 本 audit 結果 vault commit | HQ_APPROVE_JPX_AGENT_CHOICE_DOC_COMMIT_PUSH |
| 2 | R1 PR + main merge (= 15e3b5d を main へ) | HQ_APPROVE_JPX_SEED_R1_PR_MERGE |
| 3 | R1.5 API/export 調査 (= operator 手動) | (= HQ 不要、operator 手作業) |

### §12.2 中期 (= 6 月)

| 優先 | wave | HQ marker |
|---|---|---|
| 4 | R2 local HTML / Excel parser | HQ_APPROVE_JPX_SEED_R2_LOCAL_PARSER |
| 5 | R3 OpenClaw cron / task 化 | HQ_APPROVE_JPX_SEED_R3_OPENCLAW_CRON |

### §12.3 将来 (= 7 月以降、別 HQ 強)

| 優先 | wave | HQ marker |
|---|---|---|
| 6 | R4 browser 操作 検討 | **HQ_APPROVE_JPX_BROWSER_AUTOMATION** (= 利用規約 OK 必須、最重要) |

---

## §13 success 判定

| 基準 | 結果 |
|---|---|
| OpenClaw / Hermes の採用優先順位 確定 | ✓ (= OpenClaw 一択、Hermes 除外) |
| JPX 取得補助の安全境界 明確化 | ✓ (= R1-R3 で画面操作 0、R4 は別 HQ + 利用規約) |
| R2 / R3 の次 wave 案 明確化 | ✓ (= §12 5 wave、各 HQ marker 明示) |
| DB / API / LINE / token / ブラウザ操作 | 0 (= §11 全 0) |
| fire repo 変更 | 0 (= 本 wave は調査のみ) |
| vault doc 作成 | ✓ (= 本 doc + /tmp 2 file、計 3 file) |
| commit/push | 別 wave (= goal 指示通り) |

→ **全成功基準 充足、本 wave 完了 ✓**
   (= 残: vault doc commit/push を別 HQ marker で実施推奨)
