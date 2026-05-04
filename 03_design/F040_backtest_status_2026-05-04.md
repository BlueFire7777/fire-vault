# F040 (Backtest Mode) 現状調査レポート

**作成日**: 2026-05-04 (F273 Phase 1)
**目的**: F267 完了後 Run a が events=0 のまま終わった問題を解消するため、案 N1 (F040 Backtest による Layer 4 seeding) の実現性を検証する
**位置付け**: Chapter 番号なしの調査ログ (Chapter 38「運用モード/休止/旅行モード方針」と衝突しない)
**前提レポート**: [[F032_F054_diagnosis_2026-05-04]] / [[F267_implementation_2026-05-04]]
**完了基準**: [[task_completion_criteria]] § 4 (3 段階: 動いた / 機能した / 期待値達成)

---

## 1. 調査目的

F273 Phase 1 として、F040 の実装状態と Layer 4 書き込み経路を把握し、Phase 2 (実装 or 修正) と Phase 3 (seeding 実行 + Run a 再走) の方針を確定する。

ただし調査の過程で **F267 完了報告に含まれる重大な誤解** を発見したため、本レポートは「F040 調査」を超えて Pattern Store の真の構造と真の主因 D'' まで踏み込んだ。

---

## 2. F040 実装状態

### 2-1. ファイル所在

`~/fire/simulation/backtest/` (5 ファイル / 計 1,000 行):

| ファイル | 行数 | 役割 |
|---|---|---|
| `__init__.py` | 23 | パッケージ初期化 |
| `models.py` | 74 | dataclass (BacktestRun / BacktestResult / RunStatus / FillModel) |
| `runner.py` | 243 | `BacktestRunner` クラス (期間指定で集計実行 + DB 記録) |
| `aggregator.py` | 355 | 5 カテゴリ集計 (overall / by_pattern / by_lane / by_strategy_type / by_sector) |
| `report.py` | 214 | JSON / Markdown レポート出力 |
| `cli.py` | 91 | argparse CLI エントリポイント |

タスクメモ: [[02_todo/F040_Backtest_Mode_Stage_0]] (status: 完了 / 2026-04-26 / depends_on: F029, F030, F031)

### 2-2. エントリポイント

```bash
.venv/bin/python -m simulation.backtest.cli \
    --start 2025-11-01 --end 2026-05-02 \
    --patterns <ID,ID,...> \
    --fill-model ideal --mode run
```

mode=run / list / report の 3 モード。

### 2-3. 実装レベル: ✅ **実装済み (テスト 20 PASS、status=完了)**

ただし設計思想が「フレームワーク先行 / データ 0 件で完走」(F040 タスクメモ § 実装内容)。

### 2-4. 動作可能性

```
.venv/bin/python -m simulation.backtest.cli --help
```
は通る (タスクメモ § テスト「全 20 PASS」根拠)。実 run も `backtest_runs` 1 件 / `backtest_results` 12 件の過去実行履歴あり。

---

## 3. Pattern Store の真の 4 階層 (重大な誤解の訂正)

### 3-1. 要件書第37章 (v3.4) と実装の整合

**`~/fire/patterns/store.py:6-9`** のコメント:

```
Layer 1: Pattern Archive       — 全履歴（100+）
Layer 2: Valid Pattern Library — 現在有効なパターン群（20-100）
Layer 3: Active Priority Set   — 主力パターン（3-15）
Layer 4: Death Note / Rehab    — 凍結・再検証
```

`store.py:63-69` の `Status → Layer` マッピング:

| Status | Layer |
|---|---|
| candidate | Layer 1 (ARCHIVE) |
| paper_validating | Layer 2 (VALID_LIBRARY) |
| approved_active | **Layer 3 (ACTIVE_PRIORITY)** |
| watchlist | Layer 2 (VALID_LIBRARY) |
| frozen / death_note / rehab | **Layer 4 (DEATH_REHAB)** |

### 3-2. F267 完了報告 (2026-05-04 午後) の誤解の訂正

**[[F267_implementation_2026-05-04]] § 4 で「Layer 4 (instances / match_records) が空」と書いたが、これは完全に誤解**。真の Layer 4 = **Death Note / Rehab (凍結対象)** であり、「過去の類似局面 instances」とは別概念。

