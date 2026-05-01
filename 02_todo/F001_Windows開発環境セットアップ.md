---
id: F001
phase: P0: 基盤・環境構築
priority: 最優先
status: 完了(Mac移行により役目終了)
owner: Fujiwara
depends_on: []
chapter: "16"
created: 2026-04-21
updated: 2026-04-24
---

# F001: Windows PC 開発環境セットアップ(Mac移行により完了扱い)

## 履歴

### 2026-04-21 Windows PC 暫定環境構築
- Git v2.47.1.windows.2
- GitHub BlueFire7777 作成、2FA設定、Private運用方針
- Python 3.12.10
- Node.js v24.15.0 (Krypton LTS)
- VS Code 1.115.0 + 拡張
- Obsidian 1.12.7
- Windows fire-vault 作成、要件書39章配置、タスク運用ルール配置
- Google Sheets TODO 管理表作成

### 2026-04-23 Mac mini 到着、移行作業
- fire-vault を Windows で ZIP 化
- Google Drive 経由で転送
- Shift-JIS 文字化け問題 → Claude 側で UTF-8 ZIP 再生成
- Mac mini の ~/fire-vault/ に ditto コマンドで展開成功

### 2026-04-24 v3.3 方針転換
- v3.2 差分パッチ受領後、議論の結果 **半自動主軸モデル(v3.3)** に方針転換
- 全自動発注(Computer Use)を撤回、LINE通知 + 人間発注に
- 要件書全40章を v3.3 対応に更新
- TODO v7 → v8 に更新(OpenClaw 関連タスク追加)
- 新Claude アカウントで継続

## 次のタスク

F017 Claudeアカウント引き継ぎ・新ランタイム確認 → F242 OpenClaw セットアップ

## 関連

- [[../タスク運用ルール|タスク運用ルール]]
