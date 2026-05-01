---
type: requirement_chapter
chapter: "16"
title: "Mac miniセットアップ"
version: v3.1
updated: 2026-04-21
---

# 第16章: Mac miniセットアップ

## Mac miniセットアップの要点

1. **システム設定** → 省エネルギー → ディスプレイをオフにするまでの時間を「しない」に設定
2. **コンピュータのスリープ**も「しない」に設定
   - これにより、場中にMac miniが勝手にスリープしてエージェントが止まる事態を防ぐ
3. **自動再起動**の設定
   - システム設定 → 省エネルギー → 停電後に自動的に再起動を有効化
   - 停電・瞬断後にFIREが自動復帰できるようにする
4. **自動ログイン**の設定(任意)
5. **ファイアウォールと画面共有**の設定

## 開発環境のセットアップ

1. **Homebrew**のインストール
2. **Python 3.12**のインストール: `brew install python@3.12`
3. **Node.js**(Dashboard用)のインストール: `brew install node`
4. **Git**のセットアップ: user.name / user.email 設定
5. **FIREリポジトリ**のクローン: `git clone https://github.com/自分のアカウント/fire.git ~/fire`
6. **環境変数**の設定
   - `RAKUTEN_API_KEY`
   - `LINE_CHANNEL_TOKEN`
   - `NEWS_API_KEY`
   - `TDNET_API_KEY`

## 常駐ランタイムの自動起動設定(launchd)

FIREのスケジューラ(常駐ランナー)は、Mac miniの起動時に自動で立ち上がるように設定する。macOSでは**launchd**を使うのが標準的な方法。

`~/Library/LaunchAgents/com.fire.runtime.plist` を作成し、`launchctl load` で登録する。

## リモートアクセスの設定

Mac miniはディスプレイなしのヘッドレス運用も可能。外出先や別の部屋からFIREを監視・操作するために以下を設定する。

- **画面共有(VNC)**: 同一ネットワーク内からiPadやMacBookで操作できる
- **SSH**: ターミナルからリモートでコマンド実行
- **Tailscale(推奨)**: VPNを使わずに外出先から自宅のMac miniに安全にアクセスできる無料ツール

## セットアップ完了後の動作確認チェックリスト

| 確認項目 | 確認方法 |
|---|---|
| スリープしないか | 数時間放置してSSHで接続できるか確認 |
| 常駐ランタイムが起動しているか | `launchctl list | grep fire` で確認 |
| ログが出力されているか | `tail -f ~/fire/logs/runtime.log` で確認 |
| 環境変数が読み込まれているか | Pythonで `os.environ` 確認 |
| LINE通知が届くか | テスト通知を送信して確認 |
| 楽天証券APIへの接続確認 | 残高取得APIをテスト実行 |

## 本章の要件ID

- **R-16-01** ディスプレイ・コンピュータスリープしない
- **R-16-02** 停電後自動再起動
- **R-16-03** 自動ログイン設定(任意)
- **R-16-04** ファイアウォール・画面共有
- **R-16-05** Homebrew/Python3.12/Node/Gitインストール
- **R-16-06** FIREリポジトリclone
- **R-16-07** 環境変数設定(RAKUTEN/LINE/NEWS/TDNET)
- **R-16-08** launchd com.fire.runtime.plist
- **R-16-09** SSH/Tailscaleリモートアクセス
- **R-16-10** セットアップ完了後動作確認チェックリスト


---

## ナビゲーション

[[FIRE_要件書_第15章_Claude_Code実装に向けた次のステップ|← 第15章: Claude Code実装に向けた次のステップ]]　|　[[FIRE_要件書_第17章_執行品質設計(穴2対策)|第17章: 執行品質設計(穴2対策) →]]

- 索引: [[_index|要件書 章索引]]
- トップ: [[../00_index/README|README]]
- タスク: [[../02_todo/_index|TODO索引]]
- 運用ルール: [[../タスク運用ルール|タスク運用ルール]]
