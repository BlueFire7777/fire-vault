---
id: F236-R1.1
phase: P5: 通知 / 第 14 章 LINE 通知配信
priority: 高
status: 完了 (2026-05-11、in-place sanitize 実施)
owner: BlueFire7777 (Fujiwara)
depends_on:
  - F236-R1 (LineBotClient log mask、新規 log の masked 形式化)
  - F062-R4.2 (output recipient masking)
chapter: 第 14 章 / 第 26 章
---

# F236-R1.1: Legacy LINE Log Sanitize / Rotate

最終更新: 2026-05-11

## 目的

F236-R1 で **新規** LINE log は recipient_id masked 形式になったが、
既存 `logs/notifications/notifications_line.log` には F062-R3 / R4
試行時を含む **過去ログに full recipient_id (U/C/R 始まりの LINE
userId) が大量に残存** していた。F062-R5 本番 Advisory 送信前に、
過去ログを sanitize して recipient leak 経路を完全に塞ぐ。

## 対象ファイル

- `logs/notifications/notifications_line.log` (= LineBotClient._log の
  追記先)
- ファイル mode: 0o644 (sanitize 後も維持)
- git status: **ignored** (= `logs/` は .gitignore 対象)
  → fire commit 対象外、本タスクは vault docs のみ

## 採用方針: in-place sanitize (退避ではなく)

タスク仕様「退避ファイルにも full recipient_id が残るなら、退避では
なく sanitize」に基づき、退避 backup を作らず in-place で書き換え。
データ消失防止のため atomic write (tempfile + os.replace) を採用。

書き換え方針:
- 各行の col 3 (= to カラム) を `format_masked_recipient_field(...)`
  で masked 形式に置換
- 既に masked 形式 (= "user:" / "group:" / "room:" / "unknown:" 始まり)
  の行は触らない
- timestamp / mode / message 本文は保持

## 実施結果 (= 値そのものは記録しない、件数のみ)

### sanitize before
| field | count |
|---|---|
| total_lines        | 901 |
| masked (touched 不要) | 5 |
| ucr_full (= U/C/R 始まり full ID) | 792 |
| other_to (= U/C/R 以外の非 masked) | 103 |
| short (= col < 4 の異常行) | 1 |

### sanitize after
| field | count |
|---|---|
| total_lines        | 901 (= 行数保持) |
| masked             | 900 |
| ucr_full           | **0** ★ |
| other_to           | **0** ★ |
| short              | 1 |

### 補足検査
| 検査 | 結果 |
|---|---|
| unmasked U/C/R full ID の to カラム残存 | 0 件 ✅ |
| 60 文字以上連続 ASCII (= token 候補) 残存 | 0 件 ✅ |
| masked field 数 (= "user:prefix=" 等) | 900 件 |
| file mode (sanitize 後) | 0o644 ✅ |
| file size | 190,327 → 209,542 (= masked 形式は full ID より少し長い) |

## sanitize 手順 (実行ログ、reproducibility 用)

```python
# 簡略再掲、値は出力しない
import os, re, tempfile
from pathlib import Path
from notifications.recipient_mask import format_masked_recipient_field

p = Path("logs/notifications/notifications_line.log")
masked_re = re.compile(r'^(user|group|room|unknown):')

# 1. read all lines
# 2. col 3 (to) を、既に masked でない場合のみ
#    format_masked_recipient_field で置換
# 3. tempfile に書き出し → 元 file mode を chmod で復元 →
#    os.replace で atomic 置換
```

退避ファイル: **作っていない** (= 退避先にも full ID が残るので)。

## 安全要件遵守

| 項目 | 結果 |
|---|---|
| 実 LINE 送信なし                                        | ✅ (= LineBotClient.send_text 未呼出) |
| token 平文出力なし (画面 / file / report)                | ✅ |
| recipient_id 平文出力なし (画面 / report)                | ✅ (件数のみ表示) |
| sanitize 後の log file 内に full recipient_id 残存        | **0 件** ✅ |
| DB write なし                                           | ✅ |
| data/fire.db / fire.develop.db / fire.staging.db 全 unchanged | ✅ |
| 自動発注 / 楽天操作 / Computer Use                       | 0 ✅ |
| TODO Excel 未更新                                        | ✅ |
| --no-verify 未使用                                       | ✅ |
| scripts/seed_pattern_layer1.py / historical_indicators.py | 未接触 ✅ |
| unrelated modified を stage / commit                    | しない ✅ |
| fire コード変更                                         | なし (= logs/ は .gitignore 対象、コード変更不要) |

## sanitize 後の挙動

新規 LINE 送信があった場合は F236-R1 の `_log` 改修で必ず masked 形式
で追記される (例: `2026-05-11T...Z\tDRY\tuser:prefix=U:len=33:hash8=ab12cd34
\ttest message`)。

つまり今後 logs/notifications/notifications_line.log には full
recipient_id が **構造的に追加されない**。

## 注意 / 残課題

- **本タスクで実 LINE 送信は行っていない** (= 過去ログ書き換えのみ)。
  新規 masked 形式の追記は F062-R5 / 通常 Advisory 送信時に発生する。
- 本タスクは 1 回限りの運用作業。スクリプトは vault doc 内に簡略再掲
  したのみで、`scripts/` 配下に永続化していない (= 再実施が必要に
  なるケースが想定されないため)。
- 必要に応じて将来 log rotate (= logrotate / launchd / cron で日次
  rotate) を導入する場合は別 task で検討。

## 次タスク

1. ★ F236-R1.1 完了 (本書記録)
2. F062-R5 First Production Advisory Small Launch (HQ 判断後に開始)
   → recipient/token leak が log file まで含めて全 sink で 0 になり、
     本番 Advisory 送信前準備が完全に整った
3. 並走候補:
   - F286-DATA-R3 daily refresh cron 化 (post_launch_high_priority)
   - F242 OpenClaw / F022 FIRE Runner / F013 launchd
     (post_launch_high_priority)
