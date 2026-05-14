---
id: F062-no-send-trial-checklist-and-v0-readiness
phase: 本番 v0 Launch / Phase C-D-E readiness
priority: 高
status: 設計 v1.0 (= Wave 40.5、軽量 wave、F282 試走前)
owner: BlueFire7777 (Fujiwara)
date: 2026-05-13
depends_on:
  - Wave 40-pre (= DATA-R3 launchd 設計 v1.0)
  - Wave 40-post (= F062 morning advisory launchd 設計 v1.0)
  - HQ 補足 (= Phase P2 軽量 wave、本線 + 2 lane)
related:
  - 03_design/FIRE_production_v0_launch_plan_2026-05-13.md
  - 03_design/F286_DATA_R3_daily_refresh_launchd_2026-05-13.md
  - 03_design/F062_morning_advisory_launchd_2026-05-13.md
chapter: production-v0 / Phase D / Phase E readiness
---

# F062 No-Send Trial Checklist + v0 Readiness — Wave 40.5

最終更新: 2026-05-13

## §1 目的

Wave 45 (= F062 morning advisory plist 実配置 + 1 週間 no-send 試走) で
即着手できる pre-flight / run-time / post-check / abort 条件を明文化。
合わせて Production v0 Launch Plan Phase D (= no-send trial) / Phase E
(= D-Day v0 開始) の GO/NO-GO 条件 + HQ approve marker 順序を整理。

本 wave は **軽量設計のみ** (= 本線 + 2 Codex lane)。F282 本番試走
(5/16 02:00) 直前のため実行系変更を最小化。

## §2 現在地

| 項目 | 状態 |
|---|---|
| Phase A foundation | 完成 (= W37 staging coverage smoke) |
| Phase B foundation (DATA-R3 launchd 設計) | 完成 (= Wave 40-pre v1.0) |
| Phase C foundation (F062 launchd 設計) | 完成 (= Wave 40-post v1.0) |
| Phase D/E readiness | 本 wave で整理 (= Wave 40.5) |
| F282 本番試走 | 5/16 02:00 待機 |
| 6/9 D-Day v0 | 確定 (= W44.5-pre、火曜) |

完成済 foundation:
- DATA-R3: 月-金 06:30 JST 起動、aggregate runner で 4 sub-runner --dry-run
- F062 morning advisory: 月-金 08:45 JST 起動、preview / no-send mode

## §3 Wave 45 no-send trial pre-flight checklist (= plist 配置前)

Lane A §A 反映。Wave 45 実作業の直前に逐次実行する 12 項目:

1. `test ! -e ~/Library/LaunchAgents/jp.fire.morning-advisory.plist`
   で実配置がまだ無いことを確認
2. `sed -n '1,220p' ~/fire/docs/launchd/jp.fire.morning-advisory.plist`
   で Wave 45 配置予定 plist 内容を確認
3. `plutil -lint ~/fire/docs/launchd/jp.fire.morning-advisory.plist` が `OK`
4. `launchctl list | grep jp.fire.daily-refresh` で DATA-R3 既配置確認
   (= Wave 41 完了前提)
5. `launchctl print gui/$(id -u)/jp.fire.daily-refresh | grep -A8 StartCalendarInterval`
   で月-金 06:30 確認
6. `stat -f '%Sm %N' ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist`
   で F282 plist mtime baseline 記録
7. `stat -f '%Sm %N' ~/fire/data/fire.db ~/fire/data/fire.develop.db ~/fire/data/fire.staging.db`
   で 3 環境 DB mtime baseline 記録
8. `grep -E 'LINE_|CHANNEL_|JQUANTS_|TOKEN|SECRET' ~/fire/docs/launchd/jp.fire.morning-advisory.plist`
   が 0 件
9. `python scripts/jobs/run_f062_research_advisory_line_preview.py --help | grep -E 'no-send|preview-only|dry-run'`
   で preview 系 option 確認
10. `python scripts/jobs/run_f062_line_production_send_smoke.py --help | grep hq-approved-send || true`
    で runner-level `--hq-approved-send` absent → 送信拒否 pattern 確認
11. `grep -R "max_chunks=1\|partial_delivery=False" scripts/jobs -n`
    で 1 chunk send / partial delivery 禁止 pattern 確認
12. `test "$HQ_APPROVE_LAUNCHD_MORNING_ADVISORY" = "1"`
    (= Wave 45 実作業直前のみ必須、未承認なら plist 配置しない)

## §4 Wave 45 no-send trial run-time checklist (= launchctl load 直後)

Lane A §B 反映。launchctl load 1 回のみ実行後 10 項目:

