---
type: design_record
id: F282
title: "環境分離 (3 環境化、Stage 3 開始前必須)"
version: v1.0
created: 2026-05-08
updated: 2026-05-08
related_tasks: [F281, F271, F058, F273, F274, F275, F276, F235, F266]
related_chapters: [12, 13, 19, 32, 34, 35]
---

# F282 環境分離 (3 環境化、Stage 3 開始前必須)

## §1. 概要

- ID: F282
- Phase: P9: 運用・保守
- Category: Infrastructure
- 要件書章: 該当なし (新規論点、要件書 v3.x で追記候補)
- Priority: 最優先
- Status: 設計中 → 実装着手予定 (2026-05-08)
- Owner: Claude (Mac mini) 実装、本部 (Claude.ai) 設計監督、Fujiwara 戦略判断 + 全 MR 承認 (Phase 1)
- 想定工数: 2-3 日 (約 13-17 時間)
- 着手日: 2026-05-08 予定
- 完了日: 2026-05-10 予定
- 依存: F281-Phase2-A 完了 (済)、test pattern archive 化 (済)、F271 v1.5 (済)
- 後続: 短縮版 Phase 3 (target_patterns 拡張 → F273_BRE 4,700 件評価)、Stage 3 開始 (5/30 ± 数日、Active Light モード)

### 目的

Mac mini が production DB を直接触る現状を解消し、production / staging / develop の 3 環境分離を Stage 3 実弾運用開始前に確立する。実弾運用中の試行錯誤・migration・bug fix を develop / staging で安全に並列実施できる基盤を整備し、F058 test pattern 残置のような構造的見落としの再発を防止する。

### 完了基準

- git: dev → develop rename + staging branch 新設 + main は production 用維持
- DB: fire.db (production) / fire.staging.db / fire.develop.db の 3 ファイル分離
- 環境変数: FIRE_ENV=production/staging/development で DB_PATH 自動切替
- wrapper script: 環境別実行コマンド整備 (例: bin/fire-dev / bin/fire-staging)
- pytest: 環境切替テスト追加 + 全 1295+ PASS 維持
- GitHub branch protection: main は直接 push 禁止、MR 経由必須
- 運用ガイド: 本ファイル (~/fire-vault/03_design/F282_environment_isolation_2026-05-08.md)
- 初期データ同期: production fire.db → staging.db / develop.db への snapshot 完了

---

## §2. 背景 (Fujiwara 指摘 2026-05-07)

2026-05-07、Fujiwara から環境分離の必要性を指摘される。

### Fujiwara 原文

> 「あとさ、今後の開発的に、フィーチャー(develop)、staging、本番環境の git 構造にしておいた方がよくない?
>   このままだとずっと本番環境を触ることになってしまわない?」

### 指摘の背景

本セッションで以下の構造的問題が連鎖的に発覚:

1. F281-Phase1 設計時の F200 既存実装見落とし (本部側 Vault 突合不在、F271 v1.4 §6-22-septenary 記録)
2. F281-Phase2-A 段階 A の positions schema 設計欠陥 (Codex CRITICAL 検出、run_id なしで他 run データ混入リスク)
3. approved_active = test_e2e_smoke pattern の 5 タスク連続見落とし (F058 → F273 → F274 → F275 → F276 → F281 を経て残置、F271 v1.5 §6-22-octonary 記録)
4. 本部側で「Mac mini に指示文を投げたかどうか」を Fujiwara 確認せず推測判定した認識ミス (本セッション内で発生)

これら 4 件の構造的見落としは、いずれも production DB を直接触っている現状の運用では「事故が起きてから気づく」事後検出となっている。環境分離により、production を保護しつつ develop で先行検証する経路を確立することが根本解決となる。

### Fujiwara 指摘の正当性

Mac mini Vault 突合 (2026-05-07) で判明した事実:

- 環境分離 (production/staging/develop) を議論した todo / 設計記録 / 要件書規定は **すべて不在**
- 過去議論ゼロ = 「過去に検討して止まった」のではなく「初めて議論される話題」
- 本部もこれまで一度も提案していなかった = **本部側の構造的見落とし**

つまり Fujiwara の指摘は FIRE プロジェクト全体で初めて出てきた論点であり、Stage 3 実弾運用開始前のこのタイミングが導入の絶好機。

### 技術的伏線の存在

`patterns/store.py:88` で DB_PATH 環境変数経由の切替可能設計が既に実装済 (F271 v1.5 観点 11 適用の Mac mini 突合で発覚):

```python
DB_PATH = Path(os.getenv("DB_PATH", str(FIRE_HOME / "data" / "fire.db")))
```

つまり「環境分離の実装」とは「既存の DB_PATH 環境変数を活用した運用ルール整備」が中心であり、新規大規模実装は不要。これが工数 2-3 日に収まる根拠。

### ストレージ的制約ゼロ

- ディスク空き: 847 GB
- production DB: 354 MB
- 3 環境化で約 1.06 GB 増加 (0.13% 増、影響実質ゼロ)

---

## §3. 環境構造 (案 W1)

### 3-1. production / staging / develop の役割

