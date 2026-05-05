# TODO Excel Vault 同期戦略 (2026-05-05、F278 Phase 4 確立)

## §1 概要

F278 Phase 4 で確立した TODO Excel と Vault の同期運用ルール。
現状の手動運用が F277残作業 / F278-Pre / F278 Phase 1-3 で実証済の
ため、現状継続 + 運用ルール明文化のみとする。

## §2 現行運用フロー

### §2.1 タスク起票

1. 本部 (Claude.ai) が必要に応じてタスク起票内容案を作成
   - タスク名 / Phase / Priority / 工数 / 依存 / 詳細 / 備考
   - Fujiwara 確認質問付きの場合あり (例: F261-fix Q1-Q4)
2. Fujiwara が起票内容を承認、TODO Excel に手動反映
   - 行追加位置 (関連 ID 近接 or 末尾)
   - 既存ID との関連明記
3. 反映後、本部 + Mac mini はタスク ID で参照可能

### §2.2 ステータス更新

1. Mac mini Claude Code がタスク完了報告 (本部に commit hash + 工数 + 結果)
2. 本部が完了承認、Fujiwara に進捗共有
3. Fujiwara が TODO Excel の完了ステータスを手動更新
4. 本部は提案・要請のみ実施、自動更新は行わない

### §2.3 Vault 集約ファイルとの参照関係

TODO Excel = タスク管理の正本 (一次情報)
Vault 03_design/ 配下 = 設計記録・Codex 監査記録・handover (二次情報)
タスク ID で相互参照、Vault ファイル名にタスク ID を含める命名規則

例:
- F277_B_handover_2026-05-04.md (F277-B 完了記録)
- F278_pre_critical_handling_2026-05-04.md (F278-Pre §3.6 ライブラリ)
- F050_handover_2026-05-XX.md (F050 retroactive 監査、commit message 集約)
- F051 commit message (修正 commit に集約)
- F053_handover_2026-05-05.md (F053 時系列内 fix 確認)
- F142_handover_2026-05-05.md (F142 既知失敗解消)
- f271_v12_candidates_2026-05-05.md (F271 v1.2 候補集約)
- todo_excel_vault_sync_strategy_2026-05-05.md (本ファイル)

## §3 役割分担

| 役割 | 担当 | 範囲 |
|---|---|---|
| タスク起票内容作成 | 本部 (Claude.ai) | タスク詳細案 + Fujiwara 確認質問 |
| TODO Excel 反映 | Fujiwara | 起票内容の Excel 行追加 + 完了ステータス更新 |
| 設計判断 | 本部 + Fujiwara | 戦略レベル判断 (例: F261-fix Q1-Q4) |
| 実装 | Mac mini Claude Code | コード修正 + テスト + commit |
| Codex 連携 | Mac mini Claude Code | Codex pre-commit + 全体 review |
| Vault 記録 | 本部 + Mac mini Claude Code | 設計記録 / handover / 監査記録 |
| 戦略決裁 | Fujiwara | F266 最終ゲート、Stage 3 開始日合意 |

## §4 タスク種別ごとの記録粒度ルール

F278 Phase 1-3 で確立した記録粒度ルール:

### §4.1 修正 commit を発行する場合
- commit message に詳細集約 (Phase / 紐づく F / Codex 検出内容 /
  修正方針 / 検証結果)
- Vault 記録は不要 (commit message が一次情報、git log で追跡可能)
- 例: F050 案 b-3 (commit 74de54e)、F051 案 a (commit f1c628f)、
      F141 案 b-1 (commit ef97e8e)

### §4.2 修正 commit を発行しない場合 (時系列内 fix 済 / 申し送りのみ)
- Vault に handover ファイル作成 (~/fire-vault/03_design/F___handover_<date>.md)
- §1-§7 の構造化記録 (概要 / 検出内容 / 既存解消 / HEAD 現状 / 監査結論 /
  Phase 5 参考価値 / 関連)
- 例: F053_handover_2026-05-05.md

### §4.3 設計判断 / handover (修正に紐づかない記録)
- Vault に handover ファイル作成
- 設計選択肢 + 採択理由 + 関連 commit hash
- 例: F277_B_handover_2026-05-04.md、F142_handover_2026-05-05.md

### §4.4 ルール改訂候補 / パターンライブラリ
- Vault に集約ファイル作成 (例: f271_v12_candidates_2026-05-05.md)
- F278 Phase 5 で正式採用判断対象

## §5 自動化評価 (現時点の結論)

### §5.1 評価軸

a. TODO Excel → Vault 自動同期 (Excel 更新時に Vault に記録自動生成)
b. Vault → TODO Excel 自動同期 (Vault commit 時に Excel 自動更新)
c. 半自動化 (本部が起票内容を Markdown 出力 → Fujiwara が Excel 取込)
d. 現状継続 (本部起票 → Fujiwara 手動 Excel 反映)

### §5.2 結論: d (現状継続)

根拠:
1. 現状運用が機能している実証あり
   - F277残作業 / F278-Pre / F278 Phase 1-3 すべて TODO Excel 反映なしで完遂
   - 必要時のみ起票 (F261-fix 起票で TODO Excel v9_22 更新済)
2. 自動化の費用対効果が不明確
   - Excel ↔️ Vault の整合性チェック実装に数日工数想定
   - Stage 3 起動前の優先順位は F261-fix / F276 / F266
3. Stage 3 起動後の運用実績で再評価
   - Stage 3 で events ≥ 50 達成までの蓄積期間に運用パターンが固まる
   - その時点で半自動化の必要性が顕在化すれば再判断

### §5.3 将来の自動化検討タイミング

以下のいずれかが満たされた時点で再評価:
- Stage 3 起動後 1-3 ヶ月運用実績取得
- TODO Excel と Vault の整合性ミスマッチが運用上問題化
- タスク数が現在の 100+ 件から 200+ 件に拡大

## §6 関連ファイル

- ~/fire-vault/タスク運用ルール.md (運用ルール本体、本ファイルへの参照リンク追加)
- ~/fire-vault/CLAUDE.md (プロジェクト全体ルール)
- FIRE_開発TODO_v9_22_F261fix追加.xlsx (TODO Excel 最新版、Fujiwara管理)
- ~/fire-vault/03_design/F277_B_handover_2026-05-04.md
- ~/fire-vault/03_design/F278_pre_critical_handling_2026-05-04.md
- ~/fire-vault/03_design/f271_v12_candidates_2026-05-05.md

## §7 Phase 5 改訂時の参考価値

本ファイル §4 の「タスク種別ごとの記録粒度ルール」は F278 Phase 1-3
で実証されたフォーマット、F271 v1.2 改訂時に「retroactive 遡及監査の
記録粒度ルール」として §3.6.13 補強観点に組み込み判断対象。
