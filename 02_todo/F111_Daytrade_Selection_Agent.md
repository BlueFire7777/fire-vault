---
id: F111
phase: P4: LINE/通知基盤
priority: 最優先
status: 完了
owner: Fujiwara
depends_on: [F031, F032, F054]
chapter: "04,07"
created: 2026-05-02
updated: 2026-05-02
tags: [P4, agents, daytrade]
---

# F111: Daytrade Selection Agent (1-3 銘柄厳選 + 戦略コメント生成)

## 概要

F054 `extract_candidates` の出力 (CANDIDATE リスト) から、デイトレ目線で
**1-3 銘柄に厳選** + R-07-04 の **戦略コメント 7 要素** を付与するエージェント。
朝 8:40 通知に間に合うレイテンシ要件のため、Claude API 呼び出しなしの
テンプレートベース実装。

要件根拠:
- 第 4 章 (システム構成 - エージェント群: Daytrade Selection Agent)
- 第 7 章 R-07-01 (デイトレ候補 朝 5 件 → 8:40 で 1-3 銘柄厳選)
- 第 7 章 R-07-04 (戦略コメント 7 要素)

## 実装内容

### 主要モジュール

- `agents/daytrade_selection.py` (新規)
  - `DaytradeCandidate` dataclass: 選定 1 件 (symbol / pattern_id / score / rank /
    strategy 7 要素 / selection_reason)
  - `DaytradeSelectionResult` dataclass: 1 日分の選定結果 (candidates / skipped / summary)
  - `DaytradeSelectionAgent` クラス
    - `select_candidates(candidates, max_selected=3, min_score=0.6, exclude_low_confidence=True)`
    - `_generate_strategy_comment(pattern_id, pattern_name, payload)` → 7 要素 dict
    - `_parse_pattern_name(pattern_name)` → segment 5 種辞書
  - 4 つのラベル辞書: MATERIAL_LABELS / REGIME_LABELS / PRICE_ACTION_LABELS / TIME_WINDOW_LABELS

### 選定ロジック (6 step)

1. 入力検証: `event_type=candidate` かつ `payload` 非空のみ通過
2. 重複除外: 同一 symbol → 最高 score を残す
3. ソート: score 降順 → confidence 降順 (high>medium>low) → similar_count 降順
4. 足切り: score < min_score (0.6) または confidence=low なら skipped
5. 上位 max_selected (3) 件を採用、残りは skipped
6. 各候補に戦略コメント 7 要素を生成

### R-07-04 戦略コメント 7 要素

| キー | テンプレート例 |
|---|---|
| `basic_policy` | "上方修正を起点とする前場高値ブレイクを狙う" |
| `entry_condition` | "前場高値ブレイクが確認できた段階で寄り付き〜前場で約定。強地合いの状況下で機能" |
| `skip_condition` | "寄り付き失速 / 出来高が想定より少ない / 板が薄い場合は見送り" |
| `tp_strategy` | "TP 価格に到達したら利確 (指値)。半分利確の場合は残り半分はトレール" |
| `sl_strategy` | "SL 価格を逆指値で必ず設定。SL 到達で機械的に損切" |
| `monitor_reason` | "再現性スコア 0.78 (confidence=medium)、類似局面 5 件で execute 判定" |
| `valid_time_window` | "前場" or "前場 (実運用では場中の値動きを見て柔軟に調整)" (any 時) |

### F054 payload 構造の差分対応

仕様書の `best_match` dict 想定とは異なり、F054 実装は **flat キー**:

```python
# F054 actual payload
{
    "decision": "execute",
    "reproducibility_score": 0.78,
    "confidence": "medium",
    "similar_count": 5,
    "best_pattern_id": "pid-guidance",        # flat
    "best_pattern_name": "guidance_upside-...", # flat
}
```

→ `payload.get("best_pattern_id")` / `payload.get("best_pattern_name")` で取得。
spec の `payload.get("best_match", {}).get(...)` ではない。

