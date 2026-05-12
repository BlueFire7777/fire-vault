---
id: F282-snapshot-retention-cleanup-plan
phase: W30 snapshot retention cleanup 設計
priority: 中
status: 設計 v1.0 (= Wave 34 W34-3 L1b 反映、cleanup は別 wave + HQ 明示承認後)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-12
depends_on:
  - Wave 30 (= 完了、実 VACUUM INTO 試行 + snapshot 2 件作成)
  - Wave 34 起票承認 (= 2026-05-12)
---

# W30 snapshot retention cleanup 設計

## 0. 概要

Wave 30 で実 VACUUM INTO 試行成功 → /Users/bluefire/fire/data/snapshot/
に snapshot 2 件作成 (= retention 案 B、1 週間保持)。本 doc は **削除計画
の設計のみ**、本 wave (= Wave 34) では削除しない。

## 1. cleanup 対象 baseline (= 削除前 sha256 / size、本 wave 取得)

| file | size (bytes) | mtime | sha256 |
|---|---|---|---|
| /Users/bluefire/fire/data/snapshot/fire.staging.db | 353,128,448 | 2026-05-12 21:43:59 | 749916c877670de68740bb85ad7903fa5b7e1f8488dac123edaf9e355e17ee72 |
| /Users/bluefire/fire/data/snapshot/fire.develop.db | 353,128,448 | 2026-05-12 21:44:00 | 749916c877670de68740bb85ad7903fa5b7e1f8488dac123edaf9e355e17ee72 |

★ 両者 sha256 完全一致 (= 同 source production fire.db からの VACUUM INTO で
決定論的出力、期待通り)

## 2. retention 方針 (= Wave 30 案 B 採用)

- 保持期間: **2026-05-12 〜 2026-05-19 月曜頃** (= 約 1 週間)
- 削除タイミング: **5/19 試走 GO 判定後、別 wave + HQ 明示承認後**
- 本 wave (= Wave 34) では **削除しない**

## 3. cleanup 着手判断基準

| 基準 | 確認 |
|---|---|
| F282 1 週間試走 GO 判定 | 5/19 月曜 (= F282_weekly_snapshot_trial_monitoring 参照) |
| F282 launchd 自動実行安定動作 | 5/16 + daily check 全 PASS |
| snapshot 検証用途完了 | 必要なら手動 sha256 / integrity_check 再確認 |
| HQ 明示承認 | HQ_APPROVE_SNAPSHOT_CLEANUP=1 想定 |

**全 4 件 充足 → cleanup wave 起票可**。

## 4. cleanup 実行手順 (= 別 wave 用設計)

### Step 1: 削除前確認 (= baseline 一致性)

```bash
# 1-a: 現状確認
stat -f "%m %z %N" /Users/bluefire/fire/data/snapshot/fire.staging.db
stat -f "%m %z %N" /Users/bluefire/fire/data/snapshot/fire.develop.db
# 期待: 本 doc § 1 baseline と完全一致 (= 改ざんなし)

# 1-b: sha256 再計算
shasum -a 256 /Users/bluefire/fire/data/snapshot/fire.staging.db
shasum -a 256 /Users/bluefire/fire/data/snapshot/fire.develop.db
# 期待: 749916c8...e17ee72 (両 file 同一、本 doc § 1 と一致)
```

### Step 2: baseline mismatch 検出時

```bash
# 一致しない → cleanup 中断 + incident
# → 07_incidents/W30_snapshot_tampering_YYYY-MM-DD.md 作成
# → HQ 報告
```

### Step 3: 削除 (= baseline 一致確認後)

```bash
# 3-a: 本体削除
rm /Users/bluefire/fire/data/snapshot/fire.staging.db
rm /Users/bluefire/fire/data/snapshot/fire.develop.db

# 3-b: dir empty 確認
ls -la /Users/bluefire/fire/data/snapshot/
# 期待: total 0 (= 空 dir)
```

### Step 4: dir 削除 (= 任意)

```bash
# 4-a: 空 dir なら削除
rmdir /Users/bluefire/fire/data/snapshot/
# または保持 (= 将来別 snapshot 試行用)
```

## 5. 削除後確認項目

| 項目 | 確認方法 | 期待 |
|---|---|---|
| data/snapshot/ 内 0 件 | `ls data/snapshot/` | 出力なし or dir 不在 |
| production fire.db unchanged | `stat -f "%m %z" data/fire.db` | baseline 一致 |
| 既存 staging.db unchanged | 同上 fire.staging.db | baseline 一致 |
| 既存 develop.db unchanged | 同上 fire.develop.db | baseline 一致 |
| disk free 増加 | `df -h data/` | 約 706 MB 増 |

## 6. cleanup 失敗時

| 失敗 | 対応 |
|---|---|
| rm エラー (= permission / 不在) | HQ 報告、調査後再試行 |
| sha256 mismatch | incident doc + HQ 報告、cleanup 中止 |
| 削除後 既存 DB mtime 変化 | 緊急 incident (= 別 file 誤削除 risk) |

## 7. 安全制約 (= cleanup wave + 本 wave 共通)

- **既存 production / 既存 staging / 既存 develop DB は絶対に削除しない**
- delete 対象は `/Users/bluefire/fire/data/snapshot/` 配下のみ
- W30 で確立した snapshot 専用 path から逸脱しない
- 本 wave (= Wave 34) では rm / unlink 実行 0

## 8. 想定 cleanup wave (= Wave 35+ 候補)

```
Wave 35-a (= 仮): F282 GO 判定 + cleanup
- W35-1 L5 plan + 5/19 GO 判定 + baseline 確認
- W35-2 L1a cleanup 詳細手順
- W35-3 L3 (本線) sha256 + rm 実行
- W35-4 L4 audit
- W35-5 L6 regression
- HQ_APPROVE_SNAPSHOT_CLEANUP=1 必須
```

## 9. 関連リンク

- [[F282_weekly_snapshot_trial_monitoring_2026-05-12|F282 試走監視テンプレ]]
- [[F282_weekly_snapshot_launchd_2026-05-12|F282 launchd 設計]]
- [[../02_todo/FIRE_CODEX_R1_WAVE30_results|Wave 30 results (= snapshot 作成元)]]
- [[../02_todo/FIRE_CODEX_R1_WAVE34_plan|Wave 34 plan]]