| 環境 | 役割 | データの性質 | 触れる主体 |
|---|---|---|---|
| production | Stage 3 実弾運用専用 | 実取引データのみ蓄積 (Stage 3 開始後) | Mac mini (paper_live_results 書き込みのみ)、Fujiwara (発注操作経由) |
| staging | production 模擬環境、本番投入前最終検証 | production からの定期 snapshot + 検証中の変更 | Mac mini (検証 + Backtest + Run a/b/c 模擬) |
| develop | 機能開発・自由実験環境 | production からの初期 snapshot + 開発中の変更、自由 reset 可 | Mac mini (試行錯誤、migration 試走、Backtest 等すべて) |

**役割の補足:**

**production:**
- Stage 3 実弾運用 (Lane A1 Active Light) が稼働する環境
- Mac mini が直接触る機会は最小化、paper_live_results / paper_live_positions の書き込みのみ (実取引データの蓄積)
- 設定変更・migration・新機能導入は MR 経由でしか反映されない

**staging:**
- production と同じ schema + 最新スナップショットデータ
- MR 1 (develop → staging) で受領した変更をテスト
- Run a/b/c 模擬実施、F053 promotion_check 検証、Backtest など
- 検証完了後は破棄 → 次の MR 検証用に再 snapshot
- Stage 3 開始前は **Run a/b/c の本番実施環境**として機能

**develop:**
- Mac mini が自由に試行錯誤できる環境
- migration 失敗時のロールバックも自由 (DB ファイル削除 → snapshot 復元)
- 「壊して試す」が可能になる ← 環境分離の最大のメリット
- 開発初期は production スナップショットから派生、定期 reset 推奨

### 3-2. git branch / DB / 環境変数の対応

| 環境 | git branch | DB ファイル | FIRE_ENV |
|---|---|---|---|
| production | main | `~/fire/data/fire.db` | production |
| staging | staging | `~/fire/data/fire.staging.db` | staging |
| develop | develop (現 dev rename) | `~/fire/data/fire.develop.db` | development |

**DB_PATH 環境変数による切替:**

`patterns/store.py:88` 既存実装を活用:

```python
DB_PATH = Path(os.getenv("DB_PATH", str(FIRE_HOME / "data" / "fire.db")))
```

FIRE_ENV から DB_PATH を導出する wrapper script (案):

```bash
# bin/fire-env-switch.sh
case "$FIRE_ENV" in
  production) export DB_PATH="$FIRE_HOME/data/fire.db" ;;
  staging)    export DB_PATH="$FIRE_HOME/data/fire.staging.db" ;;
  development) export DB_PATH="$FIRE_HOME/data/fire.develop.db" ;;
  *) echo "FIRE_ENV not set or invalid: $FIRE_ENV"; exit 1 ;;
esac
```

**branch 戦略:**

**main:**
- production 環境専用、Stage 3 実弾運用稼働中
- 直接 push 禁止 (GitHub branch protection で強制)
- MR 経由でのみ更新 (Fujiwara 承認必須、Phase 1/2 共通)
- merge commit のみ許可 (squash / rebase は禁止、履歴保持)

**staging:**
- production 模擬環境、検証用
- MR 1 (develop → staging) で受領
- Codex CI 通過 + 本部レビュー (Phase 1: Fujiwara 承認 / Phase 2: 部分本部委譲)
- 検証完了 → MR 2 (staging → main) 起票

**develop:**
- 現 dev branch を rename
- Mac mini が自由に commit (Codex pre-commit 一発通過必須)
- 機能完成時に MR 1 起票

**既存 dev branch の扱い:**

現 `~/fire dev` branch を develop に rename。理由:
- 過去の commit 履歴を完全保持 (rename は履歴に影響なし)
- 既存 commit (cadfe61 等) の参照が壊れない
- F281-Phase2-A 完了済の状態を develop に引き継ぎ

**`~/fire-vault` の branch 戦略:**

`~/fire-vault` は引き続き main branch のみで運用。理由:
- Vault は設計記録・要件書・ログが中心、環境別の意味が薄い
- `~/fire-vault main` = 全環境共通の Vault ベース
- ただし環境別の運用ガイド (例: staging 検証手順) は Vault に章別整理

**branch / DB / FIRE_ENV の整合性チェック:**

wrapper script で起動時にチェック:

```bash
# bin/fire-check-env.sh (起動時実行)
CURRENT_BRANCH=$(git -C ~/fire rev-parse --abbrev-ref HEAD)
case "$CURRENT_BRANCH" in
  main)    EXPECTED_ENV="production" ;;
  staging) EXPECTED_ENV="staging" ;;
  develop) EXPECTED_ENV="development" ;;
  *) echo "WARNING: branch $CURRENT_BRANCH not mapped"; exit 1 ;;
esac

if [ "$FIRE_ENV" != "$EXPECTED_ENV" ]; then
  echo "ERROR: branch=$CURRENT_BRANCH but FIRE_ENV=$FIRE_ENV"
  echo "Expected FIRE_ENV=$EXPECTED_ENV"
  exit 1
fi
```

これにより「develop branch に居るのに production DB を触る」事故を構造的に防止。

---

## §4. 実装方針

### 4-1. 既存伏線 (DB_PATH 環境変数) の活用

F271 v1.5 観点 11 適用の Mac mini Vault 突合 (2026-05-07) で発見された事実:

