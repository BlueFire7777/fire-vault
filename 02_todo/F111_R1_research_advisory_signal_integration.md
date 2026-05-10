# F111-R1 Research Lane Advisory Signal Integration

> **Status**: ✅ COMPLETED 2026-05-10
> **Source**: F119 evaluate_signals_with_interpretation 出力 +
> F286-R2-H orthogonal_cuts 命名規約
> **Mode**: 完全 in-memory (DB write 一切なし、staging read-only も未使用)
> **Result**: ★★★ F111 候補に Research Lane / F119 由来 advisory metadata を
> 接続するための純関数群 + DaytradeCandidate wiring 完了。
> 自動発注経路への接続は __post_init__ で構造的に禁止。★★★

---

## タスク名

F111-R1 Research Lane Advisory Signal Integration

---

## 背景

- F286 Research Lane は R2-H まで完了済み
- F119 Evaluation by interpretation × sector × month も完了済み
- F119 により r2g3_recommended_v2 / R2-H insights を評価レーン側で
  継続集計できるようになった
- 次は F111 Daytrade Selection 側に Research Lane / F119 評価情報を
  接続する番

### F119 主要結果 (= advisory が参照する数値)

    overall                          h20 +2.58% / win 56.6%
    use_signal_strong                h20 +6.20% / win 64.1%
    use_signal_normal                h20 +2.88% / win 59.1%
    5月                              h20 +7.04% / win 71.7%
    6月                              h20 +5.73% / win 74.5%
    8月                              h20 +6.02% / win 65.2%
    3月                              h20 -1.77% / win 31.7%
    normal × 情報通信 × 5月         h20 +11.42% / win 88.0%
    normal × 情報通信 × 8月         h20 +10.01% / win 80.0%
    top50 × normal × 8月            h20 +10.76% / win 85.0%
    top50 × normal × 5月            h20 +9.44% / win 85.0%
    不動産 × 3月                    h20 -5.36% / win 33.3%
    情報通信 × 3月                  h20 -2.95% / win 40.0%
    cautious × 10月                 h20 -3.36% / win 23.3%

---

## F119 から F111 へ接続した目的

★ **重要**: F119 の優位は主に **20 日保有寄り** (= h20 中心)。
  したがって F111-R1 では「**daytrade 即時売買**」には接続しない。

目的は:

1. Live Advisory 候補に Research Lane / F119 由来 advisory metadata を
   付与すること
2. Fujiwara が候補を **手動レビュー** する際の判断補強情報を提供する
   こと
3. F115 / F140 / F133 即時 SL/TP / 自動発注ロジックへは
   **構造的に接続しない** こと

---

## 実装ファイル一覧

| 種別 | ファイル | 行数 | 内容 |
|---|---|---|---|
| feat | `agents/research_advisory.py` | 450 | ResearchAdvisory dataclass + builder + matchers |
| feat | `agents/daytrade_selection.py` | +27/-1 | DaytradeCandidate に advisory フィールド + attach_advisories |
| test | `tests/agents/test_research_advisory.py` | 778 | 52 ケース (invariant / matching / orchestration / wiring) |

### モジュール構成

    agents/
    └── research_advisory.py   (450 行、新規)
        ├── ResearchAdvisory (dataclass)
        │   - 21 フィールド (research_* / market_* / sector_* /
        │     month / top_bucket / f119_* / position_sizing_note /
        │     advisory_comment / advisory_version /
        │     manual_review_required / auto_order_allowed)
        │   - __post_init__ で manual_review_required=True /
        │     auto_order_allowed=False を強制矯正 (caller が逆値を
        │     渡しても上書き)
        ├── CUT_TYPE_KEYS (12 cut)
        ├── POSITION_SIZING_NOTES (5 interpretation)
        ├── _matches_cut (cut_type / group_key → ctx 該当判定)
        ├── _format_flag (mean/win % 整形)
        ├── _extract_expected_metrics (3-key → 2-key → 1-key →
        │   overall fallback、min_count=20 ガート)
        ├── _collect_flags (insight_records → matching only)
        ├── _build_advisory_comment (multi-line Fujiwara 向け)
        ├── build_advisory (純関数 orchestrator)
        └── derive_top_bucket (post_cap_rank → top10/30/50/100)

    agents/daytrade_selection.py (修正)
    └── DaytradeCandidate.advisory: Optional[ResearchAdvisory] = None
    └── DaytradeSelectionAgent.attach_advisories(result, advisories)

    tests/agents/test_research_advisory.py (新規、52 PASS)
    ├── ResearchAdvisory 不変性 (5 ケース)
    ├── _matches_cut 全 12 cut パターン (12 ケース)
    ├── _collect_flags (5 ケース)
    ├── _extract_expected_metrics fallback chain (5 ケース)
    ├── build_advisory orchestration (10 ケース)
    ├── derive_top_bucket bool/None 排除 (8 ケース)
    ├── _build_advisory_comment (2 ケース)
    └── DaytradeCandidate wiring + 既存 ctor 無回帰 (5 ケース)

