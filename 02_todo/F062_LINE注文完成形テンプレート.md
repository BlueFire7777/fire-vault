---
id: F062
phase: P5: 通知
priority: 高
status: 完了
owner: Fujiwara
depends_on: [F111, F115, F236]
chapter: "06"
created: 2026-05-02
updated: 2026-05-02
tags: [P5, notifications, line, template, milestone]
---

# F062: LINE 注文完成形テンプレート — 注文指示経路完成

## 概要

F115 `TradeOrder` を第 6 章 v3.3 の発注完成形メッセージに整形する関数。
シンプルな `format_order_message(order, paper_live=False) -> str`。

これで **F054 → F111 → F115 → F062 → F236.send_to_room → Fujiwara スマホ → iSPEED 発注**
の経路が実装レベルで完成。第 6 章 v3.3 の「LINE は FIRE と Fujiwara の唯一の発注
指示経路」が wired up された状態。

要件根拠:
- 第 6 章 v3.3 R-06-03 (通知必須項目 11 個)
- 第 6 章 v3.3 通知例 (200 株建てフォーマット)
- 第 6 章 R-06-04/05 (株数別利確パターン)

## 実装内容

### 主要モジュール

- `notifications/templates/__init__.py` (新規): `format_order_message` /
  `PAPER_LIVE_MARK` を export
- `notifications/templates/order.py` (新規):
  - `format_order_message(order, paper_live=False) -> str`: メイン関数
  - `_format_price(price) -> str`: 3 桁カンマ区切り (整数 / 小数 2 桁)
  - `PAPER_LIVE_MARK` 定数: "※ Paper Live (仮想売買、本番取引ではありません)"
  - `TYPE_CHECKING` で `TradeOrder` 型 import (循環回避)

### キーポイント

- **R-06-04/05 段数別表示**:
  - `len(take_profits) == 1` → 1 段表示
    - `quantity == tp.quantity` の場合: `"利確: 1,575"` (シンプル)
    - 一致しない場合 (部分利確等): `"利確: 1,575 で 50株"` (株数明示)
  - 2 段以上 → `"利確1: ..."` `"利確2: ..."` で連番表示
- **paper_live フラグ**:
  - True → 末尾に空行 + `PAPER_LIVE_MARK`
  - False (デフォルト) → マーク無し (Stage 3 移行時のスムーズな切替)
- **価格フォーマット**: 整数価格は `3,420` (カンマのみ)、小数は `1,500.50` (2 桁)
- **エラー処理**: `quantity <= 0` または `take_profits=[]` で `ValueError`

### 第 6 章 v3.3 通知例との一致度

完全一致 (200 株 2 段サンプル):
```
【エントリー通知】
銘柄: テスト銘柄
行動: 即発注
新規: 3,420 で 200株(信用買建)
逆指値: 3,360
利確1: 3,520 で 100株
利確2: 3,580 で 100株
有効期限: 10:15 まで
無効条件: 3,400 を明確に割れたら見送り
戦略: 前場高値ブレイク初動狙い
```

→ TestE2E `test_chapter6_sample_format` で文字列完全一致を assert (全行)

## テスト

- `tests/notifications/templates/test_order.py`: **15 / 15 PASS**
  - TestBasicFormatting (4): 100株1段 / 200株2段 / 11項目 / paper_live なし
  - TestPaperLiveMark (2): True 付加 / False 無し
  - TestPriceFormatting (3): 整数 / 小数 / 大数
  - TestTakeProfitVariations (3): qty 一致簡易表示 / qty 不一致明示 / 300株
  - TestErrorCases (2): quantity=0 / take_profits=[]
  - TestE2E (1): 第 6 章 v3.3 通知例と完全一致
- 累計 **637 PASS** (622 → 637、F040-F058 + F100 + F236 + F111 + F115 既存全 PASS 維持)

## 出力サンプル

### Paper Live モード (Stage 2)

200 株 2 段利確:
```
【エントリー通知】
銘柄: テスト銘柄
行動: 即発注
新規: 3,420 で 200株(信用買建)
逆指値: 3,360
利確1: 3,520 で 100株
利確2: 3,580 で 100株
有効期限: 10:15 まで
無効条件: 3,400 を明確に割れたら見送り
戦略: 前場高値ブレイク初動狙い

※ Paper Live (仮想売買、本番取引ではありません)
```

100 株 1 段利確 (F115 typical):
```
【エントリー通知】
銘柄: 7203
行動: 即発注
新規: 1,500 で 100株(信用買建)
逆指値: 1,470
利確: 1,575
有効期限: 10:15 まで
無効条件: 1492 を割れたら見送り
戦略: 上方修正を起点とする前場高値ブレイクを狙う

※ Paper Live (仮想売買、本番取引ではありません)
```

### 本番モード (将来 Stage 3)
Paper Live マーク無し。Fujiwara がスマホでそのまま iSPEED 発注。

## 完了条件の達成状況

| 条件 | 状態 |
|---|---|
| notifications/templates/order.py 新規 | ✅ |
| format_order_message 実装 | ✅ |
| 100 株 1 段 / 200 株+ 2 段 分岐 | ✅ |
| paper_live フラグ動作 | ✅ |
| 価格 3 桁カンマ区切り | ✅ |
| 15 ケース全 PASS | ✅ |
| 既存 622 PASS 非破壊 | ✅ |
| 累計 637 PASS | ✅ |
| 第 6 章 v3.3 通知例とほぼ一致 | ✅ (完全一致) |

## 注文指示経路完成

```
F054 candidates  →  F111 selection (1-3 件 + 戦略 7 要素)
                 →  F115 trade_decision (R-06-03 11 項目 TradeOrder)
                 →  F062 LINE 整形 (第 6 章 v3.3 メッセージ)
                 →  F236 send_to_room(Room.ENTRY)
                 →  Fujiwara スマホ
                 →  Fujiwara が iSPEED で発注
```

これで **「LINE は FIRE と Fujiwara の唯一の発注指示経路」** (第 6 章 v3.3) が
実装レベルで完成。Stage 2 (Paper Live) 運用開始の安全装置がすべて揃った状態。

## 関連リンク

- 要件書: 第 6 章 v3.3 (LINE 通知設計、発注完成形)
- 関連: [[F111_Daytrade_Selection_Agent]] / [[F115_Trade_Decision_Agent]] /
  [[F236_LINE_5段階アラート]]
- 次タスク候補:
  - **F130** 許容損失準攻撃型本格化 (F115 仮値 0.5% の本格ルール)
  - **F140** Execution Quality Gate (F115 フックの中身、流動性/スプレッド/出来高チェック)
  - **F063** その他 LINE メッセージタイプ (利確指示 / 損切指示 / 朝レポート等)
  - **パターン蓄積運用** (実材料発生時の `propose_new_candidate` 起動)
- コード: `~/fire/notifications/templates/order.py`

## スコープ外メモ

- LINE 実送信 → 既存 `notifications.router.send_to_room()` (F236) に呼び出し側で渡す
- 他のメッセージタイプ (利確指示/損切指示/朝レポート/週次レポート等) → F063/F070 系
- TradeOrder の生成・検証 → F115 で実施済み
- 多言語対応 → 日本語のみ