正しい記述: 「Pattern Store の Layer 1 (Pattern Archive) に **実データ紐付きのパターン定義** が空 (= テンプレートのみ)」。詳細は § 4-2 で展開。

### 3-3. 現在の Layer 配分

| Layer | 件数 | 内訳 |
|---|---|---|
| Layer 3 (Active Priority) | 1 | `material_initial-strong_market-breakout-AM-highliq@v1.0` (rank=B、symbol=TEST_E2E、detected_at=2026-05-01T14:02:34) |
| Layer 1 (Archive、status=candidate) | 8 | `earnings_upside / guidance_upside / buyback / dividend_increase / order / alliance / policy_benefit / theme_flow` (全て symbol=0000、detected_at=2026-05-02T06:01:42) |
| Layer 2 / Layer 4 | 0 | — |

→ **9 件すべて (symbol=TEST_E2E or 0000、detected_at=ダミー時刻) のテンプレートのみ**。

### 3-4. 「instance」の要件書定義

要件書第37章 v3.4 全文を確認: 「Pattern Archive (Layer 1) は **全履歴。勝ち・負け・候補・廃止・Rehab 候補を全部含む巨大な蓄積庫**」と書かれており、「instance」という用語は明示されない。

実装上は **patterns テーブルの 1 行が 1 パターン定義 (= 1 instance 相当)** と解釈できる。実データに紐付いた pattern_id ごとに `(symbol, detected_at)` が必要で、これが SimilarityEngine.search() の `fetch_pattern_feature_vector()` で features を引く鍵となる。

### 3-5. F031 / F032 の動作 (確認済み)

- `~/fire/patterns/similarity.py:189-200`: `SimilarityEngine.search()` は `SELECT * FROM patterns WHERE 1=1 AND status != 'death_note'` で全 Layer 1〜3 のパターンを取得し、各 `(pattern_id)` について `fetch_pattern_feature_vector()` を呼ぶ
- `similarity.py:225-230`: `if not vec_past: continue` で **features が無いパターンは skip**
- `~/fire/patterns/reproducibility.py:38-39`:
  - `MIN_SIMILAR_COUNT_FOR_EXECUTE = 5` (5 件未満は execute 不可)
  - `MIN_SIMILAR_COUNT_FOR_REFERENCE = 2` (2 件未満は判断不能 = decision="pass")

---

## 4. F040 → Layer 4 書き込み経路

### 4-1. 既存経路の有無: ❌ **無い (設計通り)**

`runner.py` + `aggregator.py` を全文読破した結果:

- F040 は **`positions` テーブル (closed + label IS NOT NULL) を集計**するだけ
- `INSERT INTO patterns` / `INSERT INTO pattern_state_history` などの「Layer 1〜4 への書き込み」は**一切なし**
- `aggregate_overall / by_pattern / by_lane / by_strategy_type / by_sector` の 5 集計を `backtest_results` テーブルに保存するのみ
- 既存 `backtest_runs: 1 件 / backtest_results: 12 件` は過去のテスト実行 (positions=0 で空集計だった可能性大)

→ **F040 は集計レポーターであり、Layer 1 を seed する機能を持たない (タスクメモにも記載なし、要件書 R-19-XX にも記載なし)**。

### 4-2. **真の問題発見 — Layer 1 が「実データ紐付きでない」**

患者の症状を再現した結果:

```
SQL: SELECT pattern_id, symbol, detected_at FROM patterns
patterns 9 件 すべて symbol=TEST_E2E (1 件) or symbol=0000 (8 件)
detected_at もダミー時刻 (2026-05-01T14:02 / 2026-05-02T06:01:42)

各 (symbol, detected_at) で features をルックアップ:
  TEST_E2E / 2026-05-01T14:02:34 → 0 行
  0000 / 2026-05-02T06:01:42 → 0 行 (× 8 件)
```

→ `fetch_pattern_feature_vector(pattern_id)` が **全 9 件で None を返す**
→ `SimilarityEngine.search()` の loop 内で `if not vec_past: continue` で全 skip
→ **search() の戻り値が空 list**
→ `ReproducibilityEngine.evaluate()` で `n_similar=0 < MIN_SIMILAR_COUNT_FOR_REFERENCE=2`
→ **decision="pass" / 全銘柄で events=0 連鎖**