- `patterns/store.py:88` で既に DB_PATH 環境変数経由の切替実装済
- `patterns/labels.py:28` / `materials/fetcher.py` / `patterns/frequency.py` / `metrics.py` / `live_log.py` 等、各モジュールが DB_PATH default 引数で受取設計

これにより環境分離の実装は「新規 DB 切替実装」ではなく「既存 DB_PATH の運用ルール整備」が中心になる。具体的には:

(a) DB_PATH が一貫して環境変数から読まれる経路を全モジュールで確認
(b) ハードコードされた fire.db 参照箇所がないか grep + 修正 (もしあれば)
(c) FIRE_ENV → DB_PATH の派生 wrapper script 整備
(d) wrapper 経由の起動を強制する運用ルール (= 直接 `python -m ...` を許さない、`bin/fire-*` 経由のみ許可) を Vault 化

**ハードコード fire.db 参照のリスク確認:**

Mac mini Vault 突合 (2026-05-07) では DB_PATH 切替伏線は確認済だが、全モジュールでハードコード参照がゼロかは未検証。実装時に `grep -rn "fire\.db\|fire/data/fire" ~/fire/` で全件確認、ハードコード箇所は環境変数読み込みに修正。

### 4-2. FIRE_ENV → DB_PATH wrapper 設計

**wrapper script 構成:**

```
bin/
├── fire-env-switch.sh    # FIRE_ENV から DB_PATH 派生
├── fire-check-env.sh     # branch と FIRE_ENV の整合性チェック
├── fire-dev              # development 環境用ラッパー
├── fire-staging          # staging 環境用ラッパー
└── fire-prod             # production 環境用ラッパー (Fujiwara 専用)
```

各ラッパーの動作 (例: `bin/fire-dev`):

```bash
#!/usr/bin/env bash
set -euo pipefail

export FIRE_ENV=development
source ~/fire/bin/fire-env-switch.sh
source ~/fire/bin/fire-check-env.sh

cd ~/fire
exec .venv/bin/python "$@"
```

呼び出し例:

```bash
# development 環境で paper_live 走らせる
~/fire/bin/fire-dev -m simulation.paper_live --batch-replay --days-back 5

# staging 環境で migration 試走
~/fire/bin/fire-staging -m scripts.setup.migrate_lane_code
```

### 4-3. branch 戦略 (dev → develop rename / staging 新設)

**branch rename の実施手順:**

**Step 1: 現 dev branch の状態確認**

```bash
cd ~/fire
git checkout dev
git status  # M scripts/seed_pattern_layer1.py 維持確認
git log -1  # 直近 commit (d4843df / e250239 / 6a15989) 確認
```

**Step 2: dev → develop rename (ローカル + remote)**

```bash
git branch -m dev develop          # ローカル rename
git push origin -u develop         # 新 develop branch を origin に push
git push origin --delete dev       # 旧 dev branch を origin から削除
```

**Step 3: staging branch 新設 (main からの fork)**

```bash
git checkout main
git pull origin main
git checkout -b staging
git push origin -u staging
```

**Step 4: GitHub branch protection 設定 (UI 操作 or API)**

- main: 直接 push 禁止、MR 経由のみ、Fujiwara 承認必須
- staging: 直接 push 禁止、MR 経由のみ、本部 or Fujiwara 承認 (Phase 1: Fujiwara)
- develop: 直接 push 許可 (Mac mini 開発用)

**Step 5: `~/fire-vault` は変更なし**

`~/fire-vault main` branch のみで運用継続。

**既存 working tree の維持:**

`M scripts/seed_pattern_layer1.py` (Q2 案 b、`find_firing_pairs_multi`) は develop branch に維持。F281 Phase 3 の patterns 抽出ロジック見直しで再活用予定。

---

## §5. MR 承認モデル (案 D 段階的)

Fujiwara 戦略判断 (2026-05-07) で確定: 案 D = Phase 1 (案 A: Fujiwara 全 MR 承認) → Phase 2 (案 C: ラベル制で部分本部委譲)

### 5-1. Phase 1 (環境分離直後 ~ Stage 3 開始 1 ヶ月後まで)

**期間:** 2026-05-10 (環境分離完了想定) ~ 2026-06-30 ± 数日 (Stage 3 開始 1 ヶ月後 = 7 月初旬)

**承認モデル:** 案 A (Fujiwara 全 MR 承認)

**MR 1: develop → staging**

flow:
- Mac mini → develop で機能実装 + commit
- Mac mini → MR 1 作成 (GitHub UI or CLI)、ラベル付与 (任意、Phase 1 では参考情報のみ)
- Codex CI → 自動チェック (test + Codex pre-commit + lint)
- 本部 (Claude.ai) → コードレビュー、設計妥当性確認 (本部側ミス再発防止のため、観点 8 + ルール 13 + 観点 11 を全件適用)
- 本部 → Fujiwara に MR 内容サマリ + レビュー結果を提示、承認可否提案
- Fujiwara → GitHub UI で approve または修正指示
- Mac mini → merge 実行 (squash 禁止、merge commit で履歴保持)

**MR 2: staging → main**

flow:
- staging で検証期間 (機能規模に応じて数時間 ~ 数日)
  - Run a/b/c 模擬実施 (staging DB)
  - F053 promotion_check
  - 既存機能の regression テスト
  - Codex CI 通過確認