---

## commit hash 一覧

| commit | type | 内容 |
|---|---|---|
| `07453f7` | feat | F111-R1: add research advisory module |
| `f4a63f1` | feat | F111-R1: wire research advisory into DaytradeCandidate |
| `1a8b03b` | test | F111-R1: add research advisory tests |
| (本 commit) | docs | F111-R1: vault |
| (次 commit) | docs | F111-R1: log milestone |

---

## ResearchAdvisory dataclass の安全仕様

### 21 フィールド

    # Research Lane (watchlist_ranker / interpretation rule)
    research_final_score: Optional[float]
    research_post_cap_rank: Optional[int]
    research_rank_label: Optional[str]                # A1/A2/B/C/D
    research_interpretation: Optional[str]            # use_signal_*/suppress_signal/unknown
    research_interpretation_detail: Optional[str]
    # 直交軸
    market_regime: Optional[str]
    sector_flow: Optional[str]
    sector_17: Optional[str]                          # 日本語 sector 名
    month_of_year: Optional[int]                      # 1..12
    top_bucket: Optional[str]                         # top10/top30/top50/top100
    # F119 insights
    f119_boost_flags: list[str]                       # 該当 strong cuts
    f119_avoid_flags: list[str]                       # 該当 avoid cuts
    f119_caution_flags: list[str]                     # 該当 caution cuts
    f119_expected_h20_return: Optional[float]
    f119_expected_h20_win_rate: Optional[float]
    f119_expected_h5_return: Optional[float]
    # 運用ヒント
    position_sizing_note: str
    advisory_comment: str                             # multi-line
    advisory_version: str = "F111-R1-v1"
    # 自動発注禁止 (絶対不変)
    manual_review_required: bool = True
    auto_order_allowed: bool = False

### __post_init__ で強制矯正される 2 フィールド

    def __post_init__(self) -> None:
        # 不変: 自動発注禁止 / 手動レビュー必須
        # 万一 caller が False/True を逆に渡しても上書きする。
        self.manual_review_required = True
        self.auto_order_allowed = False

これにより:

- `ResearchAdvisory(manual_review_required=False, auto_order_allowed=True)`
  と書いても結果は `manual_review_required=True / auto_order_allowed=False`
- to_dict() で出力した dict にも常に safe value が入る
- テスト `test_caller_cannot_force_auto_order_via_constructor` で網羅検証

---

## DaytradeCandidate wiring 内容

### 追加フィールド (1 行 + コメント)

    @dataclass
    class DaytradeCandidate:
        ...
        selected_at: str
        selection_reason: str
        # F111-R1: Research Lane / F119 由来 advisory metadata
        # (None = advisory 未付与。発注ロジックには接続しない)
        advisory: Optional[ResearchAdvisory] = None

- default `None` のため既存 ctor 呼び出し (F115 / F133 / F140 / 各テスト)
  は完全に backward compatible
- `to_dict()` の `asdict(self)` で advisory も再帰的に dict 化される
- F115 / F133 / F140 / F132 / 即時 SL/TP のロジックは `advisory` を
  参照しない (= 自動発注経路非接続)

### attach_advisories ヘルパ

    def attach_advisories(
        self,
        result: DaytradeSelectionResult,
        advisories: Mapping[str, ResearchAdvisory],
    ) -> DaytradeSelectionResult:
        """選定結果の各 candidate.symbol に対応する ResearchAdvisory
        を付与する純関数 (in-place 更新 + 同 result を返す)。

        - advisories は symbol → ResearchAdvisory の dict
        - 該当 symbol が無い候補は advisory=None のまま
        - 自動発注接続は禁止 (advisory は metadata のみ)
        """

---

## F119 matching cut の仕様

### CUT_TYPE_KEYS (12 cut、interpretation_evaluation.py の集計と一致)

    cut_type                                | 構成 keys
    --------------------------------------- | ----------------------------------------
    overall                                 | () 常に true
    interpretation                          | (interpretation,)
    sector_17                               | (sector_17,)
    month_of_year                           | (month_of_year,)
    top_bucket                              | (top_bucket,)
    interpretation_sector_17                | (interpretation, sector_17)
    interpretation_month                    | (interpretation, month_of_year)
    sector_17_month                         | (sector_17, month_of_year)
    top_bucket_interpretation               | (top_bucket, interpretation)
    interpretation_sector_17_month          | (interpretation, sector_17, month_of_year)
    top_bucket_interpretation_sector_17     | (top_bucket, interpretation, sector_17)
    top_bucket_interpretation_month         | (top_bucket, interpretation, month_of_year)