1. `launchctl load ~/Library/LaunchAgents/jp.fire.morning-advisory.plist` 1 回のみ
2. `launchctl list jp.fire.morning-advisory` 登録確認、`PID=-` 期待
3. `launchctl print gui/$(id -u)/jp.fire.morning-advisory | grep LimitLoadToSessionType`
   = `Aqua`
4. `launchctl print ... | grep RunAtLoad` = `false`
5. `launchctl print ... | grep -A20 StartCalendarInterval` で 5 weekday entry
6. `launchctl print ... | grep -E 'Hour|Minute|Weekday'` で月-金 08:45 のみ
7. `launchctl print ... | grep -E 'LINE_|CHANNEL_|JQUANTS_|TOKEN|SECRET'` 0 件
8. `launchctl print gui/$(id -u)/jp.fire.weekly-snapshot | grep -E 'last exit|LastExit|02:00|StartCalendarInterval'`
   で F282 LastExit 0 + 5/16 02:00 維持
9. `launchctl print gui/$(id -u)/jp.fire.daily-refresh | grep -E '06:30|StartCalendarInterval|last exit|LastExit'`
   で DATA-R3 月-金 06:30 維持
10. `stat -f '%Sm %N' ~/Library/LaunchAgents/jp.fire.weekly-snapshot.plist ~/Library/LaunchAgents/jp.fire.daily-refresh.plist`
    で F282/DATA-R3 plist 不変

## §5 Wave 45 post-check checklist (= 1 週間試走中の毎朝)

Lane A §C 反映。試走中 5 営業日連続で実行する 12 項目:

1. `launchctl print gui/$(id -u)/jp.fire.morning-advisory | grep -i runs`
   累積起動回数 前日比 +1
2. `launchctl print ... | grep -Ei 'last exit|LastExit'` exit code 0
3. `stat -f '%Sm %N' ~/fire/logs/cron/morning-advisory.log` mtime 当日朝
4. `grep 'F062 morning advisory preview OK' ~/fire/logs/cron/morning-advisory.log` 1 件以上
5. `test ! -s ~/fire/logs/cron/morning-advisory.err || grep -Ei 'warn|warning' ~/fire/logs/cron/morning-advisory.err`
   err 空 or warning のみ
6. `test -s /tmp/f062_morning_preview_$(date +%F).txt`
   当日 preview file 生成
7. `grep -R -Ei 'sent|line api|push message|broadcast' ~/fire/logs/cron/morning-advisory.* /tmp/f062_morning_preview_$(date +%F).txt`
   送信痕跡 0
8. `grep -R -Ei 'token|channel_token|secret|bearer' ~/fire/logs/cron/morning-advisory.* /tmp/f062_morning_preview_$(date +%F).txt`
   0 件
9. `stat -f '%Sm %N' ~/fire/data/fire.db ~/fire/data/fire.develop.db ~/fire/data/fire.staging.db`
   pre-flight baseline と一致
10. `sqlite3 ~/fire/data/fire.db 'select count(*) from advisory_decisions;'`
    pre-flight baseline と一致
11. `grep -Ei 'daily refresh.*OK|completed|success' ~/fire/logs/cron/daily-refresh.log`
    DATA-R3 当日朝完了
12. `grep -R 'chunk' ~/fire/logs/cron/morning-advisory.log | tail -20`
    max_chunks=1 前提から逸脱なし

## §6 abort / NO-GO 条件

Lane A §D 反映。試走中に検出したら即 unload + 調査 12 項目:

1. `runs` increment 0 → 起動失敗、`launchctl unload` 対象
2. `last exit code` non-zero → 即 NO-GO
3. log 不在 / mtime stale → 即 NO-GO
4. LINE 送信痕跡 (`sent`/`push`/`broadcast`/`line api`) → 即 unload + token rotate
5. token 露出 (`token`/`channel_token`/`secret`/`bearer`) → 即 unload + secret hygiene
6. 3 環境 DB mtime 変化 → 即 unload + 調査
7. `advisory_decisions` row 増加 → 即 unload + 調査 (= 設計違反)
8. DATA-R3 完了 log 不在なのに F062 preview 生成 → daily-refresh dependency 設計違反、NO-GO
9. F282 plist mtime 変化 → 即 unload + F282 整合性確認
10. chunk 数 > 1 検出 → send 0 継続、log ERROR、別 wave 調査
11. plist `EnvironmentVariables` に `LINE_*`/`CHANNEL_*`/`JQUANTS_*` 混入 → 配置/load しない
12. `HQ_APPROVE_LAUNCHD_MORNING_ADVISORY` 未取得で配置/load 試行 → Wave 45 停止

## §7 DATA-R3 daily refresh 依存条件

Lane A §E + Lane B graph 反映。試走中継続監視 10 項目:

1. `stat -f '%Sm %N'` で 3 plist mtime baseline 比較
   (= morning-advisory / daily-refresh / weekly-snapshot)
