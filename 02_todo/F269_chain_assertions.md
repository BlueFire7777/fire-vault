---
id: F269
phase: P9: 運用・保守
priority: 高
status: 未着手
owner: Fujiwara
depends_on: [F267]
chapter: "32,40"
created: 2026-05-04
updated: 2026-05-04
---

# [F269] Chain 異常検知 assert (再発防止)

## 基本情報

- フェーズ: P9: 運用・保守
- カテゴリ: Runtime guard
- 優先度: 高
- 依存: F267
- 担当: Fujiwara (Claude 補助)
- 想定工数: 0.5 日
- ステータス: 未着手
- 着手日: -
- 完了日: -

## タスク詳細

2026-05-04 朝の事故 (`/tmp/full_chain.sh` が exit 0 で完走したのに events=0
が 3 run 続いた、5h54m 待ち) の再発防止。

Chain ジョブが「動いた」と「機能した」を区別して落ちる仕組みを実装。
exit code を events / candidate / coverage の検証結果に紐づける。

主要な変更:
- `/tmp/full_chain.sh` を `~/fire/scripts/chain/full_chain.sh` に昇格
  (バージョン管理対象化)
- 4 種 assert を追加:
  - **events>0 必須** (1 つでも 0 なら exit 42 で異常終了)
  - **candidate>0 必須** (extract_candidates 全 skip の検出)
  - **entry>0 警告** (events はあるが entry 0 なら警告ログ)
  - **features coverage>0.5 警告** (Run 期間内の (symbol, dt) ペアの
    50% 以上で `_has_features=True` を確認、F267 の universe 正常性監視)
- 異常時は LINE 通知 (将来 F236 連携、Stage 3 後)

## 成果物

- `~/fire/scripts/chain/full_chain.sh` 新規 (`/tmp/` 版から昇格)
- `~/fire/scripts/chain/assertions.py` 新規 (4 種 assert ロジック、テスト
  容易性のため Python 化)
- `~/fire/tests/scripts/chain/test_assertions.py` 新規 (assert 各分岐の
  テスト)
- (任意) launchd plist 更新 (Chain 完了後の assert ジョブ起動)

## 関連ドキュメント

- 設計書: [[03_design/F267_implementation_2026-05-04|F267 実装レポート]]
  (セクション 6「F269 (TODO 起票済): Chain 異常検知 assert」)
- 学び: [[log.md]] (2026-05-04 朝セクション「Chain が exit 0 でも success
  とは限らない」)
- 関連タスク: [[F267_features_pipeline|F267]] (前提、coverage 計算の universe
  source)
- 関連タスク: [[F236_LINE_5段階アラート|F236]] (将来連携、異常時通知)

## 進捗チェックリスト

- [ ] F267 完了確認 (Run a で events>0 確認後に着手)
- [ ] `/tmp/full_chain.sh` の現状コードを scripts/chain/ に移管
- [ ] assert ロジック 4 種実装 (events>0 / candidate>0 / entry>0 警告 /
      coverage>0.5 警告)
- [ ] exit code 設計 (0=正常、42=events=0、43=candidate=0、警告は exit 0
      + ログのみ)
- [ ] 単体テスト (各 assert の境界値)
- [ ] 既存 Chain ジョブ動作確認 (smoke、現状 events>0 で exit 0、events=0 で
      exit 42)
- [ ] launchd plist 更新 (Chain 後ジョブとして登録 or Chain 内 trap)

## 作業ログ

- **2026-05-04 朝**: events=0 事故を受け再発防止策として起票

## 完了条件

`/tmp/full_chain.sh` が `~/fire/scripts/chain/full_chain.sh` に昇格し、
assert 4 種が動作。

`events=0` のときに Chain が **exit 42** で停止し、ログに異常理由が
記録され、(将来) LINE 通知される状態。

Run a / Run b / Run c が events=0 で「成功」と扱われる事故が構造的に
不可能になる。