### _matches_cut のルール

- `cut_type = "overall"` → ctx 内容に関係なく True (全候補で発火)
- `cut_type` が CUT_TYPE_KEYS に無い → False
- `group_key` を `" × "` で split (= aggregate_by_two_keys と同じ separator)
- part 数 と CUT_TYPE_KEYS\[cut_type\] の長さが一致しないと False
- 各 part を ctx 値と一致比較
- ctx 値が None / 空文字 → "(none)"
- `month_of_year` の int は `f"{v:02d}"` で zero-pad
- それ以外は `str(v)` 比較

---

## expected metrics fallback 順

`_extract_expected_metrics(cut_summaries, ctx, min_count=20)` は以下の
優先順で最 specific な cut を採用する。`overall` のみ count 0 でも採用、
それ以外は `count >= min_count (=20)` を要求。

### 1. 3-key cut (最 specific)

    "interpretation_sector_17_month"
    例: "use_signal_normal × 情報通信 × 05" (count 25, +11.42%, win 88%)

### 2. 2-key cut

    "interpretation_month"
    例: "use_signal_normal × 05" (count 90, +7.14%, win 71.9%)

### 3. 1-key cut

    "interpretation"
    例: "use_signal_normal" (count 1552, +2.88%, win 59.1%)

### 4. overall (fallback)

    "overall"
    例: count 2200, +2.58%, win 56.6%

### min_count=20 gate

- F119Thresholds.min_count = 20 と整合
- count 5 の specific cut は skip され、上位の安定 cut にフォールバック
- 全 cut が None または min_count 未満 → `(None, None, None)` を返す

戻り値: `(expected_h20_return, expected_h20_win_rate, expected_h5_return)`

---

## position_sizing_note (interpretation 別 qualitative ヒント)

実際のサイズ計算は F130/F131 が stage / regime / 余力から行うため、
advisory は qualitative note のみを保持する。

    use_signal_strong   "余力の 5-10% / 銘柄 上限 (R2-H 推奨)"
    use_signal_normal   "通常 (R-05-02 ルール準拠)"
    use_signal_cautious "normal の半分目安"
    suppress_signal     "原則 0、強い裁量根拠時のみ Fujiwara 再判断"
    unknown             "interpretation 不明 → 原則見送り"

---

## advisory_comment (multi-line Fujiwara 向け)

    {symbol=7203 / } interpretation={...} / sector={...} / month={MM} / bucket={top*}
    expected: h20={±X.XX%}/win={X.X%}, h5={±X.XX%}
    boost: {flag1; flag2; ...}
    avoid: {flag1; ...}
    caution: {flag1; ...}
    ※ 自動発注禁止 / Fujiwara 手動発注前提 / manual review required

最後の 1 行 (自動発注禁止 / Fujiwara 手動発注前提 / manual review required)
は無条件で追加される (= LINE 通知や手動レビュー時の visual reminder)。

---

## 安全要件遵守

### manual_review_required=True 強制

✅ ResearchAdvisory.__post_init__ で True に上書き
✅ 全 52 テストで invariant 検証
✅ test_caller_cannot_force_auto_order_via_constructor: 逆値投入でも上書き確認

### auto_order_allowed=False 強制

✅ ResearchAdvisory.__post_init__ で False に上書き
✅ 全 52 テストで invariant 検証

### 自動発注 / broker / 楽天証券 / Computer Use 未接続

✅ agents/trade_decision.py (F115) は advisory を参照しない
✅ risk/execution_gate.py (F140) は advisory を参照しない
✅ risk/time_guard.py (F133) は advisory を参照しない
✅ risk/loss_control.py (F132) は advisory を参照しない
✅ Computer Use / Playwright / broker SDK 一切不使用
✅ advisory は metadata 専用 (= candidate.advisory フィールドのみ)

### DB write なし

✅ agents/research_advisory.py: import sqlite3 なし、Path 操作なし
✅ DaytradeCandidate.advisory 追加は in-memory dataclass field のみ
✅ staging / develop / production DB すべて未接続
✅ pure in-memory (test もモック DB すら使わない、純関数のみ)

### 既存 modified 未接触

✅ `scripts/seed_pattern_layer1.py` 一切触らず (F111-R1 commit から除外)
✅ `simulation/research_lane/historical_indicators.py` 一切触らず
✅ git status で modified のまま残存していることを 3 commit 前後で確認

---

## tests 結果