2. `ls -l ~/fire/logs/cron/*` で log path 独立性確認
3. `launchctl print gui/$(id -u)/jp.fire.daily-refresh | grep -E '06:30|Weekday'`
   DATA-R3 維持確認
4. `launchctl print gui/$(id -u)/jp.fire.morning-advisory | grep -E '08:45|Weekday'`
   F062 月-金 08:45 のみ確認
5. `launchctl print gui/$(id -u)/jp.fire.weekly-snapshot | grep -E '02:00|Saturday|Weekday'`
   F282 土 02:00 維持
6. `grep -R -Ei 'fire\.db|fire\.develop\.db|fire\.staging\.db' ~/fire/logs/cron/morning-advisory.log`
   で preview が DB target を触っていない確認
7. `stat -f '%Sm %N'` で 3 環境 DB 書き込み競合 0
8. `grep -R -Ei 'weekly-snapshot|daily-refresh' ~/fire/logs/cron/morning-advisory.*`
   不要な cross-job 呼び出し 0
9. `grep -R -Ei 'partial_delivery=True|chunk [2-9]|chunks=[2-9]' ~/fire/logs/cron/morning-advisory.*`
   0 件
10. F062 no-send trial 中は `run_f062_line_production_send_smoke.py` に
    送信用 marker を渡さず、runner-level 送信拒否 pattern 維持

### DATA-R3 完了条件 (= F062 起動前提)

F062 morning advisory 起動条件:
- 当日 06:30 起動の `jp.fire.daily-refresh` exit 0
- `logs/cron/daily-refresh.log` に当日朝の "OK" log line
- 4 sub-runner (F100/F101/F111/F119) いずれも `--dry-run` 経路完了
- staging DB が最新化済 (= W41 実装 sub-D2.2 後)

DATA-R3 NG なら F062 自体は launchd schedule で起動するが、内部で
freshness check NG → silent abort + exit 0 + report 出力。

## §8 Phase D no-send trial GO/NO-GO

Lane B §1 反映。

### GO 条件

- 5 営業日 全 PASS
- LINE 送信 0
- token 参照 0
- DB write 0 (= advisory_decisions row 不変)
- send_guard refuse 100% (= --hq-approved-send absent → 拒否)
- DATA-R3 完了 + freshness OK 連携動作確認
- exit 0
- F282 / DATA-R3 不干渉

### NO-GO 条件

- 任意項目失敗 → Wave 45 unload + 別 wave HQ 承認後再試走
- LINE 痕跡発見 → 即 unload + token rotate + 全停止
- DB write 検出 → 即 unload + 調査
- F282 試走干渉 → 即 unload + F282 整合性確認

## §9 Phase E Production v0 launch GO/NO-GO

Lane B §2 反映。

### GO 条件 ※ W44.5-post audit で marker 6/7 明示

- Phase D no-send trial GO 達成 (= marker 5、自然遷移)
- `HQ_APPROVE_LINE_TOKEN_PRODUCTION` 取得 (= marker 6) + LINE channel token production 投入完了 (= W52)
- `HQ_APPROVE_PRODUCTION_V0_LAUNCH` 取得 (= marker 7) (= W53 / 6/8 月夕方想定)
- send_guard pattern: `--hq-approved-send` marker 投入で送信許可 (= marker 7 後のみ)
- record-decisions DB write 経路確認 (= staging→production 移行)
- 5/14 軽量 wave (= 本 wave) 含む全 foundation 整合性確認
- v0 safety check 全 PASS (= v0 Launch Plan §4)

### NO-GO 条件

- Phase D NO-GO
- LINE token 漏洩疑い
- record-decisions duplicate 検出
- F282 / DATA-R3 / F062 干渉

## §10 HQ approve marker 順序

Lane B §3 反映。

| 順 | marker | wave | 役割 |
|---|---|---|---|
| 1 | (F282 GO 判定、自然遷移) | 5/19 | Wave 41 着手前提 |
| 2 | `HQ_APPROVE_LAUNCHD_DAILY` | Wave 41 | DATA-R3 plist 配置 + load |
| 3 | (DATA-R3 no-write 試走 GO) | Wave 41 完了 | Wave 45 前提 |
| 4 | `HQ_APPROVE_LAUNCHD_MORNING_ADVISORY` | Wave 45 | F062 plist 配置 + load |
| 5 | (no-send trial GO = Phase D GO) | Wave 45 完了 | Wave 52 前提 |
| 6 | `HQ_APPROVE_LINE_TOKEN_PRODUCTION` | Wave 52 | LINE token 投入 |
| 7 | `HQ_APPROVE_PRODUCTION_V0_LAUNCH` | Wave 53 | D-Day 朝 advisory 実送信 |

