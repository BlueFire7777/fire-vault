---
type: index
version: v3.3
updated: 2026-04-24
---

# FIRE TODO 索引

**正本: [TODO管理表 Google Sheets](https://docs.google.com/spreadsheets/d/1Zf0rmQHxuBLWsBriV3hKH51eGiJ5u96b/edit?gid=1782363065#gid=1782363065)**

本ノートは Obsidian 上での簡易索引。タスクの追加・ステータス更新・優先度変更は必ず Google Sheets 側で行うこと。

## v3.3 での変更

- 自動発注関連タスク(F104 等)を削除
- OpenClaw 関連タスクを最優先に追加(F242: OpenClaw セットアップ)
- LINE 緊急ポジション整理部屋実装(F236)
- Gmail API 約定取込(F240)
- Live Advisory 昇格条件検証(F241)
- Full Auto フェーズは優先度「低」に一律変更

## 現在進行中

- [[F001_Windows開発環境セットアップ]] ✅ 完了(Mac 移行により役目終了)

## Mac 到着後の着手順(v3.3)

1. F017 Claudeアカウント引き継ぎ・新ランタイム確認 ← **いまここ**
2. F022 FIRE Runner 骨組み
3. F242 OpenClaw セットアップ(Mac mini 常駐基盤)🆕
4. F013 Mac mini launchd 常駐自動起動設定
5. F235 楽天証券 約定通知メール設定 🆕
6. F004 Gitリポジトリ初期化
7. F005 CLAUDE.md 初稿作成
8. F006 README.md 初稿作成
9. Pattern Store / Feature Store 実装(P1)
10. LINE 通知基盤 + 緊急ポジション整理部屋(P4)

## フェーズ一覧(v3.3)

| フェーズ | タスク数(目安) | 開始条件 |
|---|---|---|
| P0: 基盤・環境構築 | 約20 | 今ここ |
| P1: Pattern/Feature Store | 約20 | P0完了 |
| P2: Simulation/Backtest | 約10 | P1主要完了 |
| P3: Paper Live | 約5 | P2完了 |
| P4: LINE/通知基盤 | 約25 | 並行可(緊急ポジション整理含む) |
| P5: Dashboard | 約12 | 並行可 |
| P6: 証券API連携 | 約5(Gmail API 中心) | P5後半 |
| P7: Live Advisory | 約5(旧 Semi Auto から改名) | P3完了後 |
| P8: Full Auto(将来) | 約5(優先度低) | オプション |
| P9: 運用・保守 | 5 | 継続 |

## 関連

- [[../タスク運用ルール|タスク運用ルール]]
- [[../01_requirements/_index|要件書 v3.3 章索引]]