- Mac mini → MR 2 作成、ラベル付与 (任意)
- Codex CI → main 用テスト追加実行 (production 影響範囲確認)
- 本部 → 検証結果整理、Fujiwara に最終承認可否提案
- Fujiwara → GitHub UI で approve (R-13-08 整合の最終 gate)
- Mac mini → merge 実行
- → production DB に変更反映、Stage 3 実弾運用稼働中なら影響開始

**Fujiwara 承認時の通知経路:**
- LINE 通知 (F236 SYSTEM ルーム経由) で MR 作成を即時通知
- 通知に MR URL + ラベル + 影響範囲サマリ含める
- Fujiwara が GitHub UI で approve / 修正指示

### 5-2. Phase 2 (Stage 3 開始 1 ヶ月運用後)

**期間:** 2026-07-01 ± 数日 ~ (Stage 3 安定運用後、必要に応じて Phase 3 へ移行)

**承認モデル:** 案 C (ラベル制で部分本部委譲)

**ラベル定義:**

| ラベル | 内容例 | MR 1 承認者 | MR 2 承認者 |
|---|---|---|---|
| [docs] | Vault 改訂、README、コメント | 本部 | Fujiwara |
| [refactor] | 機能変更なしのリファクタ | 本部 + Codex | Fujiwara |
| [test] | テスト追加・修正のみ | 本部 + Codex | Fujiwara |
| [chore] | 雑務 (依存更新、CI 設定) | 本部 + Codex | Fujiwara |
| [feat] | 新機能追加 | Fujiwara | Fujiwara |
| [migration] | DB schema 変更 | Fujiwara | Fujiwara |
| [stage3] | 実弾運用への直接影響 | Fujiwara | Fujiwara |
| [hotfix] | 緊急 bug fix | Fujiwara 即時 LINE | Fujiwara 即時 LINE |

**重要原則:**
- MR 2 (staging → main) は **全ラベルで Fujiwara 承認必須** (実弾影響)
- MR 1 (develop → staging) のみラベルに応じて部分委譲
- ラベル付与は Mac mini が MR 作成時に行う、ラベル誤付与時は本部が修正

**Phase 切替判定基準:**

Phase 1 → Phase 2 移行は以下を全て満たす場合に Fujiwara 戦略判断:

1. Stage 3 開始から 1 ヶ月以上経過
2. Active Light 仕様の実取引データ蓄積 (1 trade 数 / 損失 stop / pause 条件の妥当性確認)
3. 環境分離運用が安定 (Phase 1 期間中の MR で大きな事故ゼロ)
4. F271 v1.5+ §6-22 系新事例が Phase 1 期間中に発生していない (本部側ミス再発防止が機能していることの実証)
5. Fujiwara が「軽微な変更を本部承認に委譲しても安全」と判断

判定時期: 2026-07-01 前後、本部から Fujiwara に Phase 2 移行可否 Q を発行。

### 5-3. Phase 切替判定基準 (詳細)

**Phase 1 期間中の運用記録:**

本部側で Phase 1 期間中の以下を log.md に記録:
- 全 MR の一覧 (commit hash / ラベル / 承認日時 / Fujiwara コメント)
- Codex CI で検出された CRITICAL 件数
- 本部レビューで指摘された設計問題件数
- Fujiwara 承認時の修正指示件数
- F271 v1.5+ §6-22 系新事例の有無

これらを Phase 2 移行判定時に Fujiwara が参照、移行可否を判断。

**Phase 2 への正式移行手順:**
1. 本部から Fujiwara に Phase 2 移行 Q 発行 (運用記録サマリ + 移行可否提案)
2. Fujiwara 戦略判断 (移行 / 延期 / 別案)
3. 移行確定なら F282 設計記録改訂 (v1.1)、ラベル定義を Vault 化
4. Mac mini に Phase 2 運用開始指示
5. 最初の Phase 2 MR で運用テスト (本部承認の挙動確認)

---

## §6. データ同期方針

### 6-1. production → staging snapshot タイミング

**snapshot 頻度:**
- **定期**: 週 1 回 (毎週月曜 朝、Mac mini 自動 cron)
- **MR 検証前**: MR 1 (develop → staging) 受領前に staging DB を最新 snapshot に reset
- **手動**: Fujiwara 指示時 or 本部判断時

**snapshot 実施手順:**

**Step 1: production DB の整合性確認**

```bash
sqlite3 ~/fire/data/fire.db "PRAGMA integrity_check;"
# 結果が "ok" であることを確認、それ以外なら snapshot 中断 + 本部報告
```

**Step 2: production DB を staging にコピー**

```bash
cp ~/fire/data/fire.db ~/fire/data/fire.staging.db
cp ~/fire/data/fire.db-wal ~/fire/data/fire.staging.db-wal 2>/dev/null || true
cp ~/fire/data/fire.db-shm ~/fire/data/fire.staging.db-shm 2>/dev/null || true
```

**Step 3: staging DB の整合性確認**

```bash
sqlite3 ~/fire/data/fire.staging.db "PRAGMA integrity_check;"
```

**Step 4: snapshot ログ記録 (`~/fire/data/snapshot_log.txt`)**

```bash
echo "$(date -u +%Y-%m-%dT%H:%M:%SZ) production -> staging snapshot OK" \
  >> ~/fire/data/snapshot_log.txt
```