### 新規 52 PASS

    tests/agents/test_research_advisory.py
    ├── TestResearchAdvisoryInvariants (5)
    │   - test_default_manual_review_and_no_auto_order
    │   - test_caller_cannot_force_auto_order_via_constructor
    │   - test_to_dict_keeps_invariants
    │   - test_default_lists_are_empty_and_independent
    │   - test_to_dict_contains_all_required_fields
    ├── TestMatchesCut (12)
    │   - overall / unknown / interpretation / sector_17_japanese /
    │     month_padding / top_bucket / two_keys / three_keys /
    │     top_bucket_three_keys / none_value / empty_string /
    │     part_count_mismatch / cut_type_keys_table_complete
    ├── TestCollectFlags (5)
    ├── TestExtractExpectedMetrics (5)
    ├── TestBuildAdvisory (10)
    ├── TestDeriveTopBucket (8)
    ├── TestBuildAdvisoryComment (2)
    └── TestDaytradeCandidateAdvisoryIntegration (5)

### regression

- F111 / F115 / F140 / F133 / F132 全テスト 99 PASS (= 既存 + 0)
- フル pytest **2,711 PASS** (= 2,659 baseline + 52 新規)

---

## Codex pre-commit 結果

| commit | Codex 判定 |
|---|---|
| 07453f7 (feat module) | ✅ OK |
| f4a63f1 (feat wiring) | ✅ OK |
| 1a8b03b (tests)        | ✅ OK |

✅ 全 3 commit で Codex review 通過
✅ CRITICAL 指摘なし (= 修正対応 0 件)
✅ pre-commit hook (`scripts/hooks/pre-commit`) 全件正常通過

---

## --no-verify 未使用

✅ 全 3 commit で `--no-verify` flag 不使用
✅ pre-commit を bypass する一切の手段を行わず
✅ FIRE_ALLOW_WORKFLOW_CHANGE も不要 (workflow 変更なし)

---

## unrelated modified 未接触

git status に残っている既存 modified ファイルは F111-R1 commit から
完全除外:

- `scripts/seed_pattern_layer1.py` (modified のまま、本タスク無関係)
- `simulation/research_lane/historical_indicators.py` (modified のまま、
  本タスク無関係)

git status を 3 commit 前後で確認、`Changes not staged for commit:` 欄に
2 ファイル残存することを確認済み。

---

## 制約遵守チェックリスト

- ✅ 自動発注禁止 (ResearchAdvisory.auto_order_allowed=False 強制)
- ✅ 楽天証券操作の自動化なし
- ✅ Computer Use 未使用
- ✅ Playwright / Cron 強制クローズ未接続
- ✅ LINE 通知は候補提示・注文完成形提示まで (advisory は metadata)
- ✅ 発注は Fujiwara 手動実行前提 (manual_review_required=True)
- ✅ production / develop DB 不用意な変更なし
- ✅ 原則 DB write なし (= write は 1 件もなし)
- ✅ smoke は staging read-only (= F111-R1 では smoke 自体不要、純関数のみ)
- ✅ Codex pre-commit 必須 (× 3 全件通過)
- ✅ --no-verify 禁止 (× 3 全件で flag 不使用)
- ✅ 個別 commit 厳守 (3 個別 commit)
- ✅ scripts/seed_pattern_layer1.py の既存 modified 未接触
- ✅ simulation/research_lane/historical_indicators.py 未接触
- ✅ unrelated modified file を stage / commit しない

---

## 次タスク提案

### 推奨 1: F111-R2 Advisory Wiring Runner (read-only orchestrator)

実際に F119 evaluate_signals_with_interpretation 出力 →
AdvisoryBuilder → DaytradeSelectionAgent.attach_advisories の
**連結スクリプト** (read-only runner) を組む。

- 入力: F286-R2-F4 baseline signals + R2-G3 recommended_v2 +
  F119 evaluate 出力
- 出力: candidate × advisory (JSON / Markdown)
- DB write 不要、staging read-only も使わず in-memory orchestration

### 推奨 2: F062-R1 LINE Advisory Notification Template

LINE 5 部屋 (ENTRY 部屋) の通知に advisory metadata を載せる
テンプレート拡張。

- F062 既存 LINE template に advisory_comment / position_sizing_note /
  f119_boost_flags の見出しを追加
- 既存 R-06-03 11 項目 TradeOrder 整形は触らず、advisory ブロックを
  下段に追加するだけ
- LINE 配信は pure in-memory composition

### 推奨 3: R2-G4 5d 用 rule 設計

F119 で確認された h5 短期 vs h20 中期の乖離 (cautious h5 -3.06% / h20
+0.59%) を踏まえ、5 日保有用の interpretation rule を設計。

優先度: 1 > 2 > 3 (Live Advisory 接続準備の流れに合わせる)

---

## 関連参照

- 02_todo/F119_interpretation_evaluation.md (前段)
- 02_todo/F286_R2_H_orthogonal_cuts.md
- 02_todo/F286_R2_G3_rule_finalization.md
- 02_todo/F111_Daytrade_Selection_Agent.md (本体)
- log.md milestone (本タスク完了時に追記)
