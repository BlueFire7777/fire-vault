# FIRE 開発 Obsidian Vault

日本株AIエージェントシステム「FIRE」の開発・運用記録用 Vault。

## v3.3 の方針(2026-04-24 更新)

**半自動主軸モデル**へ方針転換。詳細は [[01_requirements/_index|要件書 v3.3 章索引]] 参照。

- ✅ OpenClaw 採用(オーケストレーション層・常駐基盤)
- ✅ LINE通知 + 人間発注(Fujiwara さん)が主軸
- ✅ 強制クローズは LINE 緊急アラート 5段階
- ❌ Computer Use による自動発注は撤回
- ❌ 独立 Cron による強制クローズ自動化は撤回
- 🎯 Stage 3 (Live Advisory) が FIRE の実質的ゴール
- 🎯 Stage 4 (Full Auto) は将来オプション

## クイックリンク

- 🔗 **[TODO管理表 (Google Sheets - 正本)](https://docs.google.com/spreadsheets/d/1Zf0rmQHxuBLWsBriV3hKH51eGiJ5u96b/edit?gid=1782363065#gid=1782363065)**
- [[02_todo/_index|TODO 索引(Obsidian側)]]
- [[01_requirements/_index|要件書 v3.3 章索引]]
- [[タスク運用ルール]]

## ディレクトリ構成

| ディレクトリ | 用途 |
|---|---|
| 00_index | トップページ・索引 |
| 01_requirements | 要件書・仕様関連(v3.3 対応) |
| 02_todo | タスクごとのノート(F001, F002...) |
| 03_design | 設計書(Pattern Store, Feature Store, Agent仕様など) |
| 04_daily | 日次メモ |
| 05_weekly_review | 週次レビュー |
| 06_monthly | 月次レビュー |
| 07_incidents | 障害・事故・気づき |
| 08_reference | 参考資料・外部リンク |

## 現在のフェーズ

**開発初期 (P0: 基盤・環境構築)**

- 開発マシン: Mac mini M4 Pro(到着・稼働中)
- 現在: Windows → Mac 移行完了 → F017(新Claudeランタイム)→ 次は F022/F242(OpenClaw セットアップ)
- 新Claudeアカウントで v3.3 の方針に基づき開発継続

## 関連リソース

- TODO 管理(正本): Google Sheets(上記リンク)
- FIRE コードリポジトリ: ~/fire(F004 で初期化予定)
- GitHub: github.com/BlueFire7777/fire(F004 で作成予定・Private)
- 要件書 PDF: 原本 v3.1 + 差分 v3.2 + v3.3 は本 Vault の .md
- TODO ローカルバックアップ: FIRE_開発TODO_v8.xlsx

## 役割分担(管理の二重化を避ける)

- Google Sheets = TODO管理の正本
- Obsidian (本Vault) = タスクの詳細メモ・実装ログ・設計書・日次レビュー
- ローカル xlsx = Google Sheets のバックアップ
