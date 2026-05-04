# 週次レビューテンプレ (F233)

FIRE プロジェクトの週次レビューを Obsidian で記入するためのテンプレ一式。

## 目的

毎週 30 分で以下を 1 画面に俯瞰する:

- ステージ位置 (Stage 0/1/2/3 × M-3〜M+3)
- 開発進捗 (TODO Excel 連携)
- 運用成績 (Stage 2 以降)
- データ・パイプライン健全性
- リスク・ブロッカー
- 学び・改善
- 来週の計画
- メタ・ジャーナル (任意)

時系列の生ログ ([[log.md]]) はそのまま残し、本テンプレはサマリ層として
棲み分ける。

## ファイル構成

```
_templates/weekly_review/
├── README.md                         (本ファイル)
├── weekly_review_template.md         (本体テンプレ)
└── examples/
    └── weekly_review_2026-05-04_W18.md   (記入例)
```

## 使い方

### 新規ファイル作成

#### A. 手動コピー (一番確実)

```bash
cp ~/fire-vault/_templates/weekly_review/weekly_review_template.md \
   ~/fire-vault/05_weekly_review/weekly_review_2026-05-10_W19.md
```

その後 Obsidian で開いて `{{date:YYYY-MM-DD}}` 等のプレースホルダを実値に
置換 (Obsidian 上では既に展開された状態で開かれない点に注意)。

#### B. Obsidian core Templates plugin (推奨)

1. Obsidian 設定 → コアプラグイン → テンプレート → テンプレートフォルダ
   の場所を `_templates/weekly_review` に設定
2. 新規ファイル `05_weekly_review/weekly_review_2026-05-10_W19.md` を作成
3. コマンドパレット → 「テンプレート: テンプレート挿入」→
   `weekly_review_template` を選択
4. `{{date:YYYY-MM-DD}}` / `{{time:HH:mm}}` / `{{date:YYYY-[W]WW}}` が
   自動展開される

#### C. Templater plugin (将来導入時)

Templater 導入時は本テンプレ内の `{{...}}` を以下に一括置換:

| 既存 (core Templates) | Templater |
|---|---|
| `{{date:YYYY-MM-DD}}` | `<% tp.date.now("YYYY-MM-DD") %>` |
| `{{date:YYYY-[W]WW}}` | `<% tp.date.now("YYYY-[W]WW") %>` |
| `{{time:HH:mm}}` | `<% tp.date.now("HH:mm") %>` |
| `{{title}}` | `<% tp.file.title %>` |

現状 Templater 未導入。core Templates で十分。

### 命名規則

```
weekly_review_YYYY-MM-DD_WNN.md
```

- `YYYY-MM-DD` = レビュー記入日 (= 月曜朝記入なら月曜の日付)
- `WNN` = ISO 週番号 (`{{date:YYYY-[W]WW}}` で自動展開、ゼロパディング 2 桁)
- 例: `weekly_review_2026-05-04_W18.md` (W18 = 2026-04-27〜05-03 を振り返る)

### 配置先

```
05_weekly_review/weekly_review_YYYY-MM-DD_WNN.md
```

`05_weekly_review/` 直下にフラット配置。年で分けたい場合は将来
`05_weekly_review/2026/` に移行可能 (現状はファイル数少なくフラットで OK)。

### 推奨実施タイミング

- **第一推奨**: 毎週日曜 21:00 (週内に学びを定着、来週月曜 09:00 始動に
  間に合う)
- **第二推奨**: 月曜朝 08:00 (寝起きで前週を振り返る、ただし材料起因
  銘柄の事前リサーチ時間と競合する点に注意)
- **バックフィル**: 火曜以降にずれ込んだら省略せず必ず書く (連続性が
  最大の価値)

## TODO Excel との連携

### Wikilink 記法

タスク ID は Obsidian Wikilink で記入:

```markdown
- [x] [[F267_features_pipeline|F267]]: extract_features batch モード + universe
- [ ] [[F268_announcements_backfill|F268]]: announcements 過去 6 ヶ月遡及取得
```

ノートが `02_todo/` に未作成でも記入可 (リンク切れ表示になるが、将来
ノート作成で自動接続)。

### TODO_Master Google Sheets との同期

現状は手動コピペ運用:

1. 週次レビュー記入時、`02_todo/` の status を確認
2. 完了したタスクを Google Sheets でも「未着手 → 完了」に更新
3. 新規派生タスク (F267, F268, F269 のような) があれば Sheets にも追加
   起票

将来 F234 候補で Google Sheets API 自動同期検討。

## データ取得 SQL (セクション 4 用)

`~/fire/data/fire.db` (SQLite) にクエリ。Run a/b/c が走行中の時間帯は DB
ロック競合を避けるため、Chain 完了後に取得すること。

### features 行数 / カバレッジ

```sql
-- 総行数
SELECT COUNT(*) FROM features;

-- distinct (symbol, dt) ペア数
SELECT COUNT(DISTINCT symbol || '|' || dt) FROM features;

-- Core500 カバレッジ (universe ファイル参照)
-- universe 件数 = ~/fire/data/universe/core500.txt の行数 - コメント
-- 期間内 (例 直近 120 営業日) を where 句で絞ってカウント
```

### announcements 行数 / カバレッジ

```sql
SELECT COUNT(*) FROM announcements;
SELECT MIN(announced_date), MAX(announced_date) FROM announcements;
SELECT COUNT(DISTINCT announced_date) FROM announcements;
```

### market_prices_daily

```sql
SELECT COUNT(*) AS rows,
       COUNT(DISTINCT symbol) AS symbols,
       COUNT(DISTINCT date) AS days,
       MIN(date), MAX(date)
FROM market_prices_daily;
```

### パターン状態

```sql
SELECT layer, status, COUNT(*) AS n
FROM patterns
GROUP BY layer, status
ORDER BY layer, status;
```

active = `status='approved_active'` を主に観察。

### Chain 直近実行結果

```sql
SELECT run_id,
       n_ticks,
       n_events,
       status,
       started_at,
       ended_at
FROM paper_live_runs
ORDER BY started_at DESC
LIMIT 5;
```

### F053 promotion_check

```bash
cd ~/fire
.venv/bin/python -m simulation.paper_live --check-promotion <run_id>
```

### F241 live_advisory_check

```bash
cd ~/fire
.venv/bin/python -m simulation.paper_live --check-live-advisory <run_id> \
    --skip-line --skip-emergency       # Stage 3 直前以外はスキップ可
```

## 過去レビューのインデックス化

ファイル数が 4 件を超えたら `_templates/weekly_review/index.md` で集約管理
(本タスクスコープ外、F234 候補で起票)。それまでは Obsidian の File Explorer
で `05_weekly_review/` を直接開く運用。

## 月次レビューとの関係

月初の月次レビュー (`06_monthly/` 配下、現状未作成) は本週次レビュー 4〜5 件を集約して
書く。月次レビュー側で「今月の学び」を要件書 v3.x への反映候補として
正式起票する流れ。

## 関連リンク

- 元タスク: [[F233_週次レビューテンプレ整備]]
- 要件書: 第 32 章 (本番移行基準) / 第 38 章 (運用モード) /
  第 40 章 (実装ロードマップ)
- 関連: [[F230_Paper_Live_batch_replay]] / [[F241_live_advisory_check]]