これが [[F267_implementation_2026-05-04]] § 4 で観測された `decision="pass" / similar_count=0` の真の理由。

### 4-3. 真の主因 D'' (D' を更に深く特定)

| 仮説 | 状態 |
|---|---|
| A: features 未生成 | ✅ F267 で解消 (1.07M 行) |
| B: announcements 不足 | ⏸ F268 で対応予定 |
| C: パターン起動条件 4 重 AND | ❓ 切り分け不能 |
| D' (前回): Pattern Store Layer 4 (instances) 空 | ❌ **誤り** (Layer 4 = Death Note、無関係) |
| **D'' (新): Pattern Store Layer 1 (実データ紐付きパターン) 空** | **❌ 真の主因確定** |

### 4-4. F035 propose_new_candidate も解にならない可能性

`agents/pattern_research.py:99-160` の `propose_new_candidate()`:

- pattern 命名 5 セグメントを受け取り `pattern_name + pattern_id` を生成して `CandidateProposal` を返すだけ
- `register_proposal(proposal, approved_by)` を呼ぶと `store.register_candidate(symbol="0000" by default)` で patterns テーブルに INSERT する
- → **新規パターンも symbol="0000" のテンプレート**になる
- → 同じ問題 (features に紐づかない) が再発

→ F035 は「人間が承認したパターン定義の登録」用途であり、過去データから自動的に発火事象を発見して登録する機能 (要件書 R-21-XX で言及される設計) は **実装されていない可能性**。

### 4-5. 経路追加実装の必要範囲

events=0 解消には新規実装が必要:

```
[新規 seeder 設計案]
  入力: Core500 × 過去 6 ヶ月 features (1.07M 行)
   ↓
  発火条件 (例): D5 breakout_flag=1 AND A1 material_strength_score >= 5
   ↓
  発火 (symbol, dt) ペアを抽出
   ↓
  patterns テーブルに INSERT (symbol=実銘柄、detected_at=実 dt、pattern_id=テンプレート pattern_id を再利用)
   ↓
  → SimilarityEngine.search() が features 紐付き pattern を多数 hit
   ↓
  → ReproducibilityEngine.evaluate() で similar_count >= 2 を達成
   ↓
  → Run a で events>0
```

これは F040 でもなく、F035 でもなく、**新規タスク (= F274 候補)** として起票が必要。

---

## 5. 計算量見積

### 5-1. 元の計画 (F040 を seeder として使う案 N1) の見積もり

→ **不適用** (F040 は seeder ではないことが確定)

### 5-2. 新規 Layer 1 seeder の見積もり

入力規模:
- Core500: 500 銘柄
- 過去 6 ヶ月: 120 営業日 (確認済み: market_prices_daily の DISTINCT date)
- = 60,000 (symbol, dt) ペア
- 各ペアで 18 features (technical 9 + time 3 + material 6) → features は **既に揃っている (F267 済)**

処理時間 (推定):
- 発火判定 1 ペアあたり 0.001 秒 (DB read + 単純条件評価のみ) → 60,000 × 0.001 = **60 秒**
- patterns テーブルへの INSERT は発火件数次第 (1% 発火と仮定 = 600 件 INSERT × 0.01 秒 = 6 秒)
- **合計 ≦ 数分** (並列化不要)

実装工数:
- 設計 (発火条件定義 + テスト戦略): 4 時間
- 実装 (1 ファイル + tests): 8 時間
- 動作確認 + 調整: 2 時間
- **合計 約 1.5 〜 2 日**

### 5-3. 並列化可能性

ペア独立なので multiprocessing 可能だが、処理が軽量で SQLite 書き込みが律速のため **並列化不要**。

### 5-4. 全体所要時間レンジ