**Step 5:** Mac mini が本部に snapshot 完了報告 (MR 検証前 snapshot の場合)

**自動化 (cron):**

```cron
# crontab -l (Mac mini)
0 7 * * 1 ~/fire/bin/fire-snapshot-staging.sh
```

毎週月曜 7:00 JST に staging を最新 snapshot に更新。

### 6-2. develop の初期化方針

**develop DB の初期化タイミング:**
- **定期**: 月 1 回 (毎月 1 日、Mac mini 自動 cron)
- **手動**: Mac mini が試行錯誤で DB 状態を壊した時、Fujiwara 指示時
- **環境分離初期化時**: 一度のみ、production からの初期 snapshot

**初期化手順:**

**Step 1: 既存 develop DB を archive (壊した時の調査用)**

```bash
mv ~/fire/data/fire.develop.db ~/fire/data/fire.develop.db.bak.$(date +%Y%m%d)
```

**Step 2: production からコピー**

```bash
cp ~/fire/data/fire.db ~/fire/data/fire.develop.db
```

**Step 3: develop 専用のテストデータを生成 (任意)**

```bash
# 例: F058 e2e smoke test 用 pattern を develop だけに注入
~/fire/bin/fire-dev -m scripts.dev.inject_test_patterns
```

**Step 4: 古い archive (90 日以上前) を削除**

```bash
find ~/fire/data/ -name "fire.develop.db.bak.*" -mtime +90 -delete
```

**自動化 (cron):**

```cron
# crontab -l (Mac mini)
0 7 1 * * ~/fire/bin/fire-init-develop.sh
```

毎月 1 日 7:00 JST に develop DB を最新 production snapshot で初期化。

### 6-3. 環境間データ汚染防止

**データ汚染リスクと対策:**

**リスク 1:** develop で実験した patterns / migration が production に混入する

対策:
- `bin/fire-check-env.sh` で branch と FIRE_ENV の整合性を起動時チェック
- main branch では FIRE_ENV=production 必須、それ以外なら exit
- develop で生成したデータは git に含めない (.gitignore で `*.db` を除外)

**リスク 2:** production の実弾 trade データが develop / staging に流出する (Fujiwara のプライバシー / 戦略情報)

対策:
- snapshot は Mac mini ローカルのみで実施、外部に送信しない
- GitHub remote には DB ファイル自体を含めない (.gitignore で除外)
- staging / develop DB へのアクセスも Mac mini のみ

**リスク 3:** Mac mini が間違った環境で migration を実行する

対策:
- `bin/fire-*` wrapper 経由でしか `python -m ...` を許可しない運用
- 直接 python コマンドの使用は本部レビューで指摘 + 修正
- migration スクリプト冒頭で FIRE_ENV を確認、想定外なら exit

**リスク 4:** snapshot のタイミングずれで staging が古い production を持ち、MR 検証結果が production 反映時と異なる

対策:
- MR 検証前は必ず staging を最新 snapshot に reset (§6-1 Step 1-4)
- 週次 cron で常に staging を fresh に保つ
- MR 1 受領時に snapshot 日時をチェック、古ければ再 snapshot

**.gitignore 追記項目:**

```gitignore
# ~/fire/.gitignore に追加
/data/*.db
/data/*.db-wal
/data/*.db-shm
/data/*.bak.*
/data/snapshot_log.txt
```

これにより DB ファイルが GitHub に push される事故を構造的に防止。

---

## §7. 実装ステップ

F282 環境分離実装は 6 ステップ × 約 2-3 日 (合計 13-17 時間)。Mac mini が実装、本部が監督、Fujiwara が各ステップ完了時に承認。

### 7-1. branch 整備 (約 1 時間)

**手順:**

**Step 1: 現状確認**

```bash
cd ~/fire
git status        # M scripts/seed_pattern_layer1.py 維持確認
git log -5         # 直近 commit (d4843df / e250239 / 6a15989) 確認
git branch -a      # dev / main / origin/dev / origin/main 確認
```

**Step 2: dev → develop rename**

```bash
git branch -m dev develop
git push origin -u develop
git push origin --delete dev
```

**Step 3: staging branch 新設**

```bash
git checkout main
git pull origin main
git checkout -b staging
git push origin -u staging
git checkout develop  # 作業 branch を develop に戻す
```

**Step 4: GitHub UI で default branch を develop に変更**

- Settings → Branches → Default branch → develop

**Step 5: GitHub branch protection 設定 (Settings → Branches → Branch protection rules)**

- main: Require pull request、approvals 1+、Restrict pushes
- staging: Require pull request、approvals 1+
- develop: 直接 push 許可

**完了確認:**

```bash
git branch -a
# develop (現在), main, staging, origin/develop, origin/main, origin/staging が表示される
```

**commit / push:** branch 整備自体は commit 不要 (branch 作成のみ)。Vault 側に F282 設計記録 (本ファイル) を commit する作業は 7-6 で実施。

### 7-2. DB ファイル作成 + snapshot (約 30 分)

**手順:**

**Step 1: production DB の整合性確認**

```bash
sqlite3 ~/fire/data/fire.db "PRAGMA integrity_check;"
# "ok" を確認
```

**Step 2: staging / develop DB を作成 (production からコピー)**