### 整理ノート (= Lane B 提示の整合性反映)

既存 v0 Launch Plan の `HQ_APPROVE_NO_SEND_TRIAL` は独立 marker として
扱わず、Wave 45 `HQ_APPROVE_LAUNCHD_MORNING_ADVISORY` 取得後の Phase D
GO 判定として自動連動させる。HQ marker は 7 段 (= 上記表) で固定。

### token 投入は最後段

LINE token は Wave 52 (= Phase D GO 後) で初めて env / secrets/ 構造に
投入する。no-send trial 期間中は token 参照 0 / env 全体読出 0 を厳守。

## §11 残課題

Lane B §6 反映。今後の wave で順次整理が必要:

- freshness gate 接続テストの具体的 test 関数定義 (= Wave 45 着手時)
- DATA-R3 no-write 試走中の freshness gate 動作確認手順
- LINE token 投入時の env 設定手順 (= secrets/ 構造、AppleScript 経由?)
- record-decisions staging→production 移行手順
- D-Day 当日のロールバック手順 (= 緊急 unload 経路)

優先度:
1. (高) D-Day 当日のロールバック手順 → Wave 51-52 で整理必須
2. (高) LINE token 投入時の env 設定手順 → Wave 52 で整理必須
3. (中) freshness gate 接続テスト → Wave 45 起票時に整理
4. (中) record-decisions 移行手順 → Wave 52 で整理
5. (低) DATA-R3 no-write 中の freshness gate 動作確認 → Wave 41 で整理

## §12 Wave 41 / Wave 45 / v0 launch への引き継ぎ

### Wave 41 (= 5/19 F282 GO 後着手)

- 既設計: `03_design/F286_DATA_R3_daily_refresh_launchd_2026-05-13.md`
- 必須 marker: `HQ_APPROVE_LAUNCHD_DAILY`
- 期間: 5/19 - 5/26 (= 1 週間 no-write 試走 + 5 営業日 daily check)
- 引き継ぐ checklist: §3 と類似構造で DATA-R3 用に派生
- F282 不干渉: §7 と同じ要領

### Wave 45 (= Wave 41 GO 後着手)

- 既設計: `03_design/F062_morning_advisory_launchd_2026-05-13.md`
- 必須 marker: `HQ_APPROVE_LAUNCHD_MORNING_ADVISORY`
- 期間: 5/26 - 6/2 (= plist 配置 + 1 週間 no-send 試走 + Phase D 判定)
- 直接利用する checklist: 本 doc §3 / §4 / §5 / §6 / §7
- Phase D GO 判定: §8

### Wave 52 (= Phase D GO 後着手)

- 必須 marker: `HQ_APPROVE_LINE_TOKEN_PRODUCTION`
- 期間: 6/8-6/9 (= D-Day 直前)
- LINE token を production 専用 env / secrets/ に投入
- record-decisions staging→production 経路確認
- v0 safety check 全 PASS 確認

### Wave 53 (= D-Day、2026-06-09 火曜) ※ W44.5-pre で曜日確定

- 必須 marker: `HQ_APPROVE_PRODUCTION_V0_LAUNCH`
- 朝 08:45 で初回 F062 advisory 実送信
- LINE 着信確認 (= Fujiwara)
- record-decisions 1 行追加確認
- log OK / F282 干渉 0 確認
- Phase E GO 判定

---

## 安全要件 (= 本 doc 自体)

| 項目 | 結果 |
|---|---|
| 実 file 変更 (本 doc 以外) | 0 |
| plist 配置 / 変更 | 0 |
| launchctl load / unload | 0 |
| DB write | 0 |
| LINE 送信 | 0 |
| token / secret / channel_token 参照 | 0 |
| env 全体読出 | 0 |
| 実 API call | 0 |
| F282 試走干渉 | 0 (= plist mtime 完全不変) |
| DATA-R3 設計干渉 | 0 |
| pytest 実行 | 0 (= 4090 PASS 静的維持) |
| F101 staging probe | 0 |
| VACUUM INTO | 0 |
| workflow / --no-verify / TODO Excel | 0 |

## 関連リンク

- [[FIRE_production_v0_launch_plan_2026-05-13|v0 Launch Plan v1.0]]
- [[F286_DATA_R3_daily_refresh_launchd_2026-05-13|DATA-R3 launchd 設計]]
- [[F062_morning_advisory_launchd_2026-05-13|F062 morning advisory 設計]]
- [[../02_todo/FIRE_CODEX_R1_WAVE40_5_plan|Wave 40.5 plan]]
- [[../02_todo/FIRE_CODEX_R1_WAVE40_5_results|Wave 40.5 results]]
- `/tmp/codex_wave40_5/results/lane_{a,b}.txt` (= 2 lane stdout)