## テスト

- `tests/agents/test_daytrade_selection.py`: **20 / 20 PASS**
  - TestParsePatternName (2): 標準 / 不正
  - TestGenerateStrategyComment (4): 日本語ラベル / any 系 / 7 キー / score 反映
  - TestSelectCandidatesEmpty (3): 空 / event_type 不一致 / payload None
  - TestSelectCandidatesScoring (5): score 順 / tie-break / 足切り / low 除外 / max 超過
  - TestSelectCandidatesDedup (2): 同一 symbol 高 score 残し / dedup → max
  - TestEndToEnd (3): 5 入力→3 選定+2 skipped / 0 入力 / 大量 50 件→3 件
  - TestLabelCompleteness (1): 初期 8 パターンの material 全て label 定義済
- 累計 **600 PASS** (580 → 600、F040-F058 + F100 + F236 既存全 PASS 維持)

## CLI smoke 結果

### F054 → F111 連結 (実 DB)
```
Phase 1: F054 extract_candidates
  raw candidates: 0  ← 現状 features 12 件 / patterns 9 件で類似マッチなし

Phase 2: F111 select_candidates (real data)
  selected: 0 / skipped: 0
  summary: {n_total_input: 0, n_valid: 0, n_selected: 0, n_skipped: 0}
  → 0 件で空完走 (期待通り、F040-F058 と同じフレームワーク先行方針)
```

### 戦略コメントサンプル (mock candidate, Fujiwara 視認用)

入力: `guidance_upside-strong_market-breakout-AM-highliq@v1.0` / score=0.78 /
similar=5 / confidence=medium

出力:
```
rank=1 symbol=7203 score=0.78
pattern_name: guidance_upside-strong_market-breakout-AM-highliq@v1.0
selection_reason: 再現性スコア 0.78 / 類似 5 件 / confidence=medium

--- 戦略コメント (R-07-04 7 要素) ---
[basic_policy]      上方修正を起点とする前場高値ブレイクを狙う
[entry_condition]   前場高値ブレイクが確認できた段階で寄り付き〜前場で約定。強地合いの状況下で機能
[skip_condition]    寄り付き失速 / 出来高が想定より少ない / 板が薄い場合は見送り
[tp_strategy]       TP 価格に到達したら利確 (指値)。半分利確の場合は残り半分はトレール
[sl_strategy]       SL 価格を逆指値で必ず設定。SL 到達で機械的に損切
[monitor_reason]    再現性スコア 0.78 (confidence=medium)、類似局面 5 件で execute 判定
[valid_time_window] 前場
```

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| agents/daytrade_selection.py 新規 | ✅ |
| 3 dataclass + Agent クラス | ✅ |
| select_candidates 1-3 件選定動作 | ✅ |
| _generate_strategy_comment 7 要素 | ✅ |
| 20 ケース全 PASS | ✅ |
| 既存 580 PASS 非破壊 | ✅ |
| 累計 600 PASS | ✅ |
| CLI smoke (F054→F111 連結) | ✅ |
| 戦略コメントサンプル可読 | ✅ |

## 関連リンク

- 要件書: 第 4 章 (エージェント群) / 第 7 章 R-07-01, R-07-04
- 関連: [[F031_類似局面検索エンジン]] / [[F032_再現性スコア合成エンジン]] /
  [[F054_tick_py中身実装]] / [[F056_F031F032テスト追加]]
- 次タスク候補: **F115** (Trade Decision Agent、株数計算 + 注文指示生成) /
  **F062** (LINE 注文完成形テンプレート連携) / **F070** (朝レポート生成)
- コード: `~/fire/agents/daytrade_selection.py`

## スコープ外メモ

- 株数計算 → F115 Trade Decision Agent
- 資金管理 → F130 許容損失実装
- LINE 通知整形 → F062 発注完成形テンプレート
- 朝レポート全体 → F070 朝レポート生成
- Claude API 呼び出し (テンプレートで足りるため不採用)