```bash
cp ~/fire/data/fire.db ~/fire/data/fire.staging.db
cp ~/fire/data/fire.db ~/fire/data/fire.develop.db
```

**Step 3: 整合性確認**

```bash
sqlite3 ~/fire/data/fire.staging.db "PRAGMA integrity_check;"
sqlite3 ~/fire/data/fire.develop.db "PRAGMA integrity_check;"
```

**Step 4: snapshot ログ作成**

```bash
echo "$(date -u +%Y-%m-%dT%H:%M:%SZ) initial snapshot: production -> staging,develop" \
  > ~/fire/data/snapshot_log.txt
```

**Step 5: ファイルサイズ確認**

```bash
ls -la ~/fire/data/*.db
# production / staging / develop の 3 ファイルが約 354 MB ずつ
```

**完了確認:** 3 ファイルすべて存在 + 整合性 ok + ログ記録あり

### 7-3. wrapper script 実装 (約 4-6 時間)

**新規ファイル:**

```
bin/fire-env-switch.sh      # FIRE_ENV から DB_PATH 派生 (§4-2 参照)
bin/fire-check-env.sh       # branch と FIRE_ENV の整合性チェック (§3-2 参照)
bin/fire-dev                # development 環境ラッパー
bin/fire-staging            # staging 環境ラッパー
bin/fire-prod               # production 環境ラッパー
bin/fire-snapshot-staging.sh  # 週次 snapshot script
bin/fire-init-develop.sh    # 月次 develop 初期化 script
```

**ハードコード fire.db 参照の修正:**

```bash
cd ~/fire
grep -rn "fire\.db\|fire/data/fire" --include="*.py" | grep -v "test_" | grep -v ".venv" | grep -v "os.getenv"
# ハードコード参照箇所を特定、すべて os.getenv("DB_PATH", default) 経由に修正
```

**script 実行権限付与:**

```bash
chmod +x ~/fire/bin/fire-*
```

**動作確認:**

```bash
~/fire/bin/fire-dev -c "import os; print(os.getenv('DB_PATH'))"
# 出力: /Users/bluefire/fire/data/fire.develop.db

~/fire/bin/fire-staging -c "import os; print(os.getenv('DB_PATH'))"
# 出力: /Users/bluefire/fire/data/fire.staging.db
```

**commit:**

```bash
git add bin/
git commit -m "feat(f282): wrapper scripts for environment switching (FIRE_ENV → DB_PATH)"
```

### 7-4. pytest 環境切替テスト (約 2-3 時間)

**新規 test ファイル:**

```
tests/scripts/test_fire_env_switch.py
- test_fire_env_production_uses_production_db
- test_fire_env_staging_uses_staging_db
- test_fire_env_development_uses_develop_db
- test_fire_env_invalid_exits_with_error
- test_branch_env_mismatch_blocks_execution

tests/scripts/test_db_path_consistency.py
- test_no_hardcoded_fire_db_in_production_code
  (grep ベースで *.py を走査、fire.db ハードコード参照がないか確認)
- test_all_db_modules_use_db_path_env
```

**動作確認:**

```bash
cd ~/fire
.venv/bin/pytest tests/scripts/test_fire_env_switch.py -v
.venv/bin/pytest tests/scripts/test_db_path_consistency.py -v

# 全体 pytest で regression 確認
.venv/bin/pytest 2>&1 | tail -5
# 1295 → 1300+ PASS 想定 (新規 5+ テスト追加)
```

**Codex pre-commit:** 通常通り、`--no-verify` 不使用で一発通過必須

**commit:**

```bash
git add tests/scripts/
git commit -m "test(f282): environment switching tests + db_path consistency"
```

### 7-5. cron 自動化設定 (約 1 時間)

**snapshot 自動化 (週 1):**

`crontab -e` で以下を追加:
```cron
0 7 * * 1 ~/fire/bin/fire-snapshot-staging.sh >> ~/fire/data/cron.log 2>&1
```

**develop 初期化自動化 (月 1):**

`crontab -e` で以下を追加:
```cron
0 7 1 * * ~/fire/bin/fire-init-develop.sh >> ~/fire/data/cron.log 2>&1
```

**動作確認:**

```bash
crontab -l
# 2 行登録されていること

# 即時テスト実行
~/fire/bin/fire-snapshot-staging.sh
~/fire/bin/fire-init-develop.sh
cat ~/fire/data/cron.log
# 正常完了ログ確認
```

### 7-6. 運用ガイド Vault 化 + 最終 commit (約 2-3 時間)

**Vault 設計記録:**

`~/fire-vault/03_design/F282_environment_isolation_2026-05-08.md` (本ファイル)

```bash
cd ~/fire-vault
git add 03_design/F282_environment_isolation_2026-05-08.md
git commit -m "docs(f282): 環境分離 (3 環境化) 設計記録 + MR 承認モデル"
```

**F271 v1.5 → v1.6 化:**

`~/fire-vault/03_design/F271_v1.3_2026-05-07.md` (ファイル名維持)

追加内容:
- HQ-Lean v1.1 ルール 16 新設: 「production DB 直接操作禁止、必ず `bin/fire-*` wrapper 経由で実施」
- 観点 12 新設: 「環境分離後の MR は branch 整合性 + DB 整合性 + snapshot 鮮度を確認」
- 改訂履歴に v1.6 行追加