| 段階 | 最良 | 標準 | 最悪 |
|---|---|---|---|
| 設計 + 実装 | 1 日 | 1.5 日 | 2 日 |
| seeder 実行 (本番) | 1 分 | 5 分 | 30 分 |
| Run a 再走 (32 分 〜 91 分実績) | 32 分 | 91 分 | 120 分 |
| 検証 + Fujiwara 確認 | 30 分 | 1 時間 | 半日 |
| **合計 (Phase 2 + Phase 3)** | **約 1.5 日** | **約 2.5 日** | **約 3 日** |

---

## 6. Phase 2 の必要作業 — 判定: ケース F (前提が違う)

### 6-1. ユーザー指定の 4 ケースは不適合

| ケース | 適合判定 |
|---|---|
| ケース A (実装済み + Layer 4 経路あり) | ❌ Layer 4 = Death Note、F040 は経路無 |
| ケース B (実装済み + Layer 4 経路なし) | ❌ Layer 4 が前提として違う |
| ケース C (部分実装) | ❌ F040 自体は完了済 |
| ケース D (未着手) | ❌ F040 は完了済 |

→ **新ケース F (前提が違う)**: F040 は seeder ではなく集計レポーター。真の問題は Layer 1 が「実データ紐付きパターン」を持たないこと。

### 6-2. Phase 2 の真の作業内容

1. **F040 を seeder として使うのは諦める** (F040 は集計用途で完了済、現状でも価値はある)
2. **新規タスク起票**: 「Layer 1 seeder 機構の設計と実装」 (= F274 候補)
   - 過去データを Core500 × 120 日でスキャン
   - 発火条件を定義 (要件書第36章 / F036 Live Research Log の設計を参照、要追加調査)
   - 発火した (symbol, dt) を patterns テーブルに INSERT
   - 既存の 9 件テンプレートを各実 (symbol, dt) に複製する形が現実的
3. **F035 propose_new_candidate の register_proposal 拡張**: symbol_seed の引数に実銘柄を渡せるようにする (要件書を読み込んで検証)
4. **発火条件の定義**: 第36章 (初期特徴量セット) や第30章 (特徴量の初期絞り込み) からルール抽出が必要

### 6-3. Phase 2 の前提リスク

仮に Layer 1 を 600〜6000 件 (1〜10% 発火) で seed しても、SimilarityEngine が **features ベクトル類似度**でフィルタするため、5 件以上の類似 hit を担保できるかは未検証。少なすぎる発火件数だと similar_count < 2 のまま。逆に多すぎる発火件数だと「全部似ている」状態で score 0.5 以下になり decision="reference_only" 止まりの可能性。

→ Phase 2 の最初に **小規模試走 (1 銘柄 × 10 日)** で発火条件と閾値の sanity check を入れる。

---

## 7. Phase 3 への引き継ぎ事項

### 7-1. Layer 1 seeding 完了後の Run a 再走条件

- patterns テーブル件数: 9 → **N (発火件数次第、目標 50 件以上)**
- 各 pattern が `(実銘柄, 実 dt)` を持ち、その時点の features が引ける状態
- Run a を再走して `n_events_total > 0` を確認

### 7-2. events>0 達成見込み

`MIN_SIMILAR_COUNT_FOR_REFERENCE = 2` だけ突破すれば `decision="reference_only"` (events 発生)、5 件突破で `decision="execute"` (本気の events 発生)。

楽観: 50〜100 件 seed で達成可能。
悲観: similarity_score の閾値や regime_fit の制約で「同じパターンが 100 件あっても score 0.5 以下」となれば達成しない。

### 7-3. 達成しなかった場合の次の仮説 (D''' の存在可能性)

- D''' 候補 1: SimilarityEngine の特徴ベクトル定義 (`category_distance` の重み) が現実離れ
- D''' 候補 2: 各パターンの発火条件 (例: `material_initial-strong_market-breakout-AM-highliq`) が過去データに対して 1〜2 件しか発火しない (= seeding しても件数増えない)
- D''' 候補 3: `regime_fit` 計算で「現在の地合い」と「過去の類似局面の地合い」が一致しない (regime データ未取得 = F104 待ち)

**特に D''' 候補 3 は深刻**: F104 (指数四本値取得) 未着手で `regime` collector が ready=False のまま (F267 で確認済)。regime_fit_factor が 0.0 を返すと合成スコアが伸びない可能性。

---