```bash
cd ~/fire-vault
git add 03_design/F271_v1.3_2026-05-07.md
git commit -m "docs(f271): F271 v1.6 ルール 16 + 観点 12 追加 (環境分離 MR 規律)"
```

**`~/fire develop` で最終確認:**

```bash
cd ~/fire
git status                     # M scripts/seed_pattern_layer1.py のみ
git log --oneline -10           # F282 関連 commit が並ぶ
```

**push:**

```bash
cd ~/fire-vault && git push origin main
cd ~/fire && git push origin develop
```

**完了報告:** 本部に F282 完了報告、Fujiwara 最終承認 → main merge

---

## §8. リスクと対策

### リスク 1: Mac mini が production を直接触る事故

**リスクシナリオ:**
- Mac mini が wrapper script を経由せず直接 `.venv/bin/python -m ...` を実行
- DB_PATH が production を指したまま development コードが走る
- migration スクリプトが production に適用される

**対策:**
- 直接 python コマンド禁止のルール 16 を Vault 化 (F271 v1.6)
- `bin/fire-check-env.sh` で起動時に branch と FIRE_ENV の整合性チェック
- 整合性違反なら `exit 1` で停止
- pytest `test_branch_env_mismatch_blocks_execution` で挙動を test 化

### リスク 2: snapshot タイミングずれによる検証結果の乖離

**リスクシナリオ:**
- staging が古い production スナップショット (1 ヶ月前)
- MR 1 検証で「OK」判定だが、main merge 時に「あれ、想定と違う」発生
- 直近 1 ヶ月の paper_live_results データが staging に欠けていたため

**対策:**
- MR 1 受領前に staging を最新 snapshot に必ず reset (§6-1 Step 1-4)
- 週次 cron で常に staging を fresh に保つ
- snapshot ログ (`snapshot_log.txt`) で snapshot 日時を本部レビュー時に確認

### リスク 3: develop DB の試行錯誤汚染

**リスクシナリオ:**
- Mac mini が develop で migration 試走、失敗、中途半端な状態
- 次の機能開発で「develop で動いたのに staging で動かない」発生
- 原因が DB 状態の汚染と気づかず、コードを疑って時間ロス

**対策:**
- 月 1 cron で develop 初期化、ゴミ蓄積を定期掃除
- archive (`.bak.YYYYMMDD`) を 90 日保持、過去状態の復元可能
- 試行錯誤後に「一旦 develop reset」を Mac mini が判断できるよう運用ガイド化

### リスク 4: GitHub に DB ファイルが push される事故

**リスクシナリオ:**
- .gitignore 不備で fire.db / fire.staging.db / fire.develop.db が GitHub に push される
- 実取引データ (Stage 3 開始後) の戦略情報・トレード履歴が外部流出
- private repo でも、GitHub アカウント乗っ取り時のリスク

**対策:**
- .gitignore に `/data/*.db` / `/data/*.db-*` / `/data/*.bak.*` / `/data/snapshot_log.txt` / `/data/cron.log` を追加
- pre-commit hook で *.db ファイルの commit を block (Codex pre-commit に組込候補)
- 初期環境分離実装時に `git status` で確認、誤 commit ゼロ

### リスク 5: wrapper script のバグで実行不能

**リスクシナリオ:**
- `bin/fire-dev` に bash 構文エラー
- Mac mini が development で何も実行できない
- 開発が完全停止

**対策:**
- pytest で wrapper script 動作を test 化 (`test_fire_env_switch.py`)
- 実装時に手動動作確認 (production/staging/development 各環境で簡単な `python -c "print('ok')"` を実行)
- 本部レビューで script の構文と権限 (`chmod +x`) 確認

### リスク 6: Phase 切替時の運用混乱

**リスクシナリオ:**
- 2026-07-01 頃の Phase 切替判定で「Phase 2 移行」確定
- ラベル付与ルールが Mac mini に浸透していない
- ラベル誤付与で本来 Fujiwara 承認すべき MR が本部承認で merge される

**対策:**
- Phase 2 移行前に Vault でラベル定義を再確認 (F282 §5-2)
- 最初の Phase 2 MR は本部レビューで「ラベル妥当性」を最初にチェック
- ラベル誤付与 1 件で Phase 1 に戻る判定基準を Vault 化

### リスク 7: scripts/seed_pattern_layer1.py:40 の技術的負債残置

**リスクシナリオ:**
F282 環境分離後も `scripts/seed_pattern_layer1.py:40` に DEFAULT_DB_PATH がハードコード残置 (Q2 案 b 維持優先のため)。F281 Phase 3 再活用時に `bin/fire-dev` / `fire-staging` 経由で実行されない経路で seed が実行されると、production fire.db を意図せず触る可能性。

**対策:**
- F281 Phase 3 着手時に `scripts/seed_pattern_layer1.py:40` を `os.getenv("DB_PATH", default)` 経由に修正
- F271 v1.6 観点 12 (環境分離後の MR 整合性) で `seed_pattern_layer1.py` 関連 MR は branch + DB 整合性確認を必須化
- F271 v1.6 ルール 16 (production DB 直接操作禁止) の例外として記録、F281 Phase 3 で解消する技術的負債としてマーク

---

## §9. 受入基準

F282 完了の受入基準 (すべて満たすこと):

**branch 構造:**
- ✅ `~/fire branches`: develop / staging / main の 3 branch 存在
- ✅ `~/fire-vault branches`: main のみ (変更なし)
- ✅ origin に同 3 branch push 済
- ✅ default branch = develop に設定済
- ✅ GitHub branch protection: main / staging で MR 必須

**DB 構造:**
- ✅ `~/fire/data/fire.db` (production) 存在 + 整合性 ok
- ✅ `~/fire/data/fire.staging.db` 存在 + production と同期 + 整合性 ok
- ✅ `~/fire/data/fire.develop.db` 存在 + production と同期 + 整合性 ok
- ✅ 3 ファイルすべて約 354 MB
- ✅ `snapshot_log.txt` に初期 snapshot 記録あり

**wrapper script:**
- ✅ `bin/fire-env-switch.sh` 実装済 + 実行権限あり
- ✅ `bin/fire-check-env.sh` 実装済 + 実行権限あり
- ✅ `bin/fire-dev` / `fire-staging` / `fire-prod` 実装済 + 実行権限あり
- ✅ `bin/fire-snapshot-staging.sh` 実装済
- ✅ `bin/fire-init-develop.sh` 実装済

**ハードコード解消:**
- ✅ `~/fire` 配下の Python コードに fire.db ハードコード参照なし
- ✅ すべて `os.getenv("DB_PATH", default)` 経由
- ※ 例外: `scripts/seed_pattern_layer1.py:40` (Q2 案 b 維持、リスク 7 で記録、F281 Phase 3 で解消予定)

**pytest:**
- ✅ `test_fire_env_switch.py`: 5+ test 全 PASS
- ✅ `test_db_path_consistency.py`: 2 test 全 PASS
- ✅ 全体 pytest: 1295 → 1300+ PASS、regression なし

**Codex pre-commit:**
- ✅ 各 commit で一発通過、`--no-verify` 不使用

**cron 自動化:**
- ✅ 週次 snapshot (月曜 7:00 JST) 登録済
- ✅ 月次 develop 初期化 (毎月 1 日 7:00 JST) 登録済
- ✅ 即時テスト実行で正常完了確認

**.gitignore:**
- ✅ `/data/*.db` / `*.db-*` / `*.bak.*` / `snapshot_log.txt` / `cron.log` 追加
- ✅ `git status` で *.db ファイルが untracked にならない確認

**Vault:**
- ✅ `~/fire-vault/03_design/F282_environment_isolation_2026-05-08.md` 作成 + commit
- ✅ `~/fire-vault/03_design/F271_v1.3_2026-05-07.md` v1.5 → v1.6 改訂 + commit
- ✅ `~/fire-vault/log.md` milestone 追記 + commit
- ✅ push origin main 完了

**MR 承認モデル開始:**
- ✅ Phase 1 (案 A: Fujiwara 全 MR 承認) 運用開始
- ✅ 最初の MR 1 (develop → staging) 起票テストで動作確認
- ✅ 最初の MR 2 (staging → main) 起票テストで動作確認

**Stage 3 開始準備:**
- ✅ F282 完了で短縮版 Phase 3 着手準備完了
- ✅ Stage 3 開始予定 5/30 ± 数日に向けて経路維持

---

## §10. 関連リンク

### 関連タスク

- F271 v1.6: 本部運用ルール (環境分離後の MR 規律 = ルール 16 + 観点 12)
- F281 v1.2: 戦略ポートフォリオ (Active Light §5-bis、本タスクで staging 検証経路確立)
- F058: e2e smoke test (test pattern 残置 = 環境分離が機能していれば防げた事例)
- F273 / F274 / F275 / F276: Pattern Store / SCORE 構造系 (短縮版 Phase 3 で staging 検証)
- F235: 楽天証券連携 (Stage 3 開始前必須範囲は別途確定)
- F266: Stage 3 最終ゲート (環境分離込みで R-32-01 / F053 / F241 達成判定)

### 関連設計記録

- `~/fire-vault/03_design/F281_strategy_portfolio_design_2026-05-07.md` (v1.2)
- `~/fire-vault/03_design/F271_v1.3_2026-05-07.md` (v1.6 化)
- `~/fire-vault/02_todo/F282_environment_isolation.md` (新規 todo)

### 関連要件書

- 第 12 章: 全自動運用モードと移行ステップ (Stage 0-4)
- 第 19 章: Simulation / Paper Live 方針
- 第 32 章: 本番移行基準の厳密化 (R-32-01 7 項目)
- 第 34 章: 本番移行基準の詳細
- 第 35 章: Mac mini 常駐基盤の時間帯別分担
- 第 13 章: Evaluation Agent と Dashboard (R-13-08 ユーザー承認制)

### 後続タスク

- 短縮版 Phase 3: target_patterns 拡張 + F273_BRE 4,700 件評価 (staging 環境)
- F282 Phase 2 移行判定 (2026-07-01 ± 数日、Stage 3 開始 1 ヶ月後)
- F281 Phase 3: score モデル感度分析 + features 寄与度 + patterns 抽出ロジック (Stage 3 開始後、staging 環境で並走実施)

### 改訂履歴

- 2026-05-08 v1.0: 初版 (F282 環境分離 3 環境化、案 W1 + α + D 確定、Fujiwara レビュー済 §1-10)