## 8. 推奨される Phase 2 着手判断

### 8-1. 工数再見積

元の 1.5 日想定 (F040 動作確認 + Layer 4 seeding 実行) は **本 Phase 1 の調査時間として消費** (約 2 時間)。残りの Phase 2 + Phase 3 の作業は新ケース F として再見積:

| 項目 | 元見積 | 新見積 |
|---|---|---|
| F040 seeding 実装 + 実行 | 0.5 日 | (廃止) |
| Layer 1 seeder 設計 + 実装 | — | 1.5 〜 2 日 |
| seeder 実行 + Run a 再走 | 1 日 | 0.5 日 |
| 検証 + Fujiwara 確認 | — | 0.5 日 |
| **合計** | 1.5 日 | **2.5 〜 3 日** |

### 8-2. 着手 GO/NO-GO の推奨: **GO (ただし scope 再定義必須)**

- **GO 推奨理由**:
  - 真の主因 D'' を特定できた (Phase 1 の最大成果)
  - Layer 1 seeder の方向性は要件書 v3.4 と整合 (Pattern Archive の本来の役割)
  - 工数増 (1.5 日 → 2.5 〜 3 日) は許容範囲、Stage 3 への道筋が見える
  - 実装難易度は中程度 (F267 同等)

- **GO 条件 (Fujiwara への確認事項)**:
  1. 新タスク **F274 (仮称) = Layer 1 seeder 機構の設計と実装** を起票する判断
  2. **F040 はそのまま「集計レポーター」として残す** (削除しない、Stage 3 以降の集計用途として価値あり)
  3. 発火条件の定義に要件書第36章 / 第30章を使うが、**新規 Pattern 設計判断は Fujiwara 承認が必要** (R-21-13 自動反映禁止)
  4. **小規模試走 (1 銘柄 × 10 日)** を Phase 2 の最初に入れる前提

### 8-3. NO-GO 推奨の代替案

NO-GO の場合の代替:
- A: F035 propose_new_candidate の手動実行を Fujiwara が 50 件以上行う (規律的だが工数が読めない)
- B: F104 (指数四本値) を先に取得して regime collector 解禁 → SimilarityEngine の score 計算改善 → 既存 9 件パターンで decision="reference_only" 達成可能性検証
- C: 設計レベルで Pattern Store / SimilarityEngine の根本見直し (要件書改訂、半月〜1 ヶ月)

→ A/B は不確実性高、C は時間かかりすぎ。**B (F104 先行) は副案として併走価値あり** (regime_fit が 0.0 でなくなれば score 計算が変わる)。

---

## 9. 関連リンク

- 起点診断: [[F032_F054_diagnosis_2026-05-04]]
- F267 完了 (誤解の訂正対象): [[F267_implementation_2026-05-04]]
- 完了基準: [[task_completion_criteria]]
- 要件書 (Pattern Store 真の構造): [[01_requirements/FIRE_要件書_第37章_Pattern_Storeの4階層構造とActive_Priority_Set]]
- 要件書 (Pattern Research): [[01_requirements/FIRE_要件書_第21章_Pattern_Research_Agent___Pattern_Store_詳細]]
- 既存タスクメモ: [[02_todo/F040_Backtest_Mode_Stage_0]] / [[02_todo/F035_Pattern_Research_Agent]] / [[02_todo/F036_Live_Research_Log]]
- 次タスク候補: F274 (Layer 1 seeder)、F104 (指数四本値、副案)、F268 (announcements 遡及、優先度低下)、F269 (Chain assert、引き続き)

---

## 10. F271 完了基準による F273 Phase 1 自己評価

| 段階 | 結果 | 根拠 |
|---|---|---|
| 動いた | ✅ | 全 grep / sqlite / 実装読み込みが完走、エラーなし |
| 機能した | ✅ | F040 実態確認 + Layer 4 真の意味確認 + 真の主因 D'' 特定 + Phase 2 ケース判定 (ケース F) 完了 |
| 期待値達成 | ✅ | Phase 2 の作業内容 (新ケース F + F274 起票) と工数再見積 (2.5〜3 日) が確定し、Fujiwara が GO/NO-GO 判断できる材料が揃った |

---

(以上)
